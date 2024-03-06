# 继承的构造函数

人们有时会对类成员函数或成员变量的作用域问题感到困惑，尤其是，当基类与派生类的同名成员不在同一个作用域内时：

```cpp
struct B {
    void f(double);
};
struct D : B {
    void f(int);
};
B b;   b.f(4.5);    // OK
// 调用的到底是 B::f(doube)还是 D::f(int)呢？
// 实际情况往往会让人感到意外：调用的 f(int)函数实参为 4
D d;   d.f(4.5); 
```

在 C++98 标准里，可以将普通的重载函数从基类“晋级”到派生类里来解决这个问题：

```cpp
struct B {
    void f(double);
};

struct D : B {
    using B::f;     // 将类 B 中的 f()函数引入到类 D 的作用域内
    void f(int);    // 增加一个新的 f()函数
};

B b;   b.f(4.5);    // OK
// 可行：调用类 D 中的 f(double)函数
// 也即类 B 中的 f(double)函数
D d;   d.f(4.5); 
```

普通重载函数可以通过这种方式解决，那么，对于构造函数又该怎么办呢？ 我曾经说过“不能像应用于普通成员函数那样，将上述语法应用于构造函数，这如历史偶然一样”。为了解决构造函数的“晋级”问题，C++11 提供了这种能力：

```cpp
class Derived : public Base {
public:
    // 提升 Base 类的 f 函数到 Derived 类的作用范围内
    // 这一特性已存在于 C++98 标准内
    using Base::f;
    void f(char);     // 提供一个新的 f 函数
    void f(int);  // 与 Base 类的 f(int)函数相比更常用到这个 f 函数
    // 提升 Base 类的构造函数到 Derived 的作用范围内
    // 这一特性只存在于 C++11 标准内
    using Base::Base;
    Derived(char);    // 提供一个新的构造函数
    // 与 Base 类的构造函数 Base(int)相比
    // 更常用到这个构造函数
    Derived(int);
    // …
}; 
```

如果这样用了，仍然可能困惑于派生类中继承的构造函数，这个派生类中定义的新成员变量需要初始化（译注：基类并不知道派生类的新增成员变量，当然不会对其进行初始化。）：

```cpp
struct B1 {
    B1(int) { }
};
struct D1 : B1 {
    using B1::B1; // 隐式声明构造函数 D1(int)
    int x;
};
void test()
{
    D1 d(6);  // 糟糕：调用的是基类的构造函数，d.x 没有初始化
    D1 e;    // 错误：类 D1 没有默认的构造函数
} 
```

我们可以通过使用成员初始化（member-initializer）消除以上的困惑：

```cpp
struct D1 : B1 {
    using B1::B1;    // 隐式声明构造函数 D1(int)
    // 注意：x 变量已经被初始化
    // （译注：在声明的时候就提供初始化）
    int x{0};
};
void test()
{
    D1 d(6);    // d.x 的值是 0
} 
```

参考：

*   the C++ draft 8.5.4 List-initialization [dcl.init.list]
*   [N1890=05-0150 ] Bjarne Stroustrup and Gabriel Dos Reis:

    [Initialization and initializers](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2005/n1890.pdf)

    (an overview of initialization-related problems with suggested solutions).

*   [N1919=05-0179] Bjarne Stroustrup and Gabriel Dos Reis:

    [Initializer lists](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2005/n1919.pdf).

*   [N2215=07-0075] Bjarne Stroustrup and Gabriel Dos Reis :

    [Initializer lists (Rev. 3)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2215.pdf) .

*   [N2640=08-0150] Jason Merrill and Daveed Vandevoorde:

    [Initializer Lists — Alternative Mechanism and Rationale (v. 2)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2640.pdf) (final proposal).