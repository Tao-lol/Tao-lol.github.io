---
title: 使用 .NET Core 创建 Windows 服务
tags:
  - .NET
categories:
  - 编程
date: 2019-12-14 20:50:51
---

> 作者：Dotnet Core Tutorials  
> 原文：
> * Part 1 - The “Microsoft” Way: https://dotnetcoretutorials.com/2019/09/19/creating-windows-services-in-net-core-part-1-the-microsoft-way/  
> * Part 2 - The “Topshelf” Way: https://dotnetcoretutorials.com/2019/09/27/creating-windows-services-in-net-core-part-2-the-topshelf-way/  
> * Part 3 – The “.NET Core Worker” Way: https://dotnetcoretutorials.com/2019/12/07/creating-windows-services-in-net-core-part-3-the-net-core-worker-way/
> 
> 译者：Lamond Lu  
> 译文：
> * Part 1 - 使用官方推荐方式: https://www.cnblogs.com/lwqlun/p/11621186.html  
> * Part 2 - 使用 Topshelf 方式: https://www.cnblogs.com/lwqlun/p/11625789.html  
> * Part 3 - 使用 .NET Core 工作器方式: https://www.cnblogs.com/lwqlun/p/12038062.html  


<!--more-->

![ ](65831-20191005211343706-2050001309.png)

# 使用官方推荐方式
&emsp;&emsp;创建 Windows 服务来运行批处理任务或者运行后台任务，是一种非常常见的模式，但是由于云服务（Amazon Lambda, Azure WebJobs 以及 Azure Functions）的激增，你可能不会经常使用 Windows 服务了。个人而言，我非常喜欢使用 Azure WebJobs，因为我可以直接编写一个控制台程序，而不需要考虑如何云中运行它，一个批处理文件可以将其装换成一个自动化任务，并且可以保证 7 * 24 小时的运行。  
&emsp;&emsp;但是也许你还没有使用云服务，或者你有一堆要作为 Windows 服务运行的旧版应用程序需要转换为 .NET Core，但是不能完全将他们转换为 “ 无服务器 ”（serverless）应用。 那么这边文章就是适合你的。  
&emsp;&emsp;在许多方面，.NET Core 中的 Windows 服务和 .NET Framework 中的 Windows 服务完全相同。但是，在编写服务的时候，你可能会遇到一些小问题。此外，本文中，我们仅介绍 “ Microsoft ” 方式的 Windows 服务创建，在后续，我会继续介绍如何使用第三方库 TopShelf 来简化这个过程。  

## 安装
&emsp;&emsp;由于 Visual Studio 没有提供创建 Windows 服务的模板，所以我们需要通过创建控制台程序的方式来创建一个 Windows 服务。  
创建完成之后，我们需要安装一个 Nuget 程序包，这个程序包会将一些 Windows 特定的 API 添加到 .NET Core 中，这些 API 实际上已经在完整框架中提供了，但是其中许多是 Windows 特有的，例如 Windows 服务。因此, 它们并没有包含在 .NET Core 的基础库中，但是可以通过将 Nuget 程序包的方式引入到 .NET Core 中。  
&emsp;&emsp;下面我们就可以在 Package Manager Console 中输入以下命令。
```
Install-Package Microsoft.Windows.Compatibility
```

## 代码
&emsp;&emsp;以上引入的 Nuget 程序包中，最让我们感兴趣的是 `ServiceBase` 类。这是一个用于编写 Windows 服务的基类，它提供了一系列的事件钩子，包含服务启动、结束、暂停等。  
&emsp;&emsp;下面呢，我们将在代码中创建一个类，这个类负责将一些简单的日志输出到一个临时文件中。我们将使用这个例子来了解其中的原理。我们的代码如下：
```cs
class LoggingService : ServiceBase
{
    private const string _logFileLocation = @"C:\temp\servicelog.txt";
 
    private void Log(string logMessage)
    {
        Directory.CreateDirectory(Path.GetDirectoryName(_logFileLocation));
        File.AppendAllText(_logFileLocation, DateTime.UtcNow.ToString() + " : " + logMessage + Environment.NewLine);
    }
 
    protected override void OnStart(string[] args)
    {
        Log("Starting");
        base.OnStart(args);
    }
 
    protected override void OnStop()
    {
        Log("Stopping");
        base.OnStop();
    }
 
    protected override void OnPause()
    {
        Log("Pausing");
        base.OnPause();
    }
}
```

