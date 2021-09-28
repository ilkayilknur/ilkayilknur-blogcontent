C# 10 ve .NET 6'nın en önemli temalarından biri yazdığımız kodu basitleştirmek ve azaltmak. Bu kapsamda gerek .NET içerisinde gerekse C# içerisinde pek çok yenilik geliyor. Bu yazıdaki konumuz ise metotlardaki null argument kontrolü. Build konferansında C#'ın yeni versiyonu ilk duyurulduğunda beni heyecanlandıran özelliklerden birisi basitleştirilmiş ve yükü compilerın üstlendiği null argument check özelliğiydi.

Kısaca bir null argument kontrolü yaptığımız kod yazarsak.

```csharp
public void Foo(Person person)
{
     if (person == null)
        throw new ArgumentNullException(nameof(person));
}
```

C# 10 ile bu yazdığımız kod aşağıdaki gibi olacaktı. 

```csharp
public void Foo(Person person!)
{
    
}
```

Ancak bu özellik ne yazık ki C# 10 ile beraber gelemeyecek. Bununla beraber .NET 6 ile yazdığımız kodu biraz daha kısaltacak yeni bir metot geliyor. Bu metot `ArgumentNullException.ThrowIfNull` metodu. Bu metodu kullanarak en azından ekstra if statementlarından kurtulabiliriz. 

Yeni gelen metotla beraber yukarıdaki kod aşağıdaki gibi olacak. 

```csharp
public void Foo(Person person)
{
    ArgumentNullException.ThrowIfNull(person);
}
```

`ThrowIfNull` metodunun .NET içerisindeki implementasyonu ise aşağıdaki gibi. 

```csharp
public static void ThrowIfNull([NotNull] object? argument, [CallerArgumentExpression("argument")] string? paramName = null)
{
   if (argument is null)
   {
       Throw(paramName);
   }
}

[DoesNotReturn]
private static void Throw(string? paramName) =>
   throw new ArgumentNullException(paramName);
```
<a href="https://source.dot.net/#System.Private.CoreLib/ArgumentNullException.cs,c742289497493bb8" target="_blank">https://source.dot.net/#System.Private.CoreLib/ArgumentNullException.cs,c742289497493bb8</a>

Yukarıda gördüğünüz `CallerArgumentExpression` attribute'ü size yeni gelebilir. Bu da bir başka makalenin konusu olsun. :upside_down_face: 

Bir sonraki yazıda görüşmek üzere.