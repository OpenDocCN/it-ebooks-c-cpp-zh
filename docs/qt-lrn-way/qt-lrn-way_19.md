# 十九、事件(event)

前面说了几个标准对话框，下面不打算继续说明一些组件的使用，因为这些使用很难讲完，很多东西都是与实际应用相关的。实际应用的复杂性决定了我们根本不可能把所有组件的所有使用方法都说明白。这次来说说 Qt 相对高级一点的特性：事件。

事件(event)是有系统或者 Qt 本身在不同的时刻发出的。当用户按下鼠标，敲下键盘，或者是窗口需要重新绘制的时候，都会发出一个相应的事件。一些事件是在对用户操作做出响应的时候发出，如键盘事件等；另一些事件则是由系统自动发出，如计时器事件。

一般来说，使用 Qt 编程时，我们并不会把主要精力放在事件上，因为在 Qt 中，需要我们关心的事件总会发出一个信号。比如，我们关心的是 QPushButton 的鼠标点击，但我们不需要关心这个鼠标点击事件，而是关心它的 clicked()信号。这与其他的一些框架不同：在 Swing 中，你所要关心的是 JButton 的 ActionListener 这个点击事件。

Qt 的事件很容易和信号槽混淆。这里简单的说明一下，signal 由具体对象发出，然后会马上交给由 connect 函数连接的 slot 进行处理；而对于事件，Qt 使用一个事件队列对所有发出的事件进行维护，当新的事件产生时，会被追加到事件队列的尾部，前一个事件完成后，取出后面的事件进行处理。但是，必要的时候，Qt 的事件也是可以不进入事件队列，而是直接处理的。并且，事件还可以使用“事件过滤器”进行过滤。总的来说，如果我们使用组件，我们关心的是信号槽；如果我们自定义组件，我们关心的是事件。因为我们可以通过事件来改变组件的默认操作。比如，如果我们要自定义一个 QPushButton，那么我们就需要重写它的鼠标点击事件和键盘处理事件，并且在恰当的时候发出 clicked()信号。

还记得我们在 main 函数里面创建了一个 QApplication 对象，然后调用了它的 exec()函数吗？其实，这个函数就是开始 Qt 的事件循环。在执行 exec()函数之后，程序将进入事件循环来监听应用程序的事件。当事件发生时，Qt 将创建一个事件对象。Qt 的所有事件都继承于 QEvent 类。在事件对象创建完毕后，Qt 将这个事件对象传递给 QObject 的 event()函数。event()函数并不直接处理事件，而是按照事件对象的类型分派给特定的事件处理函数(event handler)。关于这一点，我们会在以后的章节中详细说明。

在所有组件的父类 QWidget 中，定义了很多事件处理函数，如 keyPressEvent()、keyReleaseEvent()、mouseDoubleClickEvent()、mouseMoveEvent ()、mousePressEvent()、mouseReleaseEvent()等。这些函数都是 protected virtual 的，也就是说，我们应该在子类中重定义这些函数。下面来看一个例子。

```cpp
 #include <QApplication> 

 #include <QWidget> 

 #include <QLabel> 

 #include <QMouseEvent> 

class EventLabel : public QLabel 
{ 

protected: 
        void mouseMoveEvent(QMouseEvent *event); 
        void mousePressEvent(QMouseEvent *event); 
        void mouseReleaseEvent(QMouseEvent *event); 
}; 

void EventLabel::mouseMoveEvent(QMouseEvent *event) 
{ 
        this->setText(QString("<center><h1>Move: (%1, %2)</h1></center>") 
                                                        .arg(QString::number(event->x()), QString::number(event->y()))); 
} 

void EventLabel::mousePressEvent(QMouseEvent *event) 
{ 
        this->setText(QString("<center><h1>Press: (%1, %2)</h1></center>") 
                                                        .arg(QString::number(event->x()), QString::number(event->y()))); 
} 

void EventLabel::mouseReleaseEvent(QMouseEvent *event) 
{ 
        QString msg; 
        msg.sprintf("<center><h1>Release: (%d, %d)</h1></center>", 
                                event->x(), event->y()); 
        this->setText(msg); 
} 

int main(int argc, char *argv[]) 
{ 
        QApplication app(argc, argv); 
        EventLabel *label = new EventLabel; 
        label->setWindowTitle("MouseEvent Demo"); 
        label->resize(300, 200); 
        label->show(); 
        return app.exec(); 
}
```

这里我们继承了 QLabel 类，重写了 mousePressEvent、mouseMoveEvent 和 MouseReleaseEvent 三个函数。我们并没有添加什么功能，只是在鼠标按下(press)、鼠标移动(move)和鼠标释放(release)时把坐标显示在这个 Label 上面。注意我们在 mouseReleaseEvent 函数里面有关 QString 的构造。我们没有使用 arg 参数的方式，而是使用 C 语言风格的 sprintf 来构造 QString 对象，如果你对 C 语法很熟悉(估计很多 C++程序员都会比较熟悉的吧)，那么就可以在 Qt 中试试熟悉的 C 格式化写法啦！

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/194031`](http://devbean.blog.51cto.com/448512/194031)