&emsp;&emsp;所以这里你会注意到，我们的类是继承了 `ServiceBase` 类，并且我们重写了几个事件方法，输出了一些日志。在服务启动时，会触发 `OnStart` 事件，在服务终止的时候，会触发 `OnStop` 事件。这里我们不应该将过于繁重的任务放置在 `OnStart` 事件中来处理。  
&emsp;&emsp;如果我们想从 `Main` 方式中启动这个服务，代码非常的简单。
```cs
static void Main(string[] args)
{
    ServiceBase.Run(new LoggingService());
}
```

&emsp;&emsp;以上就是全部代码。  

## 服务部署
&emsp;&emsp;在发布服务的时候，我们不可能仅依靠 Visual Studio 来构建我们所需要的服务，我们还需要专门针对 Windows 运行时进行构建。为此，我们需要在项目根目录的命令提示符下运行以下命令。注意，这里我们传入了一个 `-r` 标记来告诉它要构建那个平台。
```
dotnet publish -r win-x64 -c Release
```

&emsp;&emsp;命令运行完毕之后，我们可以检查以下 `/bin/release/netcoreappX.X/publish` 目录，我们可以找到所有的发布代码，但是最重要的是，这里我们可以得到一个可执行的 exe 文件。如果我们不指定运行时，我们只会获得一个 .NET Core 的 dll 程序集，使用这个程序集，我们是没有办法创建 Windows 服务的。  
&emsp;&emsp;现在我们可以将这个发布目录移动带其他的任何地方，但是现在我们就暂时使用当前的发布目录。  
&emsp;&emsp;下一步，我们需要使用管理员角色打开一个命令提示符，然后输入一下命令。
```
sc create TestService BinPath=C:\full\path\to\publish\dir\WindowsServiceExample.exe
```

&emsp;&emsp;`SC` 命令是一个标准的 Windows 命令（与 .NET Core 无关），它可以用来安装 Windows 服务。这里我们将我们的测试服务命名为 `TestService`，更重要的是，我们通过 `BinPath` 参数指定了可执行 exe 文件。  
&emsp;&emsp;运行之后，我们应该会得到以下结果。
```
[SC] CreateService SUCCESS
```

&emsp;&emsp;然后我们要做的就是启动服务。
```
sc start TestService
```

&emsp;&emsp;现在我们可以查看一下我们的日志文件，查看服务的运行情况。  
&emsp;&emsp;如果想要停止并删除服务，我们可以使用一下命令。  
```
sc stop TestService
sc delete TestService
```

## 服务调试
&emsp;&emsp;在这里，我真的认为，使用 " Microsoft " 的方式注定会失败。因为调试服务实在是太繁琐了。  
&emsp;&emsp;首先，我们将 `ServiceBase` 中重写的方法设置为受保护，这意味着我们无法在类之外访问它们，这使得调试它们变得更加困难。这里我发现最好的方法是为每个事件提供一个 public 方法, 并在受保护方法中调用这些 public 方法来完成功能，这虽然有点混乱，
```cs
public void OnStartPublic(string[] args)
{
    Log("Starting");
}
 
protected override void OnStart(string[] args)
{
    OnStartPublic(args);
    base.OnStart(args);
}
```

&emsp;&emsp;但是至少我们可以做如下了事情了。
```cs
static void Main(string[] args)
{
    var loggingService = new LoggingService();
    if (true)   //Some check to see if we are in debug mode (Either #IF Debug etc or an app setting)
    {
        loggingService.OnStartPublic(new string[0]);
        while(true)
        {
            //Just spin wait here. 
            Thread.Sleep(1000);
        }
        //Call stop here etc. 
    }
    else
    {
        ServiceBase.Run(new LoggingService());
    }
}
```

&emsp;&emsp;你的另一个选择是，在调试模式下进行项目发布，安装服务，然后附加调试器。实际上，这是 Microsoft 建议你使用的方式，但是我认为这简直一团糟。  

## 后续
&emsp;&emsp;实际上，我们可以在这里做一些其他非常有用的事情， 比如我们可以通过创建一个 install.bat 批处理文件来为我们运行 SC Create 命令。但我认为，上面我们看到的调试问题，已经让我不再想使用这种方式了。幸运的是，有一个名为 `Topshelf` 的库可以帮助我们减轻很多麻烦，在本系列的下一部分中，我们将研究如何它。  

