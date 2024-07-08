# 第十二章：序列化和传输

为了序列化一个对象，我们将抽象概念（“模型”）转换为可传输的实体，通常是模型的字符串表示，比如 XML 或 JSON，或者字节。

反序列化数据意味着将其从一系列实体转换为程序所识别的对象。例如，你可能有一个带有 `name` 属性的 `Author` 实体。

# 任务

在本章中，你将学习：

1.  序列化和反序列化对象实例。

# Android

Android 内部大量使用 XML，但在现实世界中，JSON 仍然是主要的序列化机制（尽管拥有大量工程资源的大型组织已经开始采用“protobuf”，或 Protocol Buffers）。这是一个巧妙的概念并且高性能，但超出了我们对框架和标准 API 检查的范围。

## 序列化和反序列化对象实例

在 Java 和 Android 中，反序列化可能从数据模型开始，就像这样：

反序列化为 JSON，调用 `getName` 返回 “Mike” 的 `Author` 实例可能看起来像这样：

```
{ name : "Mike" }
```

一旦它以 JSON 格式存在，就可以随网络请求传输，写入磁盘，甚至传递给另一个采用完全不同技术的程序；因为 JSON 格式具有已建立的规范，我们可以信任 Android 应用使用的 JSON 规则与 iOS 应用或甚至 Windows 或 Unix 程序使用的 JSON 规则相似。

Android 框架中实际上有三种主要的序列化模式：

+   JSON

+   XML

+   Java 序列化

