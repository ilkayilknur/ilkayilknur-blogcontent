Health check mekanizmaları uygulamamızın sağlığının yerinde olup olmadığını görebildiğimiz ve gerekli durumlarda da dışarıya bildirebildiğimiz mekanizmalar. Uygulamanızı monitör ederken bu mekanizmadan gelen sonuçları değerlendirebileceğiniz gibi aynı zamanda bu API'ları load balancer veya container orchestratora verip onların bu bilgileri kullanarak trafik yönlendirmesi veya instanceları restart etmek gibi kararlar verebilmelerini sağlayabilirsiniz. 

ASP.NET içerisindeki health check mekanizmasını kullanmak için aşağıdaki gibi health check middleware'ini ekleyip konfigüre edebiliriz.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;ConfigureServices(<span style="color:#2b91af;">IServiceCollection</span>&nbsp;services)
{
&nbsp;&nbsp;&nbsp;&nbsp;services.AddControllers();
&nbsp;&nbsp;&nbsp;&nbsp;services.AddHealthChecks()
}</pre>

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Configure(<span style="color:#2b91af;">IApplicationBuilder</span>&nbsp;app,&nbsp;<span style="color:#2b91af;">IWebHostEnvironment</span>&nbsp;env)
{
&nbsp;&nbsp;&nbsp;&nbsp;app.UseEndpoints(endpoints&nbsp;=&gt;
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;endpoints.MapControllers();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;endpoints.MapHealthChecks(<span style="color:#a31515;">&quot;/health&quot;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;});
}</pre>

Uygulamamızı ayağa kaldırıp "/health" endpointine gittiğimizde sonuç olarak HTTP 200 ve bodyde de plain text olarak `Healthy` çıktısını alırız. 

![](https://az718566.vo.msecnd.net/uploads/2020/09/24/health.png)

Buradaki health check mekanizması aslında kullanabileceğimiz en basit mekanizma. Amaç uygulamanız dışarıdan gelen requestlere cevap verebiliyor mu bunu anlamak. Bunun yanında tabi uygulamanız içerisinde kullandığınız çeşitli servislerin de durumlarını takip etmeniz gerekebilir. Uygulamanız database'e bağlanabiliyor mu veya redis'e erişebiliyor mu gibi durumları da kontrol etmemiz gerekebilir. Entity Framework kullanıyorsanız Microsoft tarafından geliştirilen `Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore` paketini projenize ekleyerek hızlı bir şekilde health check ekleyebilirsiniz. 

`dotnet add package Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore`

Ekledikten sonra basit bir konfigürasyonla DbContext üzerinden health check mekanizmasını ayağa kaldırabiliyoruz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;ConfigureServices(<span style="color:#2b91af;">IServiceCollection</span>&nbsp;services)
{
&nbsp;&nbsp;&nbsp;&nbsp;services.AddControllers();
&nbsp;&nbsp;&nbsp;&nbsp;services.AddDbContext&lt;<span style="color:#2b91af;">BlogContext</span>&gt;(options&nbsp;=&gt;
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;options.UseSqlServer(Configuration.GetConnectionString(<span style="color:#a31515;">&quot;Blog&quot;</span>));
&nbsp;&nbsp;&nbsp;&nbsp;});
&nbsp;&nbsp;&nbsp;&nbsp;services.AddHealthChecks()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.AddDbContextCheck&lt;<span style="color:#2b91af;">BlogContext</span>&gt;();
}</pre>

Bu noktada aslında health endpointine request geldiğinde basitçe `context.Database.CanConnectAsync();` kodu çalışıyor ve bunun sonucu geriye döndürülüyor. Sadece bu kodu çalıştırmak yerine database'e sorgu gönderip bu sorgunun sonucuna göre de dışarıya sonuç raporlamak isteyebilirsiniz. Bunun için DbContextCheck metodunu çağırırken `customTestQuery` parametresini kullanabiliriz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;ConfigureServices(<span style="color:#2b91af;">IServiceCollection</span>&nbsp;services)
{
&nbsp;&nbsp;&nbsp;&nbsp;services.AddControllers();
&nbsp;&nbsp;&nbsp;&nbsp;services.AddDbContext&lt;<span style="color:#2b91af;">BlogContext</span>&gt;(options&nbsp;=&gt;
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;options.UseSqlServer(Configuration.GetConnectionString(<span style="color:#a31515;">&quot;Blog&quot;</span>));
&nbsp;&nbsp;&nbsp;&nbsp;});
&nbsp;&nbsp;&nbsp;&nbsp;services.AddHealthChecks()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.AddDbContextCheck&lt;<span style="color:#2b91af;">BlogContext</span>&gt;(customTestQuery:(context,cancellationToken)&nbsp;=&gt;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//Buraya&nbsp;context&nbsp;kullanarak&nbsp;özel&nbsp;query&nbsp;yazılabilir.&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;});
}</pre>

