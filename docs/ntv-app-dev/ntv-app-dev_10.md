# 第九章：网络

尽管自包含应用程序完全可能——也许它只是使用内置的、自动更新的 Play 更新基础设施进行 HTTP 请求来更新其版本——在我们相互连接的世界中却很少见。无论您需要对用户进行身份验证、将使用情况指标发布到分析帐户、读取内容或媒体、下载或上传文件、侦听推送通知，还是只需与时间服务器同步，您都需要知道如何发出 HTTP 请求并理解 HTTP 响应。

幸运的是，HTTP 规范具有非常简单的规则集，大多数人发现它易于理解和学习。冒着过于简化的风险，想象一下 HTTP 事务由两部分组成：请求和响应。两者都以简单（通常是人类可读的）文本文件表示。每个文件的第一行描述了关于文件的一些基本属性（例如请求的 URL 和响应的状态）。接下来的行是传统的键值对语法的头信息：`header-name: Header Value`。然后是一个“空白”（空）行，然后就是主体。就是这样！要了解更多信息，请查看[“HTTP Made Really Easy”](https://oreil.ly/Qh4ZZ)。

从高级认证协议到在后台流式传输加密视频数据到上传手机上的每张图片，所有这些操作都使用了前面描述的相同基本原则。

# 任务

在本章中，您将学习：

1.  读取并从远程服务器打印文本文件。

1.  发出 HTTP POST 请求。

1.  下载二进制文件。

# Android

借助 Android 开发的开放特性，一些非常出色的第三方库已经涌现，围绕着像网络这样潜在复杂的任务。其中最受欢迎的可能是 OkHttp，由著名且值得信赖的 Android 开发者 Jake Wharton 开发，他在 Square 公司工作时编写了大量优秀的开源软件，并继续在 Google 直接贡献。如果您检查本 Android 部分末尾的注释块中的链接，您会发现大量链接来自 Square 的 GitHub 帐户。

记住，除了主存储器（RAM）以外的任何时候，你都应该考虑使用后台线程。读取本地文件、查询数据库，甚至保存偏好设置都应该在主 UI 线程之外进行。*处理网络请求和响应时尤为重要*。想象一下，一个小文件可能只需几毫秒就能触及、打开、读取、关闭并转换为`String`值，但即使这几毫秒也足以使`RecyclerView`的滑动或`NavigationDrawer`的打开显得有些卡顿。现在，想象一下进行网络请求的情况。我确信我们都见过需要 10 秒、20 秒甚至更长时间来处理的请求——有些服务器甚至不会在一分钟内超时！想象一下，在绘图线程上整个时间都在一个旋转的`for`循环中等待数据。*几乎所有情况下，在执行网络操作时，都应该使用后台线程*。

## 在远程服务器上读取和打印文本文件

话虽如此，使用纯 Java 发起网络请求并不复杂——以下六行（仅计算语句）的神奇代码将打印出任何公共可访问的远程文件：

让我们逐行分解这个过程。

首先，我们需要一个`URL`实例。`URL`实例代表一个*资源*，而不是*资源的位置*，尽管名称可能会让人误解。位置最准确地由传递给`URL`构造函数的`String`表示。

通过将我们的操作从`String`升级到`URL`，我们可以在开箱即用的标准库中获得大量好用的功能——例如通过`openConnection`方法获取`HttpUrlConnection`。虽然连接可以做很多事情，但我们可以在接下来的几行中做的最简单的事情是——获取其`InputStream`并读取出来！这看起来很像我们在第六章中处理流数据的方式——可以查看以获取复习。

就是这样！相当简单，对吧？好吧，让我们稍微加工一下。

## 发起 HTTP POST 请求

我们将利用公开可用的*jsonplaceholder.typicode.com*免费服务来模拟 POST 数据，这样我们可以看到我们的代码确实发送了我们稍后可以读取的数据。我们将使用[创建资源](https://oreil.ly/2iZxz)端点。

让我们用一个简单的空 POST 来修改我们的读取示例：

如果你仔细观察，你会发现只有一些非常细微的变化：

1.  我们将`openConnection`的返回强制转换为`HttpsUrlConnection`，而不是`HttpUrlConnection`，因为该位置使用了 HTTPS 方案。

1.  我们将连接的请求方法设置为 POST，而不是默认的 GET。

虽然这确实有效，并且您的输出应该给您创建的对象提供一个新的 ID，但没有太多其他的事情发生。在一个稍微接近真实生活的例子中，我们可能希望在请求中添加一些数据。这可能以查询参数、头信息或者 POST 主体的形式进行。对于这个服务，我们将使用 JSON POST 主体。

就像连接对象有一个内置的`InputStream`用于读取一样，它也方便地具有一个`OutputStream`用于写入。但是，这需要一些额外的管理 - 我们必须调用`connection.setDoOutput(true)`才能写入那个`OutputStream`。

让我们再次更新例子：

现在我们正在发送一些数据，服务器可以根据需要处理。同样，对于每个功能扩展，都需要更多的工作和更多的考虑，但如果从基础（第一个例子）开始，您可以对整个工作原理有一个相当好的理解。

开发者会发现，在使用网络库时，很多工作是非常相似的，即使这些库在幕后使用的操作可能完全不同。当我为当前雇主编写我们的二进制下载器时，我使用了所有标准库。后来，我们想要为所有网络请求（图片、REST、下载等）使用单一的 HTTP 客户端，所以我坐下来喝了一杯咖啡，计划在下午花点时间来重新调整我们的下载器，使其能够与 OkHttp 一起工作。我不记得这花了多长时间，或者改变了多少行，但我会说，午餐前我就在写测试了，而在 QA 之前已经完成（即：这是一个简单的任务）。

## 下载二进制文件

我们将结合到目前为止所呈现的信息，并结合我们在第六章中学到的一些内容来下载一个二进制文件。别担心，这与我们已经做过的事情并没有太大不同。

这是我在个人网站上使用的标题图片的 URL：*[*http://moagrius.com/assets/images/hero-trips.png*](http://moagrius.com/assets/images/hero-trips.png)*。它只是一张图片，但它是二进制数据 - 相同的逻辑也适用于视频、可执行文件，甚至只是几位任意的二进制数据。

让我们考虑一下需要做些什么不同的事情。我们知道我们将得到一个包含图片字节的`InputStream`，那我们该怎么处理呢？嗯，如果我们记得第六章关于文件的教训，我们知道我们会想要一个`FileOutputStream`来将这些字节保存到我们的本地设备上。那真的就是唯一的区别！让我们试试吧：

运行此方法后，您应该会在我的网站上找到一个完全解码的、视觉上准确的文件副本 - 祝贺您！

###### 注意

网络编程确实涵盖了比我们在本章中能够涵盖的更多内容，但我们将指导您正确的方向：

+   [OkHttp](https://oreil.ly/_Lk9z)和[Volley](https://oreil.ly/1fSIz)都提供了额外的网络 API。

+   [Picasso](https://oreil.ly/0VyGt)、[Glide](https://oreil.ly/_D3HE) 和 [Volley](https://oreil.ly/2gaY5) 都提供了非常简单的 API，用于加载和显示远程图像，结合了网络和解码层。

+   [Gson](https://oreil.ly/RRhOS) 和 [Jackson](https://oreil.ly/e95wK) 是用于从 REST API 读取的优秀 JSON 解析器。

+   [Retrofit](https://oreil.ly/lT-5Z) 结合了 OkHttp 进行网络请求和 Gson 进行 JSON 解析，以允许一些简单的 API 读取器。

# iOS

网络通信一直是 macOS 和 iOS 的一个优势。系统提供的专门构建和提供的对象套件既全面又深思熟虑。事实上，大多数第三方网络库只是轻量级地放在操作系统提供的类之上。有一些非常强大的选择可供选择，例如 Alamofire，但对于大多数应用程序来说，内置工具是最佳选择。因此，在本章中，我们将使用这些工具来演示 iOS 中直接在您指尖可用的强大工具。

让我们深入了解！

## 在远程服务器上读取和打印文本文件

从全球某个 Web 服务器读取数据，在其根本上是您可以通过网络执行的最简单的操作。在 iOS 中，您可以通过以下简单示例向服务器请求信息：

```
let url = URL(string: "https://www.oreilly.com")!
let task = URLSession.shared.dataTask(with: url) { (data, response, error) in
    guard let data = data else { return }
    let string = String(data: data, encoding: .utf8)!
    print(string)
}
task.resume()
```

让我们逐步了解这里发生了什么。

首先，我们创建一个指向特定文件、网站或 API 的`URL`——以此示例中的 O’Reilly Media 主页为例。接下来，我们使用共享的`URLSession`对象为我们的 URL 生成一个`URLSessionDataTask`，并在其后添加一个完成处理程序作为尾随闭包。在完成处理程序内部，我们将原始数据（作为`Data`对象传入）转换为一个`String`实例，然后`print`出该数据，这恰好是通常由 Web 浏览器显示的 HTML。最后，我们调用创建的任务上的`resume()`来启动整个过程。

###### 警告

您可能会注意到在示例代码中，我们使用的是 HTTPS URL。自 iOS 9 以来，除非在应用程序的 *Info.plist* 文件中通过`NSAppTransportSecurity`键明确允许（或完全禁用），否则所有 HTTP 请求都需要是 HTTPS。

此示例中出现了一些新的类。其中第一个新类是`URLSession`。这是驱动 iOS 中顶层网络 API 大部分功能的类。在其最基本的功能中，每个`URLSession`都是不同网络任务的协调器；在创建期间，任务直接与`URLSession`对象关联。会话对象本身可以通过在初始化器中传递`URLSessionConfiguration`对象来配置其行为。您可以创建多个会话，或者对于简单的需求，只需使用先前使用的名为`URLSession.shared`的共享会话属性。

###### 提示

`URLSession` 实例可以以多种方式进行配置。`URLSessionConfiguration` 也提供了一些标准配置以便于使用，包括 `default`、`ephemeral` 和 `background(with:)`。查看[苹果开发者文档](https://oreil.ly/_ldvZ)获取更多信息。

`URLSession` 实例也是任务的发起者。将网络任务看作操作是很容易的；它们提供了网络请求和响应的所有实际功能。与操作类似，它们通常也以异步方式运行。

有三种类型的任务：

+   `URLSessionDataTask`，用于从服务器接收数据（以 `Data` 对象返回）

+   `URLSessionDownloadTask`，主要用于从服务器检索文件，并可以暂停和恢复，同时提供下载进度更新（将在本章节的后面介绍）

+   `URLSessionUploadTask`，用于向服务器发送数据或文件（将在本章节的下一部分中介绍）

知道何时使用一种任务而不是另一种任务是很重要的。开发者很少会使用数据任务来下载大文件。此外，使用下载任务来接收简单的 JSON 响应也是不明智的（而且相当繁琐）。任务专门用于使开发者的工作更轻松，逻辑更简单。

现在，前面的示例有些简单和方法有些幼稚。以下是一个更完整和全面的示例，检查客户端错误、有效的服务器响应状态码以及空数据：

```
let url = URL(string: "https://www.oreilly.com")!
let task = URLSession.shared.dataTask(with: url) { (data, response, error) in
    if let error = error {
        print(error.localizedDescription)
        return
    }

    guard let response = response as? HTTPURLResponse, response.statusCode < 300 else {
        print("A server error occured.")
        return
    }

    guard let data = data, let string = String(data: data, encoding: .utf8) else {
        print("No data returned.")
        return
    }

    print(string)
}
task.resume()
```

接收来自服务器的数据很有用，而且绝大多数网络调用都是用于请求数据。但是，如果你花时间使用 REST API，你很快就会需要学习如何发送数据以及接收数据。

让我们看看它在 iOS 中是如何工作的。

## 发起一个 HTTP POST 请求

向服务器发送数据的最简单方法是发送一小段文本到一个 URL。可以使用以下代码完成：

```
let data = "text to send".data(using: .utf8)
let url = URL(string: "https://www.oreilly.com")!

var urlRequest = URLRequest(url: url)
urlRequest.httpMethod = "POST"
urlRequest.httpBody = data

let task =
  URLSession.shared.dataTask(with: urlRequest) { (data, response, error) in
	...
}
task.resume()
```

这只是一个简单的示例，但通常我们想要发送到服务器的数据并不是无结构的文本字符串。通常是 JSON 或键值对形式的数据。幸运的是，有一个有用的对象可以使用，称为 `URLComponents`，用于从一组值创建一个百分比编码的键值对数据字符串。你可以像这样使用它：

```
var components = URLComponents()
components.queryItems = [
    URLQueryItem(name: "name", value: "O'Reilly"),
    URLQueryItem(name: "isAwesome", value: "true")
]

let url = URL(string: "https://www.oreilly.com")!

var urlRequest = URLRequest(url: url)
urlRequest.httpMethod = "POST"
urlRequest.httpBody = components.query?.data(using: .utf8)

let task =
  URLSession.shared.dataTask(with: urlRequest) { (data, response, error) in
	...
}
task.resume()
```

首先，我们创建一个 `URLComponents` 的实例。接下来，我们向 `queryItems` 属性添加查询项，这些项是 `URLQueryItem` 类型的键值对象。每当将 `httpBody` 属性赋给创建的 `URLRequest` 时，这些对象就会被转换为一个百分比编码的文本字符串。

然而，通常我们可能想发送的不仅仅是键值对数据。以下是如何上传文件到服务器的示例：

```
let data = "file data".data(using: .utf8)!

let url = URL(string: "http://www.example.com")!
var request = URLRequest(url: url)
request.httpMethod = "POST"

let task = URLSession.shared.uploadTask(with: request, from: data)
task.resume()
```

在此代码中进行漫步时，我们首先创建一个表示我们模拟文件“文件数据”的数据字符串，该字符串将转换为准备稍后发送的`Data`对象。接下来，我们定义了一个指向*http://www.example.com*的 URL 作为占位符。然后，将其传递给我们在简单的`POST`示例中使用的新对象类型`URLRequest`。`URLRequest`是我们正在进行的请求的抽象。这样做是为了我们可以指定一些更改，以更改我们向服务器发送此请求的方式。在前面的示例中，我们指定了使用的 HTTP 方法：`POST`。这直接映射到可用的标准 REST 动词中，默认为`GET`。

最后，在此示例中，类似于前一节中的示例，我们正在使用共享 URL 会话创建一个`URLSessionTask`；但在这种情况下，我们正在使用`uploadTask`方法创建一个`URLSessionUploadTask`。为了启动任务并执行请求，我们调用`resume()`将传递的`data`对象发送到服务器。

这只是一个简单的示例。服务器可能会返回有关发送的请求的数据，包括结果是成功还是失败的信息。但是，可以通过使用`uploadTask`的重载方法，并像这样传递完成处理程序来保持此示例的简单性：

```
...

let task =
  URLSession.shared.uploadTask(with: request, from: data) { (data, response, error) in

    // Handle any client errors with error

    // Handle any server errors with response

    // Use the data object returned
}
```

大多数现代 API 将结构化数据（如 JSON）作为输入并返回 JSON 作为输出。我们简单的数据字符串示例实际上可以轻松扩展，几乎不需要更改，只需创建一个结构以提供我们数据的结构即可。这里是如何向端点发送 JSON 并处理响应的完整示例：

```
// Define an object to hold our data
struct Book: Codable {
    let title: String
    let isbn: String
}

// Populate our data
let book = Book(title: "Native Application Development", isbn: "this ISBN")

// Encode that object as a raw Data object to use in our request
let data = try! JSONEncoder().encode(book)

// Create the request
let url = URL(string: "http://www.example.com")!
var request = URLRequest(url: url)
request.httpMethod = "POST"
request.setValue("application/json", forHTTPHeaderField: "Content-Tye")

// Create and perform the task
let task =
  URLSession.shared.uploadTask(with: request, from: data) { (data, response, error) in
    // Handle client errors
    if let error = error {
        print(error.localizedDescription)
        return
    }

    // Handle server errors
    guard let response =
      response as? HTTPURLResponse, response.statusCode < 300 else {
        print("A server error occured.")
        return
    }

    // Check if data exists
    guard let data = data else {
        print("No data returned.")
        return
    }

    // Decode the JSON returned into a Book instance
    do {
        let book = try JSONDecoder().decode(Book.self, from: data)
        print("The book's title is \(book.title) and the ISBN is \(book.isbn)")
    } catch {
        print("Could not decode response to JSON")
    }
}
task.resume()
```

这是一个大的示例，但是如果您仔细观察，实际上只有几行发生了较大变化。首先，在示例顶部，我们正在定义一个用于保存数据的结构。之后不久，我们使用`JSONEncoder`将此对象转换为原始的`Data`对象。`JSONEncoder`是 Swift 标准库的一部分，提供了一种易于使用的方法来序列化和反序列化 JSON 对象。

###### 提示

有一整章专门讲解像 JSON 这样的“传输数据”。您可以查看了解有关功能和可用方法的更多信息。

下一个需要注意的重要变化是以下一行：

```
...
request.setValue("application/json", forHTTPHeaderField: "Content-Tye")
```

此命令将 HTTP 头部`Content-Type`设置为`application/json`。您可以以这种方式设置其他 HTTP 头部，但对于大多数 API，如果您要使用 JSON 作为发送数据的传输结构，则需要明确指定。

与其他示例相比，前面示例的其余部分可能看起来很熟悉，直到进入使用的完成处理程序。我们对客户端错误、服务器错误或缺少数据执行一些验证检查。然后，我们使用一个新的对象类型`JSONDecoder`，将返回的原始`data`对象（实际上只是 JSON）直接反序列化为`Book`实例。从那里，我们做一些像`print`书的`title`和`isbn`属性到命令行之类的事情。

在`URLSession`对象库中的大多数网络操作中存在相似性和模式。但是，从服务器下载文件需要额外的逻辑来处理稍微不同的一组要求。让我们看看在下一节关于下载二进制文件中的表现如何。

## 下载二进制文件

下载文件在代码中类似于从服务器请求数据。以下是执行从服务器下载请求以检索文件的示例：

```
let url = URL(string: "https://www.example.com/file.zip")!
let task =
  URLSession.shared.downloadTask(with: url) { (fileUrl, response, error) in
    // Check for client errors
    if let error = error {
        print(error.localizedDescription)
        return
    }

    // Check for server errors
    guard let response =
      response as? HTTPURLResponse, response.statusCode < 300 else {
        print("A server error occured.")
        return
    }

    // Check for a downloaded file
    guard let tempFileUrl = fileUrl else { return }
    print(tempFileUrl.path)
}
task.resume()
```

在前面的示例中，我们请求了一个特定 URL 的文件。一个主要的区别是，我们没有使用`dataTask(with:)`来创建我们的请求，而是使用了`downloadTask(with:)`。完成处理程序具有稍微不同的签名——不再传递`Data`实例，而是传递我们在示例中命名为`fileUrl`的`Url`实例。

下载任务与数据任务的不同之处在于，数据任务将请求内容存储在内存中，而下载任务将响应的缓冲区存储到临时文件中，随着下载的完成而完成。对于短时间的下载，这可能是瞬间完成的，但对于较大的下载，任务会逐步写入文件。

我们的完成处理程序在文件下载完成时执行，并包含该文件被写入的临时位置。这些文件是短暂的，将会被删除。

###### 警告

记住将文件移动到其他地方，例如*文档*目录非常重要。我们之前的示例只是打印文件的路径。查看第六章有关如何复制文件的文件基础知识。

对于较大的下载，通常最好向用户呈现某种类型的用户界面，指示已下载文件的百分比和剩余量。我们简单的完成处理程序示例没有提供这样的机制。然而，在`URLSession`子系统中，有一种处理方式：`URLSessionDownloadDelegate`协议。

### `URLSessionDownloadDelegate`

`URLSessionDownloadDelegate`包含一些可选方法，供对象在请求下载事件时实现。以下是如何实现委托、分配给`URLSession`并创建带有进度更新的下载任务的示例：

```
class NetworkClient: NSObject {
    // ...
}
extension NetworkClient: URLSessionDownloadDelegate {
    func urlSession(_ session: URLSession,
                    downloadTask: URLSessionDownloadTask,
                    didFinishDownloadingTo location: URL) {

        // Check for a server error
        guard let response =
          downloadTask.response as? HTTPURLResponse, response.statusCode < 300 else {
          return }

        // Prints the temporary file location
        print(location.path)
    }

    func urlSession(_ session: URLSession, task: URLSessionTask,
      didCompleteWithError error: Error?) {
        if let error = error {
            print(error.localizedDescription)
        }
    }

    func urlSession(_ session: URLSession,
                    downloadTask: URLSessionDownloadTask,
                    didWriteData bytesWritten: Int64,
                    totalBytesWritten: Int64,
                    totalBytesExpectedToWrite: Int64) {
        let percent = (totalBytesWritten/totalBytesExpectedToWrite) * 100
        print(percent)
    }
}

let url = URL(string: "https://www.example.com/file.zip")!

let client = NetworkClient()
let urlSession = URLSession(configuration: .default, delegate: client, delegateQueue: nil)
let task = urlSession.downloadTask(with: url)
task.resume()
```

让我们走过这段代码。

首先，我们创建了一个名为`NetworkClient`的新类，它继承自`NSObject`，这是 Objective-C 对象的基类。该类有一个扩展，实现了`URLSessionDownloadDelegate`协议。第一个方法，`urlSession(_:downloadTask:didFinishDownloadingTo:)`，是必需的。这是因为下载任务将它们下载的文件作为临时文件存储在文件系统中；每当文件下载完成，委托对象上的此方法被调用以告知您访问下载文件的位置。我们还可以通过检查`downloadTask`的`response`对象的状态码来在此方法体中处理服务器错误。

接下来的方法，`urlSession(_:task:didCompleteWithError:)`，是可选的，但在生产应用中应该实现。您可以在这里处理客户端错误，并检查是否出现了导致文件无法成功下载的问题。在我们的示例中，我们将错误的描述打印到控制台。

想知道下载进度如何通知应用程序？扩展中的最后一个方法就是查看的地方：`urlSession(_:downloadTask:didWriteData:totalBytesWritten:totalBytesExpectedToWrite:)`。此方法会定期调用，间隔由 iOS 中的网络子系统决定。我们可以使用`totalBytesWritten`和`totalBytesExpectedToWrite`的值来计算一个`percent`值。该值可用于更新从文本标签到自定义绘图例程到`UIProgressView`等一切。在我们的示例中，我们将该值打印到控制台，但请发挥想象力，看看可能的用途！

我们创建的这个类在稍后被实例化，并作为`delegate`参数传递给自定义的`URLSession`初始化程序。重要的是注意以下两行代码的发生位置：

```
...

let urlSession =
  URLSession(configuration: .default, delegate: client, delegateQueue: nil)
let task = urlSession.downloadTask(with: url)
task.resume()
```

首先，我们创建`URLSession`实例。在下一行，我们使用该对象为给定的 URL 创建`downloadTask`。我们*不*使用以前示例中使用的`URLSession.shared`实例。这是因为我们需要在初始化程序中设置委托的地方，在`URLSession`中，那就是在初始化器中。

### 暂停和恢复

对于长时间运行的下载，进度更新并非您的用户可能需要的唯一内容。部分完成的下载可能会失败。或者，也许您的用户想要在中途暂停下载，并在切换到 WiFi 网络后恢复下载。无论情况如何，使用`URLSession`暂停和恢复是可能的，但实现起来有些奇怪。

让我们来看一个示例，然后讨论发生了什么：

```
class PauseableClient: NSObject {
    let url = URL(string: "https://www.example.com/file.zip")!
    var resumeData: Data?

    func startDownload() -> URLSessionDownloadTask? {
        let task = URLSession.shared.downloadTask(with: url)
        task.resume()
        return task
    }

    func pauseDownload(for task: URLSessionDownloadTask?) {
        guard let task = task else { return }
        task.cancel { (resumeData) in
            self.resumeData = resumeData
        }
    }

    func resumeDownload() -> URLSessionDownloadTask? {
        guard let resumeData = resumeData else {
            print("Download can't be resumed!")
            return nil
        }

        let task = URLSession.shared.downloadTask(withResumeData: resumeData)
        task.resume()
        return task
    }

}

let client = PauseableClient()
var task = client.startDownload()
client.pauseDownload(for: task)
task = client.resumeDownload()
```

在这个示例中，我们有一个名为`PauseableClient`的类，其中有三个方法：`startDownload()`、`pauseDownload()`和`resumeDownload()`。在这些方法的第一个`startDownload()`中，我们使用一个 URL 启动下载任务。可以假设这是一个长时间运行的下载。我们还返回了稍后使用的`task`对象。在屏幕的稍后位置，我们调用`startDownload()`，并调用客户端的`pauseDownload(for:)`方法，将`task`传递进去。

在`pauseDownload()`方法内部，我们获取传入的`task`对象，并调用`cancel(byProducingResumeData:)`。这个方法接受一个闭包作为它唯一的参数。一旦任务被取消，闭包就会被调用，并且会传入一个作为`Data`实例给定的令牌，用于恢复下载。将此令牌存储以备将来使用，允许代码在未来可能再次恢复下载。

###### 注意

如果在`urlSession(_:task:didCompleteWithError:)`委托方法中报告了客户端错误，你可以潜在地存储恢复数据。检查传递的`error`对象中的`userInfo[NSURLSessionDownloadTaskResumeData]`。现在，有关恢复数据生成的限制有所限制。有关更多信息，请参考[Apple 的开发文档](https://oreil.ly/GITHH)中关于`URLSessionTask`的部分。

就所有意图和目的而言，我们创建的下载文件任务在这一点上已经消失了。调用`cancel`并不会实际暂停下载——它*取消*了下载。话虽如此，我们有一种方法可以在不必重新下载已收到的数据的情况下重新启动此请求。

在我们的示例中，这是在`resumeDownload()`方法中完成的。此方法检查恢复数据，然后调用`URLSession`通过方法`downloadTask(withResumeData:)`创建一个新的`URLSessionDownloadTask`。传入的恢复数据与之前任务中由`pauseDownload(for:)`保存的`resumeData`相同。

最终，一旦这个任务是从`URLSession`创建的，调用`resume()`来重新开始下载，希望能在我们暂停（或取消）之前的完成状态下再次开始下载。

### 委托

`URLSession`子系统中有比本章节中代码中展示的更多委托和委托方法。对象还应实现`URLSessionTaskDelegate`、`URLSessionDataDelegate`或`URLSessionDownloadDelegate`协议中的一些或所有方法来处理任务级事件。这些事件包括像开始和结束单个请求以及从数据或下载任务中周期性地更新进度，类似于前面展示的内容。

### 后台线程和更新 UI

`URLSessionTask` 在自己的后台线程上异步操作，无需阻塞当前线程或特别指定任何内容。然而，闭包被调用时不会返回到主线程，因此重要的是在主线程上调度任何 UI 更新或模型同步代码（或者在应用程序的其他部分使用的同步方法）。

### 应用程序传输安全性

要调用非安全的 URL，如以*http://*开头的 URL，需要在应用程序的*Info.plist*文件中为`NSAppTransportSecurity`键指定特殊配置。建议通过使用`NSExceptionDomains`键来指定某些域是不安全但可信任的方式来排除它们。可以通过将其`NSAllowsArbitraryLoads`键设置为`true`来完全关闭 App Transport Security，但这不是推荐的方法，除非绝对必要。

# 我们学到了什么

我们学到了关于如何在 Android 和 iOS 中的网络功能的许多知识：

+   我们学会了如何在两个平台上发出简单的请求。两者的过程相似，但在 Android 上使用流略有不同。

+   我们学会了如何以开发人员完全控制请求和响应的方式将数据发送到服务器。

+   我们讨论了下载二进制文件的性质以及存储它们到文件系统所需的处理方式。

+   嵌入到 iOS 中的安全控制要求默认使用 HTTPS。这与 Android 相反，后者在网络安全默认值方面要稍微灵活、开放，并且不那么限制性。

通常网络不可用或请求失败。在下一章中，我们将讨论当问题出现或者您只想告知用户某些其他操作正在进行时，如何为用户提供反馈。让我们继续前进吧！
