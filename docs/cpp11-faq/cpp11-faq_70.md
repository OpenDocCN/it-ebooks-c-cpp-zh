# 可变参数模板（Variadic Templates）

要解决的问题：

*   怎么创建一个拥有 1 个、2 个或者更多的初始化器的类？
*   怎么避免创建一个实例而只拷贝部分的结果？
*   怎么创建一个元组？

最后的问题是关键所在：考虑一下元组！如果你能创建并且访问一般的元组，那么剩下的问题也将迎刃而解。

这里有一个例子（摘自“可变参数模板简述（A brief introduction to Variadic templates）”（参见参考）），要构建一个广义的、类型安全的 printf()。这个方法比用 boost::format 好的多，但是考虑一下：

```cpp
const string pi = “pi”;
const char* m =
    “The value of %s is about %g (unless you live in %s).n”;
printf(m,  pi, 3.14159,  “Indiana”); 
```

这是除了格式字符串之外，没有其它参数的情况下调用 printf()的一个最简单的例子了，所以我们将要首先解决：

```cpp
void printf(const char* s)    
{
 while (s && *s) {
      if (*s==’%’ && *++s!=’%')  //保证没有更多的参数了
//%%（转义字符，在格式字符串中代表%
          throw runtime_error(“格式非法: 缺少参数”);
     std::cout << *s++<<endl;
 }
} 
```

这个处理好之后，我们必须处理有更多参数的 printf()：

```cpp
template<typename T, typename... Args>    // 注意这里的"..."
void printf(const char* s, T value, Args... args)   // 注意"..."
{
    while (s && *s) {
        //一个格式标记（避免格式控制符）
                    if (*s=='%' && *++s!='%') {
            std::cout << value;        
            return printf(++s, args...);//使用第一个非格式参数
        }
        std::cout << *s++;
    }
    throw std::runtime error("extra args provided to printf");
} 
```

这段代码简单地“去除”了开头的无格式参数，之后递归地调用自己。当没有更多的无格式参数的时候，它调用第一个（很简单）printf()（如上所示）。这也是标准的函数式编程在编译的时候做的(?)。注意,`<<`的重载代替了在格式控制符当中（可能会有错误）的花哨的技巧。（译注：我想这里可能指的是使用重载的`<<`输出操作符，就可以避免使用各种技巧复杂的格式控制字符串。） Args…定义的是一个叫做“参数包”的东西。这个“参数包”仅仅是一个（有各种类型的值的）队列，而且这个队列中的参数可以从头开始进行剥离（处理）。如果我们使用一个参数调用 printf()，函数的第一个定义(printf(const char*))就被调用。如果我们使用两个或者更多的参数调用 printf()，那么函数的第二个定义(printf(const char*, T value, Args… args))就会被调用，把第一个参数当作字符串，第二个参数当作值，而剩余的参数都打包到参数包 args 中，用做函数内部的使用。在下面的调用中：

```cpp
printf(++s, args…); 
```

参数包 args 被打开，所以参数包中的下一个参数被选择作为值。这个过程会持续进行，直到 args 为空（所以第一个 printf()最终会被调用）。

如果你对函数式编程很熟悉的话，你可能会发现这个语法和标准技术有一点不一样。如果发现了，这里有一些小的技术示例可能会帮助你理解。首先我们可以声明一个普通的可变参数函数模板（就像上面的 printf()）：

```cpp
template<class ... Types>
// 可变参数模板函数
//（补充：一个函数可以接受若干个类型的若干个参数）
void f(Types ... args);

f();        // OK: args 不包含任何参数
f(1);       // OK: args 有一个参数: int
f(2, 1.0);  // OK: args 有两个参数: int 和 double 
```

我们可以建立一个具有可变参数的元组类型：

```cpp
template<typename Head, typename... Tail>
//这里是一个递归
//一个元组最基本要存储它的 head（第一个（类型/值））对
//并且派生自它的 tail（剩余的（类型/值））对
//注意，这里的类型被编码，而不是按一个数据来存储
class tuple<Head, Tail...>
    : private tuple<Tail...> {     
    typedef tuple<Tail...> inherited;
public:
    tuple() { } // 默认的空 tuple

    //从分离的参数中创建元组
    tuple(typename add_const_reference<Head>::type v,
 typename add_const_reference<Tail>::type... vtail)
        : m_head(v), inherited(vtail...) { }

    // 从另外一个 tuple 创建 tuple:
    template<typename... VValues>
    tuple(const tuple<VValues...>& other)
    :    m_head(other.head()), inherited(other.tail()) { }

    template<typename... VValues>
    tuple& operator=(const tuple<VValues...>& other)  // 等于操作
    {
        m_head = other.head();
        tail() = other.tail();
        return *this;
    }

    typename add_reference<Head>::type head()
            { return m_head; }
    typename add_reference<const Head>::type head() const
            { return m_head; }

    inherited& tail() { return *this; }
    const inherited& tail() const { return *this; }
protected:
    Head m_head;
} 
```

有了定义之后，我们可以创建元组（并且复制和操作它们）：

```cpp
tuple<string,vector,double> tt("hello",{1,2,3,4},1.2);
string h = tt.head();   // "hello"
tuple<vector<int>,double> t2 = tt.tail(); 
```

要实现所有的数据类型可能会比较乏味，所以我们经常减少参数的类型，例如，可以使用标准库中的 make_tuple()函数：

```cpp
template<class... Types>
// 这个定义十分简单（参见标准 20.5.2.2）
tuple<Types...> make_tuple(Types&&... t)   
{
    return tuple<Types...>(t...);
}

string s = "Hello";
vector<int> v = {1,22,3,4,5};
auto x = make_tuple(s,v,1.2); 
```

参考：

*   Standard 14.6.3 Variadic templates
*   [N2151==07-0011] D. Gregor, J. Jarvi:

    [Variadic Templates for the C++0x Standard Library](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2151.pdf).

*   [N2080==06-0150] D. Gregor, J. Jarvi, G. Powell:

    [Variadic Templates (Revision 3)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2080.pdf).

*   [N2087==06-0157] Douglas Gregor:

    [A Brief Introduction to Variadic Templates](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2087.pdf).

*   [N2772==08-0282] L. Joly, R. Klarer:

    [Variadic functions: Variadic templates or initializer lists? — Revision 1](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2772.pdf).

*   [N2551==08-0061] Sylvain Pion:

    [A variadic std::min(T, …) for the C++ Standard Library (Revision 2)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2551.pdf) .

*   Anthony Williams:

    [An introduction to Variadic Templates in C++0x](http://www.devx.com/cplus/Article/41533).

    DevX.com, May 2009.