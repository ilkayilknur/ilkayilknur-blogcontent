Bu yazıda beni de uzun zamandan beri heyecanlandıran bir özelliği inceleyeceğiz. C# 9.0 ile beraber gelen programlama dili yeniliklerinin yanında belki de yeterli ilgiyi görmeyen ama uzun vadede pek çok şeyi değiştirebilecek olan özelliklerden biri source generatorlar. Source generatorlar bizim derleme zamanı sırasında varolan kodları analiz edip yeni kodlar yaratmamızı ve bunları da derlemeye dahil etmemizi sağlıyor. Üstelik tüm bu operasyonları  derleyici ile tam bir entegrasyon içerisinde yapabiliyoruz.

![source-generator](https://az718566.vo.msecnd.net/uploads/2020/12/06/source-generator.png)
*https://devblogs.microsoft.com/dotnet/introducing-c-source-generators/*

Aslında bir kodu analiz ederek yeni bir kod üretme olayı bizim için yeni bir şey değil. Bundan önce reflection, IL weaving, MSBuild tasklarıyla bunu bir şekilde başarabiliyorduk. Ancak bunlara baktığımızda her seçeneğin diğerine göre bir takım avantajları veya dezavantajları bulunmaktaydı. Örneğin, ASP.NET tarafında uygulamanız ayağa kalktığında içerisindeki controllerların, viewların bulunması aşaması reflection ile gerçekleşmekte. Bu nedenle uygulamanız ayağa kalktığında çok kısa bir süre de olsa bu operasyon tam olarak gerçekleşmeden gelen isteklere cevap veremez. Bunun gibi operasyonlar aslında derleme zamanında kodunuz analiz edilerek yapılabilir, gerekli kod üretilebilir ve uygulamalarımız daha hızlı ayağa kalkarak isteklere hızlıca cevap verebilir hale gelebilir. 

Bundan önceki yazılarımda App Trimmingden ve .NET 5 ile beraber daha da gelişen trimming yapısından bahsetmiştim. Bu linkerlar şu an preview aşamasında olsalarda .NET 6 ile beraber kullanımları daha da yaygınlaşıyor olacak. Bu nedenle uygulamalarımız içerisinde reflection kullandığımız noktaları azaltıp bu operasyonları derleme zamanına almamız bizim için oldukça önemli. 

Şimdi hızlıca source generator geliştirme kısmına gelirsek. Source generatorları basit bir .NET Standart projesi olarak geliştirebilmekteyiz. .NET Standart projesini yarattıktan sonra `Microsoft.CodeAnalysis.Csharp` paketini projemize eklememiz gerekiyor. 

````
dotnet add package Microsoft.CodeAnalysis.Csharp
````

Sonrasında bir class yaratıp bu classı `SourceGenerator` attribute'u ile işaretleyip aynı zamanda bu tipin `ISourceGenerator` interface'ini implemente etmesini sağlamamız gerekli. Son durumda generator olarak çalışacak olan tipimiz şu şekilde görünecek.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;">[<span style="color:#2b91af;">Generator</span>]
<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">TestGenerator</span>&nbsp;:&nbsp;<span style="color:#2b91af;">ISourceGenerator</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;<span style="color:#74531f;">Execute</span>(<span style="color:#2b91af;">GeneratorExecutionContext</span>&nbsp;<span style="color:#1f377f;">context</span>)
&nbsp;&nbsp;&nbsp;&nbsp;{
 
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;<span style="color:#74531f;">Initialize</span>(<span style="color:#2b91af;">GeneratorInitializationContext</span>&nbsp;<span style="color:#1f377f;">context</span>)
&nbsp;&nbsp;&nbsp;&nbsp;{
 
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

Bu tip içerisindeki `Initialize` metodu host (IDE, command-line compiler) tarafından  bir kez çağrılmakta ve `GeneratorInitializationContext` parametresi içerisindeki `RegisterForSyntaxNotifications` metodunu kullanarak bir `SyntaxReceiver` tanımlayarak derleme sırasında bize gelecek olan nodeları filtrelememize imkan vermekte. 

`Execute` metodu ise zaten tahmin edeceğiniz üzere kod yaratma aşamasının gerçekleştiği bölüm. Bu metoda parametre olarak gelen `GeneratorExecutionContext` tipi içerisinde de pek çok property ve metot bulunmakta. Ama en basit olarak bizim kullanacaklarımız `Compilation` propertysi ve `AddSource` metodu. Burada compilation nesnesine erişip kod üzerinde çeşitli analizler yapıp sonrasında da `AddSource` metodunu kullanarak yarattığımız kodu compilera bildirebiliriz. Örneğin aşağıda gibi bir Hello World uygulaması içerisinde basitçe bir kod yaratıp sonrasında da source olarak ekleyebiliriz.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;">[<span style="color:#2b91af;">Generator</span>]
<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">TestGenerator</span>&nbsp;:&nbsp;<span style="color:#2b91af;">ISourceGenerator</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;<span style="color:#74531f;">Execute</span>(<span style="color:#2b91af;">GeneratorExecutionContext</span>&nbsp;<span style="color:#1f377f;">context</span>)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;<span style="color:#1f377f;">builder</span>&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">StringBuilder</span>(<span style="color:maroon;">@&quot;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;using&nbsp;System;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;namespace&nbsp;HelloWorldGenerated
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;static&nbsp;class&nbsp;HelloWorld
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;static&nbsp;void&nbsp;SayHello()&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Console.WriteLine(</span><span style="color:#b776fb;">&quot;&quot;</span><span style="color:maroon;">Hello&nbsp;from&nbsp;generated&nbsp;code!</span><span style="color:#b776fb;">&quot;&quot;</span><span style="color:maroon;">);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&quot;</span>);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#1f377f;">context</span>.<span style="color:#74531f;">AddSource</span>(<span style="color:#a31515;">&quot;generatedCode.cs&quot;</span>,&nbsp;<span style="color:#2b91af;">SourceText</span>.<span style="font-weight:bold;color:#74531f;">From</span>(<span style="color:#1f377f;">builder</span>.<span style="color:#74531f;">ToString</span>(),&nbsp;<span style="color:#2b91af;">Encoding</span>.<span style="font-weight:bold;">UTF8</span>));
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;<span style="color:#74531f;">Initialize</span>(<span style="color:#2b91af;">GeneratorInitializationContext</span>&nbsp;<span style="color:#1f377f;">context</span>)
&nbsp;&nbsp;&nbsp;&nbsp;{
 
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

### Source Generatorlar Nasıl Test Edilir

Source generatorları test etmek için varolan bir proje içerisine analyzer veya project reference olarak eklemek yeterli. 

Analyzer olarak eklemek istersek...

<pre style="font-family:Consolas;font-size:19px;color:black;background:white;"><span style="color:blue;">&lt;</span><span style="color:#a31515;">ItemGroup</span><span style="color:blue;">&gt;</span>
<span style="color:blue;">&nbsp;&nbsp;&lt;</span><span style="color:#a31515;">Analyzer</span><span style="color:blue;">&nbsp;</span><span style="color:red;">Include</span><span style="color:blue;">=</span>&quot;<span style="color:blue;">C:\Users\ilkay\source\repos\SourceGeneratorPOC\SourceGeneratorPOC\bin\Debug\netstandard2.0\SourceGeneratorPOC.dll</span>&quot;<span style="color:blue;">&nbsp;/&gt;</span>
<span style="color:blue;">&lt;/</span><span style="color:#a31515;">ItemGroup</span><span style="color:blue;">&gt;</span></pre>

Project reference olarak eklemek istersek...

<pre style="font-family:Consolas;font-size:19px;color:black;background:white;"><span style="color:blue;">&lt;</span><span style="color:#a31515;">ItemGroup</span><span style="color:blue;">&gt;</span>
<span style="color:blue;">&nbsp;&nbsp;&lt;</span><span style="color:#a31515;">ProjectReference</span><span style="color:blue;">&nbsp;</span><span style="color:red;">Include</span><span style="color:blue;">=</span>&quot;<span style="color:blue;">..\SourceGeneratorPOC\SourceGeneratorPOC.csproj</span>&quot;<span style="color:blue;">&nbsp;</span>
<span style="color:blue;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;</span><span style="color:red;">OutputItemType</span><span style="color:blue;">=</span>&quot;<span style="color:blue;">Analyzer</span>&quot;<span style="color:blue;">&nbsp;/&gt;</span>
<span style="color:blue;">&lt;/</span><span style="color:#a31515;">ItemGroup</span><span style="color:blue;">&gt;</span></pre>

Source generatorı projeye ekledikten sonra doğrudan source generatorın yarattığı metodu çağırabiliriz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Program</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;<span style="font-weight:bold;color:#74531f;">Main</span>(<span style="color:blue;">string</span>[]&nbsp;<span style="color:#1f377f;">args</span>)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HelloWorldGenerated.<span style="font-weight:bold;color:#2b91af;">HelloWorld</span>.<span style="font-weight:bold;color:#74531f;">SayHello</span>();
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

Gördüğünüz üzere kod yazan kod yazmak oldukça kolay :). Ama şimdi gelin biraz daha kompleks biraz daha gerçek hayata uygun bir örnek yapalım. Diyelim ki uygulamanız ayağa kalkarken runtimeda belirli bir interface'i implemente eden tipleri bulsun ve bunları otomatik olarak çalıştırsın. Bunu reflectionla aşağıdaki gibi basitçe yazabiliriz.

<pre style="font-family:Consolas;font-size:19px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;<span style="color:#1f377f;">types</span>&nbsp;=&nbsp;<span style="color:blue;">typeof</span>(<span style="color:#2b91af;">Program</span>).Assembly.<span style="color:#74531f;">GetTypes</span>().<span style="color:#74531f;">Where</span>(<span style="color:#1f377f;">t</span>&nbsp;=&gt;&nbsp;<span style="color:#1f377f;">t</span>.<span style="color:#74531f;">IsAssignableTo</span>(<span style="color:blue;">typeof</span>(<span style="color:#2b91af;">IStartup</span>)));
<span style="color:#8f08c4;">foreach</span>&nbsp;(<span style="color:blue;">var</span>&nbsp;<span style="color:#1f377f;">type</span>&nbsp;<span style="color:#8f08c4;">in</span>&nbsp;<span style="color:#1f377f;">types</span>)
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;<span style="color:#1f377f;">startup</span>&nbsp;=&nbsp;(<span style="color:#2b91af;">IStartup</span>)<span style="font-weight:bold;color:#2b91af;">Activator</span>.<span style="font-weight:bold;color:#74531f;">CreateInstance</span>(<span style="color:#1f377f;">type</span>);
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#1f377f;">startup</span>.<span style="color:#74531f;">Execute</span>();
}</pre>

Bu kod reflectionla çalışacağı için performans olarak bize aslında bir dezavantaj getirecektir. Halbuki baktığımızda bir kodu derleme zamanında bu interface'i implemente eden tipleri bulabilir ve otomatik olarak kod yazarak reflection aşamasından kurtulabiliriz. Şimdi gelin bunu basitçe nasıl yaparız ona bakalım. 

İlk olarak bir syntax receiver yazarak sadece class declarationları filtreleyelim. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">StartupReceiver</span>&nbsp;:&nbsp;<span style="color:#2b91af;">ISyntaxReceiver</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:#2b91af;">ClassDeclarationSyntax</span>&gt;&nbsp;Candidates&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;}&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:#2b91af;">ClassDeclarationSyntax</span>&gt;();
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;<span style="color:#74531f;">OnVisitSyntaxNode</span>(<span style="color:#2b91af;">SyntaxNode</span>&nbsp;<span style="color:#1f377f;">syntaxNode</span>)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#8f08c4;">if</span>&nbsp;(<span style="color:#1f377f;">syntaxNode</span>&nbsp;<span style="color:blue;">is</span>&nbsp;<span style="color:#2b91af;">ClassDeclarationSyntax</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Candidates.<span style="color:#74531f;">Add</span>(<span style="color:#1f377f;">syntaxNode</span>&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">ClassDeclarationSyntax</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

Bu kod ile sadece class tanımlamaları ile ilgileneceğimizi belirledik ve `Execute` metodu içerisinde bu tanımlamalar üzerinden analizimizi yapacağız. Source generatorın kodu işe şu şekilde. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;">[<span style="color:#2b91af;">Generator</span>]
<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">TestGenerator</span>&nbsp;:&nbsp;<span style="color:#2b91af;">ISourceGenerator</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;<span style="color:#74531f;">Execute</span>(<span style="color:#2b91af;">GeneratorExecutionContext</span>&nbsp;<span style="color:#1f377f;">context</span>)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;<span style="color:#1f377f;">builder</span>&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">StringBuilder</span>();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#1f377f;">builder</span>.<span style="color:#74531f;">AppendLine</span>(<span style="color:maroon;">@&quot;namespace&nbsp;StartupExtensions
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;static&nbsp;class&nbsp;StartupRunner
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;static&nbsp;void&nbsp;Run()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&quot;</span>);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;<span style="color:#1f377f;">interfaceSymbol</span>&nbsp;=&nbsp;<span style="color:#1f377f;">context</span>.Compilation.<span style="color:#74531f;">GetTypeByMetadataName</span>(<span style="color:blue;">typeof</span>(<span style="color:#2b91af;">IStartup</span>).FullName);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#8f08c4;">if</span>&nbsp;(<span style="color:#1f377f;">context</span>.SyntaxReceiver&nbsp;<span style="color:blue;">is</span>&nbsp;<span style="color:#2b91af;">StartupReceiver</span>&nbsp;<span style="color:#1f377f;">startupReceiver</span>)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#8f08c4;">foreach</span>&nbsp;(<span style="color:blue;">var</span>&nbsp;<span style="color:#1f377f;">classDeclaration</span>&nbsp;<span style="color:#8f08c4;">in</span>&nbsp;<span style="color:#1f377f;">startupReceiver</span>.Candidates)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;<span style="color:#1f377f;">model</span>&nbsp;=&nbsp;<span style="color:#1f377f;">context</span>.Compilation.<span style="color:#74531f;">GetSemanticModel</span>(<span style="color:#1f377f;">classDeclaration</span>.SyntaxTree);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;<span style="color:#1f377f;">symbolInfo</span>&nbsp;=&nbsp;<span style="color:#1f377f;">model</span>.<span style="color:#74531f;">GetDeclaredSymbol</span>(<span style="color:#1f377f;">classDeclaration</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#8f08c4;">if</span>&nbsp;(<span style="color:#1f377f;">symbolInfo</span>&nbsp;<span style="color:blue;">is</span>&nbsp;<span style="color:#2b91af;">ITypeSymbol</span>&nbsp;<span style="color:#1f377f;">typeSymbol</span>&nbsp;&amp;&amp;&nbsp;<span style="color:#1f377f;">typeSymbol</span>.AllInterfaces.<span style="color:#74531f;">Any</span>(<span style="color:#1f377f;">t</span>&nbsp;=&gt;&nbsp;<span style="color:#1f377f;">t</span>.<span style="color:#74531f;">Equals</span>(<span style="color:#1f377f;">interfaceSymbol</span>,&nbsp;<span style="color:#2b91af;">SymbolEqualityComparer</span>.<span style="font-weight:bold;">Default</span>)))
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;<span style="color:#1f377f;">variableName</span>&nbsp;=&nbsp;<span style="color:#1f377f;">classDeclaration</span>.Identifier.Text.<span style="color:#74531f;">ToLowerInvariant</span>();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#1f377f;">builder</span>.<span style="color:#74531f;">AppendLine</span>(<span style="color:#a31515;">$&quot;</span>{<span style="font-weight:bold;">indent</span>}<span style="color:#a31515;">var&nbsp;</span>{<span style="color:#1f377f;">variableName</span>}<span style="color:#a31515;">=&nbsp;new&nbsp;</span>{<span style="color:#1f377f;">symbolInfo</span>.ContainingNamespace.Name}<span style="color:#a31515;">.</span>{<span style="color:#1f377f;">symbolInfo</span>.Name}<span style="color:#a31515;">();&quot;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#1f377f;">builder</span>.<span style="color:#74531f;">AppendLine</span>(<span style="color:#a31515;">$&quot;</span>{<span style="font-weight:bold;">indent</span>}{<span style="color:#1f377f;">variableName</span>}<span style="color:#a31515;">.Execute();&quot;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#1f377f;">builder</span>.<span style="color:#74531f;">AppendLine</span>(<span style="color:maroon;">@&quot;}}}&quot;</span>);
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#1f377f;">context</span>.<span style="color:#74531f;">AddSource</span>(<span style="color:#a31515;">&quot;startupRunner.cs&quot;</span>,&nbsp;<span style="color:#2b91af;">SourceText</span>.<span style="font-weight:bold;color:#74531f;">From</span>(<span style="color:#1f377f;">builder</span>.<span style="color:#74531f;">ToString</span>(),&nbsp;<span style="color:#2b91af;">Encoding</span>.<span style="font-weight:bold;">UTF8</span>));
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;<span style="color:#74531f;">Initialize</span>(<span style="color:#2b91af;">GeneratorInitializationContext</span>&nbsp;<span style="color:#1f377f;">context</span>)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#1f377f;">context</span>.<span style="color:#74531f;">RegisterForSyntaxNotifications</span>(()&nbsp;=&gt;&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">StartupReceiver</span>());
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

