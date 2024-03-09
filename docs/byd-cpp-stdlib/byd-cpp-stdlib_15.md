# Library 7\. Variant

*   Variant 库如何改进你的程序？
*   Variant 如果适用于标准库？
*   Variant
*   用法
*   Variant 总结

# Variant 库如何改进你的程序？

## Variant 库如何改进你的程序？

*   对用户指定的多种类型的进行类型安全的存储和取回

*   在标准库容器中存储不同类型的方法

*   变量访问的编译期检查

*   高效的、基于栈的变量存储

Variant 库关注的是对一组限定类型的类型安全存储及取回，即非无类的联合。Boost.Variant 库与 Boost.Any 有许多共同之外，但在功能上也有不同的考虑。在每天的编程中通常都会需要用到非无类的联合(不同的类型)。保持类型安全的一个典型方法是使用抽象基类，但 这不总是可以做到的；即使可以做得，堆分配和虚拟函数[1]的代价也可能太高。你也可以尝试用不安全的无类类型，如 `void*` (它会导致不幸)，或者是类型安全得无限制的可变类型，如 Boost.Any. 这里我们将看到 Boost.Variant，它支持限定的可变类型，即元素来自于一组支持的类型。

> [1] 尽管虚拟函数在性能方面有非常合理的代价。

许多其它的编程语言支持可变类型，它们也再次被证实是值得的。在 C++内建的对可变类型的支持非常有限，只有某种形式的联合(union)，而且主要是为了与 C 兼容而保留。Boost.Variant 通过一个类型模板 `variant` 补救了这种情形，并附随有安全的存储及取回值的工具。一个可变数据类型提供一个与当前值的类型无关的接口。如果你曾经用过别的可变类型，可能是仅能支持固定的一组类型。这个库不是这样的；你在使用 `variant` 时自己定义一组允许使用的类型，而一个程序中可以包含任意个不同的 `variant` 实例。为了取回保存在 `variant` 中的值，你要么知道当前值的真实类型，要么使用已提供的类型安全的访问者(visitor)机制。访问者机制使得 Variant 非常不同于其它可变类型的库，包括 Boost.Any (它可以持有任意类型的值)，从而为处理这些类型提供了一个安全而健壮的环境。C++ 的联合只对内建类型以及 POD 类型有用，但这个库提供的非无类联合可以支持所有类型。最后，效率方面也被考虑到了，这个库基于栈存储来保存它的值，从而避免了昂贵的堆分配。

# Variant 如何适用于标准库？

## Variant 如何适用于标准库？

Boost.Variant 允许在标准库容器中存储不同的类型。由于在 C++或 C++标准库中都没有对可变类型的真正支持，这使得 Variant 成为了标准库的一个杰出且有用的扩充。

# Variant

## Variant

### 头文件: `"boost/variant.hpp"`

通过单个头文件就包含了所有 Variant 库。

```cpp
"boost/variant/variant_fwd.hpp" 
```

包含了 `variant` 类模板的前向声明。

```cpp
"boost/variant/variant.hpp" 
```

包含了 `variant` 类模板的定义。

```cpp
"boost/variant/apply_visitor.hpp" 
```

包含了对 `variant` 应用访问者机制的功能。

```cpp
"boost/variant/get.hpp" 
```

包含了模板函数 `get`.

```cpp
"boost/variant/bad_visit.hpp" 
```

包含了异常类 `bad_visit` 的定义。

```cpp
"boost/variant/static_visitor.hpp" 
```

包含了 `visitor` 类模板的定义。

以下部分摘要包含了 `variant` 类模板中最重要的成员。其它功能，如访问者机制，类型安全的直接取回，还有更先进的特性，如通过类型列表创建类型组等等，在 "Usage" 节讨论。

```cpp
namespace boost {
  template <typename T1,typename T2=unspecified, ...,
    typename TN=unspecified>
  class variant {
  public:

    variant();

    variant(const variant& other);

    template <typename T> variant(const T& operand);

    template <typename U1, typename U2, ..., typename UN>
      variant(const variant<U1, U2, ..., UN>& operand);

    ~variant();

    template <typename T> variant& operator=(const T& rhs);

    int which() const;
    bool empty() const;
    const std::type_info& type() const;

    bool operator==(const variant& rhs) const;
    bool operator<(const variant& rhs) const;
  };
} 
```

