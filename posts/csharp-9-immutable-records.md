.NET 5.0'in final release'inin ayak seslerinin gittikçe yaklaşmasıyla beraber C#'ın da bir sonra versiyonu olan 9.0'dan yavaş yavaş bahsetmemizin zamanı geldi. Artık bildiğimiz üzere her C# versiyonuyla beraber programlama dilini dizayn eden ekip, topluluğun da katkısıyla bir tema üzerinde yoğunlaşıp bu tema üzerinden özellikleri dile eklemekte. Eski versiyonlardan bu zamana kadar olan versiyonlara bakarsak yazılımcıların işini kolaylaştıracak ufak özelliklerin yanı sıra bir tema etrafında toplanan büyük özelliklerin dile eklendiğini kolayca görebiliriz.(LINQ,Dinamik programlama,async/await,nullable reference types... gibi). 

C# 9.0 ile beraber neler gelecek diye baktığımızda yine ufak tefek kolaylaştırıcı özelliklerin yanı sıra tema olarak bu versiyonda immutable data'nın belirlendiği görüyoruz. Programlama dili tarafından immutable tiplerin implementasyonunun kolaylaştırılması aslında uzun zamandan beri ekibin gündeminde olan bir konuydu. Ancak öncelikler sebebiyle C# 9.0 ile beraber bu tema implemente edilebildi. 

Şimdi gelin öncelikle init-only propertyleri sonra da record kavramını bu yazıda inceleyelim. 

### Init-Only Properties ve Init Accessor

Bugün immutable bir propertyi getter-only-properties özelliğini kullanarak tanımladığımızda bu propertye değer ataması sadece constructor üzerinden yapılabilmekte. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Car</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Brand&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Model&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">Car</span>(<span style="color:blue;">string</span>&nbsp;brand,&nbsp;<span style="color:blue;">string</span>&nbsp;model)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Brand&nbsp;=&nbsp;brand;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Model&nbsp;=&nbsp;model;
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

C# içerisinde bir nesne yaratırken sıklıkla kullandığımız özelliklerden biri object initializerlar. Object initializerlarla hızlı bir şekilde object içerisindeki istediğimiz propertylere istediğimiz değeri atayabiliyoruz. Ancak yukarıdaki gibi property tanımlaması yaptığımızda object initializerları kullanmamız mümkün değil ve constructor yazmak durumunda kalıyoruz. 

C# 9.0 ile beraber gelen init accessor'ı ile beraber propertylerimiz constructorın yanı sıra object initializerlar kullanılarak da initialize edilebilmekte. Bu durumda yukarıdaki Car class'ını şu şekilde güncelleyebiliriz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Car</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Brand&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">init</span>;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Model&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">init</span>;&nbsp;}
}</pre>

Yeni bir nesne yaratmak istersek,

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;car&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Car</span>()
{
&nbsp;&nbsp;&nbsp;&nbsp;Brand&nbsp;=<span style="color:#a31515;">&quot;BMW&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;Model&nbsp;=&nbsp;<span style="color:#a31515;">&quot;2020&quot;</span>
};</pre>

Eğer bu şekilde getter-only propertyleri kullanmak yerine arkada saklanan readonly fieldı kendimiz tanımlayıp, init esnasında da araya girmek istersek bunu yapmamız da mümkün. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Car</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:blue;">readonly</span>&nbsp;<span style="color:blue;">string</span>&nbsp;brand;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">private</span>&nbsp;<span style="color:blue;">readonly</span>&nbsp;<span style="color:blue;">string</span>&nbsp;model;
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Brand
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">get</span>&nbsp;=&gt;&nbsp;brand;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">init</span>&nbsp;=&gt;&nbsp;brand&nbsp;=&nbsp;<span style="color:blue;">value</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Model
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">get</span>&nbsp;=&gt;&nbsp;model;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">init</span>&nbsp;=&gt;&nbsp;model&nbsp;=&nbsp;<span style="color:blue;">value</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

### Uzun Süredir Beklenen Özellik : Recordlar
Eğer daha önce functional programmingle uğraştıysanız o dünyadaki record kavramına  aşinasınızdır.  Recordları içerisinde data barından lightweight classlar olarak düşünebiliriz. Recordların classlardan farkı ise içerisinde bulunan datanın öne çıkması. Örneğin, iki recordı karşılaştırırken bu recordların aynı referansa sahip olması değil aynı dataya sahip olması dikkate alınmakta.

