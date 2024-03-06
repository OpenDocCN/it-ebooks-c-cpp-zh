# [32] 如何混合 C 和 C++编程

# [32] 如何混合 C 和 C++编程

## FAQs in section [32]:

*   [32.1] 混合 C 和 C++编程时我需要知道什么？
*   [32.2] 如何在 C++代码中包含标准的 C 头文件？
*   [32.3] 如何在 C++代码中包含非系统的 C 头文件？
*   [32.4] 如何修改我自己的 C 头文件 ， 以便更容易的在 C++代码中包含他们？
*   [32.5] 如何从 C++代码中调用非系统 C 函数 f（int，char 和 float）？ from my C++ code?")
*   [32.6] 如何创建一个 C++ 函数 f（int，char 和 float），可以由我的 C 代码调用？ that is callable by my C code?")
*   [32.7] 为什么链接器报错说 C / C++函数调用 C++ / C 函数？
*   [32.8] 如何传递一个 C++ 类对象从/到一个 C 函数？
*   [32.9] 我的 C 函数可以直接访问一个 C++ 对象的数据吗？
*   [32.10] 为什么 C++而不是为 C 让我觉得“更加远离机器”？

## 32.1 混合 C 和 C++编程时我需要知道什么？

以下是一些要点（虽然有些编译器供应商可能不需要全部要点，查看你的编译器供应商的文档）：

*   你必须使用你的 C++编译器来编译的 main()（也就是说静态初始化）
*   你的 C++编译器应该能够进行直接链接（也就是说，它有自己的专门类库）
*   你的 C 和 C++编译器可能需要来自同一个供应商，并具有兼容的版本（也就是说，它们有相同的调用约定）

此外，您需要阅读本节的其余部分，了解如何使你的 C 可调用的 C++函数和/或 C++可调用 C 函数。

顺便说一下，还有另一种途径来处理这件事：使用 C++编译器编译所有代码（甚至是你的 C 风格代码）。这几乎无需混合 C 和 C++，但是你要格外小心（也可能是，希望！ -发现了一些错误）你的 C 风格的代码。缺点是你需要更新你的 C 代码风格，主要是因为 C++编译器比 C 编译器更加严谨/挑剔。值得一提的是，清理你的 C 代码风格可能会比实际混合 C 和 C++所付出的努力要少，并且清理 C 代码风格还能给你带来一份额外收入。但是很明显，你几乎没有选择的余地，如果你不能改变 C 代码（例如，如果它是来自第三方的）。

## 32.2 如何在 C++代码中包含标准的 C 头文件？

要包含一个标准（如`<cstdio>`）头文件，你不需要做任何事情。例如：

```cpp
 // This is C++ code

 #include <cstdio>                // Nothing unusual in #include line

 int main()
 {
   std::printf("Hello world\n");  // Nothing unusual in the call either
   ...
 } 
```

如果你认为`std::printf`的`std`部分很奇怪，那么最好的办法是“适应它”。换句话说，它是使用标准库函数的标准方法，所以你不妨现在开始习惯它。

然而，如果你正在使用你的 C++编译器编译 C 代码，你恐怕不想修改所有这些`printf`调用为`std：：printf` 。幸运的是，这种情况下，C 代码将使用旧式头`<stdio.h>`而不是新型头`<cstdio>`，命名空间技术将会照顾一切：

```cpp
 /* This is C code that I'm compiling using a C++ compiler */

 #include <stdio.h>          /* Nothing unusual in #include line */

 int main()
 {
   printf("Hello world\n");  /* Nothing unusual in the call either */
   ...
 } 
```

最后的评论：如果你有不属于标准库 C 头文件，你需要遵守一些不同的准则。有两种情况：要么你不能改变头文件，要么你可以改变头文件。

## 32.3 如何在 C++代码中包含非系统的 C 头文件？

如果你是其中一个 C 头文件不是由系统提供的，你可能需要把`#include`行放到`extern"C"（/ * ... * /）`构造中。这告诉 C++编译器的功能在头文件中声明的 C 函数。

```cpp
 // This is C++ code

 extern "C" {
   // Get declaration for f(int i, char c, float x)
   #include "my-C-code.h"
 }

 int main()
 {
   f(7, 'x', 3.14);   // Note: nothing unusual in the call
   ...
 } 
```

