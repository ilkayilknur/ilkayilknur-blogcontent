# Visual Studio VNext - Visual Studio 15 Preview Yenilikleri #

Bu seneki Build konferansının en önemli duyurularından biri de Visual Studio'nun bir sonraki major versiyonu olacak olan Visual Studio 15 Preview'dı. Daha ilk preview olmasına rağmen içerisinde güzel yenilikleri barındıran Visual Studio 15'e bu yazımızda kısaca bir gözatacağız.

***Visual Studio 15 Preview adından da anlaşılacağı gibi henüz preview :). Dolayısıyla bilgisayarınıza yüklediğinizde eğer bir sorunla karşılaşırsanız bu sorunla ilgili Microsoft size destek vermeyecektir. Bu nedenle tavsiyem bilgisayarınızda yeni bir sanal makinada veya Azure üzerinde bir sanal makinada bu yenilikleri test etmeniz.***

## Yeni Installer ##

Bu yazıyı okuyorsanız tahmin ediyorum ki %90 ihtimalle hayatınızda en az bir kere makinanıza Visual Studio kurulumu yapmışsınızdır. Önce ufak bir setup dosyası download edilir sonra o setup dosyası arka planda **1-2 GB** büyüklüğünde olan kurulum dosyalarını download eder ve sonrasında da **45dk-1 saate**  yakın Visual Studio'nun kurulmasını bekleriz. Kurulum zamanı setup sırasında nelerin kurulmasını istediğinize göre de değişir. Bazen developerlar yeniden kurulumla uğraşmamak için belki ileride lazım olur diye tüm opsiyonları seçerler. Bazen de default seçimle ilerlerler. Visual Studio ekibindeki kullanım datalarına bakarsak default seçenekle ilerleyen developerların **%70**'i default içerisindeki tüm functionalityleri ve tüm app tiplerini bile kullanmıyor. 

Bu nedenlerden dolayı Visual Studio VNext'in en büyük temalarından biri bu kurulum aşamasını kolaylaştırmak ve hızlandırmak olacak. Hem setup dosyasının boyutunu ufaltma, hem kurulumu hızlandırma hem de Visual Studio'yu uygulama tipleri bazında modüler hale getirme şimdilik düşünülen yeniliklerden. Şimdi gelelim ilk preview'daki installer yeniliklerine. 

