
.NET 6 Preview 5 ile beraber gelen en heyecan verici yeniliklerden biri de .NET içerisinde implemente edilen source generatorların yavaş yavaş günyüzüne çıkması. Bu yazıda `System.Text.Json` API'larına gelen source generator desteğini inceleyeceğiz. 

Source generatorlar nedir, nasıl kullanılır gibi konularda bilgi almak isterseniz aşağıdaki blog yazılarını veya meetup videosunu izleyebilirsiniz. 

* Source Generatorlar İle Kod Yazan Kod Yazma: <a href="https://www.ilkayilknur.com/source-generatorlar-ile-kod-yazan-kod-yazma" target="_blank">https://www.ilkayilknur.com/source-generatorlar-ile-kod-yazan-kod-yazma</a>
* Source Generatorlar İçin Kod Üretim Opsiyonları: <a href="https://www.ilkayilknur.com/source-generatorlar-icin-kod-uretim-opsiyonlari" target="_blank">https://www.ilkayilknur.com/source-generatorlar-icin-kod-uretim-opsiyonlari</a>
* C#'da Source Generator'lara Bakış - İlkay İlknur: <a href="https://www.youtube.com/watch?v=ZLoto3GI-Lc" target="_blank">https://www.youtube.com/watch?v=ZLoto3GI-Lc</a>

**Bu yazıda göreceğiniz kodlar .NET 6 Preview 5 ve Visual Studio 2022 Preview ile yazılmıştır. İlerleyen versiyonlarda değişiklik olabilir.**

`System.Text.Json` API'larına gelen source generator desteği ile (de)serialization işlemleri için yaratılması gereken kodlar uygulamanın çalışma esnasında değil derleme esnasında yaratılmakta. Böylece source generatorların kullanılmadığı durumlarda uygulamaların ayağa kalkması sırasında bir takım operasyonların reflection kullanarak yapılmasından dolayı meydana gelecek olan performans ve memory maliyetlerinin önüne geçilmesi amaçlanmakta. Aynı zamanda source generatorların kullanılmasıyla linker-friendly uygulama paketleri yaratmak mümkün olabilmekte ve `System.Text.Json` librarysi üzerinde linker doğru bir şekilde çalışabilmekte.

SDK ile beraber gelen `System.Text.Json` kütüphanesinde henüz source generator desteği bulunmadığı için ilk olarak preview versiyon paketini projeye eklememiz gerekiyor.

```bash
dotnet add package System.Text.Json --PreRelease
```

İlk olarak (de)serialize işleminde kullanacağımız basit bir tip tanımlayalım. 

```csharp
public class Customer
{
    public string ID { get; set; }
    public string Name { get; set; }
    public string LastName { get; set; }
}
```

Sonrasında `JsonSerializerContext` tipinden türeyen yeni bir `partial` tip tanımlayalım.

```csharp
public partial class SerializationContext : JsonSerializerContext
{

}
```

Sonrasında da bu tanımladığımız tipin üzerinde `JsonSerializable` attribute'ünü kullanarak bu context içerisinde hangi tiplerin  (de)serialization'ı yapılacağını belirtelim. Biz yukarıda `Customer` tipini tanımlamıştık. `SerializationContext` tipinin son hali şu şekilde oluyor. Birden fazla `JsonSerializable` attribute'ü kullanarak birden fazla tip için aynı context tipini kullanabiliriz.

```csharp
[JsonSerializable(typeof(Customer))]
public partial class SerializationContext : JsonSerializerContext
{

}
```

Bu tanımlamayı yaptıktan sonra kod derlendiği ilk seferde ilgili JSON serialization kodları API tarafından derleme zamanında oluşturulmakta. Oluşturulan kodlara projenin `Analyzers` bölümünden ulaşıp inceleyebiliriz.

![system-text-json-sources](https://ilkayblog.blob.core.windows.net/uploads/2021/06/22/system-text-json-sources.png)

Source generator tarafından kodlar yaratıldıktan sonra bu yaratılan kodları `JsonSerializer` tipi ile aşağıdaki gibi kullanabiliriz. 

```csharp
var customer = new Customer()
{
    ID = "123",
    LastName = "Ilknur",
    Name = "Ilkay"
};

var json = JsonSerializer.Serialize(customer, SerializationContext.Default.Customer);
var obj = JsonSerializer.Deserialize<Customer>(json, SerializationContext.Default.Customer);
```

Aynı zamanda `Utf8JsonWriter` ile de kullanmamız mümkün.

```csharp
var customer = new Customer()
{
    ID = "123",
    LastName = "Ilknur",
    Name = "Ilkay"
};

var stream = new MemoryStream();
var writer = new Utf8JsonWriter(stream);
SerializationContext.Default.Customer.Serialize(writer, customer);
```

*`Utf8JsonReader` desteği ise şu an için eklenmemiş durumda.*

Bu yazıda `System.Text.Json` API'larına .NET 6 ile beraber gelecek olan source generator desteğini inceledik. Bundan sonraki preview versiyonlarında da bu incelediğimiz özelliklere geliştirmeler ve iyileştirmeler gelecektir.

Bir sonraki yazıda görüşmek üzere.
