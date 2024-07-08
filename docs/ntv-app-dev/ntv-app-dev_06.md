# 第五章：消息传递

消息传递在计算机科学中是一个非常广泛且有时具有争议性的主题，有许多已经出现或存在的模式和系统，如“发布/订阅”、“事件分派器”、“回调”、“观察者”、“消息队列”等。事实上，这些通常非常相似，您很难定义它们之间的实际差异。但无论如何，在任何应用程序中，这都是一个关键的功能，在移动应用程序开发中有一些共识策略，我们将在这里解释。

# 任务

在本章中，您将学习：

1.  使用回调以响应操作。

1.  将消息分派给任何感兴趣的订阅者。

1.  监听并响应系统内分派的消息。

# Android

在 Android 中，您通常使用回调进行直接的消息传递，并使用统计可用和线程安全的`LocalBroadcastManager`分派事件。请注意，`LocalBroadcastManager`发送`Intent`实例以供`BroadcastReceiver`实例接收——这与系统范围消息使用的机制相同，许多是由其他应用程序或操作系统组件提供的；然而，`LocalBroadcastManager`只通知您的应用程序中的`BroadcastReceiver`，这既是一件好事（安全），也稍微有些局限性——如果您希望在应用程序之间通信或与底层框架通信，则必须超越`LocalBroadcastManager`提供的简洁和简单 API。

## 使用回调以响应操作

在 Java 和 Android 中，传递消息的最直接方式是将回调与操作一起发送。回调是专门创建的对象实例，仅用于接收结果并对其进行操作，通常是功能接口的简单匿名实例。这里有一个例子：

某人可能会使用`Callback`接口的完整实例、lambda 表达式或方法引用来调用它：

Java 然后 Kotlin 中的 lambda 表达式

```
Callbacks.requestData(data -> Log.d("MyApp", "result received: " + data));

```

```
Callbacks.requestData { data-> Log.d("MyApp", "result received: $data") }

```

Java 然后 Kotlin 中的方法引用

```
private void handleResult(Object data) {
  Log.d("MyApp", "result received: " + data);
}
...
Callbacks.requestData(this::handleResult);

```

```
private fun handleResult(data: Any?) {
  Log.d("MyApp", "result received: $data")
}
...
Callbacks.requestData(::handleResult)

```

仅在 Java 中使用实例

```
Callbacks.requestData(new Callback(Object data) {
  Log.d("MyApp", "result received: " + data);
});

```