Yukarıdaki kodda ilk olarak `Initialize` modunda yazmış olduğumuz `StartupReceiver` tipini source generatora bildirdik. Artık syntax değişiklikleri olduğunda bu receiver içerisinde gerekli filtrelemeleri yapabileceğiz. 

Sonrasında ise ilk olarak `StringBuilder` içerisine yaratacağımız kodun giriş bölümünü ekledik. Ardından `StartupReceiver` içerisinde filtrelediğimiz class declerationların sembol bilgilerini alıp istediğimiz interface'i implemente edip etmediklerini kontrol edip eğer ediyorsa basit bir kodla bu tipi yaratıp `Execute` metodunu çağıracak kodu yazdık. En son olarak ise yarattığımız kodu compilera bildirdik.

Test edeceğimiz yerde kodu derledikten sonra basitçe yaratılan metodu çağırırsak reflectionla yaptığımız işi artık derleme zamanında yapmış olacağız. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Program</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;<span style="font-weight:bold;color:#74531f;">Main</span>(<span style="color:blue;">string</span>[]&nbsp;<span style="color:#1f377f;">args</span>)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="font-weight:bold;color:#2b91af;">StartupRunner</span>.<span style="font-weight:bold;color:#74531f;">Run</span>();
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

F12 kullanarak yaratılan koda baktığımızda...

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">namespace</span>&nbsp;StartupExtensions
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="font-weight:bold;color:#2b91af;">StartupRunner</span>
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;<span style="font-weight:bold;color:#74531f;">Run</span>()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;<span style="color:#1f377f;">type1</span>&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;SourceGeneratorTest.<span style="color:#2b91af;">Type1</span>();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#1f377f;">type1</span>.<span style="color:#74531f;">Execute</span>();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;<span style="color:#1f377f;">type3</span>&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;SourceGeneratorTest.<span style="color:#2b91af;">Type3</span>();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#1f377f;">type3</span>.<span style="color:#74531f;">Execute</span>();
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

