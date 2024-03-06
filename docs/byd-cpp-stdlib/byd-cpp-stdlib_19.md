# Library 10\. Lambda

# Library 10\. Lambda

*   Lambda 库如何改进你的程序？
*   Lambda 如何适用于标准库？
*   Lambda
*   用法
*   Lambda 总结

# Lambda 库如何改进你的程序？

## Lambda 库如何改进你的程序？

*   对函数和函数对象进行适配，使之可用于标准库算法

*   绑定参数到函数调用

*   将任意的表达式转换为可以兼容标准库算法的函数对象

*   就地定义匿名函数，提高代码的可读性和可维护性

*   在需要的时间和地点实现谓词

在使用标准库或其它采用相似设计的库时，需要依靠函数或函数对象来对算法 进行配置，你通常要编写很多小的函数对象来执行一些非常简单的操作。就象我们在 "Library 9: Bind 9" 看到的那样，这很容易成为一个问题，因为有大量的小类分散在代码中，这样很难进行维护。另外，理解函数对象被调用处的代码会很难，因为有一部分的功能被定 义在别的地方。一个好的解决办法是，想办法就在调用的地方定义这些函数或函数对象。这通常可以使代码写得更快，也更容易维护，因为函数的定义就在它被使用 的地方。这正是 Boost.Lambda 库所要提供的，就地定义匿名函数。Boost.Lambda 可以创建直接定义和调用的函数对象，或者把它保存起来晚一些再调用。这与 Boost.Bind 库所提供的很相似，但 Boost.Lambda 除了可以进行参数绑定，还有其它功能，增加了控制结构、表达式到函数对象的自动转换，还支持在 lambda 表达式中的异常处理。

术语 lambda 表达式或 lambda 函数，来源于函数式编程与 lambda 演算。一个 lambda 抽象概念定义了一个匿名函数。虽然 lambda 抽象概念在函数式编程语言(functional programming language)中非常普遍，但是在象 C++这样的命令式编程语言(imperative programming language)中则不是。但是，通过使用象表达式模板这样的先进技术，C++ 可以在语言中增加某种形式的 lambda 表达式。

创建 Lambda 库最早的动机是，可以在标准库算法中使用匿名函数。因为从 1998 年第一个 C++标准发布后，标准库的使用非常广泛，我们对于什么好什么不好的认识快速增 长，而其中一个存在疑问的就是，对于众多小函数对象的定义，好象只需要一个简单的表达式就可以满足了。显然这个库就是定位于解决这些函数对象的问题，但是 对于 lambda 函数的使用还有很大的探索空间。现在，lambda 函数已经可以使用了，我们可以把它应用于以前需要用完全不同的方法来解决的问题。令人着迷和兴奋的是，象 C++这样一种成熟的语言还可以探索出新的编程技 术。匿名函数和表达式模板的出现会带来怎样的新用法和新方法呢？事实是，我们不知道，因为我们还没有全力去试验它！尽管如此，这里所关注的是这个库明确要 解决的实际问题，即通过就地定义 lambda 表达式和函数来避免代码膨胀和功能的分散。我们可以用它做出更多惊人的事情，有了它我们可以更为简洁，这可以同时满足程序员和他们的经理，前者可以更加集 中精力在手边的问题，后者可以获得更高生产效率的好处(希望也是更容易维护的代码！)。

# Lambda 如何适用于标准库？

## Lambda 如何适用于标准库？

这个库用于解决一个使用标准库算法时常会遇见的问题，即需要为了满足算法的要求而定义很多简单的函数对象。几乎所有的标准库算法都有一个接受函数对象的版本，这个函数对象用于执行如排序、等同性检验、转换等操作。标准库通过绑定器 `bind1st` 和 `bind2nd` 支持有限的函数组合。但是，它们能做的事情非常有限，它们只能提供参数绑定，而不能绑定表达式。在 Boost.Lambda 库中，既有对绑定参数的灵活支持，也可以直接从表达式创建函数对象，对于 C++标准库来说，这是一个杰出的合作伙伴。

# Lambda

## Lambda

### 头文件: `"boost/lambda/lambda.hpp"`

它包括了本库的核心部分。

```cpp
"boost/lambda/bind.hpp" 
```

它定义了 `bind` 函数。

```cpp
"boost/lambda/if.hpp" 
```

它定义了相当于 `if` 的 lambda ，以及条件操作符。

```cpp
"boost/lambda/loops.hpp" 
```

它定义了循环结构(例如，`while_loop` 和 `for_loop`)。

```cpp
"boost/lambda/switch.hpp" 
```

它定义了相当于 switch 语句的 lambda 。

```cpp
"boost/lambda/construct.hpp" 
```

它定义了一些工具，为增加构造函数/析构函数以及 `new`/`delete` 。

```cpp
"boost/lambda/casts.hpp" 
```

它为提供了转型操作符。

```cpp
"boost/lambda/exceptions.hpp" 
```

它定义了在 lambda 表达式中进行异常处理的工具。

```cpp
"boost/lambda/algorithm.hpp" and "boost/lambda/numeric.hpp" 
```

它定义了用于嵌套函数调用的 C++标准库算法的 lambda 版本(实际上就是函数对象)。

# 用法

## 用法

与其它许多 Boost 库一样，这个库完全定义在头文件中，这意味着你不必构建任何东西就可以开始使用。但是，知道一点关于 lambda 表达式的东西肯定是有帮助的。接下来的章节会带你浏览一下这个库，还包括如何在 lambda 表达式中进行异常处理！这个库非常广泛，前面还有很多强大的东西。一个 lambda 表达式通常也称为匿名函数(*unnamed function*)。它在需要的时 候进行声明和定义，即就地进行。这非常有用，因为我们常常需要在一个算法中定义另一个算法，这是语言本身所不能支持的。作为替代，我们通过从更大的范围引 进函数和函数对象来具体定义行为，或者使用嵌套的循环结构，把算法表达式写入循环中。我们将看到，这正是 lambda 表达式可以发挥的地方。本节内有许多例子，通常例子的一部分是示范如何用"传统"的编码方法来解决问题。这样做的目的是，看看 lambda 表达式在何时以及如何帮助程序写出更具逻辑且更少的代码。使用 lambda 表达式的确存在一定的学习曲线，而且它的语法初看起来有点可怕。就象每种新的范式或工具，它们都需要去学习，但是请相信我，得到的好处肯定超过付出的代 价。

### 一个简单的开始

第一个使用 Boost.Lambda 的程序将会提升你对 lambda 表达式的喜爱。首先，请注意 lambda 类型是声明在 `boost::lambda` 名字空间中，你需要用一个 using 指示符或 using 声明来把这些 lambda 声明带入你的作用域。包含头文件 `"boost/lambda/lambda.hpp"` 就可以使用这个库的主要功能了，对于我们第一个程序这样已经足够了。

```cpp
#include <iostream>
#include "boost/lambda/lambda.hpp"
#include "boost/function.hpp"
int main() {
  using namespace boost::lambda;
  (std::cout << _1 << " " << _3 << " " << _2 << "!\n")
    ("Hello","friend","my");
  boost::function<void(int,int,int)> f=
    std::cout << _1 << "*" << _2 << "+" << _3
      << "=" <<_1*_2+_3 << "\n";
  f(1,2,3);
  f(3,2,1);
} 
```

第一个表达式看起来很奇特，你可以在脑子里按着括号来划分这个表达式；第一部分就是一个 lambda 表达式，它的意思基本上是说，"打印这些参数到 `std::cout`, 但不是立即就做，因为我还不知道这三个参数"。表达式的第二部分才是真正调用这个函数，它说，"嘿！这里有你要的三个参数"。我们再来看看这个表达式的第一部分。

```cpp
std::cout << _1 << " " << _3 << " " << _2 << "!\n" 
```

你会注意到表达式中有三个占位符，命名为 `_1`, `_2`, 和 `_3` [1]。 这些占位符为 lambda 表达式指出了延后的参数。注意，跟许多函数式编程语言的语法不一样，创建 lambda 表达式时没有关键字或名字；占位符的出现表明了这是一个 lambda 表达式。所以，这是一个接受三个参数的 lambda 表达式，参数的类型可以是任何支持 `operator&lt;&lt;` 流操作的类型。参数按 1-3-2 的顺序打印到 `cout` 。在这个例子中，我们把这个表达式用括号括起来，然后调用得到的这个函数对象，传递三个参数给它：`"Hello"`, `"friend"`, 和 `"my"`. 输出的结果如下：

[1]你可能没想到象 `_1` 这样的标识符是合法的，但它们的确是。标识符不能由数字打头，但可以由下划线打头，而数字可以出现在标识符的其它任何地方。

```cpp
Hello my friend! 
```

通常，我们要把函数对象传入算法，这是我们要进一步研究的，但是我们来试验一些更有用的东西，把 lambda 表达式存入另一个延后调用的函数，名为 `boost::function`. 这个有用的发明将在下一章 "Library 11: Function 11" 中讨论，现在你只有知道可以传递一个函数或函数对象给 `boost::function` 的实例并保存它以备后用就可以了。在本例中，我们定义了这样的一个函数 `f`，象这样：

```cpp
boost::function<void(int,int,int)> f; 
```

