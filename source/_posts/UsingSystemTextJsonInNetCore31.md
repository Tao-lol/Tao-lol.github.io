---
title: .NET Core 3.1 中的 Json 互操作最全解读
tags:
  - .NET
categories:
  - 编程
date: 2019-12-30 13:38:21
---

> https://mp.weixin.qq.com/s/OSPxIGiJ1rRw1Sz-kZqgvQ  

<!--more-->

# 前言
&emsp;&emsp;本文将会全面介绍 System.Text.Json 和 Newtonsoft.Json 的相同和异同之处，方便需要的同学做迁移使用。

# 文档比较
## 几个重要的对象
&emsp;&emsp;在 System.Text.Json 中，有几个重量级的对象，所有的 JSON 互操作，都是围绕这几个对象进行，只要理解了他们各自的用途用法，就基本上掌握了 JSON 和实体对象的互操作。

### JsonDocument
&emsp;&emsp;提供用于检查 JSON 值的结构内容，而不自动实例化数据值的机制。JsonDocument 有一个属性 RootElement，提供对JSON文档根元素的访问，RootElement 是一个 JsonElement 对象。

### JsonElement
&emsp;&emsp;提供对 JSON 值的访问，在 System.Text.Json 中，大到一个对象、数组，小到一个属性、值，都可以通过 JsonElement 进行互操作。

### JsonProperty
&emsp;&emsp;JSON 中最小的单元，提供对属性、值的访问。

### JsonSerializer
&emsp;&emsp;提供 JSON 互操作的静态类，提供了一系列 Serializer / Deserialize 的互操作的方法，其中还有一些异步/流式操作方法。

### JsonSerializerOptions
&emsp;&emsp;与上面的 JsonSerializer 配合使用，提供自定义的个性化互操作选项，包括命名、枚举转换、字符转义、注释规则、自定义转换器等等操作选项。

### Utf8JsonWriter / Utf8JsonReader
&emsp;&emsp;这两个对象是整个 System.Text.Json 的核心对象，所有的JSON互操作几乎都是通过这两个对象进行，他们提供的高性能的底层读写操作。

## 初始化一个简单的 JSON 对象
&emsp;&emsp;在 System.Text.Json 中，并未提供像 JToken 那样非常便捷的创建对象的操作，想要创建一个 JSON 对象，其过程是比较麻烦的，请看下面的代码，进行对比：
```cs
// Newtonsoft.Json.Linq;
JToken root = new JObject();
root["Name"] = "Ron";
root["Money"] = 4.5;
root["Age"] = 30;
string jsonText = root.ToString();

// System.Text.Json
string json = string.Empty;
using (MemoryStream ms = new MemoryStream())
{
    using (Utf8JsonWriter writer = new Utf8JsonWriter(ms))
    {
        writer.WriteStartObject();
        writer.WriteString("Name", "Ron");
        writer.WriteNumber("Money", 4.5);
        writer.WriteNumber("Age", 30);
        writer.WriteEndObject();
        writer.Flush();
    }
    json = Encoding.UTF8.GetString(ms.ToArray());
}
```

&emsp;&emsp;System.Text.Json 的操作便利性在这方面目前处于一个比较弱的状态，不过，从这里也可以看出，可能官方并不希望我们去直接操作 JSON 源，而是通过操作实体对象以达到操作 JSON 的目的，也可能对互操作是性能比较自信的表现吧。

## 封装和加载
&emsp;&emsp;在对 JSON 文档进行包装的用法：
```cs
var json = "{\"name\":\"Ron\",\"money\":4.5}";

var jDoc = System.Text.Json.JsonDocument.Parse(json);
var jToken = Newtonsoft.Json.Linq.JToken.Parse(json);
```

&emsp;&emsp;我发现 MS 这帮人很喜欢使用 Document 这个词,包括 XmlDocument / XDocument 等等。

