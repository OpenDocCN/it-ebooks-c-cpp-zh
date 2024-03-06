# [35] 模板

# [35] 模板

## FAQs in section [35]:

*   [35.1] 模板的设计思想是什么？
*   [35.2] 什么是 “类模板”的语法/语义？
*   [35.3] 什么是“函数模板”的语法/语义？
*   [35.4] 如何确定显式调用函数模板的哪个版本？
*   [35.5] 什么是“参数化类型”？
*   [35.6] 什么是“泛型”？
*   [35.7] 当模板类型 T 是 int 或 std::string 时，我的模板函数需要进行特殊处理。对特殊类型的 T 我该怎么实现模板特化？
*   [35.8] 哈？你能提供一个具体的模板特化的例子吗？
*   [35.9] 但是模板函数的大部分代码是相同的，是否有办法实现模板特化并且不用重复复制所有的源代码？
*   [35.10] 所有这些模板和模板特化都会降低程序执行速度，对不对？
*   [35.11] 因此 模板重载了函数，对不对？
*   [35.12] 为什么不能分开模板的声明和定义，把定义放到.cpp 文件中？
*   [35.13] 如何避免模板函数的链接错误？
*   [35.14] 如何使用 C++的关键字 export 来避免模板链接错误？
*   [35.15] 如何避免模板类的链接错误？
*   [35.16] 为什么我收到链接错误 ，当我使用模板友元的时候？
*   [35.17] 怎么理解这些繁琐的模板错误信息？
*   [35.18]当模板派生类使用一个继承自模板基类的嵌套类型时，为什么出错？
*   [35.19]当模板派生类使用使用一个继承自模板基类的成员变量时 ，为什么出错？
*   [35.20] 前一个问题可以暗伤我？难道编译器默认地产生错误代码？

## 35.1 模板的设计思想是什么？

模板像是甜饼切割器，指定如何切割 cookies 让他们看起来大致相同（虽然 Cookie 由各种面团来制作，但是他们都会有相同的基本形状）。同样，类模板是描述如何建立一个类族，让所有的类看起来是基本相同；函数模板描述如何建立一个外观类似的函数族。

类模板通常用于构建类型安全的容器（although this only scratches the surface for how they can be used）。

## 35.2 什么是 “类模板”的语法/语义？

考虑一个容器`类 class Array`,它的行为像一个整数数组：

```cpp
 // This would go into a header file such as "__Array.h__"
 class Array {
 public:
   Array(int len=10)                  : len_(len), data_(new int[len]) { }
  ~Array()                            { delete[] data_; }
   int len() const                    { return len_;     }
   const int& operator[](int i) const { return data_[check(i)]; }  ← subscript operators often come in pairs What's the deal with "const-overloading"?")
         int& operator[](int i)       { return data_[check(i)]; }  ← subscript operators often come in pairs What's the deal with "const-overloading"?")
   Array(const Array&);
   Array& operator= (const Array&);
 private:
   int  len_;
   int* data_;
   int  check(int i) const
     { if (i < 0 || i >= len_) throw BoundsViol("Array", i, len_);
       return i; }
 }; 
```

对于浮点数数组，字符数组，`std::string`数组，`std::string`数组的数组等，反复重复上述步骤将很冗长乏味。

```cpp
 // This would go into a header file such as "__Array.h__"
 template<typename T>
 class Array {
 public:
   Array(int len=10)                : len_(len), data_(new T[len]) { }
  ~Array()                          { delete[] data_; }
   int len() const                  { return len_;     }
   const T& operator[](int i) const { return data_[check(i)]; }
         T& operator[](int i)       { return data_[check(i)]; }
   Array(const Array<T>&);
   Array<T>& operator= (const Array<T>&);
 private:
   int len_;
   T*  data_;
   int check(int i) const
     { if (i < 0 || i >= len_) throw BoundsViol("Array", i, len_);
       return i; }
 }; 
```

与模板函数不同,模板类（实例化模板）在实例化时需要指明相关参数：

```cpp
 int main()
 {
   Array<int>           ai;
   Array<float>         af;
   Array<char*>         ac;
   Array<std::string>   as;
   Array< Array<int> >  aai;
   ...
 } 
```

注意最后一个例子中的两个`>`之间的空格符。如果没有这个空格符，编译器会看到一个`>>`（右移位）标记，而不是两个`>`。

## 35.3 什么是“函数模板”的语法/语义？

考虑下面函数，交换两个整型参数：

```cpp
 void swap(int& x, int& y)
 {
   int tmp = x;
   x = y;
   y = tmp;
 } 
```

