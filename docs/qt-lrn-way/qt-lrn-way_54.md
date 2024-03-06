# 五十四、拖放技术之二

很长时间没有来写博客了，前段时间一直在帮同学弄一个 spring-mvc 的项目，今天终于做完了，不过公司里面又要开始做 flex 4，估计还会忙一段时间吧！

接着上次的说，上次说到了拖放技术，今天依然是一个例子，同样是来自《C++ GUI Programming with Qt 4, 2nd Edition》的。

这次的 demo 还算是比较实用：实现的是两个 list 之间的数据互拖。在很多项目中，这一需求还是比较常见的吧！下面也就算是抛砖引玉了啊！

projectlistwidget.h

```cpp

 #ifndef PROJECTLISTWIDGET_H  

 #define PROJECTLISTWIDGET_H  

 #include <QtGui>  

class ProjectListWidget : public QListWidget  
{  
    Q_OBJECT  

public:  
    ProjectListWidget(QWidget *parent = 0);  

protected:  
    void mousePressEvent(QMouseEvent *event);  
    void mouseMoveEvent(QMouseEvent *event);  
    void dragEnterEvent(QDragEnterEvent *event);  
    void dragMoveEvent(QDragMoveEvent *event);  
    void dropEvent(QDropEvent *event);  

private:  
    void performDrag();  

    QPoint startPos;  
};  

 #endif // PROJECTLISTWIDGET_H
```

projectlistwidget.cpp

```cpp

 #include "projectlistwidget.h"  

ProjectListWidget::ProjectListWidget(QWidget *parent)  
    : QListWidget(parent)  
{  
    setAcceptDrops(true);  
}  

void ProjectListWidget::mousePressEvent(QMouseEvent *event)  
{  
    if (event->button() == Qt::LeftButton)  
        startPos = event->pos();  
    QListWidget::mousePressEvent(event);  
}  

void ProjectListWidget::mouseMoveEvent(QMouseEvent *event)  
{  
    if (event->buttons() & Qt::LeftButton) {  
        int distance = (event->pos() - startPos).manhattanLength();  
        if (distance >= QApplication::startDragDistance())  
            performDrag();  
    }  
    QListWidget::mouseMoveEvent(event);  
}  

void ProjectListWidget::performDrag()  
{  
    QListWidgetItem *item = currentItem();  
    if (item) {  
        QMimeData *mimeData = new QMimeData;  
        mimeData->setText(item->text());  

        QDrag *drag = new QDrag(this);  
        drag->setMimeData(mimeData);  
        drag->setPixmap(QPixmap(":/images/person.png"));  
        if (drag->exec(Qt::MoveAction) == Qt::MoveAction)  
            delete item;  
    }  
}  

void ProjectListWidget::dragEnterEvent(QDragEnterEvent *event)  
{  
    ProjectListWidget *source =  
            qobject_cast<ProjectListWidget *>(event->source());  
    if (source && source != this) {  
        event->setDropAction(Qt::MoveAction);  
        event->accept();  
    }  
}  

void ProjectListWidget::dragMoveEvent(QDragMoveEvent *event)  
{  
    ProjectListWidget *source =  
            qobject_cast<ProjectListWidget *>(event->source());  
    if (source && source != this) {  
        event->setDropAction(Qt::MoveAction);  
        event->accept();  
    }  
}  

void ProjectListWidget::dropEvent(QDropEvent *event)  
{  
    ProjectListWidget *source =  
            qobject_cast<ProjectListWidget *>(event->source());  
    if (source && source != this) {  
        addItem(event->mimeData()->text());  
        event->setDropAction(Qt::MoveAction);  
        event->accept();  
    }  
}
```

我们从构造函数开始看起。Qt 中很多组件是可以接受拖放的，但是默认动作都是不允许的，因此在构造函数中，我们调用 setAcceptDrops(true); 函数，让组件能够接受拖放事件。

在 mousePressEvent() 函数中，我们检测鼠标左键点击，如果是的话就记录下当前位置。需要注意的是，这个函数最后需要调用系统自带的处理函数，以便实现通常的那种操作。这在一些重写事件的函数中都是需要注意的！

然后我们重写了 mouseMoveEvent() 事件。下面还是先来看看代码：

```cpp

void ProjectListWidget::mouseMoveEvent(QMouseEvent *event)  
{  
    if (event->buttons() & Qt::LeftButton) {  
        int distance = (event->pos() - startPos).manhattanLength();  
        if (distance >= QApplication::startDragDistance())  
            performDrag();  
    }  
    QListWidget::mouseMoveEvent(event);  
}
```

在这里判断了如果鼠标拖动的时候一直按住左键(也就是 if 里面的内容)，那么就计算一个 manhattanLength() 值。从字面上翻译，这是个“曼哈顿长度”。这是什么意思呢？我们看一下 event.pos() - startPos 是什么。还记得在 mousePressEvent() 函数中，我们将鼠标按下的坐标记录为 startPos，而 event.pos() 则是鼠标当前的坐标：一个点减去另外一个点，没错，这就是向量！其实，所谓曼哈顿距离就是两点之间的距离(至于为什么叫这个奇怪的名字，大家查查百科就知道啦！)，也就是这个向量的长度。下面又是一个判断，如果大于 QApplication::startDragDistance()，我们才进行 drag 的操作。当然，最后还是要调用系统默认的鼠标拖动函数。这一判断的意义在于，防止用户因为手的抖动等因素造成的鼠标拖动。用户必须将鼠标拖动一段距离之后，我们才认为他是希望进行拖动操作，而这一距离就是 QApplication::startDragDistance() 提供的，这个值通常是 4px。

performDrag() 开始处理拖放过程。我们创建了一个 QDrag 对象，将 this 作为 parent。QDrag 使用 QMimeData 存储数据。例如我们使用 QMimeData::setText() 函数将一个字符串存储为 text/plain 类型的数据。QMimeData 提供了很多函数，用于存储诸如 URL、颜色等类型的数据。使用 QDrag::setPixmap() 则可以设置拖动发生时鼠标的样式。QDrag::exec() 会阻塞拖动的操作，直到用户完成操作或者取消操作。它接受不同类型的动作作为参数，返回值是真正执行的动作。这些动作的类型为 Qt::CopyAction，Qt::MoveAction 和 Qt::LinkAction。返回值会有这三种动作，同时增加一个 Qt::IgnoreAction 用于表示用户取消了拖放。这些动作取决于拖放源对象允许的类型，目的对象接受的类型以及拖放时按下的键盘按键。在 exec() 调用之后，Qt 会在拖放对象不需要的时候 delete 掉它。

ProjectListWidget 不仅能够发出拖动事件，而且能够接受同一应用程序中的不同 ProjectListWidget 对象的数据。在 dragEnterEvent() 中，我们使用 event->source() 获取这样的对象：如果拖放数据来自同一类型的对象，并且来自同一应用程序则返回其指针，否则返回 NULL。我们使用 qobject_cast 宏将指针转换成 ProjectListWidget* 类型，然后设置接受 Qt::MoveAction 类型的拖动。dragMoveEvent() 则和这个函数具有相同的代码，因为我们需要重写拖动移动的代码。

最后在 dropEvent() 函数中，我们取出 QDrag 中的 mimeData 数据，调用 addItem() 添加到当前的列表中。这样，一个相对完整的拖放的代码就完成了。

拖放技术是 Qt 中功能强大的一个技术，但是对于不涉及数据的同一组件中拖动，或许仅仅简单的实现 mouse event 就足够了，具体还是要自己斟酌啦！

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/193918`](http://devbean.blog.51cto.com/448512/193918)