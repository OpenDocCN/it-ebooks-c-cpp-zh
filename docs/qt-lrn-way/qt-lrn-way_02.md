# 二、Hello, world!(续)

下面来逐行解释一下前面的那个 Hello, world!程序，尽管很简单，但却可以对 Qt 程序的结构有一个清楚的认识。现在再把代码贴过来：

```cpp
 #include <QApplication> 

 #include <QLabel> 

int main(int argc, char *argv[]) 
{ 
        QApplication app(argc, argv); 
        QLabel *label = new QLabel("Hello, world!"); 
        label->show(); 
        return app.exec(); 
}
```

第 1 行和第 2 行就是需要引入的头文件。和普通的 C++ 程序没有什么两样，如果要使用某个组件，就必须要引入相应的头文件，这类似于 Java 的 import 机制。值得说明的是，Qt 中头文件和类名是一致的。也就是说，如果你要使用某个类的话，它的类名就是它的头文件名。

第 3 行是空行 :)

第 4 行是 main 函数函数头。这与普通的 C++ 程序没有什么两样，学过 C++ 的都明白。因此你可以看到，实际上，Qt 完全通过普通的 main 函数进入，这不同于 wxWidgets，因为 wxWidgets 的 Hello, world 需要你继承它的一个 wxApp 类，并覆盖它的 wxApp::OnInit 方法，系统会自动将 OnInit 编译成入口函数。不过在 Qt 中，就不需要这些了。

第 5 行，噢噢，大括号…

第 6 行，创建一个 QApplication 对象。这个对象用于管理应用程序级别的资源。QApplication 的构造函数要求两个参数，分别来自 main 的那两个参数，因此，Qt 在一定程度上是支持命令行参数的。

第 7 行，创建一个 QLabel 对象，并且能够显示 Hello, world!字符串。和其他库的 Label 控件一样，这是用来显示文本的。在 Qt 中，这被称为一个 widget(翻译出来是小东西，不过这个翻译并不好…)，它等同于 Windows 技术里面的控件(controls)和容器(containers)。也就是说，widget 可以放置其他的 widget，就像 Swing 的组件。大多数 Qt 程序使用 QMainWindow 或者 QDialog 作为顶级组件，但 Qt 并不强制要求这点。在这个例子中，顶级组件就是一个 QLabel。

第 8 行，使这个 label 可见。组件创建出来之后通常是不可见的，要求我们手动的使它们可见。这样，在创建出组建之后我们就可以对它们进行各种定制，以避免出现之后在屏幕上面会有闪烁。

第 9 行，将应用程序的控制权移交给 Qt。这时，程序的事件循环就开始了，也就是说，这时可以相应你发出的各种事件了。这类似于 gtk+ 最后的一行 gtk_main()。

第 10 行，大括号……程序结束了。

注意，我们并没有使用 delete 去删除创建的 QLabel，因为在程序结束后操作系统会回收这个空间——这只是因为这个 QLabel 占用的内存比较小，但有时候这么做会引起麻烦的，特别是在大程序中，因此必须小心。

好了，程序解释完了。按照正常的流程，下面应该编译。前面也提过，Qt 的编译不能使用普通的 make，而必须先使用 qmake 进行预编译。所以，第一步应该是在工程目录下使用

qmake -project

命令创建.pro 文件(比如说是叫 helloworld.pro)。然后再在.pro 文件目录下使用

qmake helloworld.pro (make)

或者

qmake -tp vc helloworld.pro (nmake)

生成 makefile，然后才能调用 make 或者是 nmake 进行编译。不过因为我们使用的是 IDE，所以这些步骤就不需要我们手动完成了。

值得说明一点的是，这个 qmake 能够生成标准的 makefile 文件，因此完全可以利用 qmake 自动生成 makefile——这是题外话。

好了，下面修改一下源代码，把 QLabel 的创建一句改成

```cpp
QLabel *label = new QLabel("<h2><font color='red'>Hello</font>, world!<h2>");
```

运行一下：

![](img/8.png)

同 Swing 的 JLabel 一样，Qt 也是支持 HTML 解析的。

好了，这个 Hello, world 就说到这里！明确一下 Qt 的程序结构，在一个 Qt 源代码中，一下两条语句是必不可少的：

```cpp
QApplication app(argc, argv); 
//... 
return app.exec();
```

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/194031`](http://devbean.blog.51cto.com/448512/194031)