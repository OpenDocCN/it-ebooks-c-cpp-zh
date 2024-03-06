# [10] 构造函数

## FAQs in section [10]:

*   [10.1] 构造函数做什么？
*   [10.2] `List x;` 和 `List x();`有区别吗？
*   [10.3] 如何才能够使一个构造函数直接地调用另一个构造函数？
*   [10.4] `Fred` 类的默认构造函数总是 `Fred::Fred()`吗？
*   [10.5] 当我建立一个 `Fred` 对象数组时，哪个构造函数将被调用？
*   [10.6] 构造函数应该用“初始化列表”还是“赋值”？
*   [10.7] 可以在构造函数中使用 `this` 指针吗？
*   [10.8] 什么是“命名的构造函数用法（Named Constructor Idiom）”？
*   [10.9] 为何不能在构造函数的初始化列表中初始化静态成员数据？
*   [10.10] 为何有静态数据成员的类得到了链接错误？
*   [10.11] 什么是“`static` initialization order fiasco”？
*   [10.12] 如何防止“`static` initialization order fiasco”？
*   [10.13] 对于静态数据成员，如何防止“`static` initialization order fiasco”？
*   [10.14] 如何处理构造函数的失败？
*   [10.15] 什么是“命名参数用法（Named Parameter Idiom）”？

## 10.1 构造函数做什么？

构造函数从无到有创建对象。

构造函数就象“初始化函数”。它将一连串的随意的内存位变成活的对象。至少它要初始化对象内部所使用的域。它还可以分配资源（内存、文件、信号、套接字等）

"ctor" 是构造函数(constructor)典型的缩写。

## 10.2 `List x;` 和 `List x();`有区别吗?

有非常大的区别！

假设`List` 是某个类的名称。那么函数`f()` 中声明了一个局部的 `List`对象，名称为 `x`：

```cpp
 void f()
 {
   List x;     // Local object named x (of class List)// ...
 } 
```

但是函数 `g()` 中声明了一个名称为`x()`的函数，它返回一个 `List`：

```cpp
 void g()
 {
   List x();   // Function named x (that returns a List)// ...
 } 
```

## 10.3 如何才能够使一个构造函数直接地调用另一个构造函数？

不行。

注意：如果你调用了另一个构造函数，编译器将初始化一个临时局部对象；而不是初始化`this`对象。你可以通过一个默认参数或在一个私有成员函数 `init()` 中共享它们的公共代码来使两个构造函数结合起来。

## 10.4 `Fred` 类的默认构造函数总是`Fred::Fred()`吗？

不。“默认构造函数”是能够被无参数调用的构造函数。因此，一个不带参数的构造函数当然是默认构造函数：

```cpp
 class Fred {
 public:
   Fred();   // 默认构造函数: 能够被无参数调用// ...
 }; 
```

然而，如果参数被提供了默认值，那么带参数的默认构造函数也是可能的：

```cpp
 class Fred {
 public:
   Fred(int i=3, int j=5);   // 默认构造函数: 能够被无参数调用// ...
 }; 
```

## 10.5 当建立一个 `Fred` 对象数组时，哪个构造函数将被调用？

`Fred` 的默认构造函数（以下讨论除外）。

你无法告诉编译器调用不同的构造函数（以下讨论除外）。如果你的`Fred`类没有默认构造函数，那么试图创建一个`Fred`对象数组将会导致编译时出错。

```cpp
 class Fred {
 public:
   Fred(int i, int j);
   // ... 假设 Fred 类没有默认构造函数 ...
 };

 int main()
 {
   Fred a[10];               // 错误：Fred 类没有默认构造函数
   Fred* p = new Fred[10];   // 错误：Fred 类没有默认构造函数
 } 
```

然而，如果你正在创建一个标准的`std::vector<Fred>`，而不是 `Fred`对象数组（既然数组是有害的，那么你可能应该这么做），则在 `Fred` 类中不需要默认构造函数。因为你能够给`std::vector`一个用来初始化元素的`Fred` 对象：

```cpp
 #include <vector>

 int main()
 {
   std::vector<Fred> a(10, Fred(5,7));
   // 在 std::vector 中的 10 个 Fred 对象将使用 Fred(5,7) 来初始化// ...
 } 
```

虽然应该使用`std::vector`而不是数组，但有有应该使用数组的时候，那样的话，有“数组的显式初始化”语法。它看上去是这样的：

