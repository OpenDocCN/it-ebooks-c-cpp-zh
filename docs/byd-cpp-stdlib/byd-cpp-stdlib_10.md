# Library 3\. Utility

# Utility 库如何改进你的程序？

## Utility 库如何改进你的程序？

*   编译期断言 `BOOST_STATIC_ASSERT`

*   安全的析构 `checked_delete` 和 `checked_array_delete`

*   禁止复制 `noncopyable`

*   `operator&`被重载时用 `addressof`取得对象地址

*   用`enable_if` 和 `disable_if`控制重载与特化

有些工具还不够组成它们自己的库，因此它们与其它实体被集合到一起。这就形成了 Boost.Utility，收集了一些没有更合适地方存放的、有用的工具。它们很有用，应该被加入到 Boost，但它们又太小，不足以形成自己的库。本 章介绍 Boost.Utility 中最基本的以及最广泛使用的工具。

我们将从 `BOOST_STATIC_ASSERT`开始，它是一个在编译期判断整型常量表达式的工具。然后，我们看看当你通过一个指向不完整类型的指针`delete`对象时，即当被删除的对象的内存布局未知时，会发生什么。`checked_delete` 使得这个讨论更为有趣。我们还会看到 `noncopyable` 如何防止一个类被复制，这也是本章最重要的主题。然后我们将看到 `addressof`, 它用于阻止那些重载了`operator&`的险恶的程序员[1]的病态行为。最后，我们将测试 `enable_if`, 它非常有用，可用于在名字查找时控制函数重载与模板特化是否被考虑。

> [1] 如果你认为我说的不对，请把你认为最合理的重载了`operator&`的用例发给我。

# BOOST_STATIC_ASSERT

## BOOST_STATIC_ASSERT

### 头文件: `"boost/static_assert.hpp"`

在运行期执行断言可能是你经常用到的，也是非常合理的。它是测试前置条件、后置条件以及不变式的好方 法。执行运行期断言有很多不同的方法，但是在编译期你如何进行断言呢？当然，唯一的方法就是让编译器产生一个错误，这是很平常的事情(我在无意中都做过几 千次了)，但如何从错误信息中获得有意义的信息却不是那么明显的。而且，即使你在一个编译器上找到了办法，也很难把它移植到其它编译器上。这就是使用 `BOOST_STATIC_ASSERT`的原因。它可以在不同的平台上使用，正如我们即将看到的。

### 用法

要开始使用静态断言，就要包含头文件 `"boost/static_assert.hpp"`. 该头文件定义了宏[2] `BOOST_STATIC_ASSERT`. 作为它的第一个使用范例，我们来看看如何在类作用域中使用它。考虑一个泛化的类，它要求实例化时所用的类型是一个整数类型。我们不想为所有类型提供特化， 因此我们需要在编译期进行测试，以确保我们的类的确是用一个整数类型进行实例化的。现在，我们先提前一点使用另一个 Boost 库来进行测试，它就是 Boost.Type_traits. 我们使用一个称为`is_integral`的断言，它对它的参数执行一个编译期求值，正如你从它的名字可以猜到的一样，求值的结果是表明该类型是否一个整数类型。

> [2] 是的，它是一个宏。你知道，宏也可以很有用的。

```cpp
#include <iostream>

#include "boost/type_traits.hpp"
#include "boost/static_assert.hpp"

template <typename T> class only_compatible_with_integral_types {
  BOOST_STATIC_ASSERT(boost::is_integral<T>::value);
}; 
```

有了这个断言，在实例化类 `only_compatible_with_integral_types` 时如果试图使用一个非整型的类型，就会导致一个编译期的失败。输出信息取决于编译器，但在多数编译器下输出信息会惊人地一致。

假设我们试图这样实例化：

```cpp
only_compatible_with_integral_types<double> test2; 
```

编译器将会有类似下面的输出：

```cpp
Error: use of undefined type
  'boost::STATIC_ASSERTION_FAILURE<false>' 
```

在类的作用域里，你可以明确类的要求：象在前面这样的模板中明确参数的类型就是一个明显的例子。你也可以使用断言来明确类所要求的其它前提条件，如类型的大小等等。

### 函数作用域中的 BOOST_STATIC_ASSERT

`BOOST_STATIC_ASSERT` 也可以用在函数作用域中。例如，考虑一个泛化的函数，它带有一个非类型模板参数，并且该参数只接受 1 至 10 的值。与其在运行期执行断言，我们不如在编译器使用静态断言。

```cpp
template <int i> void accepts_values_between_1_and_10() {
  BOOST_STATIC_ASSERT(i>=1 && i<=10);
} 
```

该函数的用户不能使用超出允许范围的数值来实例化这个函数。当然，断言中的表达式必须是一个纯粹的编译期表达式，也就是说，表达式中的参数和操作符都必须被编译器所认识。`BOOST_STATIC_ASSERT` 当然并不是只能用于泛型函数；我们可以在任何函数中很方便地测试条件。例如，一个函数需要一个与平台相关的前提条件，就常常需要一个断言。

