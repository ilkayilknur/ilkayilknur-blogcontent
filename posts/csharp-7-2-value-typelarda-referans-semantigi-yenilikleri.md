# C# 7.2 - Value Typelarda Referans Semantiği Yenilikleri #

Visual Studio 15.5 update'inin release olması ile beraber C#'ın bir sonraki minor release'i olan 7.2 versiyonu da artık RTM oldu. Daha önceki C# 7.1 yazımda da bahsettiğim üzere minor C# versiyonları Visual Studio içerisinde default olarak aktif olmuyor ve bu özellikleri kullanabilmemiz için ufak bir ayar yapmamız gerekiyor. Bu gerekli olan ayara da <a href="http://www.ilkayilknur.com/csharp-7-1-yenilikleri" target="_blank">C# 7.1 Yenilikleri</a> yazımdan ulaşabilirsiniz.

C# 7.2'de de yine C# 7.1'de olduğu gibi ufak ve faydalı yenilikler var. Ancak C# 7.2 sadece ufak yeniliklerin olduğu bir release değil. C# 7.2'nin ana teması, value type'larla çalışırken onları pass-by-reference mantığı ile kullanarak value typeların sahip olduğu kopyalanma özelliklerinin önüne geçip allocationı ve kopyalanma operasyonlarını azaltmak. Bu yazımızın konusu da bu tema kapsamında gelen yenilikler. C# 7.2 ile beraber gelen diğer yeniliklere de başka bir yazıda göz atacağız.

## Bu Özellikler Neden Gerekliydi ?

Value typelar, özellikle de structlar performans kritik algoritmalarda, oyunlarda, 3D modelleme vs.. gibi alanlarda oldukça fazla kullanılmakta. Bunun en önemli nedeni value typeların heap allocationa neden olmaması nedeniyle Garbage Collector'ın getirebileceği tahmin edilemeyen yüklerden ve sorunlardan olabildiğince kaçınmak. Ancak value typelar bir metoda parametre olarak geçildiklerinde eğer herhangi bir ref parametre kullanılmazsa value typeların bir kopyası yaratılıp geçilir *(pass-by-value)* . Örneğin, .NET Framework içerisindeki **System.Windows.Media.Media3D.Matrix3D**  structı içerisinde 16 tane double property bulunmakta. Bu da toplamda 128 byte ediyor. Dolayısıyla siz bu structı metotlara parametre olarak geçmeye kalkarsanız bu 128 bytelık struct sürekli olarak kopyalanarak metotlara geçilecektir. Elinizde bu structlardan oldukça fazla olduğunu düşünürseniz, bir metot çağırımı bile oldukça sıkıntılı sonuçlara yol açabilir. Bundan kaçınmak için tabi ki de C# içerisinde bazı çözümler var. Ancak C# 7.2 bu özelliklere ekstra yenilikler ekleyerek özellikle structlarla çalışmayı daha kolay ve özellikle güvenilir hale getiriyor. Dolayısıyla aşağıda anlatmaya çalışacağım yenilikleri bu konseptte değerlendirmekte fayda var.

## in Parametreleri ##

Şu ana kadar C# içerisinde bir value type'ı bir metoda referansıyla parametre olarak geçmek istediğimizde **out ve ref parametrelerini** kullanabiliyorduk. Böylece value typeların metot çağırılırken kopyalanmasının önüne geçebiliyoruz ve referansıyla gönderebiliyoruz. Bu seçeneklerden, out parametreyi kullandığımızda, out parametrede çağırılan metot bu değişkene **mutlaka bir değer ataması yapması gerekirken**, ref parametrelerde ise değişken yine referansı ile metoda gelirken metodun değişken üzerinde bir değişiklik yapması **zorunlu değil**.

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Out(<span style="color:blue;">out</span>&nbsp;<span style="color:blue;">int</span>&nbsp;x)<br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;x&nbsp;=&nbsp;12;<br/>}<br/> <br/><span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Ref(<span style="color:blue;">ref</span>&nbsp;<span style="color:blue;">int</span>&nbsp;x)<br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//Bazı&nbsp;durumlarda&nbsp;değiştirilebilir.</span><br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>&nbsp;(x&nbsp;&gt;&nbsp;10)<br/>&nbsp;&nbsp;&nbsp;&nbsp;{<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;x&nbsp;=&nbsp;9;<br/>&nbsp;&nbsp;&nbsp;&nbsp;}<br/>}</pre>

C# 7.2 ile beraber gelen **in parametreleri** ise yine değişkenin referansıyla metoda gönderilmesini sağlarken çağrılan metodun bu değişkeni değiştiremeyeceğinden emin olunmasını sağlıyor. Böylece çağırılan metot in parametreyi değiştiremeyeceği için gönderdiğiniz value type'ın özellikle de struct'ın değişmediğinden emin olabilirsiniz.

