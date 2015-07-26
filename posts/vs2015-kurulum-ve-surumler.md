# Visual Studio 2015 Sürümleri ve Kurulum Yenilikleri #

Zaman hızla akıp geçiyor valla :) Bundan 1.5 yıl önce Visual Studio 14 kod ismiyle ilk CTP'si yayınlanan Visual Studio'nun yeni versiyonu, Visual Studio 2015 ismiyle geçtiğimiz günlerde RTM(Release to Manufacturing) oldu. Yani bunun anlamı, artık rahat rahat development yaptığımız bilgisayarlarımıza kurabiliriz ve productionda kullanabiliriz.

Kurulum festivaline başlamadan önce isterseniz Visual Studio 2015'in sürümlerine bir göz atalım ve kurulum esnasında karşımıza ne gibi yenilikler çıkıyor kısaca bakalım. 

## Visual Studio 2015 Sürümleri ##

Kuruluma başlamadan önce en kritik soru tabiki de *"Hangi sürümü indirip kurmalıyım ?"* sorusu. Visual Studio 2015 ile beraber sürüm isimlerinde de bazı değişiklikler var. Visual Studio 2013'de Community,Professional,Premium ve Ultimate sürümleri bulunurken Visual Studio 2015'in Community,Professional ve Enterprise sürümleri bulunmakta. Toparlayacak olursak Community ve Professinal sürümleri aynı şekilde devam ediyor ancak Visual Studio 2013'deki **Premium ve Ultimate** sürümleri **Enterprise** sürümü adı altında birleştirildi. **Community sürümü Visual Studio 2015'de de ücretsiz.** Professional ve Ultimate için lisans almanız veya MSDN üyeliğinizin olması gerekiyor. Ürün fiyatları için [şuradaki](https://www.visualstudio.com/vs-2015-product-editions) linke bakabilirsiniz.

![](http://az718566.vo.msecnd.net/uploads/2015/07/vs15versions.png) 

Peki Community,Professional ve Enterprise sürümleri arasında ne gibi farklar var diyorsanız [buradaki](https://www.visualstudio.com/en-us/products/compare-visual-studio-2015-products-vs.aspx) link size yardımcı olacaktır.

## Kurulumda Ne Gibi Yenilikler Var ? ##

Yukarıdaki bilgiler ışığında bilgisayarımıza kuracağımız Visual Studio 2015 sürümünü seçtik ve artık kuruluma hazırız.

Visual Studio'nun kurulumu aslında her zaman basit olmuştur. Install'a tıkla sonra next, next, next :) Ancak en büyük problemlerden biri de kurulumda aslında hiç işimize yaramayacak olan componentlerinde bilgisayarımıza yüklenmesi ve bunu önleyemememizdi. Hatta [Uservoice](http://visualstudio.uservoice.com/forums/121579-visual-studio/suggestions/2207833-make-the-visual-studio-installer-more-customizable)'te en çok oy alan isteklerden biriydi bu. Microsoft'da Visual Studio 2015'de bu isteği göz ardı etmemiş ve kurulum sırasında artık bize daha uzun bir component listesi çıkarıyor ve biz arasından aslında hiç ihtiyacımız olmayacak olan componentleri kurmadan geçebiliyoruz. Tabi bunu yapmak için kurulum sırasında **Custom** kurulumu seçmemiz gerekiyor. **Typical'ı** seçersek web ve masaüstü uygulamaları geliştirmek için gerekli olan componentler otomatik olarak yükleniyor. 

![](http://az718566.vo.msecnd.net/uploads/2015/07/vs2015_setup1.jpg)

![](http://az718566.vo.msecnd.net/uploads/2015/07/vs2015_setup2.jpg)

Custom kurulumu seçtikten sonra karşımıza yukarıdaki gibi bir liste geliyor. Burada dikkatimizi çekmesi gereken bazı seçenekler var :)

- Xamarin
- Apache Cordova
- Microsoft Visual Studio Emulator for Android
- Git for Windows
- Github Extesion for Visual Studio

Bunlar Visual Studio 2015 ile beraber kurulumda ilk kez karşılaştığımız componentler. Daha önce bunların hepsini ayrı ayrı extension vs.. olarak kurmamız gerekirken artık Visual Studio 2015 kurulumu ile beraber bilgisayarımıza tek bir adımda yükleyebiliyoruz. Android developmentı için gerekli olan Java kurulumuda yine bu kurulum içerisinde gerçekleştiriliyor.

Visual Studio 2015 kurulum aşaması bu kadar. Bir sonraki adım için yeni yazıda görüşmek üzere :)