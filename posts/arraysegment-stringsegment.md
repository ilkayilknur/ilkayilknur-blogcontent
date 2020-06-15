Önceki yazılarımızda Span<T> ve Memory<T> tiplerinden bahsetmiştik. Bu yazıda da bu tiplerden biraz daha eski olan ve benzer amaçlarla kullanabileceğimiz ArraySegment<T> ve StringSegment'ten bahsedeceğiz. 

ArraySegment basit olarak yine bir array'in belirli bir bölümünü temsil eden bir struct. Bu yapıyı Span'den farklı olarak bir wrapper olarak düşünebiliriz. ArraySegment aynı zamanda ICollection<T>, IEnumerable<T>, IList<T>, IReadOnlyCollection<T>, IReadOnlyList<T> interfacelerini de implemente ediyor. 

Varolan bir array üzerinde bir ArraySegment'i şu şekilde yaratabiliriz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;array&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:blue;">int</span>[]&nbsp;{&nbsp;1,&nbsp;2,&nbsp;3,&nbsp;4,&nbsp;5,&nbsp;6,&nbsp;7,&nbsp;8,&nbsp;9,&nbsp;10&nbsp;};
<span style="color:#2b91af;">ArraySegment</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;segment&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">ArraySegment</span>&lt;<span style="color:blue;">int</span>&gt;(array,&nbsp;2,&nbsp;3);
<span style="color:green;">//segment&nbsp;:&nbsp;3,4,5</span></pre>

ArraySegment yaratırken kullandığımız parametreler orjinal arrayin kendisi, segmentin hangi offsetten başlayacağı ve ne kadar uzunlukta olacağı. ArraySegmenti yarattıktan sonra o tip üzerinden orjinal arraye, offset ve uzunluk değerlerine de ulaşmak mümkün.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;array&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:blue;">int</span>[]&nbsp;{&nbsp;1,&nbsp;2,&nbsp;3,&nbsp;4,&nbsp;5,&nbsp;6,&nbsp;7,&nbsp;8,&nbsp;9,&nbsp;10&nbsp;};
<span style="color:#2b91af;">ArraySegment</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;segment&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">ArraySegment</span>&lt;<span style="color:blue;">int</span>&gt;(array,&nbsp;2,&nbsp;3);
<span style="color:green;">//segment&nbsp;:&nbsp;3,4,5</span>
 
<span style="color:#2b91af;">Console</span>.WriteLine(segment.Offset);
<span style="color:green;">//2</span>
<span style="color:#2b91af;">Console</span>.WriteLine(segment.Count);
<span style="color:green;">//3</span>
<span style="color:#2b91af;">Console</span>.WriteLine(segment.Array.Length);
<span style="color:green;">//10</span></pre>

ArraySegment içerisinde bulunan Array propertysi her zaman orjinal array'i gösterir. Yani ArraySegment kullanarak array üzerinde yaptığınız işlemler doğrudan orjinal array üzerinde yapılır. Bunu gözönünde bulundurmamızda fayda var. Eğer böyle olmasını istemiyorsanız arrayin bir kopyasını alıp ilerlemeniz gerekir. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;array&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:blue;">int</span>[]&nbsp;{&nbsp;1,&nbsp;2,&nbsp;3,&nbsp;4,&nbsp;5,&nbsp;6,&nbsp;7,&nbsp;8,&nbsp;9,&nbsp;10&nbsp;};
<span style="color:#2b91af;">ArraySegment</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;segment&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">ArraySegment</span>&lt;<span style="color:blue;">int</span>&gt;(array,&nbsp;2,&nbsp;3);
<span style="color:green;">//segment&nbsp;:&nbsp;3,4,5</span>
segment[0]&nbsp;=&nbsp;10;
 
<span style="color:#2b91af;">Console</span>.Write(array[2]);
<span style="color:green;">//10</span></pre>

ArraySegment'in en önemli özelliklerinden biri de LINQ querylerini çalıştırabiliyor olmamız. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;array&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:blue;">int</span>[]&nbsp;{&nbsp;1,&nbsp;2,&nbsp;3,&nbsp;4,&nbsp;5,&nbsp;6,&nbsp;7,&nbsp;8,&nbsp;9,&nbsp;10&nbsp;};
<span style="color:#2b91af;">ArraySegment</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;segment&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">ArraySegment</span>&lt;<span style="color:blue;">int</span>&gt;(array,&nbsp;2,&nbsp;3);
<span style="color:green;">////segment&nbsp;:&nbsp;3,4,5</span>
<span style="color:#2b91af;">Console</span>.WriteLine(segment.Max());
<span style="color:green;">//5</span></pre>

