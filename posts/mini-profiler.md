Bu yazıda uygulamalarımızda profiling yapmak için kullanabileceğimiz araçlardan biri olan Mini Profiler'dan bahsedeceğiz. Mini Profiler kullanarak hızlı ve basit bir şekilde uygulamalarımızda hangi noktalarda ne kadar zaman harcandığını görmemiz mümkün. Böylece development esnasında performans sorunu yaşadığımız noktaları tespit edip buna göre aksiyon almamız oldukça kolaylaşmakta. Mini profiler Stackoverflow ekibi tarafından geliştirilmekte olup ve aynı zamanda uygulamalarında da aktif olarak kullanılmakta. İncelemek isteyenler için Github <a href="https://github.com/MiniProfiler/dotnet" target="_blank">linkini</a> buraya bırakıyorum. 

Şimdi gelin Mini profilerı uygulamalarımıza nasıl entegre ediyoruz kısmıyla başlayalım. ASP.NET Core uygulamalarından Mini profilerı kullanmak için aşağıdaki nuget paketini yüklememiz gerekiyor. 

`Install-Package MiniProfiler.AspNetCore.Mvc`

Paketi yükledikten sonra `Startup.cs` içerisinde mini profiler konfigürasyonunu yapıp ve middleware'ini aşağıdaki gibi eklememiz gerekiyor. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;ConfigureServices(<span style="color:#2b91af;">IServiceCollection</span>&nbsp;services)
{
&nbsp;&nbsp;&nbsp;&nbsp;services.AddMiniProfiler();
}
 
<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Configure(<span style="color:#2b91af;">IApplicationBuilder</span>&nbsp;app,&nbsp;<span style="color:#2b91af;">IWebHostEnvironment</span>&nbsp;env)
{
&nbsp;&nbsp;&nbsp;&nbsp;app.UseMiniProfiler();
}</pre>

Sonrasında Mini profiler tag helperlarını aşağıdaki gibi `_ViewImports.cshtml` dosyasına eklememiz gerekiyor. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="background:yellow;">@</span><span style="color:blue;">using</span>&nbsp;StackExchange.Profiling
<span style="background:yellow;">@addTagHelper</span>&nbsp;<span style="color:#a31515;">*,&nbsp;MiniProfiler.AspNetCore.Mvc</span></pre>

Son olarak ise Layout.cshtml dosyasına mini profiler view'ını ekliyoruz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">&lt;</span><span style="font-weight:bold;color:purple;">mini-profiler</span><span style="color:blue;">/&gt;</span>
</pre>

Bu şekilde default olarak konfigürasyon yapıp uygulamayı çalıştırdığınızda sayfanın sol üst köşesinden mini profiler view'ını görebilirsiniz. 

