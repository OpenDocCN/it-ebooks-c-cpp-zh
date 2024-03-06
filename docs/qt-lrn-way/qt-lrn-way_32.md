# 三十二、Qt 学习之路(32): 一个简易画板的实现(Graphics View)

这一次将介绍如何使用 Graphics View 来实现前面所说的画板。前面说了很多有关 Graphics View 的好话，但是没有具体的实例很难说究竟好在哪里。现在我们就把前面的内容使用 Graphics View 重新实现一下，大家可以对比一下看有什么区别。

同前面相似的内容就不再叙述了，我们从上次代码的基础上进行修改，以便符合我们的需要。首先来看 MainWindow 的代码：

mainwindow.cpp

```cpp

 #include "mainwindow.h" 

MainWindow::MainWindow(QWidget *parent) 
        : QMainWindow(parent) 
{ 
        QToolBar *bar = this->addToolBar("Tools"); 
        QActionGroup *group = new QActionGroup(bar); 

        QAction *drawLineAction = new QAction("Line", bar); 
        drawLineAction->setIcon(QIcon(":/line.png")); 
        drawLineAction->setToolTip(tr("Draw a line.")); 
        drawLineAction->setStatusTip(tr("Draw a line.")); 
        drawLineAction->setCheckable(true); 
        drawLineAction->setChecked(true); 
        group->addAction(drawLineAction); 

        bar->addAction(drawLineAction); 
        QAction *drawRectAction = new QAction("Rectangle", bar); 
        drawRectAction->setIcon(QIcon(":/rect.png")); 
        drawRectAction->setToolTip(tr("Draw a rectangle.")); 
        drawRectAction->setStatusTip(tr("Draw a rectangle.")); 
        drawRectAction->setCheckable(true); 
        group->addAction(drawRectAction); 
        bar->addAction(drawRectAction); 

        QLabel *statusMsg = new QLabel; 
        statusBar()->addWidget(statusMsg); 

        PaintWidget *paintWidget = new PaintWidget(this); 
        QGraphicsView *view = new QGraphicsView(paintWidget, this); 
        setCentralWidget(view); 

        connect(drawLineAction, SIGNAL(triggered()), 
                        this, SLOT(drawLineActionTriggered())); 
        connect(drawRectAction, SIGNAL(triggered()), 
                        this, SLOT(drawRectActionTriggered())); 
        connect(this, SIGNAL(changeCurrentShape(Shape::Code)), 
                        paintWidget, SLOT(setCurrentShape(Shape::Code))); 
} 

void MainWindow::drawLineActionTriggered() 
{ 
        emit changeCurrentShape(Shape::Line); 
} 

void MainWindow::drawRectActionTriggered() 
{ 
        emit changeCurrentShape(Shape::Rect); 
}
```

由于 mainwindow.h 的代码与前文相同，这里就不再贴出。而 cpp 文件里面只有少数几行与前文不同。由于我们使用 Graphics View，所以，我们必须把 item 添加到 QGprahicsScene 里面。这里，我们创建了 scene 的对象，而 scene 对象需要通过 view 进行观察，因此，我们需要再使用一个 QGraphcisView 对象，并且把这个 view 添加到 MainWindow 里面。

我们把 PaintWidget 当做一个 scene，因此 PaintWidget 现在是继承 QGraphicsScene，而不是前面的 QWidget。

paintwidget.h

```cpp

 #ifndef PAINTWIDGET_H 

 #define PAINTWIDGET_H 

 #include <QtGui> 

 #include <QDebug> 

 #include "shape.h" 

 #include "line.h" 

 #include "rect.h" 

class PaintWidget : public QGraphicsScene 
{ 
        Q_OBJECT 

public: 
        PaintWidget(QWidget *parent = 0); 

public slots: 
        void setCurrentShape(Shape::Code s) 
        { 
                if(s != currShapeCode) { 
                        currShapeCode = s; 
                } 
        } 

protected: 
        void mousePressEvent(QGraphicsSceneMouseEvent *event);
        void mouseMoveEvent(QGraphicsSceneMouseEvent *event);
        void mouseReleaseEvent(QGraphicsSceneMouseEvent *event);

private: 
        Shape::Code currShapeCode; 
        Shape *currItem; 
        bool perm; 
}; 

 #endif // PAINTWIDGET_H
```

paintwidget.cpp

```cpp

 #include "paintwidget.h" 

PaintWidget::PaintWidget(QWidget *parent) 
        : QGraphicsScene(parent), currShapeCode(Shape::Line), currItem(NULL), perm(false) 
{ 

} 

void PaintWidget::mousePressEvent(QGraphicsSceneMouseEvent *event) 
{ 
        switch(currShapeCode) 
        { 
        case Shape::Line: 
                { 
                        Line *line = new Line; 
                        currItem = line; 
                        addItem(line); 
                        break; 
                } 
        case Shape::Rect: 
                { 
                        Rect *rect = new Rect; 
                        currItem = rect; 
                        addItem(rect); 
                        break; 
                } 
        } 
        if(currItem) { 
                currItem->startDraw(event); 
                perm = false; 
        } 
        QGraphicsScene::mousePressEvent(event); 
} 

void PaintWidget::mouseMoveEvent(QGraphicsSceneMouseEvent *event) 
{ 
        if(currItem && !perm) { 
                currItem->drawing(event); 
        } 
        QGraphicsScene::mouseMoveEvent(event); 
} 

void PaintWidget::mouseReleaseEvent(QGraphicsSceneMouseEvent *event) 
{ 
        perm = true; 
        QGraphicsScene::mouseReleaseEvent(event); 
}
```