## 查找元素（对象）
```cs
var json = "{\"name\":\"Ron\",\"money\":4.5}";
var jDoc = System.Text.Json.JsonDocument.Parse(json);
var obj = jDoc.RootElement[0];// 这里会报错，索引仅支持 Array 类型的JSON文档

var jToken = Newtonsoft.Json.Linq.JToken.Parse(json);
var name = jToken["name"];
```

&emsp;&emsp;你看，到查找元素环节就体现出差异了，JsonDocuemnt 索引仅支持 Array 类型的JSON文档，而 JToken 则支持 object 类型的索引（充满想象），用户体验高下立判。那我们不禁要提问了，如何在 JsonDocument 中查找元素？答案如下：
```cs
var json = "{\"name\":\"Ron\",\"money\":4.5}";
var jDoc = System.Text.Json.JsonDocument.Parse(json);
var enumerate = jDoc.RootElement.EnumerateObject();
while (enumerate.MoveNext())
{
    if (enumerate.Current.Name == "name")
        Console.WriteLine("{0}:{1}", enumerate.Current.Name, enumerate.Current.Value);
}
```

&emsp;&emsp;从上面的代码来看，JsonElement 存在两个迭代器，分别是 EnumerateArray 和 EnumerateObject；通过迭代器，你可以实现查找元素的需求。你看，MS 关上了一扇门，然后又为了打开了一扇窗，还是很人性化的了。在 System.Text.Json 中，一切对象都是 Element，Object / Array / Property，都是 Element，这个概念和 XML 一致，但是和 Newtonsoft.Json 不同，这是需要注意的地方。  
&emsp;&emsp;你也可以选择不迭代，直接获取对象的属性，比如使用下面的方法：
```cs
var json = "{\"name\":\"Ron\",\"money\":4.5}";
var jDoc = System.Text.Json.JsonDocument.Parse(json);
var age = jDoc.RootElement.GetProperty("age");
```

&emsp;&emsp;上面这段代码将抛出异常，因为属性 age 不存在，通常情况下，我们会立即想用一个 ContainsKey 来作一个判断，但是很可惜，JsonElement 并未提供该方法，而是提供了一个 TryGetProperty 方法；所以，除非你明确知道 json 对象中的属性，否则一般情况下，建议使用 TryGetProperty 进行取值。  
&emsp;&emsp;就算是这样，使用 GetProperty / TryGetProperty 得到的值，还是一个 JsonElement 对象，并不是你期望的“值”。所以 JsonElement 很人性化的提供了各种 GetIntxx / GetString 方法，但是就算如此，还是可能产生意外，思考下面的代码：
```cs
var json = "{\"name\":\"Ron\",\"money\":4.5,\"age\":null}";
var jDoc = System.Text.Json.JsonDocument.Parse(json);
var property = jDoc.RootElement.GetProperty("age");
var age = property.GetInt32();
```

&emsp;&emsp;上面的代码，最后一行将抛出异常，因为你尝试从一个 null 到 int32 的类型转换，怎么解决这种问题呢，又回到了 JsonElement 上面来，他又提供了一个对值进行检查的方法：
```cs
if (property.ValueKind == JsonValueKind.Number)   
{
    var age = property.GetInt32();
}
```

&emsp;&emsp;这个时候，程序运行良好，JsonValueKind 枚举提供了一系列的类型标识，为了进一步缩小内存使用率，Json 团队用心良苦的将枚举值声明为：byte 类型（够抠）
```cs
public enum JsonValueKind : byte
{
    Undefined = 0,
    Object = 1,
    Array = 2,
    String = 3,
    Number = 4,
    True = 5,
    False = 6,
    Null = 7
}
```

&emsp;&emsp;看到这里，你是不是有点想念 Newtonsoft.Json 了呢？别着急，下面我给大家介绍一个宝贝 System.Json.dll。

## System.Json
### 基本介绍
&emsp;&emsp;System.Json 提供了对 JSON 对象序列化的基础支持，但是也是有限的支持，请看下图：
![ ](1.webp)

&emsp;&emsp;System.Json 目前已合并到 .NETCore-3.1 中，如果你希望使用他，需要单独引用：
```cs
Install-Package System.Json -Version 4.7.0
```

