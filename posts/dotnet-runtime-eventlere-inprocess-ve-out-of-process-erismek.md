Bir <a href="https://www.ilkayilknur.com/net-uygulamalarinda-kullanabilecegimiz-diagnostics-araclari" target="_blank">önceki</a> yazıda .NET içerisindeki diagnostic araçlarından bahsetmiştik. Bu araçlardan biri olan `dotnet-trace` ile uygulamalarımızın tracelerini toplayıp sonrasında da bu traceleri çeşitli araçlarla analiz edip runtime eventlerini ve daha pek çok şeyi analiz edebiliyorduk. Bu yazıda ise işi biraz daha ileri götürüp bu runtime eventlerini uygulamalarımız içerisinde nasıl işleyebileceğimizi ve kendi monitoring araçlarımızı yapmak istediğimizde farklı processler arasında iletişim kurarak bu eventlere nasıl ulaşabileceğimizi inceleyeceğiz.

.NET runtime(CoreCLR), uygulamalar içerisinde çeşitli durumlar meydana geldiğinde eventler oluşturmakta. Örneğin, Garbage collection işlemi başlayacağı zaman <a href="https://docs.microsoft.com/en-us/dotnet/fundamentals/diagnostics/runtime-garbage-collection-events" target="_blank">buradan</a> da detaylarına ulaşabileceğimiz pek çok event üretilmekte. Bu eventler sayesinde garbage collectionın neden tetiklendiğinden tutun Gen2 GC tetiklendiğinde uygulamanın suspend, restart edilmesi gibi pek çok hareketten de haberimiz olabiliyor. Böylece uygulamalarımızda bir sorun olduğunda bu eventleri inceleyerek sorunları bulabilmemiz mümkün olabiliyor. Bazı durumlarda da bu verileri uygulama içerisinde işleyip bazı sonuçlar üretmek gerekebilir. 

Şimdi ilk olarak uygulama içerisinde bu eventlere nasıl ulaşabiliriz konusuna bakalım. Uygulamalarımız içerisinde eventleri dinlemek için `EventListener` tipini kullanabiliriz. İlk olarak uygulamamızda `EventListener`'dan türeyen bir sınıf yaratalım.

```csharp
public class GCEventListener : EventListener
{
    
}
```

`EventListener` tipi içerisinde override edeceğimiz iki metot bulunmakta. Bunlar `OnEventSourceCreated` ve `OnEventWritten`. `OnEventSourceCreated` içerisinde dinlemek istediğimiz event sourceları filtreleyeceğiz ve işimize yarayacak olan eventleri enable edeceğiz. `OnEventWritten` içerisinde ise gelen eventlere erişebileceğiz. İlk olarak `OnEventSourceCreated` implementasyonuna bakalım. 

```csharp
protected override void OnEventSourceCreated(EventSource eventSource)
{
    
}
```
Biz bu yazıda örneklerde garbage collector eventlerini dinleyeceğiz. İlk olarak <a href="https://docs.microsoft.com/en-us/dotnet/core/diagnostics/well-known-event-providers" target="_blank">buradan</a> GC eventlerinin hangi providerdan geleceğini bulacağız. GC eventleri `Microsoft-Windows-DotNETRuntime` providerından geliyor. Dolayısıyla event source içerisindeki `Name` propertysine bakarak bu providerı bulabiliriz.

```csharp
protected override void OnEventSourceCreated(EventSource eventSource)
{
    if (eventSource.Name == "Microsoft-Windows-DotNETRuntime")
    {
        EnableEvents(eventSource, EventLevel.Informational);
    }
}
```
Sonrasında ise `EventListener` içerisindeki `EnableEvents` metodunu çağırarak bu event sourcetan gelen eventleri dinlemek istediğimizi belirtebiliriz. Buradaki sorunlardan biri yukarıdaki dökümandan da gördüğünüz üzere `Microsoft-Windows-DotNETRuntime` providerı sadece GC eventlerini sağlamamakta. GC eventlerinin dışında pek çok event de bu providerdan gelmekte. Bunun için sadece GC eventlerini almak için `EnableEvents` metoduna bir de keyword parametresi geçmemiz lazım. Buradan GC eventlerine baktığımızda GCKeyword değeri 0x1. Bunu da parametre olarak geçtiğimizde `OnEventSourceCreated` metodunun son hali şu şekilde olmakta. 

