# 第一章：Kotlin 基础

Kotlin 是由俄罗斯圣彼得堡的 JetBrains 团队创建的。JetBrains 以 IntelliJ Idea IDE 而闻名，后者是 Android Studio 的基础。现在 Kotlin 已经广泛应用于多种操作系统环境中。自从 Google 宣布在 Android 上支持 Kotlin 已经将近五年了。根据 [Android Developers Blog](https://oreil.ly/PrfQm)，截至 2021 年，Google Play 商店中超过 120 万款应用程序使用 Kotlin，其中包括前一千名应用程序中的 80%。

如果你拿起这本书，我们假设你已经是一名 Android 开发者，并且对 Java 很熟悉。

Kotlin 被设计与 Java 互操作。甚至它的名字，取自圣彼得堡附近的一个岛屿，是对 Java 的一个机智影射，Java 是印度尼西亚的一个岛屿。尽管 Kotlin 支持其他平台（iOS、WebAssembly、Kotlin/JS 等），Kotlin 被广泛使用的关键是其对 Java 虚拟机（JVM）的支持。由于 Kotlin 可以编译为 Java 字节码，它可以在任何支持 JVM 运行的地方运行。

本章的大部分讨论将 Kotlin 与 Java 进行比较。然而，重要的是要理解 Kotlin 不仅仅是加了一些新功能的 Java。Kotlin 是一种全新的不同的语言，与 Java 的联系几乎与它与 Scala、Swift 和 C# 的联系一样紧密。它有自己的风格和习惯用法。虽然可以以 Java 的思维写 Kotlin，但以 Kotlin 的习惯用法思考将展示语言的全部威力。

我们意识到可能有一些 Android 开发者长期以来一直使用 Kotlin，从未写过任何 Java。如果你是这样的人，你可能可以略过本章以及其对 Kotlin 语言的回顾。然而，即使你对该语言相当熟悉，这也可能是一个提醒你某些细节的好机会。

本章并不意味着要对 Kotlin 进行全面介绍，所以如果你对 Kotlin 完全陌生，我们推荐优秀的 *Kotlin 实战*。^(1) 相反，本章是对 Kotlin 基础的回顾：类型系统、变量、函数和类。即使你不是 Kotlin 语言专家，它也应为你理解本书的其余内容提供足够的基础。

和所有静态类型语言一样，Kotlin 的类型系统是 Kotlin 用来描述自身的元语言。因为这是讨论 Kotlin 的一个重要方面，我们将从回顾它开始。

# Kotlin 类型系统

像 Java 一样，Kotlin 是一种静态类型语言。Kotlin 编译器了解程序操作的每个实体的类型。它可以推断^(2) 这些实体，并使用这些推断识别代码与之相悖时将会发生的错误。类型检查允许编译器捕捉和标记整个大类编程错误。本节重点介绍 Kotlin 类型系统的一些最有趣的特性，包括 `Unit` 类型、函数类型、空安全和泛型。

## 原始类型

Java 和 Kotlin 类型系统之间最明显的区别是，Kotlin 没有 *原始类型* 的概念。

Java 有 `int`、`float`、`boolean` 等类型。这些类型的特殊之处在于它们不继承 Java 的基本类型 `Object`。例如，语句 `int n = null;` 在 Java 中是非法的。`List<int> integers;` 也是如此。为了减轻这种不一致性，每个 Java 原始类型都有一个 *装箱类型* 等价物。例如，`Integer` 是 `int` 的类比；`Boolean` 是 `boolean` 的类比，依此类推。原始类型和装箱类型之间的区别几乎已经消失，因为自 Java 5 以来，Java 编译器自动在原始类型和装箱类型之间转换。现在可以合法地说 `Integer i = 1`。

Kotlin 的类型系统中没有原始类型，它的单一基础类型`Any`类似于 Java 的`Object`，是整个 Kotlin 类型层次结构的根。

###### 注意

Kotlin 对简单类型的内部表示与其类型系统无关。Kotlin 编译器具有足够的信息来以与任何其他语言一样的效率表示 32 位整数。因此，写 `val i: Int = 1` 可能会使用原始类型或装箱类型，这取决于在代码中如何使用变量 `i`。尽可能地，Kotlin 编译器会使用原始类型。

## 空安全

Java 和 Kotlin 的第二个主要区别在于 *可空性* 是 Kotlin 类型系统的一部分。可空类型通过其名称末尾的问号来区分其非可空模拟；例如，`String` 和 `String?`，`Person` 和 `Person?`。Kotlin 编译器允许将 `null` 赋给可空类型：`var name: String? = null`。但它不允许 `var name: String = null`（因为 `String` 不是可空类型）。

`Any` 是 Kotlin 类型系统的根，就像 Java 中的 `Object`。然而，有一个显著的区别：`Any` 是所有非空类的基类，而 `Any?` 是所有可空类的基类。这是 *空安全* 的基础。换句话说，可以将 Kotlin 的类型系统视为两个相同的类型树：所有非空类型都是 `Any` 的子类型，所有可空类型都是 `Any?` 的子类型。

变量必须初始化。变量没有默认值。例如，以下代码会生成编译错误：

```
val name: String // error! Nonnullable types must be initialized!
```

正如前面所述，Kotlin 编译器使用类型信息进行推断。通常编译器可以从它已有的信息中推断出标识符的类型。这个过程称为*类型推断*。当编译器可以推断出类型时，开发人员无需指定它。例如，赋值`var name = "Jerry"`是完全合法的，尽管并未指定变量`name`的类型。编译器可以推断变量`name`必须是`String`，因为它被赋予了值`"Jerry"`（这是一个`String`）。

推断类型有时会让人感到意外。这段代码将生成编译器错误：

```
var name = "Jerry"
name = null
```

编译器为变量`name`推断了类型`String`，而不是类型`String?`。因为`String`不是可空类型，试图将`null`赋给它是非法的。

需要注意的是，*可空*类型与其*非空*对应类型并不相同。理所当然地，可空类型表现为相关非空类型的超类型。例如，下面的代码没有任何问题地编译，因为`String`是`String?`：

```
val name = Jerry
fun showNameLength(name: String?) { // Function accepts a nullable parameter
     // ...
}

showNameLength(name)
```

另一方面，以下代码根本无法编译，因为`String?` *不是* `String`：

```
val name: String? = null
fun showNameLength(name: String) { // This function only accepts non-nulls
    println(name.length)
}

showNameLength(name)               // error! Won't compile because "name"
                                   // can be null
```

仅仅改变参数的类型并不能完全解决问题：

```
val name: String? = null
fun showNameLength(name: String?) { // This function now accepts nulls
    println(name.length)            // error!
}

showNameLength(name)                // Compiles
```

这段代码会因为错误`Only safe (?.) or non-null asserted (!!.) calls are allowed on a nullable receiver of type String?`而失败。

Kotlin 要求对可空变量进行安全处理，以避免生成空指针异常。为了使代码编译通过，必须正确处理`name`为`null`的情况：

```
val name: String? = null
fun showNameLength(name: String?) {
    println(if (name == null) 0 else name.length)
    // we will use an even nicer syntax shortly
}
```

Kotlin 有特殊的操作符，`?.` 和 `?:`，可以简化与可空实体的工作：

```
val name: String? = null
fun showNameLength(name: String?) {
    println(name?.length ?: 0)
}
```

在前面的示例中，当`name`不为`null`时，`name?.length`的值与`name.length`的值相同。然而，当`name`为`null`时，`name?.length`的值为`null`。该表达式不会抛出空指针异常。因此，在前面的示例中，第一个操作符，安全操作符`?.`，在语法上等同于：

```
if (name == null) null else name.length
```

第二个操作符，*Elvis 操作符* `?:`，如果左侧表达式非空，则返回左侧表达式的值；否则返回右侧表达式的值。请注意，仅当左侧表达式为 null 时才会评估右侧表达式。

相当于：

```
if (name?.length == null) 0 else name.length
```

## 单元类型

在 Kotlin 中，*每个*东西都有一个值。一旦你理解了这一点，就不难想象即使一个方法没有明确返回任何内容，它也有一个默认值。那个默认值被命名为`Unit`。`Unit`恰好是一个对象的名称，如果它们没有任何其他值，就会使用这个值。`Unit`对象的类型方便地被命名为`Unit`。

`Unit`的整个概念对于习惯于表达式有值和语句无值之间区别的 Java 开发人员可能显得奇怪。

Java 的条件语句是区分*语句*和*表达式*的一个很好的例子，因为它同时具有两者！在 Java 中，你可以说：

```
if (maybe) doThis() else doThat();
```

然而，你不能说：

```
int n = if (maybe) doThis() else doThat();
```

语句，比如`if`语句，不返回任何值。你不能将`if`语句的值赋给一个变量，因为`if`语句不返回任何东西。对于循环语句、case 语句等也是如此。

然而，Java 的`if`语句有一个类似物，称为*三元表达式*。由于它是一个表达式，它返回一个值，并且可以被赋值。这在 Java 中是合法的（只要`doThis`和`doThat`都返回整数）：

```
int n = (maybe) ? doThis() : doThat();
```

在 Kotlin 中，没有必要有两个条件语句，因为`if`是一个表达式并返回一个值。例如，这是完全合法的：

```
val n = if (maybe) doThis() else doThat()
```

在 Java 中，返回类型为`void`的方法类似于一个语句。实际上，这有点不准确，因为`void`不是一种类型。它是 Java 语言中的保留字，表示该方法不返回任何值。当 Java 引入泛型时，引入了类型`Void`来填补这个空白（有意思！）。然而，“空白”的两种表现形式，关键字和类型，却令人困惑且不一致：返回类型为`Void`的函数必须明确返回`null`。

Kotlin 更加一致：所有函数都返回一个值并具有一个类型。如果函数的代码没有明确返回一个值，那么函数的值为`Unit`。

## 函数类型

Kotlin 的类型系统支持*函数类型*。例如，下面的代码定义了一个变量`func`，其值是一个函数，即 lambda 表达式`{ x -> x.pow(2.0) }`：

```
val func: (Double) -> Double = { x -> x.pow(2.0) }
```

由于`func`是一个接受一个`Double`类型参数并返回`Double`类型的函数，它的类型是`(Double) -> Double`。

在上一个例子中，我们显式指定了`func`的类型。然而，Kotlin 编译器可以从赋给它的值中推断出`func`变量的类型的许多信息。它知道返回类型，因为它知道`pow`的类型。然而，它没有足够的信息来猜测参数`x`的类型。如果我们提供了它，我们可以省略变量的类型说明符：

```
val func = { x: Double -> x.pow(2.0)}
```

###### 注意

Java 的类型系统不能描述一个函数类型——没有办法在类的上下文之外谈论函数。在 Java 中，要做类似前面例子的事情，我们可以使用函数类型`Function`，像这样：

```
Function<Double, Double> func
    = x -> Math.pow(x, 2.0);

func.apply(256.0);
```

变量`func`被赋予了一个类型为`Function`的匿名实例，其方法`apply`是给定的 lambda 表达式。

由于函数类型的存在，函数可以接受其他函数作为参数或将它们作为值返回。我们称这些为*高阶函数*。考虑一个 Kotlin 类型的模板：`(A, B) -> C`。它描述了一个接受两个参数的函数，类型分别为 `A` 和 `B`（无论这些类型是什么），并返回类型为 `C` 的值。因为 Kotlin 的类型语言可以描述函数，`A`、`B` 和 `C` 都可以是函数本身。

如果听起来有些抽象，那是因为确实如此。让我们更具体地说明。对于模板中的 `A`，让我们替换为 `(Double, Double)` `->` `Int`。这是一个接受两个 `Double` 并返回一个 `Int` 的函数。对于 `B`，让我们简单地替换为一个 `Double`。到目前为止，我们有 `((Double, Double)` `->` `Int,` `Double)` `->` `C`。

最后，假设我们的新功能类型返回一个 `(Double)` `->` `Int`，一个接受一个 `Double` 参数并返回一个 `Int` 的函数。以下代码显示了我们假想函数的完整签名：

```
fun getCurve(
    surface: (Double, Double) -> Int,
    x: Double
): (Double) -> Int {
    return { y -> surface(x, y) }
}
```

我们刚刚描述了一个接受两个参数的函数类型。第一个是一个接受两个 `Double` 参数的函数 (`surface`)，返回一个 `Int`。第二个是一个 `Double` (`x`)。我们的 `getCurve` 函数返回一个函数，该函数接受一个 `Double` 参数 (`y`)，并返回一个 `Int`。

将函数作为参数传递给其他函数是函数式语言的重要支柱。使用高阶函数，可以减少代码冗余，而不必像在 Java 中那样创建新的类（子类化 `Runnable` 或 `Function` 接口）。明智地使用高阶函数可以提高代码的可读性。

## 泛型

像 Java 一样，Kotlin 的类型系统支持类型变量。例如：

```
fun <T> simplePair(x: T, y: T) = Pair(x, y)
```

此函数创建一个 Kotlin `Pair` 对象，其中两个元素必须是相同类型。根据这个定义，`simplePair("Hello", "Goodbye")` 和 `simplePair(4, 5)` 都是合法的，但 `simplePair("Hello", 5)` 不合法。

在 `simplePair` 定义中标记为 `T` 的通用类型是一个类型变量：它可以取 Kotlin 类型的值（在此示例中为 `String` 或 `Int`）。使用类型变量的函数（或类）称为*泛型*。

# 变量和函数

现在我们有 Kotlin 的类型语言支持，我们可以开始讨论 Kotlin 本身的语法。

在 Java 中，顶级的语法实体是类。所有变量和方法都是某个类的成员，而类是同名文件中的主要元素。

Kotlin 没有这种限制。如果你愿意的话，可以将整个程序放在一个文件中（请不要这样做）。你还可以在任何类之外定义变量和函数。

## 变量

声明变量有两种方法：使用关键字 `val` 和 `var`。关键字是必需的，是行中的第一件事，并引入了声明：

```
val ronDeeLay = "the night time"
```

关键字 `val` 创建的变量是只读的：它不能被重新赋值。但要小心！你可能会认为 `val` 就像使用 `final` 关键字声明的 Java 变量。虽然相似，但并不完全相同！尽管不能被重新赋值，`val` 变量确实可以改变值！在 Kotlin 中，`val` 变量更像是 Java 类的字段，具有 getter 但没有 setter，如下面的代码所示：

```
val surprising: Double
    get() = Math.random()
```

每次访问 `surprising`，它都会返回不同的随机值。这是一个没有 *后备字段* 的属性的示例。我们将在本章后面讨论属性。另一方面，如果我们写了 `val rand = Random()`，那么 `rand` 的值不会改变，更像是 Java 中的 `final` 变量。

第二个关键字 `var` 创建的是一个熟悉的可变变量：就像一个小盒子，可以装入最后放入的东西。

在接下来的部分，我们将继续讨论 Kotlin 作为函数式语言的一个特性：*lambda*。

## Lambda

Kotlin 支持函数字面值：lambda。在 Kotlin 中，lambda 总是用花括号括起来。在花括号内，参数列表位于箭头 `->` 的左侧，执行 lambda 的值是箭头右侧的表达式，如下面的代码所示：

```
{ x: Int, y: Int -> x * y }
```

按照惯例，返回的值是 lambda 体中最后一个表达式的值。例如，下面代码中显示的函数的类型是 `(Int, Int) -> String`：

```
{ x: Int, y: Int -> x * y; "down on the corner" }
```

Kotlin 有一个非常有趣的特性，允许实际上扩展语言。当函数的最后一个参数是另一个函数（该函数是高阶的）时，可以将作为参数传递的 lambda 表达式移出通常界定实际参数列表的括号，如下面的代码所示：

```
// The last argument, "callback", is a function
fun apiCall(param: Int, callback: () -> Unit)
```

这个函数通常会像这样使用：

```
apiCall(1, { println("I'm called back!")})
```

但由于我们提到的语言特性，它也可以像这样使用：

```
apiCall(1) {
   println("I'm called back!")
}
```

这样更好，不是吗？多亏了这个特性，您的代码可以更易读。这个特性的更高级用法是 `DSL`。^(3)

## 扩展函数

当您需要向现有类添加一个新方法，并且该类来自您不拥有源代码的依赖项时，您会怎么做？

在 Java 中，如果类不是 `final`，您可以对其进行子类化。有时这并不理想，因为这会增加项目的复杂性。如果类是 `final`，则可以在您自己的某个实用程序类内定义静态方法，如下面的代码所示：

```
class FileUtils {
    public static String getWordAtIndex(File file, int index) {
        /* Implementation hidden for brevity */
    }
}
```

在前面的示例中，我们定义了一个函数来获取文本文件中给定索引处的单词。在使用时，您可以写成 `String word = getWordAtIndex(file, 3)`，假设您导入了 `FileUtils.getWordAtIndex` 的静态方法。这很好，我们在 Java 中已经做了这些多年，它很有效。

在 Kotlin 中，还有一件事情可以做。您可以定义一个新的方法在一个类上，即使它不是该类的真正成员函数。因此，您并没有真正扩展类，但在使用时感觉像是向类添加了一个方法。这是如何可能的呢？通过定义一个*扩展函数*，如下所示：

```
// declared inside FileUtils.kt
fun File.getWordAtIndex(index: Int): String {
    val context = this.readText()  // 'this' corresponds to the file
    return context.split(' ').getOrElse(index) { "" }
}
```

在扩展函数的声明内部，`this`指的是接收类型的实例（这里是`File`）。您只能访问公共和内部属性和方法，因此`private`和`protected`字段是不可访问的，很快您就会理解为什么。

在使用时，您可以写成 `val word = file.getWordAtIndex(3)`。正如您所见，我们在`File`实例上调用`getWordAtIndex()`函数，就像`File`类有`getWordAtIndex()`成员函数一样。这使得使用位置更加表达和可读。我们不必为新的实用程序类命名：我们可以直接在源文件的根部声明扩展函数。

###### 注意

让我们来看看`getWordAtIndex`的反编译版本：

```
public class FileUtilsKt {
    public static String getWordAtIndex(
            File file, int index
    ) {
        /* Implementation hidden for brevity */
    }
}
```

编译时，我们的扩展函数生成的字节码相当于一个以`File`作为其第一个参数的静态方法。包围类`FileUtilsKt`的名称与源文件的名称（*FileUtils.kt*）相同，带有“kt”后缀。

这解释了为什么我们不能在扩展函数中访问私有字段：我们只是添加了一个以接收类型为参数的静态方法。

还有更多！对于类属性，您可以声明*扩展属性*。思想完全相同——您并没有真正扩展类，但可以使用点符号使新属性可访问，如下面的代码所示：

```
// The Rectangle class has width and height properties
val Rectangle.area: Double
    get() = width * height
```

请注意，这次我们使用了`val`（而不是`fun`）来声明扩展属性。您可以这样使用它：`val area = rectangle.area`。

扩展函数和扩展属性允许您扩展类的功能，使用漂亮的点表示法，同时仍然保持关注点的分离。您不会因特定需求而在现有类中添加特定的代码。

# 类

Kotlin 中的类，起初看起来很像 Java 中的类：`class`关键字，后面跟随定义类的块。不过 Kotlin 的一大杀手级特性是构造函数的语法以及在其中声明属性的能力。下面的代码显示了一个简单的`Point`类的定义以及几个用法：

```
class Point(val x: Int, var y: Int? = 3)

fun demo() {
    val pt1 = Point(4)
    assertEquals(3, pt1.y)
    pt1.y = 7
    val pt2 = Point(7, 7)
    assertEquals(pt2.y, pt1.y)
}
```

## 类初始化

请注意，在前面的代码中，`Point`的构造函数嵌入在类声明中。它被称为*主构造函数*。`Point`的主构造函数声明了两个类属性，`x`和`y`，都是整数。第一个`x`是只读的。第二个`y`是可变的和可空的，并且具有默认值 3。

注意`var`和`val`关键字非常重要！声明`class Point(x: Int, y: Int)`与之前的声明*非常*不同，因为它不声明任何成员属性。没有这些关键字，标识符`x`和`y`仅仅是构造函数的参数。例如，以下代码将会生成一个错误：

```
class Point(x: Int, y: Int?)

fun demo() {
    val pt = Point(4)
    pt.y = 7 // error!  Variable expected
}
```

在这个示例中，`Point`类只有一个构造函数，即在其声明中定义的构造函数。然而，在 Kotlin 中，类不仅限于这个单一的构造函数。您还可以定义辅助构造函数和初始化块，如下所示的`Segment`类的定义：

```
class Segment(val start: Point, val end: Point) {
    val length: Double = sqrt(
            (end.x - start.x).toDouble().pow(2.0)
                    + (end.y - start.y).toDouble().pow(2.0))

    init {
        println("Point starting at $start with length $length")
    }

    constructor(x1: Int, y1: Int, x2: Int, y2: Int) :
            this(Point(x1, y1), Point(x2, y2)) {
        println("Secondary constructor")
    }
}
```

这个示例中还有一些其他值得注意的事情。首先，请注意辅助构造函数必须在其声明中委托给主构造函数，即`: this(...)`。构造函数可能有一个代码块，但必须显式地首先委托给主构造函数。

或许更有趣的是在前述声明中代码的执行顺序。假设创建一个新的`Segment`，使用辅助构造函数，打印语句将以何种顺序出现？

好吧！让我们试一试看：

```
>>> val s = Segment(1, 2, 3, 4)

Point starting at Point(x=1, y=2) with length 2.8284271247461903
Secondary constructor
```

这非常有趣。`init`块在与辅助构造函数关联的代码块之前运行！另一方面，属性`length`和`start`已经用它们的构造函数提供的值初始化。这意味着主构造函数必须在`init`块之前运行。

实际上，Kotlin 保证了这个顺序：主构造函数（如果有的话）首先运行。在它完成之后，`init`块按声明顺序（从上到下）运行。如果使用辅助构造函数创建新实例，与该构造函数关联的代码块将是最后执行的内容。

## 属性

Kotlin 变量，在构造函数中使用`val`或`var`声明，或者在类的顶层声明，实际上定义了一个*属性*。在 Kotlin 中，属性类似于 Java 字段及其 getter（如果属性是只读的，用`val`定义），或者它的 getter 和 setter（如果用`var`定义）的组合。

Kotlin 支持自定义属性的访问器和修改器，并有特殊的语法来实现，如`Rectangle`类的定义所示：

```
class Rectangle(val l: Int, val w: Int) {
    val area: Int
        get() = l * w
}
```

属性`area`是*合成的*：它从长度和宽度的值计算得出。因为对`area`进行赋值是没有意义的，所以它是一个`val`，只读的，并且没有`set()`方法。

使用标准的“点”符号访问属性的值：

```
val rect = Rectangle(3, 4)
assertEquals(12, rect.area)
```

为了进一步探索自定义属性的 getter 和 setter，考虑一个具有经常使用的哈希码（可能实例保存在`Map`中）且计算代价很高的类。作为设计决策，您决定缓存哈希码，并在类属性值更改时设置它。第一次尝试可能看起来像这样：

```
// This code doesn't work (we'll see why)
class ExpensiveToHash(_summary: String) {

    var summary: String = _summary
        set(value) {
            summary = value    // unbounded recursion!!
            hashCode = computeHash()
        }

    //  other declarations here...
    var hashCode: Long = computeHash()

    private fun computeHash(): Long = ...
}
```

前面的代码将因为无限递归而失败：对 `summary` 的赋值是对 `summary.set()` 的调用！试图在其自身的 setter 内设置属性的值是行不通的。Kotlin 使用特殊标识符 `field` 来解决这个问题。以下显示了修正后的代码版本：

```
class ExpensiveToHash(_summary: String) {

    var summary: String = _summary
        set(value) {
            field = value
            hashCode = computeHash()
        }

    //  other declarations here...
    var hashCode: Long = computeHash()

    private fun computeHash(): Long = ...
}
```

标识符 `field` 仅在自定义 getter 和 setter 内具有特殊意义，其中它指的是包含属性状态的*后备字段*。

还要注意，前面的代码演示了使用构造函数提供的值初始化具有自定义 getter/setter 的属性的习惯用法。在构造函数参数列表中定义属性是非常方便的简写形式。然而，如果几个构造函数中的属性定义具有自定义 getter 和 setter，可能会使构造函数变得难以阅读。

当必须从构造函数初始化具有自定义 getter 和 setter 的属性时，该属性与其自定义 getter 和 setter 一起在类的主体中定义。属性使用构造函数中的一个参数进行初始化（在本例中为 `_summary`）。这再次说明了构造函数参数列表中 `val` 和 `var` 关键字的重要性。参数 `_summary` 只是一个参数，并不是类属性，因为它在声明时没有任何关键字。

## 延迟初始化属性

有时，在声明变量的地方无法获得其值。对于 Android 开发者来说，一个明显的例子是在 `Activity` 或 `Fragment` 中使用的 UI 小部件。直到 `onCreate` 或 `onCreateView` 方法运行时，变量才能被初始化，以便在整个活动中引用该小部件。例如，在本例中的 `button`：

```
class MyFragment: Fragment() {
    private var button: Button? = null // will provide actual value later
}
```

变量必须进行初始化。一个标准的技术，因为我们还不知道值，是将变量设为可空，并用 `null` 进行初始化。

在这种情况下，您应该首先问自己的问题是，是否真的有必要在此时此地定义这个变量。`button` 引用真的会在多个方法中使用吗，还是只在一个或两个特定位置使用？如果是后者，您可以完全消除类全局变量。

然而，使用可空类型的问题在于，每当在代码中使用 `button` 时，您都需要检查其是否为 null。例如：`button?.setOnClickListener { .. }`。如果像这样使用几个变量，您将会看到很多令人讨厌的问号！如果您习惯于 Java 和它的简单点表示法，这看起来可能特别凌乱。

您可能会问，为什么 Kotlin 不允许我使用非空类型来声明 `button`，当您*确信*在任何东西尝试访问它之前将对其进行初始化？难道没有一种方法可以放宽编译器对此 `button` 的初始化规则吗？

是可能的。您可以使用 `lateinit` 修饰符来实现这一点，如下面的代码所示：

```
class MyFragment: Fragment() {
    private lateinit var button: Button // will initialize later
}
```

因为变量声明为 `lateinit`，Kotlin 允许您声明变量而不为其分配值。该变量必须是可变的，即 `var`，因为根据定义，您将稍后为其分配一个值。很好——问题解决了，对吧？

我们这些作者，在开始使用 Kotlin 时确实考虑到了这一点。现在，我们倾向于仅在绝对必要时使用 `lateinit`，并使用可空值替代。为什么呢？

当您使用 `lateinit` 时，您在告诉编译器：“我现在没有值可以给你。但我保证以后会给你一个值。” 如果 Kotlin 编译器能说话，它会回答：“好的！你说你知道自己在做什么。如果出了问题，那就是你的责任。” 使用 `lateinit` 修饰符时，您会为该变量禁用 Kotlin 的空安全性。如果您忘记初始化变量或在初始化之前调用某些方法，则会收到 `UninitializedPropertyAccessException`，这与在 Java 中收到 `NullPointerException` 类似。

*每次* 我们在代码中使用 `lateinit`，最终都会受到影响。我们的代码可能在所有我们预见的情况下都可以工作。我们确信没有遗漏任何东西… 但我们错了。

当您声明一个变量为 `lateinit` 时，您正在做编译器无法证明的假设。当您或其他开发人员在之后重构代码时，您精心设计的代码可能会被破坏。测试可能会捕捉到错误。或者不会。^(4) 根据我们的经验，使用 `lateinit` 总是导致运行时崩溃。我们是如何解决这个问题的？通过使用可空类型。

当您使用可空类型而不是 `lateinit` 时，Kotlin 编译器会强制您在代码中检查可空性，确切地在可能为 null 的地方。在代码中添加几个问号绝对值得以获得更健壮的代码。

## 懒加载属性

在软件工程中，推迟创建和初始化对象直到实际需要时是一种常见模式。这种模式被称为 *懒加载初始化*，在 Android 上尤其常见，因为在应用启动期间分配大量对象会导致启动时间变长。Example 1-1 是 Java 中懒加载初始化的典型案例。

##### Example 1-1\. Java 懒加载初始化

```
class Lightweight {
    private Heavyweight heavy;

    public Heavyweight getHeavy() {
        if (heavy == null) {
            heavy = new Heavyweight();
        }
        return heavy;
    }
}
```

`heavy` 字段只有在首次调用 `lightweight.getHeavy()` 时，才会使用类 `Heavyweight` 的新实例进行初始化（假设这个实例创建代价高昂）。后续调用 `getHeavy()` 将返回缓存的实例。

在 Kotlin 中，懒加载初始化是语言的一部分。通过使用 `by lazy` 指令并提供一个初始化块，剩下的懒加载实例化是隐式的，如 Example 1-2 所示。

##### Example 1-2\. Kotlin 懒加载初始化

```
class Lightweight {
    val heavy by lazy { // Initialization block
        Heavyweight()
    }
}
```

我们将在下一节中更详细地解释这种语法。

###### 注意

注意，在示例 1-1 中的代码不是线程安全的。同时调用`Lightweight`的`getHeavy()`方法的多个线程可能会得到不同的`Heavyweight`实例。

默认情况下，示例 1-2 中的代码是线程安全的。对`Lightweight::getHeavy()`的调用将被同步，以确保每次只有一个线程在初始化块中。

可以使用`LazyThreadSafetyMode`来对延迟初始化块的并发访问进行精细化控制。

Kotlin 的延迟值在运行时不会被初始化。第一次引用属性`heavy`时，初始化块将被执行。

## 委托

Lazy 属性是 Kotlin 更一般的*委托*特性的一个例子。声明使用关键字`by`定义一个委托，该委托负责获取和设置属性的值。在 Java 中，可以通过例如将其参数传递给委托对象的方法调用来实现类似的功能。

因为 Kotlin 的延迟初始化特性是惯用 Kotlin 强大功能的一个极好例子，让我们花点时间来解析它。

在示例 1-2 中声明的第一部分是`val heavy`。我们知道，这是一个只读变量`heavy`的声明。接下来是关键字`by`，引入了一个委托。关键字`by`表示声明中的下一个标识符是一个表达式，该表达式将求值为负责`heavy`值的对象。

声明中的下一个事物是标识符`lazy`。Kotlin 期望一个表达式。原来`lazy`只是一个函数！它是一个接受单个 lambda 参数并返回对象的函数。它返回的对象是一个`Lazy<T>`，其中`T`是 lambda 返回的类型。

`Lazy<T>`的实现非常简单：第一次调用时运行 lambda 并缓存其值。随后的调用将返回缓存的值。

延迟委托只是*属性委托*的众多变体之一。使用关键字`by`，你也可以定义*可观察属性*（参见[Kotlin 委托属性的文档](https://oreil.ly/6lTab)）。在 Android 代码中，延迟委托是最常用的属性委托。

## 伴生对象

或许你想知道 Kotlin 是如何处理静态变量的。别担心；Kotlin 使用*伴生对象*。伴生对象是始终与 Kotlin 类相关联的*单例对象*。虽然不是必需的，但大多数情况下伴生对象的定义放在相关类的底部，如下所示：

```
class TimeExtensions {
    //  other code

    companion object {
        const val TAG = "TIME_EXTENSIONS"
    }
}
```

Companion objects 可以有名称，扩展类，并继承接口。在这个例子中，`TimeExtension`的伴生对象被命名为`StdTimeExtension`，并继承接口`Formatter`：

```
interface Formatter {
    val yearMonthDate: String
}

class TimeExtensions {
    //  other code

    companion object StdTimeExtension : Formatter {
        const val TAG = "TIME_EXTENSIONS"
        override val yearMonthDate = "yyyy-MM-d"
    }
}
```

当从包含它的类外部引用伴生对象的成员时，必须用包含类的名称限定引用：

```
val timeExtensionsTag = TimeExtensions.StdTimeExtension.TAG
```

当 Kotlin 加载相关类时，伴生对象会被初始化。

## 数据类

在 Java 中有一类非常常见的类，它们甚至有一个名字：*POJOs*，或者叫做普通的旧 Java 对象。其思想是它们是结构化数据的简单表示。它们是数据成员（字段）的集合，其中大多数有 getter 和 setter，还有少量其他方法：`equals`、`hashCode`和`toString`。这种类如此常见，以至于 Kotlin 将其作为语言的一部分。它们被称为*数据类*。

我们可以通过将其定义为数据类来改进`Point`类的定义：

```
data class Point(var x: Int, var y: Int? = 3)
```

使用`data`修饰符声明的这个类与原始的不使用修饰符声明的类有什么区别？让我们进行一个简单的实验，首先使用原始定义的`Point`（不带`data`修饰符）：

```
class Point(var x: Int, var y: Int? = 3)

fun main() {
    val p1 = Point(1)
    val p2 = Point(1)
    println("Points are equals: ${p1 == p2}")
}
```

这个小程序的输出将是`"Points are equals: false"`。这个也许意料之外的结果的原因是，Kotlin 将`p1 == p2`编译成了`p1.equals(p2)`。由于我们第一次定义的`Point`类没有覆盖`equals`方法，这就转变成了对`Point`基类`Any`中的`equals`方法的调用。`Any`的`equals`实现仅在对象与自身比较时返回`true`。

如果我们尝试使用新定义的`Point`作为数据类来做同样的事情，程序将会打印出`"Points are equals: true"`。新定义的行为符合预期，因为数据类自动包含了`equals`、`hashCode`和`toString`方法的覆盖。每个自动生成的方法都依赖于类的所有属性。

例如，`Point`的数据类版本包含了等效于这个的`equals`方法：

```
override fun equals(o: Any?): Boolean {
    // If it's not a Point, return false
    // Note that null is not a Point
    if (o !is Point) return false

    // If it's a Point, x and y should be the same
    return x == o.x && y == o.y
}
```

除了提供`equals`和`hashCode`的默认实现外，数据类还提供了`copy`方法。以下是其使用示例：

```
data class Point(var x: Int, var y: Int? = 3)
val p = Point(1)          // x = 1, y = 3
val copy = p.copy(y = 2)  // x = 1, y = 2
```

Kotlin 的数据类非常方便，用于频繁使用的习语。

在接下来的部分中，我们将研究另一种特殊类型的类：*枚举类*。

## 枚举类

还记得开发者曾经被建议说枚举在安卓上太昂贵了吗？幸运的是，现在没有人再建议那样做：尽情使用枚举类吧！

Kotlin 的枚举类与 Java 的枚举非常相似。它们创建一个不能被子类化的类，并且有一组固定的实例。与 Java 一样，枚举不能子类化其他类型，但可以实现接口，可以有构造函数、属性和方法。以下是一些简单的示例：

```
enum class GymActivity {
    BARRE, PILATES, YOGA, FLOOR, SPIN, WEIGHTS
}

enum class LENGTH(val value: Int) {
    TEN(10), TWENTY(20), THIRTY(30), SIXTY(60);
}
```

枚举与 Kotlin 的`when`表达式非常搭配。例如：

```
fun requiresEquipment(activity: GymActivity) = when (activity) {
    GymActivity.BARRE -> true
    GymActivity.PILATES -> true
    GymActivity.YOGA -> false
    GymActivity.FLOOR -> false
    GymActivity.SPIN -> true
    GymActivity.WEIGHTS -> true
}
```

当使用`when`表达式用于变量赋值，或者作为函数的表达式体时，必须是*穷尽的*。一个穷尽的`when`表达式是指覆盖了其参数的每一个可能值（在本例中为`activity`）。确保`when`表达式是穷尽的一种标准方法是包含一个`else`子句。`else`子句匹配参数的任何未显式列出的值。

在前面的例子中，为了是穷尽的，`when`表达式必须适应函数参数`activity`的每一个可能值。该参数是类型为`GymActivity`的枚举实例之一。因为枚举具有已知的一组实例，Kotlin 可以确定所有可能的值都被显式列出，并允许省略`else`子句。

像这样省略`else`子句有一个非常好的优势：如果我们向`GymActivity`枚举添加一个新值，我们的代码将突然无法编译。Kotlin 编译器检测到`when`表达式不再是穷尽的。几乎肯定，当你向枚举添加新的情况时，你希望知道所有需要适应新值的代码位置。不包含`else`情况的穷尽`when`表达式正是这样做的。

###### 注意

如果一个`when`语句不需要返回值（例如，`when`语句的值不是函数的返回值的值），会发生什么？

如果`when`语句未用作表达式，Kotlin 编译器不会强制其为穷尽的。但是，你会收到一个 lint 警告（在 Android Studio 中为黄色标志），告诉你建议对枚举的`when`表达式进行穷尽检查。

还有一个技巧可以强制 Kotlin 将任何`when`语句解释为表达式（因此必须是穷尽的）。在[示例 1-3](https://example.org/exhaustive_property_id)中定义的扩展函数强制`when`语句返回一个值，正如我们在[示例 1-4](https://example.org/when_statement_exhaustive_id)中看到的。因为它必须有一个值，Kotlin 将坚持它是穷尽的。

##### 示例 1-3\. 强制`when`表达式是穷尽的

```
val <T> T.exhaustive: T
    get() = this
```

##### 示例 1-4\. 检查穷尽的`when`

```
when (activity) {
    GymActivity.BARRE -> true
    GymActivity.PILATES -> true
}.exhaustive // error!  when expression is not exhaustive.
```

枚举是创建具有指定静态实例集的类的一种方法。Kotlin 提供了这种能力的一个有趣的泛化，即*封闭类*。

## 封闭类

考虑以下代码。它定义了一个单一类型`Result`，具有精确的两个子类型。`Success`包含一个值；`Failure`包含一个`Exception`：

```
interface Result
data class Success(val data: List<Int>) : Result
data class Failure(val error: Throwable?) : Result
```

注意，用`enum`无法实现这一点。枚举的所有值必须是相同类型的实例。然而，在这里，有两个不同类型作为`Result`的子类型。

我们可以创建两种类型中的任何一种的新实例：

```
fun getResult(): Result = try {
    Success(getDataOrExplode())
} catch (e: Exception) {
    Failure(e)
}
```

同样，`when`表达式是管理`Result`的一种方便方式：

```
fun processResult(result: Result): List<Int> = when (result) {
    is Success -> result.data
    is Failure -> listOf()
    else -> throw IllegalArgumentException("unknown result type")
}
```

我们不得不再次添加`else`分支，因为 Kotlin 编译器不知道`Success`和`Failure`是唯一的`Result`子类。在程序的某个地方，你可能会创建`Result`的另一个子类并添加另一个可能的情况。因此，编译器需要`else`分支。

封闭类对类型起到与枚举对实例的作用。它们允许你向编译器宣告某种基类型（此处为`Result`）有一组固定的已知子类型（在本例中为`Success`和`Failure`）。要做出这样的声明，请在声明中使用关键字`sealed`，如下代码所示：

```
sealed class Result
data class Success(val data: List<Int>) : Result()
data class Failure(val error: Throwable?) : Result()
```

因为`Result`是*sealed*的，Kotlin 编译器知道`Success`和`Failure`是唯一可能的子类。因此，我们可以再次从`when`表达式中移除`else`分支：

```
fun processResult(result: Result): List<Int> = when (result) {
    is Success -> result.data
    is Failure -> listOf()
}
```

# 可见性修饰符

在 Java 和 Kotlin 中，可见性修饰符决定了变量、类或方法的作用域。在 Java 中，有三种可见性修饰符：

`private`

引用仅对其定义所在的类及其外部类（如果定义在内部类中）可见。

`protected`

引用仅对其定义所在的类及其子类可见。此外，它们还可从同一包中的类中访问。

`public`

引用在任何地方可见。

Kotlin 也有这三种可见性修饰符。但是，存在一些细微的差异。在 Java 中，你只能在类成员声明中使用它们，而在 Kotlin 中，你可以在类成员*和*顶层声明中使用它们：

`private`

声明的可见性取决于其定义位置：

+   声明为`private`的类成员仅在定义它的*类*中可见。

+   顶层的`private`声明仅在定义它的*文件*中可见。

`protected`

受保护的声明仅在定义它们的类及其子类中可见。

`public`

引用与 Java 中一样，在任何地方可见。

除了这三种不同的可见性修饰符，Java 还有第四种*package-private*，使引用仅在同一包中的类中可见。当声明没有可见性修饰符时，即为包私有可见性，这是 Java 中的默认可见性。

Kotlin 没有这样的概念。^(5) 这可能会让人感到惊讶，因为 Java 开发者经常依赖于包私有可见性来隐藏同一模块内其他包的实现细节。在 Kotlin 中，包不用于可见性作用域——它们只是命名空间。因此，Kotlin 中的默认可见性与 Java 不同——它是*public*。

Kotlin 不支持包私有可见性对我们如何设计和组织代码产生了相当大的影响。为了确保声明（类、方法、顶层字段等）的完全封装，可以在同一个文件中将所有这些声明设为`private`。

有时将几个密切相关的类分割到不同的文件中是可以接受的。然而，这些类不能访问同一包中的兄弟类，除非它们是 `public` 或 `internal`。什么是 `internal`？它是 Kotlin 支持的第四种可见性修饰符，在包含 *模块* 内的任何地方都可见。^(6) 从模块的角度来看，`internal` 和 `public` 是相同的。然而，在将此模块用作库（例如，它是其他模块的依赖项）时，`internal` 就显得很有意义。事实上，`internal` 声明对导入您库的模块是不可见的。因此，`internal` 对于隐藏对外界的声明是很有用的。

###### 注意

`internal` 修饰符并不是用于模块内部的可见性范围，这是 Java 中包私有的作用。在 Kotlin 中这是不可能的。可以使用 `private` 修饰符稍微严格限制可见性。

# 摘要

表 1-1 强调了 Java 和 Kotlin 之间的一些关键差异。

表 1-1\. Java 和 Kotlin 特性的差异

| 功能 | Java | Kotlin |
| --- | --- | --- |
| 文件内容 | 单个文件包含单个顶级类。 | 单个文件可以包含任意数量的类、变量或函数。 |
| 变量 | 使用 `final` 使变量不可变；变量默认可变。定义在类级别。 | 使用 `val` 使变量只读，或使用 `var` 进行读/写操作。定义在类级别，或可以独立存在于类外。 |
| 类型推断 | 需要数据类型。`Date date = new Date();` | 数据类型可以推断，例如 `val date = Date()`，或显式定义，例如 `val date: Date = Date()`。 |
| 包装和拆箱类型 | 在 Java 中，像 `int` 这样的原始数据类型推荐用于更昂贵的操作，因为它们比 `Integer` 等包装类型更便宜。然而，在 Java 的包装类中，有许多有用的方法。 | Kotlin 没有原始类型。一切都是对象。当编译为 JVM 时，生成的字节码会自动进行拆箱操作，如果可能的话。 |
| 访问修饰符 | 公共和受保护的类、函数和变量可以被扩展和重写。 | 作为一种函数式语言，Kotlin 在尽可能的情况下鼓励不可变性。类和函数默认为 final。 |
| 多模块项目中的访问修饰符 | 默认访问是包私有的。 | Kotlin 没有包私有，而是默认公共访问。新的 `internal` 访问提供同一模块中的可见性。 |
| 函数 | 所有函数都是方法。 | Kotlin 具有函数类型。函数数据类型看起来像 `(param: String) -> Boolean`。 |
| 可空性 | 任何非基本类型对象可以为 null。 | 只有显式可空引用，类型后缀为 `?`，可以设置为 null：`val date: Date? = new Date()`。 |
| 静态与常量 | `static` 关键字将变量附加到类定义，而不是实例。 | 没有 `static` 关键字。使用私有 `const` 或 `companion` 对象。 |

恭喜，您刚刚完成了一章涵盖 Kotlin 的基本内容。在我们开始讨论如何将 Kotlin 应用于 Android 之前，我们需要讨论 Kotlin 的内置库：集合和数据转换。理解 Kotlin 中数据转换的基本功能将为理解 Kotlin 作为一种函数式语言所需的基础打下必要的基础。

^(1) Dmitry Jemerov 和 Svetlana Isakova。*Kotlin in Action*。Manning，2017 年。

^(2) Kotlin 官方称之为类型推断，它使用编译器的部分阶段（前端组件）在您在 IDE 中编写代码时进行类型检查。这是 IntelliJ 的一个插件！有趣的事实：IntelliJ 和 Kotlin 的整体都是由编译器插件构建的。

^(3) DSL 意味着*领域特定语言*。Kotlin 中构建的 DSL 示例包括 Kotlin Gradle DSL。

^(4) 您可以使用 `this::button.isInitialized` 检查 `latenit` `button` 属性是否已初始化。依赖开发人员在所有正确的位置添加此检查并不能解决潜在问题。

^(5) 至少在 Kotlin 1.5.20 版中。截至我们撰写这些文字时，Jetbrains 正在考虑向语言添加包私有可见性修饰符。

^(6) 模块是一组一起编译的 Kotlin 文件。