&emsp;&emsp;这个 JSON 互操作包提供了几个常用的操作类型，从下面的操作类不难看出，提供的支持是非常有限的，而且效率上也不好说。
```cs
System.Json.JsonArray
System.Json.JsonObject
System.Json.JsonPrimitive
System.Json.JsonValue
```

&emsp;&emsp;首先，JsonObject 是实现 IDictionary 接口，并在内部维护一个 SortedDictionary 字典，所以他具备字典类的一切操作，比如索引等等，JsonArray 就更简单，也是一样的实现 IList 接口，然后同样的在内部维护一个 List 链表，以实现数组功能，对象的序列化都是通过 JsonValue 进行操作，序列化的方式也是非常的简单，就是对对像进行迭代，唯一值得称道的地方是，采用了流式处理。

#### 使用 System.Json 操作上面的查找过程如下
```cs
var obj = System.Json.JsonObject.Parse("{\"name\":\"ron\"}");
if (obj.ContainsKey("age"))
{
    int age = obj["age"];
}
```

&emsp;&emsp;令人遗憾的是，虽然 System.Json 已经合并到 .NETCore-3.1 的路线图中；但是，System.Text.Json 不提供对 System.Json 的互操作性，我们期待以后 System.Text.Json 也能提供 System.Json 的操作便利性。

## 序列化和反序列化
&emsp;&emsp;基本知识已经介绍完成，下面我们进入 System.Text.Json 的内部世界一探究竟。

### 互操作
&emsp;&emsp;思考下面的代码：
```cs
// 序列化
var user = new UserInfo { Name = "Ron", Money = 4.5m, Age = 30 };
var json = JsonSerializer.Serialize(user);

// 输出
{"Name":"Ron","Money":4.5,"Age":30}

// 反序列化
user = JsonSerializer.Deserialize<UserInfo>(json);
```

&emsp;&emsp;目前为止，上面的代码工作良好。让我们对上面的代码稍作修改，将 JSON 字符串进行一个转小写的操作后再进行反序列化的操作：
```cs
// 输出
{"name":"Ron","money":4.5,"age":30}

// 反序列化
user = JsonSerializer.Deserialize<UserInfo>(json);
```

&emsp;&emsp;上面的代码可以正常运行，也不会抛出异常，你可以得到一个完整的 user 对象；但是，user 对象的属性值将会丢失！这是因为 System.Text.Json 默认采用的是区分大小写匹配的方式，为了解决这个问题，我们需要引入序列化操作个性化设置，请参考下面的代码，启用忽略大小写的设置：
```cs
// 输出
{"name":"Ron","money":4.5,"age":30}

var options = new JsonSerializerOptions()   
{
    PropertyNameCaseInsensitive = true   
};
// 反序列化
user = JsonSerializer.Deserialize<UserInfo>(json,options);
```

### 格式化 JSON
&emsp;&emsp;现在你可以选择对序列化的 JSON 文本进行美化，而不是输出上面的压缩后的 JSON 文本，为了实现美化的效果，你仅仅需要在序列化的时候加入一个 WriteIndented 设置：
```cs
var options = new JsonSerializerOptions();
options.WriteIndented = true;

var user = new UserInfo { Name = "Ron", Money = 4.5m, Age = 30, Remark = "你好，欢迎！" };
var json = JsonSerializer.Serialize(user, options);
 
// 输出
{
  "Name": "Ron",
  "Money": 4.5,
  "Age": 30,
  "Remark": "\u4F60\u597D\uFF0C\u6B22\u8FCE\uFF01"
}
```

&emsp;&emsp;你看，就是这么简单，但是你也发现了，上面的 Remark 属性在序列化后，中文被转义了，这就是接下来要解决的问题。

