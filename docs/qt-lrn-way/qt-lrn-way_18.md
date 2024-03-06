# 十八、Qt 学习之路(18): Qt 标准对话框之 QInputDialog

这是 Qt 标准对话框的最后一部分。正如同其名字显示的一样，QInputDialog 用于接收用户的输入。QInputDialog 提供了一些简单的 static 函数，用于快速的建立一个对话框，正像 QColorDialog 提供了 getColor 函数一样。

首先来看看 getText 函数：

```cpp
bool isOK; 
QString text = QInputDialog::getText(NULL, "Input Dialog", 
                                                   "Please input your comment", 
                                                   QLineEdit::Normal, 
                                                   "your comment", 
                                                   &isOK); 
if(isOK) { 
       QMessageBox::information(NULL, "Information", 
                                           "Your comment is: <b>" + text + "</b>", 
                                           QMessageBox::Yes | QMessageBox::No, 
                                           QMessageBox::Yes); 
}
```

代码比较简单，使用 getText 函数就可以弹出一个可供用户输入的对话框：

![](img/33.png)

下面来看一下这个函数的签名：

```cpp
static QString QInputDialog::getText ( QWidget * parent,
                                                      const QString & title,
                                                      const QString & label,
                                                      QLineEdit::EchoMode mode = QLineEdit::Normal,
                                                      const QString & text = QString(),
                                                      bool * ok = 0,
                                                      Qt::WindowFlags flags = 0 )
```

第一个参数 parent，也就是那个熟悉的父组件的指针；第二个参数 title 就是对话框的标题；第三个参数 label 是在输入框上面的提示语句；第四个参数 mode 用于指明这个 QLineEdit 的输入模式，取值范围是 QLineEdit::EchoMode，默认是 Normal，也就是正常显示，你也可以声明为 password，这样就是密码的输入显示了，具体请查阅 API；第五个参数 text 是 QLineEdit 的默认字符串；第六个参数 ok 是可选的，如果非 NLL，则当用户按下对话框的 OK 按钮时，这个 bool 变量会被置为 true，可以由这个去判断用户是按下的 OK 还是 Cancel，从而获知这个 text 是不是有意义；第七个参数 flags 用于指定对话框的样式。

虽然参数很多，但是每个参数的含义都比较明显，大家只要参照 API 就可以知道了。

函数的返回值是 QString，也就是用户在 QLineEdit 里面输入的内容。至于这个内容有没有意义，那就要看那个 ok 参数是不是 true 了。

QInputDialog 不仅提供了获取字符串的函数，还有 getInteger，getDouble，getItem 三个类似的函数，这里就不一一介绍。

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/194031`](http://devbean.blog.51cto.com/448512/194031)