# 六、创建一个对话框(上)

首先说明一点，在 C++ GUI Programming with Qt4, 2nd 中，这一章连同以后的若干章一起，完成了一个比较完整的程序——一个模仿 Excel 的电子表格。不过这个程序挺大的，而且书中也没有给出完整的源代码，只是分段分段的——我不喜欢这个样子，我想要看到我写出来的是什么东西，这是最主要的，而不是慢慢的过上几章的内容才能看到自己的作品。所以，我打算换一种方式，每章只给出简单的知识，但是每章都能够运行出东西来。好了，扯完了，下面开始！

以前说的主要是一些基础知识，现在我们来真正做一个东西——一个查找对话框。什么？什么叫查找对话框？唉唉，先看看我们的最终作品吧！

![](img/13.png)

好了，首先新建一个工程，就叫 FindDialog 吧！嗯，当然还是 Qt Gui Application，然后最后一步注意，Base Dialog 选择 QDialog，而不是默认的 QMainWindow，因为我们要学习建立对话框嘛！名字随便起，不过我就叫 finddialog 啦！Ganarate form 还是不要的。然后 Finish 就好了。

打开 finddialog.h，开始编写头文件。

```cpp
 #ifndef FINDDIALOG_H 

 #define FINDDIALOG_H 

 #include <QtGui/QDialog> 

class QCheckBox; 
class QLabel; 
class QLineEdit; 
class QPushButton; 

class FindDialog : public QDialog 
{ 
        Q_OBJECT 

public: 
        FindDialog(QWidget *parent = 0); 
        ~FindDialog(); 
signals: 
        void findNext(const QString &str, Qt::CaseSensitivity cs); 
        void findPrevious(const QString &str, Qt::CaseSensitivity cs); 
private slots: 
        void findClicked(); 
        void enableFindButton(const QString &text); 
private: 
        QLabel *label; 
        QLineEdit *lineEdit; 
        QCheckBox *caseCheckBox; 
        QCheckBox *backwardCheckBox; 
        QPushButton *findButton; 
        QPushButton *closeButton; 
}; 

 #endif // FINDDIALOG_H
```

大家都是懂得 C++ 的啊，所以什么 #ifndef，#define 和 #endif 的含义和用途就不再赘述了。

首先，声明四个用到的类。这里做的是前向声明，否则的话是编译不过的，因为编译器不知道这些类是否存在。简单来说，所谓前向声明就是告诉编译器，我要用这几个类，而且这几个类存在，你就不要担心它们存不存在的问题啦！

然后是我们的 FindDialog，继承自 QDialog。

下面是一个重要的东西：Q_OBJECT。这是一个宏。凡是定义信号槽的类都必须声明这个宏。至于为什么，我们以后再说。

然后是 public 的构造函数和析构函数声明。

然后是一个 signal:，这是 Qt 的关键字——还记得前面说过的嘛？Qt 扩展了 C++ 语言，因此它有自己的关键字——这是对信号的定义，也就是说，FindDialog 有两个 public 的信号，它可以在特定的时刻发出这两个信号，就这里来说，如果用户点击了 Find 按钮，并且选中了 Search backward，就会发出 findPrevious()，否则发出 findNext()。

紧接着是 private slots:的定义，和前面的 signal 一样，这是私有的槽的定义。也就是说，FindDialog 具有两个槽，可以接收某些信号，不过这两个槽都是私有的。

为了 slots 的定义，我们需要访问 FindDialog 的组件，因此，我们把其中的组件定义为成员变量以便访问。正是因为需要定义这些组件，才需要对它们的类型进行前向声明。因为我们仅仅使用的是指针，并不涉及到这些类的函数，因此并不需要 include 它们的头文件——当然，你想直接引入头文件也可以，不过那样的话编译速度就会慢一些。

好了，头文件先说这些，下一篇再说源代码啦！休息，休息一下！

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/194031`](http://devbean.blog.51cto.com/448512/194031)