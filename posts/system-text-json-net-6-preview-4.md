Herkese Selamlar,

Blogda bundan önce yazdığım yazılarda `System.Text.Json` API'larından uzun uzun bahsettim. 
Eğer yazıları görmediyseniz ve okumak isterseniz aşağıdaki linkleri kullanabilirsiniz.

* .NET Core'da JSON API'ları: <a href="https://www.ilkayilknur.com/net-coreda-json-apilari" target="_blank">https://www.ilkayilknur.com/net-coreda-json-apilari</a>
* System.Text.Json API'larına .NET 5.0 İle Beraber Gelen Yenilikler: <a href="https://www.ilkayilknur.com/system-text-json-apilarina-net-5-0-ile-beraber-gelen-yenilikler" target="_blank">https://www.ilkayilknur.com/system-text-json-apilarina-net-5-0-ile-beraber-gelen-yenilikler</a>

Blogda yazdığım yazılarla `System.Text.Json API`'larına geçişi elimden geldiğince teşvik etmeye çalışıyorum. Bu yazıda da Build 2021 konferansında duyurulan .NET 6 Preview 4 içerisinde bulunan `System.Text.Json` API yeniliklerine bakacağız. 

Bu yenilikleri deneyebilmek için <a href="https://dotnet.microsoft.com/download/dotnet/6.0" target="_blank">.NET 6 Preview 4</a> indirmeniz gerekmekte. .NET 6 Preview 4'ü test etmek için <a href="https://visualstudio.microsoft.com/vs/" target="_blank">Visual Studio 16.11</a> versiyonunu veya <a href="https://visualstudio.microsoft.com/vs/mac/" target="_blank">Visual Studio For Mac 8.9</a>'u kullanabilirsiniz.

Şimdi yenilikleri incelemeye başlayalım. 

##  IAsyncEnumerable Desteği

.NET 6 ile beraber `System.Text.Json` API'larına gelen önemli yeniliklerden biri `IAsyncEnumerable<T>` desteği. Artık `IAsyncEnumerable<T>`'ları JSON array olarak serialize edebiliyoruz. Böylece db'den veya başka bir asenkron streamden gelen dataları JSON API'larını kullanarak (de)serialize edebiliyoruz. `IAsyncEnumerable<T>` desteği şu an sadece `async` metotlarda bulunmakta. Senkron metotlarda denediğinizde `NotSupportedException` alabilirsiniz.

> `IAsyncEnumerable<T>` nedir ne değildir ile ilgili detaylı bilgi almak isterseniz <a href="https://www.ilkayilknur.com/iasyncenumerable-ile-asenkron-streaming" target="_blank">buradaki</a> yazımı okuyabilirsiniz. 

Ufak bir serialize örneği yapalım.

```csharp
class Program
{
  static async Task Main(string[] args)
  {
      using var fs = new FileStream("enum.json", FileMode.CreateNew);
      await JsonSerializer.SerializeAsync(fs, GetIAsyncEnumerable());
  }


  async static IAsyncEnumerable<int> GetIAsyncEnumerable()
  {
      for (int i = 0; i < 10; i++)
      {
          await Task.Delay(100);
          yield return i;
      }
  }
}
```

Deserialize için ise iki farklı metot bulunmakta. Eğer doğrudan object içerisine deserialize etmek isterseniz klasik async metotları kullanabilirsiniz. Ancak bir array deserialize ediyorsanız yeni gelen `DeserializeAsyncEnumerable<T>` metodunu kullanabilirsiniz.

```csharp
var stream = new MemoryStream(Encoding.UTF8.GetBytes("[0,1,2,3,4,5,6,7,8,9,10]"));
await foreach (int item in JsonSerializer.DeserializeAsyncEnumerable<int>(stream))
{
    Console.WriteLine(item);
}
```

## Writeable DOM

.NET 6 öncesinde `JsonDocument` ve `JsonElement` tipleri read-only olarak DOM desteklemekteydi. Preview 4 ile beraber writable DOM desteği için yeni gelen tipler üzerinden write operasyonları da desteklenmekte. 

`System.Text.Json.Node` namespace'i içerisine yeni eklenen tipler.

```csharp
namespace System.Text.Json.Node
{
    public abstract class JsonNode {...};
    public sealed class JsonObject : JsonNode, IDictionary<string, JsonNode?> {...}
    public sealed class JsonArray : JsonNode, IList<JsonNode?> {...};
    public abstract class JsonValue : JsonNode {...};
}
```

Bu tipler aynı zamanda belirli senaryolarda `JsonSerializer` tipine alternatif olarak da kullanılabilir. Büyük bir JSON içeriğinin belirli bir bölümünün okunması veya JSON karşılığı olan POCO'ların tanımlı olmadığı yada belli bir şema olmadığı için tanımlamanın mümkün olmadığı senaryolarda da bu tipler kullanılabilir.

Aynı zamanda bugüne kadar `dynamic` erişim özellikleri de json API'larında desteklenmiyordu. Bu yeni gelen tipler içerisinde `dynamic` desteği de bulunmakta. 

Ufak kullanım örnekleri yapalım. 

```csharp
var node = JsonNode.Parse("{\"name\":\"Ilkay\"}");
var name = node["name"];

var array = JsonArray.Parse("[1,2,3,4,5]");
var firstElement = array[0];

//dynamic support
dynamic obj = new JsonObject();
obj.name = "Ilkay";
obj.surname = "Ilknur";
obj.roles = new JsonArray("manager, system admin");

var json = obj.ToJsonString();
//{"name":"Ilkay","surname":"Ilknur","roles":["manager, system admin"]}

// Create a new JsonObject using object initializers and array params
    var jObject = new JsonObject
    {
        ["MyChildObject"] = new JsonObject
        {
            ["MyProperty"] = "Hello",
            ["MyArray"] = new JsonArray(10, 11, 12)
        }
    };
```

.NET 6.0 Preview 4 ile beraber gelen `System.Text.Json` API yeniliklerini inceledik. Bu API'ların release olmasına tabi ki daha oldukça uzun zaman var. Bu API'lara güncellemeler gelebilir ancak genel anlamda şimdiye kadar gelen yenilikleri kısaca inceleyip fikir sahibi olmanın iyi olacağını düşünüyorum.

Bir sonraki yazıda görüşmek üzere.