### 成员函数

```cpp
variant(); 
```

这个构造函数对 `variant` 的类型组中的第一个类型进行缺省构造。这意味着在声明 `variant` 类型时，第一个类型必须是可以被缺省构造的，或者 `variant` 类型本身不能被缺省构造。该构造函数传播任何从第一个类型的构造函数抛出的异常。

```cpp
variant(const variant& other); 
```

这个复制构造函数复制 `other` 的当前值，并传播任何从 `other` 的当前类型的复制构造函数抛出的异常。

```cpp
template <typename T> variant(const T& operand); 
```

这个构造函数从 `operand` 构造一个新的 `variant` 。operand 的类型 `T`, 必须可以转换为限定类型组中的某个类型。复制或转换 operand 时抛出的异常将被传播。

```cpp
template <typename U1,typename U2,...,typename UN>
  variant(const variant<U1,U2,...,UN>& operand); 
```

这个构造函数允许从另一个 `variant` 类型进行构造，后者的类型组为 `U1`, `U2`…`UN`, 它们必须可以转换为 `T1`,`T2`…`TN` (被构造的 variant 的类型组)。复制或转换 operand 时抛出的异常将被传播。

```cpp
~variant(); 
```

销毁 variant, 并调用当前值的析构函数。注意，对于指针类型，不调用析构函数(销毁指针是无操作的)。析构函数不抛出异常。

```cpp
template <typename T> variant& operator=(const T& rhs); 
```

这个操作符放弃当前值，并赋予值 `rhs`. 类型 `T` 必须可以转换为 `variant` 的限定类型组中的某个类型。如果 `T` 正好是 `variant` 当前值的类型，`rhs` 被复制赋值给当前值；从 `T` 的赋值操作符抛出的异常将被传播。如果 `variant` 当前值的类型不是 `T`, 则当前值被替换为从类型 `T` 的(复制)构造函数所创建的值。从构造函数抛出的异常将被传播。这个函数还可能抛出 `bad_alloc`.

```cpp
int which() const; 
```

返回一个从零起计的索引，表示当前值类型在限定类型组中的位置。这个函数不会抛出异常。

```cpp
bool empty() const; 
```

这个函数永远返回 `false`, 因为一个 `variant` 永远不会为空。这个函数的存在是为了允许泛型代码把 `variant` 和 `boost::any` 视为同一种类型来处理。这个函数不会抛出异常。

```cpp
const std::type_info& type() const; 
```

返回当前值的 `type_info` 。这个函数不会抛出异常。

```cpp
bool operator==(const variant& rhs) const; 
```

如果 `*this` and `rhs` 相等则返回 `true` ，即 `which()==rhs.which()` 且 `*this` 的当前值与 `rhs` 根据当前值的类型的相等操作是相等的。这要求限定类型组中的所有类型都必须是可以进行等同性比较的(EqualityComparable)。当前值的类型的 `operator==` 抛出的任何异常将被传播。

```cpp
bool operator<(const variant& rhs) const; 
```

小于比较返回 `which()&lt;rhs.which()` 或者，如果该索引相等，则返回对 `*this` 的当前值与 `rhs` 调用 `operator&lt;` 所返回的结果。当前值的类型的 `operator&lt;` 抛出的任何异常将被传播。

# 用法

## 用法

在你的程序中使用 `variant`，要包含头文件 `"boost/variant.hpp"`。这个头文件包含了整个库，所以你不必知道要使用哪些单独的特性；以后，如果你要降低相关性，可以只包含那些解决问题所要的头文件。声明一个 `variant` 类型时，我们必须定义一组它可以存储的类型。最常用的办法是使用模板参数。一个可以持有类型为 `int`, `std::string`, 或 `double` 的值的 `variant` 声明如下。

```cpp
boost::variant<int,std::string,double> my_first_variant; 
```

当变量 `my_first_variant` 被创建时，它含有一个缺省构造的 `int`, 因为 `int` 是这个 `variant` 可以持有的类型中的第一种类型。我们也可以传递一个可以转换为可用类型之一的值来初始化 `variant`.

```cpp
boost::variant<int,std::string,double>
 my_first_variant("Hello world"); 
```

我们可以随时赋给新的值，只要这个新值有确定的类型并且可以转换为 `variant` 可以持有的类型中的某一种，它可以很好地工作。