这个声明表示 `f` 可以存放用三个参数调用的函数和函数对象，参数的类型全部为 `int`. 然后，我们用一个 lambda 表达式把一个函数对象赋给它，这个表达式表示了算法 *X=S*T+U*, 并且把这个算式及其结果打印到 `cout`.

```cpp
boost::function<void(int,int,int)> f=
  std::cout <<
    _1 << "*" << _2 << "+" << _3 << "=" <<_1*_2+_3 << "\n"; 
```

如你所见，在一个表达式中，占位符可以多次使用。我们的函数 `f` 现在可以象一个普通函数那样调用了，如下：

```cpp
f(1,2,3);
f(3,2,1); 
```

运行这段代码的输出如下。

```cpp
1*2+3=5
3*2+1=7 
```

任意使用标准操作符(操作符还可以被重载！)的表达式都可以用在 lambda 表达式中，并可以保存下来以后调用，或者直接传递给某个算法。你要留意，当一个 lambda 表达式没有使用占位符时(我们还没有看到如何实现，但的确可以这样用)，那么结果将是一个无参函数(对象)。作为对比，只使用 `_1` 时，结果是一个单参数函数对象；只使用 `_1` 和 `_2` 时，结果则是一个二元函数对象；当只使用 `_1`, `_2`, 和 `_3` 时，结果就是一个三元函数对象。这第一个 lambda 表达式受益于这样一个事实，即该表达式只使用了内建或常用的 C++操作符，这样就可以直接编写算法。接下来，我们看看如何绑定表达式到其它函数、类成员函数，甚至是数据成员！

### <a class="calibre36" id="ch10lev2sec3">在操作符不够用时就用绑定

到目前为止，我们已经看到如果有操作符可以支持我们的表达式，一切顺利，但并不总是如此的。有时我们需要把调用另一个函数作为表达式的一部分，这通常要借助于绑定；这种绑定与我们前面在创建 lambda 表达式时见过的绑定有所不同，它需要一个单独的关键字，`bind` (嘿，这真是个聪明的名字！)。一个绑定表达式就是一个被延迟的函数调用，可以是普通函数或成员函数。该函数可以有零个或多个参数，某些参数可以直接设定，另一些则可以在函数调用时给出。对于当前版本的 Boost.Lambda, 最多可支持九个参数(其中三个可以通过使用占位符在稍后给出)。要使用绑定器，你需要包含头文件`"boost/lambda/bind.hpp"`。

在绑定到一个函数时，第一个参数就是该函数的地址，后面的参数则是函数的参数。对于一个非静态成员函数，总是有一个隐式的 `this` 参数；在一个 `bind` 表达式中，必须显式地加上 `this` 参数。为方便起见，不论对象是通过引用传递或是通过指针传递，语法都是一样的。因此，在绑定到一个成员函数时，第二个参数(即函数指针后的第一个)就是将要调用该函数的真实对象。绑定到数据成员也是可以的，下面的例子也将有所示范：

```cpp
#include <iostream>
#include <string>
#include <map>
#include <algorithm>
#include "boost/lambda/lambda.hpp"
#include "boost/lambda/bind.hpp"
int main() {
  using namespace boost::lambda;
  typedef std::map<int,std::string> type;
  type keys_and_values;
  keys_and_values[3]="Less than pi";
  keys_and_values[42]="You tell me";
  keys_and_values[0]="Nothing, if you ask me";
  std::cout << "What's wrong with the following expression?\n";
  std::for_each(
    keys_and_values.begin(),
    keys_and_values.end(),
    std::cout << "key=" <<
      bind(&type::value_type::first,_1) << ", value="
      << bind(&type::value_type::second,_1) << '\n');
  std::cout << "\n...and why does this work as expected?\n";
  std::for_each(
    keys_and_values.begin(),
    keys_and_values.end(),
    std::cout << constant("key=") <<
      bind(&type::value_type::first,_1) << ", value="
      << bind(&type::value_type::second,_1) << '\n');
  std::cout << '\n';
  // Print the size and max_size of the container
  (std::cout << "keys_and_values.size()=" <<
    bind(&type::size,_1) << "\nkeys_and_values.max_size()="
    << bind(&type::max_size,_1))(keys_and_values);
} 
```

这个例子开始时先创建一个 `std::map` ，键类型为 `int` 且值类型为 `std::string` 。记住，`std::map` 的 `value_type` 是一个由键类型和值类型组成的 `std::pair` 。因此，对于我们的 `map`, `value_type` 就是 `std::pair&lt;int,std::string&gt;`, 所以在 `for_each` 算法中，我们传入的函数对象要接受一个这样的类型。给出这个 `pair`, 就可以取出其中的两个成员(键和值)，这正是我们第一个 `bind` 表达式所做的。

```cpp
bind(&type::value_type::first,_1) 
```

这个表达式生成一个函数对象，它被调用时将取出它的参数，即我们前面讨论的 `pair` 中的嵌套类型 `value_type` 的数据成员 `first`。在我们的例子中，`first` 是 `map` 的键类型，它是一个 `const int`. 对于成员函数，语法是完全相同的。但你要留意，我们的 lambda 表达式多做了一点；表达式的第一部分是

```cpp
std::cout << "key=" << ... 
```

它可以编译，也可以工作，但它可能不能达到目的。这个表达式不是一个 lambda 表达式；它只是一个表达式而已，再没有别的了。执行时，它打印 `key=`, 当这个表达式被求值时它仅执行一次，而不是对于每个被 `std::for_each` 所访问的元素执行一次。在这个例子中，原意是把 `key=` 作为我们的每一个 `keys_and_values` 键/值对的前缀。在早一点的那些例子中，我们也是这样写的，但那里没有出现这些问题。原因在于，那里我们用了一个占位符来作为 `operator&lt;&lt;` 的第一个参数，这样就使得它成为一个有效的 lambda 表达式。而这里，我们必须告诉 Boost.Lambda 要创建一个包含 `"key="` 的函数对象。这就要使用函数 `constant`, 它创建一个无参函数对象，即不带参数的函数对象；它仅仅保存其参数，然后在被调用时返回它。

```cpp
std::cout << constant("key=") << ... 
```

这个小小的修改使得所有输出都不一样了，以下是该程序的运行输出结果。

```cpp
What's wrong with the following expression?
key=0, value=Nothing, if you ask me
3, value=Less than pi
42, value=You tell me
...and why does this work as expected?
key=0, value=Nothing, if you ask me
key=3, value=Less than pi
key=42, value=You tell me
keys_and_values.size()=3
keys_and_values.max_size()=4294967295 
```

例子的最后一部分是一个绑定到成员函数的绑定器，而不是绑定到数据成员；语法是一样的，而且你可以看 到在这两种情形下，都不需要显式地表明函数的返回类型。这种奇妙的事情是由于函数或成员函数的返回类型可以被自动推断，如果是绑定到数据成员，其类型同样 可以自动得到。但是，有一种情形不能得到返回类型，即当被绑定的是函数对象时；对于普通函数和成员函数，推断其返回类型是一件简单的事情[2]，但对于函数对象则不可能。有两种方法绕过这个语言的限制，第一种是由 Lambda 库自己来解决：通过显式地给出 `bind` 的模板参数来替代返回类型推断，如下所示。

[2] 你也得小心行事。我们只是说它在技术上可行。

```cpp
class double_it {
public:
  int operator()(int i) const {
    return i*2;
  }
};
int main() {
  using namespace boost::lambda;
  double_it d;
  int i=12;
  // If you uncomment the following expression,
  // the compiler will complain;
  // it's just not possible to deduce the return type
  // of the function call operator of double_it.
  // (std::cout << _1 << "*2=" << (bind(d,_1)))(i);
  (std::cout << _1 << "*2=" << (bind<int>(d,_1)))(i);
  (std::cout << _1 << "*2=" << (ret<int>(bind(d,_1))))(i);
} 
```

有两种版本的方法来关闭返回类型推断系统，短格式的版本只需把返回类型作为模板参数传给 `bind`, 另一个版本则使用 `ret`, 它要括住不能进行自动推断的 lambda/bind 表达式。在嵌套的 lambda 表达式中，这很容易会就得乏味，不过还有一种更好的方法可以让推断成功。我们将在本章稍后进行介绍。

请注意，一个绑定表达式可以由另一个绑定表达式组成，这使得绑定器成为了进行函数组合的强大工具。嵌套的绑定有许多强大的功能，但是要小心使用，因为这些强大的功能同时也带来了读写以及理解代码上的额外的复杂性。

### <a class="calibre36" id="ch10lev2sec4">我不喜欢 _1, _2, and _3，我可以用别的名字吗？

有的人对预定义的占位符名称不满意，因此本库提供了简便的方法来把它们[3]改为任意用户想用的名字。这是通过声明一些类型为 `boost::lambda::placeholderX_type` 的变量来实现的，其中 `X` 为 1, 2, 或 3\. 例如，假设某人喜欢用 `Arg1`, `Arg2`, 和 `Arg3` 来作占位符的名字：

[3] 技术上，是增加新的名字。

