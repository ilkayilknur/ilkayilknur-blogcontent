Önceki yazılarımdan <a href="https://www.ilkayilknur.com/net-coreda-json-apilari" target="_blank">birinde</a> .NET Core 3.0 ile beraber gelen yeni `System.Text.Json` API'larından bahsetmiştim. O yazıyı, bu API'ların mevcutta kullandığımız `Newtonsoft.Json` kütüphanesine göre oldukça yeni olduğundan bu nedenle de özellik bakımından arada farklar olabileceğinden, .NET 5.0 ile beraber bu farkın biraz daha kapanacağından bahsederek sonlandırmıştım. Bu yazının konusu da `System.Text.Json` API'larına .NET 5.0 ile beraber gelen yenilikler. Vakit kaybetmeden hızlıca yenilikleri inceleyelim. 

### String'den farklı bir tipte key'e sahip Dictinary desteği

.NET 5.0 öncesinde key tipi stringden farklı olan dictionaryleri serialize & deserialize etmek istediğimizde `NotSupportedException` alıyorduk. .NET 5.0 ile beraber artık bu limitasyon ortadan kalkıyor.

Örnek yaparsak,
<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;dictionary&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Dictionary</span>&lt;<span style="color:blue;">int</span>,&nbsp;<span style="color:blue;">string</span>&gt;()
{
&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;34,&nbsp;<span style="color:#a31515;">&quot;istanbul&quot;</span>&nbsp;},
&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;6,<span style="color:#a31515;">&quot;Ankara&quot;</span>&nbsp;},
&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;1,<span style="color:#a31515;">&quot;Adana&quot;</span>}
};
 
<span style="color:blue;">var</span>&nbsp;json&nbsp;=&nbsp;<span style="color:#2b91af;">JsonSerializer</span>.Serialize(dictionary);
<span style="color:green;">//json:&nbsp;{&quot;34&quot;:&quot;istanbul&quot;,&quot;6&quot;:&quot;Ankara&quot;,&quot;1&quot;:&quot;Adana&quot;}</span>
<span style="color:blue;">var</span>&nbsp;dictionary2&nbsp;=&nbsp;<span style="color:#2b91af;">JsonSerializer</span>.Deserialize&lt;<span style="color:#2b91af;">Dictionary</span>&lt;<span style="color:blue;">int</span>,&nbsp;<span style="color:blue;">string</span>&gt;&gt;(json);
</pre>


###  Fieldlar için serialize & deserialize desteği

.NET 5.0 öncesindeki eksikliklerden biri de fieldların serialize & deserialize edilmesi eksikliğiydi.  .NET 5.0 ile beraber artık fieldlarında serialize & deserialize edilmesini sağlayabiliyoruz. Ancak bu özellik default olarak açık değil. Bu yüzden `JsonSerializerOptions` içerisindeki `IncludeFields` propertysini true'ya çekmek gerekiyor.

Örnek yaparsak,
<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;plaka&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Plaka</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;Kod&nbsp;=&nbsp;1,
&nbsp;&nbsp;&nbsp;&nbsp;Sehir&nbsp;=&nbsp;<span style="color:#a31515;">&quot;Adana&quot;</span>
};
<span style="color:blue;">var</span>&nbsp;options&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">JsonSerializerOptions</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;IncludeFields&nbsp;=&nbsp;<span style="color:blue;">true</span>
};
 
<span style="color:blue;">var</span>&nbsp;json&nbsp;=&nbsp;<span style="color:#2b91af;">JsonSerializer</span>.Serialize(plaka,&nbsp;options);
<span style="color:green;">//json:&nbsp;{&quot;Kod&quot;:1,&quot;Sehir&quot;:&quot;Adana&quot;}</span>
<span style="color:blue;">var</span>&nbsp;plaka2&nbsp;=&nbsp;<span style="color:#2b91af;">JsonSerializer</span>.Deserialize&lt;<span style="color:#2b91af;">Plaka</span>&gt;(json,&nbsp;options);
 
<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Plaka</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">int</span>&nbsp;Kod;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Sehir;
}</pre>

### Immutable class & structlar için serialize & deserialize desteği