```cpp
 class Fred {
 public:
   Fred(int i, int j);
   // ... 假设 Fred 类没有默认构造函数...
 };

 int main()
 {
   Fred a[10] = {
     Fred(5,7), Fred(5,7), Fred(5,7), Fred(5,7), Fred(5,7),
     Fred(5,7), Fred(5,7), Fred(5,7), Fred(5,7), Fred(5,7)
   };

   // 10 个 Fred 对象将使用 Fred(5,7) 来初始化.
   // ...
 } 
```

当然你不必每个项都做`Fred(5,7)`—你可以放任何你想要的数字，甚至是参数或其他变量。重点是，这种语法是（a）可行的，但（b）不如`std::vector`语法漂亮。记住这个：数组是有害的—除非由于编译原因而使用数组，否则应该用`std::vector` 取代。

## 10.6 构造函数应该用“初始化列表”还是“赋值”？

初始化列表。事实上，构造函数应该在初始化列表中初始化*所有*成员对象。

例如，构造函数用初始化列表`Fred::Fred() : x_(`*whatever*`) { }`来初始化成员对象 `x_`。这样做最普通的好处是提高性能。如，*whatever*表达式和成员变量 `x_` 相同，*whatever*表达式的结果直接由内部的`x_`来构造——编译器不会产生对象的两个拷贝。即使类型不同，使用初始化列表时编译器通常也能够做得比使用赋值更好。

建立构造函数的另一种（错误的）方法是通过赋值，如：`Fred::Fred() { x_ =`*whatever*`; }`。在这种情况下，*whatever*表达式导致一个分离的，临时的对象被建立，并且该临时对象被传递给`x_`对象的赋值操作。然后该临时对象会在 ；处被析构。这样是效率低下的。

这好像还不是太坏，但这里还有一个在构造函数中使用赋值的效率低下之源：成员对象会被以默认构造函数完整的构造，例如，可能分配一些缺省数量的内存或打开一些缺省的文件。但如果 *whatever*表达式和／或赋值操作导致对象关闭那个文件和／或释放那块内存，这些工作是做无用功（举例来说，如默认构造函数没有分配一个足够大的内存池或它打开了错误的文件）。

结论：其他条件相等的情况下，使用初始化列表的代码会快于使用赋值的代码。

注意：如果`x_`的类型是诸如`int`或者`char*` 或者`float`之类的内建类型，那么性能是没有区别的。但即使在这些情况下，我个人的偏好是为了对称，仍然使用初始化列表而不是赋值来设置这些数据成员。

## 10.7 可以在构造函数中使用 `this` 指针吗？

某些人认为不应该在构造函数中使用`this`指针，因为这时`this`对象还没有完全形成。然后，只要你小心，是可以在构造函数（在函数体甚至在初始化列表中）使用`this`的。

以下是始终可行的：构造函数的函数体（或构造函数所调用的函数）能可靠地访问基类中声明的数据成员和／或构造函数所属类声明的数据成员。这是因为所有这些数据成员被保证在构造函数函数体开始执行时已经被完整的建立。

以下是始终不可行的：构造函数的函数体（或构造函数所调用的函数）不能向下调用被派生类 重定义的虚函数。如果你的目的是得到派生类重定义的函数，那么你将无功而返。注意，无论你如何调用虚成员函数：显式使用`this`指针（如，`this->method()`），隐式的使用`this`指针（如，`method()`），或甚至在`this`对象上调用其他函数来调用该虚成员函数，你都不会得到派生类的重写函数。这是底线：即使调用者正在构建一个派生类的对象，在基类的构造函数执行期间，对象还不是一个派生类的对象。

以下是有时可行的：如果传递 `this` 对象的任何一个数据成员给另一个数据成员的初始化程序，你必须确保该数据成员已经被初始化。好消息是你能使用一些不依赖于你所使用的编译器的显著的语言规则，来确定那个数据成员是否已经（或者还没有）被初始化。坏消息是你必须知道这些语言规则（例如，基类子对象首先被初始化（如果有多重和／或虚继承，则查询这个次序！），然后类中定义的数据成员根据在类中声明的次序被初始化）。如果你不知道这些规则，则不要从`this`对象传递任何数据成员（不论是否显式的使用了`this`关键字）给任何其他数据成员的初始化程序！如果你知道这些规则，则需要小心。

## 10.8 什么是“命名的构造函数法（Named Constructor Idiom）”？

为你的类的用户提供的一种更直觉的和／或更安全的构造操作技巧。

问题在于构造函数总是有和类相同的名字。因此，区分类的不同的构造函数是通过参数列表。但如果有许多构造函数，它们之间的区别有时就会很敏感并且有错误倾向。

使用命名的构造函数法（Named Constructor Idiom），在`private:`节和`protected:`节中声明所有类的构造函数，并提供返回一个对象的`public` `static` 方法。这些方法由此称为“命名的构造函数（Named Constructors）”。一般，每种不同的构造对象的方法都有一个这样的静态方法。

