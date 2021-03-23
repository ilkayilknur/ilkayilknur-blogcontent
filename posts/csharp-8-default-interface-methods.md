Herkese Selamlar,

Bu yazıda C# 8.0 ile beraber ile gelen default interface metotları özelliğini inceleyeceğiz. Bu özelliği incelemeden önce varolan interface yapısını da kısaca inceleyip aradaki farkları öyle görmemizde fayda var.

Default metotların olmadığı yani C# 7.0 versiyonunda bir interface içerisinde tanımladığımız üyeler her zaman `abstract` ve `public` olma özelliğini taşıyordu. 

Örneğin,
```csharp
interface IA
{
    void DoWork();
}
```
Yukarıdaki gibi bir interface tanımladığımızda bu interface'i implemente edecek olan tipler `DoWork` metodunu da implemente etmek zorundalar. İlerleyen dönemlerde eğer bu `IA` interface'i içerisine ihtiyaca göre yeni metotlar veya başka üyeler eklenirse implemente eden taraflar da mutlaka bu eklemelere göre implementasyonlarını güncellemek zorundalar. Bu durumda aslında interface üzerinde yapılan tüm değişiklikler implemente edenler tarafında aksiyon alınması gereken değişikliklere dönüşmekte.

Peki default interface metot özelliği ile neler değişiyor diye bakarsak, pek çok değişikliğin olduğunu göreceğiz. Ancak ilk olarak büyük yenilikten başlayalım ve sonra detaylı olarak diğer değişikliklerede değinelim. 

C# 8.0 ile beraber interface içerisine metot tanımlaması yaptıktan sonra içerisinde default implementasyonunu da yapabiliyoruz. Böylece yeni eklenen üyeler için interface'i implemente eden tipler içerisinde aksiyon alınması gereken durumlar olmuyor. Bu şekilde yeni eklenen üyeler interface'i implemente eden taraflar "rahatsız" edilmeden kullanılabiliyor.

Örneğin,
```csharp
interface IA
{
    void DoWork();

    void LogToConsole()
    {
        Console.WriteLine("Logging to console...");
    }
}
```

Yukarıda gördüğünüz gibi `LogToConsole` isimli bir metot tanımlayıp bu metodun default implementasyonunu belirtebiliyoruz. Bu şekilde `IA` interface'ini implemente eden yerler eğer bu metodu override etmek istemezlerse herhangi bir şey yapmalarına gerek kalmıyor. 

```csharp
public class Test : IA
{
    public void DoWork()
    {
        Console.WriteLine("Doing the super important job...");
    }
}
```
Şimdi gelelim bu metodu çağırma kısmına...

```csharp
static void Main(string[] args)
{
    IA test = new Test();
    test.LogToConsole();
}
```
Yukarıda gördüğünüz gibi default metot implementasyonunu çağırabiliyoruz. Peki bir de farklı bir kullanım olarak aşağıdaki gibi çağırmayı denersek...

![](https://az718566.vo.msecnd.net/uploads/2021/03/07/default-interface-error.png)

Bu şekilde baktığımızda aslında default metotların inherit edilmediğini görüyoruz. Bu metotları çağırmak için ilgili tipi interface'e cast etmemiz gerekiyor.

Default metotları özelleştirmek isterseniz implemente ettiğiniz tip veya interface içerisinde bunu yapmanız mümkün. Yukarıdaki şekilde interface'e cast edip kullandığınızda metodun en son implemente edilen versiyonu kullanılacaktır. 

Örneğin,

```csharp
interface IA
{
    void DoWork();

    public void LogToConsole()
    {
        Console.WriteLine("Logging to console...");
    }
}

public class Test : IA
{
    public void DoWork()
    {
        Console.WriteLine("Doing the super important job...");
    }

    void IA.LogToConsole()
    {
        Console.WriteLine("Logging to console in test...");
    }
}
```

Yukarıdaki kodu aşağıdaki gibi bir kodla çalıştırırsak...

```csharp
static void Main(string[] args)
{
    IA test = new Test();
    test.LogToConsole();
}
```

Console'da aşağıdaki çıktıyı göreceğiz.

```bash
Logging to console in test...
```
Biraz daha karışık bir örnek yaparsak.
```csharp
interface IA
{
    void X()
    {
        Console.WriteLine("Calling default implementation");
    }
}

interface IB : IA
{
    void IA.X()
    {
        Console.WriteLine("Calling the implementation on IB");
    }
}

interface IC : IB
{
    void IA.X()
    {
        Console.WriteLine("Calling the implementation on IC");
    }
}

class Test : IC
{

}
```

Bu senaryoda aşağıdaki şekilde bir kullanımda bulunursak...

```csharp
static void Main(string[] args)
{

    IA test = new Test();
    test.X();
    IB test2 = new Test();
    test2.X();
    IC test3 = new Test();
    test3.X();
}
```

Sonuç aşağıdaki en spesifik implementasyon `IC` interface'inde yapıldığı için aşağıdaki gibi olacak. Aynı zamanda runtimeda en spesifik override alındığı için diamond problemi de olmamakta. 

```bash
Calling the implementation on IC
Calling the implementation on IC
Calling the implementation on IC
```

Şimdi gelelim C# 8.0 ile beraber interfacelere gelen diğer yeniliklere. C# 8.0 ile beraber interface üyeleri `private, protected, internal, public, virtual, abstract, sealed, static, extern` modifierlarına sahip olabiliyorlar.

Static üyelerin kullanımına kısaca bakarsak... 
```csharp
interface IA
{
    protected static int X;

    public static void SetX(int x)
    {
        X = x+10;
    }

    void LogToConsole()
    {
        Console.WriteLine(X);
    }
}

public class Test : IA
{

}
```
Yukarıda gördüğümüz gibi static değişken ve metotlar tanımlayabiliyoruz interfaceler içerisinde. Bu metotlarıda aşağıdaki gibi çağırabiliriz. 

```csharp
static void Main(string[] args)
{
    IA.SetX(100);
    IA test = new Test();
    test.LogToConsole();
}
```
Yukarıdaki kodu çalıştırdığımızda consoleda `110` yazısını göreceğiz. Static interface üyelerinin interfaceleri parameterize etmede nasıl kullanılabileceği ile ilgili detaylı bilgiyi <a href="https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/tutorials/default-interface-methods-versions#provide-parameterization" target="_blank">buradan</a> alabilirsiniz.

C# 8.0 ile beraber gelen en değişik özelliklerden biri kesinlikle default interface metotları. Java <a href="https://docs.oracle.com/javase/tutorial/java/IandI/defaultmethods.html" target="_blank">tarafında</a> da uzun zamandan beri aslında aynı özellik bulunmakta. C# 8.0 ile beraber C# tarafına eklenmiş oldu. İmplemente eden tarafları kırmadan interfaceleri genişletebilmek özellikle library geliştirenler için iyi bir özellik olacaktır. 

Bir sonraki yazıda görüşmek üzere,
