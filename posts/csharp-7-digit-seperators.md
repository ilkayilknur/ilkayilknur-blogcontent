# C# 7.0 - Digit Separators #

*Not : Bu makaledeki örnekleri denemek için gereken ortam kurulum bilgisini [buradan](http://ilkayilknur.com/csharp-7-ozelliklerini-nasil-test-ederiz) alabilirsiniz.*

C# 7.0 ile beraber gelmesi planlanan ufak özelliklerden biri de digit separatorler. Digit separatorler ile numeric değer tanımlamaları sırasında istediğiniz bölümde basamakları ayırabiliyorsunuz. Böylece kod yazarken tanımlamış olduğunuz numeric değişkenlerin taşıdıkları değerler daha okunabilir olurken hem de siz tanımlama esnasında basamakları ayırabildiğiniz için hata yapma olasılığınız daha düşük oluyor. Digit separator olarak ise _ kullanıyoruz.

Örneğin,

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">int</span>&nbsp;x&nbsp;=&nbsp;1_000_000;<br/><span style="color:blue;">double</span>&nbsp;y&nbsp;=&nbsp;1_00.0_9;</pre>

Digit separatorlerin bir diğer kullanım alanı da [binary literal'lar](http://www.ilkayilknur.com/csharp-7-binary-literals). Binary literal tanımlamaları esnasında da digit separatorleri kullanmamız mümkün.

Örneğin,

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">int</span>&nbsp;x&nbsp;=&nbsp;0b11_00_0_01;<br/><span style="color:blue;">double</span>&nbsp;y&nbsp;=&nbsp;0b1_00;</pre>

Bu ufak özellikle ilgili yazımızda bu kadar. Bir sonraki C# 7.0 özelliğinde görüşmek üzere...
