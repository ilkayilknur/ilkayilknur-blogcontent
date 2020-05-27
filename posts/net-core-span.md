Bu yazıda konumuz .NET Core 2.1 ile beraber gelen ve özellikle .NET Core 3.0 ve sonrasında framework tarafından da oldukça fazla kullanılan `Span<T>` tipi. Span<T> tipi en basit tanımla bellekte ardışık olarak bulunan bir bölgeye type ve memory safe olarak erişmemizi sağlayan bir value type(struct). Span ile array, string gibi tiplerin bulunduğu managed heapteki bir bölgeye erişebilirken aynı zamanda stackalloc ile yaratılan stackteki bir bölgeye de veya native memoryde bir bölgeye de erişebiliyoruz. Üstelik bu işlemlerin hepsini Span sayesine ortak API üzerinden ve yüksek performanslı bir şekilde gerçekleştirebiliyoruz.

Span'in getirdiği en büyük faydalardan biri yukarıda bahsettiğim gibi memory bölgesinin nasıl yaratıldığından bağımsız olarak bize ortak bir API sunması. Diyelim ki bir array üzerinden işlem yapan metodunuz var ama bu array ister managed heapte saklanıyor olsun ister stackalloc ile stack üzerinde yaratılmış olsun. Biz bunun için 2 farklı metot yazmak yerine span kullanarak tüm implementasyonu bu ortak tip üzerinden gerçekleştirebiliyoruz. Bunu yaparken de hem yüksek performanslı bir şekilde yapıyoruz hem de memoryde yeni bir allocationa sebep olmayarak garbage collector için arkamızda herhangi bir şey bırakmıyoruz.

Yukarıdaki iki paragrafı okuduktan sonra konu size biraz karmaşık gelmiş olabilir. Aslında biraz daha basitleştirmemiz gerekirse Span memory üzerinde belirli bir alanı gören bir pencere olarak da düşünülebilir. Bu pencere tüm arrayi görebildiği gibi, arrayin sadece bir kısmını da görebilir. 

