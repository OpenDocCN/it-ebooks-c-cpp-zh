# Library 2\. Conversion

# Conversion 库如何改进你的程序？

## Conversion 库如何改进你的程序？

*   可理解、可维护，以及一致的多态类型转换

*   静态向下转型使用比`static_cast`更 安全的结构

*   进行范围判断的数字转换确保正确的值逻辑以及更少的调试时间

*   正确且可重用的文字转换导致更少的编码时间

C++的多功能性是它获得成功的主要原因之一，但有时也是麻烦的来源，因 为语言各部分的复杂性。例如，数字转换规则以及类型提升规则都很复杂。其它转换虽然简单，但也很乏味；多少次我们需要写一个安全的函数[1]来进 行`string`s 和 `int`s, `double`s 和 `string`s 之间的转换？在你写的每个库和程序里，类型转换都可能是有问题的，这就是 Conversion 库可以帮助你的地方。它提供了防止危险转换及可复用的类型转换工具。

> [1] 避免使用 `sprintf` 及其相关函数。

Conversion 库由四个转换函数组成，分别提供了更好的类型安全性(`polymorphic_cast`), 更高效的类型安全防护(`polymorphic_downcast`), 范围检查的数字转换(`numeric_cast`), 以及文字转换(`lexical_cast`)。这些类 cast 函数共享 C++转型操作符的语义。与 C++的转型 操作符一样，这些函数具有一个重要的品质，类型安全性，这是它们与 C 风格转型的区别：它们明确无误地表达了程序员的意图[2]。我们所写的代码的重要性不仅在于它可以 正确执行。更重要的是代码可否清晰地表达我们的意图。这个库使得我们可以更容易地扩展我们的 C++词汇表。

> [2] 它们也可以被重载，以使得它们比 C++转型操作符更高级。

# polymorphic_cast

## polymorphic_cast

### 头文件: `"boost/cast.hpp"`

C++中的多态转型是用 `dynamic_cast`来实现的。`dynamic_cast`有一个有时会导致错误代码的特性，那就是它对于所使用的不同类型会有不同的行为。在用于一个引用类型时，如果转型失败，`dynamic_cast` 会抛出一个`std::bad_cast`异常。这样做的原因很简单，因为 C++里不允许有空的引用，所以要么转型成功，要么转型失败而你获得一个异常。当然，在 `dynamic_cast` 用于一个指针类型时，失败时将返回空指针。

`dynamic_cast`的这种对指针和引用类型的不同行为以前被认为是一个有用 的特性，因为它允许程序员表达他们的意图。典型地，如果转型失败不是一种逻辑错误，就使用指针转型，如果它确是一种错误，就使用引用转型。不幸的是，两种 方法之间的区别仅在于一个*号和一个&号，这种细微的差别是不自然的。如果想把指针转型失败作为错误处理，该怎么办？为了通过自动抛出异常来清楚 地表达这一点，也为了让代码更一致，Boost 提供了`polymorphic_cast`. 它在转型失败时总是抛出一个 `std::bad_cast` 异常。

在《The C++ Programming Language 3rd Edition》中，Stroustrup 对于指针类型的`dynamic_cast`说了以下一段话，事实是它可以返回空指针：

"偶尔可能会不小心忘了测试指针是否为空。如果这困扰了你，你可以写一转型函数在转型失败时抛出异常。"

`polymorphic_cast` 正是这样一个转型函数。

### 用法

`polymorphic_cast` 的用法类似于 `dynamic_cast`, 除了 (正是它的意图) 在转型失败时总是抛出一个 `std::bad_cast` 异常。`polymorphic_cast` 的另一个特点是它是一个函数，必要时可以被重载。作为对我们的 C++词汇表的一个自然扩展，它使得代码更清晰，类型转换也更少错误。要使用它，就要包含头文件`"boost/cast.hpp"`. 这个函数泛化了要转换的类型，并接受一个要进行转型的参数。

```cpp
template <class Target, class Source>
  polymorphic_cast(Source* p); 
```

要注意的是，`polymorphic_cast` 没有针对引用类型的版本。原因是那是`dynamic_cast`已经实现了的，没有必须让 `polymorphic_cast` 重复 C++语言中已有的功能。以下例子示范了与 `dynamic_cast`类似的语法。

### 向下转型和交叉转型

使用`dynamic_cast` 或 polymorphic_cast 可能有两种典型的情况：从基类向派生类的向下转型，或者交叉转型，即从一个基类到另一个基类。以下例子示范了使用`polymorphic_cast`的两类转型。这里有两个基类，`base1` 和 `base2`, 以及一个从两个基类公有派生而来的类 `derived` 。

```cpp
#include <iostream>
#include <string>
#include "boost/cast.hpp"

class base1 {
public:
  virtual void print() {
    std::cout << "base1::print()\n";
  }

  virtual ~base1() {}
};

class base2 {
public:

  void only_base2() {
    std::cout << "only_base2()\n";
  }

  virtual ~base2() {}
};

class derived : public base1, public base2 {
public:

  void print() {
    std::cout << "derived::print()\n";
  }

  void only_here() {
    std::cout << "derived::only_here()\n";
  }
  void only_base2() {
    std::cout << "Oops, here too!\n";
  }
};

int main() {
  base1* p1=new derived;

 p1->print();

  try {
    derived* pD=boost::polymorphic_cast<derived*>(p1);
    pD->only_here();
    pD->only_base2();

    base2* pB=boost::polymorphic_cast<base2*>(p1);
    pB->only_base2();

  }
  catch(std::bad_cast& e) {
    std::cout << e.what() << '\n';
  }

  delete p1;
} 
```

