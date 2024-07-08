# 第十三章：扩展

有时候，Android 和 iOS 提供的功能并不足够，有时候，第三方库或内部对象提供的功能也不够。当然，您可以创建子类，但这并不总是可行的。

幸运的是，这两个平台都有增强现有对象的方法。

# 任务

在本章中，您将学习如何为现有的 API 添加功能。

# Android

虽然本书的大部分内容都以 Java 作为 Android 开发的语言，默认使用 Java，而且大部分 Android 开发确实都是使用 Java，但我们现在必须提到，在 Android 框架中，*只有 Kotlin 能够支持扩展*。这是什么意思？我们来定义一下“扩展”这个词的含义：“修改或添加类的功能”。然而，在带有 Kotlin 扩展的项目中，Java 类也可以利用扩展功能！

## 为现有的 API 添加功能

在 Java 中，如果导入一个名为 `CalendarPicker` 的第三方库，您基本上只能使用该库提供的公共 API。当然，您可以修改源代码，但此时您已经创建了自己的 `CalendarPicker`，它不会是与原始类相同的实例。您还可以创建 `CalendarPicker` 的子类，但同样的注意事项适用。不可能添加或更改现有的 `CalendarPicker` API 的功能。

在 Kotlin 中，这不再成立。不仅可以向 `CalendarPicker` 添加一个方法，使得所有 `CalendarPicker` 实例都可以访问，而且实际上可以向标准库类如 `LinkedList`，甚至带有泛型类型如 `HashSet<String>` 的类添加方法。事实上，Kotlin 中甚至有一个 `Any` 类，可以扩展以向所有类实例添加功能。Kotlin 扩展的经典示例正是这样做的：

```
fun Any?.toString(): String {
    if (this == null) {
      return "null"
    }
    return toString()
}
```

这将 Kotlin 的 `?` 空安全操作符添加到通用 `Object.toString` 方法的*主体*中。通过添加前述代码，您可以安全地调用任何对象的 `toString` 方法；如果对象为 `null`，则不会抛出 `NullPointerException`，而是会返回字符串 `"null"`。相当强大！

您可以将扩展添加到任何您喜欢的包或目录结构中（只要它是 Kotlin 文件，在*src*目录中）。例如，在项目中右键单击任何包，选择“New > Kotlin File/Class”，然后随便取个名字——比如现在叫“extensions”（Android Studio 将附加“.kt”扩展名并显示不同的图标，表示它是一个 Kotlin 文件）。

您可以向此文件或任何文件添加多个扩展函数，并且所有这些函数都可以从项目中的任何 Kotlin 类访问。假设您添加了以下的 `String` 辅助函数：

```
fun String.from(start:String): String {
  val index = indexOf(start) + start.length
  return substring(index)
}
```

然后在某个 Kotlin 类中，您可以调用类似这样的内容并获得预期的结果：

```
val original = "Hello world!"
val modified = original.from("Hello")
Log.d("MyTag", modified)
```

