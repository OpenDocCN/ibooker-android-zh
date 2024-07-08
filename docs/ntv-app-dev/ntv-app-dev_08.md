# 第七章：持久性

数据的持久性允许应用程序形成一个看似是内存的东西。在移动软件开发中有许多实现这一点的方法，但最常见的是通过使用关系数据库。从一开始，Android 和 iOS 就具有连接、读取和写入数据库的能力。

这样做的结果是，用户在应用程序中的会话不再是短暂的存在，而是与时间点相关联的存在。数据可以被存储，信息可以被保存，状态可以被恢复。通过非常不同的架构和方法，Android 和 iOS 都具有一组共享的功能，以一种使其足够相似以便进行并行讨论的方式来驱动它们。

# 任务

在本章中，您将学习：

1.  建立数据库连接。

1.  创建数据库表或持久对象。

1.  将数据写入该表或持久对象。

1.  从该表或持久对象中读取数据。

# Android

在 Android 中，框架提供的数据库管理系统是 SQLite。确切的版本取决于 Android 操作系统的级别，但在撰写本文时，它的版本范围从 Android 1 的 3.4 到 Android 27 的 3.19。查看[Android 的开发者文档](https://oreil.ly/lQegA)获取最新信息。

SQLite 是一种基于 SQL 的关系型数据库管理系统（DBMS），非常类似于 MySQL 和 PostgreSQL。在数据类型、函数和实现细节（如`ALTER TABLE`）方面存在差异，但它们仍然非常相似，如果您有任何 SQL 数据库的经验，您将很快熟悉 SQLite。

## 建立数据库连接。

在 Android 中（但不是传统的 Java），您将使用的主要类是`SqliteOpenHelper`，但它是`abstract`的，因此*必须是一个子类*。这里是一个简化的示例：

`onUpgrade`方法在更改数据库架构时添加或迁移数据非常有用。当您的数据库版本号（助手构造函数的第四个参数）发生变化时，此方法仅运行一次。实际上，我们应该说“增加”而不是“改变”，因为在设备上运行低于存在的数据库版本将立即崩溃。

请注意，约定是使用公共常量作为表和列名称；这将使在其他类中从表中读取或写入数据变得更容易。

要从此帮助类获取数据库实例，我们调用`getWritableDatabase`或`getReadableDatabase`方法，取决于我们的需求。可写的执行所有可读的操作，因此如果您计划进行读取和写入操作，请使用前者。唯一需要可读版本的情况是确保在执行操作时底层数据不会发生更改。

这是如何从您的帮助类中获取可写数据库实例的方法：

从那时起，你就可以访问一个 `SqliteDatabase` 实例，它具有用于常见操作如查询和插入的 API，以及用于执行任意 SQL 字符串的更广泛的 API，如 `execSql`。

## 创建数据库表或持久对象

在子类中，你将有机会重写 `onCreate` 和 `onUpgrade` 方法；现在让我们专注于前者，它将在帮助程序第一次创建时触发一次（一般来说，这将在帮助程序第一次实例化时发生，并且是一个持久值，因此这将在应用程序第一次安装后不久发生）。后续启动不会触发 `onCreate`。

`onCreate` 是创建初始模式的绝佳机会。例如，假设我们想要创建一个只有一个表（暂时）`USAGE_EVENTS` 的数据库，其中包含一个标识符、一个事件名称和事件发生时的时间戳。我们将使用标准 SQL 在我们的帮助程序的 `onCreate` 方法中创建数据库；让我们扩展之前显示的简单示例，但实际上我们只是添加一些 `String` 常量来组成一个 SQL `create table` 语句并在 `onCreate` 中执行它：

## 将数据写入该表或持久对象

框架提供了 `SQLiteDatabase` API，正如我们已经看到的，它完全能够执行任意的 SQL 命令，但也提供了用于 CRUD 函数的方法，包括插入。此外，API 支持数据库事务，我们可以利用它来额外控制数据库写入。在以下简单示例中，可能看起来事务对我们没有太多帮助，但实际上它将捕获错误并给我们一个机会来对其做出反应；此外，也许更关键的是，如果插入操作失败——或者如果我们在 `try` 块中添加更多失败的数据库操作——事务将 *不会* 被设置为成功，并且会悄悄地回滚 `try` 部分的所有语句块。

尽管这个示例看起来很简单，但它展示了你可能想要用于任何或所有数据库事务的模式——只需在 `try` 块中添加更多操作，并在 `catch` 块中执行任何额外操作。（记住，抛出一个 `Exception` 将导致事务隐式回滚；没有显式的“回滚”方法。）这种模式适用于更复杂的修改操作：

## 从该表或持久对象中读取数据

`SqliteDatabase` 方法 `query` 是从数据库中获取信息的最直接方式。话虽如此，你可能会想要使用的签名需要八个参数！让我们假装一切都很好，直接开始吧。

这是最基本的版本（只有七个参数！）。这将从表中获取所有内容——所有行填充了所有列——并返回一个 `Cursor` 实例来浏览该数据集。这实际上是 `"SELECT * FROM EVENTS"`：

让我们来看一个更现实的查询：

在大多数 SQL 语言中，上述内容实际上是`SELECT ID FROM EVENTS WHERE NAME = \"Request Data\" LIMIT 1`。占位符是一个常见的概念，可以帮助保持一致性，并提供一些防止 SQL 注入的识别。

现在我们成功执行了查询，那么我们的数据在哪里？它在一个内存数据集中，我们可以使用`Cursor` API 访问；`query`方法返回一个`Cursor`实例。这里是一个例子：

您可以看到我们使用游标通过其索引访问每一列，但也有一个`Cursor`方法通过名称查找该索引的`getColumnByIndex`，因此您可能会这样做：

就是这样！还有大量其他可用的 API（和第三方对象关系映射器[ORM]），以及您可以使用原始 SQL 做的事情几乎是无限的。如果您正在开发一个经常使用数据库读取的应用程序，您应该查看`compileStatement`方法，该方法编译一个 SQL 字符串，并允许您在每次调用时重新定义值。这种方法非常快速，可以在处理大型数据集时真正提高感知性能。

在撰写本文时，Google 建议我们使用 Room（来自[Android 开发文档](https://oreil.ly/rtG9C)）：

> 尽管这些 API（SQLiteDatabase 等）功能强大，但它们相当低级，并且需要大量的时间和精力来使用：
> 
> +   没有原始 SQL 查询的编译时验证。随着数据图的变化，您需要手动更新受影响的 SQL 查询。这个过程可能耗时且容易出错。
> +   
> +   您需要使用大量样板代码在 SQL 查询和数据对象之间进行转换。
> +   
> 出于这些原因，我们强烈建议使用 Room 持久性库作为访问应用程序 SQLite 数据库中信息的抽象层。

### 为什么选择 SQLite？为什么不用 Room？为什么不用 realm？为什么不用<插入我喜欢的 ORM 名称>？

我们（作者）之所以选择 SQLite 作为我们的工作示例，有几个原因：首先，与 Android 生态系统中的其他模式和工具集一样，这比 Room 早了一段距离。其次，SQL 是开发人员从几乎任何语言都可能具有一些了解并能够在平台之间传递的东西。

# iOS

iOS 和 Android 之间数据持久性的主要区别与所使用的技术有关。在 Android 中，如您所见，SQLite 数据库是一种常见的方法，有直接与数据库进行交互的类和对象，通过数据库连接和 SQL。而 iOS 则将数据库层技术抽象化，有一个用于数据持久性的第一方对象图称为 Core Data。虽然 Core Data *可以*使用 SQLite 数据库（通常是这样）来持久化数据，但它可以完全在内存中作为临时存储，或输出到 XML 文件。然后，数据库成为开发人员可以大部分忽略的实现细节。

## 设置并连接到持久性层

尽管开发人员应该忘记 Core Data 对象图与原始 SQLite 数据库之间的差异，但两者之间存在等价物，我们将利用这些等价物来指导本章关于数据持久化的内容，以与 Android 进行比较和对比。建立数据库连接 *并不* 是您需要使用 Core Data 进行的操作；原始的 SQL 查询和连接由 Core Data 自身处理。然而，设置的一个等效部分是启动 Core Data 的“堆栈”。

### 设置 Core Data 堆栈

在新项目或现有项目中开始使用 Core Data 并不太困难，但首先需要明确 Core Data 到底是什么。让我们从持久化层开始。对于我们的目的，我们假设我们处理的是一个以 SQLite 数据库为根的 Core Data 堆栈。这可以说是 Core Data 开发中最常见的方法。

在 SQLite 数据库和持久存储之上是一个称为 `NSPersistentStoreCoordinator` 的对象。此对象处理数据库、托管对象模型和托管对象上下文之间的通信。它使用定义的托管对象模型，将这些对象和关系转换为数据库表和 SQL。它还优化了大部分交付的对象，并且真正是内置于 Core Data 中的许多魔法和优化背后的“大脑”。您可以将其视为 SQLite 和应用程序其余部分之间的交通协调员。

每个项目都使用一个 *.xcdatamodeld* 文件定义托管对象模型，或者 `NSManagedObjectModel`。这是持久化存储协调器用于确定所有结构的方式。开发人员使用此文件添加新的实体、关系和属性；这本质上是创建和定义项目中的数据模型的方式。作为其一部分创建的对象称为“托管对象”，它们继承自 `NSManagedObject`，这是 Core Data 框架的一部分。在此级别及我们需要讨论的下一个类别中，大多数与维护和处理 Core Data 相关的工作都是在此完成：托管对象上下文。

您可以将托管对象上下文，或者 `NSManagedObjectContext`，看作是一种草稿本，您可以在其中更改数据模型对象实例中包含的数据，然后将其持久化到数据库中。托管对象上下文是您将用于为 Core Data 提供对象创建和更新上下文的对象。在 Core Data 应用程序中通常有两种类型的上下文，视图上下文（在主线程上运行）和后台或私有上下文，用于保存对象，有效地将它们从 `NSManagedObject` 实例翻译成数据库表行 `INSERT` 和 `UPDATE` 命令。

为了使用核心数据，这个堆栈必须在需要持久性之前设置并正常运行。通常，这意味着当应用程序启动时。事实上，大多数设置都是在应用程序委托内的`application(_:didFinishLaunchingWithOptions:)`方法中完成的。

所有这些设置过去都很繁琐，但现在核心数据有一个方便的类来初始化我们的核心数据堆栈，称为`NSPersistentContainer`。持久容器处理了为托管对象模型文件所需的初始化，并具有一个方便的方法，可以异步加载持久存储。以下是它的使用示例：

```
let persistentContainer = NSPersistentContainer(name: "MyModel")
persistentContainer.loadPersistentStores { (description, error) in
    // Completion handler
}
```

这将创建一个针对名为“MyModel”的托管对象模型的容器，并将其存储在名为`persistentContainer`的变量中。下一行加载持久存储，最终在我们的示例中是 SQLite 数据库，从磁盘加载，并使用完成处理程序检查任何错误并在之后执行代码。如果将此代码放在应用程序委托的启动代码中，应用程序将在应用程序包中查找名为*MyModel.xcdatamodeld*的托管对象模型文件，并使用它来创建或加载名为*MyModel.sqlite*的 SQLite 数据库文件，存储在*Application Support*目录中。

###### 注意

您可以自定义数据库文件存放的位置。历史上*文档*目录一直被用作存储 SQLite 数据库的地方。您的需求会有所不同，但您可以在创建`NSPersistentContainer`并将其添加到`persistentStoreDescriptions`后添加`NSPersistentStoreDescription`来实现这一点。还有其他选项可用于描述持久容器，但这些选项超出了本章的范围。

在完成闭包内部，您应该检查错误，然后更新任何可能正在等待持久存储加载的用户界面。一个更完整的持久容器加载示例可能如下所示：

```
let persistentContainer = NSPersistentContainer(name: "MyModel")
persistentContainer.loadPersistentStores { (description, error) in
    guard let error = error else {
        // Display error to user and/or attempt to do this again
        return
    }

    // Update the user interface and/or start using Core Data
}
```

现在我们的核心数据堆栈已经运行起来了，让我们看看如何定义我们的对象。

## 定义并创建数据库表或持久对象

我们的模型对象是在 Xcode 中创建的托管对象模型文件中定义的。要添加到现有项目中，请转到文件 > 新建 > 文件，并在核心数据部分下选择数据模型。文件的名称是您在初始化期间将传递给持久容器的名称。

在 Xcode 中，您可以通过单击“添加实体”按钮来编辑此文件，以创建一个新的托管对象。在实体描述中，将有属性、关系和获取的属性的区域。您可以向实体添加单个属性，更改实体的名称，并在此编辑器内为此实体分配一个支持类。

个别属性被添加为属性。这些对应于在托管对象模型中为每个实体生成（或手动提供）的原始数据类型，如 Swift 类中的`String`和`Int`。重要的是要记住，托管对象模型仅定义实体的描述，实际上根本不与数据库交互。它只是提供了从持久化存储协调器可以理解的 Swift 代码的映射。

要添加新属性，请单击编辑器的属性区域下面的“+”按钮。如果你要在名为`MyEntity`的实体上添加一个名为`title`且类型为`String`的新属性，Xcode 在编译期间会为你自动生成一个隐藏在 Xcode 项目中的类。该类可能看起来像这样：

```
import Foundation
import CoreData

@objc(MyEntity)
public class MyEntity: NSManagedObject {

}

extension MyEntity {

    @nonobjc public class func fetchRequest() -> NSFetchRequest<MyEntity> {
        return NSFetchRequest<MyEntity>(entityName: "MyEntity")
    }

    @NSManaged public var title: String?

}
```

类本身继承自`NSManagedObject`，与我们的实体描述`MyEntity`同名。但实际上，它可以是任何我们想要的名字。这是一个重要的区别：支持模型对象的类可以与实体描述分开命名。还要注意，`@NSManaged`属性表示 Core Data 管理该属性的存储。这意味着这不是一个普通的存储属性。事实上，每当写入属性时，它实际上是由`setValue:`支持的。

这种灵活性使我们能够向托管对象添加不写入数据库的属性。从根本上讲，这意味着我们可以做一些事情，比如为根本不必与数据库交互的托管对象添加作为便捷属性的计算属性。例如，我们可以定义一个名为`Person`的托管对象，它在 Core Data 中有`firstName`和`lastName`属性，但提供了一个`fullName`属性，它只是组合了这些属性，而不在数据库中重复数据：

```
public class Person: NSManagedObject {
	@NSManaged public var firstName: String
	@NSManaged public var lastName: String
	public var fullName: String {
		return "\(firstName) \(lastName)"
	}
}
```

以我们的`Person`托管对象为例（在托管对象模型中创建了相应的实体描述之后），让我们继续看看如何持久化一些数据。

## 写入和持久化数据到 SQLite

关于在 Core Data 中写入数据的第一点非常重要的事情是，你*不希望*在主线程上进行写入。事实上，我们的持久化容器中实际上有一个方便的方法，让我们可以快速轻松地切换到后台线程来写入一些数据，如下所示：

```
persistentContainer.performBackgroundTask { (managedObjectContext) in
	// Some operation...
}
```

现在，正在执行的操作可能是读取。它也可能是写入。我们现在将专注于写入这一方面。实际上，让我们看看我们之前的`Person`托管对象，看看我们如何创建一个新对象并将其添加到数据存储中。

在 Core Data 中创建和更新对象时，所有操作首先在托管对象上下文中进行。我们可以在这个上下文中随意修改对象，而不会影响数据库，直到我们明确保存更改为止。因此，我们还需要一个上下文来创建新对象。以下是如何创建一个新的`Person`实例并为其分配一些属性的示例：

```
persistentContainer.performBackgroundTask { (managedObjectContext) in
    let person = Person(context: managedObjectContext)
    person.firstName = "Mike"
    person.lastName = "Dunn"
}
```

实际上，这很简单。我们在传递给我们的闭包的托管对象上下文中创建了一个名为`person`的新`Person`。然后，我们为`firstName`和`lastName`赋值。

然而，这个对象还没有被持久化。事实上，这个对象将存在，直到闭包完成，然后它将与托管对象上下文一起被销毁。幸运的是，持久化数据也并不困难。只需一行代码，加上一些错误检查的`do`和`try`块，就像这样：

```
persistentContainer.performBackgroundTask { (managedObjectContext) in
    let person = Person(context: managedObjectContext)
    person.firstName = "Mike"
    person.lastName = "Dunn"

    // Save the context
    do {
        try managedObjectContext.save()
    } catch {
        print("Error during save. \(error)")
    }
}
```

你会注意到，我们只保存*上下文*，而不是对象本身。这个操作保存了所有在该上下文中的更改。所以，如果我们创建了一百万个`Person`，它们都会同时保存。如果我们创建了不同的对象类型，并在其他对象中更改了属性，它们也将同时持久化。保存整个上下文而不是单个对象可能会导致性能问题。因此，建议使用大数据集进行分析和测试，以防止出现这类问题。

要更新一个对象，你只需在托管对象上下文中获取它，更新所需的属性，然后保存上下文即可。让我们看看如何从已保存的 SQLite 存储中获取和读取数据。

## 从 SQLite 读取数据

尽管本节标题称“从 SQLite 读取数据*”，但实际上我们将通过托管对象上下文读取数据。如果*选择*返回到 SQLite 存储，它将联系持久化存储协调器寻求帮助。如我们稍后展示的使用`NSFetchRequest`将导致访问持久存储，可能会成为一个非常缓慢的路径。然而，托管对象上下文通常在内存中缓存了大量数据以提升性能，并且在获取后获取的数据可以像普通对象一样被利用和操作。

要查找一个对象，我们必须创建一个获取请求，并像这样对托管对象上下文执行它：

```
let managedObjectContext = persistentContainer.viewContext
let fetchRequest = NSFetchRequest<Person>(entityName: "Person")
do {
	let persons = try managedObjectContext.fetch(fetchRequest)
} catch {
	fatalError("Fetch could not be completed.")
}
```

让我们一起看看这里发生了什么。

首先，我们通过`viewContext`属性从持久化容器中获取一个只读的托管对象上下文。这允许我们查看对象，但不能保存更改。接下来，我们为托管对象模型中名为`Person`的实体创建一个类型为`Person`的获取请求。然后，我们使用该获取请求从我们的托管对象上下文中进行`fetch(_:)`操作。这个调用可能会`throw`异常，因此它被包裹在一个`do`和`catch`块中。如果无法完成获取操作，我们将抛出一个致命错误，错误消息为“无法完成获取操作”。

此时，我们应该有一个填充了的`persons`对象，我们可以用它来迭代读取我们已经持久化的所有人的列表。

Core Data 还有很多功能。它是每个 iOS 开发者工具包中非常强大的工具，可以用来创建性能优化的对象图，以便轻松持久化数据。它的最大缺点包括初始时较为冗长的文本和大量的设置工作。然而，项目开始时的一些准备工作可能会避免您编写复杂且脆弱的 SQL 语句，而是获得一些属性名称和值的编译时检查。

###### 提示

当为表视图获取结果时，可以使用名为`NSFetchedResultsController`的对象，它接受结果和一个`NSFetchRequest`，并在数据变化时自动更新表格。这使得与 Core Data 的交互更容易，例如对于`UITableView`和`UICollectionView`。

# 我们学到了什么

Android 和 iOS 在许多方面都很相似，正如您在本书中已经看到并将继续看到的那样。然而，本章确实表明，有时它们可能大相径庭。

+   Android 的原始 SQLite 交互方法提供了在 iOS 上使用 Core Data 时所不具备的架构简单性，允许开发人员利用他们可能已经具备的服务器端语言知识跳入其中。

+   Core Data 不是一个数据库。它是一个复杂的对象图框架，*非常*强大，并且在某种程度上与 SQLite 本身解耦，试图隐藏其复杂性。

+   这两种方法都允许数据的持久化，并且在应用的未来版本中需要进行充分的规划和维护。

一旦开始讨论对象持久化，下一个逻辑步骤就是并发，这恰好是下一章要讨论的内容。让我们排队并立即前往那里！
