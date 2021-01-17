Herkese Selamlar, 

Bu yazıda C# 8.0 ile beraber gelen ve çok az bilindiğini düşündüğüm indexing ve slicing operasyonlarında kullanabileceğimiz bir özelliği inceleyeceğiz. Tabi öncelikle programlama dili tarafındaki kolaylıklara geçmeden önce .NET Standart 2.1 ve .NET Core 3.0 ile beraber gelen `System.Index` ve `System.Range` tiplerine bir bakalım. 

 ```csharp
public readonly struct Index : IEquatable<Index>
{
    public Index(int value, bool fromEnd = false)
    {
     
    }
}
 ```

`System.Index` tipi bizim indeks bazlı işlemlerimizi kolaylaştırmak amacıyla getirilen bir tip. Bu tipi kullanarak kolay bir şekilde baştan veya sondan başlayarak indeksleme yapabiliyoruz ve bazı hesaplama kodlarından kurtulabiliyoruz. 

Örneğin baştan veya sondan belirli bir elemana ulaşmak istediğimizde aşağıdaki gibi bir kod yazabiliriz.

```csharp
var array = new[]
{
    1, 2, 3, 4, 5, 6, 7, 8, 9, 10
};

var secondElement = array[1];
var secondElementFromEnd = array[array.Length - 2];
```

Peki `System.Index` tipiyle bunu nasıl yapabiliriz bir de ona bakalım. 

```csharp
static void Main(string[] args)
{
    var array = new[]
    {
        1, 2, 3, 4, 5, 6, 7, 8, 9, 10
    };

    var secondElementIndex = Index.FromStart(1);
    var secondElement = array[secondElementIndex];
    var secondElementFromEndIndex = Index.FromEnd(2);
    var secondElementFromEnd = array[secondElementFromEndIndex];
}
```

Yukarıda görüldüğü gibi Index tipi içerisindeki yardımcı metotları kullanarak operasyonumuza uygun olan bir index yaratıp sonrasında da bu index'i normal bir indexer kullanımında olduğu gibi kullandık. Bu bizi bazı noktalarda hatalı kod yazmaktan kurtarabilir. Özellikle elimizdeki collectionın sonundan indeksleme yapmak istediğimizde hatalı kod yazma şansımız yüksek. Bu nedenle bu tipi kullanarak işlerimizi kolaylaştırabiliriz.



`System.Range` tipi ise yine yukarıda bahsettiğim .NET Standart ve .NET Core versiyonları ile beraber gelen bir diğer tip. Bu tip de bizim slicing operasyonlarımızı kolaylaştırmayı amaçlamakta.

```csharp
public readonly struct Range : IEquatable<Range>
{
    public Index Start { get; }

    public Index End { get; }

    public Range(Index start, Index end)
    {
        Start = start;
        End = end;
    }
}
```

Tanımlamasından da anlayabileceğiniz üzere Range tipi içerisinde başlangıç ve bitiş noktalarını bulunduran basit bir struct. 

Range kullanımını göstermek amacıyla array içerisinden ikinci elemanla altıncı eleman arasındaki bölgeyi almak istersek şu şekilde bir kod yazabiliriz. 

```csharp
var array = new[]
{
    1, 2, 3, 4, 5, 6, 7, 8, 9, 10
};

var range = new Range(1,6);
var slice = array[range];
```

Gördüğümüz üzere `System.Index` ve `System.Range` tipleri bizim işlerimizi bazı noktalarda kolaylaştırmakta. Ama farkındaysanız bu tanımlamaları bu şekilde boilerplate kod yazarak yapmak yerine özel bir syntax ile yapsak nasıl olurdu diye düşünmeden de edemiyoruz. İşte C# 8.0 ile beraber gelen hat(^) ve range(..) operatörlerini yukarıdaki operasyonları kısaltma amacıyla kullanabiliyoruz. 

Örneğin sondan ikinci elemana erişmek için yukarıda yazdığımız kod yerine aşağıdaki kodu yazabiliriz.

```csharp
var secondElementFromEnd = array[^2];
```

Range için kullandığımız örneği de şu şekilde değiştirmemiz mümkün.

```csharp
var slice = array[1..6];
```

Range operatörünün kullanım şekilleri biraz daha karmaşıklaştırılabilir tabi ki. 

```csharp
var slice = array[..6];
var slice = array[3..];
var slice = array[1..^2];
```

Yukarıdaki kullanımların hepsi geçerli kullanımlar. Senaryolarınıza göre bunları kullanabilirsiniz. Range operatörünü kullanırken memory allocation bakımından dikkat etmemiz gereken bir nokta var. Yukarıdaki şekilde bir kullanımda bulunduğumuzda her seferinde yeni bir array yaratılıp elemanlar bu arrayin içerisine kopyalanmakta. Bundan kaçınmak için range operatörünü `Span` ve `Memory` tipleriyle kullanıp memory allocationlardan kurtulabilirsiniz.

```csharp
var array = new[]
{
    1, 2, 3, 4, 5, 6, 7, 8, 9, 10
};

var slice = array.AsSpan()[3..];
```

Bu özelliği aynı zamanda string tipiyle de kullanmamız mümkün. Burada da compiler kodu otomatik olarak substring metoduna çevirmekte. Bu detayı da bilmekte fayda var. 

```csharp
var str = "This is a test!";
var range = str[1..^3];
```

Bu kod arka planda şuna çevriliyor. 

```csharp
string text = "This is a test!";
int length = text.Length;
int num = 1;
int length2 = length - 3 - num;
string text2 = text.Substring(num, length2);
```

C# 8.0 ile beraber gelen bu ufak özellikle daha okunabilir, daha az hataya açık kod yazabileceğimizi düşünüyorum.

Bir sonraki yazıda görüşmek üzere