```cpp
my_first_variant=24;
my_first_variant=2.52;
my_first_variant="Fabulous!";
my_first_variant=0; 
```

在第一个赋值后，所含值的类型为 `int`; 第二个赋值后，类型为 `double`; 第三个后，类型为 `std::string`; 最后，又变回 `int`. 如果我们想看看，我们可以用函数 `boost::get` 取出这个值，如下：

```cpp
assert(boost::get<int>(my_first_variant)==0); 
```

注意，如果调用 `get` 失败(当 `my_first_variant` 所含值不是类型 `int` 时就会发生)，会抛出一个类型为 `boost::bad_get` 的异常。为了避免在失败时得到一个异常，我们可以传给 `get` 一个 `variant` 指针，这样 `get` 将返回一个指向它所含值的指针，或者如果给定类型与 `variant` 的值的类型不符则返回空指针。以下是它的用法：

```cpp
int* val=boost::get<int>(&my_first_variant);
assert(val && (*val)==0); 
```

函数 `get` 是访问所含值的一种直接方法，事实上它与 `boost::any` 的 `any_cast` 很相似。注意，类型必须完全符合，包括相同的 cv-限定符(`const` 和 `volatile`)。但是，可以使用限制更多的 cv-限定符。如果类型不匹配且传给 `get` 的是一个 `variant` 指针，将返回空指针。否则，抛出一个类型为 `bad_get` 的异常。

```cpp
const int& i=boost::get<const int>(my_first_variant); 
```

过分依赖于 `get` 的代码很容易变得脆弱；如果我们不知道所含值的类型，我们可能会想测试所有可能的组合，就如下面这个例子的做法。

```cpp
#include <iostream>
#include <string>
#include "boost/variant.hpp"

template <typename V> void print(V& v) {
  if (int* pi=boost::get<int>(&v))
    std::cout << "It's an int: " << *pi << '\n';
  else if (std::string* ps=boost::get<std::string>(&v))
    std::cout << "It's a std::string: " << *ps << '\n';
  else if (double* pd=boost::get<double>(&v))
    std::cout << "It's a double: " << *pd << '\n';

  std::cout << "My work here is done!\n";
}

int main() {
  boost::variant<int,std::string,double>
    my_first_variant("Hello there!");
  print(my_first_variant);
  my_first_variant=12;
  print(my_first_variant);
  my_first_variant=1.1;
  print(my_first_variant);
} 
```

函数 `print` 现在可以正确工作，但如果我们决定改变 `variant` 的类型组的话会怎样？我们将引入一个微妙的 bug，而不能在编译期捉住它；函数 `print` 不能打印任何其它我们没有预先想到的类型的值。如果我们没有使用模板函数，而是要求一个明确的 `variant` 类型，我们就要为不同类型的 `variant` 重载多个相同功能的函数。下一节将讨论访问 `variant` 的概念，以及这种(类型安全的)访问机制解决的问题。

### 访问 Variants

让我们从一个例子开始，它解释了为什么使用 `get` 并没有你想要的那么可靠。从前面父子的代码开始，我们来修改一下 `variant` 可以包含的类型，并对 `variant` 的一个 `char` 值来调用 `print` 。

```cpp
int main() {
  boost::variant<int,std::string,double,char>
    my_first_variant("Hello there!");

  print(my_first_variant);
  my_first_variant=12;
  print(my_first_variant);
  my_first_variant=1.1;
  print(my_first_variant);
  my_first_variant='a';
  print(my_first_variant);
} 
```

虽然我们给 `variant` 的类型组增加了 `char` ，并且程序的最后两行设置了一个 `char` 值并调用 `print` ，编译器也不会有意见 (注意，`print` 是以 `variant` 的类型来特化的，所以它可以很容易适应新的 `variant` 定义)。以下是这个程序的运行结果：

```cpp
It's a std::string: Hello there!
My work here is done!
It's an int: 12
My work here is done!
It's a double: 1.1
My work here is done!
My work here is done! 
```

