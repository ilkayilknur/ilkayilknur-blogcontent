##Azure Websites Always On Özelliği

Web sitelerinizi Azure websites üzerinde host etmeye başladığınızda web sitenizin açılma süresinin zaman zaman uzadığını gözlemlemeniz hiç de süpriz bir durum değil. Bunun nedeni web sitenizin belirli bir süre istek almadığından dolayı Azure tarafından unload edilmesi. Site unload edildikten sonra gelecek olan ilk istekte web sitesi tekrardan initialize edildiğinden dolayı web sitesinin cevap vermesi de haliyle uzun sürmekte.

Azure'un perpespektifinden baktığımızda ise sitenizin belirli bir sürede hiç istek almadığı durumda kaynakların verimli kullanılması açısından sitenin unload edilmesi doğru bir hareket olarak görünse de aslında bizler için durum hiç de öyle değil. Sonuçta web sitelerimizin her zaman beklediğimiz gibi hızlı bir şekilde yanıt vermesini isteriz. Bu nedenle sitenizin Azure tarafından unload edilmesinden kaçınmak için yapabileceğiniz tek bir şey var. O da web sitenizin belirli aralıklarla istek alarak sürekli olarak ayakta kalması.

Azure websites tarafında bu mekanizmayı otomatik olarak sağlayan **Always-On** özelliğini açarak web sitenizin Azure tarafından belirli sürelerle pinglenmesini sağlayabilirsiniz. Böylece siteniz belirli aralıklarla istek aldığından dolayı Azure tarafından unload edilmeyecek ve websitenizin yanıt vermesi süresi ara ara da olsa artmayacaktır.

Şimdi gelelim Azure websites tarafında always-on özelliğini nasıl aktifleştireceğimize. Öncelikli olarak belirtmem gerekirse always-on özelliği şu an sadece **basic ve standart** modda kullanılabiliyor. Eğer web siteniz free veya shared modda ise web sitenizi belirttiğim modlara yükseltebilir veya aşağıda bahsettiğim azure scheduler servisi ile de aynı mekanizmayı kurabilirsiniz. Tabi scheduler kullandığınızda scheduler için ekstra para vereceğinizi aklınızda bulundurmanızda fayda var. Always-on özelliğini kullanmanın ise size getirdiği ekstra bir masraf yok.

Always-on özelliğini devreye almak için portalden web sitenizi seçip **Configure** sekmesine geçtikten sonra always-on özelliğini aktifleştirebilirsiniz.

![](http://az718566.vo.msecnd.net/uploads/2015/Alwayson.gif)

Websiteniz basic veya standart modda ise yapmanız gereken bu kadar basit. Eğer değilse azure scheduler servisini kullanarak web sitenize belirli aralıklarla istekte bulunmamız da mümkün. Bunun için aşağıdaki gibi yeni bir azure scheduler jobı yaratarak ilerleyebilirsiniz.

![](http://az718566.vo.msecnd.net/uploads/2015/scheduler.gif)

Umarım faydalı olmuştur.