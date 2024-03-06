# Library 11\. Function

# 11\. Function

*   Function 库如何改进你的程序？
*   Function 如何适用于标准库？
*   Function
*   用法
*   Function 总结

# Function 库如何改进你的程序？

## Function 库如何改进你的程序？

*   保存函数指针和函数对象，用于后续的调用

在含有回调的设计中，常常需要保存函数和函数对象，而且某些函数或类也是 通过函数指针或函数对象来配 制其客户化功能。传统上，通常使用函数指针来实现回调及延迟调用的函数。但是，仅仅使用函数指针会有很多限制，更好的方法是采用泛型机制来定义要被保存的函数的署名特 征，而让调用者来决定提供哪一种类型的函数实体(函数指针或 函数对象)。这样就可以使用任何行为类似于函数的实体，例 如，使用 Boost.Bind 和 Boost.Lambda 所返回的结果。这意味着可以给这些被保存的函数 增加状态(因为函数对象是一种类)。这种泛化由 Boost.Function 提供。这个库用于保存并然后调用函数或函数对象。

# Function 如何适用于标准库？

## Function 如何适用于标准库？

本库提供了当前标准库所不具备的功能。在那些对商业逻辑的表示层进行 解耦的框架中，使用泛型回调是非常自然的、常见的。由于 C++标 准库不支持保存函数指针和函数对象以供稍后的调用，因此这个工具为标准库提供了非常重要的扩展。而且，本库完全兼容于标准库的绑定器(`bind1st` 和 `bind2nd`)，就象前面所 讨论过的其它绑定器一样，如 Boost.Bind 和 Boost.Lambda.

# Function

## Function

### 头文件: `"boost/function.hpp"`

头文件 `"function.hpp"` 包含了带有从 0 到 10 个参数的函数原型(这是实现所定义的，在当前实现中缺省的上限就是 10[1])。你也可以根据你的需要，只包含相应参数数量的头文件，文件名为 `"function/functionN.hpp"`, 其中 N 可以从 0 到 10。Boost.Function 有两种不同的接口，其中一种的好处在于它的语法接近于函数声明(而且不要求函数的签名包含参数的数量)，另一种接口的好处在于可以在多个编译器中工作。选择哪一种接口，至少部分地取决于你使用的编译器。如果可以，就使用我们称为首选语法(preferred syntax)的那种。在本章中，两种格式都会用到。

> [1] Boost.Function 可以配置为支持多达 127 个参数。

### 使用首选语法的声明

一个 `function` 的声明包括该 `function` 所兼容的函数或函数对象的签名以及返回类型。结果以及参数的类型以单个参数的方式全部提供给模板。例如，声明一个 `function` ，它返回 `bool` 并接受一个类型 `int` 的参数，如下：

```cpp
boost::function<bool (int)> f; 
```

可以在括号中给出参数列表，以逗号分隔，就象普通的函数声明一样。所以，声明一个没有返回值(`void`)并带有类型分别为 `int` 和 `double` 的两个参数的函数，就象这样：

```cpp
boost::function<void (int,double)> f; 
```

### 使用兼容语法的声明

声明 `function`s 的第二种方法是，分别给出函数调用的返回类型及参数类型作为模板类型参数。并且，要在 `function` 类的名字中加上后缀，后缀是一个表示 `function` 可接受的参数数量的整数。例如，声明一个返回 `bool` 并接受一个类型 `int` 的参数的函数，方法如下：

```cpp
boost::function1<bool,int> f; 
```

这里的数字是对应函数可接受的参数数量，在上例中有一个参数(`int`)，所以是 `function1` 。更多的参数就意味着要给出更多的模板类型参数并且改变这个数字后缀。一个 `function` ，返回 `void` 并接受类型为 `int` 和 `double` 的两个参数，如下：

```cpp
boost::function2<void,int,double> f; 
```

事实上，这个库由一组类组成，其中每个类带有不同数量的参数。如果包含头文件 `"function.hpp"` 就无需关心这一点，但如果包含带数字的头文件，你就必须包含正确数字的头文件。

首选语法更容易阅读，也更象在声明一个函数，所以你应该尽可能使用它。不幸的是，虽然首选语法是完全 合法的 C++并且更易读，但是还不是所有编译器都可以支持它。如果你的编译器正好不能处理这种语法，你就必须使用另一种格式。如果你需要编写最大兼容性的 代码，你也只能选择使用另一种格式。让我们来看一下 `function` 的接口中最重要的部分。

### 成员函数

```cpp
function(); 
```

缺省构造函数创建一个空的函数对象。如果一个空的 `function` 被调用，将会抛出一个类型为 `bad_function_call` 的异常。

```cpp
template <typename F> function(F g); 
```

这个泛型的构造函数接受一个兼容的函数对象，即这样一个函数或函数对象，它的返回类型与被构造的 `function` 的返回类型或者一样，或者可以隐式转换，并且它的参数也要与被构造的 `function` 的参数类型或者一样，或者可以隐式转换。注意，也可以使用另外一个 `function` 实例来进行构造。如果这样做，并且 `function f` 为空，则被构造的 function 也为空。使用空的函数指针和空的成员函数指针也会产生空的 `function` 。

```cpp
template <typename F> function(reference_wrapper<F> g); 
```

