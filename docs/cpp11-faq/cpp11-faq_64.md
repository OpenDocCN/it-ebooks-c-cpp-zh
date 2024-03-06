# 模板别名（正式的名称为"template typedef"）

# 模板别名（正式的名称为"template typedef"）

如何在某个模板类的基础上，通过写死（绑定）部分模板参数类型，来定制出一个新的模板类？

尝尝我这种写法如何：

```cpp
template<class T>
// 采用了自定义内存分配器的 std::vector
using Vec = std::vector<T,My_alloc<T>>;

// 使用 My_alloc 为元素分配存储空间
Vec<int> fib = { 1, 2, 3, 5, 8, 13 };

// verbose 与 fib 类型一致
vector<int,My_alloc<int>> verbose = fib; 
```

有了 using 语法，定义模板别名则可一目了然：***using 模板别名 = 引用细节;***。在这之前，我们也曾徘徊在”typedef”的经典和复杂之间左右为难，但没有哪一个方案能做到完美平衡，直到后来我们干脆弃而转向言简意赅的 using 语法。

即使模板存在特化，using 语法也能照常使用。（需注意：为模板及其各种特化形式可以设定一个统一的别名，但反之则不然：模板特化操作不能通过别名进行）。例如：

```cpp
template<int>
// idea: int_exact_trait<N>::type 用于表达含有 N 个 bit 的数值类型
struct int_exact_traits {
    typedef int type;
};

template<>
struct int_exact_traits<8> {
    typedef char type;
};

template<>
struct int_exact_traits<16> {
    typedef char[2] type;
};

// ...

template<int N>
// 定义别名用以简化书写
// 译注：给模板的通用版本取别名，则其所有的特化版本自动获得该别名，
//    例如对于 8bit 的特化版本，现在可直接使用别名
    using int_exact = typename int_exact_traits<N>::type;
    // int_exact<8> 是含有 8 个 bit 的数值类型
int_exact<8> a = 7; 
```

除了在模板方面身担重任外，using 语法也可作为对普通类型定义别名的另一种选择（相较于 typedef 更合吾意）。

```cpp
typedef void (*PFD)(double);    // C 样式
using PF = void (*)(double);    // using 加上 C 样式的类型
using P = [](double)->void;  // using 和函数返回类型后置语法 
```

参考：

参考：

*   the C++ draft: 14.6.7 Template aliases; 7.1.3 The typedef specifier
*   [N1489=03-0072] Bjarne Stroustrup and Gabriel Dos Reis: [Templates aliases for C++](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2003/n1489.pdf) .
*   [N2258=07-0118] Gabriel Dos Reis and Bjarne Stroustrup: [Templates Aliases (Revision 3)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2258.pdf) (final proposal).

（翻译：张潇，dabaitu）