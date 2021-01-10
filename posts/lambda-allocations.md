Bir süre önce Twitter ve LinkedIn hesabımda <a href="https://marketplace.visualstudio.com/items?itemName=MukulSabharwal.ClrHeapAllocationAnalyzer" target="_blank">CLR Heap Allocation Analyzer</a> isimli bir Visual Studio extensionından bahsetmiştim. Bu extensionı kullanan çoğu kişi farkında olmadan yaptığı bazı allocationlardan kolayca kurtulabilirken özellikle tek bir noktada takılıp kalabiliyorlar. O da lambda expresionların neden olduğu allocationlar. Oldukça sinsice gerçekleşen bir allocation olduğu için farkedilmesi de kolay değil.

Şimdi diyelim ki aşağıdaki gibi bir senaryomuz var.

```csharp
class Test
{
    public void DoSomething(string[] args)
    {
        int i = 0;
        Run(() => i++);
    }

    public void Run(Action action)
    {
        action();
    }
}
```

`Run` metodu çalıştıracağı bir action parametresi bekliyor ve bunu çalıştırıyor. Bizde lambda ifadesi ile çalıştırmak istediğimiz kodu parametre olarak geçiyoruz. Bu lambda ifadesinde bir de ek olarak `DoSomething` metodu **içerisinde tanımladığımız bir değişkeni** kullanıyoruz. Şimdi bakalım compiler bu kodu nasıl çeviriyormuş. 

```csharp
internal class Test
{
    [CompilerGenerated]
    private sealed class <>c__DisplayClass0_0
    {
        public int i;

        internal void <DoSomething>b__0()
        {
            i++;
        }
    }

    public void DoSomething(string[] args)
    {
        <>c__DisplayClass0_0 <>c__DisplayClass0_ = new <>c__DisplayClass0_0();
        <>c__DisplayClass0_.i = 0;
        Run(new Action(<>c__DisplayClass0_.<DoSomething>b__0));
    }

    public void Run(Action action)
    {
        action();
    }
}
```
Lambda expressionları aslında runtime tarafında bilinen bir şey olmadığı için bir C# özelliği olarak kod derlenirken çeşitli çevrimler yapılıyor. Compiler arka planda genel adı `closure` olan yeni bir sınıf tanımlayıp yazdığımız kodu bu sınıfa metot olarak eklerken dışarıdan eriştiğimiz değişkeni de bir field olarak tanımlıyor. Main metodu içerisinde de bu sınıftan bir nesne yaratıp fielda değeri atayıp sonrasında da bir `Action` yaratıyor. Bu metodu eğer biz bir for döngüsü içerisinde çağırsaydık kullandığımız parametrenin yerine göre aslında döngünün çalışma sayısı kadar nesne yaratılma durumu olabilirdi. Bu da kısa süreli yaşayan çok fazla nesne yaratılmasına neden olarak GC üzerinde ekstra yük oluşturabilirdi. Bu durum yazdığımız normal uygulamalar için çok fazla sorun teşkil etmese de performansın kritik olduğu senaryolarda, çok fazla yük alan yerlerde soruna neden olabilir.

 Peki dışarıdan bir field kullanmadığımız senaryoyu düşünelim. O zaman bakalım nasıl olacak.

```csharp
class Test
{
    public void DoSomething(string[] args)
    {
        Run(() =>
        {
            var x = 0;
            for (int i = 0; i < 10; i++)
            {
                x++;
            }
            Console.WriteLine(x);
        });
    }

    public void Run(Action action)
    {
        action();
    }
}
```

Bu class derlendikten sonra...

```csharp
internal class Test
{
    [Serializable]
    [CompilerGenerated]
    private sealed class <>c
    {
        public static readonly <>c <>9 = new <>c();

        public static Action <>9__0_0;

        internal void <DoSomething>b__0_0()
        {
            int num = 0;
            for (int i = 0; i < 10; i++)
            {
                num++;
            }
            Console.WriteLine(num);
        }
    }

    public void DoSomething(string[] args)
    {
        Run(<>c.<>9__0_0 ?? (<>c.<>9__0_0 = new Action(<>c.<>9.<DoSomething>b__0_0)));
    }

    public void Run(Action action)
    {
        action();
    }
}
```

Gördüğünüz üzere lambda ifadesinde dışarıdan herhangi bir değişkene erişmediğimiz senaryolarda action, sınıf içerisindeki fieldda singleton olarak tutuluyor ve her çağırıldığında aslında aynı instance kullanılıyor. Burada allocationlardan büyük oranda kaçınabiliyoruz. 