Immutable olarak tanımlanan class ve structların serialize & deserialize desteği de yine bir önceki versiyondaki eksikliklerden biriydi. Aşağıdaki gibi bir classımız olduğunu düşünelim. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Plaka</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">int</span>&nbsp;Kod&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Sehir&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">Plaka</span>(<span style="color:blue;">int</span>&nbsp;kod,&nbsp;<span style="color:blue;">string</span>&nbsp;sehir)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Kod&nbsp;=&nbsp;kod;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Sehir&nbsp;=&nbsp;sehir;
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">Plaka</span>()
&nbsp;&nbsp;&nbsp;&nbsp;{
 
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

JsonSerializer bir jsonı deserialize etmek istediğinde parametresiz constructorı kullanacağı için hiçbir zaman doğru bir şekilde deserialize işlemi yapamayacaktı. .NET 5.0 ile beraber gelen `JsonConstructor` attribute'ü ile beraber JsonSerializer'a kullanması gereken constructorı bildirip doğru bir şekilde deserialize etmesini sağlayabiliyoruz. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;plaka&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;<span style="color:#2b91af;">Plaka</span>(1,&nbsp;<span style="color:#a31515;">&quot;Adana&quot;</span>);
 
<span style="color:blue;">var</span>&nbsp;json&nbsp;=&nbsp;<span style="color:#2b91af;">JsonSerializer</span>.Serialize(plaka);
<span style="color:green;">//json:&nbsp;{&quot;Kod&quot;:1,&quot;Sehir&quot;:&quot;Adana&quot;}</span>
<span style="color:blue;">var</span>&nbsp;plaka2&nbsp;=&nbsp;<span style="color:#2b91af;">JsonSerializer</span>.Deserialize&lt;<span style="color:#2b91af;">Plaka</span>&gt;(json);
<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Plaka</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">int</span>&nbsp;Kod&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Sehir&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;[<span style="color:#2b91af;">JsonConstructor</span>]
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">Plaka</span>(<span style="color:blue;">int</span>&nbsp;kod,&nbsp;<span style="color:blue;">string</span>&nbsp;sehir)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Kod&nbsp;=&nbsp;kod;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Sehir&nbsp;=&nbsp;sehir;
&nbsp;&nbsp;&nbsp;&nbsp;}
 
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">Plaka</span>()
&nbsp;&nbsp;&nbsp;&nbsp;{
 
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

Eğer `class` içerisinde sadece bir tane parametreli constructor varsa JsonSerializer `JsonConstructor` attribute'ünü belirtmeye gerek kalmadan otomatik olarak o constructorı kullanır. 

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Plaka</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">int</span>&nbsp;Kod&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Sehir&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:#2b91af;">Plaka</span>(<span style="color:blue;">int</span>&nbsp;kod,&nbsp;<span style="color:blue;">string</span>&nbsp;sehir)
&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Kod&nbsp;=&nbsp;kod;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Sehir&nbsp;=&nbsp;sehir;
&nbsp;&nbsp;&nbsp;&nbsp;}
}</pre>

`Structlarda` ise her zaman parametresiz constructor bulunduğu için structlar için mutlaka yukarıda bahsettiğimiz attribute'ü kullanmak zorundayız.

Peki JsonSerializer parametreli constructor içerisinde hangi parametreye hangi değeri göndereceğini nereden biliyor diye düşünebilirsiniz. JsonSerializer property adını otomatik olarak `camelCase'e` çeviriyor ve constructor içerisinde o parametreyi arıyor. Eğer bulursa ilgili parametreye ilgili değeri gönderiyor. Bulamadığı durumlarda `InvalidOperationException` alabilirsiniz. 

### Reference Handling

