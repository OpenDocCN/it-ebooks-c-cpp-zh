# Library 1\. Smart_ptr

# Library 1\. Smart_ptr

*   Smart_ptr 库如何改进你的程序？
*   何时我们需要智能指针？
*   Smart_ptr 如何适应标准库？
*   scoped_ptr
*   scoped_array
*   shared_ptr
*   shared_array
*   intrusive_ptr
*   weak_ptr
*   Smart_ptr 总结

# Smart_ptr 库如何改进你的程序？

## Smart_ptr 库如何改进你的程序？

*   使用`shared_ptr`进行对象的生存期自动管理，使得分享资源所有权变得有效且安全。

*   使用`weak_ptr`可以安全地观测共享资源，避免了悬挂的指针。

*   使用`scoped_ptr` 和 `scoped_array`限制资源的使用范围，使得代码更易于编写和维护，并有助于写出异常安全的代码。

智能指针解决了资源生存期管理的问题(尤其是动态分配的对象[1]). 智能指针有各种不同的风格。多数都有一种共同的关键特性：自动资源管理。这种特性可能以不同的方式出现：如动态分配对象的生存期控制，和获取及释放资源 (文件, 网络连接)。Boost 的智能指针主要针对第一种情况，它们保存指向动态分配对象的指针，并在正确的时候删除这些对象。你可能觉得奇怪为什么这些智能指针 不多做一点工作。它们不可以很容易就覆盖所有资源管理的不同情况吗？是的，它们可以(在一定范围内它们可以)，但不是没有代价的。通用的解决方案意味着更 高的复杂性，而对于 Boost 的智能指针，可用性比灵活性具有更高的优先级。但是，通过对可定制删除器的支持，Boost 的最智能的智能指针(`boost::shared_ptr`)可以支持那些不是使用`delete`进行析构的资源。`Boost.Smart_ptr`的五个智能指针类型是专门特制的，适用于每天的编程中最常见的需求。

> [1] 因为泛型智能指针可以处理任何类型的资源。

# 何时我们需要智能指针？

## 何时我们需要智能指针？

有三种典型的情况适合使用智能指针：

*   资源所有权的共享

*   要编写异常安全的代码时

*   避免常见的错误，如资源泄漏

共享所有权是指两个或多个对象 需要同时使用第三个对象的情况。这第三个对象应该如何(或者说何时)被释放？为了确保释放的时机是正确的，每个使用这个共享资源的对象必须互相知道对方， 才能准确掌握资源的释放时间。从设计或维护的观点来看，这种耦合是不可行的。更好的方法是让这些资源所有者将资源的生存期管理责任委派给一个智能指针。当 没有共享者存在时，智能指针就可以安全地释放这个资源了。

异常安全，简单地说就是在异常 抛出时没有资源泄漏并保证程序状态的一致性。如果一个对象是动态分配的，当异常抛出时它不会自动被删除。由于栈展开以及指针离开作用域，资源可以会泄漏直 至程序结束(即使是程序结束时的资源回收也不是由语言所保证的)。不仅可能程序会由于内存泄漏而耗尽资源，程序的状态也可能变得混乱。智能指针可以自动地 为你释放这些资源，即使是在异常发生的情况下。

避免常见的错误。忘记调用 `delete` 是书本中最古老的错误(至少在这本书中)。一个智能指针不关心程序中的控制路径；它只关心在它所指向的对象的生存期结束时删除它。使用智能指针，你不再需要知道何时删除对象。并且，智能指针隐藏了释放资源的细节，因此使用者不需要知道是否要调用 `delete`, 有些特殊的清除函数并不总是删除资源的。

安全和高效的智能指针是程序员的军火库中重要的武器。虽然 C++标准库中提供了 `std::auto_ptr`, 但是它不能完全满足我们对智能指针的需求。例如，`auto_ptr`不能用作 STL 容器的元素。Boost 的智能指针类填充了标准所留下来的缺口。

本章主要关注 scoped_ptr, shared_ptr, intrusive_ptr, 和 weak_ptr. 虽然剩下的 `scoped_array` 和 `shared_array` 有时候也很有用，但它们用的不是很多，而且它们与已讨论的非常相近，这里就不重复讨论它们了。

# Smart_ptr 如何适应标准库？

## Smart_ptr 如何适应标准库？

Smart_ptr 库已被提议包含进标准库中，主要有以下三个原因：

*   标准库现在只提供了一个`auto_ptr`, 它仅是一类智能指针，仅仅覆盖了智能指针族谱中的一个部分。`shared_ptr` 提供了不同的，也是更重要的功能。

*   Boost 的智能指针专门为了与标准库良好合作而设计，并可作为标准库的自然扩充。例如，在 `shared_ptr`之前，还没有一个标准的智能指针可用作容器的元素。

*   长久以来，现实世界中的程序员已经在他们的程序中大量使用这些智能指针类，它们已经得到了充分的验证。

以上原因使得 Smart_ptr 库成为了 C++标准库的一个非常有用的扩充。Boost.Smart_ptr 的 `shared_ptr` (以及随同的助手 `enable_shared_from_this`) 和 `weak_ptr` 已被收入即将发布的 Library Technical Report。

# scoped_ptr

## scoped_ptr

### 头文件: `"boost/scoped_ptr.hpp"`

`boost::scoped_ptr` 用于确保动态分配的对象能够被正确地删除。`scoped_ptr` 有着与`std::auto_ptr`类似的特性，而最大的区别在于它不能转让所有权而`auto_ptr`可以。事实上，`scoped_ptr`永远不能被复制或被赋值！`scoped_ptr` 拥有它所指向的资源的所有权，并永远不会放弃这个所有权。`scoped_ptr`的这种特性提升了我们的代码的表现，我们可以根据需要选择最合适的智能指针(`scoped_ptr` 或 `auto_ptr`)。

要决定使用`std::auto_ptr`还是`boost::scoped_ptr`, 就要考虑转移所有权是不是你想要的智能指针的一个特性。如果不是，就用`scoped_ptr`. 它是一种轻量级的智能指针；使用它不会使你的程序变大或变慢。它只会让你的代码更安全，更好维护。

下面是`scoped_ptr`的摘要，以及其成员的简要描述：

```cpp
namespace boost {

  template<typename T> class scoped_ptr : noncopyable {
  public:
    explicit scoped_ptr(T* p = 0); 
    ~scoped_ptr(); 

    void reset(T* p = 0); 

    T& operator*() const; 
    T* operator->() const; 
    T* get() const; 

    void swap(scoped_ptr& b); 
  };

  template<typename T> 
    void swap(scoped_ptr<T> & a, scoped_ptr<T> & b); 
} 
```

### 成员函数

```cpp
explicit scoped_ptr(T* p=0) 
```

构造函数，存储`p`的一份拷贝。注意，`p` 必须是用`operator new`分配的，或者是 null. 在构造的时候，不要求`T`必须是一个完整的类型。当指针`p`是调用某个分配函数的结果而不是直接调用`new`得到的时候很有用：因为这个类型不必是完整的，只需要类型`T`的一个前向声明就可以了。这个构造函数不会抛出异常。

```cpp
~scoped_ptr() 
```

删除被指物。类型`T`在被销毁时必须是一个完整的类型。如果`scoped_ptr`在它被析构时并没有保存资源，它就什么都不做。这个析构函数不会抛出异常。

```cpp
void reset(T* p=0); 
```

重置一个 `scoped_ptr` 就是删除它已保存的指针，如果它有的话，并重新保存 `p`. 通常，资源的生存期管理应该完全由`scoped_ptr`自己处理，但是在极少数时候，资源需要在`scoped_ptr`的析构之前释放，或者`scoped_ptr`要处理它原有资源之外的另外一个资源。这时，就可以用`reset`，但一定要尽量少用它。(过多地使用它通常表示有设计方面的问题) 这个函数不会抛出异常。

```cpp
T& operator*() const; 
```

返回一个到被保存指针指向的对象的引用。由于不允许空的引用，所以解引用一个拥有空指针的`scoped_ptr`将导致未定义行为。如果不能肯定所含指针是否有效，就用函数`get`替代解引用。这个函数不会抛出异常。

```cpp
T* operator->() const; 
```

返回保存的指针。如果保存的指针为空，则调用这个函数会导致未定义行为。如果不能肯定指针是否空的，最好使用函数`get`。这个函数不会抛出异常。

```cpp
T* get() const; 
```

返回保存的指针。应该小心地使用`get`，因为它可以直接操作裸指针。但是，`get`使得你可以测试保存的指针是否为空。这个函数不会抛出异常。`get`通常在调用那些需要裸指针的函数时使用。

```cpp
operator unspecified_bool_type() const 
```

返回`scoped_ptr`是否为非空。返回值的类型是未指明的，但这个类型可被用于 Boolean 的上下文中。在 if 语句中最好使用这个类型转换函数，而不要用`get`去测试`scoped_ptr`的有效性

```cpp
void swap(scoped_ptr& b) 
```

交换两个`scoped_ptr`的内容。这个函数不会抛出异常。

### 普通函数

```cpp
template<typename T> void swap(scoped_ptr<T>& a,scoped_ptr<T>& b) 
```

这个函数提供了交换两个 scoped pointer 的内容的更好的方法。之所以说它更好，是因为 `swap(scoped1,scoped2)` 可以更广泛地用于很多指针类型，包括裸指针和第三方的智能指针。[2] `scoped1.swap(scoped2)` 则只能用于它的定义所在的智能指针，而不能用于裸指针。

> [2] 你可为那些不够智能，没有提供它们自己的交换函数的智能指针创建你的普通 swap 函数。

### 用法

`scoped_ptr`的用法与普通的指针没什么区别；最大的差别在于你不必再记得在指针上调用`delete`，还有复制是不允许的。典型的指针操作(`operator*` 和 `operator-&gt;`)都被重载了，并提供了和裸指针一样的语法。用`scoped_ptr`和用裸指针一样快，也没有大小上的增加，因此它们可以广泛使用。使用`boost::scoped_ptr`时，包含头文件`"boost/scoped_ptr.hpp"`. 在声明一个`scoped_ptr`时，用被指物的类型来指定类模板的参数。例如，以下是一个包含`std::string`指针的`scoped_ptr`：

```cpp
boost::scoped_ptr<std::string> p(new std::string("Hello")); 
```

当`scoped_ptr`被销毁时，它对它所拥有的指针调用`delete` 。

