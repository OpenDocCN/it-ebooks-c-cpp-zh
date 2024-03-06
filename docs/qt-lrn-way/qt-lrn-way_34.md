# 三十四、国际化(下)

上次说了国际化的过程，现在来看一下具体的国际化的相关代码。

在代码中，我们使用 tr()将需要翻译的字符串标记出来。lupdate 工具就是提取出 tr()函数中的相关字符串。tr()函数是 QObject 类的一个 static 函数，其签名如下：

```cpp

static QString tr(const char *sourceText, const char *comment = 0, int n = -1);
```

虽然我们只传了一个参数，但是实际上 tr()函数是接受 3 个参数的。第一个参数是我们需要翻译的文字，如果使用 qm 文件有对应的字符串，则使用对应的字符串进行替换，否则将显示 sourceText 参数指定的字符串。第二个参数是一个注释，用于解释前面的 sourceText 的含义，比如 table 一词既可以当做桌子翻译，又可以当成表格翻译，这时你就需要提供这个注释。或许你会问，使用翻译工具的时候不是有源代码吗？问题是，有可能人家不使用这个翻译工具，而使用别的工具，这样就不能保证会有这个源代码的预览；并且，你的程序不一定必须要发布源代码的；翻译人员往往只得到我们导出的 ts 文件，如果你加上注释，就可以方便翻译人员进行翻译。最后一个参数 n 用于指定字符串是否为复数。我们知道，很多语言，比如英语，很多名词的单复数形式是不相同的，为了解决这个问题，Qt 在 tr()函数中提供了一个参数 n。请看如下代码：

```cpp

int n = messages.count(); 
showMessage(tr("%n message(s) saved", "", n));
```

对于 n 的值的不同，Qt 会翻译成不同的文字，例如：

| n | 翻译结果 |
| :-- | :-- |
| 0 | 0 message saved |
| 1 | 1 message saved |
| 2 | 2 messages saved |
| 5 | 5 messages saved |

tr()函数是 QObject 的函数，如果你的类不是继承自 QObject，就不能直接使用 tr()函数。比如我们在 main()函数中希望增加一句设置 MainWindow 的 title 的代码：

```cpp

w.setWindowTitle(tr("MyApp"));
```

直接这样写是无法通过编译的，因为 main()函数是全局函数，所以这个 tr()是找不到的。解决办法一是显式地调用 QObject 的函数：

```cpp

w.setWindowTitle(QObject::tr("MyApp"));
```

或者，你可以使用 QCoreApplication 的 translate()函数。你一定还记得，我们的 main()函数的第一句总是 QApplication app;，其实，QApplication 就是 QCoreApplication 的子类。所以，我们也能这样去写：

```cpp

w.setWindowTitle(app.translate("MyApp"));
```

由于在 Qt 程序中，QCoreApplication 是一个单例类，因此，Qt 提供了一个宏 qApp，用于很方便的访问 QCoreApplication 的这个单例。所以，在其他文件中，我们也可以直接调用 qApp.translate()来替换 tr()，不过这并没有必要。

如果你的翻译文本中包含了需要动态显示的数据，比如我们上次代码中的

```cpp

QMessageBox::information(NULL, tr("Path"), tr("You selected\n%1").arg(path));
```

这句你当然可以写成

```cpp

QMessageBox::information(NULL, tr("Path"), "You selected\n" + path);
```

但这种连接字符串的方式就不能够使用 tr()函数了！因此，如果你需要像 C 语言的 printf()函数这种能够格式化输出并且需要翻译时，你必须使用我们例子中的%1 加 arg()函数！

如果你想要翻译函数外部的字符串，你需要使用两个宏 QT_TR_NOOP()和 QT_TRANSLATE_NOOP()。前者是用来翻译一个字符串，后者可以翻译多个字符串。它们的使用方法如下：

```cpp

QString FriendlyConversation::greeting(int type) 
 { 
         static const char *greeting_strings[] = { 
                 QT_TR_NOOP("Hello"), 
                 QT_TR_NOOP("Goodbye") 
         }; 
         return tr(greeting_strings[type]); 
 } 
static const char *greeting_strings[] = { 
         QT_TRANSLATE_NOOP("FriendlyConversation", "Hello"), 
         QT_TRANSLATE_NOOP("FriendlyConversation", "Goodbye") 
 }; 

 QString FriendlyConversation::greeting(int type) 
 { 
         return tr(greeting_strings[type]); 
 } 

 QString global_greeting(int type) 
 { 
         return qApp->translate("FriendlyConversation", 
                                                        greeting_strings[type]); 
 }
```

好了，以上就是我们用到的大部分函数和宏。除此之外，如果我们运行前面的例子就会发现，实际上我们只是翻译了菜单等内容，打开文件对话框并没有被翻译。原因是我们没有给出国际化的信息。那么，怎么才能让 Qt 翻译这些内建的文字呢？我们要在 main()函数中添加几句：

```cpp

int main(int argc, char *argv[]) 
{ 
        QApplication a(argc, argv); 
        QTranslator qtTranslator; 
        qtTranslator.load("myapp.qm"); 
        a.installTranslator(&qtTranslator); 
        QTranslator qtTranslator2; 
        qtTranslator2.load("qt_zh_CN.qm"); 
        a.installTranslator(&qtTranslator2); 
        MainWindow w; 
        w.resize(800, 600); 
        w.show(); 
        return a.exec(); 
}
```

我们又增加了一个 QTranslator 对象。Qt 实际上是提供了内置字符串的翻译 qm 文件的。我们需要在 Qt 安装目录下的 translations 文件夹下找到 qt_zh_CN.qm，然后同前面一样，将它复制到 exe 所在目录。现在再运行一下程序：哈哈已经完全变成中文了吧！

至此，我们的 Qt 程序的国际化翻译部分就结束啦！

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/193918`](http://devbean.blog.51cto.com/448512/193918)