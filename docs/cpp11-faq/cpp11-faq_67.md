# 统一初始化的语法和语义

# 统一初始化的语法和语义

按照对象的类型以及初始化时的上下文，C++提供了五花八门的对象初始化的方式。若不慎误用，可能会产生匪夷所思的谬误，而且还伴随着莫名其妙的错误（调试）信息。考虑如下的代码：

```cpp
string a[] = { “foo”, ” bar” };  //正确：初始化数组变量
//错误：初始化列表应用在了非聚合的向量上
vector<string> v = { “foo”, ” bar” };
void f(string a[]);
f( { “foo”, ” bar” } );   //语法错误，把一个块（block）作为了参数 
```

以及：

```cpp
int a = 2;         //“赋值风格”的初始化
int aa[] = { 2, 3 }; //用初始化列表进行的赋值风格的初始化
complex z(1,2);   //“函数风格”的初始化
x = Ptr(y);     // “函数风格”的转换/赋值/构造操作 
```

再如：

```cpp
int a(1);  //变量的定义
int b();    //函数的声明
int b(foo);    // 变量的定义，或者函数的声明 
```

要记住这么多种初始化规则，并从中选用最合适的一种，绝非易事。

C++11 的解决方法是对于所有的初始化，均可使用“{}-初始化变量列表”：

```cpp
X x1 = X{1,2};
X x2 = {1,2};     // 此处的'='可有可无
X x3{1,2};
X* p = new X{1,2};

struct D : X {
    D(int x, int y) :X{x,y} { /* … */ };
};

struct S {
    int a[3];
    // 对于旧有问题的解决方案
    S(int x, int y, int z) :a{x,y,z} { /* … */ };
}; 
```

与以往相比最为关键的变动是，X{a}方式的初始化，在所有的语境中都能构造出同样的结果，所以凡是能用“{}”的初始化，得到的结果都是一致的。例如：

```cpp
X x{a};
X* p = new X{a};
z = X{a};         // 使用了类型转换
f({a});           // a 作为函数的 X 型实参
return {a};       // a 作为函数的 X 型返回值 
```

其他参考文档：

*   the C++ draft section ???
*   [N2215==07-0075 ] Bjarne Stroustrup and Gabriel Dos Reis: [Initializer lists (Rev. 3)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2215.pdf) .
*   [N2640==08-0150] Jason Merrill and Daveed Vandevoorde: [Initializer Lists — Alternative Mechanism and Rationale (v. 2)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2640.pdf) (final proposal).

（翻译：张潇）