Örneğin,
<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;In(<span style="color:blue;">in</span>&nbsp;<span style="color:blue;">int</span>&nbsp;x)<br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;x&nbsp;=&nbsp;12;<br/>}</pre>

Yukarıdaki gibi gelen bir in parametreye bir değer ataması yapmak isterseniz derleme hatası alıyorsunuz.


in parametre kabul eden bir metodu çağırmak istediğimizde iki şekilde çağırma yapabilmemiz mümkün. Birincisi out parametrelerde olduğu gibi in parametrenin başına in keywordünü eklemek.

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Main(<span style="color:blue;">string</span>[]&nbsp;args)<br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">int</span>&nbsp;k&nbsp;=&nbsp;1;<br/>&nbsp;&nbsp;&nbsp;&nbsp;In(<span style="color:blue;">in</span>&nbsp;k);<br/>}</pre>

Diğeri ise in keywordünü eklemeden çağırmak. Eğer in parametre yolladığınızı kodu okuyan kişinin bilmesini istiyorsanız parametrenin başına in yazmanız faydalı olabilir. Ama kullanım şekli tamamen size kalmış. 

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Main(<span style="color:blue;">string</span>[]&nbsp;args)<br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">int</span>&nbsp;k&nbsp;=&nbsp;1;<br/> <br/>&nbsp;&nbsp;&nbsp;&nbsp;In(<span style="color:blue;">in</span>&nbsp;k);&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br/>&nbsp;&nbsp;&nbsp;&nbsp;In(k);<br/>}</pre>

in parametrelerin en büyük avantajı value typelar ile elde ediliyor.Reference typelarda etkisinin pek olmadığını belirtmemizde fayda var. 

## ref readonly returns

in parametrelerle, parametre referansıyla ve değiştirelemez bir şekilde metoda parametre geçilirken bu özellikle de dönüş değeri olan value type referansıyla döndürülür ve bu referans üzerinden döndürülen value type değiştirilemez. Bu özelliğin faydalı olabileceği kullanım alanlarında birini kısaca anlatmaya çalışalım.

Örneğin Point isimli  bir struct olduğunu düşünelim. 

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">struct</span>&nbsp;<span style="color:#2b91af;">Point</span><br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">int</span>&nbsp;X&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">set</span>;&nbsp;}<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">int</span>&nbsp;Y&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">set</span>;&nbsp;}<br/>}<br/></pre>

Bunun için Default değerini kapsayacak bir property koymak istersek şu şekilde bir ekleme yapmamız gerekir.

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">struct</span>&nbsp;<span style="color:#2b91af;">Point</span><br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">int</span>&nbsp;X&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">set</span>;&nbsp;}<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">int</span>&nbsp;Y&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">set</span>;&nbsp;}<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:#2b91af;">Point</span>&nbsp;Default&nbsp;=&gt;&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Point</span>();<br/>}</pre>

Burada her Default propertysi kullanıldığında stackde yeni bir Point yaratılır. Halbuki bunun yerine **ref readonly** olarak döndürürsek tek bir Point yaratıp sürekli olarak bunun referansını döndürebiliriz. 

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">struct</span>&nbsp;<span style="color:#2b91af;">Point</span><br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">int</span>&nbsp;X&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">set</span>;&nbsp;}<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">int</span>&nbsp;Y&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">set</span>;&nbsp;}<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:#2b91af;">Point</span>&nbsp;defaultp&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Point</span>();<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">ref</span>&nbsp;<span style="color:blue;">readonly</span>&nbsp;<span style="color:#2b91af;">Point</span>&nbsp;Default&nbsp;=&gt;&nbsp;<span style="color:blue;">ref</span>&nbsp;defaultp;<br/>}</pre>

Bu şekilde Point structını tanımladıktan sonra Point üzerinden Default propertysine aşağıdaki gibi ulaşabiliriz. 

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Main(<span style="color:blue;">string</span>[]&nbsp;args)<br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">ref</span>&nbsp;<span style="color:blue;">readonly</span>&nbsp;<span style="color:blue;">var</span>&nbsp;defaultp&nbsp;=&nbsp;<span style="color:blue;">ref</span>&nbsp;<span style="color:#2b91af;">Point</span>.Default;<br/>}</pre>