```cpp
void expects_ints_to_be_4_bytes() {
  BOOST_STATIC_ASSERT(sizeof(int)==4);
} 
```

### 总结

你所看到的这种静态断言在 C++中正变得象运行期断言`assert`那样常用。这应该至少部分地归功于"元编程革命"，它使得一个程序中更多的计算量在编译期执行。表达编译期断言的唯一方法就是让编译器产生一个错误。为了让断言可用，错误提示必须可以传达有用的信息，但这很难做到可移植(事实上，根本不可能做到)。这正是 `BOOST_STATIC_ASSERT` 所要做的，它在大多数的编译器下提供了编译期断言的一致输出。它可用于名字空间、类、函数以及作用域。

以下情形下使用 `BOOST_STATIC_ASSERT` ：

*   当条件可以在编译期进行求值

*   对类型的要求可以在编译期表示

*   你需要对两个或以上的整型常量间的关系进行断言

# checked_delete

## checked_delete

### 头文件: `"boost/checked_delete.hpp"`

通过指针来删除一个对象时，执行的结果取决于执行删除时被删除的类型是否可知。对一个指向不完整类型的指针执行`delete`几乎不可能有编译器警告，这会导致各种各样的麻烦，由于析构函数可以没有被执行。换句话说，即进行清除的代码没有被执行。`checked_delete` 在对象析构时执行一个静态断言，测试类是否可知，以确保析构函数被执行。

### 用法

`checked_delete` 是一个`boost`名字空间中的模板函数。它用于删除动态分配的对象，对于动态分配的数组，同样有一个称为 `checked_array_delete`的模板函数。这些函数接受一个参数：要删除的指针，或是要删除的数组。这两个函数都要求在销毁对象时(即对象被传给函数时)，这些被删除的类型必须是可知的。使用这些函数，要包含头文件`"boost/checked_delete.hpp"`. 使用这些函数时，你只需象调用`delete`那样简单地调用它们。以下程序前向声明了一个类`some_class`, 而没有定义它。有些编译器允许对一个指向 `some_class` 的指针被删除(稍后再讨论这个)，但使用 `checked_delete` 后，就不能通过编译了，除非有一个 `some_class` 的定义。

```cpp
#include "boost/checked_delete.hpp"

class some_class;

some_class* create() {
  return (some_class*)0;
}

int main() {
  some_class* p=create();
  boost::checked_delete(p2);
} 
```

如果你试图编译这段代码，对函数 `checked_delete&lt;some_class&gt;` 的实例化将失败，因为 `some_class` 是一个不完整的类型。你的编译器会输出类似下面的信息：

```cpp
checked_delete.hpp: In function 'void
boost::checked_delete(T*) [with T = some_class]':
checked_sample.cpp:11:   instantiated from here
boost/checked_delete.hpp:34: error: invalid application of 'sizeof' to an incomplete type
boost/checked_delete.hpp:34: error: creating array with
size zero ('-1')
boost/checked_delete.hpp:35: error: invalid application of
'sizeof' to an incomplete type
boost/checked_delete.hpp:35: error: creating array with
size zero ('-1')
boost/checked_delete.hpp:32: warning: 'x' has incomplete type 
```

错误信息的前面部分清楚地说明了问题：`checked_delete` 遇到了一个不完整的类型。但我们的代码中哪里存在不完整的类型呢？接下来的章节我们来讨论它。

### 究竟是什么问题？

在我们深入了解 `checked_delete`的好处之前，让我们先来彻底弄清楚问题所在。如果你试图删除一个指针，而该指针指向的是一个带有非平凡析构函数[4]的不完整类型[3]，结果将是未定义的行为。这是如何发生的呢？让我们来看一个例子。

> [3] 不完整的类型是指已声明但未定义的类型。
> 
> [4] 标准说法是，类的一个或多个直接基类，或者一个或多个非静态数据成员，具有用户定义的析构函数。

```cpp
// deleter.h
class to_be_deleted;

class deleter { 
public:
  void delete_it(to_be_deleted* p);
};

// deleter.cpp
#include "deleter.h"

void deleter::delete_it(to_be_deleted* p) {
  delete p;
}

// to_be_deleted.h
#include <iostream>
class to_be_deleted
{
public:
  ~to_be_deleted() {
    std::cout << 
      "I'd like to say important things here, please.";
  }
};

// Test application
#include "deleter.h"
#include "to_be_deleted.h"

int main() {
  to_be_deleted* p=new to_be_deleted;

  deleter d;
  d.delete_it(p);
} 
```