例如，假设我们正在建立一个描绘 X-Y 平面的`Point`类。通常有两种方法指定一个二维空间坐标：矩形坐标(X+Y)，极坐标(Radius+Angle)（半径＋角度）。（不必担心已经忘了这些；重点不在于坐标系统的析解；重点在于有几种方法来创建一个`Point`对象。）不幸的是，这两种坐标系统的参数是相同的：两个 `float`。这将在重载构造函数中导致一个“重载不明确”的错误：

```cpp
 class Point {
 public:
   Point(float x, float y);     // 矩形坐标 _
   Point(float r, float a);     // 极坐标 (半径和角度)
   // 错误：重载不明确：Point::Point(float,float)
 };

 int main()
 {
   Point p = Point(5.7, 1.2);   // 不明确：哪个坐标系统？
 } 
```

解决这个不明确错误的一种方法是使用命名的构造函数法（Named Constructor Idiom）：

```cpp
 #include <cmath>               // To get sin() and cos()

 class Point {
 public:
   static Point rectangular(float x, float y);      // 矩形坐标 _
   static Point polar(float radius, float angle);   // 极坐标
   // 这些 static 方法称为“命名的构造函数（named constructors）”
   // ...
 private:
   Point(float x, float y);     // 矩形坐标
   float x_, y_;
 };

 inline Point::Point(float x, float y)
 : x_(x), y_(y) { }

 inline Point Point::rectangular(float x, float y)
 { return Point(x, y); }

 inline Point Point::polar(float radius, float angle)
 { return Point(radius*cos(angle), radius*sin(angle)); } 
```

现在，`Point`的用户有了一个清晰的和明确的语法在任何一个坐标系统中创建`Point`对象：

```cpp
 int main()
 {
   Point p1 = Point::rectangular(5.7, 1.2);   // 显然是矩形坐标
   Point p2 = Point::polar(5.7, 1.2);         // 显然是极坐标
 } 
```

如果期望`Point`有派生类，则确保你的构造函数在`protected:`节中。

命名的构造函数法也能用于总是通过`new`来创建对象。

## 10.9 为何不能在构造函数的初始化列表中初始化静态成员数据？

因为必须显式定义类的静态数据成员。

```cpp
//Fred.h:

 class Fred {
 public:
   Fred();
   // ...
 private:
   int i_;
   static int j_;
 };

//Fred.cpp (或 Fred.C 或其他)：

 Fred::Fred()
   : i_(10)  // 正确：能够（而且应该）这样初始化成员数据
   , j_(42)  // 错误：不能象这样初始化静态成员数据
 {
   // ...
 }

 // 必须这样定义静态数据成员：
 int Fred::j_ = 42; 
```

## 10.10 为何有静态数据成员的类得到了链接错误？

因为静态数据成员必须被显式定义在一个编辑单元中。如果不这样做，你就可能得到`"undefined external"`链接错误。例如：

```cpp
 // Fred.h

 class Fred {
 public:
   // ...
 private:
   static int j_;   // 声明静态数据成员：Fred::j_
   // ...
 }; 
```

链接器会向你抱怨（`"Fred::j_ is not defined"`），除非你在一个源文件中定义（而不仅仅是声明）`Fred::j_`：

```cpp
 // Fred.cpp

 #include "Fred.h"

 int Fred::j_ = some_expression_evaluating_to_an_int;

 // Alternatively, if you wish to use the implicit 0 value for static ints:
 // int Fred::j_; 
```

通常定义`Fred`类的静态数据成员的地方是`Fred.cpp`文件（或者`Fred.C`或者你使用的其他扩展名）。

## 10.11 什么是“`static` initialization order fiasco”？

你的项目的微妙杀手。

*`static` initialization order fiasco*是对 C++的一个非常微妙的并且常见的误解。不幸的是，错误发生在`main()`开始之前，很难检测到。

简而言之，假设你有存在于不同的源文件`x.cpp` 和`y.cpp`的两个静态对象`x` 和 `y`。再假定`y`对象的构造函数会调用`x`对象的某些方法。

就是这些。就这么简单。

结局是你完蛋不完蛋的机会是 50%-50%。如果碰巧`x.cpp`的编辑单元先被初始化，这很好。但如果`y.cpp`的编辑单元先被初始化，然后`y`的构造函数比`x`的构造函数先运行。也就是说，`y`的构造函数会调用`x`对象的方法，而`x`对象还没有被构造。

我听说有些人受雇于麦当劳，享受他们的切碎肉的新工作去了。