```cpp
#include <iostream>
#include <vector>
#include <string>
#include "boost/lambda/lambda.hpp"
boost::lambda::placeholder1_type Arg1;
boost::lambda::placeholder2_type Arg2;
boost::lambda::placeholder3_type Arg3;
template <typename T,typename Operation>
void for_all(T& t,Operation Op) {
  std::for_each(t.begin(),t.end(),Op);
}
int main() {
  std::vector<std::string> vec;
  vec.push_back("What are");
  vec.push_back("the names");
  vec.push_back("of the");
  vec.push_back("placeholders?");
  for_all(vec,std::cout << Arg1 << " ");
  std::cout << "\nArg1, Arg2, and Arg3!";
} 
```

你定义的占位符变量可以象 `_1`, `_2`, 和 `_3` 一样使用。另外请注意这里的函数 `for_all` ，它提供了一个简便的方法，当你经常要对一个容器中的所有元素进行操作时，可以比用 `for_each` 减少一些键击次数。这个函数接受两个参数：一个容器的引用，以及一个函数或函数对象。该容器中的每个元素将被提供给这个函数或函数对象。我认为它有时会非常有用，也许你也这样认为。运行这个程序将产生以下输出：

```cpp
What are the names of the placeholders?
Arg1, Arg2, and Arg3! 
```

创建你自己的占位符可能会影响其它阅读你的代码的人；多数知道 Boost.Lambda (或 Boost.Bind) 的程序员都熟悉占位符名称 `_1`, `_2`, 和 `_3`. 如果你决定把它们称为 `q`, `w`, 和 `e`, 你就需要解释给你的同事听它们有什么意思。(而且你可能要经常重复地进行解释！)

### <a class="calibre36" id="ch10lev2sec5">我想给我的常量和变量命名！

有时，给常量和变量命名可以提高代码的可读性。你也记得，我们有时需要创建一个不是立即求值的 lambda 表达式。这时可以使用 `constant` 或 `var`; 它们分别对应于常量或变量。我们已经用过 `constant` 了，基本上 `var` 也是相同的用法。对于复杂或长一些的 lambda 表达式，对一个或多个常量给出名字可以使得表达式更易于理解；对于变量也是如此。要创建命名的常量和变量，你只需要定义一个类型为 `boost::lambda::constant_type&lt;T&gt;::type` 和 `boost::lambda::var_type&lt;T&gt;::type` 的变量，其中的 `T` 为被包装的常量或变量的类型。看一下以下这个 lambda 表达式的用法：

```cpp
for_all(vec,
  std::cout << constant(' ') << _ << constant('\n')); 
```

总是使用 `constant` 会很让人讨厌。下面是一个例子，它命名了两个常量，`newline` 和 `space`，并把它们用于 lambda 表达式。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include "boost/lambda/lambda.hpp"
int main() {
  using boost::lambda::constant;
  using boost::lambda::constant_type;
  constant_type<char>::type newline(constant('\n'));
  constant_type<char>::type space(constant(' '));
  boost::lambda::placeholder1_type _;
  std::vector<int> vec;
  vec.push_back(0);
  vec.push_back(1);
  vec.push_back(2);
  vec.push_back(3);
  vec.push_back(4);
  for_all(vec,std::cout << space << _ << newline);
  for_all(vec,
    std::cout << constant(' ') << _ << constant('\n'));
} 
```

这是一个避免重复键入的好方法，也可以使 lambda 表达式更清楚些。下面是一个类似的例子，首先定义一个类型 `memorizer`, 用于跟踪曾经赋给它的所有值。然后，用 `var_type` 创建一个命名变量，用于后面的 lambda 表达式。你将会看到命名常量要比命名变量更常用到，但也有些情形会需要使用命名变量[4]。

[4] 特别是使用 lambda 循环结构时。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include "boost/lambda/lambda.hpp"
template <typename T> class memorizer {
  std::vector<T> vec_;
public:
  memorizer& operator=(const T& t) {
    vec_.push_back(t);
    return *this;
  }
  void clear() {
    vec_.clear();
  }
  void report() const {
    using boost::lambda::_1;
    std::for_each(
      vec_.begin(),
      vec_.end(),
      std::cout << _1 << ",");
  }
};
int main() {
  using boost::lambda::var_type;
  using boost::lambda::var;
  using boost::lambda::_1;
  std::vector<int> vec;
  vec.push_back(0);
  vec.push_back(1);
  vec.push_back(2);
  vec.push_back(3);
  vec.push_back(4);
  memorizer<int> m;
  var_type<memorizer<int> >::type mem(var(m));
  std::for_each(vec.begin(),vec.end(),mem=_1);
  m.report();
  m.clear();
  std::for_each(vec.begin(),vec.end(),var(m)=_1);
  m.report();
} 
```

这就是它的全部了，但在你认为自己已经明白了所有东西之前，先回答这个问题：在以下声明下 `T` 应该是什么类型？

```cpp
constant_type<T>::type hello(constant("Hello")); 
```

它是一个 `char*`? 一个 `const char*`? 都不是，它的正确类型是一个含有六个字符(还有一个结束用的空字符)的数组的常量引用，所以我们应该这样写：

```cpp
constant_type<const char (&)[6]>::type
  hello(constant("Hello")); 
```

这很不好看，而且对于需要修改这个字符串的人来说也很痛苦，所以我更愿意使用 `std::string` 来写。

```cpp
constant_type<std::string>::type
  hello_string(constant(std::string("Hello"))); 
```

这次，你需要比上一次多敲几个字，但你不需要再计算字符的个数，如果你要改变这个字符串，也没有问题。

### <a class="calibre36" id="ch10lev2sec6">*ptr_fun* 和 *mem_fun* 到哪去了？

也许你还在怀念它们，由于 Boost.Lambda 创建了与标准一致的函数对象，所以没有必要再记住这些标准库中的适配器类型了。一个绑定了函数或成员函数的 lambda 表达式可以很好地工作，而且不论绑定的是什么类型，其语法都是一致的。这可以让代码更注重其任务而不是某些奇特的语法。以下例子说明了这些好处：

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>
#include "boost/lambda/lambda.hpp"
#include "boost/lambda/bind.hpp"
void plain_function(int i) {
  std::cout << "void plain_function(" << i << ")\n";
}
class some_class {
public:
  void member_function(int i) const {
    std::cout <<
      "void some_class::member_function(" << i << ") const\n";
  }
};
int main() {
  std::vector<int> vec(3);
  vec[0]=12;
  vec[1]=10;
  vec[2]=7;
  some_class sc;
  some_class* psc=&sc;
  // Bind to a free function using ptr_fun
  std::for_each(
    vec.begin(),
    vec.end(),
    std::ptr_fun(plain_function));
  // Bind to a member function using mem_fun_ref
  std::for_each(vec.begin(),vec.end(),
    std::bind1st(
      std::mem_fun_ref(&some_class::member_function),sc));
  // Bind to a member function using mem_fun
  std::for_each(vec.begin(),vec.end(),
    std::bind1st(
      std::mem_fun(&some_class::member_function),psc));
  using namespace boost::lambda;
  std::for_each(
    vec.begin(),
    vec.end(),
    bind(&plain_function,_1));
  std::for_each(vec.begin(),vec.end(),
    bind(&some_class::member_function,sc,_1));
  std::for_each(vec.begin(),vec.end(),
    bind(&some_class::member_function,psc,_1));
} 
```

这里真的不需要用 lambda 表达式吗？相对于使用三个不同的结构来完成同一件事情，我们可以只需向 `bind` 指出要干什么，然后它就会去做。在这个例子中，需要用 `std::bind1st` 来把 `some_class` 的实例绑定到调用中；而对于 Boost.Lambda，这是它工作的一部分。因此，下次你再想是否要用 `ptr_fun`, `mem_fun`, 或 `mem_fun_ref` 时，停下来，使用 Boost.Lambda 来代替它！

### <a class="calibre36" id="ch10lev2sec7">无须*<functional>*的算术操作

我们常常要按顺序对一些元素执行算术操作，而标准库提供了多个函数对象来执行算术操作，如 `plus`, `minus`, `divides`, `modulus`, 等等。但是，这些函数对象需要我们多打很多字，而且常常需要绑定一个参数，这时应该使用绑定器。如果要嵌套这些算术操作，表达式很快就会变得难以使用，而 这正是 lambda 表达式可以发挥巨大作用的地方。因为我们正在处理的是操作符，既是算术上的也是 C++术语上的，所以我们有能力使用 lambda 表达式直接编写我们的算法代码。作为一个小的动机，考虑一个简单的问题，对一个数值增加 4。然后再考虑另一个问题，完成与标准库算法(如 `transform`)同样的工作。虽然第一个问题非常自然，而第二个则完全不一样(它需要你手工写循环)。但使用 lambda 表达式，只需关注算法本身。在下例中，我们先使用 `std::bind1st` 和 `std::plus` 对容器中的每个元素加 4，然后我们使用 `lambda` 来减 4。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>
#include "boost/lambda/lambda.hpp"
#include "boost/lambda/bind.hpp"
int main() {
  using namespace boost::lambda;
  std::vector<int> vec(3);
  vec[0]=12;
  vec[1]=10;
  vec[2]=7;
  // Transform using std::bind1st and std::plus
  std::transform(vec.begin(),vec.end(),vec.begin(),
    std::bind1st(std::plus<int>(),4));
  // Transform using a lambda expression
  std::transform(vec.begin(),vec.end(),vec.begin(),_1-=4);
} 
```

