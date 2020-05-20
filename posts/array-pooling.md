Bir önceki yazıda <a href="https://ilkayilknur.com/object-pooling-nedir-net-core-icerisinde-nasil-kullanilir" target="_blank">object pooling</a> konusundan ve .NET Core içerisinde nasıl kullanabileceğimizden bahsetmiştik. Bu yazının konusu da array pooling. Peki array pooling konusu nereden çıktı diye bakarsak. :) <a href="https://docs.microsoft.com/en-us/aspnet/core/performance/performance-best-practices?view=aspnetcore-3.1" target="_blank">ASP.NET Core best practices</a> dökümanına baktığımızda şu şekilde bir tavsiye ile karşılaşıyoruz. 

> Do pool buffers by using an ArrayPool<T> to store large arrays.

Peki bu tavsiyenin esas sebebi ne? Uygulamalarımız içerisinde herhangi bir nesne yarattığımızda GC(Garbage Collection) bu nesnelerin bellekte yaratılmasından ve kullanıldıktan sonra da bellekten silinmesinden sorumlu. Garbage collection belleği temel anlamda iki parça şeklinde yönetiyor. Bunlardan biri Small Object Heap diğeri ise Large Object Heap. Small object heap büyük oranda yaratılan nesnelerin saklandığı segment. Bu alan Gen 0, Gen 1, Gen 2 diye bölümlere ayrılmış durumda. Large object heap ise 85 000 byte ve üzerindeki nesnelerin saklanması için özel olarak ayrılmış olan kısım. Yani yarattığınız nesne eğer 85 000 byte'tan büyük ise bu nesne otomatik olan large object heapte yaratılıyor. Peki large object heap'te yeteri kadar yer kalmadığında GC ne yapıyor? GC memory üzerinden toptan bir temizleme işlemi başlatıyor,  small ve large object heap üzerinde toptan bir bellek temizleme işlemi gerçekleştiriyor. Bu oldukça zaman alan ve uygulamalarımızın performansına zarar veren bir işlem. Bu nedenle bu duruma düşmemek ve bundan kaçınmak oldukça önemli. Yani GC için ne kadar az iş çıkarsak o kadar iyi diyebiliriz. Garbage collection konusutabi ki oldukça karmaşık bir konu. Daha detaylı bilgi almak isterseniz <a href="https://docs.microsoft.com/en-us/dotnet/standard/garbage-collection/" target="_blank">buradaki</a> dökümana bakabilirsiniz.

Aslında bizim uygulamalarımız içerisinde yarattığımız nesnelerin büyük çoğunluğu zaten 85 000 bytetan ufak olan nesneler. Bu değerden büyük olan ve large object heapte saklanması gereken nesneler genel anlamda arrayler. Yani siz kodunuz içerisinde sık çalışan bir yerde bu şekilde büyük ölçekte arraylerle çalışmanız gerekiyorsa bir yerden sonra GC üzerinde yük yaratmaya başlayabilirsiniz. Bu nedenle de işte dökümanlarda özellikle büyük arrayler için pooling kullanılması önerilmekte. Pooling implementasyonuyla amaçlanan arraylerin paylaşılması ve large object heap dediğimiz segmentin daha efektif kullanılması ve GC üzerindeki baskının azaltılması. Şimdi gelin array poolingi nasıl kullanabiliriz ona bakalım. 

.NET Core içerisinde array pooling için kullanabileceğimiz `ArrayPool<T>` tipi `System.Buffers` namespace'i içerisinde bulunmakta. Bu tipin içerisindeki `Shared` propertysini kullanarak paylaşılan poola ulaşabiliyoruz. Poola ulaştıktan sonra zaten kullanabileceğimiz 2 metot bulunmakta. Bunlarda biri **Rent** diğeri de **Return**.

<pre style="font-family:Consolas;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;array&nbsp;=&nbsp;<span style="color:#2b91af;">ArrayPool</span>&lt;<span style="color:blue;">int</span>&gt;.Shared.Rent(10);
</pre>