### 字符转义的问题
&emsp;&emsp;在默认情况下，System.Text.Json 序列化程序对所有非 ASCII 字符进行转义；这就是中文被转义的根本原因。但是在内部，他又允许你自定义控制字符集的转义行为，这个设置就是：Encoder，比如下面的代码，对中文进行转义的例外设置，需要创建一个 TextEncoderSettings 对象，并将 UnicodeRanges.All 加入允许例外范围内，并使用 JavaScriptEncoder 根据 TextEncoderSettings 创建一个 JavaScriptEncoder 对象即可。
```cs
var encoderSettings = new TextEncoderSettings();
encoderSettings.AllowRanges(UnicodeRanges.All);
var options = new JsonSerializerOptions();
options.Encoder = JavaScriptEncoder.Create(encoderSettings);
options.WriteIndented = true;
var user = new UserInfo { Name = "Ron", Money = 4.5m, Age = 30, Remark = "你好，欢迎！" };
var json = JsonSerializer.Serialize(user, options);

// 输出
{
  "Name": "Ron",
  "Money": 4.5,
  "Age": 30,
  "Remark": "你好，欢迎！"
}
```

&emsp;&emsp;还有另外一种模式，可以不必设置例外而达到不转义的效果，这个模式就是 “非严格 JSON” 模式，将上面的 JavaScriptEncoder.Create(encoderSettings) 替换为下面的代码：
```cs
options.Encoder = JavaScriptEncoder.UnsafeRelaxedJsonEscaping;
```

### 序列化相关 - 异步/流式
&emsp;&emsp;System.Text.Josn 提供了一系列丰富的 JSON 互操作，这其中包含异步和流式处理，这点也是和 Newtonsoft.Json 最大的不同，但不管是那种方式，都要牢记，最后都是通过下面的两个类来实现：
```cs
System.Text.Json.Utf8JsonReader
System.Text.Json.Utf8JsonWriter
```

### 自定义 JSON 名称和值
&emsp;&emsp;在默认情况下，输出的 JSON 属性名称保持和实体对象相同，包括大小写的都是一致的，枚举类型在默认情况下被序列化为数值类型。System.Text.JSON 提供了一系列的设置和扩展来帮助开发者实现各种自定义的需求。下面的代码可以设置默认的 JSON 属性名称，这个设置和 Newtonsoft.Json 基本一致。
```cs
public class UserInfo
{
    [JsonPropertyName("name")] public string Name { get; set; }
    public decimal Money { get; set; }
    public int Age { get; set; }
    public string Remark { get; set; }
}
```

&emsp;&emsp;UserInfo 的 属性 Name 在输出为 JSON 的时候，其字段名称将为：name，其他属性保持大小写不变。

### 对所有属性设置为 camel 大小写
```cs
var options = new JsonSerializerOptions
{
    PropertyNamingPolicy = JsonNamingPolicy.CamelCase
};

jsonSerializer.Serialize(user, options);
```

### 自定义名称策略
&emsp;&emsp;比如我们的系统，目前采用全小写的模式，那么我可以自定义一个转换器，并应用到序列化行为中。
```cs
public class LowerCaseNamingPolicy : JsonNamingPolicy
{
    public override string ConvertName(string name) => name.ToLower();
}

var options = new JsonSerializerOptions();
// 应用策略
options.PropertyNamingPolicy = new LowerCaseNamingPolicy();

var user = new UserInfo { Name = "Ron", Money = 4.5m, Age = 30 };
var json = JsonSerializer.Serialize(user, options);
```

### 将枚举序列化为名称字符串而不是数值
```cs
var options = new JsonSerializerOptions();
// 添加转换器
options.Converters.Add(new JsonStringEnumConverter());

var user = new UserInfo { Name = "Ron", Money = 4.5m, Age = 30 };
var json = JsonSerializer.Serialize(user, options);
```
## 排除不需要序列化的属性
&emsp;&emsp;在默认情况下，所有公共属性将被序列化为 JSON。但是，如果你不想让某些属性出现在 JSON 中，可以通过下面的几种方式实现属性排除。

### 排除所有属性值为 null 属性
```cs
var options = new JsonSerializerOptions();
options.Encoder = JavaScriptEncoder.UnsafeRelaxedJsonEscaping;
options.IgnoreNullValues = true;
var user = new UserInfo { Name = "Ron", Money = 4.5m, Age = 30, Remark = null};
var json = JsonSerializer.Serialize(user, options);

// 输出，可以看到，Remark 属性被排除
{"name":"Ron","Money":4.5,"Age":30}
```

