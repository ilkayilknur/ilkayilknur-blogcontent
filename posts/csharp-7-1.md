Daha önceki blog yazılarımda ve videolarımda C#'ın minor releaselerinin çıkarılması için çalışmalar yapıldığından bahsetmiştim. Şimdiye kadar baktığımızda her yeni Visual Studio versiyonu ile beraber C#'ın da yeni versiyonu çıkıyordu. Bu nedenle hem büyük kapsamlı yenilikler hem de ufak yenilikler doğrudan tek bir versiyon içerisinde bulunuyordu. Halbuki dil içerisindeki bazı yenilikler geliştirilmesi bitse bile yeni versiyon çıkana kadar biz developerlarla buluşamıyordu. 

İşte C# 7.1, bu noktada C#'ın ilk minor release'i olmakta. Visual Studio 2017 kullananlar için 15.3 versiyonu ile beraber gelirken, .NET Core'da ise 2.0 SDK'i ile beraber geliyor. Ancak bu versiyonları edinseniz bile C# 7.1 yenilikleri otomatik olarak kapalı geliyor. Bu nedenle ayrıca açmak için ufak bir ayar yapmaya ihtiyacımız var. Bunun için projemize sağ tıklayıp, **Properties** seçeneğini seçiyoruz ve sonra da çıkan ekrandan **Build** tabına geçiyoruz. En altta bir Advanced butonu olacak ve o butona tıkladığımızda ise şöyle bir ekran karşımıza gelecek.

