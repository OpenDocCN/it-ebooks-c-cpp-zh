# 第十四章 数据结构

# 第十四章 数据结构

### 目录

*   14.1 概述
*   14.2 元组
*   14.3 Boost.Any
*   14.4 Boost.Variant
*   14.5 练习

![](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 该书采用 [Creative Commons License](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 授权

## 14.1\. 概述

在 Boost C++ 库中， 把一些类型定义为 container 显得不太合适， 所以就并没有放在 第十三章 *容器* 里。 而把他们放在本章就比较合适了。 举例来说， `boost::tuple` 就扩展了 C++ 的数据类型 `std::pair` 用以储存多个而不只是两个值。

除了 `boost::tuple`， 这一章还涵盖了类 `boost::any` 和 `boost::variant` 以储存那些不确定类型的值。 其中 `boost::any` 类型的变量使用起来就像弱类型语言中的变量一样灵活。 另一方面， `boost::variant` 类型的变量可以储存一些预定义的数据类型， 就像我们用 `union` 时候一样。

## 14.2\. 元组

[Boost.Tuple](http://www.boost.org/libs/tuple/) 库提供了一个更一般的版本的 `std::pair` —— `boost::tuple` 。 不过 `std::pair` 只能储存两个值而已, `boost::tuple` 则给了我们更多的选择。

```cpp
#include <boost/tuple/tuple.hpp> 
#include <boost/tuple/tuple_io.hpp> 
#include <string> 
#include <iostream> 

int main() 
{ 
  typedef boost::tuple<std::string, std::string> person; 
  person p("Boris", "Schaeling"); 
  std::cout << p << std::endl; 
} 
```

*   下载源代码

为了使用 `boost::tuple`， 你必须要包含头文件： `boost/tuple/tuple.hpp` 。 若想要让元组和流一起使用， 你还需要包含头文件： `boost/tuple/tuple_io.hpp` 才行。

其实， `boost::tuple` 的用法基本上和 `std::pair` 一样。 就像我们在上面的例子里看到的那样， 两个值类型的 `std::string` 通过两个相应的模板参数存储在了元组里。

当然 `person` 类型也可以用 `std::pair` 来实现。 所有 `boost::tuple` 类型的对象都可以被写入流里。 再次强调， 为了使用流操作和各种流操作运算符， 你必须要包含头文件： `boost/tuple/tuple_io.hpp` 。 显然，我们的例子会输出： `(Boris Schaeling)` 。

`boost::tuple` 和 `std::pair` 之间最重要的一点不同点： 元组可以存储无限多个值！

```cpp
#include <boost/tuple/tuple.hpp> 
#include <boost/tuple/tuple_io.hpp> 
#include <string> 
#include <iostream> 

int main() 
{ 
  typedef boost::tuple<std::string, std::string, int> person; 
  person p("Boris", "Schaeling", 43); 
  std::cout << p << std::endl; 
} 
```

*   下载源代码

我们修改了实例， 现在的元组里不仅储存了一个人的 firstname 和 lastname， 还加上了他的鞋子的尺码。 现在， 我们的例子将会输出： `(Boris Schaeling 43)` 。

就像 `std::pair` 有辅助函数 `std::make_pair()` 一样， 一个元组也可以用它的辅助函数 `boost::make_tuple()` 来创建。

```cpp
#include <boost/tuple/tuple.hpp> 
#include <boost/tuple/tuple_io.hpp> 
#include <iostream> 

int main() 
{ 
  std::cout << boost::make_tuple("Boris", "Schaeling", 43) << std::endl; 
} 
```

*   下载源代码

就像下面的例子所演示的那样， 一个元组也可以存储引用类型的值。

```cpp
#include <boost/tuple/tuple.hpp> 
#include <boost/tuple/tuple_io.hpp> 
#include <string> 
#include <iostream> 

int main() 
{ 
  std::string s = "Boris"; 
  std::cout << boost::make_tuple(boost::ref(s), "Schaeling", 43) << std::endl; 
} 
```

*   下载源代码

因为 "Schaeling" 和 43 是按值传递的，所以就直接存储在了元组中。 与他们不同的是： person 的第一个元素是一个指向 `s` 的引用。 Boost.Ref 中的 `boost::ref()` 就是用来创建这样的引用的。 相对的， 要创建一个常量的引用的时候， 你需要使用 `boost::cref()` 。

在学习了创建元组的方法之后， 让我们来了解一下访问元组中元素的方式。 `std::pair` 只包含两个元素， 故可以使用属性 `first` 和 `second` 来访问其中的元素。 但元组可以包含无限多个元素， 显然， 我们需要用另一种方式来解决访问的问题。

```cpp
#include <boost/tuple/tuple.hpp> 
#include <string> 
#include <iostream> 

int main() 
{ 
  typedef boost::tuple<std::string, std::string, int> person; 
  person p = boost::make_tuple("Boris", "Schaeling", 43); 
  std::cout << p.get<0>() << std::endl; 
  std::cout << boost::get<0>(p) << std::endl; 
} 
```

*   下载源代码

我们可以用两种方式来访问元组中的元素： 使用成员函数 `get()` ， 或者将元组传给一个独立的函数 `boost::get()` 。 使用这两种方式时， 元素的索引值都是通过模板参数来指定的。 例子中就分别使用了这两种方式来访问 `p` 中的第一个元素。 因此， `Boris` 会被输出两次。

另外， 对于索引值合法性的检查会在编译期执行， 故访问非法的索引值会引起编译期错误而不是运行时的错误。

对于元组中元素的修改， 你同样可以使用 `get()` 和 `boost::get()` 函数。

```cpp
#include <boost/tuple/tuple.hpp> 
#include <boost/tuple/tuple_io.hpp> 
#include <string> 
#include <iostream> 

int main() 
{ 
  typedef boost::tuple<std::string, std::string, int> person; 
  person p = boost::make_tuple("Boris", "Schaeling", 43); 
  p.get<1>() = "Becker"; 
  std::cout << p << std::endl; 
} 
```

*   下载源代码

`get()` 和 `boost::get()` 都会返回一个引用值。 例子中修改了 lastname 之后将会输出： `(Boris Becker 43)` 。

Boost.Tuple 除了重载了流操作运算符以外， 还为我们提供了比较运算符。 为了使用它们， 你必须要包含相应的头文件： `boost/tuple/tuple_comparison.hpp` 。

```cpp
#include <boost/tuple/tuple.hpp> 
#include <boost/tuple/tuple_comparison.hpp> 
#include <string> 
#include <iostream> 

int main() 
{ 
  typedef boost::tuple<std::string, std::string, int> person; 
  person p1 = boost::make_tuple("Boris", "Schaeling", 43); 
  person p2 = boost::make_tuple("Boris", "Becker", 43); 
  std::cout << (p1 != p2) << std::endl; 
} 
```

*   下载源代码

上面的例子将会输出 `1` 因为两个元组 `p1` 和 `p2` 是不同的。

同时， 头文件 `boost/tuple/tuple_comparison.hpp` 还定义了一些其他的比较操作， 比如用来做字典序比较的大于操作等。

Boost.Tuple 还提供了一种叫做 Tier 的特殊元组。 Tier 的特殊之处在于它包含的所有元素都是引用类型的。 它可以通过构造函数 `boost::tie()` 来创建。

```cpp
#include <boost/tuple/tuple.hpp> 
#include <boost/tuple/tuple_io.hpp> 
#include <string> 
#include <iostream> 

int main() 
{ 
  typedef boost::tuple<std::string&, std::string&, int&> person; 

  std::string firstname = "Boris"; 
  std::string surname = "Schaeling"; 
  int shoesize = 43; 
  person p = boost::tie(firstname, surname, shoesize); 
  surname = "Becker"; 
  std::cout << p << std::endl; 
} 
```

*   下载源代码

上面的例子创建了一个 tier `p`， 他包含了三个分别指向 `firstname`， `surname` 和 `shoesize` 的引用值。 在修改变量 `surname` 的同时， tier 也会跟着改变。

就像下面的例子展示的那样，你当然可以用 `boost::make_tuple()` 和 `boost::ref()` 来代替构造函数 `boost::tie()` 。

```cpp
#include <boost/tuple/tuple.hpp> 
#include <boost/tuple/tuple_io.hpp> 
#include <string> 
#include <iostream> 

int main() 
{ 
  typedef boost::tuple<std::string&, std::string&, int&> person; 

  std::string firstname = "Boris"; 
  std::string surname = "Schaeling"; 
  int shoesize = 43; 
  person p = boost::make_tuple(boost::ref(firstname), boost::ref(surname), boost::ref(shoesize)); 
  surname = "Becker"; 
  std::cout << p << std::endl; 
} 
```

*   下载源代码

`boost::tie()` 在一定程度上简化了语法， 同时， 也可以用作“拆箱”元组。 在接下来的这个例子里， 元组中的各个元素就被很方便的“拆箱”并直接赋给了其他变量。

```cpp
#include <boost/tuple/tuple.hpp> 
#include <string> 
#include <iostream> 

boost::tuple<std::string, int> func() 
{ 
  return boost::make_tuple("Error message", 2009); 
}

int main() 
{ 
  std::string errmsg; 
  int errcode; 

  boost::tie(errmsg, errcode) = func(); 
  std::cout << errmsg << ": " << errcode << std::endl; 
} 
```

*   下载源代码

通过使用 `boost::tie()` ， 元组中的元素：字符串“Error massage”和错误代码“2009”就很方便地经 `func()` 的返回值直接赋给了 `errmsg` 和 `errcode` 。

## 14.3\. Boost.Any

像 C++ 这样的强类型语言要求给每个变量一个确定的类型。 而以 JavaScript 为代表的弱类型语言却不这样做， 弱类型的每个变量都可以存储数组、 布尔值、 或者是字符串。

库 [Boost.Any](http://www.boost.org/libs/any/) 给我们提供了 `boost::any` 类， 让我们可以在 C++ 中像 JavaScript 一样的使用弱类型的变量。

```cpp
#include <boost/any.hpp> 

int main() 
{ 
  boost::any a = 1; 
  a = 3.14; 
  a = true; 
} 
```

*   下载源代码

为了使用 `boost::any`， 你必须要包含头文件： `boost/any.hpp`。 接下来， 你就可以定义和使用 `boost::any` 的对象了。

需要注明的是： `boost::any` 并不能真的存储任意类型的值； Boost.Any 需要一些特定的前提条件才能工作。 任何想要存储在 `boost::any` 中的值，都必须是可拷贝构造的。 因此，想要在 `boost::any` 存储一个字符串类型的值， 就必须要用到 `std::string` ， 就像在下面那个例子中做的一样。

```cpp
#include <boost/any.hpp> 
#include <string> 

int main() 
{ 
  boost::any a = 1; 
  a = 3.14; 
  a = true; 
  a = std::string("Hello, world!"); 
} 
```

*   下载源代码

如果你企图把字符串 "Hello, world!" 直接赋给 `a` ， 你的编译器就会报错， 因为由基类型 `char` 构成的字符串在 C++ 中并不是可拷贝构造的。

想要访问 `boost::any` 中具体的内容， 你必须要使用转型操作： `boost::any_cast` 。

```cpp
#include <boost/any.hpp> 
#include <iostream> 

int main() 
{ 
  boost::any a = 1; 
  std::cout << boost::any_cast<int>(a) << std::endl; 
  a = 3.14; 
  std::cout << boost::any_cast<double>(a) << std::endl; 
  a = true; 
  std::cout << boost::any_cast<bool>(a) << std::endl; 
} 
```

*   下载源代码

通过由模板参数传入 `boost::any_cast` 的值， 变量会被转化成相应的类型。 一旦你指定了一种非法的类型， 该操作会抛出 `boost::bad_any_cast` 类型的异常。

```cpp
#include <boost/any.hpp> 
#include <iostream> 

int main() 
{ 
  try 
  { 
    boost::any a = 1; 
    std::cout << boost::any_cast<float>(a) << std::endl; 
  } 
  catch (boost::bad_any_cast &e) 
  { 
    std::cerr << e.what() << std::endl; 
  } 
} 
```

*   下载源代码

上面的例子就抛出了一个异常， 因为 `float` 并不能匹配原本存储在 `a` 中的 `int` 类型。 记住， 在任何情况下都保证 `boost::any` 中的类型匹配是很重要的。 在没有通过模板参数指定 `short` 或 `long` 类型时， 同样会有异常抛出。

既然 `boost::bad_any_cast` 继承自 `std::bad_cast`， `catch` 当然也可以捕获相应类型的异常。

想要检查 `boost::any` 是否为空， 你可以使用 `empty()` 函数。 想要确定其中具体的类型信息， 你可以使用 `type()` 函数。

```cpp
#include <boost/any.hpp> 
#include <typeinfo> 
#include <iostream> 

int main() 
{ 
  boost::any a = 1; 
  if (!a.empty()) 
  { 
    const std::type_info &ti = a.type(); 
    std::cout << ti.name() << std::endl; 
  } 
} 
```

*   下载源代码

上面的例子同时用到了 `empty()` 和 `type()` 函数。 `empty()` 将会返回一个布尔值， 而 `type()` 则会返回一个在 `typeinfo` 中定义的 `std::type_info` 值。

作为对这一节的总结， 最后一个例子会向你展示怎样用 `boost::any_cast` 来定义一个指向 `boost::any` 中内容的指针。

```cpp
#include <boost/any.hpp> 
#include <iostream> 

int main() 
{ 
  boost::any a = 1; 
  int *i = boost::any_cast<int>(&a); 
  std::cout << *i << std::endl; 
} 
```

*   下载源代码

你需要做的就是传递一个 `boost::any` 类型的指针， 作为 `boost::any_cast` 的参数； 模板参数却没有任何改动。

## 14.4\. Boost.Variant

[Boost.Variant](http://www.boost.org/libs/variant/) 和 Boost.Any 之间的不同点在于 Boost.Any 可以被视为任意的类型， 而 Boost.Variant 只能被视为固定数量的类型。 让我们来看下面这个例子。

```cpp
#include <boost/variant.hpp> 

int main() 
{ 
  boost::variant<double, char> v; 
  v = 3.14; 
  v = 'A'; 
} 
```

*   下载源代码

Boost.Variant 为我们提供了一个定义在 `boost/variant.hpp` 中的类： `boost::variant` 。 既然 `boost::variant` 是一个模板， 你必须要指定至少一个参数。 Variant 所存储的数据类型就由这些参数来指定。 上面的例子就给 `v` 指定了 `double` 类型和 `char` 类型。 注意， 一旦你将一个 `int` 值赋给了 `v`， 你的代码将不会编译通过。

当然， 上面的例子也可以用一个 `union` 类型来实现， 但是与 union 不同的是： `boost::variant` 可以储存像 `std::string` 这样的 class 类型的数据。

```cpp
#include <boost/variant.hpp> 
#include <string> 

int main() 
{ 
  boost::variant<double, char, std::string> v; 
  v = 3.14; 
  v = 'A'; 
  v = "Hello, world!"; 
} 
```

*   下载源代码

要访问 `v` 中的数据， 你可以使用独立的 `boost::get()` 函数。

```cpp
#include <boost/variant.hpp> 
#include <string> 
#include <iostream> 

int main() 
{ 
  boost::variant<double, char, std::string> v; 
  v = 3.14; 
  std::cout << boost::get<double>(v) << std::endl; 
  v = 'A'; 
  std::cout << boost::get<char>(v) << std::endl; 
  v = "Hello, world!"; 
  std::cout << boost::get<std::string>(v) << std::endl; 
} 
```

*   下载源代码

`boost::get()` 需要传入一个模板参数来指明你需要返回的数据类型。 若是指定了一个非法的类型， 你会遇到一个运行时而不是编译期的错误。

所有 `boost::variant` 类型的值都可以被直接写入标准输入流这样的流中， 这可以在一定程度上让你避开运行时错误的风险。

```cpp
#include <boost/variant.hpp> 
#include <string> 
#include <iostream> 

int main() 
{ 
  boost::variant<double, char, std::string> v; 
  v = 3.14; 
  std::cout << v << std::endl; 
  v = 'A'; 
  std::cout << v << std::endl; 
  v = "Hello, world!"; 
  std::cout << v << std::endl; 
} 
```

*   下载源代码

想要分别处理各种不同类型的数据， Boost.Variant 为我们提供了一个名为 `boost::apply_visitor()` 的函数。

```cpp
#include <boost/variant.hpp> 
#include <boost/any.hpp> 
#include <vector> 
#include <string> 
#include <iostream> 

std::vector<boost::any> vector; 

struct output : 
  public boost::static_visitor<> 
{ 
  void operator()(double &d) const 
  { 
    vector.push_back(d); 
  } 

  void operator()(char &c) const 
  { 
    vector.push_back(c); 
  } 

  void operator()(std::string &s) const 
  { 
    vector.push_back(s); 
  } 
}; 

int main() 
{ 
  boost::variant<double, char, std::string> v; 
  v = 3.14; 
  boost::apply_visitor(output(), v); 
  v = 'A'; 
  boost::apply_visitor(output(), v); 
  v = "Hello, world!"; 
  boost::apply_visitor(output(), v); 
} 
```

*   下载源代码

`boost::apply_visitor()` 第一个参数需要传入一个继承自 `boost::static_visitor` 类型的对象。 这个类必须要重载 `operator()()` 运算符来处理 `boost::variant` 每个可能的类型。 相应的， 例子中的 `v` 就重载了三次 operator() 来处理三种可能的类型： `double`， `char` 和 `std::string`。

再仔细看代码， 不难发现 `boost::static_visitor` 是一个模板。 那么，当 `operator()()` 有返回值的时候， 就必须返回一个模板才行。 如果 operator() 像例子那样没有返回值时， 你就不需要模板了。

`boost::apply_visitor()` 的第二个参数是一个 `boost::variant` 类型的值。

在使用时， `boost::apply_visitor()` 会自动调用跟第二个参数匹配的 `operator()()` 。 示例程序中的 `boost::apply_visitor()` 就自动调用了三个不同的 operator 第一个是 `double` 类型的， 第二个是 `char` 最后一个是 `std::string`。

`boost::apply_visitor()` 的优点不只是“自动调用匹配的函数”这一点。 更有用的是， `boost::apply_visitor()` 会确认是否 `boost::variant` 中的每个可能值都定义了相应的函数。 如果你忘记重载了任何一个函数， 代码都不会编译通过。

当然， 如果对每种类型的操作都是一样的， 你也可以像下面的示例一样使用一个模板来简化你的代码。

```cpp
#include <boost/variant.hpp> 
#include <boost/any.hpp> 
#include <vector> 
#include <string> 
#include <iostream> 

std::vector<boost::any> vector; 

struct output : 
  public boost::static_visitor<> 
{ 
  template <typename T> 
  void operator()(T &t) const 
  { 
    vector.push_back(t); 
  } 
}; 

int main() 
{ 
  boost::variant<double, char, std::string> v; 
  v = 3.14; 
  boost::apply_visitor(output(), v); 
  v = 'A'; 
  boost::apply_visitor(output(), v); 
  v = "Hello, world!"; 
  boost::apply_visitor(output(), v); 
} 
```

*   下载源代码

既然 `boost::apply_visitor()` 可以在编译期确定代码的正确性， 你就该更多的使用它而不是 `boost::get()`。

## 14.5\. 练习

You can buy [solutions to all exercises](http://en.highscore.de/shop/index.php?p=boost-solution) in this book as a ZIP file.

1.  自定义一种数据类型： `configuration` 它可以存储一个 name-value 对。 Name 为 `std::string` 类型， 而 value 可为 `std::string` 或者 `int` 或者 `float` 类型。 在 `main()` 函数里， 用 `configuration` 存储下列 name-value 对: path=C:\Windows, version=3， pi=3.1415。 通过向便准输出流输出来验证你对数据类型的设计。

2.  在输出后， 将对象中的 path 修改为 C:\Windows\System。 再次向标准输出流输出以验证你的设计。