注： 对于系统提供的 C 头文件（如`<cstdio>`）和你可以更改的 C 头文件，准则略有不同

## 32.4 如何修改我自己的 C 头文件 ， 以便更容易的在 C++代码中包含他们？

如果你包含了不是由系统提供的 C 头文件，并且如果你能够改变的 C 头文件，你应该着重考虑通过添加 `extern"C"（...）`块到头文件，这样使 C++用户在 C++代码中更容易使用`#include`。由于 C 编译器通不过头文件含有 `extern"C"`的结构，你需要把`extern"C"{}`行包裹在`#ifdef`预编译块中，这样他们不会被正常的 C 编译器编译。

步骤#1：将以下行添加到你 C 头文件的顶部（注：符号`__cplusplus`当且仅当编译器是 C++编译器的时候被定义）：

```cpp
 #ifdef __cplusplus
 extern "C" {
 #endif 
```

步骤#2：将以下行添加到你 C 头文件的最底部：

```cpp
 #ifdef __cplusplus
 }
 #endif 
```

现在您可以`#include`你的 C 头文件，不用在 C++代码包含任何的`EXTERN "C"`：

```cpp
 // This is C++ code

 // Get declaration for f(int i, char c, float x)
 #include "my-C-code.h"   // Note: nothing unusual in #include line

 int main()
 {
   f(7, 'x', 3.14);       // Note: nothing unusual in the call
   ...
 } 
```

注： 对于系统提供的 C 头文件（如`<cstdio>`）和你可以更改的 C 头文件，准则略有不同

注：`#define`宏有 4 中罪恶： 罪恶#1 , 罪恶#2 , 罪恶#3 和罪恶#4 。但有时他们仍然有用。只要别忘了使用后洗清“罪恶”的双手。

## 32.5 如何从 C++代码中调用非系统 C 函数`f`（`int`，`char`和`float`）？

如果你有一个个人的 C 函数要调用，由于一些其他原因，你没有或不想在函数声明中`#include`一个 C 头文件，你可以在 C++代码通过`extern"C"`语法声明单个的 C 函数。当然，你需要使用完整的函数原型：

```cpp
extern "C" void f(int i, char c, float x); 
```

可以使用大括号声明几个 C 函数：

```cpp
 extern "C" {
   void   f(int i, char c, float x);
   int    g(char* s, const char* s2);
   double sqrtOfSumOfSquares(double a, double b);
 } 
```

在此之后你可以象调用 C++函数那样调用该函数：

```cpp
 int main()
 {
   f(7, 'x', 3.14);   // Note: nothing unusual in the call
   ...
 } 
```

## 32.6 如何创建一个 C++ 函数`f`（`int`，`char`和`float`），可以由我的 C 代码调用？

通过使用`EXTERN 的"C"`结构通知 C++编译器`f（int，char,float）`可由一个 C 编译器调用 ：

```cpp
 // This is C++ code

 // Declare f(int,char,float) using extern "C":
 extern "C" void f(int i, char c, float x);

 ...

 // Define f(int,char,float) in some C++ module:
 void f(int i, char c, float x)
 {
   ...
 } 
```

通过`extern"C"`行告诉编译器应该使用 C 调用约定和名字校正(name mangling)（例如，以下划线开头）来进行链接。由于 C 不支持重载，所以你不能编写可以由 C 程序调用的重载函数。

## 32.7 为什么链接器报错说 C / C++函数调用 C++ / C 函数？

如果你没有设置对`EXTERN`的`"C"`，你有时会得到链接错误，而不是编译器错误。这是由于 C++编译器通常是“校正(mangle)”函数名称（例如，为了支持函数重载），这和 C 编译器不同。

关于如何使用`EXTERN`的 `"C"`请参考前两个的 FAQs。

## 32.8 如何传递一个 C++ 类对象从/到一个 C 函数？

下面是一个例子（关于`extern"C"`，见前面的两个 FAQs）。