如果你觉得不用工作，在卧室的一角玩俄罗斯方块是令人兴奋的，你可以到此为止。相反，如果你想通过用一种系统的方法防止灾难，来提高自己 继续工作而存活的机会，你可能想阅读下一个 FAQ。

注意：static initialization order fiasco 不作用于内建的／固有的类型，象`int` 或 `char*`。例如，如果创建一个`static` `float`对象，不会有静态初始化次序的问题。静态初始化次序真正会崩溃的时机只有在你的`static`或全局对象有构造函数时。

## 10.12 如何防止“`static` initialization order fiasco”？

使用“首次使用时构造（construct on first use）”法，意思就是简单地将静态对象包裹于函数内部。

例如，假设你有两个类，`Fred` 和 `Barney`。有一个称为`x`的全局`Fred`对象，和一个称为`y`的全局`Barney`对象。`Barney`的构造函数调用了`x`对象的`goBowling()`方法。 `x.cpp`文件定义了`x`对象：

```cpp
 // File x.cpp
 #include "Fred.hpp"
 Fred x; 
```

`y.cpp`文件定义了`y`对象：

```cpp
 // File y.cpp
 #include "Barney.hpp"
 Barney y; 
```

`Barney`构造函数的全部看起来可能是象这样的：

```cpp
 // File Barney.cpp
 #include "Barney.hpp"

 Barney::Barney()
 {
   // ...
   x.goBowling();
   // ...
 } 
```

正如以上所描述的，由于它们位于不同的源文件，那么 `y` 在 `x` 之前构造而发生灾难的机率是 50%。

这个问题有许多解决方案，但一个非常简便的方案就是用一个返回`Fred`对象引用的全局函数`x()`，来取代全局的`Fred`对象 `x`。

```cpp
 // File x.cpp

 #include "Fred.hpp"

 Fred& x()
 {
   static Fred* ans = new Fred();
   return *ans;
 } 
```

由于静态局部对象只在控制流第一次越过它们的声明时构造，因此以上的`new Fred()`语句只会执行一次：`x()`被第一次调用时。每个后续的调用将返回同一个`Fred`对象（`ans`指向的那个）。然后你所要做的就是将 `x` 改成 `x()`：

```cpp
 // File Barney.cpp
 #include "Barney.hpp"

 Barney::Barney()
 {
   // ...
   x().goBowling();
   // ...
 } 
```

由于该全局的`Fred`对象在首次使用时被构造，因此被称为*首次使用时构造法（Construct On First Use Idiom）*

这种方法的不利方面是`Fred`对象不会被析构。*C++ FAQ Book*有另一种技巧消除这个影响（但面临了“static *de*-initialization order fiasco”的代价）。

注意：对于内建／固有类型，象`int` 或 `char*`，不必这样做。例如，如果创建一个静态的或全局的`float`对象，不需要将它包裹于函数之中。静态初始化次序真正会崩溃的时机只有在你的`static`或全局对象有构造函数时。

## 10.13 对于静态数据成员，如何防止“`static` initialization order fiasco”？

使用与描述过的相同的技巧，但这次使用静态成员函数而不是全局函数而已。

假设类 `X` 有一个`static` `Fred`对象：

```cpp
 // File X.hpp

 class X {
 public:
   // ...

 private:
   static Fred x_;
 }; 
```

自然的，该静态成员被分开初始化：

```cpp
 // File X.cpp

 #include "X.hpp"

 Fred X::x_; 
```

自然的，`Fred`对象会在 `X` 的一个或多个方法中被使用：

```cpp
 void X::someMethod()
 {
   x_.goBowling();
 } 
```

但现在“灾难情景”就是如果某人在某处不知何故在`Fred`对象被构造前调用这个方法。例如，如果某人在静态初始化期间创建一个静态的 `X` 对象并调用它的`someMethod()`方法，然后你就受制于编译器是在`someMethod()`被调用之前或之后构造 `X::x_`。（ANSI/ISO C++委员会正在设法解决这个问题，但诸多的编译器对处理这些更改一般还没有完成；关注此处将来的更新。）

无论何种结果，将`X::x_` 静态数据成员改为静态成员函数总是最简便和安全的：

```cpp
 // File X.hpp

 class X {
 public:
   // ...

 private:
   static Fred& x();
 }; 
```

自然的，该静态成员被分开初始化：

```cpp
 // File X.cpp

 #include "X.hpp"

 Fred& X::x()
 {
   static Fred* ans = new Fred();
   return *ans;
 } 
```

然后，简单地将 `x_` 改为 `x()`：

```cpp
 void X::someMethod()
 {
   x().goBowling();
 } 
```

