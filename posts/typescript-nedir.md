## Typescript Nedir ? ##

Evet bu yazımızın konusu "*Typescript nedir ?*". Typescript yeni bir programlama dili değil belki ancak bu yazıda istedim ki bilmeyenler için kısaca bir özet geçelim, bilenler ve kullananlar için de bilgi tazelemesi olsun.

Son yıllarda eminim ki Javascriptin önlenemez bir şekilde yükselişini farketmişinizdir. Javascript artık sadece browserda istemci tarafında işler yapmak için kullandığımız bir dil olmaktan çok daha farklı bir yerde. [NodeJS](https://nodejs.org/en/) gibi platformlarla artık Javascript serverda da çalışabiliyor. İstemci tarafında çalışan büyük masaüstü uygulamaları geliştirmek de artık Javascript ile mümkün. Bakınız [electron.js](https://github.com/atom/electron).  

Gördüğünüz gibi aslında başlarda sadece browserda çalışan bir programlama dili olarak tasarlanan Javascript zaman geçtikçe çok daha farklı ve tahmin edilemez bir noktaya ulaştı. Javascript'in ilk tasarlandığı zamanlarda büyük ölçekli uygulamalar geliştirmek için kullanılması çok da planlanmadığından aslında içerisinde büyük ölçekli uygulamalar geliştirirken bize oldukça yardımcı olan classlar, modüller, interfaceler vb.. yapıları da içerisinde bulundurmuyordu. Bu nedenle büyük ölçekli uygulamaları Javascript ile geliştirmek ve geliştirdikten sonra bu uygulamaların bakımını yapmak çokta kolay değil.

Javascript programlama dili aynı zamanda dinamik bir dil. Dinamik olması ona büyük bir güç katarken aynı zamanda development esnasında da bize bir o kadar zorluk çıkarıyor. Kod içerisindeki hataları bulmamız ve kodun bakımını yapmamız zorlaşıyor. Statik programlama dillerinin(C#,Java vb..) development esnasında bize sağladığı güzellikleri (refactoring, find references vb..) düşünürsek Javascript doğası gereği bu yeteneklerden de mahrum kalıyor. Eğer 100-200 satırlı bir kod yazıyorsanız bu eksiklikler size çok kritik gelmese de 10000-20000 satırlık bir uygulama yazıyorsanız emin olabilirsiniz ki bu özellikler size büyük zaman kazandıracaktır.

![](http://az718566.vo.msecnd.net/uploads/2016/01/TSlogo.png)

İşte Typescript'in amacı da aslında yukarıda bahsettiğim Javascript'in sıkıntılı yanlarından bizi kurtarak büyük ölçekli uygulamalar geliştirmemizi sağlamak. Bu nedenle Typescript aslında Javascript'in bir üst kümesi olarak konumlanıyor. Yani Javascript'in sahip olduğunu yapıların daha fazlasını içerisinde barındırıyor. Ayrıca Typescript statik bir dil olduğu için statik programlama dillerinin sahip olduğu yeteneklere de sahip oluyorsunuz.

Typescript kodu derlendiğinde çıktı olarak Javascript kodu üretiliyor. Yani Typescript içerisinde kullandığımız enum, interface gibi yapılar geçerli bir Javascript koduna çevriliyor. Kodumuzun düzenini sağlamak için kullandığımız bazı yapılar ise(örneğin interfaceler) Javascripte bile çevrilmiyor. Onlar sadece derleme işlemi sırasında Typescript tarafından kullanılıyor. Böylece temiz bir Javascript çıktısı elimizde bulunuyor. Ayrıca Typescript sadece derleme esnasında görev aldığı için çalışma zamanında Typescript'ten doğabilecek bir performans sorunu da söz konusu değil. Çünkü Typescript kodu derlenip Javascript kodu üretildiğinde Typescript'in işi burada bitiyor. Bunun yanında dışarıya çıkan kod tamamen Javascript olduğu için Typescript kullandığınız için herhangi ekstra bir runtime veya kütüphane vs.. kullanmanız gerekmiyor.

Bu kadar teorik bilgi sanırım yeterli. Şimdi istiyorsanız kısaca bir Typescript'e göz atalım. Örnek yapabilmek için hızlıca [Typescript Playground'u](http://www.typescriptlang.org/Playground) kullanacağız. Typescript Playground, online olarak Typescript'i denememizi ve örnek yapabilmemizi sağlayan bir ortam. Sol tarafa Typescript olarak yazdığımız kodun Javascript'e çevrilmiş halini anında sağ tarafta görebiliyoruz. Unutmadan bahsediyim Typescript Playground da Typescript kullanılarak geliştirilmiş.

Typescript'in Javascript'in bir üst kümesi olduğundan bahsetmiştik. Yani yazdığımız Javascript kodları aynı zamanda geçerli bir Typescript kodu. Dolayısıyla Typescript içerisinde alışık olduğunuz gibi Javascript yazabilirsiniz veya elinizde daha önceden yazılmış Javascript kodları varsa bunları Typescript içerisinde kullanabilirsiniz. Eğer aşağıdaki kodu playground'a kopyalarsanız Typescript'in kodu başarılı bir şekilde derlediğini göreceksiniz.

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;button&nbsp;=&nbsp;document.createElement(<span style="color:#a31515;">&#39;button&#39;</span>);<br/>button.textContent&nbsp;=&nbsp;<span style="color:#a31515;">&quot;Hello&quot;</span>;<br/>button.onclick&nbsp;=&nbsp;<span style="color:blue;">function</span>&nbsp;()&nbsp;{<br/>&nbsp;&nbsp;&nbsp;&nbsp;alert(<span style="color:#a31515;">&quot;Hello&nbsp;Typescript&nbsp;!&quot;</span>);<br/>};</pre>

![](http://az718566.vo.msecnd.net/uploads/2016/01/TSJSConversion.png)

Örneği biraz daha geliştirelim. Örneğin Typescript içerisinde OOP dillerinde olduğu gibi classlar kullanabiliyoruz. 

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">class</span>&nbsp;Hello&nbsp;{<br/>&nbsp;&nbsp;&nbsp;&nbsp;WriteHelloWorld()&nbsp;{<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;alert(<span style="color:#a31515;">&quot;Hello&nbsp;World&quot;</span>);<br/>&nbsp;&nbsp;&nbsp;&nbsp;}<br/>}<br/> <br/><span style="color:blue;">var</span>&nbsp;hello&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;Hello();<br/>hello.WriteHelloWorld();</pre>

Typescript tarafından da bu kod aşağıdaki gibi Javascript'e çevriliyor. Dolayısıyla biz classlar üzerinden operasyonlarımızı daha kolay bir şekilde yönetirken Typescript işin Javascript tarafıyla kendisi ilgileniyor.

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;Hello&nbsp;=&nbsp;(<span style="color:blue;">function</span>&nbsp;()&nbsp;{<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">function</span>&nbsp;Hello()&nbsp;{<br/>&nbsp;&nbsp;&nbsp;&nbsp;}<br/>&nbsp;&nbsp;&nbsp;&nbsp;Hello.prototype.WriteHelloWorld&nbsp;=&nbsp;<span style="color:blue;">function</span>&nbsp;()&nbsp;{<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;alert(<span style="color:#a31515;">&quot;Hello&nbsp;World&quot;</span>);<br/>&nbsp;&nbsp;&nbsp;&nbsp;};<br/>&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;Hello;<br/>})();<br/><span style="color:blue;">var</span>&nbsp;hello&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;Hello();<br/>hello.WriteHelloWorld();<br/></pre>

Typescript sahip olduğu statik programlama yetenekleri nedeniyle kod yazarken bize tanımladığımız değişkenlerle ilgili tip bilgisi verebiliyor. Aynı zamanda hatalı bir kod yazdığımızda da bu hatayı runtime sırasında değil kodu yazma esnasında alıyoruz. Belki de bu Typescriptin sağladığı en güzel fayda diyebiliriz.

![](https://ilkayblog.blob.core.windows.net/uploads/2016/01/TypescripStatic.gif)

Gördüğünüz gibi Typescript Javascript kodu yazmak istediğimizde işimizi oldukça kolaylaştırıyor. Bu yazı yazıldığı sıralarda Typescript 1.7 versiyonunda ve hızla 2.0 versiyonuna doğru ilerliyor. Pek çok büyük projede şu an Typescript etkin bir şekilde kullanılıyor. Mesela [Angular 2](http://blogs.msdn.com/b/typescript/archive/2015/03/05/angular-2-0-built-on-typescript.aspx) projesi şu an Typescript kullanılarak geliştiriliyor. Yani geleceği oldukça parlak bir programlama dili. Eğer şimdiye kadar hiç duymadıysanız veya kullanmadıysanız bir gözatmanızda fayda var derim. İleride eminim bir şekilde karşınıza çıkacaktır. ;)

Bir sonraki yazıda görüşmek üzere.