```cpp
 //Fred.h:

 /* This header can be read by both C and C++ compilers */
 #ifndef FRED_H
 #define FRED_H

 #ifdef __cplusplus
   class Fred {
   public:
     Fred();
     void wilma(int);
   private:
     int a_;
   };
 #else
   typedef
     struct Fred
       Fred;
 #endif

 #ifdef __cplusplus
 extern "C" {
 #endif

 #if defined(__STDC__) || defined(__cplusplus)
   extern void c_function(Fred*);   /* ANSI C prototypes */
   extern Fred* cplusplus_callback_function(Fred*);
 #else
   extern void c_function();        /* K&R style */
   extern Fred* cplusplus_callback_function();
 #endif

 #ifdef __cplusplus
 }
 #endif

 #endif /*FRED_H*/

 // Fred.cpp:

 // This is C++ code

 #include "Fred.h"

 Fred::Fred() : a_(0) { }

 void Fred::wilma(int a) { }

 Fred* cplusplus_callback_function(Fred* fred)
 {
   fred->wilma(123);
   return fred;
 }

 //main.cpp:

 // This is C++ code

 #include "Fred.h"

 int main()
 {
   Fred fred;
   c_function(&fred);
   ...
 }

 //c-function.c

 /* This is C code */

 #include "Fred.h"

 void c_function(Fred* fred)
 {
   cplusplus_callback_function(fred);
 } 
```

不像 C++代码，C 代码将无法告诉你两个指针是否指向同一个对象，除非指针完全相同。例如，在 C++可以很容易地检查一个派生类`Derived*`指针`dp`和`Base*`指针`bp`是否指向同一个对象，你可以使用`if(dp == bp)`。C++编译器自动转换指针为相同的类型，在这种情况下，转换为`Base*`，然后比较他们。根据不同的 C++编译器的实现细节，这种转换有时会改变一个指针值的位数据（bits）。但是 C 编译器不会知道该怎么做指针转换，所以比如从`Derived*`到`Base*`的转换，必须在由 C++编译器的编译的代码中，而不是在 C 编译器编译的 C 代码中。

*注意：你必须特别小心转换为`void*`指针，因为该转换将不会允许**C**或**C++**编译器做适当的指针调整！例如（继续前段内容），如果你把`dp`和`bp`赋值到两个`void *`指针比如说是`dpv`和`bpv`，有可能`dpv != bpv`即使`dp==bp`。不要说我没有提醒你！*

## 32.9 我的 C 函数可以直接访问一个 C++ 对象的数据吗？

有时。

（有关传递 C+ + 对象到/从 C 函数的基本内容，阅读以前的 FAQ）。

你可以安全地从 C 函数访问 C++对象的数据，如果 C++类：

*   没有虚函数（包括继承虚函数）
*   它的所有数据在相同的访问级别（私有/保护/公共）
*   虚函数没有完全包含的子对象

C++类有任何基类（或任何完全包含子对象具有基类），访问数据将在技术上是不可移植的，因为继承体系下的类的布局与语言无关。然而在实践中，所有 C++编译器都使用相同的方式：首先是基类对象（多重继承按照左到右的顺序），然后是成员对象。

此外，如果类（或任何基类）含有虚函数，几乎所有 C++编译器给对象添加一个 `void *`，在第一个虚拟函数的位置或在对象的开始位置。同样，这也不是语言要求的，但几乎所有的编译器都是这样实现的。

如果类有任何虚基类，这将更复杂，更不便于移植。一个常见的实现技术是，在对象的最后位置放置一个虚基类对象（ `v` ）（无论`v`在继承层次结构中的位置）。对象的其它部分按照正常顺序布局。每一个虚基类`v`的派生类，实际上都有一个指向`v`的指针 。

## 32.10 为什么 C++而不是为 C 让我觉得“更加远离机器”？

因为你就是。

作为一个面向对象编程语言，C++允许你对问题域建模，这将允许你使用问题域的语言编程，而不是解决方案域的语言。

C 的优势之一是，它已“没有任何隐藏的机制”：所见即所得。你可以阅读一个 C 程序，“看到”每个时钟周期。在 C++中可不是这样，老的 C 程序员（如我们许多人曾经是），对于这个特性往往会很矛盾（也许是“敌视”？）。但当他们转变到面向对象思想以后，他们往往认识到，虽然 C++的隐藏一些机制，但是它也提供了更高的抽象和更简洁的表达，从而能够在保持运行时性能的同时降低后期维护成本。

当然你可能会编写糟糕的代码不管使用任何语言，C++并不保证好的质量，可重用性，抽象，或任何“优异的”测试指标。

*C++中无法阻止糟糕的程序员编写的糟糕的程序，但是它能够让优秀的开发人员创建出色的软件。*