### 排除指定标记属性
&emsp;&emsp;可以为某个属性应用 JsonIgnore 特性，标记为不输出到 JSON。
```cs
public class UserInfo
{
    [JsonPropertyName("name")] public string Name { get; set; }
    public decimal Money { get; set; }
    [JsonIgnore]public int Age { get; set; }
    public string Remark { get; set; }
}

var user = new UserInfo { Name = "Ron", Money = 4.5m, Age = 30, Remark =null};
var json = JsonSerializer.Serialize(user);

// 输出，属性 Age  已被排除
{"name":"Ron","Money":4.5,"Remark":null}
```

### 排除所有只读属性
&emsp;&emsp;还可以选择对所有只读属性进行排查输出 JSON，比如下面的代码，Password 是不需要输出的，那么我们只需要将 Password 设置为 getter，并应用 IgnoreReadOnlyProperties = true 即可。
```cs
public class UserInfo
{
    [JsonPropertyName("name")] public string Name { get; set; }
    public decimal Money { get; set; }
    [JsonIgnore] public int Age { get; set; }
    public int Password { get; }
    public string Remark { get; set; }
}

var options = new JsonSerializerOptions
{
    IgnoreReadOnlyProperties = true
};
var user = new UserInfo { Name = "Ron", Money = 4.5m, Age = 30, Remark = null };
var json = JsonSerializer.Serialize(user, options);

// 输出
{"name":"Ron","Money":4.5,"Remark":null}
```

### 排除派生类的属性
&emsp;&emsp;在某些情况下，由于业务需求的不同，需要实现实体对象的继承，但是在输出 JSON 的时候，希望只输出基类的属性，而不要输出派生类型的属性，以避免产生不可控制的数据泄露问题；那么，我们可以采用下面的序列化设置。比如下面的 UserInfoExtension 派生自 UserInfo，并扩展了一个属性为身份证的属性，在输出 JSON 的时候，我们希望不要序列化派生类，那么我们可以在 Serialize 序列化的时候，指定序列化的类型为基类：UserInfo，即可达到隐藏派生类属性的目的。
```cs
public class UserInfo
{
    [JsonPropertyName("name")] public string Name { get; set; }
    public decimal Money { get; set; }
    [JsonIgnore] public int Age { get; set; }
    public int Password { get; }
    public string Remark { get; set; }
}

public class UserInfoExtension : UserInfo
{
    public string IdCard { get; set; }
}

var user = new UserInfoExtension { Name = "Ron", Money = 4.5m, Age = 30, Remark = null };
var json = JsonSerializer.Serialize(user, typeof(UserInfo));

// 输出
{"name":"Ron","Money":4.5,"Password":0,"Remark":null}
```

### 仅输出指定属性（排除属性的逆向操作）
&emsp;&emsp;在 Newtonsoft.Json 中，我们可以通过指定 MemberSerialization 和 JsonProperty 来实现输出指定属性到 JSON 中，比如下面的代码：
```cs
[Newtonsoft.Json.JsonObject(Newtonsoft.Json.MemberSerialization.OptIn)]
public class UserInfo
{
    [Newtonsoft.Json.JsonProperty("name")] public string Name { get; set; }
    public int Age { get; set; }
}
 
var user = new UserInfo() { Age = 18, Name = "Ron" };
var json = Newtonsoft.Json.JsonConvert.SerializeObject(user);

// 输出
{"name":"Ron"}
```

&emsp;&emsp;不过，很遗憾的告诉大家，目前 System.Text.Json 不支持这种方式；为此，我特意去看了 corefx 的 issue，我看到了下面这个反馈：
![ ](2.webp)

&emsp;&emsp;现在确定方向了，当 .NETCore 合并到主分支 .NET 也就是 .NET5.0 的时候，官方将提供支持，在此之前，还是使用推荐 Newtonsoft.Json 。