这个构造函数与前一个类似，但它接受的函数对象包装在一个 `reference_wrapper` 中，这用以避免通过值来传递而产生函数或函数对象的一份拷贝。这同样要求函数对象兼容于 `function` 的签名。

```cpp
function& operator=(const function& g); 
```

赋值操作符保存 `g` 中的函数或函数对象的一份拷贝；如果 `g` 为空，被赋值的函数也将为空。

```cpp
template<typename F> function& operator=(F g); 
```

这个泛型赋值操作符接受一个兼容的函数指针或函数对象。注意，也可以用另一个 `function` 实例(带有不同但兼容的签名)来赋值。这同样意味着，如果 `g` 是另一个 `function` 实例且为空，则赋值后的函数也为空。赋值一个空的函数指针或空的成员函数指针也会使 `function` 为空。

```cpp
bool empty() const; 
```

这个成员函数返回一个布尔值，表示该 function 是含有一个函数/函数对象或是为空。如果有一个目标函数或函数对象可被调用，它返回 `false` 。因为一个 function 可以在一个布尔上下文中测试，或者与 0 进行比较，因此这个成员函数可能会在未来版本的库中被取消，你应该避免使用它。

```cpp
void clear(); 
```

这个成员函数清除 `function`, 即它不再关联到一个函数或函数对象。如果 function 已经是空的，这个调用没有影响。在调用后，function 肯定为空。令一个 function 为空的首选方法是赋 0 给它；`clear` 可能在未来版本的库中被取消。

```cpp
operator safe_bool() const 
```

这个转型函数返回一个未指定类型(由 `safe_bool` 表示)，它可以用于布尔上下文中。如果 `function` 为空，返回值为 `false`. 如果 function 中保存了一个函数指针或函数对象，则返回值为 `true`. 注意，使用一个不同于 `bool` 的类型，可以使得这个转型函数十分安全且不会被重载干扰，同时还提供了直接在布尔上下文中测试 `function` 实例的惯用法。它等同于表达式 `!!f`, 其中 `f` 为一个 `function` 实例。

```cpp
result_type operator()(Arg1 a1, Arg2 a2, ..., ArgN aN) const; 
```

调用操作符是调用 `function` 的方法。你不能调用一个空的 `function` ，那样会抛出一个 `bad_function_call` 异常，即当 `!f.empty()`, `if (f)`, 或 `if (!!f)` 返回 `true` 时。调用操作符的执行会调用 `function` 中的函数或函数对象，并返回它的结果。

# 用 法

## 用 法

要开始使用 Boost.Function, 就要包含头文件 `"boost/function.hpp"`, 或者某个带数字的版本，从 `"boost/function/function0.hpp"` 到 `"boost/function/function10.hpp"`. 如果你知道你想保存在 `function` 中的函数的参数数量，这样做可以让编译器仅包含需要的头文件。如果包含 `"boost/function.hpp"`, 那么就会把其它的头文件也包含进去。

理解被存函数的最佳方法是把它想象为一个普通的函数对象，该函数对象用于封装另一个函数(或函数对象)。这个被存的函数的最大用途是它可以被多次调用，而无须在创建 `function` 时立即使用。在声明 `function`s 时，声明中最重要的部分是函数的签名。这部分即是告诉 `function` 它将保存的函数或函数对象的签名和返回类型。我们已经看到，有两种方法来执行这个声明。这里有一个完整的程序，程序声明了一个 `boost::function` ，它可以保存返回 `bool` (或某个可以隐式转换为 `bool` 的类型)并接受两个参数的类函数实体，第一个参数可以转换为 `int`, 第二个参数可以转换为 `double`.

```cpp
#include <iostream>
#include "boost/function.hpp"

bool some_func(int i,double d) {
  return i>d;
}

int main() {
  boost::function<bool (int,double)> f;
  f=&some_func;
  f(10,1.1);
} 
```

当 `function f` 首次创建时，它不保存任何函数。它是空的，可以在一个布尔上下文中进行测试。如果你试图调用一个没有保存任何函数或函数对象的 `function` ，它将抛出一个类型 `bad_function_call` 的异常。为了避免这个问题，我们用普通的赋值语法把一个指向 `some_func` 的指针赋值给 `f` 。这导致 `f` 保存了到 `some_func` 的指针。最后，我们用参数 10 (一个 `int`) 和 1.1 (一个 `double`)来调用 `f` (用函数调用操作符)。要调用一个 `function`, 你必须提供被存函数或函数对象所期望的准确数量的参数。

### 回调的基础

我们先来看看在没有 Boost.Function 以前我们如何实现一个简单的回调，然后再把代码改为使用 `function`, 并看看会带来什么优势。我们从一个支持某种简单的回调形式的类开始，它可以向任何对新值关注的对象报告值的改变。这里的回调是一种传统的 C 风格回调，即使 用普通函数。这种回调用可用于象 GUI 控制这样的场合，它可以通知观察者用户改变了它的值，而不需要对监听该信息的客户有任何特殊的知识。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include "boost/function.hpp"

void print_new_value(int i) {
  std::cout << 
    "The value has been updated and is now " << i << '\n';
}

void interested_in_the_change(int i) {
  std::cout << "Ah, the value has changed.\n";
}

