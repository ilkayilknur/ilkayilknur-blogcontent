# Roslyn Syntax Tree API'larına Giriş #

Daha önce **.NET Compiler Platform (Roslyn)** ile ilgili yazdığımız tüm yazılarda ve videolarda Roslyn projesinin derleyicileri bir kara kutu olmaktan çıkardığından ve artık bizim de derleyicilerin kullandığı yapıları kullanıp elimizdeki kodu analiz edebileceğimizden çokca bahsetmiştik.

Bugün bu konuya syntax tree API'ları ile giriş yapacağız. Yazdığımız kodlar derleme işlemi sırasında ilk olarak derleyici tarafından parse edilirler ve bu işlemin sonucunda bir syntax tree oluşturulur. Bu syntax tree içerisinde yazmış olduğumuz kod içerisindeki herşey hiyerarşik bir şekilde tutulur. 

Örneğin, aşağıdaki gibi son derece basit bir kodumuz olduğunu düşünelim.

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">C</span><br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;M()<br/>&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;}<br/>}</pre>

Bu kod derleyici tarafından derlendiğinde aşağıdaki gibi bir syntax tree oluşturulur. 

![](http://az718566.vo.msecnd.net/uploads/2016/02/SyntaxTree.png)

Gördüğümüz gibi yazmış olduğumuz kod içerisinde bulunan her bir tanımlama, boşluk, keyword vb... tüm bileşenler syntax tree içerisinde eksiksiz tutuluyor. Syntax tree'ler derleme aşamasının en önemli yapılarından biri. Çünkü kod içerisindeki tüm bileşenler syntax tree içerisinde düzenleniyor ve kategorilere ayrılıyor. Derleme işlemi sırasında parserdan sonraki adımlarda da bu aşamada üretilen syntax tree kullanılıyor.

Derleyici tarafından yaratılan syntax tree içerisinde 3 farklı tipte eleman bulunuyor. 

- Syntax Node
- Syntax Token
- Syntax Trivia

Syntax Node'lar kod içerisindeki tanımlamaları,ifadeleri, koşul ifadeleri vb... dil içerisinde temel yapıları temsil ederler. Örneğin class tanımlaması en tepede bir syntax node ile ifade edilir. Roslyn içerisinde her yapıya özel SyntaxNode tipinden türetilmiş özel SyntaxNode sınıfları vardır. Yukarıdaki syntax tree resminde mavi olan nodelar birer syntax node'dur.

Syntax Token'lar yine dil içerisinde bulunan en ufak bileşenleri temsil eden yapılardır. Syntax tokenların tree içerisinde altında hiçbir zaman başka tokenlar veya syntax node'lar bulunmaz. Roslyn içerisinde SyntaxToken'lar için tek bir tip bulunurken bu tip içerisindeki Kind propertysi ile token tipleri birbirinden ayrıştırılabilir. Yukarıdaki syntax tree resmindeki yeşil renkli nodelar syntax tokendır. 

Syntax Trivia'lar da kod içerisinde bulunan ancak derleme işlemi için çokta büyük anlam ifade etmeyen bileşenler için kullanılır. Bu bileşenler kod içerisindeki yorumlar, boşluklar gibi yapılardır. Syntax trivia'lar syntax tokenlara bağlıdırlar. Her bir syntax token'ın LeadingTrivia ve TrailingTrivia collectionları vardır. Bu collectionlar ile token'a bağlı olan trivia'lara erişilebilir.

Bu kadar teori şimdilik yeterli :) Şimdi hemen bir console uygulaması açalım ve bu API'ları nasıl kullanacağımıza bakalım.

Projemizi yarattıktan sonra yapmamız gereken ilk şey Microsoft.CodeAnalysis nuget paketini yüklemek. 

    Install-Package Microsoft.CodeAnalysis

Bu işlemi de gerçekleştirdikten sonra artık hazırız. Varolan bir kodun syntax tree'sini çıkarabilmek için **Microsoft.CodeAnalysis.CSharp** namespace'i içerisinde **CSharpSyntaxTree** tipinin **ParseText** metodunu kullanacağız. Bu metot string olarak kendisine verilen bir kodun syntax treesini verir. 

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Program</span><br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Main()<br/>&nbsp;&nbsp;&nbsp;&nbsp;{<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;tree&nbsp;=&nbsp;<span style="color:#2b91af;">CSharpSyntaxTree</span>.ParseText(<span style="color:maroon;">@&quot;class&nbsp;C<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;void&nbsp;M()<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;}<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&quot;</span>);<br/>&nbsp;&nbsp;&nbsp;&nbsp;}<br/>}</pre>

ParseText metodunu çağırdıktan sonra artık elimizde bir syntax tree var. Bu syntax tree üzerinde istediğimiz elemanlara ulaşabiliriz ve ağaç üzerinde gezebiliriz. İlk olarak yaratılan veri yapısı bir ağaç olduğu için ağacın root elemanına ulaşalım.

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Program</span><br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Main(<span style="color:blue;">string</span>[]&nbsp;args)<br/>&nbsp;&nbsp;&nbsp;&nbsp;{<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;tree&nbsp;=&nbsp;<span style="color:#2b91af;">CSharpSyntaxTree</span>.ParseText(<span style="color:maroon;">@&quot;class&nbsp;C<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;void&nbsp;M()<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;}<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&quot;</span>);<br/> <br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;root&nbsp;=&nbsp;tree.GetRoot();<br/>&nbsp;&nbsp;&nbsp;&nbsp;}<br/>}&nbsp;&nbsp;&nbsp;</pre>

