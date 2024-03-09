# [33] 成员函数指针

## FAQs in section [33]:

*   [33.1] “成员函数指针”类型不同于“函数指针”吗？
*   [33.2] 如何将一个成员函数指针传递到信号处理函数，X 事件回调函数，系统调用来启动一个线程/任务等？
*   [33.3] 为什么我总是收到编译错误（类型不匹配）当我尝试用一个成员函数作为中断服务例程？ when I try to use a member function as an interrupt service routine?")
*   [33.4] 为什么取 C++函数的地址我会遇到问题？
*   [33.5] 使用成员函数指针调用函数时我如何才能避免语法错误？
*   [33.6] 如何创建和使用一个成员函数指针数组？
*   [33.7] 可以转换成员函数指针为 void *吗？
*   [33.8] 可以转换函数指针为 void *吗？
*   [33.9] 我需要类似函数指针的功能，但需要更多的灵活性和/或线程安全，是否有其他方法？
*   [33.10] 什么是 functionoid，为什么我要使用它？
*   [33.11] 可以让 functionoids 快于正常的函数调用吗？
*   [33.12] functionoid 和仿函数(functor)有什么区别？

## 33.1 “成员函数指针”类型不同于“函数指针”吗？

对。

考虑下面的函数：

`int f(char a, float b);`

函数的类型不同取决于它是否是一个普通函数或某些类非静态成员函数：