# 使用 Topshelf 方式
&emsp;&emsp;在前一篇文章中，我给大家介绍了，如何基于微软推荐方式使用 .NET Core 创建 Windows 服务。我们发现使用这种方式，我们很容易就可以搭建和运行一个 Windows 服务，但是问题是使用这种方式，代码调试将非常困难。  
&emsp;&emsp;那么现在就是 `Topshelf` 出场的时候了。`Topshelf` 是一个 .NET Standard 库，它消除了在 .NET Framework 和 .NET Core 中创建 Windows 服务的那些麻烦。  

## 安装
&emsp;&emsp;与微软推荐方式类似，这里 Visual Studio 并没有提供一个基于 `Topshelf` 创建 Windows 服务的模板，所以我们依然需要通过创建普通控制台程序的方式，来创建一个 Windows 服务。  
&emsp;&emsp;然后，我们需要通过 Package Manager Console，运行以下命令，安装 `Topshelf` 类库。
```
Install-Package Topshelf
```

## 代码
&emsp;&emsp;下面我们就来使用 `Topshelf` 重构之前的服务代码。
```cs
public class LoggingService : ServiceControl
{
    private const string _logFileLocation = @"C:\temp\servicelog.txt";
 
    private void Log(string logMessage)
    {
        Directory.CreateDirectory(Path.GetDirectoryName(_logFileLocation));
        File.AppendAllText(_logFileLocation, 
            DateTime.UtcNow.ToString() + " : " + logMessage + Environment.NewLine);
    }
 
    public bool Start(HostControl hostControl)
    {
        Log("Starting");
        return true;
    }
 
    public bool Stop(HostControl hostControl)
    {
        Log("Stopping");
        return true;
    }
}
```

&emsp;&emsp;代码看起来是不是很简单？  
&emsp;&emsp;这里我们的服务类继承了 `ServiceControl` 类（实际上并不需要，但是这可以为我们的工作打下良好的基础）。我们必须实现服务开始和服务结束两个方法，并且像以前一样记录日志。  
&emsp;&emsp;在 `Program.cs` 文件的 `Main` 方法中，我们要写的代码也非常的简单。我们可以直接使用 `HostFactory.Run` 方法来启动服务。
```cs
static void Main(string[] args)
{
    HostFactory.Run(x => x.Service<LoggingService>());
}
```
&emsp;&emsp;这看起来真是太简单了。但这并不是 `HostFactory` 类的唯一功能。这里我们还可以设置
* 服务的名称
* 服务是否自动启动
* 服务崩溃之后的重启时间

```cs
static void Main(string[] args)
{
    HostFactory.Run(x =>
        {
            x.Service<LoggingService>();
            x.EnableServiceRecovery(r => r.RestartService(TimeSpan.FromSeconds(10)));
            x.SetServiceName("TestService");
            x.StartAutomatically();
         }
    );
}
```

&emsp;&emsp;这里其实能说的东西很多，但是我建议你还是自己去看看 `Topshelf` 的文档，学习一下其他的配置选项。基本上你能使用 Windows 命令行完成的所有操作，都可以使用代码来设置：https://topshelf.readthedocs.io/en/latest/configuration/config_api.html  

## 部署服务
&emsp;&emsp;和之前一样，我们需要针对不同的 Windows 环境发布我们的服务。在 Windows 命令提示符下，我们可以在项目目录中执行以下命令：
```
dotnet publish -r win-x64 -c Release
```

&emsp;&emsp;现在我们就可以查看一下 `bin\Release\netcoreappX.X\win-x64\publish` 目录，我们会发现一个编译好的 exe，下面我们就会使用这个文件来安装服务。  
&emsp;&emsp;在上一篇文章中，我们是使用 `SC` 命令来安装 Windows 服务的。使用 `Topshelf` 我们就不需要这么做了，`Topshelf` 提供了自己的命令行参数来安装服务。基本上使用代码能完成的配置，都可以使用命令行来完成。  
&emsp;&emsp;你可以查看相关的文档：http://docs.topshelf-project.com/en/latest/overview/commandline.html 
```
WindowsServiceExample.exe install
```