差别是令人惊讶的！在使用"传统"方法进行加 4 时，对于未经训练的眼睛来说，很难看出究竟在干什么。从代码中我们看到，我们将一个缺省构造的 `std::plus` 实例的第一个参数绑定到 4。而 lambda 表达式则写成从元素减 4。如果你认为使用 `bind1st` 和 `plus` 的版本还不坏，你可以试试更长的表达式。

Boost.Lambda 支持 C++中的所有算术操作符，因此几乎不再需要仅为了算术函数对象而包含 `&lt;functional&gt;` 。以下例子示范了这些算术操作符中某些的用法。`vector vec` 中的每个元素被加法和乘法操作符修改。

```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include "boost/lambda/lambda.hpp"
int main() {
  using namespace boost::lambda;
  std::vector<int> vec(3);
  vec[0]=1;
  vec[1]=2;
  vec[2]=3;
  std::for_each(vec.begin(),vec.end(),_1+=10);
  std::for_each(vec.begin(),vec.end(),_1-=10);
  std::for_each(vec.begin(),vec.end(),_1*=3);
  std::for_each(vec.begin(),vec.end(),_1/=2);
  std::for_each(vec.begin(),vec.end(),_1%=3);
} 
```

简洁、可读、可维护，这就是使用 Boost.Lambda 所得到的代码的风格。跳过 `std::plus`, `std::minus`, `std::multiplies`, `std::divides`, 和 `std::modulus`; 使用 Boost.Lambda，你的代码总会更好。

### <a class="calibre36" id="ch10lev2sec8">编写可读的谓词

标准库中的许多算法都有一个版本是接受一个一元或二元的谓词的。这些谓词是普通函数或函数对象，当 然，lambda 表达式也可以。对于会经常用到的谓词，当然应该定义函数对象，但通常，它们只使用一两次并且再不会碰到。在这种情况下，lambda 表达式是更好的选择，这既是因为代码可以更容易理解(所有功能都在同一个地方)，也是因为代码不会被一些极少使用的函数对象搞混。作为一个具体的例子，我 们在容器中查找具有某个特定值的元素。如果该元素类型已经定义了 `operator==` ，则可以直接使用算法 `find` ，但如果要使用其它标准来查找元素呢？以下给出类型 `search_for_me` ，你如何使用 `find` 来查找第一个元素，其满足成员函数 `a` 返回 `"apple"`的条件？

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <string>
class search_for_me {
  std::string a_;
  std::string b_;
public:
  search_for_me() {}
  search_for_me(const std::string& a,const std::string& b)
    : a_(a),b_(b) {}
  std::string a() const {
    return a_;
  }
  std::string b() const {
    return b_;
  }
};
int main() {
  std::vector<search_for_me> vec;
  vec.push_back(search_for_me("apple","banana"));
  vec.push_back(search_for_me("orange","mango"));
  std::vector<search_for_me>::iterator it=
    std::find_if(vec.begin(),vec.end(),???);
  if (it!=vec.end())
    std::cout << it->a() << '\n';
} 
```

首先，我们需要用 `find_if`,[5] 但是标记了 `???` 的地方应该怎样写呢？一种办法是：用一个函数对象来实现该谓词的逻辑。

[5] `find` 使用 `operator==; find_if` 则要求一个谓词函数(或函数对象)。

```cpp
class a_finder {
  std::string val_;
public:
  a_finder() {}
  a_finder(const std::string& val) : val_(val) {}
  bool operator()(const search_for_me& s) const {
    return s.a()==val_;
  }
}; 
```

这个函数对象可以这样使用：

```cpp
std::vector<search_for_me>::iterator it=
  std::find_if(vec.begin(),vec.end(),a_finder("apple")); 
```

这可以，但两分钟(或几天)后，我们想要另一个函数对象，这次要测试成员函数 `b`. 等等…这类事情很快就会变得乏味。正如你确信的那样，这是 lambda 表达式的另一个极好的例子；我们需要某种灵活性，可以在需要的地方和需要的时间直接创建谓词。我们可以这样来写前述的 `find_if` 。

```cpp
std::vector<search_for_me>::iterator it=
  std::find_if(vec.begin(),vec.end(),
    bind(&search_for_me::a,_1)=="apple"); 
```

我们 `bind` 到成员函数 `a`, 并且测试它是否等于 `"apple"`，这就是我们的一元谓词，它就定义在使用的地方。但是等一下，正如它们说的，还有更多的东西。在处理数值类型时，我们可以在所有算术操作符、比较和逻辑操作符中选择。这意味着哪怕是复杂的谓词也可以直接了当地定义。仔细阅读以下代码，看看谓词是如何表示的。

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <string>
#include "boost/lambda/lambda.hpp"
int main() {
  using namespace boost::lambda;
  std::vector<int> vec1;
  vec1.push_back(2);
  vec1.push_back(3);
  vec1.push_back(5);
  vec1.push_back(7);
  vec1.push_back(11);
  std::vector<int> vec2;
  vec2.push_back(7);
  vec2.push_back(4);
  vec2.push_back(2);
  vec2.push_back(3);
  vec2.push_back(1);
  std::cout << *std::find_if(vec1.begin(),vec1.end(),
    (_1>=3 && _1<5) || _1<1) << '\n';
  std::cout << *std::find_if(vec2.begin(),vec2.end(),
    _1>=4 && _1<10) << '\n';
  std::cout << *std::find_if(vec1.begin(),vec1.end(),
    _1==4 || _1==5) << '\n';
  std::cout << *std::find_if(vec2.begin(),vec2.end(),
    _1!=7 && _1<10) << '\n';
  std::cout << *std::find_if(vec1.begin(),vec1.end(),
    !(_1%3)) << '\n';
  std::cout << *std::find_if(vec2.begin(),vec2.end(),
    _1/2<3) << '\n';
} 
```

如你所见，创建这些谓词就象写出相应的逻辑一样容易。这正是我喜欢使用 lambda 表达式的地方，因为它可以被任何人所理解。有时候我们也需要选择 lambda 表达式以外的机制，因为那些必须理解这些代码的人的能力；但是在这里，除了增加的价值以外没有其它了。

### <a class="calibre36" id="ch10lev2sec9">让你的函数对象可以与 Boost.Lambda 一起使用

不是所有的表达式都适合使用 lambda 表达式，复杂的表达式更适合使用普通的函数对象，而且会多次重用的表达式也应该成为你代码中的一等公民。它们应该被收集为一个可重用函数对象的库。但是， 你也可能想把这些函数对象用在 lambda 表达式中，你希望它们可以与 Lambda 一起使用；不是所有函数对象都能做到。问题是函数对象的返回类型不能象普通函数那样被推断出来；这是语言的固有限制。但是，有一个定义好的方法来把这个重 要的信息提供给 Lambda 库，以使得 `bind` 表达式更加干净。作为这个问题的一个例子，我们看以下函数对象：

```cpp
template <typename T> class add_prev {
  T prev_;
public:
  T operator()(T t) {
    prev_+=t;
    return prev_;
  }
}; 
```

对于这样一个函数对象，lambda 表达式不能推断出返回类型，因此以下例子不能编译。

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include "boost/lambda/lambda.hpp"
#include "boost/lambda/bind.hpp"
int main() {
  using namespace boost::lambda;
  std::vector<int> vec;
  vec.push_back(5);
  vec.push_back(8);
  vec.push_back(2);
  vec.push_back(1);
  add_prev<int> ap;
  std::transform(
    vec.begin(),
    vec.end(),
    vec.begin(),
    bind(var(ap),_1));
} 
```

问题在于对 `transform` 的调用。

```cpp
std::transform(vec.begin(),vec.end(),vec.begin(),bind(var(ap),_1)); 
```

当绑定器被实例化时，返回类型推断的机制被使用…而且失败了。因此，这段程序不能通过编译，你必须显式地告诉 `bind` 返回类型是什么，象这样：

```cpp
std::transform(vec.begin(),vec.end(),vec.begin(),
  bind<int>(var(ap),_1)); 
```

这是为 lambda 表达式显式设置返回类型的正常格式的缩写，它等价于这段代码。

```cpp
std::transform(vec.begin(),vec.end(),vec.begin(),
  ret<int>(bind<int>(var(ap),_1))); 
