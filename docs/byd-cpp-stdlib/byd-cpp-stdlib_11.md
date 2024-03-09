# Library 4\. Operators

*   Operators 库如何改进你程序？
*   Operators
*   用法
*   Operators 总结

# Operators 库如何改进你的程序？

## Operators 库如何改进你的程序？

*   提供一组完整的比较操作符

*   提供一组完整的算术操作符

*   提供一组完整的迭代器操作符

C++定义的操作符可以分成几组。当你在一个类中碰到某组操作符中的一个，通常你还会碰到同组中的其它操作符。例如，如果一个类提供了 `operator==`, 你通常还会看到 `operator!=` ，或许还有 `operator&lt;`, `operator&lt;=`, `operator&gt;`, 和 `operator&gt;=`. 有时，一个类仅提供了 `operator&lt;` 以定义一个次序，这个类的对象就可以用于关联容器，但通常忘了类的使用者想要更多的操作符。同样地，一个具有值语义的类可以只提供了 `operator+` 而没有 `operator+=` 或 `operator-` 会限制它的使用。当你为你的类定义了一组操作符中的一个时，你应该也提供该组中其余的操作符，以避免令人惊讶。不幸的是，为一个类增加多个操作符以支持比 较或算术运算是很麻烦并且容易出错的，还有，迭代器类必须根据它所依照的迭代器种类来提供一组特定的操作符以确保其功能正确。

定义多个所需的操作符除了沉闷以外，还必须确保它们的语义符合用户的期望。否则，这个类将没有任何实际的用途。但我们可以无须全部依靠手工来实现它们。如你所知，某些操作符是根据其它操作符实现的，如 `operator+` 就可以参照 `operator+=`实现，这意味着可以自动实现部分工作。事实上，这正是 Boost.Operators 的目的。它允许你只定义所需的比较或算术操作符的一个子集，然后基于你提供的操作符自动定义其它的操作符，Boost.Operators 保证了正确的操作符语义，并减少了你犯错的机会。

Operators 库的另一个好处在于为不同操作符给出了明确的概念命名，例如支持`operator+` 和 `operator+=`的类称为 addable，支持`operator&lt;&lt;` 和 `operator&gt;&gt;`的类称为 shiftable，等等。这很重要，有两个原因：一个统一的命名方法更为易懂；而且这些概念以及其后命名的类，可以是类接口的一部分，清晰地表明了重要的行为。

### Operators 如何配合标准库？

在使用标准库容器和算法时，通常至少要支持一些关系操作符(最常见的是 `operator&lt;`) 以提供排序，从而可以在关联容器中按顺序存储对象。常见的惯例是仅定义所需操作符的最小集合，其副作用是类的定义不够完整，也更难理解。另一方面，如果定 义全部完整的操作符，就会有语义错误的风险。这种情况下，Operators 库帮助我们确保类的行为是正确的，并同时满足标准库和用户的要求。最后，对于 定义了算术操作符的类型，也有一些操作符是依照其它操作符而实现的，所以 Boost.Operators 在这里也很有帮助。

# Operators

## Operators

### 头文件: `"boost/operators.hpp"`