我们把继承自 QWidget 改成继承自 QGraphicsScene，同样也会有鼠标事件，只不过在这里我们把鼠标事件全部转发给具体的 item 进行处理。这个我们会在下面的代码中看到。另外一点是，每一个鼠标处理函数都包含了调用其父类函数的语句。

shape.h

```cpp

 #ifndef SHAPE_H 

 #define SHAPE_H 

 #include <QtGui> 

class Shape 
{ 
public: 

        enum Code { 
                Line, 
                Rect 
        }; 

        Shape(); 

        virtual void startDraw(QGraphicsSceneMouseEvent * event) = 0; 
        virtual void drawing(QGraphicsSceneMouseEvent * event) = 0; 
}; 

 #endif // SHAPE_H
```

shape.cpp

```cpp

 #include "shape.h" 

Shape::Shape() 
{ 
}
```

Shape 类也有了变化：还记得我们曾经说过，Qt 内置了很多 item，因此我们不必全部重写这个 item。所以，我们要使用 Qt 提供的类，就不需要在我们的类里面添加新的数据成员了。这样，我们就有了不带有额外的数据成员的 Shape。那么，为什么还要提供 Shape 呢？因为我们在 scene 的鼠标事件中需要修改这些数据成员，如果没有这个父类，我们就需要按照 Code 写一个长长的 switch 来判断是那一个图形，这样是很麻烦的。所以我们依然创建了一个公共的父类，只要调用这个父类的 draw 函数即可。

```cpp

line.h
 #ifndef LINE_H 

 #define LINE_H 

 #include <QGraphicsLineItem> 

 #include "shape.h" 

class Line : public Shape, public QGraphicsLineItem 
{ 
public: 
        Line(); 

        void startDraw(QGraphicsSceneMouseEvent * event); 
        void drawing(QGraphicsSceneMouseEvent * event); 
}; 

 #endif // LINE_H
```

line.cpp

```cpp

 #include "line.h" 

Line::Line() 
{ 
} 

void Line::startDraw(QGraphicsSceneMouseEvent * event) 
{ 
        setLine(QLineF(event->scenePos(), event->scenePos())); 
} 

void Line::drawing(QGraphicsSceneMouseEvent * event) 
{ 
        QLineF newLine(line().p1(), event->scenePos()); 
        setLine(newLine); 
}
```

Line 类已经和前面有了变化，我们不仅仅继承了 Shape，而且继承了 QGraphicsLineItem 类。这里我们使用了 C++的多继承机制。这个机制是很危险的，很容易发生错误，但是这里我们的 Shape 并没有继承其他的类，只要函数没有重名，一般而言是没有问题的。如果不希望出现不推荐的多继承(不管怎么说，多继承虽然危险，但它是符合面向对象理论的)，那就就想办法使用组合机制。我们之所以使用多继承，目的是让 Line 类同时具有 Shape 和 QGraphicsLineItem 的性质，从而既可以直接添加到 QGraphicsScene 中，又可以调用 startDraw()等函数。

同样的还有 Rect 这个类：

```cpp

rect.h
 #ifndef RECT_H 

 #define RECT_H 

 #include <QGraphicsRectItem> 

 #include "shape.h" 

class Rect : public Shape, public QGraphicsRectItem 
{ 
public: 
        Rect(); 

        void startDraw(QGraphicsSceneMouseEvent * event); 
        void drawing(QGraphicsSceneMouseEvent * event); 
}; 

 #endif // RECT_H
```

rect.cpp

```cpp

 #include "rect.h" 

Rect::Rect() 
{ 
} 

void Rect::startDraw(QGraphicsSceneMouseEvent * event) 
{ 
        setRect(QRectF(event->scenePos(), QSizeF(0, 0))); 
} 

void Rect::drawing(QGraphicsSceneMouseEvent * event) 
{ 
        QRectF r(rect().topLeft(), 
                         QSizeF(event->scenePos().x() - rect().topLeft().x(), event->scenePos().y() - rect().topLeft().y())); 
        setRect(r); 
}
```

Line 和 Rect 类的逻辑都比较清楚，和前面的基本类似。所不同的是，Qt 并没有使用我们前面定义的两个 Qpoint 对象记录数据，而是在 QGraphicsLineItem 中使用 QLineF，在 QGraphicsRectItem 中使用 QRectF 记录数据。这显然比我们的两个点的数据记录高级得多。其实，我们也完全可以使用这样的数据结构去重定义前面那些 Line 之类。

这样，我们的程序就修改完毕了。运行一下你会发现，几乎和前面的实现没有区别。这里说“几乎”，是在第一个点画下的时候，scene 会移动一段距离。这是因为 scene 是自动居中的，由于我们把 Line 的第一个点设置为(0, 0)，因此当我们把鼠标移动后会有一个偏移。

看到这里或许并没有显示出 Graphics View 的优势。不过，建议在 Line 或者 Rect 的构造函数里面加上下面的语句，

```cpp

setFlag(QGraphicsItem::ItemIsMovable, true); 
setFlag(QGraphicsItem::ItemIsSelectable, true);
```

此时，你的 Line 和 Rect 就已经支持选中和拖放了！值得试一试哦！不过，需要注意的是，我们重写了 scene 的鼠标控制函数，所以这里的拖动会很粗糙，甚至说是不正确，你需要动动脑筋重新设计我们的类啦！

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/193918`](http://devbean.blog.51cto.com/448512/193918)