### 在反序列化的时候，允许 JSON 文本包含注释
&emsp;&emsp;默认情况下，System.Text.JSON 不支持源 JSON 文本包含注释，比如下面的代码，当你不使用 ReadCommentHandling = JsonCommentHandling.Skip 的设置的时候，将抛出异常，因为在字段 Age 的后面有注释 /* age */。
```cs
var jsonText = "{\"Name\":\"Ron\",\"Money\":4.5,\"Age\":30/* age */}";
var options = new JsonSerializerOptions
{
    ReadCommentHandling = JsonCommentHandling.Skip,
    AllowTrailingCommas = true,
};
var user = JsonSerializer.Deserialize<UserInfoExtension>(jsonText);
```

### 允许字段溢出
&emsp;&emsp;在接口数据出现变动时，极有可能出现源 JSON 文本和实体对象属性不匹配的问题，JSON 中可能会多出一些实体对象不存在的属性，这种情况我们称之为“溢出”，在默认情况下，溢出的属性将被忽略，如果希望捕获这些“溢出”的属性，可以在实体对象中声明一个类型为：Dictionary 的属性，并对其应用特性标记：JsonExtensionData。  
&emsp;&emsp;为了演示这种特殊的处理，我们声明了一个实体对象 UserInfo，并构造了一个 JSON 源，该 JSON 源包含了一个 UserInfo 不存在的属性：Money，预期该 Money 属性将被反序列化到属性 ExtensionData 中。
```cs
public class UserInfo
{
    public string Name { get; set; }
    public int Age { get; set; }
    [JsonExtensionData] public Dictionary<string, object> ExtensionData { get; set; }
}

var jsonText = "{\"Name\":\"Ron\",\"Money\":4.5,\"Age\":30}";
var user = JsonSerializer.Deserialize<UserInfo>(jsonText);
```

![输出截图](3.webp)

&emsp;&emsp;有意思的是，被特性 JsonExtensionData 标记的属性，在序列化为 JSON 的时候，他又会将 ExtensionData 的字典都序列化为单个 JSON 的属性，这里不再演示，留给大家去体验。

## 转换器
&emsp;&emsp;System.Text.Json 内置了各种丰富的类型转换器，这些默认的转换器在程序初始化 JsonSerializerOptions 的时候就默认加载，在 JsonSerializerOptions 内部，维护着一个私有静态成员 s_defaultSimpleConverters，同时还有一个公有属性 Converters ，Converters 属性在 JsonSerializerOptions 的构造函数中被初始化；从下面的代码中可以看到，默认转换器集合和公有转换器集是相互独立的，System.Text.Json 允许开发人员通过 Converters 添加自定义的转换器。
```cs
public sealed partial class JsonSerializerOptions
{
    // The global list of built-in simple converters.
    private static readonly Dictionary<Type, JsonConverter> s_defaultSimpleConverters = GetDefaultSimpleConverters();
    // The global list of built-in converters that override CanConvert().
    private static readonly List<JsonConverter> s_defaultFactoryConverters = GetDefaultConverters();
    // The cached converters (custom or built-in).
    private readonly ConcurrentDictionary<Type, JsonConverter> _converters = new ConcurrentDictionary<Type, JsonConverter>();

    private static Dictionary<Type, JsonConverter> GetDefaultSimpleConverters()
    {
        ...
    }

    private static List<JsonConverter> GetDefaultConverters()
    {
       ...
    }

    public IList<JsonConverter> Converters { get; }
    ...
}
```

