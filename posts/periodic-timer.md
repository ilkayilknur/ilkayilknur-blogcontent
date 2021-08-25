.NET içerisinde pek çok alternatifini bulabileceğimiz yapılardan biri de `Timer` yapıları. Geçmişten bugüne kadar farklı farklı noktalarda farklı ihtiyaçlara cevap veren timer yapıları framework içerisine eklendi ve aktif olarak kullanılmakta.

.NET 6 öncesinde kullanabileceğimiz timer tiplerini kısaca listelersek...
* `System.Threading.Timer`
* `System.Timers.Timer`
* `System.Windows.Forms.Timer`
* `System.Web.UI.Timer`
* `System.Windows.Threading.DispatcherTimer`

.NET 6 ile beraber ise bu listeye bir timer tipini daha ekliyor olacağız. .NET 6 Preview 7 ile beraber bu listeye dahil olan timer tipinin adı `PeriodicTimer`. `PeriodicTimer` da tabi ki mevcut olan timer tiplerin içerisindeki bir takım problemleri çözmek amacıyla framework içerisine eklendi. Mevcut timer tiplerinde sorun yaratan konuların başında bu timer tiplerinin callback tabanlı çalışması yatmaktaydı. Bu callback yapıları bazen memory leaklere neden olabilmekte ve sorun çıkarabilmekte. Aynı zamanda bu callbackler içerisinde `sync over async` kod yazmamız gerekmekte. Mevcut timer tiplerinde timer ticklendiğinde çalışacak olan kodlar eğer çalışma süreleri uzun sürerse arka arkaya tetiklenebilmekte ve eğer kodumuzu buna göre yazmazsak sorunlara neden olabilmekte.

Tüm bu nedenlerden dolayı .NET 6 ile beraber kullanımı oldukça basit bir timer yapısı hayatımıza giriyor. Şimdi kod kısmına geçelim.

Bir `PeriodicTimer` instance'ı yaratmak oldukça basit. Sadece constructora ne kadar sürede bir tetikleneceğini bildirmemiz gerekmekte.

```csharp
var timer = new PeriodicTimer(TimeSpan.FromSeconds(3));
```
Sonrasında da `WaitForNextTickAsync` metodunu çağırarak bir sonraki tetiklenmeyi async olarak bekleyebilir, ardındanda çalışacak olan kodu çalıştırabiliriz.

```csharp
var timer = new PeriodicTimer(TimeSpan.FromSeconds(3));

while (await timer.WaitForNextTickAsync())
{
    Console.WriteLine("Timer tick");
}
```

`WaitForNextTickAsync` metodunu tekrar çağırmadığımız sürece timer tekrardan verilen periyot kadar bekleme operasyonunu başlatmadığı için yazdığımız kodun üst üste çalışması gibi bir durum da olmamakta.

Şimdi gelelim `PeriodicTimer`'ın durdurulması senaryosuna. Bunun için iki farklı yöntem bulunmakta. Bunlardan biri `WaitForNextTickAsync` metoduna bir `CancellationToken` verilmesi. Diğeri ise `PeriodicTimer`'ın dispose metodunun çağırılması. `CancellationToken` kullanımında eğer operasyon iptal edilirse  exception fırlatılırken, dispose etme senaryosunda `WaitForNextTickAsync` metodundan `false` dönmekte.

Bu yazıda .NET 6 ile beraber gelen `PeriodicTimer`'ı inceledik. Bir sonraki yazıda görüşmek üzere.

Kaynak : <a href="https://github.com/dotnet/runtime/issues/31525" target="_blank">https://github.com/dotnet/runtime/issues/31525</a>