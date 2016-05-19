# C# 7.0 Özelliklerini Nasıl Test Ederiz ? #

Bir [önceki](http://ilkayilknur.com/csharp-7-da-bizi-neler-bekliyor) blog yazısında C# 7.0 yeniliklerinden bahsettiğim videomu paylaşmıştım. Diyelim ki videoyu izlediniz veya başka yerlerden okuyup heyecanlandınız ve C# 7.0 özelliklerini test etmek istiyorsunuz :) Bu özellikleri test edebilmeniz için önünüzde 2 yol var. Biri kısmen kolay olan yol diğeri ise oldukça zorlu :) Önce kolay olandan başlayalım.

## Visual Studio 15 Preview Üzerinde Test Etmek ##
Şu anda C# 7.0 özelliklerini test etmenin en kolay yolu **Visual Studio 15 Preview** kullanmak. Visual Studio 15 Preview sürümü adı üzerinde henüz preview olduğu için production makinanıza kurmanızı kesinlikle  tavsiye etmiyorum. Bu yüzden bilgisayarınızda bir sanal makina yaratıp orada Visual Studio 15'i kurup testlerinizi yapabilirsiniz. Bir diğer alternatif ise Azure üzerinde Visual Studio 15 Preview imajlı bir VM yaratmak.

Visual Studio 15 Preview Installer Linki : [http://aka.ms/vsnext](http://aka.ms/vsnext)

Visual Studio 15 Preview'ı kurdunuz ve hemen bir proje yaratıp özellikleri test etmek istiyorsunuz.  Bir örnek yaptınız ama o da ne projeniz derlenmiyor ve hata veriyor. 

![](http://az718566.vo.msecnd.net/uploads/2016/05/csharp7_compiler_error.png)

Bu hatayı almamızın sebebi C# 7.0 özelliklerinin her birinin feature flagleri arkasına alınması. Yani ekip hızlı bir şekilde istediği özellikleri açıp kapatabiliyor gerekli olduğunda. Bu hatayı aşmanın yolu ise oldukça basit. Test yaptığımız projeye conditional compilation symbols olarak **__DEMO__** ve **__DEMO_EXPERIMENTAL__** 'ı eklemek. 

![](http://az718566.vo.msecnd.net/uploads/2016/05/compiler_flags.png)

Bu işlemi de yaptıktan sonra kodunuz başarılı olarak derlenecektir. Ancak her Visual Studio 15 Preview versiyonunda, duyurulan C# 7.0 özellikleri olmayabilir. Örneğin bu yazının yazıldığı dönemde bulunan Visual Studio 15 Preview içerisinde Tuple özellikleri henüz bulunmuyor.

## Roslyn Source Code'unu Çalıştırıp Test Etmek ##
Gelelim diğer opsiyona. Github üzerinden Roslyn projesinin kodlarını indirip projeyi derleyip aynı şekilde özellikleri test edebilirsiniz. Daha önce bunu denemiş biri olarak uyarmam gerekir ki bu iş göründüğü kadar kolay değil. Kısaca yol göstermem gerekirse

- Roslyn projesi Github linki : [https://github.com/dotnet/roslyn](https://github.com/dotnet/roslyn)
- Roslyn source code'u nasıl derlenir ? : [https://github.com/dotnet/roslyn/wiki/Building%20Testing%20and%20Debugging](https://github.com/dotnet/roslyn/wiki/Building%20Testing%20and%20Debugging)
- Hangi özellik hangi branch üzerinde geliştiriliyor ? [https://github.com/dotnet/roslyn/blob/master/docs/Language%20Feature%20Status.md](https://github.com/dotnet/roslyn/blob/master/docs/Language%20Feature%20Status.md)

Şimdiden bu opsiyonla uğraşacaklara kolaylıklar diliyorum :) Daha öncede bahsettiğim gibi en kolay opsiyon Visual Studio 15 Preview üzerinden testleri yapmak. 

Bir sonraki yazıda görüşmek üzere.