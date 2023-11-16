---
title: C# 中 ConfigureAwait 相关答疑 FAQ
tags:
  - .NET
categories:
  - 编程
date: 2020-01-20 16:55:53
---

> 译文：https://www.cnblogs.com/ms27946/p/ConfigureAwait-FAQs-In-CSharp.html  
> 原文：https://devblogs.microsoft.com/dotnet/configureawait-faq/  

<!--more-->

&emsp;&emsp;.NET 加入 async/await 特性已经有 7 年了。这段时间，它蔓延的非常快，广泛；不只在 .NET 生态系统，也出现在其他语言和框架中。在 .NET 中，他见证了许多了改进，利用异步在其他语言结构（additional language constructs）方面，提供了支持异步的 API，在基础设施中标记 async/await 作为最基本的优化（特别是在 .NET Core 的性能和分析能力上）。  
&emsp;&emsp;然而，async/await 另一方面也带来了一个问题，那就是 **ConfigureAwait** 。在这片文章中，我会解答它们。我尝试在这篇文章从头到尾变得更好读，能作为一个友好的答疑清单，能为以后提供参考。

# 什么是 SynchronizationContext
&emsp;&emsp;`System.Threading.SynchronizationContext`[文档](https://docs.microsoft.com/en-us/dotnet/api/system.threading.synchronizationcontext)描述它“它提供一个最基本的功能，在各种同步模型中传递同步上下文”，除此之外并无其他描述。  
&emsp;&emsp;对于它的 99% 的使用案例，`SynchronizationContext`只是提供一个虚拟的`Post`的方法的类，它传递一个委托异步执行（这里面其实还有其他很多虚拟成员变量，但很少用到，并且与我们这次讨论不相关）。这个类的`Post`方法仅仅只是调用`ThreadPool.QueueUserWorkItem`来异步执行前面传递的委托。但是，那些派生类能够覆写`Post`方法，这样就能在大多数合适的地方和时间执行。  
&emsp;&emsp;举个例子，Windows Forms 有一个`SynchronizationContext`[派生类](https://github.com/dotnet/winforms/blob/94ce4a2e52bf5d0d07d3d067297d60c8a17dc6b4/src/System.Windows.Forms/src/System/Windows/Forms/WindowsFormsSynchronizationContext.cs)，它覆写了`Post`方法，这个方法所做的其实就等价于`Control.BeginInvoke`。那就是说所有调用这个`Post`方法都将会引起这个委托在这个控件相关联的线程上被调用，这个线程被称为 “UI 线程”。Windows Forms 依靠 Win32 上的消息处理程序以及还有一个“消息循环”在 UI 线程上运行，它只是简单的等待处理新到达的消息。那些消息可能是鼠标移动和点击，也可能是键盘输入、系统事件，委托以及可调用的委托等。所以为 Windows Forms 应用程序的 UI 线程提供一个`SynchronizationContext`实例，为了让它能够在 UI 线程上执行委托，需要做的就只是简单将委托传递给`Post`。  
&emsp;&emsp;对于 WPF 来说也是如此。它也有它自己的`SynchronizationContext`[派生类](https://github.com/dotnet/wpf/blob/ac9d1b7a6b0ee7c44fd2875a1174b820b3940619/src/Microsoft.DotNet.Wpf/src/WindowsBase/System/Windows/Threading/DispatcherSynchronizationContext.cs)，覆写了`Post`，同样类似的，将传递一个委托给 UI 线程（通过调用 `Dispatcher.BeinInvoke`），在这个例子中是受 WPF Dispatcher 而不是 Windows Forms 控件管理的。  
&emsp;&emsp;对于 Windows 运行时（WinRT）。它同样有自己的`SynchronizationContext`[派生类](https://github.com/dotnet/runtime/blob/60d1224ddd68d8ac0320f439bb60ac1f0e9cdb27/src/libraries/System.Runtime.WindowsRuntime/src/System/Threading/WindowsRuntimeSynchronizationContext.cs)，覆写`Post`，通过`CoreDispatcher`排队委托给 UI 线程。  
&emsp;&emsp;这不仅仅只是“在 UI 线程上运行委托”。任何人都能实现`SynchronizationContext`来覆写`Post`来做任何事。例如，我也许不关心线程运行委托所做的事，但是我想确保所有在我编写的`SynchronizationContext`的方法`Post`都能以一定程度的并发度执行。我可以实现这样一个自定义的`SynchronizationContext`类，像下面一样：
```cs
internal sealed class MaxConcurrencySynchronizationContext: SynchronizationContext
{
    private readonly SemaphoreSlim _semaphore;

    public MaxConcurrencySynchronizationContext(int maxConcurrencyLevel) =>
        _semaphore = new SemaphoreSlim(maxConcurrencyLevel);

    public override void Post(SendOrPostCallback d, object state) =>
        _semaphore.WaitAsync().ContinueWith(delegate
        {
            try { d(state); } finally { _semaphore.Release(); }
        }, default, TaskContinuationOptions.None, TaskScheduler.Default);

    public override void Send(SendOrPostCallback d, object state)
    {
        _semaphore.Wait();
        try { d(state); } finally { _semaphore.Release(); }
    }
}
```

&emsp;&emsp;事实上，单元测试框架 xunit 提供了一个`SynchronizationContext`[派生类](https://github.com/xunit/xunit/blob/d81613bf752bb4b8774e9d4e77b2b62133b0d333/src/xunit.execution/Sdk/MaxConcurrencySyncContext.cs)与上面非常相似，它用来限制与能够并行运行的测试相关的代码量。  
&emsp;&emsp;所有的这些好处就根抽象一样：它提供一个单独的 API，用来根据具体实现的创造者的期望来对委托进行排队处理（it provides a single API that can be used to queue a delegate for handling however the creator of the implementation desires），而不需要知道具体实现的细节。  
&emsp;&emsp;所以，如果我们在编写类库的时候，并且想要进行和执行相同的工作，那么就排队委托给原来位置的“上下文”，那么我就只需要获取这个“同步上下文”，并占有它，然后当完成我的工作时调用这个上下文中的`Post`来传递我想要调用的委托。于 Windows Forms，我不必知道我应该获取一个`Control`并且调用它的`BegeinInvoke`，或者对于 WPF，我不用知道我应该获取一个`Dispatcher`并且调用它的`BeginInvoke`，又或是在 xunit，我应该获取它的上下文并排队传递；我只需要获取当前的`SynchronizationContext`并调用它。为了这个目的，`SynchronizationContext`提供一个`Currenct`属性，为了实现上面说的，我可以像下面这样编写代码：
```cs
public void DoWork(Action worker, Action completion)
{
    SynchronizationContext sc = SynchronizationContext.Current;
    ThreadPool.QueueUserWorkItem(_ => {
                try {
            worker();
        }
        finally {
            sc.Post(_ => completion(), null);
        }
    });
}
```

&emsp;&emsp;框架公开了一个自定义上下文，从`Current`使用了 `SynchronizationContext.SetSynchronizationContext`方法。（A framework that wants to expose a custom context from `Current` uses the `SynchronizationContext.SetSynchronizationContext` method.）

# 什么是 TaskScheduler
&emsp;&emsp;对于“调度器”，`SynchronizationContext`是一个抽象类。并且个别的框架有时候拥有自己的抽象，`System.Threading.Task`也不例外。当任务被那些排队及执行的委托支持（backed）时，它们与`System.Threading.Task.TaskScheduler`相关。就好比`SynchronizationContext`提供一个虚拟的`Post`方法对委托的调用进行排队（稍后实现通过使用典型的委托机制来调用委托），`TaskScheduler`提供一个抽象方法`QueueTask`（稍后通过`ExecuteTask`方法来调用该任务）。  
&emsp;&emsp;默认的调度器会通过`TaskScheduler.Default`返回的是一个线程池，但是可能派生自`TaskScheduler`并相关的方法，来完成以何时何地的调用任务的这个行为。举个例子，核心库包含`System.Threading.Tasks.ConcurrentExclusiveSchedulerPair`类型。这个类的实例暴露了两个`TaskScheduler`属性，一个调用自`ExclusiveScheduler`，另一个调用自`ConcurrentScheduler`。那些被调度到`ConcurrentScheduler`的任务可能是并行运行的，但是在构建它时，会受制于被受限的`ConcurrentExclusiveSchedulerPair`（与前面展示的 *MaxConcurrencySynchronizationContext* 相似），当一个正在运行的任务被调度器调度到`ExclusiveScheduler`时，`ConcurrentScheduler`任务将不会执行，一次只运行一个独立任务。这样的话，它行为就很像一个读写锁。  
&emsp;&emsp;像`SynchronizationContext`，`TaskScheduler`都有一个`Current`属性，它会返回一个 “current”`Taskscheduler`。而不像`SynchronizationContext`，这里不存在方法可以设置当前调度器。相反，当前的调度器是一个与当前正在运行的任务相关，并且这个调度器作为启动任务的一部分提供给给系统。例如下面这个程序将会输出 “True”，与`StartNew`一起使用的 lambda 在`ConcurrentExclusiveSchedulerPair`的`ExclusiveScheduler`方法上调用，并且将会看到`TaskScheduler.Current`被设置（原文：as the lambda used with `StartNew` is executed on the `ConcurrentExclusiveSchedulerPair`'s `ExclusiveScheduler` and will see `TaskScheduler.Current` set to that scheduler）：
```cs
using System;
using System.Threading.Tasks;

class Program {
    static void Main(string[] arg)
    {
        var cesp = new ConcurrentExclusiveSchedulerPair();
        Task.Factory.StartNew(() => {
            Console.WriteLine(TaskScheduler.Current == cesp.ExclusiveScheduler);
        }, default, TaskCreationOption.None, cesp.ExclusiveScheduler).Wait();
    }
}
```

&emsp;&emsp;有趣的是，`TaskScheduler`提供一个静态的方法`FromCurrentSynchronizationContext`，它创建一个新的调度器，那些排队的任务在任意的返回的`SynchronizationContext.Current`都会运行，使用它的`Post`方法为任务进行排队。

# SynchronizationContext 和 TaskScheduler 相关如何等待
&emsp;&emsp;考虑到一个 UI app 使用 Button。一旦点击这个按钮，我们想要从网站下载一个文本，以及设置这个 Button 的文本内容。并且这个 Button 只能被当前的 UI 线程访问，该线程拥有它，所以当我们成功下载新的日期和时间文本，并且想要存储回 Button 的 Content 值，我们只需要做的就是访问该控件所属的线程。如果不这样，我们就会得到这样一个错误：
```
System.InvalidOperationException: 'The calling thread cannot access this object because a different thread owns it.'
```

&emsp;&emsp;如果我们手写出来，我们可以使用前面显示的`SynchronizationContext`设置的`Current`封送回原始上下文，就如`TaskScheduler`
```cs
private static readonly HttpClient s_httpClient = new HttpClient();

private void downloadBtn_Click(object sender, RoutedEventArgs e)
{
    s_httpClient.GetStringAsync("http://example.com/currenttime").ContinueWith(downloadTask =>
    {
        downloadBtn.Content = downloadTask.Result;
    }, TaskScheduler.FromCurrentSynchronizationContext());
}
```

&emsp;&emsp;或者直接使用`SynchronizationContext`：
```cs
private static readonly HttpClient s_httpClient = new HttpClient();

private void downloadBtn_Click(object sender, RoutedEventArgs e)
{
    SynchronizationContext sc = SynchronizationContext.Current;
    s_httpClient.GetStringAsync("http://example.com/currenttime").ContinueWith(downloadTask =>
    {
        sc.Post(delegate
        {
            downloadBtn.Content = downloadTask.Result;
        }, null);
    });
}
```

&emsp;&emsp;这些方法都是显式使用了回调函数。我们应该用`async`/`await`写下面非常自然的代码：
```cs
private static readonly HttpClient s_httpClient = new HttpClient();

private async void downloadBtn_Click(object sender, RoutedEventArgs e)
{
    string text = await s_httpClient.GetStringAsync("http://example.com/currenttime");
    downloadBtn.Content = text;
}
```

&emsp;&emsp;这么做才能成功的在 UI 线程上设置`Content`的值，因为这和上面手动实现的版本一样，在默认情况下，这个正在等待 Task 只会关注`SynchronizationContext.Current`，与`TaskScheduler.Current`一样。在C#中，当你一旦使用`await`，编译器就会转换代码去请求（调用`GetAwaiter`）这个可等待的（在这个例子中就是`Task`）等待者（在例子中说的就是`TaskAwaiter<string>`）（原文：ask the "awaitable" for an "awaiter"）。而等待着的责任就是负责连接（调用）回调函数（经常性的作为一个 “continuation“），当这个等待的对象已经完成的时候，它会在状态机里触发回调，以及只要在回调函数一旦在某个时间点注册，它所做的就是捕捉上下文/调度器。尽管没有用确切的代码（这里有额外的优化和工作上的调整），它看起来就像这样：
```cs
object scheduler = SynchronizationContext.Current;
if (scheduler is null && TaskScheduler.Current != TaskScheduler.Default)
{
    scheduler = TaskScheduler.Current;
}
```

&emsp;&emsp;换句话说，就是首先判断 scheduler 是否有被赋值过，如果没有，那是否还有非默认的`TaskScheduler`。如果有，那么在当准备好调用回调函数的时候，它将使用的是这个捕捉到的调度器；否则它一般调用回调函数作为这个等待的 task 操作完成时的一部分。

# ConfigureAwait(false) 做了什么事
&emsp;&emsp;`ConfigureAwait`方法并没有什么特别的：编译器或者运行时不会以任何特殊的方式识别出它。它只是简单的返回一个结构体（`ConfigureTaskAwaitable`），它包装了原始的 task，被调用时指定了一个布尔值。要记住，`await`能用在任何正确的模式下的任何类。通过返回不同的类型，即当编译器访问`GetAwaiter`方法（是这模式的一部分）返回的实例，它是从`ConfigureAwait`返回的类型，而不是任务 task 直接返回的，并且它提供了一个钩子（hook），这个钩子通过自定义的 awaiter 改变了行为。  
&emsp;&emsp;特别是，不是等待从`ConfigureAwait(continueOnCapturedContext: false)`返回的类型，与其等待 Task，还不如直接在前面显示的逻辑的那样，捕获这个上下文/调度器。上一个展示的逻辑看起来就会像下面一样更加有效：
```cs
object scheduler = null;
if (continueOnCapturedContext)
{
    scheduler = SynchronizationContext.Current;
    if (scheduler is null && TaskScheduler.Current != TaskScheduler.Default)
    {
        scheduler = TaskScheduler.Current;
    }
}
```

&emsp;&emsp;也就是说，通过指定一个 *false* ，即使这里有要回调的当前上下文或调度器，它也会假装没有。

# 为什么我会要用到 ConfigureAwait(false)
&emsp;&emsp;`ConfigureAwait(continueOnCapturedContext: false)`主要用来避免在原始上下文或调度器上强制调用回调。这有以下好处：
&emsp;&emsp;**提高性能**。这里主要的开销就是回调会排队入队列而不仅仅只是调用回调，它们都还要涉及其它额外的工作（比如指定额外的分配），也是因为它在某些我们想要的优化上，在运行时是不能使用的（当我们明确的知道回调函数是如何调用的时候，我们能做更多的优化，但是如果它被随意的传递给一个实现抽象的类，我们有时就会受到限制）。对于每次热路径（hot paths），甚至是检查当前的`SynchronizationContext`以及`TaskScheduler`的所花的额外开销（它们都涉及到访问静态线程），这些都会增加一定量的开销。如果`await`后边的代码实际上在原始上下文中没有长时间运行，使用`ConfigureAwait(false)`就能避免前面提到的所有的开销：它根本不需要入队列，它能运用它所有能优化的点，并且避免不必要的静态线程访问。
&emsp;&emsp;**避免死锁**。有一个库方法，它在网络下载资源，并在其结果上使用`await`。你调用它并且同步阻塞等待结果的返回，比如通过操作返回的`Task`使用`.Wait()`、`.Result`、`.GetAwaiter().GetResult()`。那现在我们来考虑一下，在当前上下文在受操作数量限制运行为 1 时（`SynchronizationContext`），如果你调用它会发生什么，它是否像早前显示的 *MaxConcurrencySynchronizationContext* 那样，又或者是隐含的只有一个线程能使用的上下文，例如 UI 线程。所以你在一个线程上调用方法，然后阻塞它到网络下载任务完成。这个操作会启动网络下载并等待它。因为在默认情况下，这个操作会捕捉当前的同步上下文，之所以它会这么做，是因为当网络下载任务完成之后，它会入队列返回`SynchronizationContext`，回调函数会调用剩余的操作。（原文： it does so, and when the network download completes, it queues back to the `SynchronizationContext` the callback that will invoke the remainder of the operation）。但是只有一个线程能处理这个已经入队列的回调函数，而且就是当前由于你的代码因这个操作等待完成而被阻塞的线程。这个操作除非这个回调函数已被处理，否则是不会完成的。这就发生了死锁！（回调函数相关的线程上下文又被阻塞）这种情况也会发生在没有限制并发，哪怕是 1 的情况，一旦资源以任何方式受到限制的时候也是如此。除了使用 *MaxConcurrencySynchronizationContext* 设置限度为 4，想象一下相同的场景。与其只让其中一个操作调用，我们可以入四个上下文来调用，它们每一个都会调用并阻塞等待它完成。现在我还是阻塞全部的资源，当等待异步访问完成的时候，只有一件事，即如果它们的回调函数能够被完全使用的上下文处理，那么就允许那些异步方法完成。再一次，死锁。

&emsp;&emsp;取而代之的是库方法使用`ConfigureAwait(false)`，那它就不会将回调入队列给原始上下文，这样就避免了死锁的场景。

# 为什么我会要用到 ConfigureAwait(true)
&emsp;&emsp;除非你纯粹是想要表明你明确不会使用`ConfigureAwait(false)`（例如来消除（silence）静态分析警告或类似的警告）而使用它，否则你没必要用到。`ConfigureAwait(true)`没有意义。当去比较`await task`和`await task.ConfigureAwait(true)`时，它们是一样的。如果你在生产代码中看到有`ConfigureAwait(true)`，你可以毫不犹豫的删掉它。  
&emsp;&emsp;`ConfigureAwait`接受一个布尔值，是因为有一些合适的场景，其中你可能想要一个变量来控制配置。但是 99% 的使用案例都是使用硬编码传递一个固定的 *false* 参数，即`ConfigureAwait(false)`。

# 合适应该用 ConfigureAwait(false)
&emsp;&emsp;这取决于：你实现的应用程序代码或是通用目的的库代码？  
&emsp;&emsp;当在编写应用程序时，你一般想要默认行为（它为什么要默认行为）。如果一个app 模型/环境（如 Windows Forms，WPF，ASP.NET Core 等等）发布一个自定义的`SynchronizationContext`，这大部分无疑都有一个好理由：它提供了一种代码方式，它关心同步上下文与 app 模型/环境适当的交互。所以如果你在 Windows Forms 应用程序编写一个事件处理程序，在 xunit 编写一个单元测试，在 ASP.NET MVC 编写一个控制器，无论这个 app 模型实际上是否发布了这个`SynchronizationContext`，如果它存在你就可以想使用它。其意思就是默认情况（即`ConfigureAwait(true)`）。你只需要简单地使用`await`，然后正确的事情就会发生，它维护回调/延续会被传递回原始的上下文，如果它存在。这就回产生一个标准：如果你在应用程序级别的代码，不需要用`ConfigureAwait(false)`。如果你回想下前面的点击事件处理程序的例子，就像下面代码这样：
```cs
private static readonly HttpClient s_httpClient = new HttpClient();

private async void downloadBtn_Click(object sender, RoutedEventArgs e)
{
    string text = await s_httpClient.GetStringAsync("http://example.com/currenttime");
    downloadBtn.Content = text;
}
```

&emsp;&emsp;值设置`downloadBtn.Content = text`它需要返回到原始的上下文。如果代码违反了这个准则，在不该使用`ConfigureAwait(false)`的地方使用了它：
```cs
private static readonly HttpClient s_httpClient = new HttpClient();

private async void downloadBtn_Click(object sender, RoutedEventArgs e)
{
    string text = await s_httpClient.GetStringAsync("http://example.com/currenttime").ConfigureAwait(false); // bug
    downloadBtn.Content = text;
}
```

&emsp;&emsp;这样其结果就是坏行为。这在 ASP.NET 中以来的`HttpContext.Current`也是一样的；使用`ConfigureAwait(false)`并且尝试使用`HttpContext.Current`，可能回导致一些问题。  
&emsp;&emsp;与之比较，通用类库被称为“通用”，一部分原因是因为使用者不关心他们具体使用的环境。你可以在 web app 使用它们，也可以在客户端 app 使用它们，或者是测试，它都不关心，一个类库被用到哪个 app 模型是未知的。变得不可未知就是说它们没准备做任何事，在 app 中以特殊的方式与之交互，例如它不会访问 UI 控件，因为通用类库对你的 UI 控件一无所知。由于我们不会在特定的环境中运行代码，这样我们就能避免强制 continuation / callback 回传给原始上下文，我们做的就是调用`ConfigureAwait(false)`，并且它会带来性能和可靠性的好处。这样就会产生通用的准则：如果你在编写通用类库，那么你就应该使用`ConfigureAwait(false)`。这就是原因，例如，在 .NET Core 运行时类库中，你到处可见（或绝大多数）在使用`ConfigureAwait(false)`的地方使用了`await`；有极少数例外，如果没有的话，那有可能是 bug 被修复了。例如[这个 PR](https://github.com/dotnet/corefx/pull/38610)，它修复了在`HttpClient`中忘记调用`ConfigureAwait(false)`。  
&emsp;&emsp;既然是作为准则，当然也有例外的地方它是没有意义的。举个例子，有一个较大的例外（或者说至少需要考虑的一种情况），在通用类库中，那些需要调用的委托的 api。这种情况，类库调用者要传递可能会被库调用的应用程序级别的代码，这会有效的会使库的那些通用的假设变得毫无意义（In such cases, the caller of the library is passing potentially app-level code to be invoked by the library, which then effectively renders those “general purpose” assumptions of the library moot）。考虑以下例子，一个异步版本的 Linq 的 `Where` 方法如`public static async IAsyncEnumerable<T> WhereAsync(this IAsyncEnumerable<T> source, Func<T,bool> predicate)`这里的 *predicate* 必须要在调用者的原`ConfigureAwait(false)`。  
&emsp;&emsp;这些特殊的例子，通用的标准就是一个非常好的开始点：如果你正在写类库/应用程序级未知的代码，那么请使用`ConfigureAwait(false)`，否则不要使用。

# ConfigureAwait(false) 会保证回调不会在原始上下文运行吗
&emsp;&emsp;不，它保证它不会把回调入队列到原始上下文。但是这并不意味着在代码`await task.ConfiureAwait(false)`后面就不会运行在原始上下文中。那是因为在已经完成的可等待者上等待，它只需要同步地运行`await`，而不用强制到入队列返回。所以你在 await 一个 task，它早就在它等待的时间内完成了，无论你是否使用了`ConfigureAwait(false)`，代码会在之后在当前线程上立即执行，无论这个上下文是否还是当前的。

# 只在方法中只第一次用 await 用 ConfigureAwait(false) 以及剩下的代码不用可以吗
&emsp;&emsp;一般情况下是不行的。见上一个 FAQ。如果这个`await task.ConfigureAwait(false)`涉及到这个 task 在其等待的时间内已经完成了（这种情况极其容易发生），那么`ConfigureAwait(false)`就显得没有意义了，这个线程会继续执行这个异步方法之后的代码，并且与之前具有相同的上下文。  
&emsp;&emsp;一个重要的例外就是，如果你知道第一次 await 总是会异步的完成，并且这个等待的将会调用回调，在一个自定义同步上下问和调度器的自由的环境。举个例子，CryptoStream 是 .NET 运行时类库的类，它确保了密集型计算的代码不会作为同步调用者调用的一部分运行，所以它使用了[自定义的 awaiter](https://github.com/dotnet/runtime/blob/4f9ae42d861fcb4be2fcd5d3d55d5f227d30e723/src/libraries/System.Security.Cryptography.Primitives/src/System/Security/Cryptography/CryptoStream.cs#L205) 来确保所有事情在第一次 await 之后都会运行在线程池线程下。然而，在那个例子中，你将会注意到下个 await 仍然使用了`ConfiureAwait(false)`；在技术上，这是没必要的，但是它会让代码看起来更加容易，否则每次看到这个代码的时候，都不要分析去理解为什么不用`ConfiureAwait(false)`。

# 我能使用 Task.Run 从而避免使用 ConfigureAwait(false) 吗
&emsp;&emsp;对，如果你这么写：
```cs
Task.Run(async delegate
{
    await SomethingAsync(); // 将看不到原始上下文
});
```

&emsp;&emsp;然后在 SomethingAsync() 之后调用`ConfigureAwait(false)`将会是一个空操作，因为这个委托作为参数传递给`Task.Run`，它将在线程池线程上执行，堆栈上没有更高级别的用户代码，如`SynchronizationContext.Current`就会返回 *null* 。尽管如此，`Task.Run`隐含的使用了`TaskScheduler.Default`，它的意思在里边查找 `TaskScheduler.Current`，其委托也会返回`Default`。这意思就是说不管你是否使用了`ConfigureAwait(false)`，它都会展示相同的行为。同时它也不会做任何保证 lambda 里面的代码会执行。如果你有如下代码：
```cs
Task.Run(async delegate
{
    SynchronizationContext.SetSynchronizationContext(new SomeCoolSyncCtx());
    await SomethingAsync(); // will target SomeCoolSyncCtx
});
```

&emsp;&emsp;然后在 SomethingAsync 里面的代码实际上将会看到`SynchronizationContext.Current`实例对象就是 *SomeCoolSyncCtx* ，await和任何没有配置的await，这两者在 SomethingAsync 内都会返回给它。所以为了使用这个方法，你必须要理解你可能正在排队的代码做的所有事情或有可能什么也没做，以及这个操作是否会组织你的操作。  
&emsp;&emsp;这个方法的代价就是需要创建/排队一个额外的任务对象。这对于你的 app 或类库是否重要，取决于你的性能敏感度。  
&emsp;&emsp;还要记住，这些技巧可能会导致更多问题乃至超过它们的价值，并会产生其他意想不到的结果。例如，静态分析工具（如 Roslyn 分析器）已经写了一个去表示等待时它不会使用`ConfigureAwait(false)`，如[CA2007](https://docs.microsoft.com/en-us/visualstudio/code-quality/ca2007?view=vs-2019)。如果你启用了这样一个分析器，随后又使用了一些技巧来避免使用`ConfigureAwait(false)`，那么分析器就会去标记它，并且实际上会为你做更多事。那么如果你之后因为它吵闹（noisiness）又关闭了分析器，最后你会在代码里会丢失你实际上应该要调用`ConfigureAwait(false)`。

# 我能使用 SynchronizationContext.SetSynchronizationContext 来避免使用 ConfigureAwait(false) 吗
&emsp;&emsp;不，好吧，也许吧。它取决于具体设计的代码。

&emsp;&emsp;一些开发者可能会写下面这样的代码：
```cs
Task t;
SynchronizationContext old = SynchronizationContext.Current;
SynchronizationContext.SetSynchronizationContext(null);
try
{
    t = CallCodeThatUsesAwaitAsync(); // 在这里不会看到原始上下文
}
finally { SynchronizationContext.SetSynchronizationContext(old); }
await t; // 仍然会得到原始上下文
```

&emsp;&emsp;我们希望看到在 CallCodeThatUsesAwaitAsync 代码里的当前上下文是 *null* 。并且的确如此。然而，上面代码将不会影响`await TaskScheduler.Current`的等待结果，所以如果代码在自定义的`TaskScheduler`上运行，`await CallCodeThatUsesAwaitAsync`（这里不会使用`ConfigureAwait(false)`）将会看到并排队返回的自定义`TaskScheduler`。  
&emsp;&emsp;这里所有相同的警告同样应用前面的`Task.Run`相关的 FAQ：这里的变通方法有性能的含义，而在 try 中的代码也可以通过设置不同的上下文来组织这些尝试（或者通过非默认的调度器调用代码）。  
&emsp;&emsp;使用这种模式，你需要小心这种细微的差异：
```cs
SynchronizationContext old = SynchronizationContext.Current;
SynchronizationContext.SetSynchronizationContext(null);
try
{
    await t;
}
finally { SynchronizationContext.SetSynchronizationContext(old); }
```

&emsp;&emsp;发现问题了么？这是很难发现但同时又是潜在的问题又很大的。这里它无法保证 await 将会在原始上下文中调用 callback / continuation，就是说重新设置`SynchronizationContext`返回给原始上下文也许不会发生在原始线程，最终的结果就会导致在这个线程的后续工作上会看到错误的上下文（为了解决这个问题，需要编写一个在调用任何用户代码之前通常是要手动重设自定义同步上下文，这是一个良好的应用模式）。即使它发生了在相同的线程上运行，在此之前也需要一段时间，这种上下文这段时间内不会得到适当的修复。但如果它运行在不同的线程上，它最终将在那个线程设置错的上下文。如此等等，这非常不理想。

# 我正使用 GetAwaiter().GetResult() 。我还需要使用 ConfigureAwait(false) 吗
&emsp;&emsp;不，ConfigureAwait 只影响回调。特别是，awaiter 模式要求要求公开一个`IsCompleted`属性，`GetResult`方法以及一个`OnCompleted`方法（作为可选择的，还有方法`UnsafeOnCompleted`）。ConfigureAwait 只影响`{Unsafe}OnCompleted`的行为，所以如果你只是直接调用 awaiter 的`GetResult`方法，无论你是在`TaskAwaiter`或是`ConfiguredTaskAwaitable.ConfiguredTaskAwaiter`做的任何事，这没有任何不同。所以如果你在代码中看到`task.ConfigureAwait(false).GetAwaiter().GetResult()`这样的代码，你可以用`task.GetAwaiter().GetResult()`替换（不过你还是得考虑你是否真的想阻塞它）。

# 我知道我在环境中运行，绝不会用到自定义同步上下文或任务调度器。那我能跳过使用 ConfigureAwait(false) 吗
&emsp;&emsp;也许。它取决于你是如何保证“绝不”的。上一个 FAQ 需要注意的是，因为你正在工作的 app 模型不会设置自定义的同步上下文并且也不会在自定义的任务调度器上调用你的代码，不意味着一些其他的用户或库代码没有这么做。所以你得保证那中情况不会发生，或者至少估量它可能的风险。

# 我听说在 .NET Core 中 ConfigureAwait(false) 已经不在必要了，是真的吗
&emsp;&emsp;不。它还是需要的，当在.NET Core中它与在 .NET Framework 运行需要的理由同样明确。在这方面并没有任何改变。

&emsp;&emsp;但是，改变的是一些环境，这个环境是否发布了它们自己的同步上下文。特别是，在 .NET Framework 的 ASP.NET 类有它自己的[同步上下文](https://github.com/microsoft/referencesource/blob/3b1eaf5203992df69de44c783a3eda37d3d4cd10/System.Web/AspNetSynchronizationContextBase.cs)，而 .NET Core 就没有。那意思就是说，在默认情况下，运行在 .NET Core 的代码是不会看到自定义的同步上下文的，在这样的话，在环境中就大大减少了`ConfigureAwait(false)`的需要。  
&emsp;&emsp;但是，这不意味着永远都不需要自定义的同步上下文或任务调度器。如果一些用户代码（或在你项目中使用的其他类库代码）设置了自定义同步上下文并且调用了你的代码，或在一个被自定义调度器调度的任务中调用了你的代码，那么在 ASP.NET Core 中你的 await 也许就能看到非默认的上下文或调度器，这样就会导致你要使用`ConfigureAwait(false)`。当然，在这种情况下，如果你想避免同步阻塞（无论如何在你的应用程序中都应该这么考虑）并且你不介意细微的性能开销，在这种受限的情况下，你尽可能的不要使用`ConfigureAwait(false)`。

# 当在异步流中使用 await foreach 时，我能使用 ConfigureAwait 吗
&emsp;&emsp;能。具体例子详见 [MSDN Magazine article](https://docs.microsoft.com/en-us/archive/msdn-magazine/2019/november/csharp-iterating-with-async-enumerables-in-csharp-8)。

&emsp;&emsp;`await foreach`绑定了一个模式，它被用来迭代异步流`IAsyncEnumerable`，它也能被用来迭代那些由正确 API 之下（surface area）返回的信息（原文：it can also be used to enumerate something that exposes the right API surface area.）。.NET 运行时库包含了一个`IAsyncEnumerable`的 [ConfigureAwait 拓展方法](https://github.com/dotnet/runtime/blob/91a717450bf5faa44d9295c01f4204dc5010e95c/src/libraries/System.Private.CoreLib/src/System/Threading/Tasks/TaskAsyncEnumerableExtensions.cs#L25-L26)，它返回一个自定义类型，这个类型包装了`IAsyncEnumerable`和一个布尔值。当编译器生成对可枚举的`MoveNextAsync`和`DisposeAsync`方法调用时，那些调用都会返回已配置的可枚举结构类，并且它会以触发配置的方式来执行等待。

# 当 await using 一个 DisposeAsync 对象时，能使用 ConfigureAwait 吗
&emsp;&emsp;可以，尽管有点小麻烦。

&emsp;&emsp;在上个 FAQ 关于`IAsyncEnumerable`的描述，.NET 运行时类库暴露一个`IAsyncDisposable`的拓展方法`ConfigureAwait`，它实现了在以合适的模式下，使用`await using`能很好的工作（即暴露了合适的`DisposeAsync`方法）：
```cs
await using (var c = new MyAsyncDisposableClass().ConfigureAwait(false))
{
    ...
}
```
&emsp;&emsp;这里的问题是，变量 c 现在还不是 MyAsyncDisposableClass 类，而是一个`System.Runtime.CompilerServices.ConfiguredAsyncDisposable`，它是从`IAsyncDisposable`上的拓展方法`ConfigureAwait`返回的类型。

&emsp;&emsp;为了解决这个问题，你需要多写一行：
```cs
var c = new MyAsyncDisposableClass();
await using (c.ConfigureAwait(false))
{
    ...
}
```

&emsp;&emsp;现在这个 c 变量就是 MyAsyncDisposableClass 类型。这对 c 来说也是有影响的，它增加了 c 的范围。如果你介意的话，你可以用大括号把整个都包起来。


# 我已经用了 ConfigureAwait(false) ，但是在 await 后，AsyncLocal 仍然流到了代码中。这是 bug 吗
&emsp;&emsp;不。这是意料之中的事。AsyncLocal 数据流是作为`ExecutionContext`的一部分，它是从`SynchronizationContext`独立出来的。除非你显式的调用`ExecutionContext.SuppressFlow()`来禁止`ExecutionContext`，否则无论你是否使用了`ConfigureAwait`来避免捕捉原始同步上下文，`ExecutionContext`（就是AsyncLocal 数据）都将总会在等待中横穿流动。更多信息详尽[这篇文章](https://devblogs.microsoft.com/pfxteam/executioncontext-vs-synchronizationcontext/)。

# 语言能帮助我在库中避免显式使用 ConfigureAwait(false) 吗
&emsp;&emsp;库作者有时候要表示他们需要使用`ConfigureAwait(false)`的失望，并要求使用侵入式更低的替代方法。  
&emsp;&emsp;目前他们还不需要，至少不需要构建到语言/编译器/运行时内。对于这种情况的解决方案，这里有许多提议，如：
* https://github.com/dotnet/csharplang/issues/645
* https://github.com/dotnet/csharplang/issues/2542
* https://github.com/dotnet/csharplang/issues/2649
* https://github.com/dotnet/csharplang/issues/2746