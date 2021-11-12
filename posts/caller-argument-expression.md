Caller information attributes are one of the useful ways to obtain caller information to a method in C#. C# 10 brings a new member to the caller information attributes.

Before mentioning the new member, let's see why the team had decided to implement it.

Consider we have a `Exists` method with multiple assertions. 

```csharp
bool Exists(string[] array, int index)
{
    Debug.Assert(array != null);
    Debug.Assert(index >= 0);

    return true;
}
```
When we call the `Exists` method with an invalid argument such as a null array, the `Debug.Assert` throws an exception, and the exception stack trace includes file path, line number, and method name. 

```bash
Process terminated. Assertion failed.
   at Program.<<Main>$>g__Exists|0_0(String[] array, Int32 findIndex) in C:\Users\ilkay\source\repos\ConsoleApp34\ConsoleApp34\Program.cs:line 11
   at Program.<Main>$(String[] args) in C:\Users\ilkay\source\repos\ConsoleApp34\ConsoleApp34\Program.cs:line 7

C:\Users\ilkay\source\repos\ConsoleApp34\ConsoleApp34\bin\Debug\net6.0\ConsoleApp34.exe (process 10336) exited with code -2146232797.
Press any key to close this window . . .
```

As you see, the stack trace isn't very helpful, and we need to open the source code and find the related line to see which assertion has failed. However, it would be great to see the failed expression in the exception message.

In C# 10, you can use the new `CallerArgumentExpression` attribute and capture the expression passed to a method. Let's write a simple `Assert` method using the `CallerArgumentExpression` attribute and use it in the `Exists` method. 

```csharp
void Assert(bool condition, [CallerArgumentExpression("condition")] string expression = default)
{
    if (!condition)
    {
        Console.WriteLine($"Condition has failed. Condition={expression}");
    }
}

bool Exists(string[] array, int index)
{
    Assert(array != null);
    Assert(index >= 0);

    return true;
}
```

Calling the `Exists` method with a null array argument will produce the following output in the console.

```bash
Condition has failed. Condition=array != null
```
.NET 6 introduces the new null checking API(`ArgumentNullException.ThrowIfNull`) that uses the `CallerArgumentExpression` to obtain the expression passed to the method.

Source code of the `ThrowIfNull` method.

```csharp
public class ArgumentNullException : ArgumentException
{
    /// <summary>Throws an <see cref="ArgumentNullException"/> if <paramref name="argument"/> is null.</summary>
    /// <param name="argument">The reference type argument to validate as non-null.</param>
    /// <param name="paramName">The name of the parameter with which <paramref name="argument"/> corresponds.</param>
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
}
```

As you see, it is so easy to obtain the expression information passed to a method in C# 10. I think testing and validation libraries will use the `CallerArgumentExpression` attribute to provide more meaningful assertion messages in the coming releases.

See you next time!