class notifier {
  typedef void (*function_type)(int);
  std::vector<function_type> vec_;
  int value_;
public:
  void add_observer(function_type t) {
    vec_.push_back(t);
  }

  void change_value(int i) {
    value_=i;
    for (std::size_t i=0;i<vec_.size();++i) {
      (*vec_[i])(value_);
    }
  }
};

int main() {
  notifier n;
  n.add_observer(&print_new_value);
  n.add_observer(&interested_in_the_change);

  n.change_value(42);
} 
```

这里的两个函数，`print_new_value` 和 `interested_in_the_change`, 它们的函数签名都兼容于 `notifier` 类的要求。这些函数指针被保存在一个 `vector` 内，并且无论何时它的值被改变，这些函数都会在一个循环里被调用。调用这些函数的一种语法是：

```cpp
(*vec_[i])(value_); 
```

值(`value_`)被传递给解引用的函数指针(即 `vec_[i]` 所返回的)。另一种写法也是有效的，即这样：

```cpp
vec_i; 
```

这种写法看起来更好看些，但更为重要的是，它还可以允许你把函数指针更换为 Boost.Function 而没有改变调用的语法。现在，工作还是正常的，但是，唉，函数对象不能用于这个 `notifier` 类。事实上，除了函数指针以外，别的任何东西都不能用，这的确是一种局限。但是，如果我们使用 Boost.Function，它就可以工作。重写这个 `notifier` 类非常容易。

```cpp
class notifier {
  typedef boost::function<void(int)> function_type;
  std::vector<function_type> vec_;
  int value_;
public:
  template <typename T> void add_observer(T t) {
    vec_.push_back(function_type(t));
  }

  void change_value(int i) {
    value_=i;
    for (std::size_t i=0;i<vec_.size();++i) {
      vec_i;
    }
  }
}; 
```

首先要做的事是，把 `typedef` 改为代表 `boost::function` 而不是函数指针。之前，我们定义的是一个函数指针；现在，我们使用泛型方法，很快就会看到它的用途。接着，我们把成员函数 `add_observer` 的签名改为泛化的参数类型。我们也可以把它改为接受一个 `boost::function`，但那样会要求该类的用户必须也知道 `function` 的使用方法[2]，而不是仅仅知道这个观察者类型的要求就行了。应该注意到 `add_observer` 的这种变化并不应该是转向 `function` 的结果；无论如何代码应该可以继续工作。我们把它改为泛型的；现在，不管是函数指针、函数对象，还是 `boost::function` 实例都可以被传递给 `add_observer`, 而无须对已有用户代码进行任何改动。把元素加入到 `vector` 的代码有一些修改，现在需要创建一个 `boost::function&lt;void(int)&gt;` 实例。最后，我们把调用这些函数的语法改为可以使用函数、函数对象以及 `boost::function` 实例[3]。这种对不同类型的类似函数的"东西"的扩展支持可以立即用于带状态的函数对象，它们可以实现一些用函数很难做到的事情。

> [2] 他们应该知道 Boost.Function,但如果他们不知道呢？我们添加到接口上的任何东西都必须及时向用户解释清楚。
> 
> [3] 现在我们知道，一开始我们就应该用这种语法。

```cpp
class knows_the_previous_value {
  int last_value_;
public:
  void operator()(int i) {
    static bool first_time=true;
    if (first_time) {
      last_value_=i;
      std::cout << 
        "This is the first change of value, \
so I don't know the previous one.\n";
      first_time=false;
      return;
    }
    std::cout << "Previous value was " << last_value_ << '\n';
    last_value_=i;
  }
}; 
```

这个函数对象保存以前的值，并在值被改变时把旧值输出到 `std::cout` 。注意，当它第一次被调用时，它并不知道旧值。这个函数对象在函数中使用一个静态 `bool` 变量来检查这一点，该变量被初始化为 `true`. 由于函数中的静态变量是在函数第一次被调用时进行初始化的，所以它仅在第一次调用时被设为 `true` 。虽然也可以在普通函数中使用静态变量来提供状态，但是我们必须知道那样不太好，而且很难做到多线程安全。因此，带状态的函数对象总是优于带静态变量的普通函数。`notifier` 类并不关心这是不是函数对象，只要符合要求就可以接受。以下更新的例子示范了它如何使用。

```cpp
int main() {
  notifier n;
  n.add_observer(&print_new_value);
  n.add_observer(&interested_in_the_change);
  n.add_observer(knows_the_previous_value());

  n.change_value(42);
  std::cout << '\n';
  n.change_value(30);
} 
```

关键一点要注意的是，我们新增的一个观察者不是函数指针，而是一个 `knows_the_previous_value` 函数对象的实例。运行这段程序的输出如下：

```cpp
The value has been updated and is now 42
Ah, the value has changed.
This is the first change of value, so I don't know the previous one.

