 Kod yazarken dikkat etmemiz gereken en önemli noktalardan biri gereksiz memory kullanımından kaçınmak. Yanlış memory kullanımı dediğimizde aklımıza ilk gelen tiplerden biri de `string` tipi. String yapısı gereği immuatable bir tip  olduğu için string üzerinde değişiklik yapmak istediğimizde, farklı stringleri  birleştirmek  istediğimizde vs..  yeni  bir string yaratmamız gerekiyor. Daha öncesinde stringleri belirli bölgesinden kesmek istediğimizde de `Substring` metodunu kullanarak yeni bir stringin yaratılmasına sebep olabiliyorduk. Ancak yeni gelen <a href="https://ilkayilknur.com/arraysegment-ve-stringsegment-nedir-nasil-kullanilir" target="_blank">StringSegment</a> veya <a href="https://ilkayilknur.com/net-coreda-span-ve-memory-tipleri">Span</a> tipleriyle yeni string yaratılmasından da kaçınmamız şu an mümkün. 

 Runtime esnasında elimizdeki belirli stringleri birleştirip yeni bir string yaratmak istediğimizde ve bu işlemi optimize bir şekilde yapmak istediğimizde kullanmamız gereken tip `StringBuilder`. Ancak StringBuilder string birleştirme operasyonlarını optimize bir şekilde yapsa da arka planda birleştirilecek olan stringleri bir bufferda sakladığını için ekstra heap allocationa neden olmakta. Bu buffer eklediğimiz stringlerin boyutuna göre de yerine göre resize edilmekte ve değerler yeni buffera kopyalanmakta. Bu da tabi ki performans kritik senaryolarda bir yük getirmekte. Bunu optimize etmek için eğer oluşturulacak olan stringin final uzunluğu biliniyorsa StringBuilder yaratılırken bir capacity belirtmek bizi en azından bufferın arka planda resize edilmesinden kurtaracaktır.
 
 Bunun yanında özellikle sık çalışan kodlarda sürekli olarak StringBuilder nesnesi yaratmak arka planda aynı zamanda yeni buffer yaratılmasına neden olacaktır. Bu nedenle sürekli olarak buffer yaratılıp sonra GC tarafından temizlenmesinin önüne geçmek için StringBuilderlar bir object pool içerisinde saklanıp gerektiği zaman pooldan alınıp iş bittiği zaman poola geri bırakılabilir(<a href="https://ilkayilknur.com/object-pooling-nedir-net-core-icerisinde-nasil-kullanilir" target="_blank">Object Pooling</a>).
 
.NET Core 2.1 ile beraber string tipinin içerisine eklenen `Create` metoduyla arka planda buffer allocationa neden olmadan da etkin bir şekilde string yaratmak mümkün hale geldi. Bu metodu kullanırken de göreceğiniz üzere string parametre olarak geçtiğimiz delegate içerisinde **mutable** durumda. `Create` metodunu kullanabilmemiz için ilk şart oluşacak olan stringin uzunluğunu önceden biliyor olmak. Eğer önceden bilmiyorsak veya hesaplayamıyorsak bu metodu kullanmamız mümkün değil. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Create&lt;<span style="color:#2b91af;">TState</span>&gt;(<span style="color:blue;">int</span>&nbsp;length,&nbsp;<span style="color:#2b91af;">TState</span>&nbsp;state,&nbsp;System.Buffers.<span style="color:#2b91af;">SpanAction</span>&lt;<span style="color:blue;">char</span>,&nbsp;<span style="color:#2b91af;">TState</span>&gt;&nbsp;action);
</pre>

Create metodunun parametrelerine bakarsak,
* length: Oluşturulacak stringin uzunluğu.
* state: stringi yaratırken kullanacağımız ve delegate'e parametre olarak gelecek olan object.(Eğer birden fazla değişkeni parametre geçmemiz gerekirse tuple kullanabiliriz.)
* action: string yaratılırken çağrılacak olan delegate.

Şimdi gelin önce basit bir kullanım örneğiyle başlayalım.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:#2b91af;">List</span>&lt;<span style="color:blue;">string</span>&gt;&nbsp;list&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:blue;">string</span>&gt;()
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#a31515;">&quot;test&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#a31515;">&quot;test2&quot;</span>
};
 
<span style="color:blue;">var</span>&nbsp;str3&nbsp;=&nbsp;<span style="color:blue;">string</span>.Create(9,&nbsp;list,&nbsp;(c,&nbsp;state)&nbsp;=&gt;
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">int</span>&nbsp;index&nbsp;=&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;state[0].AsSpan().CopyTo(c);
&nbsp;&nbsp;&nbsp;&nbsp;index&nbsp;+=&nbsp;state[0].Length;
&nbsp;&nbsp;&nbsp;&nbsp;state[1].AsSpan().CopyTo(c.Slice(index));
});</pre>

