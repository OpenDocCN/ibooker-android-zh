# 第十八章：建模我们的图书馆

到目前为止，我们在图书馆应用程序上已经取得了相当大的进展；我们已经为应用程序放置了一些基本的结构元素，我们正在一些屏幕之间过渡，并列出一些数据。不幸的是，这些数据是静态的、硬编码的数据——这不会让我们在很长时间内走得很远。为了继续进行应用程序的其余部分，我们需要查看我们的数据模型。现在注意它将使我们能够制作出更易于维护和更健壮的东西。

现在，静态数据本身并不是一件可怕的事情。事实上，在我们谈论到网络在第九章之前，我们将继续在本教程中使用“静态”数据。目前我们提供图书列表的方式存在的一个很大问题是它不太适用于应用程序内部的其他地方。

目前，我们仅在应用程序的目录视图中显示图书。然而，稍后我们将搜索图书、保存图书等。让我们找出一种更为适应未来的前进方式。

# 列表视图中的动态数据

我们的图书列表视图是开始寻找使数据显示更动态化的好地方。如果您还记得，图书列表当前作为主对象类型`Book`的临时属性`sampleData`存储，以便更容易找到和使用。

然而，随着我们向应用程序添加显示数据的更多视图，我们会发现这有些重复和繁琐。让我们通过在 UI 控制器和列表视图之间添加一层来使其更加健壮。

## Android

“数据源”的概念在 Android 中存在，并且在某些组件中（如下一代媒体播放器`ExoPlayer`）只是存在。此外，Android 还有一个“内容提供者”的概念，可以为您的应用程序管理手机上的联系人提供信息。但是，使用“适配器”模式的组件往往会将数据源保留为任意的。您可以拥有对象的编译列表，设备上的 XML 文件，在内存中的 JSON 字符串或通过网络与 REST Web 服务通信。

对于我们的第一个草稿，我们只是将所有对象添加到静态的`array`对象中。虽然这样做是可以的，但显然不容易维护——每次将书籍添加到我们的图书馆时，我们都必须更新该数据结构并发布一个新版本的应用程序，并希望所有用户都能勤奋地更新。长远来看，我们可能希望使用能够由图书管理员更新并偶尔调用适当服务端点（URL）更新我们的书籍列表的 REST Web 服务。我们肯定希望将数据持久化到设备上，并且如果我们希望能够使用广为人知和高度审查的 SQL 进行排序或过滤，我们可能希望将这些 Web 服务的 JSON 响应转换为数据库行。

现在，让我们简单地创建一个接口，提供给我们的`Adapter`所需的信息，并更新`Adapter`以适应这个契约，然后随着我们的需求和能力的发展进行更改。

让我们再次查看我们的`Adapter`，看看如何以可扩展、动态的方式满足它的需求：

嗯。看起来我们引用了几次我们静态的书籍数据源 —— 一次是在视图被回收时，我们想要更新行的视觉属性（目前是书籍的标题），另一次是确定我们数据源中书籍的总数，以便`RecyclerView`知道何时停止滚动。听起来很简单的组合：

现在让我们更新`Adapter`，使其接受任何实现该接口的类：

简单！如果我们想继续使用静态数组，我们可以创建一个简单的类来实现`BookDataSource`，但是非常容易使用硬编码的语料库：

你可能没有注意到，但是这个`BookDataSource`接口契约被所有`List<Book>`的实现隐式满足了 —— 你可以提供一个如下简单的数据源：

有了这种类型的数据源，你可以隐式地使用所有奇妙的`List` API，比如`add`和`addAll`、`set`、`remove`、`subList`等。

因此，虽然我们现在仍然使用硬编码的书籍列表，但除了用新的数据源更新`Adapter`之外，我们不必对我们的`RecyclerView`或`Adapter`进行任何更改 —— 也许是从 Web 服务器或本地数据库读取的数据源。太好了！即使使用与以前相同的数据，我们的列表视图现在也更加动态化。稍后在本章中，我们将进一步删除我们的静态`Book`数组。现在，让我们看看如何在 iOS 上实现同样的功能。

## iOS