![](https://az718566.vo.msecnd.net/uploads/2020/05/27/span-diagram.jpg)

Eğer fotoğrafçılıkla ilgileniyorsanız span'i zoom lenslere de benzetebilirsiniz. İsterseniz lensle daha geniş açı çalışıp daha geniş bir alanı çekebilirken dilerseniz lensi ayarlayarak daha dar açı çalışıp sahneye daha da yakınlaşabilirsiniz. Olay aslında bundan ibaret. Şimdi isterseniz kod kısmına geçelim. 

Bir arrayden Span yaratmak için şunu yapmamız yeterli.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;array&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:blue;">int</span>[]&nbsp;{&nbsp;1,&nbsp;2,&nbsp;3,&nbsp;4,&nbsp;5,&nbsp;6,&nbsp;7,&nbsp;8,&nbsp;9,&nbsp;10&nbsp;};
<span style="color:#2b91af;">Span</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;span&nbsp;=&nbsp;array;</pre>

veya eğer elimizdeki tip span kullanımını destekliyorsa `AsSpan()` extension metodunu kullanabiliriz.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:#2b91af;">Span</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;span&nbsp;=&nbsp;array.AsSpan();
</pre>

Span tipini bir kere yarattıktan sonra index üzerinden o memory alanındaki ilgili değere ulaşmamız veya oradaki değeri değiştirmemiz mümkün. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;array&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:blue;">int</span>[]&nbsp;{&nbsp;1,&nbsp;2,&nbsp;3,&nbsp;4,&nbsp;5,&nbsp;6,&nbsp;7,&nbsp;8,&nbsp;9,&nbsp;10&nbsp;};
<span style="color:#2b91af;">Span</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;span&nbsp;=&nbsp;array.AsSpan();
<span style="color:blue;">int</span>&nbsp;sum&nbsp;=&nbsp;0;
<span style="color:blue;">for</span>&nbsp;(<span style="color:blue;">int</span>&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;&lt;&nbsp;array.Length;&nbsp;i++)
{
&nbsp;&nbsp;&nbsp;&nbsp;sum&nbsp;+=&nbsp;span[i];
}
<span style="color:#2b91af;">Console</span>.WriteLine(<span style="color:#a31515;">$&quot;Sum:</span>{sum}<span style="color:#a31515;">&quot;</span>);
<span style="color:green;">//Sum:55</span></pre>

Yukarıda spanleri bir pencere olarak düşünebileceğimizden bahsetmiştik. Şimdiye kadar yarattığımız spanler tüm arrayi kapsayan spanlerdi. Elimizdeki arrayin sadece belirli kısmını kapsayan span yaratmak için 2 farklı yol bulunmakta. Bunlardan biri eğer elinizde memoryde saklanan tipin referansı var ise aşağıdaki gibi bir kullanımda bulunmak mümkün.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;array&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:blue;">int</span>[]&nbsp;{&nbsp;1,&nbsp;2,&nbsp;3,&nbsp;4,&nbsp;5,&nbsp;6,&nbsp;7,&nbsp;8,&nbsp;9,&nbsp;10&nbsp;};
<span style="color:blue;">var</span>&nbsp;span&nbsp;=&nbsp;array.AsSpan(start:&nbsp;4,&nbsp;length:&nbsp;5);
<span style="color:blue;">var</span>&nbsp;span2&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Span</span>&lt;<span style="color:blue;">int</span>&gt;(array:&nbsp;array,&nbsp;start:&nbsp;4,&nbsp;length:&nbsp;5);</pre>

Eğer elimizde ilgili tipin referansı değil de o tip üzerinden yaratılan bir span varsa bu span üzerindeki Slice metodunu kullanarak daha ufak bir alanı gösteren yeni bir span yaratabiliriz.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;array&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:blue;">int</span>[]&nbsp;{&nbsp;1,&nbsp;2,&nbsp;3,&nbsp;4,&nbsp;5,&nbsp;6,&nbsp;7,&nbsp;8,&nbsp;9,&nbsp;10&nbsp;};
<span style="color:#2b91af;">Span</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;span&nbsp;=&nbsp;array;
<span style="color:blue;">var</span>&nbsp;span2&nbsp;=&nbsp;span.Slice(start:&nbsp;4,&nbsp;length:&nbsp;5);</pre>

### `ReadOnlySpan<T>` Tipi

`ReadOnlySpan<T>` tipi `Span<T>` karakteristiklerinin hepsini sağlarken sadece altında bulunan memorydeki değerin değiştirilememsini sağlıyor. Örneğin span yarattığımızda array içerisindeki ilgili indexteki değeri değiştirebilirken ReadOnlySpan ile bunu yapmamız mümkün değil.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;array&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:blue;">int</span>[]&nbsp;{&nbsp;1,&nbsp;2,&nbsp;3,&nbsp;4,&nbsp;5,&nbsp;6,&nbsp;7,&nbsp;8,&nbsp;9,&nbsp;10&nbsp;};
<span style="color:#2b91af;">Span</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;span&nbsp;=&nbsp;array;
span[0]&nbsp;=&nbsp;300;&nbsp;<span style="color:green;">//Kod&nbsp;derlenir.&nbsp;</span>
 
<span style="color:#2b91af;">ReadOnlySpan</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;span2&nbsp;=&nbsp;array;
span2[0]&nbsp;=&nbsp;300;&nbsp;<span style="color:green;">//Derleme&nbsp;hatası</span></pre>

Diyelim ki bir metoda elinizdeki array'in sadece belirli bir indexten sonraki elemanlarını parametre olarak göndermek istiyorsunuz ve bu metodun da arrayin elemanlarını değiştirememesini sağlamak istiyorsunuz. Bunu kolay bir şekilde ReadOnlySpan kullanarak yapabilirsiniz.

### String İle `ReadonlySpan<T>` Kullanımı 

Şimdiye kadar span kullanımına basit olması açısında array üzerinden değindik. Span kullanımının en çok işimize yarayacağı kısımlardan biri de string operasyonları. Bir stringden AsSpan extension metoduyla ReadOnlySpan<char> yaratabiliyoruz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;str&nbsp;=&nbsp;<span style="color:#a31515;">&quot;This&nbsp;is&nbsp;a&nbsp;string&quot;</span>;
<span style="color:#2b91af;">ReadOnlySpan</span>&lt;<span style="color:blue;">char</span>&gt;&nbsp;a&nbsp;=&nbsp;str.AsSpan();</pre>

### Stackalloc ile Span Kullanımı

Yazının başında stackalloc ile stack üzerinde yaratılan arraylere de Span ile erişebileceğimizden bahsetmiştik. C# 7.2 ve .NET Core 2.1 öncesinde stackalloc kullandığımızda unsafe kod yazmamız gerekiyordu. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">unsafe</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">int</span>&nbsp;length&nbsp;=&nbsp;3;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">int</span>*&nbsp;numbers&nbsp;=&nbsp;<span style="color:blue;">stackalloc</span>&nbsp;<span style="color:blue;">int</span>[length];
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">for</span>&nbsp;(<span style="color:blue;">var</span>&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;&lt;&nbsp;length;&nbsp;i++)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;numbers[i]&nbsp;=&nbsp;i;
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

Şu an ise span ile unsafe kod yazmadan bu alana erişebilmemiz mümkün. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">int</span>&nbsp;length&nbsp;=&nbsp;3;
<span style="color:#2b91af;">Span</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;numbers&nbsp;=&nbsp;<span style="color:blue;">stackalloc</span>&nbsp;<span style="color:blue;">int</span>[length];
<span style="color:blue;">for</span>&nbsp;(<span style="color:blue;">var</span>&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;&lt;&nbsp;length;&nbsp;i++)
{
&nbsp;&nbsp;&nbsp;&nbsp;numbers[i]&nbsp;=&nbsp;i;
}</pre>

### Span Kısıtlamaları ve `Memory<T>, ReadOnlyMemory<T>`

Span yapısı gereği bir **ref struct** olduğu için haliyle bazı kısıtlamara sahip. ref struct olmasının tabi ki en büyük nedenlerinden biri heapte allocationa neden olmaması. Span aynı zamanda ref struct olması nedeniyle boxingi önlemek amacıyla object, dynamic veya interface variablelarına atanamıyor. Await ile de kullanılamıyor. Ayrıca bir referans tipi içerisinde bir field olamıyor. Örneğin bir class içerisinde span property koyamıyorsunuz. 

Eğer yukarıdaki kısıtlara takılıp span kullanamıyorsanız Span'in **ref struct olmayan** ve aynı zamanda managed heapte de saklanabilen versiyonu `Memory<T>` ve `ReadOnlyMemory<T>` tiplerini kullanabilirsiniz. 

<pre style="font-family:Consolas;font-size:13px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;array&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:blue;">int</span>[]&nbsp;{&nbsp;1,&nbsp;2,&nbsp;3,&nbsp;4,&nbsp;5,&nbsp;6,&nbsp;7,&nbsp;8,&nbsp;9,&nbsp;10&nbsp;};
<span style="color:#2b91af;">Memory</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;memory&nbsp;=&nbsp;array;
<span style="color:#2b91af;">Memory</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;memory2&nbsp;=&nbsp;array.AsMemory();
<span style="color:#2b91af;">Memory</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;memory3&nbsp;=&nbsp;memory2.Slice(start:&nbsp;2,&nbsp;length:&nbsp;2);</pre>

Bu yazıda Span ve memory tiplerinden ve kısaca kullanımlarından bahsetmeye çalıştık. Bir sonraki yazıda bu tiplerin özellikle String işlemlerinde nasıl kullanılabileceğini ve performans olarak ne gibi farklar yaratabileceğinden bahsedeceğiz. 

Bir sonraki yazıda görüşmek üzere