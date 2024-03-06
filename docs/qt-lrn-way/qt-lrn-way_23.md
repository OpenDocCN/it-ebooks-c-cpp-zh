# 二十三、自定义事件

这部分将作为 Qt 事件部分的结束。我们在前面已经从大概上了解了 Qt 的事件机制。下面要说的是如何自定义事件。

Qt 允许你创建自己的事件类型，这在多线程的程序中尤其有用，当然，也可以用在单线程的程序中，作为一种对象间通讯的机制。那么，为什么我需要使用事件，而不是使用信号槽呢？主要原因是，事件的分发既可以是同步的，又可以是异步的，而函数的调用或者说是槽的回调总是同步的。事件的另外一个好处是，它可以使用过滤器。

Qt 中的自定义事件很简单，同其他类似的库的使用很相似，都是要继承一个类进行扩展。在 Qt 中，你需要继承的类是 QEvent。**注意，在 Qt3 中，你需要继承的类是 QCustomEvent，不过这个类在 Qt4 中已经被废除(这里的废除是不建议使用，并不是从类库中删除)。**

继承 QEvent 类，你需要提供一个 QEvent::Type 类型的参数，作为自定义事件的类型值。这里的 QEvent::Type 类型是 QEvent 里面定义的一个 enum，因此，你是可以传递一个 int 的。重要的是，你的事件类型不能和已经存在的 type 值重复，否则会有不可预料的错误发生！因为系统会将你的事件当做系统事件进行派发和调用。在 Qt 中，系统将保留 0 - 999 的值，也就是说，你的事件 type 要大于 999\. 具体来说，你的自定义事件的 type 要在 QEvent::User 和 QEvent::MaxUser 的范围之间。其中，QEvent::User 值是 1000，QEvent::MaxUser 的值是 65535。从这里知道，你最多可以定义 64536 个事件，相信这个数字已经足够大了！但是，即便如此，也只能保证用户自定义事件不能覆盖系统事件，并不能保证自定义事件之间不会被覆盖。为了解决这个问题，Qt 提供了一个函数：registerEventType(),用于自定义事件的注册。该函数签名如下：

```cpp
static int QEvent::registerEventType ( int hint = -1 );
```

函数是 static 的，因此可以使用 QEvent 类直接调用。函数接受一个 int 值，其默认值为-1，返回值是创建的这个 Type 类型的值。如果 hint 是合法的，不会发生任何覆盖，则会返回这个值；如果 hint 不合法，系统会自动分配一个合法值并返回。因此，使用这个函数即可完成 type 值的指定。这个函数是线程安全的，因此不必另外添加同步。

你可以在 QEvent 子类中添加自己的事件所需要的数据，然后进行事件的发送。Qt 中提供了两种发送方式：

static bool QCoreApplication::sendEvent(QObjecy * receiver, QEvent * event)：事件被 QCoreApplication 的 notify()函数直接发送给 receiver 对象，返回值是事件处理函数的返回值。使用这个函数必须要在栈上创建对象，例如：

```cpp
QMouseEvent event(QEvent::MouseButtonPress, pos, 0, 0, 0);
QApplication::sendEvent(mainWindow, &event);
```

static bool QCoreApplication::postEvent(QObject * receiver, QEvent * event)：事件被 QCoreApplication 追加到事件列表的最后，并等待处理，该函数将事件追加后会立即返回，并且注意，该函数是线程安全的。另外一点是，使用这个函数必须要在堆上创建对象，例如：

```cpp
QApplication::postEvent(object, new MyEvent(QEvent::registerEventType(2048)));
```

这个对象不需要手动 delete，Qt 会自动 delete 掉！因此，如果在 post 事件之后调用 delete，程序可能会崩溃。另外，postEvent()函数还有一个重载的版本，增加一个优先级参数，具体请参见 API。通过调用 sendPostedEvent()函数可以让已提交的事件立即得到处理。

如果要处理自定义事件，可以重写 QObject 的 customEvent()函数，该函数接收一个 QEvent 对象作为参数。注意，在 Qt3 中这个参数是 QCustomEvent 类型的。你可以像前面介绍的重写 event()函数的方法去重写这个函数：

```cpp
void CustomWidget::customEvent(QEvent *event) {
        CustomEvent *customEvent = static_cast<CustomEvent *>(event);
        // ....
}
```

另外，你也可以通过重写 event()函数来处理自定义事件：

```cpp
bool CustomWidget::event(QEvent *event) {
        if (event->type() == MyCustomEventType) {
                CustomEvent *myEvent = static_cast<CustomEvent *>(event);
                // processing...
                return true;
        }

        return QWidget::event(event);
}
```

这两种办法都是可行的。

好了，至此，我们已经概略的介绍了 Qt 的事件机制，包括事件的派发、自定义等一系列的问题。下面的章节将继续我们的学习之路！

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/194031`](http://devbean.blog.51cto.com/448512/194031)