如果我们还要交换浮点数，长整形，字符串，集合，和文件系统等，我们就会疲于编写除了类型不同的相似的编码行。重复是电脑理想的工作，因此要用函数模板：

```cpp
 template<typename T>
 void swap(T& x, T& y)
 {
   T tmp = x;
   x = y;
   y = tmp;
 } 
```

对给定的类型每次我们使用 swap()的时候，编译器将根据上述定义，并自动产生另外一个“模板函数”作为上述函数模板的实例化。例如：

```cpp
 int main()
 {
   int         i,j;  /*...*/  swap(i,j);  // Instantiates a swap for int
   float       a,b;  /*...*/  swap(a,b);  // Instantiates a swap for float
   char        c,d;  /*...*/  swap(c,d);  // Instantiates a swap for char
   std::string s,t;  /*...*/  swap(s,t);  // Instantiates a swap for std::string
   ...
 } 
```

注：“模板函数”是一个“函数模板”的实例化形态。

## 35.4 如何确定显式调用函数模板的哪个版本？

当你调用一个函数模板时，编译器试图推断模板类型。大部分情况下，编译器可以成功的做到这一点，但有时你可能想要帮助编译器推断出正确的类型-要么是因为它不能推断出模板类型，或者是因为它会推断出错误类型。

例如，你可能会调用一个函数模板没有模板指定的参数类型，或者你可能想让编译器在选择正确的函数模板之前，迫使它对参数做一些转换（promotions）。在这些情况下，你需要明确地告诉编译器应该调用函数的模板哪个实例化。

下面是一个示例函数模板，模板参数 T 没有出现在函数的参数列表中。在这种情况下， 编译器无法推断出模板参数类型在函数被调用时。

```cpp
 template<typename T>
 void f()
 {
   ...
 } 
```

若要调用该函数把 `T`作为`int`或`std::string`，你可以这样做：

```cpp
 #include <string>

 void sample()
 {
   f<int>();          // type T will be int in this call
   f<std::string>();  // type T will be std::string in this call
 } 
```

这里是另一个函数，它的模板参数出现在函数的正式参数列表中（也就是说，编译器*可以*根据实际参数的类型推导出模板类型）：

```cpp
 template<typename T>
 void g(T x)
 {
   ...
 } 
```

现在如果你想强制实行参数转换，在编译器推断模板类型之前，你可以使用上述技术。例如，如果你只是简单调用`g(42)`，你会得到`g<int>（42）`，但如果你想传递 42 给`g<long>()`，你可以这样做： `g<long>（42）`。（当然你也可以明确地转换参数，如可以`g（long（42））`，甚至`g（42L）`，当然如果这样的话本例子就没有什么意义了。）

同样，如果你调用`g（“xyz”）`，你最终会调用`g<char*>（char*）`，但如果你想调用`std::string`版本`g<>()`，你可以这样`g<std::string>（”xyz“）`。（同样你也可以转换参数，例如`g(std::string(“xyz”)`，不过那将是另一回事。）

## 35.5 什么是“参数化类型”？

换句话说，“类模板”。

参数化类型是一个类型，是参数化的类型或者值。 `list<int>`是一个被另外一个类型(`int`)参数化的类型（ `List` ）。

## 35.6 什么是“泛型”？

还是“类模板”另一种说法。

不要 与“一般性(generality)”混淆（“一般性(generality)”这只是避免过于具体的解决方案），“泛型”是指类模板。

## 35.7 当模板类型`T`是 `int`或`std::string`时，我的模板函数需要进行特殊处理。对特殊类型的 T 我该怎么实现模板特化？

在展示如何做到这一点之前，让我们确保你不会搬起石头砸自己的脚。对于用户来说是否该函数的行为不同？换言之，是否可以观察到的行为有实质性的不同？如果是这样，你可能是在自找苦吃，你可能迷惑用户--你最好使用不同名称的函数--不要使用模板，不要使用重载。例如，如果接受`int`类型的代码要插入一些东西到容器并且对结果排序，但接受`std::string`类型的代码要从容器中删除东西并且不对结果排序，这两个函数不应该是可以重载的函数对--他们可以观察的行为是不同的，所以他们应该有不同的函数名称。

但是，如果该函数的可观察到的行为是一致的，对于所有 T 类型仅仅局限在各自实现细节上的不同，那么就请继续读下去。让我们看看这方面的一个例子（仅仅是概念上，不是 C++代码）：

```cpp
 template<typename T>
 void foo(const T& x)
 {
   switch (typeof(T)) {  ← conceptual only; not C++
     case int:
       ...  ← implementation details when T is int
       break;

     case std::string:
       ...  ← implementation details when T is std::string
       break;

     default:
       ...  ← implementation details when T is neither int nor std::string
       break;
   }
 } 
```

