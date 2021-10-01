.NET içerisinde uzun süredir varolan ve sıklıkla kullanılan özelliklerden biri de LINQ. Geçtiğimiz versiyonlarda LINQ tarafına pek yeni özellik eklenmese de performans ve memory kullanımı bakımından pek çok iyileştirme yapılmıştı. .NET 6 ile beraber ise LINQ tarafında uzun süredir istenilen bazı API'lar bu versiyonla beraber hayatımıza giriyor olacak. Şimdi lafı çok fazla uzatmadan yeni gelecek LINQ API'larını inceleyelim.


## Index Ve Range Desteği

Daha önce blogda C# 8 ile beraber gelen <a href="https://www.ilkayilknur.com/csharp-8-0-ile-daha-kolay-indeksleme-ve-slicing" target="_blank">Index ve Range desteğinden</a> bahsetmiştim. .NET 6 ile beraber index ve range desteği çeşitli LINQ API'larına da ekleniyor. 

Eklenen metotlara kısaca bir bakalım. 

```csharp
namespace System.Linq
{
    public static partial class Enumerable
    {
        public static TSource ElementAt<TSource>(this IEnumerable<TSource> source, Index index);
        public static TSource ElementAtOrDefault<TSource>(this IEnumerable<TSource> source, Index index);
        public static IEnumerable<TSource> Take<TSource>(this IEnumerable<TSource> source, Range range);
    }
}
```
```csharp
namespace System.Linq
{
    public static partial class Queryable
    {
        public static TSource ElementAt<TSource>(this IQueryable<TSource> source, Index index);
        public static TSource ElementAtOrDefault<TSource>(this IQueryable<TSource> source, Index index);
        public static IQueryable<TSource> Take<TSource>(this IQueryable<TSource> source, Range range);
    }
}
```

Ufak bir örnek yapalım.

```csharp
var list = new List<int>()
{
    1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12
};

var elements = list.ElementAt(^2);
var elements2 = list.Take(2..);
```

## Default Değer Belirtmeyi Sağlayan SingleOrDefault, LastOrDefault ve FirstOrDefault Metot Overloadları 

.NET 6 ile beraber `SingleOrDefault`, `LastOrDefault` ve `FirstOrDefault` metotlarını kullandığımızda ek olarak kendi istediğimiz bir default değeri dönmesini sağlayabiliyoruz. 

Eklenen API'lar şu şekilde.

```csharp
namespace System.Linq
{
    public static class Enumerable
    {
        public static TSource SingleOrDefault<TSource>(this IEnumerable<TSource> source, TSource defaultValue);
        public static TSource SingleOrDefault<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate, TSource defaultValue);

        public static TSource FirstOrDefault<TSource>(this IEnumerable<TSource> source, TSource defaultValue);
        public static TSource FirstOrDefault<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate, TSource defaultValue);

        public static TSource LastOrDefault<TSource>(this IEnumerable<TSource> source, TSource defaultValue);
        public static TSource LastOrDefault<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate, TSource defaultValue);
    }

    public static class Queryable
    {
        public static TSource SingleOrDefault<TSource>(this IQueryable<TSource> source, TSource defaultValue);
        public static TSource SingleOrDefault<TSource>(this IQueryable<TSource> source, Func<TSource, bool> predicate, TSource defaultValue);

        public static TSource FirstOrDefault<TSource>(this IQueryable<TSource> source, TSource defaultValue);
        public static TSource FirstOrDefault<TSource>(this IQueryable<TSource> source, Func<TSource, bool> predicate, TSource defaultValue);

        public static TSource LastOrDefault<TSource>(this IQueryable<TSource> source, TSource defaultValue);
        public static TSource LastOrDefault<TSource>(this IQueryable<TSource> source, Func<TSource, bool> predicate, TSource defaultValue);
    }
}
```

Bir örnek yapalım. 

```csharp
var list = new List<int>()
{
    1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12
};

var firstElement = list.FirstOrDefault(-1); //returns 1
var element = list.FirstOrDefault(t => t > 13, -1); //returns -1 instead of default int
```

## Distinct, Except, Intersect, Union, Min ve Max İçin *By Metotları

.NET 6 ile beraber yeni eklenecek API'ların bir diğeri de `keySelector` belirtmemizi sağlayan *By metotları.

İlk olarak eklenen metotları görelim.