Şimdi gelin yukarıdaki Car classını recordları kullanarak implemente edelim.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">record</span>&nbsp;Car
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Brand&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">init</span>;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Model&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">init</span>;&nbsp;}
}</pre>

Şu anda baktığımızda aslında sadece class keywordu yerine record keywordu geldi gibi duruyor. Peki compiler arka planda record keywordünü görünce neler yapıyor bir de ona bakalım.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Car</span>&nbsp;:&nbsp;<span style="color:#2b91af;">IEquatable</span>&lt;<span style="color:#2b91af;">Car</span>&gt;
{
	<span style="color:blue;">protected</span>&nbsp;<span style="color:blue;">virtual</span>&nbsp;<span style="color:#2b91af;">Type</span>&nbsp;EqualityContract
	{
		[System.Runtime.CompilerServices.NullableContext(1)]
		[CompilerGenerated]
		<span style="color:blue;">get</span>
		{
			<span style="color:blue;">return</span>&nbsp;<span style="color:blue;">typeof</span>(<span style="color:#2b91af;">Car</span>);
		}
	}
 
	<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Brand
	{
		<span style="color:blue;">get</span>;
		<span style="color:blue;">set</span>;
	}
 
	<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Model
	{
		<span style="color:blue;">get</span>;
		<span style="color:blue;">set</span>;
	}
 
	[System.Runtime.CompilerServices.NullableContext(2)]
	<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">bool</span>&nbsp;<span style="color:blue;">operator</span>&nbsp;!=(<span style="color:#2b91af;">Car</span>?&nbsp;r1,&nbsp;<span style="color:#2b91af;">Car</span>?&nbsp;r2)
	{
		<span style="color:blue;">return</span>&nbsp;!(r1&nbsp;==&nbsp;r2);
	}
 
	[System.Runtime.CompilerServices.NullableContext(2)]
	<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">bool</span>&nbsp;<span style="color:blue;">operator</span>&nbsp;==(<span style="color:#2b91af;">Car</span>?&nbsp;r1,&nbsp;<span style="color:#2b91af;">Car</span>?&nbsp;r2)
	{
		<span style="color:blue;">return</span>&nbsp;(<span style="color:blue;">object</span>)r1&nbsp;==&nbsp;r2&nbsp;||&nbsp;(r1?.Equals(r2)&nbsp;??&nbsp;<span style="color:blue;">false</span>);
	}
 
	<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">override</span>&nbsp;<span style="color:blue;">int</span>&nbsp;GetHashCode()
	{
		<span style="color:blue;">return</span>&nbsp;(EqualityComparer&lt;<span style="color:#2b91af;">Type</span>&gt;.Default.GetHashCode(EqualityContract)&nbsp;*&nbsp;-1521134295&nbsp;+&nbsp;EqualityComparer&lt;<span style="color:blue;">string</span>&gt;.Default.GetHashCode(Brand))&nbsp;*&nbsp;-1521134295&nbsp;+&nbsp;EqualityComparer&lt;<span style="color:blue;">string</span>&gt;.Default.GetHashCode(Model);
	}
 
	<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">override</span>&nbsp;<span style="color:blue;">bool</span>&nbsp;Equals(<span style="color:blue;">object</span>?&nbsp;obj)
	{
		<span style="color:blue;">return</span>&nbsp;Equals(obj&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">Car</span>);
	}
 
	<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">virtual</span>&nbsp;<span style="color:blue;">bool</span>&nbsp;Equals(<span style="color:#2b91af;">Car</span>?&nbsp;other)
	{
		<span style="color:blue;">return</span>&nbsp;(<span style="color:blue;">object</span>)other&nbsp;!=&nbsp;<span style="color:blue;">null</span>&nbsp;&amp;&amp;&nbsp;EqualityContract&nbsp;==&nbsp;other!.EqualityContract&nbsp;&amp;&amp;&nbsp;EqualityComparer&lt;<span style="color:blue;">string</span>&gt;.Default.Equals(Brand,&nbsp;other!.Brand)&nbsp;&amp;&amp;&nbsp;EqualityComparer&lt;<span style="color:blue;">string</span>&gt;.Default.Equals(Model,&nbsp;other!.Model);
	}
 
	<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">virtual</span>&nbsp;Car&lt;Clone&gt;$()
	{
		<span style="color:blue;">return</span>&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Car</span>(<span style="color:blue;">this</span>);
}</pre>

Compiler tarafından arka planda yaratılan sınıfa baktığımızda compilerın bir sınıf yaratıp sonra da `IEquatable<T>` interface'ini implemente ettirdiğini görüyoruz. Böylece bu record üzerinden yaratılacak iki nesne karşılaştırıldığında bu nesneler referans olarak değil içerisindeki veriler baz alınarak karşılaştırılmış olacak. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;car&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Car</span>()
{
&nbsp;&nbsp;&nbsp;&nbsp;Brand&nbsp;=&nbsp;<span style="color:#a31515;">&quot;BMW&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;Model&nbsp;=&nbsp;<span style="color:#a31515;">&quot;2020&quot;</span>
};
 
<span style="color:blue;">var</span>&nbsp;car2&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Car</span>()
{
&nbsp;&nbsp;&nbsp;&nbsp;Brand&nbsp;=&nbsp;<span style="color:#a31515;">&quot;BMW&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;Model&nbsp;=&nbsp;<span style="color:#a31515;">&quot;2020&quot;</span>
};
 
<span style="color:#2b91af;">Console</span>.WriteLine(<span style="color:#a31515;">$&quot;IsEqual:</span>{car&nbsp;==&nbsp;car2}<span style="color:#a31515;">&quot;</span>);</pre>

Kodu çalıştırırsak

![](https://ilkayblog.blob.core.windows.net/uploads/2020/08/27/record-equality.png)

### with Expressions

Immutable tiplerle çalışırken nesne üzerinde bir değişiklik yapmak istediğimizde ilgili değişikliği içeren yeni bir nesne yaratmamız gerekiyor. Bunun .NET içerisinde en güzel örneği stringler. String üzerinde değişiklik yapamıyoruz ama değişikliği içeren yeni bir string yaratabiliyoruz.

C# 9.0'dan önce immutable nesne üzerinden değişiklik yaparak yeni bir nesne yaratmak istediğimizde With metotları yazıp bunları kullanarak yeni nesneler yaratabiliyorduk.

Örneğin,

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Car</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Brand&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">init</span>;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Model&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">init</span>;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">Car</span>&nbsp;With(<span style="color:blue;">string</span>&nbsp;brand)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Car</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Brand&nbsp;=&nbsp;brand,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Model&nbsp;=&nbsp;<span style="color:blue;">this</span>.Model
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;};
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>


Kullanımı

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;car&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Car</span>()
{
&nbsp;&nbsp;&nbsp;&nbsp;Brand&nbsp;=&nbsp;<span style="color:#a31515;">&quot;BMW&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;Model&nbsp;=&nbsp;<span style="color:#a31515;">&quot;2020&quot;</span>
};
 
<span style="color:blue;">var</span>&nbsp;newCar&nbsp;=&nbsp;car.With(<span style="color:#a31515;">&quot;Mercedes&quot;</span>);</pre>

Ancak tabi bu metotlari yazmak ve maintain etmek her zaman kolay olmuyor. C# 9.0 ile beraber gelen with expressionslarla With metotları yazmaktan da kurtuluyoruz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;car&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Car</span>()
{
&nbsp;&nbsp;&nbsp;&nbsp;Brand&nbsp;=&nbsp;<span style="color:#a31515;">&quot;BMW&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;Model&nbsp;=&nbsp;<span style="color:#a31515;">&quot;2020&quot;</span>
};
 
<span style="color:blue;">var</span>&nbsp;newCar&nbsp;=&nbsp;car&nbsp;<span style="color:blue;">with</span>&nbsp;{&nbsp;Brand&nbsp;=&nbsp;<span style="color:#a31515;">&quot;Mercedes&quot;</span>&nbsp;};</pre>

### Positional Records

Recordları yukarıdaki gibi object initializerlar ile initialize edebileceğimiz gibi recordlar içerisindeki fieldları sırayla initialize edip aynı zamanda sırayla da deconstruct etmek isteyebiliriz. Bu şekilde davranışlara sahip olan recordlara positional records deniyor. Bir üst kısımda bahsettiğimiz record tipi ise nominal recordlar.

Car recordını positional olarak tanımlamak istersek.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">record</span>&nbsp;Car(<span style="color:blue;">string</span>&nbsp;Brand,&nbsp;<span style="color:blue;">string</span>&nbsp;Model);
</pre>

Recordu positional olarak tanımladıktan sonra artık recordu object initializer yerine constructor ile yaratabilir sonrasında da içerisindeki fieldları tanımlamada belirttiğimiz sırayla deconstruct edebiliriz.

İlk olarak positional recordlarda compiler constructor ve deconstruct metodunu arka planda nasıl yaratıyor ona bakalım. 


<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Car</span>&nbsp;:&nbsp;<span style="color:#2b91af;">IEquatable</span>&lt;<span style="color:#2b91af;">Car</span>&gt;
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">protected</span>&nbsp;<span style="color:blue;">virtual</span>&nbsp;<span style="color:#2b91af;">Type</span>&nbsp;EqualityContract
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[System.Runtime.CompilerServices.NullableContext(1)]
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[CompilerGenerated]
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">get</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:blue;">typeof</span>(<span style="color:#2b91af;">Car</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Brand
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">get</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">set</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Model
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">get</span>;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">set</span>;
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">Car</span>(<span style="color:blue;">string</span>&nbsp;Brand,&nbsp;<span style="color:blue;">string</span>&nbsp;Model)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">this</span>.Brand&nbsp;=&nbsp;Brand;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">this</span>.Model&nbsp;=&nbsp;Model;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">base</span>..ctor();
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;[System.Runtime.CompilerServices.NullableContext(2)]
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">bool</span>&nbsp;<span style="color:blue;">operator</span>&nbsp;!=(<span style="color:#2b91af;">Car</span>?&nbsp;r1,&nbsp;<span style="color:#2b91af;">Car</span>?&nbsp;r2)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;!(r1&nbsp;==&nbsp;r2);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;[System.Runtime.CompilerServices.NullableContext(2)]
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">static</span>&nbsp;<span style="color:blue;">bool</span>&nbsp;<span style="color:blue;">operator</span>&nbsp;==(<span style="color:#2b91af;">Car</span>?&nbsp;r1,&nbsp;<span style="color:#2b91af;">Car</span>?&nbsp;r2)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;(<span style="color:blue;">object</span>)r1&nbsp;==&nbsp;r2&nbsp;||&nbsp;(r1?.Equals(r2)&nbsp;??&nbsp;<span style="color:blue;">false</span>);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">override</span>&nbsp;<span style="color:blue;">int</span>&nbsp;GetHashCode()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;(EqualityComparer&lt;<span style="color:#2b91af;">Type</span>&gt;.Default.GetHashCode(EqualityContract)&nbsp;*&nbsp;-1521134295&nbsp;+&nbsp;EqualityComparer&lt;<span style="color:blue;">string</span>&gt;.Default.GetHashCode(Brand))&nbsp;*&nbsp;-1521134295&nbsp;+&nbsp;EqualityComparer&lt;<span style="color:blue;">string</span>&gt;.Default.GetHashCode(Model);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">override</span>&nbsp;<span style="color:blue;">bool</span>&nbsp;Equals(<span style="color:blue;">object</span>?&nbsp;obj)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;Equals(obj&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">Car</span>);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">virtual</span>&nbsp;<span style="color:blue;">bool</span>&nbsp;Equals(<span style="color:#2b91af;">Car</span>?&nbsp;other)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;(<span style="color:blue;">object</span>)other&nbsp;!=&nbsp;<span style="color:blue;">null</span>&nbsp;&amp;&amp;&nbsp;EqualityContract&nbsp;==&nbsp;other!.EqualityContract&nbsp;&amp;&amp;&nbsp;EqualityComparer&lt;<span style="color:blue;">string</span>&gt;.Default.Equals(Brand,&nbsp;other!.Brand)&nbsp;&amp;&amp;&nbsp;EqualityComparer&lt;<span style="color:blue;">string</span>&gt;.Default.Equals(Model,&nbsp;other!.Model);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">virtual</span>&nbsp;Car&lt;Clone&gt;$()
&nbsp;&nbsp;&nbsp;&nbsp;{
		<span style="color:blue;">return</span>&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Car</span>(<span style="color:blue;">this</span>);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">protected</span>&nbsp;<span style="color:#2b91af;">Car</span>(<span style="color:#2b91af;">Car</span>&nbsp;original)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Brand&nbsp;=&nbsp;original.Brand;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Model&nbsp;=&nbsp;original.Model;
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">void</span>&nbsp;Deconstruct(<span style="color:blue;">out</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Brand,&nbsp;<span style="color:blue;">out</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Model)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Brand&nbsp;=&nbsp;<span style="color:blue;">this</span>.Brand;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Model&nbsp;=&nbsp;<span style="color:blue;">this</span>.Model;
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>


Positional recordları construct ve deconstruct  etmek istersek.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;">
<span style="color:blue;">var</span>&nbsp;car&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Car</span>(<span style="color:#a31515;">&quot;BMW&quot;</span>,&nbsp;<span style="color:#a31515;">&quot;2012&quot;</span>);
 
<span style="color:blue;">var</span>&nbsp;(brand,&nbsp;model)&nbsp;=&nbsp;car;
 
<span style="color:#2b91af;">Console</span>.WriteLine(<span style="color:#a31515;">$&quot;Model:</span>{model}<span style="color:#a31515;">,&nbsp;Brand:</span>{brand}<span style="color:#a31515;">&quot;</span>);</pre>

### Recordlar ve Inheritance

Recordlarda inheritance desteği de bulunmakta. Inheritance desteğinden bahsederken özellikle değinmemiz gereken kritik iki konu bulunmakta. Ama önce ufak bir inheritance örneği yapalım. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">record</span>&nbsp;Car
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Model&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">init</span>;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Brand&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">init</span>;&nbsp;}
}
 
<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">record</span>&nbsp;SportsCar:<span style="color:#2b91af;">Car</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">int</span>&nbsp;MaxPower&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">set</span>;&nbsp;}
}</pre>


<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:#2b91af;">Car</span>&nbsp;car&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Car</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;Brand&nbsp;=&nbsp;<span style="color:#a31515;">&quot;BMW&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;Model&nbsp;=&nbsp;<span style="color:#a31515;">&quot;2012&quot;</span>
};
 
