# 第六章：文件

作为开发者，您可能因为各种原因想要读取、写入或检查文件。例如，您可能想要修改图像并将其保存到磁盘，将视频下载到用户的 SD 卡（在确定可用后），或者使用简单的索引 JSON 平面文件数据库。文件处理逻辑在应用程序开发中经常出现，因此大多数开发者都需要对基础知识有很好的掌握。

# 任务

在本章中，您将学习到：

1.  从文件获取诸如磁盘上大小或最后修改日期等属性。

1.  从文件读取和写入数据。

1.  将数据从一个文件复制到另一个文件。

# Android

在 Android 中，开发者可能会查询设备上外部 SD 卡的位置，压缩一组文件以进行上传，将 XML 写入以跟踪用户首选项，从资产获取位图，读取资源文件，或以持久方式记录事件。即使是 SQLite 数据库（由 AOSP 框架提供的数据库）也存在为单个文件，因此您可以使用相同的逻辑确定其大小，或者创建其导出的副本。

在 Java 中读写文件已经发展了很长一段时间，现代版本的语言包括流式 API，更新的 `java.nio.file` 包（“nio”表示“新输入/输出”），以及像 `Files` 和 `Paths` 这样的便利的帮助类，提供了多种文件系统访问方法。不幸的是，这些对大多数 Android 开发者并不可用。截至撰写本文时，只有最近的两个 Android OS 版本（大约占所有 Android 安装的 21%）支持 `java.nio.file` 包。流式 API 和 `Files` 帮助类在任何 Android 版本中目前都不可用。

但不要担心！通过一点创造力，我们可以利用现有的框架和标准库 API 来完成几乎我们需要做的一切。我们将主要使用的包是 `java.io`（您猜对了...“输入和输出”），并且我们将经常使用的类之一是 `java.io.File`。`File` 实例是本地文件系统上位置的抽象表示。请注意，`File` 实例表示文件或目录；存在像 `isDirectory` 或 `isFile` 这样的 API 来区分它们。

要获取对现有文件的引用，可以将路径传递给 `File` 构造函数：`File file = new File("path/to/file.ext");`。在 Android 应用程序中，文件系统可能会受到限制，并且您的应用程序将被分配一个特殊的目录在设备上，您将获得对其读写权限——您可以通过在任何 `Context` 实例上调用 `getFilesDir()` 来获取该目录，这将返回一个表示系统为您的应用程序创建的该目录的已构造的 `File` 实例。因此，在 Android 应用程序中，您可以通过将该目录作为根传递来创建一个新文件：

```
File file = new File(context.getFilesDir(), "path/to/file.ext");
```

如果文件不存在，您需要创建它：`file.createNewFile();`。确保路径也存在；您可以通过调用`file.getParentFile().mkdirs()`来实现这一点，这将在根目录和路径中的最高级目录之间创建任意数量的目录。也就是说，使用前述示例，`file.mkdirs()`将创建一个名为“path”的文件夹，然后在其中再创建一个名为“to”的文件夹。请注意复数形式；`file.mkdir()`只会创建由`File`实例指定的单个目录。

## 获取文件的属性，比如大小或最后修改日期。

所以，现在您拥有了一个`File`实例的句柄，您可以从中读取数据，向其中写入数据，或检查其属性，比如大小或修改日期：

## 从文件中读取和写入数据。

当您立即考虑如何将`String`写入`File`时，这实际上不是 Java 和 Android 中的默认操作 —— 考虑到像图像、音频和视频这样的二进制文件，以及像 ZIP 和 TAR 这样的压缩文件。虽然有用于从`File`实例读取和写入`Strings`目录的 API，如`FileReader`和`FileWriter`，它们提供了简化的`String`读取和写入操作，但我们将集中在 Java 在读取和写入数据时的基本模式上：字节流。依赖于字节级数据的好处在于，相同的操作可以用于任何事物：写文本文件、流式传输多媒体、下载图像等。

不要感到害怕！您的程序中的每样东西都已经在某个层面上以字节的形式表示了，我们可以通过标准库中的现有 API 轻松地访问这些信息。例如，我们可以通过在`String`实例上调用`getBytes()`来从`String`中获取字节 —— 这一点很简单。