```csharp
namespace System.Linq
{
    public static class Enumerable
    {
        public static IEnumerable<TSource> DistinctBy<TSource, TKey>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector);
        public static IEnumerable<TSource> DistinctBy<TSource, TKey>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector, IEqualityComparer<TKey>? comparer);
        
        public static IEnumerable<TSource> ExceptBy<TSource, TKey>(this IEnumerable<TSource> first, IEnumerable<TSource> second, Func<TSource, TKey> keySelector);
        public static IEnumerable<TSource> ExceptBy<TSource, TKey>(this IEnumerable<TSource> first, IEnumerable<TSource> second, Func<TSource, TKey> keySelector, IEqualityComparer<TKey>? comparer);
        public static IEnumerable<TSource> ExceptBy<TSource, TKey>(this IEnumerable<TSource> first, IEnumerable<TKey> second, Func<TSource, TKey> keySelectorFirst);
        public static IEnumerable<TSource> ExceptBy<TSource, TKey>(this IEnumerable<TSource> first, IEnumerable<TKey> second, Func<TSource, TKey> keySelectorFirst, IEqualityComparer<TKey>? comparer);

        public static IEnumerable<TSource> IntersectBy<TSource, TKey>(this IEnumerable<TSource> first, IEnumerable<TSource> second, Func<TSource, TKey> keySelector);
        public static IEnumerable<TSource> IntersectBy<TSource, TKey>(this IEnumerable<TSource> first, IEnumerable<TSource> second, Func<TSource, TKey> keySelector, IEqualityComparer<TKey>? comparer);
        public static IEnumerable<TSource> IntersectBy<TSource, TKey>(this IEnumerable<TSource> first, IEnumerable<TKey> second, Func<TSource, TKey> keySelectorFirst);
        public static IEnumerable<TSource> IntersectBy<TSource, TKey>(this IEnumerable<TSource> first, IEnumerable<TKey> second, Func<TSource, TKey> keySelectorFirst, IEqualityComparer<TKey>? comparer);
        
        public static IEnumerable<TSource> UnionBy<TSource, TKey>(this IEnumerable<TSource> first, IEnumerable<TSource> second, Func<TSource, TKey> keySelector);
        public static IEnumerable<TSource> UnionBy<TSource, TKey>(this IEnumerable<TSource> first, IEnumerable<TSource> second, Func<TSource, TKey> keySelector, IEqualityComparer<TKey>? comparer);
        
        public static TSource MinBy<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, TResult> selector);
        public static TSource MinBy<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, TResult> selector, IComparer<TResult>? comparer);
        
        public static TSource MaxBy<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, TResult> selector);
        public static TSource MaxBy<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, TResult> selector, IComparer<TResult>? comparer);
        
        // Missing min & max overloads accepting custom comparers added for completeness
        public static TResult Min<TSource, TResult>(this IEnumerable<TSource> source, IComparer<TResult>? comparer);
        public static TResult Max<TSource, TResult>(this IEnumerable<TSource> source, IComparer<TResult>? comparer);
    }

    public static class Queryable
    {
        public static IQueryable<TSource> DistinctBy<TSource, TKey>(this IQueryable<TSource> source, Expression<Func<TSource, TKey>> keySelector);
        public static IQueryable<TSource> DistinctBy<TSource, TKey>(this IQueryable<TSource> source, Expression<Func<TSource, TKey>> keySelector, IEqualityComparer<TKey>? comparer);

        public static IQueryable<TSource> ExceptBy<TSource, TKey>(this IQueryable<TSource> source1, IEnumerable<TSource> source2, Expression<Func<TSource, TKey>> keySelector);
        public static IQueryable<TSource> ExceptBy<TSource, TKey>(this IQueryable<TSource> source1, IEnumerable<TSource> source2, Expression<Func<TSource, TKey>> keySelector, IEqualityComparer<TKey>? comparer);
        public static IQueryable<TSource> ExceptBy<TSource, TKey>(this IQueryable<TSource> source1, IEnumerable<TKey> source2, Expression<Func<TSource, TKey>> keySelectorFirst);
        public static IQueryable<TSource> ExceptBy<TSource, TKey>(this IQueryable<TSource> source1, IEnumerable<TKey> source2, Expression<Func<TSource, TKey>> keySelectorFirst, IEqualityComparer<TKey>? comparer);

        public static IQueryable<TSource> IntersectBy<TSource, TKey>(this IQueryable<TSource> source1, IEnumerable<TSource> source2, Expression<Func<TSource, TKey>> keySelector);
        public static IQueryable<TSource> IntersectBy<TSource, TKey>(this IQueryable<TSource> source1, IEnumerable<TSource> source2, Expression<Func<TSource, TKey>> keySelector, IEqualityComparer<TKey>? comparer);
        public static IQueryable<TSource> IntersectBy<TSource, TKey>(this IQueryable<TSource> source1, IEnumerable<TKey> source2, Expression<Func<TSource, TKey>> keySelectorFirst);
        public static IQueryable<TSource> IntersectBy<TSource, TKey>(this IQueryable<TSource> source1, IEnumerable<TKey> source2, Expression<Func<TSource, TKey>> keySelectorFirst, IEqualityComparer<TKey>? comparer);

        public static IQueryable<TSource> UnionBy<TSource, TKey>(this IQueryable<TSource> source1, IEnumerable<TSource> source2, Expression<Func<TSource, TKey>> keySelector);
        public static IQueryable<TSource> UnionBy<TSource, TKey>(this IQueryable<TSource> source1, IEnumerable<TSource> source2, Expression<Func<TSource, TKey>> keySelector, IEqualityComparer<TKey>? comparer);

        public static TSource MinBy<TSource, TResult>(this IQueryable<TSource> source, Expression<Func<TSource, TResult>> selector);
        public static TSource MinBy<TSource, TResult>(this IQueryable<TSource> source, Expression<Func<TSource, TResult>> selector, IComparer<TResult>? comparer);

        public static TSource MaxBy<TSource, TResult>(this IQueryable<TSource> source, Expression<Func<TSource, TResult>> selector);
        public static TSource MaxBy<TSource, TResult>(this IQueryable<TSource> source, Expression<Func<TSource, TResult>> selector, IComparer<TResult>? comparer);

        // Missing min & max overloads accepting custom comparers added for completeness
        public static TResult Min<TSource, TResult>(this IQueryable<TSource> source, IComparer<TResult>? comparer);
        public static TResult Max<TSource, TResult>(this IQueryable<TSource> source, IComparer<TResult>? comparer);
    }
}
```