以上代码试图 `delete` 一个指向不完整类型`to_be_deleted`的指针，这会导致未定义行为。注意，`to_be_deleted` 在 `deleter.h`中是前向声明的；`deleter.cpp` 包含了 `deleter.h` 而没有包含 `to_be_deleted.h`: 而`to_be_deleted.h` 中为`to_be_deleted`定义了一个非平凡析构函数。这种麻烦很容易出现，尤其是在使用智能指针的时候。我们要做的就是在调用`delete`时确认类型是完整的，这正是 `checked_delete` 所做的。

### checked_delete 来解决问题

前面的例子说明了删除不完整类型时不进行确认很可能会引起麻烦，而且不是所有编译器会对此给出警告。编写泛型代码时，避免这种情况是非常必要的。使用 `checked_delete`重写这个例子，你只需要把 `delete p` 改为 `checked_delete(p)`.

```cpp
void deleter::do_it(to_be_deleted* p) {
  boost::checked_delete(p);
} 
```

`checked_delete` 基本上就是一个判断类是否完整的断言，它的实现如下：

```cpp
template< typename T > inline void checked_delete(T * x) {
  typedef char type_must_be_complete[sizeof(T)];
  delete x;
} 
```

这里的想法是创建一个`char`的数组，数组的元素数量为`T`的大小。如果 `checked_delete` 被一个不完整的类型 `T` 所实例化，编译将会失败，因为 `sizeof(T)` 会返回 0, 而创建一个 0 个元素的(自动)数组是非法的。你也可以用 `BOOST_STATIC_ASSERT` 来执行这个断言。

```cpp
BOOST_STATIC_ASSERT(sizeof(T)); 
```

在编写要求使用完整类型进行实例化的模板时，这个工具非常方便。对于数组，也有一个相应的"checked deleter"，称为 `checked_array_delete`, 它的用法类似于 `checked_delete`.

```cpp
to_be_deleted* p=new to_be_deleted[10];
boost::checked_array_delete(p); 
```

### 总结

删除一个动态分配的对象时，必须调用它的析构函数。如果这个类型是不完整的，即只有声明没有定义，那么析构函数可能会没被调用。这是一种潜在的危险状态，所以应该避免它。对于类模板及函数模板，风险会更大，因为无法预先知道会使用什么类型。使用 `checked_delete` 和 `checked_array_delete`, 可以解决这个删除不完整类型的问题。它没有运行期的额外开销，只是直接调用 `delete`, 因此说 `checked_delete` 带来的安全性实际上是免费的。

如果你需要在调用`delete`时确保类型是完整的，就使用 `checked_delete` 。

# noncopyable

## noncopyable

### 头文件: `"boost/utility.hpp"`

通常编译器都是程序员的好朋友，但并不总是。它的好处之一在于它会自动为我们提供复制构造函数和赋值操作符，如果我们决定不自己动手去 做的话。这也可能会导致一些不愉快的惊讶，如果这个类本身就不想被复制(或被赋值)。如果真是这样，我们就需要明确地告诉这个类的使用者复制构造以及赋值 是被禁止的。我不是说在代码中进行注释说明，而是说要禁止对复制构造函数以及赋值操作符的访问。幸运的是，当类带有不能复制或不能赋值的基类或成员函数 时，编译器生成的复制构造函数及赋值操作符就不能使用。`boost::noncopyable` 的工作原理就是禁止访问它的复制构造函数和赋值操作符，然后使用它作为基类。

### 用法

要使用 `boost::noncopyable`, 你要从它私有地派生出不可复制类。虽然公有继承也可以，但这是一个坏习惯。公有继承对于阅读类声明的人而言，意味着 IS-A (表示派生类 IS-A 基类)关系，但表明一个类 IS-A `noncopyable` 看起来有点不太对。要从`noncopyable`派生，就要包含 `"boost/utility.hpp"` 。

```cpp
#include "boost/utility.hpp"

class please_dont_make_copies : boost::noncopyable {};

int main() {
  please_dont_make_copies d1;
  please_dont_make_copies d2(d1);
  please_dont_make_copies d3;
  d3=d1;
  } 
```

这个例子不能通过编译。由于`noncopyable`的复制构造函数是私有的，因此对`d2`进行复制构造的尝试会失败。同样，由于`noncopyable`的赋值操作符也是私有的，因此将`d1`赋值给`d3`的尝试也会失败。编译器会给出类似下面的输出：

```cpp
noncopyable.hpp: In copy constructor
' please_dont_make_copies::please_dont_make_copies (const please_dont_make_copies&)':
boost/noncopyable.hpp:27: error: '
  boost::noncopyable::noncopyable(const boost::noncopyable&)' is
private
noncopyable.cpp:8: error: within this context
boost/noncopyable.hpp: In member function 'please_dont_make_copies&
  please_dont_make_copies::operator=(const please_dont_make_copies&)':
boost/noncopyable.hpp:28: error: 'const boost::noncopyable&
  boost::noncopyable::operator=(const boost::noncopyable&)' is private
noncopyable.cpp:10: error: within this context 
```

