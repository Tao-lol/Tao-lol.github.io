---
title: 编程小知识
tags:
  - .NET
categories:
  - 编程
date: 2019-12-01 17:01:01
---

&emsp;
<!--more-->

# .NET
#### .NET Core 依赖注入的三大生命周期
> https://mp.weixin.qq.com/s/PUdolCF8I9GCXlt1DI_BMw

三大生命周期：
1. 多例模式 Transient：每解析一次接口，就会实例化一个对象。每次对象都是唯一且不同的。每次解析请求，实例的都是一个全新的对象。
2. Scoped 英文释为范围，区域：第一次请求，它会实例出一个对象 `IA a = new A();` 并缓存下来，接下来的每一次请求，会判断是否是同一个 HttpContext，如若是同一个，那么它仍然返回这个 `a` 对象。  
这里解释下，什么叫一次请求。同一次请求，它的 HttpContext 肯定是同一个，所以返回的都是 a。这里 Scoped 其实就是一次 http 请求的作用域，同一次请求，可能多次请求其他的服务，也可能多次请求同一个控制器，只是这个人进了商场买东西，不管做什么，还是这个人。但是它一旦出了这个商场（结束了本次 http 请求）再次进入商场，就是新的请求了，就需要重新过安检（虽然还是那个人）。
3. 单例模式 Singleton：只要服务器不嗝屁，不管解析多少次接口，拿到的都是 a。换句话说就是，只要皇帝不死，太子一直是太子！从商场建成开业到商场倒闭关门，此次程序服务过程中，单例返回的始终是这个 a。

---

## 控制台应用程序

### 创建单实例应用程序
```cs
using System;
using System.Threading;
using System.Reflection;

public class Program
{
    public static void Main()
    {
        // Indicates whether this is the first application instance
        bool firstApplicationInstance;

        // Obtain the mutex name from the full assembly name
        string mutexName = Assembly.GetEntryAssembly().FullName;

        using(Mutex mutex = new Mutex(false, mutexName, out firstApplicationInstance))
        {
            if(!firstApplicationInstance)
            {
                Console.WriteLine("This application is already running.");
            }
            else
            {
                Console.WriteLine("ENTER to shut down");
                Console.ReadLine();
            }
        }
    }
}
```

---

## ASP.NET Core

#### 添加 HTTPS 支持
> HTTPS: https://www.cnblogs.com/JulianHuang/p/11858800.html  
> HSTS: https://www.cnblogs.com/JulianHuang/p/12156997.html

&emsp;&emsp;我们利用 Visual Studio 2019 项目模板构建 ASP.Net Core 项目，勾选 HTTPS 支持，会默认添加 HTTPS 支持：
```cs
app.UseHttpsRedirection()  // 强制 Http 请求跳转到 Https
app.UseHsts()
```
&emsp;&emsp;HSTS（HTTP Strict Transport Protocol）的作用是强制浏览器使用 HTTPS 与服务器创建连接，避免原有的 301 重定向 Https 时可能发生中间人劫持。  
&emsp;&emsp;服务器开启 HSTS 的方法是，当客户端通过HTTPS发出请求时，在服务器返回的超文本传输协议响应头中包含 Strict-Transport-Security 字段。非加密传输时设置的 HSTS 字段无效。它告诉浏览器为特定的主机头和特定的时间范围缓存证书。  

&emsp;&emsp;若使用 Kestrel 作为边缘（face-to-internet）Web 服务器， 参见下面的服务配置：
* 为 STS header 设置了 preload 参数，Preload 不是 RFC HSTS 规范的一部分，但是浏览器支持在全新安装时预加载 HSTS 网站
* 指定子域或排除的子域 使用 HSTS 协议
* 设置浏览器缓存 [访问站点的请求均使用 HTTPS 协议] 这一约定的时间，默认是 30 天。

```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();

    services.AddHsts(options =>
    {
        options.Preload = true;
        options.IncludeSubDomains = true;
        options.MaxAge = TimeSpan.FromDays(60);
        options.ExcludedHosts.Add("example.com");
        options.ExcludedHosts.Add("www.example.com");
    });

    services.AddHttpsRedirection(options =>
    {
        options.RedirectStatusCode = StatusCodes.Status307TemporaryRedirect;
        options.HttpsPort = 5001;
    });
}
```