解决上述问题的办法就是是通过模板特化。不要使用`switch`语句，你需要把代码分解成单独的函数。第一个函数是默认的情况--当 `T`是`int`或`std::string`以外的任何其他类型时候的代码：

```cpp
 template<typename T>
 void foo(const T& x)
 {
   ...  ← implementation details when T is neither int nor std::string
 } 
```

下一步是两个特例，第一个是`int`特例 的代码：

```cpp
 template<>
 void foo<int>(const int& x)
 {
   ...  ← implementation details when T is int
 } 
```

接着是`std::string`特例 的代码：

```cpp
 template<>
 void foo<std::string>(const std::string& x)
 {
   ...  ← implementation details when T is std::string
 } 
```

好啦，大功告成！编译器将自动选择正确的特例实现根据所使用的`T`的类型。

## 35.8 哈？你能提供一个具体的模板特化的例子吗？

可以。

下面我个人使用模板特化的几种常见情况是字符串化。我通常使用模板， 将不同类型的对象字符串化，但通常需要字符串化某些特定的类型，例如当字符串化 布尔变量的时候，我喜欢用“true”与“false”来代替“1”和“0”，所以当 T 是布尔类型时，我使用 std::boolalpha 。此外，我喜欢浮点输出包含所有的数字（这样我就可以看得很小的差异，等等），因此当 T 是一个浮点类型时候，我使用`std::setprecision`。最终的结果通常如下所示：

```cpp
 #include <iostream>
 #include <sstream>
 #include <iomanip>
 #include <string>
 #include <limits>

 template<typename T> inline std::string stringify(const T& x)
 {
   std::ostringstream out;
   out << x;
   return out.str();
 }

 template<> inline std::string stringify<bool>(const bool& x)
 {
   std::ostringstream out;
   out << std::boolalpha << x;
   return out.str();
 }

 template<> inline std::string stringify<double>(const double& x)
 {
   const int sigdigits = std::numeric_limits<double>::digits10;
   // or perhaps std::numeric_limits<double>::max_digits10 if that is available on your compiler
   std::ostringstream out;
   out << std::setprecision(sigdigits) << x;
   return out.str();
 }

 template<> inline std::string stringify<float>(const float& x)
 {
   const int sigdigits = std::numeric_limits<float>::digits10;
   // or perhaps std::numeric_limits<float>::max_digits10 if that is available on your compiler
   std::ostringstream out;
   out << std::setprecision(sigdigits) << x;
   return out.str();
 }

 template<> inline std::string stringify<long double>(const long double& x)
 {
   const int sigdigits = std::numeric_limits<long double>::digits10;
   // or perhaps std::numeric_limits<long_double>::max_digits10 if that is available on your compiler
   std::ostringstream out;
   out << std::setprecision(sigdigits) << x;
   return out.str();
 } 
```

从概念上来讲他们都做同样的事情：把参数字符串化。这意味着可观察的行为是一致的，因此特化不会迷惑用户。但对于`bool`和浮点类型，细节的实现略有不同，因此模板特化是一个好的解决方法。

## 35.9 但是模板函数的大部分代码是相同的，是否有办法实现模板特化并且不用重复复制所有的源代码？

是。

例如，假设你的模板函数有很多共同的代码，与类型 T 相关的特定代码相对很少（仅仅是概念展示;不是 C++）：

```cpp
 template<typename T>
 void foo(const T& x)
 {
   ... common code that works for all T types ...

   switch (typeof(T)) {  ← conceptual only; not C++
     case int:
       ... small amount of code used only when T is int ...
       break;

     case std::string:
       ... small amount of code used only when T is std::string...
       break;

     default:
       ... small amount of code used when T is neither int nor std::string ...
       break;
   }

   ... more common code that works for all T types ...
 } 
```

如果盲目地跟从模板特化 FAQ 的建议，你最终将需要重复`switch`语句之前和之后的所有代码。两全其美的方式—既不重复相同代码又可以实现`T`的特定代码，是分离`switch`语句到一个单独的函数`foo_part()`，并使用模板特殊化：

```cpp
 template<typename T> inline void foo_part(const T& x)
 {
   ... small amount of code used when T is neither int nor std::string ...
 }

 template<> inline void foo_part<int>(const int& x)
 {
   ... small amount of code used only when T is int ...
 }

 template<> inline void foo_part<std::string>(const std::string& x)
 {
   ... small amount of code used only when T is std::string ...
 } 
```

主要的`foo()`函数是一个简单的模板-没有特化。请注意，`switch`语句已经被替换为`foo_part()`调用：

