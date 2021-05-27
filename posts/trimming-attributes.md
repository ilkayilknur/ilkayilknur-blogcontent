Bundan önceki yazılarımdan <a href="https://www.ilkayilknur.com/dotnet-uygulama-yayinlama-opsiyonlari" target="_blank">birinde</a> uygulamalarımızı yayınlarken kullanabileceğimiz opsiyonlardan bahsetmiştim. O yazıda .NET'in kurulu olmadığı yerlerde uygulamayı .NET runtime'ı ile birlikte deploy etme opsiyonunun deployment paketlerini ne kadar büyüttüğünü de görmüştük. Bu sorunun çözümü olarak uygulamanızı publish ederken sadece uygulamanın kullandığı ve uygulamanın çalışması için gerekli olan kütüphanelerin publish edilebilmesini sağlayan trimming yapısına da kısaca değinmiştik.

.NET 3.1 ile gelen trimming yapısı bizim yazdığımız kodun statik analizini yaparak uygulamanın kullandığı assemblyleri bulmakta ve bu assemblyleri paket içerisine dahil etmekte. .NET 5.0 ile beraber gelen member trimming yapısı da bunu bir ileri seviyeye taşıyarak assemblyler içerisindeki kullanılmayan tipleri de silerek paket boyutunun daha da küçülmesini sağlamakta. Tabi bu trimming yapılarının .NET 3.1 ve .NET 5 içerisinde henüz *experimental* olduğunu da belirtmemiz gerek.

Trimming yapılırken statik kod analizi yapıldığı için problem çıkarabilecek noktalardan biri de reflection kullanımları. Reflection kullanarak tiplerin içerisindeki üyelere dinamik olarak eriştiğimiz senaryolarda veya tip isminin string olarak saklandığı, sonrasında da bu string isim üzerinden reflection kullanarak yaptığımız dinamik kod üretmeler vs.. gibi senaryolar trimming yapısı tarafından tam anlamıyla analiz edilemediği için ihtiyacımız olan bazı tipler veya tiplerin içerisindeki üyelerin trimmer tarafından silinmesi söz konusu olabilir. Bunu önlemek için `DynamicallyAccessedMembersAttribute` attribute'ünü kullanarak trimmera hangi üyeleri saklaması gerektiğini söyleyebiliyoruz.

Şu şekilde bir örnek kullanım gösterebiliriz. 
```csharp
public void Foo([DynamicallyAccessedMembers(DynamicallyAccessedMemberTypes.PublicMethods)] Type type)
```
Yukarıda bu metot içerisine parametre olarak gelen tipin public metotlarını kullanacağımızı belirtiyoruz. Böylece trimmer public metotları trim etmiyor.

Aynı zamanda flag yapısıyla birden fazla değeri de belirtebiliriz. 

```csharp
public void Foo([DynamicallyAccessedMembers(DynamicallyAccessedMemberTypes.PublicMethods | DynamicallyAccessedMemberTypes.PublicConstructors)] Type type)
```

Bu attribute'ü string fieldlarda da kullanabilmekteyiz. Tip isimlerinin string fieldlarda saklandığı senaryolarda da trimmera bildirimde bulunabiliriz. 

```csharp
public void Foo([DynamicallyAccessedMembers(DynamicallyAccessedMemberTypes.PublicConstructors)] string typeName)
{

}
```

Trimming esnasında trimmera bildirimde bulunabileceğimiz bir diğer attribute ise `DynamicDependencyAttribute` attribute'ü. Dinamik kullanımlarda trimmera dependencyleri bildirebiliyoruz. 

```csharp
[DynamicDependency(DynamicallyAccessedMemberTypes.All, "System.Collections.Stack", "mscorlib")]
[DynamicDependency(DynamicallyAccessedMemberTypes.All, "System.Collections.Queue", "mscorlib")]
public IEnumerable foo(bool lifo)
{
    string typename = (lifo) ? "System.Collections.Stack" : "System.Collections.Queue";
    return (IEnumerable) Activator.CreateInstance("mscorlib", typename).Unwrap();
}
```

Yukarıdaki attribute kullanımı ile trimmera basitçe eğer `foo` metodunu trim etmeyeceksen paket içerisine `System.Collections.Stack` ve `System.Collections.Queue` tiplerini dahil etmelisin çünkü bu tiplere metot içerisinde ihtiyaç duyacağım mesajını iletmiş oluyoruz. Böylece dinamik kullanımlarda da trimmerın istenmeyen sonuçlara yol açmasını önleyebiliyoruz.

Kendi yazdığımız kodlarda yukarıdaki gibi bildirimleri eklemek çok zor olmasa gerek. Peki diyelim ki third party kütüphaneler kullanıyoruz ve bu kütüphaneler yukarıda bahsettiğim attributeleri kodlarına yerleştirmemiş olsunlar. Sonrasında trim ederken bu kütüphanelerden kaynaklanan bir sorun olduğunda nasıl müdahale edebiliriz sorusu akıllara geliyor. Bunun gibi soruna neden olan ancak kod değişimi de yapamadığımız senaryolarda trimmera bildirimde bulunabileceğimiz çeşitli opsiyonlar bulunmakta. Bu opsiyonlar basitçe xml formatında dosya oluşturup, içerisine trim seçeneklerini ekleyip sonrasında da trimmera bu xml dosyasını parametre geçmek şeklinde olmakta. Bu konuyla ilgili detaylı bilgileri de <a href="https://docs.microsoft.com/en-us/dotnet/core/deploying/trimming-options" target="_blank">buradan</a> alabilirsiniz. 

Bir sonraki yazıda görüşmek üzere.

Kaynaklar
* https://docs.microsoft.com/en-us/dotnet/core/deploying/trim-self-contained
* https://docs.microsoft.com/en-us/dotnet/core/deploying/trimming-options
* https://devblogs.microsoft.com/dotnet/app-trimming-in-net-5/