Bu yazıda .NET Core uygulamalarını monitör ederken kullanabileceğimiz .NET Core 3.0 ile beraber gelen `dotnet-counters` isimli araçtan bahsedeceğiz. dotnet-counters aracı basit olarak EventCounter API'ı üzerinden gönderilen counterları okumamızı veya saklamamızı sağlayan bir araç. EventCounter API'ları windows tarafında kullandığımız performance counterların aslında cross-platform versiyonu. EventCounter üzerinden istediğimiz metriği cross platform olarak loglayabiliriz. Böylece uygulamalarımızda bir problem olduğunda kolayca dotnet-counters aracını çalıştırıp sorunla ilgili ilk araştırmaları yapmamız mümkün hale gelmekte.

dotnet-counters aracını aşağıdaki komutu kullanarak yükleyebiliriz.

`dotnet tool install --global dotnet-counters`

Aracı global olarak yükledikten sonra `dotnet-counters ps` komutuyla monitor edebileceğimiz dotnet processlerini listeleyebiliriz. 

![](https://az718566.vo.msecnd.net/uploads/2020/08/20/counters-ps.png)

Monitör etmek istediğimiz processi bulduktan sonra aşağıdaki komutla loglanan counterları terminalde görebiliriz. 

`dotnet-counters monitor --process-id <PID>`

![](https://az718566.vo.msecnd.net/uploads/2020/08/20/dotnet-counters-monitor-1.png)

Bu komutu çalıştırdığımızda uygulamamızda desteklenen tüm counterların çıktılarını göremeyebiliriz. Bu nedenle ilk olarak desteklenen counterları belirleyip sonrasında monitor komutunu çalıştırmak daha doğru olacaktır. 

Providerlara göre gruplu olarak counterları listelemek için aşağıdaki komutu kullanabiliriz.

`dotnet-counters list`

Sonrasında işimize yarayacak olan counterları bulduktan sonra monitor komutunu counter listesi vererek kullanabiliriz. Örneğin Microsoft.AspNetCore.Hosting counterlarını listelemek istersek. 

`dotnet-counters monitor --process-id <PID> --counter-list Microsoft.AspNetCore.Hosting`

![](https://az718566.vo.msecnd.net/uploads/2020/08/20/dotnet-counters-monitor-2.png)

Monitor komutunu kullanırken belirtebileceğimiz son parametre ise refresh interval. Refresh interval'ı belirterek counterların kaç saniyede bir güncellenerek ekranda gösterileceğini belirtebilirsiniz. 

`dotnet-counters monitor --process-id <PID> --refresh-interval <seconds>`

dotnet-counters aracıyla topladığımız verileri terminalde görüntülemek yerine daha sonra incelemek adına dosyaya da kaydedebiliriz. Bunun için de `dotnet-counters collect` komutunu kullanabiliriz. Bu komutu kullanırken monitor komutundaki parametrelere ek olarak dosyayı hangi formatta saklayacağını ve dosyanın adının ne olacağını belirtmemiz gerekiyor. 

`dotnet-counters collect --process-id <PID> --format <json|csv> --output <fileName>`

### Custom EventCounter Tanımlama

`dotnet-counters` aracını kullanırken ASP.NET Core veya kullandığımız kütüphanelerin sağladığı  event counterları görüntülemenin yanı sıra kendi yazdığımız counterları da monitör edebilmemiz mümkün. Bunun için öncelikle bir EventSource tanımlaması yapıp, sonrasında da içerisine ilgili metriği loglayabiliriz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;">[<span style="color:#2b91af;">EventSource</span>(Name&nbsp;=&nbsp;<span style="color:#a31515;">&quot;MyCustomSource&quot;</span>)]
<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">MyEventSource</span>&nbsp;:&nbsp;<span style="color:#2b91af;">EventSource</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:#2b91af;">MyEventSource</span>&nbsp;Instance&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">MyEventSource</span>();
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:#2b91af;">EventCounter</span>&nbsp;counter;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">MyEventSource</span>()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;counter&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">EventCounter</span>(<span style="color:#a31515;">&quot;custom-counter&quot;</span>,&nbsp;<span style="color:blue;">this</span>);&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Log()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;counter.WriteMetric(1);
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

Metriklerin loglanmasını da bir action filter üzerinden tetikleyebiliriz.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">CustomFilter</span>&nbsp;:&nbsp;<span style="color:#2b91af;">ActionFilterAttribute</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">override</span>&nbsp;<span style="color:blue;">void</span>&nbsp;OnActionExecuted(<span style="color:#2b91af;">ActionExecutedContext</span>&nbsp;context)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">MyEventSource</span>.Instance.Log();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">base</span>.OnActionExecuted(context);
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

`EventCounter` tipi metric loglarken kullanabileceğimiz en temel tip. Bu tipi kullanarak loglanan değerlerin ortalaması dotnet-counters'da gösterilmekte.  Bunun yanında `IncrementingEventCounter, IncrementingPollingCounter, PollingCounter` tiplerini kullanarak loglama operasyonlarını gerçekleştirebiliriz. 

Son olarak bir örnek yaparsak diyelim ki custom bir API'ımız var ve biz bu API'a bir saniyede gelen request sayısını görmek istiyoruz. Bunun için şu şekilde `EventCounter` kullanımı yapabiliriz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;">[<span style="color:#2b91af;">EventSource</span>(Name&nbsp;=&nbsp;<span style="color:#a31515;">&quot;MyCustomSource&quot;</span>)]
<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">MyEventSource</span>&nbsp;:&nbsp;<span style="color:#2b91af;">EventSource</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:#2b91af;">MyEventSource</span>&nbsp;Instance&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">MyEventSource</span>();
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:#2b91af;">IncrementingPollingCounter</span>&nbsp;counter;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:blue;">long</span>&nbsp;totalRequestCount;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">MyEventSource</span>()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;counter&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">IncrementingPollingCounter</span>(<span style="color:#a31515;">&quot;weather-api-request-rate&quot;</span>,&nbsp;<span style="color:blue;">this</span>,&nbsp;()&nbsp;=&gt;&nbsp;totalRequestCount)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DisplayRateTimeScale&nbsp;=&nbsp;<span style="color:#2b91af;">TimeSpan</span>.FromSeconds(1),
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;};
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Log()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Interlocked</span>.Increment(<span style="color:blue;">ref</span>&nbsp;totalRequestCount);
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

Sonrasında ise yukarıda yazdığımız action filterı ilgili API'a ekliyoruz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;">[<span style="color:#2b91af;">HttpGet</span>]
[<span style="color:#2b91af;">CustomFilter</span>]
<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">IEnumerable</span>&lt;<span style="color:#2b91af;">WeatherForecast</span>&gt;&nbsp;Get()
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;rng&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Random</span>();
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:#2b91af;">Enumerable</span>.Range(1,&nbsp;5).Select(index&nbsp;=&gt;&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">WeatherForecast</span>
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Date&nbsp;=&nbsp;<span style="color:#2b91af;">DateTime</span>.Now.AddDays(index),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;TemperatureC&nbsp;=&nbsp;rng.Next(-20,&nbsp;55),
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Summary&nbsp;=&nbsp;Summaries[rng.Next(Summaries.Length)]
&nbsp;&nbsp;&nbsp;&nbsp;})
&nbsp;&nbsp;&nbsp;&nbsp;.ToArray();
}</pre>

Uygulamayı çalıştırıp sonrasında da dotnet-counters aracıyla monitor etmeye başlıyoruz. 

`dotnet-counters monitor -p 5944 --counter-list MyCustomSource`

![](https://az718566.vo.msecnd.net/uploads/2020/08/20/custom-event-counter.gif)

Bir sonraki yazıda görüşmek üzere,