The value has been updated and is now 30
Ah, the value has changed.
Previous value was 42 
```

在这里最大的优点不是放宽了对函数的要求(或者说，增加了对函数对象的支持)，而是我们可以使用带状态的对象，这是非常需要的。我们对 `notifier` 类所做的修改非常简单，而且用户代码不受影响。如上所示，把 Boost.Function 引入一个已有的设计中是非常容易的。

### 类成员函数

Boost.Function 不支持参数绑定，这在每次调用一个 `function` 就要调用同一个类实例的成员函数时是需要的。幸运的是，如果这个类实例被传递给 `function` 的话，我们就可以直接调用它的成员函数。这个 `function` 的签名必须包含类的类型以及成员函数的签名。换言之，显式传入的类实例要作为隐式的第一个参数，`this`。这样就得到了一个在给出的对象上调用成员函数的函数对象。看一下以下这个类：

```cpp
class some_class {
public:
  void do_stuff(int i) const {
    std::cout << "OK. Stuff is done. " << i << '\n';
  }
}; 
```

成员函数 `do_stuff` 要从一个 `boost::function` 实例里被调用。要做到这一点，我们需要 function 接受一个 `some_class` 实例，签名的其它部分为一个 `void` 返回以及一个 `int` 参数。对于如何把 `some_class` 实例传给 function，我们有三种选择：传值，传引用，或者传址。如何要传值，代码就应该这样写[4]

> [4] 很少会有理由来以传值的方式传递对象参数。

```cpp
boost::function<void(some_class,int)> f; 
```

注意，返回类型仍旧在最开始，后跟成员函数所在的类，最后是成员函数的参数类型。它就象传递一个 `this` 给一个函数，该函数暗地里用类实例调用一个非成员函数。要把函数 `f` 配置为成员函数 `do_stuff`, 然后调用它，我们这样写：

```cpp
f=&some_class::do_stuff;
f(some_class(),2); 
```

如果要传引用，我们要改一下函数的签名，并传递一个 `some_class` 实例。

```cpp
boost::function<void(some_class&,int)> f;
f=&some_class::do_stuff;
some_class s;
f(s,1); 
```

最后，如果要传 `some_class` 的指针[5]，我们就要这样写：

> [5] 裸指针或智能指针皆可。

```cpp
boost::function<void(some_class*,int)> f;
f=&some_class::do_stuff;
some_class s;
f(&s,3); 
```

好了，所有这些传递"虚拟 `this`"实例的方法都已经在库中提供。当然，这种技术也是有限制的：你必须显式地传递类实例；而理想上，你更愿意这个实例被绑定在函数中。乍一看，这似乎是 Boost.Function 的缺点，但有别的库可以支持参数的绑定，如 Boost.Bind 和 Boost.Lambda. 我们将在本章稍后的地方示范这些库会给 Boost.Function 带有什么好处。

### 带状态的函数对象

我们已经看到，由于支持了函数对象，就可以给回调函数增加状态。考虑这样一个类，`keeping_state`, 它是一个带状态的函数对象。`keeping_state` 的实例记录一个总和，它在每次调用操作符执行时被增加。现在，将该类的一个实例用于两个 `boost::function` 实例，结果有些出人意外。

```cpp
#include <iostream>
#include "boost/function.hpp"

class keeping_state {
  int total_;
public:
  keeping_state():total_(0) {}

  int operator()(int i) {
    total_+=i;
    return total_;
  }

  int total() const {
    return total_;
  }
};

int main() {
  keeping_state ks;
  boost::function<int(int)> f1;
  f1=ks;

  boost::function<int(int)> f2;
  f2=ks;

  std::cout << "The current total is " << f1(10) << '\n';
  std::cout << "The current total is " << f2(10) << '\n';
  std::cout << "After adding 10 two times, the total is " 
    << ks.total() << '\n';
} 
```

写完这段代码并接着执行它，程序员可能期望保存在 `ks` 的总和是 20，但不是；事实上，总和为 0。以下是这段程序的运行结果。

```cpp
The current total is 10
The current total is 10
After adding 10 two times, the total is 0 
```

原因是每一个 `function` 实例(`f1` 和 `f2`)都含有一个 `ks` 的拷贝，这两个实例得到的总和都是 10，但 `ks` 没有变化。这可能是也可能不是你想要的，但是记住，`boost::function` 的缺省行为是复制它要调用的函数对象，这一点很重要。如果这导致不正确的语义，或者如果某些函数对象的复制代价太高，你就必须把函数对象包装在 `boost::reference_wrapper` 中，那样 `boost::function` 的复制就会是一个 `boost::reference_wrapper` 的拷贝，它恰好持有一个到原始函数对象的引用。你无须直接使用 `boost::reference_wrapper` ，你可以使用另两个助手函数，`ref` 和 `cref`。 这两函数返回一个持有到某特定类型的引用或 `const` 引用的 `reference_wrapper`。在前例中，要获得我们想要的语义，即使用同一个 `keeping_state` 实例，我们就需要把代码修改如下：

```cpp
int main() {
  keeping_state ks;
  boost::function<int(int)> f1;
  f1=boost::ref(ks);

  boost::function<int(int)> f2;
  f2=boost::ref(ks);

  std::cout << "The current total is " << f1(10) << '\n';
  std::cout << "The current total is " << f2(10) << '\n';
  std::cout << "After adding 10 two times, the total is " 
    << ks.total() << '\n';
} 
```

`boost::ref` 的用途是通知 `boost::function`，我们想保存一个到函数对象的引用，而不是一个拷贝。运行这个程序有以下输出：

```cpp
The current total is 10
The current total is 20
After adding 10 two times, the total is 20 
```

这正是我们想要的结果。使用 `boost::ref` 和 `boost::cref` 的不同之处就象引用与 `const` 引用的差异，对于后者，你只能调用其中的常量成员函数。以下例子使用一个名为 `something_else` 的函数对象，它有一个 `const` 的调用操作符。

```cpp
class something_else {
public:
  void operator()() const {
    std::cout << "This works with boost::cref\n";
  }
}; 
```

对于这个函数对象，我们可以使用 `boost::ref` 或 `boost::cref`.

```cpp
something_else s;
boost::function0<void> f1;
f1=boost::ref(s);
f1();
boost::function0<void> f2;
f2=boost::cref(s);
f2(); 
```

如果我们改变了 `something_else` 的实现，使其函数为非`const`, 则只有 `boost::ref` 可以使用，而 `boost::cref` 将导致一个编译期错误。

```cpp
class something_else {
public:
  void operator()() {
    std::cout << 
      "This works only with boost::ref, or copies\n";
  }
};