我们来看看 `polymorphic_cast` 是如何工作的，首先我们创建一个 `derived` 的实例，然后通过不同的基类指针以及派生类指针来操作它。对`p1`使用的第一个函数是`print`, 它是`base1` 和 `derived`的一个虚拟函数。我们还使用了向下转型，以便可以调用 `only_here`, 它仅在 `derived`中可用：

```cpp
derived* pD=boost::polymorphic_cast<derived*>(p1);
pD->only_here(); 
```

注意，如果 `polymorphic_cast` 失败了，将抛出一个 `std::bad_cast` 异常，因此这段代码被保护在一个 `try`/`catch` 块中。这种做法与使用引用类型的`dynamic_cast`正好是一样的。指针 `pD` 随后被用来调用函数 `only_base2`. 这个函数是`base2`中的非虚拟函数，但是在`derived`中也提供了，因此隐藏了`base2`中的版本。因而我们需要执行一个交叉转型来获得一个`base2`指针，才可以调用到 `base2::only_base2` 而不是 `derived::only_base2`.

```cpp
base2* pB=boost::polymorphic_cast<base2*>(p1);
pB->only_base2(); 
```

再一次，如果转型失败，将会抛出异常。这个例子示范了如果转型失败被认为是错误的话，使用`polymorphic_cast`可以多容易地进行错误处理。不需要测试空指针，也不会把错误传播到函数以外。正如我们即将看到的，`dynamic_cast` 有时会为这类代码增加不必要的复杂性；它还可能导致未定义行为。

### dynamic_cast 对 polymorphic_cast

为了看一下这两种转型方法之间的不同，[3] 我们把它们放在一起来比较一下复杂性。我们将重用前面例子中的类 `base1`, `base2`, 和 `derived`。你会发现在对指针类型使用`dynamic_cast`时，测试指针的有效性是一种既乏味又反复的事情，这使得测试很容易被紧张的程序员所忽略掉。

> [3] 技术上，`dynamic_cast` 是转型操作符，而 `polymorphic_cast` 是函数模板。

```cpp
void polymorphic_cast_example(base1* p) {
  derived* pD=boost::polymorphic_cast<derived*>(p);
  pD->print();

  base2* pB=boost::polymorphic_cast<base2*>(p);
  pB->only_base2();
}

void dynamic_cast_example(base1* p) {
  derived* pD=dynamic_cast<derived*>(p);
  if (!pD)
    throw std::bad_cast();
  pD->print();

  base2* pB=dynamic_cast<base2*>(p);
  if (!pB)
    throw std::bad_cast();

  pB->only_base2();

}

int main() {
  base1* p=new derived;
  try {
    polymorphic_cast_example(p);
    dynamic_cast_example(p);
  }
  catch(std::bad_cast& e) {
    std::cout << e.what() << '\n';
  }
  delete p;
} 
```

这两个函数，`polymorphic_cast_example` 和 `dynamic_cast_example`, 使用不同的方法完成相同的工作。差别在于无论何时对指针使用 `dynamic_cast` ，我们都要记住测试返回的指针是否为空。在我们的例子里，这种情况被认为是错误的，因此要抛出一个类型为 `bad_cast` 的异常。[4] 如果使用 `polymorphic_cast`, 错误的处理被局限在`std::bad_cast`的异常处理例程中， 这意味着我们不需要为测试转型的返回值而操心。在这个简单的例子中，不难记住要测试返回指针的有效性，但还是要比使用`polymorphic_cast`做更多的工作。如果是几百行的代码，再加上两三个程序员来维护这个函数的话，忘记测试或者抛出了错误的异常的风险就会大大增加。

> [4] 当然，返回指针无论如何都必须被检查，除非你绝对肯定转型不会失败。

### polymorphic_cast 不总是正确的选择

如果说失败的指针转型不应被视为错误，你就应该使用 `dynamic_cast` 而不是 `polymorphic_cast`. 例如，一种常见的情形是使用 `dynamic_cast` 来进行类型确定测试。使用异常处理来进行几种类型的转换测试是低效的，代码也很难看。这种情形下 `dynamic_cast` 就很有用了。当我们同时使用 `polymorphic_cast` 和 `dynamic_cast`时，你应该非常清楚你自己的意图。即使没有 `polymorphic_cast`, 如果人们知道使用`dynamic_cast`的方法，他仍然可以达到相同的安全性，如下例所示。

```cpp
void failure_is_error(base1* p) {

  try {
    some_other_class& soc=dynamic_cast<some_other_class&>(*p);
    // 使用 soc
   }
  catch(std::bad_cast& e) {
    std::cout << e.what() << '\n';
  }
}

void failure_is_ok(base1* p) {
  if (some_other_class* psoc=
    dynamic_cast<some_other_class*>(p)) {
    // 使用 psoc
  }
} 
```

