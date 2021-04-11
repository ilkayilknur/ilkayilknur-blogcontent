Herkese Selamlar,

Bu yazıda konumuz `foreach` döngülerinin oluşturabileceği allocationlar.  Biraz ilginç bir konu. :) Konuya hemen basitçe giriş yapalım.

Bir `foreach` döngüsü yazdığımızda yazdığımız kod compiler tarafından yaklaşık olarak aşağıdaki gibi dönüştürülür. 

Foreach Döngüsü

```csharp
List<int> Collection = new List<int>();
foreach (var item in Collection)
{
    
}
```

 Çevrilmiş hali

```csharp
List<int> Collection = new List<int>();
List<int>.Enumerator enumerator = Collection.GetEnumerator();
try
{
    while (enumerator.MoveNext())
    {
    }
}
finally
{
    ((IDisposable)enumerator).Dispose();
}
```

Şimdi burada allocationa sebep olacak noktalara tek tek bakarsak karşımıza ilk olarak `Collection.GetEnumerator();`  metot çağrısı geliyor. Bu metot içerisinde çağırılan kod aşağıdaki gibi. Kaynak https://source.dot.net/#System.Private.CoreLib/List.cs,597

```csharp
 public Enumerator GetEnumerator()
            => new Enumerator(this);
```

`List<T>.Enumerator` tipine baktığımızda da bu tipin bir `struct` olduğunu görüyoruz. `System.Collections.Generic` namespace'i içerisinde bulunan diğer collectionlar için de bu durum geçerli. Yani tüm collectionların enumeratorları birer struct. Dolayısıyla burada heap allocation oluşturacak bir durum şu an için yok. Peki aşağıdaki gibi bir durumda değişen ne olacak bir de ona bakalım. 

```csharp
IEnumerable<int> Collection = new List<int>();
foreach(var i in Collection)
{
  
}
```
Buradaki kod compiler tarafından aşağıdaki gibi çevriliyor. 

```csharp
IEnumerable<int> Collection = new List<int>();
IEnumerator<int> enumerator = ((IEnumerable<int>)Collection).GetEnumerator();
try
{
  while (enumerator.MoveNext())
  {
    Console.WriteLine(enumerator.Current);
  }
}
finally
{
  if (enumerator != null)
  {
    enumerator.Dispose();
  }
}
```

`List<T>` tipi içerisinde `IEnumerable<T>.GetEnumerator` metodunun tanımlaması ise şu şekilde.  Kaynak https://source.dot.net/#System.Private.CoreLib/List.cs,600

```csharp
IEnumerator<T> IEnumerable<T>.GetEnumerator()
            => new Enumerator(this);
```

Burada da aslında yeni bir `Enumerator` instance'ı yaratılıp dönülüyor. Sorun gözükmüyor gibi duruyor ama arada şöyle bir fark var. `Enumerator` structı `IEnumerator<T>`'ye cast ediliyor. Dolayısıyla burada bir boxing, heap allocation bulunmakta.

Şu ana kadar incelediğimiz kısmı özetlersek, eğer foreach döngüsünü tiplerin kendisi üzerinden çalıştırırsak compiler bu durumda doğru `Enumerator` tipini buluyor ve heap allocationa neden olmadan `Enumerator` 'ı elde edebiliyor. Ancak belirli bir interface üzerinden aynı `foreach` döngüsü çalıştırıldığında compiler enumeratorı  `IEnumerator<T>`'ye cast etmesi gerektiği için bir heap allocation oluşuyor.

Şimdi gelelim finally bloğuna.

```csharp
finally
{
    ((IDisposable)enumerator).Dispose();
}
```

Burada da baktığımızda bir `IDisposable` castingi var. Bu da görünürde bir allocationa neden olmakta. Ancak [buradan](https://ericlippert.com/2011/03/14/to-box-or-not-to-box/) detaylarını okuyabileceğiniz üzere C# compilerı bu noktada özel bir optimizasyon yapmakta ve value type üzerinde expose edilen bir `Dispose` metodu varsa bu metodu casting yapmadan doğrudan çağırmak. Bu nedenle de bu noktada bir heap allocation oluşmamakta.  

Kısaca tüm senaryoyu özetlersek eğer bir collection tipi üzerinden `foreach` döngüsü yazarsanız bu döngü bir allocationa neden olmamakta. Ancak interface üzerinden yazarsanız bir allocation oluşmakta. 

Şimdi bu noktaya kadar çıkarımlarımızı hep kod okuyarak yaptık. Ancak performans ve memory kullanımı ile ilgili bir şey iddia ediyorsanız bunu benchmark yazarak kanıtlamanız gerekiyor. Şimdi gelelim benchmarklara.

```csharp
[MemoryDiagnoser]
[MarkdownExporter]
public class ForeachBenchmark
{
    private IEnumerable<int> IEnumerable;
    private List<int> List;

    [GlobalSetup]
    public void Setup()
    {
        List = new List<int>()
        {
            1, 2, 3, 4, 5, 6, 7, 8, 9, 10
        };
        
        IEnumerable = new List<int>()
        {
            1, 2, 3, 4, 5, 6, 7, 8, 9, 10
        }; 
    }

    [Benchmark]
    public int ForeachOverList()
    {
        var sum = 0;
        foreach (var item in List)
        {
            sum += item;
        }

        return sum;
    }
    
    [Benchmark]
    public int ForeachOverIEnumerable()
    {
        var sum = 0;
        foreach (var item in IEnumerable)
        {
            sum += item;
        }

        return sum;
    }
}
```

Çok basit olarak iki tane metot yazdık. Bu metotlar liste içerisindeki elemanların toplamını dönmekte. Şimdi sonuçlara bakalım. 

```ini
BenchmarkDotNet=v0.12.1, OS=macOS 11.2.3 (20D91) [Darwin 20.3.0]

Intel Core i9-8950HK CPU 2.90GHz (Coffee Lake), 1 CPU, 12 logical and 6 physical cores

.NET Core SDK=5.0.201

  [Host]     : .NET Core 5.0.4 (CoreCLR 5.0.421.11614, CoreFX 5.0.421.11614), X64 RyuJIT

  DefaultJob : .NET Core 5.0.4 (CoreCLR 5.0.421.11614, CoreFX 5.0.421.11614), X64 RyuJIT
```

| Method                 | Mean      | Error    | StdDev   | Gen 0  | Gen 1 | Gen 2 | Allocated |
| ---------------------- | --------- | -------- | -------- | ------ | ----- | ----- | --------- |
| ForeachOverList        | 29.18 ns  | 0.761 ns | 2.231 ns | -      | -     | -     | -         |
| ForeachOverIEnumerable | 101.04 ns | 2.704 ns | 7.845 ns | 0.0063 | -     | -     | 40 B      |

Görüldüğü üzere `IEnumerable` üzerinden çalıştırdığımız foreach döngüsünde bir heap allocation oluşmakta. Bu allocation tabi ki çok büyük bir allocation değil. Yani toptan tüm foreach kullanımlarımızı değiştirelim gibi bir sonuç çıkarmak doğru değil. Ancak hot path dediğimiz noktalarda allocationlardan kaçınmak istediğimiz durumlarda bu noktalar göz önünde bulundurulabilir. 

Bir sonraki yazıda görüşmek üzere.