```csharp
protected override void OnEventSourceCreated(EventSource eventSource)
{
    if (eventSource.Name == "Microsoft-Windows-DotNETRuntime")
    {
        EnableEvents(eventSource, EventLevel.Informational, (EventKeywords)0x1);
    }
}
```
Dinleyeceğimiz eventleri belirledikten sonra sıra geldi gelen eventleri işlemeye. İşlemek için `OnEventWritten` metodunu kullanacağız.

```csharp
protected override void OnEventWritten(EventWrittenEventArgs eventData)
{
    
}
```
Bu metoda parametre olarak gelen `eventData` içerisinde `EventName` parametresi bulunmakta. Bu propertyi kullanarak ilgilendiğimiz eventleri bulabiliriz. Diyelim ki Gen2 tetiklendiğinde uygulamanın suspend edilmesi ve tekrardan başlatılması eventlerini yakalamak istiyoruz. <a href="https://docs.microsoft.com/en-us/dotnet/fundamentals/diagnostics/runtime-garbage-collection-events" target="_blank">Buradaki</a> dökümandan önce hangi eventleri yakalayacağımızı bulmamız gerekiyor. Dökümana göre kullanacağımız eventler `GCSuspendEEBegin_V1` ve `GCSuspendEEEnd_V1`. Bu eventleri yakalayıp içerisindeki dataları inceleyebilir veya yaratılma zamanlarını kontrol ederek uygulama ortalama olarak ne kadar suspend ediliyor gibi sonuçları çıkarabiliriz.

```csharp
protected override void OnEventWritten(EventWrittenEventArgs eventData)
{
    if (eventData.EventName == "GCSuspendEEBegin_V1")
    {
        Console.WriteLine($"Suspend started ticks = {eventData.TimeStamp.Ticks}");
    }
    else if (eventData.EventName == "GCSuspendEEEnd_V1")
    {
        Console.WriteLine($"Suspend ended ticks = {eventData.TimeStamp.Ticks}");
    }
}
```

Bir başka kullanım örneği olarak GC tetiklenme eventlerini dinleyip neden tetiklendiğini bulup duruma göre bazı alarm mekanizmalarını tetiklemek olabilir. Payload içerisindeki fieldlara ve anlamlarına yine <a href="https://docs.microsoft.com/en-us/dotnet/fundamentals/diagnostics/runtime-garbage-collection-events" target="_blank">buradan</a> ulaşabiliriz. Örneğin, large object heap allocationları nedeniyle GC tetiklenmesini yakalamak istersek aşağıdaki gibi bir kod yazabiliriz.

```csharp
protected override void OnEventWritten(EventWrittenEventArgs eventData)
{
    if (eventData.EventName == "GCStart_V2")
    {
        if ((uint)eventData.Payload[2] == 4)
        {
            //Send alarm - GC triggered due to large object heap allocation.
        }
    }
}
```

### Runtime eventlerine out-of-process erişmek
Bu bölümde de `dotnet-trace` gibi bir monitoring aracı yapmak istersek nasıl yapabiliriz konusuna bakacağız. Bunun için `Microsoft.Diagnostics.NETCore.Client` isimli paketi projemizde kullanacağız. Bu paket sayesinde runtime ile diagnostics IPC(inter-process communication) protokolünü kullanarak iletişim kurabileceğiz. 

`dotnet add package Microsoft.Diagnostics.NETCore.Client`

Bu paket içerisinde en çok kullanacağımız tip `DiagnosticsClient` tipi olacak. Bu tipi kullanarak pek çok farklı işlem yapabilmekteyiz. Şimdi vakit kaybetmeden bu operasyonlara bakalım. `DiagnosticsClient` tipini kullabilmek için öncelikle eventlerini dinleyeceğimiz processin IDsini bilmemiz gerekiyor. Bunu bulabilmek için `dotnet-trace` içerisinde `ps` komutunu kullanabiliyorduk. Bu komutun benzerini kendi yazdığımız monitoring aracındada geliştirebiliriz. DiagnosticClient içerisindeki statik `GetPublishedProcesses` metodu içerisinde diagnostic server çalışan processlerin IDlerini bize dönmekte.