<span style="color:#2b91af;">Car</span>&nbsp;sportsCar&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">SportsCar</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;Brand&nbsp;=&nbsp;<span style="color:#a31515;">&quot;BMW&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;Model&nbsp;=&nbsp;<span style="color:#a31515;">&quot;2020&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;MaxPower&nbsp;=&nbsp;1000
};</pre>

with expressionlarıyla beraber yeni bir `SportsCar` yaratmak istediğimizde compiler arka planda record içerisindeki Clone metodunu kullandığı için otomatik olarak yeni bir sports car nesnesi yaratıp sonra da ilgili değişikliği nesne üzerinde gerçekleştirmekte. Böylece tip bilgisini kaybetmeden yeni nesneyi yaratabiliyor.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:#2b91af;">Car</span>&nbsp;newSportsCar&nbsp;=&nbsp;sportsCar&nbsp;<span style="color:blue;">with</span>&nbsp;{&nbsp;Model&nbsp;=&nbsp;<span style="color:#a31515;">&quot;Ferrari&quot;</span>&nbsp;};
</pre>

Bu kullanım arka planda şu şekle dönüşmekte. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:#2b91af;">Car</span>&nbsp;car2&nbsp;=&nbsp;sportsCar.&lt;Clone&gt;$();
car2.Model&nbsp;=&nbsp;<span style="color:#a31515;">&quot;Ferrari&quot;</span>;</pre>

