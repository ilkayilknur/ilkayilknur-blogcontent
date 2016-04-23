# Visual Studio 2015 Update 2 İle Gelen Yenilikler #

Build 2016'nın en önemli duyurularından biri de Visual Studio 2015 Update 2'nin release olmasıydı. Update 2 içerisinden bolca bug fix barındırsada bahsetmeğe değer bazı yeni özellikler de barındırmakta. 

## Universal Platform Yenilikleri ##

Update 2 ile beraber Universal App'lerde kullandığımız **Microsoft.NETCore.UniversalWindowsPlatform** nuget paketininde 5.1.0 versiyonu release oldu. Visual Studio 2015 Update 2'yi yüklediğinizde bu nuget paketini update etmenizde fayda var.

Update 2 ile beraber gelen bir diğer özellik ise Universal App yaratma aşamasında karşımıza çıkan version seçim ekranı. Bu ekran ile uygulamanızın target edeceği ve minimum olarak destekleyeceği Windows 10 versiyonunu uygulama yaratırken seçebiliyorsunuz. 

![](http://az718566.vo.msecnd.net/uploads/2016/04/UWP-VersionPicker.png)

Update 2 ile beraber gelen en beğendiğim özellik ise basit ama zaman kazandıran bir özellik. Daha önceden Universal App'inizi store'a göndermek için paket yaratmak istediğinizde her seferinde live accountunuzla login olmak zorunda kalıyordunuz. Ancak Update 2 ile beraber bu sorunu çözmüşler ve Store'a bir kere login olduğunuzda Visual Studio artık bu account bilgilerini otomatik olarak hatırlıyor. 

![](http://az718566.vo.msecnd.net/uploads/2016/04/UWP-CreateAppPackagesDialog.png)

## Interactive Window Yenilikleri ##

Visual Studio 2015 Update 1 ile beraber gelen en önemli yeniliklerden biri de Interactive Window'du. (Eğer daha önce Interactive Window'u duymadıysanız yazmış olduğum [yazıyı](http://www.ilkayilknur.com/c-sharp-interactive-window) okuyabilirsiniz.) Update 2 ile beraber Interactive Window tarafında da yenilikler mevcut. Bu yenilikler çalışmış olduğunuz proje içerisindeki kodları hızlı bir şekilde Interactive Window üzerinden çalıştırmanıza olanak sağlıyor. Bunun için Visual Studio içerisinde çalıştırmak istediğiniz kodu seçip sağ tıklayıp "*Execute in Interactive*"'i tıklamanız yeterli. 

![](http://az718566.vo.msecnd.net/uploads/2016/04/execute-in-interactive.gif)

## Editör Yenilikleri ##

Update 2 içerisinde güzel editör yenilikleri de bulunmakta. Bunlardan ilki using/import eklerken artık fuzzy matching yapılması. Yani eğer bir tipin adını yanlış yazarsanız Visual Studio tipin adından tahminde bulunuyor ve size uygun öneriyi sunup hem tipin adını düzeltiyor hem de ilgili using/import'u ekliyor. 

![](http://az718566.vo.msecnd.net/uploads/2016/04/fuzzy-search.gif)

Update 2 içerisinde ayrıca yeni refactoringler de bulunuyor. Bunlardan ilki delegate işletimlerinde null propagator operatörü kullanımını öneren refactoring. 

![](http://az718566.vo.msecnd.net/uploads/2016/04/null-propogation-refactoring.gif)

Diğer refactoring ise make method synchronous refactoringi. 

![](http://az718566.vo.msecnd.net/uploads/2016/04/make-sync-refactoring.gif)

Bu yazıda değinmek istediğim yenilikler bunlar. Update 2 ile beraber gelen tüm yenilikleri [https://www.visualstudio.com/en-us/news/vs2015-update2-vs.aspx](https://www.visualstudio.com/en-us/news/vs2015-update2-vs.aspx) adresinde bulabilirsiniz.