&emsp;&emsp;这里 `WindowsServiceExample.exe` 是我发布之后的 exe 文件。运行以上命令之后，服务应该就正常安装了！这里有一个小问题，我经常发现，即使配置了服务自动启动，但是服务安装之后，并不会触发启动操作。所有在服务安装之后，我们还需要通过以下命令来启动服务。
```
WindowsServiceExample.exe start
```

> &emsp;&emsp;在生产环境部署的时候，我的经验是在安装服务之后，等待 10 秒钟，再启动服务。

## 调试服务
&emsp;&emsp;当我们是使用微软推荐方式的时候，我们会遇到了调试困难的问题。大多数情况下，无论是否在服务内部运行，我们都不得不使用命令行标志、`#IF DEBUG` 指令或者配置值来实现调试。然后使用 Hack 的方式在控制台程序中模拟服务。  
&emsp;&emsp;因此，这就是为什么我们要使用 `Topshelf`。  
&emsp;&emsp;如果我们的服务代码已经在 Visual Studio 中打开了，我们就可以直接启动调试。`Topshelf` 会模拟在控制台中启动服务。我们应该能在控制台中看到以下的消息。
```
The TestService service is now running, press Control+C to exit.
```

&emsp;&emsp;这确实符合了我们的需求。它启动了我们的服务，并像真正的 Windows 服务一样在后台运行。我们可以像往常一样设置断点，基本上它遵循的流程和正常安装的服务一样。  
&emsp;&emsp;我们可以通过 ctrl+c，来关闭我们的应用，但是在运行服务执行 Stop 方法之前，它是不能被关闭的，这使我们可以调试服务的关闭流程。与调试指令和配置标志相比，这要容易的多。  
&emsp;&emsp;这里需要注意一个问题。如果你收到的以下内容的消息：
```
The TestService service is running and must be stopped before running via the console
```

&emsp;&emsp;这意味着你尝试调试的服务实际上已经作为 Windows 服务被安装在系统中了，你需要停止（不需要卸载）这个正在运行的服务，才可以正常调试。

## 后续
&emsp;&emsp;在上一篇中，有读者指出 .NET Core 中实际上已经提供了一种完全不同的方式运行 Windows 服务。它的实质是利用了 ASP.NET Core 中引入的 “ 托管服务 ” 模型，并允许它们作为 Windows 服务来运行，这真的是非常的棒。

# 使用 .NET Core 工作器方式
## 安装步骤
&emsp;&emsp;这里首先你要确保你已经安装了 .NET Core 3.0 或以上版本。在我编写这篇文章的时候，.NET Core 3.1 刚刚发布，Visual Studio 应该会提示你升级到最新版本。但是如果你想要在 .NET Core 2.x 项目中使用这个方式，应该是行不通的。  
&emsp;&emsp;如果你喜欢使用命令行创建项目，你就需要使用工作器（worker）类型创建项目：
```
dotnet new worker
```

&emsp;&emsp;如果你是一个和我一样喜欢使用 Visual Studio 的开发人员，那么你可以在 Visual Studio 中使用项目模板完成相同的功能。
![ ](65831-20191214083435124-1006535101.png)

&emsp;&emsp;这样做将创建出一个包含两个文件的项目。其中 `Program.cs` 文件是应用的启动 “ 引导程序 ” 。另外一个文件是 `worker.cs` 文件，在这个文件中，你可以编写你的服务逻辑。  
&emsp;&emsp;这看起来应该是相当的容易，但是为这个程序添加额外的并行后台服务，你还需要添加一个类，并让它继承 `BackgroundService` 类：
```cs
public class MyNewBackgroundWorker : BackgroundService
{    
    protected override Task ExecuteAsync(CancellationToken stoppingToken)    
    {        
        //Do something.     
    }
}
```

&emsp;&emsp;然后在 `Program.cs` 中，我们要做的只是把当前的 Worker 注册到服务集合（Service Collection）中即可。
```cs
.ConfigureServices((hostContext, services) =>
{
    services.AddHostedService<Worker>();
    services.AddHostedService<MyNewBackgroundWorker>();
});
```

