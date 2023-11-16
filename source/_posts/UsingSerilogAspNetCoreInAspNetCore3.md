---
title: 在 ASP.NET Core 3.0 中使用 Serilog.AspNetCore
tags:
  - .NET
categories:
  - 编程
date: 2020-01-20 16:52:42
---

> 译文：  
> &emsp;&emsp;使用 Serilog RequestLogging 减少日志详细程度：https://www.cnblogs.com/yilezhu/p/12215934.html  
> &emsp;&emsp;使用 Serilog 记录所选的终结点属性：https://www.cnblogs.com/yilezhu/p/12227271.html  
> &emsp;&emsp;使用 Serilog.AspNetCore 记录 MVC 属性：https://www.cnblogs.com/yilezhu/p/12243984.html  
> &emsp;&emsp;从 Serilog 请求日志记录中排除健康检查端点：https://www.cnblogs.com/yilezhu/p/12253361.html   

> 原文：  
> &emsp;&emsp;[Reducing log verbosity with Serilog RequestLogging](https://andrewlock.net/using-serilog-aspnetcore-in-asp-net-core-3-reducing-log-verbosity/)  
> &emsp;&emsp;[Logging the selected Endpoint Name with Serilog](https://andrewlock.net/using-serilog-aspnetcore-in-asp-net-core-3-logging-the-selected-endpoint-name-with-serilog/)  
> &emsp;&emsp;[Logging MVC properties with Serilog.AspNetCore](https://andrewlock.net/using-serilog-aspnetcore-in-asp-net-core-3-logging-mvc-propertis-with-serilog/)  
> &emsp;&emsp;[Excluding health check endpoints from Serilog request logging](https://andrewlock.net/using-serilog-aspnetcore-in-asp-net-core-3-excluding-health-check-endpoints-from-serilog-request-logging/)  

<!--more-->

# 使用 Serilog RequestLogging 减少日志详细程度
&emsp;&emsp;众所周知，ASP.NET Core 的重要改变之一是把日志记录内置于框架中。这意味着您可以（如果需要）从自己的标准日志基础设施访问所有深层基础设施日志。缺点是有时您会收到**太多**的日志。  
在这个简短的系列文章中，我将介绍如何使用 [Serilog 的 ASP.NET Core 请求日志记录功能](https://github.com/serilog/serilog-aspnetcore#request-logging)。在第一篇文章中，我将讲述如何将 Serilog 的`RequestLoggingMiddleware`添加到您的应用程序，以及它提供的好处。在后续文章中，我将描述如何进一步自定义行为。

> &emsp;&emsp;我已经将这些帖子草拟了一段时间。从那时起，[Serilog 的创建者 Nicholas Blumhardt 就在 ASP.NET Core 3.0 中使用 Serilog 撰写了一篇详尽的博客文章](https://nblumhardt.com/2019/10/serilog-in-aspnetcore-3)。这是一篇非常详细（至少我认为是这样）的文章，我强烈建议您阅读。您可以在他的文章中找到我在本系列文章中谈论的大部分内容，所以请查看！

## 原生请求日志
&emsp;&emsp;在本节中，首先让我们创建一个标准的 ASP.NET Core 3.0 的 Razor pages 应用，当然你也可以直接使用`dotnet new webapp`命令来进行创建。这将创建一个标准 *Program.cs* ，如下所示：
```cs
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }

    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            {
                webBuilder.UseStartup<Startup>();
            });
}
```

&emsp;&emsp;还有一个 *Startup.cs* ，用于配置中间件管道，`Configure`如下所示：
```cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseExceptionHandler("/Error");
        // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseStaticFiles();

    app.UseRouting();

    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapRazorPages();
    });
}
```

&emsp;&emsp;如果您运行该应用程序并导航至主页，则默认情况下，您会在控制台中看到每个请求都会产生许多的日志。以下日志是针对对主页的**单个**请求生成的（此后我还没有包括对 CSS 和 JS 文件的其他请求）（这是是开发环境请求出现的日志）：
```
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/2 GET https://localhost:5001/
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[0]
      Executing endpoint '/Index'
info: Microsoft.AspNetCore.Mvc.RazorPages.Infrastructure.PageActionInvoker[3]
      Route matched with {page = "/Index"}. Executing page /Index
info: Microsoft.AspNetCore.Mvc.RazorPages.Infrastructure.PageActionInvoker[101]
      Executing handler method SerilogRequestLogging.Pages.IndexModel.OnGet - ModelState is Valid
info: Microsoft.AspNetCore.Mvc.RazorPages.Infrastructure.PageActionInvoker[102]
      Executed handler method OnGet, returned result .
info: Microsoft.AspNetCore.Mvc.RazorPages.Infrastructure.PageActionInvoker[103]
      Executing an implicit handler method - ModelState is Valid
info: Microsoft.AspNetCore.Mvc.RazorPages.Infrastructure.PageActionInvoker[104]
      Executed an implicit handler method, returned result Microsoft.AspNetCore.Mvc.RazorPages.PageResult.
info: Microsoft.AspNetCore.Mvc.RazorPages.Infrastructure.PageActionInvoker[4]
      Executed page /Index in 221.07510000000002ms
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[1]
      Executed endpoint '/Index'
info: Microsoft.AspNetCore.Hosting.Diagnostics[2]
      Request finished in 430.9383ms 200 text/html; charset=utf-8
```

&emsp;&emsp;单个请求就是 **10 条**日志。现在，很清楚，它正在`Development`环境中运行，该环境默认情况下将 *Microsoft* 名称空间中的所有信息记录在 “Information” 或更高的级别。如果我们切换到`Production`环境，则默认模板会将 *Microsoft* 命名空间的日志过滤到 “Warning” 。现在导航到默认主页会生成以下日志（这里注意，如果你现在使用 ASP.NET Core3.1 貌似 *Microsoft* 命名空间默认日志级别已经改为`Warning`）：
```

```

&emsp;&emsp;是的，根本没有日志！上一次运行中生成的所有日志都位于 Microsoft 命名空间中，并且属于 “Information” 级别，因此将它们全部过滤掉。就个人而言，我觉得这有点麻烦。如果生产版本仅仅只是想记录**一部分内容**，而其他相关联的内容则不进行记录，这将会更有用的。

&emsp;&emsp;一种可能的解决方案是自定义应用于每个命名空间的过滤器。例如，您可以将 *Microsoft.AspNetCore.Mvc.RazorPages* 命名空间限制为 “Warning” 级别，而将更通用的 *Microsoft* 命名空间保留为 “Information” 级别。现在，您将获得精简后的日志集：
```
info: Microsoft.AspNetCore.Hosting.Diagnostics[1]
      Request starting HTTP/2 GET https://localhost:5001/
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[0]
      Executing endpoint '/Index'
info: Microsoft.AspNetCore.Routing.EndpointMiddleware[1]
      Executed endpoint '/Index'
info: Microsoft.AspNetCore.Hosting.Diagnostics[2]
      Request finished in 184.788ms 200 text/html; charset=utf-8
```

&emsp;&emsp;这些日志中包含一些有用的信息 —— URL，HTTP 方法，时间信息，端点等 —— 并且没有**太多**冗余。但是，仍然令人讨厌的是它们是四个单独的日志消息。（还是很多，如果能精简成一条日志记录是不是会好很多）  
&emsp;&emsp;这是 Serilog `RequestLoggingMiddleware`旨在解决的问题 —— 为请求中的每个步骤创建单独的日志相反，它是创建一个包含所有相关信息的“摘要”日志消息。

## 将 Serilog 添加到应用程序
&emsp;&emsp;使用 Serilog `RequestLoggingMiddleware`的一个前提条件就是您正在使用 Serilog！在本节中，我将介绍将 Serilog 添加到 ASP.NET Core 应用程序中。如果您已经安装了 Serilog，请跳至下一部分。

&emsp;&emsp;首先安装 *Serilog.AspNetCore* NuGet 软件包，再加上控制台和 [Seq](https://datalust.co/seq) 接收器【这是一个漂亮的可视化日志 UI】，以便我们可以查看日志。您可以通过运行以下命令从命令行执行此操作：
```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Seq
```

&emsp;&emsp;现在该用 Serilog 替换默认日志了。您可以通过多种方式执行此操作，但是建议的方法是在`Program.Main`执行其他任何操作之前先配置记录器。这与 ASP.NET Core 通常使用的方法背道而驰，但建议用于 Serilog。当然这会导致您的 *Program.cs* 文件变得更长：
```cs
// Additional required namespaces
using Serilog;
using Serilog.Events;

namespace SerilogDemo
{
    public class Program
    {
        public static void Main(string[] args)
        {
            // Create the Serilog logger, and configure the sinks
            Log.Logger = new LoggerConfiguration()
               .MinimumLevel.Debug()
               .MinimumLevel.Override("Microsoft", LogEventLevel.Information)
               .Enrich.FromLogContext()
               .WriteTo.Console()
               .WriteTo.Seq("http://localhost:5341")
               .CreateLogger();
            // Wrap creating and running the host in a try-catch block
            try
            {
                Log.Information("Starting host");
                CreateHostBuilder(args).Build().Run();
            }
            catch (Exception ex)
            {
                Log.Fatal(ex, "Host terminated unexpectedly");
            }
            finally
            {
                Log.CloseAndFlush();
            }
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .UseSerilog()
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                });
    }
}
```

&emsp;&emsp;尽管这样设置可能显得更为复杂，但是此设置可确保例如在 *appsettings.json* 文件格式错误或缺少配置文件的情况下仍会获取日志。如果现在运行您的应用程序，您将看到与我们最初相同的 10 条日志，只是格式略有不同：
```
[13:30:27 INF] Request starting HTTP/2 GET https://localhost:5001/  
[13:30:27 INF] Executing endpoint '/Index'
[13:30:27 INF] Route matched with {page = "/Index"}. Executing page /Index
[13:30:27 INF] Executing handler method SerilogRequestLogging.Pages.IndexModel.OnGet - ModelState is Valid
[13:30:27 INF] Executed handler method OnGet, returned result .
[13:30:27 INF] Executing an implicit handler method - ModelState is Valid
[13:30:27 INF] Executed an implicit handler method, returned result Microsoft.AspNetCore.Mvc.RazorPages.PageResult.
[13:30:27 INF] Executed page /Index in 168.28470000000002ms
[13:30:27 INF] Executed endpoint '/Index'
[13:30:27 INF] Request finished in 297.0663ms 200 text/html; charset=utf-8
```

&emsp;&emsp;现在，通过在应用程序生命周期的早期进行配置，我们的日志记录配置的更加健壮，但实际上尚未解决我们提出的问题。为此，我们将添加`RequestLoggingMiddleware`。

## 切换到 Serilog 的 RequestLoggingMiddleware
&emsp;&emsp;`RequestLoggingMiddleware`被包含在 *Serilog.AspNetCore* 中，可以被用于为每个请求添加一个单一的“摘要”日志消息。如果您已经完成了上一节中的步骤，则添加这个中间件将变得很简单。在您的`Startup`类中，在您想要记录日志的位置使用`UseSerilogRequestLogging()`进行调用：
```cs
// Additional required namespace
using Serilog;

public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // ... Error handling/HTTPS middleware
    app.UseStaticFiles();

    app.UseSerilogRequestLogging(); // <-- Add this line

    app.UseRouting();
    app.UseAuthorization();
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapRazorPages();
    });
}
```

&emsp;&emsp;与 ASP.NET Core 的中间件管道一样，**顺序很重要**。当请求到达`RequestLoggingMiddleware`中间件时，它将启动计时器，并将请求传递给后续中间件进行处理。当后面的中间件最终生成响应（或抛出异常），则响应通过中间件管道传递回到请求记录器，并在其中记录了结果并写入概要日志信息。  
&emsp;&emsp;Serilog 只能记录到达中间件的请求。在上面的例子中，我已经在`StaticFilesMiddleware`**之后**添加了`RequestLoggingMiddleware`。因此如果请求被`UseStaticFiles`处理并使管道短路的话，日志将不会被记录。鉴于静态文件中间件非常嘈杂，而且通常这是人们期望的行为（静态文件进行短路，不需要进行记录），但是如果您也希望记录对静态文件的请求，则可以在管道中 Serilog 中间件移动到更早的位置。  
&emsp;&emsp;如果我们再一次运行该应用程序，你还是会看到原来的 10 个日志消息，但你会看到一个**额外的**通过`SerilogRequestLoggingMiddleware`汇总的日志消息，倒数第二的消息：
```
# Standard logging from ASP.NET Core infrastructure
[14:15:44 INF] Request starting HTTP/2 GET https://localhost:5001/  
[14:15:44 INF] Executing endpoint '/Index'
[14:15:45 INF] Route matched with {page = "/Index"}. Executing page /Index
[14:15:45 INF] Executing handler method SerilogRequestLogging.Pages.IndexModel.OnGet - ModelState is Valid
[14:15:45 INF] Executed handler method OnGet, returned result .
[14:15:45 INF] Executing an implicit handler method - ModelState is Valid
[14:15:45 INF] Executed an implicit handler method, returned result Microsoft.AspNetCore.Mvc.RazorPages.PageResult.
[14:15:45 INF] Executed page /Index in 124.7462ms
[14:15:45 INF] Executed endpoint '/Index'

# Additional Log from Serilog
[14:15:45 INF] HTTP GET / responded 200 in 249.6985 ms

# Standard logging from ASP.NET Core infrastructure
[14:15:45 INF] Request finished in 274.283ms 200 text/html; charset=utf-8
```

&emsp;&emsp;关于此日志，有几点需要说明下：
* 它在一条消息中包含您想要的大多数相关信息 —— HTTP 方法，URL 路径，状态代码，持续时间。
* 显示的持续时间**略**短于 Kestrel 在后续消息中记录的值。这是可以预期的，因为 Serilog 仅在请求到达其中间件时才开始计时，而在返回时停止计时（在生成响应之后）。
* 在这两种情况下，使用结构日志记录时都会记录其他值。例如，记录了 RequestId 和 SpanId（[用于跟踪功能](https://devblogs.microsoft.com/aspnet/improvements-in-net-core-3-0-for-troubleshooting-and-monitoring-distributed-apps/)），因为它们是日志记录范围的一部分。您可以在登录到 seq 的请求的以下图像中看到这一点。
* 默认情况下，我们确实会丢失一些信息。例如，不再记录终结点名称和 Razor 页面处理程序。在后续文章中，我将展示如何将它们添加到摘要日志中。
* 如果想要通过`http://localhost:5341`访问 UI，你可能需要下载 seq 进行安装。由于某种不知名的原因，可能下载会很慢。所以当然你也可以关注公众号 “DotNetCore实战” 然后回复关键字 “seq” 获取下载地址。

![ ](1.png)

&emsp;&emsp;完成整理工作所剩下的就是过滤掉我们当前正在记录的信息级日志消息。在 *Program.cs* 中更新 Serilog 配置以添加额外的过滤器：
```cs
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Debug()
    .MinimumLevel.Override("Microsoft", LogEventLevel.Information)
    // Filter out ASP.NET Core infrastructre logs that are Information and below
    .MinimumLevel.Override("Microsoft.AspNetCore", LogEventLevel.Warning) 
    .Enrich.FromLogContext()
    .WriteTo.Console()
    .WriteTo.Seq("http://localhost:5341")
    .CreateLogger();
```

&emsp;&emsp;通过最后的更改，您现在将获得一组干净的请求日志，其中包含每个请求的摘要数据：
```
[14:29:53 INF] HTTP GET / responded 200 in 129.9509 ms
[14:29:56 INF] HTTP GET /Privacy responded 200 in 10.0724 ms
[14:29:57 INF] HTTP GET / responded 200 in 3.3341 ms
[14:30:54 INF] HTTP GET /Missing responded 404 in 16.7119 ms
```

&emsp;&emsp;在下一篇文章中，我将介绍如何通过记录其他数据来增强此日志。

## 总结
&emsp;&emsp;在本文中，我描述了如何使用 *Serilog.AspNetCore* 的请求日志记录中间件来减少为每个 ASP.NET Core 请求生成的日志数，同时仍记录摘要数据。如果您已经在使用 Serilog，则非常容易启用。只需在您的 *Startup.cs* 文件中调用`UseSerilogRequestLogging()`。  
&emsp;&emsp;当请求到达此中间件时，它将启动计时器。当后续的中间件生成响应（或引发异常）时，响应将通过中间件管道返回到请求记录器，记录器记录结果并编写摘要日志消息。  
&emsp;&emsp;添加请求日志记录中间件之后，您可以过滤掉默认情况下在 ASP.NET Core 3.0 中生成的更多基础结构日志，而不会丢失有用的信息。

---

# 使用 Serilog 记录所选的终结点属性
&emsp;&emsp;在我的上一篇文章中，我描述了如何配置 Serilog 的 RequestLogging 中间件为每个请求创建“摘要”日志，以替换默认情况下从 ASP.NET Core 获取的 10 个或更多日志。

&emsp;&emsp;在本文中，我将展示如何向 Serilog 的摘要请求日志中添加其他元数据，例如请求的主机名，响应的内容类型或从 ASP.NET Core 3.0 中使用的[终结点路由中间件](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/routing?view=aspnetcore-3.0#endpoint-routing-differences-from-earlier-versions-of-routing)所选择的端点名称。

## ASP.NET Core 基础结构日志很详细，但是默认情况下具有太多详细信息
&emsp;&emsp;正如我在上一篇文章中所展示的那样，在开发环境中，ASP.NET Core 基础架构将为每一个 RazorPage 处理程序生成 10 条日志消息：
![ ](2.png)

&emsp;&emsp;通过安装了 [Serilog.AspNetCore](https://github.com/serilog/serilog-aspnetcore) 的 NuGet 包后并引入`RequestLoggingMiddleware`之后，可以将其精简为一条日志消息：
![ ](3.png)

> &emsp;&emsp;本文中使用的所有日志图片均来自一款优秀的为结构化日志提供可视化界面的工具 —— [Seq](https://datalust.co/seq)

&emsp;&emsp;显然，原始的日志集更加冗长，并且其中大部分不是特别有用的信息。但是，如果您将原始的 10 条日志作为一个整体来看，则与 Serilog 摘要日志相比，它们确实会在结构日志模板中记录一些其他属性。

&emsp;&emsp;由 ASP.NET Core 基础结构记录的而 Serilog 未记录的扩展内容包括（下面这些还是英文的看着顺眼）：
* Host (`localhost:5001`)
* Scheme (`https`)
* Protocol (`HTTP/2`)
* QueryString (`test=true`)
* EndpointName (`/Index`)
* HandlerName (`OnGet`/`SerilogRequestLogging.Pages.IndexModel.OnGet`)
* ActionId (`1fbc88fa-42db-424f-b32b-c2d0994463f1`)
* ActionName (`/Index`)
* RouteData (`{page = "/Index"}`)
* ValidationState (`True`/`False`)
* ActionResult (`PageResult`)
* ContentType (`text/html; charset=utf-8`)

&emsp;&emsp;我认为如果要把上述属性中的其中一些包含在摘要日志消息中，将非常有用。例如，如果您的应用程序绑定到多个主机名，那么`Host`绝对是重要的日志。`QueryString`可能是另一个有用的字段。`EndpointName`/`HandlerName`，`ActionId`和`ActionName`似乎不那么重要，因为您应该能够推断出给定的请求路径，但是显式记录它们将帮助您更加方便的捕获错误，并使过滤针对特定操作的所有请求变得更加容易。

&emsp;&emsp;概括地说，您可以将这些属性分为两类：
* *请求* / *响应* 特性：如`Host`，`Scheme`，`ContentType`，`QueryString`，`EndpointName`
* *MVC* / *RazorPages* 相关的属性：如`HandlerName`，`ActionId`，`ActionResult`等

&emsp;&emsp;在这篇文章中，我将展示如何添加这些类别中的第一种，即与请求/响应相关的属性，在下一篇文章中，我将展示如何添加基于 MVC / RazorPages 的属性。

## 向 Serilog 请求日志添加扩展数据
&emsp;&emsp;在上一篇文章中，我展示了如何将 Serilog 请求日志记录添加到您的应用程序中，因此在此不再赘述。现在，我假设您已经进行了设置，并且您拥有一个包含以下内容的`Startup.Configure`方法：
```cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // ... Error handling/HTTPS middleware
    app.UseStaticFiles();

    app.UseSerilogRequestLogging(); // <-- Add this line

    app.UseRouting();
    app.UseAuthorization();
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapRazorPages();
    });
}
```

&emsp;&emsp;该`UseSerilogRequestLogging()`扩展方法将 Serilog `RequestLoggingMiddleware`添加到请求管道中。您还可以通过调用重载来[配置`RequestLoggingOptions`的实例](https://github.com/serilog/serilog-aspnetcore/blob/ffed9d231aefc3de7c13a03a570fb45c326632b0/src/Serilog.AspNetCore/AspNetCore/RequestLoggingOptions.cs)。此类具有几个属性，可以让您自定义请求记录器如何生成日志语句：
```cs
public class RequestLoggingOptions
{
    public string MessageTemplate { get; set; }
    public Func<HttpContext, double, Exception, LogEventLevel> GetLevel { get; set; }
    public Action<IDiagnosticContext, HttpContext> EnrichDiagnosticContext { get; set; }
}
```

&emsp;&emsp;该`MessageTemplate`属性控制将日志呈现为的字符串格式，`GetLevel`允许您控制给定日志索要记录的级别，如`Debug`/`Info`/`Warning`等。这里我们所关心的是`EnrichDiagnosticContext`属性。  
&emsp;&emsp;设置了该属性的`Action<>`之后，在生成日志消息时它将被 Serilog 中间件调用并执行。它在日志写入**之前**运行，这意味着它在中间件管道执行**之后**运行。例如，在下图中（[取自我的书《ASP.NET Core in Action》](https://www.manning.com/books/asp-dot-net-core-in-action?a_aid=aspnetcore-in-action&a_bid=5b1b11eb)），当响应“回传”到中间件管道时，在第 5 步写入日志：
![ ](4.png)

&emsp;&emsp;在管道处理**之后**写入日志这一事实意味着两件事：
* 我们可以访问 *Response* 的属性，例如状态码，经过的时间或内容类型
* 我们可以访问在管道后面设置的中间件的**功能**，例如，由`EndpointRoutingMiddleware`（通过`UseRouting()`添加的）设置的功能：`IEndpointFeature`

&emsp;&emsp;在下一部分中，我将提供一个帮助程序功能，该功能会将所有“缺少”属性添加到 Serilog 请求日志消息中。

## 在 IDiagnosticContext 中设置扩展值
&emsp;&emsp;*Serilog.AspNetCore* 会将接口`IDiagnosticContext`作为单例添加到 DI 容器中，因此您可以从任何类中访问它。然后，您可以调用`Set()`方法，将其他属性附加到请求日志消息中。

&emsp;&emsp;例如，[如文档所示](https://github.com/serilog/serilog-aspnetcore#request-logging)，您可以从操作方法中添加任意值：
```cs
public class HomeController : Controller
{
    readonly IDiagnosticContext _diagnosticContext;
    public HomeController(IDiagnosticContext diagnosticContext)
    {
        _diagnosticContext = diagnosticContext;
    }

    public IActionResult Index()
    {
        // The request completion event will carry this property
        _diagnosticContext.Set("CatalogLoadTime", 1423);
        return View();
    }
}
```

&emsp;&emsp;然后，结果摘要日志将包含属性`CatalogLoadTime`。

&emsp;&emsp;`RequestLoggingOptions`通过设置所提供`IDiagnosticContext`实例的值，我们基本上使用完全相同的方法来定制中间件所使用的方法。  
&emsp;&emsp;下面的静态帮助器类从当前`HttpContext`上下文检索值，并在值可用时对其进行设置。
```cs
public static class LogHelper 
{
    public static void EnrichFromRequest(IDiagnosticContext diagnosticContext, HttpContext httpContext)
    {
        var request = httpContext.Request;

        // Set all the common properties available for every request
        diagnosticContext.Set("Host", request.Host);
        diagnosticContext.Set("Protocol", request.Protocol);
        diagnosticContext.Set("Scheme", request.Scheme);

        // Only set it if available. You're not sending sensitive data in a querystring right?!
        if(request.QueryString.HasValue)
        {
            diagnosticContext.Set("QueryString", request.QueryString.Value);
        }

        // Set the content-type of the Response at this point
        diagnosticContext.Set("ContentType", httpContext.Response.ContentType);

        // Retrieve the IEndpointFeature selected for the request
        var endpoint = httpContext.GetEndpoint();
        if (endpoint is object) // endpoint != null
        {
            diagnosticContext.Set("EndpointName", endpoint.DisplayName);
        }
    }
}
```

&emsp;&emsp;上面的帮助器函数从 “Request”，“Response” 以及其他中间件（端点名称）设置的功能中检索值。您可以扩展它，以根据需要在请求中添加其他值。

&emsp;&emsp;您可以在你的`Startup.Configure()`方法中通过调用`UseSerilogRequestLogging`的`EnrichDiagnosticContext`属性，来注册上面的帮助类：
```cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // ... Other middleware

    app.UseSerilogRequestLogging(opts
        => opts.EnrichDiagnosticContext = LogHelper.EnrichFromRequest);

    // ... Other middleware
}
```

&emsp;&emsp;现在，当您发出请求时，您将看到添加到 Serilog 结构化日志中的所有其他属性：
![ ](5.png)

&emsp;&emsp;只要您具有通过当前 HttpContext 可供中间件管道使用的值，就可以使用此方法。但是 MVC 的相关属性是个例外，它们是 MVC 中间件“内部”的特性，例如 action 名称或 RazorPage 处理程序名称。在下一篇文章中，我将展示如何将它们添加到 Serilog 请求日志中。

## 总结
&emsp;&emsp;默认情况下，用 Serilog 的请求日志记录中间件替换 ASP.NET Core 基础结构日志记录时，与开发环境的默认日志记录配置相比，您会丢失一些信息。在本文中，我展示了如何通过自定义 Serilog `RequestLoggingOptions`来添加这些附加属性。  
&emsp;&emsp;这样的做法非常简单 —— 您可以访问`HttpContext`，因此你可以检索它包含的任何可用的值，并将它们设置为`IDiagnosticContext`所提供的属性。这些属性将作为附加属性添加到 Serilog 生成的结构化日志中。在下一篇文章中，我将展示如何将 MVC 特定的属性值添加到请求日志中。敬请期待吧！

---

# 使用 Serilog.AspNetCore 记录 MVC 属性
&emsp;&emsp;在我上篇文章中，我描述了如何配置 Serilog 的 RequestLogging 中间件以向 Serilog 的请求日志摘要中添加其他属性（例如请求主机名或选定的端点名称）。这些属性都在`HttpContext`中可用，因此可以由中间件本身直接添加。  
&emsp;&emsp;其他属性，例如 MVC 特定的功能，像操作方法 ID，RazorPages 处理程序名称或 ModelValidationState，**仅**在 MVC 上下文中可用，因此 Serilog 的中间件不能直接访问。  
&emsp;&emsp;在本文中，我将展示如何创建`action`/`page`过滤器来为您记录这些属性，以便中间件可以在后续创建日志时访问。

> &emsp;&emsp;[Serilog 的创建者 Nicholas Blumhardt 之前已经解决了这个话题](https://nblumhardt.com/2019/10/serilog-mvc-logging/)。解决方案非常相似，尽管他在他的示例中创建了一个特性，您可以使用该特性来装饰 *actions / controllers* 。我在本文中跳过了这种方法，并要求将其全局应用，我希望这将是常见的解决方案。

## 记录来自 MVC 的其他信息
&emsp;&emsp;就目前而言，ASP.NET Core 中的一个特征是许多行为被 MVC “基础结构”锁定在了 MVC 框架内部来实现。端点路由是采用 MVC 功能并将其下移到核心框架中的首要工作之一。ASP.NET Core 团队一直在努力将更多 MVC 特定功能（例如模型绑定或操作结果）从 MVC 中移除，然后“下推”到核心框架中。[有关此内容的更多信息，请参见 Ryan Nowak 在 NDC 上对 Houdini 项目的讨论](https://www.youtube.com/watch?v=7dJBmV_psW0)。

&emsp;&emsp;但是，就目前情况而言，MVC 内仍然存在一些不容易从应用程序其他部分访问的特性。当我们考虑到我们的 [Serilog 的请求记录中间件](https://www.cnblogs.com/yilezhu/p/12215934.html)的时候，这意味着有些属性我们也是不容易记录的。例如：
* HandlerName（`OnGet`）
* ActionId（`1fbc88fa-42db-424f-b32b-c2d0994463f1`）
* ActionName（`MyController.SomeApiMethod (MyTestApp)`）
* RouteData（`{action = "SomeApiMethod", controller = "My", page = ""}`）
* ValidationState（`True`/`False`）

&emsp;&emsp;在上一篇文章中我展示了如何使用 RequestLogging 中间件的扩展方法通过使用`IDiagnosticContext`将附加属性写入 Serilog 的请求日志中。这也仅适用于在`HttpContext`可用的值。在这篇文章中，我将展示如何在[过滤器](https://docs.microsoft.com/en-us/aspnet/core/mvc/controllers/filters?view=aspnetcore-3.0)中使用`IDiagnosticContext`，以及将 MVC 特定值添加到日志中。我还将展示如何在 [page 过滤器](https://docs.microsoft.com/en-us/aspnet/core/razor-pages/filter?view=aspnetcore-3.0#implement-razor-page-filters-globally)中添加 RazorPages 特定的值（如`HandlerName`）。

## 使用自定义过滤器记录 MVC 属性
&emsp;&emsp;过滤器相当于为每个请求运行的类似于 MVC 的微型中间件管道。.NET Core MVC 中有多种类型的过滤器，每种类型的过滤器在 MVC 过滤器管道中的有着不同的用途（[有关更多详细信息，请参见此文章](https://andrewlock.net/asp-net-core-in-action-filters/)）。在本文中，我们将使用最常见的过滤器之一，即 Action 过滤器。  
&emsp;&emsp;Action 过滤器在执行 MVC 操作方法之前和之后运行。他们可以访问许多 MVC 属性的值，例如正在执行的 Action 及其将被调用的参数。  
&emsp;&emsp;下面的 Action 过滤器直接实现`IActionFilter`。该`OnActionExecuting`方法在调用 action 方法之前被调用，并将额外的 MVC 特定属性添加到通过构造函数传入的`IDiagnosticContext`中。
```cs
public class SerilogLoggingActionFilter : IActionFilter
{
    private readonly IDiagnosticContext _diagnosticContext;

    public SerilogLoggingActionFilter(IDiagnosticContext diagnosticContext)
    {
        _diagnosticContext = diagnosticContext ?? throw new ArgumentNullException(nameof(diagnosticContext));
    }

    public void OnActionExecuting(ActionExecutingContext context)
    {
        _diagnosticContext.Set("RouteData", context.ActionDescriptor.RouteValues);
        _diagnosticContext.Set("ActionName", context.ActionDescriptor.DisplayName);
        _diagnosticContext.Set("ActionId", context.ActionDescriptor.Id);
        _diagnosticContext.Set("ValidationState", context.ModelState.IsValid);
    }
    // Required by the interface
    public void OnActionExecuted(ActionExecutedContext context)
    {

    }
}
```

&emsp;&emsp;在将 MVC 服务添加到应用程序中时，可以在以下位置全局注册过滤器`Startup.ConfigureServices()`：
```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers(opts =>
    {
        opts.Filters.Add<SerilogLoggingPageFilter>();
    });
    // ... other service registration
}
```

> &emsp;&emsp;无论你使用`AddControllers`，`AddControllersWithViews`，`AddMvc`，或`AddMvcCore`的方式你都可以采用同样的方式来添加全局过滤器。

&emsp;&emsp;有了这个配置之后，如果你调用一个 MVC 控制器，你在 Serilog 的请求日志消息中会看到额外的数据（`ActionName`，`ActionId`，和`RouteData`，`ValidationState`）记录：
![ ](6.png)

&emsp;&emsp;您可以在此处将所需的任何其他数据添加到日志中。只需注意记录参数值 —— 切记不要记录敏感或个人身份信息！

> &emsp;&emsp;[Nicholas Blumhardt 在他的帖子中建议](https://nblumhardt.com/2019/10/serilog-mvc-logging/)的 Action 过滤器是从`ActionFilterAttribute`派生的，因此可以将其直接用作控制器和 Action 的特性。不幸的是，这意味着您必须使用服务定位来从每个请求的`HttpContext`中检索单例的`IDiagnosticContext`。  
我的方法可以改用构造函数注入，但是不建议将其用作属性，因此必须如上所述全局使用。而且，MVC 将在我的实现中使用作用域生存期，而不是单例，因此它会在每个请求中创建一个新实例。

&emsp;&emsp;如果要记录其他集中 MVC 过滤器中的值，则可以以相同的方式实现其他过滤器，例如资源过滤器，结果过滤器或授权过滤器。

## 使用自定义 page 过滤器记录 RazorPages 属性
&emsp;&emsp;上面实现的`IActionFilter`过滤器在 MVC 和 API 控制器上能够正常运行，但它**不会**对 RazorPages 起作用。如果要为选择的给定 Razor 页面记录 HandlerName，则需要创建一个自定义的`IPageFilter`。

&emsp;&emsp;页面过滤器直接类似于 Action 过滤器，但它们仅适用于 Razor 页面。以下示例从`PageHandlerSelectedContext`中检索处理程序名称并将其记录为属性`RazorPageHandler`。在这种情况下，还需要一些样板代码，但过滤器的功能还是非常基础的 —— 调用`IDiagnosticContext.Set()`以记录属性。
```cs
public class SerilogLoggingPageFilter : IPageFilter
{
    private readonly IDiagnosticContext _diagnosticContext;

    public SerilogLoggingPageFilter(IDiagnosticContext diagnosticContext)
    {
        _diagnosticContext = diagnosticContext ?? throw new ArgumentNullException(nameof(diagnosticContext));
    }
    //Required by the interface
    public void OnPageHandlerExecuted(PageHandlerExecutedContext context)
    {
    }
    public void OnPageHandlerExecuting(PageHandlerExecutingContext context)
    {
    }
    public void OnPageHandlerSelected(PageHandlerSelectedContext context)
    {
        var name = context.HandlerMethod?.Name ?? context.HandlerMethod?.MethodInfo.Name;
        if (name != null)
        {
            _diagnosticContext.Set("RazorPageHandler", name);
        }
    }
}
```

&emsp;&emsp;请注意，我们之前编写的`IActionFilter`代码**不会**在 Razor Pages 上运行，因此，如果您也想记录 RazorPages `RouteData`或`ValidationState`等的其他详细信息，则也需要在此处添加它。该`context`属性包含您可能需要的大多数属性，例如`ModelState`和`ActionDescriptor`。

&emsp;&emsp;接下来，您需要在`Startup.ConfigureServices()`方法中注册页面过滤器：
```cs
public void ConfigureServices(IServiceCollection services)
{
    //services.AddMvcCore(
    //    opts => opts.Filters.Add<SerilogLoggingPageFilter>()
    //    );
    services.AddRazorPages().AddMvcOptions(
        opts => opts.Filters.Add<SerilogLoggingPageFilter>()
        ) ;
}
```

&emsp;&emsp;添加过滤器后，对 “Razor 页面”的请求现在可以看到添加的附加属性，`IDiagnosticContext`这些属性将添加到 Serilog 请求日志中。请参见下图中的`RazorPageHandler`属性：
![ ](7.png)

## 总结
&emsp;&emsp;默认情况下，当用 Serilog 的请求日志记录中间件替换 ASP.NET Core 基础结构中的日志记录时，您会丢失一些信息（与开发环境的默认配置相比）。在本文中，我将展示如何自定义 Serilog，`RequestLoggingOptions`以重新添加特定于 MVC 的其他属性。  
&emsp;&emsp;要将与 MVC 相关的属性添加到 Serilog 请求日志中，请创建一个`IActionFilter`并使用`IDiagnosticContext.Set()`来添加属性。要将与 Razor 页面相关的属性添加到 Serilog 请求日志中，请在`IPageFilter`中使用`IDiagnosticContext`的相同方法创建和添加属性。

&emsp;&emsp;下一节让我们一起探讨下如何从 Serilog 请求记录中排除运行状况检查端点。

---

# 从 Serilog 请求日志记录中排除健康检查端点
&emsp;&emsp;在本系列的前几篇文章中，我描述了如何配置 Serilog 的 RequestLogging 中间件以向 Serilog 的请求日志摘要中添加附加属性，例如请求主机名或选定的端点名称。我还展示了如何使用过滤器将 MVC 或 RazorPage 特定的属性添加到摘要日志。  
&emsp;&emsp;在本文中，我将展示如何过滤掉某个特定请求的摘要日志消息。当您有一个访问比较频繁的端点时，这非常有用，因为为每个请求都进行记录几乎没有什么价值。

## 健康检查访问较频繁
&emsp;&emsp;这篇文章的动机来自我们在 Kubernetes 中运行应用程序时看到的行为。Kubernetes 使用两种类型的“健康检查”（或“探针”）来检查应用程序是否正常运行：liveness probes 和 readiness probes。您可以将探测配置为向应用程序发出 HTTP 请求，作为应用程序正常运行的指示器。

> &emsp;&emsp;从 Kubernetes 1.16 版开始，存在第三种探针，即 [startup probe](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#define-startup-probes)。

&emsp;&emsp;在 ASP.NET Core 2.2+ 中提供的[健康检查终结点](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/health-checks)非常适合这些探针。您可以设置一个简单，没有任何返回值的健康检查，该健康检查对每个请求返回`200 OK`的响应，以使 Kubernetes 知道您的应用程序没有崩溃。  
&emsp;&emsp;在 ASP.NET Core 3.x 中，可以使用终结点路由来配置健康检查。您必须在 *Startup.cs* 中的`ConfigureServices`中通过调用`AddHealthChecks()`来添加必须的服务，并在`Configure`中使用`MapHealthChecks()`来添加健康检查终结点：
```cs
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // ..other service configuration

        services.AddHealthChecks(); // Add health check services
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        // .. other middleware
        app.UseRouting();
        app.UseAuthorization();
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapHealthChecks("/health"); //Add health check endpoint
            endpoints.MapControllers();
        });
    }
}
```

&emsp;&emsp;在上面的示例中，向`/healthz`发送请求将调用运行状况检查终结点。由于我没有配置任何运行的健康检查，因此只要应用程序正在运行，端点将始终返回`200`响应：
![ ](8.png)

&emsp;&emsp;这里存在的唯一的问题是 Kubernetes 将非常频繁的调用这个终结点。当然，确切的频率由您决定，但每 10 秒检查一次应该是很常见的。但是如果你想让 Kubernetes 可以快速重启有故障的 Pod 的话，您就需要一个相对较高的频率了。  
&emsp;&emsp;这本身不是问题；Kestrel 每秒可以处理数百万个请求，因此这不是性能问题。这里令人比较烦恼的问题是每个请求都会生成一定数量的日志。虽然它[没有 MVC 基础架构的请求所示的那么多 —— 每个请求 10 个日志](https://www.cnblogs.com/yilezhu/p/12215934.html)，但是即使每个请求只有 1 个日志（就像我们从 Serilog.AspNetCore 获得的那样）都可能会令人不快。  
&emsp;&emsp;这里的主要问题是成功进行健康检查请求的日志实际上并未告诉我们任何有用的信息。它们与任何业务活动都不相关，它们纯粹是基础设施。这里如果能够跳过这些请求的 Serilog 请求摘要日志会很好。在下一部分中，我将介绍我所想出的方法，该方法依赖于本系列前面几篇文章的内容，并在其基础上做出更改。

## 定制用于 Serilog 请求日志的日志级别
&emsp;&emsp;在上一篇文章中，我展示了如何在 Serilog 请求日志中包括[所选终结点](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/routing?view=aspnetcore-3.0#endpoint-routing-differences-from-earlier-versions-of-routing)。我的方法是在注册 Serilog 中间件时为`RequestLoggingOptions.EnrichDiagnosticContext`属性提供一个自定义函数：
```cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // ... Other middleware

    app.UseSerilogRequestLogging(opts
        // EnrichFromRequest helper function is shown in the previous post
        => opts.EnrichDiagnosticContext = LogHelper.EnrichFromRequest); 

    app.UseRouting();
    app.UseAuthorization();
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapHealthChecks("/healthz"); //Add health check endpoint
        endpoints.MapControllers();
    });
}
```

&emsp;&emsp;`RequestLoggingOptions`具有另一个属性，`GetLevel`该属性的`Func<>`被用于确定应用于给定请求日志的日志记录级别。默认情况下，[它设置为以下功能](https://github.com/serilog/serilog-aspnetcore/blob/dev/src/Serilog.AspNetCore/SerilogApplicationBuilderExtensions.cs#L31-L36)：
```cs
public static class SerilogApplicationBuilderExtensions
{
    static LogEventLevel DefaultGetLevel(HttpContext ctx, double _, Exception ex) =>
        ex != null
            ? LogEventLevel.Error 
            : ctx.Response.StatusCode > 499 
                ? LogEventLevel.Error 
                : LogEventLevel.Information;
}
```

&emsp;&emsp;此函数检查是否为请求引发了异常，或者响应代码是否为`5xx`错误。如果是这样，它将创建一个`Error`级别的摘要日志，否则将创建一个`Information`级别日志。

&emsp;&emsp;假设您希望将摘要日志记录为`Debug`而不是`Information`。首先，您将创建一个具有以下所需逻辑的辅助函数，如下所示：
```cs
public static class LogHelper
{
    public static LogEventLevel CustomGetLevel(HttpContext ctx, double _, Exception ex) =>
        ex != null
            ? LogEventLevel.Error 
            : ctx.Response.StatusCode > 499 
                ? LogEventLevel.Error 
                : LogEventLevel.Debug; //Debug instead of Information
}
```

&emsp;&emsp;然后，您可以在调用时设置级别功能`UseSerilogRequestLogging()`：
```cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // ... Other middleware

    app.UseSerilogRequestLogging(opts => {
        opts.EnrichDiagnosticContext = LogHelper.EnrichFromRequest;
        opts.GetLevel = LogHelper.CustomGetLevel; // Use custom level function
    });

    //... other middleware
}
```

&emsp;&emsp;现在，您的请求摘要日志将全部记录为`Debug`，除非发生错误（[Seq](https://datalust.co/seq/) 的屏幕截图）：
![ ](9.png)

&emsp;&emsp;但这如何解决我们的冗长日志的问题呢？

&emsp;&emsp;当你在配置 Serilog 时，你通常应该会定义一个最低请求级别。例如，以下简单配置将默认级别设置为`Debug()`，并将其写入控制台接收器：
```cs
Log.Logger = new LoggerConfiguration()
    .MinimumLevel.Debug()
    .WriteTo.Console()
    .CreateLogger();
```

&emsp;&emsp;因此，过滤日志的最简单方法是使日志级别低于`MinimumLevel`记录器配置中指定的级别。一般而言，如果使用最低级别`Verbose`，它将几乎总是被过滤掉。  
&emsp;&emsp;困难之处在于我们不想总是将`Verbose`用作摘要日志的日志级别。如果这样做，我们将不会获得任何非错误的请求日志，而 Serilog 中间件将变得毫无意义！  
&emsp;&emsp;相反，我们希望将日志级别设置为`Verbose`**仅**针对运行健康检查端点的请求。在下一节中，我将展示如何在不影响其他请求的情况下识别这些请求。

## 将自定义日志级别用于健康检查终结点请求
&emsp;&emsp;我们需要的是能够在写入摘要日志时识别出健康检查的请求的能力。如前所示，该`GetLevel()`方法将当前`HttpContext`作为参数，因此理论上有一些可行性。对我来说，最明显的做法是：
* 将`HttpContext.Request`路径与已知的健康检查路径列表进行比较
* 当健康检查终结点被请求时，使用选定的端点元数据来进行标识

&emsp;&emsp;第一种选择是最明显的，但是它真的不值得尝试。一旦你陷入其中，你会发现你必须开始复制请求路径并处理各种边缘情况，因此在这里我将跳过该情况。  
&emsp;&emsp;第二种方法使用了与我上一篇文章中使用的方法类似，在该方法中，我们获得了`EndpointRoutingMiddleware`为给定请求选择的`IEndpointFeature`。此功能（如果存在）提供了所选端点的显示名称和路由数据等详细信息。

&emsp;&emsp;如果我们假设健康检查是[使用默认显示名称注册](https://github.com/aspnet/AspNetCore/blob/master/src/Middleware/HealthChecks/src/Builder/HealthCheckEndpointRouteBuilderExtensions.cs#L18)的，即"`Health checks`"，则我们可以使用`HttpContext`来标识“健康检查”的请求，如下所示：
```cs
public static class LogHelper
{
    private static bool IsHealthCheckEndpoint(HttpContext ctx)
    {
        var endpoint = ctx.GetEndpoint();
        if (endpoint is object) // same as !(endpoint is null)
        {
            return string.Equals(
                endpoint.DisplayName, 
                "Health checks",
                StringComparison.Ordinal);
        }
        // No endpoint, so not a health check endpoint
        return false;
    }
}
```

&emsp;&emsp;我们可以将此功能与[默认`GetLevel`功能](https://github.com/serilog/serilog-aspnetcore/blob/dev/src/Serilog.AspNetCore/SerilogApplicationBuilderExtensions.cs#L31-L36)的自定义版本结合使用，以确保运行健康检查请求的摘要日志使用`Verbose`级别，当发生错误时使用`Error`而其他请求则使用`Information`：
```cs
public static class LogHelper
{
    public static LogEventLevel ExcludeHealthChecks(HttpContext ctx, double _, Exception ex) => 
        ex != null
            ? LogEventLevel.Error 
            : ctx.Response.StatusCode > 499 
                ? LogEventLevel.Error 
                : IsHealthCheckEndpoint(ctx) // Not an error, check if it was a health check
                    ? LogEventLevel.Verbose // Was a health check, use Verbose
                    : LogEventLevel.Information;
        }
}
```

&emsp;&emsp;这个嵌套的三目运算符有一个额外的逻辑 —— 对于无错误，我们检查是否选择了显示名为 “Health check” 的端点，如果选择了，则使用级别`Verbose`，否则使用`Information`。

> &emsp;&emsp;您可以进一步推广此代码，以允许传入其他显示名称或其他自定义使用的日志级别。为了简单起见，我在这里没有这样做，但是 [GitHub 上的相关示例代码显示了如何执行此操作](https://github.com/andrewlock/blog-examples/tree/master/SerilogRequestLogging)。

&emsp;&emsp;剩下的就是更新 Serilog 中间件`RequestLoggingOptions`以使用您的新功能：
```cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // ... Other middleware

    app.UseSerilogRequestLogging(opts => {
        opts.EnrichDiagnosticContext = LogHelper.EnrichFromRequest;
        opts.GetLevel = LogHelper.ExcludeHealthChecks; // Use the custom level
    });

    //... other middleware
}
```

&emsp;&emsp;这时候当你运行应用程序后检查日志时，您会看到标准请求的普通请求日志，但没有健康检查的日志（除非发生错误！）。在下面的屏幕截图中，我将 Serilog 配置为也记录`Verbose`日志，以便您可以查看运行状况检查请求 —— 通常会将它们过滤掉！
![ ](10.png)

## 总结
&emsp;&emsp;在本文中，我展示了如何为 Serilog 中间件的`RequestLoggingOptions`提供一个自定义函数，该函数定义了要为给定请求的日志使用的`LogEventLevel`。例如，我展示了如何使用它将默认级别更改为 Debug。如果您选择的级别低于最低级别，它将被完全过滤掉，并且不会被记录。  
&emsp;&emsp;我还展示了您可以使用这种方法来过滤通过调用健康检查端点生成的公共（低级别的）请求日志。一般来说，这些请求只有在指出问题时才有意义，但它们通常也会在成功时生成请求日志。由于这些端点被频繁调用，因此它们可以显著增加写入的日志数量（无用）。  
&emsp;&emsp;本文中的方法是检查选定的`IEndpointFeature`并检查它是否具有显示名称 “Health checks”。如果是，请求日志将使用`Verbose`级别写入，这通常会被过滤掉。为了更灵活，您可以自定义在这个帖子中显示的日志来处理多个端点名称，或者任何其他的标准。