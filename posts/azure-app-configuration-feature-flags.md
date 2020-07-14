Bu yazıda konumuz ASP.NET Core uygulamalarında feature flag kullanımı. Feature flagler uygulamalarımız içerisinde istediğimiz featureları açıp kapamamızı sağlayan bir practice. Hem yeni geliştirilen featureların test edilmesi hem de development esnasında feature branchlerin uzun süre ana branch üzerinden uzak kalmasını engellemesiyle bizlere merge sorunları yaşamamak adına kolaylık sağlamakta. Bu yazımız daha çok uygulama kısmı ile ilgili olduğu için detaylı bilgi almak isteyenler için <a href="https://www.martinfowler.com/articles/feature-toggles.html#TheBirthOfAFeatureFlag" target="_blank">linki</a> buraya bırakıyorum.  

### Azure App Configuration Servisi

Azure App Configuration servisi uygulamalarımız içerisindeki configleri merkezi bir yerden yönetmemizi sağlayan bir servis. Bu servisi kullanırken configleri merkezi bir yerden yönetmenin yanında tagleme, history tutma, karşılaştırma, Azure Key Vault entegrasyonu gibi özellikleriyle de işimizi oldukça kolaylaştırmakta. Bu servisin sağladığı özelliklerden biri de feature flag yönetimi. Şimdi gelin bu tarafa bir bakalım.

App Configuration servisini Azure portal üzerinde yarattıktan sonra soldaki menüden Feature Manager seçeneğine tıklayarak feature yönetimini yapabileceğimiz ekrana geçebiliriz. 


![](https://az718566.vo.msecnd.net/uploads/2020/07/14/feature-flags.png)


Bu ekrana geçtikten sonra Add butonuna tıklayarak yeni featureları tanımlayabiliyoruz. 

![](https://az718566.vo.msecnd.net/uploads/2020/07/14/add-feature-flags.png)

Yeni featureları ekledikten sonra listeleme ekranına döndüğümüzde şu şekilde bir listeyle karşılaşıyoruz. Bu liste üzerinden istediğimiz zaman ilgili featureları enable/disable edebiliyoruz.

![](https://az718566.vo.msecnd.net/uploads/2020/07/14/list-feature-flags.png)

Şimdi portali tarafını bırakıp kod tarafına geçelim ve portal üzerinde tanımladığımız featureları kod üzerinde nasıl kullanabiliyoruz ona bakalım. 

Uygulamalarımız içerisinde hem App Configuration servisi hem de feature management altyapısıyla çalışabilmemiz için aşağıdaki nuget paketlerini projemize eklememiz gerekiyor. 

* Microsoft.Azure.AppConfiguration.AspNetCore
* Microsoft.FeatureManagement.AspNetCore

İlk olarak Program.cs dosyasına gidip uygulamanın App Configurationı kullanması için gerekli olan tanımlamayı yapıyoruz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:#2b91af;">IHostBuilder</span>&nbsp;CreateHostBuilder(<span style="color:blue;">string</span>[]&nbsp;args)&nbsp;=&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Host</span>.CreateDefaultBuilder(args)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.ConfigureWebHostDefaults(webBuilder&nbsp;=&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;webBuilder.ConfigureAppConfiguration((hostingContext,&nbsp;config)&nbsp;=&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;settings&nbsp;=&nbsp;config.Build();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;config.AddAzureAppConfiguration(options&nbsp;=&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;options.Connect(settings[<span style="color:#a31515;">&quot;ConnectionStrings:AppConfig&quot;</span>])
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.UseFeatureFlags();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;});
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;})
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.UseStartup&lt;<span style="color:#2b91af;">Startup</span>&gt;());</pre>

Sonrasında uygulamamıza feature flags desteği ekliyoruz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;ConfigureServices(<span style="color:#2b91af;">IServiceCollection</span>&nbsp;services)
{
&nbsp;&nbsp;&nbsp;&nbsp;services.AddControllers();
&nbsp;&nbsp;&nbsp;&nbsp;services.AddFeatureManagement();
}</pre>

