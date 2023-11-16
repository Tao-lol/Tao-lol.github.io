---
title: ASP.Net Core 3.1 中使用 JWT 认证
tags:
  - .NET
categories:
  - 编程
date: 2020-01-20 16:38:44
---

> https://www.cnblogs.com/liuww/p/12177272.html

<!--more-->

# JWT 认证简单介绍
&emsp;&emsp;关于 JWT 的介绍网上很多，此处不再赘述，我们主要看看 JWT 的结构。  
&emsp;&emsp;JWT 主要由三部分组成，如下：
```
HEADER.PAYLOAD.SIGNATURE
```

---

&emsp;&emsp;`HEADER`：包含 token 的元数据，主要是加密算法，和签名的类型。  
&emsp;&emsp;如下面的信息，说明了加密的对象类型是 JWT，加密算法是 HMAC SHA-256。
```json
{"alg":"HS256","typ":"JWT"}
```

&emsp;&emsp;然后需要通过 **BASE64** 编码后存入 token 中：
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
```

---

&emsp;&emsp;`Payload`：主要包含一些声明信息（claim），这些声明是 **key-value 对** 的数据结构。  
&emsp;&emsp;通常如用户名，角色等信息，过期日期等，因为是未加密的，所以不建议存放敏感信息。
```json
{"http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name":"admin","exp":1578645536,"iss":"webapi.cn","aud":"WebApi"}
```

&emsp;&emsp;也需要通过 **BASE64** 编码后存入 token 中：
```
eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1lIjoiYWRtaW4iLCJleHAiOjE1Nzg2NDU1MzYsImlzcyI6IndlYmFwaS5jbiIsImF1ZCI6IldlYkFwaSJ9
```

---

&emsp;&emsp;`Signature`：JWT 要符合 JWS（Json Web Signature）的标准生成一个最终的签名。把编码后的 Header 和 Payload 信息加在一起，然后使用一个强加密算法，如 Hmac SHA-256，进行加密。`HS256(BASE64(Header).Base64(Payload)，secret)`
```
2_akEH40LR2QWekgjm8Tt3lesSbKtDethmJMo_3jpF4
```

---

&emsp;&emsp;最后生成的token如下：
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1lIjoiYWRtaW4iLCJleHAiOjE1Nzg2NDU1MzYsImlzcyI6IndlYmFwaS5jbiIsImF1ZCI6IldlYkFwaSJ9.
2_akEH40LR2QWekgjm8Tt3lesSbKtDethmJMo_3jpF4
```

# ASP.NET Core 3.1 Web API 中使用 JWT 认证
> 开发环境：  
> * 框架：ASP.NET Core 3.1  
> * IDE：Visual Studio 2019

&emsp;&emsp;命令行中执行执行以下命令，创建 Web API 项目：
```bash
dotnet new webapi -n Webapi -o WebApi
```

&emsp;&emsp;特别注意的是，3.x 默认是没有 JWT 的`Microsoft.AspNetCore.Authentication.JwtBearer`库的，所以需要手动添加 NuGet Package，切换到项目所在目录，执行 .NET CLI 命令：
```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer --version 3.1.0
```

&emsp;&emsp;创建一个简单的 POCO 类，用来存储签发或者验证 JWT 时用到的信息：
```cs
using Newtonsoft.Json;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace Webapi.Models
{
    public class TokenManagement
    {
        [JsonProperty("secret")]
        public string Secret { get; set; }

        [JsonProperty("issuer")]
        public string Issuer { get; set; }

        [JsonProperty("audience")]
        public string Audience { get; set; }

        [JsonProperty("accessExpiration")]
        public int AccessExpiration { get; set; }

        [JsonProperty("refreshExpiration")]
        public int RefreshExpiration { get; set; }
    }
}
```

&emsp;&emsp;然后在`appsettings.Development.json`添加 JWT 使用到的配置信息（如果是生成环境在`appsettings.json`添加即可）：
```json
"tokenManagement": {
    "secret": "123456",
    "issuer": "webapi.cn",
    "audience": "WebApi",
    "accessExpiration": 30,
    "refreshExpiration": 60
}
```

&emsp;&emsp;然后在`StartUp`类的`ConfigureServices`方法中添加读取配置信息：
```cs
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.Configure<TokenManagement>(Configuration.GetSection("tokenManagement"));
    var token = Configuration.GetSection("tokenManagement").Get<TokenManagement>();
}
```