就像我们在 Android 中使用`Adapter`模式一样，我们需要在控制器层和原始数据之间创建一些分离。我们可以通过添加一个新对象来直接与控制器进行接口交互来实现这一点。如果你还记得在`CatalogViewController`中，我们正在让视图控制器遵循`UITableViewDataSource`协议来为表视图提供数据。

这个方法一开始运行得很好，但是为什么不创建一个单独的对象直接为表视图提供数据呢？控制器可以管理该数据源对象，也可以管理表视图对象。这样就在控制器、视图（在这种情况下是表视图）和数据层之间建立了分离。

让我们通过点击应用程序菜单中的 文件 > 新建 > 文件 来向项目中添加一个名为 `ListDataSource` 的新文件。这个新对象将作为我们表视图的数据源。现在，这个文件会很基础。以下是一个示例实现：

```
import UIKit

class ListDataSource: NSObject {

}
extension ListDataSource: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return Book.sampleData.count
    }

    func tableView(_ tableView: UITableView,
        cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        // Dequeue a table view cell
        let cell =
          tableView.dequeueReusableCell(withIdentifier: "CatalogTableViewCell",
          for: indexPath)

        // Find the correct book based on the row being populated
        let book = Book.sampleData[indexPath.row]

        // Populate the table view cell title label with the book title
        cell.textLabel?.text = book.title

        return cell
    }
}
```

你可能会认出一些这段代码——或者，如果你一直在注意，会认出全部！这段代码与我们在`CatalogViewController`中用来显示书籍目录列表的代码是一样的。事实上，通过将这段代码放在一个单独的文件中，我们可以移除当前存放在`CatalogViewController`中的所有`UITableViewDataSource`代码。现在，`CatalogViewController`文件看起来是这样的：

```
class CatalogViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        // Do any additional setup after loading the view
    }
}
```

如果你构建并运行我们的应用程序，你可能会发现我们已经走得太远了。`CatalogViewController`没有必要的方法来填充表视图作为数据源，但它仍然链接在*Main.Storyboard*中。这会导致`SIGABORT`，从而使应用程序崩溃。

糟糕。让我们看看能否回去修复这个问题。事实上，在我们修复问题的同时，让我们看看是否可以通过让`CatalogViewController`中的表视图直接使用我们的新`ListDataSource`作为其数据源来使事情变得更好。

首先，我们必须移除*Main.storyboard*中的关联。打开 storyboard 并点击表视图。在 Xcode 右侧窗格中，点击 Connections inspector。在里面，你会看到已经为`dataSource`建立的连接。点击“x”按钮从 storyboard 中移除该连接。

现在，让我们将表视图切换到使用一个新的、独立的数据源对象。为了做到这一点，我们需要将表视图暴露给视图控制器。为此，我们将使用 Xcode 中的 Assistant editor 添加一个新的连接。点击项目窗口右上角的 Assistant editor 按钮。这将自动打开相应的`CatalogViewController`用于该场景——有些人可能会说“神奇”。

一旦视图控制器在两个显示器中都激活了，就在表视图上按住控制键并拖动连接到代码窗口，类似于我们在前面章节中连接浏览按钮的方式。你将把连接直接拖到这里`CatalogViewController`类定义的下方：

```
class CatalogViewController: UIViewController {

    // Drag the connection to this line

    override func viewDidLoad() {
        super.viewDidLoad()

        // Do any additional setup after loading the view
    }
}
```

会弹出一个模态框询问你是否正在创建一个`Outlet`，这是从视图控制器到特定视图的连接，或者你是否正在创建一个`Action`，这是一个控制事件（比如每当按下按钮时）。保持所有设置不变，将我们的新 outlet 命名为`tableView`。

此时，`CatalogViewController`应该看起来像这样：

```
class CatalogViewController: UIViewController {

    @IBOutlet weak var tableView: UITableView!

    override func viewDidLoad() {
        super.viewDidLoad()

        // Do any additional setup after loading the view
    }
}
```

我们已经从 storyboard 到视图控制器中建立了一个连接，直接将我们的表视图暴露给视图控制器。建立这个连接的原因是使我们能够在视图控制器类中设置表视图的数据源。看看这是如何完成的：