```cpp
 template<typename T>
 void foo(const T& x)
 {
   ... common code that works for all T types ...

   foo_part(x);

   ... more common code that works for all T types ...
 } 
```

正如你所看到的， `foo()`的函数体本身并没有任何特殊，这一切都会自动的被调用。编译器自动生成的基于 `T`类型 的`foo()`，并会生成正确的`foo_part`函数，根据实际编译时的`X`的参数类型。合适的`foo_part`的特化会被实例化。

## 35.10 所有这些模板和模板特化都会降低程序执行速度，对不对？

错误的。

这与实现代码的质量有关，结果可能会有所不同。但是不会有任何降低。模板可能会些微影响编译速度，但一旦类型在编译时被确定，它通常会生成和非模板函数（包括内联展开等）一样快的代码。

## 35.11 因此模板重载了函数，对不对？

是也不是。

函数模板参与重载函数的名称解析，但规则是不同的。对于模板重载，类型需要完全匹配。如果类型不完全匹配，类型不会被转换，函数模板从可行的函数集合中被排除。这就是所谓的“SFINAE”- Subsitution Failure Is Not An Error。例如：

```cpp
 #include <iostream>
 #include <typeinfo>

 template<typename T> void foo(T* x)
 { std::cout << "foo<" << typeid(T).name() << ">(T*)\n"; }

 void foo(int x)
 { std::cout << "foo(int)\n"; }

 void foo(double x)
 { std::cout << "foo(double)\n"; }

 int main()
 {
     foo(42);        // matches foo(int) exactly
     foo(42.0);      // matches foo(double) exactly
     foo("abcdef");  // matches foo<T>(T*) with T = char
     return 0;
 } 
```

在这个例子中， 在 main()函数中第一或第二次调用`foo`不是对`foo<T>`的调用，因为无论 42 还是 42.0 都没有提供给编译器的任何信息来推断 。然而第三个调用，包括`foo<T>`并且`T = char`，因此它会调用`foo<T>`。

## 35.12 为什么不能分开模板的声明和定义，把定义放到`.cpp`文件中？

如果你想知道的是只是如何解决这种情况，请阅读下面得两个 s。但是，为了理解要那样，首先接受这些事实：

1.  模板是不是一个类或函数。 模板是一个“模式”，编译器用来生成的相似的类或者函数。
2.  为了让编译器生成的代码，它必须同时看到模板的定义（不只是声明）和特定类型/任何用于“fill in”模板的类型。例如，如果你想使用一个`foo<int>`，编译器必须同时看到 foo 模板和你要调用具体的`foo<int>`。
3.  编译器可能不记得另外一个`.cpp`文件的细节，当编译其他`.cpp`文件的时候。它可以 ，但大多数都没有，如果你正在阅读本 FAQ，它几乎肯定不会。顺便说一句，这就是所谓的“独立编译模型”。

现在，基于这些事实，下面是一个范例，它表明为什么是这个样子。假设你有一个这样的模板`Foo`声明：

```cpp
 template<typename T>
 class Foo {
 public:
   Foo();
   void someMethod(T x);
 private:
   T x;
 }; 
```

类似地，模板成员函数的定义：

```cpp
 template<typename T>
 Foo<T>::Foo()
 {
   ...
 }

 template<typename T>
 void Foo<T>::someMethod(T x)
 {
   ...
 } 
```

现在，假设在文件`Bar.cpp`的一些代码要使用`foo<int>`：

```cpp
 // Bar.cpp

 void blah_blah_blah()
 {
   ...
   Foo<int> f;
   f.someMethod(5);
   ...
 } 
```

显然，某人某地将不得不调用“模式”的构造函数，和`someMethod()`函数以及做`T`为`int`的实例化。但是，如果你把构造函数和`someMethod()`的定义放到文件`Foo.cpp`，当编译`Foo.cpp`时，编译器将看到模板代码；当编译`Bar.cpp`时，编译器将看到`foo<int>`。但任何时候决不会同时看到模板代码和`foo<int>`。因此，通过上面的 2 号规则，它根本不会产生`foo <int>::someMethod()`的代码。

*写给专家们的话：很明显我对以上内容作了简化。这是有意为之，所以请不要大声抱怨。*如果你知道`.cpp`文件和编译单元的差别，类模板和模板类的差别，模板其实不只是美化的宏等，请不要抱怨：这个问题/解答不是为你而设。我简化它是为了新手能够“理解它”，即使这样可能会冒犯一些专家。

*提醒：*欲知解决方案，请阅读下面得两个 FAQs。

## 35.13 如何避免模板函数的链接错误？

当编译模板函数的`.cpp`文件的时候告诉 C++编译器应该使用哪个实例。

