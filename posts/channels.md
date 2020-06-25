Herkese Selamlar,

Bu yazıda `System.Threading` namespace'i altına bulunan `Channel<T>` tipini inceleyeceğiz. .NET Core 2.1 ile beraber gelen bu tip, en basit anlatımla uygulamalarımız içerisinde bir noktadan bir başka noktaya veri gönderebilmemizi sağlanmakta. Bir diğer anlatımla da uygulamalarımızda producer/consumer implementasyonları gerçekleştirirken kullanabileceğimiz bir veri yapısı olarak da düşünebiliriz.

Channel yaratabilmek için `Channel` tipi içerisinde iki adet statik metot bulunmakta. 

* `Channel.CreateBounded<T>(int capacity)` : Bu metot ile parametrede belirtilen eleman sayısı kadar eleman taşıyabilecek bir channel yaratılır.
* `Channel.CreateUnbounded<T>()`: Bu metot ile de sınırsız sayıda eleman taşıyabilecek bir channel yaratılır.  

Her iki metotta da generic parametre olarak channel içerisinde taşıyacağınız data tipini belirtmeniz gerekiyor. 

Yukarıdaki metotları kullanarak bir channel yarattıktan sonra channel içerisindeki Reader ve Writer propertyleri içerisindeki metotları kullanarak ilgili işlemleri yapabiliyoruz. Şimdi isterseniz basit bir channel yaratıp kullanımını inceleyelim. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Main(<span style="color:blue;">string</span>[]&nbsp;args)
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;channel&nbsp;=&nbsp;<span style="color:#2b91af;">Channel</span>.CreateUnbounded&lt;<span style="color:blue;">int</span>&gt;();
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//Producer</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">_</span>&nbsp;=&nbsp;<span style="color:#2b91af;">Task</span>.Run(<span style="color:blue;">async</span>&nbsp;()&nbsp;=&gt;
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">int</span>&nbsp;number&nbsp;=&nbsp;1;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">while</span>&nbsp;(<span style="color:blue;">true</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">await</span>&nbsp;channel.Writer.WriteAsync(number);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;number++;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">await</span>&nbsp;<span style="color:#2b91af;">Task</span>.Delay(1000);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;});
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//Consumer</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">_</span>&nbsp;=&nbsp;<span style="color:#2b91af;">Task</span>.Run(<span style="color:blue;">async</span>&nbsp;()&nbsp;=&gt;
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">while</span>&nbsp;(<span style="color:blue;">true</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;data&nbsp;=&nbsp;<span style="color:blue;">await</span>&nbsp;channel.Reader.ReadAsync();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Console</span>.WriteLine(<span style="color:#a31515;">$&quot;Data:</span>{data}<span style="color:#a31515;">&quot;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">await</span>&nbsp;<span style="color:#2b91af;">Task</span>.Delay(3000);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;});
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Console</span>.ReadKey();
}</pre>

Yukarıda channelı yaratıp sonrasında ilk olarak producer kısmı yazdık. Burada Writer içerisindeki WriteAsync metodunu kullanarak 1.sn aralıklarla datayı channela gönderdik. Sonrasında ise consumer tarafta Reader propertysi içerisindeki ReadAsync metoduyla channeldaki datayı 3'er saniye aralıklarla okuyup console'a yazdırdık. 

![](https://ilkayblog.blob.core.windows.net/uploads/2020/06/25/channel.gif)

Kapasite limiti olmayan channelların basitçe kullanımı bu şekilde. Ancak kapasite limiti olan channelların(bounded) kullanımında bazı senaryolar var. Bu senaryolarda etkilenen taraf channela data yazan kısım. Şimdi gelin bu tip durumlarda writer metotların davranışlarını inceleyelim. 

* `WriteAsync` : Eğer channel içerisindeki eleman sayısı belirtilen kapasite limitine ulaşmadıysa channela datayı yazarak çağıran tarafa geri döner. Ancak eleman sayısı kapasite limitine ulaştıysa bu metot channeldan data okunana kadar bekler ve çağıran tarafa geri dönmez. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;channel&nbsp;=&nbsp;<span style="color:#2b91af;">Channel</span>.CreateBounded&lt;<span style="color:blue;">int</span>&gt;(1);
<span style="color:blue;">await</span>&nbsp;channel.Writer.WriteAsync(1);
<span style="color:green;">//Yeni&nbsp;bir&nbsp;read&nbsp;gelene&nbsp;kadar&nbsp;bekler.</span>
<span style="color:blue;">await</span>&nbsp;channel.Writer.WriteAsync(2);</pre>

* `TryWrite`: Bu metot senkron olarak çalıştığı için eğer channelda yeterli yer varsa yazıp true değeri döner. Yer yoksa doğrudan false değeri döner. 

* `WaitToWriteAsync` : Bu metot channela herhangi bir değer yazmaz ancak channelda yazmak için yeterli yer olduğunda sizi haberdar eder. Özellikle while loop içerisinde condition olarak bu metot kullanılabilir. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;channel&nbsp;=&nbsp;<span style="color:#2b91af;">Channel</span>.CreateBounded&lt;<span style="color:blue;">int</span>&gt;(1);
<span style="color:blue;">while</span>&nbsp;(<span style="color:blue;">await</span>&nbsp;channel.Writer.WaitToWriteAsync())
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">await</span>&nbsp;channel.Writer.WriteAsync(1);
}</pre>

Default olarak bounded channel kullanımında writer tarafta olan değişimler bu şekilde. Ancak bounded channel yaratırken channel içerisinde kapasite dolduğunda channelın davranışını değiştirme imkanımız da var. Bunun için channel yaratırken parametre geçebildiğimiz `BoundedChannelOptions` içindeki FullMode propertysini kullanabiliyoruz. Bu propertynin alabildiği değerler şunlar. 