话虽如此，您可能会惊讶地发现，在`java.io.File`类上没有特定的`write`或`read`方法，事实上，读取和写入的过程需要另一个中介类：流（即`InputStream`或`OutputStream`的实例，或者适当的子类）。在 Java 中，流的概念简单地说就是一段可以按顺序读取的数据，从头到尾。这种方法的优点在于，我们不需要一次性将所有信息都存储在内存中。事实上，对象本身甚至可能没有明显的结束！例如，我们可能希望从实时源播放视频；使用流，我们只需按照它们的到达顺序读取字节，并根据需要缓冲或显示它们到我们的程序中。

但让我们回到我们的实际例子。我们将从写入文件开始。毫不意外，您需要一个`FileOutputStream`的实例来完成这项任务。您可以通过调用构造函数并传递您已经引用的`File`实例来获得`FileOutputStream`的实例：`OutputStream outputStream = new FileOutputStream(file);`。`FileOutputStream`的`write`方法有几种不同的签名，但让我们继续使用到目前为止我们对话的模式中的一个：`write(byte[] bytes)`。只需将一个字节数组作为唯一参数传递，这些字节将被写入由`FileOutputStream`引用的`File`实例。

因此，如果我们想要向文件写入一些文本，我们可以这样做：

哇！您刚刚向文件写入了“Hello world！”当然，就像大多数 Java 和 Android 一样，这个相对简单的概念有一些陷阱。例如，`FileOutputStream`构造函数抛出一个`FileNotFoundException`，`write`方法抛出一个`IOException`。此外，我们必须确保在完成时关闭流，这本身会抛出一个`IOException`。虽然完整表达所有这些工作的块可能有点难以理解，但我们可以将其全部封装在一个带有检查异常的方法中，并减少一些代码行数：

您可以像这样使用它：

从文件中读取数据遵循非常类似的模式。正如你可能已经猜到的那样，我们将使用`FileInputStream`而不是`FileOutputStream`。

再次，由于我们是按照二进制数据来思考的，我们需要做更多的工作才能将文件内容转换为人类可读的文本。如果您经常读写纯文本文件，有更简单的方法，如`FileWriter`和`FileReader`，但如前所述，使用字节流是一种通用的解决方案，而将其转换为`String`则是微不足道的。

类似于`FileOutputStream`，`FileInputStream`构造函数也将接受一个`File`参数：`InputStream inputStream = new FileInputStream(file);`。一旦实例化了，您可以使用普通的`read()`签名读取单个字节，或者通过传递字节数组缓冲区来读取大块数据。现在我们将讨论单字节简单的`read()`方法，但请记住，使用缓冲区通常更高效，特别是对于大文件。

`read`方法返回一个`int`，它表示读取的字节数，或者返回-1 表示文件（和流）的结尾。您可以将其转换为`char`以构建文件内容的`String`表示：

```
// inputStream is a valid instance of a FileInputStream
StringBuilder builder = new StringBuilder();
int byte = inputStream.read();
while (byte != -1) {
  builder.append((char) byte);
  byte = inputStream.read();
}
String message = builder.toString();
```

再次，其中几种方法会抛出检查异常，而且我们再次需要在完成时关闭流，因此将所有内容封装在一个单独的检查方法中可能是有用的：

除了`FileInputStream`之外，还有其他流源，您可能考虑将转换流到字符串的方法抽象化，如下所示：

## 复制数据从一个文件到另一个文件。

您可以看到我们如何轻松地将这两个操作结合起来，复制任何文件，无论是简单的文本文件还是价值一千兆字节的视频！由于我们既不知道也不关心它是文本文件还是二进制文件，我们不必围绕将`Character`实例转换或从`String`中提取字节而进行任何复杂操作——通过保持我们的逻辑不可知，您可以看到这个操作实际上是最可读的之一：

###### 警告

我们在这些示例中使用了`InputStream.read`方法以确保清晰。该方法每次调用返回一个字节。你可以通过将字节块写入通常称为“缓冲区”的大小字节数组来获得更好的性能。`InputStream`有用于这些缓冲区的`read`方法，`OutputStream`也有用于它们的`write`方法。

就是这样！现在您已经拥有在 Android 应用程序中读写文件所需的工具。务必查看开发文档中的`File`API 以获取更多信息！