ArraySegment'de de Span ve Memory tipleri gibi slicingi supportu bulunmakta. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;array&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:blue;">int</span>[]&nbsp;{&nbsp;1,&nbsp;2,&nbsp;3,&nbsp;4,&nbsp;5,&nbsp;6,&nbsp;7,&nbsp;8,&nbsp;9,&nbsp;10&nbsp;};
<span style="color:#2b91af;">ArraySegment</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;segment&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">ArraySegment</span>&lt;<span style="color:blue;">int</span>&gt;(array,&nbsp;2,&nbsp;3);
<span style="color:green;">////segment&nbsp;:&nbsp;3,4,5</span>
<span style="color:blue;">var</span>&nbsp;newSegment&nbsp;=&nbsp;segment.Slice(1,&nbsp;1);
<span style="color:green;">//newSegment:&nbsp;4</span></pre>

İki `ArraySegment<T>` instance'ının aynı olabilmesi için aynı arrayi göstermeleri, aynı offsetten başlamaları ve eşiş sayıda eleman içermeleri gerekiyor. 

#### StringSegment

StringSegment tipi de aynı ArraySegment'te olduğu stringin farklı bölgelerini belirten bir struct. Bu structı kullanarak string.Substring metodunu kullanmadan string üzerindeki belirli bir bölge üzerinde işlem yapabiliriz. Böylece yeni bir string yaratarak allocationa neden olmadan işlemleri gerçekleştirebiliriz.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">string</span>&nbsp;sample&nbsp;=&nbsp;<span style="color:#a31515;">&quot;test:0:stringsegment&quot;</span>;
<span style="color:#2b91af;">StringSegment</span>&nbsp;first&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">StringSegment</span>(sample,&nbsp;0,&nbsp;sample.IndexOf(<span style="color:#a31515;">&#39;:&#39;</span>));
<span style="color:green;">//first:&nbsp;test</span>
<span style="color:#2b91af;">StringSegment</span>&nbsp;last&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">StringSegment</span>(sample,&nbsp;sample.LastIndexOf(<span style="color:#a31515;">&#39;:&#39;</span>)&nbsp;+&nbsp;1,&nbsp;sample.Length&nbsp;-&nbsp;sample.LastIndexOf(<span style="color:#a31515;">&#39;:&#39;</span>)&nbsp;-&nbsp;1);
<span style="color:green;">//last:stringsegment</span></pre>

Bu tip de yine ArrayString gibi slicingi desteklemekte. Bunun için StringSegment.SubSegment metodunu kullanabiliriz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">string</span>&nbsp;sample&nbsp;=&nbsp;<span style="color:#a31515;">&quot;test:0:stringsegment&quot;</span>;
<span style="color:#2b91af;">StringSegment</span>&nbsp;last&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">StringSegment</span>(sample,&nbsp;sample.LastIndexOf(<span style="color:#a31515;">&#39;:&#39;</span>)&nbsp;+&nbsp;1,&nbsp;sample.Length&nbsp;-&nbsp;sample.LastIndexOf(<span style="color:#a31515;">&#39;:&#39;</span>)&nbsp;-&nbsp;1);
<span style="color:green;">//last:stringsegment&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span>
<span style="color:#2b91af;">StringSegment</span>&nbsp;segment&nbsp;=&nbsp;last.Subsegment(6);
<span style="color:green;">//segment:segment</span></pre>

Evet gördüğümüz üzere ArraySegment ve StringSegment, array ve string üzerindeki belirli alanlarda çalışabilmek için kullanabileceğimiz tiplerden bir tanesi. Bu tipleri kullanarak örneğin arrayin belirli bölgelerinde multithread olarak çalışma yapmamız mümkün. Aynı zamanda async metotlarda da kullanabiliyoruz. Span ve memory'den farkları nedir diye baktığımızda ise hatırlarsanız span ve memory birden fazla tipteki memory bölgelerinde çalışabiliyordu. String ve ArraySegment ise sadece array ve string üzerinde çalışabiliyor. Aynı zamanda bu tipler size orjinal kaynağa erişim ve o kaynağı değiştirme izni de veriyor. Yani siz ArraySegment üzerinden Array propertysine ulaşıp istediğiniz herhangi bir elemanı değiştirebilirsiniz. ReadonlySpan veya ReadonlyMemory kullanarak bunun önüne geçebilirsiniz.

Eğer sadece array veya string ile çalışacaksanız ve bu tipler üzerinde parsing veya bu tiplerin farklı bölümlerini multithread olarak işleme gibi işlemler yapacaksanız bu tipleri kullanabilirsiniz. Örneğin ASP.NET Core içerisinde headerların parsing işleminde(<a href="https://github.com/dotnet/aspnetcore/blob/master/src/Http/Headers/src/MediaTypeHeaderValue.cs" target="_blank">https://github.com/dotnet/aspnetcore/blob/master/src/Http/Headers/src/MediaTypeHeaderValue.cs</a>) doğrudan StringSegment'in kullanıldığını görebilirsiniz. Böylece parsing esnasında hiçbir şekilde gereksiz string üretimi gerçekleştirilmiyor ve etkin bir implementasyon yapılıyor. 

Bir sonraki yazıda görüşmek üzere