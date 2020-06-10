<a href="https://ilkayilknur.com/net-coreda-span-ve-memory-tipleri" target="_blank">Bir önceki yazıda</a> .NET Core'daki Span<T> ve Memory<T> tiplerinden bahsettik. Bu yazıda da bu tiplerin performans ve memory kazanımlarını ufak örneklerle inceleyeceğiz. Aslında bu yazı bir anlamda bir önceki yazıyı tamamlayacak nitelikte olacak ve eski kullanımlar yerine neden yeni gelen bu tipleri kullanmamız gerektiğini bizlere gösterecek diye düşünüyorum. 

İlk senaryomuz bir array'in sadece belirli elemanlarıyla çalışma senaryosu. Diyelim ki elinizde bir array var ancak bu arrayin sadece belirli indexten sonraki elemanlarıyla bir işlem yapmak istiyorsunuz. Bunun için bir önceki makalede bahsettiğimiz gibi Span ve Memory tiplerini kullanabiliriz. Bunun dışında istediğimiz elemandan sonraki tüm değerleri yeni bir array'e kopyalayıp o array üzerinden işlemlerimizi gerçekleştirebiliriz. Şimdi gelin bu opsiyonları koda dökelim. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;">[<span style="color:#2b91af;">MemoryDiagnoser</span>]
[<span style="color:#2b91af;">HtmlExporter</span>]
<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Runner</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">int</span>[]&nbsp;array;
 
&nbsp;&nbsp;&nbsp;&nbsp;[<span style="color:#2b91af;">GlobalSetup</span>]
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Setup()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;array&nbsp;=&nbsp;<span style="color:#2b91af;">Enumerable</span>.Range(1,&nbsp;100).ToArray();
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;[<span style="color:#2b91af;">Benchmark</span>]
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;ArrayCopy()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;copy&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:blue;">int</span>[array.Length&nbsp;-&nbsp;10];
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Array</span>.Copy(array,&nbsp;10,&nbsp;copy,&nbsp;0,&nbsp;copy.Length);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;[<span style="color:#2b91af;">Benchmark</span>]
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Span()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;span&nbsp;=&nbsp;array.AsSpan(10);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;[<span style="color:#2b91af;">Benchmark</span>]
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Memory()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;memory&nbsp;=&nbsp;array.AsMemory(10);
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

Benchmarkın setup kısmında basit olarak 100 elemanlı bir array yaratıyoruz. Sonrasında ilk olarak `Array.Copy` metodunu kullanarak array içerisindeki 10. elemandan sonraki tüm elemanları yeni bir arraye kopyalıyoruz. Diğer benchmarklarda ise 10. elemandan sonraki tüm elemanları temsil eden bir Span ve Memory yaratıyoruz. Sonuçlar ise aşağıdaki gibi.

|    Method |       Mean |     Error |    StdDev |     Median |  Gen 0 | Gen 1 | Gen 2 | Allocated |
|---------- |-----------:|----------:|----------:|-----------:|-------:|------:|------:|----------:|
| ArrayCopy | 46.3849 ns | 0.9564 ns | 2.4342 ns | 45.2222 ns | 0.0612 |     - |     - |     384 B |
|      Span |  0.4787 ns | 0.0193 ns | 0.0161 ns |  0.4709 ns |      - |     - |     - |         - |
|    Memory |  0.4974 ns | 0.0229 ns | 0.0203 ns |  0.5002 ns |      - |     - |     - |         - |

Array copy senaryosunda hem yeni bir array yaratıp heap allocationa sebep oluyoruz hem de orjinal arrayden yeni yaratılan arraye elemanları kopyalıyoruz. Bu nedenle bu işlem diğer opsiyonlara göre oldukça yavaş. Ancak Span veya Memory tipleriyle gördüğünüz gibi hızlı bir şekilde arrayin ilgili bölümünü temsil eden yapıları hızlıca yaratabiliyoruz. 