Şimdi diyelim ki elimizde bir liste var ve bu listedeki her bir elemanı birleştirip string yaratmak istiyoruz. İlk kullanım örneği olduğu için bazı değerleri statik yaparak ilerliyoruz. Liste iki elemanlı olduğu için ve kod yazarken içerisindeki değerleri görebildiğimiz için oluşacak olan stringin final uzunluğunu tahmin etmemiz mümkün. Bu nedenle ilk parametreyi 9 olarak veriyoruz. İkinci parametre ise stringi yaratırken kullanacağımız state nesnesi. Bunun için yukarıda tanımlanan `list`'i parametre olarak geçiyoruz.

Son parametre olarak da stringi initialize eden delegate'i veriyoruz. Bu delegate'in ilk parametresi stringin arkasında saklanan char arrayi temsil eden bir `Span<char>`. Bu parametreyi kullanarak stringi initialize edebileceğiz. İkinci parametre ise state parametresinin kendisi. Biz bu değişkeni zaten ikinci parametre olarak geçmiştik diyebilirsiniz. Ancak biz delegate içerisinde list değişkenini kullanırsak bu yeni bir allocationa neden olacağı için efektif olmayacaktır. Bu nedenle stringi yaratırken mutlaka state parametresini kullanarak ilerlememizde fayda var.

Delegate'in içeriğine baktığımızda ise ilk olarak listenin ilk elemanını kopyalıyoruz sonrasında ise ikinci elemanı kopyalıyoruz. Böylece baktığımızda stringleri tutmak için arkada bir buffer allocate etmeye gerek kalmıyor. 

Daha generic çalışan bir implementasyon yaparsak.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:#2b91af;">List</span>&lt;<span style="color:blue;">string</span>&gt;&nbsp;list&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:blue;">string</span>&gt;()
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#a31515;">&quot;test&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#a31515;">&quot;test2&quot;</span>
};
<span style="color:blue;">var</span>&nbsp;length&nbsp;=&nbsp;0;
<span style="color:blue;">for</span>&nbsp;(<span style="color:blue;">int</span>&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;&lt;&nbsp;list.Count;&nbsp;i++)
{
&nbsp;&nbsp;&nbsp;&nbsp;length&nbsp;+=&nbsp;list[i].Length;
}
 
<span style="color:blue;">var</span>&nbsp;str3&nbsp;=&nbsp;<span style="color:blue;">string</span>.Create(length,&nbsp;list,&nbsp;(c,&nbsp;state)&nbsp;=&gt;
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">int</span>&nbsp;index&nbsp;=&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">for</span>&nbsp;(<span style="color:blue;">int</span>&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;&lt;&nbsp;state.Count;&nbsp;i++)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;state[i].AsSpan().CopyTo(c.Slice(index));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;index&nbsp;+=&nbsp;state[i].Length;
&nbsp;&nbsp;&nbsp;&nbsp;}
});</pre>

Şimdi de son olarak ufak bir benchmarking yapalım ve aradaki farkı inceleyelim. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;">[<span style="color:#2b91af;">MemoryDiagnoser</span>]
[<span style="color:#2b91af;">Orderer</span>(BenchmarkDotNet.Order.<span style="color:#2b91af;">SummaryOrderPolicy</span>.FastestToSlowest)]
[<span style="color:#2b91af;">MarkdownExporter</span>]
<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Benchmark</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:blue;">string</span>&gt;&nbsp;list;
 
&nbsp;&nbsp;&nbsp;&nbsp;[<span style="color:#2b91af;">Params</span>(10,&nbsp;100,&nbsp;400)]
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">int</span>&nbsp;Size&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">set</span>;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;[<span style="color:#2b91af;">IterationSetup</span>]
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Setup()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;list&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:blue;">string</span>&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">for</span>&nbsp;(<span style="color:blue;">int</span>&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;&lt;&nbsp;Size;&nbsp;i++)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;list.Add(<span style="color:#a31515;">&quot;Test&quot;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;[<span style="color:#2b91af;">Benchmark</span>]
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;StringCreate()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;length&nbsp;=&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">for</span>&nbsp;(<span style="color:blue;">int</span>&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;&lt;&nbsp;list.Count;&nbsp;i++)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;length&nbsp;+=&nbsp;list[i].Length;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:blue;">string</span>.Create(length,&nbsp;list,&nbsp;(c,&nbsp;state)&nbsp;=&gt;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">int</span>&nbsp;index&nbsp;=&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">for</span>&nbsp;(<span style="color:blue;">int</span>&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;&lt;&nbsp;state.Count;&nbsp;i++)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;state[i].AsSpan().CopyTo(c.Slice(index));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;index&nbsp;+=&nbsp;state[i].Length;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;});
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;[<span style="color:#2b91af;">Benchmark</span>]
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;StringBuilderDefault()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;builder&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">StringBuilder</span>();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">for</span>&nbsp;(<span style="color:blue;">int</span>&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;&lt;&nbsp;list.Count;&nbsp;i++)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;builder.Append(list[i]);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;builder.ToString();
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;[<span style="color:#2b91af;">Benchmark</span>]
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;StringBuilderInitialCapacity()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;capacity&nbsp;=&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">for</span>&nbsp;(<span style="color:blue;">int</span>&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;&lt;&nbsp;list.Count;&nbsp;i++)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;capacity&nbsp;+=&nbsp;list[i].Length;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;builder&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">StringBuilder</span>(capacity);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">for</span>&nbsp;(<span style="color:blue;">int</span>&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;&lt;&nbsp;list.Count;&nbsp;i++)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;builder.Append(list[i]);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;builder.ToString();
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

