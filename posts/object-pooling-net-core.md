Herkese Selamlar,

Bu yazıda object pooling konusundan bahsedeceğiz. Uygulamalarımızda çalıştığımız bazı tiplerin yaratılması ve kullanıldıktan sonra da destroy edilmesi masraflı olabilmekte. Bu tarzdaki nesnelere çok fazla ihtiyaç duyduğumuz durumlarda da performans problemleriyle karşılaşmamız oldukça mümkün. Object pooling konusu bu gibi sıkıntıların önüne geçebilmemiz için uygulayacağımız yöntemlerden biri. Object pooling türkçe çevirisinden de anlayabileceğimiz üzere bir nesne havuzu :D Bu şekilde masraflı olan nesnelerimizi sürekli olarak yaratıp destroy etmek yerine bu nesneleri yaratıp bir havuzda saklıyoruz. Sonrasında ihtiyaç duyduğumuz durumlarda ise nesneyi yeniden yaratmak yerine havuzdan yaratılmış olan nesneyi alıp kullanıyoruz. Kullanımımız bittikten sonrada nesneyi havuza geri bırakıyoruz. 

Çok basit bir object pool implementasyonu yapmak istersek aşağıdaki gibi bir yapı kurabiliriz. 

```csharp
public class ObjectPool<T>
{
    private readonly ConcurrentBag<T> _objects;
    private readonly Func<T> _objectGenerator;

    public ObjectPool(Func<T> objectGenerator)
    {
        _objectGenerator = objectGenerator ?? throw new ArgumentNullException(nameof(objectGenerator));
        _objects = new ConcurrentBag<T>();
    }

    public T Get() => _objects.TryTake(out T item) ? item : _objectGenerator();

    public void Return(T item) => _objects.Add(item);
}
```

*https://docs.microsoft.com/en-us/dotnet/standard/collections/thread-safe/how-to-create-an-object-pool*

Gördüğümüz üzere object pool'un get ve return olmak üzere 2  metodu var. Get metodu havuzdan ilgili nesneyi almamızı sağlarken return metodu da ilgili nesneyi başka yerlerin de kullanması için havuza geri bırakıyor.

Object pooling .NET Core içerisinde pek çok projede oldukça fazla kullanılmakta. Örneğin ASP.NET Core içerisinde StringBuilder oldukça fazla kullanıldığı içeride bir StringBuilder havuzu tutulmakta bu havuzdan StringBuilderlar kullanılmakta. EF Core tarafında da yine DbContextler için pooling yapmak mümkün. Roslyn projesine de baktığımızda pek çok farklı tip için pooling kullanılmakta. 

* https://github.com/dotnet/roslyn/blob/master/src/Dependencies/PooledObjects/PooledStringBuilder.cs
* https://github.com/dotnet/roslyn/blob/master/src/Dependencies/PooledObjects/PooledDictionary.cs
* https://github.com/dotnet/roslyn/blob/master/src/Dependencies/PooledObjects/PooledDelegates.cs
* https://github.com/dotnet/roslyn/blob/master/src/Dependencies/PooledObjects/PooledHashSet.cs

Gördüğümüz üzere .NET ekosistemi içerisindeki pek çok yerde object pooling kullanılmakta. Dolayısıyla yukarıdaki basit implementasyondan ziyade .NET Core tarafında daha gelişmiş bir object pooling yapısı bulunmakta. `Microsoft.Extensions.ObjectPool` namespace'inde altında bulunan tipler uygulamalarımız içerisinde object pooling kullanabilmemizi sağlayan bir tipler.

`ObjectPool<T>` tipi tüm object poollar için kullanılan bir abstract tip. Object pool yaratmak için providerlardan faydalanıyoruz. Providerlar object poolların yaratılmasından ve pool içerisinde bulunacak maksimum eleman sayısı gibi bazı özelliklerinin belirlenmesinden sorumlu. Biz eğer bu konularda herhangi bir farklılaştırmaya gitmeyeceksek `DefaultObjectPoolProvider` tipini kullanabiliriz. DefaultObjectPoolProvider kullanıldığında pool içerisinde tutulacak nesne sayısı **Environment.ProcessorCount'un 2 katı olmakta.** Pool yaratırken kullanılan kavramlardan biri de policyler. Bu policyler de nesnelerin nasıl yaratılacağını ve poola geri döndürüleceğini belirlemekte. Bunun için de yine `DefaultPooledObjectPolicy`'i kullanabiliriz. Son durumda basit bir object pool yaratmak için şu şekilde bir kod yazabiliriz.

```csharp
public class ExpensiveObject
{
    public ExpensiveObject()
    {

    }
}
class Program
{
    static void Main(string[] args)
    {
        var provider = new DefaultObjectPoolProvider();
        var pool = provider.Create(new DefaultPooledObjectPolicy<ExpensiveObject>());
    }
}
```

Object pooling konusundaki en önemli kısımlardan biri de nesnelerin poola geri dönmeden önce statelerini sıfırlanması. Örneğin bir StringBuilder kullandığımızda eğer StringBuilder'ın içeriğini sıfırlamazsak bizden sonra aynı nesneyi kullanan yerler de önceki kullananların içeriğini StringBuilder içerisinde görür. Yukarıda, tiplerin nasıl yaratılacağından ve poola geri döndürüleceğinden policyler sorumlu demiştik. Bunun için StringBuilder kullanımına örnek olması açısından şu şekilde bir policy yaratabiliriz. 

```csharp
public class StringBuilderPolicy : DefaultPooledObjectPolicy<StringBuilder>
{
    public override bool Return(StringBuilder obj)
    {
        obj.Clear();
        return true;
    }
}
```

.NET Core içerisinde StringBuilderlar için daha optimize edilmiş olan bir `Microsoft.Extensions.ObjectPool.StringBuilderPooledObjectPolicy` bir policy tipi mevcut. StringBuilder için kendiniz yazmak yerine bu tipi kullanmanızı tavsiye ederim. 

### ASP.NET CORE DEPENDENCY INJECTION KULLANIMI 

Object pool tipini ASP.NET Core içerisindeki dependency injection yapısıyla kullanmamız mümkün. Bunun için şu şekilde DI containera ekleyebiliriz. 

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<ObjectPoolProvider, DefaultObjectPoolProvider>();
    services.AddSingleton(t =>
    {
        var provider = t.GetService<ObjectPoolProvider>();
        return provider.Create<ExpensiveObject>();
    });

    services.AddControllersWithViews();
}
```
Sonrasında da controller içerisine yada middlewarelar içerisinde object poolu kullanabiliriz. 

```csharp
public class HomeController : Controller
{
    private readonly ObjectPool<ExpensiveObject> pool;

    public HomeController(ObjectPool<ExpensiveObject> pool)
    {
        this.pool = pool;
    }

    public IActionResult Index()
    {
        return View();
    }
}
```

Gördüğünüz üzere object pooling gerekli durumlarda kolayca yapabileceğimiz bir optimizasyon. Tabi ki her durumda object pooling kullanmak mantıklı değil. Özellikle sık kullandığımız ve yaratılması,destroy edilmesi masraflı tipler için kullanmak mantıklı. Aksi takdirde bu implementasyon sisteme ekstra bir yük getirmekten başka bir işe yaramayacaktır. Bu nedenle her optimizasyonda olduğu gibi gerekli benchmark testlerini yaparak ilerlemek en doğrusu olacaktır. 

Bir sonraki yazıda görüşmek üzere