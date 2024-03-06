# 返回值类型后置语法

# 返回值类型后置语法

考虑下面这段代码：

```cpp
template<class T, class U>
??? mul(T x, U y)
{
    return x*y;
} 
```

函数 mul()的返回类型要怎么写呢？当然，是“x*y 类型”，但是这并不是一个数据类型，我们如何才能一开始就得到它的真实数据类型呢？在初步了解 C++0x 之后，你可能一开始想到使用 decltype 来推断“x*y”的数据类型：

```cpp
template<class T, class U>
decltype(x*y) mul(T x, U y) // 注意这里的作用域
{
    return x*y;
} 
```

但是，这种方式是行不通的，因为 x 和 y 不在作用域内。但是，我们可以这样写：

```cpp
template<class T, class U>
// 难看别扭，且容易产生错误
decltype(*(T*)(0)**(U*)(0)) mul(T x, U y)    
{
    return x*y;
} 
```

如果称这种用法为“还可以”，就已经是过誉了。

C++11 的解决办法是将返回类型放在它所属的函数名的后面：

```cpp
template<class T, class U>
auto mul(T x, U y) -> decltype(x*y)
{
    return x*y;
} 
```

这里我们使用了 auto 关键字，（auto 在 C++11 中还有根据初始值推导数据类型的意义），在这里它的意思变为“返回类型将会稍后引出或指定”。

返回值后置语法最初并不是用于模板和返回值类型推导的，它实际是用于解决作用域问题的。

```cpp
struct List {
    struct Link { /* ... */ };
    Link* erase(Link* p);   // 移除 p 并返回 p 之前的链接
    // ...
};

List::Link* List::erase(Link* p) { /* ... */ } 
```

第一个 List::是必需的，这仅是因为 List 的作用域直到第二个 List::才有效。更好的表示方式是：

```cpp
auto List::erase(Link* p) -> Link* { /* ... */ } 
```

现在，将函数返回类型后置，Link*就不需要使用明确的 List::进行限定了。

参考：

*   the C++ draft section ???
*   [Str02] Bjarne Stroustrup. Draft proposal for “typeof”. C++ reflector message c++std-ext-5364, October 2002.
*   [N1478=03-0061] Jaakko Jarvi, Bjarne Stroustrup, Douglas Gregor, and Jeremy Siek:

    [Decltype and auto](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2003/n1478.pdf).

*   [N2445=07-0315] Jason Merrill:

    [New Function Declarator Syntax Wording](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2445.html).

*   [N2825=09-0015] Lawrence Crowl and Alisdair Meredith:

    [Unified Function Syntax](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2009/n2825.html).