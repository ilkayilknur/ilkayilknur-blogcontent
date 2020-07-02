.NET ekosistemi içerisinde JSON işlemleri denince akla gelen ilk kütüphane Newtonsoft.Json kütüphanesi. Microsoft yıllar boyunca .NET içerisinde yüksek  performanslı ve gömülü bir çözüm  geliştirmek yerine bu kütüphaneyi hem kullandı hem de developerlara tavsiye etti. Ancak .NET Core 3.0 ile bu hikaye biraz değişti. .NET Core 3.0 ile beraber Microsoft JSON operasyonları için yeni API'lar implemente etti ve artık ASP.NET Core içerisinde de bu API'ları kullanarak Newtonsoft.Json kütüphanesine olan bağımlılıktan kurtuldu. Bu kütüphane özellikle son zamanlarda gelen Span, Memory gibi tipleri alt tarafta etkin bir şekilde kullanarak oldukça performanslı ve daha az memory kullanan bir altyapıyı bizlere sunuyor. Şimdi gelin bu yeni API'lara biraz yakından bakalım. 

Yeni API'ların bulunduğu namespaceler şunlar.

* `System.Text.Json`
* `System.Text.Json.Serialization`

En basit anlamda serialize/deserialize işlemleri için `JsonSerializer` tipi içerisindeki static Serialize/Deserialize metotlarını kullanabiliriz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;person&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Person</span>()
{
&nbsp;&nbsp;&nbsp;&nbsp;Id&nbsp;=&nbsp;1,
&nbsp;&nbsp;&nbsp;&nbsp;Name&nbsp;=&nbsp;<span style="color:#a31515;">&quot;ilkay&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;Surname&nbsp;=&nbsp;<span style="color:#a31515;">&quot;ilknur&quot;</span>
};
 
<span style="color:blue;">var</span>&nbsp;json&nbsp;=&nbsp;<span style="color:#2b91af;">JsonSerializer</span>.Serialize(person);
<span style="color:blue;">var</span>&nbsp;obj&nbsp;=&nbsp;<span style="color:#2b91af;">JsonSerializer</span>.Deserialize&lt;<span style="color:#2b91af;">Person</span>&gt;(json);</pre>

Serialize/deserialize işlemleri sırasında default olan bazı değerleri değiştirmek istediğimizde `JsonSerializerOptions` nesnesi yaratıp ilgili metotlara parametre olarak geçebiliriz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:#2b91af;">JsonSerializerOptions</span>&nbsp;options&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">JsonSerializerOptions</span>()
{
&nbsp;&nbsp;&nbsp;&nbsp;WriteIndented&nbsp;=&nbsp;<span style="color:blue;">true</span>,
&nbsp;&nbsp;&nbsp;&nbsp;PropertyNamingPolicy&nbsp;=&nbsp;<span style="color:#2b91af;">JsonNamingPolicy</span>.CamelCase
};
<span style="color:blue;">var</span>&nbsp;json&nbsp;=&nbsp;<span style="color:#2b91af;">JsonSerializer</span>.Serialize(person,options);
<span style="color:blue;">var</span>&nbsp;obj&nbsp;=&nbsp;<span style="color:#2b91af;">JsonSerializer</span>.Deserialize&lt;<span style="color:#2b91af;">Person</span>&gt;(json,options);</pre>


JsonSerializer tipi aynı zamanda stream bazlı serialize/deserialize işlemleri için asenkron metotlar bulundurmakta.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;stream&nbsp;=&nbsp;<span style="color:#2b91af;">File</span>.OpenRead(<span style="color:#a31515;">&quot;c:</span><span style="color:#b776fb;">\\</span><span style="color:#a31515;">person.json&quot;</span>);
<span style="color:blue;">var</span>&nbsp;person&nbsp;=&nbsp;<span style="color:blue;">await</span>&nbsp;<span style="color:#2b91af;">JsonSerializer</span>.DeserializeAsync&lt;<span style="color:#2b91af;">Person</span>&gt;(stream);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
<span style="color:blue;">await</span>&nbsp;<span style="color:#2b91af;">JsonSerializer</span>.SerializeAsync(stream,&nbsp;person);
</pre>

Yeni JSON API'larındaki dikkat çeken implementasyonlardan biri de Utf-8 byte array'e doğrudan çevirme opsiyonu.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">byte</span>[]&nbsp;data&nbsp;=&nbsp;<span style="color:#2b91af;">JsonSerializer</span>.SerializeToUtf8Bytes(person);
</pre>

JSON serialize/deserialize operasyonları esnasında belli propertylerin ignore edilmesi veya farklı isimle temsil edilmesi gibi işlemleri aynı newtonsoft.json kütüphanesinde olduğu gibi attributeler aracılığıyla yapabiliyoruz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Person</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;[<span style="color:#2b91af;">JsonIgnore</span>]
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">int</span>&nbsp;Id&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">set</span>;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Name&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">set</span>;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Surname&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">set</span>;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;[<span style="color:#2b91af;">JsonPropertyName</span>(<span style="color:#a31515;">&quot;ttl&quot;</span>)]
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">int</span>&nbsp;TimeToLive&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">set</span>;&nbsp;}
}</pre>

