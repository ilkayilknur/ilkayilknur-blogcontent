Bir önceki [yazıda](https://www.ilkayilknur.com/foreach-dongulerinde-olusabilecek-allocationlar) `foreach` döngülerinde oluşabilecek olan allocationlardan bahsetmiştik. Bu yazıda ise `params` kullanımını incelleyeceğiz ve oluşabilecek allocationlara bakacağız. 

Değişen sayıda parametre kabul eden metotlarda `params` keywordünü kullanıp, metodu çağıranların parametreleri kolay bir şekilde sırayla geçmesini sağlayabiliyoruz.

Örnek yapmamız gerekirse...

```csharp
public void DoWork(params string[] parameters)
{
    foreach (var param in parameters)
    {
        Console.WriteLine(param);
    }
}
```

Yukarıda tanımladığımız metodu aşağıdaki gibi çeşitli şekillerde çağırabiliyoruz.

```csharp
DoWork();
DoWork("Ilkay");
DoWork("Ilkay","Osman","Mehmet","Ahmet");
```

Peki `params` keywordünü kullandığımızda compiler yukarıdaki çağrımları nasıl çeviriyor bir de ona bakalım.

İlk olarak `DoWork();` kullanımına baktığımızda parametresiz kullanım compiler tarafından aşağıdaki gibi çevriliyor. 

```csharp 
DoWork(Array.Empty<string>());
```

`Array.Empty<T>()` metodu her seferinde boş bir array yaratıyor algısı oluştursada aslında bu boş arrayleri tip bazında sadece bir kez yaratıp sonrasında cacheliyor. Dolayısıyla memory bakımından etkin bir kullanım sağlıyor. Eğer uygulamalarınızda array kabul eden metotlar vs.. varsa ve boş arrayi parametre olarak göndermeniz gerekiyorsa mutlaka `Array.Empty<T>` metodunu kullanmanızı tavsiye ederim. 

Her seferinde yeni array yaratan etkin olmayan çözüm.

```csharp
DoWork2(new int[0]);
```

Yukarıdaki etkin olmayan çözüm yerine kullanılması gereken çözüm.

```csharp
DoWork2(Array.Empty<int>());
```

Dolayısıyla bu noktada parametresiz kullanımda memory kullanımı konusunda çok problemli bir durum yok. Şimdi gelelim parametre geçtiğimiz kullanıma.

`DoWork("Ilkay","Osman","Mehmet","Ahmet");` metot çağırımı aşağıdaki gibi çevrilmekte.

```csharp
string[] array = new string[4];
array[0] = "Ilkay";
array[1] = "Osman";
array[2] = "Mehmet";
array[3] = "Ahmet";
DoWork(array);
```

Burada gördüğümüz üzere her çağrımda yeni bir array yaratılıyor ve sonrasında da array initialize edilip metoda parametre olarak geçiliyor. Dolayısıyla siz `DoWork` metodu içerisinde her ne kadar allocationdan kaçınırsanız kaçının sizin metodunuz çağırılırken ister istemez bir allocationa neden olunuyor. 

Şimdi gelin bu zamana kadar söylediklerimizi ufak bir benchmark yaparak kanıtlayalım. 

```csharp
[MemoryDiagnoser]
public class ParamsBenchmark
{
    [Benchmark]
    public int MultipleParameters()
    {
        return Sum(1, 2, 3, 4, 5, 6, 7, 8, 9);
    }

    [Benchmark]
    public int ZeroParameter()
    {
        return Sum();
    }

    public int Sum(params int[] arguments)
    {
        var sum = 0;
        foreach (var param in arguments)
        {
            sum += param;
        }

        return sum;
    }
}
```

Gördüğünüz üzere `Sum` metodu hiçbir allocationa sebep olmuyor. Biz bu `Sum` metodunu çağırarak çağırırken oluşacak olan allocationları ölçebiliyoruz.  Sonuçlara bakarsak.

```ini
BenchmarkDotNet=v0.12.1, OS=macOS 11.2.3 (20D91) [Darwin 20.3.0]
Intel Core i9-8950HK CPU 2.90GHz (Coffee Lake), 1 CPU, 12 logical and 6 physical cores
.NET Core SDK=5.0.201
  [Host]     : .NET Core 5.0.4 (CoreCLR 5.0.421.11614, CoreFX 5.0.421.11614), X64 RyuJIT
  DefaultJob : .NET Core 5.0.4 (CoreCLR 5.0.421.11614, CoreFX 5.0.421.11614), X64 RyuJIT

```

| Method             |      Mean |     Error |    StdDev |  Gen 0 | Gen 1 | Gen 2 | Allocated |
| ------------------ | --------: | --------: | --------: | -----: | ----: | ----: | --------: |
| MultipleParameters | 9.9199 ns | 0.1899 ns | 0.1683 ns | 0.0102 |     - |     - |      64 B |
| ZeroParameter      | 0.7491 ns | 0.0212 ns | 0.0199 ns |      - |     - |     - |         - |

Sonuçlardan da görebileceğiniz üzere çoklu parametreyle çağırdığımızda bir heap allocation oluşmakta. Bu noktada allocationlardan kaçınmanın yolu var mı derseniz bunun cevabını framework içerisindeki kullanımlara bakarak bulabiliriz.

`string.Format` metodununun overloadlarına bakalım. 

```csharp
public static string Format(string format, object? arg0)
public static string Format(string format, object? arg0, object? arg1)
public static string Format(string format, object? arg0, object? arg1, object? arg2)
public static string Format(string format, params object?[] args)
```

`string.Concat` metoduna da bir bakalım. 

```csharp
public static string Concat(object? arg0) => arg0?.ToString() ?? string.Empty;
public static string Concat(object? arg0, object? arg1)
public static string Concat(object? arg0, object? arg1, object? arg2)
public static string Concat(params object?[] args)
```

Yukarıdaki tanımlamalardan aslında yazacağımı çoktan tahmin etmiş olabilirsiniz. Bir optimizasyon yöntemi olarak çok sık kullanılacağını düşündüğünüz versiyonları, ayrı bir metot olarak tanımlayıp oluşabilecek olan allocationların çoğundan kaçınabilirsiniz.

Bir diğer alternatif olarak `params` kullanmak yerine `stackalloc` ile stack üzerinde array yaratıp bu arrayi parametre olarak geçebilirsiniz. Bu yöntemin tabi bazı limitleri var. Her senaryo için uygun değil.

Örnek olarak,

```csharp
public int Sum(Span<int> parameters)
{
    var sum = 0;
    foreach (var param in parameters)
    {
        sum += param;
    }

    return sum;
}
```

Metodu şu şekilde çağırabiliriz. 

```csharp
var sum = Sum(stackalloc int[9]
{
    1,2,3,4,5,6,7,8,9
});
```

Bu kullanımdaki limitler için tekrara düşmemek adına [Span<T>](https://www.ilkayilknur.com/net-coreda-span-ve-memory-tipleri) ve [stackalloc](https://www.ilkayilknur.com/stackalloc-ifadesi-nedir-nasil-kullanilir) ile yazdığım makalelerin linklerini bırakıyorum. Oradan daha detaylı bilgi alabilirsiniz.

Konuyu özetlersek, `params` kullanımında parametre olarak geçtiğimiz her durumda bir heap allocation olmakta. Compiler ne yazık ki şu ana kadar `stackalloc`, `Span<T>` gibi yapıları kullanarak daha az allocationa sebep olan bir çözüm sunmuyor. Bu konuyla ilgili bazı proposallar var ancak şu an için herhangi bir versiyona dahil edilmedi bu geliştirmeler. Bu nedenle allocationlardan kaçınmak istediğimiz noktalarda yukarıda bahsettiğim seçenekleri kullanabilirsiniz. 

Bir sonraki yazıda görüşmek üzere.



