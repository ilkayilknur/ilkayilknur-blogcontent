C# 5.0 ile beraber gelen async/await keywordleri ile beraber .NET içerisinde oldukça kolay bir şekilde asenkron programlama yapabilmekteyiz. Normal durumda asenkron operasyonları yönetmek için oldukça fazla ve karmaşık callback kodu yazmamız gerekirken programlama dili pek çok işlemi bizim yerimize arka planda optimize bir şekilde halletmekte. async/await keywordleri pek çok işlemi arka planda gizlese de bu keywordleri doğru bir şekilde kullanmak da oldukça önemli. 

.NET içerisinde asenkron programlama deyince async/await dışında akla gelen şeylerden biri de `Task` tipi. Task tipi en basit anlatımla asenkron operasyonu temsil eden bir tip. Bu tipi  kullanarak asenkron operasyonla ilgili tüm detaylara erişebilmemiz mümkün. Son zamanlarda ise .NET Core içerisinde bazı asenkron operasyonların dönüş değerinin `ValueTask` olduğunu görmekteyiz. Bu yazımızın konusu da ValueTask kullanımı. Şimdi gelin ilk olarak ValueTask'ın çıkış noktasından başlayalım.

Bugün asenkron bir metot yazmak istediğimizde aşağıdaki şekilde yazabiliyoruz.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">async</span>&nbsp;<span style="color:#2b91af;">Task</span>&lt;<span style="color:#2b91af;">List</span>&lt;<span style="color:#2b91af;">User</span>&gt;&gt;&nbsp;GetAsync()
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;users&nbsp;=&nbsp;<span style="color:blue;">await</span>&nbsp;GetUsersAsync();
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;ConvertToUsersList(users);
}</pre>

Bunun yanında asenkron metotların bazı durumlarda senkron çalışabilmesi gibi bir durum da olabilmekte. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">async</span>&nbsp;<span style="color:#2b91af;">Task</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;GetCountAsync()
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(cache==0)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cache&nbsp;=&nbsp;<span style="color:blue;">await</span>&nbsp;CountAsync();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;cache;
}</pre>

Yukarıdaki gibi metot yazdığımızda metot sadece cache variable'ı 0 olduğu durumda asenkron çalışmakta. Diğer tüm durumlarda data zaten hazır olduğu için metot senkron olarak çalışıp yanıtı dönmekte. Bu gibi durumlarda metot senkron olarak çalışmasına rağmen arka planda sürekli olarak yeni bir Task tipi üretilmekte. Task tipi bir class olduğu için heapte sürekli olarak Task üretilmesi GC üzerine de ekstra bir yük getirmekte. Özellikle hot path dediğimiz noktalarda bu şekilde büyük oranda senkron çalışan metotların kullanılması arka planda yaratılan Taskların sayısını da oldukça arttıracaktır. Runtime bazı durumlarda arkada yaratılan Taskları cachelese de her durum için cacheleme yapması mümkün olmamakta.

Yukarıda bahsettiğimiz gibi durumların önüne geçmek için .NET Core içerisine ValueTask tipi eklendi. ValueTask bir struct olduğu için senkron çalışma ve başarılı sonuçlanma durumlarında allocationa neden olmamakta. ValueTask kullanmak için metodun dönüş tipini ValueTask olarak değiştirmemiz yeterli. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">async</span>&nbsp;<span style="color:#2b91af;">ValueTask</span>&lt;<span style="color:blue;">int</span>&gt;&nbsp;GetCountAsync()
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>(cache==0)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cache&nbsp;=&nbsp;<span style="color:blue;">await</span>&nbsp;CountAsync();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;cache;
}</pre>

### ValueTask Kısıtlamaları

ValueTask tipi Task tipinin allocationa neden olmayan bir alternatifi gibi gözüksede Task tipine nazaran önemli kullanım  kısıtlamaları bulundurmakta. Öncelikle ValueTask'ı gördüğümüzde tek yapmamız gereken şey sadece bir kez `await` edip kullanmak ve o ValueTask'a bir daha dokunmamak. Birden fazla kez await etmek, `WhenAll, WhenAny` gibi combinator metotlar içerisinde kullanmak gibi ihtiyaçlarınız varsa ValueTask bunun için doğru bir tercih değil. Eğer bu gibi kullanımlarda bulunmak isterseniz `AsTask()` extension metodunu kullanıp Task nesnesi üzerinden bu işlemleri gerçekleştirmeniz gerekmekte.

Doğru ve kaçınmamız gereken kullanımlar şu şekilde. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">async</span>&nbsp;<span style="color:#2b91af;">Task</span>&lt;<span style="color:#2b91af;">List</span>&lt;<span style="color:#2b91af;">User</span>&gt;&gt;&nbsp;GetAsync()
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//Dogru</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;count&nbsp;=&nbsp;<span style="color:blue;">await</span>&nbsp;GetCountAsync();
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//Yanlış</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;valTask&nbsp;=&nbsp;GetCountAsync();
&nbsp;&nbsp;&nbsp;&nbsp;count&nbsp;=&nbsp;<span style="color:blue;">await</span>&nbsp;valTask;
&nbsp;&nbsp;&nbsp;&nbsp;count&nbsp;=&nbsp;<span style="color:blue;">await</span>&nbsp;valTask;
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:green;">//Yanlış</span>
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;valTask2&nbsp;=&nbsp;GetCountAsync();
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;task&nbsp;=&nbsp;valTask.AsTask();
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;task2&nbsp;=&nbsp;valTask.AsTask();
}</pre>

Peki artık Task tipini bırakıp doğrudan ValueTask mı kullanmalıyız? Bunun genel olarak cevabı hayır. Çünkü Task tipi şu anda pek çok operasyonu destekleyen bir tip. Asenkron operasyonların çok farklı şekilde yönetiminde kolayca kullanacağımız bir tip. ValueTask tipi çok özel senaryolarda kullanımı avantaj getiren bir tip. Bunun için öncelikle gerekli performans ölçümlerini yaptıktan sonra kullanmakta fayda var. Özellikle ValueTask döndüğümüz metotların nasıl kullanılacağı da oldukça önemli. Desteklenmeyecek şekilde bir kullanımda bulunulacaksa ValueTask yerine Task kullanıp ve bu taskları da cachelenebilecekse cachelemek daha doğru bir çözüm olacaktır. Bu nedenle ölçüm yapıp kazanım getirdiğinden emin olduktan sonra değişimleri yapmakta fayda var. 

Bu konu ilginizi çektiyse bu yazıda kaynak olarak kullandığım <a href="https://devblogs.microsoft.com/dotnet/understanding-the-whys-whats-and-whens-of-valuetask/" target="_blank">Understanding the Whys, Whats, and Whens of ValueTask</a> makalesini okuyabilirsiniz. 