例如，考虑`foo.h`头文件包含以下模板函数声明：

```cpp
 // File "foo.h"
 template<typename T>
 extern void foo(); 
```

现在假设文件`foo.cpp`实际上定义的模板函数：

```cpp
 // File "foo.cpp"
 #include <iostream>
 #include "foo.h"

 template<typename T>
 void foo()
 {
   std::cout << "Here I am!\n";
 } 
```

假设文件`main.cpp`中使用这个模板函数通过调用`foo<int>()`：

```cpp
 // File "main.cpp"
 #include "foo.h"

 int main()
 {
   foo<int>();
   ...
 } 
```

如果你编译和（试图）链接这两个`.cpp`文件，大多数编译器将生成链接错误。有三种的解决方案。第一个解决方案是物理上在`.h`文件中定义，即使它不是一个内联函数。这种解决办法可能（或可能不会！）造成重大代码膨胀，意味着可执行文件的大小可能会显显著增加（或者，如果你的编译器足够聪明，可能不会这么做）。

另一个解决办法是保留定义在`.cpp`文件中，只添加行`template void foo<int>()`到`.cpp`文件：

```cpp
 // File "foo.cpp"
 #include <iostream>
 #include "foo.h"

 template<typename T> void foo()
 {
   std::cout << "Here I am!\n";
 }

 template void foo<int>(); 
```

如果你不能修改`foo.cpp`，只需创建一个新的`.cpp`文件，例如`foo-impl.cpp`如下：

```cpp
 // File "foo-impl.cpp"
 #include "foo.cpp"

 template void foo<int>(); 
```

请注意， `foo-impl.cpp`文件包含`.cpp`文件，而不是`.h`文件。如果你觉着这样很乱，跳个踢踏舞，想想堪萨斯，跟着我重复，“我要这么做即使它很混乱。” 你需要信任我。如果不信任或者致使好奇，前面的 FAQ 给出了理由。

## 35.14 如何使用 C++的关键字`export`来避免模板链接错误？