在 Kotlin 中没有直接的等价物，因为函数参数被定义为 lambda 而不是接口。请改用 lambda 的语法，并参阅[Kotlin 文档](https://oreil.ly/lr8Ja)了解有关高阶函数的信息。

正如您所见，这是一条非常直接的路径，并允许您作为开发人员定义您的程序对操作结果的反应，具有极大的灵活性。

这里有一个简单但功能性的例子。这次我们同时使用回调来处理成功的操作和失败的结果：

您可以像这样使用它：

在其他地方调用它，如下所示：

更实际的例子可能是这样的：你有许多可能抛出异常的操作，比如需要身份验证并返回必须成功解析的 JSON 响应的网络请求，然后是本地磁盘或数据库操作来保存结果。你可能有一个处理所有这些逻辑的`Activity`，并希望有通用的失败处理逻辑（例如，向用户显示一个包含描述失败的友好文本的`Snackbar`，但不中断用户体验或使应用崩溃）。类似这样的类可能会很有帮助：

你可以在类似这样的控制器中使用这些实用程序：

这些是使用回调的基础！这是一个非常直接的模式，你可能已经见过或者使用过类似的东西。

## 向任何感兴趣的订阅者发送消息

一个不太熟悉的 API 是 Android 特有的，不在框架外部可用的`LocalBroadcastManager`。让我们直接开始吧。

`LocalBroadcastManager`是一个单例——整个应用程序中只使用和管理一个实例。你可以通过将任何`Context`实例传递给静态的`getInstance`方法来访问单例实例：`LocalBroadcastManager lbm = LocalBroadcastManager.getInstance(context);`。你永远不需要担心构造函数或配置。此外，它在开箱即用时是线程安全的！

这个类作为单例的真正好处在于，你能够在两个互不了解的对象之间发送和接收消息。例如，你可以从管理应用程序中多个不同`RecyclerView`的`Adapter`类广播消息，而不知道当接收到该消息时会发生什么（如果有的话）。在一个列表视图中，你可能希望根据特定类型的消息更新列表；在另一个列表中，你可能希望有非常不同的反应——甚至在相同的消息中`finish`掉`Activity`。`Adapter`不需要知道`RecyclerView`的实例或包含它们的各种`Activity`实例，反之亦然——`LocalBroadcastManager`发出警报，任何感兴趣的方可以根据需要做出反应。

`LocalBroadcastManager`实例有两个我们经常使用的方法：它们是`sendBroadcast`和`registerReceiver`。正如你可能已经猜到的那样，`sendBroadcast`发送消息并通知任何已注册接收该类型消息的其他类。第三个方法`unregisterReceiver`也很重要，这样我们就可以在应用程序中删除侦听器时停止侦听消息（从而防止内存泄漏！）。

这样的消息由`Intent`类表示，与用于启动新`Activities`和描述系统范围派发的相同类。`Intent`实例有一个“action”属性，在构造函数中可以通过传递一个`String`来设置；甚至可以使用`addAction`方法后续添加更多操作。

`LocalBroadastManager`使用`sendBroadcast`方法来发送`Intent`：

一旦调用这个方法，之前在同一个全局`LocalBroadcastManager`实例上注册过这个动作的任何`BroadcastReceiver`都会调用它们的`onReceive`方法，并且第二个参数将是您发送的`Intent`实例。请注意，只有在`IntentFilter`包含与`Intent`实例的动作匹配的动作的`BroadcastReceiver`实例才会被触发。因此，在前面的例子中，如果您创建一个使用不同动作构造的`Intent`，如`Intent intent = new Intent("user-login");`，则本例中的`BroadcastReceiver`将不会触发，因为动作不匹配。

## 监听和响应系统内分发的消息

让我们看看如何注册以便在广播时收到通知。

首先，我们需要一个`BroadcastReceiver`实例，这是一个扩展基础`BroadcastReceiver`类的对象实例，它将会对我们感兴趣的消息做出响应：

接下来，我们需要一个`IntentFilter`实例，它实际上只是我们感兴趣的一系列字符串动作类型的列表：

###### 注意

`IntentFilter`提供了其他功能，但最常见的是基本的`String`动作过滤器。

现在我们可以将这些传递给`registerReceiver`方法，这样每次使用“data-received”动作发送广播时，我们都会收到通知：

请注意，`LocalBroadcastReceiver`是线程安全的，因此您可以在后台线程发送广播，并在 UI 线程安全地监听。但广播本身是异步的，这意味着它不一定按照程序中语句的顺序触发。然而，有一个`broadcastSync`方法可以立即和串行地发送广播。

还非常重要的是记得注销不再使用的`BroadcastReceivers`，比如在已经`finished`的`Activity`中定义的那些。否则可能会发生内存泄漏。

在 Java 和 Android 中有许多第三方消息系统，包括 Square 的 Otto 和 GreenRobot 的 EventBus。还有非常流行的 RxJava Android 库使用的`Observable` API，但在我看来，观察者模式与传统的发布-订阅模式有足够的不同，值得单独讨论。

最后，在 Java 中编写自己的事件总线（一个完整的事件分发和接收系统）非常简单；我们已经在[大约 40 行代码](https://oreil.ly/en-O6)中编写了一个。

# iOS

在 iOS 中有几种处理消息传递的方式。与 Android 类似，其中一种常见的模式是通过回调和通知来实现。让我们来分析这些模式之间的区别，并确定何时使用其中一种成熟的模式。

## 使用回调来响应操作

Swift 和 iOS 中最常见的消息传递形式是通过闭包实现的。

### 闭包

闭包是作为对象而存在的独立函数。它们可以作为另一个对象的属性，并作为方法参数传递或存储以供将来使用。在其最基本的形式中，闭包可能看起来像这样：

```
var someClosure = { print("I'm a closure!") }
someClosure() // Executes our closure, which prints out a message to the console
```

本质上，闭包是可以随时调用的弹簧加载的代码片段。它们的另一个名称是“匿名函数”，它们非常有用！

在异步情况下，闭包作为回调函数的使用是最强大且最合适的，比如等待网络调用完成时。以下是一个假设的 API 客户端如何利用闭包与服务器通信而不会导致整个应用程序在等待服务器响应时无响应的示例：

```
class NetworkService {
	var completion: ((Bool, Error?) -> ())?

	func fetchData(for url: URL) {
		...
	}
	func onSuccess() {
		completion?(true, nil)
	}
	func onError(error: Error) {
		completion?(false, error)
	}
}

let api = NetworkService()
api.completion = { success, error in
	if success {
		print("Success!")
	} else {
		print("Uh-oh!")
	}
}
api.fetchData()
```

让我们逐步了解这段代码。

首先，`completion`属性是一个存储在类中可以稍后使用的闭包实例。需要注意的是，这是一个可为空的属性，如果我们不想在网络调用完成时执行任何操作，它可以是`nil`。

接下来，`fetchData`是一个占位方法，当我们的假设 API 接收到响应时最终调用`onSuccess`或`onError`。

现在，这是重要的部分。每当调用`onSuccess`或`onError`时，它们都会调用存储的闭包，并通过方法参数传递一些数据。闭包可以传递任何数据，但在这个例子中，传递给闭包的是一个布尔值，指示操作是否成功，以及一个可选的错误（如果发生错误）。

移动到这个对象的使用方法，我们可以看到该类被实例化为`api`，并且从之前的`completion`变量中传递了一个闭包，用于将操作成功或失败的信息打印到控制台。这个闭包在`NetworkService`类中的`onSuccess`和`onError`中被引用和调用。

最后，我们调用从我们创建的对象中调用`fetchData`，以便启动网络调用并从 API 接收响应。

### 逃逸闭包和非逃逸闭包

闭包理解中较难的一个方面是内存管理。每当创建一个闭包时，其中包含的变量和实例被“闭合”，并且创建这些对象的强引用。其结果是，即使这些对象超出作用域，它们仍然存在以便闭包使用。以下是一个例子：

```
class Incrementor {
    var count = 0

    func increment() {
        count += 1
        print(count)
    }
}

let incrementor = Incrementor()
let closure = {
    incrementor.increment() // 1
    incrementor.increment() // 2
}
closure() // Prints "1\n2"
```

首先，我们创建一个对象，每次调用`increment`时自增一次。我们实例化了这种类型的对象。然后，我们将该对象传递给一个闭包，以便对它进行强引用。这允许该对象在通常会超出作用域时继续存在。这样做的最终结果是，当调用`closure()`时，对象仍然存在，并且能够在每次调用时将我们的计数增加一次（在本例中，两次）。

这很重要，因为它涉及到所创建对象的内存管理。Swift 及其前身 Objective-C 容易受到称为保留周期的错误影响。当一个对象因其他对象持有其引用而无法退出作用域并从内存中清除时，就会发生这种情况。由于闭包对其中包含的对象创建了强引用，它们实质上是在“计分板”上为拥有该对象的其他对象增加了一个记号。这告诉编译器不要清除它，以便其他对象可以使用它。在前面的例子中，对于闭包来说，这是有帮助的，因为`incrementor`保持在作用域内，并且可以在稍后调用闭包时调用。

当闭包作为 Swift 的参数传递时，如果闭包不超出被调用的函数的生存期，那么该闭包被称为`nonescaping`闭包。默认情况下，对于每个作为参数传递的闭包，都会隐式声明为这种类型的闭包。一个非逃逸闭包的示例如下：

```
class Incrementor {
    var count = 0

    func increment(with closure: () -> ()) {
        count += 1
        closure()
    }
}

let incrementor = Incrementor()
let printCount = {
    print(incrementor.count)
}

incrementor.increment(with: printCount) // 1
incrementor.increment(with: printCount) // 2
```

我们已经修改了`Incrementor`类的`increment()`方法，现在它接受一个闭包作为方法参数，因此方法签名现在是`increment(with:)`。当`count`变量增加一个单位时，立即调用此闭包。在我们的示例中，我们传递了一个闭包，它只是直接从`incrementor`对象中调用`print`打印`count`变量。我们直接引用了`incrementor`，这意味着编译器创建了一个强引用，并且`incrementor`在我们的应用程序完成之前不会从内存中释放。

在我们的例子中，传递给`increment(with:)`的参数`closure`是一个非逃逸闭包。它永远不会超出被调用的函数的生存期。让我们看看逃逸闭包的示例是什么样子：

```
class Incrementor {
    var count = 0
    var printingMethod: (() -> ())?

    func increment(with closure: () -> ()) {
        if printingMethod == nil {
            printingMethod = closure
        }
        count += 1
        printingMethod?()
    }
}

let incrementor = Incrementor()
let printCount = {
    print(incrementor.count)
}

incrementor.increment(with: printCount)
incrementor.increment(with: printCount)
```

正如您所看到的，我们添加了一个新的`printingMethod`来存储我们传入的闭包。然后，在`increment(with:)`内部，我们将传入的闭包赋值给`printingMethod`变量。如果您在代码编辑器中尝试此操作，当您尝试为`printingMethod`分配一个值时，您将收到一个编译器错误，因为您没有指示此闭包是一个逃逸闭包。这是一个简单的修复方法。只需在传递给`increment(with:)`方法的闭包中添加一个`escaping`关键字，如下所示：

```
func increment(with closure: @escaping () -> ()) {
	if printingMethod == nil {
		printingMethod = closure
	}
	count += 1
	printingMethod?()
}
```

现在完整的示例看起来是这样的：

```
class Incrementor {
    var count = 0
    var printingMethod: (() -> ())?

    func increment(with closure: @escaping () -> ()) {
        if printingMethod == nil {
            printingMethod = closure
        }
        count += 1
        printingMethod?()
    }
}

let incrementor = Incrementor()
let printCount = {
    print(incrementor.count)
}

incrementor.increment(with: printCount)
incrementor.increment(with: printCount)
```

这解决了编译器错误。但是我们创建了一个讨厌的 bug，称为保留周期。它们很容易创建，但很难调试。在我们的小例子中并不太明显，但如果你有一个大对象或者一个被多次创建的对象，你可能会很快耗尽应用程序的可用内存，或者遇到意外的副作用和不期而遇的问题。那么我们的 bug 在哪里呢？

记得我们提到过创建闭包会创建一个强引用，或者给一个对象添加一个“记分牌”，这样它就不会在所有引用都消失之前被清除出内存？在我们的例子中，我们在`printCount`闭包中创建了一个对`incrementor`的强引用。但是，当我们将其存储为`increment(with:)`方法内部的`printingMethod`时，我们创建了对闭包的强引用。对象正在存储对自身的强引用，以便该对象永远不会超出作用域并从内存中清除！

幸运的是，Swift 有一种方法可以通过捕获列表将强引用转换为所谓的弱引用。让我们在我们传递给`increment(with:)`方法的闭包内部将`incrementor`声明为弱引用，像这样：

```
let printCount = { [weak incrementor] in
    guard let incrementor = incrementor else { return }
    print(incrementor.count)
}
```

注意，我们使用了`[weak incrementor]`来指示`incrementor`应该是一个弱引用而不是一个强引用。我们还在闭包中添加了一个`guard`语句，以确保在尝试访问之前`incrementor`不为`nil`。这意味着我们不再有保留周期，因为在存储`increment(with:)`方法内部的闭包到`printingMethod`时，`Incrementor`没有存储自身的强引用；它存储的是一个弱引用版本。因此，当最后一个`incrementor`被调用时，虚拟记分牌可以归零，并且对象可以从内存中释放。

现在，闭包显然是在对象之间传递消息的更现代方法，但有时它们可能有点过度。它们也容易出现内存引用错误，正如刚刚展示的那样，并且在不仔细注意和故意处理的情况下容易出现线程错误。在 Cocoa Touch 中，人们发现它们并不是对象传递消息的唯一方式。让我们来看看代理与基于闭包的回调函数有何不同。

### 代理

自从它的创立以来，代理已经成为 Cocoa Touch 的一部分。它们提供了逻辑简单性，但通常比闭包更冗长。这是我们之前使用委托而不是闭包编写的相同`NetworkService`类：

```
protocol NetworkServiceDelegate: class {
	func fetchDidComplete(success: Bool, with error: Error?)
}

class NetworkService {
	weak var delegate: NetworkServiceDelegate?

	func fetchData(for url: URL) {
		...
	}
	func onSuccess() {
		delegate?.fetchDidComplete(success: true, with: nil)
	}
	func onError(error: Error) {
		delegate?.fetchDidComplete(success: false, with: error)
	}
}

class APIClient {
	init() {
		let api = NetworkService()
		api.delegate = self
		api.fetchData()
	}
}
extension APIClient: NetworkServiceDelegate {
	func fetchDidComplete(success: Bool, with error: Error?) {
		if success {
			print("Success!")
		} else {
			print("Uh-oh!")
		}
	}
}
```

在这段代码中，我们可以看到一个名为`NetworkServiceDelegate`的协议被定义了。我们的回调方法的方法签名与我们的`completion`闭包类似，带有`Bool`和`Error`参数，但是使用的是命名而不是匿名。

接下来，我们在`NetworkService`中提供了一个地方来存储我们的委托，这个属性被适当命名为`delegate`。这个属性被标记为`weak`，以防止委托存储其父对象的引用循环；这将防止任一对象被从内存中释放，并且很容易被忽略。

你可以看到很多代码是相同的，但是不是通过在`NetworkService`中调用闭包，而是直接通过`delegate?.fetchDidComplete(success:with:)`调用委托。

要调用我们的 API，我们需要创建一个对象，实例化`NetworkService`，将自身设置为代理，并调用`fetchData()`从网络获取数据。在这个示例中，这就是`APIClient`类的目的。

最后，在示例开始时我们`NetworkServiceDelegate`协议所需的代理方法实际实现写成了`APIClient`类的扩展。每当我们调用`fetchData`方法时，将调用`fetchDidComplete(success:with:)`方法，并将成功或失败的消息打印到控制台。

正如您很可能看到的那样，这比闭包稍微啰嗦一些，但调用非常直接。现在，它可能很快变得难以控制，但考虑到这种回调模式在 Cocoa Touch 中的深入根植，您最终将不可避免地遇到它。一个好的经验法则是，在任何需要异步性或有一个闭包可能会被调用的路径时使用闭包，并在您（或多个对象）想要同步调用有关是否应完成操作或调用状态的情况下使用委托。

最终，闭包和委托可能不符合必要的要求，而需要额外且不必要的复杂性。当需要多个闭包并行执行或多个委托接收相同消息时，很可能是考虑`NotificationCenter`和相关内容的时候了。

## 分发消息给任何感兴趣的订阅者

通知是向应用程序中的其他部分发送消息的一种内置和方便的方式，以实现组件之间的解耦。为了接收正在发送的通知，一个对象会“监听”一个共享对象，以便发布具有特定名称的消息。该消息可以附带数据，并传播到订阅的任何对象中。Cocoa Touch 将这些称为“通知”，在其他语言和框架中，这种模式有时被称为“发布-订阅”或“观察者”。在 Android 上的等效物是`LocalBroadcastManager`。

###### 注意

当我们说“通知”时，我们指的是一个`Notification`对象，而不是推送通知。并且，为了使事情更加复杂，对于 Android 来说，“通知”意味着系统托盘中的事件。

让我们从发布通知开始。通知名称本质上是字符串，因此您可以像这样使用原始字符串发布通知：

```
NotificationCenter.default.post(name: Notification.Name("didFinish"), object: nil)
```

在这段代码块中，共享的`NotificationCenter.default`对象用于发布名称为`didFinish`的通知。现在，这是一个很好的入门方式，但不适合包含在发送到 App Store 的成品应用程序中。问题在于使用原始字符串时，这种实现非常脆弱。希望监听您的通知的对象将使用通知的名称来做出反应。例如，如果您决定将通知名称更改为其他名称，例如`didFinishDownload`，那么任何订阅`didFinish`的对象都将不会收到您的通知。

修复这个问题并不难。您会注意到`post`方法并不直接接收原始字符串，而是将其包装在名为`Notification.Name`的枚举中。为了解决这个问题，我们可以在类或结构体上创建一个此枚举类型的属性，并使用它来*存储*原始字符串，而不是直接传递和监听它。这种改变允许任何对象针对该属性进行目标设置，因此属性值的任何更改（例如通知名称）都将自动被我们的监听器捕获。以下是这种做法的示例：

```
class SomeObject {
	public static let didFinishNotification = Notification.Name("didFinish")
}

NotificationCenter.default.post(name: SomeObject. didFinishNotification, object: nil)
```

在我们的示例中，类`SomeObject`定义了一个名为`didFinishNotification`的新静态属性。这个属性包含我们在第一个示例中在调用站点定义的枚举的值。稍后，在通知发布方法中，我们使用这个属性而不是自己声明值。这样做可以让我们将通知的名称从“didFinish”更改为更符合最佳实践、防止名称冲突的名称，例如“SomeObjectDidFinishNotification”。

也可以通过使用稍有不同的`post`方法，在通知中附加数据，这是一种相当常见的做法。以下是一个示例：

```
NotificationCenter.default
    .post(name: SomeObject. didFinishNotification, object: nil, userInfo:
    ["downloadCount": 3])
```

在这个示例中，我们传入了一个包含键名为“downloadCount”、值为`3`的字典。只要符合`Hashable`协议，我们可以传递各种数据，例如对象。如果我们想要传递一个对象，我们可以使用以下代码：

```
let anObject = SomeObject()
let count = 3

NotificationCenter.default
    .post(name: SomeObject. didFinishNotification, object: nil, userInfo:
    ["someObject": anObject, "downloadCount": count])
```

您会注意到在我们的示例中，我们再次使用预定义的原始字符串。这对于演示示例很容易，但在成品代码中最好避免使用，因为它会创建脆弱的连接。我们可以通过在类上包含与此键对应的属性来类似解决通知名称的问题，如下所示：

```
class SomeObject {
	...
}

// Use an extension to encapsulate your notification code
extension SomeObject {
	public static let didFinishNotification =
      Notification.Name("SomeObjectDidFinishNotification")
	public static let didFinishNotificationObjectKey = "someObjectKey"
	public static let didFinishNotificationDownloadCountKey = "downloadCount"
}

NotificationCenter.default
    .post(name: SomeObject. didFinishNotification, object: nil,
    userInfo: [SomeObject. didFinishNotificationObjectKey: anObject,
    SomeObject. didFinishNotificationDownloadCountKey: count])
```

通过为字典键使用静态属性，您可以为通知添加一些编译时安全性，使您的代码更加稳定。当我们的通知属性开始超出一个或两个通知名称时，将代码拆分成扩展是标准做法，如前面所示。

我们正在发布通知并传递数据，但还没有人在听我们说什么。让我们开始订阅这些通知，看看是什么样子。

## 监听并对系统中分派的消息做出反应

在 Swift 中订阅通知有两种方式：一种是使用选择器（`@objc` 方法），另一种是使用闭包。两者在本质上非常相似，但在代码和逻辑上有轻微的差异。以下是使用选择器的示例（这种方式在某种程度上更为常见）：

```
class SomeObject {
	func listenForNotifications() {
		// Subscribe to the notification
		NotificationCenter.default
        .addObserver(self, selector: #selector(didFinishDownload(_:)),
        name: SomeObject.didFinishNotification, object: nil)
	}

	@objc func didFinishDownload(_ notification: Notification) {
		// Get the notification payload
		let downloadCount =
       notification.userInfo?[SomeObject.didFinishNotificationDownloadCountKey]
		print(downloadCount)
	}
}
```

让我们来分解这段代码。

首先，我们在示例类中添加了一个 `listenForNotifications` 方法。在这个方法中，我们将自己添加为先前创建的通知 `SomeObject.didFinishNotification` 的观察者。我们将任何对象作为观察者，作为我们控制的一个子对象的示例，但通过将 `self` 作为第一个参数传递，对象直接进行了分配。我们的目标是 `didFinishDownload(_:)` 选择器，在此之后使用 `@objc` 关键字定义了这个方法。

现在，在 `didFinishDownload(_:)` 方法的一部分中，传递了一个 `Notification` 对象。这个对象包含一个 `userInfo` 属性。这是一个可选的 `Dictionary`，我们可以在其中访问我们在上一个示例中传递的通知载荷。使用我们之前定义的通知载荷键 `SomeObject.didFinishNotificationDownloadCountKey`，我们可以访问通知中的数据，然后在下一行将其打印到控制台。

## 使用闭包而不是选择器

使用闭包来观察通知与之前的选择器样式观察非常相似。以下是与之前示例中的选择器不同的闭包形式的相同通知观察：

```
class SomeObject {
	private var observer: AnyObject?

	func listenForNotifications() {
		// Subscribe to the notification
		self.observer = NotificationCenter.default
      .addObserver(forName: SomeObject.didFinishNotification, object: nil,
      queue: nil) { notification in
  let downloadCount =
    notification.userInfo?[SomeObject.didFinishNotificationDownloadCountKey]
    print(downloadCount)
		}
	}
}
```

首先，重要的是指出选择器和闭包观察之间的主要区别：观察者存储为名为 `observer` 的变量。这个对象是在 `listenForNotifications()` 方法内部调用 `addObserver(forName:object:queue:using:)` 方法的结果。我们需要存储对这个对象的引用，否则我们的通知实际上会被释放，从而永远不会被调用。稍后，一旦我们取消订阅通知，我们将将此属性设置为 `nil`，以允许从内存中释放所有内容。

传递进来的闭包体与我们之前的 `didFinishDownload(_:)` 选择器相同；它获取作为通知一部分传递的载荷，然后将值打印到控制台。

## 停止监听通知

我们已经展示了如何发布通知和如何监听通知。这个过程的最后一步是展示如何停止监听通知。考虑到发布通知和监听通知所需的所有设备，从观察者中移除自己实际上非常简单。

```
// Selector style
NotificationCenter.default.removeObserver(self)

// Closure style
NotificationCenter.default.removeObserver(observer)
self.observer = nil
```

这会取消对象对其可能正在监听的所有通知的订阅，对于选择器样式的通知来说，这有点像是使用锤子，但通常是断开与 `NotificationCenter` 的关系的简洁方式。对于选择器样式的通知订阅，最好的做法是按名称取消订阅通知。方法调用如下所示：

```
NotificationCenter.default
    .removeObserver(self, name: SomeObject.didFinishNotification, object: nil)
```

此代码仅取消订阅 `SomeObject.didFinishNotification` 通知，并保留所有其他订阅。

###### 警告

删除对象作为观察者从通知中移除的重要性可能并不显而易见。通过移除对象，实际上是在减少该对象的引用计数，如果没有其他对象引用它，它将被释放出内存。内存泄漏是一个常见且容易犯的错误，特别是在基于闭包的通知观察中。始终将自己从观察者中移除，并不要忘记为基于闭包的观察对象设置为 `nil`。

### 针对特定对象的通知

虽然之前没有提到，但在观察通知时可以针对特定对象，这样你就只会接收到该对象发布的通知。可以通过在 `addObserver` 方法中传递对象作为参数来实现：

```
let objectToObserve = SomeObject()

// You will only receive notifications that objectToObserve posts
NotificationCenter.default
    .addObserver(self, selector: #selector(didFinishDownload(_:)),
    name: SomeObject.didFinishNotification, object: objectToObserve)

There a few different types of combinations available, for example:

// Receive all notifications from specificed object
NotificationCenter.default
    .addObserver(self, selector: #selector(didFinishDownload(_:)),
    name: nil, object: objectToObserve)

// Receive all didFinishNotification notifications from all objects
NotificationCenter.default
    .addObserver(self, selector: #selector(didFinishDownload(_:)),
    name: SomeObject.didFinishNotification, object: nil)

// Receive all notifications from all objects (don't do this)
NotificationCenter.default
    .addObserver(self, selector: #selector(didFinishDownload(_:)),
    name: nil, object: nil)
```

### 线程

通知是在发布它们的同一线程上接收的。这意味着，为了在后台线程接收到通知时更新用户界面，使用诸如 `DispatchQueue` 的内容将其调度到主线程非常重要。可以通过像这样用 `DispatchQueue.main.async { ... }` 包裹需要在主线程上运行的代码块来完成这一点：

```
class SomeObject {
	...

	@objc func listenForNotifications(_ notification: Notification) {
		DispatchQueue.main.async {
			updateUI()
		}
	}
}
```

### 键-值观察

Cocoa Touch 有一个完整的消息传递系统，由于主题的复杂性而未触及：键-值观察（KVO）。可以观察对象的单个属性，并在该属性的值更新时接收更新。不幸的是，它是一个设计不佳且较老的 API；只能在继承自 `NSObject` 的对象上使用。即使小心使用，也很容易出错。甚至在作者看来，前面提到的方法通常也不是更合适的选项之一。

然而，如果您感兴趣，可以从苹果的[键-值观察编程指南](https://oreil.ly/j8oQV)获取有关 KVO 的信息。

# 我们学到了什么

在本章中我们覆盖了很多内容。你已经看到了：

+   如何在 iOS 和 Android 中以各种方式传递消息。有直接回调，这是一种非常一对一的消息传递方式。

+   在 Android (`LocalBroadcastManager`) 和 iOS (`NotificationCenter`) 中处理解耦消息传递的方式。

在你的应用程序中传递消息有许多方法。根据情况的不同，界限可能变得模糊，但通过这里描述的方法，你可以开始使应用程序的不同部分彼此交流。这对于创建可维护和解耦的架构至关重要。