在这个例子中，指针 `p` 被解引用[5] 并被转型为 `some_other_class`的引用。这调用了`dynamic_cast`的异常抛出版本。例子中的第二部分使用了不会抛出异常的版本来转型到指针类型。你是否认为这是清晰、简明的代码，答案取决于你的经验。经验丰富的 C++程序员会非常明白这段程序。是不是所有看到这段代码的人都十分熟悉`dynamic_cast`呢，或者他们不知道`dynamic_cast`的 行为要取决于进行转型的是指针还是引用呢？你或者一个维护程序员是否总能记得对空指针进行测试？维护代码的程序员是否知道要对指针进行解引用才可以在转型 失败时获得异常？你真的想在每次你需要这样的行为时都写相同的逻辑吗？抱歉说了这么多，这只是想表明，如果转型失败应该要抛出异常，那么 `polymorphic_cast` 要比 `dynamic_cast` 更坚固也更清晰。它要么成功，产生一个有效的指针，要么失败，抛出一个异常。简单的规则总是更容易被记住。

> [5] 如果指针 `p` 为空，该例将导致未定义行为，因为它解引用了一个空指针。

我们还没有看到如何通过重载 `polymorphic_cast` 来解决一些不常见的转型需求，但你应该知道这是可能的。何时你会想改变多态转型的缺省行为呢？有一种情形是句柄/实体类(handle/body-classes), 向下转型的规则可能会与缺省的不同，或者是根本不允许。

### 总结

必须记住，其它人将要维护我们写的代码。这意味着我们必须确保代码以及它的意图是清晰并且易懂的。这一点可以通过注释部分地解决，但对于任何人，更容易的方法是不需加以说明的代码。当(指针)转型失败被认为是异常时，`polymorphic_cast` 比`dynamic_cast`更能清晰地表明代码的意图，它也导致更短的代码。如果转型失败不应被认为是错误，则应该使用`dynamic_cast`，这使得`dynamic_cast`的使用更为清楚。仅仅使用 `dynamic_cast` 来表明两种不同的意图很容易出错，而不够清楚。抛出异常与不抛出异常这两个不同的版本对于大多数程序员而言太微妙了。

何时使用 `polymorphic_cast` 和 `dynamic_cast`:

*   当一个多态转型的失败是预期的时候，使用 `dynamic_cast&lt;T*&gt;`. 它清楚地表明转型失败不是一种错误。

*   当一个多态转型必须成功以确保逻辑的正确性时，使用 `polymorphic_cast&lt;T*&gt;`. 它清楚地表明转型失败是一种错误。

*   对引用类型执行多态转型时，使用 `dynamic_cast`.

# polymorphic_downcast

## polymorphic_downcast

### 头文件: `"boost/cast.hpp"`

有时 `dynamic_cast` 被认为太过低效(的确如此)。执行`dynamic_cast`需要额外的运行时间。为了避免这些代价，常常会诱使你使用 `static_cast`, 它没有这些性能代价。`static_cast` 用于向下转型可能在危险的，并会导致错误，但它的确比`dynamic_cast`要快。如果这些加速是需要的，那我们就要确保向下转型的安全性。`dynamic_cast` 会测试向下转型的结果，并在失败时返回空指针或抛出异常，而`static_cast` 则仅仅执行需要的指针运算，并将保证转型有效的责任留给了程序员。为了确保用 `static_cast` 进行向下转型是安全的，你必须确保对每次要执行的转型进行测试。`polymorphic_downcast` 用`dynamic_cast`进行了转型的测试，但仅是在调试模式下；然后它就使用 `static_cast` 去执行转型。在发布模式下，只执行 `static_cast` 。这样的转型方法意味着你知道它不可能失败，所以没有错误处理，也没有异常抛出。那么如果在非调试模式下 `polymorphic_downcast` 失败了，会发生什么呢？未定义的行为。你的计算机可能崩溃。地球可以停止自转。你可能飞到云上。你唯一可以肯定的是你的程序可能会发生不好的事情。如果 `polymorphic_downcast` 是在调试模式下失败的，它对`dynamic_cast`产生的空指针执行断言(并退出)。

在讨论用`polymorphic_downcast`更换`dynamic_cast`可以如何加速你的程序之前，你应该先检查一下设计。转型的优化几乎就代表着设计的问题。如果向下转型真的是必须的，并且被证实是性能的瓶颈，`polymorphic_downcast` 就是你需要的。你可以在测试时发现错误的转型，而不是在产品中(发布模式构建)，如果你曾经听到过从电话另一端传来的用户的尖叫，你就该知道在测试时找出错误是多么的重要，它使生活更轻松。很有可能你就是用户，而且知道发现并报告别人的错误是多么的讨厌。因此，在真正需要的时候才用 `polymorphic_downcast` ，而且要小心。

### 用法

`polymorphic_downcast` 用于那些你应该用而又不想用`dynamic_cast`的情形，原因是你确认将要发生的转型肯定会成功，而且你需要提升它带来的性能。注意：一定要确保使用的`polymorphic_downcast`所有可能的类型及转换组合都经过测试。否则，不要使用 `polymorphic_downcast`; 用 `dynamic_cast` 代替它。当你决定继续使用`polymorphic_downcast`, 包含头文件`"boost/cast.hpp"`.

