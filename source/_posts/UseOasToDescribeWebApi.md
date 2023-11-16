---
title: 使用 OAS（OpenAPI 标准）来描述 Web API
tags:
  - 程序设计
categories:
  - 编程
date: 2020-01-20 16:45:24
---

> https://www.cnblogs.com/cgzl/p/12217617.html

<!--more-->

&emsp;&emsp;无论哪种类型的 Web API 都可能需要给其他开发者使用，所以 API 的开发者体验是很重要的。API 的开发者体验，简写为 API DX（Developer Experience），它包含很多东西。例如如何使用 API、文档、技术支持等等，但是最重要的还是 API 的设计。如果 API 设计的不好，那么使用该 API 构建的软件就需要增加在时间、人力、金钱等方面的投入。有时候 API 会被错用，甚至带来毁灭性后果。最后抱怨该 API 的用户越来越多，慢慢地，客户就会停止使用该 API。   
&emsp;&emsp;API 的目的是让人们可以简单的使用它来达到自己的目的。目前行业内有很多 API 风格，例如：REST、gRPC、GraphQL、SOAP、RPC 等等。但是每个风格都遵循一些基本的设计原则。

# 用户就是上帝，为用户设计 API
&emsp;&emsp;和构建任何东西一样，你需要一个计划，你需要在真正做之前来决定你想要的是什么。API 设计也是一样的。  
&emsp;&emsp;API 并不是用来盲目地暴露一些数据或业务处理能力。它就像我们每天使用的任何形式的接口一样，例如微波炉的操作按钮，是来帮助用户完成他们的目标的。所以需要从用户的视角来决定一个 API 的设计目标。在整个设计过程中，必须牢记以用户的视角去设计，如果以开发者的角度去设计，那么问题就大了。  
&emsp;&emsp;如果以开发者的视角去设计 API，那么通常的后果是开发出的 API 会很注重功能实现的过程和原理，而不是用户如何能简单平滑地使用这个 API 来达到他们的目的。所以一定要注重用户的需求，而不要让内部实现细节、原理什么的来骚扰用户。最后再次强调，要设计出让用户容易理解和容易使用的 API。  
&emsp;&emsp;所以 API 就是用户看到的，它表示出用户能使用它做什么。API 的实现细节，也就是如果完成的该功能的细节，需要对用户隐藏。

# 识别 API 的目标
&emsp;&emsp;记住首先考虑用户的感受之后，下面就需要考虑用户能拿它来做什么了，也就是识别 API 的目标。

&emsp;&emsp;识别 API 的目标，最基本的要对以下方面有深刻、精准的认识：
1. Who，谁可以使用这个API？ 
2. What，用户拿这个API能做什么事？  
3. How，用户如何做这件事？ 
4. What need，用户想要做这件事的话还需要什么？ 
5. What return，用户会得到什么？ 
 
&emsp;&emsp;1 就是指 API 的用户，4、5 分别表示输入输出。  

## 针对 2、3 解释一下
&emsp;&emsp;通常针对 2. What（用户拿 API 能做什么）可以导致（分解）多个 3. How（多个步骤），这样的话每个步骤就是一个 API 的目标。   
&emsp;&emsp;比如说，用户想去淘宝买一个商品，那么怎么买？首先需要把商品添加到购物车，然后再结账。那么这个 API 就应该有两个目标：添加商品到购物车 以及 结账。   
&emsp;&emsp;如果不这样分解到话，通常设计出的 API 会缺失一些目标。 

## 针对1，也解释一下
&emsp;&emsp;首先应该识别出不同种类的用户，这里的用户可能是人，也可能是其他的程序。通常通过检查输入和输出就可以识别出用户。 
 
&emsp;&emsp;总结一下就 6 个方面： 
* 用户 
* 能做什么 
* 如何做 - 分解步骤 
* 输入 
* 输出 
* 目标 

# 避免从开发者角度设计 API
&emsp;&emsp;这部分包含几个方面，包括： 
* 开发者所在公司的组织结构（参考康威定律）；
* 数据，例如数据使用了开发者所在公司内部的一些专有术语，或者干脆把内部数据库模型暴露了出来；
* 不要暴露实现细节，避免受到业务逻辑实现细节的影响；
* 避免受到软件架构的影响，比如说在开发者公司内部查询产品名称和产品价格是两个 API，那么给用户使用的 API 必须整合一下，不能让用户分两步查询。 
 
&emsp;&emsp;最重要的还是要时刻牢记，你所设计的这些东西都是用户真正需要的吗？ 

&emsp;&emsp;下面切入正题： 

# 使用 API 描述格式来描述 API
&emsp;&emsp;这里我以 RESTful 风格的 API 为例。想要了解使用 ASP.NET Core 3.x 构建 RESTful API，这里有一个教程：https://www.bilibili.com/video/av77957694/ 。
 
&emsp;&emsp;很多人使用 Excel 或者纸和笔来进行 API 的设计工作。但是如果想要在设计阶段精准描述一个 API，尤其是它的数据，那么最好使用一个结构化的工具，例如 API 描述格式。  
&emsp;&emsp;API 描述格式会为 API 提供一个标准化的描述，并且它很像代码。它的优势主要有：
* 有助于在项目团队中共享设计 
* 了解这种格式的人或者工具可以很简单的理解它。 
 
