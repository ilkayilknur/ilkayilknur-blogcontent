Bu yazımızda .NET Core 2.1 ile beraber gelen `System.IO.Pipelines` namespace'i altına bulunan `Pipe` yapısını inceleyeceğiz. Pipe'lar kısaca .NET içerisinde performanslı IO yapmamızı sağlayan bir yapı. Geleneksel yöntemlerle streaming datasını parse eden uygulamalarda daha az allocationa neden olan ve performanslı çalışan bir kod yazmamız gerektiğinde pek çok farklı implementasyon yapmamız gerekirken Pipe yapısı bu implementasyonların hepsini built-in olarak içerisinde barındırması itibariyle bize güzel ve kullanımı kolay bir altyapı sağlıyor.

Şimdi gelin ilk olarak Pipeline'lar olmadan önce nasıl kod yazıyorduk onu inceleyelim. Bir network streamden satır satır data okuyup işleyen şu kodu düşünelim. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">async</span>&nbsp;<span style="color:#2b91af;">Task</span>&nbsp;ProcessLinesAsync(<span style="color:#2b91af;">NetworkStream</span>&nbsp;stream)
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;buffer&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:blue;">byte</span>[1024];
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">await</span>&nbsp;stream.ReadAsync(buffer,&nbsp;0,&nbsp;buffer.Length);
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Process&nbsp;a&nbsp;single&nbsp;line&nbsp;from&nbsp;the&nbsp;buffer</span>
&nbsp;&nbsp;&nbsp;&nbsp;ProcessLine(buffer);
}</pre>

Bu koddaki sorunlara baktığımızda şunlar karşımıza çıkıyor.

* Read operasyonundan dönen data içerisinde birden fazla satır data olabilir.
* Read operasyonundan dönen datada bir satır sonu bulunmayabilir ve satır sonu bulunana kadar okumaya devam etmek gerekebilir.