*   它的类型是“int(*)(char, float）”，如果它是一个普通的函数
*   它的类型是“`int (Fred::*)(char,float)`”，如果它是类 Fred 的非静态成员函数

注意：如果它是类 Fred 的静态成员函数 ，它的类型和普通函数是相同的：“int(*)(char, float)”。

## 33.2 如何将一个成员函数指针传递到信号处理函数，X 事件回调函数，系统调用来启动一个线程/任务等？

不要。

由于成员函数是没有意义，如果没有一个对象来触发的话，所以你不能这样直接调用（如 X 窗口系统代码用 C++重写的话，它很可能会传递对象的引用，不仅仅是函数指针，当然了对象将包含所需的函数，甚至更多）。

作为对现有软件的补丁，使用顶级(top-level)函数（非成员函数）作为包装器，包装器接受通过一些其他技术实例化的对象为参数。取决于你要调用的函数，这个“其他技术”有可能很琐碎或者不需要你做太多的工作。对于启动一个新线程的系统调用，例如，可能要求你传递一个 void *类型的函数指针，这种情况下你可以传递 void*类型的对象指针。许多实时操作系统要启动一个新任务时候于此类似。最坏的情况，你可以将对象指针存储在全局变量中，对于 Unix 的信号处理程序来说可能需要这样处理（但一般来说不希望使用全局变量）。在任何情况下，顶级（top-level）函数将负责调用相应的对象的成员函数。

下面是一个最坏的例子（要使用全局变量）。中断发生时候假设你要调用 `Fred::memberFn()` ：

```cpp
 class Fred {
 public:
   void memberFn();
   static void staticMemberFn();  // A static member function can usually handle it
   ...
 };

 // Wrapper function uses a global to remember the object:
 Fred* object_which_will_handle_signal;

 void Fred_memberFn_wrapper()
 {
   object_which_will_handle_signal->memberFn();
 }

 int main()
 {
   /* signal(SIGINT, Fred::memberFn);& */ // Can NOT do this
   signal(SIGINT, Fred_memberFn_wrapper);  // OK
   signal(SIGINT, Fred::staticMemberFn);   // OK usually; see below
   ...
 } 
```

注： 静态成员函数并不需要一个实际对象来触发，因此静态成员函数指针和普通函数指针“通常”是兼容的。然而，尽管它可能在大多数编译器上工作，但是严格来说它必须是带有`extern"C"`修饰的非成员函数。因为“C 链接器”不仅不知道“名字校正(mangle)”等，而且还不知道不同的调用约定，而 C 和 C++的调用约定可能不同。

## 33.3 为什么我总是收到编译错误（类型不匹配）当我尝试用一个成员函数作为中断服务例程？

这是前两个问题的特殊情况，因此，阅读前两个 FAQ 问题的答案。

非静态成员函数有一个隐藏的参数，对应于`this`指针，该`this`指针指向的对象的实例。系统的中断硬件/固件不能提供有关`this`指针参数。你必须使用“普通”函数（非类成员）或静态成员函数作为中断服务例程。

一个可行的办法是使用一个静态成员函数作为中断服务程序，并让该静态函数去负责查找在中断时候应该调用的实例/成员函数。实际效果是，中断的时候成员函数被调用，但是出于技术原因你需要调用一个中间函数。

## 33.4 为什么取 C++函数的地址我会遇到问题？

简单答案：如果你试图把它存储到（或者传递到）函数指针，这就会产生问题-这是前面 FAQ 问题的必然结果。

详细回答：在 C++成员函数有一个隐含的参数，它指向对象（内部成员函数的 this 指针）。普通 C 函数和成员函数有不同的函数调用约定，所以他们的指针类型（成员函数指针与普通函数指针）是不同的，不相容的。C++中引入了新的指针类型，称为成员指针，它只能供一个实例对象调用。

注意： 不要试图强制转换成员函数指针为普通函数指针，结果是不确定的，可能是灾难性的。例如，一个成员函数指针不需要包含确切函数的机器地址。正如在最后一个例子，如果你有一个普通 C 函数的指针，使用一个顶层（非成员）函数或静态 （类）成员函数。

## 33.5 使用成员函数指针调用函数时我如何才能避免语法错误？

同时使用 `typedef` *和* `#define`宏。

**步骤 1：**创建 typedef：

```cpp
 class Fred {
 public:
   int f(char x, float y);
   int g(char x, float y);
   int h(char x, float y);
   int i(char x, float y);
   ...
 };

 // FredMemFn points to a member of Fred that takes (char,float)
 typedef  int (Fred::*FredMemFn)(char x, float y); 
```

**第 2 步：**创建一个`#define`宏：

```cpp
#define CALL_MEMBER_FN(object,ptrToMember)  ((object).*(ptrToMember)) 
```

（ 通常我不喜欢`#define`宏，但在成员函数指针中你应该使用他们，因为他们可以提高可读性和代码的易用性。）

以下是如何使用这些功能：

```cpp
 void userCode(Fred& fred, FredMemFn memFn)
 {
   int ans = CALL_MEMBER_FN(fred,memFn)('x', 3.14);
   // Would normally be: int ans = (fred.*memFn)('x', 3.14);

   ...
 } 
```

我强烈建议使用这些功能。在实践中，成员函数调用更比刚才复杂，可读性和代码的易写性的区别很大。comp.lang.C++不得不忍受成千上万的程序员的询问语法错误的帖子，。几乎所有这些错误都会消失如果他们使用了这些功能。

注：#define 宏有 4 中罪恶： 罪恶#1 , 罪恶#2 , 罪恶#3 和罪恶#4 。但有时他们仍然有用。只要别忘了使用后洗清“罪恶”的双手。

## 33.6 如何创建和使用一个成员函数指针数组？

*同时*使用`typedef` *和* `#define`宏的前面描述，你就完成 90％。

**步骤 1：**创建 typedef：

```cpp
 class Fred {
 public:
   int f(char x, float y);
   int g(char x, float y);
   int h(char x, float y);
   int i(char x, float y);
   ...
 };

 // FredMemFn points to a member of Fred that takes (char,float)&
 typedef  int (Fred::*FredMemFn)(char x, float y); 
```

**第 2 步：**创建一个`#define`宏：

```cpp
#define CALL_MEMBER_FN(object,ptrToMember)  ((object).*(ptrToMember)) 
```

现在简单地创建成员函数的指针数组：

```cpp
FredMemFn a[] = { &Fred::f, &Fred::g, &Fred::h, &Fred::i }; 
```

也可以简单地调用成员函数的指针：

```cpp
 void userCode(Fred& fred, int memFnNum)
 {
   // Assume  memFnNum  is between 0 and 3 inclusive:
   CALL_MEMBER_FN(fred, a[memFnNum]) ('x', 3.14);
 } 
```

注：#define 宏有 4 中罪恶： 罪恶#1 , 罪恶#2 , 罪恶#3 和罪恶#4 。但有时他们仍然有用。虽然感到耻辱和负罪感，如果像宏这样的结构如果能够改进你的软件，那么就使用它。

## 33.7 可以转换成员函数指针为`void *`吗？

否！

```cpp
 class Fred {
 public:
   int f(char x, float y);
   int g(char x, float y);
   int h(char x, float y);
   int i(char x, float y);
   ...
 };

 // FredMemFn points to a member of __Fred__ that takes (char,float)
 typedef  int (Fred::*FredMemFn)(char x, float y);

 #define CALL_MEMBER_FN(object,ptrToMember)  ((object).*(ptrToMember))

 int callit(Fred& o, FredMemFn p, char x, float y)
 {
   return CALL_MEMBER_FN(o,p)(x, y);
 }

 int main()
 {
   FredMemFn p = &Fred::f;
   void* p2 = (void*)p;                  // ← illegal!!
   Fred o;
   callit(o, p, 'x', 3.14f);             // okay
   callit(o, FredMemFn(p2), 'x', 3.14f); // might fail!!
   ...
 } 
```

*请*不要给我发电子邮件， 如果碰巧上述情况*在*您的特定的操作系统和特定的编译器的特定版本中没有问题。我不在乎这些。这中做法是非法的，句号！

## 33.8 可以转换函数指针为`void *`吗？

否！

```cpp
 int f(char x, float y);
 int g(char x, float y);

 typedef int(*FunctPtr)(char,float);

 int callit(FunctPtr p, char x, float y)
 {
   return p(x, y);
 }

 int main()
 {
   FunctPtr p = f;
   void* p2 = (void*)p;              // ← illegal!!
   callit(p, 'x', 3.14f);            // okay
   callit(FunctPtr(p2), 'x', 3.14f); // might fail!!
   ...
 } 
```

*请*不要给我发电子邮件， 如果碰巧上述情况*在*您的特定的操作系统和特定的编译器的特定版本中没有问题。我不在乎这些。这中做法是非法的，句号！

## 33.9 我需要类似函数指针的功能，但需要更多的灵活性和/或线程安全，是否有其他方法？

使用 functionoid。

## 33.10 什么是 functionoid，为什么我要使用它？

Functionoids 是基于 steroids 的函数。严格来说比函数功能更强大，而其额外的功能解决了使用函数指针时所面临的一些（不是全部）的挑战。

让我们举一个例子说明传统函数指针的使用，然后我们将其转化为使用 functionoids 的例子。传统的函数指针的思想是定义一堆兼容的函数：The traditional function-pointer idea is to have a bunch of compatible functions:

```cpp
 int funct1(...params...) { ...code... }
 int funct2(...params...) { ...code... }
 int funct3(...params...) { ...code... } 
```

然后，你通过函数指针来调用：

```cpp
 typedef int(*FunctPtr)(...params...);

 void myCode(FunctPtr f)
 {
   ...
   f(...args-go-here...);
   ...
 } 
```

有时，人们创建函数指针数组：

```cpp
 FunctPtr array[10];
 array[0] = funct1;
 array[1] = funct1;
 array[2] = funct3;
 array[3] = funct2;
 ... 
```

在这种情况下，通过访问该数组来调用函数：

```cpp
 arrayi; 
```

使用 functionoids，首先创建了有一个纯虚函数的的基类：

```cpp
 class Funct {
 public:
   virtual int doit(int x) = 0;
   virtual ~Funct() = 0;
 };

 inline Funct::~Funct() { }  // defined even though it's pure virtual; it's faster this way; trust me 
```

然后，你可以创建三个派生类来替代 3 个函数：

```cpp
 class Funct1 : public Funct {
 public:
   virtual int doit(int x) { ...code from funct1... }
 };

 class Funct2 : public Funct {
 public:
   virtual int doit(int x) { ...code from funct2... }
 };

 class Funct3 : public Funct {
 public:
   virtual int doit(int x) { ...code from funct3... }
 }; 
```

然后，不是传递一个函数指针而是传递一个`Funct *`。我创建 `typedef`称为`FunctPtr`,只是为了代码看起来类似以前的方法：

```cpp
 typedef Funct* FunctPtr;

 void myCode(FunctPtr f)
 {
   ...
   f->doit(...args-go-here...);
   ...
 } 
```

你可以用同样的方式来创建数组：

```cpp
 FunctPtr array[10];
 array[0] = new Funct1(_...ctor-args..._);
 array[1] = new Funct1(_...ctor-args..._);
 array[2] = new Funct3(_...ctor-args..._);
 array[3] = new Funct2(_...ctor-args..._);
 ... 
```

首先这给出了一个 functionoids 比函数指针功能更强大的事实，即 functionoid 可以传递参数可以传递到构造函数（如上图所示的 ctor - argS）而函数指针版本则没有。可以想象 functionoid 对象为一个 freeze-dried 函数调用（重点在调用这个词）。 不像一个函数指针，functionoid 是（概念上）一个指向了部分被调用函数的指针。想象目前的技术，让你通过传递一部分，但是不是全部参数给一个函数，然后让你 freeze-dry（部分完成）函数调用。就好像这种技术可让你使用某种神奇的指针，指针指向那个 freeze-dry 部分完成的函数调用。然后你通过使该指针传递其余参数，系统神奇地结合你原来传递的参数（即是 freeze-dried 的参数），结合函数先前计算的局部变量（被 freeze-dried 之前），加上所有新传递的`args`，从函数上次被 freeze-dried 的地方开始继续执行函数 。这听起来像是科幻小说，但它正是概念上 functionoids 可以办到的。 另外 ，它可以让你反复地使用各种不同的“剩余的参数”来“完成”freeze-dried 函数调用，你要你喜欢，多少次都可以。另外 ，允许（不是必须）你改变 freeze-dried 的状态当调用的时候，这意味着 functionoids 可以记得从一个调用到下一个的信息。

好吧，让我们回到现实，我会举一两个例子来解释上面叙述的意义。

假设原有函数（在老式的函数指针样式下）采取略有不同的参数。

```cpp
 int funct1(int x, float y)
 { ...code... }

 int funct2(int x, const std::string& y, int z)
 { ...code... }

 int funct3(int x, const std::vector<double>& y)
 { ...code... } 
```

当参数不同的时候，老式的函数指针的方法是很难凑效，因为函数调用方不知道需要传递哪些参数（呼叫者仅仅有一个函数指针，而不是函数的名称或，当参数不同的时候需要的参数个数和参数类型）（不要给我发送电子邮件，我承认你可以做到这一点，但你必须花费很多精力并且收拾残局。无论如何不要给我写邮件 –请使用 functionoids 代替）。

使用 functionoids 有时情况会好很多。由于 functionoid 可以看作是一个 freee-dried 函数调用 ，只需象上面的`y`和/或者`z`一样，可以传递它们到相应的构造函数。你还可以通过共同`args`参数（在上例中的 `int`类型的`x`参数）到`ctor`，但你不必-这样做。你也可以直接传递他们到的纯虚函数`doIt()`。 下面假设你想传递 X 到`doIt()`和传递`y`和/或 `z`到构造函数：

```cpp
 class Funct {
 public:
   virtual int doit(int x) = 0;
 }; 
```

然后，你可以创建三个派生类，而不是三个函数：

```cpp
 class Funct1 : public Funct {
 public:
   Funct1(float y) : y_(y) { }
   virtual int doit(int x) { ...code from funct1... }
 private:
   float y_;
 };

 class Funct2 : public Funct {
 public:
   Funct2(const std::string& y, int z) : y_(y), z_(z) { }
   virtual int doit(int x) { _...code from funct2..._ }
 private:
   std::string y_;
   int z_;
 };

 class Funct3 : public Funct {
 public:
   Funct3(const std::vector<double>& y) : y_(y) { }
   virtual int doit(int x) { _...code from funct3..._ }
 private:
   std::vector<double> y_;
 }; 
```

当你创建的 functionoids 数组的时候，构造函数的参数被 freeze-dried 到 functionoid：

```cpp
 FunctPtr array[10];

 array[0] = new Funct1(3.14f);

 array[1] = new Funct1(2.18f);

 std::vector<double> bottlesOfBeerOnTheWall;
 bottlesOfBeerOnTheWall.push_back(100);
 bottlesOfBeerOnTheWall.push_back(99);
 ...
 bottlesOfBeerOnTheWall.push_back(1);
 array[2] = new Funct3(bottlesOfBeerOnTheWall);

 array[3] = new Funct2("my string", 42);

 ... 
```

因此，当用户在调用这些 functionoids 的`doIt()`的时候，他提供的“剩余”`args`，函数调用会把传递到构造函数与传递到`doIt()`的参数结合起来：

```cpp
array[i]->doit(12); 
```

正如我以前说的，functionoids 的优点之一是，你可以有多个实例，比方说在你的数组里面 Funct1，这些实例可以有不同的参数，被 freeze-dried 到构造函数。例如， 数组`[0]`和数组`[1]`的类型都是`Funct1`，但数组`[0] -> doIt（12）`的行为和数组`[1] –>doIt（12）`的行为是不一样的，因为这将取决于传递给调用`doIt(`)函数的 12 和传递给构造函数的 `args`。

