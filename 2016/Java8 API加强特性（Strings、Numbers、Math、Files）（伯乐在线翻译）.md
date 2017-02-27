# Java 8 API by Example: Strings, Numbers, Math and Files #
# Java8 API加强特性（Strings、Numbers、Math、Files）#
Plenty of tutorials and articles cover the most important changes in Java 8 like [lambda expressions](http://winterbe.com/posts/2014/03/16/java-8-tutorial/) and [functional streams](http://winterbe.com/posts/2014/07/31/java8-stream-tutorial-examples/). But furthermore many existing classes have been enhanced in the JDK 8 API with useful features and methods.

大量教程和文章覆盖 Java 8 中最重要的变化，像[lambda 表达式](http://winterbe.com/posts/2014/03/16/java-8-tutorial/)和[功能流](http://winterbe.com/posts/2014/07/31/java8-stream-tutorial-examples/)。但是另外 JDK 8 API 现有的许多类得到增强，携带有用的特性和方法。

This article covers some of those smaller changes in the Java 8 API - each described with easily understood code samples. Let's take a deeper look into Strings, Numbers, Math and Files.

本篇文章覆盖 Java 8 API 中一些小改变--每个变化都带有简单易理解的代码示例描述。让我们更深入的了解 Strings，Numbers，Math 和Files 类。
## Slicing Strings ##
## 分隔字符串 ##
Two new methods are available on the String class: join and chars. The first method joins any number of strings into a single string with the given delimiter:

String 类中有两个新方法可用：`join` 和 `chars`。第一个方法根据给定的分隔符连接任意数量的字符串为单一字符串：
```
String.join(":", "foobar", "foo", "bar");
// => foobar:foo:bar
```

The second method `chars` creates a stream for all characters of the string, so you can use stream operations upon those characters:

第二个方法 `chars` 为字符串中的所有字符创建流，这样你就可以对那些字符使用流操作：
```
"foobar:foo:bar"
    .chars()
    .distinct()
    .mapToObj(c -> String.valueOf((char)c))
    .sorted()
    .collect(Collectors.joining());
// => :abfor
```

Not only strings but also regex patterns now benefit from streams. Instead of splitting strings into streams for each character we can split strings for any pattern and create a stream to work upon as shown in this example:

现在不仅仅是字符串从流中受益，而且正则表达式模式也从流中受益。我们可以以任意模式分隔字符串并且创建流用于操作，而不是将字符串的每个字符分隔为流，如本例所示：
```
Pattern.compile(":")
    .splitAsStream("foobar:foo:bar")
    .filter(s -> s.contains("bar"))
    .sorted()
    .collect(Collectors.joining(":"));
// => bar:foobar
```

Additionally regex patterns can be converted into predicates. Those predicates can for example be used to filter a stream of strings:

此外正则表达式模式可以转换成谓词。（译者注：predicate 一词来自数理逻辑（Mathematical Logic）中的定义，可以被认为是一种布尔函数或者叫布尔逻辑。知乎中的说明我觉得挺好，有兴趣可以翻阅：[https://www.zhihu.com/question/25404942](https://www.zhihu.com/question/25404942)）这些谓词可以被用于过滤一个字符串流，例如：
```
Pattern pattern = Pattern.compile(".*@gmail\\.com");
Stream.of("bob@gmail.com", "alice@hotmail.com")
    .filter(pattern.asPredicate())
    .count();
// => 1
```

The above pattern accepts any string which ends with `@gmail.com` and is then used as a Java 8 `Predicate` to filter a stream of email addresses.

上述模式接收以 `@gmail.com` 结尾的任意字符串，然后被作为 Java 8 的 `Predicate` 用于过滤一个电子邮件地址流。
## 处理数字 ##
Java 8 adds additional support for working with unsigned numbers. Numbers in Java had always been signed. Let's look at `Integer` for example:

Java 8 为使用无符号数提供额外的支持。Java 中的数字总是有符号的。让我们来看看 `Integer`，例如：

An `int` represents a maximum of 2^32 binary digits. Numbers in Java are per default signed, so the last binary digit represents the sign (0 = positive, 1 = negative). Thus the maximum positive signed `int` is 2^31 - 1 starting with the decimal zero.

一个 `int` 能够表示的最大二进制数为 2^32 。在 Java 中数字总是有符号的，所以最后一个二进制位数字代表符号（0 = 正，1 = 负）。因此最大的正整数为 2^32 - 1，二进制以数字 0 开始。

You can access this value via `Integer.MAX_VALUE`:

你能够通过 `Integer.MAX_VALUE` 访问正整数最大值：
```JAVA
System.out.println(Integer.MAX_VALUE);      // 2147483647
System.out.println(Integer.MAX_VALUE + 1);  // -2147483648
```

Java 8 adds support for parsing unsigned ints. Let's see how this works:

Java 8 为解析无符号整数添加了支持。让我们看看它是如何工作的：
```
long maxUnsignedInt = (1l << 32) - 1;
String string = String.valueOf(maxUnsignedInt);
int unsignedInt = Integer.parseUnsignedInt(string, 10);
String string2 = Integer.toUnsignedString(unsignedInt, 10);
// 译者注：maxUnsignedInt=4294967295，string=4294967295，unsignedInt=-1， string2=4294967295
```

As you can see it's now possible to parse the maximum possible unsigned number 2^32 - 1 into an integer. And you can also convert this number back into a string representing the unsigned number.

正如你所见，现在可能将最大无符号数 2^32 - 1 解析到一个整数中。而且你也可以将这个数字转换回一个代表无符号数字的字符串。

This wasn't possible before with `parseInt` as this example demonstrates:

以前使用 `parseInt`是不可能做到的，（译者注：java 8 以前没有 `parseUnsignedInt`，调用 `parseInt` 只能够将小于 2^31 - 1 的字符串解析正整数，大于该值则会抛出异常）正如下面例子所示：
```
try {
    Integer.parseInt(string, 10);
}
catch (NumberFormatException e) {
    System.err.println("could not parse signed int of " + maxUnsignedInt);
}
```

The number is not parseable as a signed int because it exceeds the maximum of 2^31 - 1.

2^32 - 1 无法解析为一个正整数，因为它超过了最大值 2^31 - 1。

## Do the Math ##
## 处理算术 ##
The utility class `Math` has been enhanced by a couple of new methods for handling number overflows. What does that mean? We've already seen that all number types have a maximum value. So what happens when the result of an arithmetic operation doesn't fit into its size?

工具类 `Math` 增加了一些处理数值溢出的新方法。这意味着什么？我们已经看到所有的数值类型都有一个最大值。那么当一个算术运算的结果不适合它的大小会发生什么？
```
System.out.println(Integer.MAX_VALUE);      // 2147483647
System.out.println(Integer.MAX_VALUE + 1);  // -2147483648
```

As you can see a so called **integer overflow** happens which is normally not the desired behavior.

正如你所看到的，一个所谓的**整数溢出**发生，这通常不是期望的行为。

Java 8 adds support for strict math to handle this problem. `Math` has been extended by a couple of methods who all ends with `exact`, e.g. `addExact`. Those methods handle overflows properly by throwing an `ArithmeticException` when the result of the operation doesn't fit into the number type:

Java 8 为严谨的数学提供支持，用于解决这个问题。`Math` 类扩展了一些方法，所有这些方法都以 `exact` 结尾，例如 `addExact`。这些方法恰当的处理溢出，当运算结果不适合该数值类型时，抛出一个异常 `ArithmeticException`：
```
try {
    Math.addExact(Integer.MAX_VALUE, 1);
}
catch (ArithmeticException e) {
    System.err.println(e.getMessage());
    // => integer overflow
}
```

The same exception might be thrown when trying to convert longs to int via toIntExact:

相同的异常可能被抛出，当你尝试通过 'toIntexact' 将 long 类型转为 int 类型时：
```
try {
    Math.toIntExact(Long.MAX_VALUE);
}
catch (ArithmeticException e) {
    System.err.println(e.getMessage());
    // => integer overflow
}
```

## Working with Files ##
## 处理文件 ##
The utility class `Files` was first introduced in Java 7 as part of Java NIO. The JDK 8 API adds a couple of additional methods which enables us to use functional streams with files. Let's deep-dive into a couple of code samples.

工具类 `Files` 在 Java 7 中作为 Java NIO 的一部分首次出现。JDK 8 API 增加了一些额外的方法，这些方法能够让我们对文件使用功能流。让我们深入了解一些代码示例。

## Listing files ##
## 罗列文件 ##
The method `Files.list` streams all paths for a given directory, so we can use stream operations like `filter` and `sorted` upon the contents of the file system.

方法 `Files.list` 返回一个给定目录的所有路径的流，所以我们能够在文件系统内容之上使用流操作，例如 `filter` 和 `sorted` 
```
try (Stream<Path> stream = Files.list(Paths.get(""))) {
    String joined = stream
        .map(String::valueOf)
        .filter(path -> !path.startsWith("."))
        .sorted()
        .collect(Collectors.joining("; "));
    System.out.println("List: " + joined);
}
```

The above example lists all files for the current working directory, then maps each path to it's string representation. The result is then filtered, sorted and finally joined into a string. If you're not yet familiar with functional streams you should read my [Java 8 Stream Tutorial](http://winterbe.com/posts/2014/07/31/java8-stream-tutorial-examples/).

上面的例子罗列出当前工作目录的所有文件，然后映射每个路径到它的字符串表示。映射结果之后会被过滤、排序，并且最后被合成一个字符串。如果你还不熟悉功能流，你应该阅读我的 《[Java 8 流教程](http://winterbe.com/posts/2014/07/31/java8-stream-tutorial-examples/)》

You might have noticed that the creation of the stream is wrapped into a try/with statement. Streams implement `AutoCloseable` and in this case we really have to close the stream explicitly since it's backed by IO operations.

你可能已经注意到流的创建被包裹在 try/with 语句中。流实现了 `AutoCloseable` 接口，并且在这种情况下，我们不得不明确的关闭流操作，因为它被 IO 操作支持。

***The returned stream encapsulates a DirectoryStream. If timely disposal of file system resources is required, the try-with-resources construct should be used to ensure that the stream's close method is invoked after the stream operations are completed.***

**返回的数据流封装了 `DirectoryStream`。如果需要及时处理文件资源，`try-with-resources` 结构应该被使用，确保流的关闭方法能够在流操作完成之后被调用。**

## Finding files ##
## 查找文件 ##
The next example demonstrates how to find files in a directory or it's sub-directories.

下一个例子描述怎样查找一个目录或者它的子目录。
```
Path start = Paths.get("");
int maxDepth = 5;
try (Stream<Path> stream = Files.find(start, maxDepth, (path, attr) ->
        String.valueOf(path).endsWith(".js"))) {
    String joined = stream
        .sorted()
        .map(String::valueOf)
        .collect(Collectors.joining("; "));
    System.out.println("Found: " + joined);
}
```

The method `find` accepts three arguments: The directory path `start` is the initial starting point and `maxDepth` defines the maximum folder depth to be searched. The third argument is a matching predicate and defines the search logic. In the above example we search for all JavaScript files (filename ends with .js).

方法 `find` 接受三个参数：目录路径 `start` 是初始起点，同时 `maxDepth` 限定要搜索的文件夹最大的深度。第三个参数是一个匹配谓词，并定义搜索逻辑。在上面的例子，我们搜索所有的 JavaScript 文件（文件名以 .js 结尾）。

We can achieve the same behavior by utilizing the method `Files.walk`. Instead of passing a search predicate this method just walks over any file.

我们可以利用方法 `Files.walk` 实现相同的行为。这个方法仅仅遍历所有文件，而不是传入一个搜索谓词。

```
Path start = Paths.get("");
int maxDepth = 5;
try (Stream<Path> stream = Files.walk(start, maxDepth)) {
    String joined = stream
        .map(String::valueOf)
        .filter(path -> path.endsWith(".js"))
        .sorted()
        .collect(Collectors.joining("; "));
    System.out.println("walk(): " + joined);
}
```

In this example we use the stream operation `filter` to achieve the same behavior as in the previous example.

在这个例子中，我们使用流操作 `filter` 达到与之前例子相同的效果。

## Reading and writing files ##
## 读写文件 ##
Reading text files into memory and writing strings into a text file in Java 8 is finally a simple task. No messing around with readers and writers. The method `Files.readAllLines` reads all lines of a given file into a list of strings. You can simply modify this list and write the lines into another file via `Files.write`:

在 Java 8 中读文本文件和写字符串到一个文本文件是一个简单的事情。不会对读与写感到混乱。方法 `Files.readAllLine` 读取给定文件的所有行到一个字符串列表。你可以简单地修改此列表，以及通过 `Files.write` 写所有行到另外一个文件。
```
List<String> lines = Files.readAllLines(Paths.get("res/nashorn1.js"));
lines.add("print('foobar');");
Files.write(Paths.get("res/nashorn1-modified.js"), lines);
```

Please keep in mind that those methods are not very memory-efficient because the whole file will be read into memory. The larger the file the more heap-size will be used.

请记住，这些方法并不节省的，因为整个文件将被读到内存中。文件越大，越多的堆内存将被使用。

As an memory-efficient alternative you could use the method `Files.lines`. Instead of reading all lines into memory at once, this method reads and streams each line one by one via functional streams.

作为节省内存的选择，你可以使用 `Files.lines` 。这个方法通过功能流依次读取每行并转化为流，而不是一次性读取所有的行到内存中。
```
try (Stream<String> stream = Files.lines(Paths.get("res/nashorn1.js"))) {
    stream
        .filter(line -> line.contains("print"))
        .map(String::trim)
        .forEach(System.out::println);
}
```

If you need more fine-grained control you can instead construct a new buffered reader:

如果你需要更细粒度控制，你需要取代上边的方式，构建一个新的读缓存：
```
Path path = Paths.get("res/nashorn1.js");
try (BufferedReader reader = Files.newBufferedReader(path)) {
    System.out.println(reader.readLine());
}
```

Or in case you want to write to a file simply construct a buffered writer instead:

或者假设你想要写入一个文件，可以简单构建一个写缓存：
```
Path path = Paths.get("res/output.js");
try (BufferedWriter writer = Files.newBufferedWriter(path)) {
    writer.write("print('Hello World');");
}
```

Buffered readers also have access to functional streams. The method `lines` construct a functional stream upon all lines denoted by the buffered reader:

读缓存也能访问功能流。方法 `lines` 在读缓存所表示的所有行之上建立功能流。
```
Path path = Paths.get("res/nashorn1.js");
try (BufferedReader reader = Files.newBufferedReader(path)) {
    long countPrints = reader
        .lines()
        .filter(line -> line.contains("print"))
        .count();
    System.out.println(countPrints);
}
```

So as you can see Java 8 provides three simple ways to read the lines of a text file, making text file handling quite convenient.

所以正如你所看到的，Java 8 提供了三种方式按行读取一个文本文件，使得文本处理比较方便。

Unfortunately you have to close functional file streams explicitly with try/with statements which makes the code samples still kinda cluttered. I would have expected that functional streams auto-close when calling a terminal operation like `count` or `collect` since you cannot call terminal operations twice on the same stream anyway.

不幸的是，你必须明确调用 try/with 语法关闭文件的功能流，这样依旧使得实例代码有点混乱。我本来期望功能流能够在调用最终操作(terminal operation)，例如 `count` 或者 `collect` 方法时自动关闭，因为你无论如何都不可能在相同的流中调用两次最终操作(terminal operation)。

I hope you've enjoyed this article. All code samples are [hosted on GitHub](https://github.com/winterbe/java8-tutorial) along with plenty of other code snippets from all the [Java 8 articles](http://winterbe.com/java/) of my blog. If this post was kinda useful to you feel free to [star](https://github.com/winterbe/java8-tutorial) the repo and [follow me](https://twitter.com/winterbe_) on Twitter.

我希望你喜欢这篇文章。所有的代码实例托管在 [github](https://github.com/winterbe/java8-tutorial) 上，有大量来自我所有 Java 8 博客文章的代码段。如果这篇文章对你有点儿用处，请给我的 github 仓库[点赞](https://github.com/winterbe/java8-tutorial)，也可以在 twitter 上[关注](https://twitter.com/winterbe_)我。

Keep on coding!

继续编程！