这个输出显示了一个问题。最后一个"My work here is done!"之前没有值的报告。原因是很简单，`print` 不能输出除了它原来设计好的那些类型(`std::string`, `int`, 和 `double`)以外的任何值，但它可以干净地编译和运行。如果 `variant` 的当前类型不被 `print` 支持，它的值就会被简单地忽略掉。使用 `get` 还有更多潜在的问题，例如 if 语句的顺序要与类的层次相一致。注意，这并不是说你应该完全避免使用 `get`；它只是说有些时候它不是最好的方法。有一种更好的机制，可以允许我们规定哪些类型的值可以接受，并且这些规定是在编译期生效的。这是就 `variant` 访问机制的作用。通过把一个访问器应用到 `variant`，编译器可以保证它们完全兼容。Boost.Variant 中这些访问器是带有一些函数调用操作符的函数对象，这些函数调用操作符接受与它们所访问的 `variant` 可以包含的类型组相对应的参数。

现在我们用访问器来重写那个声名狼籍的函数 `print` ，如下：

```cpp
class print_visitor : public boost::static_visitor<void> {
public:
  void operator()(int i) const {
    std::cout << "It's an int: " << i << '\n';
  }

  void operator()(std::string s) const {
    std::cout << "It's a std::string: " << s << '\n';
  }

  void operator()(double d) const {
    std::cout << "It's a double: " << d << '\n';
  }

}; 
```

要让 `print_visitor` 成为 `variant` 的一个访问器，我们要让它派生自 `boost::static_visitor` 以获得正确的 `typedef` (`result_type`), 并明确地声明这个类是一个访问器类型。这个类实现了三个重载版本的函数调用操作符，分别接受一个 `int`, 一个 `std::string`, 和一个 `double` 。为了访问 `variant`, 你要用函数 `boost::apply_visitor`(visitor, variant). 如果我们用对 `apply_visitor`的调用来替换前面的 `print` 调用，我们可以得到如下代码：

```cpp
int main() {
  boost::variant<int,std::string,double,char>
    my_first_variant("Hello there!");

  print_visitor v;

  boost::apply_visitor(v,my_first_variant);
  my_first_variant=12;
  boost::apply_visitor(v,my_first_variant);
  my_first_variant=1.1;
  boost::apply_visitor(v,my_first_variant);
  my_first_variant='a';
  boost::apply_visitor(v,my_first_variant);
} 
```

这里，我们创建了一个 `print_visitor`, 名为 `v`, 并把它应用于赋值后的 `my_first_ variant` 。因为我们没有一个函数调用操作符接受 `char`, 这段代码会编译失败，是吗？错！一个 `char` 可以转换为一个 `int`, 所以这个访问器可以兼容我们的 `variant` 类型。以下是程序运行的结果。

```cpp
It's a std::string: Hello there!
It's an int: 12
It's a double: 1.1
It's an int: 97 
```

这里我们可以学到两件事情：第一个是字母 `a` 的 ASCII 码值为 97, 更重要的是第二个，如果一个访问器以传值的方式传递参数，则传送的值可以应用隐式转换。如果我们想访问器只能使用精确的类型(同时也避免拷贝从 `variant` 得到的值)，我们必须修改访问器的调用操作符传递参数的方式。以下这个版本的 `print_visitor` 只能使用类型 `int`, `std::string`, 和 `double`; 以及可以隐式转换到这些类型的引用的其它类型。

```cpp
class print_visitor : public boost::static_visitor<void> {
public:
  void operator()(int& i) const {
    std::cout << "It's an int: " << i << '\n';
  }

  void operator()(std::string& s) const {
    std::cout << "It's a std::string: " << s << '\n';
  }

  void operator()(double& d) const {
    std::cout << "It's a double: " << d << '\n';
  }
}; 
```

如果再编译一下这个程序，编译器就不高兴了，它会输出如下信息：

```cpp
c:/boost_cvs/boost/boost/variant/variant.hpp:
In member function `typename Visitor::result_type boost::detail:: variant::
invoke_visitor<Visitor>::internal_visit(T&, int)
[with T = char, Visitor = print_visitor]':

[Snipped lines of irrelevant information here]

c:/boost_cvs/boost/boost/variant/variant.hpp:807:
error: no match for call to `(print_visitor) (char&)'
variant_sample1.cpp:40: error: candidates are:
 void print_visitor::operator()(int&) const
variant_sample1.cpp:44: error:
 void print_visitor::operator()(std::string&) const
variant_sample1.cpp:48: error:
 void print_visitor::operator()(double&) const 
```

