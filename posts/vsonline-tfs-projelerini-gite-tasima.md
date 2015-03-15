Son zamanlarda source control sistemlerine baktığımızda Git'in kullanım oranının oldukça hızlı bir şekilde yükseldiğini görüyoruz. Gerek open source projelerin bir numaralı adresinin [Github](http://www.github.com "Github Link") olması gerekse Git'in yapı  ve kullanım olarak diğer source control sistemlerine göre çok daha basit olması Git'i oldukça popüler yapmakta.

Gelelim bu yazımızın konusu olan visual studio online TFS üzerinde bulunan projelerimizi Git'e history bilgisini kaybetmeden nasıl taşıyacağız konusuna. Ama ilk önce yapmamız gereken bir ayar var.

## Visual Studio Online'da Basic Authentication'ı Aktifleştirmek ##
Taşıma işlemine başlamadan önce Visual Studio Online hesabımızda basic authentication'ı aktifleştirmemiz gerekiyor. Bunun için visual studio online hesabınızla giriş yaptıktan sonra ilk olarak sağ üst köşeden profilinize girmeniz gerekiyor.

![](http://az718566.vo.msecnd.net/uploads/2015/03/VSOnline1.jpg)

Sonrasında **"Credentials"** sekmesine basıp basic authentication'ı aktifleştireceğimiz ekrana geçiyoruz.

![](http://az718566.vo.msecnd.net/uploads/2015/03/VSOnline2.jpg)

Bu ekran üzerinde ilk olarak en alttaki **"Enable alternate credentials"** linkine tıklıyoruz.

![](http://az718566.vo.msecnd.net/uploads/2015/03/VSOnline3.jpg)

Son olarak ise basic authentication ile kullanacağımız kullanıcı adı ve şifreyi belirledikten sonra kaydedip sıradaki adım olan projeyi taşıma adımına geçiyoruz.

![](http://az718566.vo.msecnd.net/uploads/2015/03/VSOnline4.jpg)

## Git-tf ##
Visual studio online TFS üzerindeki projelerimizi Git'e aktarmak için git-tf isimli aracı kullanacağız. git-tf'i [buradaki](http://www.microsoft.com/en-us/download/details.aspx?id=30474 "git-tf download link") linkten yükleyebilir yada ayarlarla, bağımlılıklar vs.. hiç uğraşmamak için [Chocolatey](https://chocolatey.org/) üzerinden aşağıdaki komutla git-tf'i bilgisayarınıza yükleyebilirsiniz.

    cinst git-tf

git-tf'i yükledikten sonra yapmamız gereken aşağıdaki komutla beraber TFS'te bulunan projeyi bir git reposu gibi bilgisayarımıza çekmek. git-tf burada devreye girip tfs projenizin checkin historysini git commitlerine çevirerek lokalimize bir git reposu gibi indirilmesini sağlıyor.

	git-tf clone https://hesapadi.visualstudio.com/DefaultCollection $/ProjeAdi/BranchAdi --deep

Bu komut sırasında git-tf bize tfs'in credential bilgilerini soracak. Username ve password olarak yazının başında basic authentication'ı aktifleştirdiğimiz bölümde vermiş olduğumuz username ve password'ü girmeliyiz. 

git-tf clone'lama işini bitirdikten sonra projeyi artık lokalden istediğimiz yere pushlayabiliriz. Bu işlemi yapmadan önce ilk olarak TFS ile ilgili dosyaları proje içerisinden silmemiz gerekiyorki sonra ortalık karışmasın :) Bu nedenle git-tf'in projemizi download ettiğimiz klasör içerisindeki **vspscc** uzantılı dosyaları siliyoruz. Sonrasında .sln uzantılı solution dosyasını da bir text editör ile açıp içerisindeki **GlobalSection(TeamFoundationVersionControl)** ile başlayan sectionı siliyoruz. 

Son olarak projeye bir [.gitignore](https://github.com/github/gitignore/blob/master/VisualStudio.gitignore) dosyası da ekleyerek projemizi git'e göndermeye hazır hale getiriyoruz.

Tüm geçiş sürecini bitirdik. Şimdi sıra geldi yaptığımız değişiklikleri commitleyip, bilgisayarımızdaki repoyu remote'a göndermeye.

	git commit -m "dosyalar tfsten gite taşındı."
	git remote add origin https://github.com/ilkayilknur/repo.git
	git push origin master

Herşey bu kadar basit. Ancak git-tf ile projeyi clone'lama sürecinin biraz uzun sürdüğünü belirtmemde fayda var. Onun haricinde de zaten git kullanan birisi için yapılan çok da farklı birşey yok.