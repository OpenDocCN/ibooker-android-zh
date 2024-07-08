# 第十章：用户反馈

在任何程序中，能够在短暂的 UI 中向用户提供信息非常重要。特别是在移动开发中，如果每个警告、提示或通知都出现在 UI 中，我们很快就会用完可用的空间。这两个框架都提供了多种功能来实现这一点。

# 任务

在本章中，您将学习：

1.  使用框架提供的工具在各种情况下向用户显示反馈。

1.  更新状态栏。

# Android

Android 框架提供了几个 API，用于显示关于事件、错误或状态变化的即时反馈。有些只显示文本，但其他一些则允许各种程度的定制和交互。我们将检查三个主要的反馈类，并向您展示如何编程以满足您的特定需求。

## 使用框架提供的工具向用户显示反馈

虽然您可以使用标准布局和绘图方法以任何您想要的方式向用户显示信息，但 Android 框架提供了三个主要的 API 来图形化地向用户提供反馈：

+   `Toast` 类

+   `Snackbar` 类

+   `Dialog` 类

`Toast` 类早于 `Snackbar`，虽然基本用法非常相似，但可能更简单一些。另一方面，如果正确使用，`Snackbar` 可以提供更好的用户体验，并具有 `Toast` 消息无法实现的灵活性。对话框是支持各种元素的模态接口，包括用于向用户显示信息并从用户那里接收信息的元素。

让我们开始吧。

### 提示

提示消息是与实现相关的。某个 Android 操作系统的版本可能会选择以不同的方式实现细节，或者在 Android 操作系统标准较为宽松的较旧设备上，甚至可能会因设备的品牌和型号而有所不同。

