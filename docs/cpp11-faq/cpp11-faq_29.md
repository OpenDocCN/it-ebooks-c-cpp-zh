# auto – 从初始化中推断数据类型

# auto – 从初始化中推断数据类型

考虑下面的代码：

```cpp
auto x = 7; 
```

这里的变量 x 被整数 7 初始化，所以 x 的实际数据类型是 int。auto 的通用形式如下：

```cpp
auto x = expression; 
```

这样，这个表达式计算结果的数据类型就是变量 x 的数据类型。

当数据类型未知或不便书写时，使用 auto 可让编译器自动根据用以初始化变量的表达式类型来推导变量类型。考虑如下代码：

```cpp
template<class T> void printall(const vector<T>& v)
{
      // 根据 v.begin()的返回值类型自动推断 p 的数据类型
      for (auto p = v.begin(); p!=v.end(); ++p)
          cout << *p << “n”;
} 
```

为了表示同样的意义，在 C++98 中，我们不得不这样写：

```cpp
template<class T> void printall(const vector<T>& v)
{
    for (typename vector<T>::const_iterator p = v.begin();
       p!=v.end(); ++p)
        cout << *p << “n”;
} 
```

当变量的数据类型依赖于模板参数时，如果不使用 auto 关键字，将很难确定变量的数据类型。例如：

```cpp
template<class T,classs U> void multiply (const vector<T>& vt,
const vector<U>& vu)
{
    // …
    auto tmp = vt[i]*vu[i];
    // …
} 
```

在这里，tmp 的数据类型应该与模板参数 T 和 U 相乘之后的结果的数据类型相同。对于程序员来说，要通过模板参数确定 tmp 的数据类型是一件很困难的事情。但是，对于编译器来说，一旦确定了 T 和 U 的数据类型，推断 tmp 的数据类型将是轻而易举的一件事情。

auto 特性是 C++11 中最早被提出并被实现的特性。早在 1984 年，我就在我的 Cfont 中实现了 auto 特性，但是由于一些兼容性问题，它没有被纳入以前的标准。当 C++98 和 C99 同意删除“implicit int”之后，这些兼容性问题已经不复存在了，也就是 C++语言对“每个变量和函数都要有确切的数据类型”的要求消失了。auto 关键字原来的含义（表示 local 变量）是多余而无用的——标准委员会的成员们在数百万行代码中仅仅只找到几百个用到 auto 关键字的地方，并且大多数出现在测试代码中，有的甚至就是一个 bug。

auto 主要用于简化代码，因此并不会影响标准库规范。

参考：

*   the C++ draft section 7.1.6.2, 7.1.6.4, 8.3.5 (for return types)
*   [N1984=06-0054] Jaakko Jarvi, Bjarne Stroustrup, and Gabriel Dos Reis:[Deducing the type of variable from its initializer expression (revision 4)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n1984.pdf).