这些大致按受欢迎程度排序。虽然 Java 标准库或 Android 框架都提供了各种程度的支持，我们还将看看像 [Gson](https://oreil.ly/RRhOS) 这样的第三方库。虽然 Gson 是 Google 产品，但也有一些非常流行的替代品——如果你不认为 Gson 或 org.json 有吸引力，或者你的应用程序不使用大量 JSON，请快速搜索一下。

### org.json

参见 [此包的开发者文档](https://oreil.ly/5Zw0T) 以供参考。

使用 `org.json` 包进行基本序列化和反序列化非常简单。让我们考虑前面使用的 `Author` 类。要将其序列化，我们可以实例化一个 `JSONObject` 实例，将我们的属性复制到其中，然后调用 `toString` 方法：

这将打印出 `{"name":"Mike"}`。

反序列化更加自动化：

这将记录 `Mike`。

数组（列表）也有类似的功能，但通过这种方式处理大对象并处理检查异常会有相当多的转换，而且不像 Gson 那样常见。我们来看看这个。

使用 Gson，序列化工作方式非常类似，但你不需要中介包装器：

```
Author author = new Author();
author.setName("Mike");
Log.d("MyTag", new Gson().toJson(author));
```

这将输出 `{"mName":"Mike"}`。

你注意到了吗？`mName`，而不是 `name`。Gson 默认使用属性名，而不是 getter 或 setter 方法名。

这种类型的注释——使用`m`前缀成员变量和`s`前缀静态变量——被称为匈牙利命名法。你会看到 AOSP 本身完全使用匈牙利命名法，因此许多 Android 开发者也开始使用这种风格。正如接下来将要看到的，这有一个简单的修复方法，但请注意，匈牙利命名法在 Gson（或类似库）中确实存在问题。请注意，匈牙利（或任何任意的）命名法的问题在 Kotlin 中并不存在。如果你的`Author`类如 Kotlin 示例所示定义，输出将如你所料：`{"name": "Mike"}`。

回到 Java 示例。这个问题很常见，但修改起来也很简单——你可以使用`@SerializedName`注解任何属性以使用其他名称。例如，假设`Author`类看起来像这样：

```
public class Author {
  @SerializedName("name")
  private String mName;
  public String getName() {
    return mName;
  }
  public void setName(String name) {
    mName = name;
  }
}
```

然后输出将如你所料：`{"name":"Mike"}`。请注意，这两者皆可，并且它将从注解值反序列化。让我们假设我们已经更新了`Author`类，用于下一个示例，反序列化：

```
String json = "{name:'Mike'}";
Author author = new Gson().fromJson(json, Author.class);
Log.d("MyTag", author.getName());
```

这将输出`Mike`，就像你所期望的那样。

Gson 的最大好处在于它将以递归策略处理此问题，而`org.json`类通常不会。

### org.xmlpull

另一个常见的传输方式是 XML。如果你曾经为网络编写过代码，肯定见过 XML 或其某个变种——如果你看到充斥着尖括号的数据，那很可能是 XML 或者其衍生物。

虽然 Android 框架中的 Java 标准库没有提供任何用于处理 XML 的内置 API，但 Android 确实提供了一个第三方包，该包是经过维护和审查的：`org.xmlpull`。该包提供了称为“XML pull 解析器”的对象实例，我们可以用它们来读取和遍历 XML 数据。“Pull 解析器”仅仅是从流中请求标记的解析器之一——相反的“推送解析器”在遇到标记时将标记发送到解析器，而不是在请求时发送。这实际上是一个语义上的细节，对于你实际使用这些类的方式并不重要。

无论如何，我们将提供尽可能基本的 XML 解析示例，但即使这个小样本，你也会注意到它冗长而不容易阅读。如果你从 XML 获取数据并需要解析它，请深入研究`org.xml`类，或找一个良好的第三方替代方案。以下是基本操作：

欲深入了解`org.xml`包，请参阅[Android 开发者文档](https://oreil.ly/OEm5A)。

### Java 序列化

Java 序列化也许是最容易理解和从远处看似乎是一个很好的策略。然而，在实践中，存在许多弊端——我们将简要地触及它们，但这并不意味着 Java 序列化从来没有适合的情况。

基本前提很简单：将对象实例转换为字节。然后，您可以将这些字节保存到磁盘或通过网络发送，并根据需要重新组装原始对象。然而，这要求任何试图反序列化以这种方式序列化的对象的程序都必须具有对象的确切 Java 定义——Java 虚拟机（JVM）必须具有与对象的完全限定类名匹配的类，并且如果任何方法或属性在任一方面被访问且不匹配，那么您基本上已经到了尽头。

举个例子，想象一下，您从第三方承包商那里继承了您的应用程序代码。他们使用了一个名为`Chapter`的类，该类具有多种对于表示该章节的对象有用的属性和方法。然而，包名（因此完全限定类名或“规范”名称）包括承包商的域，当您接管应用程序源代码时，您不再拥有匹配的包名。通常情况下，这没有问题，您可以自己定义一个名为`Chapter`的类，该类可能具有与旧`Chapter`类相似的名称或功能的方法和属性，也可能没有。然而，承包商违反了社区共识：他们通过将`Chapter`实例序列化为字节并将该 blob 保存到本地 SQLite 数据库中，将`Chapter`实例存储在用户的收藏列表中。这意味着您*必须*维护一个包名与先前的规范名称完全匹配，包括承包商的域。当然，当您想要定义在您自己的应用程序中如何工作的`Chapter`实例时，这个中间类只存在足够长的时间来从数据库反序列化，转换为`Chapter`类的更新版本，并将其属性保存到我们更新的数据库中。这不仅令人困惑，因为我们必须向每位新开发者讲述这个故事，而且每次打开 Android Studio 时，未使用的承包商包都位于您的源代码顶部，满是仅仅为了使以前版本的用户可以将其内容迁移到新应用程序而存在的骨架类。

话虽如此，让我们深入探讨一下，如果您决定需要的话，如何使用 Java 序列化。

您需要了解的第一件事是，要使用内置的 Java 序列化，被序列化的类必须实现`Serializable`接口。现在，与该接口没有约定（没有您必须实现的方法）；它只是必须实现接口本身。

您还应提供一个`Long`序列化 ID，格式为`private static final long serialVersionUID = 12345467890;`。这里允许使用任何有效的`Long`，并且它应该在您的可序列化类中是唯一的。大多数 Java 或 Android-aware IDE 可以为您生成正确名称和值的这个值。

接下来，剩下的不多了。你将使用我们在 第六章 学到的 `InputStream` 和 `OutputStream` 类的子类将对象写入文件。让我们看看如何序列化一个简单的类——事实上，我们将使用之前展示过的相同的 `Author` 类：

现在，你有一个名为“data.obj”的文件，其中包含表示您对象实例的字节。请注意，这是一个完整的复制，可以在反序列化时将其还原为序列化时的状态，只要类文件在您希望反序列化时位于 CLASSPATH 上。读取它也非常熟悉：

就是这样！如前所述，社区的大部分已经将 JSON 视为网络计算的标准序列化策略，但正如您所见，还有很多选择。要更有趣，快速搜索一下“protobuffs”——这是一种新的、谷歌式的数据传输方式。

# iOS

有多种方法可以在数据和传输实体之间进行转换。通常情况下，在网络请求期间完成这些转换，最常见的两种数据传输格式是 JSON 和 XML，近年来 JSON 比 XML 更受欢迎。在 Swift 对象实例之间进行这两种格式之间的转换过程非常不同。还有一种几乎完全由 Apple 使用的格式叫做属性列表。它们是专用的 XML，iOS（和 macOS）应用程序可以用来读取和写入数据。

Swift 的更新使得解析数据变得更加直接和不易出错，但仍然存在相当的复杂性。换句话说，我们有很多内容要涵盖，让我们深入了解一下。

## 序列化和反序列化对象实例

从历史上看，在 Objective-C 的早期日子里，将对象解析为 JSON 是不必要复杂或需要第三方库的。事实上，即使是 Swift 的早期日子也充满了复杂性和困扰。幸运的是，Apple 关注了开发者的需求，并在 Swift 3 中带来了一种新的现代化的 JSON 序列化和反序列化方法。

### JSON

让我们从一个看起来像这样的 Swift 对象开始：

```
struct Author {
	let name: String
}
```

JSON 表示可能看起来像这样：

```
{ "name": "Mike" }
```

在与服务器通信时，通常需要发送和接收类似这样的 JSON 并将其转换为纯 Swift 对象。将应用程序需要后续读取的数据（本章中涵盖的任何其他格式）作为 JSON 保存在本地也很常见。使用之前定义的 `Author` 结构，将该对象的实例转换为先前显示的 JSON 将看起来类似于这样：

```
struct Author: Codable {
    let name: String
}

let author = Author(name: "Mike")
let rawData = try? JSONEncoder().encode(author)
```

通过这个例子，你可以看到我们为我们的对象添加了一个叫做 `Codable` 的协议。这个协议是两个其他协议 `Encodable` 和 `Decodable` 的组合。它们通过一些语法糖和预期值为 Swift 编译器提供了一组功能。

实现 `Codable` 的对象必须实现 `encode(to:)` 和 `init(from:)`。如果没有提供实现，Swift 编译器不会报错，而是通过查看对象的属性并修改名为 `CodingKeys` 的特殊嵌套枚举类型来自动生成适当的实现。使用这个枚举，编译器可以创建正确的 `encode(to:)` 和 `init(from:)` 实现。

你可以称其为魔法，但实际上是编译器在对代码做出实用决策。

在我们的示例中，通过 `JSONEncoder` 能够将 `author` 编码为一个 `Data` 对象，可以用于本地存储在设备上或通过网络发送到服务器。

`Codable` 还帮助将我们的对象从 JSON 转换（或“反序列化”）回一个普通的 Swift 对象。以下是一个示例：

```
let rawJson = String("{\"name\":\"Mike\"}").data(using: .utf8)!
let author = try? JSONDecoder().decode(Author.self, from: rawJson)
```

首先，我们定义了一个名为 `rawJson` 的 `Data` 对象，其中包含我们可能从服务器收到的 JSON。接下来，我们将该 JSON 传递给 `JSONDecoder` 的实例，并在 `decode(_:from:)` 方法中进行解码。这个方法还需要一个对象类型来尝试将数据转换为；在本例中，我们传递了 `Author.self`，这是前面示例中定义的 `Author` 结构体。解码器将对象解码回一个名为 `author` 的本地 Swift 对象。

###### 注意

现在，如果我们给 `JSONDecoder` 提供了一个无法将数据解码回的对象类型，它将会产生一个错误。我们在对此方法的调用中标记了 `try?`，这对于本示例已足够，但在一个发布的应用程序中，你可能需要通过一些用户反馈或日志处理这个问题。

`Codable` 中还包含更多功能，但超出了本章的范围。现在，让我们来谈谈另一种流行的传输格式：XML。

### XML

XML 是比 JSON 更古老的格式。XML 有各种标准，包括一些非常具体的传输类型，如简单对象访问协议（SOAP），但为了我们的示例，我们将使用稍微修改过的 JSON 版本来保持简单。让我们从这个 XML 开始：

```
<?xml version="1.0" encoding="UTF-8"?>
<author type="human">
	<name>Mike</name>
</author>
```

这个 XML 描述了一个名为“Mike”的作者，碰巧是人类（也是这本书的作者）。

如果你期望 `Codable` 提供与 `JSONEncoder` 和 `JSONDecoder` 一样简单的简洁性，我恐怕要给你些坏消息：iOS 中的 `XMLParser` 要复杂得多。

让我们从相同的 XML 开始，并创建一个开始解析的对象：

```
class SomeObject: NSObject {
    func parseSomeXML() {
        let xml = "<?xml version=\"1.0\" encoding=\"UTF-8\"?>
 <author type=\"human\"><name>Mike</name></author>"
        let rawData = xml.data(using: .utf8)!

        let parser = XMLParser(data: rawData)
        parser.delegate = self;
        parser.parse()
    }
}
```

我们有一个名为 `SomeObject` 的 `NSObject`，它有一个名为 `parseSomeXml` 的方法，我们已经创建了这个方法。这个方法有一个名为 `xml` 的字符串变量，其中包含我们之前编码的 XML。在下一行中，我们将它转换为一个使用 UTF-8 编码的 `Data` 对象。接下来，我们使用 `rawData` 对象实例化一个 `XMLParser`。然后，我们将自己设置为处理解析的代理。最后，我们调用 `parse()` 来开始解析。

如果你现在尝试运行这段代码，会出现错误，因为 `SomeObject` 目前没有实现它应该实现的 `XMLParserDelegate` 协议来正确处理解析操作。这是驱动解析的核心，所以让我们深入了解我们正在实现的每个协议方法的内容。

XML 解析是同步的。文档逐个元素地扫描和遍历。在我们的 XML 解析代理中，我们将使用四种方法。它们是：

1.  `parser(_:didStartElement:namespaceURI:qualifiedName:attributes:)`

1.  `parser(_:foundCharacters:)`

1.  `parser(_:didEndElement:namespaceURI:qualifiedName:)`

1.  `parserDidEndDocument(_:)`

让我们来看看列表中的第一个方法：

```
func parser(_ parser: XMLParser, didStartElement elementName: String, namespaceURI: String?,
  qualifiedName qName: String?, attributes attributeDict: [String : String] = [:]) {
    if elementName == "author" {
        author = Author()
        if let type = attributeDict["type"] {
            author?.type = type
        }
    }
}
```

当我们遍历文档并找到新的 XML 元素时，此方法被调用。在我们之前的示例 XML 中，唯一有效的元素是 `author` 和 `name`。如果我们找到一个新的 `author` 元素，我们会在 `SomeObject` 类中新增一个名为 `author` 的属性，创建一个新的 `Author` 实例。这作为一个临时存储，用于在解析该元素时存储所找到的数据。

发现的第一段数据是属性 `type`。如果你回忆一下，我们的作者有一个类型，在我们特定的 XML 元素中，对于 `Mike` 作者，这是 `human`。我们将临时 `Author` 实例的 `type` 属性设置为此值。

随着文档的解析，我们将找到的下一个元素是 `name`。一旦该元素开始，我们无需执行任何特殊操作，但是我们需要捕获 `<name>` 和 `</name>` 标签之间的数据。我们列表中的下一个方法 `parser(_:foundCharacters:)` 允许我们像这样执行：

```
func parser(_ parser: XMLParser, foundCharacters string: String) {
    characters += string
}
```

在这个方法的主体内部，我们将找到的任何字符存储在另一个新增的属性 `characters` 中，该属性添加到我们的 `SomeObject` 实例中。这个属性充当了一个临时缓冲区，用于在进程的下一步中保留字符，该步骤在找到此标签的闭合元素时发生：

```
func parser(_ parser: XMLParser, didEndElement elementName: String, namespaceURI: String?,
  qualifiedName qName: String?) {
    if elementName == "name" {
        author?.name = characters
    }
    characters = ""
}
```

当解析器找到元素的闭合标签时，将调用此委托方法。在我们的情况下，这是 `</name>`。我们将之前的字符串缓冲区 `characters` 添加到正在构建的 `author` 对象的 `name` 属性中。

最后，我们将创建的内容通过 `print()` 语句显示在 XML 文档的末尾：

```
func parserDidEndDocument(_ parser: XMLParser) {
    print(author.name)
    print(author.type)
}
```

###### 注意

如果在 XML 文档中有更多项，我们将继续构建更多的作者实例，并且我们需要在开始新元素并创建临时对象时保存它们。

这是完整示例：

```
struct Author {
    var name: String?
    var type: String?
}

class SomeObject: NSObject {
    var author: Author?
    var characters: String = ""

    func parseSomeXML() {
        let xml = "<?xml version=\"1.0\" encoding=\"UTF-8\"?>
 <author type=\"human\"><name>Mike</name></author>"
        let rawData = xml.data(using: .utf8)!

        let parser = XMLParser(data: rawData)
        parser.delegate = self;
        parser.parse()
    }
}
extension SomeObject: XMLParserDelegate {
    func parser(_ parser: XMLParser, didStartElement elementName: String,
      namespaceURI: String?, qualifiedName qName: String?, attributes attributeDict:
      [String : String] = [:]) {
        if elementName == "author" {
            author = Author()
            if let type = attributeDict["type"] {
                author?.type = type
            }
        }
    }

    func parser(_ parser: XMLParser, foundCharacters string: String) {
        characters += string
    }

    func parser(_ parser: XMLParser, didEndElement elementName: String,
      namespaceURI: String?,
      qualifiedName qName: String?) {
        if elementName == "name" {
            author?.name = characters
        }
        characters = ""
    }

    func parserDidEndDocument(_ parser: XMLParser) {
        print(author?.name)
        print(author?.type)
    }
}
```

我们已经仔细查看了直接的 XML，但是对于基于 XML 的内容，稍微少一些冗长的呢？

### 属性列表

属性列表或 plist 文件是一种基于 XML 的格式，传统上用于应用程序中的数据序列化和反序列化。让我们看看如何从文件系统中读取 plist 文件：

```
// Our .plist file on the filesystem
let plistURL = URL(...)!
guard let plistData = Data(contentsOf: plistURL) else { return }

let object = PropertyListDecoder().decode(Author.self, from: plistData)
```

这看起来与我们的 JSON 解码示例非常相似。请注意 `PropertyListDecoder` 的使用，它在功能上类似于 `JSONDecoder`，但处理的是 plist 数据而不是 JSON 数据。

写入 plist 文件与 JSON 示例类似：

```
let author = Author(name: "Mike")
let data = try? PropertyListEncoder().encode(author)
data?.write(to: ...) // save the plist file to the device
```

这将以 XML 格式输出 plist 文件。对于导出属性列表文件，还有其他可用的选项。实际上，有三种常见类型可以导出：XML、二进制或 Open Step。

这里是如何导出为二进制 plist 文件的示例：

```
let author = Author(name: "Mike")

let encoder = PropertyListEncoder()
encoder.outputFormat = .binary // set the output type to binary data
let data = try? encoder.encode(author)

data?.write(to: ...) // save the plist file to the device
```

## iOS 笔记

尽管 `Codable` 看起来像是魔法，但在编译器层面确实发生了一些非常有趣的事情，可以选择覆盖。例如，要在 JSON 中使用与 Swift 定义的 `Author` 结构中声明的名称不同的属性名称，以下代码可以起作用：

```
struct Author: Codable {
    let name: String

    private enum CodingKeys: String, CodingKey {
        case name = "something"
    }
}

let author = Author(name: "Mike")
let rawData = try? JSONEncoder().encode(author)
```

此示例将 JSON 输出中的 `name` 属性更改为 `something`。运行此代码将输出以下 JSON：

```
{ "something" : "Mike" }
```

此外，可以为完全手动的序列化和反序列化提供自己单独的 `Encodable` 和 `Decodable` 方法 `encode(to:)` 和 `init(from:)` 的实现。

# 我们学到了什么

Android 和 iOS 在序列化和反序列化诸如 XML 和 JSON 等数据方面有着非常相似的方法。同样，还有一条类似的路径，可以将 Java、Kotlin 或 Swift 对象序列化为本地平台格式。尽管语言和框架存在差异，但这是两个平台都让跨平台操作变得更加简单的领域。

在接下来的章节中，我们将解释如何扩展基础框架对象（以及其他对象），以添加一些功能。让我们来看看！
