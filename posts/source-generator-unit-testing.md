Bugüne kadar source generatorlar ile ilgili pek çok konuyu blogdaki yazılarda işledik. Hatta geçtiğimiz haftalarda bu konuyla ilgili Teknolot topluluğunun düzenlediği <a href="https://www.ilkayilknur.com/teknolot-source-generatorlara-bakis-meetupi" target="_blank">meetupta</a> da detaylı olarak source generatorları anlatma fırsatı yakaladım. Gerek o meetupta gerekse blogdaki yazılarda bahsetmek isteyip ancak fırsat bulamadığım tek bir konu kaldı. O da source generatorlara nasıl unit test yazarız konusu.

Source generatorları geliştirme aşamasında Visual Studio üzerinden manuel test etmek biraz zor olabiliyor. Bu nedenle unit test yazarak unit testler üzerinden testleri yürütmek bizler için biraz daha kolay bir opsiyon olabilir. Tabi ki unit testlerin anlamı ve değeri Visual Studio üzerinden yapılan manuel testlerden çok daha farklı ancak manuel test yapmanın da zor olabildiği bir ortamda unit test yazmak çok daha mantıklı bir çözüm olarak önümüze gelebilir. 

Şimdi gelelim source generatorlar için nasıl unit test yazacağız kısmına. Örnek olması açısından ben source generator meetupında demosunu yaptığım `INotifyPropertyChanged` implementasyonunu gerçekleştiren kodu test edeceğim. Bu source generatorın benzerine <a href="https://github.com/davidwengier/SourceGeneratorPlayground/blob/main/Samples/Auto%20Notify.Generator.cs" target="_blank">buradan</a> da ulaşabilirsiniz.

Source generatorların unit testini yazmanın aslında normal unit test yazmaktan pek bir farkı yok. Derlenecek kodu veriyoruz, o kod üzerinde source generator çalışıyor ve çıktı üzerinde karşılaştırmalar yapıyoruz. 

Temel anlamda bir source generator unit test şablonu aşağıdaki gibi olmakta.

```csharp
using Microsoft.CodeAnalysis;
using Microsoft.CodeAnalysis.CSharp;
using MyTeknolotGenerator;
using NUnit.Framework;
using System.Reflection;

namespace Generator.Tests
{
    public class Tests
    {
        [Test]
        public void Test1()
        {
            //Arrange
            var code = @"
namespace Demo
{
    public partial class MyVM
    {
        [Notifier.AutoNotify]
        private string _name;

        [Notifier.AutoNotify]
        private int _clickCount;
    }
}";
            var generator = new AutoNotifyGenerator();
            var driver = CSharpGeneratorDriver.Create(generator);

            //Act
            driver.RunGeneratorsAndUpdateCompilation(CreateCompilation(code), out var updatedCompilation, out var diagnostics);

            //Assert

        }

        private static Compilation CreateCompilation(string source)
            => CSharpCompilation.Create("compilation",
                new[] { CSharpSyntaxTree.ParseText(source) },
                new[] { MetadataReference.CreateFromFile(typeof(Binder).GetTypeInfo().Assembly.Location) },
                new CSharpCompilationOptions(OutputKind.ConsoleApplication));
    }
}
```

Unit test yazarken tabi ki esas görevi `CSharpGeneratorDriver` üstleniyor. Generator instance'ı yaratıp `CSharpGeneratorDriver`'a parametre olarak geçtiğimizde `CSharpGeneratorDriver` üzerinden bir takım operasyonlar yaparak source generatorları execute ettirebiliyoruz. Bunlardan ilki `RunGeneratorsAndUpdateCompilation` metodunu kullanarak çalıştırma ve `out` parametrelerinden gelen yeni `compilation` ve `diagnostics` instancelarını elde etmek ve sonrasında bu objectler üzerinden kontroller yapmak.

```csharp
//Assert

Assert.IsTrue(diagnostics.IsEmpty);
Assert.AreEqual(3, updatedCompilation.SyntaxTrees.Count());

var generatedSyntaxTree = updatedCompilation.SyntaxTrees.Last();
var classDeclaration = generatedSyntaxTree.GetRoot().DescendantNodes()
    .OfType<ClassDeclarationSyntax>().SingleOrDefault();
Assert.AreEqual("MyVM", classDeclaration.Identifier.Text);
```

Yukarıdaki gibi dilediğiniz kontrolleri `out` parametresi üzerinden gelen objectler üzerinden yapabilirsiniz. Bir diğer opsiyon da `GetRunResult` metodunu çağırarak doğrudan result üzerinden karşılaştırmaları yapmak. Bu şekilde ilerlediğinizde oluşturulan source kodlar üzerinden de karşılaştırma yapabilirsiniz. 

```csharp
public class Tests
    {
        [Test]
        public void Test1()
        {
            //Arrange
            var code = @"
namespace Demo
{
    public partial class MyVM
    {
        [Notifier.AutoNotify]
        private string _name;

        [Notifier.AutoNotify]
        private int _clickCount;
    }
}";
            var generator = new AutoNotifyGenerator();
            GeneratorDriver driver = CSharpGeneratorDriver.Create(generator);

            //Act
            driver = driver.RunGeneratorsAndUpdateCompilation(CreateCompilation(code), out var _, out var _);
            var result = driver.GetRunResult();
            //Assert

            Assert.IsTrue(result.Diagnostics.IsEmpty);
            Assert.AreEqual(2, result.GeneratedTrees.Count());
            Assert.AreEqual(generator, result.Results.First().Generator);
            var generatedSource = result.Results.First().GeneratedSources[1].SourceText.ToString();
            var expectedCode = @"
namespace Demo
{
    using System.ComponentModel;
    public partial class MyVM : INotifyPropertyChanged
    {
        public event System.ComponentModel.PropertyChangedEventHandler PropertyChanged;
        public string name
        {
                get { return _name;}
                set { 
                     _name=value;
                    PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(name)));
                    }
        }

        public int clickCount
        {
                get { return _clickCount;}
                set { 
                     _clickCount=value;
                    PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(nameof(clickCount)));
                    }
        }
}}
";
            Assert.AreEqual(expectedCode, generatedSource);
        }

        private static Compilation CreateCompilation(string source)
            => CSharpCompilation.Create("compilation",
                new[] { CSharpSyntaxTree.ParseText(source) },
                new[] { MetadataReference.CreateFromFile(typeof(Binder).GetTypeInfo().Assembly.Location) },
                new CSharpCompilationOptions(OutputKind.ConsoleApplication));
    }
```

Burada dikkat edilmesi gereken nokta ise `GeneratorDriver`'ın immutable olması. Bu nedenle generatorların çağırılması sonrasında dönen instance'ın tekrardan `driver` variable'ına atanması gerekiyor. Böylece bir sonraki adımda `GetRunResult` metodunu çağırıp birleştirilmiş olan sonuçları alıp kontrol edebiliyoruz. Birden fazla generatorın çalışması gibi senaryoları da yine bu şekilde test edebiliriz.

Bu yazıda source generatorların unit testlerinin yazılması konusunu işledik. Source generator tarafına bulaştığınızda ve manuel test yapmanın bazı zorluklarını gördüğünüzde umarım bu yazı sizin için faydalı olur. :)

Bir sonraki yazıda görüşmek üzere.

Kaynak: <a href="https://github.com/dotnet/roslyn/blob/main/docs/features/source-generators.cookbook.md" target="_blank">https://github.com/dotnet/roslyn/blob/main/docs/features/source-generators.cookbook.md</a>