![](https://az718566.vo.msecnd.net/uploads/2020/08/05/mini-profiler-default.png)

Bu view'a tıkladığımızda ise daha detaylı görünüme geçebiliyoruz. 

![](https://az718566.vo.msecnd.net/uploads/2020/08/05/mini-profiler-maximize.png)

Sağ alttaki show trivial linkine tıkladığımızda ise hangi action filterlarda, hangi viewlarda vs.. ne kadar vakit harcandığını görmemiz mümkün. 

![](https://az718566.vo.msecnd.net/uploads/2020/08/05/mini-profiler-trivial.png)

### Entity Framework Core Entegrasyonu 

Mini profilerın en güzel özelliklerinden biri de entity framework core entegrasyonu olması ve bu entegrasyon sayesinde tüm database sorguları, connection hareketleri vs.. gibi işlemleri kolayca görebilmemiz. Bu entegrasyonu aktifleştirmek için öncelikle entegrasyon paketini yüklememiz gerekiyor.

`Install-Package MiniProfiler.EntityFrameworkCore`

Sonrasında `Startup.cs`'e gidip ufak bir ekleme yapiyoruz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;ConfigureServices(<span style="color:#2b91af;">IServiceCollection</span>&nbsp;services)
{
&nbsp;&nbsp;&nbsp;&nbsp;services.AddMiniProfiler()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.AddEntityFramework();
}</pre>

Uygulamayı yeniden başlattığımızda DB tarafında ne kadar vakit harcanıyor, hangi sql sorguları çalışıyor hepsini görebiliyoruz. 

![](https://az718566.vo.msecnd.net/uploads/2020/08/05/mini-profiler-sql-2.png)

![](https://az718566.vo.msecnd.net/uploads/2020/08/05/mini-profiler-sql.png)


### Daha Detaylı Profiling

Yukarıdaki ekran görüntülerine baktığınızda mini profilerın default olarak her actionı bir bütün olarak gösterdiğini ve altındaki işlemleri ayrıca göremediğimizi farketmişsinizdir. Örneğin bir action içerisinde 2-3 farklı yere requestte bulunuyorsak bu operasyonları ayrı ayrı takip etmemiz default kullanımda mümkün değil. Bunun için kendimiz ayrıca profiling adımları tanımlamamız gerekiyor.

Profiling adımlarını tanımlamak için `MiniProfiler.Current` içerisindeki `Step` metodunu kullanabiliriz. Step metodunu çağırırken o adıma vereceğimiz adı yazmamız yeterli. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">IActionResult</span>&nbsp;Index()
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">using</span>&nbsp;(<span style="color:#2b91af;">MiniProfiler</span>.Current.Step(<span style="color:#a31515;">&quot;UserCheck&quot;</span>))
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">using</span>&nbsp;(<span style="color:#2b91af;">MiniProfiler</span>.Current.Step(<span style="color:#a31515;">&quot;User&nbsp;Role&nbsp;Check&quot;</span>))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
 
}</pre>

Sonrasında uygulamayı çalıştırdığımızda tanımladığımız adımların ayrı ayrı gözüktüğünü göreceğiz. 

![](https://az718566.vo.msecnd.net/uploads/2020/08/05/mini-profiler-step.png)

Bazı profiling adımlarının belirtilen sürenin üzerinde sürdüğü durumlarda profiling raporunda ayrı bir step olarak gözükmesini de isteyebiliriz. Bunun için de StepIf metodunu kullanabiliriz. Örneğin UserRoleCheck adımı 100ms'den uzun sürdüğü durumlarda gözükmesini istersek şu şekilde bir tanımlama yapabiliriz. 
<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">using</span>&nbsp;(<span style="color:#2b91af;">MiniProfiler</span>.Current.Step(<span style="color:#a31515;">&quot;UserCheck&quot;</span>))
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">using</span>&nbsp;(<span style="color:#2b91af;">MiniProfiler</span>.Current.StepIf(<span style="color:#a31515;">&quot;User&nbsp;Role&nbsp;Check&quot;</span>,&nbsp;100))
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

Mini profiler kullanırken uygulama dışında bağlandığımız servislerin profilingini yapmak için Custom Timing yapısını kullanabiliriz. Custom Timing yapısı aslında Entity Framework entegrasyonundaki yapının benzerini bizim kurmamıza imkan veriyor. Yani bir kategori belirleyip o kategori altında çalışan sorguları veya requestleri profiling raporunda görebiliyoruz. 

CustomTiming metodunun örnek kullanımı ise şu şekilde. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">using</span>&nbsp;(<span style="color:#2b91af;">MiniProfiler</span>.Current.CustomTiming(<span style="color:#a31515;">&quot;http&quot;</span>,&nbsp;<span style="color:#a31515;">&quot;http://www.ilkayilknur.com&quot;</span>,&nbsp;<span style="color:#a31515;">&quot;GET&quot;</span>))
{
}</pre>

Burada ilk parametre kategorinin adı, ikinci parametre çalışan komutun kendisi, üçüncü parametre ise executionType. Bu şekilde bir custom timing tanımladıktan sonra profiling raporunda şu şekilde görebiliyoruz. 

![](https://az718566.vo.msecnd.net/uploads/2020/08/05/mini-profiler-custom-timing.png)

![](https://az718566.vo.msecnd.net/uploads/2020/08/05/mini-profiler-custom-timing-2.png)

### View İçerisinde Profiling

Kod tarafında profiling yapabilmek için kullanmamız gereken bileşenleri inceledik. View tarafında da ekstra adımlar eklemek istediğimizde profile tag helperını kullanabiliriz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">&lt;</span><span style="font-weight:bold;color:purple;">profile</span>&nbsp;<span style="font-weight:bold;color:purple;">name</span><span style="color:blue;">=</span><span style="color:blue;">&quot;UserMenuGeneration&quot;</span><span style="color:blue;">&gt;</span>
 
<span style="color:blue;">&lt;/</span><span style="font-weight:bold;color:purple;">profile</span><span style="color:blue;">&gt;</span></pre>


### Web API İçerisinde Mini Profiler Kullanımı

Web API'lar içerisinde herhangi bir UI olmadığı için API'ların profile edilmesinde bir başka mini profiler özelliğini kullanacağız. Mini profilerı default olarak konfigüre ettiğinizde iki endpointin dışarı açıldığını göreceksiniz. 

Bunlar, 

* /mini-profiler-resources/results-index : Son profiling sessionların listesi
* /mini-profiler-resources/results : Son profiling sessionın detayları

Bu endpointleri kullanarak Web API'larda da profiling yapmamız mümkün. Bu endpointler aynı zamanda MVC uygulamalarında da bulunmakta.

![](https://az718566.vo.msecnd.net/uploads/2020/08/05/web-api-profiling.png)

![](https://az718566.vo.msecnd.net/uploads/2020/08/05/web-api-profiling-2.png)

Endpointlerin base kısmını(mini-profiler-resources) değiştirmek isterseniz konfigürasyon sırasında RouteBasePath'i kullanabilirsiniz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;">services.AddMiniProfiler(options=&gt;
{
&nbsp;&nbsp;&nbsp;&nbsp;options.RouteBasePath&nbsp;=&nbsp;<span style="color:#a31515;">&quot;profiles&quot;</span>;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
});</pre>


### Mini Profiler Özelleştirme

Şimdiye kadar çoğunlukla mini profilerı default ayarlarıyla kullanmayı inceledik. Ancak mini profilerın pek çok özelleştirme ayarı mevcut. Bu ayarlara `MiniProfilerOptions` içerisinden ulaşabiliriz. Ben en çok dikkatimi çeken özelliklerden bahsedeceğim. 

* **ShouldProfile**: Mini profiler default olarak gelen tüm requestleri profile eder. Bu özelliği değiştirmek istediğimizde veya production ortamlarında gerektiğinde sadece belirli bölümlerin profile edilmesini istediğimizde bu propertyi kullanabiliriz.   

* **ResultsAuthorize, ResultsAuthorizeAsync, ResultsListAuthorize,ResultsListAuthorizeAsync**  : Yukarıda bahsettiğim iki adrese kimlerin erişebileceğini belirleyebileceğiniz yerler. Özellikle production ortamlarında herkesin profiling sessionlarına erişmesini istemeyeceksinizdir. 

* **EnableMvcFilterProfiling, EnableMvcViewProfiling** : Filter ve Viewların ayrı adımlar olarak profiling raporlarında gösterilip gösterilmeyeceğini belirleyen property. Default değeri true. 

### Profiling Sessionların Saklanması

Mini profilerın bir diğer kuvvetli yönü de pek çok provider desteğiyle profiling verilerinin saklanmasına yardımcı olması. Mini profiler providerlarıyla aşağıdaki yerlerde profiling verilerini saklayabiliyoruz. 

* SqlServer
* Redis
* Sqlite
* MySQL
* MongoDB
* RavenDB
* PostgreSQL
* SqlServerCE

Hangi ortamda saklamak istedigimiz konusunda seçimi yaptıktan sonra ilgili provider paketini yükleyip ilerleyebiliriz. Örneğin SqlServerda verileri saklamak istersek öncelikle `MiniProfiler.Providers.SqlServer` nuget paketini projemize ekliyoruz. Sonrasında ise options içerisindeki Storage propertysine ilgili tanımlamayı yapıyoruz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;ConfigureServices(<span style="color:#2b91af;">IServiceCollection</span>&nbsp;services)
{
&nbsp;&nbsp;&nbsp;&nbsp;services.AddMiniProfiler(options&nbsp;=&gt;
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;options.Storage&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">SqlServerStorage</span>(<span style="color:#a31515;">&quot;ConnectionString&quot;</span>);&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;});
&nbsp;&nbsp;&nbsp;&nbsp;services.AddControllers();
}</pre>

Bir sonraki yazıda görüşmek üzere,