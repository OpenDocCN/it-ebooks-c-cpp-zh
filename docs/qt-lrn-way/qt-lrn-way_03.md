# 三、初探信号槽

看过了简单的 Hello, world! 之后，下面来看看 Qt 最引以为豪的信号槽机制！

所谓信号槽，简单来说，就像是插销一样：一个插头和一个插座。怎么说呢？当某种事件发生之后，比如，点击了一下鼠标，或者按了某个按键，这时，这个组件就会发出一个信号。就像是广播一样，如果有了事件，它就漫天发声。这时，如果有一个槽，正好对应上这个信号，那么，这个槽的函数就会执行，也就是回调。就像广播发出了，如果你感兴趣，那么你就会对这个广播有反应。干巴巴的解释很无力，还是看代码：

```cpp
 #include <QtGui/QApplication> 

 #include <QtGui/QPushButton> 

int main(int argc, char *argv[]) 
{ 
        QApplication a(argc, argv); 
        QPushButton *button = new QPushButton("Quit"); 
        QObject::connect(button, SIGNAL(clicked()), &a, SLOT(quit())); 
        button->show(); 
        return a.exec(); 
}
```

这是在 Qt Creator 上面新建的文件，因为前面已经详细的说明怎么新建工程，所以这里就不再赘述了。这个程序很简单，只有一个按钮，点击之后程序退出。(顺便说一句，Qt 里面的 button 被叫做 QPushButton，真搞不明白为什么一个简单的 button 非得加上 push 呢？呵呵)

主要是看这一句：

```cpp
QObject::connect(button, SIGNAL(clicked()), &a, SLOT(quit()));
```

QObject 是所有类的根。Qt 使用这个 QObject 实现了一个单根继承的 C++。它里面有一个 connect 静态函数，用于连接信号槽。

当一个按钮被点击时，它会发出一个 clicked 信号，意思是，向周围的组件们声明：我被点击啦！当然，其它很多组件都懒得理他。如果对它感兴趣，就告诉 QObject 说，你帮我盯着点，只要 button 发出 clicked 信号，你就告诉我——想了想之后，说，算了，你也别告诉我了，直接去执行我的某某某函数吧！就这样，一个信号槽就形成了。具体来说呢，这个例子就是 QApplication 的实例 a 说，如果 button 发出了 clicked 信号，你就去执行我的 quit 函数。所以，当我们点击 button 的时候，a 的 quit 函数被调用，程序退出了。所以，在这里，clicked()就是一个信号，而 quit()就是槽，形象地说就是把这个信号插进这个槽里面去。

Qt 使用信号槽机制完成了事件监听操作。这类似与 Swing 里面的 listener 机制，只是要比这个 listener 简单得多。以后我们会看到，这种信号槽的定义也异常的简单。值得注意的是，这个信号槽机制仅仅是使用的 QObject 的 connect 函数，其他并没有什么耦合——也就是说，完全可以利用这种机制实现你自己的信号监听！不过，这就需要使用 qmake 预处理一下了！

细心的你或许发现，在 Qt Creator 里面，SIGNAL 和 SLOT 竟然变颜色了！没错，Qt 确实把它们当成了关键字！实际上，Qt 正是利用它们扩展了 C++ 语言，因此才需要使用 qmake 进行预处理，比便使普通的 C++ 编译器能够顺利编译。另外，这里的 signal 和 Unix 系统里面的 signal 没有任何的关系！哦哦，有一点关系，那就是名字是一样的！

信号槽机制是 Qt 关键部分之一，以后我们还会再仔细的探讨这个问题的。

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/194031`](http://devbean.blog.51cto.com/448512/194031)