&emsp;&emsp;针对 REST 而言，OpenAPI Specification（OAS）就是一个非常流行的 API 描述格式规范。 

## OAS
&emsp;&emsp;API 描述格式是一种数据格式，它的目标就是描述 API。   
&emsp;&emsp;而 OAS（OpenAPI Specification）是一个与编程语言无关的 REST API 描述格式。它是由 OAI（OpenAPI Initiative）所提倡的。OAI 是 Linux 基金会下面的一个组织，专注于提供与供应商无关的描述格式。而 OAS 则是社区驱动的一种格式，任何人都可以做贡献。 

## OAS vs Swagger
&emsp;&emsp;OAS 原来叫 Swagger Specification，2015 年 11 月这个格式被贡献给了 OAI，并在 2016 年 1 月更名为 OpenAPI Specification。Swagger 规范最后的 2.0 版本就变成了 OpenAPI 2.0。目前最新的 OAS 应该是 3.0 大版本 

## YAML
&emsp;&emsp;OAS 文档可以使用 YAML 或 JSON 格式，我使用 YAML。 

## 像写代码一样描述 API
&emsp;&emsp;OAS 文档就是一个文本文件，可以纳入版本控制系统，例如 Git 等。所以在设计迭代的时候很容易进行版本管理和变化追踪。 

## 编辑器
&emsp;&emsp;OAS 有一个在线的专用编辑器：http://editor.swagger.io/ 。
![ ](1.png)

&emsp;&emsp;左边是代码编辑区域，右边是渲染结果。 

&emsp;&emsp;但是我更习惯于本地编辑器，我使用 VSCode，并安装 Swagger Viewer 和 openapi-lint 两个插件。 
![ ](2.png)

## 共享 API 描述， 对 API 进行文档记录
&emsp;&emsp;OAS 文档可以用来生成 API 对引用文档，这个引用文档可以展示出所有可用的资源以及相应的操作。通常我会使用 Swagger UI，它就是上图右侧的部分。 

## 生成代码
&emsp;&emsp;使用 API 描述格式进行描述的 API，其代码也可以部分生成。通常是一个代码骨架。 

## 什么时候使用 API 描述格式
&emsp;&emsp;肯定是在设计接口如何表达 API 目标和概念，以及数据的时候。 

# 使用 OAS 来描述 REST API 的资源以及 Action
## 创建 OAS 文档
&emsp;&emsp;建立一个 products.yaml 文件。  
&emsp;&emsp;然后在里面输入 api 或 open 等字符串，会出现两个提示选项： 
![ ](3.png)

&emsp;&emsp;先选择下面那个选项，其结果是： 
![ ](4.png)

* 第 1 行是 Open API 的版本 
* 第 4 行`info`的`version`是指 API 的版本，而`info`这个版本必须使用双引号`""`括起来，否则 OAS 解析器会把它当成数字，从而导致文档验证失败（因为它的类型应该是字符串）。 
* 第 5 行`paths`，`paths`属性应该包含该 API 可用的资源。这里面使用`{}`仅仅是为了让文档验证通过，因为我目前还没有写什么内容。在 YAML 里，`{}`表示一个空的对象，而非空的对象则不需要这对大括号。 

## 描述资源
&emsp;&emsp;为了描述 products 这个资源，就需要填写`paths`属性： 
![ ](5.png)

&emsp;&emsp;这里`description`属性不是强制的，但是它可以用来描述该资源。 

## 描述资源的操作
&emsp;&emsp;OAS 文档里描述的资源肯定包含一些操作，否则文档就不合理。   
&emsp;&emsp;看代码： 
![ ](6.png)

&emsp;&emsp;我为`/products`这个资源添加了一个 GET Action（`get`属性），然后我对这个`get`也进行了描述。   
&emsp;&emsp;`summary`相当于是对这个 Action 的一个概括性描述，而`description`则能提供更详细的描述信息。    
&emsp;&emsp;这里`description`是支持多行文本的，但是在 YAML 里面要想支持多行文本，那么 *string* 属性必须以`|`管道符 开头。   
&emsp;&emsp;注意，这里第 1 行`openapi`下面的波浪线表示文档验证失败。   
 
&emsp;&emsp;在 OAS 文档里，一个操作必须在`responses`属性里提供至少一个响应： 
![ ](7.png)

&emsp;&emsp;一个 Action 可能有多种响应结果，每种可能的响应结果都要在`responses`属性中描述。   
&emsp;&emsp;每个响应都以**状态码**进行标识，并且必须包含一个`description`属性。   
&emsp;&emsp;注意：状态码数字必须用双引号`""`括起来，因为它的类型本应该是字符串，而这里的 200 是一个数字。 
 
&emsp;&emsp;下面我再添加一个 POST Action： 
![ ](8.png)

&emsp;&emsp;这里还是针对`/products`这个资源，我就不过多解释了。 