```
class CatalogViewController: UIViewController {

    @IBOutlet weak var tableView: UITableView!
    lazy var dataSource: ListDataSource = {
        return ListDataSource()
    }()

    override func viewDidLoad() {
        super.viewDidLoad()
        tableView.dataSource = dataSource
    }
}
```

让我们来看看代码的步骤。

首先要注意的是，我们在`CatalogViewController`内部添加了一个名为`dataSource`的新的`lazy`属性。`lazy`运算符使得此对象在访问属性时才被实例化。这通常被认为是数据源的良好实践，因为偶尔需要启动对象所需的时间。在我们的情况下，我们的数据源相当轻量级，但养成这样的良好习惯也是好的。它还允许我们保持初始化代码的封装性。在这种情况下，我们只是简单地创建了一个`ListDataSource`的新实例。

接下来，在`viewDidLoad`内部，我们添加了一些新代码。这个方法是视图生命周期的一部分；当它被调用时，我们可以安全地假设我们的输出和操作已经连接，视图控制器的`view`已加载。这个方法在视图生命周期中只被调用一次，但没问题，因为我们只需要一次。

在`viewDidLoad`内部，我们将表视图的`dataSource`属性设置为我们在视图控制器内创建的数据源。如果您构建并运行应用程序，您会注意到目录加载与之前相同。

在这一点上，您可能会问自己：为什么这很重要呢？创建一个完整的数据层似乎比让视图控制器直接提供它们的数据更复杂。

公平的观点。

然而，如果您还记得，我们的应用程序仍在使用静态的、硬编码的数据。让我们看看当我们过渡到更便携的东西时会发生什么。您将看到将数据抽象出视图层的全部威力。让我们开始吧！

# 是时候让我们的模型对象真实起来了

好吧。我们的`Book`对象有一些属性。但是，说实话，大多数书籍的元数据远不止我们到目前为止展示的这些。此外，如果我们想要改变我们的数据，我们必须发布一个新版本的应用程序。并且，这些应用程序必须保持同步。随着将书籍保存到以后和应用程序复杂度增加，这变得越来越困难。

正如 Dr. Phil（是的，那个 Dr. Phil）会说的，“现在是我们的模型对象*真实起来*的时候了。”

幸运的是，我们可以用两种方法解决这个问题：

1.  通过从静态、硬编码的数据切换到像 JSON 这样的可移植格式。

1.  通过从静态的、硬编码的数据源切换到通过服务器提供的东西，或者是应用程序之外的另一个真实数据源。

正如本章前面提到的那样，我们将在本书的第六章中解决第 2 点。然而，让我们先看看第 1 点：转向 JSON。

# 为一而设 JSON，为所有设 JSON

我们决定使用 JSON。以下是我们的`Book`对象在 JSON 中的一个示例：

```
{
    "title": "...",
    "authors": ["..."],
    "isbn": "...",
    "pageCount": 0,
    "fiction": true
}
```

事实上，如果我们把我们之前添加到`Book`对象的`sampleData`属性中的数据转换为 JSON，并将其保存为一个名为*catalog.json*的新文件，我们最终得到一个如下所示的文件：

```
[
	{
		"title": "Fight Club",
		"authors": ["Chuck Palahniuk"],
		"isbn": "978-0393039764",
		"pageCount": 208,
		"fiction": true
	},
	{
		"title": "2001: A Space Odyssey",
		"authors": ["Arthur C. Clarke"],
		"isbn": "978-0451457998",
		"pageCount": 296,
		"fiction": true
	},
	{
		"title": "Ulysses",
		"authors": ["James Joyce"],
		"isbn": "978-1420953961",
		"pageCount": 682,
		"fiction": true
	},
	{
		"title": "Catch-22",
		"authors": ["Joseph Heller"],
		"isbn": "978-1451626650",
		"pageCount": 544,
		"fiction": true
	},
	{
		"title": "The Stand",
		"authors": ["Stephen King"],
		"isbn": "978-0307947307",
		"pageCount": 1200,
		"fiction": true
	},
	{
		"title": "On The Road",
		"authors": ["Jack Kerouac"],
		"isbn": "978-0143105466",
		"pageCount": 416,
		"fiction": true
	},
	{
		"title": "Heart of Darkness",
		"authors": ["Joseph Conrad"],
		"isbn": "978-1503275928",
		"pageCount": 78,
		"fiction": true
	},
	{
		"title": "A Brief History of Time",
		"authors": ["Stephen Hawking"],
		"isbn": "978-0553380163",
		"pageCount": 212,
		"fiction": false
	},
	{
		"title": "Dispatches",
		"authors": ["Michael Herr"],
		"isbn": "978-0679735250",
		"pageCount": 272,
		"fiction": false
	},
	{
		"title": "Harry Potter and Prisoner of Azkaban",
		"authors": ["J.K. Rowling"],
		"isbn": "978-0439136365",
		"pageCount": 448,
		"fiction": true
	},
	{
		"title": "Dragons Love Tacos",
		"authors": ["Adam Rubin", "Daniel Salmieri"],
		"isbn": "978-0803736801",
		"pageCount": 40,
		"fiction": true
	}
]
```