```cpp
#include <iostream>
#include "boost/cast.hpp"

struct base {
  virtual ~base() {};
};

struct derived1 : public base {
  void foo() {
    std::cout << "derived1::foo()\n";
  }
};

struct derived2 : public base {
  void foo() {
    std::cout << "derived2::foo()\n";
  }
};

void older(base* p) {
  // Logic that suggests that p points to derived1 omitted
  derived1* pd=static_cast<derived1*>(p);
  pd->foo(); // <-- What will happen here?
}

void newer(base* p) {
  // Logic that suggests that p points to derived1 omitted
  derived1* pd=boost::polymorphic_downcast<derived1*>(p);
  // ^-- The above cast will cause an assertion in debug builds
  pd->foo();
}

int main() {
       derived2* p=new derived2;
       older(p); // <-- Undefined
       newer(p); // <-- Well defined in debug build
} 
```

函数`older`中的`static_cast` 会编译成功，[6] 但它会带来坏运气，成员函数`foo`的存在使得错误(可能有，但不保证)被错过，直到有人拿着一份错误报告，用调试器在别的地方查找奇怪的行为。当使用`static_cast`将指针向下转型为 `derived1*`, 编译器没有选择，只能相信程序员，转型是有效的。但事实上，传送给`older`的指针是指向一个`derived2`实例的。因此，`older`里的指针 `pd` 指向了一个完全不同的类型，这意味着什么都可能发生。这就是使用`static_cast`进行向下转型的风险。转型总是"成功"的，但指针可能是无效的。

> [6] 至少它会被编译。

在对函数`newer`的调用里，"更好的 `static_cast`," `polymorphic_downcast` 不仅捕捉到了错误，并且使用断言指出了发生错误的地方。当然，这仅在调试模式下是真的，使用`dynamic_cast`来测试转型是否成功。把一个无效的转型留在发布版本中会导致不幸。换言之，就算你在调试模式下获得了额外的安全性，但这并不足以代表你已经试过了所有可能的转换。

### 总结

使用`static_cast`进行向下转换通常是危险的。你不应该这样做，但如果一定要，使用`polymorphic_downcast`可以增加一点安全性。它在调试模式下增加了测试，可以帮助你发现转型的错误，但你必须测试所有可能的转型以确保它的安全使用。

*   如果你正在使用向下转型并需要在发布版本中获得`static_cast`的速度，就用 `polymorphic_downcast`; 至少在测试时你可以在出错时得到断言的帮助。

*   如果不能测试所有可能的转型，就不要使用 `polymorphic_downcast`.

记住这是一种优化方法，你应该在确定需要它们时才使用。

# numeric_cast

## numeric_cast

### 头文件: `"boost/cast.hpp"`

整数类型间的转换经常会产生意外的结果。例如，`long` 可以拥有比`short`更大范围的值，那么当从 `long` 赋值到`short` 并且 `long`的数值超出了 `short`的 范围时会发生什么？答案是结果是由实现定义的(比"你不可能明确知道"好听一点的说法)。相同大小整数间的有符号数到无符号数的转换是好的，只要有符号数 的数值是正的，但如果有符号数的数值是负的呢？它将被转换为一个大的无符号数，如果这不是你的真实意图，那么就真的是一个问题了。`numeric_cast` 通过测试范围是否合理来确保转换的有效性，当范围超出时它会抛出异常。

在我们全面认识 `numeric_cast`之前，我们必须弄清楚支配整数类型的转换及提升的规则。规则有很多并有时很微妙，即使是经验丰富的程序员也会被它们欺骗。与其写出所有这些规则[7]并展开它们，我更愿意给出一些有关转换的例子，它们会引起未定义或令人惊讶的行为，然后再解释所使用的转换规则。

```cpp
 [7]. C++标准在§4.5-4.9 中讨论数字类型的提升及转换。 
```

当从一种数字类型赋值给另一种数字类型的变量时，就会发生类型转换。在目标类型可以保存源类型的所有数值的情况下，这种转换是完全安全的，否则就是不安全的。例如，`char` 通常不能保存`int`的最大值，所以当从`int`到`char`的赋值发生时，很大可能`int`的值不能被表示为`char`. 当类型可以表示的数值范围不同时，我们必须确认用于转换的实际数值在目标类型的有效范围之内。否则，我们就会进入实现定义行为的范畴；那就是在把一个超出数字类型可能的数值范围的值赋给这个数字类型时会发生的事情。[8] 实现定义行为意味着具体实现可以自由地做任何它想做的；不同的系统可能有完全不同的行为。`numeric_cast` 可以确保转换是有效的、合法的，否则就不允许转换。

> [8] 无符号数也算，尽管它的行为是有定义的。

### 用法

`numeric_cast` 是一个看起来象 C++的转型操作符的函数模板，它泛化了目标类型及源类型。源类型可以从函数的参数隐式推导得到。使用`numeric_cast`, 要包含头文件`"boost/cast.hpp"`。以下两个转换使用 `numeric_cast` 安全地将 `int` 转换为 `char`, 以及将 `double` 转换为 `float`.

