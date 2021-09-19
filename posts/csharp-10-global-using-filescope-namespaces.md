.NET 6'nın ilk release candidate sürümünün çıkmasıyla beraber gerek framework içerisindeki özelliklerin gerekse C# 10 içerisindeki özelliklerin daha stabil ve değişikliğe uğramasının da düşük bir olasılıkta olduğunu düşünebiliriz. Bu nedenle de artık yavaş yavaş C# 10 içerisindeki özellikleri incelemenin de vaktinin geldiğini düşünüyorum. 

Bu yazıda C# 10 içerisinde bulunan ve adapte olması en kolay olan özelliklerden bahsedeceğiz. Vakit kaybetmeden incelemeye başlayalım. 

## Global Usings

Bu özellik kısaca bir proje içerisinde çalışırken sıklıkla kullandığımız namespaceleri global olarak tanımlamamızı sağlıyor. Global olarak tanımladığımız namespaceleri proje içerisindeki tüm dosyalar içerisinden otomatik olarak kullanabiliyoruz.

Kısaca bir örnek verirsek, diyelim ki projemizde Entity Framework kullanıyoruz ve proje içerisindeki tüm dosyalar içerisinde `using Microsoft.EntityFrameworkCore;` şeklinde bir tanımlama yapıyoruz. Bu kod kalabalığından ve tekrardan kurtulmak için `using` statementının başına `global` keywordünü ekleyerek bu namespace'in tüm dosyalarda otomatik importlu şekilde gelmesini sağlayabiliriz.

Global usings özelliğinin kullanımında tavsiye edilen yöntem her proje başına bir `using.cs` gibi bir dosya tanımlanması ve global olarak import edilecek olan namespacelerin bu dosya içerisinden eklenmesi ve yönetilmesi. Aksi takdirde farklı dosyalar içerisinde global tanımlamalar yapıldığında namespacelerin yönetimi oldukça zorlaşabilir.

.NET 6 ile beraber gelen dikkat çekici yeniliklerden biri de uygulama templatelerinin güncelleniyor olması. Bu güncelleme sonrasında farklı uygulama tiplerine göre sık kullanılan bazı namespaceler otomatik olarak global bir şekilde import edilmiş olarak geliyor. SDK tipine göre global import edilen namespaceleri aşağıda görebilirsiniz.

- Microsoft.NET.SDK:
    - System
    - System.Collections.Generic
    - System.IO
    - System.Linq
    - System.Net.Http
    - System.Threading
    - System.Threading.Tasks

- Microsoft.NET.SDK.Web:
    - *Microsoft.NET.SDK içerisinde ekli olanlar ve aşağıdaki namespaceler.*
    - System.Net.Http.Json
    - Microsoft.AspNetCore.Builder
    - Microsoft.AspNetCore.Hosting
    - Microsoft.AspNetCore.Http
    - Microsoft.AspNetCore.Routing
    - Microsoft.Extensions.Configuration
    - Microsoft.Extensions.DependencyInjection
    - Microsoft.Extensions.Hosting
    - Microsoft.Extensions.Logging
- Microsoft.NET.SDK.Worker
    - *Microsoft.NET.SDK içerisinde ekli olanlar ve aşağıdaki namespaceler.*
    - Microsoft.Extensions.Configuration
    - Microsoft.Extensions.DependencyInjection
    - Microsoft.Extensions.Hosting
    - Microsoft.Extensions.Logging
    
Bu otomatik import özelliğinde hoşlanmadıysanız veya global olarak eklenecek olan namespaceleri kendiniz kontrol etmek isterseniz proje dosyası içerisine `<ImplicitUsings>disable</ImplicitUsings>` ekleyebilirsiniz.

## File Scoped Namespaces

C# 10 içerisinde belki de en sevilecek özelliklerden biri olmaya aday olan bir özellik file scoped namespaces. Bu özelliği uzun uzun anlatmaya gerek yok. En basit anlatımla namespace tanımlamalarını kısaltabiliyoruz.

C# 10 öncesi 

```csharp
namespace MyProduct
{
    class X
    {

    }
}
```

C# 10 ile beraber 

```csharp
namespace MyProduct;
class X
{

}
```

Görüldüğü gibi file-scoped namespaceleri kullanarak ekstra curly brace ve indentationdan kurtulmuş oluyoruz. Bu özelliğin tabi ki bazı limitasyonları bulunmakta. Özelliğin adından da anlayabileceğiniz üzere tanımladığımız namespace dosya bazında geçerli. Bu nedenle birden fazla namespace tanımlaması yapmamız mümkün değil. Aynı zamanda bir dosya içerisinde hem file scoped hem de normal namespace kullanımı da yapamıyoruz. Ancak çoğu dosyada zaten bunlara pek ihtiyacımız olmayacağı için bence oldukça iyi bir özellik olduğunu söyleyebilirim. 

Bu yazıda C# 10 ile beraber gelecek olan nispeten daha basit özellikleri inceledik. 

Bir sonraki yazıda görüşmek üzere.