### 不需要手工删除

让我们看一个程序，它使用`scoped_ptr`来管理`std::string`指针。注意这里没有对`delete`的调用，因为`scoped_ptr`是一个自动变量，它会在离开作用域时被销毁。

```cpp
#include "boost/scoped_ptr.hpp"
#include <string>
#include <iostream>

int main() {
  {
  boost::scoped_ptr<std::string> 
  p(new std::string("Use scoped_ptr often."));

  // 打印字符串的值
  if (p)
    std::cout << *p << '\n';

  // 获取字符串的大小
  size_t i=p->size();

  // 给字符串赋新值
  *p="Acts just like a pointer";

  } // 这里 p 被销毁，并删除 std::string 
} 
```

这段代码中有几个地方值得注明一下。首先，`scoped_ptr`可以测试其有效性，就象一个普通指针那样，因为它提供了隐式转换到一个可用于布尔表达式的类型的方法。其次，可以象使用裸指针那样调用被指物的成员函数，因为重载了`operator-&gt;`. 第三，也可以和裸指针一样解引用`scoped_ptr`，这归功于`operator*`的重载。这些特性正是`scoped_ptr`和其它智能指针的用处所在，因为它们和裸指针的不同之处在于对生存期管理的语义上，而不在于语法上。

### 和 auto_ptr 几乎一样

`scoped_ptr` 与 `auto_ptr`间的区别主要在于对拥有权的处理。`auto_ptr`在复制时会从源`auto_ptr`自动交出拥有权，而`scoped_ptr`则不允许被复制。看看下面这段程序，它把`scoped_ptr` 和 `auto_ptr`放在一起，你可以清楚地看到它们有什么不同。

```cpp
void scoped_vs_auto() {

  using boost::scoped_ptr;
  using std::auto_ptr;

  scoped_ptr<std::string> p_scoped(new std::string("Hello"));
  auto_ptr<std::string> p_auto(new std::string("Hello"));

  p_scoped->size();
  p_auto->size();

  scoped_ptr<std::string> p_another_scoped=p_scoped;
  auto_ptr<std::string> p_another_auto=p_auto;

  p_another_auto->size();
  (*p_auto).size();
} 
```

这个例子不能通过编译，因为`scoped_ptr`不能被复制构造或被赋值。`auto_ptr`既可以复制构造也可以赋值，但这们同时也意味着它把所有权从`p_auto` 转移给了 `p_another_auto`, 在赋值后`p_auto`将只剩下一个空指针。这可能会导致令人不快的惊讶，就象你试图把`auto_ptr`放入容器内时所发生的那样。[3] 如果我们删掉对`p_another_scoped`的赋值，程序就可以编译了，但它的运行结果是不可预测的，因为它解引用了`p_auto`里的空指针`(*p_auto)`.

> [3] 永远不要把`auto_ptr`放入标准库的容器里。如果你试一下，通常你会得到一个编译错误；如果你没有得到错误，你就麻烦了。

由于`scoped_ptr::get`会返回一个裸指针，所以就有可能对`scoped_ptr`做一些有害的事情，其中有两件是你尤其要避免的。第一，不要删除这个裸指针。因为它会在`scoped_ptr`被销毁时再一次被删除。第二，不要把这个裸指针保存到另一个`scoped_ptr` (或其它任何的智能指针)里。因为这样也会两次删除这个指针，每个`scoped_ptr`一次。简单地说，尽量少用`get`, 除非你要使用那些要求你传送裸指针的遗留代码！

### scoped_ptr 和 Pimpl 用法

`scoped_ptr`可以很好地用于许多以前使用裸指针或`auto_ptr`的地方，如在实现 pimpl 用法时。[4]pimpl 用法背后的思想是把客户与所有关于类的私有部分的知识分隔开。由于客户是依赖于类的头文件的，头文件中的任何变化都会影响客户，即使仅是对私有节或保护节 的修改。pimpl 用法隐藏了这些细节，方法是将私有数据和函数放入一个单独的类中，并保存在一个实现文件中，然后在头文件中对这个类进行前向声明并保存 一个指向该实现类的指针。类的构造函数分配这个 pimpl 类，而析构函数则释放它。这样可以消除头文件与实现细节的相关性。我们来构造一个实现 pimpl 用法的类，然后用智能指针让它更为安全。