下一节我们将测试这是如何工作的。很清楚从`noncopyable`派生将禁止复制和赋值。这也可以通过把复制构造函数和赋值操作符定义为私有的来实现。 我们来看一下怎么样做。

### 使类不能复制

再看一下类 `please_dont_make_copies`, 为了某些原因，它不能被复制。

```cpp
class please_dont_make_copies {
public:
  void do_stuff() {
    std::cout <<
      "Dear client, would you please refrain from copying me?";
  }
}; 
```

由于编译器生成了复制构造函数和赋值操作符，所以现在不能禁止类的复制和赋值。

```cpp
please_dont_make_copies p1;
please_dont_make_copies p2(p1);
please_dont_make_copies p3;
p3=p2; 
```

解决的方法是把复制构造函数和赋值操作符声明为私有的或是保护的，并增加一个缺省构造函数(因为编译器不再自动生成它了)。

```cpp
class please_dont_make_copies {
public:
  please_dont_make_copies() {}

  void do_stuff() {
    std::cout << 
      "Dear client, would you please refrain from copying me?";
  }
private:
  please_dont_make_copies(const please_dont_make_copies&);
  please_dont_make_copies& operator=
    (const please_dont_make_copies&);
}; 
```

这可以很好地工作，但它不能马上清晰地告诉 `please_dont_make_copies`的使用者它是不能复制的。下面看一下换成 `noncopyable` 后，如何使得类更清楚地表明不能复制，并且也可以打更少的字。

### 用 noncopyable

类 `boost::noncopyable` 被规定为作为私有基类来使用，它可以有效地关闭复制构造和赋值操作。用前面的例子来看看使用`noncopyable`后代码是什么样子的：

```cpp
#include "boost/utility.hpp"

class please_dont_make_copies : boost::noncopyable {
public:
  void do_stuff() {
    std::cout << "Dear client, you just cannot copy me!";
 }
}; 
```

不再需要声明复制构造函数或赋值操作符。由于我们是从`noncopyable`派生而来的，编译器不会再生成它们了，这样就禁止了复制和赋值。简洁可以带来清晰，尤其是象这样的基本且清楚的概念。对于阅读这段代码的使用者来说，马上就清楚地知道这个类是不能复制和赋值的，因为 `boost::noncopyable` 在类定义的一开始就出现了。最后要提醒的一点是：你还记得类的缺省访问控制是私有的吗？这意味着缺省上继承也是私有的。你也可以象这样写，来更加明确这个事实：

```cpp
class please_dont_make_copies : private boost::noncopyable { 
```

这完全取决于观众；有些程序员认为这种多余的信息是令人讨厌并且会分散注意力，而另一些程序员则认同这种清晰性。由你来决定哪一种方法适合你的类和你的程序员。无论哪一种方法，使用 `noncopyable` 都要比"忘记"复制构造函数和赋值操作符的方法更加明确，也比私有地声明它们更为清晰。

### 记住 the Big Three

正如我们看到的那样，`noncopyable` 为禁止类的复制和赋值提供了一个方便的办法。但何时我们需要这样做呢？什么情况下我们需要自定义复制构造函数或赋值操作符？这个问题有一个通用的答案，一 个几乎总是正确的答案：无论何时你需要定义析构函数、复制构造函数、或赋值操作符三个中的任意一个，你也需要定义另外两个[5]。它们三者间的互动性非常重要，其中一个存在，其它的通常也都必须要有。我们假设你的一个类有一个成员是指针。你定义了一个析构函数用于正确地释放空间，但你没有定义复制构造函数和赋值操作符。这意味着你的代码中至少存在两个潜在的危险，它们很容易被触发。

> [5] 这个定律的名字叫 the Big Three，来自于 C++ FAQs (详情请见参考书目[2])。

```cpp
class full_of_errors {
  int* value_;
public:
  full_of_errors() {
    value_=new int(13);
  }

 ~full_of_errors() {
    delete value_;
  }
}; 
```

使用这个类时，如果你忽视了编译器为这个类生成的复制构造函数和赋值操作符，那么至少有三种情况会产生错误。

```cpp
full_of_errors f1;
full_of_errors f2(f1);
full_of_errors f3=f2;
full_of_errors f4;
f4=f3; 
```

注意，第二行和第三行是调用复制构造函数的两个等价的方法。它们都会调用生成的复制构造函数，虽然语法有所不同。最后一个错误在最后一行，赋值操作符使得同一个指针被至少两个`full_of_errors`实例所删除。正确的方法是，我们需要自己的复制构造函数和赋值操作符，因为我们定义了我们自己的析构函数。以下是正确的方法：

```cpp
 class not_full_of_errors {
    int* value_;
  public:
    not_full_of_errors() {
      value_=new int(13);
    }

    not_full_of_errors(const not_full_of_errors& other) :
      value_(new int(*other.value_)) {}

    not_full_of_errors& operator=
      (const not_full_of_errors& other) {
      *value_=*other.value_;
      return *this;
    }

    ~not_full_of_errors() {
      delete value_;
    }
}; 
```