###### 注意

Apache Commons Java 库有一个众所周知且备受推崇的文件和输入/输出模块`apache.commons.io`。特别有用的是`IOUtils`和`FileUtils`辅助类。

# iOS

在 iOS 中，文件操作驱动了一些非常强大技术的基础。最终，在任何足够复杂的应用程序中，都会出现需要在文件系统上读写文件的用例。由于 iOS 的沙盒性质，有一些关于数据组织方式的先决知识是必要的，才能开始处理文件。

## 从文件中获取大小或上次修改日期等属性

实际上，文件访问允许的主要领域有两个：应用程序包容器和应用程序数据容器。让我们从应用程序包容器开始。

### 应用程序包

应用程序包包括应用程序二进制文件以及与应用程序一起编译和交付的所有资源。在安装时，该目录经过代码签名以防止篡改。因此，不可能更改应用程序包内的文件或写入该目录（后文详述）。

通过 Swift 中的`Bundle`类提供对应用程序包中文件的访问。一个应用程序可以通过框架拥有多个包，因此通过`main`类变量来定位应用程序包。为了访问名为*image.png*的文件，需要创建文件 URL，如下所示：

```
 let file = Bundle.main.url(forResource: "image", withExtension: "png")
```

如果该文件位于名为*sample-images*的子目录中，则变量将被实例化如下：

```
 let file =
   Bundle.main.url(forResource: "image", withExtension: "png", subdirectory:
   "sample-images")
```

### 数据（和文档）

从应用程序包中使用静态且不可更改的文件很快变得受限。最终，将会出现需要读取和写入用户生成的文档和数据的需求。为每个安装在 iOS 中的应用程序创建了一组三个目录：

+   文档

+   库

+   临时

每个目录都有特定的用途，如以下各节所述。

### 文档

用户生成的内容和需要用户访问的文件应存放在这里。这些文件默认会通过 iTunes 和 iCloud 进行备份。应用程序还可以启用文件共享，让用户直接与这些文件交互。

### Library

在应用程序的*Library*文件夹中，通常有一些预定义的目录，用于放置额外的文件。其中两个最重要的目录是：*Application Support* 和 *Caches*。需要持久保存以备将来使用但不向用户显示的数据应存储在*Application Support*中。缓存数据应存储在*Caches*中。

### tmp

这个目录用于写入和读取临时文件。应用程序有责任通过删除未使用的文件来清理此目录。但是，在应用程序不使用时，系统偶尔可能会清理此目录。

### 访问目录

几乎不可能直接构建指向应用程序数据容器内文件或目录的 URL，因为这些文件的路径复杂且不透明。这是苹果故意设计的，需要开发者使用称为`FileManager`的通用文件操作类。

例如，要访问应用程序的*Documents*目录中名为*image.png*的文件，可以构造如下的 URL：

```
let file = try? FileManager.default
  .url(for: .documentDirectory, in: .userDomainMask, appropriateFor: nil, create:
  false).appendingPathComponent("image.png")
```

在模拟器上，访问此文件的直接路径是*~/Library/Developer/CoreSimulator/Devices/CF5BCBA7-C7CA-4484-AB54-7BE938D67ECB/data/Containers/Data/Application/313B2DDD-ABDD-4D14-B6CD-85847F29EF2C/Documents/image.png*。

注意使用`.documentDirectory`枚举值来定位应用程序数据容器内特定目录的直接路径。使用`FileManager`已为您完成了查找所有父目录的工作。此类还提供了几个其他预定义键。例如，存储在应用程序的*Application Support*目录中名为*data.json*的平面文件的路径如下所示：

```
let jsonFile = try? FileManager.default
    .url(for: .applicationSupportDirectory, in: .userDomainMask, appropriateFor:
    nil, create: false).appendingPathComponent("data.json")
```

这里展示了如何创建临时目录中稍后访问的文件*download.dat*的路径：

```
let tempFile =
  try? FileManager.default.temporaryDirectory.appendingPathComponent("download.dat")
```

### 文件属性

现在已经介绍了应用程序文件结构的基础知识，让我们来看看文件操作本身。值得注意的是，在 Swift 标准库中，这些操作非常简单，通常只需要几行代码即可完成。