[http://aka.ms/vsnext](http://aka.ms/vsnext) adresine gidip sağdaki seçenekten yeni installerı download edip installer yeniliklerini test edebilirsiniz. Soldaki installer ise bildiğimiz klasik Visual Studio installerı.

![](http://az718566.vo.msecnd.net/uploads/2016/04/VSInstaller.png)

Installerı download edip çalıştırdığımızda önceki versiyonların installerlarına nazaran farklı bir ekranla karşılaşıyoruz. 

![](http://az718566.vo.msecnd.net/uploads/2016/04/vs15-installer.png)

Ekrandan da görüleceği gibi artık Visual Studio çeşitli uygulama tiplerine uygun paketlerle geliyor. Yani eğer sadece .NET development yapacaksanız .NET Development paketini kurup hızlı bir şekilde kurulumu tamamlayabilirsiniz. Eğer sadece editör olarak kullanacağım diyorsanız core editörü seçip 300 MB'lık bir download yapıp Visual Studio'yu kurabiliyorsunuz. Şu anda sadece yukarıda resimde gördüğünüz kadar paket mevcut. Ancak zaman içerisinde Universal App paketi, Cloud App paketi gibi daha pek çok farklı paket installera eklenecektir.

Paket bazılı kurulumla beraber kurulum süreleri de gözle görülür bir şekilde düşüyor. Daha önce Visual Studio kurulumuyla 1-1.5 saatimizi harcarken artık çok daha kısa bir sürede kurulumu tamamlayabiliyoruz. Örneğin sadece core editör paketini yüklemek isterseniz max 10 dk içerisinde kurulum tamamlanıyor.***(Paket downloadu yapıldığı için internet hızına göre değişiklik gösterebilir.)***

## Hızlandırılmış İlk Açılım ##

1-1.5 saat bekleyip Visual Studio kurulumunu tamamladıktan sonra ilk açtığımızda kullanacağımız dile göre seçim yaparız ve bir süre Visual Studio'nun ilk kullanıma hazırlanmasını bekleriz. Her ne kadar uyarıda bir kaç dakika sürebilir dese de bu süreç oldukça uzun sürebilmekte. Benim de şahsen Visual Studio kurulumunda en sinir  olduğum şeylerden biridir bu :) 

![](http://az718566.vo.msecnd.net/uploads/2016/04/vs15-preparing-screen.png)

Visual Studio 15 ile beraber artık bu süre de oldukça kısalıyor ve 1 dakikayı bile bulmadan Visual Studio ilk kez bile olsa hızlıca açılıyor. Şahsen en sevdiğim özelliklerden biride bu Visual Studio 15 içerisindeki :D

## Klasör Bazlı Çalışabilme ##

Yeni Visual Studio'nun en önemli özelliklerinden biri de klasör bazlı çalışabilme. Bundan önce Visual Studio'da bir projeyi açmak istediğimizde o projenin illa ki bir solution dosyası olması gerekiyordu. Ancak yeni Visual Studio ile beraber bu zorunluluk ortadan kalktı. Artık solution dosyası olmadan da doğrudan klasör bazlı olarak projeyi açıp çalışabiliyoruz. Böylece python, javascript gibi solution dosyası gerektirmeyen proje tipleriyle Visual Studio içerisinde çalışmak daha da kolay.

![](http://az718566.vo.msecnd.net/uploads/2016/04/vs-open-folder.png)

Solution Explorer içerisinde klasör görünümü

![](http://az718566.vo.msecnd.net/uploads/2016/04/solution-explorer-folder-view.png)


## Code Style Rules ##

Roslyn projesiyle beraber custom analyzer yazabilmeye başlamamızla en çok sorulan sorulardan biri de proje içerisindeki code standartlarını korumak için bir analyzer yapabilmenin mümkün olup olmadıydı. Bununla ilgili 1-2 proje olsa da tam anlamıyla çözülemeyen bir sorundu bu. Artık Visual Studio 15'le beraber kodlama standartlarına uygun kurallar tanımlayabiliyoruz ve bu kuralların kontrolü projenin derleme aşamasında yapılıyor. Eğer kurala uygun olmayan kullanımlar varsa bunların fixleri için de Visual Studio size ilgili code fixi otomatik olarak sunuyor. 

Code style kuralları şu anda **Tools => Options** ekranı içerisinde **Text Editor => C#** altında yer alıyor. Bu ekran içerisinde pek çok kural ayarlamak mümkün. Örneğin, projeler içerisinde this ve var keywordlerinin kullanımlarını belirleyebiliyorsunuz. Developerlar var keywordünü hiç kullanmamalı yada tipin belirgin olduğu durumlarda var keywordu kullanılabilir gibi kuralları ayarlayabiliyorsunuz.

![](http://az718566.vo.msecnd.net/uploads/2016/04/code-style-general.png)

Eğer isimlendirmelerle ilgili kurallar belirlemek isterseniz bu kuralları da yine aynı ekran üzerinden ayarlayabiliyorsunuz. Örneğin projede metot isimlendirmelerinde pascal casing kullanılsın diye ayarlarsanız Visual Studio derleme esnasında sizi isimlendirme kurallarına uymadığınız konusunda uyarıyor ve bunu düzeltmeniz için size code fix sunuyor. 


![](http://az718566.vo.msecnd.net/uploads/2016/04/code-style-naming.png)

Kuralları belirledikten sonra Visual Studio belirlediğimiz kurallara göre build esnasında ilgili uyarıları yapıyor.

![](http://az718566.vo.msecnd.net/uploads/2016/04/code-style-warning-message.png)

**"Show Potential Fixes"**'a basarsak

![](http://az718566.vo.msecnd.net/uploads/2016/04/code-style-code-fix.png)

Code style rule ekranı içerisinde yapılabilecek pek çok farklı ayar bulunmakta. Bununla ilgili daha detaylı bir şekilde başka bir blog yazısı yazmayı planlıyorum.

## Diğer Özellikler ##

Build'deki sessionları izlerseniz Visual Studio 15 içerisinde  yukarıda bahsettiklerime ek olarak gösterilen pek çok farklı özellik olduğunu göreceksiniz. Intellisense substring matching, intellisense filtering, "how do I" (en heyecanlanmadığım özellik :( ), code fix ile proje veya nuget paket referansı ekleme (en efsane özellik :) ) gibi özelliklerin hepsi private buildler üzerinden gösterildi. Bunun anlamı bu özellikle şu an için kullanıcılar tarafından test edilmek için hazır değiller. Tahmin ediyorum ki ilerleyen preview sürümlerinde bu özellikler de yavaş yavaş Visual Studio 15 içerisine eklenecektir.

Build'de Visual Studio 15 ile ilgili oturumlara aşağıda ulaşabilirsiniz.

- The Future Of Visual Studio : [https://channel9.msdn.com/Events/Build/2016/B859](https://channel9.msdn.com/Events/Build/2016/B859)
- The Future Of C# (Visual Studio 15 Yenilikleri 39. dakikada başlıyor) : [https://channel9.msdn.com/Events/Build/2016/B889](https://channel9.msdn.com/Events/Build/2016/B889)

Visual Studio 15 Preview yenilikleri şimdilik bu kadar. Yeni preview sürümleri çıktıkça içerisinde bulunan yeni özellikleri yine blogdan paylaşıyor olacağım.

Görüşmek üzere...