# C# 7.0 - Tuples #

C# 7.0 ile beraber gelecek olan önemli özelliklerden biri de Tuple'lar. Tuple tiplerine aslında çok da yabancı değiliz. Tuple tipiyle ilk olarak .NET Framework 4.0 ile tanışmıştık. Hani şu içerisinde **Item1, Item2, Item3** diye propertyler olan tipler :) Çoğu zaman aslında kullanmak istediğimiz ama bu property isimlerinden dolayı kullanma konusunda içimizin rahat olmadığı tipler :)

Tupleların en önemli kullanım alanları aslında bir metottan birden fazla değer döndürmek zorunda kaldığımız durumlar. C# içerisinde aslında bir metottan birden fazla değer döndürmek istediğimizde out parametrelerini kullanabiliyoruz. 

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">private</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Foo(<span style="color:blue;">int</span>&nbsp;param1,&nbsp;<span style="color:blue;">out</span>&nbsp;<span style="color:blue;">int</span>&nbsp;intReturnParam,&nbsp;<span style="color:blue;">out</span>&nbsp;<span style="color:blue;">string</span>&nbsp;stringReturnParam)<br/>{<br/> <br/>}</pre>

Ancak out parametreler maalesef async metotlarda kullanılamıyor. Dolayısıyla bu durumda Tuple tipini kullanmak durumunda kalıyoruz. 

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">private</span>&nbsp;<span style="color:blue;">async</span>&nbsp;<span style="color:#2b91af;">Task</span>&lt;<span style="color:#2b91af;">Tuple</span>&lt;<span style="color:blue;">int</span>,&nbsp;<span style="color:blue;">string</span>&gt;&gt;&nbsp;FooAsync(<span style="color:blue;">int</span>&nbsp;param1)<br/>{<br/> <br/>}<br/> <br/><span style="color:blue;">private</span>&nbsp;<span style="color:blue;">async</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Barrier()<br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;result&nbsp;=&nbsp;<span style="color:blue;">await</span>&nbsp;FooAsync(1);<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Debug</span>.WriteLine(<span style="color:#a31515;">$&quot;</span>{result.Item1}<span style="color:#a31515;">&nbsp;:&nbsp;</span>{result.Item2}<span style="color:#a31515;">&quot;</span>);<br/>}</pre>

Bu durumda da metodu çağıran veya kullanan yazılımcılar aslında bu async metottan ne döndüğü konusunda hiçbir şekilde bilgi sahibi olamıyorlar. Item1, Item2 fieldlarını ne işe yaradığını, hangisinde hangi bilgi olduğunu iyi bir şekilde dökümante etmeniz gerekiyor. Ayrıca mevcut Tuple tipi bir class olduğu için heap allocationa neden oluyor. 

Tüm bu nedenlerden dolayı aslında gerçek anlamda Tuple desteği C# 7.0 ile beraber geliyor. Peki C# 7.0'da Tuple'lar nasıl olacak. Gelelim bu kısma. 

Metotlarda birden fazla değer döndürmek istediğimizde döndüreceğimiz alanların tiplerini ve isimlerini parantez içerisinde yazmamız gerekecek. 

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">static</span>&nbsp;(<span style="color:blue;">int</span>&nbsp;count,&nbsp;<span style="color:blue;">string</span>&nbsp;value)&nbsp;Foo()<br/>{<br/> <br/>}<br/> <br/><span style="color:blue;">static</span>&nbsp;<span style="color:#2b91af;">Task</span>&lt;(<span style="color:blue;">int</span>&nbsp;count,&nbsp;<span style="color:blue;">string</span>&nbsp;value)&gt;&nbsp;FooAsync()<br/>{<br/> <br/>}</pre> 

Yukarıda gördüğünüz gibi Foo metodunun dönüş değeri bir tuple ve bu tuple'ın içerisinde count ve value diye 2 tane alan bulunuyor. Ayrıca Task döndüren asenkron metotların da dönüş tipleri gördüğünüz gibi tuple olabiliyor. Bu şekilde tuple dönüş tiplerini tanımladıktan sonra peki metot içerisinde bir tuple nasıl tanımlıyoruz kısmına bakalım. Tanımlama için birden fazla kullanım senaryosu var aslında. İlk olarak normal bir object yaratırmış gibi tuple yaratma. 

 <pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">static</span>&nbsp;(<span style="color:blue;">int</span>&nbsp;count,&nbsp;<span style="color:blue;">string</span>&nbsp;value)&nbsp;Foo()<br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;retVal&nbsp;=&nbsp;<span style="color:blue;">new</span>(<span style="color:blue;">int</span>&nbsp;count,&nbsp;<span style="color:blue;">string</span>&nbsp;value)<br/>&nbsp;&nbsp;&nbsp;&nbsp;{<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;count&nbsp;=&nbsp;1,<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;value&nbsp;=&nbsp;<span style="color:#a31515;">&quot;Bar&quot;</span><br/>&nbsp;&nbsp;&nbsp;&nbsp;};<br/> <br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;retVal;<br/>}</pre>