something_else s;
boost::function0<void> f1;
f1=boost::ref(s); // This still works
f1(); 
boost::function0<void> f2;
f2=boost::cref(s); // This doesn't work; 
                   // the function call operator is not const
f2(); 
```

如果一个 `function` 包含一个被 `boost::reference_wrapper` 所包装的函数对象，那么复制构造函数与赋值操作就会复制该引用，即 `function` 的拷贝将引向原先的函数对象。

```cpp
int main() {
  keeping_state ks;
  boost::function1<int,int> f1;  // 译注：原文为 boost::function<int,int> f1，有误
  f1=boost::ref(ks);

  boost::function1<int,int> f2(f1);  // 译注：原文为 boost::function<int,int> f2(f1)，有误 
  boost::function1<short,short> f3;  // 译注：原文为 boost::function<short,short> f3，有误 
  f3=f1;

  std::cout << "The current total is " << f1(10) << '\n';
  std::cout << "The current total is " << f2(10) << '\n';
  std::cout << "The current total is " << f3(10) << '\n';
  std::cout << "After adding 10 three times, the total is " 
    << ks.total() << '\n';
} 
```

这等同于使用 `boost::ref` 并把函数对象 `ks` 赋给每一个 function 实例。

给回调函数增加状态，可以发挥巨大的能力，这也正是使用 Boost.Function 与使用函数对象相比具有的非常突出的优点。

### 与 Boost.Function 一起使用 Boost.Bind

当我们把 Boost.Function 与某个支持参数绑定的库结合起来使用时，事情变得更为有趣。Boost.Bind 为普通函数、成员函数以及成员变量提供参数绑定。这非常适合于 Boost.Function, 我们常常需要这类绑定，由于我们使用的类本身并不是函数对象。那么，我们用 Boost.Bind 把它们转变为函数对象，然后我们可以用 Boost.Function 来保存它们并稍后调用。在将图形用户界面(GUIs)与如何响应用户的操作进行分离时，几乎总是要使用某种回调方法。如果这种回调机制是基于函数指针的， 就很难避免对可以使用回调的类型的某些限制，也就增加了界面表现与业务逻辑之间的耦合风险。通过使用 Boost.Function，我们可以避免这些事情，并且当与某个支持参数绑定的库结合使用时，我们可以轻而易举地把上下文提供给调用的函数。这是本库 最常见的用途之一，把业务逻辑即从表示层分离出来。

以下例子包含一个艺术级的磁带录音机，定义如下：

```cpp
class tape_recorder {
public:
  void play() {
    std::cout << "Since my baby left me...\n";
  }

  void stop() {
    std::cout << "OK, taking a break\n";
  }

  void forward() {
    std::cout << "whizzz\n";
  }

  void rewind() {
    std::cout << "zzzihw\n";
  }

  void record(const std::string& sound) {
    std::cout << "Recorded: " << sound << '\n';
  }
}; 
```

这个磁带录音机可以从一个 GUI 进行控制，或者也可能从一个脚本客户端进行控制，或者从别的源进行控 制，这意味着我们不想把这些函数的执行与它们的实现耦合起来。建立这种分离的一个常用的方法是，用专门的对象负责执行命令，而让客户对命令如何执行毫无所 知。这也被称为命令模式(Command pattern)，并且在它非常有用。这种模式的特定实现中的一个问题是，需要为每个命令创建单独的类。以下片断示范了它看起来是个什么样子：

```cpp
class command_base {
public:
  virtual bool enabled() const=0;
  virtual void execute()=0;

  virtual ~command_base() {}
};

class play_command : public command_base {
  tape_recorder* p_;
public:
  play_command(tape_recorder* p):p_(p) {}

  bool enabled() const {
    return true;
  }

  void execute() {
    p_->play();
  }
};

class stop_command : public command_base {
  tape_recorder* p_;
public:
  stop_command(tape_recorder* p):p_(p) {}

  bool enabled() const {
    return true;
  }

