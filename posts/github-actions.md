Uzun bir aradan sonra herkese selamlar,

Bu yazıda konumuz Github Actions. Github actions özet olarak Github üzerinde yazılım geliştirme akışlarını otomatize etmemizi sağlayan bir özellik. Daha detaylı bir şekilde ifade etmemiz gerekirse kodun build, test veya deploy edilmesi gibi taskları yaratabileceğimiz ve bu taskları kullarak da CI(Continuous Integration)/CD(Continuous Delivery) akışlarınızı doğrudan Github repositorysinde saklayıp çalıştıracabileceğimiz bir ortam sağlıyor bize Github Actions. Üstelik bu özellik public repositoryler için ücretsiz. Private repositoryler içinse aylık limitlemeler mevcut. Limitleri aştığınız durumlarda aştığınız sürenin ücretini ödemeniz gerekiyor.

![](https://az718566.vo.msecnd.net/uploads/2020/04/actions-pricing.PNG)

Şimdi gelin isterseniz Github Actions kullanımına ufak senaryolarla kısaca bir bakalım.

## Github Actions ile Pull Requestleri Derlemek
Bir Github repositorysi içerisindeki tanımlı akışlara Actions sekmesinden erişebiliyoruz. 

![](https://az718566.vo.msecnd.net/uploads/2020/04/actions-tab.png)

Bu sekmeye eğer ilk kez giriyorsanız size yardımcı olması açısından pek çok hazır task önünüze geliyor. 

![](https://az718566.vo.msecnd.net/uploads/2020/04/action-get-started.png)

Github Actions tarafında pek çok programlama dili ve platform desteği bulunmakta. Biz bu yazıda basit bir ASP.NET Core web projesinin derlenmesini inceleyeceğiz. Bu nedenle burada bulunan .NET Core taskine tıklayarak devam ediyoruz ve karşımıza yml uzantılı bir workflow dosyası geliyor. Bu dosya içerisinde .NET Core uygulamasının derlenmesi için gerekli adımlar tanımlanmış. 

![](https://az718566.vo.msecnd.net/uploads/2020/04/yml-get-started.png)

Bu dosyanın içerisindeki tanımlamalara kısaca baktığımızda ilk olarak bu workflow'un adı ve hangi durumlarda tetikleneceğini belirten tanımlamalar bulunuyor. Yukarıdaki tanımlamada workflowun master branche yapılan pushlarda ve master branche karşı yapılan pull requestlerde çalışacağı yazıyor. Bizim senaryomuzda bu workflow pull requestlerde çalışacağı için üstteki push ile yapılan tanımlamayı silebiliriz. Github actions içerisindeki tetikleyicilerle çok farklı durumlarda tetiklenen akışlar yaratmanız mümkün. Bu nedenle tetikleyicilerin listesine <a href="https://help.github.com/en/actions/reference/events-that-trigger-workflows" target="_blank">buradan</a> bakmanız faydalı olacaktır.

Bu kısmı hallettikten sonra gelelim jobs altındaki tanımlamalara. Burada ilk olarak workflowun çalışacağı runner'ın hangisi olduğunu belirtmemiz gerekiyor. Burayı biz tanımlamadaki gibi ubuntu olarak bırakabiliriz. Ama senaryonuza göre farklı işletim sistemleri de mevcut. Ayrıca matrix workflow desteği de bulunmakta. Yani kodunuzun testini aynı anda farklı işletim sistemlerinde de çalıştırabilirsiniz. Şu an için desteklenen environmentlar aşağıdaki gibi.

![](https://az718566.vo.msecnd.net/uploads/2020/04/runner-environments.png)

Runnerın environmentını da belirttikten sonra geriye çalışacak taskların tanımlanması kalıyor. Bu kısımda test adımını silip geri kalan kısımları aynı kalacak şekilde kaydediyoruz. dotnet commandleri için solution ve project dosyalarının pathlerini de klasör yapınıza göre ayrıca belirtmeniz de gerekebilir. 

YML uzantılı dosyayı kaydettikten sonra artık master branchine karşı yapılan her pull requestte tanımladığımız workflow çalışacak ve durum pull request içerisinde aşağıdaki gibi görüntülenecek. 

![](https://az718566.vo.msecnd.net/uploads/2020/04/pr-check.png)

Gördüğünüz gibi oldukça hızlı bir şekilde her pull request yaratıldığında çalışacak bir workflow tanımladık. Siz de kendi senaryolarınıza yukarıda linkini verdiğim trigger tiplerine göre tetiklenecek workflowlarınızı tanımlayabilirsiniz. 

## Github Actions İle Azure App Service'e Deployment

Gelelim Github Actions ile deployment senaryosuna. Diyelim ki repositorymiz içerisinde bir staging branchi olsun ve bu branche yapılan commitler doğrudan staging ortamına deploy edilsin. 

İlk olarak Github Actions tarafından Azure'a deployment yapılabilmesi için gereken yetkilendirme işini halletmemiz gerekiyor. Bunu yapmanın 2 yolu var. Biri publish profile ile ilerlemek diğeri ise Azure Login ile ilerlemek. Biz basit olması nedeniyle publish profile ile ilerleyeceğiz. Azure login seçeneği için <a href="https://github.com/Azure/login" target="_blank">buradan</a> bilgi alabilirsiniz.

Publish profile ile ilerlemek istediğimizde ilk olarak Azure Portal'e geçip deployment yapmak istediğimiz app service'in publish profile'ını download edip içeriğini kopyalıyoruz. Sonrasında Github'a geri dönerek settings sekmesine geçiyoruz ve oradan da secrets'a tıklayarak yeni bir secret yaratıyoruz. 

![](https://az718566.vo.msecnd.net/uploads/2020/04/publish-profile.png)

Burada secret'a verdiğimiz ismi bir köşeye kaydedelim. Bir sonraki adımda bu isim işimize yarayacak. Şimdi tekrardan Actions sekmesine geri dönüyoruz ve bir önceki bölümdeki gibi yeni bir workflow tanımlıyoruz. Bu workflow staging branchine commit yollandığında çalışacak ve web app deploymentı yapacak. İlk olarak template üzerinden workflow dosyasını yaratıyoruz ve tetikleneceği zamanı değiştiriyoruz. Sonrasında da dotnet publish ile bir deployment paketi yaratıyoruz. Workflow dosyamızın son hali şu şekilde oluyor. 

```yaml
name: Staging Deployment

on:
  push:
    branches: [ staging ]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Setup .NET Core
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 3.1.101
    - name: Install dependencies
      run: dotnet restore
    - name: Build
      run: |
              dotnet build --configuration Release --no-restore
              dotnet publish -c Release -o './app'
```
Şimdi gelelim Azure'a deployment yapma kısmına. Bunun için Azure WebApp actionını kullanacağız. Ancak bu actionı kullanmadan önce kısaca Github Actions Marketplace'ten bahsetmek istiyorum. Github Actions Marketplace içerisinde workflowlarınız içerisinde kullanabileceğiniz hazır actionlar bulunmakta. İstediğiniz actionı aratarak hızlı bir şekilde actiona ve nasıl kullanılacağına erişebiliyorsunuz. 

![](https://az718566.vo.msecnd.net/uploads/2020/04/actions-marketplace.png)

Burada Azure WebApp'e tıklayıp çıkan kodu alıp workflow içerisinde en alta yerleştiriyoruz ve içerisindeki gerekli alanları dolduruyoruz. Workflow dosyasının son hali ise şu şekilde oluyor. 

```yaml
name: Staging Deployment

on:
  push:
    branches: [ staging ]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Setup .NET Core
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 3.1.101
    - name: Install dependencies
      run: dotnet restore
    - name: Build
      run: |
              dotnet build --configuration Release --no-restore
              dotnet publish -c Release -o './app'
    - name: Azure WebApp
      uses: Azure/webapps-deploy@v2
      with:
        app-name: github-actions-test
        publish-profile: ${{ secrets.publish_profile }}
        package: './app'
```

Azure WebApps actionını konfigüre ederken publish profile kısmına yukarıda bahsettiğim gibi yarattığımız secretın adını **${{ secrets.publish_profile }}** şeklinde giriyoruz. Hepsi bu kadar. Bundan sonra staging branchine atılan her commit otomatik olarak App Service'e deploy ediliyor.

Gördüğünüz gibi Github Actions ile oldukça hızlı bir şekilde akışları oluşturup istediğiniz durumlarda tetiklenmesini sağlayabiliyorsunuz. Beni burada etkileyen kısım ise tetikleyiciler oldu. Çok geniş çerçevede bir tetikleyici listesi mevcut.

Bir sonraki yazıda görüşmek üzere








 