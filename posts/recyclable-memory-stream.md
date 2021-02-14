Gerek blogda performansla ilgili yazdığım yazılarda gerekse .NET Conf'ta yaptığım "[Yüksek Performanslı Uygulama Geliştirme](https://www.youtube.com/watch?v=4gzIVZgqLBU)" oturumunda memory allocationları azaltmakla ilgili pek çok yöntemden bahsettim. Özellikle LOH(Large Object Heap) allocationları konusunda oldukça dikkatli olmamız gerektiğinden çünkü LOH'da yer kalmadığı durumlarda uygulamalarımızda full GC tetiklendiğinden dolayı kısa süreli pauselar olabileceğinden pek çok kez bahsettim. 

Bu yazıda da `MemoryStream`  kullandığımız durumlarda memory allocationları nasıl azaltabileceğimizden ve `MemoryStream`'in arkasındaki bufferı nasıl daha optimize kullanabileceğimizi inceleyeceğiz. 

[Buradan da](https://source.dot.net/#System.Private.CoreLib/MemoryStream.cs,1a4dcb744a23ba6f) görebileceğiniz üzere her yeni `MemoryStream` instance'ı yaratıldığında(dışarıdan buffer kabul etmediği constructor kullanıldığında) arka planda `buffer` olarak yeni bir byte array yaratılıyor. Yine implementasyondan da görebileceğiniz gibi yazma işlemi sırasında eğer arkada kullandığı buffer yeterli büyüklükte değilse otomatik olarak  mevcut büyüklüğünün iki katı büyüklükte yeni bir array allocate edip kapasitesini iki katına çıkarıyor. Sonrasında da eski bufferdan yeni buffera array kopyalama işlemi yapılıyor. Eğer `MemoryStream`'in arkasındaki bufferın büyüklüğü 85K bytes'ı geçerse bu array bu seferde LOH üzerinde yaratılıyor olacak. Eğer performans kritik bir uygulama geliştiriyorsanız ve `MemoryStream` kullanıyorsanız bu yazdıklarım eminim canınızı sıkmıştır :) Öncelikle `MemoryStream`'e yazma işlemi yapılırken arka planda fazla sayıda ve neredeyse kullan-at mantığında array oluşturmak oldukça büyük bir maliyet ve GC üzerinde de bir yük. Bu arraylerin boyutlarının büyüdüğü durumlarda ve LOH üzerinde oluşmaya başladığı senaryolarda ise bu yük daha da artıyor.

İşte `RecyclableMemoryStream` tipi tam da bu noktada yardımımıza koşuyor ve `MemoryStream` tipinin yukarıda bahsettiğim performans ve allocation perspektifinden can sıkabilecek iç implementasyonlarını oldukça iyi bir şekilde optimize ediyor. Özellikle arka planda kullanılan bufferların bir poolda tutulması, bufferların genişlemesi gerektiği senaryolarda da bir linked list yapısı oluşturarak yine pool içerisindeki bufferları kullanması büyük avantaj sağlıyor. 

Öncelikle `RecyclableMemoryStream` tipini kullanabilmek için projemize `Microsoft.IO.RecyclableMemoryStream` paketini yüklememiz gerekiyor.

```bash
dotnet add package Microsoft.IO.RecyclableMemoryStream
```

Paketi yükledikten sonra ilk olarak bir `RecyclableMemoryStreamManager` instance'ı oluşturmamız gerekiyor. `RecyclableMemoryStreamManager` tipi `RecyclableMemoryStream`'in arka planındaki bufferların yönetiminden sorumlu. Senaryolarınıza uygun olacak şekilde bu instance'ı uygulamanız içerisinde singleton olarak da kullanabilirsiniz yada farklı farklı durumlar için farklı poollar tutmak isterseniz birden fazla instance da yaratabilirsiniz. `RecyclableMemoryStreamManager` instance'ını yarattıktan sonra bu instance üzerinden de `GetStream` metodunu kullanarak bufferları  `RecyclableMemoryStreamManager` tarafından yönetilen bir `RecyclableMemoryStream` instance'ı alabilirsiniz. Bu metottan gelen `RecyclableMemoryStream`'i klasik `MemoryStream` kullanırmış gibi kullanıp sonrasında `Dispose` etmelisiniz. Böylece dispose edilme sırasında arkasında kullanılan buffer veya buffer listesindeki tüm bufferlar tekrardan `RecyclableMemoryStreamManager` tarafından tanımlanan poola geri bırakılabilir.

```csharp
public async Task MySuperImportantMethodAsync()
{
    var streamManager = new RecyclableMemoryStreamManager();
    await using (var stream = streamManager.GetStream())
    {
        //memory streami burada kullanın
    }
}
```



`RecyclableMemoryStream`'in kullanım şekli `MemoryStream`'den çok farklı olmasa da arka planda çalışan mantık oldukça farklı. Biraz detaylandırırsak `RecyclableMemoryStreamManager` arka planda iki farklı tipte pool bulundurmakta. Bunlardan biri small pool diğeri de large pool. Small pool default olarak kullanılmakta ve stream üzerinde bir yazma veya okuma işlemi yapıldığında eğer buffer büyüklüğü yeterli değilse pooldan yeni bir buffer alınıp arka planda tutulan buffer listesine eklenmekte ve stream içerisinde kullanılmakta. 

`RecyclableMemoryStream` içerisindeki `GetBuffer` metodunu kullanarak arka planda tutulan buffera erişmek isterseniz `RecyclableMemoryStream` size bu bufferı tek bir parça olarak verebilmek için large pool içerisinden bir buffer almakta bu buffer içerisinde size tüm buffer içeriğini dönmekte. Burada pooling mekanizması olduğu için size dönen bufferın büyüklüğü aslında tam stream'in size'ından büyük olabilir. Bu nedenle aynı [array poolling](https://www.ilkayilknur.com/new-coreda-array-pooling) mekanizmasında olduğu gibi doğru bir şekilde kullanmakta fayda var. Large pool içerisindeki bufferların boyutu sizin konfigürasyonunuza göre lineer veya eksponansiyel olarak büyüyebilmekte. Small pooldaki bufferların büyüklüğü default olarak 128KB iken large poolda bu rakam 1MB.

`RecyclableMemoryStreamManager` instance' ı yaratırken arka planda yaratılan pool ile ilgili aşağıdaki optimizasyonları yapabiliyoruz.

* `blockSize`: poolda saklanacak her bir blockun büyüklüğü
* `largeBufferMultiple`: large buffer eksponansiyel veya lineer olarak büyürken bu değerin katları veya eksponansiyel olarak büyür. 
* `maximumBufferSize`: Poolda saklanacak olan bufferların maksimum büyüklüğü. Bu değerden büyük olan bufferlar yaratılır ama poolda saklanmaz.
* `useExponentialLargeBuffer`: Large bufferlar için eksponansiyel büyüme stratejisinin kullanılıp kullanılamayacağı.

Ayrıca `RecyclableMemoryStreamManager` içerisindeki `MaximumFreeLargePoolBytes`, `MaximumFreeLargePoolBytes` propertyleri ile pool içerisinde hazır olarak ne kadar buffer bulunacağı belirtilebilir. Böylece eğer belirtilen değer kadar poolda buffer varsa poola geri döndürülmek istenen bufferlar discard edilir ve GC tarafından toplanır.

`RecyclableMemoryStream` kullanırken `ToArray` metodunu çağırdığımızda ise `GetBuffer` metodundan farklı olarak tamamen yeni bir array yaratılıp buffer içeriği bu array'e kopyalanmakta. Dolayısıyla `RecyclableMemoryStream`'in sağlamış olduğu hiçbir avantajdan da yararlanamıyoruz. Bu nedenle `RecyclableMemoryStreamManager` içerisindeki `ThrowExceptionOnToArray` propertysini kullanarak `ToArray` çağrılarında exception fırlattırıp kullanılmasının önüne geçebilirsiniz. 

`RecyclableMemoryStream` kullanırken arka plandaki pooling mekanizmalarının nasıl işlediğini debug edebilmek için ETW eventlerini de kullanabilirsiniz. Aynı zamanda  `RecyclableMemoryStreamManager` içerisindeki çeşitli eventleri de kullanarak bazı durumlardan haberdar olmak mümkün. Bunlarla ilgili detaylı bilgi edinmek için projenin [GitHub](https://github.com/microsoft/Microsoft.IO.RecyclableMemoryStream) repositorysine bakabilirsiniz.

Bir sonraki yazıda görüşmek üzere,



Kaynak : https://github.com/microsoft/Microsoft.IO.RecyclableMemoryStream 