使用*image.png*作为应用程序主包中包含的图像的示例，可以直接在访问文件的`URL`对象上获取文件大小，如下所示：

```
let url = Bundle.main.url(forResource: "image", withExtension: "png")
if let resourceValues = try? url.resourceValues(forKeys: [.fileSizeKey]) {
	print(resourceValues.fileSize)
}
```

变量`url`是一个指向应用程序包中文件名为*image*且文件扩展名为*png*（即*image.png*）的`URL`对象。有一个在`URL`上的同步方法可以提取指定键的“资源值”（其他操作系统可能称为“文件属性”）。在这个例子中，我们提供了`.fileSizeKey`键，它对应文件在磁盘上的大小。

通过包含除文件大小键外的其他键，可以获取附加的文件属性。例如，要获取文件的最后修改日期，可以提供`.contentModificationDateKey`键，而不是`.fileSizeKey`。此外，您可以像这个示例中一样同时获取这两个属性：

```
let url = Bundle.main.url(forResource: "image", withExtension: "png")
if let resourceValues = try? url.resourceValues(forKeys: [.fileSizeKey,
. contentModificationDateKey]) {
	print(resourceValues.fileSize)
	print(resourceValues.contentModificationDate)
}
```

## 从文件读取和写入数据

Foundation 框架和 Swift 标准库中提供了几个对象的简单便利方法来读取和写入文件，尤其是`String`和`Data`。例如，要从应用程序的*Documents*目录中读取名为*file.txt*的文本文件并将其转换为字符串对象，请使用以下代码：

```
let file =
  try? FileManager.default.url(for: .documentDirectory, in: .userDomainMask,
  appropriateFor: nil, create: false).appendingPathComponent("file.txt")

// Read the file into a string
let contents = try? String(contentsOf: file, encoding: .utf8)
```

更新文件类似，需要调用`write(to:atomically:encoding:)`，如下所示：

```
let file =
  try? FileManager.default.url(for: .documentDirectory, in: .userDomainMask,
  appropriateFor: nil, create: false).appendingPathComponent("file.txt")

// Read the file from the Documents directory
var contents = try? String(contentsOf: file, encoding: .utf8)

...

// Write the string back to the same file
try? contents.write(to: file, atomically: false, encoding: .utf8)
```

`Data`对象并没有太大区别。这些对象旨在提供与 C 风格字节数组分离的原始字节数据访问。例如，假设您有一个图像文件的 URL；您可以将图像读入内存，并像此处所示将图像写入磁盘：

```
// Read the file's data into a Data object
var data = try? Data(contentsOf: imageFileUrl)

...

// Write the data back to the same file
try? data?.write(to: imageFileUrl)
```

### 所有的道路都通向文件管理器。

最终，在编写 iOS 应用程序和执行文件操作的过程中，将需要更复杂的操作。在这些情况下，有一个名为`FileManager`的多用途类可提供一个共享的、线程安全的实例，非常适用于复杂的文件操作。

使用`FileManager`读取相同的*file.txt*文件需要一些额外的逻辑：

```
// Provide a file to read in the user's Documents directory
let file =
  try? FileManager.default.url(for: .documentDirectory, in: .userDomainMask,
  appropriateFor: nil, create: false).appendingPathComponent("file.txt")

// Get the contents of the file as a Data object
if let contents = FileManager.default.contents(atPath: file.path) {
	// Create a String from the raw data
	let contentsString = String(data: contents, encoding: .utf8)!
	print(contentsString)
}
```

首先要注意的一点是使用`file.path`。这将 URL 对象转换为文件系统中文件的绝对路径的字符串表示；这是必需的，因为`FileManager`对 URL 的支持不完整。接下来，`if let`语句将文件的原始内容读入数据对象中。如果`FileManager`无法找到或访问所请求的文件，则提供了一些防止空值的安全性。最后，在本示例中，假设文件是可访问的，使用指定的编码（UTF-8）从数据对象中实例化字符串。

写入文件类似。以下是如何直接使用`FileManager`将字符串写入文件的示例：