所以，无论何时，一个类的 the big three：复制构造函数、(虚拟)析构函数、和赋值操作符，中的任何一个被手工定义，在决定不需要定义其余两个之前必须认真仔细地考虑清楚。还有，如果你不想要复制，记得使用 `boost::noncopyable` ！

### 总结

有很多类型需要禁止复制和赋值。但是，我们经常忽略了把这些类型的复制构造函数和赋值操作符声明为私 有的，而把责任转嫁给了类的使用者。即使你使用了私有的复制构造函数和赋值操作符来确保它们不被复制或赋值，但是对于使用者而言这还不够清楚。当然，编译 器会友好地提醒试图这们做的人，但错误来自何处也不是清晰的。最好我们可以清晰地做到这一点，而从 `noncopyable` 派生就是一个清晰的声明。当你看一眼类型的声明就可以马上知道了。编译的时候，错误信息总会包含名字 noncopyable. 而且它也节省了一些打字，这对于某些人而言是关键的因素。

以下情形下使用 `noncopyable` ：

*   类型的复制和赋值都不被允许

*   复制和赋值的禁止应该尽可能明显

# addressof

## addressof

### 头文件: `"boost/utility.hpp"`

要取得一个变量的地址，我们要依赖于返回的值是否真的是这个变量的地址。但是，技术上重载`operator&`是有可能的，这意味着存有恶意的人可以破坏你的地址相关的代码。`boost::addressof` 被用于获得变量的地址，不管取址操作符是否被误用。通过使用一些灵巧的内部机制，模板函数 `addressof` 确保可以获得真实的对象及其地址。

### 用法

为确保获得一个对象的真实地址，你要使用 `boost::addressof`. 它定义在 `"boost/utility.hpp"`. 它常用于原本要使用 `operator&` 的地方，它接受一个参数，该参数为要获得地址的那个对象的引用。

```cpp
#include "boost/utility.hpp"

class some_class {};

int main() {
  some_class s;
  some_class* p=boost::addressof(s);
} 
```

在进一步学习如何使用 `addressof`的细节前，了解一下`operator&`为何以及如何不一定会返回对象的地址是非常有用的。

### 快速了解一下存有恶意的人

如果你真的，真的，真的需要重载 `operator&`, 或者只是想试验一下操作符重载可能的用法，这的确很容易。当你重载 `operator&`时，它的语义肯定会与多数用户(以及函数！)所期望的不同，所以千万不要为了好玩而做这件事；除非有非常好的理由，否则不要去做它。以下有一段 code-breaker 代码：

```cpp
class codebreaker {
public:
  int operator&() const {
    return 13;
  }
}; 
```

对于这个类，任何人想获取一个`codebreaker`实例的地址都会得到一个不可思议的数字 13.

```cpp
template <typename T> void print_address(const T& t) {
  std::cout << "Address: " << (&t) << '\n';
}

int main() {
  codebreaker c;
  print_address(c);
} 
```

这不难做到，但是在实际的代码中这样做有没有好的理由？也许没有，因为除非是用在局部的类上，否则它是不安全的。原因是，虽然获取一个不完整类型的地址是合法的，但如果是要获取一个带有用户自定义`operator&`的不完整类型的地址则是未定义的行为。因为我们不能保证这不会发生，所以我们最好不要重载 `operator&`.

### 迅速的解决方法

即使一个类的 `operator&` 被重载了，也还是有办法获得这个类的实例的真实地址。`addressof` 使用了一些幕后的巧妙方法[6]来获得真实的地址，而不会受任何 `operator&` 的欺骗。如果你把函数(`print_address`)改为使用 `addressof`, 你就可以得到以下代码：

> [6] 非常出名的一种 ingenious hack.

```cpp
template <typename T> void print_address(const T& t) {
  std::cout << "&t: " << (&t) << '\n';
  std::cout << "addressof(t): " << boost::addressof(t) << '\n';
} 
```

执行时，该函数将给出如下输出(或类似于以下的输出，因为准确的地址值取决于你的系统).

```cpp
&t: 13
addressof(t): 0012FECB13 
```

差不多就是这样了！如果有什么情况让你知道或怀疑一个类的`operator&`被重载了，而你又需要确保得到真实的地址(由于 `operator&` 被重载而变得不可信了), 你就应该使用 `addressof`.

### 总结

没有多少有力的论点支持重载 `operator&`,[7] 但由于这是可能的，总有些人会这样做。当你编写一些需要依赖于获得对象真实地址的代码时，`addressof` 可以帮助你确保得到真实的地址。在编写泛型代码时，没有办法知道将会操作什么类型，因此如果需要获取参数化类型的地址的话，就使用 `addressof`.

> [7] 即使是定制的硬件设备驱动程序

当你需要获得一个对象的真实地址时，使用 `addressof` ，不必管 `operator&` 的语义。