如果我们把 functionoids 数组的例子变为一个本地的 functionoid，你将会看到 functionoids 的另一个优点。为了热身，让我们回到老式的函数指针的方法，想象你要传递一个比较函数到`sort()`或`binarySearch()`例程。`sort()`或`binarySearch()`例程被称作`childRoutine()`和比较函数指针类型被称为`FunctPtr`：

```cpp
 void childRoutine(FunctPtr f)
 {
   ...
   f(...args...);
   ...
 } 
```

然后，不同的调用方根据自己的判断传递不同的函数指针：

```cpp
 void myCaller()
 {
   ...
   childRoutine(funct1);
   ...
 }

 void yourCaller()
 {
   ...
   childRoutine(funct3);
   ...
 } 
```

我们可以很容易地转化为一个使用 functionoids 的例子：

```cpp
 void childRoutine(Funct& f)
 {
   ...
   f.doit(_...args..._);
   ...
 }

 void myCaller()
 {
   ...
   Funct1 funct(_...ctor-args..._);
   childRoutine(funct);
   ...
 }

 void yourCaller()
 {
   ...
   Funct3 funct(_...ctor-args..._);
   childRoutine(funct);
   ...
 } 
```

鉴于这样的例子，我们可以看到 functionoids 优于函数指针的两个好处。上面讲述了在“ctor args”的好处，再加上 functionoids 能够在一个线程安全的环境下保持调用之间的状态。与普通的函数指针相比，人们通常通过使用静态数据来保持状态，不过静态数据是在本质上不是线程安全的---所有线程共享静态数据。但是 functionoid 方法本质上是线程安全的，因为这些代码是与线程本地数据想关联的。实现是很琐碎的：改变老式的静态数据为一个 functionoid 对象实例； 并且该实现可以证明数据不仅是线程局部的，而且也可以安全的进行递归调用：每次调用`yourCaller()`将有自己独特的有自己独特的数据成员的`Funct3`对象实例。

