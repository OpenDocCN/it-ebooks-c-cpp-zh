# 显式转换操作符

# 显式转换操作符

C++98 标准提供隐式和显式两种构造函数，也就是说，声明为显式形式的构造函数所定义的转换只能用于显式转换，而其他形式的构造函数则用于隐式转换。例如：

```cpp
struct S { S(int); };    // “普通构造函数”默认是隐式转换
S s1(1);        // ok, 直接构造
S s2 = 1;    // ok, 隐式拷贝构造
void f(S);
// 能通过编译（但是经常会产生意外结果——如果 S 是 vector 类型会怎么样呢？）
// 译注：详见下一用例的解释
f(1);

struct E { explicit E(int); };    // 显式构造函数
E e1(1);        // ok
E e2 = 1;    // 错误（但是常常会让人感到意外——这怎么会错呢？）
void f(E);
// 该处会产生编译错误（而非编译通过），以避免因隐式类型转换而得到莫名其妙的结果。
// 例如 std::vector::vector(int size), 该构造函数在标准库中定义为显式类型转换，
// （译注：以避免程序员为了初始化一个只含有一个元素 10 的数组而写出如下代码:
//    vector<int> vec = 10; 
//   而实际上该代码的含义却是定义一个初始包含 10 个元素的数组）
f(1); 
```

然而，禁止从构造函数作隐式转换（以避免问题），并没有堵住全部漏洞。如果某个类本身禁止改动，那么可以从另一个不同的类中定义一个转换操作符。例如：

```cpp
struct S { S(int) { } /* … */ };
struct SS {>
    int m;
    SS(int x) :m(x) { }
    // 在 struct S 无须定义 S(SS)——所谓的“非侵入”式做法
    operator S() { return S(m); }
};
SS ss(1);    // ok; 默认构造函数
S s1 = ss;   // ok; 隐式转换为 S 后调用拷贝构造函数
S s2(ss);    // ok; 隐式转换为 S 后调用直接构造函数
void f(S);
f(ss);        // ok; 隐式转换为 S 后传参 
```

（译注：这段代码的意义，实际是通过 SS 作为中间桥梁，将 int 转换为 S。）

遗憾的是，C++98 中无法定义”显式转换操作符”来完全禁止某个类相关的隐式转换（因为除此之外鲜有用武之地）。C++11 则高瞻远瞩，添加了这个特性。例如：

```cpp
struct S { S(int) { } };
struct SS {
    int m;
    SS(int x) :m(x) { }
    // 因为结构体 S 中没有定义构造函数 S(SS)
    // 无法将 SS 转换为 S，所以只好在 SS 中定义一个返回 S 的转换操作符，
    // 将自己转换为 S。
    // 转换动作，可以由目标类型 S 提供，也可以由源类型 SS 提供。）
     explicit operator S() { return S(m); }
};
SS ss(1);    // ok; 默认构造函数
S s1 = ss;    // 错误; 拷贝构造函数不能使用显式转换
S s2(ss);    // ok; 直接构造函数可以使用显式转换
void f(S);
f(ss);        // 错误; 从 SS 向 S 的转换必须是显式的.
// 译注: 强制类型转换也可使用显式转换，例如
// S s3 = static_cast<S>(ss); 
```

参考:

*   Standard: 12.3 Conversions
*   [N2333=07-0193] Lois Goldthwaite, Michael Wong, and Jens Maurer:

    [Explicit Conversion Operator (Revision 1)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2333.html).