# enable_if

## enable_if

### 头文件: `"boost/utility/enable_if.hpp"`

有时候，我们希望控制某个函数或类模板的特化是否可以加入到重载决议时使用的重载或特化的集合中。例如，考虑一个重载的函数，它有一个版本是带一个`int`参数的普通函数，另一个版本是一个函数模板，它要求参数类型 `T` 具有一个名为`type`的嵌套类型。它们看起来可能象这样：

```cpp
void some_func(int i) {
  std::cout << "void some_func(" << i << ")\n";
}

template <typename T> void some_func(T t) {
  typename T::type variable_of_nested_type;
  std::cout << 
    "template <typename T> void some_func(" << t << ")\n";
} 
```

现在，想象一下当你在代码中调用 `some_func` 将发生什么。如果参数的类型为 `int`, 第一个版本将被调用。如果参数的类型是 `int`以外的其它类型，则第二个(模板)版本将被调用。

这没问题，只要这个类型有一个名为`type`的嵌套类型，但如果它没有，这段代码就不能通过编译。这会是一个问题吗？好的，考虑一下如果你用其它整数类型来调用，如`short`, 或 `char`, 或 `unsigned long`，那么又会发生什么。

```cpp
#include <iostream>

void some_func(int i) {
  std::cout << "void some_func(" << i << ")\n";
}

template <typename T> void some_func(T t) {
  typename T::type variable_of_nested_type;
  std::cout << 
    "template <typename T> void some_func(" << t << ")\n";
}

int main() {
  int i=12;
  short s=12;

  some_func(i);
  some_func(s);
} 
```

编译这段程序时，你将从失败的编译器中得到类似以下的输出：

```cpp
enable_if_sample1.cpp: In function 'void some_func(T)
  [with T = short int]':
enable_if_sample1.cpp:17:   instantiated from here
enable_if_sample1.cpp:8: error:
  'short int' is not a class, struct, or union type

Compilation exited abnormally with code 1 at Sat Mar 06 14:30:08 
```

就是这样。`some_func` 的模板版本被选为最佳的重载，但这个版本中的代码对于类型`short`而言是无效的。我们怎样才能避免它呢？好的，我们希望仅对含有名为 type 的嵌套类型的类使用模板版本的 `some_func` ，而对于其它没有这个嵌套类型的类则忽略它。我们能够做到。最简单的方法，但不一定是实际中总能使用的方法，是把模板版本的返回类型改为如下：

```cpp
template <typename T> typename T::type* some_func(T t) {
  typename T::type variable_of_nested_type;
  std::cout <<
    "template <typename T> void some_func(" << t << ")\n";
  return 0;
} 
```

如果你没有学过 SFINAE (匹配失败不是错误),[8] 很可能现在你的脸上会有困惑的表情。编译修改过的代码，我们的例子会通过编译。`short` 被提升为 `int`, 并且第一个版本被调用。这种令人惊奇的行为的原因是模板版本的 `some_func` 不再包含在重载决议的集合内了。它被排除在内是因为，编译器看到了这个函数的返回类型要求模板类型`T` 要有一个嵌套类型`type` ，而它知道 `short` 不满足这个要求，所以它把这个函数模板从重载决议集合中删掉了。这就是 Daveed Vandevorde 和 Nicolai Josuttis 教给我们的 SFINAE, 它意味着宁可对有问题的类型不考虑函数的重载，也不要产生一个编译器错误。如果类型有一个符合条件的嵌套类型，那么它就是重载决议集合的一部分。

> [8] 见参考书目[3]。

```cpp
class some_class {
public:
  typedef int type;
};

int main() {
  int i=12;
  short s=12;

  some_func(i);
  some_func(s);
  some_func(some_class());
} 
```

运行该程序的输出如下：

```cpp
void some_func(12)
void some_func(12)
template <typename T> void some_func(T t) 
```

这种办法可以用，但它不太好看。在这种情形下，我们可以不管原来的 `void` 返回类型，我们可以用其它类型替换它。但如果不是这种情形，我们就要给函数增加一个参数并给它指定一个缺省值。

```cpp
template <typename T>
  void some_func(T t,typename T::type* p=0) {
  typename T::type variable_of_nested_type;   
  std::cout << "template <typename T> void some_func(T t)\n";
} 
```

这个版本也是使用 SFINAE 来让自己不会被无效类型所使用。这两种解决方案的问题都在于它们有点难看，我们把它们弄成了公开接口的一部分，并且它们只能在某些情形下使用。Boost 提供了一个更干净的解决方法，这种方法不仅在语法上更好看，而且提供了比前面的解决方法更多的功能。

### 用法

要使用 `enable_if` 和 `disable_if`, 就要包含头文件 `"boost/utility/enable_if.hpp"`. 在第一个例子中，我们将禁止第二个版本的 `some_func` ，如果参数的类型是整型的话。象一个类型是否整型这样的类型信息可以用另一个 Boost 库`Boost.Type_traits`来取得。`enable_if` 和 `disable_if` 模板都通过接受一个谓词来控制是否启用或禁止一个函数。