  void execute() {
    p_->stop();
  }
}; 
```

这并不是一个非常吸引的方案，因为它使得代码膨胀，有许多简单的命令类，而它们只是简单地负责调用一个对象的单个成员函数。有时候，这是必需的，因为这些命令可能需要实现业务逻辑和调用函数，但通常它只是由于我们所使用的工具有所限制而已。这些命令类可以这样使用：

```cpp
int main() {
  tape_recorder tr;

  // 使用命令模式
  command_base* pPlay=new play_command(&tr);
  command_base* pStop=new stop_command(&tr);

  // 在按下某个按钮时调用
  pPlay->execute();
  pStop->execute();

  delete pPlay;
  delete pStop;
} 
```

现在，不用再创建额外的具体的命令类，如果我们实现的命令都是调用一个返回 `void` 且没有参数(先暂时忽略函数 record, 它带有一个参数)的成员函数的话，我们可以来点泛化。不用再创建一组具体的命令，我们可以在类中保存一个指向正确成员函数的指针。这是迈向正确方向[6]的一大步，就象这样：

> [6] 虽然损失了一点效率。

```cpp
class tape_recorder_command : public command_base {
  void (tape_recorder::*func_)(); 
  tape_recorder* p_;
public:

  tape_recorder_command(
    tape_recorder* p,
    void (tape_recorder::*func)()) : p_(p),func_(func) {}

  bool enabled() const {
    return true;
  }

  void execute() {
    (p_->*func_)();
  }
}; 
```

这个命令模式的实现要好多了，因为它不需要我们再创建一组完成相同事情的独立的类。这里的不同在于我们保存了一个 `tape_recorder` 成员函数指针在 `func_` 中，它要在构造函数中提供。命令的执行部分可能并不是你要展现给你的朋友看的东西，因为成员指针操作符对于一些人来说可能还不太熟悉。但是，这可以被看为一个低层的实现细节，所以还算好。有了这个类，我们可以进行泛化处理，不再需要实现单独的命令类。

```cpp
int main() {
  tape_recorder tr;

  // 使用改进的命令模式
  command_base* pPlay=
    new tape_recorder_command(&tr,&tape_recorder::play);
  command_base* pStop=
    new tape_recorder_command(&tr,&tape_recorder::stop);

  // 从一个 GUI 或一个脚本客户端进行调用
  pPlay->execute();
  pStop->execute();

  delete pPlay;
  delete pStop;
} 
```

你可能还没有理解，我们已经在开始实现一个简单的 `boost::function` 版本，它已经可以做到我们想要的。不要重复发明轮子，让我们重点关注手边的工作：分离调用与实现。以下是一个全新实现的 `command` 类，它更容易编写、维护以及理解。

```cpp
class command {
  boost::function<void()> f_;
public:
  command() {}
  command(boost::function<void()> f):f_(f) {}

  void execute() {
    if (f_) {
      f_();
    }
  }

  template <typename Func> void set_function(Func f) {
    f_=f;
  }

  bool enabled() const {
    return f_;
  }
}; 
```

通过使用 Boost.Function，我们可以立即从同时兼容函数和函数对象——包括由绑定器生成的函数对象——的灵活性之中获益。这个 `command` 类把函数保存在一个返回 `void` 且不接受参数的 `boost::function` 中。为了让这个类更加灵活，我们提供了在运行期修改函数对象的方法，使用一个泛型的成员函数，`set_function`.

```cpp
template <typename Func> void set_function(Func f) {
  f_=f;
} 
```

通过使用泛型方法，任何函数、函数对象，或者绑定器都兼容于我们的 `command` 类。我们也可以选择把 `boost:: function` 作为参数，并使用 `function` 的转型构造函数来达到同样的效果。这个 `command` 类非常通用，我们可以把它用于我们的 `tape_recorder` 类或者别的地方。与前面的使用一个基类与多个具体派生类(在那里我们使用指针来实现多态的行为)的方法相比，还有一个额外的优点就是，它更容易管理生存期问题，我们不再需要删除命令对象，它们可以按值传递和保存。我们在布尔上下文中使用 `function f_` 来测试命令是否可用。如果函数不包含一个目标，即一个函数或函数对象，它将返回 `false`, 这意味着我们不能调用它。这个测试在 `execute` 的实现中进行。以下是使用我们这个新类的一个例子：

```cpp
int main() {
  tape_recorder tr;

  command play(boost::bind(&tape_recorder::play,&tr));
  command stop(boost::bind(&tape_recorder::stop,&tr));
  command forward(boost::bind(&tape_recorder::stop,&tr));
  command rewind(boost::bind(&tape_recorder::rewind,&tr));
  command record;

  // 从某些 GUI 控制中调用...
  if (play.enabled()) {
    play.execute();
  }

  // 从某些脚本客户端调用...
  stop.execute();

  // Some inspired songwriter has passed some lyrics
  std::string s="What a beautiful morning...";
  record.set_function(
    boost::bind(&tape_recorder::record,&tr,s));
  record.execute();
} 
```

为了创建一个具体的命令，我们使用 Boost.Bind 来创建函数对象，当通过这些对象的调用操作符进行调用时，就会调用正确的 `tape_recorder` 成员函数。这些函数对象是自完备的；它们无参函数对象，即它们可以直接调用，无须传入参数，这正是 `boost::function&lt;void()&gt;` 所表示的。换言之，以下代码片断创建了一个函数对象，它在配置好的 `tape_recorder` 实例上调用成员函数 play 。

```cpp
boost::bind(&tape_recorder::play,&tr) 
```

通常，我们不能保存 `bind` 所返回的函数对象，但由于 Boost.Function 兼容于任何函数对象，所以它可以。

```cpp
boost::function<void()> f(boost::bind(&tape_recorder::play,&tr)); 
```

注意，这个类也支持调用 `record`, 它带有一个类型为 `const std::string&` 的参数，这是由于成员函数 `set_function`. 因为这个函数对象必须是无参的，所以我们需要绑定上下文以便 `record` 仍旧能够获得它的参数。当然，这是绑定器的工作。因而，在调用 `record` 之前，我们创建一个包含被录音的字符串的函数对象。

```cpp
std::string s="What a beautiful morning...";
record.set_function(
  boost::bind(&tape_recorder::record,&tr,s)); 