Operators 库由多个基类组成。每一个类生成与其名字概念相关的操作符。你可以用继承的方式来 使用它们，如果你需要一个以上的功能，则需要使用多重继承。幸运的是，Operators 中定义了一些复合的概念，在大多数情况下可以无须使用多重继承。 下面将介绍最常用的一些 Operator 类，包括它们所表示的概念，以及它们对派生类的要求。某些情况下，使用 Operators 时，对真实概念的要求会 不同于对该概念基类的要求。例如，概念 addable 要求有一个操作符 `T operator+(const T& lhs,const T& rhs)` 的定义，而 Operators 的基类 `addable` 却要求有一个成员函数，`T operator+=(const T& other)`. 使用这个成员函数，基类 `addable` 为派生类自动增加了 `operator+`. 在以下章节中，都是首先给出概念，然后再给出对派生自该概念的类的要求。我没有重复本库中的所有概念，仅是挑选了最重要的一些；你可以在 [www.boost.org](http://www.boost.org) 上找到完整的参考文档。

#### less_than_comparable

`less_than_comparable` 要求类型`T`具有以下语义。

```cpp
bool operator<(const T&,const T&);
bool operator>(const T&,const T&);
bool operator<=(const T&,const T&);
bool operator>=(const T&,const T&); 
```

要派生自 `boost::less_than_comparable`, 派生类(`T`)必须提供：

```cpp
bool operator<(const T&, const T&); 
```

注意，返回值的类型不必是真正的 `bool`, 但必须可以隐式转换为 `bool`. C++标准中的概念 LessThanComparable 要求提供`operator&lt;` ，所以从 `less_than_comparable` 派生的类必须符合该要求。作为回报，`less_than_comparable` 将依照 `operator&lt;` 实现其余的三个操作符。

#### equality_comparable

`equality_comparable` 要求类型`T`具有以下语义。

```cpp
bool operator==(const T&,const T&);
bool operator!=(const T&,const T&); 
```

要派生自 `boost::equality_comparable`, 派生类(`T`)必须提供：

```cpp
bool operator==(const T&,const T&); 
```

同样，返回值的类型不必是 `bool`, 但必须可以隐式转换为 `bool`. C++标准中的概念 EqualityComparable 要求必须提供 `operator==` ，因此从 `equality_comparable` 派生的类必须符合该要求。`equality_comparable` 类为 `T` 提供 `bool operator!=(const T&,const T&)`.

#### addable

`addable` 概念要求类型`T`具有以下语义。

```cpp
T operator+(const T&,const T&);
T operator+=(const T&); 
```

要派生自 `boost::addable`, 派生类(`T`)必须提供：

```cpp
T operator+=(const T&); 
```

返回值的类型必须可以隐式转换为 `T`. 类 `addable` 为 `T` 实现 `T operator+(const T&,const T&)`.

#### subtractable

`subtractable` 概念要求类型`T`具有以下语义。

```cpp
T operator-(const T&,const T&);
T operator-=(const T&);  // 译注：原文为 T operator+=(const T&); 有误 
```

要派生自 `boost::subtractable`, 派生类(`T`)必须提供：

```cpp
T operator-=(const T&,const T&); 
```

返回值的类型必须可以隐式转换为 `T`. 类 `addable` 为 `T` 实现 `T operator-(const T&,const T&)`.

#### orable

`orable` 概念要求类型`T`具有以下语义。

```cpp
T operator|(const T&,const T&);
T operator|=(const T&,const T&); 
```

要派生自 `boost::orable`, 派生类(`T`)必须提供：

```cpp
T operator|=(const T&,const T&); 
```

返回值的类型必须可以隐式转换为 `T`. 类 `addable` 为 `T` 实现 `T operator|(const T&,const T&)`.

#### andable

`andable` 概念要求类型`T`具有以下语义。

```cpp
T operator&(const T&,const T&);
T operator&=(const T&,const T&); 
```

要派生自 `boost::andable`, 派生类(`T`)必须提供：

```cpp
T operator&=(const T&,const T&); 
```

返回值的类型必须可以隐式转换为 `T`. 类 `addable` 为 `T` 实现 `T operator&(const T&,const T&)`.

#### incrementable

`incrementable` 概念要求类型`T`具有以下语义。

```cpp
T& operator++(T&);
T operator++(T&,int); 
```

要派生自 `boost::incrementable`, 派生类(`T`)必须提供：

```cpp
T& operator++(T&); 
```

返回值的类型必须可以隐式转换为 `T`. 类 `addable` 为 `T` 实现 `T operator++(T&,int)`.

#### decrementable

`decrementable` 概念要求类型`T`具有以下语义。

```cpp
T& operator--(T&);
T operator--(T&,int); 
```

要派生自 `boost::decrementable`, 派生类(`T`)必须提供：

```cpp
T& operator--(T&); 
```

返回值的类型必须可以隐式转换为 `T`. 类 `addable` 为 `T` 实现 `T operator--(T&,int)`.

#### equivalent

`equivalent` 概念要求类型`T`具有以下语义。

```cpp
bool operator<(const T&,const T&);
bool operator==(const T&,const T&); 
```

要派生自 `boost::equivalent`, 派生类(`T`)必须提供：

```cpp
bool operator<(const T&,const T&); 
```

返回值的类型必须可以隐式转换为 `bool`. 类 `equivalent` 为 `T` 实现 `T operator==(const T&,const T&)`. 注意，等价(equivalence)和相等(equality)准确的说是不一样的；两个等价(equivalent)的对象并不一定是相等的(equal)。但对于这里的 `equivalent` 概念而言，它们是一样的。

### 解引用操作符

对于迭代器，有两个概念特别有用，`dereferenceable` 和 `indexable`, 分别表示了解引用的两种情况：一个是`*t`, `t`是一个支持解引用的迭代器(显然所有迭代器都支持)，另一个是 indexing, `t[x]`, `t`是一个支持下标操作符寻址的类型，而 `x` 通常是一个整数类型。在更高的抽象级别，它们两个通常一起使用，合称迭代器操作符，包括这两个解引用操作符和一些简单的算术操作符。

#### dereferenceable

`dereferenceable` 概念要求类型`T`具有以下语义，假设 `T` 是操作数，`R` 是引用类型，而 `P` 是指针类型(例如，`T` 是一个迭代器类型，`R` 是该迭代器的`value_type`的引用，而 `P` 则是该迭代器的`value_type`的指针)。

```cpp
P operator->() const;
R operator*() const; 
```

要派生自 `boost::dereferenceable`, 派生类(`T`)必须提供：

```cpp
R operator*() const; 
```

另外，`R`的一元 `operator&` 必须可以被隐式转换为 `P`. 这意味着 `R` 不必一定要是引用类型，它可以是一个代理类(proxy class)。类`dereferenceable` 为 `T` 实现 `P operator-&gt;() const`.

#### indexable

`indexable` 概念要求类型`T`具有以下语义，假设 `T` 是操作数，`R` 是引用类型，`P` 是指针类型，而 `D` 是 `difference_type` (例如，`T` 是一个迭代器类型，`R` 是该迭代器的`value_type`的引用，`P` 是该迭代器的`value_type`的指针，而 `D` 则是 `difference_type`)。

```cpp
R operator[](D) const;
R operator+(const T&,D); 
```

要派生自 `boost::indexable`, 派生类(`T`)必须提供：

```cpp
R operator+(const T&,D); 
```

类 `indexable` 为 `T` 实现 `R operator[](D) const`.

### 复合算术操作符

到目前为止我们看到的概念都只代表了最简单的功能。但是，还有一些高级的，或是复合的概念，它们由几个简单概念组合而成，或是在复合概念之上再增加简单的概念而成。例如，一个类是 `totally_ordered` 的，如果它同时是 `less_than_comparable` 的和 `equality_comparable`的。这些组合很有用，因为它们减少了代码的数量，同时还表明了重要且常用的概念。由于它们只是表示了已有概念的组合，所以这些概念很容易用一个表格来表示它们所包含的简单概念。例如，如果一个类派生自 `totally_ordered`, 它必须实现 `less_than_comparable` 所要求的操作符(`bool operator&lt;(const T&,const T&)`) 和 `equality_comparable` 所要求的操作符(`bool operator==(const T&,const T&)`)。

| 组合概念 | 由以下概念组成 |
| --- | --- |
| totally_ordered | less_than_comparableequality_comparable |
| additive | addablesubtractable |
| multiplicative | multipliabledividable |
| integer_multiplicative | multiplicativemodable |
| arithmetic | additivemultiplicative |
| integer_arithmetic | additiveinteger_multiplicative |
| bitwise | andableorablexorable |
| unit_steppable | incrementabledecrementable |
| shiftable | left_shiftableright_shiftable |
| ring_operators | additivemultipliable |
| ordered_ring_operators | ring_operatorstotally_ordered |
| field_operators | ring_operatorsdividable |
| ordered_field_operators | field_operatorstotally_ordered |
| euclidian_ring_operators | ring_operatorsdividablemodable |
| ordered_ euclidian_ring_operators | euclidean_ring_operatorstotally_ordered |

# 用法

## 用法

为了开始使用 Operators 库，为你的类实现适用的操作符，就要包含头文件`"boost/operators.hpp"`, 并从一个或多个 Operator 基类(它们的名字与它们所表示的概念一样)进行派生，它们都定义在名字空间 `boost`中。注意，继承不一定要是公有的，私有继承也可以。在这一节，我们将看到几个使用不同概念的例子，并关注一下在 C++里以及在概念上，算术操作符和关系操作符是如何工作的。作为第一个例子，我们定义一个类，`some_class`, 带有一个 `operator&lt;`. 我们决定把`operator&lt;`所暗指的等价关系定义为 `operator==`. 这个工作可以通过从`boost::equivalent`继承而完成。

```cpp
#include <iostream>
#include "boost/operators.hpp"

class some_class : boost::equivalent<some_class> {
  int value_;
public:
  some_class(int value) : value_(value) {}

  bool less_than(const some_class& other) const {
    return value_<other.value_;
  }
};

bool operator<(const some_class& lhs, const some_class& rhs) {
  return lhs.less_than(rhs);
}

int main() {
  some_class s1(12);
  some_class s2(11);

  if (s1==s2) 
    std::cout << "s1==s2\n";
  else
    std::cout << "s1!=s2\n";
} 
```

`operator&lt;` 依照成员函数 `less_than`实现。`equivalent` 基类的要求就是派生的类必须提供 `operator&lt;` 。从`equivalent`派生时，我们要把派生类`some_class`作为模板参数传送。在 `main` 里，使用了 Operators 库为我们生成的`operator==` 。接下来，我们再看一来 `operator&lt;` ，看看其它的关系操作符如何依照 less than 实现。

### 对比较操作符的支持

我们最常实现的关系操作符就是 less than，也就是 `operator&lt;`. 为了要把对象存入关联容器，或者是为了要排序，我们都要提供它。然而，通常我们仅支持这一个操作符，这样会把类的使用者弄糊涂。例如，多数人知道对`operator&lt;`的结果取反就相当于 `operator&gt;=`.[1] Less than 也可以用来计算 greater than, 等等。所以，一个支持 less than 关系的类的使用者有充足的理由相信，支持(至少隐式地支持)其它的比较操作符也应该是类的接口的一部分。唉，如果我们仅仅支持了 `operator&lt;` 而忽略了其它的，这个类就不是它可以的或者它应该的那样有用了。这里有一个类，它已经可以用于标准库容器的排序程序。

> [1] 尽管也有很多人以为是 `operator&gt;`!

```cpp
class thing {
  std::string name_;
public:
  thing() {}
  explicit thing(const std::string& name):name_(name) {}

  friend bool operator<(const thing& lhs, const thing& rhs) {
    return lhs.name_<rhs.name_;
  } 
}; 
```

这个类支持排序，也可以被存入关联容器中，但它可能还不能满足用户的期望！例如，如果一个用户需要知道 `thing a` 是否大于 `thing b`, 他就要这样写：

```cpp
// is a greater than b?
if (b<a) {} 
```

虽然这段代码是正确的，但是它未能清晰地表达作者的意图，而这对于代码的正确性是很重要的。如果用户想知道 `a` 是否小于或等于 `b`, 他不得不这样写：

```cpp
// is a less than, or equal to, b?
if (!(b<a)) {} 
```

同样，这段代码是正确的，但它会让人糊涂；对于多数不留意的读者来说，代码的意图真的很不清晰。如果要引入等价的概念，代码将变得更令人糊涂，而我们是支持等价关系的(否则我们的类不能存入关联容器中)。

```cpp
// is a equivalent to b?
if (!(a<b) && !(b<a)) {} 
```

请注意，等价和相等是不一样的，后面的章节将展开讨论这个主题。在 C++中，前面所述的所有关系特性都有不同的表示方式，它们是通过不同的操作符来明确地进行测试的。前面的例子应该是象这样(等价关系可能是个例外，但我们在这先不管它)：

```cpp
if (a>b) {}
if (a<=b) {}
if (a==b) {} 
```

现在，注释是多余的了，因为代码已经讲清楚了一切。照这个样子，代码是不能编译的，因为 `thing` 类不支持 `operator&gt;`, `operator&lt;=`, 或 `operator==`. 但是，对于具有`less_than_comparable`概念的类型，这些操作符(除了 `operator==`)都能被表达，Operators 库可以帮助我们。我们要做的全部工作就是让 `thing` 派生自 `boost::less_than_comparable`, 如下：

```cpp
class thing : boost::less_than_comparable<thing> { 
```

仅仅通过指定一个基类，就可以依照`operator&lt;`实现所有的操作符，`thing` 类现在可以按你所期望的那样工作了。如你所见，要从 Operators 库中的类派生出 `thing` ，我们必须把 `thing` 作为模板参数传递给基类。这种技术将在后面的章节里讨论。注意，`operator==` 对于支持 `less_than_comparable`的类并无定义，但我们还有一个概念可用，即 `equivalent`. 从 `boost::equivalent` 派生就可以增加 `operator==`, 但是要注意，这里的 `operator==` 是定义为等价关系，而不是相等关系。等价意味着严格的弱序关系[2]。我们最后版本的类 `thing` 看起来应该是这样的：

> [2] 如果你对严格弱序感到奇怪，可以跳到下一节，但是不要忘了稍后回到这里！

```cpp
class thing : 
  boost::less_than_comparable<thing>,
  boost::equivalent<thing> {

  std::string name_;
public:
  thing() {}
  explicit thing(const std::string& name):name_(name) {}

  friend bool operator<(const thing& lhs,const thing& rhs) {
    return lhs.name_<rhs.name_;
  } 
}; 
```

这个版本在`thing`的定义中仅给出了一个操作符，保持了定义的简洁，依靠派生自 `less_than_comparable` 和 `equivalent`, 它提供了一整组有用的操作符。

```cpp
bool operator<(const thing&,const thing&);
bool operator>(const thing&,const thing&);
bool operator<=(const thing&,const thing&);
bool operator>=(const thing&,const thing&);
bool operator==(const thing&,const thing&); 
```

你肯定见过很多提供了多个操作符的类。这些类的定义很难阅读，由于有太多的操作符函数的声明/实现。通过从`operators`中的概念类派生，你提供了相同的接口，但做得更清楚也更少代码。在类的定义中提及这些概念，可以使熟悉`less_than_comparable` 和 `equivalent`的读者清楚地知道这个类支持这些关系操作符。

### Barton-Nackman 技巧

在前面两个例子中，我们看到从 operator 基类继承的方法，一个看起来怪怪的地方是，把派生类传给基类作为模板参数。这是一种著名的技巧，被称为 Barton-Nackmann 技巧[3] 或 Curiously Recurring Template Pattern[4]。这种技巧所解决的问题是循环的依赖性。考虑一下实现一个泛型类，它为另一个定义了`operator&lt;`的类提供`operator==` 。顺便说一下，这就是这个库(当然还有 mathematics 库)中称为 `equivalent` 的概念。很明显，任何类要利用提供了这种服务的具体实现，它都要了解提供服务的这个类，我们以这个类所实现的概念来命名它，称之为 `equivalent` 类。然而，我们刚刚还在说 `equivalent` 要了解那个它要为之定义`operator==`的类！这是一种循环的依赖性，乍一看，好象没有办法可以解决。但是，如果我们把 `equivalent` 实现为类模板，然后指定那个要定义`operator==`的类为模板的参数，这样我们就已经有效地把相关类型，也即是那个派生类，加入到 `equivalent` 的作用域中了。以下例子示范了如何使用这个技巧。

> [3] 由 John Barton 和 Lee Nackmann "发明"。
> 
> [4] 由 James Coplien"发明"。

```cpp
#include <iostream>

template <typename Derived> class equivalent {
public:
  friend bool operator==(const Derived& lhs,const Derived& rhs) {
    return !(lhs<rhs) && !(rhs<lhs);
  }
};

class some_class : equivalent<some_class> {
  int value_;
public:
  some_class(int value) : value_(value) {}
  friend bool operator<(const some_class& lhs,
    const some_class& rhs) {
    return lhs.value_<rhs.value_;
  }
};

int main() {
  some_class s1(4);
  some_class s2(4);

  if (s1==s2)
    std::cout << "s1==s2\n";
} 
```

基类 `equivalent` 接受一个要为之定义`operator==`的类型为模板参数。它通过使用`operator&lt;`为该参数化类型实现泛型风格的`operator==`。然后，类 `some_class` 想要利用 `equivalent` 的服务，就从它派生并把自己作为模板参数传递给 `equivalent`。因此，结果就是为类型`some_class`定义了 `operator==` ，是依照 `some_class`的 `operator&lt;` 实现的。这就是 Barton-Nackmann 技巧的全部内容。这是一种简单且非常有用的模式，相当优美。

### 严格弱序(Strick Weak Ordering)

在本书中，我已经两次提到了严格弱序(strict weak orderings)，如果你不熟悉它，本节将离开主题一会，给你解释一下。严格弱序是两个对象间的一种关系。首先我们来一点理论的讲解，然后再具体地讨论。一个函数 `f(a,b)` 如果实现了一种严格弱序关系，这里的 `a` 和 `b` 是同一类型的两个对象，我们说， `a` 和 `b` 是等价的，如果`f(a,b)` 是 false 并且 `f(b,a)` 也是 false。这意味着 `a` 不在 `b`之前，而且 `b` 也不在 `a`之前。因此我们可以认为它们是等价的。此外，`f(a,a)` 必须总是 `false`[5]，而且如果 `f(a,b)` 为 `true`, 则 `f(b,a)` 必须为 `false`.[6] 还有，如果 `f(a,b)` 与 `f(b,c)` 均为 `true`, 则有 `f(a,c)`.[7] 最后，如果 `f(a,b)` 为 `false` 且 `f(b,a)` 也为 `false`, 并且如果 `f(b,c)` 为 `false` 且 `f(c,b)` 也为 `false`, 则 `f(a,c)` 为 `false` 且 `f(c,a)` 为 `false`.[8]

> [5] 即自反性。
> 
> [6] 即反称性。
> 
> [7] 即传递性。
> 
> [8] 即等价关系的传递性。

我们前面的例子(类 `thing`)可以有助于澄清这个理论。`thing`的小于比较是依照`std::string`的小于比较实现的。也就是说，是一种字面的比较。因此，给出一个包含字符串"First"的 `thing a` ，和一个包含字符串"Second"的 `thing b` ，还有一个包含字符串"Third"的 `thing c` ，我们可以 `assert` 前面给出的定义和公理。

```cpp
#include <cassert>
#include <string>
#include "boost/operators.hpp"

// Definition of class thing omitted

int main() {
  thing a("First");
  thing b("Second");
  thing c("Third");

  // assert that a<b<c 
  assert(a<b && a<c && !(b<a) && b<c && !(c<a) && !(c<b));

  // 等价关系
  thing x=a;
  assert(!(x<a) && !(a<x));

  // 自反性
  assert(!(a<a));

  // 反对称性
  assert((a<b)==!(b<a));

  // 传递性
  assert(a<b && b<c && a<c);

  // 等价关系的传递性
  thing y=x;
  assert( (!(x<a) && !(a<x)) && 
    (!(y<x) && !(x<y)) && 
    (!(y<a) && !(a<y))); 
} 
```

现在，所有这些 `assert`s 都成立，因为 `std::string` 实现了严格弱序[9]。就象 `operator&lt;` 可以定义一个严格弱序，`operator&gt;`也可以。稍后，我们将看到一个非常具体的例子，看看如果我们未能区分等价(它是一个严格弱序所要求的)与相等(它不是严格弱序所要求的)之间的不同，将会发生什么。

> [9] 事实上，`std::string` 定义了一个全序，全序即是严格弱序外加一个要求：等价即为相等。

### 避免对象膨胀

在前面的例子中，我们的类派生自两个基类：`less_than_comparable&lt;thing&gt;` 和 `equivalent&lt;thing&gt;`. 根据你所使用的编译器，你需要为这个多重继承付出一定的代价；`thing` 可能要比它所需的更大。标准允许编译器使用空类优化来创建一个没有数据成员、没有虚拟函数、也没有重复基类的基类，这样在派生类的对象中只会占用零空间， 而多数现代的编译器都会执行这种优化。不幸的是，使用 Operators 库常常会导致从多个基类进行继承，这种情况下只有很少编译器会使用空类优化。为了 避免潜在的对象大小膨胀，Operators 支持一种称为基类链(base class chaining) 的技术。每个操作符类接受一个可选的额外的模板参数，该参数来自于它的派生类。采用以下方法：一个概念类派生自另一个，后者又派生自另一个，后者又派生自 另一个…(你应该明白了吧)，这样就不再需要多重继承了。这种方法很容易用。不要再从几个基类进行继承了，只要简单地把几个类链在一起就行 了，如下所示：

```cpp
// Before
boost::less_than_comparable<thing>,boost::equivalent<thing>
// After
boost::less_than_comparable<thing,boost::equivalent<thing> > 
```

这种方法避免了从多个空基类进行继承，而从多个基类继承可能会阻碍你的编译器进行空类优化，使用从一 个空基类链进行继承的方法，增加了编译器进行空类优化的机会，也减少了派生类的大小。你可以用你的编译器做一下试验，看看你可以从这个技术中获得多少好 处。注意，基类链长度是有限制的，这取决于编译器。对于程序员可以接受的基类链长度也是很有限的！这意味着对那些需要从很多 operator 类进行继承的类来说，我们需要把它们组合起来。更好的方法是，使用 Operators 库已提供的复合概念。

以我的测试来看，在某个对多重继承不执行空类优化的流行编译器上[10]， 使用基类链和使用多重继承所得到的类的大小有很大的差别。使用基类链确实可以避免类型增大的负作用，而使用多重继承则会有类型的增大，对于一个普通类型， 大小将增加 8 个字节(无可否认，8 个额外的字节对于多数应用来说并不是问题)。如果被包装的类型的大小非常小，那么多重继承带来的额外开销就不是可以接受 的了。由于基类链很容易使用，我们应该在所有情况下都使用它！

> [10] 我这样说一方面是因为没有必要讲出它的名字，另一方面也因为每个人都知道我说的是 Microsoft 的老编译器 (他们的新编译器可能不是)。

### Operators 与不同的类型

有时候，一个操作符要包括一个以上的类型。例如，考虑一个字符串类，它支持通过`operator+` 和 `operator+=` 从字符数组进行字符串连接。这种情况下，Operators 库也可以帮忙，使用双参数版本的操作符模板。这个字符串类可能拥有一个接受`char*`的转换构造函数，但正如我们将看到的，这不能解决这个类的所有问题。以下是我们要用的字符串类。

```cpp
class simple_string {
public:
  simple_string();

  explicit simple_string(const char* s);
  simple_string(const simple_string& s);

  ~simple_string();

  simple_string& operator=(const simple_string& s);

  simple_string& operator+=(const simple_string& s);
  simple_string& operator+=(const char* s);

  friend std::ostream& 
    operator<<(std::ostream& os,const simple_string& s);
}; 
```

如你所见，我们为`simple_string`增加两个版本的 `operator+=` 。一个接受 `const simple_string&`, 另一个接受 `const char*`. 这样，我们的类支持如下用法：

```cpp
simple_string s1("Hello there");
simple_string s2(", do you like the concatenation support?");
s1+=s2;
s1+=" This works, too"; 
```

虽然前面的工作符合要求，但我们还没有提供二元的`operator+`，这个疏忽肯定是类的使用者所不乐意的。注意，对于我们的`simple_string`，我们可以通过忽略显式的转换构造函数来允许字符串连接。但是，这样做会导致对字符缓冲的一次额外(不必要)的复制，而仅仅节省了一个操作符的定义。

```cpp
// 以下不能编译
simple_string s3=s1+s2;
simple_string s4=s3+" Why does this class behave so strangely?"; 
```

现在让我们来用 Operators 库来为这个类提供漏掉的操作符。注意共有三个操作符没有提供。

```cpp
simple_string operator+(const simple_string&,const simple_string&);
simple_string operator+(const simple_string& lhs, const char* rhs);
simple_string operator+(const char* lhs, const simple_string& rhs); 
```

如果手工定义这些操作符，很容易就会忘记那个接受一个 `const simple_string&` 和一个 `const char*`的重载。而使用 Operators 库，你就不会忘记了，因为库已经为你实现了这些漏掉的操作符！我们想为 `simple_string` 做的就是加一个 addable 概念，所以我们只要简单地从`boost::addable&lt;simple_string&gt;`派生 `simple_string` 就行了。

```cpp
class simple_string : boost::addable<simple_string> { 
```

但是，在这个例子中，我们还想要一个允许`simple_string` 和 `const char*`混合使用的操作符。为此，我们需要指定两个类型，结果类型是`simple_string`, 以及第二参数类型是 `const char*`. 我们可以利用基类链来避免增大类的大小。

```cpp
class simple_string :     
  boost::addable<simple_string,
    boost::addable2<simple_string,const char*> > { 
```

这就是为了支持我们想要的全部操作符所要做的全部事情！如你所见，我们用了一个不同的 operator 类：`addable2`. 如果你用的编译器支持模板偏特化，你就不需要限定这个名字；你可以用 `addable` 代替 `addable2`. 为了对称性，还有一个版本的类，它带有后缀"1"。它可以增加可读性，它总是明确给出参数的数量，它带给我们以下对`simple_string`的派生写法：

```cpp
class simple_string :     
  boost::addable1<simple_string,
    boost::addable2<simple_string,const char*> > { 
```

选择哪种写法，完全取决于你的品味，如果你的编译器支持模板偏特化，最简单的选择是忽略所有后缀。

```cpp
class simple_string :     
  boost::addable<simple_string,
    boost::addable<simple_string,const char*> > { 
```

### 相等与等价的区别

为类定义关系操作符时，很重要的一点是分清楚相等和等价。要使用关联容器，就要求有等价关系，它通过概念 LessThanComparable[11]定义了一个严格弱序。这个关系只需最小的假设，并且对于要用于标准库容器的类型来说，这是最低的要求。但是，相等与等价之间的区别有时会令人混淆，弄明白它们之间的差别是很重要的。如果一个类支持概念 LessThanComparable, 通常它也就支持等价的概念。如果两个元素进行比较，没有一个比另一个小，我们称它们是等价的。但是，等价并不意味着相等。例如，有可能在一个 less than 关系中忽略某些特性，但并不意味着它们就是相等的[12]。为了举例说明这一点，我们来看一个类，`animal`, 它同时支持等价关系和相等关系。

> [11] 大写的概念，如 LessThanComparable 直接来自于 C++标准。所有 Boost.Operators 中的概念使用小写名字。
> 
> [12] 这意味着一个严格弱序，但不是一个全序。

```cpp
class animal : boost::less_than_comparable<animal, 
boost::equality_comparable<animal> > {
  std::string name_;
  int age_;
public:
  animal(const std::string& name,int age)
    :name_(name),age_(age) {}

  void print() const {
    std::cout << name_ << " with the age " << age_ << '\n';
  }   

  friend bool operator<(const animal& lhs, const animal& rhs) {
    return lhs.name_<rhs.name_;
  }

  friend bool operator==(const animal& lhs, const animal& rhs) {
    return lhs.name_==rhs.name_ && lhs.age_==rhs.age_;
  }

}; 
```

请注意 `operator&lt;` 和 `operator==`的实现间的区别。在 less than 关系中仅使用了动物的名字，而在相等检查中则同时比较了名字和年龄。这种方法没有任何错误，但是它会导致有趣的结果。现在让我们把这个类的一些元素存入 `std::set`. 和其它关联容器一样，`set` 仅依赖于概念 LessThanComparable. 以下面例子中，我们创建四个不一样的动物，然后试图把它们插入一个 `set`, 完全假装我们不知道相等和等价之间的差别。

```cpp
#include <iostream>
#include <string>
#include <set>
#include <algorithm>
#include "boost/operators.hpp"
#include "boost/bind.hpp"

int main() {
  animal a1("Monkey", 3);
  animal a2("Bear", 8);
  animal a3("Turtle", 56);
  animal a4("Monkey", 5);

  std::set<animal> s;
  s.insert(a1);
  s.insert(a2);
  s.insert(a3);
  s.insert(a4);

  std::cout << "Number of animals: " << s.size() << '\n';
  std::for_each(s.begin(),s.end(),boost::bind(&animal::print,_1));
  std::cout << '\n';

  std::set<animal>::iterator it(s.find(animal("Monkey",200)));
  if (it!=s.end()) {
    std::cout << "Amazingly, there's a 200 year old monkey "
      "in this set!\n";
    it->print();
  }

  it=std::find(s.begin(),s.end(),animal("Monkey",200));
  if (it==s.end()) {
    std::cout << "Of course there's no 200 year old monkey "
      "in this set!\n";
  }
} 
```

运行这个程序，会有以下完全荒谬的输出结果。

```cpp
Number of animals: 3
Bear with the age 8
Monkey with the age 3
Turtle with the age 56

Amazingly, there's a 200 year old monkey in this set!
Monkey with the age 3
Of course there's no 200 year old monkey in this set! 
```

问题不在于猴子的年龄——它的确不寻常——而在于没有了解这两种关系概念间的区别。首先，当四个 `animal`s (`a1`, `a2`, `a3`, `a4`)被插入到 `set`, 第二只猴子，`a4`, 其实并没有被插入，因为 `a1` 和 `a4` 是等价的。原因是，`std::set` 使用表达式 `!(a1&lt;a4) && !(a4&lt;a1)` 来决定是否已有一个匹配的元素。由于这个表达式的结果为 `true` (我们的 `operator&lt;` 不比较年龄), 所以插入失败了[13]。然后，当我们询问这个 set，使用`find`查找一个 200 岁的猴子时，它找到了这样一只怪物。同样，这是由于`animal`的等价关系，仅依赖于`animal`的`operator&lt;` ，因而还是不关心年龄。最后，我们再次用 `find` 在 `set` 中定位这只猴子(`a1`), 但这次我们调用 `operator==` 来判断它是否匹配，从而没有找到匹配的猴子。通过对这些猴子的讨论，不难理解相等与等价之间的差别，但你必须知道在给定的上下文中使用的是哪一个概念。

> [13] 一个 set, 根据定义, 不存在重复的元素。

### 算术类型

定义算术类型时，Operators 库尤其有用。对于一个算术类型，通常有很多操作符要定义，而手工 去做这些工作是一项令人畏缩和沉闷的任务，并很可能发生错误或疏忽。Operators 库中定义的概念使这项工作变得容易，你只需为类定义最少的必须的操 作符，剩下的操作符就可以自动实现。考虑一个支持加法和减法的类。假设这个类使用一个内建类型作为实现。现在要增加适当的操作符，并确保它们不仅可以用于 这个类的实例，还可以用于可转换为这个类的内建类型。你将要提供 12 个不同的加法和减法操作符。当然，更容易(也是更安全)的方法是，使用`addable` 和 `subtractable`类的双参数形式。现在假设你还需要增加一组关系操作符。你可能要自己增加 10 个操作符，但现在你知道了最容易的方法是使用 `less_than_comparable` 和 `equality_comparable`. 这样做之后，你拥有了 22 个操作符而只定义了 6 个。然而，你可能也注意到了这些概念对于数值类型来说是很常见的。的确如此，作为这四个类的替换，你可以仅使用 `additive` 和 `totally_ordered`.

我们先从四个概念类进行派生开始：`addable`, `subtractable`, `less_than_comparable`, 和 `equality_comparable`. 类`limited_type`, 仅仅包装了一个内建类型并将所有操作符前转给那个类型。它限制可用操作的数量，仅提供了关系操作符和加减法。

```cpp
#include "boost/operators.hpp"

template <typename T> class limited_type : 
  boost::addable<limited_type<T>,
    boost::addable<limited_type<T>,T,
      boost::subtractable<limited_type<T>,
        boost::subtractable<limited_type<T>,T,
          boost::less_than_comparable<limited_type<T>,
            boost::less_than_comparable<limited_type<T>,T,
              boost::equality_comparable<limited_type<T>,
                boost::equality_comparable<limited_type<T>,T >
> > > > > > > {

  T t_;
public:
  limited_type():t_() {}
  limited_type(T t):t_(t) {}

  T get() {
    return t_;
  }

  // 为 less_than_comparable 提供
  friend bool operator<(
      const limited_type<T>& lhs, 
      const limited_type<T>& rhs) {
    return lhs.t_<rhs.t_;
  }

  // 为 equality_comparable 提供
  friend bool operator==(
      const limited_type<T>& lhs, 
      const limited_type<T>& rhs) {
    return lhs.t_==rhs.t_;
  }

  // 为 addable 提供
  limited_type<T>& operator+=(const limited_type<T>& other) {
    t_+=other.t_;
    return *this;
  }

  // 为 subtractable 提供
  limited_type<T>& operator-=(const limited_type<T>& other) {
    t_-=other.t_;
    return *this;
  }
}; 
```

这是一个不错的例子，示范了使用 Operators 库后实现变得多么容易。仅需实现几个必须实现的操 作符，就很容易地获得了全组的操作符，而类也变得更易懂以及更易于维护。(即使实现这些操作符是困难的，你也可以把注意力集中于正确地实现它们)。这个类 唯一的潜在问题就是，它派生自八个不同的 operator 类，使用了基类链的方式，对于某些人而言，这可能不好阅读。我们可以通过使用复合概念来大大简化 我们的类。

```cpp
template <typename T> class limited_type : 
  boost::additive<limited_type<T>, 
    boost::additive<limited_type<T>,T, 
      boost::totally_ordered<limited_type<T>, 
        boost::totally_ordered<limited_type<T>,T > > > >  { 
```

这更好看，而且也减少了击键的次数。

### 仅在应用使用 Operators 时使用它

很明显 operators 应该仅在适当的时候使用，但出于某些原因，operators 的某些"很酷 的因素"常常诱使一些人在不清楚它们的语义时就使用它们。很多情形下都需要操作符，如在同一类型的实例间存在某种关系时，又或者在创建一个算术类型时。但 也有一些不太清晰的情形，你就需要考虑使用者的真正期望，如果用户的期望是模糊的，最好还是选择用成员函数。

多年以来，Operators 已经被用于一些不平常的服务。增加操作符用于连接字符串，和使用位移操作符进行 I/O，就是两个操作符不再具有数学意义而被用于其它语义用途的最常见的例子。也有人对于在`std::map`中使用下标操作符访问元素表示质疑(当 然其它人认为这是很自然的。他们是对的)。有时候，把操作符用于某种与内建类型规则不一致的用途是有意义的。而其它时候，它则是非常错误的，会引起混乱和 歧义。当你决定将一个操作符重载为与内建类型不一致的意义时，你必须很小心地去做。你必须确保它的意义是明显的，并且优先级是正确的。这也是在 IOStream 库中使用位移操作符进行 I/O 的原因。位移操作符清晰地表明了将某物移向某个方向，并且位移操作符的优先级比多数操作符都低。如果你创建 一个表示汽车的类，可能发现 `operator-=` 很方便。但是，对于使用者这个操作符意味着什么？有些人可能认为它被用来表示在驾驶中使用的汽油。其它人可能认为它被用来表示汽车价值的贬值(当然会计师 会这样想)。增加这个操作符是错误的，因为它没有一个清晰的意图，而一个成员函数可以更清楚地为这些操作命名。不要仅仅为了它可以写出"酷"的代码而增加 操作符。要因为它们真的有用而增加它们，确认增加所有适用的操作符，并且确认使用 Boost.Operators 库！

### 弄明白它是如何工作的

我们现在来看一看这个库是如何工作的，以进一步加深你对于如何正确使用它的理解。对于 Boost.Operators, 这并不难。我们来看看如何实现对 `less_than_comparable` 的支持。你需要了解你要增加支持的那个类，并且你要为这个类增加操作符，这个操作符将用于实现该类的其它相关操作符。`less_than_comparable` 要求我们提供 `operator&lt;`, `operator&gt;`, `operator&lt;=`, 和 `operator&gt;=`. 现在，你已经知道如何依照`operator&lt;`来实现 `operator&gt;`, `operator&lt;=`, and `operator&gt;=` 。下面是一种可能的实现方法。

```cpp
template <class T>
class less_than1
{
public:
  friend bool operator>(const T& lhs,const T& rhs)  { 
    return rhs<lhs; 
  }

  friend bool operator<=(const T& lhs,const T& rhs) { 
    return !(rhs<lhs); 
  }

  friend bool operator>=(const T& lhs,const T& rhs) { 
    return !(lhs<rhs); 
  }
}; 
```

对于 `operator&gt;`, 你只需要交换两个参数的顺序。对于 `operator&lt;=`, 注意到 `a&lt;=b` 即意味着 `b` 不小于 `a`. 因此，实现的方法就是以相反的参数顺序调用 `operator&lt;` 并对结果取反。对于 `operator&gt;=`, 同样由于 `a&gt;=b` 意味着 `a` 不小于 `b`. 因此，实现方法就是对调用 `operator&lt;` 的结果取反。这是一个可以工作的例子：你可以直接使用它并且它将完成正确的工作。然而，如果可以提供一个支持`T`与兼容类型之间进行比较的版本就更好了，这只要简单地增加几个重载就可以了。出于对称性的考虑，你应该允许兼容类型出现在操作符的左边(这一点在手工增加操作符时很容易忘记；人们通常仅留意到操作符的右边要接受其它类型。当然，你的双类型版本 `less_than` 不会犯如此愚蠢的错误，对吗？)

```cpp
template <class T,class U>
class less_than2
{
public:
  friend bool operator<=(const T& lhs,const U& rhs) { 
    return !(lhs>rhs); 
  }

  friend bool operator>=(const T& lhs,const U& rhs) { 
    return !(lhs<rhs); 
  }

  friend bool operator>(const U& lhs,const T& rhs) {
    return rhs<lhs; 
  }

  friend bool operator<(const U& lhs,const T& rhs)  { 
    return rhs>lhs; 
  }

  friend bool operator<=(const U& lhs,const T& rhs) { 
    return !(rhs<lhs); 
  }

  friend bool operator>=(const U& lhs,const T& rhs) { 
    return !(rhs>lhs); 
  }
}; 
```

这就是了！两个功能完整的 `less_than` 类。当然，要与 Operators 库中的 `less_than_comparable` 具有同样的功能，我们必须去掉表示使用几个类型的后缀。我们真正想要的是一个版本，或者说至少是一个名字。如果你使用的编译器支持模板偏特化，你就是幸运 的，因为基本上只要几行代码就可以实现了。但是，还有很多程序员没有这么幸运，所以我们还是要用健壮的方法来实现它，以完全避开偏特化。首先，我们知道我 们需要某个东西用来调用 `less_than`, 它是一个接受一个或两个类型参数的模板。我们也知道第二个类型是可选的，我们可以给它加一个缺省类型，我们知道用户不会传递这样一个类型给这个模板。

```cpp
struct dummy {};
template <typename T,typename U=dummy> class less_than {}; 
```

我们需要某种机制来选择正确版本的`less_than` (`less_than1` 或 `less_than2`)；我们可以无需借助模板偏特化，而通过使用一个辅助类来做到，这个辅助类有一个类型参数，并内嵌一个接受另一个类的嵌套模板 `struct` 。然后，使用全特化，我们可以确保类型 `U` 是 `dummy`时，`less_than1` 将被选中。

```cpp
template <typename T> struct selector {
  template <typename U> struct type {
    typedef less_than_2<U,T> value;
  };
}; 
```

前面这个版本创建了一个名为 `value` 的类型定义，这个类型正是我们已经创建的 模板的一个正确的实例化。

```cpp
template<> struct selector<dummy> {
  template <typename U> struct type {
    typedef less_than1<U> value;
  };
}; 
```

全特化的 `selector` 创建了另一个版本`less_than1`的 `typedef` 。为了让编译器更容易做，我们将创建另一个辅助类，专门负责收集正确的类型，并把它存入适当的 typedef `type`.

```cpp
template <typename T,typename U> struct select_implementation {
  typedef typename selector<U>::template type<T>::value type;
}; 
```

这种语法看上去不讨人喜欢，因为`selector`类中的嵌套模板 `struct`，但类的使用者并不需要看到这段代码，所以这不是什么大问题。现在我们有了所有的因素，我们需要从中选择一个正确的实现，我们最终从`select_implementation&lt;T,U&gt;::type`派生`less_than`，前者将会是 `less_than1` 或 `less_than2`, 这取决于用户给出了一个还是两个类型。

```cpp
template <typename T,typename U=dummy> class less_than : 
  select_implementation<T,U>::type {}; 
```

就是它了！我们现在有了一个完全可用的 `less_than`, 由于我们付出的额外努力，增加了一种检测并选择正确的实现版本的机制，用户现在可以以最容易的方式来使用它。我们还正确地了解了 `operator&lt;` 如何用于创建一个`less_than_comparable`类所用的其它操作符。对其它操作符完成同样的任务只需要小心行事，并弄清楚不同的操作符是如何共同组成新的概念的就行了。

### 剩下的事情

我们还没有讨论 Operators 库中的剩余部分，迭代器助手类。我不想给出示例了，因为你主要是在 定义迭代器类型时会用到它们，这需要额外的解释，这超出了本章甚至是本书的范围。我在这里之所以提及它们，是因为如果你正在定义迭代器类型而不借助于 Boost.Iterators 的话，你肯定会想用它些助手类的。解引用操作符帮助你定义正确的操作符而无须顾及你是否在需要一个代理类。在定义智能指针 是它们也很有用，智能指针通常要求定义 `operator-&gt;` 和 `operator*`. 迭代器助手类把不同类型的迭代器所需的概念组合在了一起。例如，一个随机访问迭代器必须是 `bidirectional_iterable`, `totally_ordered`, `additive`, 和 `indexable`的。定义新的迭代器类型时，更适当的做法是借助于 Boost.Iterator 库，不过 Operators 库也可以帮忙。

# Operators 总结

## Operators 总结

为用户自定义类型提供一组正确的关系操作符和算术操作符是非常重要的，而且正确地实现它也是一个重大 的挑战。通过使用 Operators 库，这个任务大大地简化了，正确性和对称性也随之而来。除此之外，这个库还提供了一组完整的操作符定义，这些类所支持 的概念被适当地命名和定义，可以在定义你的类时明确这些概念(也是通过 Operators 库！)。在本章中，我们已经看了几个例子，关于如何使用这个库来 改进带有操作符的程序，使程序更为简单，正确性也更有保证。一个可悲的事实是，为用户自定义类型提供重要的关系操作符和算术操作符常常被忽略掉，部分的原 因是由于为了正确获得它们需要做大量的工作。现在这种情形不会再出现了，因为有了 Boost.Operators。

在提供关系操作符和算术操作符时要重点考虑的一点是，首先要确认提供它们是有必要的。当类型间存在顺 序关系时，或者在创建数值类型时，总是需要提供操作符的，但对于其它类型，操作符可能就不能清晰地传递设计的意图。操作符几乎总是提供语法上的好处，这种 语法上的好处不应被低估。不幸的是，操作符又是诱人的。明智地使用它们，它们就会发挥巨大的威力。当你决定为一个类增加操作符， Boost.Operators 库可以为你的工作提高质量和效率。结论是，你应该在深思熟虑之后再决定是否给你的类增加操作符，当你决定要增加时，就使用 Operators 库。

Operators 库是多个人的贡献的结果。它从 David Abrahams 开始，并接受了 Jeremy Siek, Aleksey Gurtovoy, Beman Dawes, 和 Daryle Walker 等人的有价值的补充。正如多数 Boost 库一样，无数其它人的贡献才形成了今天这个库。