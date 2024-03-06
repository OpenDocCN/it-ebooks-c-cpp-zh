# 九、Meta-Object 系统

前面说过，Qt 使用的是自己的预编译器，它提供了对 C++ 的一种扩展。利用 Qt 的信号槽机制，就可以把彼此独立的模块相互连接起来，不需要实现知道模块的任何细节。

为了达到这个目的，Qt 提出了一个 Meta-Object 系统。它提供了两个关键的作用：信号槽和内省。

面向对象程序设计里面会讲到 Smalltalk 语言有一个元类系统。所谓元类，就是这里所说的 Meta-Class。如果写过 HTML，会知道 HTML 标签里面也有一个 meta，这是用于说明页面的某些属性的。同样，Qt 的 Meta-Object 系统也是类似的作用。内省又称为反射，允许程序在运行时获得类的相关信息，也就是 meta-information。什么是 meta-information 呢？举例来说，像这个类叫什么名字？它有什么属性？有什么方法？它的信号列表？它的槽列表？等等这些信息，就是这个类的 meta-information，也就是“元信息”。这个机制还提供了对国际化的支持，是 QSA(Qt Script for Application)的基础。

标准 C++ 并没有 Qt 的 meta-information 所需要的动态 meta-information。所以，Qt 提供了一个独立的工具，moc，通过定义 Q_OBJECT 宏实现到标准 C++ 函数的转变。moc 使用纯 C++ 实现的，因此可以在任何编译器中使用。

这种机制工作过程是：

首先，Q_OBJECT 宏声明了一些 QObject 子类必须实现的内省的函数，如 metaObject()，tr(),qt_metacall()等；

第二，Qt 的 moc 工具实现 Q_OBJECT 宏声明的函数和所有信号；

第三，QObject 成员函数 connect()和 disconnect()使用这些内省函数实现信号槽的连接。

以上这些过程是 qmake，moc 和 QObject 自动处理的，你不需要去考虑它们。如果实现好奇的话，可以通过查看 QMetaObject 的文档和 moc 的源代码来一睹芳容。

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/194031`](http://devbean.blog.51cto.com/448512/194031)