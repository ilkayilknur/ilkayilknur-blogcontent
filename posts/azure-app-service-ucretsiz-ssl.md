Herkese Selamlar,

Bu yazıda Azure App Servicelere ücretsiz olarak sunulan Let's Encrypt sertifikalarını nasıl yükleyebileceğimizden ve bu işlemleri nasıl otomatize edebileceğimizden bahsedeceğiz. 

Şu an hali hazırda Azure'da bulunan bir app service'te HTTPS'i enable etmek için dışarıdan bir SSL sertifikası satın alıp yükleyebiliyoruz. Bunun yanında SSL sertifikalarının ücretsiz olarak üretilmesi ve dağıtılması amacıyla kurulan <a href="https://letsencrypt.org/" target="_blank">Let's Encrypt</a> üzerinden de SSL sertifikalarını ücretsiz olarak almak mümkün. Let's Encrypt tarafından üretilen sertifikaların geçerlilik süresi 90 gün. Bu nedenle 90 günde bir manuel olarak sertifika yüklemek yerine app service üzerinde <a href="https://github.com/sjkp/letsencrypt-siteextension" target="_blank">Let's Encrypt site extensionını</a> kullanarak bu işlemleri otomatize edeceğiz. Şimdi gelin ilk adımdan en son adıma kadar otomatize etmemiz için yapmamız gereken işlemlerden bahsedelim. 

İlk olarak site extensionın belirli aralıklarla çalışıp bizim app servisimize erişerek sertifikaları yükleyebilmesi için yetkilendirme işlerini halletmemiz gerekiyor. Bunun için ilk olarak Azure AD'de bir app yaratıyoruz. 

![](https://az718566.vo.msecnd.net/uploads/2020/05/06/azure-ad-app-registration.png)

Appi yarattıktan sonra **Certificates&secrets** bölümüne geçip buradan bir client secret yaratıyoruz ve bu yarattığımız client secretı bir köşede saklıyoruz. İleride ihtiyacımız olacak! 

![](https://az718566.vo.msecnd.net/uploads/2020/05/06/add-client-secret.png)

Şimdi de yukarıda yarattığımız app'in Azure App Service'e erişebilmesi için App Service'in içerisinde bulunduğu resource group'a bu app'i Contributer olarak ekliyoruz. Bunun için resource group içerisindeki Access Control(IAM) bölümüne geçmemiz gerekiyor. 

![](https://az718566.vo.msecnd.net/uploads/2020/05/06/resource-group-IAM.png)

![](https://az718566.vo.msecnd.net/uploads/2020/05/06/add-role-assignment.png)


Şimdi App Service'e geçip yukarıda bahsettiğimiz site extensionı kuruyoruz. 

![](https://az718566.vo.msecnd.net/uploads/2020/05/06/extension-install.png)

Extensionı kurduktan sonra extensiona tıklayıp detay sayfasından Browse'a tıklıyoruz ve ayar ekranına geliyoruz. Burada bazı ayarları yapmamız gerekiyor. 

![](https://az718566.vo.msecnd.net/uploads/2020/05/06/site-settings.png)

Yukarıdaki settinglerden en çok karıştırılan genelde ilk 4 setting oluyor. Bunlardan kısaca bahsedeceğim. 

**Tenant:** Azure AD'nin tenant IDsi. Bu değeri azure AD'ye girdiğinizde overview ekranında bulabilirsiniz. 

**Subscription:** App Service'in içerisinde bulunduğu subscription'ın IDsi.

**ClientId:** İlk adımda app register etmiştik. Orada register ettiğimiz app'in client IDsi. Register ettiğimiz app'e tıkladığımızda en üstte bu değeri görebiliriz. 

![](https://az718566.vo.msecnd.net/uploads/2020/05/06/app-client-id.png)

**ClientSecret:** En başta app yarattıktan sonra bir client secret yaratmıştık ve buradaki secretı size bir köşede saklayın demiştim :) İşte o secret buraya gelecek. 

Diğer alanlar gayet açıklayıcı olduğu için detaya girmiyorum. En son olarak da en alttaki **Update Application Settings and Virtual Directory** checkbox'ını da seçmemiz gerekiyor. Sonrasında next'e tıklayıp bir sonraki adıma geçebiliriz.

Bir sonraki adımda app servicete ayarlanan hostnameler listeleniyor. Eğer hostname'inizin ayarını yapmadıysanız öncelikle ilgili ayarlamayı yapıp ondan sonra bu ekrana gelmeniz gerekiyor. Bu ekranı da next diyip geçiyoruz. 

![](https://az718566.vo.msecnd.net/uploads/2020/05/06/custom-domains.png)

Evet son adıma geldik. 

![](https://az718566.vo.msecnd.net/uploads/2020/05/06/install-certificate.png)

Burada hangi hostname için sertifika yaratacağımızı seçip **Request and Install Certificate** butonuna tıklıyoruz ve işlemi tamamlıyoruz. Artık sitenizi açıp sertifikasını görebilirsiniz. Sertifikalar en başta bahsettiğim gibi 90 günlük. Ancak bu site extension sertifikaların yenilenme operasyonunu da kendisi hallediyor. Dolayısıyla günü kaçırma, unutma gibi sorunlarla da uğraşmıyoruz :)

Bir sonraki yazıda görüşmek üzere,