İlk maddedeki sorunu çözmek istediğimizde aşağıdaki gibi bir implementasyon yapabiliriz.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">async</span>&nbsp;<span style="color:#2b91af;">Task</span>&nbsp;ProcessLinesAsync(<span style="color:#2b91af;">NetworkStream</span>&nbsp;stream)
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;buffer&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:blue;">byte</span>[1024];
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;bytesBuffered&nbsp;=&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;bytesConsumed&nbsp;=&nbsp;0;
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">while</span>&nbsp;(<span style="color:blue;">true</span>)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;bytesRead&nbsp;=&nbsp;<span style="color:blue;">await</span>&nbsp;stream.ReadAsync(buffer,&nbsp;bytesBuffered,&nbsp;buffer.Length&nbsp;-&nbsp;bytesBuffered);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>&nbsp;(bytesRead&nbsp;==&nbsp;0)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;EOF</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Keep&nbsp;track&nbsp;of&nbsp;the&nbsp;amount&nbsp;of&nbsp;buffered&nbsp;bytes</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bytesBuffered&nbsp;+=&nbsp;bytesRead;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;linePosition&nbsp;=&nbsp;-1;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">do</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Look&nbsp;for&nbsp;a&nbsp;EOL&nbsp;in&nbsp;the&nbsp;buffered&nbsp;data</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;linePosition&nbsp;=&nbsp;<span style="color:#2b91af;">Array</span>.IndexOf(buffer,&nbsp;(<span style="color:blue;">byte</span>)<span style="color:#a31515;">&#39;</span><span style="color:#b776fb;">\n</span><span style="color:#a31515;">&#39;</span>,&nbsp;bytesConsumed,&nbsp;bytesBuffered&nbsp;-&nbsp;bytesConsumed);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>&nbsp;(linePosition&nbsp;&gt;=&nbsp;0)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Calculate&nbsp;the&nbsp;length&nbsp;of&nbsp;the&nbsp;line&nbsp;based&nbsp;on&nbsp;the&nbsp;offset</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;lineLength&nbsp;=&nbsp;linePosition&nbsp;-&nbsp;bytesConsumed;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Process&nbsp;the&nbsp;line</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ProcessLine(buffer,&nbsp;bytesConsumed,&nbsp;lineLength);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Move&nbsp;the&nbsp;bytesConsumed&nbsp;to&nbsp;skip&nbsp;past&nbsp;the&nbsp;line&nbsp;we&nbsp;consumed&nbsp;(including&nbsp;\n)</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bytesConsumed&nbsp;+=&nbsp;lineLength&nbsp;+&nbsp;1;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">while</span>&nbsp;(linePosition&nbsp;&gt;=&nbsp;0);
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

Read operasyonu sonucunda birden fazla satır gelmesi sorununu çözdük. Şimdi gelelim diğer soruna. Bir satır eğer 1024 bytetan büyük ise satır sonuna kadar okumaya devam edip elimizdeki bufferı resize etmemiz gerekiyor. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">async</span>&nbsp;<span style="color:#2b91af;">Task</span>&nbsp;ProcessLinesAsync(<span style="color:#2b91af;">NetworkStream</span>&nbsp;stream)
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">byte</span>[]&nbsp;buffer&nbsp;=&nbsp;ArrayPool&lt;<span style="color:blue;">byte</span>&gt;.Shared.Rent(1024);
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;bytesBuffered&nbsp;=&nbsp;0;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;bytesConsumed&nbsp;=&nbsp;0;
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">while</span>&nbsp;(<span style="color:blue;">true</span>)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Calculate&nbsp;the&nbsp;amount&nbsp;of&nbsp;bytes&nbsp;remaining&nbsp;in&nbsp;the&nbsp;buffer</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;bytesRemaining&nbsp;=&nbsp;buffer.Length&nbsp;-&nbsp;bytesBuffered;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>&nbsp;(bytesRemaining&nbsp;==&nbsp;0)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Double&nbsp;the&nbsp;buffer&nbsp;size&nbsp;and&nbsp;copy&nbsp;the&nbsp;previously&nbsp;buffered&nbsp;data&nbsp;into&nbsp;the&nbsp;new&nbsp;buffer</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;newBuffer&nbsp;=&nbsp;ArrayPool&lt;<span style="color:blue;">byte</span>&gt;.Shared.Rent(buffer.Length&nbsp;*&nbsp;2);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Buffer</span>.BlockCopy(buffer,&nbsp;0,&nbsp;newBuffer,&nbsp;0,&nbsp;buffer.Length);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Return&nbsp;the&nbsp;old&nbsp;buffer&nbsp;to&nbsp;the&nbsp;pool</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ArrayPool&lt;<span style="color:blue;">byte</span>&gt;.Shared.Return(buffer);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;buffer&nbsp;=&nbsp;newBuffer;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bytesRemaining&nbsp;=&nbsp;buffer.Length&nbsp;-&nbsp;bytesBuffered;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;bytesRead&nbsp;=&nbsp;<span style="color:blue;">await</span>&nbsp;stream.ReadAsync(buffer,&nbsp;bytesBuffered,&nbsp;bytesRemaining);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>&nbsp;(bytesRead&nbsp;==&nbsp;0)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;EOF</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Keep&nbsp;track&nbsp;of&nbsp;the&nbsp;amount&nbsp;of&nbsp;buffered&nbsp;bytes</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bytesBuffered&nbsp;+=&nbsp;bytesRead;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">do</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Look&nbsp;for&nbsp;a&nbsp;EOL&nbsp;in&nbsp;the&nbsp;buffered&nbsp;data</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;linePosition&nbsp;=&nbsp;<span style="color:#2b91af;">Array</span>.IndexOf(buffer,&nbsp;(<span style="color:blue;">byte</span>)<span style="color:#a31515;">&#39;</span><span style="color:#b776fb;">\n</span><span style="color:#a31515;">&#39;</span>,&nbsp;bytesConsumed,&nbsp;bytesBuffered&nbsp;-&nbsp;bytesConsumed);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>&nbsp;(linePosition&nbsp;&gt;=&nbsp;0)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Calculate&nbsp;the&nbsp;length&nbsp;of&nbsp;the&nbsp;line&nbsp;based&nbsp;on&nbsp;the&nbsp;offset</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;lineLength&nbsp;=&nbsp;linePosition&nbsp;-&nbsp;bytesConsumed;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Process&nbsp;the&nbsp;line</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ProcessLine(buffer,&nbsp;bytesConsumed,&nbsp;lineLength);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//&nbsp;Move&nbsp;the&nbsp;bytesConsumed&nbsp;to&nbsp;skip&nbsp;past&nbsp;the&nbsp;line&nbsp;we&nbsp;consumed&nbsp;(including&nbsp;\n)</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bytesConsumed&nbsp;+=&nbsp;lineLength&nbsp;+&nbsp;1;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">while</span>&nbsp;(linePosition&nbsp;&gt;=&nbsp;0);
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

Bu implementasyona baktığımızda da oldukça fazla buffer kopyalaması yaptığımızı görüyoruz. Bunun da çözümü arkada bir buffer listesi tutup kopyalama işlemlerini ve memory kullanımını azaltmak. Yani özetle baktığımızda etkin, performanslı, az allocationa neden olan bir kod yazmaya kalktığımızda pek çok şeyi düşünüp implemente edip, doğru çalıştığından emin olmamız gerekiyor. Bu da oldukça zor. Şimdi gelelim pipeline'lar bu konuyu nasıl çözüyorlar ona bakalım. 

### System.IO.Pipelines

Yukarıda yaptığımız implementasyona nazaran Pipelineların en büyük avantajı buffer yönetimini arka planda etkin bir biçimde kendisinin yapması. Bu da bize hem performanslı hem de daha az memory tüketen bir altyapı sağlıyor. Aynı zamanda kod yazarken performans ve etkinlik gibi konulara odaklanmak yerine business tarafına daha çok odaklanmamızı sağlamakta.

Pipe yapısını kullanabilmemiz için öncelikle projemize `System.IO.Pipelines` nuget paketini yüklememiz gerekiyor. Paketi yükledikten sonra kolayca aşağıdaki gibi Pipe yaratabiliyoruz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;pipe&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Pipe</span>();
</pre>

Pipe yarattıktan içerisinde Writer ve Reader isimli iki property görüyoruz. PipeWriter bizim pipe'a data yazmamızı sağlarken PipeReader ise pipe'tan data okumamızı sağlayan kısım. Önce pipe'a veri yazma kısmına bakalım. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">async</span>&nbsp;<span style="color:#2b91af;">Task</span>&nbsp;FillPipeAsync(<span style="color:#2b91af;">Socket</span>&nbsp;socket,&nbsp;<span style="color:#2b91af;">PipeWriter</span>&nbsp;writer)
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">while</span>&nbsp;(<span style="color:blue;">true</span>)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Memory</span>&lt;<span style="color:blue;">byte</span>&gt;&nbsp;memory&nbsp;=&nbsp;writer.GetMemory();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">try</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">int</span>&nbsp;bytesRead&nbsp;=&nbsp;<span style="color:blue;">await</span>&nbsp;socket.ReceiveAsync(memory,&nbsp;<span style="color:#2b91af;">SocketFlags</span>.None);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>&nbsp;(bytesRead&nbsp;==&nbsp;0)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;writer.Advance(bytesRead);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">catch</span>&nbsp;(<span style="color:#2b91af;">Exception</span>&nbsp;ex)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LogError(ex);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">FlushResult</span>&nbsp;result&nbsp;=&nbsp;<span style="color:blue;">await</span>&nbsp;writer.FlushAsync();
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>&nbsp;(result.IsCompleted)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;writer.Complete();
}</pre>

Buradaki koda baktığımızda ilk olarak PipeWriterdan bir buffer istiyoruz. Buradaki buffer array pooldan geliyor. Sonrasında socket üzerinden `ReceiveAsync` metodunu çağırarak verinin buffera yazılmasını sağlıyoruz. Burada buffer `Memory<T>` tipinde olduğu için buffer yaratma veya kopyalama gibi sorunlarla uğraşmıyoruz. `Advance` metodunu kullanarak writera ne kadar data yazıldığını söylüyoruz. Böylece arkadaki bufferların etkin bir şekilde yönetimini pipe'a bırakıyoruz. Sonrasında da `FlushAsync` metoduyla yazdığımız verinin Reader tarafından erişilebilir olmasını sağlıyoruz. 

Şimdi de `Reader` tarafına bakalım. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">async</span>&nbsp;<span style="color:#2b91af;">Task</span>&nbsp;ReadPipeAsync(<span style="color:#2b91af;">PipeReader</span>&nbsp;reader)
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">while</span>&nbsp;(<span style="color:blue;">true</span>)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">ReadResult</span>&nbsp;result&nbsp;=&nbsp;<span style="color:blue;">await</span>&nbsp;reader.ReadAsync();
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">ReadOnlySequence</span>&lt;<span style="color:blue;">byte</span>&gt;&nbsp;buffer&nbsp;=&nbsp;result.Buffer;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">SequencePosition</span>?&nbsp;position&nbsp;=&nbsp;<span style="color:blue;">null</span>;
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">do</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;position&nbsp;=&nbsp;buffer.PositionOf((<span style="color:blue;">byte</span>)<span style="color:#a31515;">&#39;</span><span style="color:#b776fb;">\n</span><span style="color:#a31515;">&#39;</span>);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>&nbsp;(position&nbsp;!=&nbsp;<span style="color:blue;">null</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ProcessLine(buffer.Slice(0,&nbsp;position.Value));
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;buffer&nbsp;=&nbsp;buffer.Slice(buffer.GetPosition(1,&nbsp;position.Value));
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">while</span>&nbsp;(position&nbsp;!=&nbsp;<span style="color:blue;">null</span>);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;reader.AdvanceTo(buffer.Start,&nbsp;buffer.End);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>&nbsp;(result.IsCompleted)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;reader.Complete();
}</pre>