```

这并不是什么新问题；对于在标准库算法中使用函数对象都有同样的问题。在标准库中，解决的方法是增加 `typedef`s 来表明函数对象的返回类型及参数类型。标准库还提供了助手类来完成这件事，即类模板 `unary_function` 和 `binary_function`，要让我们的例子类 `add_prev` 成为合适的函数对象，可以通过定义所需的 `typedef`s (对于一元函数对象，是`argument_type` 和 `result_type`，对于二元函数对象，是`first_argument_type`, `second_argument_type`, 和 `result_type`)，也可以通过派生自 `unary_function/binary_function` 来实现。

```cpp
template <typename T> class add_prev : public std::unary_function<T,T> 
```

这对于 lambda 表达式是否也足够好了呢？我们可以简单地复用这种方法以及我们已有的函数对象吗？唉，答案是否定的。这种 `typedef` 方法有一个问题：对于泛化的调用操作符，当返回类型或参数类型依赖于模板参数时会怎么样？或者，当存在多个重载的调用操作符时会怎么样？由于语言支持模板的 `typedef`s, 这些问题可以解决，但是现在不是这样的。这就是为什么 Boost.Lambda 需要一个不同的方法，即一个名为 `sig` 的嵌套泛型类。为了让返回类型推断可以和 `add_prev` 一起使用，我们象下面那样定义一个嵌套类型 `sig` ：

```cpp
template <typename T> class add_prev :
  public std::unary_function<T,T> {
  T prev_;
public:
  template <typename Args> class sig {
  public:
    typedef T type;
   };
// Rest of definition 
```

模板参数 `Args` 实际上是一个 tuple，包含了函数对象(第一个元素)和调用操作符的参数类型。在这个例子中，我们不需要这些信息，返回类型和参数类型都是 `T`. 使用这个改进版本的 `add_prev`, 再不需要在 lambda 表达式中使用返回类型推断的缩写，因此我们最早那个版本的代码现在可以编译了。

```cpp
std::transform(vec.begin(),vec.end(),vec.begin(),bind(var(ap),_1)); 
```

我们再来看看 tuple 作为 `sig` 的模板参数是如何工作的，来看另一个有两个调用操作符的函数对象，其中一个版本接受一个 `int` 参数，另一个版本接受一个 `const std::string` 引用。我们必须要解决的问题是，"如果传递给 `sig` 模板的 tuple 的第二个元素类型为 `int`, 则设置返回类型为 `std::string`; 如果传递给 `sig` 模板的 tuple 的第二个元素类型为 `std::string`, 则设置返回类型为 `double`"。为此，我们增加一个类模板，我们可以对它进行特化并在 `add_prev::sig` 中使用它。

```cpp
template <typename T> class sig_helper {};
// The version for the overload on int
template<> class sig_helper<int> {
public:
  typedef std::string type;
};
// The version for the overload on std::string
template<> class sig_helper<std::string> {
public:
  typedef double type;
};
// The function object
class some_function_object {
  template <typename Args> class sig {
    typedef typename boost::tuples::element<1,Args>::type
      cv_first_argument_type;
    typedef typename
      boost::remove_cv<cv_first_argument_type>::type
      first_argument_type;
  public:
    // The first argument helps us decide the correct version
    typedef typename
      sig_helper<first_argument_type>::type type;
  };
  std::string operator()(int i) const {
    std::cout << i << '\n';
    return "Hello!";
  }
  double operator()(const std::string& s) const {
    std::cout << s << '\n';
    return 3.14159265353;
  }
}; 
```

这里有两个重要的部分要讨论：首先是助手类 `sig_helper`, 它由类型 `T` 特化。这个类型可以是 `int` 或 `std::string`, 依赖于要使用哪一个重载版本的调用操作符。通过对这个模板进行全特化，来定义正确的 `typedef` `type`。第二个要注意的部分是 `sig` 类，它的第一个参数(即 tuple 的第二个元素)被取出，并去掉所有的 `const` 或 `volatile` 限定符，结果类型被用于实例化正确版本的 `sig_helper` 类，后者具有正确的 `typedef type`. 这是为我们的类定义返回类型的一种相当复杂(但是必须！)的方法，但是多数情况下，通常都只有一个版本的调用操作符；所以正确地增加嵌套 `sig` 类是一件普通的工作。

我们的函数对象可以在 lambda 表达式中正确使用是很重要的，在需要时定义嵌套 `sig` 类是一个好主意；它很有帮助。

### Lambda 表达式中的控制结构

我们已经看到强大的 lambda 表达式可以很容易地创建，但是许多编程上的问题需要我们可以表示条件，在 C++中我们使用 `if`-`then`-`else`, `for`, `while`, 等等。在 Boost.Lambda 中有所有的 C++控制结构的 lambda 版本。要使用选择语句，`if` 和 `switch`, 就分别包含头文件 `"boost/lambda/if.hpp"` 和 `"boost/lambda/switch.hpp"`。要使用循环语句，`while`, `do`, 和 `for`, 就包含头文件 `"boost/lambda/loops.hpp"`. 关键字不能被重载，所以语法与你前面使用过的有所不同，但是也有很明显的关联。作为第一个例子，我们来看看如何在 lambda 表达式中创建一个简单的 if-then-else 结构。格式是 *if_then_else(*条件*, then-*语句*, else-*语句*)*。还有另外一种语法形式，即 *if*(*条件*)[then-*语句*].else*[else-*语句*]* 。

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <string>
#include "boost/lambda/lambda.hpp"
#include "boost/lambda/bind.hpp"
#include "boost/lambda/if.hpp"
int main() {
  using namespace boost::lambda;
  std::vector<std::string> vec;
  vec.push_back("Lambda");
  vec.push_back("expressions");
  vec.push_back("really");
  vec.push_back("rock");
  std::for_each(vec.begin(),vec.end(),if_then_else(
    bind(&std::string::size,_1)<=6u,
    std::cout << _1 << '\n',
    std::cout << constant("Skip.\n")));
  std::for_each(vec.begin(),vec.end(),
    if_(bind(&std::string::size,_1)<=6u) [
      std::cout << _1 << '\n'
    ]
    .else_[
      std::cout << constant("Skip.\n")
    ] );
} 
```

如果你是从本章开头一直读到这的，你可能会觉得上述代码非常好读；但如果你是跳到这来的，就可能觉得惊讶了。控制结构的确增加了阅读 lambda 表达式的复杂度，它需要更长一点的时间来掌握它的用法。当你掌握了它以后，它就变得很自然了(编写它们也一样！)。采用哪一种格式完全取决于你的爱好；它们做得是同一件事。

在上例中，我们有一个 `string` 的 `vector`，如果 `string` 元素的大小小于等于 6，它们就被输出到 `std::cout`; 否则，输出字符串 `"Skip"`。在这个 `if_then_else` 表达式中有一些东西值得留意。

```cpp
if_then_else(
  bind(&std::string::size,_1)<=6u,
  std::cout << _1 << '\n',
  std::cout << constant("Skip.\n"))); 
```

首先，条件是一个谓词，它必须是一个 lambda 表达式！其次，*then*-语句必须也是一个 lambda 表达式！第三，*else*-语句必须也是一个 lambda 表达式！头两个都很容易写出来，但最后一个很容易忘掉用 `constant` 来把字符串("Skip\n")变成一个 lambda 表达式。细心的读者会注意到例子中使用了 `6u`, 而不是使用 `6`, 这是为了确保执行的是两个无符号类型的比较。这样做的原因是，我们使用的是非常深的嵌套模板，这意味着如果这样一个 lambda 表达式引发了一个编译器警告，输出信息将会非常、非常长。你可以试一下去掉这个 `u`，看看你的编译器会怎样！你将看到一个关于带符号类型与无符号类型比较的警告，因为 `std::string::size` 返回一个无符号类型。

控制结构的返回类型是 `void`, 除了 `if_then_else_return`, 它调用条件操作符。让我们来仔细看看所有控制结构，从 `if` 和 `switch` 开始。记住，要使用 `if`-结构，必须包含 `"boost/lambda/if.hpp"`。对于 `switch`, 必须包含 `"boost/lambda/switch.hpp"`。以下例子都假定名字空间 `boost::lambda` 中的声明已经通过 using 声明或 using 指令，被带入当前名字空间。

```cpp
(if_then(_1<5,
  std::cout << constant("Less than 5")))(make_const(3)); 
```

`if_then` 函数以一个条件开始，后跟一个 *then*-部分；在上面的代码中，如果传给该 lambda 函数的参数小于 5 (`_1&lt;5`), `"Less than 5"` 将被输出到 `std::cout`. 你会看到如果我们用数值 3 调用这个 lambda 表达式，我们不能直接传递 3，象这样。

```cpp
(if_then(_1<5,std::cout << constant("Less than 5")))(3); 
```

这会引起一个编译错误，因为 3 是一个 `int`, 而一个类型 `int` (或者任何内建类型)的左值不能被 `const` 限定。因此，我们在这里必须使用工具 `make_const`，它只是返回一个对它的参数的 `const` 引用。另一个方法是把整个 lambda 表达式用于调用 `const_parameters`, 象这样：

```cpp
(const_parameters(
  if_then(_1<5,std::cout << constant("Less than 5"))))(3); 
```

`const_parameters` 对于避免对多个参数分别进行 `make_const` 非常有用。注意，使用该函数时，lambda 表达式的所有参数都被视为 `const` 引用。

现在来看另一种语法的 `if_then` 。

```cpp
(if_(_1<5)
  [std::cout << constant("Less than 5")])(make_const(3)); 
```

这种写法更类似于 C++关键字，但它与 `if_then` 所做的完全一样。函数 `if_` (注意最后的下划线)后跟括起来的条件，再后跟 *then*-语句。重复一次，选择哪种语法完全取决于你的口味。

现在，让我们来看看 *if-then-else* 结构；它们与 `if_then` 很相似。

```cpp
(if_then_else(
  _1==0,
  std::cout << constant("Nothing"),
  std::cout << _1))(make_const(0));
(if_(_1==0)
  [std::cout << constant("Nothing")].
  else_[std::cout << _1])(make_const(0)); 
```

使用第二种语法增加 else-部分时，要留意 `else_` 前面的点。

lambda 表达式的返回类型是 `void`, 但是有一个版本会返回一个值，它使用条件操作符。对于这种表达式的类型有一些不平常的规则(我在这里略过它们，你可以在 Boost.Lambda 的在线文档或 C++ 标准[§5.16] 找到详细的说明)。这里有一个例子，返回值被赋给一个变量，就象你在使用条件操作符一样。

```cpp
int i;
int value=12;
var(i)=(if_then_else_return
  (_1>=10,constant(10),_1))(value); 
```

这个结构没有第二种语法。这些就是 *if-then-else*, 我们再看看 *switch*-语句，它与标准 C++ switch 有些不同。

```cpp
(switch_statement
  _1,
  case_statement<0>
    (var(std::cout) << "Nothing"),
  case_statement<1>
    (std::cout << constant("A little")),
  default_statement
    (std::cout << _1))
  )(make_const(100)); 
```

对 `switch_statement` 的调用从条件变量开始，即我们这里的 `_1`, lambda 表达式的第一个参数。它后跟(最多九个)表现为整型的 case 常量；它们必须是整型的常量表达式。我们提供了两个这样的常量，0 和 1 (注意，它们可以是任何可作为整型类型的值)。最手，我们加一个可选的 `default_statement`, 它在 `_1` 不匹配任何一个常量时被执行。注意，在每一个 case 常量后都隐式地增加了一个 `break`-语句，所以无需从 switch 显式退出(这对于代码的维护是一件好事[6])。

[6] Spokesmen of fall-through case-statements; please excuse this blasphemy.

现在我们来看循环语句，`for`, `while`, 和 `do`. 要使用它们中的任意一个，你必须首先包含头文件 `"boost/lambda/loops.hpp"`。Boost.Lambda 中与 C++的 `while` 相对应的是 `while_loop`。

```cpp
int val1=1;
int val2=4;
(while_loop(_1<_2,
  (++_1,std::cout << constant("Inc...\n"))))(val1,val2); 
```

`while_loop` 语句执行到条件为 `false` 止；这里的条件是 `_1&lt;_2`, 后跟循环体，即表达式 `++_1,std::cout &lt;&lt; constant("Inc...\n")`. 当然，条件和循环体本身必须是有效的 lambda 表达式。另一种语法更接近 C++语法，就象 `if_` 那样。

```cpp
int val1=1;
int val2=4;
(while_(_1<_2)
  [++_1,std::cout << constant("Inc...\n")])(val1,val2); 
```

格式是 `while_(`条件`)[`子语句`]`, 它可以节省不少输入…但是我个人认为对于 `while` 而言函数调用语法更容易读，虽然我(不太合理)认为 `if_` 比 `if_then(...)` 容易看。从外表看，`do_while_loop` 与 `while_loop` 非常相似，但它的子语句至少被执行一次(不象 `while`, 它的条件在每次执行后求值)。

```cpp
(do_while_loop(_1!=12,std::cout <<
  constant("I'll run once")))(make_const(12)); 
```

另一种语法是

```cpp
(do_[std::cout <<
  constant("I'll run once")].while_(_1!=12))(make_const(12)); 
```

最后是 `for` 循环的对应，`for_loop`. 在以下例子中，使用了一个命名的延期变量来让 lambda 表达式更可读。我们在前面介绍 `constant` 和 `var` 时已经介绍过延期变量。命名的延期变量用于避免重复为常量和变量敲入 `constant` 或 `var` 。它们对你要用的东西进行命名并稍后可被引用。常用的循环格式是 `for_loop`(*init-*语句*,* 条件*,* 表达式*,* 语句)，即它与一个普通语句相似，但它是函数的一部分(参数)。

```cpp
int val1=0;
var_type<int>::type counter(var(val1));
(for_loop(counter=0,counter<_1,++counter,var(std::cout)
  << "counter is " << counter << "\n"))(make_const(4)); 
```

采用另一种语法，语句被分为初始化、条件和表达式。

```cpp
(for_(counter=0,counter<_1,++counter)[var(std::cout)
  << "counter is " << counter << "\n"])(make_const(4)); 
```

这个例子把延期变量 `counter` 初始化为 0，条件为 `counter&lt;_1`, 表达式为 `++counter`.

总结一下本节的控制结构。对于我遇到并使用 lambda 表达式来解决的多数问题，事实上也可以不用它们，但有些时候，它们的确真的是救命者。对于选择哪一种语法版本，最好的办法可能是两种都用，然后感觉一下哪一种最适合于你。要注意的一点是，使用 `switch` 和循环结构时，lambda 表达式很快就会变得很大，如果你还不能熟练使用这个库，你将很难弄懂这样的表达式。这时应当小心，如果一个表达式看起来让你的程序员同事难以分析，不要考虑使用独立的函数对象(或者让他们更加熟练地使用 Boost.Lambda！)。

### Lambda 表达式中的类型转换

在 lambda 表达式中有四种特殊的"转型操作符"[7] 来进行类型的转换：`ll_dynamic_cast`, `ll_static_cast`, `ll_reinterpret_cast`, 和 `ll_const_cast`. 这些名字与对应的 C++关键字不一样，因为它们不能被重载。要使用这些类型转换，就要包含头文件 `"boost/lambda/casts.hpp"`. 这些函数与相对应的 C++转型操作符用法类似；它们带一个显式的模板参数，即要转成的目标类型，以及一个隐式的模板参数，即源类型。在我们的第一个例子中，我们将使用两个类，名为 `base` 和 `derived`. 我们将创建两个指向 `base` 的指针，一个指向 `base` 实例，另一个指向 `derived` 实例。然后我们尝试使用 `ll_dynamic_cast` 来从这两个指针分别获得一个 `derived*` 。

[7] 技术上，它们是返回函数对象的模板函数。

```cpp
#include <iostream>
#include "boost/lambda/lambda.hpp"
#include "boost/lambda/casts.hpp"
#include "boost/lambda/if.hpp"
#include "boost/lambda/bind.hpp"

class base {
public:
  virtual ~base() {}
  void do_stuff() const {
    std::cout << "void base::do_stuff() const\n";
  }
};

class derived : public base {
public:
  void do_more_stuff() const {
    std::cout << "void derived::do_more_stuff() const\n";
  }
};

int main() {
  using namespace boost::lambda;
  base* p1=new base;
  base* p2=new derived;
  derived* pd=0;
  (if_(var(pd)=ll_dynamic_cast<derived*>(_1))
    [bind(&derived::do_more_stuff,var(pd))].
    else_[bind(&base::do_stuff,*_1)])(p1);
  (if_(var(pd)=ll_dynamic_cast<derived*>(_1))
    [bind(&derived::do_more_stuff,var(pd))].
    else_[bind(&base::do_stuff,*_1)])(p2);
} 
```

在 `main` 中，我们做的第一件事情是创建 `p1` 和 `p2`; `p1` 指向一个 `base`, 而 `p2` 则指向一个 `derived`. 在第一个 lambda 表达式中，被赋值的 `pd` 变成了条件；它被隐式转换为 `bool`, 如果它为 `true`, *then*-部分被求值。这里，我们 `bind` 到成员函数 `do_more_stuff`. 如果 `ll_dynamic_cast` 失败了，延期变量 `pd` 将为 0, 则 *else*-部分被执行。因此，在我们例子中，lambda 表达式的第一次执行将调用 `base` 上的 `do_stuff` ，而第二次执行则调用 `derived` 上的 `do_more_stuff` ，运行这个程序的输出如下。

```cpp
void base::do_stuff() const
void derived::do_more_stuff() const 
```

注意，在这个例子中，参数 `_1` 被解引用，但这并不是必须的；如果需要它会隐式地完成。如果一个 `bind` 表达式的某个参数必须总是一个指针类型，你可以自己强制对它解引用。否则，把这件杂事留给 Boost.Lambda.

`ll_static_cast` 对于避免警告非常有用。不要用它来抑制重要的信息，但可以用来减少噪音。在前面的一个例子中，我们创建了一个 `bind` 表达式来求一个 `std::string` 的长度(使用 `std::string::size`)并将该长度与另一个整数进行比较。`std::string::size` 的返回类型是一个无符号类型，把它与一个有符号整数进行比较(这很常见)，编译器会产生一个警告，说有符号与无符号的比较是危险的动作。但是，因为这发生 在一个 lambda 表达式中，编译器忠实地跟踪到问题的根源，告诉你某个嵌套模板的某部分应对此严重问题负责。结果是一个非常长的警告信息，由于低信噪比的原因，它可能会掩 盖了其它的问题。在泛型代码中，这有时会成为问题，因为所使用的类型不在我们的控制之内。因而，在评估过可能潜在的问题后，你常会发现使用 `ll_static_cast` 来抑制不想要的警告是有好处的。以下例子包含了示范这种行为的代码。

```cpp
#include <iostream>
#include <string>
#include <algorithm>
#include "boost/lambda/lambda.hpp"
#include "boost/lambda/casts.hpp"
#include "boost/lambda/if.hpp"
#include "boost/lambda/bind.hpp"

template <typename String,typename Integral>
void is_it_long(const String& s,const Integral& i) {
  using namespace boost::lambda;
  (if_then_else(bind(&String::size,_1)<_2,
    var(std::cout) << "Quite short...\n",
    std::cout << constant("Quite long...\n")))(s,i);
}

int main() {
  std::string s="Is this string long?";
  is_it_long(s,4u);
  is_it_long(s,4);
} 
```

泛型函数 `is_it_long` (请不要关注它是一个有点过于做作的例子)用一个 `Integral` 类型的常数变量引用来执行一个 lambda 表达式。现在，这个类型是有符号的还是无符号的并不在我们的控制之内，因此很有机会某个用户会不小心引发了一个非常冗长的警告，正如这个例子所示范的，因为用了一个有符号整数调用 `is_it_long` 。

```cpp
is_it_long(s,4); 
```

确保用户不会无意地引发此事(除了要求只能使用无符号类型)的唯一办法是让这个参数变成无符号整数类型，不管它原来是什么。这正是 `ll_static_cast` 的工作，因此我们把函数 `is_it_long` 改成这样：

```cpp
template <typename String,typename Integral>
void is_it_long(const String& s,const Integral& i) {
  using namespace boost::lambda;
  (if_then_else(bind(&String::size,_1)<
    ll_static_cast<typename String::size_type>(_2),
    var(std::cout) << "Quite short...\n",
    std::cout << constant("Quite long...\n")))(s,i);
} 
```

这种情况不会经常发生(至少我没碰到几次)，但只要它发生了，这种解决方法就可用。`ll_const_cast` 和 `ll_reinterpret_cast` 的用法与我们已经看到的相似，所以就不再举例了。小心地使用它们，如果没有非常必要的原因(我想不到有什么原因)，尽量不要使用 `ll_reinterpret_cast` 。这是对称的；如果你需要用它，很大机会你做了一些不应该做的事情。

### 构造与析构

当有必要在 lambda 表达式中创建或销毁对象时，就需要一些特殊的处理和语法。首先，你不可能获取构造函数或析构函数的地址，也就不可能对它们使用标准的 `bind` 表达式。此外，操作符 `new` 和 `delete` 有固定的返回类型，因此它们对于任何类型都不能返回 lambda 表达式。如果你需要在 lambda 表达式中创建或销毁对象，先要确保包含头文件 `"boost/lambda/construct.hpp"`, 它包含了模板 `constructor`, `destructor`, `new_ptr`, `new_array`, `delete_ptr`, 以及 `delete_array`. 我们来看看如何使用它们，并主要关注 `constructor` 和 `new_ptr`, 它们在构造对象时是最常用的。

我们的第一个例子是一个以智能指针作为元素的容器，我们想在 lambda 表达式中重设智能指针的内容。这通常需要一个对 `operator new` 的调用；例外的情形是使用了客户化的分配机制，或者是某种工厂方法(factory method)。我们要用 `new_ptr` 来做，如果你想要或者需要的话，通常也可以在赋值表达式中使用 `constructor` 。我们两种方法都试一下。我们将定义两个类，`base` 和 `derived`, 以及一个 `boost::shared_ptr&lt;base&gt;` 的 `std::map` ，它以 `std::strings` 为索引。在阅读本例中的 lambda 表达式之前，先来一下深呼吸；它们将是你在本章所见到的最复杂的两个 lambda 表达式。虽然复杂，但是要理解它们是干什么的还是很明白的。只是要花你一点时间。

```cpp
#include <iostream>
#include <map>
#include <string>
#include <algorithm>
#include "boost/lambda/lambda.hpp"
#include "boost/lambda/construct.hpp"
#include "boost/lambda/bind.hpp"
#include "boost/lambda/if.hpp"
#include "boost/lambda/casts.hpp"
#include "boost/shared_ptr.hpp"

class base {
public:
  virtual ~base() {}
};

class derived : public base {
};

int main() {
  using namespace boost::lambda;

  typedef boost::shared_ptr<base> ptr_type;
  typedef std::map<std::string,ptr_type> map_type;

  map_type m;
  m["An object"]=ptr_type(new base);
  m["Another object"]=ptr_type();
  m["Yet another object"]=ptr_type(new base);

  std::for_each(m.begin(),m.end(),
    if_then_else(!bind(&ptr_type::get,
      bind(&map_type::value_type::second,_1)),
      (bind(&map_type::value_type::second,_1)=
        bind(constructor<ptr_type>(),bind(new_ptr<derived>())),
        var(std::cout) << "Created a new derived for \"" <<
          bind(&map_type::value_type::first,_1) << "\".\n"),
      var(std::cout) << "\"" <<
        bind(&map_type::value_type::first,_1)
        << "\" already has a valid pointer.\n"));

  m["Beware, this is slightly tricky"]=ptr_type();
  std::cout << "\nHere we go again...\n";

  std::for_each(m.begin(),m.end(),
    if_then_else(!bind(&map_type::value_type::second,_1),
      ((bind(static_cast<void (ptr_type::*)(base*)>
        (&ptr_type::reset<base>),
        bind(&map_type::value_type::second,_1),
        bind(new_ptr<base>()))),
      var(std::cout) << "Created a new derived for \""
        << bind(&map_type::value_type::first,_1)
        << "\".\n"),
      var(std::cout) << "\"" <<
        bind(&map_type::value_type::first,_1)
        << "\" already has a valid pointer.\n"));
} 
```

你都看懂了，是吗？以防万一有些混乱，我来解释一下在这个例子中发生了什么。首先，这两个 lambda 表达式做的是同一件事情。它们对 `std::map` 中的每一个当前为 null 的元素设置有效的指针。以下是程序运行的输出：

```cpp
"An object" already has a valid pointer.
"Another object" already has a valid pointer.
"Yet another object" already has a valid pointer.
<small class="calibre41">// 译注：前面这三行在原书中有重复，共六行，与程序运行结果不符</small>

Here we go again...
"An object" already has a valid pointer.
"Another object" already has a valid pointer.
Created a new derived for "Beware, this is slightly tricky".
"Yet another object" already has a valid pointer. 
```

输出显示我们设法把有效的对象放入 `map` 的每一个元素，但是这是如何办到的呢？

两个表达式完成了相同的工作，但各自有不同的方法。从第一个开始，我们来把这个 lambda 表达式切开来看看它是如何工作的。当然，第一部分是条件，它很普通：[8]

[8] 它还可以更简单，我们即将会看到。

```cpp
!bind(&ptr_type::get,bind(&map_type::value_type::second,_1)) 
```

这样更容易看了，对吗？从最里面的 `bind` 开始读这个表达式，它告诉我们将绑定到成员 `map_type::value_type::second` (它是一个 `ptr_type`)，然后我们再绑定成员函数 `ptr_type::get` (它返回 `shared_ptr` 的指向物)，最后我们对整个表达式执行 `operator!`. 由于指针可以隐式转换为 `bool`, 因此这是一个有效的布尔表达式。看完条件部分，我们来看 *then*-部分。

```cpp
bind(&map_type::value_type::second,_1)=
  bind(constructor<ptr_type>(),
    bind(new_ptr<derived>())), 
```

这里有三个 `bind` 表达式，第一个(我们从最左边开始，因为这个表达式有一个赋值)取出成员 `map_type::value_type::second`, 它是一个智能指针。这就是我们要赋给它一个新的 `derived` 的那个值。第二和第三个表达式是嵌套的，所以我们从里向外读。里面的 `bind` 负责堆上的一个 `derived` 实例的缺省构造，我们再在它的结果之上 `bind` 一个对 `ptr_type` (智能指针类型)的 `constructor` 的调用，然后把它的结果赋值(使用普通的赋值符)给最先的那个 `bind` 表达式。然后，我们给这个 *then*-部分再加一个表达式，打印出一个简短的信息和元素的键值。

```cpp
var(std::cout) << "Created a new derived for \"" <<
  bind(&map_type::value_type::first,_1) << "\".\n") 
```

最后，我们加上语句的 *else*-部分，它打印出元素的键值和一些文字。

```cpp
var(std::cout) << "\"" <<
  bind(&map_type::value_type::first,_1)
  << "\" already has a valid pointer.\n")); 
```

把这个表达式分开来读，很明显它们并不是那么复杂，虽然整个 看起来很可怕。很重要的一点是，缩进和分离这些代码可以更容易阅读。我们可以写出类似的表达式来完成这件工作，它与这一个版本非常不同，更难阅读，但是它 的效率稍高一些。这里要注意的是，通常都会有好几种方法来写 lambda 表达式，就象其它编程问题的情况一样。在写代码之前应该多想一下，因为你的选择会影响最终结果的可读性。作为比较，以下是我提到的另一个版本：

```cpp
std::for_each(m.begin(),m.end(),
  if_then_else(!bind(&map_type::value_type::second,_1),
    ((bind(static_cast<void (ptr_type::*)(base*)>
      (&ptr_type::reset<base>),
      bind(&map_type::value_type::second,_1),
      bind(new_ptr<derived>()))),
    var(std::cout) << "Created a new derived for \"" <<
      bind(&map_type::value_type::first,_1) << "\".\n"),
    var(std::cout) << "\"" <<
      bind(&map_type::value_type::first,_1)
      << "\" already has a valid pointer.\n")); 
```

这不是好的代码，这些代码由于类型转换和复杂的嵌套 `bind`s 而变得混乱，与前面那个版本相比，这个版本很容易使我们偏离主要逻辑。为了弄明白它，我们再来把这个表达式切开成几个部分。首先，我们有条件部分，它很简单(这个表达式中没有其它东西！)；我们利用对 `shared_ptr` 的了解，它告诉我们有一个到 `bool` 的隐式转换可用。因此我们可以去掉在前一个版本中使用的到成员函数 `get` 的 `bind` 。

```cpp
!bind(&map_type::value_type::second,_1) 
```

这个条件部分与原先的表达式一样工作。接下来的部分是：

```cpp
bind(static_cast<void (ptr_type::*)(base*)>
  (&ptr_type::reset<base>),
  bind(&map_type::value_type::second,_1),
  bind(new_ptr<derived>())) 
```

这真的很难分析，所以我们先避开它。我们不使用赋值，而是直接使用成员函数 `reset`, 它不仅是泛化的而且还是重载的。因此我们需要执行 `static_cast` 来告诉编译器我们要用那个版本的 `reset` 。这里主要是 `static_cast` 让表达式变得复杂，但是从最里面的表达式开始分析，我们就可以弄懂它。我们绑定一个调用到 `operator new`, 创建一个 `derived` 实例，然后我们把结果绑定到智能指针(通过成员 `map_type::value_type::second`), 然后再绑定到 `shared_ptr` 的成员函数 `reset`. 结果是，对元素中的智能指针调用 `reset` ，参数是一个新构造的 `derived` 实例。虽然我们在前一个例子中也完成了同样的工作，但这个版本更难以理解。

要记住，通常有一些因素让 lambda 表达式更容易或者更难阅读和理解，要认真考虑这些因素并尽可能选择容易的形式。这对于获得这个库所提供的能力是必要的，而且它会影响到你后面的程序员。

### 抛出及捕获异常

我们已经来到本章的最后一节，讨论 lambda 表达式中的异常处理。如果你对这个话题的反应是怀疑在 lambda 表达式中是否需要异常处理，这就和我的第一感觉一样了。但是，这可能还不是你的想法。你写过在处理数据的循环中执行局部异常处理的代码吗？是的，手写的循 环可以避免使用 Boost.Lambda 库，因此把异常处理放入 lambda 表达式是很自然的。

要使用 Boost.Lambda 的异常处理工具，就要包含头文件 `"boost/lambda/exceptions.hpp"`. 我们来重用一下前面看到的类 `base` 和 `derived` ，并象我们前面做过的那样执行 `dynamic_cast` ，不过这次我们要对引用执行转型而不是对指针，这意味着如果失败的话，`dynamic_cast` 将抛出一个异常。这使得这个例子比前面那个更直观，因为我们不需要再使用 `if` 语句。

```cpp
#include <iostream>
#include "boost/lambda/lambda.hpp"
#include "boost/lambda/casts.hpp"
#include "boost/lambda/bind.hpp"
#include "boost/lambda/exceptions.hpp"

int main() {
  using namespace boost::lambda;

  base* p1=new base;
  base* p2=new derived;

  (try_catch(
    bind(&derived::do_more_stuff,ll_dynamic_cast<derived&>(*_1)),
      catch_exception<std::bad_cast>(bind(&base::do_stuff,_1))))(p1);
  (try_catch(
    bind(&derived::do_more_stuff,
      ll_dynamic_cast<derived&>(*_1)),
        catch_exception<std::bad_cast>(
          bind(&base::do_stuff,_1))))(p2);
} 
```

这些表达式示范了把一个表达式包装到一个对 tr`y_catch` 的调用。`try_catch` 的通常形式为：

```cpp
try_catch(_expression_,
  catch_exception<T1>(_expression_),
  catch_exception<T2>(_expression_,
  catch_all(_expression_)) 
```

在这段例子代码中，表达式对 `derived&` 使用 `dynamic_cast`。第一个转型由于 `p1` 指向一个 `base` 实例而失败；第二个转型则由于 `p2` 指向一个 `derived` 实例而成功。请留意对占位符的解引用(`*_1`)。这是必须的，因为我们是传送指针参数给表达式的，而 `dynamic_cast` 要求的是对象或引用。如果你需要 `try_catch` 处理几种类型的异常，就要确保把最特化的类型放在前面，就象普通的异常处理代码一样。[9]

9] 否则，一个更为通用的类型将会匹配该异常而不能查找到更为特殊的类型。具体详情请参考你喜欢的 C++书籍。