Sonuçlara bakarsak...

<table class="table">
<thead><tr><th>                Method</th><th>Size</th><th>Mean</th><th>Error</th><th>StdDev</th><th>Median</th><th>Gen 0</th><th>Gen 1</th><th>Gen 2</th><th>Allocated</th>
</tr>
</thead><tbody><tr><td>StringCreate</td><td>10</td><td>1.028 &mu;s</td><td>0.0303 &mu;s</td><td>0.0839 &mu;s</td><td>1.000 &mu;s</td><td>-</td><td>-</td><td>-</td><td>104 B</td>
</tr><tr><td>StringBuilderDefault</td><td>10</td><td>2.333 &mu;s</td><td>0.3873 &mu;s</td><td>1.1420 &mu;s</td><td>1.600 &mu;s</td><td>-</td><td>-</td><td>-</td><td>448 B</td>
</tr><tr><td>StringBuilderInitialCapacity</td><td>10</td><td>1.273 &mu;s</td><td>0.0291 &mu;s</td><td>0.0574 &mu;s</td><td>1.300 &mu;s</td><td>-</td><td>-</td><td>-</td><td>256 B</td>
</tr><tr><td>StringCreate</td><td>100</td><td>3.152 &mu;s</td><td>0.4089 &mu;s</td><td>1.2056 &mu;s</td><td>3.500 &mu;s</td><td>-</td><td>-</td><td>-</td><td>824 B</td>
</tr><tr><td>StringBuilderDefault</td><td>100</td><td>2.385 &mu;s</td><td>0.0512 &mu;s</td><td>0.0718 &mu;s</td><td>2.400 &mu;s</td><td>-</td><td>-</td><td>-</td><td>2280 B</td>
</tr><tr><td>StringBuilderInitialCapacity</td><td>100</td><td>3.216 &mu;s</td><td>0.4154 &mu;s</td><td>1.2116 &mu;s</td><td>2.400 &mu;s</td><td>-</td><td>-</td><td>-</td><td>1696 B</td>
</tr><tr><td>StringCreate</td><td>400</td><td>3.224 &mu;s</td><td>0.0647 &mu;s</td><td>0.0664 &mu;s</td><td>3.200 &mu;s</td><td>-</td><td>-</td><td>-</td><td>3224 B</td>
</tr><tr><td>StringBuilderDefault</td><td>400</td><td>19.884 &mu;s</td><td>1.4567 &mu;s</td><td>4.2492 &mu;s</td><td>20.500 &mu;s</td><td>-</td><td>-</td><td>-</td><td>7896 B</td>
</tr><tr><td>StringBuilderInitialCapacity</td><td>400</td><td>5.924 &mu;s</td><td>0.4355 &mu;s</td><td>1.2773 &mu;s</td><td>6.300 &mu;s</td><td>-</td><td>-</td><td>-</td><td>6496 B</td>
</tr></tbody></table>

Gördüğümüz gibi `Create` metodu kullandığımızda hem kullanılan memory daha düşük hem de daha performanslı bir şekilde stringi yaratabiliyoruz. Ancak tabi ki her performansla ilgili yazdığımız yazıda olduğu gibi burada da kullanımları kendi senaryolarınızla karşılaştırıp, benchmarking yapmanız ve ona göre karar vermeniz en doğrusu. Özellikle çok fazla çalışan ve string üretilen kodlarda kullanmak size büyük kazançlar sağlayabilir. Ancak çok sık çalışmayan yerlerde beklediğiniz faydayı da göremeyebilirsiniz. Her yerde bu metodu kullanmak yerine ihtiyaç duyduğumuz doğru yerlerde kullanabiliyor olmak en güzeli. 

Bir sonraki yazıda görüşmek üzere, 