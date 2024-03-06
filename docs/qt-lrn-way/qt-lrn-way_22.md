# 二十二、事件过滤器

Qt 创建了 QEvent 事件对象之后，会调用 QObject 的 event()函数做事件的分发。有时候，你可能需要在调用 event()函数之前做一些另外的操作，比如，对话框上某些组件可能并不需要响应回车按下的事件，此时，你就需要重新定义组件的 event()函数。如果组件很多，就需要重写很多次 event()函数，这显然没有效率。为此，你可以使用一个事件过滤器，来判断是否需要调用 event()函数。

QOjbect 有一个 eventFilter()函数，用于建立事件过滤器。这个函数的签名如下：

```cpp
virtual bool QObject::eventFilter ( QObject * watched, QEvent * event )
```

如果 watched 对象安装了事件过滤器，这个函数会被调用并进行事件过滤，然后才轮到组件进行事件处理。在重写这个函数时，如果你需要过滤掉某个事件，例如停止对这个事件的响应，需要返回 true。

```cpp
bool MainWindow::eventFilter(QObject *obj, QEvent *event)
 {
         if (obj == textEdit) {
                 if (event->type() == QEvent::KeyPress) {
                         QKeyEvent *keyEvent = static_cast<QKeyEvent*>(event);
                         qDebug() << "Ate key press" << keyEvent->key();
                         return true;
                 } else {
                         return false;
                 }
         } else {
                 // pass the event on to the parent class
                 return QMainWindow::eventFilter(obj, event);
         }
 }
```

上面的例子中为 MainWindow 建立了一个事件过滤器。为了过滤某个组件上的事件，首先需要判断这个对象是哪个组件，然后判断这个事件的类型。例如，我不想让 textEdit 组件处理键盘事件，于是就首先找到这个组件，如果这个事件是键盘事件，则直接返回 true，也就是过滤掉了这个事件，其他事件还是要继续处理，所以返回 false。对于其他组件，我们并不保证是不是还有过滤器，于是最保险的办法是调用父类的函数。

在创建了过滤器之后，下面要做的是安装这个过滤器。安装过滤器需要调用 installEventFilter()函数。这个函数的签名如下：

```cpp
void QObject::installEventFilter ( QObject * filterObj )
```

这个函数是 QObject 的一个函数，因此可以安装到任何 QObject 的子类，并不仅仅是 UI 组件。这个函数接收一个 QObject 对象，调用了这个函数安装事件过滤器的组件会调用 filterObj 定义的 eventFilter()函数。例如，textField.installEventFilter(obj)，则如果有事件发送到 textField 组件是，会先调用 obj->eventFilter()函数，然后才会调用 textField.event()。

当然，你也可以把事件过滤器安装到 QApplication 上面，这样就可以过滤所有的事件，已获得更大的控制权。不过，这样做的后果就是会降低事件分发的效率。

如果一个组件安装了多个过滤器，则最后一个安装的会最先调用，类似于堆栈的行为。

**注意，如果你在事件过滤器中 delete 了某个接收组件，务必将返回值设为 true。否则，Qt 还是会将事件分发给这个接收组件，从而导致程序崩溃。**

事件过滤器和被安装的组件必须在同一线程，否则，过滤器不起作用。另外，如果在 install 之后，这两个组件到了不同的线程，那么，只有等到二者重新回到同一线程的时候过滤器才会有效。

事件的调用最终都会调用 QCoreApplication 的 notify()函数，因此，最大的控制权实际上是重写 QCoreApplication 的 notify()函数。由此可以看出，Qt 的事件处理实际上是分层五个层次：重定义事件处理函数，重定义 event()函数，为单个组件安装事件过滤器，为 QApplication 安装事件过滤器，重定义 QCoreApplication 的 notify()函数。这几个层次的控制权是逐层增大的。

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/194031`](http://devbean.blog.51cto.com/448512/194031)