查看[开发者文档](https://oreil.ly/oGVHb)以获取有关 Kotlin 扩展的更多信息。

# iOS

Swift 的一个核心语言特性是通过扩展为类型添加新功能。事实上，这也是 Objective-C 语言运行时的一个重要部分，尽管名称为“category”而不是“extension”。这种能力在 Swift 中得到了很好的实现，实际上在实践中非常普遍。

## 添加功能到现有 API

这里有一个简单的例子：

```
class ExtendableObect {
    // ...
}

extension ExtendableObect {
    func helloTacos() {
        print("Hello, tacos!")
    }
}
```

首先，我们定义一个名为`ExtendableObject`的类。在我们的例子中，该类没有主体，因为它对示例并不重要。真正的示例核心在于接下来我们定义扩展的部分。

我们在`ExtendableObject`上使用`extension ExtendableObject { ... }`进行声明。在该定义中，我们声明和定义一个名为`helloTacos()`的新方法，方便地将字符串`Hello, tacos!`打印到控制台。我们可以通过实例化对象并像调用对象的任何其他方法一样调用`helloTacos()`来使用我们新定义的方法，如下所示：

```
let object = ExtendableObject()
object.helloTacos()
```

###### 注意

有一件非常重要的事情需要注意。您可以向现有对象类型添加新方法和函数，但无法添加新的存储属性，除非导入 Objective-C 运行时并使用关联值。一般情况下不建议这样做，除非是非常罕见的情况。

### 用于代码组织的扩展

由于扩展在 Swift 语言中是如此重要的一部分，因此在 Swift 项目中，通常会使用扩展来组织代码。通常，您会看到在类的底部将协议的实现放在扩展中。

这里有一个例子：

```
protocol AwesomeProtocol {
    func beAwesome()
}

class ExtendableObect { /* ... */ }
extension ExtendableObect: AwesomeProtocol {
    func beAwesome() {
        print("You are awesome!")
    }
}
```

首先，我们声明一个名为`AwesomeProtocol`的新协议，其中包含必需的方法`beAwesome()`。接下来，我们声明`ExtendableObject`。可以直接通过将其作为`ExtendableObject`定义的一部分来实现该协议：

```
class ExtendableObject: AwesomeProtocol { /*  ... */}
```

然而，由于 Swift 的协议导向性质，随着对象必须实现的协议数量增加以及应用程序复杂性的增加，这种方法可能变得难以控制和维护。我们之前的例子中，通过分离扩展，使得我们可以将协议符合性放在一起，便于将来查找和维护。

这完全是一种主观的代码组织方式，但在 Swift 社区中算是一种事实上的标准。随意选择使用或不使用！

谈到 Swift 中的协议——也可以对其进行扩展！让我们来看看。

### 扩展协议

在 Swift 中，扩展类和结构与扩展协议略有不同。语法类似，但背后的机制不同。以下是如何在 Swift 中扩展协议的方法：

```
protocol ExtandableProtocol {
    func doSomething()
}
extension ExtendableProtocol {
    func printSomething() {
        print("Something :)")
    }
}

class SomeObject: ExtendableProtocol {
    func doSomething() {
		// ...
    }
}
```

###### 注意

我们正在扩展的协议不是在 Swift 中编写的 Objective-C 协议——它是一个纯粹的 Swift 对象。这很重要，因为只有纯粹的 Swift 协议可以进行扩展。也不可能扩展以`@objc`为前缀的方法。

本例创建了一个名为`ExtendableProtocol`的协议，其中声明了一个`doSomething()`方法。接下来，协议的扩展添加了一个名为`printSomething`的新方法，并将其声明和定义为扩展的一部分。最后，定义了一个类`SomeObject`来实现`ExtendableProtocol`协议。

调用`SomeObject`的实例可能看起来像这样：

```
let object = SomeObject()
object.doSomething()
object.printSomething()
```

相当简单，与类和结构扩展并没有太大的区别，对吧？

协议扩展的强大之处在于当您需要跨不同基类的对象共享通用功能时，它变得显而易见。看看这个例子：

```
protocol Typeable { /* ... */ }
extension Typeable {
    func printType() {
        let objectType = String(describing: type(of: self))
        print("This object is a type of \(objectType)")
    }
}

class BaseClassA { }
class BaseClassB { }
class BaseClassC { }

class TacoTruck: BaseClassA, Typeable { }
class Dog: BaseClassB, Typeable { }
class Cat: BaseClassC, Typeable { }

let tacoTruck = TacoTruck()
let dog = Dog()
let cat = Cat()

tacoTruck.printType()
dog.printType()
cat.printType()
```

首先，我们声明一个协议`Typeable`，并在该协议上声明一个扩展，输出对象的类型。接下来，声明三个完全不同且没有任何关联的基类。然后，声明三个更多的子对象，每个都继承自不同的父类，但都实现了之前的协议`Typeable`。然后，我们实例化每个类。最后，我们调用同一个方法`printType`，这个方法只在一个地方声明，但对于每个单独的对象都能完全相同地工作。

协议扩展是 Swift 中的一个强大补充，它使代码更清晰、更可维护、更具架构性。

# 我们学到了什么

在 Java 中不容易添加功能，但 Kotlin 和 Swift 都提供了扩展现有对象的内置机制。这两种方法都有局限性，但两种语言和平台都能提供的功能确实有明确的用例。

在我们关于任务的最后一章中，我们的注意力转向了不是向建设方向，而是通过测试来维护我们已经构建的内容。两个平台都围绕测试提供了优秀的工具。让我们深入了解一下可用的内容！
