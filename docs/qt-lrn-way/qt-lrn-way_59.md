# 五十九、进程间交互

所谓 IO 其实不过是与其他设备之间的数据交互。在 Linux 上这个概念或许会更加清楚一些。Linux 把所有设备都看作是一种文件，因此所有的 IO 都归结到对文件的数据交互。同样，与其他进程之间也存在着数据交互，这就是进程间交互。

为什么需要进程间交互呢？Qt 虽然是一个很庞大的库，但是也不能面面俱到。每个需求都提供一种解决方案是不现实的。比如操作系统提供了查看当前文件夹下所有文件的命令(Windows 下是 dir， Linux 下是 ls)，那么 Qt 就可以通过调用这个命令获取其中的信息。当然这不是一个很恰当的例子，因为 Qt 同样提供了相同的操作。不过，如果你使用版本控制系统，比如 SVN，然后你希望通过 SVN 的版本号生成自己系统的 build number，那么就不得不调用 svn 命令获取当前仓库的版本号。这些操作都涉及到进程间交互。

Qt 使用 QProcess 类完成进程间交互。我们从例子开始看起。由于比较简单，所以没有把 main() 函数贴上来，大家顺手写下就好的啦！

mainwindow.h

```cpp

 #ifndef MAINWINDOW_H  

 #define MAINWINDOW_H  

 #include <QtGui>  

class MainWindow : public QMainWindow  
{  
    Q_OBJECT  

public:  
    MainWindow(QWidget *parent = 0);  
    ~MainWindow();  

private slots:  
    void openProcess();  

private:  
    QProcess *p;  
};  

 #endif // MAINWINDOW_H
```

mainwindow.cpp

```cpp

 #include "mainwindow.h"  

MainWindow::MainWindow(QWidget *parent)  
    : QMainWindow(parent)  
{  
    p = new QProcess(this);  
    QPushButton *bt = new QPushButton("execute notepad", this);  
    connect(bt, SIGNAL(clicked()), this, SLOT(openProcess()));  
}  

MainWindow::~MainWindow()  
{  

}  

void MainWindow::openProcess()  
{  
    p->start("notepad.exe");  
}
```

这个窗口很简单，只有一个按钮，当你点击按钮之后，程序会调用 Windows 的记事本。这里我们使用的是

```cpp

p->start("notepad.exe");
```

语句。QProcess::start() 接受两个参数，第一个是要执行的命令或者程序，这里就是 notepad.exe；第二个是一个 QStringList 类型的数据，也就是需要传递给这个程序的运行参数。注意，这个程序是需要能够由系统找到的，一般是完全路径。但是这里为什么只有 notepad.exe 呢？因为这个程序实际是放置在 Windows 系统文件夹下，是已经添加到了系统路径之中，因此不需要再添加本身的路径。

下面我们再看一个更复杂的例子，调用一个系统命令，这里我使用的是 Windows，因此需要调用 dir；如果你是在 Linux 进行编译，就需要改成 ls 了。

mainwindow.h

```cpp

 #ifndef MAINWINDOW_H  

 #define MAINWINDOW_H  

 #include <QtGui>  

class MainWindow : public QMainWindow  
{  
    Q_OBJECT  

public:  
    MainWindow(QWidget *parent = 0);  
    ~MainWindow();  

private slots:  
    void openProcess();  
    void readResult(int exitCode);  

private:  
    QProcess *p;  
};  

 #endif // MAINWINDOW_H
```

mainwindow.cpp

```cpp

 #include "mainwindow.h"  

MainWindow::MainWindow(QWidget *parent)  
    : QMainWindow(parent)  
{  
    p = new QProcess(this);  
    QPushButton *bt = new QPushButton("execute notepad", this);  
    connect(bt, SIGNAL(clicked()), this, SLOT(openProcess()));  
}  

MainWindow::~MainWindow()  
{  

}  

void MainWindow::openProcess()  
{  
    p->start("cmd.exe", QStringList() << "/c" << "dir");  
    connect(p, SIGNAL(finished(int)), this, SLOT(readResult(int)));  
}  

void MainWindow::readResult(int exitCode)  
{  
    if(exitCode == 0) {  
        QTextCodec* gbkCodec = QTextCodec::codecForName("GBK");  
        QString result = gbkCodec->toUnicode(p->readAll());  
        QMessageBox::information(this, "dir", result);  
    }  
}
```

我们仅增加了一个 slot 函数。在按钮点击的 slot 中，我们通过 QProcess::start() 函数运行了指令 cmd.exe /c dir 这里是说，打开系统的 cmd 程序，然后运行 dir 指令。如果有对参数 /c 有疑问，只好去查阅 Windows 的相关手册了哦，这已经不是 Qt 的问题了。然后我们 process 的 finished() 信号连接到新增加的 slot 上面。这个 signal 有一个参数 int。我们知道，对于 C/C++ 程序而言，main() 函数总是返回一个 int，也就是退出代码，用于指示程序是否正常退出。这里的 int 参数就是这个退出代码。在 slot 中，我们检查退出代码是否是 0，一般而言，如果退出代码为 0，说明是正常退出。然后把结果显示在 QMessageBox 中。怎么做到的呢？原来，QProcess::readAll() 函数可以读出程序输出内容。我们使用这个函数将所有的输出获取之后，由于它的返回结果是 QByteArray 类型，所以再转换成 QString 显示出来。另外注意一点，中文本 Windows 使用的是 GBK 编码，而 Qt 使用的是 Unicode 编码，因此需要做一下转换，否则是会出现乱码的，大家可以尝试一下。

好了，进程间交互就说这么说，通过查看文档你可以找到如何用 QProcess 实现进程过程的监听，或者是令 Qt 程序等待这个进程结束后再继续执行的函数。

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/193918`](http://devbean.blog.51cto.com/448512/193918)