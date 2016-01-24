#Typescript Development Araçları#

Bir önceki yazımızda "[Typescript Nedir ?](http://ilkayilknur.com/typescript-nedir)" konusuna değinmiştik. O yazıda örneklerimizi online bir araç olan [Typescript Playground](http://www.typescriptlang.org/Playground)'u kullanarak gerçekleştirmiştik. Bu yazımızda Typescript ile uygulama geliştirmek istediğimizde bilgisayarımızda kullanabileceğimiz araçları inceleyeceğiz. Zaman kaybetmeden ilkiyle başlayalım. 

## Visual Studio 2015 & 2013 ##

Visual Studio kullanarak Typescript ile uygulamalar geliştirebiliyoruz. Visual Studio 2013 için bir [extension](https://visualstudiogallery.msdn.microsoft.com/77b22d98-4345-41ef-a98f-ac25563b7ec5) yüklemek gerekirken Visual Studio 2015 ile bu extension default olarak kurulu geliyor. Ayrıca ücretsiz olarak kullanabileceğiniz [Visual Studio  Community Edition](https://www.visualstudio.com/tr-tr/products/visual-studio-community-vs.aspx) ile de Typescript development yapmak mümkün.

Yukarıda bahsettiğim Visual Studio sürümlerinden herhangi birini bilgisayarınıza kurduğunuzda yeni bir project yaratmak istediğinizde diğer diller seçeneğinde artık Typescript'i de görüyor olacaksınız. 

![](http://az718566.vo.msecnd.net/uploads/2016/01/VSTypescript.png)

"*HTML Application with Typescript*" proje tipi üzerinden yeni bir proje yaratırsanız doğrudan Typescript'in sağladığı tüm yeteneklere ve özelliklere erişiminiz olacaktır.

![](http://az718566.vo.msecnd.net/uploads/2016/01/VSTypescriptIntellisense.PNG)

Eğer hali hazırda bir javascript kodunuz varsa solution içerisine bir Typescript dosyası eklediğinizde **Visual Studio** projenize otomatik olarak Typescript desteği ekleyecektir.


### NPM ile Typescript Compilerını Yükleme ###

Bundan sonra bahsedeceğimiz Visual Studio haricindeki alternatifleri kullanmadan önce bilgisayarımıza NPM üzerinden Typescript compilerını kurmamız gerekiyor. Çünkü Visual Studio bu kurulumları otomatik olarak yaparken diğer alternatifler için bizim kurulum işlemini manuel olarak yapmamız gerekiyor. Eğer bilgisayarınızda NPM yok ise öncelikle [https://nodejs.org/](https://nodejs.org/) üzerinden bilgisayarımıza uygun nodejs sürümünü yüklememiz gerekiyor. Sonrasında da aşağıdaki commandi kullanarak Typescript compilerını bilgisayarımıza yükleyebiliriz.

    npm install -g typescript

Geçtiğimiz -g parametresi Typescript'i global olarak bilgisayarımıza kurmamızı sağlıyor. Typescript compilerını bu şekilde bilgisayarımıza kurduktan sonra alternatif geliştirme araçlarını artık kullanabiliriz.

### tsconfig.json ###

Yine bundan sonra araçları kullanırken mutlaka kullanamamız gereken yapılardan biri de **tsconfig.json dosyası**. tsconfig.json dosyası Typescript compilerına derleme ile ilgili bir takım bilgileri iletmemizi sağlıyor. Derleyeceğiniz Typescript dosyalarının bulunduğu klasörün root'una tsconfig.json dosyası koyarsanız Typescript compilerı da buradaki bilgiler doğrultusunda Typescript kodlarınızı Javascript'e çevirecektir. 

tsconfig.json dosyasının içerisini kısaca incelersek, aslında karşımıza compilerOptions, files ve exclude isminde üç farklı alan çıkıyor.  **files** alanı Typescript compilerının klasör içerisinde sadece belirtilen Typescript dosyalarının derlenmesini sağlıyor. **exclude** alanı ise files'ın tam tersi olarak belirtilen dosyaların veya klasörlerin Typescript compilerı tarafından derlenmemesi gerektiğini belirtiyor. Bu iki alan da opsiyonel alanlar. Yani bu alanların ikisini de boş geçerseniz Typescript compilerı klasör içerisindeki bütün Typescript dosyalarını derleyecektir.

**compilerOption** alanı ile de derleyeciye derleme operasyonu ile ilgili direktifler verebiliyoruz. Örneğin Typescript kodlarının Ecmascript 5 versiyonuna göre çevrilmesi veya çevrilirken commentlerin silinmesi gibi pek çok farklı direktifi bu alandan compilera iletebiliyoruz. tsconfig.json dosyasında bulunan tüm alanlarla ile ilgili detaylı bilgiyi [burada](https://github.com/Microsoft/TypeScript/wiki/tsconfig.json) bulabilirsiniz.

Aşağıda örnek bir tsconfig.json dosyası bulabilirsiniz.

    {
    "compilerOptions": {
        "module": "commonjs",
        "noImplicitAny": true,
        "removeComments": true,
        "preserveConstEnums": true,
        "out": "../../built/local/tsc.js",
        "sourceMap": true
    },
    "files": [
        "core.ts",
        "sys.ts",
        "types.ts",
        "scanner.ts",
        "parser.ts",
        "utilities.ts",
        "binder.ts",
        "checker.ts",
        "emitter.ts",
        "program.ts",
        "commandLineParser.ts",
        "tsc.ts",
        "diagnosticInformationMap.generated.ts"
    ]


## Visual Studio Code ##

Windows, Mac ve Linux platformlarında kullanabileceğimiz ücretsiz alternatiflerden biri de [Visual Studio Code](https://code.visualstudio.com/).

Visual Studio Code'u yüklediğinizde .ts uzantılı dosyaları açtığınızda otomatik olarak ilgili intellisense ve development kolaylıkları size sağlanıyor. Eğer birden fazla Typescript dosyanız varsa ve bunları birbirlerine referans alıp kullanmak isterseniz öncelikle çalıştığınız klasör içerisine **tsconfig.json** dosyasını eklemeniz gerekiyor. 

![](http://az718566.vo.msecnd.net/uploads/2016/01/tsconfig.PNG)

Sonrasında ise `Shift+Alt+B'ye` basarak derleme işlemini başlatıyoruz.

![](http://az718566.vo.msecnd.net/uploads/2016/01/ConfigureRunnner.PNG)

**Configure Task Runner** seçeneğine basarak devam ediyoruz. Sonrasında klasör içerisine tasks.json isimli bir dosya ekleniyor. Burada Visual Studio Code bizim klasör içerisindeki dosyalarımızı nasıl derleyeceği ile ilgili bilgileri tutuyor. Biz Typescript kullandığımız için aşağıda gördüğünüz üzere Typescript compilerının nasıl çağırılacağı ile ilgili detaylar bu dosya içerisinde tutuluyor. Bu dosya içerisindeki args kısmını siliyoruz. Çünkü yaratılan template içerisindeki örnek olması açısından bir dosya adı eklenmiş. Biz bunu silerek devam ediyoruz. Çünkü derlenecek olan dosyaların adını zaten tsconfig.json dosyası içerisinde vermiştik.

![](http://az718566.vo.msecnd.net/uploads/2016/01/tasksjson.png)

tasks.json dosyasını kaydettikten sonra tekrar `Shift+Alt+B'ye` basıyoruz ve klasörümüz içerisindeki Typescript dosyalarını derliyoruz. Derleme işlemi bittikten sonra sol tarafta .js uzantılı dosyaların oluştuğunu göreceksiniz.

![](http://az718566.vo.msecnd.net/uploads/2016/01/vscodeend.PNG)

## Sublime Text ##

Sublime Text de yine Visual Studio Code gibi Mac, Windows ve Linux platformlarında kullanabileceğimiz alternatif araçlardan biri.

Eğer bilgisayarınızda Sublime Text kurulu değilse buradaki adresten [kurulumu](http://www.sublimetext.com/) yapıp sonra da [Package Manager](https://packagecontrol.io/installation#st2)'ı Sublime Text'e kurmanız gerekiyor. Bu kurumları yaptıktan sonra package control üzerinden Typescript extensionını Sublime Text'e yükleyebilirsiniz.

    Package Control --> Install Package --> TypeScript

Bu adımları tamamladıktan sonra `Ctrl+B` ile Typescript dosyalarınızı derleyebilirsiniz. Tabi öncelikle çalıştığınız ana klasör içerisine yukarıda bahsettiğim **tsconfig.json** dosyasını koymanız gerekiyor.

![](http://az718566.vo.msecnd.net/uploads/2016/01/TSSublime.png)

## Notepad :) ##

Son olarak bahsetmek istediğim alternatif ise Notepad :) Tabi Notepad içerisinde herhangi bir şekilde intellisense vs.. gibi güzellikleri kullanmamız mümkün değil. Ancak diyelim ki mevcut bir Typescript kodunuz var ve hızlı bir şekilde ufak bir değişiklik yapacaksınız. Herhangi bir editör vs.. gerek kalmadan değişikliği yapıp commandline üzerinden derleyip sonucunu görebilirsiniz.

Notepad'le veya herhangi bir editörle değişiklik yaptınız diyelim. Hemen tsconfig.json dosyasının bulunduğu klasörünüzün rootuna commandline'ı kullanarak gidin. Sonrasında da **tsc** commandini çalıştırın. Bu komutu çalıştırdıktan sonra Typescript compilerı klasör içerisindeki Typescript dosyalarınızı tsconfig.json'daki bilgilere göre derleyecektir.

![](http://az718566.vo.msecnd.net/uploads/2016/01/TSCCompiler.gif)

Hatta ufak bir ipucu da veriyim. Eğer tsc commandine ek olarak -w parametresini geçerseniz, üzerinde değişiklik yaptığınız dosya her değiştiğinde Typescript compilerı değişiklik yapıldığını algılar ve otomatik olarak tekrardan derleme işlemini arka planda otomatik olarak yapar.

Gördüğünüz gibi Typescript ile geliştirme yapmak için kullanabileceğiniz pek çok farklı ortam var. İster bir C# developer olun ister front-end developer olun iki taraftan da aşina olduğunuz ve günlük yaşantınızda kullandığınız araçları işletim sistemi kısıtlaması olmadan kullanarak Typescript kodu yazıp derleyebilirsiniz.

Bir sonraki yazıda görüşmek üzere...