随意添加额外的书籍；现在是个人喜好和私人爱好的时候。

# 将模型层切换为 JSON

如果您回想一下，到目前为止，我们在数据层代码中仍在使用`Book`对象提供的`sampleData`。让我们在两个项目中都移除它，并切换到直接使用我们的 JSON。

## Android

现在，我们有一个名为 *catalog.json* 的文件，其中包含所有 JSON `Book` 表示，保存在磁盘上。假设在 Android 应用中，它被保存为一个资产，在项目级别的 */assets/* 文件夹中。这个文件夹具有特殊属性和 API，使得像我们描述的这样的操作更容易使用，并且可能不会立即可用——如果在项目源中看不到名为“assets”的文件夹，请确保您在左侧窗格中的文件列表上方的下拉菜单中选择了“Android”项目视图，然后在 Android Studio 中右键单击项目名称，选择新建，然后文件夹，然后资产文件夹。您可以在此目录中创建一个新的文本文件，复制上述 JSON，并将文件保存为 *catalog.json*。

让我们将我们的 `BookDataSource` 转换为使用它。首先，我们将在 UI 线程上读取文件，但这只是第一步——任何时候涉及到磁盘或网络传输时，请确保您在后台线程中工作。

如果我们回顾一下第 Chapter 6 中的示例代码，您可能还记得如何从磁盘读取文件。如果您回想起 Chapter 12 中的练习，您可能还记得几种将 JSON 解析为有效的 Java 对象实例的方法。我们将同时使用这两种技术。

让我们的数据源类提供此功能，尽管随着时间的推移，您可能会发现将一些逻辑分离出来更合适。现在，让我们从一个基本的 `List<Book>` 数据源开始，但我们会装扮它，以便在实例化时，它读取 *catalog.json*，将每个 JSON 对象转换为一个 Java `Book` 实例，并将所有这些 `Book` 添加到自身（一个 `ArrayList`）中。

如果您运行它，您将获得与 *catalog.json* 中的 JSON 对象数量相等的 `Book` 实例数组；然而，如果您使用了 Java 实现，这些实例将是裸的。每个属性将是默认的。这是因为我们在定义 `Book` 属性时使用了匈牙利命名法：

```
public class Book {
  private final String mTitle;
  private final String[] mAuthors;
  private final String mIsbn;
  private final int mPageCount;
  private final boolean mIsFiction;
  ...
}
```

当 Gson 查看 `Book` 类并尝试从它手头的 JSON 中找到要映射的属性时，它会发现 `Book` 类实例想要一个“mTitle”，而不关心“title”——另一方面，JSON 中没有“mTitle”、“mIsbn”或“mAnything”的参考——它使用人类可读的键名。幸运的是，这是 Android 中的一个常见问题，使用 Gson 的 `SerializedName` 注解可以轻松解决。提供一个 `String` 给这个注解，Gson 将检查该 `String` 和实际的属性名。进行以下更新并再次运行。在 Java 中：

```
public class Book {

  public static final Book[] SAMPLE_DATA = {
      new Book("Fight Club", new String[]{"Chuck Palahniuk"}, "978-0393039764", 208, true),
      new Book("2001: A Space Odyssey", new String[]{"Arthur C. Clarke"}, "978-0451457998",
        296, true),
      new Book("Ulysses", new String[]{"James Joyce"}, "978-1420953961", 682, true),
      new Book("Catch-22", new String[]{"Joseph Heller"}, "978-1451626650", 544, true),
      new Book("The Stand", new String[]{"Stephen King"}, "978-0307947307", 1200, true),
      new Book("On The Road", new String[]{"Jack Kerouac"}, "978-0143105466", 416, true),
      new Book("Heart of Darkness", new String[]{"Joseph Conrad"}, "978-1503275928", 78,
      true),
      new Book("A Brief History of Time", new String[]{"Stephen Hawking"}, "978-0553380163",
        212, false),
      new Book("Dispatches", new String[]{"Michael Herr"}, "978-0679735250", 272, false),
      new Book("Harry Potter and Prisoner of Azkaban", new String[]{"J.K. Rowling"},
      "978-0439136365", 448, true),
      new Book("Dragons Love Tacos", new String[]{"Adam Rubin", "Daniel Salmieri"},
      "978-0803736801", 40, true)
  };

  @SerializedName("title")
  private String mTitle;
  @SerializedName("authors")
  private String[] mAuthors;
  @SerializedName("isbn")
  private String mIsbn;
  @SerializedName("pageCount")
  private int mPageCount;
  @SerializedName("fiction")
  private boolean mIsFiction;

  public Book(String title, String[] authors, String isbn, int pageCount,
  boolean isFiction) {
    mTitle = title;
    mAuthors = authors;
    mIsbn = isbn;
    mPageCount = pageCount;
    mIsFiction = isFiction;
  }

  public String getTitle() {
    return mTitle;
  }

  public void setTitle(String title) {
    mTitle = title;
  }

  public String[] getAuthors() {
    return mAuthors;
  }

  public void setAuthors(String[] authors) {
    mAuthors = authors;
  }

  public String getIsbn() {
    return mIsbn;
  }

  public void setIsbn(String isbn) {
    mIsbn = isbn;
  }

  public int getPageCount() {
    return mPageCount;
  }

  public void setPageCount(int pageCount) {
    mPageCount = pageCount;
  }

  public boolean isFiction() {
    return mIsFiction;
  }

  public void setFiction(boolean fiction) {
    mIsFiction = fiction;
  }

}
```

这次，你的`Book`实例应该被完全和适当地填充。就这样！你有一个工作的序列化协议。让我们检查一下可能在你的视线之外的一些功能。首先，你可能会注意到前面的代码中这个难以阅读的繁琐：

`Gson`可以很好地将单个`Book`实例转换为 JSON 字符串，反之亦然，但是当我们开始处理集合时，情况变得不那么简单。一个选择是创建一个具有泛型的类：

这可以使用传统的 Gson 方法`fromJson`来调用：

```
String json = // ... some string represetning multiple Book JSON objects
Books books = new Gson().fromJson(json, Books.class);
```

但是，如果失败了，我们可以使用`TypeToken`类来动态生成泛型。你正在创建一个`TypeToken`子类的实例，然后调用它的`getType`方法，因此语法实际上是这样的：

```
new TypeToken<GENERICIZED_DATA_TYPE>() {
  // no op
}.getType();
```

在那个例子中，“GENERICIZED_DATA_TYPE”可以是包括任意数量泛型的数据类型；它可能像`new TypeToken<Date>...`那样简单，或者像`new TypeToken<List<Map<String, List<Date>>>...`那样有几层嵌套。

回到手头的例子。使用这个新数据源将自动更新你的`Adapter`和`RecyclerView`，而且你甚至不需要改变你的适配器代码；只需确保在向`Adapter`类提供数据源时，构造函数包含一个`Context`参数，以便我们可以获取正确的文件目录：

这里的另一个有趣的技巧是，`Books`本身是`Book`实例的`List`，所以不必像前面的代码中看到的更新数据源那样，可以使用实用方法读取文件并直接将该字符串转换为数据源。假设我们恢复到没有方法的原始`Books`类：

假设我们还可以访问 第六章 中的方法。我们可以在`Activity`的`onCreate`方法中做类似这样的事情：

因为`Books`也是一个`ArrayList`，它将由*catalog.json*中的每本书填充，并能够成功地满足`BookDataSource`的`get`和`size`合约要求。

现在我们已经设置好了，不再需要`Book`类上的静态数据结构`SAMPLE_DATA`，可以安全地将其删除了。Android Studio 的一个方便功能是“重构”选项。除了有重命名项目中所有出现的方法或变量的功能外，它还可以运行“安全删除”，仅在项目中不再使用变量时才删除它。在更新前的更新后，在`Book.SAMPLE_DATA`上尝试这个选项，你应该可以安全地删除它。

## iOS

在 iOS 中，通过使用内置的`Codable`协议，开始使用 JSON 是非常容易的。这是一个由`Encodable`和`Decodable`协议组成的协议。它们为 Swift 编译器提供了一些必要的理解来推断代码。

想要了解更多关于`Codable`的详细信息，请阅读第十二章。不过，现在让我们看一个例子，说明`Codable`如何让我们在应用程序中轻松地利用 JSON。

让我们在`Book`结构体内部添加对`Codable`的支持：

```
struct Book: Codable {
    let title: String
    let authors: [String]
    let isbn: String
    let pageCount: Int
    let fiction: Bool
}
```

就是这样。你可能注意到，我们还从对象中移除了`sampleData`属性。这是有意为之的，因为我们现在要切换`ListDataSource`，从使用`sampleData`改为使用我们新创建的*catalog.json*，其中包含我们图书馆全部图书的目录。

让我们来看看`ListDataSource`：

```
class ListDataSource: NSObject {
    lazy var data: [Book] = {
        do {
            guard let rawCatalogData =
                try? Data(contentsOf:
                Bundle.main.bundleURL.appendingPathComponent("catalog.json")) else {
                return []
            }
            return try JSONDecoder().decode([Book].self, from: rawCatalogData)
        } catch {
            print("Catalog.json was not found or is not decodable.")
        }
        return []
    }()

}
extension ListDataSource: UITableViewDataSource {
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return data.count
    }

    func tableView(_ tableView: UITableView,
        cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        // Dequeue a table view cell
        let cell =
          tableView.dequeueReusableCell(withIdentifier: "CatalogTableViewCell",
          for: indexPath)

        // Find the correct book based on the row being populated
        let book = data[indexPath.row]

        // Populate the table view cell title label with the book title
        cell.textLabel?.text = book.title

        return cell
    }
}
```

我们添加了一个名为`data`的新`lazy`属性。这个属性是一个`Book`数组。每当首次访问该属性时，会读取*catalog.json*文件到一个名为`rawCatalogData`的`Data`类型对象中。然后，这个属性会被传递给一个`JSONDecoder`对象，以解码为`[Book]`类型的对象。

注意这里是`[Book]`，而不是原始的`Book`。声明如此的原因是因为 JSON 实际上包含了一个 JSON 对象数组。Swift 编译器聪明地在调用站点动态将*catalog.json*转换为 Swift 内部对象时解释了我们的声明。如果找不到我们的 JSON 文件——或者 Swift 编译器无法将其解码为任何东西——我们将返回一个空数组`[]`。

现在，我们还更新了用于驱动我们表视图的代码，在遵循`UITableViewDataSource`协议时。需要注意的代码行是：

```
func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
	return data.count
}
```

还要注意方法`tableView(_:cellForRowAt:)`中的`data[indexPath.row]`。

构建并运行应用程序，你会发现它与之前的功能一样——它显示了我们图书馆中所有书籍的列表——但现在书籍是通过应用程序捆绑包中的 JSON 文件动态提供给表视图的。

# 我们学到了什么

在这一章的短时间内，我们学到了：

+   如何将紧密耦合的代码从视图层分离到一个新的数据层

+   如何创建一个专门为向视图提供数据的离散对象

+   如何切换到使用 JSON 文件作为应用程序数据源

在下一章中，我们将进一步探讨；我们将使用我们的新数据层，为我们喜欢的书籍提供一些额外的保存功能。请继续阅读！