# 使用 OpenAPI 和 JSON Schema 来描述 API 的数据
&emsp;&emsp;OAS 依赖于 JSON Schema 标准来对所有的数据（查询参数，body 参数，响应 body 等）进行描述。   
&emsp;&emsp;注意，OAS 使用的其实是 JSON Schema 的一个子集，并不包含所有的 JSON Schema 特性， 并且还添加了一些 OAS 独有的特性到这个子集里。 

## 描述查询参数
&emsp;&emsp;如果我们的 get 操作里需要一些查询参数（查询字符串，Query String），那么可以使用`parameters`这个属性： 
![ ](9.png)

&emsp;&emsp;这里`parameters`属性是一个集合或数组，每个集合元素使用`-`开头。 

&emsp;&emsp;为了描述一个参数，至少需要`name`，`in`和`schema`三个属性。在本例中，还包含`required`和`description`两个可选的属性。 
* `in`表示参数的位置，这里值为 *query* ，表述它是查询字符串（Query String，例如`api/products?searchTerm=xxx`）。  
* `required`为 *false* 表示不是必填参数。`required`是可选的，如果没有写的话，那么它的值就是 *false* 。但是最好还是写上`required`属性。 
* 它的数据结构使用`schema`属性来表示，这里就是一个简单的字符串类型。但是它其实是一个 JSON schema，所以它可以是复杂的对象类型。 
* `description`属性也是可选的，但是最好还是写上吧，有个描述更好。 

## 使用 JSON Schema 来描述数据
&emsp;&emsp;假设一个对象有三个属性：编号（ *string* ），名称（ *string* ），价格（ *number* ）。那么使用 JSON Schema 来描述它就应该是这样的： 
![ ](10.png)

&emsp;&emsp;还没完，我还必须指出属性是否是必填的，然后我再加上一个 remark 属性，它不是必填的： 
![ ](11.png)

&emsp;&emsp;JSON Schema 通过`required`这个集合属性来表示哪些属性是必填的。 

&emsp;&emsp;此外， 我还可以在这里添加`description`和`example`（示例）属性： 
![ ](12.png)

&emsp;&emsp;此外 JSON Schema 还支持 对象属性类型： 
![ ](13.png)

&emsp;&emsp;JSON Schema 的东西比较多，具体可以查找一下官方文档。 

## 描述响应
&emsp;&emsp;在 OAS 文档里，操作响应返回的 body 里的数据是用`content`属性来表示： 
![ ](14.png)

&emsp;&emsp;这里需要注意的就是该操作的结果是产品的数组，所以类型是 *array* ， 而 *array* 的`items`属性就包含着数组元素的 schema。 

## 描述 body 参数
&emsp;&emsp;像 POST 这样的 Action，它的参数是在请求的 body 里面。   
&emsp;&emsp;body 参数需要使用 requestBody 属性描述，看代码： 
![ ](15.png)

&emsp;&emsp;这个 body 参数的内容也是使用 JSON Schema 来描述的。 

## 描述路由参数
&emsp;&emsp;像`api/products/{productId}`这样的 URI 里，productId 就是一个路由/路径参数。   
&emsp;&emsp;它可以这样描述： 
![ ](16.png)

&emsp;&emsp;这里面 name 的值必须和`{}`里面的值一样。   
&emsp;&emsp;`in`的值为 *path* ，表示是路径参数。   
&emsp;&emsp;路径参数是必填的，所以`required`为 *true* 。不然解析器会报错。 

# 可复用组件
&emsp;&emsp;OAS 允许使用可复用的组件，例如 schema，参数，响应等等，使用它们的时候添加个引用就行。 
 
&emsp;&emsp;假设针对`/products`这个资源一共有两个操作：一个是返回一组产品，另一个返回单个产品。这时候返回产品的 JSON Schema 就可以使用一个可复用的 schema。   
&emsp;&emsp;可复用的组件要放在`components`区域，它是 OAS 文档的一个根级属性。看例子： 
![ ](17.png)

&emsp;&emsp;这里面，可复用的 schema 被定义在`schemas`属性里，每个可重用的 schema 的名字就是`schemas`的值，这里就是 product。它下就包含着可重用的组件：一个 JSON Schema。 

## 引用定义好的 schema
&emsp;&emsp;引用定义好的 schema 需要使用到 JSON 引用。JSON 引用这个属性的名字是`$ref`，它的值是一个 URL。这个 URL 可指向本文档内部甚至外部的组件。这里我只引用文档内部的组件。 
![ ](18.png)

&emsp;&emsp;而针对那个 get Action 的返回结果（数组类型），需要把 JSON 引用放在 *array* 的`items`属性里。 

## 可复用参数
&emsp;&emsp;直接看代码： 
![ ](19.png)

&emsp;&emsp;和可复用 schema 类似，可复用参数也放在`components`下面，它所在的区域是`parameters`。其引用方式也类似，就不过多介绍了。 

&emsp;&emsp;除了在 Action 级别引用可复用参数，在资源这个级别也可以这样做： 
![ ](20.png)

# 预览
![ ](21.png)
![ ](22.png)
