# 二十一、event()

今天要说的是 event()函数。记得之前曾经提到过这个函数，说在事件对象创建完毕后，Qt 将这个事件对象传递给 QObject 的 event()函数。event()函数并不直接处理事件，而是将这些事件对象按照它们不同的类型，分发给不同的事件处理器(event handler)。

event()函数主要用于事件的分发，所以，如果你希望在事件分发之前做一些操作，那么，就需要注意这个 event()函数了。为了达到这种目的，我们可以重写 event()函数。例如，如果你希望在窗口中的 tab 键按下时将焦点移动到下一组件，而不是让具有焦点的组件处理，那么你就可以继承 QWidget，并重写它的 event()函数，已达到这个目的：

```cpp
bool MyWidget::event(QEvent *event) {
        if (event->type() == QEvent::KeyPress) {
                QKeyEvent *keyEvent = static_cast<QKeyEvent *>(event);
                if (keyEvent->key() == Qt::Key_Tab) {
                        // 处理 Tab 鍵
                        return true;
                }
        }

        return QWidget::event(event);
}
```

event()函数接受一个 QEvent 对象，也就是需要这个函数进行转发的对象。为了进行转发，必定需要有一系列的类型判断，这就可以调用 QEvent 的 type()函数，其返回值是 QEvent::Type 类型的枚举。我们处理过自己需要的事件后，可以直接 return 回去，对于其他我们不关心的事件，需要调用父类的 event()函数继续转发，否则这个组件就只能处理我们定义的事件了。

event()函数返回值是 bool 类型，如果传入的事件已被识别并且处理，返回 true，否则返回 false。如果返回值是 true，QApplication 会认为这个事件已经处理完毕，会继续处理事件队列中的下一事件；如果返回值是 false，QApplication 会尝试寻找这个事件的下一个处理函数。

event()函数的返回值和事件的 accept()和 ignore()函数不同。accept()和 ignore()函数用于不同的事件处理器之间的沟通，例如判断这一事件是否处理；event()函数的返回值主要是通知 QApplication 的 notify()函数是否处理下一事件。为了更加明晰这一点，我们来看看 QWidget 的 event()函数是如何定义的：

```cpp
bool QWidget::event(QEvent *event) {
        switch (e->type()) {
        case QEvent::KeyPress:
                 keyPressEvent((QKeyEvent *)event);
                if (!((QKeyEvent *)event)->isAccepted())
                        return false;
                break;
        case QEvent::KeyRelease:
                keyReleaseEvent((QKeyEvent *)event);
                if (!((QKeyEvent *)event)->isAccepted())
                        return false;
                break;
                // more...
        }
        return true;
}
```

QWidget 的 event()函数使用一个巨大的 switch 来判断 QEvent 的 type，并且分发给不同的事件处理函数。在事件处理函数之后，使用这个事件的 isAccepted()方法，获知这个事件是不是被接受，如果没有被接受则 event()函数立即返回 false，否则返回 true。

另外一个必须重写 event()函数的情形是有自定义事件的时候。如果你的程序中有自定义事件，则必须重写 event()函数以便将自定义事件进行分发，否则你的自定义事件永远也不会被调用。关于自定义事件，我们会在以后的章节中介绍。

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/194031`](http://devbean.blog.51cto.com/448512/194031)