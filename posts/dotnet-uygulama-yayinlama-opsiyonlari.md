Biz yazılımcılar için çalışan kodu yazmak kadar uygulamayı doğru ve sağlıklı bir şekilde yayına almak da oldukça önemli. Bildiğiniz üzere .NET Core ile beraber artık uygulamalarımızı cross-platform çalıştırabiliyoruz. Belki çoğu zaman sadece `dotnet publish` komutunun en yalın halini bazen de Visual Studio üzerinden `publish` ekranını kullanarak hızlı bir şekilde uygulamaları yayına alıyor olabiliriz. Aslında baktığımızda buralarda kullanabileceğimiz bir çok opsiyon bulunmakta. Bu yazımızda da biraz bu opsiyonları inceleyeceğiz.

.NET uygulamalarını yayınlama opsiyonlarımıza baktığımızda karşımıza iki farklı opsiyon çıkmakta.

Bunlar,

* Framework bağımlı olarak uygulama yayınlama
* Self-contained olarak uygulama yayınlama

### Framework bağımlı olarak uygulama yayınlama 
Framework bağımlı olarak uygulama yayınlama aslında uygulamanın bulunacağı ortamda .NET runtime'ının kurulu olduğu varsayımıyla yaptığımız yayınlama. Bu nedenle sadece uygulamamızın çalışması için gereken çıktıları yayınlamamız yeterli. 

Bir uygulamayı framework bağımlı olarak publish etmek istediğimizde aşağıdaki `dotnet publish` commandini çalıştırabiliriz. 

`dotnet publish --self-contained false`

Bu şekilde çalıştırdığımızda .dll uzantılı ve cross-platfom çalışacabilecek çıktılar üretilir.

Eğer belirli bir platform üzerinde çalışacak executable çıktı da üretmek isterseniz yukarıdaki commande ek olarak bir runtime identifier eklemeniz gerekli.

`dotnet publish --self-contained false -r <RID>`

* -r : Runtime identifier. Uygulamanın hangi platformda çalışacağını belirtir. <a href="https://docs.microsoft.com/en-us/dotnet/core/rid-catalog" target="_blank">Buradan</a> opsiyonları görebilirsiniz. 

Örnek verirsek,

`dotnet publish Blog.Web.csproj -r win-x64 -c Release --self-contained false`

Yukaridaki commandi kullanarak bir proje yayınladığımızı düşünelim. Çıktı olarak cross-platform olarak `dotnet Blog.Web.dll` komutuyla çalıştırabileceğimiz bir .dll bir de runtime identifier olarak windows belirttiğimiz için `Blog.Web.exe` dosyası üretilecek.

Bu yöntemin en önemli avantajı uygulamanın çalışacağı yerde runtime yüklü olduğu için sadece uygulama için gerekli olan dosyaların deploy edilmesi nedeniyle daha ufak deployment paketlerinin olması. Uygulamanız aynı zamanda runtime updatelerinden, güvenlik güncellemelerinden de otomatik olarak yararlanabiliyor. 

Bu yöntemin sıkıntı oluşturabilecek yanlarından biri runtime'ın uygulamanın çalışacağı yerde kurulu olması gerekliliği. Bu bazı durumlarda sıkıntılı olabilir. Örneğin, bazı .NET versiyonları release olur olmaz Azure'daki servislerden bulunmayabiliyor. Böyle durumlarda en yeni versiyonu hedefleyen uygulamaları deploy etmeniz mümkün olmayabilir. Bu gibi durumlarda da self-contained şekilde uygulamalarımızı yayınlayabiliyoruz. Şimdi gelin bir de o tarafı inceleyelim. 

### Self-contained olarak uygulama yayınlama

Self-contained yayınlama opsiyonunun en avantajlı noktası uygulamalarımızı runtime'ın yüklü olmadığı yerlerde de çalıştırabilmemiz. Self-contained olarak yayınlanan uygulama paketinde uygulamanın çalışması için gerekli olan çıktıların yanı sıra runtime da bulunuyor. Bu şekilde uygulama kendisini çalıştıracak olan runtime'ı yanında getirmiş oluyor. Uygulamayı self-contained yayınlamak için kullanabileceğimiz komut şu.

`dotnet publish -r <RID>`

Uygulamamızı self-contained olarak yayınladığımızda uygulamamız artık cross-platform çalışamamakta. Aynı zamanda uygulama paketinde artık runtime da bulunduğu için uygulama paketinin büyüklüğü incelediğimiz ilk opsiyona göre oldukça büyük olmakta. 

Bu opsiyonun bir diğer sıkıntısı ise uygulamanın çalışacağı runtime'ı siz kontrol ettiğiniz için runtime'a gelecek olan ufak updatelerden, güvenlik güncellemelerinden yararlanamıyor oluşunuz. Bu nedenle bir güncelleme çıktığında güncellenen runtime'ı içerisinde bulunduran yeni bir uygulama paketi yaratmanız ve deploy etmeniz gerekmekte. Uygulamanızın çalışacağı runtime üzerinde tam kontrolünüz olmasını istiyorsanız bu sizin için istenilen bir özellik olabilir.

### Single-file Yayınlama

Uygulamalarımızı yayınlama opsiyonlarında yukarıdakilere ek olarak single file yayınlama opsiyonunu kullanabiliriz. Bu opsiyonlarla beraber uygulamalarımız belirttiğimiz platforma özel tek bir executable dosya olarak yayınlanmakta. 

Single-file yayınlama opsiyonunu kullanmak isterseniz dotnet publish komutuna `-p:PublishSingleFile=true` parametresini ekleyebilirsiniz. 

`dotnet publish -r <RID> -p:PublishSingleFile=true`

### App Trimming

Single-file yayınlama opsiyonu hem framework bağımlı olma hem de self-contained uygulama paketi yaratma senaryolarında geçerli. Self-contained durumda tabi ki yaratılan dosyanın büyüklüğü diğer opsiyona göre daha büyük olmakta. Bu nedenle .NET Core 3.1 ve .NET 5.0 içerisinde hala daha preview olan bir özellik bulunmakta. Bu özellik sayesinde self-contained uygulama paketi yaratma aşamasında uygulamanızın kullanmadığı assemblyler paketten çıkarılıyor. Böylece self-contained paketin boyutunun ufaltılması amaçlanıyor. Bu özelliği kullanabilmek için dotnet publish komutuna `/p:PublishTrimmed=true` parametresini geçmemiz yeterli.

Trimming özelliği .NET 5.0 ile beraber biraz daha gelişiyor ve sadece kullanılmayan assemblyler değil aynı zamanda assembly içerisinde kullanılmayan tipler de silinebiliyor. Bu uygulama şu an için experimental olarak .NET 5.0 içerisinde bulunmakta. Bunu denemek için de yukarıdaki parametreye ek olarak `/p:TrimMode=Link` parametresini geçebilirsiniz. Bu özellik kodun build edilmesi esnasında yapılan analize dayanarak gerçekleştirilmekte. Ancak özellikle reflection gibi senaryolarda kullanılıp/kullanılmayan tiplerin belirlenmesi biraz sıkıntılı bir durum. Bu yüzden bu özelliğin .NET 6.0 gibi daha olgunlaşacağını düşünebiliriz.

Bir sonraki yazıda görüşmek üzere,

Kaynak: <a href="https://docs.microsoft.com/en-us/dotnet/core/deploying/" target="_blank">https://docs.microsoft.com/en-us/dotnet/core/deploying/</a>