### 内置转换器
&emsp;&emsp;在 System.Text.Json 内置的转换器集合中，涵盖了所有的基础数据类型，这些转换器的设计非常精妙，他们通过注册一系列的类型映射，在通过 Utf8JsonWriter / Utf8JsonReader 的内置方法 GetTypeValue / TryGetTypeValue 方法得到值，代码非常精练，复用性非常高，下面是内置类型转换器：
```cs
private static IEnumerable<JsonConverter> DefaultSimpleConverters
{
    get
    {
        // When adding to this, update NumberOfSimpleConverters above.
        yield return new JsonConverterBoolean();
        yield return new JsonConverterByte();
        yield return new JsonConverterByteArray();
        yield return new JsonConverterChar();
        yield return new JsonConverterDateTime();
        yield return new JsonConverterDateTimeOffset();
        yield return new JsonConverterDouble();
        yield return new JsonConverterDecimal();
        yield return new JsonConverterGuid();
        yield return new JsonConverterInt16();
        yield return new JsonConverterInt32();
        yield return new JsonConverterInt64();
        yield return new JsonConverterJsonElement();
        yield return new JsonConverterObject();
        yield return new JsonConverterSByte();
        yield return new JsonConverterSingle();
        yield return new JsonConverterString();
        yield return new JsonConverterUInt16();
        yield return new JsonConverterUInt32();
        yield return new JsonConverterUInt64();
        yield return new JsonConverterUri();
    }
}
```

### 自定义类型转换器
&emsp;&emsp;虽然 System.Text.Json 内置了各种各样丰富的类型转换器，但是在各种业务开发的过程中，总会根据业务需求来决定一些特殊的数据类型的数据，下面，我们就以经典的日期/时间转换作为演示场景。  
&emsp;&emsp;我们需要将日期类型输出为 Unix 时间戳而不是格式化的日期内容，为此，我们将实现一个自定义的时间格式转换器，该转换器继承自 JsonConverter。
```cs
public class JsonConverterUnixDateTime : JsonConverter<DateTime>
{
    private static DateTime Greenwich_Mean_Time = TimeZoneInfo.ConvertTime(new DateTime(1970, 1, 1), TimeZoneInfo.Local);
    private const int Limit = 10000;
    public override DateTime Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
    {
        if (reader.TokenType == JsonTokenType.Number)
        {
            var unixTime = reader.GetInt64();
            var dt = new DateTime(Greenwich_Mean_Time.Ticks + unixTime * Limit);
            return dt;
        }
        else
        {
            return reader.GetDateTime();
        }
    }
    public override void Write(Utf8JsonWriter writer, DateTime value, JsonSerializerOptions options)
    {
        var unixTime = (value - Greenwich_Mean_Time).Ticks / Limit;
        writer.WriteNumberValue(unixTime);
    }
}
```

### 应用自定义的时间转换器
&emsp;&emsp;转换器的应用形式有两种，分别是将转换加入 JsonSerializerOptions.Converters 和给需要转换的属性添加特性标记 JsonConverter。

#### 加入Converters 方式
```cs
var options = new JsonSerializerOptions();
options.Converters.Add(new JsonConverterUnixDateTime());
var user = new UserInfo() { Age = 30, Name = "Ron", LoginTime = DateTime.Now };
var json = JsonSerializer.Serialize(user, options);
var deUser = JsonSerializer.Deserialize<UserInfo>(json, options);

// JSON 输出
{"Name":"Ron","Age":30,"LoginTime":1577655080422}
```

#### 应用 JsonConverter 特性方式
```cs
public class UserInfo
{
    public string Name { get; set; }
    public int Age { get; set; }
    [JsonConverter(typeof(JsonConverterUnixDateTime))]
    public DateTime LoginTime { get; set; }
}

var user = new UserInfo() { Age = 30, Name = "Ron", LoginTime = DateTime.Now };
var json = JsonSerializer.Serialize(user);
var deUser = JsonSerializer.Deserialize<UserInfo>(json);

// JSON 输出
{"Name":"Ron","Age":30,"LoginTime":1577655080422}
```

&emsp;&emsp;注意上面的 UserInfo.LoginTime 的特性标记，当你想小范围的对某些属性单独应用转换器的时候，这种方式费用小巧而有效。

# 结束语
&emsp;&emsp;本文全面的介绍了 System.Text.Json 在各种场景下的用法，并比较和 Newtonsoft.Json 使用上的不同，也通过实例演示了具体的使用方法，进一步深入讲解了 System.Text.Json 各种对象的原理，希望对大家在迁移到.NETCore-3.1 的时候有所帮助。