Şimdi buraya kadar okuduktan sonra diyeceksiniz ki LINQ ifadeleri yazarken mutlaka dışarıdan bir parametre kullanıyoruz ve bunu optimize etmenin bir yolu yok mu? Bunun aslında bir yolu yok. Yani LINQ ifadeleri kullanıyorsanız o kolaylığın yanında bu allocationların da bedelini ödüyorsunuz. Bunu değiştirmenin pek bir yolu yok. Buradaki allocationlardan tamamen kaçınmak isterseniz klasik for/foreach döngülerini tercih etmeniz gerekmekte. Çoğu performans kritik projede (örneğin, roslyn, .net runtime,aspnet) özellkle hot path dediğimiz noktalarda LINQ kullanımının yasak olmasının nedeni bu. .NET tarafından örnek vermemiz gerekirse, .NET 6.0 ile beraber gelecek olan en önemli özelliklerden biri applerin startup zamanlarının iyileştirilmesi olacak. Bu doğrultuda yapılan değişikliklerden biri de uygulama ayağa kalkarken meydana gelen allocationlardan kurtulmak. Bu nedenle startup sırasında yapılan LINQ sorguları da klasik for/foreach döngülerine çevriliyor. İlgili değişiklikleri <a href="https://github.com/dotnet/runtime/issues/44598" target="_blank">buradan</a> inceleyebilirsiniz. 

Bizler kendi uygulamalarımızı geliştirirken bazı durumlarda lambda expressionları kabul eden metotlar vs.. yazmamız gerekebilir. Bu gibi durumları optimize etmenin hiçbir yolu yok mu diye sorarsanız aslında ufak bir optimizasyon yöntemi var. O da lambda ifadesi içerisinde kullanılacak parametreyi lambda expressiona parametre olarak geçmek. Biraz karışık geldi itiraf ediyorum :) Örnek üzerinde inceleyelim.

```csharp
public class Lambda
{
    public void Run(Action action)
    {
        action();
    }

    public void Call()
    {
        var foo = new Foo();
        Run(() => foo.MyProperty++);
    }
}
```
Şimdi yukarıda Run metodu çağırılırken `foo` nesnesi içerisindeki fielda erişildiği için allocationa neden oluyor. Peki bu metodu şu hale çevirsek nasıl olur.

```csharp
public class Lambda
{
    public void Run(Foo param, Action<Foo> action)
    {
        action(param);
    }

    public void Call()
    {
        var foo = new Foo();
        Run(foo, (param) => param.MyProperty++);
    }
}

public class Foo
{
    public int MyProperty { get; set; }
}
```

Bu durumda dışarıdan hiçbir değişkene erişmediğimiz için yukarıda bahsettiğimiz gibi ekstra allocationdan kurtulmuş olacağız daha optimize bir şekilde kodumuz çalışmış olacak. Tabi her zaman metodu çağıracak developerın `Foo` tipini kullanacağını bilemeyiz. Belki biraz daha generic hale getirebiliriz bu metodu.

```csharp
public class Lambda
{
    public void Run<TValue>(TValue state, Action<TValue> action)
    {
        action(state);
    }

    public void Call()
    {
        var foo = new Foo();
        Run(foo, (param) => param.MyProperty++);
    }
}

public class Foo
{
    public int MyProperty { get; set; }
}
```

Bu şekilde bir kullanımla en azından developerların bir tane tipi lambda expression içerisinde kullanmasını sağlamış oluyoruz. Böylece belki de çoğu senaryoda ekstra allocationlardan kurtulmuş oluyoruz. Bu bahsettiğim kullanım şu an .NET içerisindeki performans kritik olan metotlarda da bulunmakta. Örneğin, daha önce bahsettiğim `string.Create` <a href="https://ilkayilknur.com/string-create-metodu-nasil-kullanilir" target="_blank">metodunda</a> da bu şekilde bir implementasyon bulunmakta. 

Lambda expression kullandığımız yerlerde bizler bazı şeyleri farkında olarak dışarıdaki variablelara erişmekten kaçınabiliriz. Ancak bizden sonra kodu değiştirecek olan kişinin bu gibi şeyler gözünden kaçabilir. Bunun için de C# 9.0 ile beraber gelen static lambda özelliğini kullanabiliriz. 

Yukarıdaki gördüğümüz `Call` metodunu düşünürsek

```csharp
public void Call()
{
    var foo = new Foo();
    Run(foo, static (param) => param.MyProperty++);
}
```
Burada lambdanın başına static koyarak dışarından hiçbir değişkene erişilemeyeceğini garanti etmiş oluyoruz. Erişmek istersek de aşağıdaki gibi bir compiler hatası alıyoruz.

![static-lambda-error](https://az718566.vo.msecnd.net/uploads/2020/12/19/static-lambda.png)

Bu yazıda lambda expressionların neden olduğu sinsi allocationları kısaca inceledik. Bu yazıdan çıkan sonuç tabi ki de lambda expression kullanmayalım olmamalı :) Bu allocationları farkında olarak gerektiği durumlarda daha optimize kullanmak, bazı durumlara göre belki de hiç kullanmamak gerekebilir. Duruma göre değerlendirip kararlar alınabilir. Bazen bazı kolaylıklara sahip olmak için burada da gördüğümüz üzere bazı bedeller ödememiz gerekiyor. Ödemek istemezsek de en basit çözüm yardımımıza yetişiyor. 

Bir sonraki yazıda görüşmek üzere,