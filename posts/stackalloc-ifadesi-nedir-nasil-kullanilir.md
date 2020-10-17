Bu yazıda kod yazarken belki intellisense pencereseninde adını görüp merak ettiğiniz belki de arada kullandığınız `stackalloc` ifadelerinden bahsedeceğiz. `stackalloc` ifadesi en basit anlatımla stack üzerinde belirtilen kadar bir blok memory allocate etmek için kullanılır. Stack üzerinde allocate edilen bu alan metot sonlanmasıyla beraber otomatik olarak boşa çıkacağı için GC üzerinde de herhangi bir yük oluşturmamakta. Özellikle ufak arraylerle çalıştığımız senaryolarda stackalloc tercih etmek GC üzerindeki yükü hafifletecektir. 

C# 7.2 öncesinde stackalloc ifadelerini kullanmak istediğimizde unsafe kod yazmamız gerekiyordu. Bu da özellikle bu ifadenin daha geniş kapsamlı kullanımını limitlemekteydi. Ancak C# 7.2 ile beraber stackalloc ifadesiyle allocate ettiğimiz alana <a href="https://www.ilkayilknur.com/net-coreda-span-ve-memory-tipleri" target="_blank">Span<T> veya ReadOnlySpan<T></a> kullanarak erişebiliyoruz. 

stackalloc ifadesiyle yarattığımız alanı doğrudan Span veya ReadOnlySpan'e aşağıdaki gibi assign edebiliriz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:#2b91af;">Span</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;span&nbsp;=&nbsp;<span style="color:blue;">stackalloc</span>&nbsp;<span style="color:blue;">int</span>[10];
</pre>

Stackalloc ile bir alan yaratılırken array initializers ile ilk değer ataması da yapabiliyoruz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:#2b91af;">Span</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;span&nbsp;=&nbsp;<span style="color:blue;">stackalloc</span>&nbsp;<span style="color:blue;">int</span>[5]&nbsp;{&nbsp;1,&nbsp;2,&nbsp;3,&nbsp;4,&nbsp;5&nbsp;};
</pre>

Stack üzerinde alan allocate etmek ve bu memory alanına erişmek daha hızlı olsa da tabi ki bazı limitleri bulunmakta. Stack üzerinde çok fazla alan kullanımı `StackOverflowException` hatalarına sebep olabilir. Burada yapılacak en doğru kullanım stack üzerinde yaratılan alan boyutunu limitlemek. Eğer size gereken alan limitlediğiniz alandan fazlaysa gereken alanı heap üzerinde yaratmak. Hem heap üzerinde hem de stack üzerinde yarattığımız alanlara erişimi `Span<T> ve ReadOnlySpan<T>` kullanarak yapabildiğimiz için yazdığımız kod üzerinde de değişiklik yapmamız gerekmemekte. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">const</span>&nbsp;<span style="color:blue;">int</span>&nbsp;MaxStackLimit&nbsp;=&nbsp;1024;
<span style="color:#2b91af;">Span</span>&lt;<span style="color:blue;">byte</span>&gt;&nbsp;buffer&nbsp;=&nbsp;inputLength&nbsp;&lt;=&nbsp;MaxStackLimit&nbsp;?&nbsp;<span style="color:blue;">stackalloc</span>&nbsp;<span style="color:blue;">byte</span>[inputLength]&nbsp;:&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:blue;">byte</span>[inputLength];
</pre>

stackalloc kullanımında yapılan hatalardan biri de döngü içerisinde sürekli olarak stackalloc ile memory allocate etmek. Döngü sayısının arttığı durumlarda bu şekilde bir kullanım yine StackOverflowExceptiona sebep olabilmekte. Bunun yerine ilgili memory alanını döngüye girmeden allocate edip döngü içerisinde bu alanı yeniden kullanmak en optimize ve doğru kullanım. 

StackOverflowException almaya müsait kullanım 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">for</span>&nbsp;(<span style="color:blue;">int</span>&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;&lt;&nbsp;length;&nbsp;i++)
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Span</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;array&nbsp;=&nbsp;<span style="color:blue;">stackalloc</span>&nbsp;<span style="color:blue;">int</span>[length];
}</pre>

Daha optimize kullanım

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:#2b91af;">Span</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;array&nbsp;=&nbsp;<span style="color:blue;">stackalloc</span>&nbsp;<span style="color:blue;">int</span>[length];
<span style="color:blue;">for</span>&nbsp;(<span style="color:blue;">int</span>&nbsp;i&nbsp;=&nbsp;0;&nbsp;i&nbsp;&lt;&nbsp;length;&nbsp;i++)
{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
}</pre>

stackalloc yerine göre kullanıldığında heap üzerinde allocationa neden olmadan hızlı bir şekilde ilgili stack alanını kullanıp metot bitiminde boşa çıkarabilmemizi sağlayan kuvvetli bir araç. Ancak doğru yerde kullanmak ve doğru şekilde kullanmak oldukça önemli. Doğru şekilde kullanılmadığı durumlarda istenmeyen sonuçlarla karşılaşmamız oldukça olası. Örneğin, .NET Runtime içerisinde stackalloc kullanımında 1K byte limiti bulunmakta. Eğer ihtiyacınız olan 1K byte'tan büyükse bu alanı heap üzerinde yaratmanız tavsiye ediliyor. Bu gibi tavsiyelere uyulmadığı veya gözden kaçırıldığı durumda sorunlarla karşılaşmanız çok olası. Örnek durumlardan birine <a href="https://github.com/dotnet/runtime/issues/43286" target="_blank">buradan</a> ulaşabilirsiniz. 

Bir sonraki yazıda görüşmek üzere,