```cpp
char c=boost::numeric_cast<char>(12);
float f=boost::numeric_cast<float>(3.001); 
```

一个最常见的数字转换问题是将来自一个更宽范围的值赋给范围较窄的类型。我们来看看 numeric_cast 如何帮忙。

### 从较大的类型到较小类型的赋值

从较大的类型(例如`long`)向较小的类型(例如`short`)赋值，有可能数值过大或过小而不能被目标类型所表示。如果这发生了，结果是(是的，正如你猜到的)实现所定义的。我们稍后将讨论无符号类型的潜在问题；我们先从有符号类型开始。C++中有四个内建的有符号类型：

*   `signed char`

*   `short int (short)`

*   `int`

*   `long int (long)`

没有人可以绝对肯定哪个类型比其它的大[9]，但典型地，上面的列表是按大小递增的，除了 `int` 和 `long` 通常具有相同的值范围。但它们都是独立的类型，即使是有相同的大小。想查看你的系统上的类型大小，可以使用 `sizeof(T)` 或 `std::numeric_limits&lt;T&gt;::max()` 和 `std::numeric_limits&lt;T&gt;::min()`.

> [9] 当然，有符号类型与无符号类型的范围是不同的，即使它们有相同的大小。

当把一个有符号整数类型赋给另一个时，C++标准说：

> "若目标类型为有符号类型，在数值可以被目标类型表示时，值不改变；否则，值为实现定义。"[10]
> 
> > [10] 见 C++标准 §4.7.3

以下代码段示范了看起来象是正确的赋值是如何导致实现定义的数值，最后看看如何通过`numeric_cast`的帮助避免它们。

```cpp
#include <iostream>
#include "boost/cast.hpp"
#include "boost/limits.hpp"

int main() {
  std::cout << "larger_to_smaller example\n";

  // 没有使用 numeric_cast 的转换
  long l=std::numeric_limits<short>::max();

  short s=l;
  std::cout << "s is: " << s << '\n';
  s=++l;
  std::cout << "s is: " << s << "\n\n";

  // 使用 numeric_cast 的转换
  try {
    l=std::numeric_limits<short>::max();
    s=boost::numeric_cast<short>(l);
    std::cout << "s is: " << s << '\n';
    s=boost::numeric_cast<short>(++l);
    std::cout << "s is: " << s << '\n';
  }
  catch(boost::bad_numeric_cast& e) {
    std::cout << e.what() << '\n';
  }
} 
```

通过使用 `std::numeric_limits`, `long l` 被初始化 `short` 可以表示的最大值。该值被赋给 `short s` 并输出。然后，`l` 被加一，这意味着它的值不能再被`short`所表示；它超出了 `short` 所能表示的范围。把 `l` 的新值赋给 `s`, `s` 再次被输出。你可能要问输出的值是什么？好的，因为赋值的结果属于实现定义的行为，这取决于你使用的平台。在我的系统中，使用我的编译器，它变成了一个大的负值，即它被回绕了。必须运行前面的代码才知道在你的系统中会有什么结果[11]。接着，再次执行相同的操作，但这次用了 `numeric_cast`. 第一个转型成功了，因为数值在范围之内。而第二个转型却会失败，结果是抛出一个 `bad_numeric_cast` 异常。程序的输出如下。

> [11] 这种行为和结果在 32 位平台上十分常见。

```cpp
larger_to_smaller example
s is: 32767
s is: -32768

s is: 32767
bad numeric cast: loss of range in numeric_cast 
```

比避开实现定义行为更为重要的是，`numeric_cast` 帮助我们避免了错误，否则会很难捕捉到这些错误。那个奇怪的数值可能被传送到应用程序的其它部分，程序可能会继续工作，但几乎可以肯定将产生错误的结果。 当然，这仅对于特定的数值会发生这样的情况，如果这些数值很少出现，那么错误将很难被发现。这种错误非常阴险，因为它们仅仅对某些特定值会发生，而不是总会发生。

精宽或取值范围的损失并不常见，如果你不确定一个值对于目标类型是否过大或过小，`numeric_cast` 就是你可以使用的工具。你甚至可以在不需要的时候使用 `numeric_cast` ；维护的程序员可能没有象你一样的洞察力。注意，虽然我们在这里只讨论了有符号类型，但同样的原理可应用于于无符号类型。

### 特殊情况：目标类型为无符号整数

无符号整数类型有一个非常有趣的特性，任何数值都有可以合法地赋给它们！对于无符号类型而言，无所谓正或负的溢出。数值被简单地对目标类型最大值加一取模。什么意思？看看以下例子会更清楚一些。

```cpp
#include <iostream>
#include "boost/limits.hpp"

int main() {
  unsigned char c;
  long l=std::numeric_limits<unsigned char>::max()+14;

  c=l;
  std::cout << "c is:       " << (int)c << '\n';
  long reduced=l%(std::numeric_limits<unsigned char>::max()+1);
  std::cout << "reduced is: " << reduced << '\n';
} 
```

运行这个程序的输出如下：

