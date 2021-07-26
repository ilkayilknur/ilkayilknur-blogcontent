.NET 6 ile beraber gelen yeni tiplerden biri de `PriorityQueue<TElement, TPriority>` tipi. `System.Collections.Generic` namespace'i içerisinde bulunan bu tipin aslında uzun bir mazisi var diyebiliriz. İlk olarak Ocak 2015'te yapılan <a href="https://github.com/dotnet/runtime/issues/14032" target="_blank">feature requesti</a> ile başlayan süreç tipin tartışılması, implementasyon detaylarına karar verilmesi, prototiplenmesi derken .NET 6'nın release olması ile beraber son bulacak. Peki priority queue nedir bir de ona bakalım.

Şu ana kadar .NET içerisinde FIFO(`Queue<T>`) ve LIFO(`Stack<T>`) mantığında çalışan tipler bulunmaktaydı. `PriorityQueue<TElement, TPriority>` bu tiplerden farklı olarak içerisinde bulunan itemları geliş zamanlarından farklı olarak başka bir önceliğe göre sıralama imkanı vermekte. Bu tipi kullanarak özel öncelik mantığına sahip producer-consumer yapıları kurabiliyoruz.

*Priority queue internal implementasyonunda quaternary min-heap kullanılmakta. Detaylı bilgi için merak edenler <a href="https://en.wikipedia.org/wiki/D-ary_heap" target="_blank">buraya</a> bakabilir.*


Şimdi basit bir örnek yapıp çalışma mekanizmasına daha yakından bakalım. 

`PriorityQueue<TElement, TPriority>` görüldüğü üzere sizden iki farklı generic parametre tipi istiyor. Bunlardan biri içerisinde saklayacağınız itemların tipi diğeri de öncelik olarak kullanacağınız tip. Basit bir şekilde `string` ve `int` olarak bir kullanım yapalım. 

```csharp
var queue = new PriorityQueue<string, int>();
queue.Enqueue("Dört", 4);
queue.Enqueue("Iki", 2);
queue.Enqueue("Alti", 6);
queue.Enqueue("Bir", 1);

while (queue.TryDequeue(out var element, out var priority))
{
    Console.WriteLine($"Element={element}, Priority={priority}");
}
```
Yukarıdaki kodu çalıştıralım...

```bash
Element=Bir, Priority=1
Element=Iki, Priority=2
Element=Dört, Priority=4
Element=Alti, Priority=6
```

Görüldüğü üzere öncelik değeri olarak ne kadar düşük verirsek o itemın kuyruk içerisindeki önceliği o kadar yüksek oluyor ve ilk olarak o itemı kuyruktan alabiliyoruz. `TPriority` olarak primitive tipler kullanabilmenin yanı sıra kendi yarattığımız kompleks tipleri de kullanmamız mümkün. Bu tipleri kullandığımızda queue yaratma aşamasında parametre olarak bir `IComparer<T>` implementasyonu belirtmemiz gerekir. Aynı zamanda farklı bir önceliklendirme algoritması uygulamak istersek yine özel bir `IComparer<T>` implementasyonu kullanabiliriz.

Basitçe kullanımını gördüğümüz `PriorityQueue<TElement, TPriority>` tipinin API setine de bir bakalım. 

```csharp
namespace System.Collections.Generic
{
    public class PriorityQueue<TElement, TPriority>
    {
        public PriorityQueue();
        public PriorityQueue(IComparer<TPriority>? comparer);
        public PriorityQueue([TupleElementNames(new[] { "Element", "Priority" })] IEnumerable<(TElement Element, TPriority Priority)> items);
        public PriorityQueue(int initialCapacity);
        public PriorityQueue([TupleElementNames(new[] { "Element", "Priority" })] IEnumerable<(TElement Element, TPriority Priority)> items, IComparer<TPriority>? comparer);
        public PriorityQueue(int initialCapacity, IComparer<TPriority>? comparer);

        public PriorityQueue<TElement, TPriority>.UnorderedItemsCollection UnorderedItems { get; }
        public int Count { get; }
        public IComparer<TPriority> Comparer { get; }

        public void Clear();
        public TElement Dequeue();
        public void Enqueue(TElement element, TPriority priority);
        public TElement EnqueueDequeue(TElement element, TPriority priority);
        public void EnqueueRange([TupleElementNames(new[] { "Element", "Priority" })] IEnumerable<(TElement Element, TPriority Priority)> items);
        public void EnqueueRange(IEnumerable<TElement> elements, TPriority priority);
        public int EnsureCapacity(int capacity);
        public TElement Peek();
        public void TrimExcess();
        public bool TryDequeue([MaybeNullWhen(false)] out TElement element, [MaybeNullWhen(false)] out TPriority priority);
        public bool TryPeek([MaybeNullWhen(false)] out TElement element, [MaybeNullWhen(false)] out TPriority priority);

        public sealed class UnorderedItemsCollection : IEnumerable<(TElement Element, TPriority Priority)>, IEnumerable, IReadOnlyCollection<(TElement Element, TPriority Priority)>, ICollection
        {
            public int Count { get; }

            public PriorityQueue<TElement, TPriority>.UnorderedItemsCollection.Enumerator GetEnumerator();

            public struct Enumerator : IEnumerator<(TElement Element, TPriority Priority)>, IEnumerator, IDisposable
            {
                [TupleElementNames(new[] { "Element", "Priority" })]
                public (TElement Element, TPriority Priority) Current { get; }

                public void Dispose();
                public bool MoveNext();
            }
        }
    }
}
```

Bu tipin API setinde dikkat çeken noktalardan biri tuple desteği. Hem kuyruk yaratırken hem de  sonrasında `EnqueueRange` metodunu kullanırken tuple desteği bulunmakta. Bu `System.Collections.Generic` namespace'i içerisindeki tiplerde sık gördüğümüz bir durum değil. 

Bir diğer ilginç metot ise `EnqueueDequeue` metodu. Bu metot kuyruğa yeni bir item ekledikten sonra kuyruktan ilk sıradaki itemı almanızı sağlıyor. Bu operasyon çoğu senaryoda arka arkaya `Enqueue` ve `Dequeue` metotlarını çağırmaktan çok daha etkin.

Bu yazıda C++, Java gibi platformlarda uzun zamandır bulunan ancak .NET 6 ile beraber .NET içerisine eklenecek olan `PriorityQueue<TElement, TPriority>` tipini inceledik. 

Bir sonraki yazıda görüşmek üzere.

Kaynak: <a href="https://github.com/dotnet/runtime/issues/14032" target="_blank">https://github.com/dotnet/runtime/issues/14032</a>


