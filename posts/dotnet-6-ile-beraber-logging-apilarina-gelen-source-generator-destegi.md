Bir önceki <a href="https://www.ilkayilknur.com/net-6-preview-5-ile-system-text-json-apilarina-gelen-source-generator-destegi" target="_blank">yazıda</a> .NET 6 ile beraber `System.Text.Json` API'larına gelecek olan source generator desteğinden bahsetmiştik. Bu yazıda da yine .NET 6 ile beraber logging API'larına gelecek olan source generator desteğinden bahsedeceğiz.

.NET 6 ile beraber logging API'larına gelen source generation desteğinin temel olarak dayandığı ve source generationın sağladığı tip `LoggerMessageAttribute` tipi. Bu attribute'ü kullanarak logging metotlarının otomatik olarak generate edilmesini sağlayabiliyoruz. Generate edilen kaynak kod içerisinde <a href="https://docs.microsoft.com/en-us/dotnet/core/extensions/high-performance-logging" target="_blank">LoggerMessage.Define</a> kullanıldığı için aslında kısa yoldan hızlı bir şekilde hatasız olarak .NET içerisindeki en performanslı logging yöntemini de kullanmış oluyoruz. 

Şimdi gelelim bu source generatorın nasıl kullanılacağına. İlk olarak projemize `Microsoft.Extensions.Logging` paketinin .NET 6 için olan preview paketini yüklememiz gerekiyor. 

```bash
dotnet add package Microsoft.Extensions.Logging --PreRelease
```

*Bu yazının yazıldığı dönemde source generation desteği Microsoft.Extensions.Logging paketi tarafından sağlanıyordu. İlerleyen preview versiyonlarında bu desteğin  Microsoft.Extensions.logging.Abstractions paketine kaydırılması söz konusu.*

Paketi yükledikten sonra altyapıyı hazırlamış oluyoruz. Sonrasında loglama yapacağımız tipleri yazmamız gerekiyor. Bu noktada farklı opsiyonlarımız bulunmakta. İlk olarak static tip üzerinden bir implementasyon yapalım. 

```csharp
public static partial class Logger
{
    [LoggerMessage(
        EventId = 0,
        Level = LogLevel.Information,
        Message = "Data could not found in cache: `{key}`")]
    public static partial void DataCouldNotFoundInCache(
        ILogger logger, string key);
}
```

Burada görüldüğü üzere hem tipi hem de metodu `partial` olarak tanımlıyoruz. Böylece source generator kullanarak hem tipi hem de metodu extend edebiliyoruz. Bu kullanımda `ILogger` implementasyonu tüm tanımlı `partial` metotlara parametre olarak geçilmek durumunda.

Arka planda generate edilen kodu da aşağıda görebiliriz.

```csharp
partial class Logger 
{
    [global::System.CodeDom.Compiler.GeneratedCodeAttribute("Microsoft.Extensions.Logging.Generators", "6.0.0.0")]
    private static readonly global::System.Action<global::Microsoft.Extensions.Logging.ILogger, global::System.String, global::System.Exception?> __DataCouldNotFoundInCacheCallback =
        global::Microsoft.Extensions.Logging.LoggerMessage.Define<global::System.String>(global::Microsoft.Extensions.Logging.LogLevel.Information, new global::Microsoft.Extensions.Logging.EventId(0, nameof(DataCouldNotFoundInCache)), "Data could not found in cache: `{key}`", true); 

    [global::System.CodeDom.Compiler.GeneratedCodeAttribute("Microsoft.Extensions.Logging.Generators", "6.0.0.0")]
    public partial void DataCouldNotFoundInCache(global::System.String key)
    {
        if (_logger.IsEnabled(global::Microsoft.Extensions.Logging.LogLevel.Information))
        {
            __DataCouldNotFoundInCacheCallback(_logger, key, null);
        }
    }
}
```

Bir diğer kullanım şekli ise instance bazlı tanımlama. Bu yöntemde `ILogger`'ı constructordan parametre olarak alabiliriz. Böylece her metoda parametre olarak geçmek durumunda kalmayız.

```csharp
public partial class Logger
{
    private readonly ILogger _logger;

    public Logger(ILogger logger)
    {
        _logger = logger;
    }

    [LoggerMessage(
        EventId = 0,
        Level = LogLevel.Information,
        Message = "Data could not found in cache: `{key}`")]
    public partial void DataCouldNotFoundInCache(string key);
}
```

Şimdiye kadar yaptığımız kullanımlarda gördüğünüz üzere `LogLevel` attribute içerisinde tanımlandı ve bu levelı herhangi bir şekilde değiştirmemiz mümkün değil. Ancak log level değerini metoda parametre olarak ekleyip bu değerin dinamik olarak dışarıdan verilmesini de sağlayabiliriz.

```csharp
public partial class Logger
{
    private readonly ILogger _logger;

    public Logger(ILogger logger)
    {
        _logger = logger;
    }

    [LoggerMessage(
        EventId = 0,
        Message = "Data could not found in cache: `{key}`")]
    public partial void DataCouldNotFoundInCache(LogLevel level, string key);
}
```

Bu yazıda .NET 6 ile beraber logging tarafına gelecek olan source generator desteğinden bahsettik. Source generatorlar bu alanda da bizim boilerplate kod yazmamızın önüne geçerek en doğru ve performanslı bir şekilde implementasyon yapmamızı da sağlayan bir altyapı sunuyor. 

Bir sonraki yazıda görüşmek üzere.

Kaynak: <a href="https://docs.microsoft.com/en-us/dotnet/core/extensions/logger-message-generator" target="_blank">https://docs.microsoft.com/en-us/dotnet/core/extensions/logger-message-generator</a>



