Blogda daha önce yazdığım [yazıda](https://www.ilkayilknur.com/source-generatorlar-ile-kod-yazan-kod-yazma) source generatorların ne olduğundan ve nerelerde kullanılabileceğinden bahsetmiştim. Source generatorlardaki en sıkıntılı noktalardan biri aslında kod üretme kısmı. Özellikle yazdığınız source generator karmaşık bir hal aldığında işler biraz karışabiliyor. Bu yazıda source generator implemente ederken kullanabileceğimiz üç farklı kod üretim yöntemini inceleyeceğiz. Lafı çok uzatmadan yöntemleri incelemeye başlayalım. 

## String Birleştirme

Source generator implemente ederken kod üretmek için kullanılan en yaygın ve akla ilk gelen yöntem string birleştirme yöntemi. Bu yöntemle aslında generate edeceğiniz kodu source generator kodu içerisinde saklamış oluyorsunuz. Örneğin, basit bir hello world örneği kodunu aşağıda gösterirsek. 

```csharp
[Generator]
public class SampleGenerator : ISourceGenerator
{
    public void Initialize(GeneratorInitializationContext context)
    {
    }

    public void Execute(GeneratorExecutionContext context)
    {
        var builder = new StringBuilder(@"
using System;
namespace HelloWorldGenerated
{
public static class HelloWorld
{
    public static void SayHello() 
    {
        Console.WriteLine(""Hello from generated code!"");
    }
}
}");

        context.AddSource("generatedCode.cs", SourceText.From(builder.ToString(), Encoding.UTF8));
    }
}
```

Yukarıda gördüğünüz gibi kodların belkide başlangıç kısmını `StringBuilder` 'a verebiliyor olsak da conditional durumlar olduğunda, farklı durumlar için farklı şekilde kod yaratmak istediğimizde işler biraz karışacaktır. Örneğin, `roslyn-sdk` reposundaki source generator kod örnekleri içerisinden bir metodu aşağıda gösterirsek. 

```csharp
public static string GenerateClassFile(string className, string csvText, CsvLoadType loadTime,
    bool cacheObjects)
{
    StringBuilder sb = new StringBuilder();
    using CsvTextFieldParser parser = new CsvTextFieldParser(new StringReader(csvText));

    //// Usings
    sb.Append(@"
#nullable enable
namespace CSV {
using System.Collections.Generic;
");
    //// Class Definition
    sb.Append($"    public class {className} {{\n");


    if (loadTime == CsvLoadType.Startup)
    {
        sb.Append(@$"
static {className}() {{ var x = All; }}
");
    }

    (string[] types, string[] names, string[]? fields) = ExtractProperties(parser);
    int minLen = Math.Min(types.Length, names.Length);

    for (int i = 0; i < minLen; i++)
    {
        sb.AppendLine(
            $"        public {types[i]} {StringToValidPropertyName(names[i])} {{ get; set;}} = default!;");
    }

    sb.Append("\n");

    //// Loading data
    sb.AppendLine($"        static IEnumerable<{className}>? _all = null;");
    sb.Append($@"
public static IEnumerable<{className}> All {{
get {{");

    if (cacheObjects)
        sb.Append(@"
    if(_all != null)
        return _all;
");
    sb.Append(@$"
    List<{className}> l = new List<{className}>();
    {className} c;
");

    // This awkwardness comes from having to pre-read one row to figure out the types of props.
    do
    {
        if (fields == null) continue;
        if (fields.Length < minLen) throw new Exception("Not enough fields in CSV file.");

        sb.AppendLine($"                c = new {className}();");
        string value = "";
        for (int i = 0; i < minLen; i++)
        {
            // Wrap strings in quotes.
            value = GetCsvFieldType(fields[i]) == "string"
                ? $"\"{fields[i].Trim().Trim(new char[] {'"'})}\""
                : fields[i];
            sb.AppendLine($"                c.{names[i]} = {value};");
        }

        sb.AppendLine("                l.Add(c);");

        fields = parser.ReadFields();
    } while (!(fields == null));

    sb.AppendLine("                _all = l;");
    sb.AppendLine("                return l;");

    // Close things (property, class, namespace)
    sb.Append("            }\n        }\n    }\n}\n");
    return sb.ToString();
}

```

Gördüğünüz üzere string birleştirme konusunda işler biraz karmaşıklaşabiliyor. Bu durumda kodların indentationını da korumak da oldukça zor olmakta. Ancak şu anda baktığımızda herhangi bir bağımlılık gerektirmeyen ve en basit çözüm olarak bu yöntem karşımıza çıkıyor. Özellikle basit ve yaratacağınız kodun çok karmaşık olmadığı durumlarda bu yöntem tercih edilebilir. 

## Roslyn API'larını Kullanarak Kod Yaratma

Bir diğer opsiyonumuz Roslyn API'larını kullanmak. Roslyn API'ları ile yaratacağımız kodun syntax treesini tanımlayarak kodun generate edilmesini tamamen roslyn API'larına bırakabiliyoruz. Baştan söylememde fayda var. Generate etmek istediğimiz kodu Roslyn API'ları ile tanımlamak çok da basit değil. Buna bir de Roslyn API'larının dökümantasyonunun çok iyi olmaması eklenince işler oldukça zorlaşıyor. Ufak bir örnek yapıp aşağıdaki kodu Roslyn API'ları ile generate etmek istersek nasıl bir kod yazacağımızı inceleyelim. 

Yaratmak istediğimiz kod.

```csharp
using System;
namespace HelloWorldGenerated
{
	public static class HelloWorld
	{
    public static void SayHello() 
    {
        Console.WriteLine(""Hello from generated code!"");
    }
	}
}
```

Yukarıdaki kodun syntax tree olarak çevrilmiş hali.

```csharp
var source = CompilationUnit()
    .WithUsings(
        SingletonList<UsingDirectiveSyntax>(
            UsingDirective(
                IdentifierName("System"))))
    .WithMembers(
        SingletonList<MemberDeclarationSyntax>(
            NamespaceDeclaration(
                    IdentifierName("HelloWorldGenerated"))
                .WithMembers(
                    SingletonList<MemberDeclarationSyntax>(
                        ClassDeclaration("HelloWorld")
                            .WithModifiers(
                                TokenList(
                                    new[]
                                    {
                                        Token(SyntaxKind.PublicKeyword),
                                        Token(SyntaxKind.StaticKeyword)
                                    }))
                            .WithMembers(
                                SingletonList<MemberDeclarationSyntax>(
                                    MethodDeclaration(
                                            PredefinedType(
                                                Token(SyntaxKind.VoidKeyword)),
                                            Identifier("SayHello"))
                                        .WithModifiers(
                                            TokenList(
                                                new[]
                                                {
                                                    Token(SyntaxKind.PublicKeyword),
                                                    Token(SyntaxKind.StaticKeyword)
                                                }))
                                        .WithBody(
                                            Block(
                                                SingletonList<StatementSyntax>(
                                                    ExpressionStatement(
                                                        InvocationExpression(
                                                                MemberAccessExpression(
                                                                    SyntaxKind.SimpleMemberAccessExpression,
                                                                    IdentifierName("Console"),
                                                                    IdentifierName("WriteLine")))
                                                            .WithArgumentList(
                                                                ArgumentList(
                                                                    SeparatedList<ArgumentSyntax>(
                                                                        new SyntaxNodeOrToken[]
                                                                        {
                                                                            Argument(
                                                                                LiteralExpression(
                                                                                    SyntaxKind
                                                                                        .StringLiteralExpression,
                                                                                    Literal(""))),
                                                                            MissingToken(SyntaxKind
                                                                                .CommaToken),
                                                                            Argument(
                                                                                IdentifierName("Hello")),
                                                                            MissingToken(SyntaxKind
                                                                                .CommaToken),
                                                                            Argument(
                                                                                QueryExpression(
                                                                                    FromClause(
                                                                                            Identifier(
                                                                                                "code"),
                                                                                            PrefixUnaryExpression(
                                                                                                SyntaxKind
                                                                                                    .LogicalNotExpression,
                                                                                                LiteralExpression(
                                                                                                    SyntaxKind
                                                                                                        .StringLiteralExpression,
                                                                                                    Literal(
                                                                                                        ""))))
                                                                                        .WithType(
                                                                                            IdentifierName(
                                                                                                "generated"))
                                                                                        .WithInKeyword(
                                                                                            MissingToken(
                                                                                                SyntaxKind
                                                                                                    .InKeyword)),
                                                                                    QueryBody(
                                                                                        SelectClause(
                                                                                                IdentifierName(
                                                                                                    MissingToken(
                                                                                                        SyntaxKind
                                                                                                            .IdentifierToken)))
                                                                                            .WithSelectKeyword(
                                                                                                MissingToken(
                                                                                                    SyntaxKind
                                                                                                        .SelectKeyword)))))
                                                                        })))))))))))))
    .NormalizeWhitespace()
    .ToFullString();
```

Burada bu kodu yukarıdaki gibi syntax tree kodu haline nasıl getirelim dediğinizi duyar gibiyim. Hem çok fazla bilgi gerekiyor hem de çok fazla kod yazmak gerekiyor. Bu aşamada benim de sıklıkla kullandığım [Roslyn Quoter](https://roslynquoter.azurewebsites.net) websitesini kullanabilirsiniz. Generate etmek istediğiniz kodu siteye yazarsanız hızlı bir şekilde syntax tree versiyonunu alabilirsiniz. Bu yöntem karmaşık senaryolarda maintain etmesi zor bir durum ortaya çıkaracaktır. Ancak basit senaryolarda, olasılıkların az olduğu durumlarda veya heyecan aradığınız durumlarda :)  kullanılabilir olduğunu düşünüyorum. 

## Scriban Templates

Son olarak bahsedeceğim yöntem ise Scriban kullanmak. Scriban open-source olarak yayınlanan bir text templating kütüphanesi. Çok karmaşık senaryolarda bile rahatlıkla kullanabileceğiniz hatta template içerisine koşullarınızı yazıp kod yaratımını bu koşullara göre yapabileceğiniz güzel bir kütüphane. Örnek olarak önceki blog yazımda yaptığım [örneği](https://github.com/ilkayilknur/SourceGeneratorPOC) scriban kullanarak yapmak istersem aşağıdaki gibi bir template yaratıp sonrasında da ilgili tanımlamaların listesini parametre olarak geçmem yeterli. 

```csharp
var template = Template.Parse(@"namespace StartupExtensions
{
    public static class StartupRunner
    {
        public static void Run()
        {
            {{ for type in types }}
            var {{ type.variableName }}= new {{ type.typeName }};
            {{ type.variableName }}.Execute();
            {{end}}
        }
    }
}");
var source = template.Render(new {Types = types});
context.AddSource("startupRunner.cs", SourceText.From(source, Encoding.UTF8));
```

Aslında işi ileri götürerek bu templateleri ayrı dosyalarda tutup dosyalardan da okuyabiliriz. Böylece bir sürü `StringBuilder` kullanımından da kurtulmuş oluruz. Yazdığımız kod daha temiz gözükür. Açıkcası karmaşıklaşan senaryolarda scriban templatelerini kullanmak bana daha mantıklı geliyor. Dünya üzerinde de kullanıldığıyla ilgili pek çok örnek gördüm.

Bu yazıda source generator implemente ederken kullanabileceğimiz kod yaratma yöntemlerini işledik. Hepsinin kendince avantajları ve dezavantajları bulunmakta. Kendi senaryonuza göre hangisini kullanacağınıza ufak implementasyonlar yaparak karar verebilirsiniz.

Bir sonraki yazıda görüşmek üzere.