&emsp;&emsp;到目前为止，我们完成了一些基础工作，下面在 Web API 中注入 JWT 的验证服务，并在中间件管道中启用 Authentication 中间件。  
&emsp;&emsp;`StartUp`类中要引用 JWT 验证服务的命名空间：
```cs
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
```

&emsp;&emsp;然后在`ConfigureServices`方法中添加如下逻辑：
```cs
services.AddAuthentication(x =>
{
    x.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme;
    x.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme;
}).AddJwtBearer(x =>
{
    x.RequireHttpsMetadata = false;
    x.SaveToken = true;
    x.TokenValidationParameters = new TokenValidationParameters
    {
        ValidateIssuerSigningKey = true,
        IssuerSigningKey = new SymmetricSecurityKey(Encoding.ASCII.GetBytes(token.Secret)),
        ValidIssuer = token.Issuer,
        ValidAudience = token.Audience,
        ValidateIssuer = false,
        ValidateAudience = false
    };
});
```

&emsp;&emsp;在`Configure`方法中启用验证：
```cs
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseHttpsRedirection();

    app.UseAuthentication();
    app.UseRouting();

    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
```

---

&emsp;&emsp;上面完成了 JWT 验证的功能，下面就需要添加签发 token 的逻辑。我们需要添加一个专门用来用户认证和签发 token 的控制器，命名成 AuthenticationController，同时添加一个请求的 DTO 类：
```cs
public class LoginRequestDTO
{
    [Required]
    [JsonProperty("username")]
    public string Username { get; set; }

    [Required]
    [JsonProperty("password")]
    public string Password { get; set; }
}
```
```cs
[Route("api/[controller]")]
[ApiController]
public class AuthenticationController : ControllerBase
{
    [AllowAnonymous]
    [HttpPost, Route("requestToken")]
    public ActionResult RequestToken([FromBody] LoginRequestDTO request)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest("Invalid Request");
        }

        return Ok();
    }
}
```

&emsp;&emsp;目前上面的控制器只实现了基本的逻辑，下面我们要创建签发 token 的服务，去完成具体的业务。第一步我们先创建对应的服务接口，命名为 IAuthenticateService：
```cs
public interface IAuthenticateService
{
    bool IsAuthenticated(LoginRequestDTO request, out string token);
}
```

&emsp;&emsp;接下来，实现接口：
```cs
public class TokenAuthenticationService : IAuthenticateService
{
    public bool IsAuthenticated(LoginRequestDTO request, out string token)
    {
        throw new NotImplementedException();
    }
}
```

&emsp;&emsp;在`Startup`的`ConfigureServices`方法中注册服务：
```cs
services.AddScoped<IAuthenticateService, TokenAuthenticationService>();
```

&emsp;&emsp;在 Controller 中注入 IAuthenticateService 服务，并完善 Action：
```cs
public class AuthenticationController : ControllerBase
{
    private readonly IAuthenticateService _authService;
    public AuthenticationController(IAuthenticateService authService)
    {
        this._authService = authService;
    }
    [AllowAnonymous]
    [HttpPost, Route("requestToken")]
    public ActionResult RequestToken([FromBody] LoginRequestDTO request)
    {
        if (!ModelState.IsValid)
        {
            return BadRequest("Invalid Request");
        }

        string token;
        if (_authService.IsAuthenticated(request, out token))
        {
            return Ok(token);
        }

        return BadRequest("Invalid Request");
    }
}
```

&emsp;&emsp;正常情况，我们都会根据请求的用户和密码去验证用户是否合法，需要连接到数据库获取数据进行校验，我们这里为了方便，假设任何请求的用户都是合法的。  
&emsp;&emsp;这里单独加个用户管理的服务，不在 IAuthenticateService 这个服务里面添加相应逻辑，主要遵循了职责单一原则。首先和上面一样，创建一个服务接口 IUserService：
```cs
public interface IUserService
{
    bool IsValid(LoginRequestDTO req);
}
```

&emsp;&emsp;实现 IUserService 接口：
```cs
public class UserService : IUserService
{
    //模拟测试，默认都是人为验证有效
    public bool IsValid(LoginRequestDTO req)
    {
        return true;
    }
}
```

&emsp;&emsp;同样注册到容器中：
```cs
services.AddScoped<IUserService, UserService>();
```

