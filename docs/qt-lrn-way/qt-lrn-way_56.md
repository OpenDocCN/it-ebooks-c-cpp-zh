# 五十六、剪贴板操作

剪贴板的操作经常和前面所说的拖放技术在一起使用，因此我们现在先来说说剪贴板的相关操作。 大家对剪贴板都很熟悉。我们可以简单的把它理解成一个数据的存储池，可以把外面的数据放置进去，也可以把里面的数据取出来。剪贴板是由操作系统维护的，所以这提供了跨应用程序数据交互的一种方式。Qt 已经为我们封装好很多关于剪贴板的操作，因此我们可以在自己的应用中很容易的实现。下面还是从代码开始:

clipboarddemo.h

```cpp

 #ifndef CLIPBOARDDEMO_H  

 #define CLIPBOARDDEMO_H  

 #include <QtGui/QWidget>  

class ClipboardDemo : public QWidget  
{  
    Q_OBJECT  

public:  
    ClipboardDemo(QWidget *parent = 0);  

private slots:  
    void setClipboard();  
    void getClipboard();  
};  

 #endif // CLIPBOARDDEMO_H
```

clipboarddemo.cpp

```cpp

 #include <QtGui>  

 #include "clipboarddemo.h"  

ClipboardDemo::ClipboardDemo(QWidget *parent)  
    : QWidget(parent)  
{  
    QVBoxLayout *mainLayout = new QVBoxLayout(this);  
    QHBoxLayout *northLayout = new QHBoxLayout;  
    QHBoxLayout *southLayout = new QHBoxLayout;  

    QTextEdit *editor = new QTextEdit;  
    QLabel *label = new QLabel;  
    label->setText("Text Input: ");  
    label->setBuddy(editor);  
    QPushButton *copyButton = new QPushButton;  
    copyButton->setText("Set Clipboard");  
    QPushButton *pasteButton = new QPushButton;  
    pasteButton->setText("Get Clipboard");  

    northLayout->addWidget(label);  
    northLayout->addWidget(editor);  
    southLayout->addWidget(copyButton);  
    southLayout->addWidget(pasteButton);  
    mainLayout->addLayout(northLayout);  
    mainLayout->addLayout(southLayout);  

    connect(copyButton, SIGNAL(clicked()), this, SLOT(setClipboard()));  
    connect(pasteButton, SIGNAL(clicked()), this, SLOT(getClipboard()));  
}  

void ClipboardDemo::setClipboard()  
{  
    QClipboard *board = QApplication::clipboard();  
    board->setText("Text from Qt Application");  
}  

void ClipboardDemo::getClipboard()  
{  
    QClipboard *board = QApplication::clipboard();  
    QString str = board->text();  
    QMessageBox::information(NULL, "From clipboard", str);  
}
```

main.cpp

```cpp

 #include "clipboarddemo.h"  

 #include <QtGui>  

 #include <QApplication>  

int main(int argc, char *argv[])  
{  
    QApplication a(argc, argv);  
    ClipboardDemo w;  
    w.show();  
    return a.exec();  
}
```

main() 函数很简单，就是把我们的 ClipboardDemo 类显示了出来。我们重点来看 ClipboardDemo 中的代码。

构造函数同样没什么复杂的内容，我们把一个 label。一个 textedit 和两个 button 摆放到窗口中。这些代码已经能够很轻易的写出来了；然后进行了信号槽的连接。

```cpp

void ClipboardDemo::setClipboard()  
{  
    QClipboard *board = QApplication::clipboard();  
    board->setText("Text from Qt Application");  
}  

void ClipboardDemo::getClipboard()  
{  
    QClipboard *board = QApplication::clipboard();  
    QString str = board->text();  
    QMessageBox::information(NULL, "From clipboard", str);  
}
```

在 slot 函数中，我们使用 QApplication::clipboard() 函数访问到系统剪贴板。这个函数的返回值是 QClipboard 的指针。我们可以从这个类的 API 中看到，通过 setText()，setImage() 或者 setPixmap() 函数可以将数据放置到剪贴板内，也就是通常所说的剪贴或者复制的操作；使用 text()，image() 或者 pixmap() 函数则可以从剪贴板获得数据，也就是粘贴。

另外值得说的是，通过上面的例子可以看出，QTextEdit 默认就是支持 Ctrl+C, Ctrl+V 等快捷键操作的。不仅如此，很多 Qt 的组件都提供了很方便的操作，因此我们需要从文档中获取具体的信息，从而避免自己重新去发明轮子。

QClipboard 提供的数据类型很少，如果需要，我们可以继承 QMimeData 类，通过调用 setMimeData() 函数让剪贴板能够支持我们自己的数据类型。

在 X11 系统中，鼠标中键(一般就是滚轮)可以支持剪贴操作的。为了实现这一功能，我们需要向 QClipboard::text() 函数传递 QClipboard::Selection 参数。例如，我们在鼠标按键释放的事件中进行如下处理：

```cpp

void MyTextEditor::mouseReleaseEvent(QMouseEvent *event)  
{  
    QClipboard *clipboard = QApplication::clipboard();  
    if (event->button() == Qt::MidButton  
            && clipboard->supportsSelection()) {  
        QString text = clipboard->text(QClipboard::Selection);  
        pasteText(text);  
    }  
}
```

这里的 supportsSelection() 在 X11 平台返回 true，其余平台都是返回 false 的。 另外，QClipboard 提供了 dataChanged() 信号，以便监听剪贴板数据变化。

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/193918`](http://devbean.blog.51cto.com/448512/193918)