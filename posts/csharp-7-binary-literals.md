# C# 7.0 - Binary Literals #

*Not : Bu makaledeki örnekleri denemek için gereken ortam kurulum bilgisini [buradan](http://ilkayilknur.com/csharp-7-ozelliklerini-nasil-test-ederiz) alabilirsiniz.*

C# 7.0 ile beraber gelmesi muhtemel özelliklerden biride Binary Literals özelliği. Aslında C# design meetinglerini takip edenler için bu özellik süpriz bir özellik değil. Çünkü aslında aynı özellik C# 6.0'da da planlanıyordu ancak release döneminde bu özellik ne yazık ki C# 6.0'a dahil edilmedi.

İşin magazin tarafını bir kenara bırakırsak :) binary literals ile kod içerisinde binary olarak tanımlamalar yapabiliyorsunuz. Bunu yapmak için binary ifadenin başına **0b** veya **0B** yazmanız yeterli. 

Örneğin,

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Main(<span style="color:blue;">string</span>[]&nbsp;args)<br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">int</span>&nbsp;x&nbsp;=&nbsp;0b1100001;<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">double</span>&nbsp;y&nbsp;=&nbsp;0b100;<br/>}</pre>

Gördüğünüz gibi kullanımı oldukça basit. Gelelim bu özellik nerelerde işimize yarayacak kısmına. Enumlara Flags attribute'ünü ekleyip 2'nin üssü bir biçimde değerler verdiğimizde enumlar üzerinde bitwise operasyonlar yapabiliyoruz. (Enumlar üzerinde bitwise operasyonlar ile ilgili yazmış olduğum yazıyı inceleyebilirsiniz. [http://www.ilkayilknur.com/coklu-enum-degerleriyle-calismak](http://www.ilkayilknur.com/coklu-enum-degerleriyle-calismak)) Burada 2'nin üssü değerleri verirken binary literalleri kullanabiliriz. Böylece hızlı bir şekilde ve daha az hata olasılığıyla hızlı bir şekilde kodumuzu yazabiliriz. 

<pre style="font-family:Consolas;font-size:13;color:black;background:white;">[<span style="color:#2b91af;">Flags</span>]<br/><span style="color:blue;">enum</span>&nbsp;<span style="color:#2b91af;">Colors</span><br/>{<br/>&nbsp;&nbsp;&nbsp;&nbsp;Red&nbsp;=&nbsp;0b1,<br/>&nbsp;&nbsp;&nbsp;&nbsp;Green&nbsp;=&nbsp;0b10,<br/>&nbsp;&nbsp;&nbsp;&nbsp;Blue&nbsp;=&nbsp;0b100<br/>}</pre>

Gördüğünüz gibi binary literallerin bana göre en önemli kullanım alanı burası olacak. Bunun yanında C# ekibi binary ifadeleri öğrenen developerların da C# içerisinde eğitim amaçlı binary literalleri kullanacaklarını düşünüyorlar.

C# 7.0 binary literals özelliğiyle ilgili yazımız bu kadar. Zaten gördüğünüz gibi oldukça ufak bir özellik :)
Bir sonraki yazıda görüşmek üzere...