![](http://az718566.vo.msecnd.net/uploads/2017/11/advanced-build.PNG)

Gördüğünüz gibi Language Version olarak latest major versiyon seçili. Yani default olarak C# 7.0 kullanılıyor. Language version seçeneklerini açtığımızda ise şu şekilde bir liste ile karşılaşıyoruz. 

![](http://az718566.vo.msecnd.net/uploads/2017/11/advanced-build.png)


Buradan eğer **"latest minor version"** seçeneğini seçersek projede artık release olan tüm C# minor versiyonlar updateler geldikçe otomatik olarak kullanımda olacaktır. Bunları yapmak yerine belirli C# versiyonunu seçip projede hep o versiyonun kullanılmasını da sağlayabilirsiniz. Bu ayarı yaparken unutulmaması gereken bir nokta da bu ayarın build konfigürasyonu başına yapılması. Yani yaptığınız değişikliğin hem debug hem de release modda yapılmış olması gerekiyor. Eğer csproj dosyasını manuel editleyerek yapmak isterseniz bunu göz önüne almayı unutmayın. Eğer Visual Studio üzerinden yukarıdaki gibi ayar yapmak istiyorsanız da projeye sağ tıklayıp Properties'e tıkladığınızda gelen ekranda sol yukarıdaki Configuration combobox'ında **All Configurations'ı** seçmeyi unutmayın.

![](http://az718566.vo.msecnd.net/uploads/2017/11/advanced-build-configuration.png)

Şimdi gelelim C# 7.1 özelliklerine...

## Async Main ##

C# 5.0 ile beraber async/await geldiğinden beri özellikle demo yaparken Main metodunun async olamaması ile ilgili hep bir istisna olduğundan bahsediyorduk. Methodu async yapamadığımız için de async bir metot çağırırken bazı farklı kullanımlarda bulunuyorduk. Artık yıllar sonra artık bu durum sona erdi ve Main metot Task veya Task<<int>int> döndürebilir hale geldi. Main metot içerisinde async metotları gönül rahatlığıyla artık await edebiliriz :)

## Default Literals ##

C# içerisinde bir tipin default değerini alabilmek için default expressionları kullanabiliyorduk. 

Örneğin,

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;x&nbsp;=&nbsp;<span style="color:blue;">default</span>(<span style="color:blue;">string</span>);
<span style="color:blue;">var</span>&nbsp;y&nbsp;=&nbsp;<span style="color:blue;">default</span>(<span style="color:#2b91af;">Func</span>&lt;<span style="color:blue;">string</span>,&nbsp;<span style="color:blue;">int</span>,&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:blue;">string</span>&gt;&gt;);&nbsp;<span style="color:green;">//&nbsp;&lt;----&nbsp;:(</span></pre>

Sadece string gibi tiplerin default değerini alırken yazım olarak çok zorlanmasak da özellikle kompleks generic tipler olduğunda onları yazmak bazen zor olabiliyordu. Yukarıda verdiğim ikinci  örnek bu zorluğa ufak bir örnek olabilir. C# 7.1 ile beraber compiler tipini tahmin edebilidiği durumlarda artık sizin default a ekstra bir tip vermenizi zorunlu kılmıyor. Bu nedenle doğrudan default literal'i kullanabiliyorsunuz. 1-2 kullanım yaparsak örnek olarak...

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">static</span>&nbsp;<span style="color:blue;">async</span>&nbsp;<span style="color:#2b91af;">Task</span>&nbsp;Main(<span style="color:blue;">string</span>[]&nbsp;args)
{
&nbsp;&nbsp;&nbsp;&nbsp;Foo(<span style="color:blue;">default</span>,&nbsp;12);
}
 
<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">int</span>&nbsp;Foo()
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:blue;">default</span>;
}
 
<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Foo(<span style="color:blue;">string</span>&nbsp;k,&nbsp;<span style="color:blue;">int</span>&nbsp;y)
{
 
}</pre>

 Yukarıda görüldüğü gibi metotların alacakları parametrelerde, döndüreceği değerlerde ve daha başka pek çok noktada sadece default literalini kullanabiliriz. Burada kritik nokta compilerın default değerini alacağı tipi tahmin edebilmesi. Tahmin edemediği durumlarda zaten sizden tipi ayrıca belirtmenizi isteyecektir. 

 ## Tuple isimlendirme geliştirmeleri ##

C# 7.0 ile beraber gelen benim en favori özelliğim tuplelar. C# 7.1 ile beraber de tuple içerisindeki fieldların isimlendirilmesi ile ilgili ufak ama güzel bir yenilik geliyor. Tuple kullanımında tuple yaratırken fieldlara dışarıdan isim verebiliyorduk. 

Örneğin,
<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;count&nbsp;=&nbsp;5;
<span style="color:blue;">var</span>&nbsp;sum&nbsp;=&nbsp;50;
<span style="color:blue;">var</span>&nbsp;retVal&nbsp;=&nbsp;(sum:&nbsp;sum,&nbsp;count:&nbsp;count);</pre>

Bu şekilde kullanımlarda tuple içerisine koyduğumuz değişkenlerin isimleriyle tuple içerisindeki fieldların ismini büyük oranda aynı oluyor. Dolayısıyla aslında bunu biz belirtmesek de compiler bunu arka planda kendisi yapsa nasıl olur :) C# 7.1 ile artık bu mümkün

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;count&nbsp;=&nbsp;5;
<span style="color:blue;">var</span>&nbsp;sum&nbsp;=&nbsp;50;
<span style="color:blue;">var</span>&nbsp;retVal&nbsp;=&nbsp;(sum,&nbsp;count);
 
<span style="color:#2b91af;">Console</span>.WriteLine(<span style="color:#a31515;">$&quot;Count:</span>{retVal.count}<span style="color:#a31515;">&nbsp;,&nbsp;Sum:</span>{retVal.sum}<span style="color:#a31515;">&quot;</span>);</pre>

Yukarıda gördüğünüz gibi tuple yaratırken içerisine verdiğimiz değişkenlerin isimleriyle aynı isimde fieldlar yaratılıyor. 

Gördüğünüz gibi C# 7.1 içerisinde ufak ama günlük hayatımızda kod yazarken bize yardımcı olacak özellikler gelmekte. Bir sonraki release olacak olan 7.2 release'inde çok daha farklı yenilikler geliyor olacak. Artık bu özellikler için 2 yıl yeni Visual Studio versiyonunu beklemeye gerek yok.