Microsoft tarafından geliştirilen ve desteği verilen hazır health check mekanizmaları bunlar. Bunun yanında open-source olarak geliştirilen ve pek çok farklı servisin durumunu check edebileceğimiz bir <a href="https://github.com/xabaril/AspNetCore.Diagnostics.HealthChecks" target="_blank">kütüphane</a> de bulunmakta. Örnek olarak redis tarafını check etmemiz gerekirse ilk olarak ilgili redis health check paketini projemize ekliyoruz.

`dotnet add package AspNetCore.HealthChecks.Redis`

Sonrasında aşağıdaki gibi health check mekanizmasını konfigüre edebiliriz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;ConfigureServices(<span style="color:#2b91af;">IServiceCollection</span>&nbsp;services)
{
&nbsp;&nbsp;&nbsp;&nbsp;services.AddControllers();
&nbsp;&nbsp;&nbsp;&nbsp;services.AddHealthChecks().AddRedis(Configuration[<span style="color:#a31515;">&quot;RedisConnectionString&quot;</span>]);
}</pre>

Birden fazla bağımlı servisiniz olduğunda hepsinin durumlarını ayrı ayrı check edip bunların sonucunda toplu olarak bir sonuç dönmek isteyebilirsiniz. Bunun için şu şekilde çoklu health check eklememiz mümkün. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;ConfigureServices(<span style="color:#2b91af;">IServiceCollection</span>&nbsp;services)
{
&nbsp;&nbsp;&nbsp;&nbsp;services.AddControllers();
&nbsp;&nbsp;&nbsp;&nbsp;services.AddDbContext&lt;<span style="color:#2b91af;">BlogContext</span>&gt;(options&nbsp;=&gt;
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;options.UseSqlServer(Configuration.GetConnectionString(<span style="color:#a31515;">&quot;Blog&quot;</span>));
&nbsp;&nbsp;&nbsp;&nbsp;});
&nbsp;&nbsp;&nbsp;&nbsp;services.AddHealthChecks()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.AddDbContextCheck&lt;<span style="color:#2b91af;">BlogContext</span>&gt;()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.AddRedis(Configuration.GetConnectionString(<span style="color:#a31515;">&quot;Redis&quot;</span>));
}</pre>

#### Custom Health Check Yazmak

Bazı durumlarda elimizde bulunan kütüphaneler işimizi görmeyebilir ve kendi health check kodumuzu yazmamız gerekebilir. Bunun için `IHealthCheck` interface'ini implemente eden bir class yaratıp içerisine kodumuzu yazabiliriz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">CustomHealthCheck</span>&nbsp;:&nbsp;<span style="color:#2b91af;">IHealthCheck</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">Task</span>&lt;<span style="color:#2b91af;">HealthCheckResult</span>&gt;&nbsp;CheckHealthAsync(<span style="color:#2b91af;">HealthCheckContext</span>&nbsp;context,&nbsp;<span style="color:#2b91af;">CancellationToken</span>&nbsp;cancellationToken&nbsp;=&nbsp;<span style="color:blue;">default</span>)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">HealthCheckResult</span>&nbsp;result;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//HealthCheck&nbsp;kodu</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//Başarılı&nbsp;ise&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;<span style="color:#2b91af;">HealthCheckResult</span>.Healthy();
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//Başarısız&nbsp;ise&nbsp;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;result&nbsp;=&nbsp;<span style="color:#2b91af;">HealthCheckResult</span>.Unhealthy();
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;result;
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

