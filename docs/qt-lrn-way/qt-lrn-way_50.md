# 五十、QString

这段时间回家，一直没有来得及写，今天才发现博客的编辑器有了新版。还是先来试试新版编辑器的功能吧！

今天要说的是 QString。之所以把 QString 单独拿出来，是因为 string 是很常用的一个数据结构，甚至在很多语言中，比如 JavaScript，都是把 string 作为一种同 int 等一样的基本数据结构来实现的。

每一个 GUI 程序都需要 string，这些 string 可以用在界面上的提示语，也可以用作一般的数据结构。C++语言提供了两种字符串的实现：C 风格的字符串，以'\0‘结尾；std::string，即标准模版库中的类。Qt 则提供了自己的字符串实现：QString。QString 以 16 位 Uniode 进行编码。我们平常用的 ASCII 等一些编码集都作为 Unicode 编码的子集提供。关于编码的问题，我们会到以后的时候再详细说明。

在使用 QString 的时候，我们不需要担心内存分配以及关于'\0'结尾的这些注意事项。QString 会把这些问题解决。通常，你可以把 QString 看作是一个 QChar 的向量。另外，与 C 风格的字符串不同，QString 中间是可以包含'\0'符号的，而 length()函数则会返回整个字符串的长度，而不仅仅是从开始到'\0'的长度。

同 Java 的 String 类类似，QString 也重载的+和+=运算符。这两个运算符可以把两个字符串连接到一起，正像 Java 里面的操作一样。QString 可以自动的对占用内存空间进行扩充，这种连接操作是恨迅速的。下面是这两个操作符的使用：

```cpp

QString str = "User: ";  
str += userName + "\n";
```

QString 的 append()函数则提供了类似的操作，例如：

```cpp

str = "User: ";  
str.append(userName);  
str.append("\n");
```

C 语言中有 printf()函数作为格式化输出，QString 则提供了一个 sprintf()函数实现了相同的功能：

```cpp

str.sprintf("%s %.1f%%", "perfect competition", 100.0);
```

这句代码将输出：perfect competition 100.0%，同 C 语言的 printf()一样。不过前面我们也见到了 Qt 提供的另一种格式化字符串输出的函数 arg():

```cpp

str = QString("%1 %2 (%3s-%4s)")  
      .arg("permissive").arg("society").arg(1950).arg(1970);
```

这段代码中，%1, %2, %3, %4 作为占位符，将被后面的 arg()函数中的内容依次替换，比如%1 将被替换成 permissive，%2 将被替换成 society，%3 将被替换成 1950，%4 将被替换曾 1970，最后，这句代码输出为：permissive society (1950s-1970s). arg()函数比起 sprintf()来是类型安全的，同时它也接受多种的数据类型作为参数，因此建议使用 arg()函数而不是传统的 sprintf()。 使用 static 的函数 number()可以把数字转换成字符串。例如：

```cpp

QString str = QString::number(54.3);
```

你也可以使用非 static 函数 setNum()来实现相同的目的：

```cpp

QString str;  
str.setNum(54.3);
```

而一系列的 to 函数则可以将字符串转换成其他基本类型，例如 toInt(), toDouble(), toLong()等。这些函数都接受一个 bool 指针作为参数，函数结束之后将根据是否转换成功设置为 true 或者 false：

```cpp

bool ok;  
double d = str.toDouble(&ok);  
if(ok)  
{  
    // do something...  
} else {  
    // do something...  
}
```

对于 QString，Qt 提供了很多操作函数，例如，使用 mid()函数截取子串：

```cpp

QString x = "Nine pineapples";  
QString y = x.mid(5, 4);            // y == "pine"  
QString z = x.mid(5);               // z == "pineapples"
```

mid()函数接受两个参数，第一个是起始位置，第二个是取串的长度。如果省略第二个参数，则会从起始位置截取到末尾。正如上面的例子显示的那样。

函数 left()和 rigt()类似，都接受一个 int 类型的参数 n，都是对字符串进行截取。不同之处在于，left()函数从左侧截取 n 个字符，而 right()从右侧开始截取。下面是 left()的例子：

```cpp

QString x = "Pineapple";  
QString y = x.left(4);      // y == "Pine"
```

函数 indexOf()返回字符串的位置，如：

```cpp

QString x = "sticky question";  
QString y = "sti";  
x.indexOf(y);               // returns 0  
x.indexOf(y, 1);            // returns 10  
x.indexOf(y, 10);           // returns 10  
x.indexOf(y, 11);           // returns -1
```

函数 startsWith()和 endsWith()可以检测字符串是不是以某个特定的串开始或结尾，例如：

```cpp

if (url.startsWith("http:") && url.endsWith(".png"))  
{  
}
```

这段代码等价于

```cpp

if (url.left(5) == "http:" && url.right(4) == ".png")  
{  
}
```

不过，前者要比后者更加清楚简洁，并且性能也更快一些。

QString 还提供了 replace()函数供实现字符串的替换功能；trimmed()函数去除字符串两侧的空白字符(注意，空白字符包括空格、Tab 以及换行符，而不仅仅是空格)；toLower()和 toUpper()函数会将字符串转换成小写大写字符串；remove()和 insert()函数提供了删除和插入字符串的能力；simplified()函数可以将串中的所有连续的空白字符替换成一个，并且把两端的空白字符去除，例如" \t ”会返回一个空格" "。

将 const char *类型的 C 风格字符串转换成 QString 也是很常见的需求，简单来说，QString 的+=即可完成这个功能：

```cpp

str += " (1870)";
```

这里，我们将 const char * 类型的字符串" (1870)"转换成为 QString 类型。如果需要显式的转换，可以使用 QString 的强制转换操作，或者是使用函数 fromAscii()等。为了将 QString 类型转成 const char *字符串，需要进行两步操作，一是使用 toAscii()获得一个 QByteArray 类型对象，然后调用它的 data()或者 constData()函数，例如：

```cpp

printf("User: %s\n", str.toAscii().data());
```

为了方便使用，Qt 提供了一个宏 qPrintable()，这个宏等价于 toAscii().constData()，例如：

```cpp

printf("User: %s\n", qPrintable(str));
```

我们调用 QByteArray 类上面的 data()或者 constData()函数，将获得 QByteArray 内部的一个 const char*类型的字符串，因此，我们不需要担心内存泄漏等的问题，Qt 会替我们管理好内存。不过这也暗示我们，注意不要使用这个指针太长时间，因为如果 QByteArray 被 delete，那么这个指针也就成为野指针了。如果这个 QByteArray 对象没有被放在一个变量中，那么当语句结束后，QbyteArray 对象就会被 delete，这个指针也就被 delete 了。

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/193918`](http://devbean.blog.51cto.com/448512/193918)