Ağacın rootuna ulaştıktan sonra artık ağaç içerisindeki her bir elamana da rahatlıkla ulaşabiliriz. Örneğin basit bir LINQ sorgusuyla tüm metot tanımlamalarına ulaşmamız mümkün.

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Program</span><br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Main(<span style="color:blue;">string</span>[]&nbsp;args)<br/>&nbsp;&nbsp;&nbsp;&nbsp;{<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;tree&nbsp;=&nbsp;<span style="color:#2b91af;">CSharpSyntaxTree</span>.ParseText(<span style="color:maroon;">@&quot;class&nbsp;C<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;void&nbsp;M()<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;}<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&quot;</span>);<br/> <br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;root&nbsp;=&nbsp;tree.GetRoot();<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;methods&nbsp;=&nbsp;root.DescendantNodes().OfType&lt;<span style="color:#2b91af;">MethodDeclarationSyntax</span>&gt;();<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>&nbsp;(<span style="color:blue;">var</span>&nbsp;method&nbsp;<span style="color:blue;">in</span>&nbsp;methods)<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Console</span>.WriteLine(method.ToString());<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br/>&nbsp;&nbsp;&nbsp;&nbsp;}<br/>}</pre>
 
 Bu kod çalıştığında parse ettiğimiz kod içerisinde tek metot olduğu için ve o metot da **M metodu** olduğu için M metodu doğrudan console'a yazılacak.
 
 ![](http://az718566.vo.msecnd.net/uploads/2016/02/roslyn-linq-output.png)
 
 Olayı biraz daha karmaşıklaştıralım ve parametre kabul eden metotların listesini çıkaralım.
 
 <pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Program</span><br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Main(<span style="color:blue;">string</span>[]&nbsp;args)<br/>&nbsp;&nbsp;&nbsp;&nbsp;{<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;tree&nbsp;=&nbsp;<span style="color:#2b91af;">CSharpSyntaxTree</span>.ParseText(<span style="color:maroon;">@&quot;class&nbsp;C<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;void&nbsp;M()<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;}<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;void&nbsp;M2(int&nbsp;a)<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;}<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;void&nbsp;M3(int&nbsp;a,string&nbsp;b)<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;}<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&quot;</span>);<br/> <br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;root&nbsp;=&nbsp;tree.GetRoot();<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;methods&nbsp;=&nbsp;root.DescendantNodes().OfType&lt;<span style="color:#2b91af;">MethodDeclarationSyntax</span>&gt;().Where(t&nbsp;=&gt;&nbsp;t.ParameterList.Parameters.Any());<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">foreach</span>&nbsp;(<span style="color:blue;">var</span>&nbsp;method&nbsp;<span style="color:blue;">in</span>&nbsp;methods)<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Console</span>.WriteLine(method.ToString());<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<br/>&nbsp;&nbsp;&nbsp;&nbsp;}<br/>}</pre>
 
 Parse ettiğimiz koda baktığımızda yazdığımız kodun çıktısının **M2 ve M3 metotları** olması lazım. Hemen kodu çalıştırıp sonucunu görelim.
 
![](http://az718566.vo.msecnd.net/uploads/2016/02/roslyn-linq-output-2.png)

Şimdi eminim aklınıza takılan şöyle bir sorun var. Bu sorguları bu şekilde iyi güzel yazıyoruzda metot tanımlamalarının **MethodDeclarationSyntax** tipinde olduğunu veya parametrelerin bu tip içerisindeki **ParameterList.Parameters** collectionında tutulduğunu nasıl bileceğiz ? Bu konu tabi sadece bizim değil aynı zaman da Roslyn'i ve Roslyn-Visual Studio entagrasyonunu yazan ekibin de bir sorunuydu zamanında. Bu nedenle geliştirdikleri bir araçla bu sorunu çözdüler. Şimdi bu araç artık Visual Studio içerisinde extension olarak sunuluyor. 

**Syntax Visualizer** dediğimiz pencere yardımıyla istediğimiz kodun syntax treesini herhangi bir kod çalıştırmadan görebiliriz ve her bir elemanın tüm propertylerine de ulaşabiliriz. Bu extensionı yüklemek için **.NET Compiler Platform SDK**'ini yüklememiz gerekiyor. Bu extension aynı zamanda Roslyn API'larıyla yaptığımız kod analizlerini Visual Studio extensionı olarak yayınlamamız sağlayan templateları da sağlıyor. Extensionı kurmak için [buradan](https://visualstudiogallery.msdn.microsoft.com/2ddb7240-5249-4c8c-969e-5d05823bcb89) VS Gallery'e gidebilir veya Visual Studio içerisiden **Tools => Extensions and Updates** menüsü üzerinde .NET Compiler Platform SDK'ini yükleyebilirsiniz.

Extension'ı yükledikten sonra **View => Other Windows => Syntax Visualizer** adımlarıyla Syntax Visualizer'ı açabilirsiniz.

![](http://az718566.vo.msecnd.net/uploads/2016/02/syntax-visualizer.gif)

Parse ettiğimiz kodu da Visual Studio içerisine kopyalarsak aşağıda gördüğünüz gibi hızlı bir şekilde elemanlara ulaşabilir ve propertylerini hızlı bir şekilde kontrol edebiliriz.

![](http://az718566.vo.msecnd.net/uploads/2016/02/syntax-visualizer-2.gif)

Bu yazımızda syntax tree API'larını kısa bir giriş yaptık ve LINQ ile syntax tree üzerinde elemanlara nasıl erişilebiliriz konusunu inceledik. Bir sonraki yazıda  syntax tree üzerinde dolaşmanın farklı bir yolunu inceleyeceğiz. 

Görüşmek üzere...