Bu API'ların neden geldiğini basitçe `Distinct` API'ı üzerinden açıklarsak, `Distinct` API'ını kullandığımızda bu API default equality comparer'ı kullanarak distinct elemanlı buluyor. Eğer complex bir type üzerinden bu metodu kullanmak istersek tipimizin `IEquatable<T>` interface'ini implemente etmesi gerekmekte. `DistictBy` metoduyla bu zorunluluğu ortadan kaldırıp metoda parametre olarak `keySelector` geçebiliyoruz. 

Örnek yaparsak

```csharp
var list = new List<Product>();
var distinctProducts = list.DistinctBy(t => t.SKU);
var cheapestProduct = list.MinBy(t => t.Price);
```

## Chunk Metodu

.NET 6 ile beraber gelen en beğendiğim API'lardan biri de `Chunk` API'ı. Elimizde bulunan bir `IEnumerable<T>` veya `IQueryable<T>`'den belirttiğimiz uzunlukta chunklar oluşturabiliyoruz. 

Eklenen API'lar
```csharp
namespace System.Linq
{
    public static class Enumerable
    {
        public static IEnumerable<T[]> Chunk(this IEnumerable<T> source, int size);
    }
    public static class Queryable
    {
        public static IQueryable<T[]> Chunk(this IQueryable<T> source, int size);
    }
}
```

Örnek yaparsak.

```csharp
var list = new List<int>
{
    1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13
};

IEnumerable<int[]>? chunks = list.Chunk(3);
/*
Chunks:
[1,2,3]
[4,5,6]
[7,8,9]
[10,11,12]
[13]
*/
```

## Zip Metoduyla Üç Farklı Sequence'i Birleştirme Olanağı

.NET 6 ile beraber `Zip<T>` metodunu kullanarak üç farklı sequence'ı birleştirebiliyoruz. 

Gelen yeni API'lar şu şekilde.

```csharp
namespace System.Linq
{
    public static class Enumerable
    {
        public static IEnumerable<(TFirst First, TSecond Second, TThird Third)> Zip<TFirst, TSecond, TThird>(this IEnumerable<TFirst> first, IEnumerable<TSecond> second, IEnumerable<TThird> third);
    }

    public static class Queryable
    {
        public static IQueryable<(TFirst First, TSecond Second, TThird Third)> Zip<TFirst, TSecond, TThird>(this IQueryable<TFirst> source1, IEnumerable<TSecond> source2, IEnumerable<TThird> source3);
    }
}
```

## `TryGetNonEnumeratedCount` Metodu

`TryGetNonEnumeratedCount` metodu elimizde bulunan sequence'in enumerate etmeye zorlamadan boyutunu almayı sağlıyor. Bu metot özellikle bir sequence'i enumerate etmeden önce buffer allocate etmek istersek önceden sequence'in büyüklüğünü bulup daha efektif bir şekilde buffer allocate etmemize yardımcı olabilir. 

Eklenen API

```csharp
namespace System.Linq
{
    public static class Enumerable
    {
        public static bool TryGetNonEnumeratedCount(this IEnumerable<T> source, out int count);
    }
}
```

Örnek kullanım.

```csharp
List<T> buffer = source.TryGetNonEnumeratedCount(out int count) ? new List<T>(capacity: count) : new List<T>();
foreach (T item in source)
{
    buffer.Add(item);
}
```

Bu yazıda da .NET 6 ile beraber LINQ tarafında gelen yenilikleri inceledik. 

Bir sonraki yazıda görüşmek üzere.