如果你对性能敏感并且关心每次调用`X::someMethod()`的额外的函数调用的开销，你可以设置一个`static` `Fred&`来取代。正如你所记得的，静态局部对象仅被初始化一次（控制流程首次越过它们的声明处时），因此，将只调用`X::x()`一次：`X::someMethod()`首次被调用时：

```cpp
 void X::someMethod()
 {
   static Fred& x = X::x();
   x.goBowling();
 } 
```

注意：对于内建／固有类型，象`int` 或 `char*`，不必这样做。例如，如果创建一个静态的或全局的`float`对象，不需要将它包裹于函数之中。静态初始化次序真正会崩溃的时机只有在你的`static`或全局对象有构造函数时。

## 10.14 如何处理构造函数的失败？

抛出一个异常。详见 [17.2]。

## 10.15 什么是“命名参数法（Named Parameter Idiom）”？

发掘方法链的非常有用的方法。

命名参数法（Named Parameter Idiom）解决的最基本问题是 C++仅支持位置相关的参数。例如，函数调用者不能说“这个值给形参`xyz`，另一个值给形参`pqr`”。在 C++（和 C 和 Java）中只能说“这是第一个参数，这是第二个参数等”。Ada 语言提出并实现的命名参数，对于带有大量的可缺省参数的函数尤其有用。

多年来，人们构造了很多方案来弥补 C 和 C++缺乏的命名参数。其中包括将参数值隐藏于一个字符串参数，然后在运行时解析这个字符串。例如，这就是`fopen()`的第二个参数的做法。另一种方案是将所有的布尔参数联合成一个位映射，然后调用者将这堆转换成位的常量共同产生一个实际的参数。例如，这就是`open()`的第二个参数的做法。这些方法可以工作，但下面的技术产生的调用者的代码更明显，更容易写，更容易读，而且一般来说更雅致。

这个想法，称为命名参数法（Named Parameter Idiom），它是将函数的参数变为以新的方式创建的类的方法，这些方法通过引用返回`*this`。然后你只要将主要的函数改名为那个类中的无参数的“随意”方法。

举一个例子来解释上面那段。

这个例子实现“打开一个文件”的概念。该概念逻辑上需要一个文件名的参数，和一些允许选择的参数，文件是否被只读或可读写或只写的方式打开；如果文件不存在，是否创建它；是从末尾写（添加"append"）还是从起始处写（覆盖"overwrite"）；如果文件被创建，指定块大小； I/O 是否有缓冲区，缓冲区大小；文件是被共享还是独占访问；以及其他可能的选项。如果我们用常规的位置相关的参数的函数实现这个概念，那么调用者的代码会非常难读：有 8 个可选的参数，并且调用者很可能犯错误。因此我们使用命名参数用法来取代。

在实现它之前，假如你想接受函数的所有默认参数，看一下调用者的代码是什么样子：

```cpp
File f = OpenFile("foo.txt"); 
```

那是简单的情况。现在看一下如果你想改变一大堆的参数：

```cpp
 File f = OpenFile("foo.txt").
            readonly().
            createIfNotExist().
            appendWhenWriting().
            blockSize(1024).
            unbuffered().
            exclusiveAccess(); 
```

注意这些“参数”，被公平的以随机的顺序（位置无关的）调用并且都有名字。因此，程序员不必记住参数的顺序，而且这些名字是（正如所希望的）意义明显的。

以下是如何实现：首先创建一个新的类(`OpenFile`)，该类包含了所有的参数值作为 `private:` 数据成员。然后所有的方法（`readonly()`, `blockSize(unsigned)`, 等）返回`*this`（也就是返回一个`OpenFile`对象的引用，以允许方法被链状调用）。最后完成一个带有必要参数（在这里，就是文件名）的常规的，参数位置相关的`OpenFile`的构造函数。

```cpp
 class File;

 class OpenFile {
 public:
   OpenFile(const string& filename);
   // 为每个数据成员设置默认值
   OpenFile& readonly();  // 将 readonly_ 变为 true
   OpenFile& createIfNotExist();
   OpenFile& blockSize(unsigned nbytes);
   // ...
 private:
   friend File;
   bool readonly_;       // 默认为 false [举例]
   // ...
   unsigned blockSize_;  // 默认为 4096 [举例]
   // ...
 }; 
```

要做的另外一件事就是使得 `File`的构造函数带一个`OpenFile`对象：

```cpp
 class File {
 public:
   File(const OpenFile& params);
   // vacuums the actual params out of the OpenFile object
   // ...
 }; 
```

注意`OpenFile` 将 `File` 声明为友元。