&emsp;&emsp;接下来，就要完善 TokenAuthenticationService 签发 token 的逻辑，首先要注入 IUserService 和 TokenManagement，然后实现具体的业务逻辑，这个 token 的生成还是使用的 Jwt 的类库提供的 API，具体不详细描述。  
&emsp;&emsp;特别注意下 TokenManagement 的注入是以 IOptions 的接口类型注入的，还记得是在`StartpUp`中吗？我们是通过配置项的方式注册 TokenManagement 类型的。
```cs
public class TokenAuthenticationService : IAuthenticateService
{
    private readonly IUserService _userService;
    private readonly TokenManagement _tokenManagement;
    public TokenAuthenticationService(IUserService userService, IOptions<TokenManagement> tokenManagement)
    {
        _userService = userService;
        _tokenManagement = tokenManagement.Value;
    }
    public bool IsAuthenticated(LoginRequestDTO request, out string token)
    {
        token = string.Empty;
        if (!_userService.IsValid(request))
            return false;
        var claims = new[]
        {
            new Claim(ClaimTypes.Name,request.Username)
        };
        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_tokenManagement.Secret));
        var credentials = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
        var jwtToken = new JwtSecurityToken(_tokenManagement.Issuer, _tokenManagement.Audience, claims, expires: DateTime.Now.AddMinutes(_tokenManagement.AccessExpiration), signingCredentials: credentials);

        token = new JwtSecurityTokenHandler().WriteToken(jwtToken);

        return true;
    }
}
```

&emsp;&emsp;准备好测试试用的 API，打上`Authorize`特性，表明需要授权。
```cs
[ApiController]
[Route("[controller]")]
[Authorize]
public class WeatherForecastController : ControllerBase
{
    private static readonly string[] Summaries = new[]
    {
        "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
    };

    private readonly ILogger<WeatherForecastController> _logger;

    public WeatherForecastController(ILogger<WeatherForecastController> logger)
    {
        _logger = logger;
    }

    [HttpGet]
    public IEnumerable<WeatherForecast> Get()
    {
        var rng = new Random();
        return Enumerable.Range(1, 5).Select(index => new WeatherForecast
        {
            Date = DateTime.Now.AddDays(index),
            TemperatureC = rng.Next(-20, 55),
            Summary = Summaries[rng.Next(Summaries.Length)]
        })
        .ToArray();
    }
}
```

&emsp;&emsp;现在我们可以测试验证了，我们可以使用 Postman 来进行 HTTP 请求，先启动 HTTP 服务，获取 URL，先测试一个访问需要授权的接口，但没有携带 token 信息，返回是 401，表示未授权。
![ ](683694-20200110184421844-68601763.png)

&emsp;&emsp;下面我们先通过认证接口，获取 token，居然报错，查询了下，发现 HS256 算法的秘钥长度最新为 128 位，转换成字符至少 16 字符，之前设置的秘钥是 123456，所以导致异常。
```
System.ArgumentOutOfRangeException: IDX10603: Decryption failed. Keys tried: 'HS256'. Exceptions caught: '128'. token: '48' (Parameter 'KeySize') at
```

&emsp;&emsp;更新秘钥：
```json
"tokenManagement": {
    "secret": "123456123456123456",
    "issuer": "webapi.cn",
    "audience": "WebApi",
    "accessExpiration": 30,
    "refreshExpiration": 60
}
```

&emsp;&emsp;重新发起请求，成功获取 token：
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1lIjoiYWRtaW4iLCJleHAiOjE1Nzg2NDUyMDMsImlzcyI6IndlYmFwaS5jbiIsImF1ZCI6IldlYkFwaSJ9.
AehD8WTAnEtklof2OJsvg0U4_o8_SjdxmwUjzAiuI-o
```
![ ](683694-20200110184411897-1612763683.png)

&emsp;&emsp;把 token 带到之前请求的 API 中，重新测试，成功获取数据：
![ ](683694-20200110184403969-331881185.png)

# 总结
&emsp;&emsp;基于 token 的认证方式，让我们构建分布式/松耦合的系统更加容易。任何地方生成的 token，只有拥有相同秘钥，就可以在任何地方进行签名校验。  
&emsp;&emsp;当然要用好 JWT 认证方式，还有其他安全细节需要处理，比如 Payload 中不能存放敏感信息，使用 HTTPS 的加密传输方式等等，可以根据业务实际需要再进一步安全加固。  
&emsp;&emsp;同时我们也发现使用 token，就可以摆脱 Cookie 的限制，所以 JWT 是移动 APP 开发的首选。
