# 二十、事件接收与忽略

本章内容也是关于 Qt 事件。或许这一章不能有一个完整的例子，因为对于事件总是感觉很抽象，还是从底层上理解一下比较好的吧！

前面说到了事件的作用，下面来看看我们如何来接收事件。回忆一下前面的代码，我们在子类中重写了事件函数，以便让这些子类按照我们的需要完成某些功能，就像下面的代码：

```cpp
void MyLabel::mousePressEvent(QMouseEvent * event)
{
        if(event->button() == Qt::LeftButton) {
                // do something
        } else {
                QLabel::mousePressEvent(event);
        }
}
```

上面的代码和前面类似，在鼠标按下的事件中检测，如果按下的是左键，做我们的处理工作，如果不是左键，则调用父类的函数。这在某种程度上说，是把事件向上传递给父类去响应，也就是说，我们在子类中“忽略”了这个事件。

我们可以把 Qt 的事件传递看成链状：如果子类没有处理这个事件，就会继续向其他类传递。其实，Qt 的事件对象都有一个 accept()函数和 ignore()函数。正如它们的名字，前者用来告诉 Qt，事件处理函数“接收”了这个事件，不要再传递；后者则告诉 Qt，事件处理函数“忽略”了这个事件，需要继续传递，寻找另外的接受者。在事件处理函数中，可以使用 isAccepted()来查询这个事件是不是已经被接收了。

事实上，我们很少使用 accept()和 ignore()函数，而是想上面的示例一样，如果希望忽略事件，只要调用父类的响应函数即可。记得我们曾经说过，Qt 中的事件大部分是 protected 的，因此，重写的函数必定存在着其父类中的响应函数，这个方法是可行的。为什么要这么做呢？因为我们无法确认父类中的这个处理函数没有操作，如果我们在子类中直接忽略事件，Qt 不会再去寻找其他的接受者，那么父类的操作也就不能进行，这可能会有潜在的危险。另外我们查看一下 QWidget 的 mousePressEvent()函数的实现：

```cpp
void QWidget::mousePressEvent(QMouseEvent *event)
{
        event->ignore();
        if ((windowType() == Qt::Popup)) {
                event->accept();
                QWidget* w;
                while ((w = qApp->activePopupWidget()) && w != this){
                        w->close();
                        if (qApp->activePopupWidget() == w) // widget does not want to dissappear
                                w->hide(); // hide at least
                }
                if (!rect().contains(event->pos())){
                        close();
                }
        }
}
```

请注意第一条语句，如果所有子类都没有覆盖 mousePressEvent 函数，这个事件会在这里被忽略掉，这暗示着这个组件不关心这个事件，这个事件就可能被传递给其父组件。

不过，事情也不是绝对的。在一个情形下，我们必须使用 accept()和 ignore()函数，那就是在窗口关闭的时候。如果你在窗口关闭时需要有个询问对话框，那么就需要这么去写：

```cpp
void MainWindow::closeEvent(QCloseEvent * event)
{
        if(continueToClose()) {
                event->accept();
        } else {
                event->ignore();
        }
}

bool MainWindow::continueToClose()
{
        if(QMessageBox::question(this,
                                            tr("Quit"),
                                            tr("Are you sure to quit this application?"),
                                            QMessageBox::Yes | QMessageBox::No,
                                            QMessageBox::No)
                == QMessageBox::Yes) {
                return true;
        } else {
                return false;
        }
}
```

这样，我们经过询问之后才能正常退出程序。

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/194031`](http://devbean.blog.51cto.com/448512/194031)