```
let example = "I love tacos."

// Convert the string to a Data object
let exampleData = example.data(using: .utf8)

// Create the file using the preceding data object (and overwrite any existing files)
FileManager.default.createFile(
    atPath: sharedFile.path, contents: exampleData, attributes: nil) // returns a Bool
```

文件操作的成功与否通过`createFile(atPath:contents:attributes:)`返回布尔值 true 或 false 来表示。这与使用`String`和`Data`直接使用的更现代的 Swift 操作符`throws`形成对比。

那么为什么在看起来不像是现代 API（它位于旧版 Foundation 框架而不是 Swift 标准库中）且稍微比`Data`和`String`实例方法更繁琐时，还要使用`FileManager`呢？答案在于`FileManager`的预期用途。当需要完成与文件和目录的更复杂交互时，`FileManager`非常出色。`FileManager`擅长检查目录层次结构，提供有关文件存在性的反馈，删除文件，覆盖文件，更新文件属性等等。但在简单的数据读写方面则表现得不如其他 API。

## 将数据从一个文件复制到另一个文件

使用仅提供`String`方法复制文件将需要将文件读入内存中的`String`实例，然后使用`write(to:atomically:encoding:)`将该字符串写入到另一个文件中。这不是复制文件的最有效方式，对于超过设备可用内存量的较大文件可能不容易实现。

使用以下代码片段可以在`FileManager`中完成文件复制：

```
// Provide an original file location
let originalFile = try? FileManager.default
    .url(for: .documentDirectory, in: .userDomainMask, appropriateFor: nil, create: false)
    .appendingPathComponent("file.txt")

// Provide a location where the copied file should go
let copiedFile = try? FileManager.default
    .url(for: .documentDirectory, in: .userDomainMask, appropriateFor: nil, create: false)
    .appendingPathComponent("newFile.txt")

// Copy the file
try? FileManager.default.copyItem(at: originalFile, to: copiedfile)
```

与使用`String`的第一个例子相比，这要高效得多，因为`FileManager`在复制文件之前不需要将文件先读入内存。此外，`FileManager`利用了 Apple File System (APFS)，这是苹果的专有文件系统，通过制作对象的克隆来进行高效的复制过程。

在`FileManager`中还有更多等待发现的秘密。不过，这已足以让你开始 iOS 中的基本文件操作。请随时查阅苹果提供的文档，了解所有可能的操作。

### URL 与字符串的比较

本章中的代码示例中，使用 URL 来提供文件路径信息，而不是`String`对象。这是因为 URL 对象通常更适合内部高效地存储路径信息。类似地，它们在通过添加目录、更改名称等方式改变文件路径表示时通常更高效。

一些由`FileManager`和其他基于 Foundation 的 API 提供的方法使用字符串而不是 URL。使用 URL 对象的路径变量可以将其转换为字符串路径。例如：

```
let fileURL = Bundle.main.url(forResource: "file", withExtension: "txt")!
let filePath = fileURL.path
```

### iCloud 和 iTunes 备份

我们尚未涵盖的一件事是 iCloud 文件同步。iCloud 中的文件与用户*Documents*目录中的普通文件类似；然而，使用不同的容器来直接定位文件。用户*Documents*和*Application* *Support*目录中的所有文件都会自动备份到 iCloud 和 iTunes 中。

可以并且建议从 iTunes 备份和 iCloud 同步中排除某些文件。例如，可以通过以下代码直接从 URL 设置文件的`isExcludedFromBackupKey`来排除未来可以重新下载的来自网络服务的大型视频文件：

```
var values = URLResourceValues()
values.isExcludedFromBackup = true
try? fileUrl.setResourceValues(values)
```

# 我们学到了什么

正如你可能已经注意到的，文件在 Android 和 iOS 之间是一个有趣的交集。它们同时又是截然不同但又根本相同的。归根结底，我们处理的是比特和字节，但数据访问的方式代表了两种非常不同的架构范式。

在 Android 中，使用流来读写数据与 iOS 更加程序化和内置的方法形成鲜明对比。此外，iOS 中还残留着 UNIX 的痕迹，贯穿其整个文件结构。

希望这两种方法的概述为你提供了一个起点，让你开始理解如何处理两个平台的文件系统操作。在下一章中，我们将讨论如何将数据持久化超越文件系统，进入数据库和对象图的领域。