Okuma kısmına gelince `ReadAsync` metodundan dönen `ReadResult` içindeki bufferın tipinin `ReadOnlySequence` olduğunu görüyoruz. ReadOnlySequence tipi bufferları linked list olarak tutan yapı. Daha da açık anlatmak gerekirse `Memory<T>` instancelarını birbirine bağlayan bir struct. Sonrasında bu liste üzerinde satır sonunun pozisyonunu buluyoruz. Bulduktan sonra da `Slice` metoduyla ilgili memory alanını `ProcessLine` metoduna parametre geçiyoruz. Span ve Memory tiplerinden hatırlayacağımız üzere Slice operasyonunda herhangi bir allocation vs.. olmadığı için bu işlem oldukça hızlı ve etkin olarak yapılıyor. `AdvanceTo` metoduylada pipe üzerinden ne kadar data consume ettiğimizi bildiriyoruz. Burada dikkat etmemiz gereken nokta da `AdvanceTo` metodunu çağırdıktan sonra önceki Read operasyonuyla gelen bufferı kullanmamak. Bu noktada o buffer artık tekrardan Pipe'ın yönetimine giriyor. Artık yapmamız gereken tekrardan ReadAsync metodunu kullanarak Pipe'tan yeni datayı okumak.

### Pipeların ASP.NET Core İçerisinde Kullanımı

Pipeların ilk olarak ortaya çıktığı yer Kestrel web serverı. Kestrel ekibi web serverın performansını arttırmak için Pipe implementasyonunu yapıyor ve sonrasında da bu implementasyon olgunlaştıktan sonra frameworkün içerisine ekleniyor. 

Şu anda ASP.NET Core içerisinde pek çok noktada pipelar streamlerin yerine kullanılıyor. Request bodysini okumak istediğimizde `Request.Body` stream dönerken `Request.BodyReader` `PipeReader` dönmekte. Aynı şekilde `Response.Body` yine stream dönerken `Response.BodyWriter` `PipeWriter` dönüyor. Bu nedenle daha performanslı ve etkin implementasyonlar için Pipe versiyonlarını kullanmamızda fayda var.

Kaynak kodlar vs.. gibi konularda <a href="https://devblogs.microsoft.com/dotnet/system-io-pipelines-high-performance-io-in-net/" target="_blank">buradaki</a> makaleden faydalandım. İlgilenmek isteyen linke tıklayıp daha detaylı bilgi alabilirler. 

Bir sonraki yazıda görüşmek üzere,