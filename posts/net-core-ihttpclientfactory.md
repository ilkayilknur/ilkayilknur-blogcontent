Uygulamalarımızda en sık kullandığımız tiplerden biri de şüphesiz ki `HttpClient`. Bu yazıda ise özellikle ASP.NET Core uygulamalarımızda HttpClient tipini nasıl en doğru şekilde kullanabiliriz konusunu inceleyeceğiz. 

İlk olarak HttpClient ile ilgili doğru bilinen bir yanlıştan bahsedelim. HttpClient tipine baktığımızda `IDisposable` interface'ini implemente ettiğini görüyoruz. Bu nedenle HttpClient ile işimiz bittiğinde aynı Stream vb.. gibi tiplerde olduğu gibi dispose etmemiz gerektiğini düşünebilirsiniz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">IActionResult</span>&nbsp;Index()
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">using</span>&nbsp;(<span style="color:blue;">var</span>&nbsp;client&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">HttpClient</span>())
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;response&nbsp;=&nbsp;client.GetStringAsync(<span style="color:#a31515;">&quot;http://www.fakeapi.com&quot;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;View();
}</pre>

Bu kullanımdaki sıkıntı HttpClient nesnesi dispose olduğunda arka planda kullandığı socketin de aynı anda serbest bırakılmaması. Bu nedenle yüksek request alan API veya web sayfalarında bu şekilde bir kullanımda bulunulması socketlerin tükenmesi gibi hataların oluşmasına neden olmakta. Bu hataların önüne geçmek ve daha etkin çalışmak için uygulama içerisinde HttpClient tipini singleton veya statik kullanmak tavsiye edilen bir yöntem. <a href="https://aspnetmonsters.com/2016/08/2016-08-27-httpclientwrong/" target="_blank">YOU'RE USING HTTPCLIENT WRONG AND IT IS DESTABILIZING YOUR SOFTWARE</a>