* `Wait(default)`: Eğer channel içerisinde kapasiteye ulaşılırsa channeldan data okunana kadar write operasyonu bekler. 
* `DropNewest` : Channelda kapasite dolduğunda en son eklenen eleman silinerek channelda yer açılır. 
* `DropOldest` : Channelda kapasite dolduğunda en eski eklenen eleman silinerek channelda yer açılır.
* `DropWrite` : Channelda kapasite dolduğunda yapılan write operasyonları yoksayılır. 

### Küçük Optimizasyonlar

Eğer yarattığınız channelların tek bir readerı veya writerı olacaksa bunları channel yaratırken belirtirseniz arka planda daha etkin ve verimli bir implementasyon kullanılmasını sağlayabilirsiniz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;channel&nbsp;=&nbsp;<span style="color:#2b91af;">Channel</span>.CreateBounded&lt;<span style="color:blue;">int</span>&gt;(<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">BoundedChannelOptions</span>(1)
{
&nbsp;&nbsp;&nbsp;&nbsp;SingleReader=<span style="color:blue;">true</span>,
&nbsp;&nbsp;&nbsp;&nbsp;SingleWriter&nbsp;=&nbsp;<span style="color:blue;">true</span>,
});
 
<span style="color:blue;">var</span>&nbsp;channel2&nbsp;=&nbsp;<span style="color:#2b91af;">Channel</span>.CreateUnbounded&lt;<span style="color:blue;">int</span>&gt;(<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">UnboundedChannelOptions</span>()
{
&nbsp;&nbsp;&nbsp;&nbsp;SingleReader&nbsp;=&nbsp;<span style="color:blue;">true</span>,
&nbsp;&nbsp;&nbsp;&nbsp;SingleWriter&nbsp;=&nbsp;<span style="color:blue;">true</span>,
});</pre>

### Channelların C# 8.0 Async Enumerables İle Kullanımı

Channeldan data okurken C# 8.0 ile beraber gelen async enumerables özelliğini kullanabiliyoruz. 

Örneğin,

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;channel&nbsp;=&nbsp;<span style="color:#2b91af;">Channel</span>.CreateUnbounded&lt;<span style="color:blue;">int</span>&gt;();
<span style="color:green;">//Producer</span>
<span style="color:blue;">_</span>&nbsp;=&nbsp;<span style="color:#2b91af;">Task</span>.Run(<span style="color:blue;">async</span>&nbsp;()&nbsp;=&gt;
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">int</span>&nbsp;number&nbsp;=&nbsp;1;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">while</span>&nbsp;(<span style="color:blue;">true</span>)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">await</span>&nbsp;channel.Writer.WriteAsync(number);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;number++;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">await</span>&nbsp;<span style="color:#2b91af;">Task</span>.Delay(1000);
&nbsp;&nbsp;&nbsp;&nbsp;}
});
 
<span style="color:blue;">await</span>&nbsp;<span style="color:blue;">foreach</span>&nbsp;(<span style="color:blue;">var</span>&nbsp;item&nbsp;<span style="color:blue;">in</span>&nbsp;channel.Reader.ReadAllAsync())
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Console</span>.WriteLine(item);
}</pre>

`ReadAllAsync` metodu `IAsyncEnumerable` donüyor ve channelda data olduğu sürece foreach döngüsüyle okuyabiliyoruz. Eğer channelda data yoksa yenisi gelene kadar bekliyoruz ve foreach döngüsü sonlanmıyor. Ancak writer üzerinden Complete metodu çağırılırsa channela daha fazla eleman eklenmeyeceği mesajı verilir ve bundan sonra foreach metodu channeldaki tüm elemanları işledikten sonra sonlanır. 

Örneğin,

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;channel&nbsp;=&nbsp;<span style="color:#2b91af;">Channel</span>.CreateUnbounded&lt;<span style="color:blue;">int</span>&gt;();
<span style="color:green;">//Producer</span>
<span style="color:blue;">_</span>&nbsp;=&nbsp;<span style="color:#2b91af;">Task</span>.Run(<span style="color:blue;">async</span>&nbsp;()&nbsp;=&gt;
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">int</span>&nbsp;number&nbsp;=&nbsp;1;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">while</span>&nbsp;(<span style="color:blue;">true</span>)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">await</span>&nbsp;channel.Writer.WriteAsync(number);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;number++;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>&nbsp;(number&nbsp;==&nbsp;10)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;channel.Writer.Complete();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">await</span>&nbsp;<span style="color:#2b91af;">Task</span>.Delay(1000);
&nbsp;&nbsp;&nbsp;&nbsp;}
});
 
<span style="color:blue;">await</span>&nbsp;<span style="color:blue;">foreach</span>&nbsp;(<span style="color:blue;">var</span>&nbsp;item&nbsp;<span style="color:blue;">in</span>&nbsp;channel.Reader.ReadAllAsync())
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Console</span>.WriteLine(item);
}
 
<span style="color:#2b91af;">Console</span>.WriteLine(<span style="color:#a31515;">&quot;Done&quot;</span>);
<span style="color:#2b91af;">Console</span>.ReadKey();</pre>

![](https://ilkayblog.blob.core.windows.net/uploads/2020/06/25/channel2.gif)

Channellar .NET Core ve ASP.NET Core içerisinde de belirli yerlerde kullanılmakta. Siz de örneğin asp.net core içerisinde background servislere data gönderme gibi alanlarda bu tipten yararlanabilirsiniz. 

Bir sonraki yazıda görüşmek üzere,