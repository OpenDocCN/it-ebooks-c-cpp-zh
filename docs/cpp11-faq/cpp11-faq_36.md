# 控制默认函数——默认或者禁用

# 控制默认函数——默认或者禁用

的复制构造函数和赋值操作符的情况下，便编译器会为我们生成默认的复制构造函数和赋值操作符，以内存复制的形式完成对象的复制。虽然这种机制可以为我们节省很多编写复制构造函数和赋值操作符的时间，但是在某些情况下，比如我们不希望对象被复制，这种机制却是多此一举。）

关于类的“禁止复制”，现在可以使用 delete 关键字完美地直接表达：

```cpp
class X {
    // …
    X& operator=(const X&) = delete; // 禁用类的赋值操作符
    X(const X&) = delete;
}; 
```

相反地，我们也可以使用 default 关键字，显式地指明我们希望使用默认的复制操作：

```cpp
class Y {
    // …
    // 使用默认的赋值操作符和复制构造函数
    Y& operator=(const Y&) = default; // 默认的复制操作
    Y(const Y&) = default;
}; 
```

显式地使用 default 关键字声明使用类的默认行为，对于编译器来说明显是多余的，但是对于代码的阅读者来说，使用 default 显式地定义复制操作，则意味着这个复制操作就是一个普通的默认的复制操作。 将默认的操作留给编译器去实现将更加简单，更少的错误发生 ，并且通常会产生更好的目标代码。

“default”关键字可以用在任何的默认函数中，而“delete”则可以用于修饰任何函数。例如，我们可以通过下面的方式排除一个不想要的函数参数类型转换：

```cpp
struct Z {
    // …
    Z(long long); // 可以通过 long long 初始化
    Z(long) = delete; // 但是不能将 long long 转换为 long 进行初始化(?)
}; 
```

参考：

*   the C++ draft section ???
*   [N1717==04-0157] Francis Glassborow and Lois Goldthwaite: [explicit class and default definitions](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2004/n1717.pdf) (an early proposal).
*   Bjarne Stroustrup: [Control of class defaults](http://wiki.dinkumware.com/twiki/pub/Wg21portland/EvolutionWorkingGroup/controlofdefaults.pdf) (a dead end).
*   [N2326 = 07-0186] Lawrence Crowl: [Defaulted and Deleted Functions](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2326.html) .