```

执行这个保存在 `record` 的函数对象，将在 `tape_recorder` 实例 `tr` 上执行 `tape_recorder::record`，并传入字符串。有了 Boost.Function 和 Boost.Bind, 就可以实现解耦，让调用代码对于被调用代码一无所知。以这种方式结合使用这两个库非常有用。你已经在这个 `command` 类中看到了，现在我们该清理一下了。由于 Boost.Function 的杰出功能，你所需的只是以下代码：

```cpp
typedef boost::function<void()> command; 
```

### 与 Boost.Function 一起使用 Boost.Lambda

与 Boost.Function 兼容于由 Boost.Bind 创建的函数对象一样，它也支持由 Boost.Lambda 创建的函数对象。你用 Lambda 库创建的任何函数对象都兼容于相应的 `boost::function`. 我们在前一节已经讨论了基于绑定的一些内容，使用 Boost.Lambda 的主要不同之处是它能做得更多。我们可以轻易地创建一些小的、无名的函数，并把它们保存在 `boost::function` 实例中以用于后续的调用。我们已经在前一章中讨论了 lambda 表达式，在那一章的所有例子中所创建的函数对象都可以保存在一个 `function` 实例中。`function` 与创建函数对象的库的结合使用会非常强大。

### 代价的考虑

有一句谚语说，世界上没有免费的午餐，对于 Boost.Function 来说也是如此。与使用函数指针相比，使用 Boost.Function 也有一些缺点，特别是对象大小的增加。显然，一个函数指针只占用一个函数指针的空间大小(这当然了！)，而一个 `boost::function`实 例占的空间有三倍大。如果需要大量的回调函数，这可能会成为一个问题。函数指针在调用时的效率也稍高一些，因为函数指针是被直接调用的，而 Boost.Function 可能需要使用两次函数指针的调用。最后，可能在某些需要与 C 库保持后向兼容的情形下，只能使用函数指针。

虽然 Boost.Function 可能存在这些缺点，但是通常它们都不是什么实际问题。额外增加的大小非常小，而且(可能存在的)额外的函数指针调用所带来的代价与真正执行目标函数所花费的时间相比通常都是非常小的。要求使用函数而不能使用 Boost.Function 的情形非常罕见。使用这个库所带来的巨大优点及灵活性显然超出这些代价。

### 幕后的细节

至少了解一下这个库如何工作的基础知识是非常值得的。我们来看一下保存并调用一个函数指针、一个成员 函数指针和一个函数对象这三种情形。这三种情形是不同的。要真正看到 Boost.Function 如何工作，只有看源代码——不过我们的做法有些不同，我们试着搞清楚这些不同的版本究竟在处理方法上有些什么不同。我们也有一个 不同要求的类，即当调用一个成员函数时，必须传递一个实例的指针给 `function1` (这是我们的类的名字)的构造函数。`function1` 支持只有一个参数的函数。与 Boost.Function 相比一个较为宽松的投条件是，即使是对于成员函数，也只需要提供返回类型和参数类型。这个要求的直接结果就是，构造函数必须被传入一个类的实例用于成员函数的调用(类型可以自动推断)。

我们将要采用的方法是，创建一个泛型基类，它声明了一个虚拟的调用操作符函数；然后，从这个基类派生三个类，分别支持三种不同形式的函数调用。这些类负责所有的工作，而另一个类，`function1`, 依据其构造函数的参数来决定实例化哪一个具体类。以下是调用器的基类，`invoker_base`.

```cpp
template <typename R, typename Arg> class invoker_base {
public:
  virtual R operator()(Arg arg)=0;
}; 
```

接着，我们开始定义 `function_ptr_invoker`, 它是一个具体调用器，公有派生自 `invoker_base`. 它的目的是调用普通函数。这个类也接受两个类型，即返回类型和参数类型，它们被用于构造函数，构造函数接受一个函数指针作为参数。

```cpp
template <typename R, typename Arg> class function_ptr_invoker 
  : public invoker_base<R,Arg> {
  R (*func_)(Arg);
public:
  function_ptr_invoker(R (*func)(Arg)):func_(func) {}

  R operator()(Arg arg) {
    return (func_)(arg);
  }
}; 
```

这个类模板可用于调用任意一个接受一个参数的普通函数。调用操作符简单地以给定的参数调用保存在 `func_` 中的函数。请注意(的确有些奇怪)声明一个保存函数指针的变量的那行代码。

```cpp
R (*func_)(Arg); 
```

你也可以用一个 `typedef` 来让它好读一些。

```cpp
typedef R (*FunctionT)(Arg);
FunctionT func_; 
```

接着，我们需要一个可以处理成员函数调用的类模板。记住，它要求在构造时给出一个类实例的指针，这一点与 Boost.Function 的做法不一样。这样可以节省我们的打字，因为是编译器而不是程序员来推导这个类。

```cpp
template <typename R, typename Arg, typename T> 
class member_ptr_invoker : 
  public invoker_base<R,Arg> {
  R (T::*func_)(Arg);
  T* t_;
public:
  member_ptr_invoker(R (T::*func)(Arg),T* t)
    :func_(func),t_(t) {}

  R operator()(Arg arg) {
    return (t_->*func_)(arg);
  }
}; 
```

这个类模板与普通函数指针的那个版本很相似。它与前一个版本的不同在于，构造函数保存了一个成员函数指针与一个对象指针，而调用操作符则在该对象(`t_`)上调用该成员函数(`func_`)。

最后，我们需要一个兼容函数对象的版本。这是所有实现中最容易的一个，至少在我们的方法中是这样。通过使用单个模板参数，我们只表明类型 `T` 必须是一个真正的函数对象，因为我们想要调用它。说得够多了。

```cpp
template <typename R, typename Arg, typename T> 
class function_object_invoker : 
  public invoker_base<R,Arg> {
  T t_;
public:
  function_object_invoker(T t):t_(t) {}

  R operator()(Arg arg) {
    return t_(arg);
  }
}; 
```

现在我们已经有了这些适用的积木，剩下来的就是把它们放在一起组成我们的自己的 `boost::function`, 即 `function1` 类。我们想要一种办法来发现要实例化哪一个调用器。然后我们可以把它存入一个 `invoker_base` 指针。这里的窃门就是，提供一些构造函数，它们有能力去检查对于给出的参数，哪种调用器是正确的。这仅仅是重载而已，用了一点点手法，包括泛化两个构造函数。

```cpp
template <typename R, typename Arg> class function1 {
  invoker_base<R,Arg>* invoker_;
public:
  function1(R (*func)(Arg)) : 
  invoker_(new function_ptr_invoker<R,Arg>(func)) {}

  template <typename T> function1(R (T::*func)(Arg),T* p) : 
    invoker_(new member_ptr_invoker<R,Arg,T>(func,p)) {}

  template <typename T> function1(T t) : 
    invoker_(new function_object_invoker<R,Arg,T>(t)) {}

  R operator()(Arg arg) {
    return (*invoker_)(arg);
  }

  ~function1() {
    delete invoker_;
  }
}; 
```

如你所见，这里面最难的部分是正确地定义出推导系统以支持函数指针、类成员函数以及函数对象。无论使用何种设计来实现这类功能的库，这都是必须的。最后，给出一些例子来测试我们这个方案。

```cpp
bool some_function(const std::string& s) {
  std::cout << s << " This is really neat\n";
  return true;
}