> 请注意： UseHsts 对于本地回送 hosts 并不生效。
> * `localhost`：IPv4 回送地址
> * `127.0.0.1`：IPv4 回送地址
> * `[::1]`：IPv6 回送地址
> 
> 这也是开发者在本地启动时 抓不到`Strict-Transport-Security`响应头的原因。

---

#### 自定义中间件
> https://mp.weixin.qq.com/s/PUdolCF8I9GCXlt1DI_BMw

中间件是一种装配到应用程序管道以处理请求和响应的软件，每个组件可以：
1. 选择是否将请求传递到管道中的下一个组件。
2. 可在调用管道中的下一个组件前后执行工作。即中间件的事前逻辑和事后逻辑。

如果出现错误，它不会再进行下一个组件，而是原路返回。  

中间件的写法：
1. 命名：类名后缀 Middleware
2. 构造函数：注入 RequestDelegate next
3. 异步方法命名：async Task Invoke(HttpContext context)

```cs
public class CustomMiddleware
{
    private readonly RequestDelegate _next;

    public CustomMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext context)
    {
        // Do Something

        await _next(context);
    }
}
```

```cs
public static class CustomExtension
{
    public static IApplicationBuilder UseCustom(this IApplicationBuilder app)
    {
        return app.UseMiddleware<CustomMiddleware>();
    }
}
```

```cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    // app.UseMiddleware<CustomMiddleware>();
    app.UseCustom();

    // app.UseXXX
}
```

---

#### ASP.NET Core 主机地址过滤 HostFiltering
> https://www.cnblogs.com/yyfh/p/11855862.html

&emsp;&emsp;`HostFilteringMiddleware`做的是对请求主机头的限制，也相当于一个请求主机头白名单，标识着某些主机头你可以访问，其余的你别访问了我这边未允许。  

##### 如何使用
1. 配置`HostFilteringOptions`
```cs
services.PostConfigure<HostFilteringOptions>(options =>
{
    if (options.AllowedHosts == null || options.AllowedHosts.Count == 0)
    {
        // "AllowedHosts": "localhost;127.0.0.1;[::1]"
        var hosts = Configuration["AllowedHosts"]?.Split(new[] { ';' }, StringSplitOptions.RemoveEmptyEntries);
        // Fall back to "*" to disable.
        options.AllowedHosts = (hosts?.Length > 0 ? hosts : new[] { "*" });
    }
});
```

> `HostFilteringOptions`：
> * `AllowedHosts`：允许访问的 Host 主机
> * `AllowEmptyHosts`：是否允许请求头 Host 的值为空访问，默认为 true
> * `IncludeFailureMessage`：返回错误信息，默认为 true

2. 在 `Configure` 方法中添加 HostFiltering 中间件
```cs
public void Configure(Microsoft.AspNetCore.Builder.IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseHostFiltering();
    app.Run(context =>
    {
        return context.Response.WriteAsync("Hello World! " + context.Request.Host);
    });
}
```

3. `appsettings.json`
```json
{
  "AllowedHosts": "127.0.0.1"
}
```

![ ](1098068-20191114112620796-67212582.gif)

&emsp;&emsp;当我们请求带着 Host 头去访问的时候，通过该中间件判断该 Host 头是否在我们预先配置好的里面，如果在里面那么就继续请求下一个中间件，如果说不在则返回 400。

---

#### ASP.NET Core 3.x 并发限制
> https://www.cnblogs.com/yyfh/p/11843358.html

&emsp;&emsp;`Microsoft.AspNetCore.ConcurrencyLimiter` ASP.NET Core 3.0 后增加的，用于传入的请求进行排队处理，避免线程池的不足。  
&emsp;&emsp;我们日常开发中可能常做的给某 Web 服务器配置连接数以及，请求队列大小，那么今天我们看看如何在通过中间件形式实现一个并发量以及队列长度限制。

