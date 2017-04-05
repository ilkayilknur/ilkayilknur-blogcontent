# C# 7.0 - out variables

C# 7.0 ile beraber gelecek olan ufak ama oldukça kullanışlı özelliklerden biri de out variable'lar. Aslında yazının devamını okuyunca bu özellik daha önce gelmedi mi diye de düşünebilirsiniz. Çünkü bu özelliğin ilk olarak C# 6.0 ile beraber gelmesi planlanıyordu ancak Visual Studip 2015 RC versiyonuna geldiğinde C# ekibi bu feature'ı release'e kadar yetiştiremeyeceğini düşündüğü için C# 6.0'dan çıkardı. Şimdi gelelim out variable'ların kullanımına. 

out keyword'ü bizim bir metottan birden fazla değer döndürmemiz gerektiği durumlarda başvurduğumuz bir keyword. Örneğin, .NET Framework içerisindeki int.TryParse metoduna bakarsak bu metodun boolean bir değer döndürdüğünü görüyoruz. Aynı zamanda bir de out parametre alıyor ve buradan da parse ettiği değeri metodu çağıran yere geri veriyor. 
Bir örnek yaparsak,

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">int</span>&nbsp;result;<br/><span style="color:blue;">string</span>&nbsp;str&nbsp;=&nbsp;<span style="color:#a31515;">&quot;123123&quot;</span>;<br/><span style="color:blue;">bool</span>&nbsp;isSucceeded&nbsp;=&nbsp;<span style="color:blue;">int</span>.TryParse(str,&nbsp;<span style="color:blue;">out</span>&nbsp;result);</pre>
</
out keyword'ünün kullanımının en sıkıntılı yanı out parametresi olarak vereceğimiz değişkeni metodu çağırmadan önce tanımlamış olmamız gerekliliği. Çünkü genelde kod yazarken bu tanımlamaları önceden yapmayı unutuyoruz ve out parametresini gördüğümüz anda bir üst satıra geçip o değişkeni tanımlamak zorunda kalıyoruz. Bu da bizi kod yazarken yavaşlatıyor. Artık C# 7.0 ile beraber bu dikkat dağınıklığı ve kod yazarken yavaşlama durumu sona eriyor ve out parametresi olan bir metodu çağırırken inline olarak out parametreyi tanımlayabiliyoruz. 

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">string</span>&nbsp;str&nbsp;=&nbsp;<span style="color:#a31515;">&quot;123123&quot;</span>;<br/><span style="color:blue;">bool</span>&nbsp;isSucceeded&nbsp;=&nbsp;<span style="color:blue;">int</span>.TryParse(str,&nbsp;<span style="color:blue;">out</span>&nbsp;<span style="color:blue;">int</span>&nbsp;result);</pre>

Bu şekilde gördüğünüz gibi result değişkenini out parametre geçerken tanımlıyoruz ve metot çağırımından sonra da bu değişkeni kullanabiliyoruz. Hatta isterseniz out parametre tanımlarken var keywordünü de kullanbilirsiniz.

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">string</span>&nbsp;str&nbsp;=&nbsp;<span style="color:#a31515;">&quot;123123&quot;</span>;<br/><span style="color:blue;">bool</span>&nbsp;isSucceeded&nbsp;=&nbsp;<span style="color:blue;">int</span>.TryParse(str,&nbsp;<span style="color:blue;">out</span>&nbsp;var&nbsp;result);</pre>


## out Parametreden Kurtulma 

out parametre kabul eden metotları kullanırken belki de sadece metodun dönüş değerini kullanacağımız ve out parametreyi aslında sadece mecbur kaldığımız için tanımladığımız durumlar olabilir. Mesela, int.TryParse metoduna verdiğiniz string'in bir int value'nun string karşılığı olup olmadığını merak ediyor olabilirsiniz. Dolayısıyla out parametresi olarak verdiğimiz değere hiçbir yerde ihtiyacınız olmayacaktır. Bu gibi durumlar için de yine C# 7.0 out variables syntaxını kullanarak out variable tanımlamaktan kurtulabiliriz. Bunun için out variable'ın adını yazmak yerine _ koyarsak out parametre tanımlamaktan kurtulmuş oluyoruz. Arka plandaki gerekli işlemleri derleyici kendisi hallediyor.

<pre style="font-family:Consolas;font-size:13;color:black;background:white;"><span style="color:blue;">string</span>&nbsp;str&nbsp;=&nbsp;<span style="color:#a31515;">&quot;123123&quot;</span>;<br/><span style="color:blue;">if</span>&nbsp;(<span style="color:blue;">int</span>.TryParse(str,&nbsp;<span style="color:blue;">out</span>&nbsp;<span style="color:blue;">var</span>&nbsp;<span style="color:blue;">_</span>))<br/>{<br/> <br/>}</pre>


C# 7.0 içerisindeki belki de en ufak özelliklerden biri out variable'lar. Ancak çözdüğü probleme baktığımızda bu versiyonda en çok kullanılacak olan özelliklerden biri olmaya aday.

Bir sonraki yazıda görüşmek üzere...