```cpp
c is:       13
reduced is: 13 
```

这个例子把一个明显超出`unsigned char`可以表示的数值赋给它，然后再计算得到同样的数值。赋值的动作可以用这一行代码来示范：

```cpp
long reduced=l%(std::numeric_limits<unsigned char>::max()+1); 
```

这种行为通常被称为数值回绕(value wrapping)。如果你想用这个特性，就没有必要在这种情况下使用 `numeric_cast`。此外，`numeric_cast` 也不接受它。`numeric_cast`的意图是捕捉错误，而错误应该是因为用户的误解而引起的。如果目标类型不能表示赋给它的数值，就抛出一个 `bad_numeric_cast` 异常。因为无符号整数的算法是明确定义的，不会引起程序员的重大错误[12]。对于 `numeric_cast`, 重要的是确保获得实际的数值。

> [12] 观点是：如果你真的想要数值回绕，就不要使用 `numeric_cast`.

### 有符号和无符号整数类型的混用

混用有符号和无符号类型可能很有趣[13]， 特别是执行算术操作时。普通的赋值也会产生微妙的问题。最常见的问题是将一个负值赋给无符号类型。结果几乎可以肯定不是你原来的意图。另一种情形是从无符 号类型到同样大小的有称号类型的赋值。不知什么原因，人们总是会很容易忘记无符号类型可以持有比同样大小的有符号类型更大的值。特别是在表达式或函数调用 中更容易忘记。以下例子示范了如何通过`numeric_cast`来捕捉这种常见的错误。

> [13] 当然这是一个高度主观的问题，你的观点可能不同。

```cpp
#include <iostream>
#include "boost/limits.hpp"
#include "boost/cast.hpp"

int main() {
  unsigned int ui=std::numeric_limits<unsigned int>::max();
  int i;

  try {
    std::cout << "Assignment from unsigned int to signed int\n";
    i=boost::numeric_cast<int>(ui);
  }
  catch(boost::bad_numeric_cast& e) {
    std::cout << e.what() << "\n\n";
  }

  try {
    std::cout << "Assignment from signed int to unsigned int\n";
    i=-12;
    ui=boost::numeric_cast<unsigned int>(i);
  }
  catch(boost::bad_numeric_cast& e) {
    std::cout << e.what() << "\n\n";
  }
} 
```

输出清晰地表明了预期的错误。

```cpp
Assignment from unsigned int to signed int
bad numeric cast: loss of range in numeric_cast
Assignment from signed int to unsigned int
bad numeric cast: loss of range in numeric_cast 
```

基本的规则很简单：无论何时在不同的类型间执行类型转换，都应该使用 `numeric_cast`来保证转换的安全。

### 浮点数类型

`numeric_cast` 不能帮助我们在浮点数间的转换中避免精度的损失。原因是`float`, `double`, 和 `long double`间的转换不象整数类型间的隐式转换那样敏感。记住这点很重要，因为你可能会认为以下代码应该抛出异常。

```cpp
double d=0.123456789123456;
float f=0.123456;

try {
  f=boost::numeric_cast<float>(d);
}
  catch(boost::bad_numeric_cast& e) {
    std::cout << e.what();
} 
```

运行这段代码不会有异常抛出。在许多实现中，从 `double` 到 `float` 的转换都会导致精度的损失，虽然 C++标准没有保证会这样。我们所能知道的就是，`double` 至少具有 `float` 的精度。

从浮点数类型转为整数类型又会怎样呢？当一个浮点数类型被转换为一个整数类型，它会被截断；小数部分会被扔掉。`numeric_cast` 对截断后的数值与目标类型进行相同的检查，就象在两个整数类型间的检查一样。

```cpp
double d=127.123456789123456;
char c;
std::cout << "char type maximum: ";
std::cout << (int)std::numeric_limits<char>::max() << "\n\n";

c=d;
std::cout << "Assignment from double to char: \n";
std::cout << "double: " << d << "\n";
std::cout << "char:   " << (int)c << "\n";

std::cout << "Trying the same thing with numeric_cast:\n";

try {
  c=boost::numeric_cast<char>(d);
  std::cout << "double: " << d;
  std::cout << "char:   " << (int)c;
}
  catch(boost::bad_numeric_cast& e) {
    std::cout << e.what();
} 
```

象前面的代码那样进行范围检查以确保有效的赋值是一件令人畏缩的工作。虽然规则看起来很简单，但是有很多组合要被考虑。例如，测试从浮点数到整数的代码看起来就象这样：

```cpp
template <typename INT, typename FLOAT>
  bool is_valid_assignment(FLOAT f) {
    return std::numeric_limits<INT>::max() >=
      static_cast<INT>(f);
  } 
```

尽管我已经提起过在一个浮点数类型被转换时，小数部分会被丢弃，在这个实现中还得很容易忽略这个错误。这对于算术类型的转换和提升是自然的。去掉 `static_cast` 就可以正确地测试，因为这样 `numeric_limits&lt;INT&gt;::max` 的结果会被转换为浮点数类型[14]。如果是浮点数类型转为整数类型，它会被截断；换句话说，这个函数的问题在于丢失了小数部分。

> [14] 这是正常的算术转换结果。

