<a href="https://www.ilkayilknur.com/csharp-9-ile-immutable-data-ile-calisma-recordlar-ve-init-only-propertyler" target="_blank">Bir önceki</a> yazıda C# 9.0'ın teması olan immutable data ile çalışma yeniliklerinden bahsetmiştik. Bu yazıda ise C# 9.0 içerisinde yer alan diğer yeniliklere bakacağız.  Vakit kaybetmeden incelemeye hızlıca başlayalım. 

### Target-typed new expressions

Bir nesne yaratılırken eğer nesne tipi tahmin edilebiliyorsa constructor çağırma öncesinde tip belirtilmesi artık zorunlu olmayacak. Özellikle parametre sayısı fazla olan generic tiplerden nesne yaratırken yazılan uzun kodların önüne geçilebilir.

Örneğin,

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:#2b91af;">List</span>&lt;<span style="color:blue;">string</span>&gt;&nbsp;list&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">List</span>&lt;<span style="color:blue;">string</span>&gt;();
</pre>

Burada nesnenin tipi sol taraftaki tanımlamadan tahmin edilebileceği için aşağıdaki gibi bir kullanım mümkün olacak. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:#2b91af;">List</span>&lt;<span style="color:blue;">string</span>&gt;&nbsp;list&nbsp;=&nbsp;<span style="color:blue;">new</span>();
</pre>

### Parameter null Checking

Metotlara geçilecek olan parametrelerin null kontrolü parametrenin sonuna eklenecek olan ! işaretiyle artık otomatik bir şekilde yapılabilecek.

Örneğin,

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;DoWork(<span style="color:#2b91af;">Person</span>&nbsp;person)
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">if</span>&nbsp;(person&nbsp;==&nbsp;<span style="color:blue;">null</span>)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">throw</span>&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">ArgumentNullException</span>(<span style="color:blue;">nameof</span>(person));
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

Bu şekilde null kontrolü yapmak yerine aşağıdaki gibi tanımlamayla bu işlemi otomatik bir hale getirebiliriz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;DoWork(<span style="color:#2b91af;">Person</span>&nbsp;person!)
{
&nbsp;&nbsp;
}</pre>

### Lambda discard parameters

C# 9.0 ile beraber lambda parametrelerini de discard edebiliyoruz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:#2b91af;">Func</span>&lt;<span style="color:blue;">int</span>,&nbsp;<span style="color:blue;">int</span>,&nbsp;<span style="color:blue;">string</span>&gt;&nbsp;func&nbsp;=&nbsp;(<span style="color:blue;">_</span>,&nbsp;<span style="color:blue;">_</span>)&nbsp;=&gt;&nbsp;<span style="color:blue;">string</span>.Empty;
</pre>

### Static Lambdalar

C# 9.0 ile beraber lambdaları artık statik olarak tanımlayabiliyoruz. Lambdaları statik olarak tanımladığımızda lambdalar lokal değişkenlere, this veya base'e ulaşamadığı için ekstra bir heap allocationa sebep olmuyorlar. Statik olmayan lambdalarda bu durumlar gözden kaçabildiği için statik olarak tanımladığımızda erişim gerçekleştiği durumda compiler derleme hatası vereceği için bu hataların önüne geçmemiz oldukça kolay. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:green;">//Lambda&nbsp;static&nbsp;olarak&nbsp;tanımlandığı&nbsp;için&nbsp;kod&nbsp;derlenirken&nbsp;hata&nbsp;alınacak.</span>
<span style="color:blue;">int</span>&nbsp;x;
DoWork(<span style="color:blue;">static</span>&nbsp;(<span style="color:blue;">_</span>,&nbsp;<span style="color:blue;">_</span>)&nbsp;=&gt;&nbsp;x.ToString());</pre>


### Target-Typed Conditional Expression

Conditional expressionlarda eğer tipler arasında ortak bir tip varsa veya implicit olarak çevrim yapılabiliyorsa compiler artık bu işlemi arka planda otomatik olarak yapıyor ve derleme hatası alınmıyor. 