Return methodu parametre olarak istediğiniz arrayin minimum kaç elemanlı olacağını sizden istiyor. Yani buradan dönecek olan arrayin boyutu sizin verdiğiniz değerden büyük olabilir. Bu nedenle kodunuzda buna göre değişiklikler yapmanız gerekebilir. Return metodu ise biri opsiyonel olmak üzere 2 parametre almakta. 

<pre style="font-family:Consolas;color:black;background:white;"><span style="color:#2b91af;">ArrayPool</span>&lt;<span style="color:blue;">int</span>&gt;.Shared.Return(array,&nbsp;clearArray:&nbsp;<span style="color:blue;">false</span>);
</pre>

Return metodunun ilk parametresi poola döndürülecek olan array olurken 2. parametre arrayin poola geri döndürülürken içeriğinin temizlenip temizlenmeyeceğini belirtiyor. Bu parametrenin default değeri false. Bu şekilde bırakırsanız Rent metodundan dönecek olan array içeriği bir önceki kullanımdan kalan değerlerle dolu olabilir. Bu nedenle bu konuya dikkat etmekte fayda var.

Tüm pooling mekanizmalarında olduğu gibi pooldan aldığımız arrayi işimiz bittikten sonra poola geri koymamız oldukça önemli. Herhangi bir exception vs.. gibi durumlarında eğer arrayi poola geri bırakmazsak memory leak gibi problemler karşılaşmamız oldukça olası. 

ArrayPool tipini kullanırken mümkün olduğu kadar Shared poolu kullanmakta fayda var. Ancak bazı durumlarda özel bir array pool yaratmak gerekebilir. Örneğin Shared pool içerisinde bulunan arraylerin maximum büyüklüğü 2^20(1024*1024). Eğer bu değerlerden farklı olarak bir ihtiyacımız varsa özel bir pool yaratmamız gerekebilir. Bunun için de `ArrayPool<T>.Create` metodunu kullanabiliriz.

Object pooling yazısının en sonunda da bahsettiğim gibi bu tür optimizasyonları yaparken mutlaka benchmark testlerini yaparak kullanmakta fayda var. Örneğin 10 elemanlı bir array için aşağıdaki gibi bir benchmark testi yazdığımızda bakalım nasıl bir sonuçla karşılaşacağız. 

<pre style="font-family:Consolas;color:black;background:white;">[<span style="color:#2b91af;">MemoryDiagnoser</span>]
<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Runner</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">const</span>&nbsp;<span style="color:blue;">int</span>&nbsp;arraySize&nbsp;=&nbsp;10;
&nbsp;&nbsp;&nbsp;&nbsp;[<span style="color:#2b91af;">Benchmark</span>(Baseline&nbsp;=&nbsp;<span style="color:blue;">true</span>)]
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;New()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;array&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:blue;">byte</span>[arraySize];
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;[<span style="color:#2b91af;">Benchmark</span>]
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Pool()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;array&nbsp;=&nbsp;<span style="color:#2b91af;">ArrayPool</span>&lt;<span style="color:blue;">byte</span>&gt;.Shared.Rent(arraySize);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">ArrayPool</span>&lt;<span style="color:blue;">byte</span>&gt;.Shared.Return(array);
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>


![](https://ilkayblog.blob.core.windows.net/uploads/2020/05/20/smallArrayBenchmark.png)

Array'in eleman sayısını 1000'e çektiğimizde ise karşımıza şu sonuç çıkıyor. 

![](https://ilkayblog.blob.core.windows.net/uploads/2020/05/20/largeArray.png)

Gördüğümüz üzere array boyutunun küçük olduğu durumlarda yeni array yaratmak daha hızlı olurken array uzunluğu arttıkça array pooldan array almak çok daha hızlı oluyor. Bu nedenle bu tip optimazasyonlara girmeden önce ilgili testleri yapmak çok kritik.

Bir sonraki yazıda görüşmek üzere.