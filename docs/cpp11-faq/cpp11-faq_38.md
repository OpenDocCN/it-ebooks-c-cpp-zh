# 委托构造函数（Delegating constructors）

在 C++98 中，如果你想让两个构造函数完成相似的事情，可以写两个大段代码相同的构造函数，或者是另外定义一个 init()函数，让两个构造函数都调用这个 init()函数。例如：

```cpp
class X {
    int a;
    // 实现一个初始化函数
    validate(int x) {
        if (0<x && x<=max) a=x; else throw bad_X(x);
    }
public:
    // 三个构造函数都调用 validate()，完成初始化工作
    X(int x) { validate(x); }
    X() { validate(42); }
    X(string s) {
        int x = lexical_cast<int>(s); validate(x);
    }
    // …
}; 
```

这样的实现方式重复罗嗦，并且容易出错。并且，这两种方式的可维护性都很差。所以，在 C++0x 中，我们可以在定义一个构造函数时调用另外一个构造函数：

```cpp
class X {
    int a;
public:
    X(int x) { if (0<x && x<=max) a=x; else throw bad_X(x); }
    // 构造函数 X()调用构造函数 X(int x)
    X() :X{42} { }
    // 构造函数 X(string s)调用构造函数 X(int x)
    X(string s) :X{lexical_cast<int>(s)} { }
    // …
}; 
```

（译注：在一个构造函数中调用另外一个构造函数，这就是委托的意味，不同的构造函数自己负责处理自己的不同情况，把最基本的构造工作委托给某个基础构造函数完成，实现分工协作。）

参考：

*   the C++ draft section 12.6.2
*   N1986==06-0056 Herb Sutter and Francis Glassborow: Delegating Constructors (revision 3).