这个错误指出了问题：没有一个候选的函数接受 `char` 参数！为什么说类型安全的编译期访问机制是一个强大的机制，这正是一个重要的原因。它使得访问机制强烈依赖于类型，避免了讨厌的类型变换。创建访问器与创建其它函数对象一样容易，因此学习曲线并不陡峭。当 `variant` 中的类型组可能会改变时(它们总是倾向于变化！)，创建访问器要比单单依赖 `get` 更可靠。虽然开始需要更高的代价，但绝对是值得的。

### 泛型访问器

通过使用访问器机制和泛型的调用操作符，可以创建能够接受任意类型的泛型访问器(无论是在语法上还是语义上，都可以实现泛型调用操作符)。这对于统一地处理不同的类型非常有用。C++的操作符就是"通用"性的一个典型例子，如算术和 IO 流的位移操作符。以下例子使用 `operator&lt;&lt;` 来输出 `variant` 的值到一个流。

```cpp
#include <iostream>
#include <sstream>
#include <string>
#include <sstream>
#include "boost/variant.hpp"

class stream_output_visitor :
  public boost::static_visitor<void> {
  std::ostream& os_;
public:
  stream_output_visitor(std::ostream& os) : os_(os) {}

  template <typename T> void operator()(T& t) const {
    os_ << t << '\n';
  }
};

int main() {
  boost::variant<int,std::string> var;
  var=100;
  boost::apply_visitor(stream_output_visitor(std::cout),var);
  var="One hundred";
  boost::apply_visitor(stream_output_visitor(std::cout),var);
} 
```

主要思想是 `stream_output_visitor` 中的调用操作符是一个成员函数模板，它在访问每一种类型(本例中是 `int` 和 `std::string`)时分别实例化。因为 `std::cout &lt;&lt; 100` 和 `std::cout &lt;&lt; std::string("One hundred")` 都已经有定义了，所以这段代码可以编译并工作良好。

当然，操作符仅是可以使用泛型访问器的一个例子；它们常常应用于更多的类型。在某些值上调用函数，或 者将它们作为参数传给其它的函数时，要求就是对于所有传给操作符的类型都要有相应的成员函数存在，并且对于被调用的函数要有合适的重载。这种泛型调用操作 符的另一个有趣的方面是，可以对某些类型特化其行为，但对于其余类型则仍允许泛型的实现。在某种意义上，这涉及到模板特化，即基于类型信息的行为特殊化。

### 二元访问器

我们前面看到的访问器都是一元的，即它们只接受一个 `variant` 作为唯一的参数。二元访问器接受两个(可能是不同的) `variant`. 这种概念对于实现两个 `variant` 间的关系很有用。作为例子，我们为 `variant` 类型将创建一个按字典顺序的排序。为此，我们使用一个来自于标准库的非常有用的组件：`std::ostringstream`. 它接受任意可流输出的东西，并且在需要时产生一个独立的 `std::string` 。我们从而可以按字典序比较完全不同的 `variant` 类型，只要假设所有限定的类型都支持流输出。和普通的访问器一样，二元访问器也派生自 `boost::static_visitor`, 并且用模板参数表示调用操作符的返回类型。因为我们是创建一个谓词，因此返回类型为 `bool`. 以下是一个我们即将用到的二元谓词。

```cpp
class lexicographical_visitor :
  public boost::static_visitor<bool> {
public:
  template <typename LHS,typename RHS>
  bool operator()(const LHS& lhs,const RHS& rhs) const {
    return get_string(lhs)<get_string(rhs);
  }
private:
  template <typename T> static std::string
  get_string(const T& t) {
    std::ostringstream s;
    s << t;
    return s.str();
  }

  static const std::string& get_string(const std::string& s) {
    return s;
  }
}; 
```

这里的调用操作符泛化了它的两个参数，这意味着它接受任意两种类型的组合。对于 `variant` 的可用类型组的要求就是它们必须是可流输出(OutputStreamable)的。成员函数模板 `get_string` 使用一个 `std::ostringstream` 来把它的参数转换为字符串表示，所以要求参数必须是可流输出的(为了使用 `std::ostringstream`, 记得要包含头文件 `&lt;sstream&gt;`)。成员函数 `get_string` 针对类型为 `std::string` 的参数进行特化，由于类型已经符合要求，所以它跳过了 `std::ostringstream` 而直接返回它的参数。在两个参数都转为 `std::string` 以后，剩下的就是使用 `operator&lt;` 来比较它们了。现在我们把这个访问器放入测试代码，来对一个容器中的元素进行排序(我们还将重用我们在本章前面创建的 `stream_output_visitor` )。

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
#include "boost/variant.hpp"