Span ve memorynin en önemli kullanım alanlarından biri de stringler. Diyelim ki bir stringi parse ediyorsunuz. Bunun için normalde string içerisindeki herhangi bir alanı `String.Substring` metodunu kullanarak alabiliriz. Ancak stringler immutable tipler oldukları için bu metotların sonucunda yeni bir string yaratılır ve bize dönülür. Bu da yeni bir allocationa neden olur. Bundan kaçınmak için de Span ve Memory kullanabiliriz. Hemen kodlara bakalım. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;">[<span style="color:#2b91af;">MemoryDiagnoser</span>]
[<span style="color:#2b91af;">HtmlExporter</span>]
<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Runner</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">string</span>&nbsp;text;
 
&nbsp;&nbsp;&nbsp;&nbsp;[<span style="color:#2b91af;">GlobalSetup</span>]
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Setup()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;text&nbsp;=&nbsp;<span style="color:#a31515;">&quot;1234:123123123:12312311&quot;</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;[<span style="color:#2b91af;">Benchmark</span>]
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Substring()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;firstIndex&nbsp;=&nbsp;text.IndexOf(<span style="color:#a31515;">&#39;:&#39;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;lastIndex&nbsp;=&nbsp;text.LastIndexOf(<span style="color:#a31515;">&#39;:&#39;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;text.Substring(firstIndex&nbsp;+&nbsp;1,&nbsp;lastIndex&nbsp;-&nbsp;firstIndex&nbsp;-&nbsp;1);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;[<span style="color:#2b91af;">Benchmark</span>]
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">ReadOnlySpan</span>&lt;<span style="color:blue;">char</span>&gt;&nbsp;Span()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;firstIndex&nbsp;=&nbsp;text.IndexOf(<span style="color:#a31515;">&#39;:&#39;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;lastIndex&nbsp;=&nbsp;text.LastIndexOf(<span style="color:#a31515;">&#39;:&#39;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;text.AsSpan(firstIndex&nbsp;+&nbsp;1,&nbsp;lastIndex&nbsp;-&nbsp;firstIndex&nbsp;-&nbsp;1);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;[<span style="color:#2b91af;">Benchmark</span>]
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">ReadOnlyMemory</span>&lt;<span style="color:blue;">char</span>&gt;&nbsp;Memory()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;firstIndex&nbsp;=&nbsp;text.IndexOf(<span style="color:#a31515;">&#39;:&#39;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;lastIndex&nbsp;=&nbsp;text.LastIndexOf(<span style="color:#a31515;">&#39;:&#39;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;text.AsMemory(firstIndex&nbsp;+&nbsp;1,&nbsp;lastIndex&nbsp;-&nbsp;firstIndex&nbsp;-&nbsp;1);
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

Benchmark sonucu da şu şekilde.

|    Method |     Mean |    Error |   StdDev |  Gen 0 | Gen 1 | Gen 2 | Allocated |
|---------- |---------:|---------:|---------:|-------:|------:|------:|----------:|
| Substring | 20.92 ns | 0.106 ns | 0.099 ns | 0.0076 |     - |     - |      48 B |
|      Span | 11.07 ns | 0.036 ns | 0.032 ns |      - |     - |     - |         - |
|    Memory | 13.02 ns | 0.058 ns | 0.051 ns |      - |     - |     - |         - |


Gördüğünüz gibi bu benchmark sonunda da Span ve Memory'nin performansı Substring'e göre oldukça yüksek. Diğer bir yandan da allocationa neden olmuyor. 

Bu makalede bir önceki makaleyi tamamlaması açısında Span ve Memory tiplerinin performans ve memory kullanımı açısından nasıl performanslı olabileceğini gördük. Şu anda .NET Core içerisinde çoğu API'da zaten Span ve Memory desteği bulunmakta. Ayrıca internal olarak da .NET Core içerisinde çoğu API da bu tipleri kullanarak büyük bir performans ve memory kazanımları sağlamakta. .NET Core 3.0 ile beraber gelen yeni `System.Text.Json` API'larının da çok performanslı çalışmasının sebeplerinden biri de yine alt katmanda bu tipleri oldukça etkin kullanması. Biz bu makalede gördüğünüz gibi sadece çok basit örnekler üzerinde performans karşılaştırması yaptık. Ancak fazla yük alan uygulamalarınızda özellikle bu tipleri doğru bir şekilde kullanmanız durumunda oldukça fazla kazanım elde etmeniz mümkün.

Bir sonraki makalede görüşmek üzere

