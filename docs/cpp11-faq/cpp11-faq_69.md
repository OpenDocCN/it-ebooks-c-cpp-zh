# 用户定义数据标识（User-defined literals）

# 用户定义数据标识（User-defined literals）

C++提供了许多内建数据类型的数据标识（2.14 节变量）：

```cpp
123    // int 整型
1.2    // double 双精度型
1.2F    // float 浮点型
’a'    // char 字符型
1ULL    // unsigned long long64 位无符号长整型
0xD0    // hexadecimal unsigned 十六进制无符号整型
"as"    // string 字符串 
```

但是在 C++98 中并没有为用户自定义的变量类型提供数据标识。这就违反甚至冲突于“用户自定类型应该和内建类型一样得到支持”的原则。在特殊情况下，人们有以下的需求：

```cpp
"Hi!"s            //字符串，不是“以零字符为终结的字符数组”
1.2i            //虚数
123.4567891234df    //十进制浮点型（IBM）
101010111000101b    //二进制
123s            //秒
123.56km          //不是英里（单位）
1234567890123456789012345678901234567890x        //扩展精度 
```

C++11 通过在变量后面加上一个后缀来标定所需的类型以支持“用户定义数据标识”，例如：

```cpp
constexpr complex operator "" i(long double d)    // 设计中的数据标识
{
    return {0,d};    //complex 是一个数据标识
}

// 将 n 个字符构造成字符串 std::string 对象的数据标识
std::string operator""s (const char* p, size_t n)    
{
    return string(p,n);    // 需要释放存储空间
} 
```

这里需要注意的是，constexpr 的使用可以进行编译时期的计算。使用这一功能，我们可以这样写：

```cpp
template <class T> void f(const T&);
f("Hello");    // 传递 char*指针给 f()
f("Hello"s);    // 传递（5 个字符的）字符串对象给 f()
f("Hello n"s);    // 传递（6 个字符的）字符串对象给 f()

auto z = 2+1i;    // 复数 complex(2,1) 
```

基本（实现）方法是编译器在解析什么语句代表一个变量之后，再分析一下后缀。用户自定义数据标识机制只是简简单单的允许用户制定一个新的后缀，并决定如何对它之前的数据进行处理。要想重新定义一个内建的数据标识的意义或者它的参数、语法是不可能的。一个数据标识操作符可以使用它（前面）的数据标识传递过来的处理过的值（如果是使用新的没有定义过的后缀的值）或者没有处理过的值（作为一个字符串）。

要得到一个没有处理过的字符串，只要使用一个单独的 const char*参数即可，例如：

```cpp
Bignum operator"" x(const char* p)
{
    return Bignum(p);
}

void f(Bignum);
f(1234567890123456789012345678901234567890x); 
```

这个 C 语言风格的字符串”1234567890123456789012345678901234567890″被传递给了操作符 operator”” x()。注意，我们并没有明确地把数字转换成字符串。

有以下四种数据标识的情况，可以被用户定义后缀来使用用户自定义数据标识：

*   整型标识：允许传入一个 unsigned long long 或者 const char*参数
*   浮点型标识：允许传入一个 long double 或者 const char*参数
*   字符串标识：允许传入一组(const char*,size_t)参数
*   字符标识：允许传入一个 char 参数。

注意，你为字符串标识定义的标识操作符不能只带有一个 const char*参数（而没有大小）。例如：

```cpp
//警告，这个标识操作符并不能像预想的那样子工作
string operator"" S(const char* p);  
"one two"S;    //错误，没有适用的标识操作符 
```

根本原因是如果我们想有一个“不同的字符串”，我们同时也想知道字符的个数。后缀可能比较短（例如，s 是字符串的后缀，i 是虚数的后缀，m 是米的后缀，x 是扩展类型的后缀），所以不同的用法很容易产生冲突，我们可以使用 namespace（命名空间）来避免这些名字冲突：

```cpp
namespace Numerics {
    // …
    class Bignum { /* … */ };
    namespace literals {
        operator"" X(char const*);
    }
}

using namespace Numerics::literals; 
```

参考：

*   Standard 2.14.8 User-defined literals
*   [N2378==07-0238] Ian McIntosh, Michael Wong, Raymond Mak, Robert Klarer, Jens Mauer, Alisdair Meredith, Bjarne Stroustrup, David Vandevoorde:

    [User-defined Literals (aka. Extensible Literals (revision 3))](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2378.pdf).