int main() {
  boost::variant<int,std::string> var1="100";
  boost::variant<double> var2=99.99;

  std::cout << "var1<var2: " <<
    boost::apply_visitor(
      lexicographical_visitor(),var1,var2) << '\n';

  typedef std::vector<
    boost::variant<int,std::string,double> > vec_type;

  vec_type vec;
  vec.push_back("Hello");
  vec.push_back(12);
  vec.push_back(1.12);
  vec.push_back("0");

  stream_output_visitor sv(std::cout);
  std::for_each(vec.begin(),vec.end(),sv);

  lexicographical_visitor lv;
  std::sort(vec.begin(),vec.end(),boost::apply_visitor(lv));

  std::cout << '\n';
  std::for_each(vec.begin(),vec.end(),sv);
}; 
```

首先，我们将访问应用于两个 `variants`, `var1` 和 `var2`, 如下：

```cpp
boost::apply_visitor(lexicographical_visitor(),var1,var2) 
```

如你所见，与一元访问器不同的是，有两个 `variant` 被传递给函数 `apply_visitor`. 一个更为常见的用例是使用这个谓词来对元素进行排序，我们这样来做：

```cpp
lexicographical_visitor lv;
std::sort(vec.begin(),vec.end(),boost::apply_visitor(lv)); 
```

当 `sort` 算法被执行时，它使用我们传入的谓词来比较它的元素，它是一个 `lexicographical_visitor` 实例。注意，`boost::variant` 已经定义了 `operator&lt;`, 所以不使用谓词也可以对容器进行排序。

```cpp
std::sort(vec.begin(),vec.end()); 
```

但是这种缺省的排序是首先使用 `which` 来检查当前值的索引，所以元素的排列顺序将是 12, 0, Hello, 1.12, 而我们想要的是按字典序来排序。因为 `variant` 类已经提供了 `operator&lt;` 和 `operator==` ，所以 `variant` 可以用作所有标准库容器的元素类型。当缺省的关系比较不够用时，你需要用二元访问器来实现一个。

### 更多应该知道的事情

我们并没有涉及到 Boost.Variant 库的所有功能。其它更为先进的特性不如我们已经提到的那么常用。但是，我会简要地说一下，因此你将至少知道在需要时可以找到哪些可用的东西。宏 `BOOST_VARIANT_ENUM_PARAMS`, 可用于为 `variant` 类型重载/特化函数和类模板。这个宏用于列举 `variant` 可以包含的类型组。还有支持使用类型序列来创建 `variant` 类型，即通过 `make_variant_over`编译期列表来表示 `variant` 的类型组。还有递归的 `variant` 类型，可用于创建它们自己类型的表达式，递归 `variant` 类型使用 `recursive_wrapper`, `make_recursive_variant`, 和 `make_recursive_variant_over`. 如果你需要这些额外的特性，在线文档可以很好地解释它们。

# Variant 总结

## Variant 总结

类别联合(discriminated unions)在日常编程中非常有用，这个事实无须惊讶，Boost.Variant 库提供了高效且易用的 `variant` 类型，它正是基于类别联合的。因为 C++的联合对于很多类型很难使用(它只支持内建类型和 POD 类型)，长期以来一直需要别的东西来取代它。许多创建类别联合的尝试都存在某些重要的缺点。例如，早期的尝试通常仅支持固定的一组类型，的确妨碍了维护性 和灵活性。Boost.Variant 通过模板避免了这些限制，理论上允许创建任意的 `variant` 类型。在处理类别联合时类型转换代码总会成为问题所在；在处理前需要测试当前值的类型，这导致了维护的麻烦。Boost.Variant 提供了简单的值取回操作以及类型安全的访问机制，这是解决问题的新颖方法。最后，效率也是早期的尝试所关心的，这个库也很好地照顾到了效率，它使用基于栈 的存储，而不是基于堆的。

Boost.Variant 是一个成熟的库，有非常多的特性，使用 `variant` 类型容易且高效。这是 Boost.Any 库的补充，同样应该成为你的专业 C++工具箱中的一员。

Boost.Variant 的作者是 Eric Friedman 和 Itay Maman.