Bu yöntemin de beraberinde getirdiği bir sorun var. O da HttpClient'ın DNS değişikliklerini algılamaması. Uzun süren processler içerisinde bu şekilde bir kullanım yaptığınızda oluşacak DNS değişikliklerinden HttpClient malesef haberdar olmuyor. Bunun nedeni HttpClient'ın default constructorının arka planda yarattığı `HttpMessageHandler`. [Github issue](https://github.com/dotnet/runtime/issues/18348)

Hem bu gibi DNS güncelleme sorunlarının önüne geçmek için hem de HttpClient konfigurasyonunu merkezileştirilmesi için .NET Core 2.1 ile beraber gelen IHttpClientClientFactory'i kullabiliriz. IHttpClientFactory'nin birden fazla farklı kullanım şekli var. Öncelikle en basit olanından başlayalım. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;ConfigureServices(<span style="color:#2b91af;">IServiceCollection</span>&nbsp;services)
{
&nbsp;&nbsp;&nbsp;&nbsp;services.AddHttpClient();
&nbsp;&nbsp;&nbsp;&nbsp;services.AddControllers();
}</pre>

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">HomeController</span>&nbsp;:&nbsp;<span style="color:#2b91af;">Controller</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:blue;">readonly</span>&nbsp;<span style="color:#2b91af;">ILogger</span>&lt;<span style="color:#2b91af;">HomeController</span>&gt;&nbsp;_logger;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:blue;">readonly</span>&nbsp;<span style="color:#2b91af;">IHttpClientFactory</span>&nbsp;factory;
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">HomeController</span>(<span style="color:#2b91af;">ILogger</span>&lt;<span style="color:#2b91af;">HomeController</span>&gt;&nbsp;logger,&nbsp;<span style="color:#2b91af;">IHttpClientFactory</span>&nbsp;factory)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;_logger&nbsp;=&nbsp;logger;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">this</span>.factory&nbsp;=&nbsp;factory;
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">IActionResult</span>&nbsp;Index()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;client&nbsp;=&nbsp;factory.CreateClient();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;response&nbsp;=&nbsp;client.GetStringAsync(<span style="color:#a31515;">&quot;http://www.fakeapi.com&quot;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;View();
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>
En basit kullanım yukarıda gördüğümüz gibi factory üzerinden CreateClient metodu ile yaratılan HttpClient'ı kullanmak. Bu şekilde bir kullanımda sürekli olarak yeni bir HttpClient yaratılıyor diye düşünebilirsiniz. Bu doğru. Ancak socket yönetimi yapılan ve DNS değişikliklerini de algılayan kısım olan HttpClientMessageHandler'lar için pooling yapılıyor ve lifetimeları otomatik olarak yönetiliyor. Bu nedenle de manuel olarak HttpClient yaratılırken oluşan sorunlar burada olmuyor. 

IHttpClientFactory'nin bir diğer kullanım şekli ise named clientlar. Basitçe anlatmamız gerekirse HttpClient'a string olarak bir isim veriyoruz ve konfigürasyonunu tanımlıyoruz. Sonrasında ise verdiğimiz ismi parametre geçerek ilgili konfigürasyondaki HttpClient'ı kullanabiliyoruz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;ConfigureServices(<span style="color:#2b91af;">IServiceCollection</span>&nbsp;services)
{
&nbsp;&nbsp;&nbsp;&nbsp;services.AddHttpClient(<span style="color:#a31515;">&quot;fakeApi&quot;</span>,&nbsp;(client)&nbsp;=&gt;
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;client.BaseAddress&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Uri</span>(<span style="color:#a31515;">&quot;https://fakeapi.com&quot;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;client.DefaultRequestHeaders.Add(<span style="color:#a31515;">&quot;Api-Version&quot;</span>,&nbsp;<span style="color:#a31515;">&quot;1.0&quot;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;});
&nbsp;&nbsp;&nbsp;&nbsp;services.AddControllers();
}</pre>

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">IActionResult</span>&nbsp;Index()
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;client&nbsp;=&nbsp;factory.CreateClient(<span style="color:#a31515;">&quot;fakeApi&quot;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;response&nbsp;=&nbsp;client.GetStringAsync(<span style="color:#a31515;">&quot;http://www.fakeapi.com&quot;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;View();
}</pre>

 Bir diğer kullanım metodu ise typed client olarak isimlendirilen kullanım. Burada HttpClient'lara string olarak key vermek yerine kendi yarattığımız bir client tipi üzerinden kullanımda bulunuyoruz. Böylece named clientları kullanırken yanlış isim verme gibi sorunların önüne geçiyoruz ve doğrudan hataları build sırasında alıyoruz.

 <pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">FakeApiClient</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">HttpClient</span>&nbsp;Client&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">FakeApiClient</span>(<span style="color:#2b91af;">HttpClient</span>&nbsp;client)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;client.BaseAddress&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Uri</span>(<span style="color:#a31515;">&quot;https://fakeapi.com&quot;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;client.DefaultRequestHeaders.Add(<span style="color:#a31515;">&quot;Api-Version&quot;</span>,&nbsp;<span style="color:#a31515;">&quot;1.0&quot;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Client&nbsp;=&nbsp;client;
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">Task</span>&lt;<span style="color:blue;">string</span>&gt;&nbsp;GetAsync()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;Client.GetStringAsync(<span style="color:#a31515;">&quot;/api/weather&quot;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;ConfigureServices(<span style="color:#2b91af;">IServiceCollection</span>&nbsp;services)
{
&nbsp;&nbsp;&nbsp;&nbsp;services.AddHttpClient&lt;<span style="color:#2b91af;">FakeApiClient</span>&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;services.AddControllers();
}</pre>

Tanımlanan typed clientın kullanımı

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">HomeController</span>&nbsp;:&nbsp;<span style="color:#2b91af;">Controller</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:blue;">readonly</span>&nbsp;<span style="color:#2b91af;">ILogger</span>&lt;<span style="color:#2b91af;">HomeController</span>&gt;&nbsp;_logger;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:blue;">readonly</span>&nbsp;<span style="color:#2b91af;">FakeApiClient</span>&nbsp;apiClient;
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">HomeController</span>(<span style="color:#2b91af;">ILogger</span>&lt;<span style="color:#2b91af;">HomeController</span>&gt;&nbsp;logger,&nbsp;<span style="color:#2b91af;">FakeApiClient</span>&nbsp;apiClient)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;_logger&nbsp;=&nbsp;logger;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">this</span>.apiClient&nbsp;=&nbsp;apiClient;
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">async</span>&nbsp;<span style="color:#2b91af;">Task</span>&lt;<span style="color:#2b91af;">IActionResult</span>&gt;&nbsp;Index()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;response&nbsp;=&nbsp;<span style="color:blue;">await</span>&nbsp;apiClient.GetAsync();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;View();
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

IHttpClientFactory aynı zamanda third-party clientlarla da kullanılabilmekte. Detaylı bilgi için [buraya](https://docs.microsoft.com/tr-tr/aspnet/core/fundamentals/http-requests?view=aspnetcore-3.1#generated-clients) bakabilirsiniz. 

IHttpClientFactory üzerinden Polly kütüphanesini kullanıp retry mekanizmalarını da yönetebiliyoruz. Bunun için öncelikle `Microsoft.Extensions.Http.Polly` nuget paketini yüklememiz gerekiyor. Sonrasında örneğin transient errorlar için aşağıdaki gibi policyler tanımlayabiliyoruz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;ConfigureServices(<span style="color:#2b91af;">IServiceCollection</span>&nbsp;services)
{
&nbsp;&nbsp;&nbsp;&nbsp;services.AddHttpClient&lt;<span style="color:#2b91af;">FakeApiClient</span>&gt;()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.AddTransientHttpErrorPolicy(t&nbsp;=&gt;&nbsp;t.WaitAndRetryAsync(3,&nbsp;(p)=&gt;<span style="color:#2b91af;">TimeSpan</span>.FromMilliseconds(500)));
&nbsp;&nbsp;&nbsp;&nbsp;services.AddControllers();
}</pre>

Polly tabanlı policyler için daha detaylı bilgi almak isterseniz <a href="https://docs.microsoft.com/en-us/aspnet/core/fundamentals/http-requests?view=aspnetcore-3.1#use-polly-based-handlers" target="_blank">buraya</a>  bakabilirsiniz.

Bu yazıda ASP.NET Core uygulamalarında HttpClient tipini en doğru şekilde nasıl kullanırız konusunu inceledik.

Bir sonraki yazıda görüşmek üzere,