class some_class {
public:
  bool some_function(const std::string& s) {
    std::cout << s << " This is also quite nice\n";
    return true;
  }
};

class some_function_object {
public:
  bool operator()(const std::string& s) {
    std::cout << s << 
      " This should work, too, in a flexible solution\n";
    return true;
  }
}; 
```

我们的 `function1` 类可以接受以下所有函数。

```cpp
int main() {
  function1<bool,const std::string&> f1(&some_function);
  f1(std::string("Hello"));

  some_class s;
  function1<bool,const std::string&> 
    f2(&some_class::some_function,&s);

  f2(std::string("Hello"));

  function1<bool,const std::string&>
    f3(boost::bind(&some_class::some_function,&s,_1));

  f3(std::string("Hello"));

  some_function_object fso;
  function1<bool,const std::string&> 
    f4(fso);
  f4(std::string("Hello"));
} 
```

它也可以使用象 Boost.Bind 和 Boost.Lambda 这样的 binder 库所返回的函数对象。我们的类与 Boost.Function 中的类相比要简单多了，但是也已经足以看出创建和使用这样一个库的问题以及相关解决方法。知道一点关于一个库是如何实现的事情，对于有效使用这个库是非常有用的。

# Function 总结

## Function 总结

在以下情形时使用 Function 库

*   你需要保存一个回调函数或函数对象

*   你想要从实现中解耦函数调用，例如在 GUI 和实现间的解耦

*   你想要保存由 binder 库创建的函数对象，用于后续的调用或多次调用

Boost.Function 是对标准库的功能的重要补充。在回调机制中使用函数指针这样的著名技术被扩充至可以使用任何行为类似于函数的东西，包括由 binder 库创建的函数对象。通过使用 Boost.Function, 可以很容易地为回调增加状态，也可以把已有的类和成员函数进行适配后用作回调函数。

与使用函数指针相比，使用 Boost.Function 有几个优点：通过兼容的函数对象而不是真实的签名放松了对签名的要求；可以使用绑定器，如 Boost.Bind 和 Boost.Lambda；可以在调用函数之前检测函数是否为空，即是否存在目标函数；可以使用带状态的对象而不仅限于无状态函数。这些优点表明了使用 Boost.Function 替代 C 风格的回调可以解决这类普遍存在的问题。使用 Boost.Function 比使用函数指针要多付出一点点代价，只有这一点小代价是被禁止时，才应该考虑使用函数指针技术。

Boost.Function 由 Douglas Gregor 创建。它是一个拥有巨大威力的库，具有成熟的设计与实现，可以为用户提供额外的价值。