Bu yazıda .NET Core 3.0 ile beraber gelen, asenkron streaming yapmamıza olanak sağlayan `IAsyncEnumerable<T>` interface'ini inceleyeceğiz. Aslında bu interface'in senkron versiyonu olan `IEnumerable<T>` interface'ine çok da yabancı değiliz. 

Hatırlamak amacıyla ufak bir örnek yaparsak,

```csharp
public IEnumerable<string> ReadLines(string filePath)
{
    using var stream = File.OpenRead(filePath);
    var reader = new StreamReader(stream);
    while (true)
    {
        var line = reader.ReadLine();
        if (line == null)
        {
            break;
        }
        yield return line;
    }
}
```
Bu şekilde `IEnumerable<T>` dönen bir iterator metot tanımladıktan sonra bu metodu aşağıdaki gibi bir `foreach` döngüsüyle kullanabiliyoruz. 

```csharp
foreach (var line in ReadLines(filePath))
{

}
```
.NET platformunun da gelişmesiyle beraber en çok öne çıkan konseptlerden biri şüphesiz ki asenkron programlama. Özellikle ölçeklenebilir, yüksek performanslı uygulama geliştirebilmek için olmazsa olmazlardan. Bu nedenle zaman geçtikçe asenkron API'ların da artmasıyla bazı ihtiyaçlar doğmaya başladı ve mevcut yapılara yeni eklemeler yapılması gerekti. Bu eklemelerden biri de asenkron iteration metotları yazabilmemizi ve bu metotları kullanabilmemizi sağlamak amacıyla yapıldı. Artık `IAsyncEnumerable<T>` interface'ini kullanarak asenkron iterasyon metotları yazabiliyoruz ve bu metotları da `await foreach` ile kullanabiliyoruz. 

Şimdi kısaca `IAsyncEnumerable<T>` interface'inin içerisindeki üyelere bakarsak.

```csharp
public interface IAsyncEnumerable<out T>
{
    IAsyncEnumerator<T> GetAsyncEnumerator(CancellationToken cancellationToken = default);
}
```

Bir de `IAsyncEnumerator<T>`'a bakalım. 

```csharp
public interface IAsyncEnumerator<out T> : IAsyncDisposable
{
    T Current { get; }
    ValueTask<bool> MoveNextAsync();
}
```

Aslında baktığımızda senkron versiyonları olan `IEnumerable<T>`, `IEnumerator<T>` 'dan da çok farklı bir tarafı yok. `IAsyncEnumerator<T>` içerisindeki `MoveNextAsync` metodunun performans ve allocation açısından <a href="https://www.ilkayilknur.com/peki-nedir-bu-valuetask" target="_blank">ValueTask</a> dönmesi güzel bir detay.

Şimdi yukarıdaki örneği async hale getirmek için ne gibi değişiklikler yapmamız gerekiyor ona bakalım. Öncelikle `StreamReader` içerisindeki `ReadLine` metodunu kullanmak yerine bu metodun asenkron versiyonunu kullanabiliriz. Sonrasında da metodun dönüş tipini `IAsyncEnumerable<T>`'ye çevirebiliriz. 

```csharp
public async IAsyncEnumerable<string> ReadLinesAsync(string filePath)
{
    using var stream = File.OpenRead(filePath);
    using var reader = new StreamReader(stream);
    while (true)
    {
        var line = await reader.ReadLineAsync();
        if (line == null)
        {
            break;
        }

        yield return line;
    }
}
```
Yukarıdaki metodu foreach ile kullanmak istersek
```csharp
await foreach (var line in ReadLinesAsync(filePath))
{

}
```
### Peki CancellationToken? 
Async streaming senaryosunda önemli noktalardan biri de cancellation senaryosu. Cancellation tokenı foreach içerisinde nasıl parametre olarak geçebiliriz sorusu aklımıza gelebilir. Öncelikle iterator metot tarafına bakarsak parametre olarak cancellation tokenı alıp bu parametreyi de `[EnumeratorCancellation]` metoduyla işaretlememiz gerekiyor.

```csharp
public async IAsyncEnumerable<string> ReadLinesAsync(string filePath, [EnumeratorCancellation] CancellationToken token=default)
{
    using var stream = File.OpenRead(filePath);
    using var reader = new StreamReader(stream);
    while (true)
    {
        var line = await reader.ReadLineAsync();
        if (line == null)
        {
            break;
        }

        yield return line;
        token.ThrowIfCancellationRequested();
    }
}
```
> Yukarıdaki örnekte parametre olarak aldığınız tokenı asenkron metotlara parametre olarak geçebilirsiniz. Ancak yukarıdaki örnekte `ReadLineAsync` parametre olarak `CancellationToken` kabul etmediği için cancellation token kullanılmıyor. 


Iterator metodu tamamladıktan sonra çağırma kısmına geldiğimizde ise `WithCancellation` metodunu kullanarak cancellation tokenı parametre olarak geçebiliriz.

```csharp
await foreach (var line in ReadLinesAsync(filePath).WithCancellation(cts.Token))
{
    Console.WriteLine(line);
}
```

`IAsyncEnumerable<T>` supportu frameworkün çeşitli yerlerinde mevcut. Aynı zamanda Entity Framework ve gRPC streaming tarafında da `IAsyncEnumerable<T>`'ı kullanılabilmekte. `IAsyncEnumerable<T>` üzerinde LINQ supportu için de uygulamanıza `System.Linq.Async` nuget paketini yükleyip `IAsyncEnumerable<T>` üzerinde LINQ metotlarını kullanabilirsiniz.

Aynı zamanda paging bulunan API'larınızı çağırırken de `IAsyncEnumerable<T>` kullanabilirsiniz. Örnek göstermek açısında şu şekilde bir kod yazabiliriz. 

```csharp
public static async IAsyncEnumerable<Product> GetItemsAsync()
{
    var httpClient = new HttpClient();
    var skip = 0;
    var take = 10;
    var hasMorePage = true;

    while (hasMorePage)
    {
        var requestUrl = $"https://www.fakeApi.com/api/products?skip={skip}&take={take}";
        var response = await httpClient.GetFromJsonAsync<Container>(requestUrl);
        foreach (var product in response.Items)
        {
            yield return product;
        }
        skip += response.Items.Count;
        hasMorePage = skip < response.TotalCount;
    }
}
```
Bu yazıda .NET Core 3.0 ile beraber gelen `IAsyncEnumerable<T>` ile asenkron streaming yapılarını nasıl kurabileceğimizi inceledik. 

Bir sonraki yazıda görüşmek üzere,