Gördüğünüz gibi çalışma zamanında yapılan bir operasyonu source generator kullanarak derleme zamanında gerçekleştirdik. Böylece hem reflectiondan kurtulmuş olduk hem de reflectionın getireceği performans dezavantajından kurtulmuş olduk. 

Yukarıdaki kodlara baktıysanız anlamadığınız pek çok kavram karşınıza çıkabilir. Source generator geliştirmek tıpkı analyzer/code fix gelistirmek gibi roslyn API'larıyla oldukça içli dışlı olmanızı gerektiriyor. Aynı zamanda debugging'in de çok başarılı olduğunu söylemem mümkün değil. İnternette araştırdığım bazı debugging yöntemlerini malesef başaramadım. Çoğu zaman yarattığım koda yorum satırı ekleyerek debugging yaptım. Ama ileriki Visual Studio versiyonlarında bu deneyimler gelişecektir. 

Source generatorlarla ilgili aklımıza gelen konulardan biri de varolan kodu değiştirip değiştiremeyeceğimiz. Source generatorlar bize sadece kod yaratma imkanı veriyor. Ayrıca partial class veya metotlar kullanarak varolan tipler içerisine bir şeyler eklemeniz mümkün. Aynı zamanda extension metotlar da yaratabilirsiniz source generatorlar ile. Nasıl implemente edeceğiniz aslında tamamen size kalmış. 

Source generatorlar ile kod yaratma aşaması da bana kalırsa biraz sıkıntılı. Özellikle yaratılan kodda indentation düzgün olsun istiyorsanız biraz zaman harcamanız lazım. Ben yukarıda buna çok dikkat etmedim :) Source generatorlar şu an oldukça yeni olsalarda zamanla kullanımları oldukça artacaktır. Şu anda .NET ekibi pek çok ürün içerisinde source generator geliştirmeye başladı. Bu generatorlar ile .NET platformunun performansı daha  da artacaktır diye düşünüyorum. 

Bir sonraki yazıda görüşmek üzere,

Yazıda bahsettiğim örneği GitHub üzerinden incelemek isterseniz : <a href="https://github.com/ilkayilknur/SourceGeneratorPOC" target="_blank">https://github.com/ilkayilknur/SourceGeneratorPOC</a>