```csharp
static void Main(string[] args)
{
    switch (args[0])
    {
        case "ps":
            ListProcesses();
            break;
        default:
            break;
    }
}

private static void ListProcesses()
{
    foreach (var processId in DiagnosticsClient.GetPublishedProcesses())
    {
        Console.WriteLine($"Process Id {processId} Name {Process.GetProcessById(processId).ProcessName}");
    }
}
```
Bu şekilde bir implementasyonla kendi aracımız içerisinde de `ps` komutuyla processleri listeleyebiliriz. Şimdi yukarıdaki örnek olduğu gibi GC başlama eventini monitör eden bir komut geliştirelim. 

```csharp
var client = new DiagnosticsClient(processId);
using var session = client.StartEventPipeSession(new EventPipeProvider("Microsoft-Windows-DotNETRuntime", EventLevel.Informational, 1));
```

Yukarıdaki kodla basitçe DiagnosticClient'ı kullanarak bir event pipe sessionı başlatıyoruz. Parametrelerin ne olduğunu zaten yukarıda açıklamıştım. Bu nedenle burada detaya girmiyorum. Sessionı yarattıktan session içerisinde bulunan streami okuyarak gelen eventleri işleyebiliriz. Bu operasyonu daha da kolaylaştırmak için `Microsoft.Diagnostics.Tracing.TraceEvent` kütüphanesini kullanabiliriz. 

```csharp
static void Main(string[] args)
{
    switch (args[0])
    {
        case "ps":
            ListProcesses();
            break;
        case "monitor-gc-events":
            MonitorGCEvents(int.Parse(args[1]));
            break;
        default:
            break;
    }
}

private static void MonitorGCEvents(int processId)
{
    var client = new DiagnosticsClient(processId);
    using var session = client.StartEventPipeSession(new EventPipeProvider("Microsoft-Windows-DotNETRuntime", EventLevel.Informational, 1));
    EventPipeEventSource source = new EventPipeEventSource(session.EventStream);

    source.Clr.GCStart += Clr_GCStart;

    source.Process();
}

private static void Clr_GCStart(Microsoft.Diagnostics.Tracing.Parsers.Clr.GCStartTraceData obj)
{
    Console.WriteLine($"GC Started {obj.TimeStamp.Ticks}");
}
```
Gördüğünüz üzere `EventStream`'i kullanarak bir `EventPipeEventSource` yarattık ve bunun üzerinden istediğimiz evente subscribe olup dinleyebildik. `DiagnosticsClient` içerisinde daha pek çok farklı metot bulunmakta. Örneğin, `WriteDump` metodunu kullanarak o processin dumpını alabilirsiniz. Bazı senaryoları dinleyen monitoring araçları geliştirip belirli durumlarda dump alma işini de böylece otomatize edebilmeniz mümkün veya burada olduğu gibi özel bir CLI aracı geliştirip istediğiniz durumda dump alma olayını kendiniz tetikleyebilirsiniz.

Bu yazıda yaptığım örneklere <a href="https://github.com/ilkayilknur/DotNetDiagnostics.Sample" target="_blank">GitHub'dan</a> ulaşabilirsiniz. Örnek API metodu içerisinde large object allocationına neden olacak bir kod bulunmakta. Böylece dışarıya açılan API'ı 4-5 kez çağırırsanız GC tetiklenecek ve eventlere ulaşabileceksiniz. 

Bir sonraki yazıda görüşmek üzere,

Kaynaklar
* https://docs.microsoft.com/en-us/dotnet/fundamentals/diagnostics/runtime-garbage-collection-events
* https://github.com/dotnet/diagnostics/blob/master/documentation/design-docs/diagnostics-client-library.md
* https://www.youtube.com/watch?v=Jpoy3O6x-wM
* https://www.youtube.com/watch?v=Rei6d9nKaFQ