```cpp
#include <iostream>
#include "boost/utility/enable_if.hpp"
#include "boost/type_traits.hpp"

void some_func(int i) {
  std::cout << "void some_func(" << i << ")\n";
}

template <typename T> void some_func(
  T t,typename boost::disable_if<
    boost::is_integral<T> >::type* p=0) {
    typename T::type variable_of_nested_type;
    std::cout << "template <typename T> void some_func(T t)\n";
} 
```

虽然这看起来与我们前面所做的差不多，但它表达了一些我们使用直接的方法所不能表达的东西，而且它在函数的声明中表达了关于这个函数的重要信息。看到这些，我们可以清楚的知道这个函数要求类型`T`不能是一个整数类型。如果我们希望仅对含有嵌套类型`type`的类型启用这个函数，它也可以做得更好，而且我们还可以用另一个库 Boost.Mpl[9] 来做。如下：

> [9] Boost.Mpl 超出了本书的范围。访问 [`www.boost.org`](http://www.boost.org) 获得更多关于 Mpl 的信息。另外，也可以看一下 David Abrahams 和 Aleksey Gurtovoy 的书, C++ Template Metaprogramming!

```cpp
#include <iostream>
#include "boost/utility/enable_if.hpp"
#include "boost/type_traits.hpp"
#include "boost/mpl/has_xxx.hpp"

BOOST_MPL_HAS_XXX_TRAIT_DEF(type)

void some_func(int i) {
  std::cout << "void some_func(" << i << ")\n";
}

template <typename T> void some_func(T t,
  typename boost::enable_if<has_type<T> >::type* p=0) {
    typename T::type variable_of_nested_type;   
    std::cout << "template <typename T> void some_func(T t)\n";
} 
```

这真的很酷！我们现在可以对没有嵌套类型`type`的`T`禁用`some_func`的模板版本了，而且我们清晰地表达了这个函数的要求。这里的窍门在于使用了 Boost.Mpl 的一个非常漂亮的特性，它可以测试任意类型`T`是否内嵌有某个指定类型。通过使用宏 `BOOST_MPL_HAS_XXX_TRAIT_DEF(type)`, 我们定义了一个名为`has_type`的新的 trait，我们可以在函数`some_func`中使用它作为`enable_if`的谓词。如果谓词为`True`, 这个函数就是重载决议集合中的一员；如果谓词为 `false`, 这个函数就将被排除。

也可以包装返回类型，而不用增加一个额外的(缺省)参数。我们最后一个也是最好的一个 `some_func`, 在它的返回类型中使用 `enable_if` ，如下：

```cpp
template <typename T> typename
boost::enable_if<has_type<T>,void>::type
  some_func(T t) {
    typename T::type variable_of_nested_type;
    std::cout << "template <typename T> void some_func(T t)\n";
} 
```

如果你需要返回你想启用或禁用的类型，那么在返回类型中使用 `enable_if` 和 `disable_if` 会比增加一个缺省参数更合适。另外，有可能有的人真的为缺省参数指定一个值，那样就会破坏这段代码。有时，类模板的特化也需要被允许或被禁止，这时也可以使用 `enable_if`/`disable_if` 。不同的是，对于类模板，我们需要对主模板进行一些特别的处理：增加一个模板参数。考虑一个带有返回一个`int`的成员函数`max`的类模板：

```cpp
template <typename T> class some_class {
public:
  int max() const {
    std::cout << "some_class::max() for the primary template\n";
    return std::numeric_limits<int>::max();
  }
}; 
```

假设我们决定对于所有算术类型(整数类型及浮点数类型), 给出一个特化版本的定义，`max` 返回的是该算术类型可以表示的最大值。那么我们需要对模板类型`T`使用`std::numeric_limits`，而对其它类型我们还是使用主模板。要做到这样，我们必须给主模板加一个模板参数，该参数的缺省类型为 `void` (这意味着用户不需要显式地给出该参数)。结果主模板的定义如下：

```cpp
template <typename T,typename Enable=void> class some_class {
public:
  int max() const {
    std::cout << "some_class::max() for the primary template\n";
    return std::numeric_limits<int>::max();
  }
}; 
```

现在我们已经为提供特化版本作好了准备，该特化版本为算术类型所启用。该特性可通过 Boost.Type_traits 库获得。以下是特化版本：

```cpp
template <typename T> class some_class<T,
  typename boost::enable_if<boost::is_arithmetic<T> >::type> {
public:
  T max() const {
   std::cout << "some_class::max() with an arithmetic type\n";
   return std::numeric_limits<T>::max();
  }
}; 
```

该版本只有当实例化所用的类型为算术类型时才会启用，这时特性 `is_arithmetic` 为 `true`. 它可以正常工作是因为 `boost::enable_if&lt;false&gt;::type` 是 `void`, 会匹配到主模板。以下程序用不同的类型测试这个模板：

```cpp
#include <iostream>
#include <string>
#include <limits>
#include "boost/utility/enable_if.hpp"
#include "boost/type_traits.hpp"

// Definition of the template some_class omitted

int main() {
  std::cout << "Max for std::string: " <<
    some_class<std::string>().max() << '\n';
  std::cout << "Max for void: " << 
    some_class<void>().max() << '\n';
  std::cout << "Max for short: " << 
    some_class<short>().max() << '\n';
  std::cout << "Max for int: " << 
    some_class<int>().max() << '\n';
  std::cout << "Max for long: " << 
    some_class<long>().max() << '\n';
  std::cout << "Max for double: " << 
    some_class<double>().max() << '\n';
} 
```

我们预期前两个 `some_class` 会实例化主模板，剩下的将会实例化算术类型的特化版本。运行该程序可以看到的确如此。

```cpp
some_class::max() for the primary template
Max for std::string: 2147483647
some_class::max() for the primary template
Max for void: 2147483647
some_class::max() with an arithmetic type
Max for short: 32767
some_class::max() with an arithmetic type
Max for int: 2147483647
some_class::max() with an arithmetic type
Max for long: 2147483647
some_class::max() with an arithmetic type
Max for double: 1.79769e+308 
```

一切正常！以前，要允许或禁止重载函数和模板特化需要一些编程的技巧，多数看到代码的人都不能完全明白。通过使用 `enable_if` 和 `disable_if`, 代码变得更容易写也更容易读了，并且可以从声明中自动获得正确的类型要求。在前面的例子中，我们使用了模板 `enable_if`, 它要求其中的条件要有一个名为`value`的嵌套定义。对于多数可用于元编程的类型而言这都是成立的，但对于整型常量表达式则不然。如果没有名为`value`的嵌套类型，就要使用 `enable_if_c` 来代替，它接受一个整型常量表达式。使用 `is_arithmetic` 并直接取出它的值，我们可以这样重写`some_class`的启用条件：

```cpp
template <typename T> class some_class<T,
  typename boost::enable_if_c<
    boost::is_arithmetic<T>::value>::type> {
public:
  T max() const {
   std::cout << "some_class::max() with an arithmetic type\n";
   return std::numeric_limits<T>::max();
  }
}; 
```

`enable_if` 和 `enable_if_c`原则上并没有不同。它们的区别仅在于是否要求有嵌套类型 value。

### 总结

被称为 SFINAE 的 C++语言特性是很重要的。没有它，很多新的代码会破坏已有的代码，并且某些类型的函数重载(以及模板特化)将会无法实现。直接使用 SFINAE 来控制特定的函数或类型，使之被允许或被禁止用于重载决议，会很复杂。这样也会产生难以阅读的代码。使用 `boost::enable_if` 是更好的办法，它可以规定重载仅对某些特定类型有效。如果相同的参数用于 `disable_if`, 则规定重载对于符合条件的类型无效。虽然使用 SFINAE 也可以实现，但该库可以更好地表达相关意图。本章忽略了`enable_if` 和 `disable_if`的 lazy 版本(名为 `lazy_enable_if` 和 `lazy_disable_if`), 不过我在这里简单地提及一下。lazy 版本被用于避免实例化类型可能无效的情形(取决于条件的取值).

以下情形时使用 `enable_if` ：

*   你需要在把一个符合某些条件的函数加入到或排除出重载决议集合中。

*   你需要根据某个条件将一个类模板的特化版本加入到或排除出特化集合中。

# Utility 总结

## Utility 总结

本章介绍了几种工具类，它们可以大大简化我们的日常工作。`BOOST_STATIC_ASSERT` 提供编译期断言，它有助于我们测试前提条件或强制某些要求。对于泛型编程，`checked_delete` 在检查错误用法时非常有用，它可以节省我们大量的阅读可怕的错误信息和研究代码的时间。我们还讨论了 `addressof`, 它是一个获得对象真实地址的小工具，不用管 `operator&` 有否被重载。我们还看到了 `enable_if` 和 `disable_if` 如何控制某些函数参与重载决议，并学习了 SFINAE 有何意义！

我们也讨论了基类 `noncopyable`. 它既提供了好的习惯用法，也清楚地向任何看到这段代码的人表达了正确的意图，它值得你经常使用。总是在类中定义冗长的复制构造函数和赋值操作符，而不管它们是否需要定制，或是需要禁用，这种情况在很多代码中经常出现，它们浪费了太多的时间和金钱。

这是本书中最短的一章，我猜你一定很快就读完了。它很快就会给予你回报的，如果你马上开始使用这些工具的话。在 Boost.Utility 中还有一些其它的工具，我没在这里讨论它们。你可以访问 Boost 的网站，看看在线文档，找一下其它适合你当前工作的小工具。