请注意，我们已经得到了一些东西，但是不用付出任何代价。如果你想线程全局的数据，functionoids 可以实现：只需更改的实例数据成员为 functionoid 的静态成员，或者局部范围的静态数据。该实现和函数指针相比是伯仲之间。

functionoid 为你提供了第三种选择，而老式的函数指针方法却不行：允许 functionoid 的调用方决定他们是否希望线程局部或线程全局的数据。如果调用方希望线程全局的数据，他们需要负责的线程安全，至少他们可以有这个选择。这很容易：

```cpp
 void callerWithThreadLocalData()
 {
   ...
   Funct1 funct(...ctor-args...);
   childRoutine(funct);
   ...
 }

 void callerWithThreadGlobalData()
 {
   ...
   static Funct1 funct(...ctor-args...);  ← the static is the only difference
   childRoutine(funct);
   ...
 } 
```

Functionoids 不能解决遇到的每一个问题当需要编写柔性软件的时候，但严格来讲他们比函数指针功能更强大，至少需要评估一下。事实上，你可以很容易证明 functionoids 拥有函数指针的所有功能，因为可以想像，老式函数指针相当于一个全局的（！）functionoid 对象。既然你总是可以定义 functionoid 全局对象，你自然没有失去任何东西。证毕！

## 33.11 可以让 functionoids 快于正常的函数调用吗？

