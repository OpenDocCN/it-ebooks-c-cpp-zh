# 第四章 事件处理

# 第四章 事件处理

### 目录

*   4.1 概述
*   4.2 信号 Signals
*   4.3 连接 Connections
*   4.4 练习

![](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 该书采用 [Creative Commons License](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 授权

## 4.1\. 概述

很多开发者在听到术语'事件处理'时就会想到 GUI：点击一下某个按钮，相关联的功能就会被执行。 点击本身就是事件，而功能就是相对应的事件处理器。

这一模式的使用当然不仅限于 GUI。 一般情况下，任意对象都可以调用基于特定事件的专门函数。 本章所介绍的 [Boost.Signals](http://www.boost.org/libs/signals) 库提供了一个简单的方法在 C++ 中应用这一模式。

严格来说，Boost.Function 库也可以用于事件处理。 不过，Boost.Function 和 Boost.Signals 之间的一个主要区别在于，Boost.Signals 能够将一个以上的事件处理器关联至单个事件。 因此，Boost.Signals 可以更好地支持事件驱动的开发，当需要进行事件处理时，应作为第一选择。

## 4.2\. 信号 Signals

虽然这个库的名字乍一看好象有点误导，但实际上并非如此。 Boost.Signals 所实现的模式被命名为 '信号至插槽' (signal to slot)，它基于以下概念：当对应的信号被发出时，相关联的插槽即被执行。 原则上，你可以把单词 '信号' 和 '插槽' 分别替换为 '事件' 和 '事件处理器'。 不过，由于信号可以在任意给定的时间发出，所以这一概念放弃了 '事件' 的名字。

因此，Boost.Signals 没有提供任何类似于 '事件' 的类。 相反，它提供了一个名为 `boost::signal` 的类，定义于 `boost/signal.hpp`. 实际上，这个头文件是唯一一个需要知道的，因为它会自动包含其它相关的头文件。

Boost.Signals 定义了其它一些类，位于 boost::signals 名字空间中。 由于 `boost::signal` 是最常被用到的类，所以它是位于名字空间 boost 中的。

```cpp
#include <boost/signal.hpp> 
#include <iostream> 

void func() 
{ 
  std::cout << "Hello, world!" << std::endl; 
} 

int main() 
{ 
  boost::signal<void ()> s; 
  s.connect(func); 
  s(); 
} 
```

*   下载源代码

`boost::signal` 实际上被实现为一个模板函数，具有被用作为事件处理器的函数的签名，该签名也是它的模板参数。 在这个例子中，只有签名为 `void ()` 的函数可以被成功关联至信号 `s`。

函数 `func()` 被通过 `connect()` 方法关联至信号 `s`。 由于 `func()` 符合所要求的 `void ()` 签名，所以该关联成功建立。因此当信号 `s` 被触发时，`func()` 将被调用。

信号是通过调用 `s` 来触发的，就象普通的函数调用那样。 这个函数的签名对应于作为模板参数传入的签名：因为 `void ()` 不要求任何参数，所以括号内是空的。

调用 `s` 会引发一个触发器，进而执行相应的 `func()` 函数 - 之前用 `connect()` 关联了的。

同一例子也可以用 Boost.Function 来实现。

```cpp
#include <boost/function.hpp> 
#include <iostream> 

void func() 
{ 
  std::cout << "Hello, world!" << std::endl; 
} 

int main() 
{ 
  boost::function<void ()> f; 
  f = func; 
  f(); 
} 
```

*   下载源代码

和前一个例子相类似，`func()` 被关联至 `f`。 当 `f` 被调用时，就会相应地执行 `func()`。 Boost.Function 仅限于这种情形下适用，而 Boost.Signals 则提供了多得多的方式，如关联多个函数至单个特定信号，示例如下。

```cpp
#include <boost/signal.hpp> 
#include <iostream> 

void func1() 
{ 
  std::cout << "Hello" << std::flush; 
} 

void func2() 
{ 
  std::cout << ", world!" << std::endl; 
} 

int main() 
{ 
  boost::signal<void ()> s; 
  s.connect(func1); 
  s.connect(func2); 
  s(); 
} 
```

*   下载源代码

`boost::signal` 可以通过反复调用 `connect()` 方法来把多个函数赋值给单个特定信号。 当该信号被触发时，这些函数被按照之前用 `connect()` 进行关联时的顺序来执行。

另外，执行的顺序也可通过 `connect()` 方法的另一个重载版本来明确指定，该重载版本要求以一个 `int` 类型的值作为额外的参数。

```cpp
#include <boost/signal.hpp> 
#include <iostream> 

void func1() 
{ 
  std::cout << "Hello" << std::flush; 
} 

void func2() 
{ 
  std::cout << ", world!" << std::endl; 
} 

int main() 
{ 
  boost::signal<void ()> s; 
  s.connect(1, func2); 
  s.connect(0, func1); 
  s(); 
} 
```

*   下载源代码

和前一个例子一样，`func1()` 在 `func2()` 之前执行。

要释放某个函数与给定信号的关联，可以用 `disconnect()` 方法。

```cpp
#include <boost/signal.hpp> 
#include <iostream> 

void func1() 
{ 
  std::cout << "Hello" << std::endl; 
} 

void func2() 
{ 
  std::cout << ", world!" << std::endl; 
} 

int main() 
{ 
  boost::signal<void ()> s; 
  s.connect(func1); 
  s.connect(func2); 
  s.disconnect(func2); 
  s(); 
} 
```

*   下载源代码

这个例子仅输出 `Hello`，因为与 `func2()` 的关联在触发信号之前已经被释放。

除了 `connect()` 和 `disconnect()` 以外，`boost::signal` 还提供了几个方法。

```cpp
#include <boost/signal.hpp> 
#include <iostream> 

void func1() 
{ 
  std::cout << "Hello" << std::flush; 
} 

void func2() 
{ 
  std::cout << ", world!" << std::endl; 
} 

int main() 
{ 
  boost::signal<void ()> s; 
  s.connect(func1); 
  s.connect(func2); 
  std::cout << s.num_slots() << std::endl; 
  if (!s.empty()) 
    s(); 
  s.disconnect_all_slots(); 
} 
```

*   下载源代码

`num_slots()` 返回已关联函数的数量。如果没有函数被关联，则 `num_slots()` 返回 0。 在这种特定情况下，可以用 `empty()` 方法来替代。 `disconnect_all_slots()` 方法所做的实际上正是它的名字所表达的：释放所有已有的关联。

看完了函数如何被关联至信号，以及弄明白了信号被触发时会发生什么事之后，还有一个问题：这些函数的返回值去了哪里？ 以下例子回答了这个问题。

```cpp
#include <boost/signal.hpp> 
#include <iostream> 

int func1() 
{ 
  return 1; 
} 

int func2() 
{ 
  return 2; 
} 

int main() 
{ 
  boost::signal<int ()> s; 
  s.connect(func1); 
  s.connect(func2); 
  std::cout << s() << std::endl; 
} 
```

*   下载源代码

`func1()` 和 `func2()` 都具有 `int` 类型的返回值。 `s` 将处理两个返回值，并将它们都写出至标准输出流。 那么，到底会发生什么呢？

以上例子实际上会把 `2` 写出至标准输出流。 两个返回值都被 `s` 正确接收，但除了最后一个值，其它值都会被忽略。 缺省情况下，所有被关联函数中，实际上只有最后一个返回值被返回。

你可以定制一个信号，令每个返回值都被相应地处理。 为此，要把一个称为合成器(combiner)的东西作为第二个参数传递给 `boost::signal`。

```cpp
#include <boost/signal.hpp> 
#include <iostream> 
#include <algorithm> 

int func1() 
{ 
  return 1; 
} 

int func2() 
{ 
  return 2; 
} 

template <typename T> 
struct min_element 
{ 
  typedef T result_type; 

  template <typename InputIterator> 
  T operator()(InputIterator first, InputIterator last) const 
  { 
    return *std::min_element(first, last); 
  } 
}; 

int main() 
{ 
  boost::signal<int (), min_element<int> > s; 
  s.connect(func1); 
  s.connect(func2); 
  std::cout << s() << std::endl; 
} 
```

*   下载源代码

合成器是一个重载了 `operator()()` 操作符的类。这个操作符会被自动调用，传入两个迭代器，指向某个特定信号的所有返回值。 以上例子使用了标准 C++ 算法 `std::min_element()` 来确定并返回最小的值。

不幸的是，我们不可能把象 `std::min_element()` 这样的一个算法直接传给 `boost::signal` 作为一个模板参数。 `boost::signal` 要求这个合成器定义一个名为 `result_type` 的类型，用于说明 `operator()()` 操作符返回值的类型。 由于在标准 C++ 算法中缺少这个类型，所以在编译时会产生一个相应的错误。

除了对返回值进行分析以外，合成器也可以保存它们。

```cpp
#include <boost/signal.hpp> 
#include <iostream> 
#include <vector> 
#include <algorithm> 

int func1() 
{ 
  return 1; 
} 

int func2() 
{ 
  return 2; 
} 

template <typename T> 
struct min_element 
{ 
  typedef T result_type; 

  template <typename InputIterator> 
  T operator()(InputIterator first, InputIterator last) const 
  { 
    return T(first, last); 
  } 
}; 

int main() 
{ 
  boost::signal<int (), min_element<std::vector<int> > > s; 
  s.connect(func1); 
  s.connect(func2); 
  std::vector<int> v = s(); 
  std::cout << *std::min_element(v.begin(), v.end()) << std::endl; 
} 
```

*   下载源代码

这个例子把所有返回值保存在一个 vector 中，再由 `s()` 返回。

## 4.3\. 连接 Connections

函数可以通过由 `boost::signal` 所提供的 `connect()` 和 `disconnect()` 方法的帮助来进行管理。 由于 `connect()` 会返回一个类型为 `boost::signals::connection` 的值，它们可以通过其它方法来管理。

```cpp
#include <boost/signal.hpp> 
#include <iostream> 

void func() 
{ 
  std::cout << "Hello, world!" << std::endl; 
} 

int main() 
{ 
  boost::signal<void ()> s; 
  boost::signals::connection c = s.connect(func); 
  s(); 
  c.disconnect(); 
} 
```

*   下载源代码

`boost::signal` 的 `disconnect()` 方法需要传入一个函数指针，而直接调用 `boost::signals::connection` 对象上的 `disconnect()` 方法则略去该参数。

除了 `disconnect()` 方法之外，`boost::signals::connection` 还提供了其它方法，如 `block()` 和 `unblock()`。

```cpp
#include <boost/signal.hpp> 
#include <iostream> 

void func() 
{ 
  std::cout << "Hello, world!" << std::endl; 
} 

int main() 
{ 
  boost::signal<void ()> s; 
  boost::signals::connection c = s.connect(func); 
  c.block(); 
  s(); 
  c.unblock(); 
  s(); 
} 
```

*   下载源代码

以上程序只会执行一次 `func()`。 虽然信号 `s` 被触发了两次，但是在第一次触发时 `func()` 不会被调用，因为连接 `c` 实际上已经被 `block()` 调用所阻塞。 由于在第二次触发之前调用了 `unblock()`，所以之后 `func()` 被正确地执行。

除了 `boost::signals::connection` 以外，还有一个名为 `boost::signals::scoped_connection` 的类，它会在析构时自动释放连接。

```cpp
#include <boost/signal.hpp> 
#include <iostream> 

void func() 
{ 
  std::cout << "Hello, world!" << std::endl; 
} 

int main() 
{ 
  boost::signal<void ()> s; 
  { 
    boost::signals::scoped_connection c = s.connect(func); 
  } 
  s(); 
} 
```

*   下载源代码

因为连接对象 `c` 在信号触发之前被销毁，所以 `func()` 不会被调用。

`boost::signals::scoped_connection` 实际上是派生自 `boost::signals::connection` 的，所以它提供了相同的方法。它们之间的区别仅在于，在析构 `boost::signals::scoped_connection` 时，连接会自动释放。

虽然 `boost::signals::scoped_connection` 的确令自动释放连接更为容易，但是该类型的对象仍需要管理。 如果在其它情形下连接也可以被自动释放，而且不需要管理这些对象的话，就更好了。

```cpp
#include <boost/signal.hpp> 
#include <boost/bind.hpp> 
#include <iostream> 
#include <memory> 

class world 
{ 
  public: 
    void hello() const 
    { 
      std::cout << "Hello, world!" << std::endl; 
    } 
}; 

int main() 
{ 
  boost::signal<void ()> s; 
  { 
    std::auto_ptr<world> w(new world()); 
    s.connect(boost::bind(&world::hello, w.get())); 
  } 
  std::cout << s.num_slots() << std::endl; 
  s(); 
} 
```

*   下载源代码

以上程序使用 Boost.Bind 将一个对象的方法关联至一个信号。 在信号触发之前，这个对象就被销毁了，这会产生问题。 我们不传递实际的对象 `w`，而只传递一个指针给 `boost::bind()`。 在 `s()` 被实际调用的时候，该指针所引向的对象已不再存在。

可以如下修改这个程序，使得一旦对象 `w` 被销毁，连接就会自动释放。

```cpp
#include <boost/signal.hpp> 
#include <boost/bind.hpp> 
#include <iostream> 
#include <memory> 

class world : 
  public boost::signals::trackable 
{ 
  public: 
    void hello() const 
    { 
      std::cout << "Hello, world!" << std::endl; 
    } 
}; 

int main() 
{ 
  boost::signal<void ()> s; 
  { 
    std::auto_ptr<world> w(new world()); 
    s.connect(boost::bind(&world::hello, w.get())); 
  } 
  std::cout << s.num_slots() << std::endl; 
  s(); 
} 
```

*   下载源代码

如果现在再执行，`num_slots()` 会返回 `0` 以确保不会试图调用已销毁对象之上的方法。 仅需的修改是让 `world` 类继承自 `boost::signals::trackable`。 当使用对象的指针而不是对象的副本来关联函数至信号时，`boost::signals::trackable` 可以显著简化连接的管理。

## 4.4\. 练习

You can buy [solutions to all exercises](http://en.highscore.de/shop/index.php?p=boost-solution) in this book as a ZIP file.

1.  编写一个程序，定义一个名为 `button` 的类，表示 GUI 中的一个可点击按钮。 为该类加入两个方法 `add_handler()` 和 `remove_handler()`，它们均要求一个函数名作为参数。 如果 `click()` 方法被调用，已登记的函数将被按顺序执行。

    如下测试你的代码，创建一个 `button` 类的实例，从事件处理器内部向标准输出流写出一个信息。 调用 `click()` 函数模拟用鼠标点击该按钮。