Son olarak ise feature flaglerin runtimeda otomatik olarak güncellenmesi için gerekli olan middleware'ı ekliyoruz ve konfigürasyon aşamasını bitiriyoruz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Configure(<span style="color:#2b91af;">IApplicationBuilder</span>&nbsp;app,&nbsp;<span style="color:#2b91af;">IWebHostEnvironment</span>&nbsp;env)
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>&nbsp;(env.IsDevelopment())
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app.UseDeveloperExceptionPage();
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">else</span>
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app.UseExceptionHandler(<span style="color:#a31515;">&quot;/Home/Error&quot;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;The&nbsp;default&nbsp;HSTS&nbsp;value&nbsp;is&nbsp;30&nbsp;days.&nbsp;You&nbsp;may&nbsp;want&nbsp;to&nbsp;change&nbsp;this&nbsp;for&nbsp;production&nbsp;scenarios,&nbsp;see&nbsp;https://aka.ms/aspnetcore-hsts.</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;app.UseHsts();
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;app.UseHttpsRedirection();
&nbsp;&nbsp;&nbsp;&nbsp;app.UseStaticFiles();
 
&nbsp;&nbsp;&nbsp;&nbsp;app.UseRouting();
 
&nbsp;&nbsp;&nbsp;&nbsp;app.UseAuthorization();
 
&nbsp;&nbsp;&nbsp;&nbsp;app.UseEndpoints(endpoints&nbsp;=&gt;
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;endpoints.MapControllerRoute(
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;name:&nbsp;<span style="color:#a31515;">&quot;default&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;pattern:&nbsp;<span style="color:#a31515;">&quot;{controller=Home}/{action=Index}/{id?}&quot;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;});
&nbsp;&nbsp;&nbsp;&nbsp;app.UseAzureAppConfiguration();
}</pre>

Bu konfigürasyonla beraber artık asp.net core uygulamamıza hem config store hem de feature flag store olarak App Configuration servisini kullanacağımızı belirttik. Lokalde çalışırken configler için secret managerı kullanabilir, production ortamına geçerkende `ConnectionStrings:AppConfig` config değerine app configuration servisinin connection stringini koyarak kullanabiliriz. 

Şimdi gelelim feature flag kullanım kısmına. En basit kullanımla uygulamalarımız içerisinde IFeatureManager'ı kullanarak feature'ın enabled olup olmadığı kontrol ederek gerekli aksiyonları alabiliyoruz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">HomeController</span>&nbsp;:&nbsp;<span style="color:#2b91af;">Controller</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:blue;">readonly</span>&nbsp;<span style="color:#2b91af;">IFeatureManager</span>&nbsp;featureManager;
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">HomeController</span>(<span style="color:#2b91af;">IFeatureManager</span>&nbsp;featureManager)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">this</span>.featureManager&nbsp;=&nbsp;featureManager;
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">async</span>&nbsp;<span style="color:#2b91af;">Task</span>&lt;<span style="color:#2b91af;">IActionResult</span>&gt;&nbsp;Index()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>&nbsp;(<span style="color:blue;">await</span>&nbsp;featureManager.IsEnabledAsync(<span style="color:#a31515;">&quot;billingv3&quot;</span>))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;View(<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">HomeViewModel</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Title&nbsp;=&nbsp;<span style="color:#a31515;">&quot;You&#39;re&nbsp;using&nbsp;the&nbsp;new&nbsp;billing&nbsp;system!&quot;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;});
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">else</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;View(<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">HomeViewModel</span>()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Title&nbsp;=&nbsp;<span style="color:#a31515;">&quot;Welcome!&nbsp;Our&nbsp;new&nbsp;billing&nbsp;system&nbsp;coming&nbsp;soon!&quot;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;});
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

Bu kullanımın yanında FeatureGate attribute'ünü kullanarak controller veya içerisindeki actionların belirli feature flaglerde çalışmasını sağlayabiliriz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">enum</span>&nbsp;<span style="color:#2b91af;">FeatureFlags</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;BillingV2,
&nbsp;&nbsp;&nbsp;&nbsp;BillingV3
}</pre>

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;">[<span style="color:#2b91af;">FeatureGate</span>(<span style="color:#2b91af;">FeatureFlags</span>.BillingV3)]
<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">BillingController</span>&nbsp;:&nbsp;<span style="color:#2b91af;">Controller</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">IActionResult</span>&nbsp;Index()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;View();
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

Yada sadece action bazlı olarak uygularsak.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">BillingController</span>&nbsp;:&nbsp;<span style="color:#2b91af;">Controller</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;[<span style="color:#2b91af;">FeatureGate</span>(<span style="color:#2b91af;">FeatureFlags</span>.BillingV3)]
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">IActionResult</span>&nbsp;Index()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;View();
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

View tarafına geçtiğimizde ise feature tagini kullanarak feature bazlı content gösterebiliyoruz. Bunun için ilk olarak _ViewImports.cshtml dosyasını açıp gerekli olan tag helperı eklememiz gerekiyor. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="background:yellow;">@addTagHelper</span>&nbsp;<span style="color:#a31515;">*,&nbsp;Microsoft.FeatureManagement.AspNetCore</span>
</pre>