> [4] 这也被称为 Cheshire Cat 用法. 关于 pimpl 用法更多的说明请见 [www.gotw.ca/gotw/024.htm](http://www.gotw.ca/gotw/024.htm) 和 Exceptional C++ 。

```cpp
// pimpl_sample.hpp

#if !defined (PIMPL_SAMPLE)
#define PIMPL_SAMPLE

class pimpl_sample {
  struct impl;  // 译者注：原文中这句在 class 之外，与下文的实现代码有矛盾
  impl* pimpl_;
public:
  pimpl_sample();
  ~pimpl_sample();
  void do_something();
};

#endif 
```

这是`pimpl_sample`类的接口。`struct impl` 是一个前向声明，它把所有私有成员和函数放在另一个实现文件中。这样做的效果是使客户与`pimpl_sample`类的内部细节完全隔离开来。

```cpp
// pimpl_sample.cpp 

#include "pimpl_sample.hpp"
#include <string>
#include <iostream>

struct pimpl_sample::impl {
  void do_something_() {
    std::cout << s_ << "\n";
  }

  std::string s_;
};

pimpl_sample::pimpl_sample()
  : pimpl_(new impl) {
  pimpl_->s_ = "This is the pimpl idiom";
}

pimpl_sample::~pimpl_sample() {
  delete pimpl_;
}

void pimpl_sample::do_something() {
  pimpl_->do_something_();
} 
```

看起来很完美，但并不是的。这个实现不是异常安全的！原因是`pimpl_sample`的构造函数有可能在`pimpl`被构造后抛出一个异常。在构造函数中抛出异常意味着已构造的对象并不存在，因此在栈展开时将不会调用它的析构函数。这样就意味着分配给`pimpl_`指针的内存将泄漏。然而，有一样简单的解决方法：用`scoped_ptr`来解救！

```cpp
class pimpl_sample {
  struct impl;
  boost::scoped_ptr<impl> pimpl_;
  ...
}; 
```

让`scoped_ptr`来处理隐藏类`impl`的生存期管理，并从析构函数中去掉对`impl`的删除(它不再需要，这要感谢`scoped_ptr`)，这样就做完了。但是，你必须记住要手工定义析构函数；原因是在编译器生成隐式析构函数时，类`impl`还是不完整的，所以它的析构函数不能被调用。如果你用`auto_ptr`来保存`impl`, 你可以编译，但也还是有这个问题，但如果用`scoped_ptr`, 你将收到一个错误提示。

要注意的是，如果你使用`scoped_ptr`作为一个类的成员，你就必须手工定义这个类的复制构造函数和赋值操作符。原因是`scoped_ptr`是不能复制的，因此聚集了它的类也变得不能复制了。

最后一点值得注意的是，如果 pimpl 实例可以安全地被多个封装类(在这里是`pimpl_sample`)的实例所共享，那么用`boost::shared_ptr`来管理 pimpl 的生存期才是正确的选择。用`shared_ptr`比用`scoped_ptr`的优势在于，不需要手工去定义复制构造函数和赋值操作符，而且可以定义空的析构函数，`shared_ptr`被设计为可以正确地用于未完成的类。

### scoped_ptr 不同于 const auto_ptr

留心的读者可能已经注意到`auto_ptr`可以几乎象`scoped_ptr`一样地工作，只要把`auto_ptr`声明为`const`：

```cpp
const auto_ptr<A> no_transfer_of_ownership(new A); 
```

它们很接近，但不是一样。最大的区别在于`scoped_ptr`可以被`reset`, 在需要时可以删除并替换被指物。而对于`const auto_ptr`这是不可能的。另一个小一点的区别是，它们的名字不同：尽管`const auto_ptr`意思上和`scoped_ptr`一样，但它更冗长，也更不明显。当你的词典里有了`scoped_ptr`，你就应该使用它，因为它可以更清楚地表明你的意图。如果你想说一个资源是要被限制在作用域里的，并且不应该有办法可以放弃它的所有权，你就应该用 `boost::scoped_ptr`.

### 总结

使用裸指针来写异常安全和无错误的代码是很复杂的。使用智能指针来自动地把动态分配对象的生存期限制在一个明确的范围之内，是解决这种问题的一个有效方法，并且提高了代码的可读性、可维护性和质量。`scoped_ptr` 明确地表示被指物不能被共享和转移。正如你所看到的，`std::auto_ptr`可以从另一个`auto_ptr`那里窃取被指物，那怕是无意的，这被认为是`auto_ptr`的最大缺点。正是这个缺点使得`scoped_ptr`成为`auto_ptr`最好的补充。当一个动态分配的对象被传送给`scoped_ptr`, 它就成为了这个对象的唯一的拥有者。因为`scoped_ptr`几乎总是以自动变量或数据成员来分配的，因此它可以在离开作用域时正确地销毁对象，从而在执行流由于返回语句或异常抛出而离开作用域时，也总能释放它所管理的内存。

在以下情况时使用 `scoped_ptr` ：

*   在可能有异常抛出的作用域里使用指针

*   函数里有几条控制路径

*   动态分配对象的生存期应被限制于特定的作用域内

*   异常安全非常重要时(总应如此!)

# scoped_array

## scoped_array

### 头文件: `"boost/scoped_array.hpp"`

需要动态分配数组时，通常最好用`std::vector`来实现，但是有两种情形看起来用数组更适合： 一种是为了优化，用`vector`多少有一些额外的内存和速度开销；另一种是为了某种原因，要求数组的大小必须是固定的。[5] 动态分配的数组会遇到与普通指针一样的危险，并且还多了一个(也是最常见的一个)，那就是错误调用`delete`操作符而不是`delete[]`操作符来释放数组。我曾经在你想象不到的地方见到过这个错误，那也是它常被用到的地方，就是在你自己实现的容器类里！`scoped_array` 为数组做了`scoped_ptr`为单个对象指针所做的事情：它负责释放内存。区别只在于`scoped_array` 是用`delete[]` 操作符来做这件事的。

> [5] 如果没有非常清晰的优点，最好还是用 `std::vector` ，除非性能测试表明`scoped_array` 的好处是有保证的。

`scoped_array`是一个单独的类而不是`scoped_ptr`的一个特化，其原因是，因为不可能用元编程技术来区分指向单个对象的指针和指向数组的指针。不管如何努力，也没有人能发现一种可靠的方法，因为数组太容易退化为指针了，这使得没有类型信息可以表示它们是指向数组的。结果，只能由你来负责，使用`scoped_array`而不是`scoped_ptr`，就如你必须用`delete[]`操作符而不是用`delete`操作符一样。这样的好处是`scoped_array` 负责为你处理释放内存的事情，而你则告诉`scoped_array` 我们要处理的是数组，而不是裸指针。

`scoped_array`与`scoped_ptr`非常相似，不同的是它提供了`operator[]` 来模仿一个裸数组。

`scoped_array` 是比普通的动态分配数组更好用。它处理了动态分配数组的生存期管理问题，就如`scoped_ptr`管理对象指针的生存期一样。但是记住，多数情况下应该使用`std::vector`，它更灵活、更强大。只有当你需要确保数组的大小是固定的时候，才使用`scoped_array` 来替代 `std::vector`.

# shared_ptr

## shared_ptr

### 头文件: `"boost/shared_ptr.hpp"`

几乎所有稍微复杂点的程序都需要某种形式的引用计数智能指针。这些智能指针让我们不再需要为了管理被两个或多个对象共享的对象的生存期而编写复杂的逻辑。当引用计数降为零，没有对象再需要这个共享的对象时，这个对象就自动被销毁了。引用计数智能指针可以分为插入式(intrusive)和非插入式(non-intrusive)两 类。前者要求它所管理的类提供明确的函数或数据成员用于管理引用计数。这意味着在类的设计时就必须预见到它将与一个插入式的引用计数智能指针一起工作，或 者重新设计它。非插入式的引用计数智能指针对它所管理的类没有任何要求。引用计数智能指针拥有与它所存指针有关的内存的所有权。没有智能指针的帮助，对象 的共享会存在问题，必须有人负负责删除共享的内存。谁负责？什么时候删除？没有智能指针，你必须在管理的内存之外增加生存期的管理，这意味着在各个拥有者 之间存在更强的依赖关系。换言之，没有了重用性并增加了复杂性。

被管理的类可能拥有一些特性使得它更应该与引用计数智能指针一起使用。例如，它的复制操作很昂贵，或 者它所代表的有些东西必须被多个实例共享，这些特性都值得去共享所有权。还有一种情形是共享的资源没有一个明确的拥有者。使用引用计数智能指针可以在需要 访问共享资源的对象之间共享资源的所有权。引用计数智能指针还让你可以把对象指针存入标准库的容器中而不会有泄漏的风险，特别是在面对异常或要从容器中删 除元素的时候。如果你把指针放入容器，你就可以获得多态的好处，可以提高性能(如果复制的代价很高的话)，还可以通过把相同的对象放入多个辅助容器来进行 特定的查找。

在你决定使用引用计数智能指针后，你应该选择插入式的还是非插入式的？非插入式智能指针几乎总是更好 的选择，由于它们的通用性、不需要修改已有代码，以及灵活性。你可以对你不能或不想修改的类使用非插入式的引用计数智能指针。而把一个类修改为使用插入式 引用计数智能指针的常见方法是从一个引用计数基类派生。这种修改可能比你想象的更昂贵。至少，它增加了相关性并降低了重用性。[6] 它还增加了对象的大小，这在一些特定环境中可能会限制其可用性。[7]

> [6] 考虑一下对同一个类型使用两个以上引用计数智能指针的需要。如果两个都是插入式的，两个不同的基类可能会不兼容，而且也很浪费。如果其中一个是插入式的，那么使用非插入式的智能指针可以使基类的额外负担为零。
> 
> [7] 另一方面，非插入式智能指针要求额外的存储用于智能指针本身。

`shared_ptr` 可以从一个裸指针、另一个`shared_ptr`、一个`std::auto_ptr`、或者一个`boost::weak_ptr`构造。还可以传递第二个参数给`shared_ptr`的构造函数，它被称为删除器(deleter)。删除器稍后会被调用，来处理共享资源的释放。这对于管理那些不是用`new`分配也不是用`delete`释放的资源时非常有用(稍后将看到创建客户化删除器的例子)。`shared_ptr`被创建后，它就可象普通指针一样使用了，除了一点，它不能被显式地删除。

以下是 shared_ptr 的部分摘要；最重要的成员和相关普通函数被列出，随后是简单的讨论。

```cpp
namespace boost {

  template<typename T> class shared_ptr {
  public:
    template <class Y> explicit shared_ptr(Y* p);
    template <class Y,class D> shared_ptr(Y* p,D d);

    ~shared_ptr();

    shared_ptr(const shared_ptr & r);
    template <class Y> explicit 
      shared_ptr(const weak_ptr<Y>& r);
    template <class Y> explicit shared_ptr(std::auto_ptr<Y>& r);

    shared_ptr& operator=(const shared_ptr& r);

    void reset(); 

    T& operator*() const;
    T* operator->() const;
    T* get() const;

    bool unique() const;
    long use_count() const;

    operator unspecified_bool_type() const;  //译注：原文是 unspecified-bool-type()，有误

    void swap(shared_ptr<T>& b);
  };

  template <class T,class U>
    shared_ptr<T> static_pointer_cast(const shared_ptr<U>& r);
} 
```

### 成员函数

```cpp
template <class Y> explicit shared_ptr(Y* p); 
```

这个构造函数获得给定指针`p`的所有权。参数 `p` 必须是指向 `Y` 的有效指针。构造后引用计数设为 1。唯一从这个构造函数抛出的异常是`std::bad_alloc` (仅在一种很罕见的情况下发生，即不能获得引用计数器所需的自由空间)。

```cpp
template <class Y,class D> shared_ptr(Y* p,D d); 
```

这个构造函数带有两个参数。第一个是`shared_ptr`将要获得所有权的那个资源，第二个是`shared_ptr`被销毁时负责释放资源的一个对象，被保存的资源将以`d(p)`的形式传给那个对象。因此`p`的值是否有效取决于`d`。如果引用计数器不能分配成功，`shared_ptr`抛出一个类型为`std::bad_alloc`的异常。

```cpp
shared_ptr(const shared_ptr& r); 
```

`r`中保存的资源被新构造的`shared_ptr`所共享，引用计数加一。这个构造函数不会抛出异常。

```cpp
template <class Y> explicit shared_ptr(const weak_ptr<Y>& r); 
```

从一个 weak_ptr (本章稍后会介绍)构造 shared_ptr。这使得`weak_ptr`的使用具有线程安全性，因为指向`weak_ptr`参数的共享资源的引用计数将会自增(`weak_ptr`不影响共享资源的引用计数)。如果`weak_ptr`为空 (`r.use_count()==0`), `shared_ptr` 抛出一个类型为`bad_weak_ptr`的异常。

```cpp
template <typename Y> shared_ptr(std::auto_ptr<Y>& r); 
```

这个构造函数从一个`auto_ptr`获取`r`中保存的指针的所有权，方法是保存指针的一份拷贝并对`auto_ptr`调用`release`。构造后的引用计数为 1。而`r`当然就变为空的。如果引用计数器不能分配成功，则抛出 `std::bad_alloc` 。

```cpp
~shared_ptr(); 
```

`shared_ptr`析构函数对引用计数减一。如果计数为零，则保存的指针被删除。删除指针的方法是调用`operator delete` 或者，如果给定了一个执行删除操作的客户化删除器对象，就把保存的指针作为唯一参数调用这个对象。析构函数不会抛出异常。

```cpp
shared_ptr& operator=(const shared_ptr& r); 
```

赋值操作共享`r`中的资源，并停止对原有资源的共享。赋值操作不会抛出异常。

```cpp
void reset(); 
```

`reset`函数用于停止对保存指针的所有权的共享。共享资源的引用计数减一。

```cpp
T& operator*() const; 
```

这个操作符返回对已存指针所指向的对象的一个引用。如果指针为空，调用`operator*` 会导致未定义行为。这个操作符不会抛出异常。

```cpp
T* operator->() const; 
```

这个操作符返回保存的指针。这个操作符与`operator*`一起使得智能指针看起来象普通指针。这个操作符不会抛出异常。

```cpp
T* get() const; 
```

`get`函数是当保存的指针有可能为空时(这时 `operator*` 和 `operator-&gt;` 都会导致未定义行为)获取它的最好办法。注意，你也可以使用隐式布尔类型转换来测试 `shared_ptr` 是否包含有效指针。这个函数不会抛出异常。

```cpp
bool unique() const; 
```

这个函数在`shared_ptr`是它所保存指针的唯一拥有者时返回 `true` ；否则返回 `false`。 `unique` 不会抛出异常。

```cpp
long use_count() const; 
```

`use_count` 函数返回指针的引用计数。它在调试的时候特别有用，因为它可以在程序执行的关键点获得引用计数的快照。小心地使用它，因为在某些可能的`shared_ptr`实现中，计算引用计数可能是昂贵的，甚至是不行的。这个函数不会抛出异常。

```cpp
operator unspecified-bool-type() const; 
```

这是个到`unspecified-bool-type`类型的隐式转换函数，它可以在 Boolean 上下文中测试一个智能指针。如果`shared_ptr`保存着一个有效的指针，返回值为`True`；否则为`false`。注意，转换函数返回的类型是不确定的。把返回类型当成`bool`用会导致一些荒谬的操作，所以典型的实现采用了 safe bool idiom,[8] 它很好地确保了只有可适用的 Boolean 测试可以使用。这个函数不会抛出异常。

> [8] 由 Peter Dimov 发明的。

```cpp
void swap(shared_ptr<T>& b); 
```

这可以很方便地交换两个`shared_ptr`。`swap` 函数交换保存的指针(以及它们的引用计数)。这个函数不会抛出异常。

### 普通函数

```cpp
template <typename T,typename U>
  shared_ptr<T> static_pointer_cast(const shared_ptr<U>& r); 
```

要对保存在`shared_ptr`里的指针执行`static_cast`，我们可以取出指针然后强制转换它，但我们不能把它存到另一个`shared_ptr`里；新的 `shared_ptr` 会认为它是第一个管理这些资源的。解决的方法是用 `static_pointer_cast`. 使用这个函数可以确保被指物的引用计数保持正确。`static_pointer_cast` 不会抛出异常。

### 用法

使用`shared_ptr`解决的主要问题是知道删除一个被多个客户共享的资源的正确时机。下面是一个简单易懂的例子，有两个类 `A` 和 `B`, 它们共享一个`int`实例。使用 `boost::shared_ptr`, 你需要必须包含 `"boost/shared_ptr.hpp"`.

```cpp
#include "boost/shared_ptr.hpp"
#include <cassert>

class A {
  boost::shared_ptr<int> no_;
public:
  A(boost::shared_ptr<int> no) : no_(no) {}
  void value(int i) {
    *no_=i;
  }
};

class B {
  boost::shared_ptr<int> no_;
public:
  B(boost::shared_ptr<int> no) : no_(no) {}
  int value() const {
    return *no_;
  }
};

int main() {
    boost::shared_ptr<int> temp(new int(14));
    A a(temp);
    B b(temp);
    a.value(28);
    assert(b.value()==28);
} 
```

类 `A` 和 `B`都保存了一个 `shared_ptr&lt;int&gt;`. 在创建 `A` 和 `B`的实例时，`shared_ptr temp` 被传送到它们的构造函数。这意味着共有三个 `shared_ptr`：`a`, `b`, 和 `temp`，它们都引向同一个`int`实例。如果我们用指针来实现对一个的共享，`A` 和 `B` 必须能够在某个时间指出这个`int`要被删除。在这个例子中，直到`main`的结束，引用计数为 3，当所有 `shared_ptr`离开了作用域，计数将达到 0，而最后一个智能指针将负责删除共享的 `int`.

### 回顾 Pimpl 用法

前一节展示了使用`scoped_ptr`的 pimpl 用法，如果使用这种用法的类是不允许复制的，那么`scoped_ptr`在保存 pimpl 的动态分配实例时它工作得很好。但是这并不适合于所有想从 pimpl 用法中获益的类型(注意，你还可以用 `scoped_ptr`，但必须手工实现复制构造函数和赋值操作符)。对于那些可以处理共享的实现细节的类，应该用 `shared_ptr`。当 pimpl 的所有权被传递给一个 `shared_ptr`, 复制和赋值操作都是免费的。你可以回忆起，当使用 `scoped_ptr` 去处理 pimpl 类的生存期时，对封装类的复制是不允许的，因为 `scoped_ptr`是不可复制的。这意味着要使这些类支持复制和赋值，你必须手工定义复制构造函数和赋值操作符。当使用 `shared_ptr` 去处理 pimpl 类的生存期时，就不再需要用户自己定义复制构造函数了。注意，这时 pimpl 实例是被该类的多个对象所共享，因此如果规则是每个 pimpl 实例只能被类的一个实例使用，你还是要手工编写复制构造函数。解决的方法和我们在`scoped_ptr`那看到的很相似，只是把`scoped_ptr`换成了`shared_ptr`。

### shared_ptr 与标准库容器

把对象直接存入容器中有时会有些麻烦。以值的方式保存对象意味着使用者将获得容器中的元素的拷贝，对于那些复制是一种昂贵的操作的类型来说可能会有性能的问题。此外，有些容器，特别是 `std::vector`, 当你加入元素时可能会复制所有元素，这更加重了性能的问题。最后，传值的语义意味着没有多态的行为。如果你需要在容器中存放多态的对象而且你不想切割它 们，你必须用指针。如果你用裸指针，维护元素的完整性会非常复杂。从容器中删除元素时，你必须知道容器的使用者是否还在引用那些要删除的元素，不用担心多 个使用者使用同一个元素。这些问题都可以用`shared_ptr`来解决。

下面是如何把共享指针存入标准库容器的例子。

```cpp
#include "boost/shared_ptr.hpp"
#include <vector>
#include <iostream>

class A {
public:
  virtual void sing()=0;
protected:
  virtual ~A() {};
};

class B : public A {
public:
  virtual void sing() {
    std::cout << "Do re mi fa so la";
  }
};

boost::shared_ptr<A> createA() {
  boost::shared_ptr<A> p(new B());
  return p;
}

int main() {
  typedef std::vector<boost::shared_ptr<A> > container_type;
  typedef container_type::iterator iterator;

  container_type container;
  for (int i=0;i<10;++i) {
    container.push_back(createA());
  }

  std::cout << "The choir is gathered: \n";
  iterator end=container.end();
  for (iterator it=container.begin();it!=end;++it) {
    (*it)->sing();
  }
} 
```

这里有两个类, `A` 和 `B`, 各有一个虚拟成员函数 `sing`. `B` 从 `A`公有继承而来，并且如你所见，工厂函数 `createA` 返回一个动态分配的`B`的实例，包装在`shared_ptr&lt;A&gt;`里。在 `main`里, 一个包含`shared_ptr&lt;A&gt;`的 `std::vector` 被放入 10 个元素，最后对每个元素调用`sing`。如果我们用裸指针作为元素，那些对象需要被手工删除。而在这个例子里，删除是自动的，因为在`vector`的生存期中，每个`shared_ptr`的引用计数都保持为 1；当 `vector` 被销毁，所有引用计数器都将变为零，所有对象都被删除。有趣的是，即使 `A` 的析构函数没有声明为 `virtual`, `shared_ptr` 也会正确调用 `B`的析构函数！

上面的例子示范了一个强有力的技术，它涉及`A`里面的 protected 析构函数。因为函数 `createA` 返回的是 `shared_ptr&lt;A&gt;`, 因此不可能对`shared_ptr::get`返回的指针调用 `delete` 。这意味着如果为了向某个需要裸指针的函数传送裸指针而从`shared_ptr`中取出裸指针的话，它不会由于意外地被删除而导致灾难。那么，又是如何允许 `shared_ptr` 删除它的对象的呢？ 这是因为指针指向的真正类型是 `B`; 而`B`的析构函数不是 protected 的。这是非常有用的方法，用于给`shared_ptr`中的对象增加额外的安全性。

### shared_ptr 与其它资源

有时你会发现你要把`shared_ptr`用于某个特别的类型，它需要其它清除操作而不是简单的 `delete`. `shared_ptr`可以通过客户化删除器来支持这种需要。那些处理象 `FILE*`这样的操作系统句柄的资源通常要使用象`fclose`这样的操作来释放。要在`shared_ptr`里使用 `FILE*` ，我们要定义一个类来负责释放相应的资源。

```cpp
class FileCloser {
public:
   void operator()(FILE* file) {
    std::cout << "The FileCloser has been called with a FILE*, "
      "which will now be closed.\n";
    if (file!=0) 
      fclose(file);
  }
}; 
```

这是一个函数对象，我们用它来确保在资源要释放时调用 `fclose` 。下面是使用`FileCloser`类的示例程序。

```cpp
int main() {
  std::cout << 
    "shared_ptr example with a custom deallocator.\n"; 
  {
    FILE* f=fopen("test.txt","r");
    if (f==0) {
      std::cout << "Unable to open file\n";
      throw "Unable to open file";
    }

    boost::shared_ptr<FILE> 
      my_shared_file(f, FileCloser());

    // 定位文件指针
    fseek(my_shared_file.get(),42,SEEK_SET);
  }
  std::cout << "By now, the FILE has been closed!\n";
} 
```

注意，在访问资源时，我们需要对`shared_ptr`使用 `&*` 用法, `get`, 或 `get_pointer`。(请注意最好使用 `&*`. 另两个选择不太清晰) 这个例子还可以更简单，如果我们在释放资源时只需要调用一个单参数函数的话，就根本不需要创建一个客户化删除器类型。上面的例子可以重写如下：

```cpp
{
  FILE* f=fopen("test.txt","r");
  if (f==0) {
    std::cout << "Unable to open file\n";
    throw file_exception();
  }

  boost::shared_ptr<FILE> my_shared_file(f,&fclose);

  // 定位文件指针
  fseek(&*my_shared_file,42,SEEK_SET); 
}
std::cout << "By now, the FILE* has been closed!\n"; 
```

定制删除器在处理需要特殊释放程序的资源时非常有用。由于删除器不是 `shared_ptr` 类型的一部分，所以使用者不需要知道关于智能指针所拥有的资源的任何信息(当然除了如何使用它！)。例如，你可以使用对象池，定制删除器只需简单地把对象返还到池中。或者，一个 singleton 对象应该使用一个什么都不做的删除器。

### 使用定制删除器的安全性

我们已经看到对基类使用 protected 析构函数有助于增加使用`shared_ptr`的 类的安全性。另一个达到同样安全级别的方法是，声明析构函数为 protected (或 private) 并使用一个定制删除器来负责销毁对象。这个定制删除器必须是它要删除的类的友元，这样它才可以工作。封装这个删除器的好方法是把它实现为私有的嵌套类，如 下例所示：

```cpp
#include "boost/shared_ptr.hpp"
#include <iostream>

class A {
  class deleter {
    public:
      void operator()(A* p) {
        delete p;
      }
  };
  friend class deleter;
public:

  virtual void sing() {
    std::cout << "Lalalalalalalalalalala";
  }

  static boost::shared_ptr<A> createA() {
    boost::shared_ptr<A> p(new A(),A::deleter());
    return p;
  }

protected:
  virtual ~A() {};
};

int main() {
  boost::shared_ptr<A> p=A::createA();
} 
```

注意，我们在这里不能使用普通函数来作为 `shared_ptr&lt;A&gt;` 的工厂函数，因为嵌套的删除器是`A`私有的。使用这个方法，用户不可能在栈上创建 `A`的对象，也不可能对`A`的指针调用 `delete` 。

### 从 this 创建 shared_ptr

有时候，需要从`this`获得 `shared_ptr` ，即是说，你希望你的类被`shared_ptr`所管理，你需要把"自身"转换为`shared_ptr`的方法。看起来不可能？好的，解决方案来自于我们即将讨论的另一个智能指针`boost::weak_ptr`. `weak_ptr` 是 `shared_ptr`的一个观察者；它只是安静地坐着并看着它们，但不会影响引用计数。通过存储一个指向`this`的 `weak_ptr` 作为类的成员，就可以在需要的时候获得一个指向`this`的 `shared_ptr`。为了你可以不必编写代码来保存一个指向`this`的 `weak_ptr`，接着又从`weak_ptr`获`shared_ptr`得，Boost.Smart_ptr 为这个任务提供了一个助手类，称为 `enable_shared_from_this`. 只要简单地让你的类公有地派生自 `enable_shared_from_this`，然后在需要访问管理`this`的`shared_ptr`时，使用函数 `shared_from_this` 就行了。下面的例子示范了如何使用 `enable_shared_from_this` ：

```cpp
#include "boost/shared_ptr.hpp"
#include "boost/enable_shared_from_this.hpp"

class A;

void do_stuff(boost::shared_ptr<A> p) {
  ...
}

class A : public boost::enable_shared_from_this<A> {
public:
  void call_do_stuff() {
    do_stuff(shared_from_this());
  }
};

int main() {
  boost::shared_ptr<A> p(new A());
  p->call_do_stuff();
} 
```

这个例子还示范了你要用`shared_ptr`管理`this`的情形。类 `A` 有一个成员函数 `call_do_stuff` 需要调用一个普通函数 `do_stuff`, 这个普通函数需要一个类型为 `boost:: shared_ptr&lt;A&gt;`的参数。现在，在 `A::call_do_stuff`里, `this` 不过是一个 `A`指针, 但由于 `A` 派生自 `enable_shared_from_this`, 调用 `shared_from_this` 将返回我们所要的 `shared_ptr` 。在`enable_shared_from_this`的成员函数 `shared_from_this`里，内部存储的 `weak_ptr` 被转换为 `shared_ptr`, 从而增加了相应的引用计数，以确保相应的对象不会被删除。

### 总结

引用计数智能指针是非常重要的工具。Boost 的 `shared_ptr` 提供了坚固而灵活的解决方案，它已被广泛用于多种环境下。需要在使用者之间共享对象是常见的，而且通常没有办法通知使用者何时删除对象是安全的。`shared_ptr` 让使用者无需知道也在使用共享对象的其它对象，并让它们 无需担心在没有对象引用时的资源释放。这对于 Boost 的智能指针类而言是最重要的。你会看到 Boost.Smart_ptr 中还有其它的智能指针，但这 一个肯定是你最想要的。通过使用定制删除器，几乎所有资源类型都可以存入 `shared_ptr`。这使得`shared_ptr` 成为处理资源管理的通用类，而不仅仅是处理动态分配对象。与裸指针相比，`shared_ptr`会有一点点额外的空间代价。我还没有发现由于这些代价太大而需要另外寻找一个解决方案的情形。不要去创建你自己的引用计数智能指针类。没有比使用 `shared_ptr`智能指针更好的了。

在以下情况时使用 `shared_ptr` ：

*   当有多个使用者使用同一个对象，而没有一个明显的拥有者时

*   当要把指针存入标准库容器时

*   当要传送对象到库或从库获取对象，而没有明确的所有权时

*   当管理一些需要特殊清除方式的资源时[9]

    > [9] 通过定制删除器的帮助。

# shared_array

## shared_array

### 头文件: `"boost/shared_array.hpp"`

`shared_array` 用于共享数组所有权的智能指针。它与`shared_ptr`的关系就如`scoped_array`与`scoped_ptr`的关系。`shared_array` 与 `shared_ptr` 的不同之处主要在于它是用于数组的而不是用于单个对象的。在我们讨论 scoped_array 时，我提到过通常`std::vector`是一个更好的选择。但 `shared_array` 比 `vector`更有价值，因为它提供了对数组所有权的共享。`shared_array` 的接口与 `shared_ptr`非常相似，差别仅在于增加了一个下标操作符，以及不支持定制删除器。

由于一个指向`std::vector`的`shared_ptr`提供了比 shared_array 更多的灵活性，所以我们就不对 shared_array 的用法进行讨论了。如果你发现自己需要 `boost::shared_array`, 可以参考一下在线文档。

# intrusive_ptr

## intrusive_ptr

### 头文件: `"boost/intrusive_ptr.hpp"`

`intrusive_ptr` 是`shared_ptr`的插入式版本。有时我们必须使用插入式的引用计数智能指针。典型的情况是对于那些已经写好了内部引用计数器的代码，而我们又没有时间去重写它(或者已经不能获得那些代码了)。另一种情况是要求智能指针的大小必须与裸指针大小严格相等，或者`shared_ptr`的引用计数器分配严重影响了程序的性能(我可以肯定这是非常罕见的情况！)。从功能的观点来看，唯一需要插入式智能指针的情况是，被指类的某个成员函数需要返回`this`，以便它可以用于另一个智能指针(事实上，也有办法使用非插入式智能指针来解决这个问题，正如我们在本章前面看到的)。`intrusive_ptr` 不同于其它智能指针，因为它要求你来提供它所要的引用计数器。

当 `intrusive_ptr` 递增或递减一个非空指针上的引用计数时，它是通过分别调用函数 `intrusive_ptr_add_ref` 和 `intrusive_ptr_release`来完成的。这两个函数负责确保引用计数的正确性，并且负责在引用计数降为零时删除指针。因此，你必须为你的类重载这两个函数，正如我们后面将看到的。

以下是`intrusive_ptr`的部分摘要，只列出了最重要的函数。

```cpp
namespace boost {

  template<class T> class intrusive_ptr {
  public:
    intrusive_ptr(T* p,bool add_ref=true);

    intrusive_ptr(const intrusive_ptr& r);

    ~intrusive_ptr();

    T& operator*() const;
    T* operator->() const;
    T* get() const; 

    operator unspecified-bool-type() const; 
  };

  template <class T> T* get_pointer(const intrusive_ptr<T>& p); 

  template <class T,class U> intrusive_ptr<T>
    static_pointer_cast(const intrusive_ptr<U>& r); 
} 
```

### 成员函数

```cpp
intrusive_ptr(T* p,bool add_ref=true); 
```

这个构造函数将指针`p`保存到`*this`中。如果 `p` 非空，并且 `add_ref` 为 `true`, 构造函数将调用 `intrusive_ptr_add_ref(p)`. 如果 `add_ref` 为 `false`, 构造函数则不调用 `intrusive_ptr_add_ref`. 如果`intrusive_ptr_add_ref`会抛出异常，则构造函数也会。

```cpp
intrusive_ptr(const intrusive_ptr& r); 
```

该复制构造函数保存一份`r.get()`的拷贝，并且如果指空非空则用它调用 `intrusive_ptr_add_ref` 。这个构造函数不会抛出异常。

```cpp
~intrusive_ptr(); 
```

如果保存的指针为非空，则 `intrusive_ptr` 的析构函数会以保存的指针为参数调用函数 `intrusive_ptr_release`。 `intrusive_ptr_release` 负责递减引用计数并在计数为零时删除指针。这个函数不会抛出异常。

```cpp
T& operator*() const; 
```

解引用操作符返回所存指针的解引用。如果所存指针为空则会导致未定义行为。你应该确认`intrusive_ptr`有一个非空的指针，这可以用函数 `get` 实现，或者在 Boolean 上下文中测试 `intrusive_ptr` 。解引用操作符不会抛出异常。

```cpp
T* operator->() const; 
```

这个操作符返回保存的指针。在引用的指针为空时调用这个操作符会有未定义行为。这个操作符不会抛出异常。

```cpp
T* get() const; 
```

这个成员函数返回保存的指针。它可用于你需要一个裸指针的时候，即使保存的指针为空也可以调用。这个函数不会抛出异常。

```cpp
operator unspecified-bool-type() const; 
```

这个类型转换函数返回一个可用于布尔表达式的类型，而它绝对不是 `operator bool`, 因为那样会允许一些必须要禁止的操作。这个转换允许`intrusive_ptr`在一个布尔上下文中被测试，例如，`if (p)`, `p` 是一个 `intrusive_ptr`. 这个转换函数当`intrusive_ptr`引向一个非空指针时返回`True` ; 否则返回 `false`. 这个转换函数不会抛出异常。

### 普通函数

```cpp
template <class T> T* get_pointer(const intrusive_ptr<T>& p); 
```

这个函数返回 `p.get()`, 它主要用于支持泛型编程。[10] 它也可以用作替代成员函数 `get`, 因为它可以重载为可以与裸指针或第三方智能指针类一起工作。有些人宁愿用普通函数而不用成员函数。[11] 这个函数不会抛出异常。

> [10] 这种函数被称为 shims. 见参考书目的 [12] 。
> 
> [11]这种想法是出于以下原因，使用智能指针的成员函数时，很难分清它是操作智能指针还是操作它所指向的对象。例如， `p.get()` 和 `p-&gt;get()` 有完全不同的意思，不认真看还很难区别，而 `get_pointer(p)` 和 `p-&gt;get()` 则一看就知道不一样。对于你来说这是不是问题，主要取决于你的感觉和经验。

```cpp
template <class T,class U>
  intrusive_ptr<T> static_pointer_cast(const intrusive_ptr<U>& r); 
```

这个函数返回 `intrusive_ptr&lt;T&gt;(static_cast&lt;T*&gt;(r.get()))`. 和 `shared_ptr`不一样，你可以对保存在`intrusive_ptr`中的对象指针安全地使用`static_cast`。但是你可能出于对智能指针类型转换的用法一致性而想使用这个函数。`static_pointer_cast` 不会抛出异常。

### 用法

使用`intrusive_ptr`与使用`shared_ptr`相比，有两个主要的不同之处。第一个是你需要提供引用计数的机制。第二个是把`this`当成智能指针是合法的[12]，正如我们即将看到的，有时候这样很方便。注意，在多数情况下，应该使用非插入式的 `shared_ptr`.

> [12] 你不能用`shared_ptr` 来做到这一点，如果没有进行特殊处理的话，如 `enable_shared_from_this`.

要使用 `boost::intrusive_ptr`, 要包含 `"boost/intrusive_ptr.hpp"` 并定义两个普通函数 `intrusive_ptr_add_ref` 和 `intrusive_ptr_release`. 它们都要接受一个参数，即指向你要使用`intrusive_ptr`的类型的指针。这两个函数的返回值被忽略。通常的做法是，泛化这两个函数，简单地调用被管理类型的成员函数去完成工作(例如，调用 `add_ref` 和 `release`)。如果引用计数降为零，`intrusive_ptr_release` 应该负责释放资源。以下是你应该如何实现这两个泛型函数的示范：

```cpp
template <typename T> void intrusive_ptr_add_ref(T* t) {
  t->add_ref();
}

template <typename T> void intrusive_ptr_release(T* t) {
  if (t->release()<=0)
    delete t;
} 
```

注意，这两个函数应该定义在它们的参数类型所在的作用域内。这意味着如果这个函数接受的参数类型来自 于一个名字空间，则函数也必须定义在那里。这样做的原因是，函数的调用是非受限的，即允许采用参数相关查找，而如果有多个版本的函数被提供，那么全部名字 空间肯定不是放置它们的好地方。我们稍后将看到一个关于如何放置它们的例子，但首先，我们需要提供某类的引用计数器。

### 提供一个引用计数器

现在管理用的函数已经定义了，我们必须要提供一个内部的引用计数器了。在本例中，引用计数是一个初始化为零的私有数据成员，我们将公开 `add_ref` 和 `release` 成员函数来操作它。`add_ref` 递增引用计数而 `release` 递减它[13]。 我们可以增加一个返回引用计数当前值的成员函数，但`release`也可以做到这一点。下面的基类，`reference_counter`, 提供了一个计数器以及 `add_ref` 和 `release` 成员函数，我们可以简单地用继承来为一个类增加引用计数了。

> [13] 注意，在多线程环境下，对保持引用计数的变量的任何操作都必须同步化。

```cpp
class reference_counter {
  int ref_count_;
  public:
    reference_counter() : ref_count_(0) {}

    virtual ~reference_counter() {}

    void add_ref() { 
      ++ref_count_;
    }

    int release() {
      return --ref_count_;
    }

  protected:
    reference_counter& operator=(const reference_counter&) {
    // 无操作
      return *this;
    }
  private:
    // 禁止复制构造函数
    reference_counter(const reference_counter&); 
}; 
```

把`reference_counter`的析构函数声明为虚拟的原因是这个类将被公开继承，有可能会使用一个`reference_counter`指针来`delete`派生类。我们希望删除操作能够正确地调用派生类的析构函数。实现非常简单：`add_ref` 递增引用计数，`release` 递减引用计数并返回它。要使用这个引用计数，要做的就是公共地继承它。以下是一个类 `some_ class` ，包含一个内部引用计数，并使用 `intrusive_ptr`。

```cpp
#include <iostream>
#include "boost/intrusive_ptr.hpp"

class some_class : public reference_counter {
public:
  some_class() {
    std::cout << "some_class::some_class()\n";
  }

  some_class(const some_class& other) {
    std::cout << "some_class(const some_class& other)\n";
  }

  ~some_class() {
    std::cout << "some_class::~some_class()\n";
  }
};

int main() {
  std::cout << "Before start of scope\n";
  {
    boost::intrusive_ptr<some_class> p1(new some_class());
    boost::intrusive_ptr<some_class> p2(p1);
  }
  std::cout << "After end of scope \n";
} 
```

为了显示 `intrusive_ptr`以及函数 `intrusive_ptr_add_ref` 和 `intrusive_ptr_release` 都正确无误，以下是这个程序的运行输出：

```cpp
Before start of scope
some_class::some_class()
some_class::~some_class()
After end of scope 
```

`intrusive_ptr` 为我们打点一切。当第一个 `intrusive_ptr p1` 创建时，它传送了一个`some_class`的新实例。`intrusive_ptr` 构造函数实际上有两个参数，第二个是一个 `bool` ，表示是否要调用 `intrusive_ptr_add_ref` 。由于这个参数的缺省值是 `True`, 所以在构造 `p1`时，`some_class`实例的引用计数变为 1。然后，第二个 `intrusive_ptr`, `p2`初构造。它是从 `p1`复制构造的，当 `p2` 看到 `p1` 是引向一个非空指针时，它调用 `intrusive_ptr_add_ref`. 引用计数变为 2。然后，两个 `intrusive_ptr`都离开作用域了。首先， `p2` 被销毁，析构函数调用 `intrusive_ptr_release`. 它把引用计数减为 1。然后，`p1` 被销毁，析构函数再次调用 `intrusive_ptr_release` ，导致引用计数降为 0；这使得我们的`intrusive_ptr_release` 去 `delete` 该指针。你可能注意到 `reference_counter` 的实现不是线程安全的，因此不能用于多线程应用，除非加上同步化。

比起依赖于`intrusive_ptr_add_ref` 和 `intrusive_ptr_release`的泛型实现，我们最好有一些直接操作基类(在这里是 `reference_counter`)的函数。这样做的优点在于，即使从`reference_counter`派生的类定义在其它的名字空间，`intrusive_ptr_add_ref` 和 `intrusive_ptr_release` 也还可以通过 ADL (参数相关查找法)找到它们。修改`reference_counter`的实现很简单。

```cpp
class reference_counter {
  int ref_count_;
  public:
    reference_counter() : ref_count_(0) {}

    virtual ~reference_counter() {}

     friend void intrusive_ptr_add_ref(reference_counter* p) { 
       ++p->ref_count_;
     }

     friend void intrusive_ptr_release(reference_counter* p) {
       if (--p->ref_count_==0)
         delete p;
     }

  protected:
    reference_counter& operator=(const reference_counter&) {
    // 无操作
      return *this;
    }
  private:
    // 禁止复制构造函数
    reference_counter(const reference_counter&); 
}; 
```

### 把 this 用作智能指针

总的来说，提出一定要用插入式引用计数智能指针的情形是不容易的。大多数情况下，但不是全部情况下，非插入式智能指针都可以解决问题。但是，有一种情形使用插入式引用计数会更容易：当你需要从一个成员函数返回 `this` ，并把它存入另一个智能指针。当从一个被非插入式智能指针所拥有的类型返回 `this`时， 结果是有两个不同的智能指针认为它们拥有同一个对象，这意味着它们会在某个时候一起试图删除同一个指针。这导致了两次删除，结果可能使你的应用程序崩溃。 必须有什么办法可以通知另一个智能指针，这个资源已经被一个智能指针所引用，这正好是内部引用计数器(暗地里)可以做到的。由于 intrusive_ptr 的逻辑不直接对它们所引向的对象的内部引用计数进行操作，这就不会违反所有权或引用计数的完整性。引用计数只是被简单地递增。

让我们先看一下一个依赖于`boost::shared_ptr`来共享资源所有权的实现中潜在的问题。它基于本章前面讨论`enable_shared_from_this`时的例子。

```cpp
#include "boost/shared_ptr.hpp"

class A;

void do_stuff(boost::shared_ptr<A> p) {
  // ...
}

class A {
public:
  call_do_stuff() {
   shared_ptr<A> p(???);
    do_stuff(p);
  }
};

int main() {
  boost::shared_ptr<A> p(new A());
  p->call_do_stuff();
} 
```

类`A` 要调用函数 `do_stuff`, 但问题是 `do_stuff` 要一个 `shared_ptr&lt;A&gt;`, 而不是一个普通的`A`指针。因此，在 `A::call_do_stuff`里，应该如何创建 `shared_ptr` ？现在，让我们重写 `A` ，让它兼容于 `intrusive_ptr`, 通过从 `reference_counter`派生，然后我们再增加一个 `do_stuff`的重载版本，接受一个 `intrusive_ptr&lt;A&gt;`类型的参数。

```cpp
#include "boost/intrusive_ptr.hpp"

class A;

void do_stuff(boost::intrusive_ptr<A> p) {
  // ...
}

void do_stuff(boost::shared_ptr<A> p) {
  // ...
}

class A : public reference_counter {
public:
  void call_do_stuff() {
    do_stuff(this);
  }
};

int main() {
  boost::intrusive_ptr<A> p(new A());
  p->call_do_stuff();
} 
```

如你所见，在这个版本的 `A::call_do_stuff`里，我们可以直接把 this 传给需要一个 `intrusive_ptr&lt;A&gt;`的函数，这是由于 `intrusive_ptr`的类型转换构造函数。

最后，这里有一个特别的地方：现在 `A` 可以支持 `intrusive_ptr`了，我们也可以创建一个包装`intrusive_ptr`的`shared_ptr`，这们我们就可以调用原来版本的 `do_stuff`, 它需要一个 `shared_ptr&lt;A&gt;` 作为参数。假如你不能控制 `do_stuff`的源码，这可能是你要解决的一个非常真实的问题。这次，还是用定制删除器的方法来解决，它需要调用 `intrusive_ptr_release`. 下面是一个新版本的 `A::call_do_stuff`.

```cpp
void call_do_stuff() {
  intrusive_ptr_add_ref(this);
  boost::shared_ptr<A> p(this,&intrusive_ptr_release<A>);
  do_stuff(p);
} 
```

真是一个漂亮的方法。当没有 `shared_ptr`剩下时，定制的删除器被调用，它调用 `intrusive_ptr_release`, 递减`A`的内部引用计数。注意，如果 `intrusive_ptr_add_ref` 和 `intrusive_ptr_release` 被实现为直接操作 `reference_counter`, 你就要这样来创建 `shared_ptr` ：

```cpp
boost::shared_ptr<A> p(this,&intrusive_ptr_release); 
```

### 支持不同的引用计数器

我们前面提过可以为不同的类型支持不同的引用计数。这在集成已有的采用不同引用计数机制的类时是有必要的(例如，第三方的类使用它们自己版本的引用计数器)。又或者对于资源的释放有不同的需求，如调用`delete`以外的另一个函数。如前所述，对 `intrusive_ptr_add_ref` 和 `intrusive_ptr_release` 的调用是非受限的。这意味着在名字查找时要考虑参数(指针的类型)的作用域，从而这些函数应该与它们操作的类型定义在同一个作用域。如果你在全局名字空间里实现 `intrusive_ptr_add_ref` 和 `intrusive_ptr_release` 的泛型版本，你就不能在其它名字空间中再创建泛型版本了。例如，如果一个名字空间需要为它的所有类型定义一个特殊的版本，特化版本或重载版本必须提供给每 一个类型。否则，全局名字空间中的函数就会引起歧义。因此在全局名字空间中提供泛型版本不是一个好主意，而在其它名字空间中提供则可以。

既然我们已经用基类`reference_counter`实现了引用计数器，那么在全局名字空间中提供一个接受`reference_counter*`类型的参数的普通函数应该是一个好主意。这还可以让我们在其它名字空间中提供泛型重载版本而不会引起歧义。例如，考虑`my_namespace`名字空间中的两个类 `another_class` 和 `derived_class` ：

```cpp
namespace my_namespace {
  class another_class : public reference_counter {
  public:
    void call_before_destruction() const {
      std::cout << 
        "Yes, I'm ready before destruction\n";
    }
  };

  class derived_class : public another_class {};

   template <typename T> void intrusive_ptr_add_ref(T* t) {
     t->add_ref();
   }

  template <typename T> void intrusive_ptr_release(T* t) {
    if (t->release()<=0) {
      t->call_before_destruction();
      delete t;
    }
  }
} 
```

这里，我们实现了`intrusive_ptr_add_ref` 和 `intrusive_ptr_release`的泛型版本。因此我们必须删掉在全局名字空间中的泛型版本，把它们替换为以一个`reference_counter`指针为参数的非模板版本。或者，我们干脆从全局名字空间中删掉这些函数，也可以避免引起混乱。对于这两个类 `my_namespace::another_class` 和 `my_namespace::derived_class`, 将调用这个特殊版本(那个调用了它的参数的成员函数 `call_before_destruction` 的版本)。其它类型或者在它们定义所在的名字空间中有相应的函数，或者使用全局名字空间中的版本，如果有的话。下面程序示范了这如何工作：

```cpp
int main() {
  boost::intrusive_ptr<my_namespace::another_class> 
    p1(new my_namespace::another_class());
  boost::intrusive_ptr<A> 
    p2(new good_class());
  boost::intrusive_ptr<my_namespace::derived_class> 
    p3(new my_namespace::derived_class());
} 
```

首先，`intrusive_ptr p1` 被传入一个新的 `my_namespace::another_class`实例。在解析对 `intrusive_ptr_add_ref`的调用时，编译器会找到 `my_namespace`里的版本，即 `my_namespace::another_class*` 参数所在名字空间。因而，为那个名字空间里的类型所提供的泛型函数会被正确地调用。在查找 `intrusive_ptr_release`时也是同样。然后，`intrusive_ptr p2` 被创建并被传入一个类型`A` (我们早前创建的那个类型)的指针。那个类型是在全局名字空间里的，所以当编译器试图去找到函数 `intrusive_ptr_add_ref`的最佳匹配时，它只会找到一个版本，即接受`reference_counter`指针类型的那个版本(你应该记得我们已经从全局名字空间中删掉了泛型版本)。因为 `A` 公共继承自 `reference_counter`, 通过隐式类型转换就可以进行正确的调用。最后，`my_namespace` 里的泛型版本被用于类 `my_namespace::derived_class`; 这与 `another_class`例子中的查找是一样的。

这里最重要的教训是，在实现函数 `intrusive_ptr_add_ref` 和 `intrusive_ptr_release`时，它们应该总是定义在它们操作的类型所在的名字空间里。从设计的角度来看，这也是完美的，把相关的东西放在一起，这有助于确保总是调用正确的版本，而不用担心是否有多个不同的实现可供选择。

### 总结

在多数情况下，你不应该使用 `boost::intrusive_ptr`, 因为共享所有权的功能已在 `boost::shared_ptr`中提供，而且非插入式智能指针比插入式智能指针更灵活。但是，有时候也会需要插入式的引用计数，可能是由于旧的代码，或者是为了与第三方的类进行集成。当有这种需要时，可以用 `intrusive_ptr` ，它具有与其它 Boost 智能指针相同的语义。如果你使用过其它的 Boost 智能指针，你就会发现不论是否插入式的，所有智能指针都有一致的接口。使用`intrusive_ptr`的类必须可以提供引用计数。`ntrusive_ptr` 通过调用两个函数，`intrusive_ptr_add_ref` 和 `intrusive_ptr_release`来管理引用计数；这两个函数必须正确地操作插入式的引用计数，以保证 `intrusive_ptr`正确工作。在使用`intrusive_ptr`的类中已经内置有引用计数的情况下，实现对`intrusive_ptr`的支持就是实现这两个函数。有些情况下，可以创建这两个函数的参数化版本，然后对所有带插入式引用计数的类型使用相同的实现。多数时候，声明这两个函数的最好的地方就是它们所支持的类型所在的名字空间。

在以下情况时使用 `intrusive_ptr` ：

*   你需要把 `this` 当作智能指针来使用。

*   已有代码使用或提供了插入式的引用计数。

*   智能指针的大小必须与裸指针的大小相等。

# weak_ptr

## weak_ptr

### 头文件: `"boost/weak_ptr.hpp"`

`weak_ptr` 是 `shared_ptr` 的观察员。它不会干扰`shared_ptr`所共享的所有权。当一个被`weak_ptr`所观察的 `shared_ptr` 要释放它的资源时，它会把相关的 `weak_ptr`的指针设为空。这防止了 `weak_ptr` 持有悬空的指针。你为什么会需要 `weak_ptr`? 许多情况下，你需要旁观或使用一个共享资源，但不接受所有权，如为了防止递归的依赖关系，你就要旁观一个共享资源而不能拥有所有权，或者为了避免悬空指针。可以从一个`weak_ptr`构造一个`shared_ptr`，从而取得对共享资源的访问权。

以下是 `weak_ptr`的部分定义，列出并简要介绍了最重要的函数。

```cpp
namespace boost {

  template<typename T> class weak_ptr {
  public:
    template <typename Y>
      weak_ptr(const shared_ptr<Y>& r);

    weak_ptr(const weak_ptr& r);

    ~weak_ptr();

    T* get() const; 
    bool expired() const; 
    shared_ptr<T> lock() const;
  };  
} 
```

### 成员函数

```cpp
template <typename Y> weak_ptr(const shared_ptr<Y>& r); 
```

这个构造函数从一个`shared_ptr`创建 `weak_ptr` ，要求可以从 `Y*` 隐式转换为 `T*`. 新的 `weak_ptr` 被配置为旁观 `r`所引向的资源。`r`的引用计数不会有所改变。这意味着`r`所引向的资源在被删除时不会理睬是否有`weak_ptr` 引向它。这个构造函数不会抛出异常。

```cpp
weak_ptr(const weak_ptr& r); 
```

这个复制构造函数让新建的 `weak_ptr` 旁观与`weak_ptr r`<small class="calibre23">相关的 shared_ptr</small>所引向的资源。<small class="calibre23">shared_ptr</small>的引用计数保持不变。这个构造函数不会抛出异常。

```cpp
~weak_ptr(); 
```

`weak_ptr` 的析构函数，和构造函数一样，它不改变引用计数。如果需要，析构函数会把 `*this` 与共享资源脱离开。这个析构函数不会抛出异常。

```cpp
bool expired() const; 
```

如果所观察的资源已经"过期"，即资源已被释放，则返回 `True` 。如果保存的指针为非空，`expired` 返回 `false`. 这个函数不会抛出异常。

```cpp
shared_ptr<T> lock() const 
```

返回一个引向`weak_ptr`所观察的资源的 `shared_ptr` ，如果可以的话。如果没有这样指针(即 `weak_ptr` 引向的是空指针)，`shared_ptr` 也将引向空指针。否则，`shared_ptr`所引向的资源的引用计数将正常地递增。这个函数不会抛出异常。

### 用法

我们从一个示范`weak_ptr`的基本用法的例子开始，尤其要看看它是如何不影响引用计数的。这个例子里也包含了 `shared_ptr`，因为 `weak_ptr` 总是需要和 `shared_ptr`一起使用的。使用 `weak_ptr` 要包含头文件 `"boost/weak_ptr.hpp"`.

```cpp
#include "boost/shared_ptr.hpp"
#include "boost/weak_ptr.hpp"
#include <iostream>
#include <cassert>

class A {};

int main() {

  boost::weak_ptr<A> w;
  assert(w.expired());
  {
    boost::shared_ptr<A> p(new A());
    assert(p.use_count()==1);
    w=p;
    assert(p.use_count()==w.use_count());
    assert(p.use_count()==1);

    // 从 weak_ptr 创建 shared_ptr 
    boost::shared_ptr<A> p2(w);
    assert(p2==p);
  }
  assert(w.expired());
  boost::shared_ptr<A> p3=w.lock();
  assert(!p3);
} 
```

`weak_ptr w` 被缺省构造，意味着它初始时不旁观任何资源。要检测一个 `weak_ptr` 是否在旁观一个活的对象，你可以使用函数 `expired`. 要开始旁观，`weak_ptr` 必须要被赋值一个 `shared_ptr`. 本例中，`shared_ptr p` 被赋值给 `weak_ptr w`, 这等于说`p` 和 `w` 的引用计数应该是相同的。然后，再从`weak_ptr`构造一个`shared_ptr`，这是一种从`weak_ptr`那里获得对共享资源的访问权的方法。如果在构造`shared_ptr`时，`weak_ptr` 已经过期了，将从`shared_ptr`的构造函数里抛出一个 `boost::bad_weak_ptr` 类型的异常。再继续，当 `shared_ptr p` 离开作用域，`w` 就变成过期的了。当调用它的成员函数 `lock` 来获得一个`shared_ptr`时，这是另一种获得对共享资源访问权的方法，将返回一个空的 `shared_ptr` 。注意，从这个程序的开始到结束，`weak_ptr` 都没有影响到共享对象的引用计数的值。

与其它智能指针不同的是，`weak_ptr` 不对它所观察的指针提供重载的 `operator*` 和 `operator-&gt;`. 原因是对`weak_ptr`所观察的资源的任何操作都必须是明显的，这样才安全；由于不会影响它们所观察的共享资源的引用计数器，所以真的很容易就会不小心访问到一个无效的指针。这就是为什么你必须要传送 `weak_ptr` 给 `shared_ptr`的构造函数，或者通过调用`weak_ptr::lock`来获得一个 `shared_ptr` 。这两种方法都会使引用计数增加，这样在 `shared_ptr` 从 `weak_ptr`创建以后，它可以保证共享资源的生存，确保在我们要使用它的时候它不会被释放掉。

### 常见问题

由于在智能指针中保存的是指针的值而不是它们所指向的指针的值，因此在标准库容器中使用智能指针有一个常见的问题，就是如何在算法中使用智能指针；算法通常需要访问实际对象的值，而不是它们的地址。例如，你如何调用 `std::sort` 并正确地排序？实际上，这个问题与在容器中保存并操作普通指针是几乎一样的，但事实很容易被忽略(可能是由于我们总是避免在容器中保存裸指针)。当然我们 不能直接比较两个智能指针的值，但也很容易解决。只要用一个解引用智能指针的谓词就可以了，所以我们将创建一个可重用的谓词，使得可以在标准库的算法里使 用引向智能指针的迭代器，这里我们选用的智能指针是 `weak_ptr`。

```cpp
#include <functional>
#include "boost/shared_ptr.hpp"
#include "boost/weak_ptr.hpp"

template <typename Func, typename T> 
  struct weak_ptr_unary_t : 
    public std::unary_function<boost::weak_ptr<T>,bool> {
  T t_;
  Func func_;

  weak_ptr_unary_t(const Func& func,const T& t) 
    : t_(t),func_(func) {}

  bool operator()(boost::weak_ptr<T> arg) const {
    boost::shared_ptr<T> sp=arg.lock();
    if (!sp) {
      return false;
    }
    return func_(*sp,t_);
  }
};

template <typename Func, typename T> weak_ptr_unary_t<Func,T> 
  weak_ptr_unary(const Func& func, const T& value) {
    return weak_ptr_unary_t<Func,T>(func,value);
} 
```

`weak_ptr_unary_t` 函数对象对要调用的函数以及函数所用的参数类型进行了参数化。把要调用的函数保存在函数对象中使用使得这个函数对象很容易使用，很快我们就能看到这一点。为了使这个谓词兼容于标准库的适配器，`weak_ptr_unary_t` 要从 `std::unary_function`派生，后者保证了所有需要的 `typedefs` 都能提供(这些要求是为了让标准库的适配器可以这些函数对象一起工作)。实际的工作在调用操作符函数中完成，从`weak_ptr`创建一个 `shared_ptr` 。必须要确保在函数调用时资源是可用的。然后才可以调用指定的函数(或函数对象)，传入本次调用的参数(要解引用以获得真正的资源) 和在对象中保存的值，这个值是在构造`weak_ptr_unary_t`时给定的。这个简单的函数对象现在可以用于任意可用的算法了。为方便起见，我们还定义了一个助手函数，`weak_ptr_unary`, 它可以推出参数的类型并返回一个适当的函数对象[14]。我们来看看如何使用它。

> [14] 要使得这个类型更通用，还需要更多的设计。

```cpp
#include <iostream>
#include <string>

#include <vector>
#include <algorithm>
#include "boost/shared_ptr.hpp"
#include "boost/weak_ptr.hpp"

int main() {
  using std::string;
  using std::vector;
  using boost::shared_ptr;
  using boost::weak_ptr;

  vector<weak_ptr<string> > vec;

  shared_ptr<string> sp1(
    new string("An example"));
  shared_ptr<string> sp2(
    new string("of using"));
  shared_ptr<string> sp3(
    new string("smart pointers and predicates"));
  vec.push_back(weak_ptr<string>(sp1));
  vec.push_back(weak_ptr<string>(sp2));
  vec.push_back(weak_ptr<string>(sp3));

  vector<weak_ptr<string> >::iterator
    it=std::find_if(vec.begin(),vec.end(),
     weak_ptr_unary(std::equal_to<string>(),string("of using")));

  if (it!=vec.end()) {
    shared_ptr<string> sp(*++it);
    std::cout << *sp << '\n';
  }
} 
```

本例中，创建了一个包含`weak_ptr`的 `vector`。最有趣的一行代码(是的，它有点长)就是我们为使用`find_if`算法而创建`weak_ptr_unary_t`的那行。

```cpp
vector<weak_ptr<string> >::iterator it=std::find_if(
  vec.begin(),
  vec.end(),
  weak_ptr_unary(
    std::equal_to<string>(),string("of using"))); 
```

通过把另一个函数对象，`std::equal_to`, 和一个用于匹配的`string`一起传给助手函数`weak_ptr_unary`，创建了一个新的函数对象。由于 `weak_ptr_unary_t` 完全兼容于各种适配器(由于它是从`std::unary_function`派生而来的)，我们可以再从它组合出各种各样的函数对象。例如，我们也可以查找第一个不匹配`"of using"`的串：

```cpp
vector<weak_ptr<string> >::iterator it=std::find_if(
  vec.begin(),
  vec.end(),
std::not1(
    weak_ptr_unary(
      std::equal_to<string>(),string("of using")))); 
```

Boost 智能指针是专门为了与标准库配合工作而设计的。我们可以创建有用的组件来帮助我们可以更简单地使用这些强大的智能指针。象 `weak_ptr_unary` 这样的工具并不是经常要用到的；有一个库提供了比`weak_ptr_unary`更好用的泛型绑定器[15]。弄懂这些智能指针的语义，可以让我们更清楚地使用它们。

> <a>[15] 指 Boost.Bind 库。

### 两种从 weak_ptr 创建 shared_ptr 的惯用法

如你所见，如果你有一个旁观某种资源的 `weak_ptr` ，你最终还是会想要访问这个资源。为此，`weak_ptr` 必须被转换为 `shared_ptr`, 因为 `weak_ptr` 是不允许访问资源的。有两种方法可以从`weak_ptr`创建`shared_ptr`：把 `weak_ptr` 传递给 `shared_ptr` 的构造函数，或者调用 `weak_ptr` 的成员函数`lock`, 它返回 `shared_ptr`. 选择哪一个取决于你认为一个空的 `weak_ptr` 是错误的抑或不是。`shared_ptr` 构造函数在接受一个空的 `weak_ptr` 参数时会抛出一个 `bad_weak_ptr` 类型的异常。因此应该在你认为空的 `weak_ptr` 是一种错误时使用它。如果使用 `weak_ptr` 的函数 `lock`, 它会在`weak_ptr`为空时返回一个空的 `shared_ptr`。这在你想测试一个资源是否有效时是正确的，一个空的 `weak_ptr` 是预期中的。此外，如果使用 `lock`, 那么使用资源的正确方法应该是初始化并同时测试它，如下：

```cpp
#include <iostream>
#include <string>
#include "boost/shared_ptr.hpp"
#include "boost/weak_ptr.hpp"

int main() {
  boost::shared_ptr<std::string> 
    sp(new std::string("Some resource"));
  boost::weak_ptr<std::string> wp(sp);
  // ...
  if (boost::shared_ptr<std::string> p=wp.lock())
    std::cout << "Got it: " << *p << '\n';
  else
    std::cout << "Nah, the shared_ptr is empty\n";
} 
```

如你所见，shared_ptr `p` 被`weak_ptr wp` 的`lock`函数的结果初始化。然后 `p` 被测试，只有当它非空时资源才能被访问。由于 `shared_ptr` 仅在这个作用域中有效，所以在这个作用域之外不会有机会让你不小心用到它。另一种情形是当 `weak_ptr` 逻辑上必须非空的时候。那种情形下，不需要测试 `shared_ptr` 是否为空，因为 `shared_ptr` 的构造函数会在接受一个空`weak_ptr`时抛出异常，如下：

```cpp
#include <iostream>
#include <string>
#include "boost/shared_ptr.hpp"
#include "boost/weak_ptr.hpp"

void access_the_resource(boost::weak_ptr<std::string> wp) {
  boost::shared_ptr<std::string> sp(wp);
  std::cout << *sp << '\n';
}

int main() {
  boost::shared_ptr<std::string> 
    sp(new std::string("Some resource"));
  boost::weak_ptr<std::string> wp(sp);
  // ...
  access_the_resource(wp);  
} 
```

在这个例子中，函数 `access_the_resource` 从一个`weak_ptr`构造 `shared_ptr sp` 。这时不需要测试 `shared_ptr` 是否为空，因为如果 `weak_ptr` 为空，将会抛出一个 `bad_weak_ptr` 类型的异常，因此函数会立即结束；错误会在适当的时候被捕获和处理。这样做比显式地测试 `shared_ptr` 是否为空然后返回要更好。这就是从`weak_ptr`获得`shared_ptr`的两种方法。

### 总结

`weak_ptr` 是 Boost 智能指针拼图的最后一块。`weak_ptr` 概念是`shared_ptr`的一个重要伙伴。它允许我们打破递归的依赖关系。它还处理了关于悬空指针的一个常见问题。在共享一个资源时，它常用于那些不参与生存期管理的资源用户。这种情况不能使用裸指针，因为在最后一个 `shared_ptr` 被销毁时，它会释放掉共享的资源。如果使用裸指针来引用资源，将无法知道资源是否仍然存在。如果资源已经不存在，访问它将会引起灾难。通过使用 `weak_ptr`, 关于共享资源已被销毁的信息会传播给所有旁观的 `weak_ptr`s，这意味着不会发生无意间访问到无效指针的情形。这就象是观察员模式(Observer pattern)的一个特例；当资源被销毁，所有表示对此感兴趣的都会被通知到。

对于以下情形使用 `weak_ptr` ：

*   要打破递归的依赖关系

*   使用一个共享的资源而不需要共享所有权

*   避免悬空的指针

# Smart_ptr 总结

## Smart_ptr 总结

本章介绍了 Boost 的智能指针，它们是对 C++社区的贡献，无论怎样评价都不过份。对于一个成功的 智能指针库，它必须考虑到并正确地处理大量的细节因素。我可以肯定你曾经见过很多种智能指针，你也可能曾经参与过编写它们，因此你应该知道做好这件事所要 花费的努力。没有其它的智能指针可以和它们一样智能，因此 Boost.Smart_ptr 库具有很高的价值。

作为软件工程中的重要组成部分，Boost 的智能指针明显受到了广泛的关注和彻底的审查。因此很难列出所有的贡献者。很多人给出了有价值的意见和对当前的智能指针库进行了修正。这里列出一些突出的人员及其贡献：

*   Greg Colvin, `auto_ptr`之父, 还提出了`counted_ptr`, 最后成为现在的`shared_ptr`.

*   Beman Dawes 重新激活了对智能指针的讨论，并提议了 Greg Colvin 原先建议的语义。

*   Peter Dimov 重新设计了智能指针类，增加线程安全，`intrusive_ptr`, 以及 `weak_ptr`.

如此著名的概念不断地在发展，这是很吸引人的。毫无疑问，智能指针或者说智能资源的领域还会有更进一步的发展，但就今天而言，重要的是智能指针的质量。适 者生存，这就是为什么人们在使用 Smart_ptr 的原因。Boost 智能指针是一块精美的、精心挑选的、美味的软件巧克力，我经常吃它们(你也应该这样)。我们很快就会看到它们中的某些将成为 C++标准库的一部分，因为它 们已经被收入 Library Technical Report。