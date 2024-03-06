# 用作模板参数的局部类型

# 用作模板参数的局部类型

在 C++98 中，局部类和未命名类不能作为模板参数，这或许是一个负担，C++11 则放宽了这方面的限制：

```cpp
void f(vector<X>& v)
{
    struct Less {
        bool operator()(const X& a, const X& b)
        { return a.v<b.v; }
    };
    // C++98: 错误: Less 是局部类
        // C++11: 正确
        sort(v.begin(), v.end(), Less());
} 
```

当然除了这里的局部类之外，在 C++11 中，我们还可以采用 Lambda 表达式来做同样的事情：

```cpp
void f(vector<X>& v)
{
    sort(v.begin(), v.end(),
        [] (const X& a, const X& b) { return a.v<b.v; });
} 
```

尽管如此，我们仍然不要忘记，为一系列动作行为命名有利于文档化，是一个值得鼓励的设计风格。而且，非局部的函数体（当然也需要命名）还可以被重用于其他模块。

C++11 同时也允许模板参数使用未命名类型的值：

```cpp
template<typename T> void foo(T const& t){}
enum X { x };
enum { y };

int main()
{
    foo(x);     // C++98: ok; C++11: ok
    //（译注：y 是未命名类型的值，C++98 无法从这样的值中推断出函数模板参数）
    foo(y);     // C++98: error; C++11: ok
    enum Z { z };
    foo(z);     // C++98: error; C++11: ok
    //（译注：C++98 不支持从局部类型值推导模板参数
} 
```

参考：

*   Standard: Not yet: CWG issue 757
*   [N2402=07-0262] Anthony Williams:

    [Names, Linkage, and Templates (rev 2)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2402.pdf).

*   [N2657] John Spicer:

    [Local and Unnamed Types as Template Arguments](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2657.htm).