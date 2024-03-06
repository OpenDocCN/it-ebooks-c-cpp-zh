# 八、深入了解信号槽

信号槽机制是 Qt 编程的基础。通过信号槽，能够使 Qt 各组件在不知道对方的情形下能够相互通讯。这就将类之间的关系做了最大程度的解耦。

槽函数和普通的 C++ 成员函数没有很大的区别。它们也可以使 virtual 的；可以被重写；可以使 public、protected 或者 private 的；可以由其它的 C++ 函数调用；参数可以是任何类型的。如果要说区别，那就是，槽函数可以和一个信号相连接，当这个信号发生时，它可以被自动调用。

connect()语句的原型类似于：

```cpp
connect(sender, SIGNAL(signal), receiver, SLOT(slot));
```

这里，sender 和 receiver 都是 QObject 类型的，singal 和 slot 都是没有参数名称的函数签名。SINGAL()和 SLOT()宏用于把参数转换成字符串。

深入的说，信号槽还有更多可能的用法，如下所示。

**一个信号可以和多个槽相连：**

```cpp
connect(slider, SIGNAL(valueChanged(int)),
              spinBox, SLOT(setValue(int))); 
connect(slider, SIGNAL(valueChanged(int)),
              this, SLOT(updateStatusBarIndicator(int)));
```

注意，如果是这种情况，这些槽会一个接一个的被调用，但是它们的调用顺序是不确定的。

**多个信号可以连接到一个槽：**

```cpp
connect(lcd, SIGNAL(overflow()),
              this, SLOT(handleMathError())); 
connect(calculator, SIGNAL(divisionByZero()),
              this, SLOT(handleMathError()));
```

这是说，只要任意一个信号发出，这个槽就会被调用。

**一个信号可以连接到另外的一个信号：**

```cpp
connect(lineEdit, SIGNAL(textChanged(const QString &)),
              this, SIGNAL(updateRecord(const QString &)));
```

这是说，当第一个信号发出时，第二个信号被发出。除此之外，这种信号-信号的形式和信号-槽的形式没有什么区别。

**槽可以被取消链接：**

```cpp
disconnect(lcd, SIGNAL(overflow()),
                 this, SLOT(handleMathError()));
```

这种情况并不经常出现，因为当一个对象 delete 之后，Qt 自动取消所有连接到这个对象上面的槽。

为了正确的连接信号槽，信号和槽的参数个数、类型以及出现的顺序都必须相同，例如：

```cpp
connect(ftp, SIGNAL(rawCommandReply(int, const QString &)),
              this, SLOT(processReply(int, const QString &)));
```

这里有一种例外情况，如果信号的参数多于槽的参数，那么这个参数之后的那些参数都会被忽略掉，例如：

```cpp
connect(ftp, SIGNAL(rawCommandReply(int, const QString &)), 
            this, SLOT(checkErrorCode(int)));
```

这里，const QString &这个参数就会被槽忽略掉。

如果信号槽的参数不相容，或者是信号或槽有一个不存在，或者在信号槽的连接中出现了参数名字，在 Debug 模式下编译的时候，Qt 都会很智能的给出警告。

在这之前，我们仅仅在 widgets 中使用到了信号槽，但是，注意到 connect()函数其实是在 QObject 中实现的，并不局限于 GUI，因此，只要我们继承 QObject 类，就可以使用信号槽机制了：

```cpp
class Employee : public QObject 
{ 
        Q_OBJECT 
public: 
        Employee() { mySalary = 0; }  
        int salary() const { return mySalary; } 

public slots: 
        void setSalary(int newSalary); 

signals: 
        void salaryChanged(int newSalary); 

private: 
        int mySalary; 
};
```

在使用时，我们给出下面的代码：

```cpp
void Employee::setSalary(int newSalary) 
{ 
        if (newSalary != mySalary) { 
                mySalary = newSalary; 
                emit salaryChanged(mySalary); 
        } 
}
```

这样，当 setSalary()调用的时候，就会发出 salaryChanged()信号。注意这里的 if 判断，这是避免递归的方式！还记得前面提到的循环连接吗？如果没有 if，当出现了循环连接的时候就会产生无限递归。

本文出自 “豆子空间” 博客，请务必保留此出处 [`devbean.blog.51cto.com/448512/194031`](http://devbean.blog.51cto.com/448512/194031)