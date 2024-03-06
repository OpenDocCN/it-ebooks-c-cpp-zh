# Library 9\. Bind

# Library 9\. Bind

*   Bind 库如何改进你的程序？
*   Bind 如何适用于标准库？
*   Bind
*   用法
*   Bind 总结

# Bind 库如何改进你的程序？

## Bind 库如何改进你的程序？

*   使函数和函数对象适用于标准库算法

*   使用一致语法创建绑定器

*   强大的函数组合

在使用来自于标准库的算法时，你常常需要提供给它们一个函数或一个函数对象。这是对算法的行为进行定制的一个好方法，但你通常需要写一个新的函数对象，因为你没有组合函数或改变参数的顺序等所需的工具。虽然标准库已经提供了一些可用的工具，如 `bind1st` 和 `bind2nd`, 但是这不够用。即使功能上够用了，但这通常意味着要忍受笨拙的语法，这些语法通常会让不熟悉这些工具的程序员产生混乱。你需要的是一个解决方案，既具备所需功能，又可以使用普通的语法就地创建函数对象，这正是 Boost.Bind 所要做的。

事实上，泛型绑定器是一种 lambda 表达式，因为通过函数组合，我们可以或多或少在调用点构造一个局部的、无名的函数。在许多情形下这都是需要的，因为它达到了三个目的：减少了代码的数量， 使代码更易懂，还有行为的局部化，这意味着更有效的维护。注意，还有另一个 Boost 库，Boost.Lambda, 它具有更多的特性。Boost.Lambda 将在下一章中讨论。为什么你不直接跳到下一个库？因为多数情况下，Boost.Bind 可以完成你要绑定的所有东西，并且学习曲线没那么陡。

Bind 成功的一个关键是采用统一的语法来创建函数对象，以及对于使用该库的类型只有很少的要求。这种设计使得无需关注如何去写与你的类型一起工作的代码，而只需关注我们最关心的一点，代码如何工作以及它实际上做了什么。使用来自标准库的适配器时，如 `ptr_fun` 和 `mem_fun_ref`, 代码很容易变得过分冗长，因为我们必须提供这些适配器以便参数可以符合算法的要求。在 Boost.Bind 里不是这样的，它使用了更为精妙的推断系统，并且在自动推断不能适用时提供了一个简单的语法。使用 Bind 的结果就是，你可以写更少的代码，而且代码更易懂。

# Bind 如何适用于标准库？

## Bind 如何适用于标准库？

概念上，Bind 是已有的标准库函数 `bind1st` 和 `bind2nd` 的泛化，其额外的功能就是允许更为精妙的函数组合。它还减少了对函数指针和类成员指针使用适配器的需要，从而缩短了代码，也减少了出错的机会。Boost.Bind 还包含了对 C++标准库的一些常用的扩充，如 SGI 扩充的 `compose1` 和 `compose2`, 还有 `select1st` 和 `select2nd` 函数。因此，Bind 非常适用于标准库，而且它也真的非常好用。这些功能被公认为是需要的，最终将被引入到标准库中，也是对 STL 的扩展。Boost.Bind 已经被即将发布的 Library Technical Report 所接纳。

# Bind

## Bind

### 头文件: `"boost/bind.hpp"`

Bind 库创建函数对象来绑定到一个函数(普通函数或成员函数)。不需要直接给出函数的所有参数，参数可以稍后给，这意味着绑定器可以用于创建一个改变了它所绑定到的函数的 arity (参数数量) 的函数对象，或者按照你喜欢的顺序重排参数。

函数 `bind` 的重载版本的返回类型是未指定的，即不能保证返回的函数对象的特征是怎样的。有时，你需要将对象存于某处，而不是直接把它传送给另一个函数，这时，你要使用 Boost.Function, 它在 "Library 11: Function 11." 中讨论。弄明白 `bind` 函数返回的是什么的关键在于，理解它发生了什么转换。用 `bind` 函数的一个重载，`template&lt;class R, class F&gt; unspecified-1 bind(F f)`来作为例子，返回类型就是 (引用自在线文档)，"一个函数对象 l ，表达式 l(v1, v2, ..., vm) 等同于 f()，隐式转换为 R"。这样，这个被绑定的函数就被保存在绑定器里面，以后对这个函数对象的调用就会得到被绑定的函数的返回值(如果有)，即模板参数 `R`. 我们在这讨论的实现支持最多九个函数参数。

Bind 的实现包括许多函数和类，但作为用户来说，我们不直接使用除了重载函数 `bind` 以外的任何东西。所有绑定通过 `bind` 函数发生，我们可以无须依赖于返回值的类型。使用 `bind` 时，参数占位符(命名为 `_1`, `_2`, 等等)不需要用一个 using 声明或 using 指示来引入，因为它们位于匿名名字空间。这样，在使用 Boost.Bind 时，没有理由写出以下的代码。

```cpp
using boost::bind;
using namespace boost; 
```

