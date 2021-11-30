
I think code analyzers and fixers are one of the most powerful features in the .NET ecosystem. You can implement your own or use the existing ones to write more reliable and performant code.

In this post, we're going to look at the new code analyzers coming with .NET 6.

*Quick reminder: By default most of the new analyzers are enabled at `Info` level. You can enable these analyzers at `Warning` level by <a href="https://docs.microsoft.com/dotnet/fundamentals/code-analysis/overview#enable-additional-rules" target="_blank">configuring the analysis mode</a> like this: `<AnalysisMode>AllEnabledByDefault</AnalysisMode>`.*

### Do not call Task.WhenAll with a single argument
Instead of calling `Task.WhenAll ` method with a single argument, we can directly await the task.

```csharp
var task = SomeAsyncOperation();
await Task.WhenAll(task);
//Await the task instead of Task.WhenAll
await task;
```

### Do not call Task.WaitAll with a single argument
Instead of calling `Task.WaitAll` method with a single argument, we can use `Task.Wait`.

```csharp
var task = SomeAsyncOperation();
Task.WaitAll(task);
//Use Task.Wait instead of Task.WaitAll
task.Wait();
```

### Use the new Environment.ProcessPath property to get the process path

.NET 6 introduces a new API(`Environment.ProcessPath`) to get the process path. We can use it instead of `Process.GetCurrentProcess().MainModule.FileName`. 

```csharp
var pPath = Process.GetCurrentProcess().MainModule.FileName;
//Use Environment.ProcessPath instead of Process.GetCurrentProcess().MainModule.FileName 
pPath = Environment.ProcessPath;
```

### Use Environment.CurrentManagedThreadId to get the managed thread Id

`Environment.CurrentManagedThreadId` is more compact and efficient replacement of `Thread.CurrentThread.ManagedThreadId`. 

```csharp
var threadId = Thread.CurrentThread.ManagedThreadId;
//Use Environment.CurrentManagedThreadId instead of Thread.CurrentThread.ManagedThreadId  
threadId = Environment.CurrentManagedThreadId;
```

### Use String.Contains(char) instead of String.Contains(String)
It is efficient to replace calls to `String.Contains(String)` method with string arguments containing a single `char`.

```csharp
var testStr = "Test";
var contains = testStr.Contains("t");
//Use String.Contains(char) instead of String.Contains(string)
var contains = testStr.Contains('t');
```
### Replace Dictionary<,>.Keys.Contains calls to ContainsKey
Using `ContainsKey` or `ContainsValue` methods are more performant options when we want to check if a dictionary contains a key or value.

```csharp
var dictionary = new Dictionary<string, string>();
var containsKey = dictionary.Keys.Contains("test");
//Use ContainsKey
containsKey = dictionary.ContainsKey("test");

var containsValue = dictionary.Values.Contains("test");
//Use ContainsKey
containsValue = dictionary.ContainsValue("test");
```

### Use span-based string.Concat to avoid allocations

We can use the span-based `string.Concat` method in some string concatenation patterns and avoid unnecessary allocations. 

```csharp
var testStr = "Test";
var newStr = testStr + testStr.Substring(2) + testStr.Substring(2, 1);
//Use span-based String.Concat
newStr = string.Concat(testStr, testStr.AsSpan(2), testStr.AsSpan(2, 1));
```

### Use string.AsSpan() instead of string.Substring() when parsing

We can use `string.AsSpan` method when calling parsing methods and avoid the allocation caused by the `string.Substring` method.

```csharp
var str = "1234566";
var builder = new StringBuilder();
builder.Append(str.Substring(1));

//Replace Substring with AsSpan
builder.Append(str.AsSpan(1));
```

### Call async methods when in an async method
We can use async APIs in task returning async methods. 

For example,
```csharp
async Task SomeAsyncOperation(FileStream str)
{
    str.Read(buffer, 0, 10);

    //Prefer ReadAsync
    await str.ReadAsync(buffer, 0, 10);
}
```
### Prefer String.Equals instead of String.Compare for equality checks

It is more readable to use `String.Equals` for equality checks instead of calling `String.Compare` and checking the result.

Consider the following code.
```csharp
var str1 = "1234566";
var str2 = "Test";

if(string.Compare(str1,str2) == 0)
{

}
```
We can write the same logic with `string.Compare` method.

```csharp
var str1 = "1234566";
var str2 = "Test";

if(string.Equals(str1, str2))
{

}
```

### var str1 = "1234566";
var str2 = "Test";

if(string.Equals(str1, str2))
{

}

### Warn developers when calling Buffer.BlockCopy

`Buffer.BlockCopy` method expects the number of bytes to be copied for the `count` argument. So passing `Array.Length` could cause unexpected results. 

![](https://ilkayblog.blob.core.windows.net/uploads/2021/11/30/block-copy.png)

### Warn developers when unknown platform names are used

When developers use `SupportedOSPlatform`, `UnsupportedOSPlatform` attributes or `OperatingSystem.IsPlatform()`, `OperatingSystem.IsPlatformVersionAtLeast()` methods, they pass string literals for OS platform names. The new analyzer checks these usages and warns developers if an unknown flatform name is used.

![](https://ilkayblog.blob.core.windows.net/uploads/2021/11/30/unknown-OS-platform.png)

See you next time!