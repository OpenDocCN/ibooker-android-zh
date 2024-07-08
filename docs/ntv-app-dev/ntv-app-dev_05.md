# 第四章：用户输入

数十年来，与计算机交互的主要手段是键盘和鼠标。用户与使用的设备是紧密相连的。唯一的工作方式是坐在工作站前开始工作。最终，笔记本电脑和便携电脑增加了更多的灵活性，但输入机制基本保持不变。

然后是触摸。

如今，安卓和 iOS 设备已不再与用户保持一段距离。它们与用户保持亲密的物理接触。当用户按下按钮时，从用户的角度来看，它们直接被轻点，而不是通过触摸板或键盘快捷键。这使得输入成为将任何旧应用程序转变为理解用户的动态艺术品的关键要素之一。

输入可以采用多种形式和方式：在 Web 视图中点击链接，输入登录表单中的密码，或在屏幕上滑动以查看面孔，以了解是否与另一个孤独的灵魂有情感上的联系，或者最终可能演变成爱情。风险很高，但平台提供了一套强大的工具来获取用户的原始输入并将其转换为用户可以看到、听到或触摸到的操作结果。

# 任务

在本章中，你将学习：

1.  接收并响应点击。

1.  接收并响应键盘输入。

1.  处理复合手势。

# 安卓

虽然安卓手势 API 可能有点繁琐，但它们相当透明，作为开发者，你将拥有满足最苛刻的触摸密集型应用程序所需的所有信息和访问权限。

## 接收并响应点击

在大多数现代移动应用程序中，点击或许是最常见的用户输入形式。无论是点击按钮提交表单，点击输入文本字段将焦点设置到它上面，长按以显示上下文选项，还是双击缩放地图以放大或缩小，此事件都是选择和接受的直观表达。

因此，安卓框架让捕捉点击变得简单且高效可用。

###### 提示

基于传统原因，安卓框架在某些情况下仍然使用术语“点击”。在大多数触摸屏框架中，“点击”与“轻点”是同义词。

所有的`View`实例（包括`ViewGroups`）都可以接受`View.OnClickListener`作为一个可设置的属性（通过`setOnClickListener`）。一旦设置，框架会处理底层复杂性，当任何手势符合框架的条件时，监听器的`onClick`方法将被触发。要移除对给定视图的点击操作，只需将监听器设置为 null：`myView.setOnClickListener(null);`。