如果我们想访问捕获的异常，可以用一个特别的占位符，`_e`. 当然，在 `catch_all` 中不能这样做，就象在 `catch (...)` 中没有异常对象一样。继续前面的例子，我们可以打印出令 `dynamic_cast` 失败的原因，象这样：

```cpp
try_catch(
  bind(&derived::do_more_stuff,ll_dynamic_cast<derived&>(*_1)),
  catch_exception<std::bad_cast>
    (std::cout << bind(&std::exception::what,_e))))(p1); 
```

在处理一个派生自 `std::exception` 的异常类型时——这是很常见的情形——你可以绑定到虚拟成员函数 `what`, 就象这里所示范的那样。

但是有时候，你不想捕获异常，而是想抛出一个异常。这可以通过函数 `throw_exception` 来实现。因为你需要创建一个异常对象用来抛出，所以从一个 lambda 表达式中抛出异常通常还要使用 `constructor` 。下面这个例子定义了一个异常类，`some_exception`, 它公有派生自 `std::exception`, 如果 lambda 表达式的参数为 `true` ，就创建并抛出一个异常。

```cpp
#include <iostream>
#include <exception>
#include "boost/lambda/lambda.hpp"
#include "boost/lambda/exceptions.hpp"
#include "boost/lambda/if.hpp"
#include "boost/lambda/construct.hpp"
#include "boost/lambda/bind.hpp"

class some_exception : public std::exception {
  std::string what_;
public:
  some_exception(const char* what) : what_(what) {}

  virtual const char* what() const throw() {
    return what_.c_str();
  }
  virtual ~some_exception() throw() {}
};

int main() {
  using namespace boost::lambda;

  try {
    std::cout << "Throw an exception here.\n";

    (if_then(_1==true,throw_exception(
      bind(constructor<some_exception>(),
        constant("Somewhere, something went \
        terribly wrong.")))))(make_const(true));

    std::cout << "We'll never get here!\n";
  }
  catch(some_exception& e) {
    std::cout << "Caught exception, \"" << e.what() << "\"\n";
  }
} 
```