是。

如果你有一个非常小的 functionoid，并在实际应用中的相当常见，函数调用本身的成本可能会很高，与由 functionoid 完成工作的成本相比。在以前的 FAQ 中，functionoids 的实现使用了虚函数，这通常会花费一个函数调用成本。另一种方法使用的模板 。

下面的例子与以前的 FAQ 类似。我把调用`doIt()`修改为运算符`()()`来改善代码的可读性，也允许别人传递普通函数指针：

```cpp
 class Funct1 {
 public:
   Funct1(float y) : y_(y) { }
   int operator()(int x) { ...code from funct1... }
 private:
   float y_;
 };

 class Funct2 {
 public:
   Funct2(const std::string& y, int z) : y_(y), z_(z) { }
   int operator()(int x) { ...code from funct2... }
 private:
   std::string y_;
   int z_;
 };

 class Funct3 {
 public:
   Funct3(const std::vector<double>& y) : y_(y) { }
   int operator()(int x) { ...code from funct3... }
 private:
   std::vector<double> y_;
 }; 
```

这种做法，在以前的 FAQ 的区别是 fuctionoid 在编译时而不是在运行时被“绑定”。想象你把它作为一个参数传递：如果你在编译时已经知道你最终要传递的 functionoid，那么你可以使用以上技术，至少在典型的情况下](inline-functions.html#faq-9.3)你可以获得一个相对速度优势，就是编译器[内联代码到调用方。下面是一个例子：

```cpp
 template <typename FunctObj>
 void myCode(FunctObj f)
 {
   ...
   f(...args-go-here...);
   ...
 } 
```

编译器编译上面代码的时候，有可能内联展开的函数调用，即可能提高性能。

下面是一种调用方法：

```cpp
 void blah()
 {
   ...
   Funct2 x("functionoids are powerful", 42);
   myCode(x);
   ...
 } 
```

补充：正如在上文第一段所述，你也可以传递普通函数（尽管调用方调用时可能会招致一些花销）：

```cpp
 void myNormalFunction(int x);

 void blah()
 {
   ...
   myCode(myNormalFunction);
   ...
 } 
```

## 33.12 functionoid 和仿函数(functor)有什么区别？

functionoid 是一个对象，有一个主要方法。它基本上是 C 函数的面向对象扩展，人们会使用 functionoid 当函数有多个入口点（即不止一个“method”），和/或者需要以线程安全的方式（C 风格的解决办法是，增加一个本地的“静态”变量，但在多线程环境中不能保证线程安全）调用之间保持状态。

functor 是 functionoid 的特殊情况：这是一个其方法是“函数调用操作符”(`operator()()`)的 functionoid. 由于它重载函数调用操作符，代码可以使用和函数调用相同的语法来调用它的主体方法。例如，如果“`foo`”是一个 functor，要调用“`foo`”对象的“`operator()()`”可以使用“`foo()`”。在这样的好处在于模板，模板可以有一个可以作为函数使用的模板参数，这个参数可以是一个函数或仿函数对象。它有一个性能优势，就是仿函数对象的 “`operator()()`"方法可以被内联（如果你传递一个函数地址，那么它不能被内联）。

这是非常有用的，比如对于排序容器“比较”函数。在 C 中，比较函数总是通过指针传递（例如，参见 “`qsort()`"声明），但在 C++中参数可以是函数指针或者 functor 对象，其导致的结果就是 C++的排序容器在某些情况下，要比 C 语言中的更快（不慢）。

由于 Java 没有任何类似模板的功能，它必须使用动态绑定，动态绑定必然意味着函数调用。这通常不是什么大问题，但在 C++中，我们要让代码发挥最高性能，也就是说，C++中有一个“pay for it only if you use it”的理念，这意味着语言绝对不能随意施加任何开销到物理机器（当然是程序员有可能会，比如选择的使用如动态绑定等技术，施加一些开销，这是作为的灵活性或其他“特性”的交换，应该由设计师和程序员来决定他们是否想要这些结构带来的好处（和成本等）。