Enumları string olarak serialize/deserialize etmek istersek built-in gelen `JsonStringEnumConverter` converterını kullanabiliriz.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;person&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Person</span>()
{
&nbsp;&nbsp;&nbsp;&nbsp;Id&nbsp;=&nbsp;1,
&nbsp;&nbsp;&nbsp;&nbsp;Name&nbsp;=&nbsp;<span style="color:#a31515;">&quot;ilkay&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;Surname&nbsp;=&nbsp;<span style="color:#a31515;">&quot;ilknur&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;Role&nbsp;=&nbsp;<span style="color:#2b91af;">Roles</span>.Admin
};
<span style="color:blue;">var</span>&nbsp;serializerOptions&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">JsonSerializerOptions</span>();
serializerOptions.Converters.Add(<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">JsonStringEnumConverter</span>());
<span style="color:blue;">var</span>&nbsp;json&nbsp;=&nbsp;<span style="color:#2b91af;">JsonSerializer</span>.Serialize(person,&nbsp;serializerOptions);</pre>

### ASP.NET Core İçerisinde Kullanım

ASP.NET Core 3.0 ile beraber artık dışarıdan özellikle belirtmediyseniz alt katmanlarda bahsettiğimiz yeni Json API'ları kullanılıyor. Bu nedenle serialize/deserialize operasyonlarında değiştirmek istediğimiz detaylar olduğunda şu şekilde konfigüre edebiliriz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;ConfigureServices(<span style="color:#2b91af;">IServiceCollection</span>&nbsp;services)
{
&nbsp;&nbsp;&nbsp;&nbsp;services.AddControllers()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.AddJsonOptions(options&nbsp;=&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;options.JsonSerializerOptions.Converters.Add(<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">JsonStringEnumConverter</span>());
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;});
}</pre>

API'larınızda eğer yeni gelen System.Text.Json altyapısına geçmeye hazır değilseniz Newtonsoft.Json kütüphanesini kullanma imkanınız da mevcut. Bunun için öncelikle `Microsoft.AspNetCore.Mvc.NewtonsoftJson` paketini yükleyip sonrasında aşağıdaki gibi konfigüre etmeniz gerekiyor. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;ConfigureServices(<span style="color:#2b91af;">IServiceCollection</span>&nbsp;services)
{
&nbsp;&nbsp;&nbsp;&nbsp;services.AddControllers()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;.AddNewtonsoftJson();
}</pre>
### Dikkat Çeken Diğer Özellikler
Yeni Json API'larında dikkatimi çeken ancak bu yazıda bahsetmeyeceğim özelliklerden ikisi <a href="https://docs.microsoft.com/en-us/dotnet/standard/serialization/system-text-json-how-to#use-utf8jsonwriter" target="_blank">Utf8JsonWriter ve Utf8JsonReader</a> yapısı. Özellikle belirli senaryolarda performanslı ve az memory kullanarak belirli işlemleri gerçekleştirmek mümkün. Eğer sizin de ilginizi çektiyse linke tıklayıp inceleyebilirsiniz.

### Gözümüz Kapalı Bu Kütüphaneye Geçmeli Miyiz?

Gelelim en kritik soruya :) Bahsettiğimiz JSON API'ları takdir edersiniz ki Newtonsoft.Json kütüphanesine göre oldukça yeni. Newtonsoft.Json kütüphanesine baktığımızda oldukça eski ve olgun bir kütüphane olduğunu görüyoruz. Yıllar boyunca pek çok özelliği implemente etti bu nedenle aradığımız bazı özellikler haliyle bu yeni API'larda henüz olmayabilir. Microsoft bunun için çok güzel bir <a href="https://docs.microsoft.com/en-us/dotnet/standard/serialization/system-text-json-migrate-from-newtonsoft-how-to" target="_blank">döküman</a> hazırlayıp hangi özellikler var, yok  olmayan özelliklerle ilgili nasıl workaround yapabiliriz gibi konulara cevap veriyor. Bu dökümana göre bakıp ilerlemekte fayda var. Duruma göre değişmekle beraber doğrudan kütüphane güncellemesiyle tamamlanacak bir geçiş olmayacağını söyleyebiliriz. Ancak yeni geliştirdiğimiz uygulamalarda artık doğrudan Newtonsoft.Json kütüphanesi yerine yeni API'ları kullanmakta fayda var diye düşünüyorum. .NET 5 ile beraber JSON API'larının biraz daha genişlemesiyle ve olgunlaşmasıyla beraber de geçişlerin artacağını da tahmin etmek zor olmayacaktır. 

Bir sonraki yazıda görüşmek üzere,