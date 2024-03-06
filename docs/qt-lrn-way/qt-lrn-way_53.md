# 五十三、拖放技术之一

拖放 Drag and Drop，有时又被称为 DnD，是现代软件开发中必不可少的一项技术。它提供了一种能够在应用程序内部甚至是应用程序之间进行信息交换的机制，并且，操作系统与应用程序之间进行剪贴板的内容交换，也可以被认为是 DnD 的一部分。

DnD 其实是由两部分组成的：Drag 和 Drop。Drag 是将被拖放对象“拖动”，Drop 是将被拖放对象“放下”，前者一般是一个按下鼠标的过程，而后者则是一个松开鼠标的过程，这两者之间鼠标一直是被按下的。当然，这只是一种通常的情况，其他情况还是要看应用程序的具体实现。对于 Qt 而言，widget 既可以作为 drag 对象，也可以作为 drop 对象，或者二者都是。

下面的一个例子来自 C++ GUI Programming with Qt 4, 2nd Edition。在这个例子中，我们创建一个程序，可以将系统中的文本文件拖放过来，然后在窗口中读取内容。

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

protected:  
    void dragEnterEvent(QDragEnterEvent *event);  
    void dropEvent(QDropEvent *event);  

private:  
    bool readFile(const QString &fileName);  
    QTextEdit *textEdit;  
};  

 #endif // MAINWINDOW_H
```

mainwindow.cpp

```cpp

 #include "mainwindow.h"  

MainWindow::MainWindow(QWidget *parent)  
    : QMainWindow(parent)  
{  
    textEdit = new QTextEdit;  
    setCentralWidget(textEdit);  

    textEdit->setAcceptDrops(false);  
    setAcceptDrops(true);  

    setWindowTitle(tr("Text Editor"));  
}  

MainWindow::~MainWindow()  
{  
}  

void MainWindow::dragEnterEvent(QDragEnterEvent *event)  
{  
    if (event->mimeData()->hasFormat("text/uri-list")) {  
        event->acceptProposedAction();  
    }  
}  

void MainWindow::dropEvent(QDropEvent *event)  
{  
    QList<QUrl> urls = event->mimeData()->urls();  
    if (urls.isEmpty()) {  
        return;  
    }  

    QString fileName = urls.first().toLocalFile();  
    if (fileName.isEmpty()) {  
        return;  
    }  

    if (readFile(fileName)) {  
        setWindowTitle(tr("%1 - %2").arg(fileName, tr("Drag File")));  
    }  
}  

bool MainWindow::readFile(const QString &fileName)  
{  
    bool r = false;  
    QFile file(fileName);  
    QTextStream in(&file);  
    QString content;  
    if(file.open(QIODevice::ReadOnly)) {  
        in >> content;  
        r = true;  
    }  
    textEdit->setText(content);  
    return r;  
}
```

main.cpp

```cpp

 #include <QtGui/QApplication>  

 #include "mainwindow.h"  

int main(int argc, char *argv[])  
{  
    QApplication a(argc, argv);  
    MainWindow w;  
    w.show();  
    return a.exec();  
}
```

这里的代码并不是很复杂。在 MainWindow 中，一个 QTextEdit 作为窗口中间的 widget。这个类中有两个 protected 的函数：dragEnterEvent() 和 dropEvent()，这两个函数都是继承自 QWidget，看它们的名字就知道这是两个事件，而不仅仅是 signal。

在构造函数中，我们创建了 QTextEdit 的对象。默认情况下，QTextEdit 可以接受从其他的应用程序拖放过来的文本类型的信息。如果用户把一个文件拖到这里面，那么就会把文件名插入到文本的当前位置。但是我们希望让 MainWindow 读取文件内容，而不仅仅是插入文件名，所以我们在 MainWindow 中对 drop 事件进行了处理，因此要把 QTextEdit 的 setAcceptDrops()函数置为 false，并且把 MainWindow 的 setAcceptDrops()置为 true，以便让 MainWindow 对 drop 事件进行处理。 当用户将对象拖动到组件上面时，dragEnterEvent()函数会被回调。如果我们在事件处理代码中调用 acceptProposeAction() 函数，我们就可以向用户暗示，你可以将拖动的对象放在这个组件上。默认情况下，组件是不会接受拖放的。如果我们调用了这样的函数，那么 Qt 会自动地以光标来提示用户是否可以将对象放在组件上。在这里，我们希望告诉用户，窗口可以接受拖放。因此，我们首先检查拖放的 MIME 类型。MIME 类型为 text/uri-list 通常用来描述一个 URI 的列表。这些 URI 可以是文件名，可以是 URL 或者其他的资源描述符。MIME 类型由 Internet Assigned Numbers Authority (IANA) 定义，Qt 的拖放事件使用 MIME 类型来判断拖放对象的类型。关于 MIME 类型的详细信息，请参考 http://www.iana.org/assignments/media-types/.

当用户将对象释放到组件上面时，dropEvent() 函数会被回调。我们使用 QMimeData::urls()来或者 QUrl 的一个 list。通常，这种拖动应该只用一个文件，但是也不排除多个文件一起拖动。因此我们需要检查这个 list 是否为空，如果不为空，则取出第一个。如果不成立，则立即返回。最后我们调用 readFile() 函数读取文件内容。关于读取操作我们会在以后的章节中详细说明，这里不再赘述。 好了，至此我们的小程序就解释完毕了，运行一下看看效果吧！

对于拖动和脱离，Qt 也提供了类似的函数：dragMoveEvent() 和 dragLeaveEvent()，不过对于大部分应用而言，这两个函数的使用率要小得多。

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/193918`](http://devbean.blog.51cto.com/448512/193918)