Aşağıdaki kodlar C# 9.0 öncesinden derleme hatası verirken C# 9.0 ile beraber artık derlenebilmekte.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:#2b91af;">Person</span>&nbsp;p&nbsp;=&nbsp;customer&nbsp;??&nbsp;student;&nbsp;<span style="color:green;">//&nbsp;&nbsp;Ortak&nbsp;base&nbsp;tip</span>
<span style="color:blue;">int</span>?&nbsp;x&nbsp;=&nbsp;person&nbsp;==&nbsp;<span style="color:blue;">null</span>&nbsp;?&nbsp;0&nbsp;:&nbsp;<span style="color:blue;">null</span>;&nbsp;<span style="color:green;">//nullable&nbsp;value&nbsp;type</span></pre>

### Top-level Statements

Başlarda ne gerek vardı dediğim ama kullandıkça bağımlısı olduğum bir özellik :) Bugün basit bir uygulama yazabilmemiz için bile bir Main metot tanımlamamız ve onun içerisine de kodumuzu yazmamız gerekmekte. Basit bir uygulama için bile yazılması gereken boilerplate kod oldukça fazla. C# 9.0 ile beraber artık bu boilerplate koda gerek kalmadan çalışacak kodu yazabiliyoruz. Ana çalışacak kod using ifadeleri ile namespace veya class tanımlaması arasında olmalı. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">using</span>&nbsp;System;
 
<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Program</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Main()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">var</span>&nbsp;person&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Person</span>()
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FirstName&nbsp;=&nbsp;<span style="color:#a31515;">&quot;Ilkay&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;LastName&nbsp;=&nbsp;<span style="color:#a31515;">&quot;Ilknur&quot;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;};
 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2b91af;">Console</span>.WriteLine(<span style="color:#a31515;">$&quot;</span>{person.FirstName}<span style="color:#a31515;">&nbsp;-&nbsp;&nbsp;</span>{person.LastName}<span style="color:#a31515;">&quot;</span>);
&nbsp;&nbsp;&nbsp;&nbsp;}
}
 
<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Person</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;FirstName&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">set</span>;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;LastName&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">set</span>;&nbsp;}
}</pre>

Bu kodu artık C# 9.0 ile beraber şu hale getirebiliriz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">using</span>&nbsp;System;
 
<span style="color:blue;">var</span>&nbsp;person&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Person</span>()
{
&nbsp;&nbsp;&nbsp;&nbsp;FirstName&nbsp;=&nbsp;<span style="color:#a31515;">&quot;Ilkay&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;LastName&nbsp;=&nbsp;<span style="color:#a31515;">&quot;Ilknur&quot;</span>
};
 
<span style="color:#2b91af;">Console</span>.WriteLine(<span style="color:#a31515;">$&quot;</span>{person.FirstName}<span style="color:#a31515;">&nbsp;-&nbsp;&nbsp;</span>{person.LastName}<span style="color:#a31515;">&quot;</span>);
 
<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Person</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;FirstName&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">set</span>;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;LastName&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">set</span>;&nbsp;}
}</pre>

### Covariant returns 

Bu özellikle beraber artık programlama dili override edilen metodun dönüş tipinin daha derived tip olmasına izin vermekte. 

Örneğin,

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">abstract</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Car</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">abstract</span>&nbsp;<span style="color:#2b91af;">Car</span>&nbsp;Do();
}
 
<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">SportsCar</span>&nbsp;:&nbsp;<span style="color:#2b91af;">Car</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">override</span>&nbsp;<span style="color:#2b91af;">SportsCar</span>&nbsp;Do()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

### Pattern Matching Yenilikleri

C# 9.0 ile beraber pattern matching tarafında da iyileştirmeler ve yenilikler bulunmakta. 

#### Type Pattern İyileştirmeleri 

C# 9.0 öncesinde sadece type check yapmak için aynı zamanda identifier tanımlamak veya discard _ kullanmak gerekiyordu. C# 9.0 ile beraber artık tanımlama yapmaya gerek kalmayacak. 

C# 9.0 öncesi

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">switch</span>&nbsp;(person)
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">case</span>&nbsp;<span style="color:#2b91af;">Student</span>&nbsp;s:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DoSomething();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">case</span>&nbsp;<span style="color:#2b91af;">Customer</span>&nbsp;c:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DoSomething2();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
}</pre>