Recordların arka planda class olarak tanımlanmasından ziyade aslında value davranışları olduğundan bahsetmiştik. İki farklı recordu karşılaştırırken referans karşılaştırması değil içerisindeki valueların karşılaştırılması yapıldığından tekrardan bahsedelim.

Peki yukarıdaki gibi bir inheritance durumunda eşitlik karşılaştırması nasıl yapılacak diye düşünebiliriz. Yukarıdaki Equals implementasyonundan yola çıkarsak Car recordu Brand ve Model propertylerini karşılaştırırken SportsCar recordu ise bu propertylere ek olarak MaxPower propertysini de karşılaştırıyor. Car recordının perspektifinden baktığımızda recordlar aynı ama SportsCar perspektifinden baktığımızda da recordlar farklı gözükebilir. Ancak bu iki farklı karşılaştırmanın önüne geçmek için bir recorddan türetilen diğer recordlar base içerisinde bulunan `virtual protected property olan EqualityContract'ı` override etmek zorunda. Bu durumda `Equals` metodu içerisinde `EqualityContract` karşılaştırılması da olduğu için iki tarafta aynı şekilde karşılaştırma yapabilmekte.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Car</span>&nbsp;:&nbsp;<span style="color:#2b91af;">IEquatable</span>&lt;<span style="color:#2b91af;">Car</span>&gt;
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">protected</span>&nbsp;<span style="color:blue;">virtual</span>&nbsp;<span style="color:#2b91af;">Type</span>&nbsp;EqualityContract
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[System.Runtime.CompilerServices.NullableContext(1)]
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[CompilerGenerated]
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">get</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;<span style="color:blue;">typeof</span>(<span style="color:#2b91af;">Car</span>);
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">override</span>&nbsp;<span style="color:blue;">int</span>&nbsp;GetHashCode()
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;(EqualityComparer&lt;<span style="color:#2b91af;">Type</span>&gt;.Default.GetHashCode(EqualityContract)&nbsp;*&nbsp;-1521134295&nbsp;+&nbsp;EqualityComparer&lt;<span style="color:blue;">string</span>&gt;.Default.GetHashCode(Model))&nbsp;*&nbsp;-1521134295&nbsp;+&nbsp;EqualityComparer&lt;<span style="color:blue;">string</span>&gt;.Default.GetHashCode(Brand);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">override</span>&nbsp;<span style="color:blue;">bool</span>&nbsp;Equals(<span style="color:blue;">object</span>?&nbsp;obj)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;Equals(obj&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">Car</span>);
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">virtual</span>&nbsp;<span style="color:blue;">bool</span>&nbsp;Equals(<span style="color:#2b91af;">Car</span>?&nbsp;other)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">return</span>&nbsp;(<span style="color:blue;">object</span>)other&nbsp;!=&nbsp;<span style="color:blue;">null</span>&nbsp;&amp;&amp;&nbsp;EqualityContract&nbsp;==&nbsp;other!.EqualityContract&nbsp;&amp;&amp;&nbsp;EqualityComparer&lt;<span style="color:blue;">string</span>&gt;.Default.Equals(Model,&nbsp;other!.Model)&nbsp;&amp;&amp;&nbsp;EqualityComparer&lt;<span style="color:blue;">string</span>&gt;.Default.Equals(Brand,&nbsp;other!.Brand);
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">SportsCar</span>&nbsp;:&nbsp;Car,&nbsp;<span style="color:#2b91af;">IEquatable</span>&lt;<span style="color:#2b91af;">SportsCar</span>&gt;
{
	<span style="color:blue;">protected</span>&nbsp;<span style="color:blue;">override</span>&nbsp;<span style="color:#2b91af;">Type</span>&nbsp;EqualityContract
	{
		[System.Runtime.CompilerServices.NullableContext(1)]
		[CompilerGenerated]
		<span style="color:blue;">get</span>
		{
			<span style="color:blue;">return</span>&nbsp;<span style="color:blue;">typeof</span>(<span style="color:#2b91af;">SportsCar</span>);
		}
	}
 
	<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">override</span>&nbsp;<span style="color:blue;">int</span>&nbsp;GetHashCode()
	{
		<span style="color:blue;">return</span>&nbsp;<span style="color:blue;">base</span>.GetHashCode()&nbsp;*&nbsp;-1521134295&nbsp;+&nbsp;EqualityComparer&lt;<span style="color:blue;">int</span>&gt;.Default.GetHashCode(MaxPower);
	}
 
	<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">override</span>&nbsp;<span style="color:blue;">bool</span>&nbsp;Equals(<span style="color:blue;">object</span>?&nbsp;obj)
	{
		<span style="color:blue;">return</span>&nbsp;Equals(obj&nbsp;<span style="color:blue;">as</span>&nbsp;<span style="color:#2b91af;">SportsCar</span>);
	}
 
	<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">sealed</span>&nbsp;<span style="color:blue;">override</span>&nbsp;<span style="color:blue;">bool</span>&nbsp;Equals(Car?&nbsp;other)
	{
		<span style="color:blue;">return</span>&nbsp;Equals((<span style="color:blue;">object</span>?)other);
	}
 
	<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">virtual</span>&nbsp;<span style="color:blue;">bool</span>&nbsp;Equals(<span style="color:#2b91af;">SportsCar</span>?&nbsp;other)
	{
		<span style="color:blue;">return</span>&nbsp;<span style="color:blue;">base</span>.Equals(other)&nbsp;&amp;&amp;&nbsp;EqualityComparer&lt;<span style="color:blue;">int</span>&gt;.Default.Equals(MaxPower,&nbsp;other!.MaxPower);
	}
}</pre>


Yazının başında da değindiğim üzere daha önceden C# içerisinde immutable data ile çalışmayı kolaylaştıracak bazı yenilikler gelse de programlama dili içerisinde tam bir destek söz konusu değildi. Bu versiyonla beraber recordlar dil içerisine eklenmekte ve immutable datayla çalışmak büyük  oranda kolaylaşmakta. Hala daha bazı eksikliklerin olduğunu söyleyebiliriz. İlerleyen versiyonlarda bu eksikliklerde mutlaka giderilecektir. C# 9.0 ile beraber gelen işimizi kolaylaştıracak diğer yenilikler de bir sonraki makalenin konusu olacak :)