### 总结

`numeric_cast` 提供了算术类型间高效的范围检查转换。在目标类型可以持有所有源类型的值时，使用 `numeric_cast`没有额外的效率代价。它只在目标类型仅能表示源类型的值的子集时有影响。当转换失败时，`numeric_cast` 通过抛出一个 `bad_numeric_cast`异常来表示失败。对于数值类型间的转换有很多复杂的规则，确保转换的正确性是很重要的。

以下情况时使用 `numeric_cast`:

*   在无符号与有符号类型间进行赋值或比较时

*   在不同大小的整数类型间进行赋值或比较时

*   从一个函数返回类型向一个数值变量赋值，为了预防该函数未来的变化

在这里注意到一个模式了吗？模仿已有的语言和库的名字及行为是简化学习及使用的好方法，但也需要仔细 地考虑。增加内建的 C++转型就象沿着狭窄的小路行走；一旦迷路会带来很高的代价。遵循语言的语法及语义规则才是负责任的。事实上，对于初学者，内建的转 型操作符与看起来象转型操作符的函数可能并没有不同，所以如果行为错误将会导致灾难。`numeric_cast` 有着与 `static_cast`, `dynamic_cast`, 和 `reinterpret_cast`类似的语法和语义。如果它看起来和用起来象转型操作，它就是转型操作，是对转型操作的一个良好的扩展。

# lexical_cast

## lexical_cast

### 头文件: `"boost/lexical_cast.hpp"`

所有应用都会使用字面转换。我们把字符串转为数值，反之亦然。许多用户定义的类型可以转换为字符串或者由字符串转换而来。你常常是在需要这些转换时才编写代码，而更好的方法是提供一个可重用的实现。这就是 `lexical_cast`的用途所在。你可以把`lexical_cast` 想象为使用一个 `std::stringstream` 作为字符串与数值的表示之间的翻译器。这意味着它可以与任何用`operator&lt;&lt;`进行输出的源以及任何用`operator&lt;&lt;`进行输入的目标一起工作。这个要求对于所有内建类型与多数用户自定义类型(UDTs)都可以做到。

### 用法

`lexical_cast` 在类型之间进行转换，就象其它的类型转换操作一样。当然，使它得以工作的必须是一个转换函数，但从概念上说，你可以把它视为转型操作符。比起调用一堆的转换子程序，或者是编写自己的转换代码，`lexical_cast` 可以更好地为任何满足它的要求的类型服务。它的要求就是，源类型必须是可流输出的(OutputStreamable)，而目标类型必须是可流输入的 (InputStreamable)。另外，两种类型都必须是可复制构造的(CopyConstructible)，并且目标类型还要是可缺省构造的 (DefaultConstructible)和可赋值的(Assignable)。可流输出(OutputStreamable)意味着存在一个为该类 型定义的`operator&lt;&lt;`，可流输入(InputStreamable)则要求有一个`operator&gt;&gt;`. 对于许多类型，包括所有内建类型和标准库中的字符串类型，这个条件都满足。要使用 `lexical_cast`, 就要包含头文件 `"boost/lexical_cast.hpp"`.

### 让 lexical_cast 工作

我不想通过跟你示范手工编写转换用的代码来说明 `lexical_cast` 如何节省了你的时间，因为我可以很肯定你一定写过这样的转换代码，并且很可能不只一次。相反，只用一个例子来示范如何使用 `lexical_cast` 来进行通用的(字面上的)类型转换。

```cpp
#include <iostream>
#include <string>
#include "boost/lexical_cast.hpp"

int main() {
  // string to int
  std::string s="42";
  int i=boost::lexical_cast<int>(s);

  // float to string
  float f=3.14151;
  s=boost::lexical_cast<std::string>(f);

  // literal to double
  double d=boost::lexical_cast<double>("2.52");

  // 失败的转换
  s="Not an int";
  try {
    i=boost::lexical_cast<int>(s);
  }
  catch(boost::bad_lexical_cast& e) {
    // 以上 lexical_cast 将会失败，我们将进入这里
  }
} 
```

这个例子仅仅示范了多种字面转换情形中的几种，我想你应该同意为了完成这些工作，通常你需要更多的代码。无论何时你不确定转换是否有效，都应该用一个`try/catch` 块来保护 `lexical_cast` ，就象你在这个例子看到的那样。你可能注意到了没有办法控制这些转换的格式；如果你需要这种级别的控制，你要用 `std::stringstream`！

如果你曾经手工进行过类型间的转换，你应该知道，对于不同的类型，需要使用不同的办法来处理转换以及可能出现的转换失败。这不仅是有点不便而已，它还妨碍了用泛型代码执行转换的努力。稍后我们将看到 lexical_cast 如何帮助你实现这一点。

这个例子中的转换用手工来实现也非常简单，但可能会失去转型操作的美观和优雅。而 `lexical_cast` 做起来更简单，并且更美观。再考虑一下`lexical_cast`对与之一起工作的类型所需的简单要求。考虑到对所有符合该要求的类型的转换可以在一行代码内完成的事实。再结合该实现依赖于标准库的`stringstream`这一事实[15]，你可以看到 lexical_cast 不仅是执行字面转换的便利方法，它更是 C++编译艺术的一个示范。