运行这段程序将产生以下输出：

```cpp
Throw an exception here.
Caught exception, "Somewhere, something went terribly wrong." 
```

最有趣的地方是抛出异常的那段代码。

```cpp
throw_exception(
  bind(constructor<some_exception>(),
    constant("Somewhere, something went \
      terribly wrong.")) 
```

`throw_exception` 的参数是一个 lambda 表达式。这个例子中，它被创建并绑定到对 `some_exception` 构造函数的调用，我们把一个字符串作为 `what` 参数传给该构造函数。

这就是在 Boost.Lambda 中进行异常处理的全部内容。永远记住，要小心谨慎地使用这些工具，它们可以让生活更轻松或更困难，这取决于你能否很好地并明智地使用它们[10] 。抛出并处理异常在你的 lambda 表达式中并不常见，但有些时候还是需要的。

[10] 小心应了这句格言，"如果你只有锤子，那么所有东西看起来都象钉子"。

# Lambda 总结

## Lambda 总结

以下情形时使用 Lambda ：

*   你不想创建一个简单的函数对象

*   你需要在调用函数时调整参数顺序或 arity

*   你想就地创建与标准一致的函数对象

*   你需要灵活并可读的谓词

上述原因只是值得使用本库的几种情形。虽然多数情况下，它会与标准库算法一起用，至少部分原因是由于在其它库(就算是 Boost 库)中这样的设计还不多见。通过函数对象来进行算法配置的需要并不能验证本库的有效性，离完全弄清楚它在哪些地方可以带来好处还有很长一段距离。思考一下这个库可能的应用，一定会提高你当前的设计。

Boost.Lambda 是我最喜欢的库之一，主要是因为它提供语言本身没有的很多可用的功能。要象 STL 在全世界所有程序员的心中一样，它还缺少一些东西。要让算法高效地工作， 还需要一些函数对象以外的东西。这正是 Boost.Lambda 的推动力，它的丰富特性带来了真正简练的编程风格。有许多地方可以使用 lambda 表达式，但还有很多要探究的地方。对于 C++而言，这是某种程度上的函数式编程，它是一种仍在探索的编程范式。这些对 Lambda 库的介绍可以推动你继续对它的探究。公平地说，这种语法与"真正"的函数式编程语言相比，显得有点笨拙，而且对于新的用户来说也需要一点时间来习惯它。但 是，这对于使用本库的任何 C++程序员都有巨大的价值！我希望它也能成为你喜欢的库。

非常感谢 Jaakko Järvi 和 Gary Powell, 他们是本库的作者，并且是 C++函数式编程的真正先驱！