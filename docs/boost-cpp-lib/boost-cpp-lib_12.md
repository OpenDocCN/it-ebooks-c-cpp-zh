# 第十一章 序列化

### 目录

*   11.1 概述
*   11.2 归档
*   11.3 指针和引用
*   11.4 对象类层次结构的序列化
*   11.5 优化用封装函数
*   11.6 练习

![](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 该书采用 [Creative Commons License](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 授权

## 11.1\. 概述

Boost C++ 的 [序列化](http://www.boost.org/libs/serialization/) 库允许将 C++ 应用程序中的对象转换为一个字节序列， 此序列可以被保存，并可在将来恢复对象的时候再次加载。 各种不同的数据格式，包括 XML，只要具有一定规则的数据格式，在序列化后都产生一个字节序列。所有 Boost.Serialization 支持的格式，在某些方面来说都是专有的。 比如 XML 格式不同用来和不是用 C++ Boost.Serialization 库开发的应用程序交换数据。所有以 XML 格式存储的数据适合于从之前存储的数据上恢复同一个 C++ 对象。 XML 格式的唯一优点是序列化的 C++ 对象容易理解，这是很有用的，比如说在调试的时候。

## 11.2\. 归档

Boost.Serialization 的主要概念是归档。 归档的文件是相当于序列化的 C++ 对象的一个字节流。 对象可以通过序列化添加到归档文件，相应地也可从归档文件中加载。 为了恢复和之前存储相同的 C++ 对象，需假定数据类型是相同的。

下面看一个简单的例子。

```cpp
#include <boost/archive/text_oarchive.hpp> 
#include <iostream> 

int main() 
{ 
  boost::archive::text_oarchive oa(std::cout); 
  int i = 1; 
  oa << i; 
} 
```

*   下载源代码

Boost.Serialization 提供了多个归档类，如 `boost::archive::text_oarchive` 类，它定义在 `boost/archive/text_oarchive.hpp` 文件中。 `boost::archive::text_oarchive`，可将对象序列化为文本流。 上面的应用程序将 `22 serialization::archive 5 1` 写出到标准输出流。

可见， `boost::archive::text_oarchive` 类型的对象 `oa` 可以用来像流 (stream) 一样通过 `&lt;&lt;` 来序列化对象。 尽管如此，归档也不能被认为是可以存储任何数据的常规的流。 为了以后恢复数据，必须以相同的顺序使用和先前存储时用的一样的数据类型。 下面的例子序列化和恢复了 `int` 类型的变量。

```cpp
#include <boost/archive/text_oarchive.hpp> 
#include <boost/archive/text_iarchive.hpp> 
#include <iostream> 
#include <fstream> 

void save() 
{ 
  std::ofstream file("archiv.txt"); 
  boost::archive::text_oarchive oa(file); 
  int i = 1; 
  oa << i; 
} 

void load() 
{ 
  std::ifstream file("archiv.txt"); 
  boost::archive::text_iarchive ia(file); 
  int i = 0; 
  ia >> i; 
  std::cout << i << std::endl; 
} 

int main() 
{ 
  save(); 
  load(); 
} 
```

*   下载源代码

当 `boost::archive::text_oarchive` 被用来把数据序列化为文本流， `boost::archive::text_iarchive` 就用来从文本流恢复数据。 为了使用这些类，必须包含 `boost/archive/text_iarchive.hpp` 头文件。

归档的构造函数需要一个输入或者输出流作为参数。 流分别用来序列化或恢复数据。 虽然上面的应用程序使用了一个文件流，其他流，如 stringstream 流也是可以的。

```cpp
#include <boost/archive/text_oarchive.hpp> 
#include <boost/archive/text_iarchive.hpp> 
#include <iostream> 
#include <sstream> 

std::stringstream ss; 

void save() 
{ 
  boost::archive::text_oarchive oa(ss); 
  int i = 1; 
  oa << i; 
} 

void load() 
{ 
  boost::archive::text_iarchive ia(ss); 
  int i = 0; 
  ia >> i; 
  std::cout << i << std::endl; 
} 

int main() 
{ 
  save(); 
  load(); 
} 
```

*   下载源代码

这个应用程序也向标准输出流写了 `1`。 然而，与前面的例子相比, 数据却是用 stringstream 流序列化的。

到目前为止， 原始的数据类型已经被序列化了。 接下来的例子演示如何序列化用户定义类型的对象。

```cpp
#include <boost/archive/text_oarchive.hpp> 
#include <boost/archive/text_iarchive.hpp> 
#include <iostream> 
#include <sstream> 

std::stringstream ss; 

class person 
{ 
public: 
  person() 
  { 
  } 

  person(int age) 
    : age_(age) 
  { 
  } 

  int age() const 
  { 
    return age_; 
  } 

private: 
  friend class boost::serialization::access; 

  template <typename Archive> 
  void serialize(Archive &ar, const unsigned int version) 
  { 
    ar & age_; 
  } 

  int age_; 
}; 

void save() 
{ 
  boost::archive::text_oarchive oa(ss); 
  person p(31); 
  oa << p; 
} 

void load() 
{ 
  boost::archive::text_iarchive ia(ss); 
  person p; 
  ia >> p; 
  std::cout << p.age() << std::endl; 
} 

int main() 
{ 
  save(); 
  load(); 
} 
```

*   下载源代码

为了序列化用户定义类型的对话， `serialize()` 函数必须定义，它在对象序列化或从字节流中恢复是被调用。 由于 `serialize ()` 函数既用来序列化又用来恢复数据， Boost.Serialization 除了 `&lt;&lt;` 和 `&gt;&gt;` 之外还提供了 `&` 操作符。如果使用这个操作符，就不再需要在 `serialize()` 函数中区分是序列化和恢复了。

`serialize ()` 在对象序列化或恢复时自动调用。它应从来不被明确地调用，所以应生命为私有的。 这样的话， `boost::serialization::access` 类必须被声明为友元，以允许 Boost.Serialization 能够访问到这个函数。

有些情况下需要添加 `serialize()` 函数却不能修改现有的类。 比如，对于来自 C++ 标准库或其他库的类就是这样的。

```cpp
#include <boost/archive/text_oarchive.hpp> 
#include <boost/archive/text_iarchive.hpp> 
#include <iostream> 
#include <sstream> 

std::stringstream ss; 

class person 
{ 
public: 
  person() 
  { 
  } 

  person(int age) 
    : age_(age) 
  { 
  } 

  int age() const 
  { 
    return age_; 
  } 

private: 
  friend class boost::serialization::access; 

  template <typename Archive> 
  friend void serialize(Archive &ar, person &p, const unsigned int version); 

  int age_; 
}; 

template <typename Archive> 
void serialize(Archive &ar, person &p, const unsigned int version) 
{ 
  ar & p.age_; 
} 

void save() 
{ 
  boost::archive::text_oarchive oa(ss); 
  person p(31); 
  oa << p; 
} 

void load() 
{ 
  boost::archive::text_iarchive ia(ss); 
  person p; 
  ia >> p; 
  std::cout << p.age() << std::endl; 
} 

int main() 
{ 
  save(); 
  load(); 
} 
```

*   下载源代码

为了序列化那些不能被修改的数据类型，要定义一个单独的函数 `serialize()`，如上面的例子所示。 这个函数需要相应的数据类型的引用作为它的第二个参数。

如果要被序列化的数据类型中含有不能经由公有函数访问的私有属性，事情就变得复杂了。 在这种情况下，该数据列席就需要修改。 在上面应用程序中的 `serialize ()` 函数如果不声明为 `friend` ，就不能访问 `age_` 属性。

不过还好，Boost.Serialization 为许多 C++标准库的类提供了 `serialize()` 函数。 为了序列化基于 C++ 标准库的类，需要包含额外的头文件。

```cpp
#include <boost/archive/text_oarchive.hpp> 
#include <boost/archive/text_iarchive.hpp> 
#include <boost/serialization/string.hpp> 
#include <iostream> 
#include <sstream> 
#include <string> 

std::stringstream ss; 

class person 
{ 
public: 
  person() 
  { 
  } 

  person(int age, const std::string &name) 
    : age_(age), name_(name) 
  { 
  } 

  int age() const 
  { 
    return age_; 
  } 

  std::string name() const 
  { 
    return name_; 
  } 

private: 
  friend class boost::serialization::access; 

  template <typename Archive> 
  friend void serialize(Archive &ar, person &p, const unsigned int version); 

  int age_; 
  std::string name_; 
}; 

template <typename Archive> 
void serialize(Archive &ar, person &p, const unsigned int version) 
{ 
  ar & p.age_; 
  ar & p.name_; 
} 

void save() 
{ 
  boost::archive::text_oarchive oa(ss); 
  person p(31, "Boris"); 
  oa << p; 
} 

void load() 
{ 
  boost::archive::text_iarchive ia(ss); 
  person p; 
  ia >> p; 
  std::cout << p.age() << std::endl; 
  std::cout << p.name() << std::endl; 
} 

int main() 
{ 
  save(); 
  load(); 
} 
```

*   下载源代码

这个例子扩展了 `person` 类，增加了 `std::string` 类型的名称变量，为了序列化这个属性 property, the header file `boost/serialization/string.hpp` 头文件必须包含，它提供了合适的单独的 `serialize ()` 函数。

正如前面所提到的， Boost.Serialization 为许多 C++ 标准库类定义了 `serialize ()` 函数。 这些都定义在和 C++ 标准库头文件名称相对应的头文件中。 为了序列化 `std::string` 类型的对象，必须包含 `boost/serialization/string.hpp` 头文件。 为了序列化 `std::vector` 类型的对象，必须包含 `boost/serialization/vector.hpp` 头文件。 于是在给定的场合中应该包含哪个头文件就显而易见了。

还有一个 `serialize ()`函数的参数，到目前为止我们一直忽略没谈到，那就是 `version` 。 如果归档需要向前兼容，以支持给定应用程序的未来版本，那么这个参数就是有意义的。 接下来的例子考虑到 `person` 类的归档需要向前兼容。由于 `person` 的原始版本没有包含任何名称，新版本的 `person` 应该能够处理不带名称的旧的归档。

```cpp
#include <boost/archive/text_oarchive.hpp> 
#include <boost/archive/text_iarchive.hpp> 
#include <boost/serialization/string.hpp> 
#include <iostream> 
#include <sstream> 
#include <string> 

std::stringstream ss; 

class person 
{ 
public: 
  person() 
  { 
  } 

  person(int age, const std::string &name) 
    : age_(age), name_(name) 
  { 
  } 

  int age() const 
  { 
    return age_; 
  } 

  std::string name() const 
  { 
    return name_; 
  } 

private: 
  friend class boost::serialization::access; 

  template <typename Archive> 
  friend void serialize(Archive &ar, person &p, const unsigned int version); 

  int age_; 
  std::string name_; 
}; 

template <typename Archive> 
void serialize(Archive &ar, person &p, const unsigned int version) 
{ 
  ar & p.age_; 
  if (version > 0) 
    ar & p.name_; 
} 

BOOST_CLASS_VERSION(person, 1) 

void save() 
{ 
  boost::archive::text_oarchive oa(ss); 
  person p(31, "Boris"); 
  oa << p; 
} 

void load() 
{ 
  boost::archive::text_iarchive ia(ss); 
  person p; 
  ia >> p; 
  std::cout << p.age() << std::endl; 
  std::cout << p.name() << std::endl; 
} 

int main() 
{ 
  save(); 
  load(); 
} 
```

*   下载源代码

`BOOST_CLASS_VERSION` 宏用来指定类的版本号。 上面例子中 `person` 类的版本号设置为 1。 如果没有使用 `BOOST_CLASS_VERSION` ， 版本号缺省是 0。

版本号存储在归档文件中，因此也就是归档的一部份。 当一个特定类的版本号通过 `BOOST_CLASS_VERSION` 宏，在序列化时给定时， `serialize ()` 函数的 `version` 参数被设为给定值存储在归档中。 如果新版本的 `person` 访问一个包含旧版本序列化对象的归档时， `name_` 由于旧版本不含有这个属性而不能恢复。 通过这种机制，Boost.Serialization 提供了向前兼容归档的支持。

## 11.3\. 指针和引用

Boost.Serialization 还能序列化指针和引用。 由于指针存储对象的地址，序列化对象的地址没有什么意义，而是在序列化指针和引用时，对象的引用被自动地序列化。

```cpp
#include <boost/archive/text_oarchive.hpp> 
#include <boost/archive/text_iarchive.hpp> 
#include <iostream> 
#include <sstream> 

std::stringstream ss; 

class person 
{ 
public: 
  person() 
  { 
  } 

  person(int age) 
    : age_(age) 
  { 
  } 

  int age() const 
  { 
    return age_; 
  } 

private: 
  friend class boost::serialization::access; 

  template <typename Archive> 
  void serialize(Archive &ar, const unsigned int version) 
  { 
    ar & age_; 
  } 

  int age_; 
}; 

void save() 
{ 
  boost::archive::text_oarchive oa(ss); 
  person *p = new person(31); 
  oa << p; 
  std::cout << std::hex << p << std::endl; 
  delete p; 
} 

void load() 
{ 
  boost::archive::text_iarchive ia(ss); 
  person *p; 
  ia >> p; 
  std::cout << std::hex << p << std::endl; 
  std::cout << p->age() << std::endl; 
  delete p; 
} 

int main() 
{ 
  save(); 
  load(); 
} 
```

*   下载源代码

上面的应用程序创建了一个新的 `person` 类型的对象，使用 `new` 创建并赋值给指针 `p` 。 是指针 - 而不是 `*p` - 被序列化了。Boost.Serialization 自动地通过 `p` 的引用序列化对象本身而不是对象的地址。

如果归档被恢复， `p` 不必指向相同的地址。 而是创建新对象并将它的地址赋值给 `p` 。 Boost.Serialization 只保证对象和之前序列化的对象相同，而不是地址相同。

由于新式的 C++ 在动态分配内存有关的地方使用 智能指针 (smart pointers) ， Boost.Serialization 对此也提供了相应的支持。

```cpp
#include <boost/archive/text_oarchive.hpp> 
#include <boost/archive/text_iarchive.hpp> 
#include <boost/serialization/scoped_ptr.hpp> 
#include <boost/scoped_ptr.hpp> 
#include <iostream> 
#include <sstream> 

std::stringstream ss; 

class person 
{ 
public: 
  person() 
  { 
  } 

  person(int age) 
    : age_(age) 
  { 
  } 

  int age() const 
  { 
    return age_; 
  } 

private: 
  friend class boost::serialization::access; 

  template <typename Archive> 
  void serialize(Archive &ar, const unsigned int version) 
  { 
    ar & age_; 
  } 

  int age_; 
}; 

void save() 
{ 
  boost::archive::text_oarchive oa(ss); 
  boost::scoped_ptr<person> p(new person(31)); 
  oa << p; 
} 

void load() 
{ 
  boost::archive::text_iarchive ia(ss); 
  boost::scoped_ptr<person> p; 
  ia >> p; 
  std::cout << p->age() << std::endl; 
} 

int main() 
{ 
  save(); 
  load(); 
} 
```

*   下载源代码

例子中使用了智能指针 `boost::scoped_ptr` 来管理动态分配的 `person` 类型的对象。 为了序列化这样的指针，必须包含 `boost/serialization/scoped_ptr.hpp` 头文件。

在使用 `boost::shared_ptr` 类型的智能指针的时候需要序列化，那么必须包含 `boost/serialization/shared_ptr.hpp` 头文件。

下面的应用程序使用引用替代了指针。

```cpp
#include <boost/archive/text_oarchive.hpp> 
#include <boost/archive/text_iarchive.hpp> 
#include <iostream> 
#include <sstream> 

std::stringstream ss; 

class person 
{ 
public: 
  person() 
  { 
  } 

  person(int age) 
    : age_(age) 
  { 
  } 

  int age() const 
  { 
    return age_; 
  } 

private: 
  friend class boost::serialization::access; 

  template <typename Archive> 
  void serialize(Archive &ar, const unsigned int version) 
  { 
    ar & age_; 
  } 

  int age_; 
}; 

void save() 
{ 
  boost::archive::text_oarchive oa(ss); 
  person p(31); 
  person &pp = p; 
  oa << pp; 
} 

void load() 
{ 
  boost::archive::text_iarchive ia(ss); 
  person p; 
  person &pp = p; 
  ia >> pp; 
  std::cout << pp.age() << std::endl; 
} 

int main() 
{ 
  save(); 
  load(); 
} 
```

*   下载源代码

可见，Boost.Serialization 还能没有任何问题地序列化引用。 就像指针一样，引用对象被自动地序列化。

## 11.4\. 对象类层次结构的序列化

为了序列化基于类层次结构的对象，子类必须在 `serialize ()`函数中访问 `boost::serialization::base_object ()`。 此函数确保继承自基类的属性也能正确地序列化。 下面的例子演示了一个名为 `developer` 类，它继承自类 `person` 。

```cpp
#include <boost/archive/text_oarchive.hpp> 
#include <boost/archive/text_iarchive.hpp> 
#include <boost/serialization/string.hpp> 
#include <iostream> 
#include <sstream> 
#include <string> 

std::stringstream ss; 

class person 
{ 
public: 
  person() 
  { 
  } 

  person(int age) 
    : age_(age) 
  { 
  } 

  int age() const 
  { 
    return age_; 
  } 

private: 
  friend class boost::serialization::access; 

  template <typename Archive> 
  void serialize(Archive &ar, const unsigned int version) 
  { 
    ar & age_; 
  } 

  int age_; 
}; 

class developer 
  : public person 
{ 
public: 
  developer() 
  { 
  } 

  developer(int age, const std::string &language) 
    : person(age), language_(language) 
  { 
  } 

  std::string language() const 
  { 
    return language_; 
  } 

private: 
  friend class boost::serialization::access; 

  template <typename Archive> 
  void serialize(Archive &ar, const unsigned int version) 
  { 
    ar & boost::serialization::base_object<person>(*this); 
    ar & language_; 
  } 

  std::string language_; 
}; 

void save() 
{ 
  boost::archive::text_oarchive oa(ss); 
  developer d(31, "C++"); 
  oa << d; 
} 

void load() 
{ 
  boost::archive::text_iarchive ia(ss); 
  developer d; 
  ia >> d; 
  std::cout << d.age() << std::endl; 
  std::cout << d.language() << std::endl; 
} 

int main() 
{ 
  save(); 
  load(); 
} 
```

*   下载源代码

`person` 和 `developer` 这两个类都包含有一个私有的 `serialize ()` 函数， 它使得基于其他类的对象能被序列化。 由于 `developer` 类继承自 `person` 类， 所以它的 `serialize ()` 函数必须确保继承自 `person` 属性也能被序列化。

继承自基类的属性被序列化是通过在子类的 `serialize ()` 函数中用 `boost::serialization::base_object ()` 函数访问基类实现的。 在例子中强制要求使用这个函数而不是 `static_cast` 是因为只有 `boost::serialization::base_object ()` 才能保证正确地序列化。

动态创建对象的地址可以被赋值给对应的基类类型的指针。 下面的例子演示了 Boost.Serialization 还能够正确地序列化它们。

```cpp
#include <boost/archive/text_oarchive.hpp> 
#include <boost/archive/text_iarchive.hpp> 
#include <boost/serialization/string.hpp> 
#include <boost/serialization/export.hpp> 
#include <iostream> 
#include <sstream> 
#include <string> 

std::stringstream ss; 

class person 
{ 
public: 
  person() 
  { 
  } 

  person(int age) 
    : age_(age) 
  { 
  } 

  virtual int age() const 
  { 
    return age_; 
  } 

private: 
  friend class boost::serialization::access; 

  template <typename Archive> 
  void serialize(Archive &ar, const unsigned int version) 
  { 
    ar & age_; 
  } 

  int age_; 
}; 

class developer 
  : public person 
{ 
public: 
  developer() 
  { 
  } 

  developer(int age, const std::string &language) 
    : person(age), language_(language) 
  { 
  } 

  std::string language() const 
  { 
    return language_; 
  } 

private: 
  friend class boost::serialization::access; 

  template <typename Archive> 
  void serialize(Archive &ar, const unsigned int version) 
  { 
    ar & boost::serialization::base_object<person>(*this); 
    ar & language_; 
  } 

  std::string language_; 
}; 

BOOST_CLASS_EXPORT(developer) 

void save() 
{ 
  boost::archive::text_oarchive oa(ss); 
  person *p = new developer(31, "C++"); 
  oa << p; 
  delete p; 
} 

void load() 
{ 
  boost::archive::text_iarchive ia(ss); 
  person *p; 
  ia >> p; 
  std::cout << p->age() << std::endl; 
  delete p; 
} 

int main() 
{ 
  save(); 
  load(); 
} 
```

*   下载源代码

应用程序在 `save ()` 函数创建了 `developer` 类型的对象并赋值给 `person*` 类型的指针，接下来通过 `&lt;&lt;` 序列化。

正如在前面章节中提到的， 引用对象被自动地序列化。 为了让 Boost.Serialization 识别将要序列化的 `developer` 类型的对象，即使指针是 `person*` 类型的对象。 `developer` 类需要相应的声明。 这是通过这个 `BOOST_CLASS_EXPORT` 宏实现的，它定义在 `boost/serialization/export.hpp` 文件中。 因为 `developer` 这个数据类型没有指针形式的定义，所以 Boost.Serialization 没有这个宏就不能正确地序列化 `developer` 。

如果子类对象需要通过基类的指针序列化，那么 `BOOST_CLASS_EXPORT` 宏必须要用。

由于静态注册的原因， `BOOST_CLASS_EXPORT` 的一个缺点是可能有些注册的类最后是不需要序列化的。 Boost.Serialization 为这种情况提供一种解决方案。

```cpp
#include <boost/archive/text_oarchive.hpp> 
#include <boost/archive/text_iarchive.hpp> 
#include <boost/serialization/string.hpp> 
#include <boost/serialization/export.hpp> 
#include <iostream> 
#include <sstream> 
#include <string> 

std::stringstream ss; 

class person 
{ 
public: 
  person() 
  { 
  } 

  person(int age) 
    : age_(age) 
  { 
  } 

  virtual int age() const 
  { 
    return age_; 
  } 

private: 
  friend class boost::serialization::access; 

  template <typename Archive> 
  void serialize(Archive &ar, const unsigned int version) 
  { 
    ar & age_; 
  } 

  int age_; 
}; 

class developer 
  : public person 
{ 
public: 
  developer() 
  { 
  } 

  developer(int age, const std::string &language) 
    : person(age), language_(language) 
  { 
  } 

  std::string language() const 
  { 
    return language_; 
  } 

private: 
  friend class boost::serialization::access; 

  template <typename Archive> 
  void serialize(Archive &ar, const unsigned int version) 
  { 
    ar & boost::serialization::base_object<person>(*this); 
    ar & language_; 
  } 

  std::string language_; 
}; 

void save() 
{ 
  boost::archive::text_oarchive oa(ss); 
  oa.register_type<developer>(); 
  person *p = new developer(31, "C++"); 
  oa << p; 
  delete p; 
} 

void load() 
{ 
  boost::archive::text_iarchive ia(ss); 
  ia.register_type<developer>(); 
  person *p; 
  ia >> p; 
  std::cout << p->age() << std::endl; 
  delete p; 
} 

int main() 
{ 
  save(); 
  load(); 
} 
```

*   下载源代码

上面的应用程序没有使用 `BOOST_CLASS_EXPORT` 宏，而是调用了 `register_type ()` 模板函数。 需要注册的类型作为模板参数传入。

请注意 `register_type ()` 必须在 `save ()` 和 `load ()` 都要调用。

`register_type ()` 的优点是只有需要序列化的类才注册。 比如在开发一个库时，你不知道开发人员将来要序列化哪些类。 当然 `BOOST_CLASS_EXPORT` 宏用起来简单，可它却可能注册那些不需要序列化的类型。

## 11.5\. 优化用封装函数

在理解了如何序列化对象之后，本节介绍用来优化序列化过程的封装函数。 通过这个函数，对象被打上标记允许 Boost.Serialization 使用一些优化技术。

下面例子使用不带封装函数的 Boost.Serialization 。

```cpp
#include <boost/archive/text_oarchive.hpp> 
#include <boost/archive/text_iarchive.hpp> 
#include <boost/array.hpp> 
#include <iostream> 
#include <sstream> 

std::stringstream ss; 

void save() 
{ 
  boost::archive::text_oarchive oa(ss); 
  boost::array<int, 3> a = { 0, 1, 2 }; 
  oa << a; 
} 

void load() 
{ 
  boost::archive::text_iarchive ia(ss); 
  boost::array<int, 3> a; 
  ia >> a; 
  std::cout << a[0] << ", " << a[1] << ", " << a[2] << std::endl; 
} 

int main() 
{ 
  save(); 
  load(); 
} 
```

*   下载源代码

上面的应用程序创建一个文本流 `22 serialization::archive 5 0 0 3 0 1 2` 并将其写到标准输出流中。 使用封装函数 `boost::serialization::make_array ()` ，输出可以缩短到 `22 serialization::archive 5 0 1 2` 。

```cpp
#include <boost/archive/text_oarchive.hpp> 
#include <boost/archive/text_iarchive.hpp> 
#include <boost/serialization/array.hpp> 
#include <boost/array.hpp> 
#include <iostream> 
#include <sstream> 

std::stringstream ss; 

void save() 
{ 
  boost::archive::text_oarchive oa(ss); 
  boost::array<int, 3> a = { 0, 1, 2 }; 
  oa << boost::serialization::make_array(a.data(), a.size()); 
} 

void load() 
{ 
  boost::archive::text_iarchive ia(ss); 
  boost::array<int, 3> a; 
  ia >> boost::serialization::make_array(a.data(), a.size()); 
  std::cout << a[0] << ", " << a[1] << ", " << a[2] << std::endl; 
} 

int main() 
{ 
  save(); 
  load(); 
} 
```

*   下载源代码

`boost::serialization::make_array ()` 函数需要地址和数组的长度。 由于长度是硬编码的，所以它不需要作为 `boost::array` 类型的一部分序列化。任何时候，如果 `boost::array` 或 `std::vector` 包含一个可以直接序列化的数组，都可以使用这个函数。 其他一般需要序列化的属性不能被序列化。

另一个 Boost.Serialization 提供的封装函数是 `boost::serialization::make_binary_object ()` 。 与 `boost::serialization::make_array ()` 类似，它也需要地址和长度。 `boost::serialization::make_binary_object ()` 函数只是为了用来序列化没有底层结构的二进制数据，而 `boost::serialization::make_array ()` 是用来序列化数组的。

## 11.6\. 练习

You can buy [solutions to all exercises](http://en.highscore.de/shop/index.php?p=boost-solution) in this book as a ZIP file.

1.  开发一个应用程序，能够将任意个数有名称，部门和雇员唯一标识号构成的记录， 序列化到文件并从中恢复。 记录应该在恢复后在屏幕上显示。 用样本记录测试应用程序。

2.  扩展上面的应用程序，为每个雇员存储生日。 应用程序应该还可以恢复 在上面的练习创建的的旧版本的记录。