一般来说，提示消息是显示在现有内容上的简短、短暂的消息。来自 [Android 开发者文档](https://oreil.ly/zXgqS)：

> 提示（Toast）在小弹出窗口中提供有关操作的简单反馈。它仅填充消息所需的空间，当前活动保持可见和交互。提示会在超时后自动消失。

要显示提示消息，请使用以下方法：

就是这样！我们有一个 `Context` 实例、要显示的消息以及持续时间常量（`LENGTH_SHORT` 或 `LENGTH_LONG`）。

还有其他可用的 API，比如设置 `Toast` 在屏幕上显示的位置（其“重力”），甚至提供具有自定义 `View` 树的 `Toast` 弹出窗口，但大多数情况下，`makeText` 和 `show` 就足够了。

## Snackbar

正如之前提到的，`Toast` 和 `Snackbar` 的基础非常相似——都在现有 UI 的顶部显示短暂的消息。实际上，如果考虑到与之前显示的基本 `Toast` 调用相比，基本的 `Snackbar` 调用几乎是相同的：

`Snackbar.make` 方法的第一个参数不是 `Context` 实例，而是一个 `View` 实例，并且方法的命名略有不同（`Toast.makeText` 与 `Snackbar.make`），但它们本质上是相同的。

在用户体验方面的主要差异在于：

1.  `Snackbar` 通常从屏幕底部滑出，而 `Toast` 则通常从屏幕中心淡入。

1.  通过单个方法调用 `setAction`，`Snackbar` 可以在消息右侧提供一个简单的操作按钮。使用 `Toast` 也可以做到类似的功能，但需要进行更多的自定义编码。

还需要注意，根据 [Android 开发者文档](https://oreil.ly/XPy7s)，开发者现在被鼓励优先选择 `Snackbar` 而不是 `Toast`。

> `Snackbar` 类取代了 Toast。尽管目前仍支持 Toast，但 Snackbar 现在是向用户显示简短临时消息的首选方式。

然而，`Snackbar` 与 `Toast` 的主要区别可以在将 `Snackbar` 附加到 `CoordinatorLayout` 上时看出，`CoordinatorLayout` 应该是作为 `make` 方法的第一个参数传递的 `View` 实例的根视图。通过这样做，`Snackbar` 将获得一些额外功能，比如可以通过滑动屏幕来关闭它，还能感知到 `CoordinatorLayout` 管理的其他组件。例如，当 `Snackbar` 滑入时，`FloatingActionButton` 将向上移动并躲开。详细信息请参见 [Android 开发者文档](https://oreil.ly/n8Mbx)。

### 对话框

`Dialog` 及其子类比之前的 `Toast` 和 `Snackbar` 类更为强大和灵活，但也可能需要更多的注意、维护和配置。

来自 [开发者文档](https://oreil.ly/xPFao)：

> 对话框是一个小窗口，提示用户做出决定或输入额外信息。对话框不会填满屏幕，通常用于需要用户在继续之前采取行动的模态事件。

与前面讨论过的类一样，`Dialog` UI 可以完全自定义；还有单选或多选反馈的功能，添加用户响应功能也相对简单。

要创建并显示一个带有标题、消息和“接受”或“取消”操作按钮的 `Dialog`，您可以像这样操作：

欲了解更多关于 `Dialog` 的信息，请参阅 [开发者文档](https://oreil.ly/-VXCz)。

## 更新状态栏

在 Android 中，状态栏是屏幕的最顶部区域，您可以在那里找到像时间、电池电量、网络状态等系统信息。这种类型的系统级数据通常显示在顶部栏的右半部分，而应用级信息通常以非常小的图标形式显示在左侧边缘。这些图标称为“通知”图标，通常对应一个`Notification`实例，它是一个存在于您的应用程序之外的 UI，通常包含一个简短的消息，但也可能包含一些可操作的项目。例如，当下载完成时，您的应用程序可以发布一个`Notification`，简要向用户显示一条消息，并在状态栏中保留一个图标，以便用户稍后再次访问该消息。该消息可能还附带一些操作，比如打开您的应用程序到下载项目的详细屏幕，或者一个下载列表。`Notifications` API 功能强大，但可能不像前面讨论的一些用户反馈功能那样直观。

还要注意，随着 AOSP 的成熟，`Notifications` API 已经有了多次重大变化。Android 操作系统的版本 5 和版本 8 引入了重大变化。我们在这里只涉及基础知识，但像往常一样，请查看[开发者文档](https://oreil.ly/wGqyn)以深入了解。

在最基本的层次上，这里是如何使用*compat*库显示`Notification`（确保你有依赖`com.android.support:support-compat:xx.xx.xx`）：

当用户点击`Notification`时启动一个`Activity`，使用`PendingIntent`类：

在调用`build`之前将其添加到构建器中：

欲深入了解，请参阅[开发者文档](https://oreil.ly/rcuYz)。

###### 注意

这些内置的用户反馈机制可以为您的用户提供一致的体验。例如，使用带有接受和取消按钮的内置对话框系统可能更加符合用户的习惯，因此如果通过传统的、框架提供的 UI 来提供这些功能，用户可能更愿意订阅您的通讯（或给您的应用评分）。尽管如此，如果您的要求超出了这些 API 所允许的范围，您可以自由创建（或修改）更适合您需求的自定义 UI 组件。

# iOS

如果你已经阅读了本章的 Android 部分，你会很快发现这是 iOS 和 Android 差异最大的领域之一。Android 和 iOS 在显示用户反馈的可用方法中差异尤为显著。让我们看看有哪些选择，以及它们在哪些地方表现最为突出。

## 使用框架提供的工具来显示用户反馈

在 iOS 上向用户显示反馈有两种方式：警告视图和操作表。它们都是由同一个类`UIAlertController`创建，唯一的区别是在初始化警告视图时传入`.alert`，在操作表中使用`.actionSheet`。

###### 提示

iOS 中默认情况下不可能显示类似 Android 中的`Snackbar`风格通知。但是可以使用第三方库来实现类似的效果。然而，这超出了本书的范围。

Apple 没有绝对明确的指导意见，关于何时使用操作表和何时使用警告，但对大多数应用程序来说，已经形成的标准是在需要即时反馈且可能是意外的情况下使用警告视图；在需要用户选择并且有操作上下文的情况下使用操作表。

这里展示了一个警告视图示例：

```
let viewController = UIViewController(nibName: ..., bundle: ...)

let alert = UIAlertController(
    title: "Title", message: "This is the message", preferredStyle: .alert)
alert.addAction(
  UIAlertAction(title: "OK", style: .default, handler: { (action) in
    print("OK button pressed!")
}))

viewController.present(alert, animated: true, completion: nil)
```

让我们一起浏览代码。

首先，我们创建一个视图控制器，稍后将用来呈现我们的警告。自 iOS 8 以来，警告视图实际上是可以由另一个视图控制器呈现的`UIViewController`实例，就像任何其他视图控制器一样。接下来，我们通过`UIAlertController`的初始化方法实例化一个警告。这个类有一个标题、一条消息和一个`preferredStyle`参数。我们传入`.alert`表示我们正在创建的是一个警告视图；使用`.actionSheet`来创建操作表。

下一行包含一个名为`addAction(_:)`的方法，用于向警告视图添加动作或按钮。我们直接实例化一个`UIAlertAction`实例，并将其带有标题、样式（本例中为`.default`）和按钮处理程序传递到这个方法中。在按钮处理程序内部，每当按下按钮时，我们向控制台输出“OK button pressed!”。

最后，我们使用代码顶部创建的视图控制器来显示我们的警告视图，覆盖在视图堆栈的其余部分之上。

这个过程的核心是在警告本身调用的`addAction(_:)`方法。这是我们添加所有按钮到我们的警告或者可能的操作表中的地方。

现在，可以创建一个带有多个按钮的警告。这里有一个带有三个按钮的示例：

```
let viewController = UIViewController(nibName: nil, bundle: nil)

let alert = UIAlertController(title: "Terms & Conditions",
    message: "Hit Agree to agree to terms and conditions.", preferredStyle: .alert)

let agreeAction = UIAlertAction(title: "Agree", style: .default) { (action) in
    print("Terms and conditions accepted.")
}

let viewAction =
  UIAlertAction(title: "View Terms & Conditions", style: .default) { (action) in
    print("Blah blah blah.")
}

let cancelAction = UIAlertAction(title: "Cancel", style: .cancel) { (action) in
    print("I disagree with terms and conditions.")
}

// Add all the actions
alert.addAction(agreeAction)
alert.addAction(viewAction)
alert.addAction(cancelAction)

viewController.present(alert, animated: true, completion: nil)
```

在这里，我们使用几乎相同的逻辑来创建和显示我们的警告视图。为了更易于阅读的一个关键区别是，我们将动作单独创建然后直接添加到警告中，而不是在调用`addAction(_:)`时直接实例化它们。

另一个轻微的区别是我们使用 `.cancel` 样式来创建 `cancelAction`。可用的操作样式集合有限。`.default` 样式显示一个在两个按钮配置中加粗显示的基本按钮。`.cancel` 样式显示一个在取消按钮位置上具有指定标签的按钮。剩余的样式 `.destructive` 显示一个通常带有红色文本颜色以指示其具有破坏性潜力的按钮。

这是使用 `.destructive` 样式创建的操作：

```
let deleteAction =
  UIAlertAction(title: "Yes, Delete", style: .destructive) { (action) in
    print("All your photos were deleted. What were you thinking?")
}
```

如果有任何关于提示视图和操作表的指导意见，那就是只有在必要时才显示提示视图。它们会打断用户在应用程序中的流程，如果过度使用，很快就会变得令人厌烦并被忽视。伟大的力量需要伟大的责任。如果您同意，请点击“是”。

## 更新状态栏

在 Android 上的 `Notifications` API 允许开发者自定义状态栏中的项目，并以这种方式向用户提供反馈。在 iOS 上，开发者唯一可以使用状态栏进行反馈的方式是通过网络状态栏活动指示器。

尽管在外观上很小，但这个指示器对用户来说可能非常有帮助。它可以帮助用户诊断应用程序可能变慢的原因，并指示数据仍在加载。诸如此类的小细节可以决定一个应用程序是感觉未完成还是感觉完整和精致的区别。

以下是操作方法：

```
// Display the indicator
UIApplication.shared.isNetworkActivityIndicatorVisible = true

// Hide the indicator
UIApplication.shared.isNetworkActivityIndicatorVisible = false
```

更新显示并隐藏指示器非常容易。大多数第三方网络库都有自己的方法来完成，但几乎所有这些方法最终都会包装成如下代码：

```
class NetworkClient {
    func startDownload() {
        ...

        UIApplication.shared.isNetworkActivityIndicatorVisible = true
    }

    func downloadCompleted() {
        ...

        UIApplication.shared.isNetworkActivityIndicatorVisible = false
    }
}
```

它可能没有 Android 那么全面，但对用户仍然非常有帮助。

### 提示视图中的文本字段

不仅可以用提示视图来提供反馈，还可以从用户那里获取输入。提供了一个便利的方法来添加文本字段并进行样式化，如下所示：

```
alert.addTextField { (textField) in
	textField.placeholder = "Enter your comment"
}
```

您可以稍后在 `UIAlertAction` 中访问此文本字段的值：

```
let action = UIAlertAction(title: "Save", style: .default) { (action) in
    guard let textField = alert.textFields?[0] else { return }
    print(textField.text ?? "")
}
```

### 触觉反馈

可以通过触觉反馈以非视觉方式向手机用户提供反馈。有三种用于不同类型反馈的类别：

+   `UINotificationFeedbackGenerator`

+   `UIImpactFeedbackGenerator`

+   `UISelectionFeedbackGenerator`

举个快速的例子，为了向用户提供关于下载失败的触觉反馈，您可以使用以下代码片段：

```
let hapticFeedbackGenerator = UINotificationFeedbackGenerator()
hapticFeedbackGenerator.notificationOccurred(.error)
```

这将生成一个由 Apple 设计的感觉消极和紧急的轻敲。请谨慎使用，并记住，在发布时并非所有设备类别，例如 iPad，都支持触觉反馈。

# 我们学到了什么

这似乎是在开发 Android 和 iOS 应用程序时介绍了两种技术堆栈之间存在的最大差异之一的领域：

+   Android 有多种方式向用户提供反馈，比如 `Toast`、`Snackbar` 和 `Dialog`。同时，也可以通过警报更新 Android 状态栏来提示用户。

+   在 iOS 中，用户反馈方面的控制选项较为有限。尽管如此，这些选项使用起来简单，并遵循 UIKit 的标准 UI 约定。

+   在 iOS 中，更新状态栏实际上是不太可能的。这突显了 Android 和 iOS 之间的一个核心差异：开发者对设备的控制程度。在 iOS 中，苹果公司对此有着更为严格的控制。开发者只能指示网络活动。

通过本章我们学到了让我们的应用与用户进行交流的许多方法。在下一章中，我们将讨论提供一种比整个数据库持久化层稍微简单一些的存储信息方式。