.NET 5.0 öncesinde circular referansları yönetmede kullanılan tek yöntem bir `MaxDepth` belirtip, bu limit geçildiğinde exception fırlatılmasıydı. .NET 5.0 ile beraber `JsonSerializerOptions` içerisine eklenen `ReferenceHandling` propertysini `ReferenceHandling.Preserve` set ettiğimizde artık serializer `$id, $values ve $ref` gibi metadata propertylerini de json içerisine yazarak arrayler, collectionlar vb.. gibi yerlerde referans vermek için kullanabilmekte.

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;"><span style="color:blue;">var</span>&nbsp;employee&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;Employee
{
&nbsp;&nbsp;&nbsp;&nbsp;Name&nbsp;=&nbsp;<span style="color:#a31515;">&quot;osman&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;Reports&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;List&lt;Employee&gt;()
};
<span style="color:blue;">var</span>&nbsp;employee2&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;Employee
{
&nbsp;&nbsp;&nbsp;&nbsp;Name&nbsp;=&nbsp;<span style="color:#a31515;">&quot;mehmet&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;Manager&nbsp;=&nbsp;employee
};
<span style="color:blue;">var</span>&nbsp;employee3&nbsp;=&nbsp;<span style="color:blue;">new</span>&nbsp;Employee
{
&nbsp;&nbsp;&nbsp;&nbsp;Name&nbsp;=&nbsp;<span style="color:#a31515;">&quot;ahmet&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;Manager&nbsp;=&nbsp;employee
};
 
employee.Reports.AddRange(<span style="color:blue;">new</span>[]&nbsp;{&nbsp;employee2,&nbsp;employee3&nbsp;});
 
<span style="color:blue;">var</span>&nbsp;json&nbsp;=&nbsp;JsonSerializer.Serialize(employee,&nbsp;<span style="color:blue;">new</span>&nbsp;JsonSerializerOptions
{
&nbsp;&nbsp;&nbsp;&nbsp;ReferenceHandler&nbsp;=&nbsp;ReferenceHandler.Preserve
});
<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">class</span>&nbsp;<span style="color:#2b91af;">Employee</span>
{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;<span style="color:blue;">string</span>&nbsp;Name&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">set</span>;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;Employee&nbsp;Manager&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">set</span>;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:blue;">public</span>&nbsp;List&lt;Employee&gt;&nbsp;Reports&nbsp;{&nbsp;<span style="color:blue;">get</span>;&nbsp;<span style="color:blue;">set</span>;&nbsp;}
}</pre>

JsonSerializer'dan çıkan JSON

<pre style="font-family:Consolas;font-size:20px;color:black;background:white;">{
&nbsp;&nbsp;<span style="color:#2e75b6;">&quot;$id&quot;</span>:&nbsp;<span style="color:#a31515;">&quot;1&quot;</span>,
&nbsp;&nbsp;<span style="color:#2e75b6;">&quot;Name&quot;</span>:&nbsp;<span style="color:#a31515;">&quot;osman&quot;</span>,
&nbsp;&nbsp;<span style="color:#2e75b6;">&quot;Manager&quot;</span>:&nbsp;<span style="color:blue;">null</span>,
&nbsp;&nbsp;<span style="color:#2e75b6;">&quot;Reports&quot;</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2e75b6;">&quot;$id&quot;</span>:&nbsp;<span style="color:#a31515;">&quot;2&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2e75b6;">&quot;$values&quot;</span>:&nbsp;[
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2e75b6;">&quot;$id&quot;</span>:&nbsp;<span style="color:#a31515;">&quot;3&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2e75b6;">&quot;Name&quot;</span>:&nbsp;<span style="color:#a31515;">&quot;mehmet&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2e75b6;">&quot;Manager&quot;</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2e75b6;">&quot;$ref&quot;</span>:&nbsp;<span style="color:#a31515;">&quot;1&quot;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2e75b6;">&quot;Reports&quot;</span>:&nbsp;<span style="color:blue;">null</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2e75b6;">&quot;$id&quot;</span>:&nbsp;<span style="color:#a31515;">&quot;4&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2e75b6;">&quot;Name&quot;</span>:&nbsp;<span style="color:#a31515;">&quot;ahmet&quot;</span>,
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2e75b6;">&quot;Manager&quot;</span>:&nbsp;{
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2e75b6;">&quot;$ref&quot;</span>:&nbsp;<span style="color:#a31515;">&quot;1&quot;</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;},
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<span style="color:#2e75b6;">&quot;Reports&quot;</span>:&nbsp;<span style="color:blue;">null</span>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}
&nbsp;&nbsp;&nbsp;&nbsp;]
&nbsp;&nbsp;}
}</pre>

