# C# Interactive Window #

C# Interactive Window ile ilgili ilk [yazımı](http://www.ilkayilknur.com/roslyn-scripting-api-ve-c-interactive-window) bundan yaklaşık 3 sene önce yazmışım. Roslyn projesi ilk duyurulduğunda projenin gösterilen en önemli kısımlarından biri Interactive Window'du. Ancak zaman geçtikçe öncelikler değişti ve interactive window'un arka planda kullandığı scripting API'larının çıkışı RTM sonrasına ertelendi ve dolayısıyla interactive window'a da ancak **Visual Studio 2015 Update 1** ile kavuşabildik.

## Interactive Window Nedir ? ##

Interactive window, C# dili ile çalışan ve zengin editör desteğine sahip bir  read-eval-print-loop(REPL). **Read-eval-print-loop** terimini biraz daha açarsak, yazmış olduğunuz C# kodunu okuyan ve bunu arka planda çalıştırdıktan sonra da sonucunu ekrana yazan bir uygulama. Yani artık bir kodu denemek için en basitinden bir console uygulaması yaratmaya gerek kalmadan doğrudan kodu REPL uygulaması içerisinden çalıştırıp hızlı bir şekilde sonucunu görebiliriz. Bunun yanında yeni bir dil yeniliği geldiğinde de yine bu yeniliği hızlı bir şekilde REPL uygulaması içerisinde test etmemiz de mümkün.

[Visual Studio 2015 Update 1'i](https://www.visualstudio.com/news/vs2015-update1-vs) bilgisayarımıza kurduktan sonra C# Interactive Window'u 2 şekilde açabiliriz. Bunların ilki ve en kolayı Visual Studio'nun sağ üst köşesinde bulunan **Quick Launch.**

![](http://az718566.vo.msecnd.net/uploads/2015/12/interactive-quick-launch.PNG)

Diğer seçenek ise **View => Other Windows => C# Interactive** üst menüsünü kullanarak açmak.

![](http://az718566.vo.msecnd.net/uploads/2015/12/interactive-menu.png)

Interactive window ilk açıldığında ise aşağıdaki gibi görünüyor.

![](http://az718566.vo.msecnd.net/uploads/2015/12/Interactive-open.PNG)

Interactive  Window'un bir REPL(read-eval-print-loop) olduğundan bahsetmiştik. Dolayısıyla ilk olarak bir Hello World yazıp sonucu görmekte fayda var :) 

    > Console.WriteLine("Hello Interactive Window");
	Hello Interactive Window

Gördüğümüz gibi **Hello Interactive Window** yazısı çalıştırdığımız kodun çıktısı olarak ekrana yazıldı. O zaman işleri biraz daha karıştıralım ve önce bir değişken tanımlayalım ve sonrasında da bu değişken üzerinden bir işlem yapalım.

	> List<int> numbers = new List<int> { 4, 1, 6, 3, 9 };
	> numbers.OrderBy(t => t).ToList().ForEach(t => Console.WriteLine(t));
	1
	3
	4
	6
	9

Interactive Window içerisinde yazacağımız kodların illa tek satırdan oluşması gibi kısıt yok. Aynı kod yazar gibi Enter tuşuna basıp kod yazmaya devam edebiliriz.

	> foreach(var item in numbers.OrderBy(t=>t))
	. {
	.     Console.WriteLine(item);
	. }
	1
	3
	4
	6
	9

Interactive window içerisinde tanımladığımız değişkenler bir context içerisinde saklanıyor. Dolayısıyla önceki kod parçalarında tanımladığımız değişkenlere istediğiniz zaman window içerisinden tekrar ulaşıp kullanabiliyoruz. 

![](http://az718566.vo.msecnd.net/uploads/2015/12/interactive-window-context.PNG)

Interactive Window içerisinde metotlar yazıp sonrasında bu metotları çağırmamız da mümkün. 

	> void PrintOrderedNumbers(List<int> unorderedList)
	. {
	.     unorderedList.OrderBy(t => t).ToList().ForEach(t => Console.WriteLine(t));
	. }
	> PrintOrderedNumbers(numbers);
	1
	3
	4
	6
	9

Interactive window içerisinde kullanmak istediğimiz bir tipin bulunduğu namespace'i using ifadesi kullanarak ekleyebilmekteyiz.

	> using System.Threading;

Interactive Window içerisindeki context default olarak **async** tanımlı. Dolayısıyla **await** kullanarak istediğimiz bir async bir metodu çağırabiliriz.

	> using System.Web;
	> using System.Net;
	> WebClient client = new WebClient();
	> var response = await client.DownloadStringTaskAsync("http://ilkayilknur.com");
	> Console.WriteLine(response.Substring(0, 50));
	<!DOCTYPE html>
	<html>
	<head prefix="fb: http://

## Directives ##

Interactive Window içerisinde bazı özel komutları kullanarak özel bazı işlemler yapmamız da mümkün. Bunlardan ilki Interactive Window içerisine dll referansı ekleme komutu'i olan **#r** komutu.

![](http://az718566.vo.msecnd.net/uploads/2015/12/interactive-window-reference.PNG)

Referansı ekledikten sonra yukarıda görüldüğü gibi dll içerisindeki tipleri intellisense'te görebiliyoruz ve yazacağımız kodlarda bu tipleri kullanabiliyoruz.

Eğer interactive window ekranındaki kodları temizlemek istersek  **#cls** veya **#clear** komutlarını kullanabiliriz. Bu komutlar sadece ekranda bulunan kodları temizliyor. Yani daha önceden tanımlamış olduğumuz değişken veya metotların hepsini koruyor. Eğer interactive window'u tamamen sıfırlamak yani tüm tanımladığınız değişkenler,metotlar vb.. tümünden kurtulmak istersek kullanacağımız komut **#reset** komutu olacak.

Interactive window içerisinde dışarıdan bir dosya içerisinde bulunan kodları çalıştırmamız da mümkün. Örneğin aşağıdaki kodu bir txt dosyasına kopyalayıp bilgisayarda bu dosyayı kaydedelim.

	void PrintOrderedNumbers(List<int> unorderedList)
	{
	     unorderedList.OrderBy(t => t).ToList().ForEach(t => Console.WriteLine(t));
	}
	PrintOrderedNumbers(numbers);

Sonrasında interactive window içerisinde **#load** komutunu kullanarak bu dosya içerisindeki kodları çalıştırıp sonucunu interactive window içerisinde görebiliriz. 

	> #load "C:\Users\ilkay\Documents\GitHub\xomnicloud\xomni-sdk-dotnet\src\XOMNI.SDK.Public\bin\Debug\code.txt"
	1
	3
	4
	6
	9

Son olarak ise interactive window içerisinde komutlarla ilgili yardım almak istersek #help komutunu kullanabiliyoruz.

Görüldüğü gibi interactive window Visual Studio içerisindeki en vazgeçilmez özelliklerden biri olmaya aday. Özellikle ufak bir kod parçasının nasıl çalıştığını görmek için veya hızlıca demolar yapmak için oldukça güzel bir ortam. Hala daha bazı eksikleri olsa da (örneğin nugetten referans ekleme) ileride bu eksik noktalar da giderilecektir diye düşünüyorum.

Bir sonraki yazıda görüşmek üzere.