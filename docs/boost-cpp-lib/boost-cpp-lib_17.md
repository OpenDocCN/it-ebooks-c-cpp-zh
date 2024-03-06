# 第十六章 类型转换操作符

# 第十六章 类型转换操作符

### 目录

*   16.1 概述
*   16.2 Boost.Conversion
*   16.3 Boost.NumericConversion

![](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 该书采用 [Creative Commons License](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 授权

## 16.1\. 概述

C++标准定义了四种类型转换操作符: `static_cast`, `dynamic_cast`, `const_cast` 和 `reinterpret_cast`。 Boost.Conversion 和 Boost.NumericConversion 这两个库特别为某些类型转换定义了额外的类型转换操作符。

## 16.2\. Boost.Conversion

[Boost.Conversion](http://www.boost.org/libs/conversion/) 库由两个文件组成。分别在 `boost/cast.hpp` 文件中定义了 `boost::polymorphic_cast` 和 `boost::polymorphic_downcast` 这两个类型转换操作符, 在 `boost/lexical_cast.hpp` 文件中定义了 `boost::lexical_cast`。

`boost::polymorphic_cast` 和 `boost::polymorphic_downcast` 是为了使原来用 `dynamic_cast` 实现的类型转换更加具体。具体细节，如下例所示。

```cpp
struct father 
{ 
  virtual ~father() { }; 
}; 

struct mother 
{ 
  virtual ~mother() { }; 
}; 

struct child : 
  public father, 
  public mother 
{ 
}; 

void func(father *f) 
{ 
  child *c = dynamic_cast<child*>(f); 
} 

int main() 
{ 
  child *c = new child; 
  func(c); 

  father *f = new child; 
  mother *m = dynamic_cast<mother*>(f); 
} 
```

*   下载源代码

本例使用 `dynamic_cast` 类型转换操作符两次: 在 `func()` 函数中，它将指向父类的指针转换为指向子类的指针。在 `main()` 中, 它将一个指向父类的指针转为指向另一个父类的指针。第一个转换称为向下转换(downcast)，第二个转换称为交叉转换(cross cast)。

通过使用 Boost.Conversion 的类型转换操作符，可以将向下转换和交叉转换区分开来。

```cpp
#include <boost/cast.hpp> 

struct father 
{ 
  virtual ~father() { }; 
}; 

struct mother 
{ 
  virtual ~mother() { }; 
}; 

struct child : 
  public father, 
  public mother 
{ 
}; 

void func(father *f) 
{ 
  child *c = boost::polymorphic_downcast<child*>(f); 
} 

int main() 
{ 
  child *c = new child; 
  func(c); 

  father *f = new child; 
  mother *m = boost::polymorphic_cast<mother*>(f); 
} 
```

*   下载源代码

`boost::polymorphic_downcast` 类型转换操作符只能用于向下转换。 它内部使用 `static_cast` 实现类型转换。 由于 `static_cast` 并不动态检查类型转换是否合法，所以 `boost::polymorphic_downcast` 应该只在类型转换是安全的情况下使用。 在调试(debug builds)模式下, `boost::polymorphic_downcast` 实际上在 `assert ()`函数中使用 `dynamic_cast` 验证类型转换是否合法。 请注意这种合法性检测只在定义了`NDEBUG`宏的情况下执行，这通常是在调试模式下。

向下转换最好使用 `boost::polymorphic_downcast`, 那么 `boost::polymorphic_cast` 就是交叉转换所需要的了。 由于 `dynamic_cast` 是唯一能实现交叉转换的类型转换操作符，`boost::polymorphic_cast` 内部使用了它。 由于 `boost::polymorphic_cast` 能够在错误的时候抛出 `std::bad_cast` 类型的异常，所以优先使用这个类型转换操作符还是很有必要的。相反，`dynamic_cast` 在类型转换失败使将返回 0。 避免手工验证返回值，`boost::polymorphic_cast` 提供了自动化的替代方式。

`boost::polymorphic_downcast` 和 `boost::polymorphic_cast` 只在指针必须转换的时候使用；否则，必须使用 `dynamic_cast` 执行转换。 由于 `boost::polymorphic_downcast` 是基于 `static_cast`，所以它不能够，比如说，将父类对象转换为子类对象。 如果转换的类型不是指针，则使用 `boost::polymorphic_cast` 执行类型转换也没有什么意义，而在这种情况下使用 `dynamic_cast` 还会抛出一个 `std::bad_cast` 异常。

虽然所有的类型转换都可用 `dynamic_cast` 实现，可 `boost::polymorphic_downcast` 和 `boost::polymorphic_cast` 也不是真正随意使用的。 Boost.Conversion 还提供了另外一种在实践中很有用的类型转换操作符。 体会一下下面的例子。

```cpp
#include <boost/lexical_cast.hpp> 
#include <string> 
#include <iostream> 

int main() 
{ 
  std::string s = boost::lexical_cast<std::string>(169); 
  std::cout << s << std::endl; 
  double d = boost::lexical_cast<double>(s); 
  std::cout << d << std::endl; 
} 
```

*   下载源代码

类型转换操作符 `boost::lexical_cast` 可将数字转换为其他类型。 例子首先将整数 169 转换为字符串，然后将字符串转换为浮点数。

`boost::lexical_cast` 内部使用流(streams)执行转换操作。 因此，只有那些重载了 `operator&lt;&lt;()` 和 `operator&gt;&gt;()` 这两个操作符的类型可以转换。 使用 `boost::lexical_cast` 的优点是类型转换出现在一行代码之内，无需手工操作流(streams)。 由于流的用法对于类型转换不能立刻理解代码含义, 而 `boost::lexical_cast` 类型转换操作符还可以使代码更有意义，更加容易理解。

请注意 `boost::lexical_cast` 并不总是访问流(streams)；它自己也优化了一些数据类型的转换。

如果转换失败，则抛出 `boost::bad_lexical_cast` 类型的异常，它继承自 `std::bad_cast`。

```cpp
#include <boost/lexical_cast.hpp> 
#include <string> 
#include <iostream> 

int main() 
{ 
  try 
  { 
    int i = boost::lexical_cast<int>("abc"); 
    std::cout << i << std::endl; 
  } 
  catch (boost::bad_lexical_cast &e) 
  { 
    std::cerr << e.what() << std::endl; 
  } 
} 
```

*   下载源代码

本例由于字符串 "abc" 不能转换为 `int` 类型的数字而抛出异常。

## 16.3\. Boost.NumericConversion

[Boost.NumericConversion](http://www.boost.org/libs/numeric/conversion/) 可将一种数值类型转换为不同的数值类型。 在 C++里, 这种转换可以隐式地发生，如下面例所示。

```cpp
#include <iostream> 

int main() 
{ 
  int i = 0x10000; 
  short s = i; 
  std::cout << s << std::endl; 
} 
```

*   下载源代码

由于从 `int` 到 `short` 的类型转换自动产生，所以本例编译没有错误。 虽然本例可以运行，但结果由于依赖具体的编译器实现而结果无法预期。 数字`0x10000`对于变量 `i` 来说太大而不能存储在 `short` 类型的变量中。 依据 C++标准，这个操作的结果是实现定义的("implementation defined")。 用 Visual C++ 2008 编译，应用程序显示的是`0`。 `s` 的值当然不同于 `i` 的值。

为避免这种数值转换错误，可以使用 `boost::numeric_cast` 类型转换操作符。

```cpp
#include <boost/numeric/conversion/cast.hpp> 
#include <iostream> 

int main() 
{ 
  try 
  { 
    int i = 0x10000; 
    short s = boost::numeric_cast<short>(i); 
    std::cout << s << std::endl; 
  } 
  catch (boost::numeric::bad_numeric_cast &e) 
  { 
    std::cerr << e.what() << std::endl; 
  } 
} 
```

*   下载源代码

`boost::numeric_cast` 的用法与 C++类型转换操作符非常相似。 当然需要包含正确的头文件；就是 `boost/numeric/conversion/cast.hpp`。

`boost::numeric_cast` 执行与 C++相同的隐式转换操作。 但是，`boost::numeric_cast` 验证了在不改变数值的情况下转换是否能够发生。 前面给的应用例子，转换不能发生，因而由于`0x10000`太大而不能存储在 `short` 类型的变量上，而抛出 `boost::numeric::bad_numeric_cast` 异常。

严格来讲，抛出的是 `boost::numeric::positive_overflow` 类型的异常，这个类型特指所谓的溢出(overflow) - 在此例中是正数。 相应地，还存在着 `boost::numeric::negative_overflow` 类型的异常，它特指负数的溢出。

```cpp
#include <boost/numeric/conversion/cast.hpp> 
#include <iostream> 

int main() 
{ 
  try 
  { 
    int i = -0x10000; 
    short s = boost::numeric_cast<short>(i); 
    std::cout << s << std::endl; 
  } 
  catch (boost::numeric::negative_overflow &e) 
  { 
    std::cerr << e.what() << std::endl; 
  } 
} 
```

*   下载源代码

Boost.NumericConversion 还定义了其他的异常类型，都继承自 `boost::numeric::bad_numeric_cast`。 因为 `boost::numeric::bad_numeric_cast` 继承自 `std::bad_cast`，所以 `catch` 处理也可以捕获这个类型的异常。