# 第十一章：用户偏好设置

允许个性化应用是帮助用户体验的绝佳方式，并为用户提供了一种根据需求定制应用程序的途径。Android 和 iOS 提供了一套框架以及一套模式，以实现此目标。当然，对于更复杂的场景，开发者可能需要使用笨重和繁琐的技术。然而，大多数开发者可以通过简单且开箱即用的方法来读取和写入用户偏好设置。

# 任务

在本章中，你将学习以下内容：

1.  写入用户偏好设置。

1.  读取用户偏好设置。

1.  在多用户应用中处理用户偏好设置。

# Android

在 Android 中，如果你喜欢自己实现，也可以使用文件系统或数据库来存储用户偏好设置，但 Android 提供了开箱即用的 `SharedPreferences` API。虽然通常鼓励使用此 API 以保持一致性，但并不严格要求，也不一定适用于所有场景，如果你发现其他方法更适合你的需求，可以随意选择。

来自 [Android 开发者文档](https://oreil.ly/Bw8Eq)：

> 如果不需要存储大量数据且不需要结构化数据，应使用 SharedPreferences。SharedPreferences API 允许读取和写入原始数据类型的持久化键值对：布尔值、浮点数、整数、长整数和字符串。

默认情况下，`SharedPreferences` 并不安全——值存储在应用程序文件目录中的 XML 文件中。框架提供的 `KeyStore` API 提供了一些安全性，但在较旧的操作系统上可能存在问题。有第三方库声称提供与 `SharedPreferences` 类似的 API，并具有一定级别的安全性。

尽管有其局限性，`SharedPreferences` API 简单易用，并提供了内置的后台线程处理，所以不妨一试！

## 写入用户偏好设置

要写入键值对，我们首先需要一个 `SharedPreferences` 对象实例——Android 通过从任何 `Context` 实例调用 `getSharedPreferences(String fileName, Context.MODE_PRIVATE);` 方法提供了一个预配置的实例。或者，`Activity` 实例具有一个 `getPreferences` 方法，返回默认的偏好设置文件，并允许省略第一个参数（`fileName`）。

从那里，你将需要一个 `Editor` 实例，可以通过调用 `edit` 方法从 `SharedPreferences` 实例中获取：

在此时，你可以使用诸如 `putBoolean(String key, boolean value)` 和 `putString(String key, String value)` 等方法放置键值对。完成后，你可以在 `Editor` 实例上调用 `commit` 方法以同步保存更改，或者调用 `apply` 方法以异步保存它们：

如前所述，`SharedPreferences` 用于简单的原始值，只能接受基本数据类型，如`boolean`、`int`、`long`、`float` 和 `String`，尽管它也可以管理 `Set<String>`。

## 读取用户偏好

从`SharedPreferences`中读取用户偏好比写入更容易——你不需要`Editor`实例或者担心线程安全，因为数据的副本保存在内存中（这也使得它非常快）。

要读取之前示例中保存的布尔值，只需这样简单：

就是这样！

## 在多用户应用程序中处理用户偏好

所以这变得有点棘手。技术上讲，`SharedPreferences`是为整个应用程序而设计的——一个文件，被任何使用该应用程序的人*共享*。然而，为不同用户使用不同的`SharedPreferences`文件已经成为一种相对普遍的做法——你只需给获取器提供唯一的文件名即可。如何确定这个文件名完全取决于你，但你可能只是获取他们用户 ID 的`sha`：

# iOS

iOS 中有多种存储用户数据的方式：用户偏好，文件系统，Core Data 或 Keychain。通常，应用程序需要持久化特定用户唯一但不一定私密或安全的信息。像这样的数据最适合存储在用户偏好中——非常适合存储用户的语言环境，UI 样式偏好或者关于显示数据的测量单位选择。

幸运的是，iOS（和 macOS）提供了一个经过测试的方法来在应用级别存储这些数据：`UserDefaults`。让我们深入了解一下！

## 写入用户偏好

在我们能够读取数据之前，首先学会存储数据是很重要的。通过`UserDefaults`持久化用户偏好非常简单。下面是一个存储基本字符串值的简单示例：

```
let defaults = UserDefaults.standard
defaults.set("some string value", forKey: "someKey")
```

首先，我们获取共享的`UserDefaults`实例，然后我们将一个字符串“some string value”与名为`someKey`的键配对。这个键是我们查找数据的关键，如本章后面所示。

### 发生在幕后的事情

每个 iOS 应用程序的应用文件夹中都有一个`Library`文件夹，其中包含一个`Preferences`文件夹。在幕后，iOS 在应用程序向`UserDefaults.standard`写入值时创建或更新属性列表文件。存储的各个类型都符合`NSCoding`协议，允许它们被序列化和反序列化。也可以使自定义类符合这个协议——稍后详细介绍！

###### 警告

这个属性列表文件由`UserDefaults`更新和管理。应该将其视为实现细节，由底层子系统处理，因为可能会在未来的 iOS 版本中发生变化。

### 数据类型

`UserDefaults`能够存储多种类型的数据，包括布尔值，数字，字符串，URL，字典，数组和自定义对象。这里有一个更复杂的示例，展示了更广泛的用法：

```
let defaults = UserDefaults.standard

// Boolean
defaults.set(true, forKey: "nightMode")

// Number
defaults.set(2.0, forKey: "playbackSpeed")

// String
defaults.set("en-US", forKey: "locale")

// URL
let url = URL(string: "https://www.example.com/api")
defaults.set(url, forKey: "apiURL")
```

Swift 提供了许多方便的方法，可以将特定数据类型值传递给`UserDefault`。每个值都映射到一个`String`键。大多数情况下，使用标准的 Swift 对象就足够了。但是当您想要持久化自定义类时呢？

幸运的是，有一个解决方案：`NSCoding`。

### NSCoding 遵循

希望在`UserDefault`中持久化的对象需要符合`NSCoding`协议。该协议包含两个方法：`init(coder:)`和`encode(with:)`。这两个方法在存储对象的编码和解码功能中起着重要作用。让我们看一个简单的示例。

```
@objc(SomeObject)
class SomeObject: NSObject, NSCoding {
    let someProperty: String

    init(someProperty: String) {
        self.someProperty = someProperty
    }

    // MARK: NSCoding protocol conformance

    required convenience init?(coder aDecoder: NSCoder) {
        guard let someProperty = aDecoder.decodeObject(forKey:
        "someProperty") as? String else {
            return nil
        }
        self.init(someProperty: someProperty)
    }

    func encode(with aCoder: NSCoder) {
        aCoder.encode(someProperty, forKey: "someProperty")
    }
}

let someObject = SomeObject(someProperty: "some value")

let defaults = UserDefaults.standard
defaults.set(someObject, forKey: "myObject")
```

首先，我们为一个类声明了显式的`@objc`名称。这是因为类名在解档对象时非常重要。我们还声明该类符合`NSCoding`协议。接下来，我们有一个名为`someProperty`的字符串属性，在类的初始化器中设置该属性。在类的更深层部分，在`encode(with:)`方法中，我们使用传入该方法的给定`NSCoder`来为键`someProperty`编码`someProperty`的值。这是在我们在`UserDefaults`上调用`set(value:forKey:)`时在幕后调用的方法。

稍后，每当我们需要解码该对象时，`UserDefaults`会通过调用`init?(coder:)`初始化器来实例化该对象。在该方法内部，我们尝试设置每个已编码属性，首先解码给定键的属性，然后将其传递给我们的默认对象初始化器。因为我们手动决定哪些属性进行编码和解码，所以可以排除某些根据不同数据初始化或由其他对象设置的属性。

###### 警告

这是一个“字符串类型”的示例，如果键名拼写错误或在应用程序版本之间更改，则很容易出错。有方法可以绕过这些问题，例如使用枚举和一些内置的`NSKeyedUnarchiver`功能，如`NSKeyedUnarchiver.setClass(SomeObject.self, forClassName: "SomeObject")`。

### 使用 Codable 而不是 NSCoding

对于需要在`UserDefaults`中持久保存的自定义对象，可以跳过`NSCoding`遵循，只需使用`Codable`遵循来对对象进行 JSON 编码和解码。使用`Codable`而不是`NSCoding`的一个好处是，您可以跳过整个 Objective-C 运行时。以下是一个示例：

```
struct SomeObject: Codable {
    let someProperty: String
}

let someObject = SomeObject(someProperty: "some value")

// Store the object
let defaults = UserDefaults.standard
if let json = try? JSONEncoder().encode(someObject) {
    defaults.set(json, forKey: "myObject")
}
```

对于符合`Codable`协议的给定类型，我们可以使用`JSONEncoder`对其进行编码，然后将生成的`Data`直接设置为一个键（在我们的示例中是`myObject`）。

解码 JSON 很简单。可以通过以下代码实现：

```
// Read the value
let json = defaults.value(forKey: "myObject") as! Data
let someObject = try? JSONDecoder().decode(SomeObject.self, from: json)
```

首先，我们从`UserDefaults`中获取对象作为`Data`（强制解包以保持示例简单）。然后，我们使用`JSONDecoder`对其进行解码，并显式声明其类型为`SomeObject`。

### 删除键

可能您创建了一个键，然后决定在应用程序的将来版本中不再需要该键。在`UserDefaults`中删除键只是一个如下的调用：

```
let defaults = UserDefaults.standard
defaults.removeObject(forKey: "someKey")
```

现在我们已经写入（和删除）了数据，让我们来看看如何读取它，以便我们可以在我们的应用程序中使用它！

## 读取用户偏好设置

可以通过以下方式调用从`UserDefaults`读取数据：

```
let defaults = UserDefaults.standard
let someValue = defaults.value(forKey: "someKey")
```

这将创建一个名为`someValue`的对象，用于保存我们的数据。不幸的是，`UserDefaults`并不明确知道我们的数据被解码为什么类型，因此默认为`Any?`。但是，可以通过稍微更改我们调用以获取数据的方法来改变这种情况，使用一些专门为一组常见类型构建的方法。下面的代码展示了其中一些方法的示例：

```
let defaults = UserDefaults.standard

// Boolean
let nightMode = defaults.bool(forKey: "nightMode") // true

// Number
let playbackSpeed = defaults.double(forKey: "playbackSpeed") // 2.0

// String
let locale = defaults.string(forKey: "locale") // "en-US"

// URL
let apiURL = defaults.url(forKey: "apiURL") // https://www.example.com/api
```

遍历此处调用的方法列表，我们可以看到`bool(forKey:)`返回`Boolean`类型，`double(forKey:)`返回`Double`，`string(forKey:)`返回`String`，而`url(forKey:)`返回一个 URL 实例。还有其他可用的类型，比如其他数字类型如`Int`和`Float`。查看[Apple 开发者文档](https://oreil.ly/uIDX_)以获取有关`UserDefaults`解码可用类型的更多信息。

话虽如此，有一种类型显然缺失：我们在本章前面声明的`SomeObject`类类型！为了返回一个`SomeObject`，我们需要像这样使用`object(forKey:)`方法：

```
// SomeObject NSCoding example
let someObject = defaults.object(forKey: "someObject") as? SomeObject
```

注意，我们显式地将由`UserDefaults`返回的对象转换为`SomeObject`。如果没有显式类型要求，而`Any?`足够的话，可以跳过此步骤。

到目前为止，我们只讨论了如何为单个用户存储数据。让我们讨论一下在您的应用程序中使用多个用户以及我们如何在`UserDefaults`中管理它们。

## 在多用户应用程序中处理用户偏好设置

不幸的是，尽管 macOS 提供了一些诱人的接近功能，iOS 并未提供此类开箱即用的功能。话虽如此，可以通过将用户偏好设置存储在单独的文件中，并在必要时恢复来解决这个问题。以下是如何完成这一点的示例：

```
let defaults = UserDefaults.standard

// Get a dictionary representation of the current UserDefaults
let dictionary = defaults.dictionaryRepresentation()

// Store the dictionary to disk
let oldData = try! NSKeyedArchiver.archivedData(
    withRootObject: dictionary, requiringSecureCoding: true)
try! oldData.write(to: URL(fileURLWithPath: "user1.plist"))

// Remove all the data
dictionary.keys.forEach { key in
    defaults.removeObject(forKey: key)
}

// Get the new user preferences
let newData = try! Data(contentsOf: URL(fileURLWithPath: "user2.plist"))
if let newDictionary =
  try? NSKeyedUnarchiver.unarchiveTopLevelObjectWithData(newData) as? [String: Any] {
    // Update UserDefaults with the new data
    newDictionary.forEach { (keyValue) in
        let (key, value) = keyValue
        defaults.set(value, forKey: key)
    }
}
```

让我们一起看看正在发生什么。

首先，我们获得我们的`UserDefaults`数据的`Dictionary`表示。这用于备份我们现有用户的数据。我们称这个字典，并使用`NSKeyedArchiver`将该字典转换为一个`Data`实例，然后可以写入到应用程序的共享区域，最有可能是在应用程序的*Library/Preferences*文件夹内，但这留给读者去练习。在我们的示例中，我们将其存储到一个文件路径*user1.plist*中。

在代码中进一步进行时，我们遍历字典中的每个键，然后在`UserDefaults`上调用`removeObject(forKey:)`来清除所有这些数据。

最后，是时候开始使用新用户的数据了；让我们称呼这位用户为“用户 2”。用户 2 在*user2.plist*中有一个偏好文件，因此我们将该文件的`Data`表示形式传递给`NSKeyedUnarchiver`，并解码为一个字典对象`[String: Any]`，这与最初写入的类型相同。要将数据添加到`UserDefaults`中，我们遍历新字典中的每个键，并直接将其手动设置到`UserDefaults`中。

这并不是最干净简单的过程，但它有效。iOS 的未来版本可能会支持多个用户账户。目前，这个解决方案可行。可能需要一些时间来写入大量用户偏好列表。幸运的是，`UserDefaults`是线程安全的，所以所有这些工作可以轻松地在后台线程中完成。

# 我们学到了什么

从本章中可以得出几个要点：

+   使用操作系统提供的开箱即用技术是开始存储用户偏好的好方法。实际上，可以仅使用这种方法构建强大的系统和流程，应用程序可以遵循这种方法。

+   Android 和 iOS 都有类似的方法来存储和读取用户偏好。Android 使用`SharedPreferences`，iOS 使用`UserDefaults`。它们都提供了一个键值存储，用于在会话之间保存数据。

我们主要讨论了关于用户偏好的设备内数据存储。还有其他标准格式，如 XML 和 JSON，在这两个平台上都有很好的支持。让我们在下一章中学习更高级的数据序列化和传输。
