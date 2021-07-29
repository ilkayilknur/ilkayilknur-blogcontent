Developerlar olarak sıklıkla kullandığımız tiplerden biri de `Random` tipi. Bazen test amaçlı olarak bazen de farklı amaçlarla `Random` tipine sıklıkla başvuruyoruz. `Random` tipinin özelliklerinden biri de thread-safe olmaması. Bir `Random` instance'ının concurrent olarak kullanılması nadiren de olsa üretilen sayıların **0** olmasına bazen de randomizasyonun kalitesinin düşmesine neden olabilmekte. 

Bu noktada kullanabileceğimiz çözümler `lock` kullanmak veya her bir thread içerisinde `ThreadStatic` field içerisinde farklı bir `Random` instance'ı kullanmak olabilir. Ancak her iki durumda da developerlar `Random` tipini thread-safe bir şekilde kullanmak için ekstra kod yazmak durumunda kalmakta. Bu senaryo .NET 6 ile beraber biraz değişiyor. .NET 6 ile beraber gelen `Random.Shared` propertysini kullanarak thread-safe olarak `Random` tipini kullanabiliyoruz. 

Örnek olarak

```csharp
var x = Random.Shared.Next();
```
Bu şekilde kullanımda arkada kullanılan implementasyon `ThreadStatic` field içerisinde <a href="https://source.dot.net/#System.Private.CoreLib/Random.cs,235" target="_blank">saklandığı</a> için concurrent kullanımlarda herhangi bir sorunla karşılaşmıyoruz ve aynı zamanda kolay ve basit bir şekilde thread-safe implementasyona da sahip olmuş oluyoruz.

Bir sonraki yazıda görüşmek üzere.

Kaynak: <a href="https://github.com/dotnet/runtime/issues/43887" target="_blank">https://github.com/dotnet/runtime/issues/43887</a>