前面曾经提到过，当前的 Boost.Bind 实现支持九个占位符(`_1`, `_2`, `_3`, 等等)，也就是说最多九个参数。粗略地过一下大纲对于深入理解如何进行类型推断是有好处的，还可以知道何时/为何它不总是可以工作的。花点时间分析一下成 员函数指针与普通函数的署名特征也是很有用的。你将会看到对于普通函数和类成员函数，各有各的重载版本。还有，对于每一个数量的参数，也都有不同的重载。 我不在这里列出所有大纲了，建议你到[www.boost.org](http://www.boost.org/)参考一下 Boost.Bind 的文档。

# 用法

## 用法

Boost.Bind 为函数和函数对象提供了一致的语法，对于值语义和指针语义也一样。我们将从一些简单的例子开始，处理一些简单绑定的用法，然后再转移到通过嵌套绑定进行函数组合。弄明白如何使用 `bind` 的关键是，占位符的概念。占位符用于表示提供给结果函数对象的参数，Boost.Bind 支持最多九个参数。占位符被命名为 `_1`, `_2`, `_3`, `_4`, 直至 `_9`, 你要把它们放在你原先放参数的地方。作为第一个例子，我们定义一个函数，`nine_arguments`, 它将被一个 `bind` 表达式调用。

```cpp
#include <iostream>
#include "boost/bind.hpp"

void nine_arguments(
  int i1,int i2,int i3,int i4,
  int i5,int i6,int i7,int i8, int i9) {
  std::cout << i1 << i2 << i3 << i4 << i5
    << i6 << i7 << i8 << i9 << '\n';
}

int main() {
  int i1=1,i2=2,i3=3,i4=4,i5=5,i6=6,i7=7,i8=8,i9=9;
  (boost::bind(&nine_arguments,_9,_2,_1,_6,_3,_8,_4,_5,_7))
    (i1,i2,i3,i4,i5,i6,i7,i8,i9);
} 
```

在这个例子中，你创建了一个匿名临时绑定器，并立即把参数传递给它的调用操作符来调用它。如你所见，占位符的顺序是被搅乱的，这说明参数的顺序被重新安排了。注意，占位符可以在一个表达式中被多次使用。这个程序的输出如下。

```cpp
921638457 
```

这表示了占位符对应于它的数字所示位置的参数，即 `_1` 被第一个参数替换，`_2` 被第二个参数替换，等等。接下来，你将看到如何调用一个类的成员函数。

### 调用成员函数

我们来看一下如何用 `bind` 调用成员函数。我们先来做一些可以用标准库来做的事情，这样可以对比一下用 Boost.Bind 的方法。保存某种类型的元素在一个标准库容器中，一个常见的需要是对某些或全部元素调用一个成员函数。这可以用一个循环来完成，通常也正是这样做的，但还 有更好的方法。考虑下面这个简单的类，`status`, 我们将用它来示范 Boost.Bind 的易用性和强大的功能。

```cpp
class status {
  std::string name_;
  bool ok_;
public:
  status(const std::string& name):name_(name),ok_(true) {}

  void break_it() {
    ok_=false;
  }

  bool is_broken() const {
    return ok_;
  }

  void report() const {
    std::cout << name_ << " is " <<
      (ok_ ? "working nominally":"terribly broken") << '\n';
  }
}; 
```

如果我们把这个类的实例保存在一个 `vector`, 并且我们需要调用成员函数 `report`, 我们可能会象下面这样做。

```cpp
std::vector<status> statuses;
statuses.push_back(status("status 1"));
statuses.push_back(status("status 2"));
statuses.push_back(status("status 3"));
statuses.push_back(status("status 4"));

statuses[1].break_it();
statuses[2].break_it();

for (std::vector<status>::iterator it=statuses.begin();
  it!=statuses.end();++it) {
  it->report();
} 
```

这个循环正确地完成了任务，但它是冗长、低效的(由于要多次调用 `statuses.end()`)，并且不象使用标准库算法 `for_each` 那样清楚地表明意图。为了用 `for_each` 来替换这个循环，我们需要用一个适配器来对 `vector` 元素调用成员函数 `report` 。这时，由于元素是以值的方式保存的，我们需要的是适配器 `mem_fun_ref`.

```cpp
std::for_each(
  statuses.begin(),
  statuses.end(),
  std::mem_fun_ref(&status::report)); 
```

这是一个正确、合理的方法，它非常简洁，非常清楚这段代码是干什么的。以下是使用 `Boost.Bind` 完成相同任务的代码。[1]

> [1] 要注意的是`boost::mem_fn`, 它也被接纳进入 Library Technical Report, 它也可以在这种没有参数的情况下使用。 `mem_fn` 取代了 `std::mem_fun 和 std::mem_fun_ref`.

```cpp
std::for_each(
  statuses.begin(),
  statuses.end(),
  boost::bind(&status::report,_1)); 
```

这个版本同样的清楚、明白。这是前面所说的占位符的第一个真正的使用，我们同时告诉编译器和代码的读者，`_1` 用于替换这个函数所调用的绑定器的第一个实际参数。虽然这段代码节省了几个字符，但在这种情况下标准库的 `mem_fun_ref` 和 `bind` 之间并没有太大的不同，但是让我们来重用这个例子并把容器改为存储指针。

```cpp
std::vector<status*> p_statuses;
p_statuses.push_back(new status("status 1"));
p_statuses.push_back(new status("status 2"));
p_statuses.push_back(new status("status 3"));
p_statuses.push_back(new status("status 4"));

p_statuses[1]->break_it();
p_statuses[2]->break_it(); 
```

我们还可以使用标准库，但不能再用 `mem_fun_ref`. 我们需要的是适配器 `mem_fun`, 它被认为有点用词不当，但它的确正确完成了需要做的工作。

```cpp
std::for_each(
  p_statuses.begin(),
  p_statuses.end(),
  std::mem_fun(&status::report)); 
```

虽然这也可以工作，但语法变了，即使我们想做的事情非常相似。如果语法可以与第一个例子相同，那就更好了，所以我们所关心的是代码要做什么，而不是如何去做。使用 `bind`, 我们就无须关心我们处理的元素是指针了(这一点已经在容器类型的声明中表明了，对于现代的库来说，这样的冗余信息是不需要的)。

```cpp
std::for_each(
  p_statuses.begin(),
  p_statuses.end(),
  boost::bind(&status::report,_1)); 
```

如你所见，这与我们前一个例子完全一样，这意味着如果我们之前已经明白了 `bind` ，那么我们现在也清楚它。现在，我们已决定换用指针了，我们要面对另一个问题，即生存期控制。我们必须手工释放 `p_statuses` 中的元素，这很容易出错，也无须如此。所以，我们可能决定开始使用智能指针，并(再次)修改我们的代码。

```cpp
std::vector<boost::shared_ptr<status> > s_statuses;
s_statuses.push_back(
  boost::shared_ptr<status>(new status("status 1")));
s_statuses.push_back(
  boost::shared_ptr<status>(new status("status 2")));
s_statuses.push_back(
  boost::shared_ptr<status>(new status("status 3")));
s_statuses.push_back(
  boost::shared_ptr<status>(new status("status 4")));
s_statuses[1]->break_it();
s_statuses[2]->break_it(); 
```

现在，我们要用标准库中的哪个适配器呢？`mem_fun` 和 `mem_fun_ref` 都不适用，因为智能指针没有一个名为 `report` 的成员函数，所以以下代码编译失败。

```cpp
std::for_each(
  s_statuses.begin(),
  s_statuses.end(),
  std::mem_fun(&status::report)); 
```

不巧，标准库不能帮我们完成这个任务[2]。因此，我们不得不采用我们正想要摆脱的循环，或者使用 Boost.Bind, 它不会抱怨任何事情，而且正确地完成我们想要的。

> [2] 以后将可以这样做，因为 `mem_fn` 和 `bind` 都将成为未来的标准库的一部分。

```cpp
std::for_each(
  s_statuses.begin(),
  s_statuses.end(),
  boost::bind(&status::report,_1)); 
```

再一次，这段代码与前面的例子完全一样(除了容器的名字不同)。使用绑定的语法是一致的，不论是用于 值语义或是指针语义，甚至是用于智能指针。有时，使用不同的语法有助于理解代码，但在这里，不是这样的，我们的任务是对容器中的元素调用成员函数，没有更 多的也没有更少的事情。语法一致的价值不应被低估，因为它对于编写代码的人，以及对于日后需要维护代码的人都是有帮助的(当然，我们并不真的是在写需要维 护的代码，但为了这个主题，让我们假装是在写)。

这些例子示范了一个非常基本和常见的情形，在这种情形下 Boost.Bind 尤为出色。即使标准库也提供了完成相同工作的一些基本工具，但我们还是看到 Bind 既提供了一致的语法，也增加了标准库目前缺少的功能。

### 看一下门帘的后面

在你开始使用 Boost.Bind 后，这是无可避免的；你将开始惊讶它到底是如何工作的。这看起来就象是魔术，`bind` 可以推断出参数的类型和返回类型，它又是如何处理占位符的呢？我们将快速地看一下驱动这个东西的机制。它有助于知道一点 `bind`的 工作原理，特别是在试图解释这惊人的简洁性以及编译器对最轻微的错误给出的直接的错误信息。我们将创建一个非常简单的绑定器，至少是部分地模仿 Boost.Bind 的语法。为了避免把这个离题的讨论搞成几页那么长，我们只支持一类绑定，即接受单个参数的成员函数。此外，我们不会对 cv 限定符进行处理；我们只处理最简 单的情况。

首先，我们需要能够推断出我们要绑定的函数的返回类型、类的类型、和参数类型。我们用一个函数模板来做到这一点。

```cpp
template <typename R, typename T, typename Arg>
simple_bind_t<R,T,Arg> simple_bind(
  R (T::*fn)(Arg),
  const T& t,
  const placeholder&) {
  return simple_bind_t<R,T,Arg>(fn,t);
} 
```

这看起来有点可怕，毕竟这只是在定义整个机器的一部分。但是，这一部分的焦点在于类型推断在哪发生。你会注意到这个函数有三个模板参数，`R`, `T`, 和 `Arg`. `R` 是返回的类型，`T` 是类的类型，而 `Arg` 是(单个)参数的类型。这些模板参数组成了我们的函数的第一个参数，即 `R (T::*f)(Arg)`. 这样，传递一个带单个参数的成员函数给 `simple_bind` 将允许编译器推断出 `R` 为成员函数的返回类型，`T` 为成员函数的类，`Arg` 为成员函数的参数类型。`simple_bind` 的返回类型是一个函数对象，它使用与 `simple_bind` 相同的三个类型进行特化，其构造函数接受一个成员函数指针和一个对应类(`T`)的实例。 `simple_bind` 简单地忽略占位符(即函数的最后一个参数)，我保留这个参数的原因是为了模仿 Boost.Bind 的语法。在一个更好的实现中，我们显然应该使用这个参数，但是现在让我们先不要管它。这个函数对象的实现相当简单。

```cpp
template <typename R,typename T, typename Arg>
class simple_bind_t {
  typedef R (T::*fn)(Arg);
  fn fn_;
  T t_;
public:
  simple_bind_t(fn f,const T& t):fn_(f),t_(t) {}

  R operator()(Arg& a) {
    return (t_.*fn_)(a);
  }
}; 
```

从 `simple_bind` 的实现中我们可以看到，构造函数接受两个参数：第一个是指向成员函数的指针，第二个是一个 `const T` 引用，它会被复制并稍后用于给定一个用户提供的参数来调用其成员函数。最后，调用操作符返回 `R`, 即成员函数的返回类型，并接受一个 `Arg` 参数，即传给成员函数的那个参数的类型。调用成员函数的语法稍稍有点晦涩：

```cpp
(t_.*fn_)(a); 
```

`.*` 是成员指针操作符，它的第一个操作数是 `class T`; 另外还有一个成员指针操作符，`-&gt;*`, 它的第一个操作数是是一个 `T` 指针。剩下就是创建一个占位符，即用于替换实际参数的变量。我们可以通过在匿名名字空间中包含某种类型的变量来创建一个占位符；我们把它称为 `placeholder`:

```cpp
namespace {
  class placeholder {};
  placeholder _1;
} 
```

我们创建一个简单的类和一个小程序来测试一下。

```cpp
class Test {
public:
  void do_stuff(const std::vector<int>& v) {
    std::copy(v.begin(),v.end(),
      std::ostream_iterator<int>(std::cout," "));
  }
};

int main() {
  Test t;
  std::vector<int> vec;
  vec.push_back(42);
  simple_bind(&Test::do_stuff,t,_1)(vec);
} 
```

当我们用上述参数实例化函数 `simple_bind` 时，类型被自动推断；`R` 是 `void`, `T` 是 `Test`, 而 `Arg` 是一个 `const std::vector&lt;int&gt;` 引用。函数返回一个 `simple_bind_t&lt;void,Test,Arg&gt;` 的实例，我们立即调用它的调用操作符，并传进一个参数 `vec`.

非常不错，`simple_bind` 已经给了你关于绑定器如何工作的一些想法。现在，是时候回到 Boost.Bind 了！

### 关于占位符和参数

第一个例子示范了 `bind` 最多可以支持九个参数，但了解多一点关于参数和占位符如何工作的情况，可以让我们更好地使用它。首先，很重要的一点是，普通函数与成员函数之间有着非常大的差异，在绑定一个成员函数时，`bind` 表达式的第一个参数必须是成员函数所在类的实例！理解这个规则的最容易的方法是，这个显式的参数将取替隐式的 `this` ，被传递给所有的非静态成员函数。细心的读者将会留意到，实际上这意味着对于成员函数的绑定器来说，只能支持八个参数，因为第一个要用于传递实际的对象。以下例子定义了一个普通函数 `print_string` 和一个带有成员函数 `print_string` 的类 `some_class` ，它们将被用于 `bind` 表达式。

```cpp
#include <iostream>
#include <string>
#include "boost/bind.hpp"

class some_class {
public:
  typedef void result_type;
  void print_string(const std::string& s) const {
    std::cout << s << '\n';
  }
};

void print_string(const std::string s) {
  std::cout << s << '\n';
}

int main() {
  (boost::bind(&print_string,_1))("Hello func!");
  some_class sc;
  (boost::bind(&some_class::print_string,_1,_2))
    (sc,"Hello member!");
} 
```

第一个 `bind` 表达式绑定到普通函数 `print_string`. 因为该函数要求一个参数，因此我们需要用一个占位符(`_1`)来告诉 `bind` 它的哪一个参数将被传递为 `print_string` 的第一个参数。要调用获得的函数对象，我们必须传递一个 `string` 参数给调用操作符。参数是一个 `const std::string&`, 因此传递一个字面的字符串将引发一个 `std::string` 转型构造函数的调用。

```cpp
(boost::bind(&print_string,_1))("Hello func!"); 
```

第二个绑定器用于一个成员函数，`some_class` 的 `print_string` 。`bind` 的第一个参数是成员函数指针。但是，一个非静态成员函数指针并不真的是一个指针[3]。我们必须要有一个对象才可以调用这个函数。这就是为什么这个 `bind` 表达式必须声明绑定器有两个参数，调用它时两个参数都必须提供。

> [3] 是的，我知道这听起来很怪异。但它的确是真的。

```cpp
boost::bind(&some_class::print_string,_1,_2); 
```

要看看为什么会这样，就要考虑一下得到的这个函数对象要怎么使用。我们必须把一个 `some_class` 实例和一个 `print_string` 用的参数一起传递给它。

```cpp
(boost::bind(&some_class::print_string,_1,_2))(sc,"Hello member!"); 
```

这个调用操作符的第一个参数是 `this` ，即那个 `some_class` 实例。注意，这第一个参数可以是一个指针(智能的或裸的)或者是一个引用；`bind` 是非常随和的。调用操作符的第二个参数是那个成员函数要用的参数。这里，我们"延迟"了所有两个参数，即我们定义的这个绑定器，它的两个参数，对象本身及 成员函数的参数，都要在调用操作符时才指定。我们不是一定非这样做不可。例如，我们可以创建一个绑定器，每次调用它时，都是对同一个对象调用 `print_string` ，就象这样：

```cpp
(boost::bind(&some_class::print_string,some_class(),_1))
 ("Hello member!"); 
```

这次得到的函数对象已经包含了一个 `some_class` 实例，因此它的调用操作符只需要一个占位符(`_1`)和一个参数(一个 string)。最后，我们还可以创建一个所谓的无参(nullary)函数，它连那个 string 也绑定了，就象这样：

```cpp
(boost::bind(&some_class::print_string,
 some_class(),"Hello member!"))(); 
```

这些例子清楚地显示了 `bind` 的多功能性。它可用于延迟它所封装的函数的所有参数、部分参数、或一个参数也不延迟。它也可以把参数按照你所要的顺序进行重排；只要照你的需要排列占位符就行了。接下来，我们将看看如何用 `bind` 来就地创建排序用的谓词。

### 动态的排序标准

在对容器中的元素进行排序时，我们有时候需要创建一个函数对象以定义排序的标准，如果我们没有提供关系操作符，或者是已有的关系操作符不是我们想要的排序标准时，就需要这样做了。有些时候我们可以使用来自标准库的比较函数对象(`std::greater`, `std::greater_equal`, 等等)，但只能对已有类型进行比较，我们不能就地定义一个新的。我们将使用一个名为 `personal_info` 的类来演示 Boost.Bind 如何帮助我们。`personal_info` 包含有 first name, last name, 和 age, 并且它没有提供任何的比较操作符。这些信息在创建以后就不再变动，并且可以用成员函数 `name`, `surname`, 和 `age` 来取出。

```cpp
class personal_info {
  std::string name_;
  std::string surname_;
  unsigned int age_;

public:
  personal_info(
    const std::string& n,
    const std::string& s,
    unsigned int age):name_(n),surname_(s),age_(age) {}

  std::string name() const {
    return name_;
  }

  std::string surname() const {
    return surname_;
  }

  unsigned int age() const {
    return age_;
  }
}; 
```

我们通过提供以下操作符来让这个类可以流输出(OutputStreamable)：

```cpp
std::ostream& operator<<(
  std::ostream& os,const personal_info& pi) {
  os << pi.name() << ' ' <<
    pi.surname() << ' ' << pi.age() << '\n';
  return os;
} 
```

如果我们要对含有类型 `personal_info` 元素的容器进行排序，我们就需要为它提供一个排序谓词。为什么开始的时候我们没有为 `personal_info` 提供关系操作符呢？一个原因是，因为有几种排序的可能性，而我们不知道对于不同的用户哪一种是合适的。虽然我们也可以选择为不同的排序标准提供不同的成员 函数，但这样会加重负担，我们要在类中实现所有相关的排序标准，这并不总是可以做到的。幸运的是，我们可以很容易地用 `bind` 就地创建所需的谓词。我们先看看基于年龄(可以通过成员函数 `age` 取得)来进行排序。我们可以为此创建一个函数对象。

```cpp
class personal_info_age_less_than :
  public std::binary_function<
  personal_info,personal_info,bool> {
public:
  bool operator()(
  const personal_info& p1,const personal_info& p2) {
    return p1.age()<p2.age();
  }
}; 
```

我们让 `personal_info_age_less_than` 公有派生自 `binary_function`. 从 `binary_function` 派生可以提供使用适配器时所需的 `typedef` ，例如使用 `std::not2`. 假设有一个 `vector`, `vec`, 含有类型为 `personal_info` 的元素，我们可以象这样来使用这个函数对象：

```cpp
std::sort(vec.begin(),vec.end(),personal_info_age_less_than()); 
```

只要不同的比较方式的数量很有限，这种方式就可以工作良好。但是，有一个潜在的问题，计算逻辑被定义 在不同的地方，这会使得代码难以理解。利用一个较长的、描述清晰的名字可以解决这个问题，就象我们在这里做的一样，但是不是所有情况都会这样清晰，有很大 可能我们需要为大于、小于或等于关系提供一堆的函数对象。

那么，Boost.Bind 有什么帮助呢？实际上，在这个例子中它可以帮助我们三次。如果我们要解决这个问题，我们发现有三件事情要做，第一件是绑定一个逻辑操作，如 `std::less`. 这很容易，我们可以得到第一部分代码。

```cpp
boost::bind<bool>(std::less<unsigned int>(),_1,_2); 
```

注意，我们通过把 `bool` 参数提供给 `bind`，显式地给出了返回类型。有时这是需要的，对于有缺陷的编译器或者在无法推断出返回类型的上下文时。如果一个函数对象包含 `typedef`, `result_type`, 就不需要显式给出返回类型[4]。现在，我们有了一个接受两个参数的函数对象，两个参数的类型都是 `unsigned int`, 但我们还不能用它，因为容器中的元素的类型是 `personal_info`, 我们需要从这些元素中取出 age 并把它作为参数传递给 `std::less`. 我们可以再次使用 `bind` 来实现。

> [4] 标准库的函数对象都定义了 `result_type` ，因此它们可以与 `bind` 的返回类型推断机制共同工作。

```cpp
boost::bind(
  std::less<unsigned int>(),
  boost::bind(&personal_info::age,_1),
  boost::bind(&personal_info::age,_2)); 
```

这里，我们创建了另外两个绑定器。第一个用主绑定器的调用操作符的第一个参数(`_1`)来调用 `personal_info::age` 。第二个用主绑定器的调用操作符的第二个参数(`_2`)来调用 `personal_info::age` 。因为 `std::sort` 传递两个 `personal_info` 对象给主绑定器的调用操作符，结果就是对来自被排序的 `vector` 的两个 `personal_info` 分别调用 `personal_info::age` 。最后，主绑定器传递两个新的、内层的绑定器的调用操作符所返回的 age 给 `std::less`. 这正是我们所需要的！调用这个函数对象的结果就是 `std::less` 的结果，这意味着我们有了一个有效的比较函数对象可以用来排序容器中的 `personal_info` 对象。以下是使用它的方法：

```cpp
std::vector<personal_info> vec;
vec.push_back(personal_info("Little","John",30));
vec.push_back(personal_info("Friar", "Tuck",50));
vec.push_back(personal_info("Robin", "Hood",40));

std::sort(
  vec.begin(),
  vec.end(),
  boost::bind(
    std::less<unsigned int>(),
    boost::bind(&personal_info::age,_1),
    boost::bind(&personal_info::age,_2))); 
```

我们可以简单地通过绑定另一个 `personal_info` 成员(变量或函数)来进行不同的排序，例如，按 last name 排序。

```cpp
std::sort(
  vec.begin(),
  vec.end(),
  boost::bind(
    std::less<std::string>(),
    boost::bind(&personal_info::surname,_1),
    boost::bind(&personal_info::surname,_2))); 
```

这是一种出色的技术，因为它提供了一个重要的性质：就地实现简单的函数。它使得代码易懂且易于维护。虽然技术上可以用绑定器实现基于复杂标准的排序，但那样做是不明智的。给 `bind` 表达式添加复杂的逻辑会很快失去它的清晰和简洁。虽然有时你想用绑定来做更多的事情，但最好是让绑定器与要维护它的人一样聪明，而不是更加聪明。

### 函数组合，Part I

一个常见的问题是，将一些函数或函数对象组合成一个函数对象。假设你需要测试一个 `int` ，看它是否大于 5 且小于等于 10。使用"常规"的代码，你将这样写：

```cpp
if (i>5 && i<=10) {
  // Do something
} 
```

如果是处理一个容器中的元素，上述代码只有放在一个单独的函数时才能工作。如果你不想这样，那么用一个嵌套的 `bind` 也可以获得相同的效果(注意，这时通常不能使用标准库的 `bind1st` 和 `bind2nd`)。如果我们对这个问题进行分解，我们会发现我们需要：逻辑与(`std::logical_and`), 大于(`std::greater`), 和小于等于(`std::less_equal`)。逻辑与看起来就象这样：

```cpp
boost::bind(std::logical_and<bool>(),_1,_2); 
```

然后，我们需要另一个谓词来回答 `_1` 是否大于 5。

```cpp
boost::bind(std::greater<int>(),_1,5); 
```

然后，我们还需要另一个谓词来回答 `_1` 是否小于等于 10。

```cpp
boost::bind(std::less_equal<int>(),_1,10); 
```

最后，我们需要把它们两个用逻辑与合起来，就象这样：

```cpp
boost::bind(
  std::logical_and<bool>(),
  boost::bind(std::greater<int>(),_1,5),
  boost::bind(std::less_equal<int>(),_1,10)); 
```

这样一个嵌套的 `bind` 相对容易理解，虽然它是后序的。还有，任何人都可以逐字地阅读这段代码并弄清楚它的意图。我们用一个例子来测试一下这个绑定器。

```cpp
std::vector<int> ints;

ints.push_back(7);
ints.push_back(4);
ints.push_back(12);
ints.push_back(10);

int count=std::count_if(
  ints.begin(),
  ints.end(),
  boost::bind(
    std::logical_and<bool>(),
    boost::bind(std::greater<int>(),_1,5),
    boost::bind(std::less_equal<int>(),_1,10)));

std::cout << count << '\n';

std::vector<int>::iterator int_it=std::find_if(
  ints.begin(),
  ints.end(),
  boost::bind(std::logical_and<bool>(),
    boost::bind(std::greater<int>(),_1,5),
    boost::bind(std::less_equal<int>(),_1,10)));

if (int_it!=ints.end()) {
  std::cout << *int_it << '\n';
} 
```

使用嵌套的 `bind` 时，小心地对代码进行正确的缩入非常重要，因为如果一旦缩入错误，代码就会很难理解。想想前面那段清晰的代码，再看看以下这个容易混乱的例子。

```cpp
std::vector<int>::iterator int_it=
  std::find_if(ints.begin(),ints.end(),
    boost::bind<bool>(
    std::logical_and<bool>(),
    boost::bind<bool>(std::greater<int>(),_1,5),
      boost::bind<bool>(std::less_equal<int>(),_1,10))); 
```

当然，对于较长的代码行，这是一个常见的问题，但是在使用这里所描述的结构时更为明显，在这里长语句是合理的而不是个别例外。因此，请对你之后的程序员友好些，确保你的代码行正确缩入，这样可以让人更容易阅读。

本书的一位认真的审阅者曾经问过，在前面的例子中，为什么创建了两个相同的绑定器，而不是创建一个绑定器对象然后使用两次？答案是，因为我们不知道 `bind` 所创建的绑定器的精确类型(它是由实现定义的)，我们没有方法为它声明一个变量。还有，这个类型通常都非常复杂，因为它的署名特征包括了函数 `bind` 中所有的类型信息(自动推断的)。但是，可以用另外一个工具来保存得到的函数对象，例如来自 Boost.Function 的工具。相关方法的详情请见 "Library 11: Function 11"。

这里给出的函数组合的要点与标准库的一个著名的扩充相符，即来自 SGI STL 的函数 `compose2` ，它在 Boost.Compose 库(现在已经不用了)中也被称为 `compose_f_gx_hx` 。

### 函数组合，Part II

在 SGI STL 中的另一个常用的函数组合是 `compose1` ，在 Boost.Compose 中是 `compose_f_gx` 。这些函数提供了用一个参数调用两个函数的方法，把最里面的函数返回的结果传递给第一个函数。有时一个例子胜过千言万语，设想你需要对容器中的浮点数元素 执行两个算术操作。我们首先把值增加 10%，然后再减少 10%；这个例子对于少数工作在财政部门的人来说可能是有用的一课。

```cpp
std::list<double> values;
values.push_back(10.0);
values.push_back(100.0);
values.push_back(1000.0);

std::transform(
  values.begin(),
  values.end(),
  values.begin(),
  boost::bind(
    std::multiplies<double>(),0.90,
    boost::bind<double>(
      std::multiplies<double>(),_1,1.10)));

std::copy(
  values.begin(),
  values.end(),
  std::ostream_iterator<double>(std::cout," ")); 
```

你怎么知道哪个嵌套的 `bind` 先被调用呢？你也许已经注意到，总是最里面的 `bind` 先被求值。这意味着我们可以把同样的代码写得稍微有点不同。

```cpp
std::transform(
  values.begin(),
  values.end(),
  values.begin(),
  boost::bind<double>(
    std::multiplies<double>(),
    boost::bind<double>(
      std::multiplies<double>(),_1,1.10),0.90)); 
```

这里，我们改变了传给 `bind` 的参数的顺序，把第一个 `bind` 的参数加在了表达式的最后。虽然我不建议这样做，但它对于理解参数如何传递给 `bind` 函数很有帮助。

### bind 表达式中的是值语义还是指针语义?

当我们传递某种类型的实例给一个 `bind` 表达式时，它将被复制，除非我们显式地告诉 `bind` 不要复制它。要看我们怎么做，这可能是至关重要的。为了看一下在我们背后发生了什么事情，我们创建一个 `tracer` 类，它可以告诉我们它什么时候被缺省构造、被复制构造、被赋值，以及被析构。这样，我们就可以很容易看到用不同的方式使用 `bind` 会如何影响我们传送的实例。以下是完整的 `tracer` 类。

```cpp
class tracer {
public:
  tracer() {
    std::cout << "tracer::tracer()\n";
  }

  tracer(const tracer& other) {
    std::cout << "tracer::tracer(const tracer& other)\n";
  }

  tracer& operator=(const tracer& other) {
    std::cout <<
      "tracer& tracer::operator=(const tracer& other)\n";
    return *this;
  }

  ~tracer() {
    std::cout << "tracer::~tracer()\n";
  }

  void print(const std::string& s) const {
    std::cout << s << '\n';
  }
}; 
```

我们把我们的 `tracer` 类用于一个普通的 `bind` 表达式，象下面这样。

```cpp
tracer t;
boost::bind(&tracer::print,t,_1)
  (std::string("I'm called on a copy of t\n")); 
```

运行这段代码将产生以下输出，可以清楚地看到有很多拷贝产生。

```cpp
tracer::tracer()
tracer::tracer(const tracer& other)
tracer::tracer(const tracer& other)
tracer::tracer(const tracer& other)
tracer::~tracer()
tracer::tracer(const tracer& other)
tracer::~tracer()
tracer::~tracer()
I'm called on a copy of t

tracer::~tracer()
tracer::~tracer()  // 译注：原文没有这一行，有误 
```

如果我们使用的对象的拷贝动作代价昂贵，我们也许就不能这样用 `bind` 了。但是，拷贝还是有优点的。它意味着 `bind` 表达式以及由它所得到的绑定器不依赖于原始对象(在这里是 `t`)的生存期，这通常正是想要的。要避免复制，我们必须告诉 `bind` 我们想传递引用而不是它所假定的传值。我们要用 `boost::ref` 和 `boost::cref` (分别用于引用和 `const` 引用)来做到这一点，它们也是 Boost.Bind 库的一部分。对我们的 `tracer` 类使用 `boost::ref` ，测试代码现在看起来象这样：

```cpp
tracer t;
boost::bind(&tracer::print,boost::ref(t),_1)(
  std::string("I'm called directly on t\n")); 
```

Executing the code gives us this:

```cpp
tracer::tracer()
I'm called directly on t

tracer::~tracer()  // 译注：原文为 tracer::~tracer，有误 
```

这正是我们要的，避免了无谓的复制。`bind` 表达式使用原始的实例，这意味着没有 `tracer` 对象的拷贝了。当然，它同时也意味着绑定器现在要依赖于 `tracer` 实例的生存期了。还有一种避免复制的方法；就是通过指针来传递参数而不是通过值来传递。

```cpp
tracer t;
boost::bind(&tracer::print,&t,_1)(
  std::string("I'm called directly on t\n")); 
```

因此说，`bind` 总是执行复制。如果你通过值来传递，对象将被复制，这可能对性能有害或者产生不必要的影响。为了避免复制对象，你可以使用 `boost::ref`/`boost::cref` 或者使用指针语义。

### 虚拟函数也可以绑定

到目前为止，我们看到了 `bind`如何可以用于非成员函数和非虚拟成员函数，但是 它也可以用于绑定一个虚拟成员函数。通过 Boost.Bind, 你可以象使用非虚拟函数一样使用虚拟函数，即把它绑定到最先声明该成员函数为虚拟的基类的那个虚拟函数上。这个绑定器就可以用于所有的派生类。如果你绑定 到其它派生类，你就限制了可以使用这个绑定器的类[5]。考虑以下两个类 `base` 和 `derived` ：

> [5] 这与声明一个类指针来调用虚拟函数没有什么不同。指针指向的派生类越靠近底层，则越少的类可以绑定到指针。

```cpp
class base {
public:
  virtual void print() const {
    std::cout << "I am base.\n";
  }
  virtual ~base() {}
};

class derived : public base {
public:
  void print() const {
    std::cout << "I am derived.\n";
  }
}; 
```

我们可以用这两个类对绑定到虚拟函数进行测试，如下：

```cpp
derived d;
base b;
boost::bind(&base::print,_1)(b);
boost::bind(&base::print,_1)(d); 
```

运行这段代码可以清楚地看到结果正是我们所希望的。

```cpp
I am base.
I am derived. 
```

对于可以支持虚拟函数，你应该不会惊讶，现在我们已经示范了它和其它函数一样运行。有一个相关的注意事项，如果你 `bind` 了一个成员函数而后来它被一个派生类重新定义了，或者一个虚拟函数在基类中是公有的而在派生类中变成了私有的，那么会发生什么呢？还可以正常工作吗？如果可以，你希望是哪一种行为呢？是的，不管你是否使用 Boost.Bind，行为都不会有变化。因面，如果你 `bind`到 一个在其它类中被重新定义的函数，即它不是虚拟的并且派生类有一个相同特征的成员函数，那么基类中的版本将被调用。如果函数被隐藏，绑定器依然会被执行， 因为它显式地访问类型中的函数，这样即使是被隐藏的成员函数也可以使用。最后，如果虚拟函数在基类中声明为公有的，但在派生类中变成了私有的，那么对一个 派生类实例调用该函数将会成功，因为访问是通过一个基类实例产生的，而基类的成员函数是公有的。当然，这种情况显示出设计的确是有问题的。

### 绑定到成员变量

很多时候你需要 `bind` 数据成员而不是成员函数。例如，使用 `std::map` 或 `std::multimap` 时，元素的类型是 `std::pair&lt;key const,data&gt;`, 但你想使用的信息通常不是 key, 而是 data. 假设你想把一个 `map` 中的每个元素传递给一个函数，它接受单个 data 类型的参数。你需要创建一个绑定器，它把每个元素(类型为 `std::pair`)的 `second` 成员传给绑定的函数。以下代码举例说明如何实现：

```cpp
void print_string(const std::string& s) {
  std::cout << s << '\n';
}

std::map<int,std::string> my_map;
my_map[0]="Boost";
my_map[1]="Bind";

std::for_each(
  my_map.begin(),
  my_map.end(),
  boost::bind(&print_string, boost::bind(
    &std::map<int,std::string>::value_type::second,_1))); 
```

你可以 `bind` 到一个成员变量，就象你可以绑定一个成员函数或普通函数一样。要注意的是，要使得代码更易读(和写)，使用短的、方便的名字是个好主意。在前例中，对 `std::map` 使用一个 `typedef` 有助于提高可读性。

```cpp
typedef std::map<int,std::string> map_type;
boost::bind(&map_type::value_type::second,_1))); 
```

虽然需要 `bind` 到成员变量的时候没有象成员函数那么多，但是可以这样做还是很方便的。SGI STL (及其派生的库)的用户可能很熟悉 `select1st` 和 `select2nd` 函数。它们用于选出 `std::pair` 的 `first` 或 `second` 成员，与我们在这个例子中所做的一样。注意，`bind` 可以用于任意类型和任意名字。

### 绑定还是不绑定

Boost.Bind 库带来了很大的灵活性，但是也给程序员带来了挑战，因为有些时候本应该使用独立的函数对象的，但也会让人倾向于使用绑定器。许多工作可以也应该利用 Bind 来完成，但过度使用也是一种错误，应该在代码开始变得难以阅读、理解和维护的地方画一条分界线。不幸的是，分界线的位置是由分享(阅读、维护和扩展)代码的程序员所决定的，他们的经验决定了什么是可以接受的，什么不是。使用专门的函数对象的好处是，它们通常是无需加以说明的，而使用绑定器来提供同样清楚的信息则是一项我们必须坚持克服的挑战。例如，如果你需要创建一个你都很难弄明白的嵌套 `bind` ，有可能就是你已经过度使用了。让我们用代码来解释一下。

```cpp
#include <iostream>
#include <string>
#include <map>
#include <vector>
#include <algorithm>
#include "boost/bind.hpp"

void print(std::ostream* os,int i) {
  (*os) << i << '\n';
}

int main() {
  std::map<std::string,std::vector<int> > m;
  m["Strange?"].push_back(1);
  m["Strange?"].push_back(2);
  m["Strange?"].push_back(3);
  m["Weird?"].push_back(4);
  m["Weird?"].push_back(5);

  std::for_each(m.begin(),m.end(),
    boost::bind(&print,&std::cout,
      boost::bind(&std::vector<int>::size,
        boost::bind(
          &std::map<std::string,
            std::vector<int> >::value_type::second,_1))));
} 
```

上面这段代码实际上做了什么？有的人可以流畅地阅读这段代码[6]，但对于我们多数人来说，需要一些时间才能搞清楚它是干嘛的。是的，绑定器对 `pair` (即 `std::map&lt;std::string,std::vector&lt;int&gt; &gt;::value_type`)的成员 `second` 调用成员函数 `size` 。这种情况下，简单的问题被绑定器弄得复杂了，创建一个小的函数对象来取代这个让人难以理解的复杂绑定器是更好的选择。一个可以完成相同工作的简单函数对象如下：

> [6] 你好，Peter Dimov.

```cpp
class print_size {
  std::ostream& os_;
  typedef std::map<std::string,std::vector<int> > map_type;
public:
  print_size(std::ostream& os):os_(os) {}

  void operator()(
    const map_type::value_type& x) const {
    os_ << x.second.size() << '\n';
  }
}; 
```

这种时候使用函数对象的最大好处就是，名字是无需加以说明的。

```cpp
std::for_each(m.begin(),m.end(),print_size(std::cout)); 
```

我们把这些(函数对象以及实际调用的所有代码)和前面使用绑定器的版本作一下比较。

```cpp
std::for_each(m.begin(),m.end(),
  boost::bind(&print,&std::cout,
    boost::bind(&std::vector<int>::size,
      boost::bind(
        &std::map<std::string,
          std::vector<int> >::value_type::second,_1)))); 
```

或者，如果我们负点责任，为 `vector` 和 `map` 分别创建一个简洁的 `typedef` ：

```cpp
std::for_each(m.begin(),m.end(),
  boost::bind(&print,&std::cout,
    boost::bind(&vec_type::size,
      boost::bind(&map_type::value_type::second,_1)))); 
```

这样可以容易点分析，但它还是有点长。

虽然使用 `bind` 版本是有一些好理由，但我想观点是很清楚的，绑定器不是非用不可的工具，使用时应该负责任，要让它们物有所值。这一点在使用标准库的容器和算法时非常、非常普遍。当事情变得太过复杂时，就回到老风格的方法上。

### 让绑定器把握状态

创建一个象 `print_size` 那样的函数对象时，有几个选项可用。我们在上一节中创建的那个版本中，保存了一个到 `std::ostream` 的引用，并使用这个 `ostream` 来打印 `map_type::value_type` 参数的成员 `second` 的 `size` 函数的返回值。以下是原来的 `print_size` ：

```cpp
class print_size {
  std::ostream& os_;
  typedef std::map<std::string,std::vector<int> > map_type;
public:
  print_size(std::ostream& os):os_(os) {}

  void operator()(
    const map_type::value_type& x) const {
    os_ << x.second.size() << '\n';
 }
}; 
```

要重点关注的一点是，这个类是有状态的，状态就在于那个保存的 `std::ostream`. 我们可以通过向调用操作符增加一个 `ostream` 参数来去掉这个状态。这意味着这个函数对象将变为无状态的。

```cpp
class print_size {
  typedef std::map<std::string,std::vector<int> > map_type;
public:
  typedef void result_type;
  result_type operator()(std::ostream& os,
    const map_type::value_type& x) const {
    os << x.second.size() << '\n';
  }
}; 
```

注意，这个版本的 `print_size` 可以很好地用于 `bind`, 因为它增加了一个 `result_type typedef`. 这样用户在使用 `bind` 时就不需要显式声明函数对象的返回类型。在这个新版本的 `print_size` 里，用户需要传递一个 `ostream` 参数来调用它。这在使用绑定器时是很容易的。用这个新的 `print_size` 重写前节中的例子，我们可以得到：

```cpp
#include <iostream>
#include <string>
#include <map>
#include <vector>
#include <algorithm>
#include "boost/bind.hpp"

// 省略 print_size 的定义

int main() {
  typedef std::map<std::string,std::vector<int> > map_type;
  map_type m;
  m["Strange?"].push_back(1);
  m["Strange?"].push_back(2);
  m["Strange?"].push_back(3);
  m["Weird?"].push_back(4);
  m["Weird?"].push_back(5);

  std::for_each(m.begin(),m.end(),
    boost::bind(print_size(),boost::ref(std::cout),_1));
} 
```

细心的读者可能觉得为什么 `print_size` 不是一个普通函数，毕竟它已经不带有任何状态了。事实上，它可以是普通函数。

```cpp
void print_size(std::ostream& os,
  const std::map<std::string,std::vector<int> >::value_type& x) {
  os << x.second.size() << '\n';
} 
```

还有更多的泛化工作可以做。我们当前版本的 `print_size` 要求其调用操作符的第二个参数是一个 `const std::map&lt;std::string,std::vector&lt;int&gt; &gt;` 引用，这不够通用。我们可以做得更好一些，让调用操作符对这个类型进行泛化。这样，`print_size` 就可以使用任意类型的参数，只要该参数含有名为 `second` 的公有成员，并且该成员有一个成员函数 `size`. 以下是改进后的版本：

```cpp
class print_size {
public:
  typedef void result_type;
  template <typename Pair> result_type operator()
    (std::ostream& os,const Pair& x) const {
    os << x.second.size() << '\n';
  }
}; 
```

这个版本的用法与前一个是一样的，但它更为灵活。在创建可用于 `bind` 表达式的函数对象时，这种泛化更为重要。因为这样的函数对象可用的情形将显著增加，多数潜在的泛化都是值得做的。既然如此，我们还可以进一步放松对使用 `print_size` 的类型的要求。当前版本的 `print_size` 要求调用操作符的第二个参数是一个类似于 pair 的对象，即一个含有名为 `second` 的成员的对象。如果我们决定只要求这个参数含有成员函数 `size`, 这个函数对象就真的与它的名字相符了。

```cpp
class print_size {
public:
  typedef void result_type;
  template <typename T> void operator()
    (std::ostream& os,const T& x) const {
    os << x.size() << '\n';
  }
}; 
```

当然，尽管 `print_size` 现在是与它的名字相符了，但是我们也要求用户要做的更多了。象对于我们前面的例子，就需要手工绑定一个 `map_type::value_type::second`.

```cpp
std::for_each(m.begin(),m.end(),
  boost::bind(print_size(),boost::ref(std::cout),
    boost::bind(&map_type::value_type::second,_1))); 
```

在使用 `bind` 时，通常都需要这样的折衷，泛化只能到此为止，不要损害到可用性。如果我们走到极端，甚至去掉对成员函数 `size` 的要求，那么我们就转了一圈，回到了我们开始的地方，又回到那个对多数程序员而言都过于复杂的 `bind` 表达式了。

```cpp
std::for_each(m.begin(),m.end(),
  boost::bind(&print<sup class="docfootnote">\[7\]</sup>,&std::cout,
    boost::bind(&vec_type::size,
      boost::bind(&map_type::value_type::second,_1)))); 
```

> [7] `print` 函数显然也是需要的，没有 lambda 工具。

### 关于 Boost.Bind 和 Boost.Function

虽然本章中讨论的内容应该没有遗漏了，但是对于 Boost.Bind 和另一个库，Boost.Function，之间的配合还是值得一提，它可以提供更多的功能。我们将在 "Library 11:Function 11" 看到，不过我还是想给你一些提示。正如我们所看到的，没有一个明显的方法来保存我们的绑定器以备后用，我们只知道它们是带有某些(未知)的特征的兼容函数 对象。但是，如果使用 Boost.Function, 保存函数用于以后的调用正是那个库要做的，并且它兼容于 Boost.Bind, 可以把绑定器赋值给函数，保存它们并用于以后的调用。这是一个非常有用的概念，它可以用于适配并提高了松耦合。

# Bind 总结

## Bind 总结

在以下情形时使用 Bind ：

*   你需要绑定一个调用到一个普通函数，使用部分或全部参数

*   你需要绑定一个调用到一个成员函数，使用部分或全部参数

*   你需要嵌套组合函数对象

泛化绑定器的存在对于编写简洁、连贯的代码非常有用。它减少了为了适配函数/函数对象以及函数组合而创建的小函数对象的数量。虽然标准库已经提供了 Boost.Bind 的一小部分功能，但是 Boost.Bind 所具有的重大改进使得它在多数情况下成为了更好的选择。除了对已有功能进行简化外，Bind 还提供了强大的函数组合功能，这为程序员提供了强大的力量而且没有维护上的负作用。如果你已经花了时间学习 `bind1st`, `bind2nd`, `ptr_fun`, `mem_fun_ref`, 等等，那么转换到 Boost.Bind 对你而言几乎没有困难。如果你已经开始使用 C++标准库所提供的绑定器，我强烈建议你开始使用 Bind, 因为它更容易学习，而且更强大。

我知道许多程序员通常都有绑定器的经验，特别是函数组合。如果你用过其中之一，我希望本章能够为你提 供某些动力，推动你更进一步。此外，回想一下这种就地声明并定义的函数意味着什么，它意味着无需维护。仅仅为了提供正确的署名和执行简单的小任务而在类的 周围创建一堆小的、看起来很简单的[8]函数对象，会导致代码的分散，与之相比，使用绑定器更容易。

> [8] 但它们并不是那么简单。

Boost.Bind 库由 Peter Dimov 创建并维护，他除了令这个库实现了完整的绑定和函数组合功能之外，还令它可以很好地工作在多数编译器环境下。