# 十四、Qt 标准对话框之 QFileDialog

《Qt 学习之路》已经写到了第 15 篇，然而现在再写下去却有点困难，原因是当初并没有想到会连续的写下去，因此并没有很好的计划这些内容究竟该怎样去写。虽然前面说过，本教程主要线路参考《C++ Gui Programming with Qt 4, 2nd Edition》，然而最近的章节由于原文是一个比较完整的项目而有所改变，因此现在不知道该从何写起。

我并不打算介绍很多组件的使用，因为 Qt 有很多组件，各种组件用法众多，根本不可能介绍完，只能把 API 放在手边，边用边查。所以，对于很多组件我只是简单的介绍一下，具体用法还请自行查找(确切地说，我知道的也并不多，很多时候还是要到 API 里面去找)。

下面还是按照我们的进度，从 Qt 的标准对话框开始说起。所谓标准对话框，其实就是 Qt 内置的一些对话框，比如文件选择、颜色选择等等。今天首先介绍一下 QFileDialog。

QFileDialog 是 Qt 中用于文件打开和保存的对话框，相当于 Swing 里面的 JFileChooser。下面打开我们前面使用的工程。我们已经很有先见之明的写好了一个打开的 action，还记得前面的代码吗？当时，我们只是弹出了一个消息对话框(这也是一种标准对话框哦~)用于告知这个信号槽已经联通，现在我们要写真正的打开代码了！

修改 MainWindow 的 open 函数：

```cpp
void MainWindow::open() 
{ 
        QString path = QFileDialog::getOpenFileName(this, tr("Open Image"), ".", tr("Image Files(*.jpg *.png)")); 
        if(path.length() == 0) { 
                QMessageBox::information(NULL, tr("Path"), tr("You didn't select any files.")); 
        } else { 
                QMessageBox::information(NULL, tr("Path"), tr("You selected ") + path); 
        } 
}
```

编译之前别忘记 include QFileDialog 哦！然后运行一下吧！点击打开按钮，就会弹出打开对话框，然后选择文件或者直接点击取消，会有相应的消息提示。

QFileDialog 提供了很多静态函数，用于获取用户选择的文件。这里我们使用的是 getOpenFileName(), 也就是“获取打开文件名”，你也可以查看 API 找到更多的函数使用。不过，这个函数的参数蛮长的，而且类型都是 QString，并不好记。考虑到这种情况，Qt 提供了另外的写法：

```cpp

        QFileDialog *fileDialog = new QFileDialog(this); 
        fileDialog->setWindowTitle(tr("Open Image")); 
        fileDialog->setDirectory("."); 
        fileDialog->setFilter(tr("Image Files(*.jpg *.png)")); 
        if(fileDialog->exec() == QDialog::Accepted) { 
                QString path = fileDialog->selectedFiles()[0]; 
                QMessageBox::information(NULL, tr("Path"), tr("You selected ") + path); 
        } else { 
                QMessageBox::information(NULL, tr("Path"), tr("You didn't select any files.")); 
        }
```

不过，这两种写法虽然功能差别不大，但是弹出的对话框却并不一样。getOpenFileName()函数在 Windows 和 MacOS X 平台上提供的是本地的对话框，而 QFileDialog 提供的始终是 Qt 自己绘制的对话框(还记得前面说过，Qt 的组件和 Swing 类似，也是自己绘制的，而不都是调用系统资源 API)。

为了说明 QFileDialog::getOpenFileName()函数的用法，还是先把函数签名放在这里：

```cpp
QString QFileDialog::getOpenFileName (
          QWidget * parent = 0,
          const QString & caption = QString(),
          const QString & dir = QString(),
          const QString & filter = QString(),
          QString * selectedFilter = 0,
          Options options = 0 )
```

第一个参数 parent，用于指定父组件。注意，很多 Qt 组件的构造函数都会有这么一个 parent 参数，并提供一个默认值 0；

第二个参数 caption，是对话框的标题；

第三个参数 dir，是对话框显示时默认打开的目录，"." 代表程序运行目录，"/" 代表当前盘符的根目录(Windows，Linux 下/就是根目录了)，也可以是平台相关的，比如"C:\"等；

第四个参数 filter，是对话框的后缀名过滤器，比如我们使用"Image Files(*.jpg *.png)"就让它只能显示后缀名是 jpg 或者 png 的文件。如果需要使用多个过滤器，使用";;"分割，比如"JPEG Files(*.jpg);;PNG Files(*.png)"；

第五个参数 selectedFilter，是默认选择的过滤器；

第六个参数 options，是对话框的一些参数设定，比如只显示文件夹等等，它的取值是 enum QFileDialog::Option，每个选项可以使用 | 运算组合起来。

如果我要想选择多个文件怎么办呢？Qt 提供了 getOpenFileNames()函数，其返回值是一个 QStringList。你可以把它理解成一个只能存放 QString 的 List，也就是 STL 中的 list<string>。</string>

好了，我们已经能够选择打开文件了。保存也是类似的，QFileDialog 类也提供了保存对话框的函数 getSaveFileName，具体使用还是请查阅 API。

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/194031`](http://devbean.blog.51cto.com/448512/194031)