C++关键字`export`是设计用来消除包含一个模板定义（无论是在头文件中或通过实现文件中）的需要。但是，在写这篇文章时，支持此功能的唯一的知名编译器，是[Comeau C++](http://www.comeaucomputing.com/tryitout)。`export`关键字未来还是个未知数。说句公道话，一些编译器厂商表示他们可能永远不会实现它，而 C++标准委员会已决定大家自己定夺。

在不支持关键字`export`的编译器上，如果你希望你的代码可以通过编译，并且还希望能够有效利用支持`export`关键字的编译器。你可以这样定义模板头文件：

```cpp
 // File Foo.h

 template<typename T>
 class Foo {
   ...
 };

 #ifndef USE_EXPORT_KEYWORD
   #include "Foo.cpp"
 #endif 
```

并定义非内联函数的源代码文件如下：

```cpp
 // File Foo.cpp

 #ifndef USE_EXPORT_KEYWORD
   #define export /*nothing*/
 #endif

 export template<typename T> ... 
```

然后，如果/当你的编译器支持`export`关键字的时候，并且因为某些原因你想利用该功能，只要定义符号`USE_EXPORT_KEYWORD`即可。

要诀就是，你现在可以开发程序， 好像你的编译器已经实现了`export`关键字。如果/当你的编译器真正支持该关键字的时候，只需要定义`USE_EXPORT_KEYWORD`标志，重新编译，马上你就可以利用该功能。

## 35.15 如何避免模板类的链接错误？

当编译模板类的`.cpp`文件得手告诉你的 C++编译器应该使用哪个模板实例。（如果你已经阅读以前的问题，答案是完全一样的，所以你也许可以跳过此答案。）

作为一个例子，考虑`Foo.h`头文件包含以下模板类。请注意， `Foo<T>::f()`方法是内联的，而`Foo<T>::g()`和`Foo<T>::h()`却不是。

```cpp
 // File "Foo.h"
 template<typename T>
 class Foo {
 public:
   void f();
   void g();
   void h();
 };

 template<typename T>
 inline
 void Foo<T>::f()
 {
   ...
 } 
```

现在，假设文件`Foo.cpp`实际定义了非内联的`Foo<T>::g()`和`Foo<T>::h()`：

```cpp
 // File "Foo.cpp"
 #include <iostream>
 #include "Foo.h"

 template<typename T>
 void Foo<T>::g()
 {
   std::cout << "Foo<T>::g()\n";
 }

 template<typename T>
 void Foo<T>::h()
 {
   std::cout << "Foo<T>::h()\n";
 } 
```

假设文件`main.cpp`使用该模板创建一个`Foo<int>`并调用其方法：

```cpp
 // File "main.cpp"
 #include "Foo.h"

 int main()
 {
   Foo<int> x;
   x.f();
   x.g();
   x.h();
   ...
 } 
```

如果你编译和（试图）链接这两个`.cpp`文件，大多数编译器将生成链接错误。有三种的解决方案。第一个解决方案是物理上在`.h`文件中定义，即使它不是一个内联函数。这种解决办法可能（或可能不会！）造成重大代码膨胀，意味着可执行文件的大小可能会显显著增加（或者，如果你的编译器足够聪明，可能不会这么做）。

另一个解决办法是保留定义在`.cpp`文件中，只添加行`template class Foo<int>;`到`.cpp`文件：

```cpp
 // File "Foo.cpp"
 #include <iostream>
 #include "Foo.h"

 ...definition of Foo<T>::f() is unchanged -- see above...
 ...definition of Foo<T>::g() is unchanged -- see above...

 template class Foo<int>; 
```

如果你不能修改`foo.cpp`，只需创建一个新的`.cpp`文件，例如`foo-impl.cpp`如下：

```cpp
 // File "Foo-impl.cpp"
 #include "Foo.cpp"

 template class Foo<int>; 
```

请注意， `foo-impl.cpp`文件包含`.cpp`文件，而不是`.h`文件。如果你觉着这样很乱，跳个踢踏舞，想想堪萨斯，跟着我重复，“我要这么做即使它很混乱。” 你需要信任我。如果不信任或者致使好奇，前面的 FAQ 给出了理由。

如果你使用[Comeau C++](http://www.comeaucomputing.com/tryitout "www.comeaucomputing.com/tryitout")，你可能使用`export`关键字实现类似功能。

## 35.16 为什么我收到链接错误 ，当我使用模板友元的时候？

由于模板友类的复杂性。下面是一个常见的例子：

```cpp
 #include <iostream>

 template<typename T>
 class Foo {
 public:
   Foo(const T& value = T());
   friend Foo<T> operator+ (const Foo<T>& lhs, const Foo<T>& rhs);
   friend std::ostream& operator<< (std::ostream& o, const Foo<T>& x);
 private:
   T value_;
 }; 
```

当然在某个地方我们会用到模板：

```cpp
 int main()
 {
   Foo<int> lhs(1);
   Foo<int> rhs(2);
   Foo<int> result = lhs + rhs;
   std::cout << result;
   ...
 } 
```

当然，在某个地方需要定义各成员和友元函数：

```cpp
 template<typename T>
 Foo<T>::Foo(const T& value = T())
   : value_(value)
 { }

 template<typename T>
 Foo<T> operator+ (const Foo<T>& lhs, const Foo<T>& rhs)
 { return Foo<T>(lhs.value_ + rhs.value_); }

 template<typename T>
 std::ostream& operator<< (std::ostream& o, const Foo<T>& x)
 { return o << x.value_; } 
```

一个潜在问题是编译器如何理解类声明中的 friends 行。在看到 friends 行的时候，它还不知道友元函数本身也是模板，它假定他们不是模板函数，就像下面这样：

```cpp
 Foo<int> operator+ (const Foo<int>& lhs, const Foo<int>& rhs)
 { ... }

 std::ostream& operator<< (std::ostream& o, const Foo<int>& x)
 { ... } 
```

当你调用运算符`+`或运算符`<<`的时候，这种假设导致编译器生成一个对非模板函数的调用，但是链接器会给你一个“未定义的外部函数”错误，因为你从来没有真正的定义这些非模板函数。

解决的办法是在编译器编译类体的时候，让编译器知道运算符`+`和运算符`<<`本身是模板。有几种方法可以做到这一点；一个简单的方法是在定义函数模板类 Foo 的时候预先声明模板友元：

```cpp
 template<typename T> class Foo;  // pre-declare the template class itself
 template<typename T> Foo<T> operator+ (const Foo<T>& lhs, const Foo<T>& rhs);
 template<typename T> std::ostream& operator<< (std::ostream& o, const Foo<T>& x); 
```

在`frend`行中你也需要加入`<>`，如下所示：

```cpp
 #include <iostream>

 template<typename T>
 class Foo {
 public:
   Foo(const T& value = T());
   friend Foo<T> operator+ <> (const Foo<T>& lhs, const Foo<T>& rhs);
   friend std::ostream& operator<< <> (std::ostream& o, const Foo<T>& x);
 private:
   T value_;
 }; 
```

这些写法将有助于编译器更好地了解友元函数。值得一提的是，它会发现友元函数本身是模板。这消除了混乱。

另一种方法是在类中同时声明和定义该友元函数。例如：

```cpp
 #include <iostream>

 template<typename T>
 class Foo {
 public:
   Foo(const T& value = T());

   friend Foo<T> operator+ (const Foo<T>& lhs, const Foo<T>& rhs)
   {
        ...
   }

   friend std::ostream& operator<< (std::ostream& o, const Foo<T>& x)
   {
        ...
   }

 private:
   T value_;
 }; 
```

## 35.17 怎么理解这些繁杂的模板错误信息？

这里有一个免费工具， [可以转换错误信息便于理解](http://www.bdsoft.com/tools/stlfilt.html)。在撰写本文的时候，它工作用于下列编译器：Comeau C +，Intel C++，CodeWarrior C++，gcc，Borland C++，Microsoft Visual C++和 EDG C++。

这里有一个例子，下面是一些原始的 gcc 的错误信息：

```cpp
 rtmap.cpp: In function int main()':
 rtmap.cpp:19: invalid conversion from int' to 
    std::_Rb_tree_node<std::pair<const int, double> >*'
 rtmap.cpp:19:   initializing argument 1 of std::_Rb_tree_iterator<_Val, _Ref,
    _Ptr>::_Rb_tree_iterator(std::_Rb_tree_node<_Val>*) [with _Val =
    std::pair<const int, double>, _Ref = std::pair<const int, double>&, _Ptr =
    std::pair<const int, double>*]'
 rtmap.cpp:20: invalid conversion from int' to 
    std::_Rb_tree_node<std::pair<const int, double> >*'
 rtmap.cpp:20:   initializing argument 1 of std::_Rb_tree_iterator<_Val, _Ref,
    _Ptr>::_Rb_tree_iterator(std::_Rb_tree_node<_Val>*) [with _Val =
    std::pair<const int, double>, _Ref = std::pair<const int, double>&, _Ptr =
    std::pair<const int, double>*]'
 E:/GCC3/include/c++/3.2/bits/stl_tree.h: In member function void
    std::_Rb_tree<_Key, _Val, _KeyOfValue, _Compare, _Alloc>::insert_unique(_II,
     _II) [with _InputIterator = int, _Key = int, _Val = std::pair<const int,
    double>, _KeyOfValue = std::_Select1st<std::pair<const int, double> >,
    _Compare = std::less<int>, _Alloc = std::allocator<std::pair<const int,
    double> >]':
 E:/GCC3/include/c++/3.2/bits/stl_map.h:272:   instantiated from void std::map<_
 Key, _Tp, _Compare, _Alloc>::insert(_InputIterator, _InputIterator) [with _Input
 Iterator = int, _Key = int, _Tp = double, _Compare = std::less<int>, _Alloc = st
 d::allocator<std::pair<const int, double> >]'
 rtmap.cpp:21:   instantiated from here
 E:/GCC3/include/c++/3.2/bits/stl_tree.h:1161: invalid type argument of unary *
    ' 
```

以下是经过过滤的错误信息（注：你可以配置工具让它显示更多的信息，下面输出的设置是剪裁信息到最少）：

```cpp
 rtmap.cpp: In function int main()':
 rtmap.cpp:19: invalid conversion from int' to iter'
 rtmap.cpp:19:   initializing argument 1 of iter(iter)'
 rtmap.cpp:20: invalid conversion from int' to iter'
 rtmap.cpp:20:   initializing argument 1 of iter(iter)'
 stl_tree.h: In member function void map<int,double>::insert_unique(_II, _II)':
     [STL Decryptor: Suppressed 1 more STL standard header message]
 rtmap.cpp:21:   instantiated from here
 stl_tree.h:1161: invalid type argument of unary *' 
```

以下是上面例子的源代码：

```cpp
 #include <map>
 #include <algorithm>
 #include <cmath>

 const int values[] = { 1,2,3,4,5 };
 const int NVALS = sizeof values / sizeof (int);

 int main()
 {
     using namespace std;

     typedef map<int, double> valmap;

     valmap m;

     for (int i = 0; i < NVALS; i++)
         m.insert(make_pair(values[i], pow(values[i], .5)));

     valmap::iterator it = 100;              // error
     valmap::iterator it2(100);              // error
     m.insert(1,2);                          // error

     return 0;
 } 
```

## 35.18 当模板派生类使用一个继承自模板基类的嵌套类型时，为什么出错？

你也许很吃惊，下面的代码是无效的 C++代码，即使如此通过有些编译器：

```cpp
 template<typename T>
 class B {
 public:
   class Xyz { ... };  ← type nested in class B<T>
   typedef int Pqr;    ← type nested in class B<T>
 };

 template<typename T>
 class D : public B<T> {
 public:
   void g()
   {
     Xyz x;  ← bad (even though some compilers erroneously (temporarily?) accept it)
     Pqr y;  ← bad (even though some compilers erroneously (temporarily?) accept it)
   }
 }; 
```

这可能会让你很伤脑筋，最好坐下来听我讲。

在函数`D<T>::g()`内，名字`xyz`和`Pqr`不依赖于模板参数`T`，所以他们被称作为 nondependent 名字。另一方面`B<T>` 依赖模板参数`T`，因此 `B<T>` 称作*dependent 名字*。

规则是这样的：当查找 nondependent 名字（比如`Xyz`和`Pqr`）的时候，编译器不会查找 dependent 基类（如`B <T>`中 ）。因此，编译器不知道他们甚至还存在，更不用说知道它们也是类型。

这时，程序员有时会添加前缀`B <T>::`，例如：

```cpp
 template<typename T>
 class D : public B<T> {
 public:
   void g()
   {
     B<T>::Xyz x;  ← bad (even though some compilers erroneously (temporarily?) accept it)
     B<T>::Pqr y;  ← bad (even though some compilers erroneously (temporarily?) accept it)
   }
 }; 
```

可惜这也行不通，因为这些名字（你准备好了吗？坐下来？）不一定是类型。 "哈?!?" ？"不是类型?!?" ？。“太搞了吧！任何傻瓜都可以看到他们是类型;只要看上一眼！”，你抗议。抱歉，事实是，他们可能不是类型。原因是，有可能是`B<T>`的特化，假设`B<Foo>`，其中 `B <Foo>::Xyz`是一个数据成员。由于这种潜在的特化，编译器不能假设`B<T>::Xyz`是一个类型，直到它知道`T`。解决方案是通过`typename`关键字提示编译器：

```cpp
 template<typename T>
 class D : public B<T> {
 public:
   void g()
   {
     typename B<T>::Xyz x;  ← good
     typename B<T>::Pqr y;  ← good
   }
 }; 
```

## 35.19 当模板派生类使用使用一个继承自模板基类的成员变量时 ，为什么出错？

你也许很吃惊，下面的代码是无效的 C++代码，即使如此通过有些编译器：

```cpp
 template<typename T>
 class B {
 public:
   void f() { }  ← member of class B<T>
 };

 template<typename T>
 class D : public B<T> {
 public:
   void g()
   {
     f();  ← bad (even though some compilers erroneously (temporarily?) accept it)
   }
 }; 
```

这可能会让你很伤脑筋，最好坐下来听我讲。

在函数`D<T>::g()`内，名字`f`不依赖于模板参数`T`，所以他们被称作为 nondependent 名字。另一方面`B<T>` 依赖模板参数`T`，因此 `B<T>` 称作*dependent 名字*。

规则是这样的：当查找 nondependent 名字（比如 f）的时候，编译器不会查找 dependent 基类（如`B <T>`中 ）。

这并不意味着继承不起作用。类`D <int>`是仍然继承自类`B <int>`，编译器仍然让你可以隐式的做 is- a 转换（例如，`D<int>*`到 `B <int> *`），动态绑定仍然有效当虚函数被调用时，等等。但有一个如何查找名称的问题。

替代方案：

*   改变的`f()`的调用为`this->f()`。由于在模板中`this`指针一直是隐式实现的，`this->f()`要依赖查找，因此推迟到模板实例化时，此时所有基类都会被查找。
*   在调用`f()`之前，插入 `using B<T>::f;`语句。
*   改变的`f()`的调用为`B <T>::f()`。 但是请注意，如果`f()`是虚函数，这可能没有给你想要的东西，因为它禁止了虚函数带调用机制。

## 35.20 前一个问题可以暗伤我？难道编译器默认地产生错误代码？

是。

由于 non-dependent 类型 and non-dependent 成员不会在 dependent 模板在基础类中搜索，编译器将搜索封闭范围，比如封闭名字空间。这可能会导致它在你没有意识到的情况下（！）做错误的事情。

例如：

```cpp
 class Xyz { ... };  ← global ("namespace scope") type
 void f() { }        ← global ("namespace scope") function

 template<typename T>
 class B {
 public:
   class Xyz { ... };  ← type nested in class B<T>
   void f() { }        ← member of class B<T>
 };

 template<typename T>
 class D : public B<T> {
 public:
   void g()
   {
     Xyz x;  ← suprise: you get the global Xyz!!
     f();    ← suprise: you get the global f!!
   }
 }; 
```

`D<T>::g()`内的`Xyz`和`f`将被解析为全局变量，而不是继承自类`B <T>`，这恐怕不是你的真正意图。

别埋怨我没有警告过你。