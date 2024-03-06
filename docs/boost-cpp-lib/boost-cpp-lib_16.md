# 第十五章 错误处理

### 目录

*   15.1 概述
*   15.2 Boost.System
*   15.3 Boost.Exception

![](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 该书采用 [Creative Commons License](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 授权

## 15.1\. 概述

在执行时会有潜在失败可能的每个函数都需要一种合适的方式和它的调用者进行交互。 在 C++中，这一步是通过返回值或抛出一个异常来完成的。 作为常识，返回值经常用在处理非错误的异常中。 调用者通过返回值作出相应的反馈。

异常被通常用来标示出未预期的异常情况。 一个很好的例子是在错误的使用 `new` 时将抛出的一个动态内存分配异常类型 `std::bad_alloc` 。 由于内存的分配通常不会出现任何问题，如果总是检查返回值将会变得异常累赘。

本章介绍了两种可以帮助开发者利用错误处理的 Boost C++库：其中 Boost.System 可以由特定操作系统平台的错误代码转换出跨平台的错误代码。 借助于 Boost.System，函数基于某个特定操作系统的返回值类型可以被转换成为跨平台的类型。 另外，Boost.Exception 允许给任何异常添加额外的信息，以便利用 `catch` 相应的处理程序更好的对异常作出反应。

## 15.2\. Boost.System

[Boost.System](http://www.boost.org/libs/system/) 是一个定义了四个类的小型库，用以识别错误。 `boost::system::error_code` 是一个最基本的类，用于代表某个特定操作系统的异常。 由于操作系统通常枚举异常，`boost::system::error_code` 中以变量的形式保存错误代码 `int`。 下面的例子说明了如何通过访问 Boost.Asio 类来使用这个类。

```cpp
#include <boost/system/error_code.hpp> 
#include <boost/asio.hpp> 
#include <iostream> 
#include <string> 

int main() 
{ 
  boost::system::error_code ec; 
  std::string hostname = boost::asio::ip::host_name(ec); 
  std::cout << ec.value() << std::endl; 
} 
```

*   下载源代码

Boost.Asio 提供了独立的函数 `boost::asio::ip::host_name()` 可以返回正在执行的应用程序名。

`boost::system::error_code` 类型的一个对象可以作为单独的参数传递给 `boost::asio::ip::host_name()`。 如果当前的操作系统函数失败， 这个参数包含相关的错误代码。 也可以通过调用 `boost::asio::ip::host_name()` 而不使用任何参数，以防止错误代码是非相关的。

事实上在 Boost 1.36.0 中 `boost::asio::ip::host_name()` 是有问题的，然而它可以当作一个很好的例子。 即使当前操作系统函数成功返回了计算机名，这个函数它也可能返回一个错误代码。 由于在 Boost 1.37.0 中解决了这个问题，现在可以放心使用 `boost::asio::ip::host_name()` 了。

由于错误代码仅仅是一个数值，因此可以借助于 `value()` 方法得到它。 由于错误代码 0 通常意味着没有错误，其他的值的意义则依赖于操作系统并且需要查看相关手册。

如果使用 Boost 1.36.0， 并且用 Visual Studio 2008 在 Windows XP 环境下编译以上应用程序将不断产生错误代码 14（没有足够的存储空间以完成操作）。 即使函数 `boost::asio::ip::host_name()` 成功决定了计算机名，也会报出错误代码 14。 事实上这是因为函数 `boost::asio::ip::host_name()` 的实现有问题。

除了 `value()` 方法之外, 类 `boost::system::error_code` 提供了方法 `category()`。 这个方法可返回一个在 Boost.System 中定义的二级对象: `boost::system::category`。

错误代码是简单的数值。 操作系统开发商，例如微软，可以保证系统错误代码的特异性。 对于任何开发商来说，在所有现有应用程序中保持错误代码的独一无二是几乎不可能的。 他需要一个包含有所有软件开发者的错误代码中心数据库，以防止在不同的方案下重复使用相同的代码。 当然这是不实际的。 这是错误分类表存在的缘由。

类型 `boost::system::error_code` 的错误代码总是属于可以使用 `category()` 方法获取的分类。 通过预定义的对象 `boost::system::system_category` 来表示操作系统的错误。

通过调用 `category()` 方法，可以返回预定义变量 `boost::system::system_category` 的一个引用。 它允许获取关于分类的特定信息。 例如在使用的是 system 分类的情况下，通过使用 `name()` 方法将得到它的名字 `system`。

```cpp
#include <boost/system/error_code.hpp> 
#include <boost/asio.hpp> 
#include <iostream> 
#include <string> 

int main() 
{ 
  boost::system::error_code ec; 
  std::string hostname = boost::asio::ip::host_name(ec); 
  std::cout << ec.value() << std::endl; 
  std::cout << ec.category().name() << std::endl; 
} 
```

*   下载源代码

通过错误代码和错误分类识别出的错误是独一无二的。 由于仅仅在错误分类中的错误代码是必须唯一的，程序员应当在希望定义某个特定应用程序的错误代码时创建一个新的分类。 这使得任何错误代码都不会影响到其他开发者的错误代码。

```cpp
#include <boost/system/error_code.hpp> 
#include <iostream> 
#include <string> 

class application_category : 
  public boost::system::error_category 
{ 
public: 
  const char *name() const { return "application"; } 
  std::string message(int ev) const { return "error message"; } 
}; 

application_category cat; 

int main() 
{ 
  boost::system::error_code ec(14, cat); 
  std::cout << ec.value() << std::endl; 
  std::cout << ec.category().name() << std::endl; 
} 
```

*   下载源代码

通过创建一个派生于 `boost::system::error_category` 的类以及实现作为新分类的所必须的接口的不同方法可以定义一个新的错误分类。 由于方法 `name()` 和 `message()` 在类 `boost::system::error_category` 中被定义为纯虚拟函数，所以它们是必须提供的。 至于额外的方法，在必要的条件下，可以重载相对应的默认行为。

当方法 `name()` 返回错误分类名时，可以使用方法 `message()` 来获取针对某个错误代码的描述。 不像之前的那个例子，参数 `ev` 往往被用于返回基于错误代码的描述。

新创建的错误分类的对象可以被用来初始化相应的错误代码。 本例中定义了一个用于新分类 `application_category` 的错误代码 `ec` 。 然而错误代码 14 不再是系统错误；他的意义被开发者指定为新的错误分类。

`boost::system::error_code` 包含了一个叫作 `default_error_condition()` 的方法，它可以返回 `boost::system::error_condition`类型的对象。 `boost::system::error_condition` 的接口几乎与 `boost::system::error_code` 相同。 唯一的差别是只有类 `boost::system::error_code` 提供了方法 `default_error_condition()` 。

```cpp
#include <boost/system/error_code.hpp> 
#include <boost/asio.hpp> 
#include <iostream> 
#include <string> 

int main() 
{ 
  boost::system::error_code ec; 
  std::string hostname = boost::asio::ip::host_name(ec); 
  boost::system::error_condition ecnd = ec.default_error_condition(); 
  std::cout << ecnd.value() << std::endl; 
  std::cout << ecnd.category().name() << std::endl; 
} 
```

*   下载源代码

`boost::system::error_condition` 的使用方法与 `boost::system::error_code` 类似。 对象`boost::system::error_condition` 的 `value()` 和 `category()` 方法都可以像上面的例子中那样调用。

有或多或少两个相同的类的原因很简单：当类 `boost::system::error_code` 被当作当前平台的错误代码时， 类 `boost::system::error_condition` 可以被用作获取跨平台的错误代码。 通过调用 `default_error_condition()` 方法，可以把依赖于某个平台的的错误代码转换成 `boost::system::error_condition` 类型的跨平台的错误代码。

如果执行以上应用程序，它将显示数字 12 以及错误分类 `GENERIC`。 依赖于平台的错误代码 14 被转换成了跨平台的错误代码 12。 借助于 `boost::system::error_condition` ，可以总是使用相同的数字表示错误，无视当前操作系统。 当 Windows 报出错误 14 时，其他操作系统可能会对相同的错误报出错误代码 25。 使用 `boost::system::error_condition` ，总是对这个错误报出错误代码 12。

最后 Boost.System 提供了类 `boost::system::system_error` ，它派生于 `std::runtime_error`。 它可被用来传送发生在异常里类型为 `boost::system::error_code` 的错误代码。

```cpp
#include <boost/asio.hpp> 
#include <boost/system/system_error.hpp> 
#include <iostream> 

int main() 
{ 
  try 
  { 
    std::cout << boost::asio::ip::host_name() << std::endl; 
  } 
  catch (boost::system::system_error &e) 
  { 
    boost::system::error_code ec = e.code(); 
    std::cerr << ec.value() << std::endl; 
    std::cerr << ec.category().name() << std::endl; 
  } 
} 
```

*   下载源代码

独立的函数 `boost::asio::ip::host_name()` 是以两种方式提供的：一种是需要类型为 `boost::system::error_code` 的参数，另一种不需要参数。 第二个版本将在错误发生时抛出 `boost::system::system_error` 类型的异常。 异常传出类型为 `boost::system::error_code` 的相应错误代码。

## 15.3\. Boost.Exception

[Boost.Exception](http://www.boost.org/libs/exception/) 库提供了一个新的异常类 `boost::exception` 允许给一个抛出的异常添加信息。 它被定义在文件 `boost/exception/exception.hpp` 中。 由于 Boost.Exception 中的类和函数分布在不同的头文件中， 下面的例子中将使用 `boost/exception/all.hpp` 以避免一个一个添加头文件。

```cpp
#include <boost/exception/all.hpp> 
#include <boost/lexical_cast.hpp> 
#include <boost/shared_array.hpp> 
#include <exception> 
#include <string> 
#include <iostream> 

typedef boost::error_info<struct tag_errmsg, std::string> errmsg_info; 

class allocation_failed : 
  public boost::exception, 
  public std::exception 
{ 
public: 
  allocation_failed(std::size_t size) 
    : what_("allocation of " + boost::lexical_cast<std::string>(size) + " bytes failed") 
  { 
  } 

  virtual const char *what() const throw() 
  { 
    return what_.c_str(); 
  } 

private: 
  std::string what_; 
}; 

boost::shared_array<char> allocate(std::size_t size) 
{ 
  if (size > 1000) 
    throw allocation_failed(size); 
  return boost::shared_array<char>(new char[size]); 
} 

void save_configuration_data() 
{ 
  try 
  { 
    boost::shared_array<char> a = allocate(2000); 
    // saving configuration data ... 
  } 
  catch (boost::exception &e) 
  { 
    e << errmsg_info("saving configuration data failed"); 
    throw; 
  } 
} 

int main() 
{ 
  try 
  { 
    save_configuration_data(); 
  } 
  catch (boost::exception &e) 
  { 
    std::cerr << boost::diagnostic_information(e); 
  } 
} 
```

*   下载源代码

这个例子在 `main()` 中调用了一个函数 `save_configuration_data()` ，它调回了 `allocate()` 。 `allocate()` 函数动态分配内存，而它检查是否超过某个限度。 这个限度在本例中被设定为 1,000 个字节。

如果 `allocate()` 被调用的值大于 1,000，将会抛出 `save_configuration_data()` 函数里的相应异常。 正如注释中所标识的那样，这个函数把配置数据被存储在动态分配的内存中。

事实上，这个例子的目的是通过抛出异常以示范 Boost.Exception。 这个通过 `allocate()` 抛出的异常是 `allocation_failed` 类型的，而且它同时继承了 `boost::exception` 和 `std::exception`。

当然，也不是一定要派生于 `std::exception` 异常的。 为了把它嵌入到现有的框架中，异常 `allocation_failed` 可以派生于其他类的层次结构。 当通过 C++标准来定义以上例子的类层次结构的时候， 单独从 `boost::exception` 中派生出 `allocation_failed` 就足够了。

当抛出 `allocation_failed` 类型的异常的时候，分配内存的大小是存储在异常中的，以缓解相应应用程序的调试。 如果想通过 `allocate()` 分配获取更多的内存空间，那么可以很容易发现导致异常的根本原因。

如果仅仅通过一个函数(例子中的函数 `save_configuration_data()`)来调用 `allocate()` ，这个信息足以找到问题的所在。 然而，在有许多函数调用 `allocate()` 以动态分配内存的更加复杂的应用程序中，这个信息不足以高效的调试应用程序。 在这些情况下，它最好能有助于找到哪个函数试图分配 `allocate()` 所能提供空间之外的内存。 向异常中添加更多的信息，在这些情况下，将非常有助于进程的调试。

有挑战性的是，函数 `allocate()` 中并没有调用者名等信息，以把它加入到相关的异常中。

Boost.Exception 提供了如下的解决方案：对于任何一个可以添加到异常中的信息，可以通过定义一个派生于 `boost::error_info` 的数据类型，来随时向这个异常添加信息。

`boost::error_info` 是一个需要两个参数的模板，第一个参数叫做标签(tag)，特定用来识别新建的数据类型。 通常是一个有特定名字的结构体。 第二个参数是与存储于异常中的数据类型信息相关的。

这个应用程序定义了一个新的数据类型 `errmsg_info`，可以通过 `tag_errmsg` 结构来特异性的识别，它存储着一个 `std::string` 类型的字符串。

在 `save_configuration_data()` 的 `catch` 句柄中，通过获取 `tag_errmsg` 以创建一个对象，它通过字符串 "saving configuration data failed" 进行初始化，以便通过 `operator&lt;&lt;()` 操作符向异常 `boost::exception` 中加入更多信息。 然后这个异常被相应的重新抛出。

现在，这个异常不仅包含有需要动态分配的内存大小，而且对于错误的描述被填入到 `save_configuration_data()` 函数中。 在调试时，这个描述显然很有帮助，因为可以很容易明白哪个函数试图分配更多的内存。

为了从一个异常中获取所有可用信息，可以像例子中那样在 `main()` 的 `catch` 句柄中使用函数 `boost::diagnostic_information()` 。 对于每个异常，函数 `boost::diagnostic_information()` 不仅调用 `what()` 而且获取所有附加信息存储到异常中。 返回一个可以在标准输出中写入的 `std::string` 字符串。

以上程序通过 Visual C++ 2008 编译会显示如下的信息：

```cpp
Throw in function (unknown)
Dynamic exception type: class allocation_failed
std::exception::what: allocation of 2000 bytes failed
[struct tag_errmsg *] = saving configuration data failed 
```

正如我们所看见的，数据包含了异常的数据类型，通过 `what()` 方法获取到错误信息，以及包括相应结构体名的描述。

`boost::diagnostic_information()` 函数在运行时检查一个给定的异常是否派生于 `std::exception`。 只会在派生于 `std::exception` 的条件下调用 `what()` 方法。

抛出异常类型 `allocation_failed` 的函数名会被指定为"unknown"(未知)信息。

Boost.Exception 提供了一个用以抛出异常的宏，它包含了函数名，以及如文件名、行数的附加信息。

```cpp
#include <boost/exception/all.hpp> 
#include <boost/lexical_cast.hpp> 
#include <boost/shared_array.hpp> 
#include <exception> 
#include <string> 
#include <iostream> 

typedef boost::error_info<struct tag_errmsg, std::string> errmsg_info; 

class allocation_failed : 
  public std::exception 
{ 
public: 
  allocation_failed(std::size_t size) 
    : what_("allocation of " + boost::lexical_cast<std::string>(size) + " bytes failed") 
  { 
  } 

  virtual const char *what() const throw() 
  { 
    return what_.c_str(); 
  } 

private: 
  std::string what_; 
}; 

boost::shared_array<char> allocate(std::size_t size) 
{ 
  if (size > 1000) 
    BOOST_THROW_EXCEPTION(allocation_failed(size)); 
  return boost::shared_array<char>(new char[size]); 
} 

void save_configuration_data() 
{ 
  try 
  { 
    boost::shared_array<char> a = allocate(2000); 
    // saving configuration data ... 
  } 
  catch (boost::exception &e) 
  { 
    e << errmsg_info("saving configuration data failed"); 
    throw; 
  } 
} 

int main() 
{ 
  try 
  { 
    save_configuration_data(); 
  } 
  catch (boost::exception &e) 
  { 
    std::cerr << boost::diagnostic_information(e); 
  } 
} 
```

*   下载源代码

通过使用宏 `BOOST_THROW_EXCEPTION` 替代 `throw`， 如函数名、文件名、行数之类的附加信息将自动被添加到异常中。但这仅仅在编译器支持宏的情况下有效。 当通过 C++标准定义 `__FILE__` 和 `__LINE__` 之类的宏时，没有用于返回当前函数名的标准化的宏。 由于许多编译器制造商提供这样的宏， `BOOST_THROW_EXCEPTION` 试图识别当前编译器，从而利用相对应的宏。 使用 Visual C++ 2008 编译时，以上应用程序显示以下信息：

```cpp
.\main.cpp(31): Throw in function class boost::shared_array<char> __cdecl allocate(unsigned int)
Dynamic exception type: class boost::exception_detail::clone_impl<struct boost::exception_detail::error_info_injector<class allocation_failed> >
std::exception::what: allocation of 2000 bytes failed
[struct tag_errmsg *] = saving configuration data failed 
```

即使 `allocation_failed` 类不再派生于 `boost::exception` 代码的编译也不会产生错误。 `BOOST_THROW_EXCEPTION` 获取到一个能够动态识别是否派生于 `boost::exception` 的函数 `boost::enable_error_info()`。 如果不是，他将自动建立一个派生于特定类和 `boost::exception` 的新异常类型。 这个机制使得以上信息中不仅仅显示内存分配异常 `allocation_failed` 。

最后，这个部分包含了一个例子，它选择性的获取了添加到异常中的信息。

```cpp
#include <boost/exception/all.hpp> 
#include <boost/lexical_cast.hpp> 
#include <boost/shared_array.hpp> 
#include <exception> 
#include <string> 
#include <iostream> 

typedef boost::error_info<struct tag_errmsg, std::string> errmsg_info; 

class allocation_failed : 
  public std::exception 
{ 
public: 
  allocation_failed(std::size_t size) 
    : what_("allocation of " + boost::lexical_cast<std::string>(size) + " bytes failed") 
  { 
  } 

  virtual const char *what() const throw() 
  { 
    return what_.c_str(); 
  } 

private: 
  std::string what_; 
}; 

boost::shared_array<char> allocate(std::size_t size) 
{ 
  if (size > 1000) 
    BOOST_THROW_EXCEPTION(allocation_failed(size)); 
  return boost::shared_array<char>(new char[size]); 
} 

void save_configuration_data() 
{ 
  try 
  { 
    boost::shared_array<char> a = allocate(2000); 
    // saving configuration data ... 
  } 
  catch (boost::exception &e) 
  { 
    e << errmsg_info("saving configuration data failed"); 
    throw; 
  } 
} 

int main() 
{ 
  try 
  { 
    save_configuration_data(); 
  } 
  catch (boost::exception &e) 
  { 
    std::cerr << *boost::get_error_info<errmsg_info>(e); 
  } 
} 
```

*   下载源代码

这个例子并没有使用函数 `boost::diagnostic_information()` 而是使用 `boost::get_error_info()` 函数来直接获取错误信息的类型 `errmsg_info`。 函数 `boost::get_error_info()` 用于返回 `boost::shared_ptr` 类型的智能指针。 如果传递的参数不是 `boost::exception` 类型的，返回的值将是相应的空指针。 如果 `BOOST_THROW_EXCEPTION` 宏总是被用来抛出异常，派生于 `boost::exception` 的异常是可以得到保障的——在这些情况下没有必要去检查返回的智能指针是否为空。