##### 添加 Nuget
```
Install-Package Microsoft.AspNetCore.ConcurrencyLimiter
```

##### Queue 策略
```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddQueuePolicy(options =>
    {
        //最大并发请求数
        options.MaxConcurrentRequests = 2;
        //请求队列长度限制
        options.RequestQueueLimit = 1;
    });
    services.AddControllers();
}
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    //添加并发限制中间件
    app.UseConcurrencyLimiter();
    app.Run(async context =>
    {
        Task.Delay(100).Wait(); // 100ms sync-over-async

        await context.Response.WriteAsync("Hello World!");
    });
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseHttpsRedirection();

    app.UseRouting();

    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```

##### Stack 策略
```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddStackPolicy(options =>
    {
        //最大并发请求数
        options.MaxConcurrentRequests = 2;
        //请求队列长度限制
        options.RequestQueueLimit = 1;
    });
    services.AddControllers();
}
```

##### 总结
基于栈结构的特点，在实际应用中，通常只会对栈执行以下两种操作：
* 向栈中添加元素，此过程被称为"进栈"（入栈或压栈）；
* 从栈中提取出指定元素，此过程被称为"出栈"（或弹栈）。

队列存储结构的实现有以下两种方式：
* 顺序队列：在顺序表的基础上实现的队列结构；
* 链队列：在链表的基础上实现的队列结构。

---