### NumberHandling

Bazı durumlarda JSON içerisinde number değerler tırnak içerisinde yani string olarak yer alabilmekte. Böyle durumlarda bu alanlar deserialize edilirken int, double gibi değişkenlere deserialize edilemiyordu. .NET 5.0 ile beraber `JsonSerializerOptions` içerisindeki `NumberHandling`  propertysi ile bu alanların serialize ve deserialize edilirkenki davranışlarını kontrol edebiliyoruz. 

`JsonNumberHandling` enumı içerisinde kullanabileceğimiz dört değer bulunmakta. 

| Değer  | Açıklaması      |
|--------|-----------------|
| Strict | Number valuelar json içerisinde number olarak tanımlanan alanlardan okunur ve yazılır.(Tırnak olmadan.) |
| AllowReadingFromString | Number alanlar JSON içerisinde hem number alanlardan hem de string olarak belirtilen alanlardan okunabilir. |
| WriteAsString | Number alanlar JSON string olarak yazılır. |
| AllowNamedFloatingPointLiterals | "NaN", "Infinity", ve "-Infinity" gibi floating point constantları string olarak okunup Single.NaN gibi değerlere set edilebilir. Aynı zamanda bu constant değerler JSON'a serialize edilirken string olarak serialize edilir.  |

### Serialize İşlemi Sırasında Default Değerleri Yoksayma

.NET 5.0 ile beraber default değere sahip olan alanların serialize edilirken JSON içerisinde yer almamasını sağlayabilirsiniz. Bunun için `JsonSerializerOptions` içerisindeki `DefaultIgnoreCondition` propertysini kullanabilirsiniz. Bu propertynin alabileceği değerler ise şu şekilde. 

| Değer  | Açıklaması      |
|--------|-----------------|
| Never | Property her zaman serialize ve deserialize edilir. |
| Always | Property her zaman ignore edilir. |
| WhenWritingDefault | Eğer property default değere eşitse serialize edilirken ignore edilir. |
| WhenWritingNull | Eğer property null ise ignore edilir. Bu alan reference type değişkenler ve propertyler için kullanılır.  |

### HttpClient JSON Extension Metotları

.NET 5.0 ile beraber `HttpClient` içerisine JSON extension metotları geliyor. Metotların ne iş yaptığına zaten çok değinmeyeceğim. İsimlerinden oldukça anlaşılır durumda. Extension metotlar `System.Net.Http.Json` namespace'i içerisinde bulunmakta.

Metotların isimleri şunlar,
* GetFromJsonAsync
* PostAsJsonAsync
* PutAsJsonAsync

### C# 9.0 Record Desteği

C# 9.0 ile beraber gelen <a href="https://ilkayilknur.com/csharp-9-ile-immutable-data-ile-calisma-recordlar-ve-init-only-propertyler" target="_blank">en önemli yeniliklerden</a> biri de `recordlar`. .NET 5.0 ile beraber recordların serialize & deserialize edilmesi de mümkün.

### Performans İyileştirmeleri

.NET 5.0 ile beraber mevcut `JsonSerializer` API'larının performanslarında da iyileştirmeler yapılmış durumda. Özellikle büyük collectionların serialize & deserialize edilmesinde güzel iyileştirmeler bulunmakta. Bunlarla ilgili issueları incelemek için aşağıdaki linkleri kullanabilirsiniz.

* <a href="https://github.com/dotnet/runtime/pull/2259" target="_blank">New serializer converter model for objects and collections</a>
* <a href="https://github.com/dotnet/runtime/issues/36635" target="_blank">Is it possible to optimize JSON serialization any further?</a>
* <a href="https://github.com/dotnet/runtime/pull/35848" target="_blank">Improve deserialization perf for case-insensitive and missing-property cases</a>
* <a href="https://github.com/dotnet/corefx/pull/41845" target="_blank">Use Sse2 instrinsics to make NeedsEscaping check faster for large JSON strings</a>

Bir sonraki yazıda görüşmek üzere