C# 9.0 ile beraber şu şekilde yazabiliriz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">switch</span>&nbsp;(person)
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">case</span>&nbsp;<span style="color:#2b91af;">Student</span>:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DoSomething();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">case</span>&nbsp;Customer:
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DoSomething2();
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">break</span>;
}</pre>

#### Relational Patterns

C# 9.0 ile beraber relational patternler `(<,>,<=,>=)` de dile eklenmis oluyor.

Örneğin,

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">int</span>&nbsp;discount&nbsp;=&nbsp;person.Age&nbsp;<span style="color:blue;">switch</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;&lt;=&nbsp;18&nbsp;=&gt;&nbsp;20,
&nbsp;&nbsp;&nbsp;&nbsp;&lt;=&nbsp;30&nbsp;=&gt;&nbsp;10,
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">_</span>&nbsp;=&gt;&nbsp;0
 
};</pre>

#### Logical Patterns

Pattern combinators da diyebileceğimiz bu yenilikle beraber farklı patternleri `and, or veya not` gibi ifadelerle birleştirebiliyoruz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">int</span>&nbsp;discount&nbsp;=&nbsp;person.Age&nbsp;<span style="color:blue;">switch</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;&lt;=&nbsp;18&nbsp;=&gt;&nbsp;20,
&nbsp;&nbsp;&nbsp;&nbsp;&gt;&nbsp;18&nbsp;<span style="color:blue;">and</span>&nbsp;&lt;=&nbsp;30&nbsp;=&gt;&nbsp;10,
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">_</span>&nbsp;=&gt;&nbsp;0
};
 
<span style="color:blue;">int</span>&nbsp;price;
 
<span style="color:blue;">if</span>&nbsp;(person&nbsp;<span style="color:blue;">is</span>&nbsp;<span style="color:blue;">not</span>&nbsp;<span style="color:#2b91af;">Student</span>)
{
&nbsp;&nbsp;&nbsp;&nbsp;price&nbsp;=&nbsp;200;
}
<span style="color:blue;">else</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;price&nbsp;=&nbsp;100;
}</pre>

### Diğer Yenilikler

C# 9.0 ile beraber gelen diğer yeniliklerin listesi ise şu şekilde. 

*  Partial metotlardaki kısıtlamalar ortadan kaldırılıyor. Böylece code generation yapılan Windows Forms gibi platformlarda developerların bazı senaryolarda otomatik olarak generate edilen kodları editlemesinin önüne geçiliyor : <a href="https://github.com/dotnet/csharplang/blob/master/proposals/csharp-9.0/extending-partial-methods.md" target="_blank">https://github.com/dotnet/csharplang/blob/master/proposals/csharp-9.0/extending-partial-methods.md</a>
* C# 9.0 ile beraber function pointer tanımlamak mümkün olabilecek. Özellikle yüksek performans gerektiren senaryolarda kullanılabilir. <a href="https://github.com/dotnet/csharplang/blob/master/proposals/csharp-9.0/function-pointers.md" target="_blank">https://github.com/dotnet/csharplang/blob/master/proposals/csharp-9.0/function-pointers.md</a>

* Local functionlarda attribute artık kullanmak mümkün olacak. <a href="https://github.com/dotnet/csharplang/blob/master/proposals/csharp-9.0/local-function-attributes.md" target="_blank">https://github.com/dotnet/csharplang/blob/master/proposals/csharp-9.0/local-function-attributes.md</a>

* Module(Assembly) initializationında yaşanan bazı sorunların çözülmesi amacıyla programlama dili tarafında bu özelliğe destek geliyor: Detaylı bilgi için proposalı inceleyebilirsiniz. <a href="https://github.com/dotnet/csharplang/blob/master/proposals/csharp-9.0/module-initializers.md" target="_blank">https://github.com/dotnet/csharplang/blob/master/proposals/csharp-9.0/module-initializers.md</a>

* native-sized signed ve unsigned integer tipleri için dil desteği geliyor. <a href="https://github.com/dotnet/csharplang/blob/master/proposals/csharp-9.0/native-integers.md" target="_blank">https://github.com/dotnet/csharplang/blob/master/proposals/csharp-9.0/native-integers.md</a>

Bir sonraki yazıda görüşmek üzere,