### Web API
![https://www.cnblogs.com/cgzl/p/11924700.html](restfulapi.png)

---

## WPF

### 为什么在 .NET Core 上使用 Windows desktop
> https://www.cnblogs.com/muran/p/11808508.html

#### 使用 ReadyToRun 优化 .NET Core 应用
&emsp;&emsp;通过将应用程序程序集编译为 ReadyToRun（R2R）格式，可以缩短 .NET Core 应用程序的启动时间。R2R 是一种提前（AOT）编译的形式。它是 .NET Core 3.0 中的发布时选择功能。  
&emsp;&emsp;R2R 二进制文件通过减少应用程序加载时 JIT 需要完成的工作量来提高启动性能。R2R 二进制文件较大，因为它们既包含中间语言（IL）代码（某些情况下仍然需要此代码），也包含相同代码的本机版本，以改善启动。  

要启用 ReadyToRun 编译：
* 将 `PublishReadyToRun` 属性设置为 `true`。
* 使用显式发布 `RuntimeIdentifier`。

&emsp;&emsp;注意：编译应用程序程序集时，生成的本机代码特定于平台和体系结构（这就是发布时必须指定有效的 RuntimeIdentifier 的原因）。  

下面是一个例子：
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp3.0</TargetFramework>
    <PublishReadyToRun>true</PublishReadyToRun>
  </PropertyGroup>
</Project>
```

并使用以下命令发布：
```
dotnet publish -r win-x64 -c Release
```

注意：
* `RuntimeIdentifier` 可以将其设置为其他操作系统或芯片。也可以在项目文件中设置。
* 如果在发布过程中遇到错误，请添加 `<PublishReadyToRunShowWarnings>true</PublishReadyToRunShowWarnings>` 查看详情日志。

#### Assembly linking
&emsp;&emsp;.NET core 3.0 SDK 附带了一个工具，该工具可以通过分析 IL 和修剪未使用的程序集来减小应用程序的大小。这是 .NET Core 3.0 中的另一个发布时选择加入功能。  
&emsp;&emsp;借助 .NET Core，始终可以发布包含运行代码所需的一切的自包含应用程序，而无需在部署目标上安装 .NET。在某些情况下，该应用仅需要框架的一小部分即可运行，并且可能仅包含所使用的库就可以变得更小。  
&emsp;&emsp;使用 [IL 链接器](https://github.com/mono/linker)扫描应用程序的 IL，以检测实际需要哪些代码，然后修剪未使用的框架库。这可以大大减小某些应用程序的大小。通常，类似小型工具的控制台应用程序受益最大，因为它们倾向于使用框架的较小子集，并且通常更易于调整。  

要使用链接器：
* 将 `PublishTrimmed` 属性设置为 `true`。
* 使用显式发布 `RuntimeIdentifier`。

下面是一个例子：
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp3.0</TargetFramework>
    <PublishTrimmed>true</PublishTrimmed>
  </PropertyGroup>
</Project>
```

并使用以下命令发布：
```
dotnet publish -r win-x64 -c Release
```

&emsp;&emsp;注意：`RuntimeIdentifier` 可以将其设置为其他操作系统或芯片。也可以在项目文件中设置。  
&emsp;&emsp;经测试对于 helloworld 应用程序，大小从 〜 68 MB 减小到 〜 28 MB。  
&emsp;&emsp;修剪后使用反射或相关动态功能的应用程序或框架（包括 ASP.NET Core 和 WPF）通常会中断，因为链接器不了解这种动态行为，并且通常无法确定反射所需的框架类型在运行时。要修剪此类应用程序，您需要告知链接器有关代码中以及依赖的任何包或框架中反射所需要的任何类型。修剪后一定要测试应用程序。针对这个问题微软在 .NET 5 上正在努力改善。  
&emsp;&emsp;有关 IL Linker 的更多信息，请参阅[文档](https://aka.ms/dotnet-illink)，或访问 [mono / linker 存储库](https://github.com/mono/linker)。  
&emsp;&emsp;Assembly linking 和 ReadyToRun compiler 可用于同一应用程序。通常，Assembly linking 使应用程序更小，ready-to-run compiler 使应用程序更大一些，但在性能上有明显优势。可以在各种配置中进行测试以了解每个选项的影响。

#### 发布单文件可执行文件
&emsp;&emsp;可以使用发布单个文件的可执行文件 `dotnet publish`。这种形式的单个 `EXE` 实际上是一个自解压缩的可执行文件。它包含所有依赖项（包括本地依赖项）作为资源。在第一次启动时，它将所有依赖项复制到一个临时目录，并在该目录中加载它们。它只需要解压缩依赖项一次。当再次启动时将会很快启动，并且没有任何损失。  
&emsp;&emsp;可以通过将 `PublishSingleFile` 属性添加到项目文件或在命令行上添加新的参数来启用此发布选项。  

要生成一个独立的单个 EXE 应用程序，在这种情况下，对于 64 位 Windows：
```
dotnet publish -r win10-x64 /p:PublishSingleFile=true
```

注意：
* `RuntimeIdentifier` 可以将其设置为其他操作系统或芯片。也可以在项目文件中设置。
* 关于临时目录，请参考 [Extracting Bundled Files to Disk](https://github.com/dotnet/designs/blob/master/accepted/single-file/extract.md#extraction-location)

&emsp;&emsp;有关更多信息，请参见[单文件捆绑器](https://github.com/dotnet/core-setup/pull/5286)。  
&emsp;&emsp;Assembly trimmer, ahead-of-time compilation（通过 crossgen）和单个文件捆绑都是 .NET Core 3.0 中的所有新功能，可以一起使用，也可以单独使用。

#### 综合使用
&emsp;&emsp;通过设置属性 `<PublishSingleFile>`，`<RuntimeIdentifier>`、`<PublishTrimmed>`、`<PublishReadyToRun>`、`<PublishReadyToRunShowWarnings>` 在发布配置文件中，能够将修剪、ahead-of-time compilation 后的自包含应用程序部署为单个 .exe 文件，如下面的示例所示。
```xml
<PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp3.0</TargetFramework>
    <PublishSingleFile>true</PublishSingleFile>
    <RuntimeIdentifier>win-x64</RuntimeIdentifier>
    <PublishReadyToRun>true</PublishReadyToRun>
    <PublishReadyToRunShowWarnings>true</PublishReadyToRunShowWarnings>
    <PublishTrimmed>true</PublishTrimmed>
</PropertyGroup>
```

&emsp;&emsp;然后使用 Visual Studio 发布工具或者命令 `dotnet publish -c release` 发布。

#### MSIX
&emsp;&emsp;如果你正在寻找一种将应用程序分发给最终用户的方法，那么将其打包为 [MSIX](https://docs.microsoft.com/en-us/windows/msix/) 可能比创建单个 `.exe` 文件更好。`PublishSingleFile` 提供了一个包含所有应用程序依赖项的自解压 ZIP 文件，而 MSIX 提供了干净可靠的 Windows 集成应用程序安装和卸载。《MSDN 杂志》上写了[一篇文章](https://msdn.microsoft.com/en-us/magazine/mt833462)，不仅展示了如何打包应用程序，而且还展示了如何为 MSIX 包设置持续集成（CI），持续部署（CD）和自动更新。

---

### ViewModel 实现 INotifyPropertyChanged 接口
> https://mp.weixin.qq.com/s/QfuOWvqaSK6NcwsE7N-ojQ

```cs
public class ViewModel : INotifyPropertyChanged
{
    public event PropertyChangedEventHandler PropertyChanged;

    // [CallerMemberName] 特性要求 propertyName 具有默认值
    protected virtual void OnPropertyChanged([CallerMemberName] string propertyName = null)
    {
        PropertyChanged?.Invoke(this, new PropertyChangedEventArgs(propertyName));
    }

    private int property;

    public int Property
    {
        get { return property; }
        set
        {
            if (value != property)
            {
                property = value;
                OnPropertyChanged();
            }
        }
    }
}
```

---

## UnitTest
### 框架对比
![ ](ComparingUnitTestFrameworks.png)

# C&#35;

## 变量的命名
基本的变量命名规则如下：
* 变量名的第一个字符必须是字母、下划线 ( _ ) 或 @。
* 其后的字符可以是字母、下划线或数字。

---

## 构建某个类型的某个实例时系统所执行的操作
构建某个类型的某个实例时系统所执行的操作：
1. 把存放静态变量的空间清零。
2. 执行静态变量的初始化语句。
3. 执行基类的静态构造函数。
4. 执行（本类的）静态构造函数。
5. 把存放实例变量的空间清零。
6. 执行实例变量的初始化语句。
7. 适当地执行基类的实例构造函数。
8. 执行（本类的）实例构造函数。

&emsp;&emsp;以后如果还要构造该类型的实例，那么会直接从第5步开始执行，因为类级别的初始化工作只执行一次就够了。此外，可以通过链式调用构造函数的办法来优化第6、7两步，使得编译器在制作程序码时不再生成重复的指令。

---

## 类继承时基类与子类应该做到的几点
在类的继承体系中，位于根部的那个基类应该做到以下几点：
* 实现 IDisposable 接口，以便释放资源。
* 如果本身含有非托管资源，那就添加 finalizer，以防客户端忘记调用 Dispose() 方法。若是没有非托管资源，则不用添加 finalizer。
* Dispose 方法与 finalizer（如果有的话）都把释放资源的工作委派给虚方法，使得子类能够重写该方法，以释放它们自己的资源。

继承体系中的子类应该做到以下几点：
* 如果子类中有自己的资源需要释放，那就重写由基类所定义的那个虚方法，若是没有，则不用重写该方法。
* 如果子类自身的某个成员字段表示的是非托管资源，那么就实现 finalizer，若没有这样的字段，则不用实现 finalizer。
* 记得调用基类的同名函数。

---

## 实现 IDisposable.Dispose() 方法时的注意点
实现 IDisposable.Dispose() 方法时，要注意以下四点：
1. 把非托管资源全都释放掉。
2. 把托管资源全都释放掉（这也包括不再订阅早前关注的那些事件）。
3. 设定相关的状态标志，用以表示该对象已经清理过了。如果对象已经清理过了之后还有人要访问其中的公有成员，那么你可以通过此标志得知这一状况，从而令这些操作抛出 ObjectDisposedException。
4. 阻止垃圾回收器重复清理该对象。这可以通过 GC.SuppressFinalize(this) 来完成。

---

## 死锁的发生必须满足的条件
死锁的发生必须满足以下 4 个基本条件：  
（1）排他或互斥（Mutual Exclusion）：一个线程（ThreadA）独占一个资源，没有其他线程（ThreadB）能获取相同的资源。  
（2）占有并等待（Hold and wait）：一个排他的线程（ThreadA）请求获取另一个线程（ThreadB）占有的资源。  
（3）不可抢先（No preemption）：一个线程（ThreadA）占有的资源不能被强制拿走（只能等待 ThreadA 主动释放它锁定的资源）。  
（4）循环等待条件（Circular wait condition）：两个或多个线程构成一个循环等待链，它们锁定两个或多个相同的资源，每个线程都在等待链中下一个线程占有的资源。  
移除其中任何一个条件，都能阻止死锁的发生。

---

## C&#35; 决定两个对象是否“相等”的 4 个函数
C# 提供了 4 种不同的函数，用来决定两个对象是否“相等”：
```cs
public static bool ReferenceEquals(object left, object right);
public static bool Equals(object left, object right);
public virtual bool Equals(object right);
public static bool operator ==(MyClass left, MyClass right);
```

---

## 重写 System.Object.GetHashCode() 方法必须遵守的 3 条规则
&emsp;&emsp;.NET 中的每个对象都有哈希码，其哈希码由 System.Object.GetHashCode() 决定。如果要重写该方法，那么必须遵守下面 3 条规则：
1. 如果（实例版本的 Equals() 方法认定的）两个对象相等，那么这两个对象的哈希码也必须相同，否则，容器无法通过正确的哈希码来寻找相应的元素。
2. 对于任何一个对象 A 来说，GetHashCode() 必须在实例层面上满足这样一种不变条件（或者说，在实例层面上具备这样一种固定的性质）—— 在 A 的生命期内，无论实例 A 在执行完 GetHashCode() 方法之后还执行过其他哪些方法，当它再度执行 GetHashCode() 时，必定返回与当初相同的值。这条性质用来确保容器总是能把 A 放在正确的桶中。
3. 对于常见的输入值来说，哈希函数应该把这些值均匀地映射到各个整数上，而不应该使自己所输出的哈希码仅仅集中在几个整数上。如果每一个整数都能有相似的概率来充当对象的哈希码，那么基于哈希的容器就能够较为高效地运作。简单来说，就是要保证自己所实现的 GetHashCode() 能够让元素均匀地保存在容器的每一个桶中。此外，每个桶的元素个数也不宜太多。

---

## 重写 object.Equals() 的步骤
（1）检查是否为 null。
（2）如果是引用类型，就检查引用是否相等。
（3）检查数据类型是否相同。
（4）调用一个指定了具体类型的辅助方法，它的操作数是具体要比较的类型而不是 object。
（5）可能要检查哈希码是否相等来短路一次全面的、逐字段的比较。（相等的两个对象不可能哈希码不同。）
（6）如基类重写了 Equals()，就检查 base.Equals()。
（7）比较每一个标识字段（关键字段），判断是否相等。
（8）重写 GetHashCode()。
（9）重写 == 和 != 操作符。

---

## Math.Round 默认四舍六入五成双
> https://www.cnblogs.com/lwqlun/p/12070839.html

`Math.Round` 默认使用的并非是四舍五入的原则，而是**四舍六入五成双**的原则。  
所谓的四舍六入五成双，就是说当确定有效位数之后，**有效位数的下一位**如果**小于等于 4 就舍去**，如果**大于等于 6 就进一**，当有效位数的下一位是 5 的时候
* 如果 **5 前为奇数，就舍五进一**
* 如果 **5 前为偶数，就舍五不进**（0是偶数）

C# 中的 `Math.Round` 提供了非常多的重载方法，其中有两个重载方法是：
```cs
public static double Round (double value, int digits, MidpointRounding mode);
public static decimal Round (decimal d, int decimals, MidpointRounding mode);
```

这两个方法都提供了第三个参数 mode，mode 是一个 `MidpointRounding` 的枚举变量，它有 2 个可选值
* `AwayFromZero` - 四舍五入
* `ToEven` - 四舍六入五成双

所以如果我们希望的到一个理想中四舍五入的结果，我们可以改用如下代码：
```cs
var num = Math.Round(12.125, 2, MidpointRounding.AwayFromZero);
```

---

## 关于 null 的几个运算符
> https://mp.weixin.qq.com/s/A5xgijT0vhOvj-wCHeeuBA

### ?. 和 ?[]
Null 条件运算符在 C# 6 以后可用，仅当操作数为非 null 时才会访问成员或者访问元素。`?.`和`?[]`很好区分；我们知道`.`是访问成员或者命空间啥的，`[]`索引器访问，以下演示运算符的用法：
```cs
static double? SumNumbers(List<double[]> setsOfNumbers, int indexOfSetToSum)
{
    //如果 setsOfNumbers 非空，访问指定的索引；如果对应元素的索引不为空，求和
    return setsOfNumbers?[indexOfSetToSum]?.Sum() ;
}
        
var sum1 = SumNumbers(null, 0);
Console.WriteLine(sum1??Double.NaN);  // 输出: NaN

var numberSets = new List<double[]> { new[] { 1.0, 2.0, 3.0 }, null };

var sum2 = SumNumbers(numberSets, 0);
Console.WriteLine(sum2 ?? Double.NaN);  // 输出: 6

var sum3 = SumNumbers(numberSets, 1);
Console.WriteLine(sum3 ?? Double.NaN);  // 输出: NaN
```

### ??
Null 合并运算符，什么意思？就是如果这个值为空，就使用另外一个值，`a ?? b`,如果 a 为非 null，则结果为 a；否则结果为 b。仅当 a 为 null 时，操作才计算 b。常用场景比如：使用 `throw` 表达式作为`??`运算符的右操作数，检测数据、当获取为空时赋值默认值等等。
```cs
var comment = _blogService.GetBlogCommentById(id)
                ?? throw new ArgumentException("指定的id为查到对应数据！", nameof(id));
```

### ??=
运算符`??=`是在 C# 8.0 引入的 Null 合并赋值运算符。什么意思？就是当左操作数计算为 null 时，才能使用运算符`??=`将其有操作符的值 赋值给左操作数。实例代码如下：
```cs
List<int> numbers = null;
int? i = null;

numbers ??= new List<int>();
numbers.Add(i ??= 66);
numbers.Add(i ??= 99);

//等价于以下代码
//if (i==null)
//{
//    i = 66;
//    numbers.Add(i.Value);
//}

//if (i == null)
//{
//    i = 99;
//}
//numbers.Add(i.Value);

Console.WriteLine(string.Join(" ", numbers));  // 输出: 66 66
Console.WriteLine(i);  // output: 66
```

---

## 运算符提升（Operator Lifting）
> `four`，`five`，`nullInt`的类型都是`Nullable<int>`。

Expression | Lifted operator | Result
--- | --- | ---
-nullInt | int? -(int? x) | null
-five | int? -(int? x) | -5
five + nullInt | int? +(int? x, int? y) | null
five + five | int? +(int? x, int? y) | 10
four & nullInt | int? &(int? x, int? y) | null
four & five | int? &(int? x, int? y) | 4
nullInt == nullInt | bool ==(int? x, int? y) | true
five == five | bool ==(int? x, int? y) | true
five == nullInt | bool ==(int? x, int? y) | false
five == four | bool ==(int? x, int? y) | false
four < five | bool <(int? x, int? y) | true
nullInt < five | bool <(int? x, int? y) | false
five < nullInt | bool <(int? x, int? y) | false
nullInt < nullInt | bool <(int? x, int? y) | false
nullInt <= nullInt | bool <=(int? x, int? y) | false

## 可空逻辑运算
> 针对`Nullable<bool>`，没有条件逻辑运算符。例如`&&`，`||`。

x | y | x & y | x \| y | x ^ y | !x
--- | --- | --- | --- | --- | --- 
true | true | true | true | false | false
true | false | false | true | true | false
true | null | null | **true** | null | false
false | true | false | true | true | true
false | false | false | false | false | true
false | null | **false** | null | null | true
null | true | null | **true** | null | null
null | false | **false** | null | null | null
null | null | null | null | null | null

---

## await Task.Yield() 和 await Task.CompletedTask 有什么不同
> https://www.cnblogs.com/OpenCoder/p/12201446.html

* `Task.CompletedTask`本质上来说是返回一个已经完成的 Task 对象，所以这时如果我们用`await`关键字去等待`Task.CompletedTask`，.NET Core 认为没有必要再去线程池启动一个新的线程来执行`await`关键字之后的代码，所以实际上`await Task.CompletedTask`之前和之后的代码是在同一个线程上同步执行的，通俗易懂的说就是单线程的。这也是为什么很多文章说，使用了 await async 关键字并不代表程序就变成异步多线程的了。
* 而`Task.Yield()`就不一样了，我们可以理解`Task.Yield()`是真正使用 Task 来启动了一个线程，只不过这个线程什么都没有干，相当于在使用`await Task.Yield()`的时候，确实是在用`await`等待一个还没有完成的`Task`对象，所以这时调用线程（主线程）就会立即返回去做其它事情了，当调用线程（主线程）返回后，`await`等待的`Task`对象就立即变为完成了，这时`await`关键字之后的代码由另外一个线程池线程来执行。

---

# JavaScript
## 9 个小技巧
> http://blog.yidengxuetang.com/post/201912/11/

### 全部替换
我们知道 `string.replace()` 函数仅替换第一次出现的情况。  
你可以通过在正则表达式的末尾添加 `/g` 来替换所有出现的内容。
```js
var example = "potato potato";
console.log(example.replace(/pot/, "tom")); 
// "tomato potato"
console.log(example.replace(/pot/g, "tom")); 
// "tomato tomato"
```

### 提取唯一值
通过使用 Set 对象和展开运算符，我们可以创建一个具有唯一值的新数组。
```js
var entries = [1, 2, 2, 3, 4, 5, 6, 6, 7, 7, 8, 4, 2, 1]
var unique_entries = [...new Set(entries)];
console.log(unique_entries);
// [1, 2, 3, 4, 5, 6, 7, 8]
```

### 将数字转换为字符串
我们只需要使用带空引号的串联运算符。
```js
var converted_number = 5 + "";
console.log(converted_number);
// 5
console.log(typeof converted_number); 
// string
```

### 将字符串转换为数字
我们需要的只是 `+` 运算符。
请注意它仅适用于“字符串数字”。
```js
the_string = "123";
console.log(+the_string);
// 123

the_string = "hello";
console.log(+the_string);
// NaN
```

### 随机排列数组中的元素
```js
var my_list = [1, 2, 3, 4, 5, 6, 7, 8, 9];
console.log(my_list.sort(function() {
    return Math.random() - 0.5
})); 
// [4, 8, 2, 9, 1, 3, 6, 5, 7]
```

### 展平多维数组
只需使用展开运算符。
```js
var entries = [1, [2, 5], [6, 7], 9];
var flat_entries = [].concat(...entries); 
// [1, 2, 5, 6, 7, 9]
```

### 缩短条件语句
让我们来看这个例子：
```js
if (available) {
    addToCart();
}
```

通过简单地使用变量和函数来缩短它：
```js
available && addToCart()
```

### 动态属性名
我一直以为必须先声明一个对象，然后才能分配动态属性。
```js
const dynamic = 'flavour';
var item = {
    name: 'Coke',
    [dynamic]: 'Cherry'
}
console.log(item); 
// { name: "Coke", flavour: "Cherry" }
```

### 使用 length 调整/清空数组
我们基本上覆盖了数组的 length 。  
如果我们要调整数组的大小：
```js
var entries = [1, 2, 3, 4, 5, 6, 7];  
console.log(entries.length); 
// 7  
entries.length = 4;  
console.log(entries.length); 
// 4  
console.log(entries); 
// [1, 2, 3, 4]
```

如果我们要清空数组：
```js
var entries = [1, 2, 3, 4, 5, 6, 7]; 
console.log(entries.length); 
// 7  
entries.length = 0;   
console.log(entries.length); 
// 0 
console.log(entries); 
// []
```

---

# 其它

在 Git Bash 中用 ImageMagick 批量将图片从 png 格式转换为 gif 格式：
```
magick mogrify -format gif *.png
```