Bu şekilde bir kullanımda defaultp değişkeni içerisinde readonly referans bulunduğu için Main metot üzerinde defaultp değişkeni üzerinden bir değer ataması yapılması mümkün olmayacaktır ve compiler kodu derlemeyecektir.

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Main(<span style="color:blue;">string</span>[]&nbsp;args)<br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">ref</span>&nbsp;<span style="color:blue;">readonly</span>&nbsp;<span style="color:blue;">var</span>&nbsp;defaultp&nbsp;=&nbsp;<span style="color:blue;">ref</span>&nbsp;<span style="color:#2b91af;">Point</span>.Default;<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//ref&nbsp;readonly&nbsp;!!!</span><br/>&nbsp;&nbsp;&nbsp;&nbsp;defaultp.X&nbsp;=&nbsp;1;&nbsp;<span style="color:green;">//---&gt;&nbsp;Mümkün&nbsp;değil&nbsp;!!!</span><br/>}</pre>

ref readonly olarak dönen bir değerin kopyasını almak istersek de dönen değeri ref readonly olmayan bir değişkene atamasını yapmamız yeterli. 

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Main(<span style="color:blue;">string</span>[]&nbsp;args)<br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;defaultp&nbsp;=&nbsp;<span style="color:#2b91af;">Point</span>.Default;<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//Default&#39;in&nbsp;yeni&nbsp;bir&nbsp;kopyası&nbsp;oluşturuldu.&nbsp;</span><br/>&nbsp;&nbsp;&nbsp;&nbsp;defaultp.X&nbsp;=&nbsp;1;&nbsp;<span style="color:green;">//---&gt;&nbsp;Artık&nbsp;mümkün&nbsp;:)</span><br/>}</pre>

Bu özelliği isterseniz metotların dönüş değerleri için de kullanabilirsiniz. Biz örnek olarak property yaptık ancak bu özellik metotların dönüş değerleri için de kullanılabilir.

## readonly Structs

C# 7.2 öncesinde readonly olarak tanımlanan bir struct variable'ı üzerinde struct içerisindeki bir instance metodu çağırmak istediğimizde derleyeceği o metodun structı değiştirip değiştirmeyeceğini bilemediği için otomatik olarak structı arka planda geçici bir değişkene atayıp o değişken üzerinden ilgili metodu çalıştırıyordu. Bunun da performans olarak bazı sıkıntıları olabiliyordu. Örnek senaryo olarak <a href="https://codeblog.jonskeet.uk/2014/07/16/micro-optimization-the-surprising-inefficiency-of-readonly-fields/" target="_blank">https://codeblog.jonskeet.uk/2014/07/16/micro-optimization-the-surprising-inefficiency-of-readonly-fields/</a> bu yazıyı okuyabilirsiniz. Bu performans sorunları nedeniyle de çoğu zaman readonly kullanmaktan vazgeçilebiliyordu.


C# 7.2 ile beraber struct tanımlamasının başına readonly yazarak structların readonly olmasını sağlayabiliyoruz. Böylece derleyici de yukarıda belirttiğim senaryoda geçici bir değişken yaratmak durumunda kalmıyor ve performans sıkıntılarının da önüne geçilmiş olunuyor. 

readonly olarak tanımladığımız structların içindeki değerlere atamalar sadece constructor içerisinde yapılabiliyor. Bunun haricinde başka bir yerde atama yapmak mümkün değil. 

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">readonly</span>&nbsp;<span style="color:blue;">struct</span>&nbsp;<span style="color:#2b91af;">Point</span><br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">int</span>&nbsp;X&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;}<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">int</span>&nbsp;Y&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;}<br/> <br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;Point(<span style="color:blue;">int</span>&nbsp;x,&nbsp;<span style="color:blue;">int</span>&nbsp;y)<br/>&nbsp;&nbsp;&nbsp;&nbsp;{<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;X&nbsp;=&nbsp;x;<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Y&nbsp;=&nbsp;y;<br/>&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<br/>}</pre>

Şimdilik bu tema kapsamında bu yazıda bahsedeceklerimiz bu kadar. C# 7.2 ile beraber gelen ve yine tema kapsamında olan **Span< T>**'nin kullanımından ise başka bir yazıda bahsediyor olacağım. Bu yeniliklerin özellikle yüksek performanslı sistemlerin, oyunların optimizasyonu amacıyla yapıldığını hatırlatmakta fayda var. Biz belki kod yazarken sürekli structları veya bu yenilikleri kullanmıyor olabiliriz ancak kullandığımız frameworkler, toollar bu tipleri arka planda kullanıyorlar. Dolayısıyla bu yenilikler doğrudan olmasa bile dolaylı yoldan bizleri etkiliyor olacak. Mesela <a href="https://github.com/dotnet/corefx/pull/24997" target="_blank">buradaki</a> pull request içerisinde **corefx** içerisinde bu değişikliklerin adapte edildiğini görebilirsiniz.

Bir sonraki yazıda görüşmek üzere,