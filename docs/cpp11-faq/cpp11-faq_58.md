# POD

所谓 POD(Plain Old Data)，指的是那些可以像 C 结构体一样直接操作的“普通”类型，对于该种类型，可以直接对它用 memset()/memcpy()来进行初始化/拷贝等操作。

在 C++98 标准中，POD 实际上是受限于结构体定义时所涉之语言特性而定义的。

```cpp
struct S { int a; };    // S 属于 POD
struct SS { int a; SS(int aa) : a(aa) { } }; // SS 不属于 POD
struct SSS { virtual void f(); /* ... */ }; 
```

在 C++11 中，S 和 SS 都是“标准布局类型”(即 POD)，因为 SS 实在没什么复杂的地方：构造函数不会影响它内存布局（所以 memcpy()也能用），不过这里却不能用 memset()来初始化——因为它可能违反构造函数中定义的赋值规则(需要用 aa 来为 a 赋值）。另外，这里的 SSS 则明显不是 POD 了，因为其每个对象中都包含着虚表指针(vptr)。

C++11 中引进或重新定义了 POD、trivially-copyable 类型、trivial 类型、以及”标准布局”类型等概念，以用来处理 C++98 中原”POD”相关的一系列技术问题。

（译注：请参阅[《怎样理解 C++ 11 中的 trivial 和 standard-layout—An answer from stackoverflow》](http://www.cnblogs.com/tingshuo/archive/2013/03/28/2986236.html)）

POD 的(递归)定义如下：

*   所有的成员类型和基类都是 POD 类型
*   其余部分跟以前一样(参见[10]第九章节)

不含虚函数

不含虚基类

不含引用

不含多种访问权限(译注：对所有 non-static 成员有相同的 public/private/protected 访问控制权限)

C++11 中关于 POD 方面最重要的部分就是 POD 中允许存在不影响内存布局和性能的构造函数（译注：参见 C++11 中新引入的 default 构造函数语法）。

参考文献：

*   the C++ draft section 3.9 and 9 [10]
*   [N2294=07-0154] Beman Dawes:

    [POD’s Revisited; Resolving Core Issue 568 (Revision 4)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2294.html)

    .

（翻译：张潇）