Artık viewlar içerisinde feature bazlı contenti aşağıdaki gibi gösterebiliriz. 
<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">&lt;</span><span style="font-weight:bold;color:purple;">feature</span>&nbsp;<span style="font-weight:bold;color:purple;">name</span><span style="color:blue;">=</span><span style="color:blue;">&quot;BillingV3&quot;</span>&nbsp;<span style="font-weight:bold;color:purple;">negate</span><span style="color:blue;">=</span><span style="color:blue;">&quot;</span><span style="color:blue;">true</span><span style="color:blue;">&quot;</span><span style="color:blue;">&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">&lt;</span><span style="color:maroon;">p</span><span style="color:blue;">&gt;</span>You&#39;re&nbsp;using&nbsp;the&nbsp;new&nbsp;billing&nbsp;system!<span style="color:blue;">&lt;/</span><span style="color:maroon;">p</span><span style="color:blue;">&gt;</span>
<span style="color:blue;">&lt;/</span><span style="font-weight:bold;color:purple;">feature</span><span style="color:blue;">&gt;</span></pre>

Feature taginde çoklu feature kontrolü yapmamız da mümkün.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">&lt;</span><span style="font-weight:bold;color:purple;">feature</span>&nbsp;<span style="font-weight:bold;color:purple;">name</span><span style="color:blue;">=</span><span style="color:blue;">&quot;BillingV3,BillingV2&quot;</span>&nbsp;<span style="font-weight:bold;color:purple;">requirement</span><span style="color:blue;">=</span><span style="color:blue;">&quot;</span>All<span style="color:blue;">&quot;</span><span style="color:blue;">&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">&lt;</span><span style="color:maroon;">p</span><span style="color:blue;">&gt;</span>You&#39;re&nbsp;using&nbsp;the&nbsp;new&nbsp;billing&nbsp;system!<span style="color:blue;">&lt;/</span><span style="color:maroon;">p</span><span style="color:blue;">&gt;</span>
<span style="color:blue;">&lt;/</span><span style="font-weight:bold;color:purple;">feature</span><span style="color:blue;">&gt;</span></pre>

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">&lt;</span><span style="font-weight:bold;color:purple;">feature</span>&nbsp;<span style="font-weight:bold;color:purple;">name</span><span style="color:blue;">=</span><span style="color:blue;">&quot;BillingV3,BillingV2&quot;</span>&nbsp;<span style="font-weight:bold;color:purple;">requirement</span><span style="color:blue;">=</span><span style="color:blue;">&quot;</span>Any<span style="color:blue;">&quot;</span><span style="color:blue;">&gt;</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">&lt;</span><span style="color:maroon;">p</span><span style="color:blue;">&gt;</span>You&#39;re&nbsp;using&nbsp;the&nbsp;new&nbsp;billing&nbsp;system!<span style="color:blue;">&lt;/</span><span style="color:maroon;">p</span><span style="color:blue;">&gt;</span>
<span style="color:blue;">&lt;/</span><span style="font-weight:bold;color:purple;">feature</span><span style="color:blue;">&gt;</span></pre>

Eğer feature bazlı middleware eklemek istersek de şu şekilde bir kullanımda bulunabiliriz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;">app.UseMiddlewareForFeature&lt;VersionMiddleware&gt;(<span style="color:#a31515;">&quot;billingv3&quot;</span>);
</pre>

### Feature Flag Filters

Feature flagleri kullanırken built-in gelen filterları kullanabilir yada kendi filterlarımızı tanımlayabiliriz. Şu anda built-in olarak gelen iki filter var. Bunlar yüzde kaç oranında feature'ın enabled olacağını belirleyen `Microsoft.Percentage` diğeri de geçerlilik süresini belirten `Microsoft.TimeWindow`.

![](https://az718566.vo.msecnd.net/uploads/2020/07/14/feature-flags-filter.gif)


Bu yazıda ASP.NET Core uygulamalarında feature flag kullanımı ve Azure App Configuration servisiyle bu feature flaglerin yönetimini inceledik. Gördüğünüz gibi uygulamalarımıza feature flag yapısını entegre etmemiz oldukça kolay. Azure App Configuration servisini kullanmasanızda eklemiş olduğumuz nuget paketiyle config bazlı feature flag implementasyonu yapabilirsiniz. 

Bir sonraki yazıda görüşmek üzere,