> [15] 事实上，对于某些转换，有一些优化的方法可以避免使用 `std::stringstream` 带来的额外开销。当然，你可以在需要的时候对你自己的类型定制它的行为。

### 用 lexical_cast 进行泛型编程

作为使用`lexical_cast`进行泛型编程的简单例子，来看一下如何用它创建一个 `to_string` 函数。这个函数接受任何类型的参数(当然它要符合要求)并返回一个表示该值的 `string` 。标准库的用法当然也可以在`std::stringstream`的帮助下用几行代码完成这个任务。在这里，我们使用`lexical_cast` 来实现，只需要一个前转换函数调用及一些错误处理。

```cpp
#include <iostream>
#include <string>
#include "boost/lexical_cast.hpp"

template <typename T> std::string to_string(const T& arg) {
  try {
    return boost::lexical_cast<std::string>(arg);
  }
  catch(boost::bad_lexical_cast& e) {
    return "";
  }
}

int main() {
  std::string s=to_string(412);
  s=to_string(2.357);
} 
```

这个小程序不仅易于实现，它还因为`lexical_cast`而增加了价值。

### 使类可以用于 lexical_cast

因为 `lexical_cast` 仅要求它所操作的类型提供适当的 `operator&lt;&lt;` 和 `operator&gt;&gt;` ，所以很容易为用户自定义类型增加字面转换的支持。一个可以同时作为`lexical_cast`的目标和源的简单 UDT 看起来就象这样：

```cpp
class lexical_castable {
public:
  lexical_castable() {};
  lexical_castable(const std::string s) : s_(s) {};

  friend std::ostream operator<<
    (std::ostream& o, const lexical_castable& le);
  friend std::istream operator>>
    (std::istream& i, lexical_castable& le);

private:
  virtual void print_(std::ostream& o) const {
    o << s_ <<"\n";
  }

  virtual void read_(std::istream& i) const {
    i >> s_;
  }

  std::string s_;
};

std::ostream operator<<(std::ostream& o,
  const lexical_castable& le) {
  le.print_(o);
  return o;
}

std::istream operator>>(std::istream& i, lexical_castable& le) {
  le.read_(i);
  return i;
} 
```

`lexical_castable` 类现在可以这样用了：

```cpp
int main(int argc, char* argv[]) {
  lexical_castable le;
  std::cin >> le;

  try {
    int i = boost::lexical_cast<int>(le);
  }
  catch(boost::bad_lexical_cast&) {
     std::cout << "You were supposed to enter a number!\n";
  }
} 
```

当然，输入和输出操作符最好可以允许这个类于于其它流。如果你使用标准库的 IOStreams，或者其它使用 `operator&lt;&lt;` 和 `operator&gt;&gt;`的库，你可能已经有很多可以用于 `lexical_cast` 的类。它们不需要进行修改。直接对它们进行字面转换就行了！

### 总结

`lexical_cast` 是用于字符串与其它类型之间的字面转换的一个可重用及高效的工具。它是功能性和优雅性的结合，是杰出程序员的伟大杰作[16]。 不要在需要时实现小的转换函数，更不要在其它函数中直接插入相关逻辑，应该使用象 `lexical_cast` 这样的泛型工具。它有助于使代码更清晰，并让程序员专注于解决手上的问题。

> [16] 我知道，我总是很傲慢的，我们这些程序员，工作中常常需要数学、物理学、工程学、建筑学，和其它一些艺术和学科。这会使人畏缩，但也有无穷的回报。

以下情况时使用`lexical_cast`:

*   从字符串类型到数值类型的转换

*   从数值类型到字符串类型的转换

*   你的自定义类型所支持的所有字面转换

# Conversion 总结

## Conversion 总结

在这一章里，你学习了 Boost.Conversion 库，从 `polymorphic_cast`开始。`polymorphic_cast` 的基本原理是代码的清晰性和安全性，它使我们在代码中更灵活地表达我们的意图，还有安全性，与它的竞争者 `dynamic_cast&lt;T*&gt;`相比它更为安全，因为对结果指针的测试很容易忘记。

接着，你看到了安全的优化，使用 `polymorphic_downcast`, 它在调试模式下增加了类似于`dynamic_cast`的安全性，但却是使用 `static_cast` 来进行转换。这样比单独使用 `static_cast` 更安全。

`numeric_cast` 帮助你避免数值转换中的某些困难。还有，代码的清晰性也得到提高，从而避免了未定义的行为以及实现定义的行为。

最后一个是 `lexical_cast`. 没有重复的转换函数。这就是为什么它被提议纳入下一个版本的 C++标准库的原因。它是一个非常小巧的、用于转换不同的可流数据类型的工具。

如果你曾经看到过这些转型的实现，你会同意它们之间没有一个是复杂的。还有，它具有它们所需的洞察力、远见和知识，并正确地、可移植地、高效地实现了它们。不是所有人都认识到使用`dynamic_cast`时会发生某些错误。不是很多人都知道整数类型转换和提升的复杂规则。Boost 提供的转换操作包含了所有这些知识，并具有良好的设计和测试；它们是你所要的最好的选择。