### Health Check Datasını Monitör Etmek

API üzerinden health check kodlarını çalıştırabildiğimiz gibi belirli aralıklarla bu kodların çalışıp sonucunun çeşitli araçlara publish edilmesini de otomatik olarak sağlayabiliriz. Bunun için  yukarıda kullanmış olduğumuz open source proje içerisinde aşağıdaki araçlara otomatik olarak health check sonuçlarını publish eden paketler bulunmakta. 

* Application Insights
* Datadog
* Prometheus
* Seq

Örneğin, health check sonuçlarını application insights'a göndermek istersek önce ilgili paketi projeye ekleyip sonra konfigüre edebiliriz. 

`dotnet add package AspNetCore.HealthChecks.Publisher.ApplicationInsights`

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;ConfigureServices(<span style="color:#2b91af;">IServiceCollection</span>&nbsp;services)
{
&nbsp;&nbsp;&nbsp;&nbsp;services.AddControllers();
&nbsp;&nbsp;&nbsp;&nbsp;services.AddDbContext&lt;<span style="color:#2b91af;">BlogContext</span>&gt;(options&nbsp;=&gt;
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;options.UseSqlServer(Configuration.GetConnectionString(<span style="color:#a31515;">&quot;Blog&quot;</span>));
&nbsp;&nbsp;&nbsp;&nbsp;});
&nbsp;&nbsp;&nbsp;&nbsp;services.AddHealthChecks()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.AddDbContextCheck&lt;<span style="color:#2b91af;">BlogContext</span>&gt;()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.AddRedis(Configuration.GetConnectionString(<span style="color:#a31515;">&quot;Redis&quot;</span>))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.AddApplicationInsightsPublisher(<span style="color:#a31515;">&quot;[Instrumentation&nbsp;Key]&quot;</span>,&nbsp;saveDetailedReport:&nbsp;<span style="color:blue;">true</span>);
}</pre>


### Health Check Datasının Azure App Service Tarafından Kullanılması

Yazının başında health check verisinin load balancerlar, container orchestratorlar tarafında kullanılabileceğinden bahsetmiştik. Azure App Service tarafında da health check API'ları load balancer tarafından instance'ın sağlıklı olup olmadığını kontrol etme amaçlı olarak kullanılmakta. Bunu konfigüre etmek için app service sayfasında monitoringin altındaki health check seçeneğine tıklıyoruz. 

![](https://az718566.vo.msecnd.net/uploads/2020/09/25/health-check-option.png)

Buradan health check seçeneğini enable edip, health check API'ını Azure App Service'e bildirebiliriz. 

![](https://az718566.vo.msecnd.net/uploads/2020/09/25/health-check-configuration.png)

Azure App Service burada verdiğimiz endpointe her iki dakikada bir requestte bulunuyor. Bu API'dan HTTP 200-299 arası bir sonuç aldığı durumda instance'ı healthy olarak düşünmekte. Aksi bir durumda instance unhealthy olarak işaretlenmekte ve load balancer içerisinden çıkarılmakta. Azure App Service instance'ı load balancerdan çıkardıktan sonra bu endpointe requestte bulunmaya devam ediyor. Eğer sonuç tekrardan HTTP 200-299 arasında bir değere geri dönerse instance tekrardan load balancera geri dönüyor. Dönmediği takdirde Azure App Service instance'ı healthy duruma getirmek için instance'a restart atıyor. 

Gördüğünüz üzere health check mekanizmaları uygulamalarımızı monitör ederken kullanabileceğimiz önemli bileşenlerden biri. Aynı zamanda load balancerlar tarafından instance'ınızın sağlıklık olup olmadığıyla ilgili kararların alınması açısından da oldukça önemli bir rol oynamakta.

Bir sonraki yazıda görüşmek üzere

Kaynaklar
* <a href="https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks?view=aspnetcore-3.1" target="_blank">https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks?view=aspnetcore-3.1</a>
* <a href="https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks" target="_blank">https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks</a>
* <a href="https://azure.github.io/AppService/2020/08/24/healthcheck-on-app-service.html" target="_blank">https://azure.github.io/AppService/2020/08/24/healthcheck-on-app-service.html</a>