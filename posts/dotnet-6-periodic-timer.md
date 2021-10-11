As a developer, you've probably used timers in .NET before. There are plenty of types of timers in .NET right now, and each of them serves a different purpose.

List of types of timers in .NET 5.

* `System.Threading.Timer`
* `System.Timers.Timer`
* `System.Windows.Forms.Timer`
* `System.Web.UI.Timer`
* `System.Windows.Threading.DispatcherTimer`

.NET 6 introduces a new timer type called `PeriodicTimer`. The main purpose behind the `PeriodicTimer` is to avoid using callbacks. Avoiding callbacks can save us from dealing with memory leaks that might occur in long-lived operations, and we can write `async` code instead of using the `sync over async` approach in callbacks. Another problem that you might deal with with the current timer types is callback overlapping. If you don't write code for the callback overlap scenario, you can see unexpected behaviors in your application.

Creating a `PeriodicTimer` instance is pretty straightforward. The only parameter you need to provide is the `period` value.

```csharp
var timer = new PeriodicTimer(TimeSpan.FromSeconds(10));
```

You can call the `WaitForNextTickAsync` method to wait asynchronously between ticks and, write asynchronous code without dealing with callback overlapping problems because the `PeriodicTimer` pauses while the business code is running and resumes when the `WaitForNextTickAsync` method is called.

```csharp
var timer = new PeriodicTimer(TimeSpan.FromSeconds(10));

while (await timer.WaitForNextTickAsync())
{
    //Business logic
}
```
You can stop `PeriodicTimer` using the `CancellationToken` that you can specify while calling the `WaitForNextTickAsync` method. Another way to stop the `PeriodicTimer` is calling the `Dispose` method. When the `Dispose` method is called, `WaitForNextTickAsync` returns `false`, and you can stop executing the timer-based business logic.

Some notes about `PeriodicTimer`
* The execution context isn't captured.
* `PeriodicTimer` is intended to be used only by a single consumer at a time.

PeriodicTimer documentation: <a href="https://docs.microsoft.com/en-us/dotnet/api/system.threading.periodictimer" target="_blank">https://docs.microsoft.com/en-us/dotnet/api/system.threading.periodictimer</a>