&emsp;&emsp;实际上作为 “ 后台服务 ” 任务的运行程序，`AddHostedService` 方法已经在框架中存在了很长时间了。在之前我们已经完成的一篇关于[ ASP.NET Core 托管服务](https://dotnetcoretutorials.com/2019/01/13/hosted-services-in-asp-net-core/)的文章， 但是在当时场景中，我们托管是是整个应用，而非一个在你应用程序幕后运行的东西。  

## 运行 / 调试我们的应用
&emsp;&emsp;在默认的工作器（worker）模板中，已经包含了一个后台服务，这个服务可以将当前时间输出到控制台窗口。下面让我们点击 F5 来运行程序，看看我们能得到什么。
```
info: CoreWorkerService.Worker[0]      
      Worker running at: 12/07/2019 08:20:30 +13:00
info: Microsoft.Hosting.Lifetime[0]      
      Application started. Press Ctrl+C to shut down.
```

&emsp;&emsp;在我们启动程序之后，程序立刻就运行了！我们可以保持控制台的打开状态来调试应用，或者直接关闭窗口退出。相较于使用 " Microsoft " 方式来调试一个 Windows 服务，这简直就是天堂。  
&emsp;&emsp;这里我们需要注意的另外一件事情是编写控制台程序的平台。在最后，我们不仅在控制台窗口输出了时间，还通过依赖注入创建了一个托管 worker。我们也可以使用依赖注入容器来注入仓储，配置环境变量，获取配置等。  
&emsp;&emsp;但这里我们还没有做的事情是，将这个应用转换为 Windows 服务。  

## 将我们的应用转换成 Windows 服务
&emsp;&emsp;为了将应用转换成 Windows 服务，我们需要使用如下命令引入一个包。
```
Install-Package Microsoft.Extensions.Hosting.WindowsServices
```

&emsp;&emsp;下一步，我们需要修改 `Program.cs` 文件，添加 `UseWindowsService()` 方法的调用。
```cs
public static IHostBuilder CreateHostBuilder(string[] args) => 
    Host.CreateDefaultBuilder(args)    
        .ConfigureServices((hostContext, services) =>    
        {        
            services.AddHostedService<Worker>();   
         })
         .UseWindowsService();
```

&emsp;&emsp;以上就是所有需要变更的代码。  
&emsp;&emsp;运行我们的程序，你会发现和之前的效果完全样。但是这里最大的区别是，我们可以将当前应用以 Windows 服务的形式安装了。  
&emsp;&emsp;为了实现这一目的，我们需要发布当前项目。在当前项目目录中，我们可以运行以下命令：
```
dotnet publish -r win-x64 -c Release
```

&emsp;&emsp;然后我们就可以借助标准的 Windows 服务安装器来安装当前服务了。
```
sc create TestService BinPath=C:\full\path\to\publish\dir\WindowsServiceExample.exe
```

&emsp;&emsp;当前，你也可以使用 Windows 服务安装器的其他命令。
```
sc start TestServicesc stop TestServicesc delete TestService
```

&emsp;&emsp;最后检查一下我们的服务面板。
![ ](65831-20191214083446395-754761263.png)

&emsp;&emsp;服务已经正常工作了。

## 在 Linux 中运行服务
&emsp;&emsp;老实说，我没有太多的Linux经验，但是终归是需要了解一下...  

&emsp;&emsp;在 Linux 系统中, 如果你希望我们编写的 “ Windows ” 服务在 Linux 系统中作为服务运行，你需要做以下 2 步：
* 使用 `Microsoft.Extensions.Hosting.Systemd` 替换之前的 `Microsoft.Extensions.Hosting.WindowsServices`。
* 使用 `UseSystemd()` 替换 `UseWindowsService()`。

# Microsoft vs Topshelf vs .NET Core Workers
&emsp;&emsp;到现在为止，我们已经介绍了借助 3 种不同的方式来创建 Windows 服务。  
&emsp;&emsp;你可能会问 “ 好吧，那我到底应该选择哪一种 ？”  
&emsp;&emsp;这里呢，我们可以首先把 " Microsoft " 这种老派学院式的方式抛弃。以为它的调试实在是太麻烦了，而且没有什么实际的用处。  
&emsp;&emsp;然后剩下的就是 `Topshelf` 和 .NET Core 工作器两种方式了。在我看来，.NET Core 工作器，已经很好的融入 .NET Core 生态系统，如果你正在开发 ASP.NET Core 应用，那么使用 .NET Core 工作器就很有意义。最重要的是，当你创建一个后台服务的时候，你可以让它在一个 ASP.NET Core 网站中的任意位置运行，这非常的方便。但是缺点是安装。你必须使用 `SC` 命令来安装服务。这一部分 `Topshelf` 可能更胜一筹。  
&emsp;&emsp;`Topshelf` 总体上将非常的友好，并且具有最好的安装方式，但是使用额外的库，也增加了学习的成本。  
&emsp;&emsp;所以 `Topshelf` 和 .NET Core 工作器，大家可以自行选择，都是不错的方案。  

---

# 部署 Linux 服务
> https://www.cnblogs.com/RainFate/p/12095793.html

环境：一台全新的 Centos 7.7.1908 系统  

## 安装 .net core相关环境
参考：https://docs.microsoft.com/zh-cn/dotnet/core/install/linux-package-manager-centos7  

注册 Microsoft 密钥和源：
```
sudo rpm -Uvh https://packages.microsoft.com/config/centos/7/packages-microsoft-prod.rpm
```

安装 .NET Core SDK：
```
sudo yum install dotnet-sdk-3.0  -y
```

安装完成之后可以输入 `dotnet --version` 查看是否可以返回对应版本  

## 修改代码
程序代码需要引用 `Microsoft.Extensions.Hosting.Systemd` Nuget包
```
Install-Package Microsoft.Extensions.Hosting.Systemd
```

修改 `Program.cs` 文件，添加 `UseSystemd()` 方法调用，可以和 `UseWindowsService()` 共存
```cs
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureServices((hostContext, services) =>
        {
            services.AddHostedService<Worker>();
        })
          .UseWindowsService()
          .UseSystemd();
```

然后把发布文件移至 Linux 系统

## 部署服务
Linux 的服务是通过 `systemd` 守护进程部署的。现在在系统中我们有了一个发布后的应用程序，我们需要为 systemd 创建配置文件部署服务。步骤如下：  
创建一个 **.service** 文件（我们要部署服务，因此需要 **.service** 文件），填入以下内容。可以在 Linux 中直接创建或者通过 Windows 创建然后拷贝至 Linux。  
[参考链接](https://github.com/RainFate/WindowsServiceDemo/blob/Init/WorkerService/%E6%9C%8D%E5%8A%A1%E5%AE%89%E8%A3%85%E5%8D%B8%E8%BD%BD/Linux/testapp.service)
```
[Unit]
Description = my test app

[Service]
Type = notify
ExecStart = /usr/bin/dotnet/home/demo/WorkerService.dll

[Install]
WantedBy = multi-user.target
```

* `Description`：描述，看个人需要是否添加。不需要可以去掉。只留下 `[Service]` 和 `[Install]`
* `Type=notify`：当前服务启动完毕，会通知 `Systemd`，再继续往下执行
* `ExecStart`：启动当前服务的命令，程序如何启动，第一个路径是固定路径。第二个路径是应用程序的 dll 路径（可以自定义）
* `WantedBy`：表示该服务所在的 Target 服务组；`multi-user.target` 表示多用户命令行状态。

把 **.service** 文件移动至 **/etc/systemd/system/** 固定目录下，假设自定义文件名称为：**testapp.service**（如果使用其他名称，请更改 testapp）
![ ](1.png)

使用 `systemctl` 命令重新加载新的配置文件
```
sudo systemctl daemon-reload
```

查看相关服务状态
```
sudo systemctl status testapp
```

您应该看到类似以下的内容：
![ ](2.png)

这表明您已注册的新服务已禁用，我们可以通过运行以下命令来启动我们的服务：
```
sudo systemctl start testapp.service
```

重新运行 `sudo systemctl status testapp` 查看服务状态显示已激活正在运行中
![ ](3.png)

设置服务开机自启
```
sudo systemctl enable testapp.service
```

到此我们的服务已经完整的部署到了 Linux 系统中  
现在我们有一个运行了 `systemd` 的应用程序，我们可以看看日志记录集成。使用 `systemd` 的好处之一是可以使用 `journalctl` 访问的集中式日志记录系统。  
首先，我们可以使用 `journalctl`（访问日志的命令）查看服务日志：
```
sudo journalctl -u testapp
```
![ ](4.png)


可以看到我们的程序正在运行，可以使用 ↑ ↓ ← → 查看日志内容，或者使用 `grep` 搜索，q 键退出。

## 卸载自定义服务
```
servicename=testapp.service

systemctl stop $servicename
systemctl disable $servicename
rm -rf /etc/systemd/system/$servicename
rm -rf /etc/systemd/system/$servicename symlinks that might be related
systemctl daemon-reload
systemctl reset-failed
```