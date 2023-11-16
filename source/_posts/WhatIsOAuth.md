---
title: 理解 OAuth 2.0
tags:
  - 程序设计
categories:
  - 编程
date: 2019-12-30 13:25:42
---

> 旧：  
> http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html  

> 新：  
> http://www.ruanyifeng.com/blog/2019/04/oauth_design.html  
> http://www.ruanyifeng.com/blog/2019/04/oauth-grant-types.html  

<!--more-->

# 旧
&emsp;&emsp;[OAuth](http://en.wikipedia.org/wiki/OAuth) 是一个关于授权（authorization）的开放网络标准，在全世界得到广泛应用，目前的版本是 2.0 版。
&emsp;&emsp;本文对 OAuth 2.0 的设计思路和运行流程，做一个简明通俗的解释，主要参考材料为 [RFC 6749](http://www.rfcreader.com/#rfc6749)。
![ ](bg2014051201.png)

## 应用场景
&emsp;&emsp;为了理解 OAuth 的适用场合，让我举一个假设的例子。  
&emsp;&emsp;有一个"云冲印"的网站，可以将用户储存在 Google 的照片，冲印出来。用户为了使用该服务，必须让"云冲印"读取自己储存在 Google 上的照片。  
&emsp;&emsp;问题是只有得到用户的授权，Google 才会同意"云冲印"读取这些照片。那么，"云冲印"怎样获得用户的授权呢？  
&emsp;&emsp;传统方法是，用户将自己的 Google 用户名和密码，告诉"云冲印"，后者就可以读取用户的照片了。这样的做法有以下几个严重的缺点：

> （1）"云冲印"为了后续的服务，会保存用户的密码，这样很不安全。  
> （2）Google 不得不部署密码登录，而我们知道，单纯的密码登录并不安全。  
> （3）"云冲印"拥有了获取用户储存在 Google 所有资料的权力，用户没法限制"云冲印"获得授权的范围和有效期。  
> （4）用户只有修改密码，才能收回赋予"云冲印"的权力。但是这样做，会使得其他所有获得用户授权的第三方应用程序全部失效。  
> （5）只要有一个第三方应用程序被破解，就会导致用户密码泄漏，以及所有被密码保护的数据泄漏。

&emsp;&emsp;OAuth 就是为了解决上面这些问题而诞生的。

## 名词定义
&emsp;&emsp;在详细讲解 OAuth 2.0 之前，需要了解几个专用名词。它们对读懂后面的讲解，尤其是几张图，至关重要。

> （1）**Third-party application**：第三方应用程序，本文中又称"客户端"（client），即上一节例子中的"云冲印"。  
> （2）**HTTP service**：HTTP 服务提供商，本文中简称"服务提供商"，即上一节例子中的 Google。  
> （3）**Resource Owner**：资源所有者，本文中又称"用户"（user）。  
> （4）**User Agent**：用户代理，本文中就是指浏览器。  
> （5）**Authorization server**：认证服务器，即服务提供商专门用来处理认证的服务器。  
> （6）**Resource server**：资源服务器，即服务提供商存放用户生成的资源的服务器。它与认证服务器，可以是同一台服务器，也可以是不同的服务器。

&emsp;&emsp;知道了上面这些名词，就不难理解，OAuth 的作用就是让"客户端"安全可控地获取"用户"的授权，与"服务商提供商"进行互动。

## OAuth 的思路
&emsp;&emsp;OAuth 在"客户端"与"服务提供商"之间，设置了一个授权层（authorization layer）。"客户端"不能直接登录"服务提供商"，只能登录授权层，以此将用户与客户端区分开来。"客户端"登录授权层所用的令牌（token），与用户的密码不同。用户可以在登录的时候，指定授权层令牌的权限范围和有效期。  
&emsp;&emsp;"客户端"登录授权层以后，"服务提供商"根据令牌的权限范围和有效期，向"客户端"开放用户储存的资料。

## 运行流程
&emsp;&emsp;OAuth 2.0 的运行流程如下图，摘自 RFC 6749。
![ ](bg2014051203.png)

> （A）用户打开客户端以后，客户端要求用户给予授权。
> （B）用户同意给予客户端授权。
> （C）客户端使用上一步获得的授权，向认证服务器申请令牌。
> （D）认证服务器对客户端进行认证以后，确认无误，同意发放令牌。
> （E）客户端使用令牌，向资源服务器申请获取资源。
> （F）资源服务器确认令牌无误，同意向客户端开放资源。

&emsp;&emsp;不难看出来，上面六个步骤之中，B 是关键，即用户怎样才能给于客户端授权。有了这个授权以后，客户端就可以获取令牌，进而凭令牌获取资源。  

&emsp;&emsp;下面一一讲解客户端获取授权的四种模式。

## 客户端的授权模式
&emsp;&emsp;客户端必须得到用户的授权（authorization grant），才能获得令牌（access token）。OAuth 2.0 定义了四种授权方式：
* 授权码模式（authorization code）
* 简化模式（implicit）
* 密码模式（resource owner password credentials）
* 客户端模式（client credentials）

### 授权码模式
&emsp;&emsp;授权码模式（authorization code）是功能最完整、流程最严密的授权模式。它的特点就是通过客户端的后台服务器，与"服务提供商"的认证服务器进行互动。
![ ](bg2014051204.png)

它的步骤如下：
> （A）用户访问客户端，后者将前者导向认证服务器。
> （B）用户选择是否给予客户端授权。
> （C）假设用户给予授权，认证服务器将用户导向客户端事先指定的"重定向 URI"（redirection URI），同时附上一个授权码。
> （D）客户端收到授权码，附上早先的"重定向 URI"，向认证服务器申请令牌。这一步是在客户端的后台的服务器上完成的，对用户不可见。
> （E）认证服务器核对了授权码和重定向 URI，确认无误后，向客户端发送访问令牌（access token）和更新令牌（refresh token）。

下面是上面这些步骤所需要的参数。  

A 步骤中，客户端申请认证的 URI，包含以下参数：
* response_type：表示授权类型，必选项，此处的值固定为 "code"
* client_id：表示客户端的 ID，必选项
* redirect_uri：表示重定向 URI，可选项
* scope：表示申请的权限范围，可选项
* state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值。

下面是一个例子：
```
GET /authorize?response_type=code&client_id=s6BhdRkqt3&state=xyz
        &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
Host: server.example.com

```

C 步骤中，服务器回应客户端的 URI，包含以下参数：
* code：表示授权码，必选项。该码的有效期应该很短，通常设为 10 分钟，客户端只能使用该码一次，否则会被授权服务器拒绝。该码与客户端 ID 和重定向 URI，是一一对应关系。
* state：如果客户端的请求中包含这个参数，认证服务器的回应也必须一模一样包含这个参数。

下面是一个例子：
```
HTTP/1.1 302 Found
Location: https://client.example.com/cb?code=SplxlOBeZQQYbYS6WxSbIA
          &state=xyz
```

D 步骤中，客户端向认证服务器申请令牌的 HTTP 请求，包含以下参数：
* grant_type：表示使用的授权模式，必选项，此处的值固定为 "authorization_code"。
* code：表示上一步获得的授权码，必选项。
* redirect_uri：表示重定向 URI，必选项，且必须与 A 步骤中的该参数值保持一致。
* client_id：表示客户端 ID，必选项。

下面是一个例子：
```
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&code=SplxlOBeZQQYbYS6WxSbIA
&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb
```

E 步骤中，认证服务器发送的 HTTP 回复，包含以下参数：
* access_token：表示访问令牌，必选项。
* token_type：表示令牌类型，该值大小写不敏感，必选项，可以是 bearer 类型或 mac 类型。
* expires_in：表示过期时间，单位为秒。如果省略该参数，必须其他方式设置过期时间。
* refresh_token：表示更新令牌，用来获取下一次的访问令牌，可选项。
* scope：表示权限范围，如果与客户端申请的范围一致，此项可省略。

下面是一个例子：
```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
  "access_token":"2YotnFZFEjr1zCsicMWpAA",
  "token_type":"example",
  "expires_in":3600,
  "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
  "example_parameter":"example_value"
}
```

&emsp;&emsp;从上面代码可以看到，相关参数使用 JSON 格式发送（Content-Type: application/json）。此外，HTTP 头信息中明确指定不得缓存。

### 简化模式
&emsp;&emsp;简化模式（implicit grant type）不通过第三方应用程序的服务器，直接在浏览器中向认证服务器申请令牌，跳过了"授权码"这个步骤，因此得名。所有步骤在浏览器中完成，令牌对访问者是可见的，且客户端不需要认证。
![ ](bg2014051205.png)

它的步骤如下：
> （A）客户端将用户导向认证服务器。  
> （B）用户决定是否给于客户端授权。  
> （C）假设用户给予授权，认证服务器将用户导向客户端指定的"重定向 URI"，并在 URI 的 Hash 部分包含了访问令牌。  
> （D）浏览器向资源服务器发出请求，其中不包括上一步收到的 Hash 值。  
> （E）资源服务器返回一个网页，其中包含的代码可以获取 Hash 值中的令牌。  
> （F）浏览器执行上一步获得的脚本，提取出令牌。  
> （G）浏览器将令牌发给客户端。

下面是上面这些步骤所需要的参数。  

A 步骤中，客户端发出的 HTTP 请求，包含以下参数：
* response_type：表示授权类型，此处的值固定为 "token"，必选项。
* client_id：表示客户端的 ID，必选项。
* redirect_uri：表示重定向的 URI，可选项。
* scope：表示权限范围，可选项。
* state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值。

下面是一个例子：
```
GET /authorize?response_type=token&client_id=s6BhdRkqt3&state=xyz
    &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
Host: server.example.com
```

C 步骤中，认证服务器回应客户端的 URI，包含以下参数：
* access_token：表示访问令牌，必选项。
* token_type：表示令牌类型，该值大小写不敏感，必选项。
* expires_in：表示过期时间，单位为秒。如果省略该参数，必须其他方式设置过期时间。
* scope：表示权限范围，如果与客户端申请的范围一致，此项可省略。
* state：如果客户端的请求中包含这个参数，认证服务器的回应也必须一模一样包含这个参数。

下面是一个例子：
```
HTTP/1.1 302 Found
Location: http://example.com/cb#access_token=2YotnFZFEjr1zCsicMWpAA
          &state=xyz&token_type=example&expires_in=3600
```

&emsp;&emsp;在上面的例子中，认证服务器用 HTTP 头信息的 Location 栏，指定浏览器重定向的网址。注意，在这个网址的 Hash 部分包含了令牌。  
&emsp;&emsp;根据上面的 D 步骤，下一步浏览器会访问 Location 指定的网址，但是 Hash 部分不会发送。接下来的 E 步骤，服务提供商的资源服务器发送过来的代码，会提取出 Hash 中的令牌。

### 密码模式
&emsp;&emsp;密码模式（Resource Owner Password Credentials Grant）中，用户向客户端提供自己的用户名和密码。客户端使用这些信息，向"服务商提供商"索要授权。  
&emsp;&emsp;在这种模式中，用户必须把自己的密码给客户端，但是客户端不得储存密码。这通常用在用户对客户端高度信任的情况下，比如客户端是操作系统的一部分，或者由一个著名公司出品。而认证服务器只有在其他授权模式无法执行的情况下，才能考虑使用这种模式。
![ ](bg2014051206.png)

它的步骤如下：
> （A）用户向客户端提供用户名和密码。  
> （B）客户端将用户名和密码发给认证服务器，向后者请求令牌。  
> （C）认证服务器确认无误后，向客户端提供访问令牌。

B 步骤中，客户端发出的 HTTP 请求，包含以下参数：
* grant_type：表示授权类型，此处的值固定为 "password"，必选项。
* username：表示用户名，必选项。
* password：表示用户的密码，必选项。
* scope：表示权限范围，可选项。

下面是一个例子：
```
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded

grant_type=password&username=johndoe&password=A3ddj3w
```

C 步骤中，认证服务器向客户端发送访问令牌，下面是一个例子：
```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
  "access_token":"2YotnFZFEjr1zCsicMWpAA",
  "token_type":"example",
  "expires_in":3600,
  "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
  "example_parameter":"example_value"
}
```

上面代码中，各个参数的含义参见《授权码模式》一节。
整个过程中，客户端不得保存用户的密码。

### 客户端模式
&emsp;&emsp;客户端模式（Client Credentials Grant）指客户端以自己的名义，而不是以用户的名义，向"服务提供商"进行认证。严格地说，客户端模式并不属于 OAuth 框架所要解决的问题。在这种模式中，用户直接向客户端注册，客户端以自己的名义要求"服务提供商"提供服务，其实不存在授权问题。
![ ](bg2014051207.png)

它的步骤如下：
> （A）客户端向认证服务器进行身份认证，并要求一个访问令牌。  
> （B）认证服务器确认无误后，向客户端提供访问令牌。

A 步骤中，客户端发出的 HTTP 请求，包含以下参数：
* granttype：表示授权类型，此处的值固定为 "clientcredentials"，必选项。
* scope：表示权限范围，可选项。

```
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
```

认证服务器必须以某种方式，验证客户端身份。  

B 步骤中，认证服务器向客户端发送访问令牌，下面是一个例子。
```
HTTP/1.1 200 OK
Content-Type: application/json;charset=UTF-8
Cache-Control: no-store
Pragma: no-cache

{
  "access_token":"2YotnFZFEjr1zCsicMWpAA",
  "token_type":"example",
  "expires_in":3600,
  "example_parameter":"example_value"
}
```

上面代码中，各个参数的含义参见《授权码模式》一节。

## 更新令牌
&emsp;&emsp;如果用户访问的时候，客户端的"访问令牌"已经过期，则需要使用"更新令牌"申请一个新的访问令牌。  

客户端发出更新令牌的 HTTP 请求，包含以下参数：
* granttype：表示使用的授权模式，此处的值固定为 "refreshtoken"，必选项。
* refresh_token：表示早前收到的更新令牌，必选项。
* scope：表示申请的授权范围，不可以超出上一次申请的范围，如果省略该参数，则表示与上一次一致。

下面是一个例子：
```
POST /token HTTP/1.1
Host: server.example.com
Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&refresh_token=tGzv3JOkF0XG5Qx2TlKWIA
```

---

# 新
&emsp;&emsp;[OAuth 2.0](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html) 是目前最流行的授权机制，用来授权第三方应用，获取用户数据。  

&emsp;&emsp;**简单说，OAuth 就是一种授权机制。数据的所有者告诉系统，同意授权第三方应用进入系统，获取这些数据。系统从而产生一个短期的进入令牌（token），用来代替密码，供第三方应用使用。**

## 令牌与密码
&emsp;&emsp;令牌（token）与密码（password）的作用是一样的，都可以进入系统，但是有三点差异：
1. 令牌是短期的，到期会自动失效，用户自己无法修改。密码一般长期有效，用户不修改，就不会发生变化。
2. 令牌可以被数据所有者撤销，会立即失效。密码一般不允许被他人撤销。
3. 令牌有权限范围（scope）。对于网络服务来说，只读令牌就比读写令牌更安全。密码一般是完整权限。

&emsp;&emsp;上面这些设计，保证了令牌既可以让第三方应用获得权限，同时又随时可控，不会危及系统安全。这就是 OAuth 2.0 的优点。  

&emsp;&emsp;注意，只要知道了令牌，就能进入系统。系统一般不会再次确认身份，所以**令牌必须保密，泄漏令牌与泄漏密码的后果是一样的。**这也是为什么令牌的有效期，一般都设置得很短的原因。

## RFC 6749
&emsp;&emsp;OAuth 2.0 的标准是 [RFC 6749](https://tools.ietf.org/html/rfc6749) 文件。该文件先解释了 OAuth 是什么。
> &emsp;&emsp;OAuth 引入了一个授权层，用来分离两种不同的角色：客户端和资源所有者。......资源所有者同意以后，资源服务器可以向客户端颁发令牌。客户端通过令牌，去请求数据。

&emsp;&emsp;这段话的意思就是，**OAuth 的核心就是向第三方应用颁发令牌**。然后，RFC 6749 接着写道：
> &emsp;&emsp;（由于互联网有多种场景，）本标准定义了获得令牌的四种授权方式（authorization grant ）。

&emsp;&emsp;也就是说，**OAuth 2.0 规定了四种获得令牌的流程。你可以选择最适合自己的那一种，向第三方应用颁发令牌。**下面就是这四种授权方式：
> * 授权码（authorization-code）
> * 隐藏式（implicit）
> * 密码式（password）
> * 客户端凭证（client credentials）

&emsp;&emsp;注意，不管哪一种授权方式，第三方应用申请令牌之前，都必须先到系统备案，说明自己的身份，然后会拿到两个身份识别码：客户端 ID（client ID）和客户端密钥（client secret）。这是为了防止令牌被滥用，没有备案过的第三方应用，是不会拿到令牌的。

## 第一种授权方式：授权码
&emsp;&emsp;**授权码（authorization code）方式，指的是第三方应用先申请一个授权码，然后再用该码获取令牌。**  
&emsp;&emsp;这种方式是最常用的流程，安全性也最高，它适用于那些有后端的 Web 应用。授权码通过前端传送，令牌则是储存在后端，而且所有与资源服务器的通信都在后端完成。这样的前后端分离，可以避免令牌泄漏。  

&emsp;&emsp;第一步，A 网站提供一个链接，用户点击后就会跳转到 B 网站，授权用户数据给 A 网站使用。下面就是 A 网站跳转 B 网站的一个示意链接。
```
https://b.com/oauth/authorize?
  response_type=code&
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=read
```

&emsp;&emsp;上面 URL 中，`response_type`参数表示要求返回授权码（`code`），`client_id`参数让 B 知道是谁在请求，`redirect_uri`参数是 B 接受或拒绝请求后的跳转网址，`scope`参数表示要求的授权范围（这里是只读）。
![ ](bg2019040902.jpg)

&emsp;&emsp;第二步，用户跳转后，B 网站会要求用户登录，然后询问是否同意给予 A 网站授权。用户表示同意，这时 B 网站就会跳回`redirect_uri`参数指定的网址。跳转时，会传回一个授权码，就像下面这样。
```
https://a.com/callback?code=AUTHORIZATION_CODE
```

&emsp;&emsp;上面 URL 中，`code`参数就是授权码。
![ ](bg2019040907.jpg)

&emsp;&emsp;第三步，A 网站拿到授权码以后，就可以在后端，向 B 网站请求令牌。
```
https://b.com/oauth/token?
 client_id=CLIENT_ID&
 client_secret=CLIENT_SECRET&
 grant_type=authorization_code&
 code=AUTHORIZATION_CODE&
 redirect_uri=CALLBACK_URL
```

&emsp;&emsp;上面 URL 中，`client_id`参数和`client_secret`参数用来让 B 确认 A 的身份（`client_secret`参数是保密的，因此只能在后端发请求），`grant_type`参数的值是`AUTHORIZATION_CODE`，表示采用的授权方式是授权码，`code`参数是上一步拿到的授权码，`redirect_uri`参数是令牌颁发后的回调网址。
![ ](bg2019040904.jpg)

&emsp;&emsp;第四步，B 网站收到请求以后，就会颁发令牌。具体做法是向`redirect_uri`指定的网址，发送一段 JSON 数据。
```
{
  "access_token":"ACCESS_TOKEN",
  "token_type":"bearer",
  "expires_in":2592000,
  "refresh_token":"REFRESH_TOKEN",
  "scope":"read",
  "uid":100101,
  "info":{...}
}
```

&emsp;&emsp;上面 JSON 数据中，`access_token`字段就是令牌，A 网站在后端拿到了。
![ ](bg2019040905.jpg)

## 第二种方式：隐藏式
&emsp;&emsp;有些 Web 应用是纯前端应用，没有后端。这时就不能用上面的方式了，必须将令牌储存在前端。**RFC 6749 就规定了第二种方式，允许直接向前端颁发令牌。这种方式没有授权码这个中间步骤，所以称为（授权码）"隐藏式"（implicit）。**  

&emsp;&emsp;第一步，A 网站提供一个链接，要求用户跳转到 B 网站，授权用户数据给 A 网站使用。
```
https://b.com/oauth/authorize?
  response_type=token&
  client_id=CLIENT_ID&
  redirect_uri=CALLBACK_URL&
  scope=read
```

&emsp;&emsp;上面 URL 中，`response_type`参数为`token`，表示要求直接返回令牌。  

&emsp;&emsp;第二步，用户跳转到 B 网站，登录后同意给予 A 网站授权。这时，B 网站就会跳回`redirect_uri`参数指定的跳转网址，并且把令牌作为 URL 参数，传给 A 网站。
```
https://a.com/callback#token=ACCESS_TOKEN
```

&emsp;&emsp;上面 URL 中，`token`参数就是令牌，A 网站因此直接在前端拿到令牌。  

&emsp;&emsp;注意，令牌的位置是 URL 锚点（fragment），而不是查询字符串（querystring），这是因为 OAuth 2.0 允许跳转网址是 HTTP 协议，因此存在"中间人攻击"的风险，而浏览器跳转时，锚点不会发到服务器，就减少了泄漏令牌的风险。
![ ](bg2019040906.jpg)

&emsp;&emsp;这种方式把令牌直接传给前端，是很不安全的。因此，只能用于一些安全要求不高的场景，并且令牌的有效期必须非常短，通常就是会话期间（session）有效，浏览器关掉，令牌就失效了。

## 第三种方式：密码式
&emsp;&emsp;**如果你高度信任某个应用，RFC 6749 也允许用户把用户名和密码，直接告诉该应用。该应用就使用你的密码，申请令牌，这种方式称为"密码式"（password）。**  

&emsp;&emsp;第一步，A 网站要求用户提供 B 网站的用户名和密码。拿到以后，A 就直接向 B 请求令牌。
```
https://oauth.b.com/token?
  grant_type=password&
  username=USERNAME&
  password=PASSWORD&
  client_id=CLIENT_ID
```

&emsp;&emsp;上面 URL 中，`grant_type`参数是授权方式，这里的`password`表示"密码式"，`username`和`password`是 B 的用户名和密码。  

&emsp;&emsp;第二步，B 网站验证身份通过后，直接给出令牌。注意，这时不需要跳转，而是把令牌放在 JSON 数据里面，作为 HTTP 回应，A 因此拿到令牌。  

&emsp;&emsp;这种方式需要用户给出自己的用户名/密码，显然风险很大，因此只适用于其他授权方式都无法采用的情况，而且必须是用户高度信任的应用。

## 第四种方式：凭证式
&emsp;&emsp;**最后一种方式是凭证式（client credentials），适用于没有前端的命令行应用，即在命令行下请求令牌。**  

&emsp;&emsp;第一步，A 应用在命令行向 B 发出请求。
```
https://oauth.b.com/token?
  grant_type=client_credentials&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET
```

&emsp;&emsp;上面 URL 中，`grant_type`参数等于`client_credentials`表示采用凭证式，`client_id`和`client_secret`用来让 B 确认 A 的身份。  

&emsp;&emsp;第二步，B 网站验证通过以后，直接返回令牌。  

&emsp;&emsp;这种方式给出的令牌，是针对第三方应用的，而不是针对用户的，即有可能多个用户共享同一个令牌。

## 令牌的使用
&emsp;&emsp;A 网站拿到令牌以后，就可以向 B 网站的 API 请求数据了。  
&emsp;&emsp;此时，每个发到 API 的请求，都必须带有令牌。具体做法是在请求的头信息，加上一个`Authorization`字段，令牌就放在这个字段里面。
```
curl -H "Authorization: Bearer ACCESS_TOKEN" \
"https://api.b.com"
```

&emsp;&emsp;上面命令中，`ACCESS_TOKEN`就是拿到的令牌。

## 更新令牌
&emsp;&emsp;令牌的有效期到了，如果让用户重新走一遍上面的流程，再申请一个新的令牌，很可能体验不好，而且也没有必要。OAuth 2.0 允许用户自动更新令牌。  

&emsp;&emsp;具体方法是，B 网站颁发令牌的时候，一次性颁发两个令牌，一个用于获取数据，另一个用于获取新的令牌（refresh token 字段）。令牌到期前，用户使用 refresh token 发一个请求，去更新令牌。
```
https://b.com/oauth/token?
  grant_type=refresh_token&
  client_id=CLIENT_ID&
  client_secret=CLIENT_SECRET&
  refresh_token=REFRESH_TOKEN
```

&emsp;&emsp;上面 URL 中，`grant_type`参数为`refresh_token`表示要求更新令牌，`client_id`参数和`client_secret`参数用于确认身份，`refresh_token`参数就是用于更新令牌的令牌。  

&emsp;&emsp;B 网站验证通过以后，就会颁发新的令牌。