注意，`View.OnClickListener` 是一个简单的函数接口，只有一个方法：`onClick(View view)`。这是根据[源代码](https://oreil.ly/NGg8e)在撰写时的直接复制粘贴：

```
public interface OnClickListener {
  void onClick(View v);
}
```

这种架构意味着界面可以在几乎任何级别实现——在像 `Activity` 或 `Fragment` 这样的控制器上，或者直接在 `View` 实例上，或者在匿名类、lambda 或方法引用上。此外，点击监听器可以在 XML 布局中分配。我们将逐一查看这些方法。

使用控制器实现 `View.OnClickListener`：

使用控制器实现方法引用：

使用 lambda 表达式：

使用匿名类实例：

在总是具有相同点击行为的 `View` 子类上：

最后，您可以在布局的 XML 中使用方法名（作为 `String`）来分配点击监听器。包含的 `Activity` 必须具有与 `View.OnClickListener.onClick` 签名匹配的公共方法：

```
<!-- contents of res/layout/myactivity_layout.xml -->
<Button xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="Click me!"
    android:onClick="myClickHandler" />
```

注意，`Activity` 将自动接管关系并创建绑定逻辑，无需明确引用方法或 `View`：

```
public class MyActivity extends Activity {

  @Override
   protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.myactivity_layout);
   }

   public void myClickHandler(View view) {
    Log.d("MyTag", "View was clicked " + view.toString());
   }

}
```

请注意，一个 `View` 最多只能同时设置一个 `OnClickListener`。要有多个点击监听器，您可以更新监听器以调用其他监听器，或者创建一个支持多个监听器的小框架。例如，您可以使用以下方式在单个监听器中管理回调列表：

可以如下使用：

尽管处理轻触事件的选项似乎很广泛，但实际上这只是冰山一角。Android 框架提供了多个级别的触摸事件访问，如果需要，您可以自行实现轻触逻辑——例如，您可能希望仅在一段延迟后触发轻触，或者您可能希望有一个更宽容（或更保守）的“漫游”区域（原始触摸事件在不再被视为轻触之前可以漫游多远）。幸运的是，您可能永远不需要这样做，但我们稍后将深入研究手势管理。

## 接收并响应键盘输入

Android 框架处理键事件的方式与您可能处理过的其他 UI 框架有很大不同。即使是 `KeyEvent` —— 这可能是您期望直接处理的 API —— 也很少被开发人员直接访问。请注意，即使是当前的文档也指出：

> 由于软输入方法可以使用多种创新方式输入文本，不能保证软键盘上的任何按键都会生成键事件：这取决于输入法编辑器的决定，事实上，不鼓励发送此类事件。您不应依赖于接收软输入法上任何按键的键事件。

这仅仅说明了来自“软件”（屏幕上的）键盘的键事件不能保证。对于“硬件”键盘（物理键盘，如您在少数现代智能手机或通过蓝牙或 USB 连接的便携式键盘上找到的那种），它们是有保证的；然而，这并不是非常有帮助，因为您将要对其做出反应的大多数键输入事件将来自软键盘。此外，即使是钩住这些事件，也需要一些相当复杂的设置，包括绑定到“IME”（输入法）、注册焦点、根据需要扩展和收缩键盘等。

在深入研究开发人员文档时，我们找到了一个名为“处理键盘操作”的部分。听起来很有前途，但我们立刻又看到了一个引人注目的横幅：

> 当使用 KeyEvent 类和相关 API 处理键盘事件时，您应该期望这些键盘事件仅来自硬件键盘。您不应该依赖于从软输入法（屏幕键盘）接收任何键的键事件。

所以我们该怎么办？我们有几个策略…

首先，更常见的是，我们实际上可能更感兴趣的是在编辑文本更改其值时触发的更改事件，而不是实际的`KeyEvent`。在这些情况下，我们可以访问`TextWatcher`接口，该接口需要实现三个方法：

+   `onTextChanged`

+   `beforeTextChanged`

+   `afterTextChanged`

`TextWatchers`可以监听`TextView`实例（包括`EditText`）上的文本更改事件，使用`addTextChangedListener`监听器。

###### 注

这是少数几个允许附加多个监听器的监听器 API 之一。为了支持这一点，还有一个对应的`removeTextChangedListener`方法。

使用`TextWatcher`，我们可以检测输入文本字段的值何时发生更改，这通常正是我们在监听键事件时要做的事情。虽然`TextWatcher`接口的方法签名可能会有很大不同，但每个方法都提供对已更改的文本的访问，可以作为`Editable`实例或`CharSequence`实例：

除了文本更改之外，假设我们的用户只很少使用外部物理键盘，我们需要承认我们主要感兴趣的是软键盘行为，并稍微了解“IME”的概念。“IME”代表“输入法编辑器”，在技术上是指任何可以处理来自硬件组件的事件的东西，但实际上几乎完全是指通过`TextView`管理软键盘的内容，通常通过`EditText`实例，这是`TextView`的子类，具有内置的编辑功能。

像大多数`View`配置一样，IME 通常可以通过 XML 指令或编程语句来处理。最常见的 IME API 是“IME options”：即`android:imeOptions`或`TextView.setImeOptions`，接受表示各种 IME 标志的整数，例如“go”、“next”、“previous”、“search”、“done”和“send”（还有其他）。虽然选项的语义有时会随行为表达，但并非总是如此。例如，虽然“next”和“previous”将改变屏幕的焦点，“go”、“done”和“send”可能没有明确的不同，但应该向附加的监听器传递不同的值。

例如，你可以使用`android:imeOptions="actionSend"`创建一个`EditText`。当该`EditText`获得焦点时，屏幕上将会打开软键盘，并且会有一个专门用于“发送”操作的按钮（通常这个按钮会显示为键盘上标记为“发送”的按钮，使用设备的本地语言）。点击这个按钮将会触发注册的`TextView.OnEditorActionListener`来执行其`onEditorAction`事件（稍后我们会详细介绍）。

类似地，你可能会使用`android:imeOptions="actionNext"`，这表示软键盘会呈现一个带有“下一个”表示的按钮（通常是一个向右指向的箭头）。点击这个按钮通常会将焦点发送到视图树中的下一个可用 IME（可能是一个`EditText`）。

如果你想对 IME 按钮的行为有更具体的控制，可以使用`TextView.OnEditorActionListener`。你可以使用`setOnEditorActionListener`方法将这个监听器的实例分配给 IME（如一个`EditText`），就像分配任何监听器一样（类似地，将此值设置为`null`以删除先前附加的监听器）。

`OnEditorActionListener`实例实现了一个方法：`public boolean onEditorAction(TextView view, int actionId, KeyEvent event)`。请随意使用传递给监听器的任何参数，但通常`actionId`标志最有趣。在上一个例子中，当点击右指向按钮时，任何附加的`OnEditActionListener`实例将触发它们的`onEditAction`方法，参数包括：打开键盘的`View`实例，一个等于`EditorInfo.IME_ACTION_NEXT`的整数常量，以及描述“下一个”[按键事件](https://oreil.ly/pOZn8)的`KeyEvent`。

## 处理复合手势

如果你需要超出预设功能的手势功能，你有几种可用的机制。我们认为最直接的方法是简单地重写`ViewGroup`（或`Activity`！）的`onTouchEvent`方法，并以任何适合你需求的方式处理每个事件。每个运动事件都有一个类型标志（例如，一个手指开始一个手势[`ACTION_DOWN`]，在屏幕上移动[`ACTION_MOVE`]，结束一个手势[`ACTION_UP`]，或其他类似的方法用于多点触控）。有了这些信息和恰当使用时间戳，你可以实现你的应用可能需要的任何自定义行为。

在编写自定义手势功能时，还有其他可用的 API 可以使复杂任务变得更加容易，例如`Scroller`，尽管其名称并不实际执行任何滚动运动，但确实具有一些非常方便的惯性滚动衰减或抛物线计算方法。`VelocityTracker`用于记录运动事件并提供关于任一轴上的速度和加速度的信息。

如果这些还不够或者你的需求不需要如此精细的控制，一个简单的访问手势的方法是使用`GestureDetector`（或来自支持库的`GestureDetectorCompat`）。`GestureDetector`实例可以传递一个`GestureListener`，并提供触摸事件，以返回常见的回调，包括：

+   `onDown`

+   `onFling`

+   `onLongPress`

+   `onScroll`（可以将其视为“拖动”）

+   `onShowPress`

+   `onSingleTapUp`

要实现这一点，你需要一个`GestureDetector`实例，它需要一个`Context`实例和一个`GestureListener`实例：

`GestureDetector`实例负责大部分核算；它将使用系统提供的值，如重力和触摸误差，因此你可以确信你的应用将在与`ScrollView`或`RecyclerView`相同的条件下启动一个抛掷动作。

当一个父`ViewGroup`包含能够消耗触摸事件的`View`子项（甚至通过简单的`View.onClickListener`），一个已经复杂的手势管理系统很快就会变得难以管理。通常情况下，你可以结合使用`onInterceptTouchEvent`和`onTouchEvent`（参见[开发者文档关于前者](https://oreil.ly/qCLNx)）；在这两者之间，你几乎可以至少访问到在任何容器内发生的触摸事件。

`View`类实例可用的其他事件回调包括：^(1)

+   `onKeyDown(int, KeyEvent)`: 当新的按键事件发生时调用。

+   `onKeyUp(int, KeyEvent)`: 当按键弹起事件发生时调用。

+   `onTrackballEvent(MotionEvent)`: 当轨迹球事件发生时调用。

+   `onTouchEvent(MotionEvent)`: 当触摸屏幕动作事件发生时调用。

+   `onFocusChanged(boolean, int, Rect)`: 当视图获得或失去焦点时调用。

要了解更多关于手势检测的信息，请查阅[Android 的优秀指南](https://oreil.ly/tFb5K)。

# iOS

2007 年，苹果推出了 iPhone，随之而来的是多点触控技术的诞生。尽管现在它已经无处不在，但在当时，能够在玻璃屏幕上使用多个手指是一个革命，并改变了用户界面。触摸目前是与智能手机交互的主要方式，但绝不是唯一的方式。本章涵盖了两种最常见的输入方法：触摸和键盘。让我们深入探讨一下。

## 接收并响应轻击

iOS 中可用的触摸事件 API 可以说是行业最佳。它们随时间略有变化，但自 iOS 4 以来引入手势识别器后基本保持不变。这绝对是拦截触摸事件最简单的方法。以下是如何在视图控制器内监听图像视图上的单次轻击的示例：

```
class SomeViewController: UIViewController {
    var imageView: UIImageView!

    override func viewDidLoad() {
        super.viewDidLoad()
        imageView = UIImageView(image: ...)
        let gestureRecognizer =
          UITapGestureRecognizer(target: self, action: #selector(handleTap(_:)))
        gestureRecognizer.numberOfTapsRequired = 1
        imageView.addGestureRecognizer(gestureRecognizer)
    }

    @objc func handleTap(_ gestureRecognizer: UIGestureRecognizer) {
        print("Image tapped!")
    }
}
```

我们从声明我们的`UIViewController`子类`SomeViewController`开始。在这个类中，大部分操作发生在`viewDidLoad()`中。这是 iOS 中视图生命周期的一部分，通常在这里可以对视图控制器的视图进行设置。查看第二章获取关于视图的更多信息。

在这个方法中，类的图像视图`imageView`被设置。在下一行，我们声明了一个类型为`UITapGestureRecognizer`的手势识别器，它通过`self`指向这个类，并提供了`handleTap(_:)`方法作为触发此手势识别器时要调用的函数。

在将手势识别器的`numberOfTapsRequired`属性设置为`1`后，表示它是一个单击识别器，我们将手势识别器添加到之前定义的图像视图上。将手势识别器附加到视图是必需的，以使该识别器触发。在我们的示例中，这意味着每当触摸或点击图像视图时，它将遍历与之关联的识别器列表，并尝试解析哪些触摸有效地触发特定的手势识别器。

假设一个触摸被我们的手势识别器注册，识别器本身将调用`handleTap(_:)`，这是我们刚刚定义的动作。

###### 注意

注意`handleTap(_:)`是一个`@objc`方法。这是因为`UIGestureRecognizer`和其子类在激活手势识别器时需要传入`#selector(...)`作为触发的动作。

我们的示例中有一些样板代码，但基本上归结为两行：

```
let gestureRecognizer = UITapGestureRecognizer(target: self, action:
#selector(handleTap(_:)))
imageView.addGestureRecognizer(gestureRecognizer)
```

我们声明手势识别器并将其附加到一个视图上。

手势识别器非常强大。我们稍后将在本章讨论它们。现在，让我们把注意力转向 iOS 上的另一个主要输入源：键盘。

## 接收并响应键盘输入

与 Android 不同，从未有过带有物理键盘的 iPhone 或 iPad。从理论上讲，这种情况可能会在未来发生变化，但考虑到苹果过去的立场，这种可能性非常小。iPad 有外部键盘（包括苹果制造的一种外壳），当然也可以连接蓝牙键盘作为屏幕键盘的替代品。尽管如此，在如此依赖“软键盘”的生态系统中，UIKit 中的键盘和文本字段库却令人沮丧地复杂——而且令人震惊，考虑到 UIKit 的其他一些区域使用起来是多么简单。

例如，在 iOS 上编辑文本的主要方式是通过`UITextField`或`UITextView`。这些用户界面控件每个都有单独的委托协议，它们在功能上略有不同，但主要是名称上的区别。尽管每个委托协议都很强大，但没有一个专门的方法可以在文本字段更改时获取更新。

还有其他考虑的方法。例如，可以将文本字段连接到编辑事件处理程序，如下所示：

```
class SomeViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        textField = UITextField(frame: ...)
        textField.addTarget(self, action: #selector(textFieldDidChange(_:)),
        for: .editingChanged)
    }

    @objc func textFieldDidChange(_ textField: UITextField) {
        print(textField.text)
    }
}
```

在示例中，在名为`SomeViewController`的视图控制器中，我们定义了一个名为`textField`的`UITextField`，在`.editingChanged`事件上添加了一个目标动作`textFieldDidChange(_:)`。每当用户在文本字段中编辑文本时，`textFieldDidChange(_:)`方法将会为每个添加或更新的字符调用一次；在我们的示例中，我们通过`print(textField.text)`打印出文本字段的文本。

大多数情况下这很有效，直到文本字段通过程序编辑时。然后，我们的`textFieldDidChange(_:)`方法会变得静默，并且我们的文本更改会在没有通知的情况下悄悄地进行。

捕获文本字段编辑的更可靠方法是通过添加类似以下方式的通知观察者：

```
class SomeViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        textField = UITextField(frame: ...)
        NotificationCenter.default
            .addObserver(self, selector: #selector(textFieldDidChange(_:)),
            name: UITextField.textDidChangeNotification, object: textField)
    }

    @objc func textFieldDidChange(_ notification: Notification) {
        let textField = notification.object as! UITextField
        print(textField.text)
    }
}
```

此示例与上一个示例类似，但有几点不同之处。首先，在定义我们的`UITextField`后，我们不再监听`.editingChanged`事件；现在我们监听`UITextField.textDidChangeNotification`。我们之前的同名方法`textFieldDidChange(_:)`在每次通知观察者触发时被调用；但是，为了针对文本字段，我们需要将`notification.object`强制转换为`UITextField`，以便在后续的`print(textField.text)`行中读取`text`值。

到目前为止，我们只对`UITextField`进行操作。当您需要观察多个文本输入以及混合使用`UITextField`和`UITextView`对象时会发生什么？您的代码很快可能会变成这样：

```
class SomeViewController: UIViewController {
    var textField1: UITextField!
    var textField2: UITextField!
    var textField3: UITextField!
    var textView1: UITextView!
    var textView2: UITextView!
    var textView3: UITextView!

    override func viewDidLoad() {
        super.viewDidLoad()
        NotificationCenter.default
            .addObserver(self, selector: #selector(textFieldDidChange(_:)),
            name: UITextField.textDidChangeNotification, object: nil)
        NotificationCenter.default
            .addObserver(self, selector: #selector(textViewDidChange(_:)),
            name: UITextView.textDidChangeNotification, object: nil)
    }

    @objc func textFieldDidChange(_ notification: Notification) {
        let textField = notification.object as! UITextField
        doSomething(textField.text!)
    }

    @objc func textViewDidChange(_ notification: Notification) {
        let textView = notification.object as! UITextView
        doSomething(textView.text!)
    }

    private func doSomething(for text: String?) {
        print(text)
    }
}
```

伤心。

但是让我们将这些忧郁的想法和不完整的框架放在一边，专注于不同的事物。让我们重新回到触摸输入，并讨论更复杂的手势识别器。这是 UIKit 的一个领域，它可以从简单的逻辑成功扩展到复杂的体验，而不会给开发者带来太多负担。

## 处理复合手势

手势识别器非常适合用于单指简单的轻击手势。但是它们对于复杂的交互链也非常有用。让我们来看一个示例：

```
let doubleTapRecognizer = UITapGestureRecognizer(target: self,
action: #selector(handleTap(_:)))
doubleTapRecognizer.numberOfTapsRequired = 2
```

这段代码与我们之前为单击手势识别器编写的代码类似。然而，通过简单地更改一个属性值，我们可以将其转换为双击手势识别器。

UIKit 中预先构建了其他手势识别器。如果您希望识别三指滑动手势，可以使用以下代码创建一个：

```
let panGestureRecognizer = UIPanGestureRecognizer(target: self,
action: #selector(handlePan(_:)))
panGestureRecognizer.minimumNumberOfTouches = 3
```

或者，如果您更愿意监听一些需要超出我们能够触及的物理动作，我们介绍五指三次轻击手势：

```
let fiveFingerTapRecognizer = UITapGestureRecognizer(target: self,
action: #selector(handleTap(_:)))
fiveFingerTapRecognizer.numberOfTapsRequired = 3
fiveFingerTapRecognizer.numberOfTouchesRequired = 5
```

你可能不太可能在许多发布的应用程序中看到这种情况。然而，在触摸界面中的一个常见问题是，您通常会在一个视图上监听多个触摸事件。如果没有意外地先触发单击手势识别器，那么您如何监听单击手势*和*双击手势呢？以下是可能的解决方法：

```
// Create a double tap recognizer
let doubleTapRecognizer = UITapGestureRecognizer(target: self,
action: #selector(handleTap(_:)))
doubleTapRecognizer.numberOfTapsRequired = 2

// Create a single tap recognizer
let singleTapRecognizer = UITapGestureRecognizer(target: self,
action: #selector(handleTap(_:)))
singleTapRecognizer.numberOfTapsRequired = 1
singleTapRecognizer.require(toFail: doubleTapRecognizer)
```

首先，我们创建一个名为`doubleTapRecognizer`的双击手势识别器。我们将`numberOfTapRequired`设置为`2`。接下来，我们创建一个名为`singleTapRecognizer`的单击手势识别器。我们将轻击次数设置为`1`，然后调用一个单独的方法`require(toFail:)`，并传入之前的双击手势识别器。

`require(toFail:)` 方法是所有手势识别器都具有的方法，允许它们仅在另一个已识别的手势识别器首次失败时才触发。以这种方式连接识别器使得单击识别器等待双击手势识别器失败后才调用其处理程序。不连接这两个手势识别器意味着单击识别器将在双击手势识别器的第一次和第二次轻击时触发。

理想情况下，这使得可以轻松看到如何连接多个定义了执行优先级的复合手势。您可以创建的手势识别器组合数量基本上是无限的；它超出了本书范围以列出它们所有，但如果您有兴趣了解更多手势类型，请查看 Apple 开发者文档中关于`UIGestureRecognizer`的内容。

### 触摸事件 API

iOS 响应链的一个特性是面向所有响应者（例如视图和视图控制器）的细粒度触摸事件 API。这是一组非常强大的方法，每当触摸开始、移动、结束或取消时都会触发。然而，考虑到手势识别器的简单性和强大功能，除了在特定情况下需要更精细的触摸交互的自定义用户界面外，它们几乎总是首选的方法。对于这些情况，请查看`UIResponder`对象可用的触摸事件。

# 我们学到了什么

在本章中，我们看到了 Android 和 iOS 中监听和接收用户输入的相似之处和差异。用户输入可以是简单的触摸、复杂手势，或者是屏幕上和外部键盘的输入。

+   两个平台都有类似的机制来监听和响应简单的触摸事件。

+   Android 和 iOS 都可以从各种来源接收文本输入，但由于接收此类输入的模式略显复杂，iOS 需要一些辅助。

+   操作系统中内置了检测和响应复杂手势的方法，但两个平台都有不常用于另一个平台的手势。

触摸输入使得 Android 和 iOS 设备如此直观和亲密。了解如何处理和构建能够接收输入的应用程序是非常重要的。

在下一章中，我们将更深入地探讨那些对象和模式，它们并非像前面讨论的直接面向用户。让我们开始吧！

^(1) 来自 [Android 开发者文档](https://oreil.ly/HvEKV)