Gördüğünüz gibi aslında oldukça basit. Anonymous object tanımlar gibi tanımlıyoruz, sadece ek olarak tuple içerisinde bulunacak olan alanların isimlerini ve tiplerini parantez içerisinde yazıyoruz.  Ancak bu kullanım geçerli olsa da kullanım olarak çok da kolay değil. Bunun yerine C# 7.0 ile beraber daha özel bir tanımlama syntax'ı geliyor.

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">static</span>(<span style="color:blue;">int</span>&nbsp;count,&nbsp;<span style="color:blue;">string</span>&nbsp;value)&nbsp;Foo()<br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;retVal&nbsp;=&nbsp;(count:&nbsp;1,&nbsp;value:&nbsp;<span style="color:#a31515;">&quot;Foo&quot;</span>);<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;retVal;<br/>}</pre>

Parantez içerisinde sadece alanın adını ve değerini yazdığınızda da compiler arka planda aynı anonymous objectlerde olduğu gibi alanın tipini kendisi buluyor ve ona uygun tuple tipini yaratıyor. Ayrıca yine tipin compiler tarafından bilindiği durumlarda da tuple'lardaki field adlarının pek de önemi olmuyor. Compiler arka planda ilgili çevrimi kendisi yapıyor.

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">static</span>&nbsp;(<span style="color:blue;">int</span>&nbsp;count,&nbsp;<span style="color:blue;">string</span>&nbsp;value)&nbsp;Foo()<br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;retVal&nbsp;=&nbsp;(c:&nbsp;1,&nbsp;v:&nbsp;<span style="color:#a31515;">&quot;Foo&quot;</span>);<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;retVal;<br/>}</pre>


## Tuple Deconstruction ##

Bir metottan veya herhangi bir yerden bir tuple döndüğünde o tuple içerisindeki değerleri ayrıştırmak ve metodun devamında ayrıştırılmış halini kullanmak önemli. Bu yüzden tuple içerisindeki alanları ayrıştırıp içerisindeki değerleri değişkenlere atamak için de kolay bir syntax geliyor C# 7.0 ile. 

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Main(<span style="color:blue;">string</span>[]&nbsp;args)<br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;(var&nbsp;count,&nbsp;var&nbsp;value)&nbsp;=&nbsp;Foo();<br/>}<br/> <br/><span style="color:blue;">static</span>&nbsp;(<span style="color:blue;">int</span>&nbsp;count,&nbsp;<span style="color:blue;">string</span>&nbsp;value)&nbsp;Foo()<br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;retVal&nbsp;=&nbsp;(c:&nbsp;1,&nbsp;v:&nbsp;<span style="color:#a31515;">&quot;Foo&quot;</span>);<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;retVal;<br/>}</pre>

Main metodunda göründüğü gibi Foo metodundan dönen tuple tipi içerisindeki alanları count ve value ismindeki local değişkenlere atamasını yapabiliyoruz. Böylece aslında tuple tipleri tamamen görünmez bir şekilde kalabiliyorlar. Yani siz bir metottan tuple döndüğünü biliyorsanız, bu tuple içerisindeki alanları hemen hızlıca lokal değişkenlere alıp kodunuzu temiz tutabilirsiniz. 

C# 7.0 ile beraber gelen tuple'ların Framework içerisinde bulanan Tuple'lardan bazı farkları var. Bunlardan ilki C# 7.0 tuple'larının **struct** olması. Böylece bu tupleların yaratılmaları daha az maliyetli. Ayrıca C# 7.0 tuple'ları **mutable**. Yani bir tuple yarattıktan sonra fieldın değerini değiştirebilirsiniz. Ancak .NET Framework içerisindeki Tuple tipleri immutable. Yani yarattıktan sonra fieldın değerini sadece okuyabilirsiniz, değiştiremezsiniz.

Son olarak Tuple tiplerini denemek isterseniz test için **Visual Studio Preview 15** kullanmanız gerekiyor. Eğer test yaparken tuple kullanımından sonra derlerken aşağıdaki hatayı alırsanız nuget üzerinden "**System.ValueTuple**" prerelease paketini yüklemeniz gerekiyor. Bu şimdilik bir gereklilik ancak ürün release olduğunda buradaki tipler BCL'e eklenecektir. 

> Cannot define a class or member that utilizes tuples because the compiler required type 'System.Runtime.CompilerServices.TupleElementNamesAttribute' cannot be found. Are you missing a reference ?

Bir sonraki yazıda görüşmek üzere...

*Not : Yukarıdaki örnekler Visual Studio 15 Preview 4 ile yazılmış ve test edilmiştir.*
