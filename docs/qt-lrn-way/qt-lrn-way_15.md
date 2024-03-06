# 十五、Qt 标准对话框之 QColorDialog

继续来说 Qt 的标准对话框，这次说说 QColorDialog。这是 Qt 提供的颜色选择对话框。

使用 QColorDialog 也很简单，Qt 提供了 getColor()函数，类似于 QFileDialog 的 getOpenFileName()，可以直接获得选择的颜色。我们还是使用前面的 QAction 来测试下这个函数：

```cpp
        QColor color = QColorDialog::getColor(Qt::white, this); 
        QString msg = QString("r: %1, g: %2, b: %3").arg(QString::number(color.red()), QString::number(color.green()), QString::number(color.blue())); 
        QMessageBox::information(NULL, "Selected color", msg);
```

不要忘记 include QColorDialog 哦！这段代码虽然很少，但是内容并不少。

第一行 QColorDialog::getColor()调用了 QColorDialog 的 static 函数 getColor()。这个函数有两个参数，第一个是 QColor 类型，是对话框打开时默认选择的颜色，第二个是它的 parent。

第二行比较长，涉及到 QString 的用法。如果我没记错的话，这些用法还没有提到过，本着“有用就说”的原则，尽管这些和 QColorDialog 毫不相干，这里还是解释一下。QString("r: %1, g: %2, b: %3")创建了一个 QString 对象。我们使用了参数化字符串，也就是那些%1 之类。在 Java 的 properties 文件中，字符参数是用{0}, {1}之类实现的。其实这都是一些占位符，也就是，后面会用别的字符串替换掉这些值。占位符的替换需要使用 QString 的 arg()函数。这个函数会返回它的调用者，因此可以使用链式调用写法。它会按照顺序替换掉占位符。然后是 QString::number()函数，这也是 QString 的一个 static 函数，作用就是把 int、double 等值换成 QString 类型。这里是把 QColor 的 R、G、B 三个值输出了出来。关于 QString 类，我们会在以后详细说明。

第三行就比较简单了，使用一个消息对话框把刚刚拼接的字符串输出。

现在就可以运行这个测试程序了。看上去很简单，不是吗？

QColorDialog 还有一些其他的函数可以使用。

QColorDialog::setCustomColor()可以设置用户自定义颜色。这个函数有两个值，第一个是自定义颜色的索引，第二个是自定义颜色的 RGB 值，类型是 QRgb，大家可以查阅 API 文档来看看这个类的使用，下面只给出一个简单的用发：

```cpp
QColorDialog::setCustomColor(0, QRgb(0x0000FF));
```

getColor()还有一个重载的函数，签名如下:

```cpp
QColorDialog::getColor( const QColor & initial, QWidget * parent, const QString & title, ColorDialogOptions options = 0 )
```

第一个参数 initial 和前面一样，是对话框打开时的默认选中的颜色；

第二个参数 parent，设置对话框的父组件；

第三个参数 title，设置对话框的 title；

第四个参数 options，是 QColorDialog::ColorDialogOptions 类型的，可以设置对话框的一些属性，如是否显示 Alpha 值等，具体属性请查阅 API 文档。特别的，这些值是可以使用 OR 操作的。

QColorDialog 相对简单一些，API 文档也很详细，大家遇到问题可以查阅文档的哦！

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/194031`](http://devbean.blog.51cto.com/448512/194031)