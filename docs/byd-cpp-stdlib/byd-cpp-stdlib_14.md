# Library 6\. Any

# Library 6\. Any

# Any 库如何改进你的程序？

## Any 库如何改进你的程序？

*   任意类型的类型安全存储以及安全的取回

*   在标准库容器中存放不同类型的方法

*   可以在无须知道类型的情况下传送类型

Any 库提供一个类型, `any`, 它允许存入任意类型且稍后取回，而不损失类型安全性。它有点象是可变类型的化合物：它可以持有任意类型，但你必须知道类型才能取回值。有很多次你想在同一 个容器中存入互不相关的类型。有很多次某些代码只想从一个指针向另一个指针传送数据，而不关心数据的类型。从表面看，这些事情很容易做。它们可以通过一个 无类的类型来实现，如 `void*`. 它们也可以通过使用一个含有不同类型的 union 来实现。有很多可变类型通过一些类型标识机制来实现。不幸的是，所有这些都缺乏类型安全性，而只有在最可 控的情形下我们才应该故意绕过类型系统。标准库的容器是要通过它们所包含的类型来特化的，这意味着不可能把不同类型的元素存入容器之内。幸运的是，解决的 方案不一定要 `void*`, 因为 Any 库允许你将存入不同的类型而稍后取回。没有办法在不知道实际类型的情况下取回存入的值，类型安全从而得到保证。

在设计框架时，不可能预先知道哪些类型要和框架类一起使用。一个常见的方 法是，要求框架的使用者遵守某种接口，或者从框架所提供的某个基类进行派生。这是合理的，因为框架可能需要与不同的高级类进行通信才能使用。但是也存在这 样的情形，框架对于存入或接受的类型无须(或不能)知道任何相关信息。不要绕过类型系统去使用 `void*` 方法，框架可以使用 `any` 。

# Any 如何适用于标准库？

## Any 如何适用于标准库？

Any 的一个重要特性是，它提供了存储不同类型的对象到标准库容器中的能力。它也是一种可变数据类型，这正是 C++标准库非常需要而又缺乏的。

# Any

## Any

### 头文件: `"boost/any.hpp"`

类 `any` 允许对任意类型进行类型安全的存储和取回。不象无类类型，`any` 保存了类型信息，并且不会让你在不知道正确类型的情况下获得存入的值。当然，有办法可以让你询问关于类型的信息，也有测试保存的值的方法，但最终，调用者必须知道在 `any` 对象中的值的真实类型，否则不能访问`any`。可以把 `any` 看作为上锁的安全性。没有正确的钥匙，你不能进入其中。`any` 对它所保存的类型有以下要求：

*   CopyConstructible 它必须可以复制这个类型

*   Non-throwing destructor 就象所有析构函数应该的那样！

*   Assignable 为了保证强异常安全(不符合可赋值要求的类型也可以用于 `any`, 但没有强异常安全的保证)

以下是 `any` 的公有接口：

```cpp
namespace boost {

  class any {
  public:
    any();
    any(const any&);

    template<typename ValueType>
     any(const ValueType&);

    ~any();

    any& swap(any &);
    any& operator=(const any&);

    template<typename ValueType>
     any& operator=(const ValueType&);

    bool empty() const;
    const std::type_info& type() const;
  };
} 
```

### 成员函数

```cpp
any(); 
```

缺省构造函数创建一个空的 `any` 实例，即一个不含有值的 `any`。当然，你无法从一个空的`any`中取回值，因为没有值存在。

```cpp
any(const any& other); 
```

创建一个已有 `any` 对象的独立拷贝。`other` 中含有的值被复制并存入 `this`.

```cpp
template<typename ValueType> any(const ValueType&); 
```

这个模板构造函数存入一个传入的`ValueType`类型参数的拷贝。参数是一个 `const` 引用，因此传入一个临时对象来存入`any`是合法的。注意，该构造函数不是 explicit 的，如果是的话， `any` 会难以使用，而且也不会增加安全性。

```cpp
~any(); 
```

析构函数销毁含有的值，但注意，由于对一个裸指针的析构不会调用 operator `delete` 或 operator `delete[]` ，所以在`any`中使用指针时，你应该把裸指针包装成一个象 `shared_ptr` (见 "Library 1: Smart_ptr 1") 那样的智能指针。

```cpp
any& swap(any& other); 
```

交换存在两个 `any` 对象中的值。

```cpp
any& operator=(const any& other); 
```

如果`any`实例非空，则丢弃所存放的值，并存入`other`值的拷贝。

```cpp
template<typename ValueType>
  any& operator=(const ValueType& value); 
```

如果`any`实例非空，则丢弃所存放的值，并存入 `value` 的一份拷贝，`value`可以是任意符合`any`要求的类型。

```cpp
bool empty() const; 
```

给出`any`实例当前是否有值，不管是什么值。因而，当`any`持有一个指针时，即使该指针值为空，则 `empty` 也返回 `false` 。

```cpp
const std::type_info& type() const; 
```

给出所存值的类型。如果 `any` 为空，则类型为 `void`.

### 普通函数

```cpp
template<typename ValueType>
  ValueType any_cast(const any& operand); 
```

`any_cast` 让你访问`any`中存放的值。参数为需要取回值的 `any` 对象。如果类型 `ValueType` 与所存值不符，`any` 抛出一个 `bad_any_cast` 异常。请注意，这个语法有点象 `dynamic_cast`.

```cpp
template<typename ValueType>
  const ValueType* any_cast(const any* operand); 
```

`any_cast` 的一个重载，接受一个指向 `any` 的指针，并返回一个指向所存值的指针。如果 `any` 中的类型不是 `ValueType`, 则返回一个空指针。请再次注意，这个语法也有点象 `dynamic_cast`.

```cpp
template<typename ValueType>
  ValueType* any_cast(any* operand); 
```

`any_cast` 的另一个重载，与前一个版本相似，但前一个版本使用 `const` 指针的参数并返回 `const` 指针，这个版本则不是。

### 异常

```cpp
bad_any_cast 
```

当试图将一个`any`对象转换为该对象所存类型以外的其它类型，将抛出该异常。`bad_any_cast` 派生自 `std::bad_cast`. 注意，使用指针参数调用 `any_cast` 时，将不抛出异常(类似于对指针使用 `dynamic_cast` 时返回空指针一样)，反之对引用类型使用 `dynamic_cast` 则会在失败时抛出异常。

# 用法

## 用法

Any 库定义在名字空间 `boost` 内。你要用类 `any` 来保存值，用模板函数 `any_cast` 来取回存放的值。为了使用 `any`, 要包含头文件 `"boost/any.hpp"`. 创建一个可以存放任意值的实例是很容易的。

```cpp
boost::any a; 
```

把任意类型的值赋给它也很容易。

```cpp
a=std::string("A string");
a=42;
a=3.1415; 
```

`any`几乎可以接受任何东西！但是，为了真正能使用存放在`any`中的值，我们需要取回它，对吧？为此，我们需要知道这个值的类型。

```cpp
std::string s=boost::any_cast<std::string>(a);
// 抛出 boost::bad_any_cast. 
```

这显然不行；因为当前的 `a` 所持的是一个 `double`, `any_cast` 抛出一个 `bad_any_cast` 异常。以下这样则可以。

```cpp
double d=boost::any_cast<double>(a); 
```

`any` 只允许你在知道类型的前提下访问它的值，这是很明智的。对于这个库，典型情况下你只需记住两件事：类 `any`, 用于存放值，还有模板函数 `any_cast`, 用于取回值。

### 任意的东西！

考虑三个类，`A`, `B`, 和 `C`, 它们没有共同的基类，而我们想把它们存入一个 `std::vector`. 如果它们没有共同基类，看起来我们不得不把它们当成 `void*` 来保存，对吗？唔，not any more (相关语，没有更多的了)，因为类型 `any` 没有改变对所存值的类型的依赖。以下代码示范了如何解决这个问题。

```cpp
#include <iostream>
#include <string>
#include <utility>
#include <vector>
#include "boost/any.hpp"

class A {
public:
  void some_function() { std::cout << "A::some_function()\n"; }
};

class B {
public:
  void some_function() { std::cout << "B::some_function()\n"; }
};

class C {
public:
  void some_function() { std::cout << "C::some_function()\n"; }
};

int main() {
  std::cout << "Example of using any.\n\n";

  std::vector<boost::any> store_anything;

  store_anything.push_back(A());
  store_anything.push_back(B());
  store_anything.push_back(C());

  // 我们再来，再加一些别的东西
  store_anything.push_back(std::string("This is fantastic! "));
  store_anything.push_back(3);
  store_anything.push_back(std::make_pair(true, 7.92));

  void print_any(boost::any& a);
  // 稍后定义；打印 a 中的值

  std::for_each(
  store_anything.begin(),
  store_anything.end(),
  print_any);
} 
```

运行以上例子，将有如下输出。

```cpp
Example of using any.

A::some_function()
B::some_function()
C::some_function()
string: This is fantastic!
Oops!
Oops! 
```

好的，我们可以保存任意我们想要的东西，但我们如何取回保存在`vector`的元素中的值呢？在前例中，我们用 `for_each` 来为`vector`中的每个元素调用 `print_any()` 。

```cpp
void print_any(boost::any& a) {
  if (A* pA=boost::any_cast<A>(&a)) {
    pA->some_function();
  }
  else if (B* pB=boost::any_cast<B>(&a)) {
    pB->some_function();
  }
  else if (C* pC=boost::any_cast<C>(&a)) {
    pC->some_function();
  }
} 
```

到目前为止，`print_any` 会试着取回一个指向 `A`, `B`, 或 `C` 对象的指针。这可以使用普通函数 `any_cast` 来完成，使用要"转换"成的类型来特化该函数。看清楚些这个转换，我们试着解开这个 `any a` ，对它说我们认为 `a` 包含一个类型 `A` 的值。请注意，我们以指针方式来传递我们的 `any` 给 `any_cast` 函数。因此，返回值将会是一个指向 `A`, `B`, 或 `C` 的指针。如果 `any` 没有包含我们要转换成的类型，将返回空指针。在本例中，如果转型成功，我们就使用返回的指针来调用 `some_function` 成员函数。但 `any_cast` 也可以作一些小的调整。

```cpp
 else {
    try {
      std::cout << boost::any_cast<std::string>(a) << '\n';
    }
    catch(boost::bad_any_cast&) {
      std::cout << "Oops!\n";
    }
  }
} 
```

现在有点不同了。我们还是执行一个用我们要取回的类型来特化的 `any_cast` ，但不是用指针的方式来传递 `any` 实例，而是用 `const` 引用来传递。这改变了 `any_cast` 的行为；这种情况下，失败——即请求一个错误的类型——将会抛出一个 `bad_any_cast` 类型的异常。因此，如果我们不能绝对肯定`any`中包含的是什么类型时，我们必须用一个`try`/`catch`块来保护执行 `any_cast` 的代码。这种行为上的差异(类似于 `dynamic_cast`)给了你更大的自由度。在转型失败不是一种错误时，使用指针来传递 `any`, 但如果转型失败是一种错误，则应该使用`const`引用来传递，这样可以让 `any_cast` 在失败时抛出异常。

使用 `any` 让你能够在原来不能使用标准库容器和算法的地方下使用它们，从而让你可以写出更具有可维护性和更易懂的代码。

### 属性类

现在我们想定义一个可用于容器的属性类。我们将以字符串方式来保存属性的名字，而属性值则可以为任意 类型。虽然我们可以要求所有属性值从一个共同的基类派生而来，但这样常常不可行。例如，我们可能无法访问所有那些我们需要用作属性值的类的源代码，也有可 能有些属性值是内建类型，不可能是派生类(此外，那样也不能做出一个好的 `any` 示例)。通过把属性值存入一个 `any` 实例，我们可以让使用者去处理那些他们知道类型且感兴趣的属性值。

```cpp
#include <iostream>
#include <string>
#include <vector>
#include <algorithm>
#include "boost/any.hpp"

class property {
  boost::any value_;
  std::string name_;

public:
  property(
    const std::string& name,
    const boost::any& value)
    : name_(name),value_(value) {}

  std::string name() const { return name_; }
  boost::any& value() { return value_; }
  friend bool operator<
    (const property& lhs, const property& rhs) {
    return lhs.name_<rhs.name_;
  }
}; 
```

这个例子中的 `property` 类有一个保存为 `std::string` 的名字来标识，并用一个 `any` 来保存属性值。`any` 为该实现带来的灵活性在于，我们可以使用内建类型或用户自定义类型而无须修改这个属性类。不管它简单或是复杂，一个 `any` 实例总是可以保存任意的东西。当然，使用 `any` 同时也意味着我们预先不知道可以对一个`property`中保存的属性值进行哪些操作，我们需要首先把属性值取出来。这暗示了如果对一个属性类预先知道有哪些类型适用，我们可能要考虑`any`以外的其它方法。这在进行框架设计时是罕见的，如果我们不要求一个确定的基类，我们所能够保证的就是，我们完全不知道会用到哪些类。如果你可以获得任意类型而不需要对它做任何动作，除了保有它一段时间并稍后把它还回去，那么你会发现 `any` 是最合适的。注意，这个属性类提供了 `operator&lt;` 以允许该类可以保存在标准库的关联容器中；即使没有这个操作符，`property` 仍然可以很好地用于序列容器。

以下程序使用了我们新的、灵活的`property`类，全亏了 `any`！`property` 类的实例被存入一个 `std::vector` 并以属性名排序(译注：原文是说存入一个 `std::map`，似有误)。

```cpp
void print_names(const property& p) {
  std::cout << p.name() << "\n";
}

int main() {
  std::cout << "Example of using any for storing properties.\n";

  std::vector<property> properties;
  properties.push_back(
  property("B", 30));
  properties.push_back(
  property("A", std::string("Thirty something")));
  properties.push_back(property("C", 3.1415));

  std::sort(properties.begin(),properties.end());
  std::for_each(
  properties.begin(),
  properties.end(),
  print_names);

  std::cout << "\n";

  std::cout <<
  boost::any_cast<std::string>(properties[0].value()) << "\n";
  std::cout <<
  boost::any_cast<int>(properties[1].value()) << "\n";
  std::cout <<
  boost::any_cast<double>(properties[2].value()) << "\n";
} 
```

注意，我们不必为 `property` 的构造函数显式创建 `any` 。这是因为 `any` 的类型转换构造函数不是 explicit 的。虽然构造函数通常应该接受一个显式声明的参数，但 `any` 是这个规则的一个例外。运行这段程序会得到以下输出。

```cpp
Example of using any for storing properties.
A
B
C

Thirty something
30
3.1415 
```

在这个例子中，由于容器被排序了，我们可以按索引取回属性值，而且我们预先知道它们各自的类型，所以我们不需要用 `try`/`catch` 块来处理取回操作。从一个 `any` 实例中取回值时，如果失败表示一个真正的错误，则应该用 `const` 引用来传递 `any` 。

```cpp
std::string s=boost::any_cast<std::string>(a); 
```

当失败不应是一个错误时，用指针来传递 `any` 。

```cpp
std::string* ps=boost::any_cast<std::string>(&a); 
```

这两种不同风格的取回操作不仅在语义上有所不同，而且返回的值也不同。如果你传递一个指针参数，你会得到一个指向保存值的指针；如果你传递一个 `const` 引用参数，你会得到一个保存值的拷贝。

如果值的类型在拷贝时代价很昂贵，就应该传递指针以避免值的拷贝。

### 关于 any 的更多

`any` 还提供了其它几个成员函数，如测试一个 `any` 实例是否为空，交换两个 `any` 实例的值。以下例子示范了如何使用它们。

```cpp
#include <iostream>
#include <string>
#include "boost/any.hpp"

int main() {
  std::cout << "Example of using any member functions\n\n";

  boost::any a1(100);
  boost::any a2(std::string("200"));
  boost::any a3;

  std::cout << "a3 is ";
  if (!a3.empty()) {
    std::cout << "not ";
  }
  std::cout << "empty\n";

  a1.swap(a2);

  try {
    std::string s=boost::any_cast<std::string>(a1);
  std::cout << "a1 contains a string: " << s << "\n";
  }
  catch(boost::bad_any_cast& e) {
    std::cout << "I guess a1 doesn't contain a string!\n";
  }

  if (int* p=boost::any_cast<int>(&a2)) {
    std::cout << "a2 seems to have swapped contents with a1: "
      << *p << "\n";
  }
  else {
    std::cout << "Nope, no int in a2\n";
  }

  if (typeid(int)==a2.type()) {
    std::cout << "a2's type_info equals the type_info of int\n";
  }

} 
```

以下是这段程序的输出结果。

```cpp
Example of using any member functions

a3 is empty
a1 contains a string: 200
a2 seems to have swapped contents with a1: 100
a2's type_info equals the type_info of int 
```

让我们来更进一步分析这段代码。为了测试一个 `any` 实例是否包含值，我们调用成员函数 `empty`. 我们这样来测试 `any a3` 。

```cpp
std::cout << "a3 is ";
if (!a3.empty()) {
  std::cout << "not ";
}
std::cout << "empty\n"; 
```

因为我们是缺省构造 `a3` 的，因此 `a3.empty()` 返回 `true`. 下一件事情是交换 `a1` 和 `a2` 的内容。你可能会想为什么要交换它们的内容。一种可能的情形是当 `any` 实例的标识非常重要时(`swap` 仅交换其中包含的值)。另一个原因是在你不需要原来的值时避免拷贝。

```cpp
a1.swap(a2); 
```

最后，我们使用了成员函数 `type`, 它返回一个 `const std::type_ info&`, 用于测试所含的值是否类型 `int` 的值。

```cpp
if (typeid(int)==a2.type()) { 
```

注意，如果一个 `any` 保存了一个指针类型，则`std::type_info`表示的是相应的指针类型。

### 在 any 中保存指针

通常，测试 `empty` 足以知道对象是否真的包含有效的东西。但是，如果 `any` 持有的是一个指针，则要在解引用它之前额外小心地测试这个指针。仅仅简单地测试 `any` 是否为空是不够的，因为一个 `any` 在持有一个指针时会被认为是非空的，即使这个指针是空的。

```cpp
boost::any a(static_cast<std::string*>(0));

if (!a.empty()) {
  try {
    std::string* p=boost::any_cast<std::string*>(a);
    if (p) {
      std::cout << *p;
    }
    else {
      std::cout << "The any contained a null pointer!\n";
    }
  }
  catch(boost::bad_any_cast&) {}
} 
```

### 一个更好的办法——使用 shared_ptr

在 `any` 中保存裸指针的另一个麻烦在于析构的语义。`any` 类接受了它所存值的所有权，因为它保持一个该值的内部拷贝，并与 `any` 一起销毁它。但是，销毁一个裸指针并不会对它调用 `delete` 或 `delete[]` ！它仅仅要求归还属于指针的那点内存。这使得在 `any` 中保存指针是有问题的，所以更好的办法是使用智能指针来代替。的确，使用智能指针(见 "Library 1: Smart_ptr 1")是在一个 `any` 中保存指针的好办法。它解决了要保证所存指针指向的内存被正确释放的问题。当智能指针被销毁时，它会正确地确保内存及其中的数据被正确销毁。作为对比，要注意 `std::auto_ptr` 不是合适的智能指针。这是因为 `auto_ptr` 没有通常的复制语义；访问一个 `any` 中的值会把内存及其中数据的所有权从 `any` 转移到返回的 `auto_ptr` 中。

考虑以下代码。

```cpp
#include <iostream>
#include <string>
#include <algorithm>
#include <vector>
#include "boost/any.hpp"
#include "boost/shared_ptr.hpp" 
```

首先，我们定义两个类，`A` 和 `B`, 每个都有成员函数 `is_virtual`, 它是虚拟的，还有一个成员函数 `not_virtual`, 它不是虚拟的(如果它也是虚拟的，那么这个名字就真是糟透了)。我们想把这两类对象存入 `any` 。

```cpp
class A {
public:
  virtual ~A() {
    std::cout << "A::~A()\n";
  }

  void not_virtual() {
    std::cout << "A::not_virtual()\n";
  }

  virtual void is_virtual () {
    std::cout << "A:: is_virtual ()\n";
  }
};

class B : public A {
public:

  void not_virtual() {
    std::cout << "B::not_virtual()\n";
  }

  virtual void is_virtual () {
    std::cout << "B:: is_virtual ()\n";
  }
}; 
```

我们现在来定义一个普通函数，`foo`, 它接受一个 `any` 引用的参数，并使用 `any_cast` 来尝试将该 `any` 转为这个函数知道如何处理的类型。如果不能匹配，这个函数简单地忽略该 `any` 并返回。它分别对类型 `shared_ptr&lt;A&gt;` 和 `shared_ptr&lt;B&gt;` 进行测试，并对调用它们的 `is_virtual` (虚拟函数) 和 `not_virtual` 。

```cpp
void foo(boost::any& a) {

  std::cout << "\n";

  // 试一下 boost::shared_ptr<A>
  try {
    boost::shared_ptr<A> ptr=
    boost::any_cast<boost::shared_ptr<A> >(a);

    std::cout << "This any contained a boost::shared_ptr<A>\n";
    ptr-> is_virtual ();
    ptr->not_virtual();
    return;
  }
  catch(boost::bad_any_cast& e) {}

  // 试一下 boost::shared_ptr<B>
  try {
    boost::shared_ptr<B> ptr=
    boost::any_cast<boost::shared_ptr<B> >(a);

    std::cout << "This any contained a boost::shared_ptr<B>\n";
    ptr-> is_virtual ();
    ptr->not_virtual();
    return;
  }
  catch(boost::bad_any_cast& e) {}

  // 如果是其它东西(如一个字符串), 则忽略它
  std::cout <<
    "The any didn't contain anything that \
     concerns this function!\n";
} 
```

在 `main` 里面，我们创建两个 `any` 。然后我们引入一个作用域，再创建两个新的 `any` 。接着，我们把所有 `any` 存入 `vector` 并把其中所有元素传给函数 `foo`, 它测试它们的内容并操作它们。这时要注意我们违反了前面给出的建议，即在失败不代表错误时应该使用指针形式的 `any_cast` 。但是，因为我们在这儿用的是智能指针，这时使用异常抛出形式的`any_cast`的语法优势完全有理由忽略这个建议。

```cpp
int main() {
  std::cout << "Example of any and shared_ptr\n";

  boost::any a1(boost::shared_ptr<A>(new A));
  boost::any a2(std::string("Just a string"));

  {
    boost::any b1(boost::shared_ptr<A>(new B));
    boost::any b2(boost::shared_ptr<B>(new B));
    std::vector<boost::any> vec;
    vec.push_back(a1);
    vec.push_back(a2);
    vec.push_back(b1);
    vec.push_back(b2);

    std::for_each(vec.begin(),vec.end(),foo);
    std::cout << "\n";
  }
1
  std::cout <<
    "any's b1 and b2 have been destroyed which means\n"
    "that the shared_ptrs' reference counts became zero\n";

} 
```

程序运行时，将有如下输出。

```cpp
Example of any and shared_ptr
This any contained a boost::shared_ptr<A>
A:: is_virtual ()
A::not_virtual()
The any didn't contain anything that concerns this function!
This any contained a boost::shared_ptr<A>
B:: is_virtual ()
A::not_virtual()
This any contained a boost::shared_ptr<B>
B:: is_virtual ()
B::not_virtual()
A::~A()
A::~A()
any's b1 and b2 have been destroyed which means
that the shared_ptrs' reference counts became zero
A::~A() 
```

(译注：最后三行输出是原文没有的，但应该有)

首先，我们看到传给 `foo` 的 `any` 含有一个 `shared_ptr&lt;A&gt;`, 它恰好拥有一个 `A` 的实例。输出正是我们所期望的。

接着是我们加到 `vector` 中的含有 `string` 的 `any` 。这显示了保存一些对稍后要被调用的函数而言是未知类型的类型到一个 `any`，是很有可能的，通常也是合理的；这些函数只需要处理它们需要操作的类型！

接下来的事情更有趣了，第三个元素含有一个指向 `B` 实例的 `shared_ptr&lt;A&gt;` 。这是一个例子，说明了 `any` 如何与其它类型一样实现多态性。当然，如果我们使用裸指针，就需要用 `static_cast` 来保存指针为我们想标识的类型。注意，函数 `A::not_virtual` 被调用而不是 `B::not_virtual`. 原原因是这个指针的静态类型是 `A*`, 而不是 `B*`.

最后一个元素含有一个 `shared_ptr&lt;B&gt;` ，它也正好指向一个 `B` 的实例。再一次，我们可以控制保存在 `any` 的类型，在后面一个 try 中设置相应的参数来打开它。

在里面的那个作用域结束时，`vector` 被销毁，它又销毁了内含的 `any` 实例，后者再销毁所有的 `shared_ptr`，正确地设置引用参数为零。接着，我们的指针被安全和不费力气地销毁！

这个例子显示了一些比如何与 `any` 一起使用智能指针更为重要的东西；它(又一次)显示了存入 `any` 的类型有是简单的或是复杂的都无关紧要。如果复制被存值的代价是高得惊人的，或者如果需要共享使用和控制生存期，就应该考虑使用智能指针，就象使用标准库的容器保存值一样。同样的推理也适用于 `any`, 通常这两个原则是一致的，正如在容器中使用 `any` 就意味着要保存不同的类型。

### 输入和输出操作符怎么啦？

`any` 用户的一个常见问题是，"为什么没有输入和输出操作符？" 这真的是有原因的。让我们从输入操作符开始。输入的语义应该是什么？它应该缺省为一个 `string` 类型吗？`any`当前持有的类型应该被用于从流中读取数据吗？如果是的话，那么首先为什么要用 `any` 呢？这些问题没有好的答案，这正是为什么 `any` 没有输入操作符的原因。回答第二个问题并不容易，但差不多。让 `any` 支持一个输出操作符意味着 `any` 不再能够保存任意的类型，因为这个操作符对于保存在 `any` 中的类型增加了一个要求。如果我们不有意去使用`operator&lt;&lt;`，这本无关紧要；但一个含有不支持输出操作符的类型的 `any` 实例仍是非法的，在编译时会导致一个错误。当然，只要我们提供一个模板版本的 `operator&lt;&lt;`, 我们就可以使用 `any` 而无须要求被包含的类型支持流输出，但一旦这个操作符被实例化，这个要求还是会被打开。

看起来，这些就是没有这些操作符的原因了，对吗？如果我们给可以匹配任意东西的 `any` 提供一个有效的输出操作符，并把 `operator&lt;&lt;` 引入到只能从 `any` 类的实现细节进行访问的作用域，它会是什么样的呢？那样的话，我们可以在执行输出到一个流时选择抛出一个异常或返回一个错误代码(这个功能仅用于那些不支持 `operator&lt;&lt;` 的参数)，我们将要在运行期去做这些动作，而不影响其它代码的合法性。这种想法非常吸引我，我用几个手边的编译器上试了一下。结果不太好。我不想详细讨论它，但简而言之，这种方法需要一些很多编译器目前还不能处理的技术。但是，我们无需修改 `any` 类，我们可以创建一个利用 `any` 来保存任意类型的新类，并让这个新类支持 `operator&lt;&lt;`. 基本上，我们无论如何都需要做的是，`any` 要了解被含类型，知道如何进行输出，然后加到输出流中去。

### 增加对输出的支持——any_out

我们将定义一个类，它具有通过 `operator&lt;&lt;` 进行输出的功能。这增加了对被存类型的要求；作为可以保存在类 `any_out` 中的有效类型，必须要支持 `operator&lt;&lt;`.

```cpp
#include <iostream>
#include <vector>
#include <string>
#include <ostream>

#include "boost/any.hpp"

class any_out { 
```

该 `any_out` 类保存(任意的)值在一个 `boost::any` 类型中。总是应该选择重用，而不是重新发明！

```cpp
boost::any o_; 
```

接下来，我们声明一个抽象类 `streamer`, 它使用和 `any` 同样的设计。我们不能直接使用泛型类型，因为那样我们还不如泛化 `any_out` 算了，这样会使 `any_out` 的类型依赖于它所含值的类型，在需要存储不同类型的上下文中这样的类是没有用的。所含值的类型不能成为 `any_out` 类的标识的一部分。

```cpp
struct streamer {
  virtual void print(std::ostream& o,boost::any& a)=0;
  virtual streamer* clone()=0;
  virtual ~streamer() {}
}; 
```

这里有一个窍门：我们增加了一个泛型类 `streamer_imp`, 用所含类型来特化，并派生自 `streamer`. 因而，我们可以在 `any_out` 中保存一个 `streamer` 指针，并依靠多态性来完成剩余的工作(接下来，我们将为此增加一个虚拟成员函数)。

```cpp
template <typename T> struct streamer_imp : public streamer { 
```

现在，让我们实现一个虚拟函数 `print` ，通过执行一个到用来特化 `streamer_imp` 的类型的 `any_cast` 来输出 `any` 中所含的值。因为我们将会用与存入 `any` 中的值相同的类型来实例化一个 `streamer_imp`，因此这个转型不会失败。

```cpp
virtual void print(std::ostream& o,boost::any& a) {
  o << *boost::any_cast<T>(&a);
} 
```

复制一个 `any_out` 时需要一个克隆函数，由于我们准备保存一个 `streamer` 指针，所以虚拟函数 `clone` 负责拷贝正确的 `streamer` 类型。

```cpp
 virtual streamer* clone() {
    return new streamer_imp<T>();
  }
};

class any_out {
  streamer* streamer_;
  boost::any o_;
public: 
```

缺省构造函数用于创建一个空的 `any_out`, 并设置 `streamer` 指针为零。

```cpp
any_out() : streamer_(0) {} 
```

`any_out` 中最有趣的函数是泛型构造函数。通过存入的值来推断出类型 `T` ，并用于创建 `streamer`. 同时，值被存入 `any o_`.

```cpp
template <typename T> any_out(const T& value) :
  streamer_(new streamer_imp<T>),o_(value) {} 
```

复制构造函数很简单；我们所需做的只是确保源 `any_out a` 中的 `streamer` 非零。

```cpp
any_out(const any_out& a)
  : streamer_(a.streamer_?a.streamer_->clone():0),o_(a.o_) {}<sup class="docfootnote">\[1\]</sup>

template<typename T> any_out& operator=(const T& r) {
  any_out(r).swap(*this);
  return *this;
}

any_out& operator=(const any_out& r) {
  any_out(r).swap(*this);
  return *this;
}

~any_out() {
  delete streamer_;
} 
```

> [1] Rob Stewart 问我写这一行是否为了在让人困惑的比赛中拿冠军，或者只是想写 ():0) 。我不能肯定，但可以肯定看到这一行你会开心….

`swap` 函数用于更容易地实现异常安全的赋值。

```cpp
any_out& swap(any_out& r) {
  std::swap(streamer_, r.streamer_);
  std::swap(o_,r.o_);
  return *this;
} 
```

现在，我们来增加那个让我们到此的东西：输出操作符。它应该接受一个 `ostream` 引用和一个 `any_out` 引用。被保存在 `any_out` 中的 `any` 将被传递给 streamer 的虚拟函数 `print` 。

```cpp
 friend std::ostream& operator<<(std::ostream& o,any_out& a) {
    if (a.streamer_) {
      a.streamer_->print(o,a.o_);
    }
    return o;
  }
}; 
```

这个类不仅提供了对包含在一个泛型类中的简单(未知)类型执行流输出的方法，它还示范了 `any` 是如何设计的。这种设计，以及这种用于安全地把一个类型包装在一个多态化的表面之后的技术，是通用的，被应用于其它很多地方。例如，它可以用于创建一个泛型的函数适配器。

让我们来测试一下我们的 `any_out` 类。

```cpp
int main() {
  std::vector<any_out> vec;

  any_out a(std::string("I do have operator<<"));

  vec.push_back(a);
  vec.push_back(112);
  vec.push_back(65.535);

  // 打印 vector vec 中的所有东西
  std::cout << vec[0] << "\n";
  std::cout << vec[1] << "\n";
  std::cout << vec[2] << "\n";

  a=std::string("This is great!");
  std::cout << a;
} 
```

如果类 `X` 不支持 `operator&lt;&lt;`, 这段代码就不能编译。不幸的是，这与我们是否真的使用了 `operator&lt;&lt;` 无关，它就是不能工作。`any_out` 总是要求输出操作符可用。

```cpp
 any_out nope(X());
  std::cout << nope;

} 
```

很方便，你不这样认为吗？如果在某个特定上下文中，你计划使用的所有类型有某个共同的操作可用，你可以象我们前面为 `any_out` 类增加对 `operator&lt;&lt;` 的支持那样加上它。推广这种方法和泛化该操作并不难，这可以用来扩展 `any` 可重用性的接口。

### 谓词

在我们结束关于 `any` 用法的这一节之前，让我们看一下如何围绕 `any` 来建立一些功能，来简化使用和增加表现力。`any` 可用于在容器类中保存不同的类型，它可以很容易地保存这些值，但很难去操作它们。

首先，我们创建两个谓词，`is_int` 和 `is_string`, 它们分别用于判断一个 `any` 是否包含一个 `int` 或一个 `string` 。在我们想在一个存有不同类型对象的容器中查找特定类型时，或者是想测试一个 `any` 的类型以决定后面的动作时，这很有用。实现方法是用 `any` 的成员函数 `type` 来测试。

```cpp
bool is_int(const boost::any& a) {
  return typeid(int)==a.type();
}

bool is_string(const boost::any& a) {
  return typeid(std::string)==a.type();
} 
```

这种办法可以工作，但为每一种我们想测试的类型写一个谓词会很乏味。实现的方法是重复的，这很适合用模板的方法来解决，如下。

```cpp
template <typename T> bool contains (const boost::any& a) {
  return typeid(T)==a.type();
} 
```

函数 `contains` 让我们不必手工创建新的谓词了。这是一个示范模板如何用来最小化冗余代码的典型例子。

### 对非空值计数

对于某些应用，可能要对容器中所有元素进行迭代并测试每个 `any` 是否含有值。一个空的 `any` 可能意味着要被删除，也可能我们要为了更一步的处理而取出所有非空的 `any` 元素。在一个算法中这是很常用到的，我们创建一个函数对象，它的函数调用操作符接受一个 `any` 参数。该操作符只是测试 `any` 是否为空，如果不是则递增计数器。

```cpp
class any_counter {
  int count_;
public:
  any_counter() : count_(0) {}

  int operator()(const boost::any& a) {
    return a.empty() ? count_ : ++count_;
  }

  int count() const { return count_; }
}; 
```

对于一个保存 `any` 的容器 `C` , 计算其中的非空值个数可以这样写。

```cpp
int i=std::for_each(C.begin(),C.end(),any_counter()).count(); 
```

注意，`for_each` 算法返回的是函数对象，所以我们可以很容易地取到计数值。因为 `for_each` 是以值的方式接受参数的，所以以下代码完成的不是同一件事情。

```cpp
any_counter counter;
std::for_each(C.begin(),C.end(),counter);
int i=counter.count(); 
```

第二个版本总是得到 0, 因为函数对象 counter 在调用 `for_each` 时被复制。第一个版本可以工作，因为返回的函数对象(`counter` 的一份拷贝)被用来取出计数值。

### 从容器中取出某种类型的元素

下面是另一个好东西：一个从容器中取出某种类型元素的提取器。在把异类容器中的一部分传递给一个同类 容器时，这是一个有用的工具。手工来做这件事是乏味且容易出错的，但一个简单的函数对象可以为我们照看好一切。我们泛化这个函数对象，用取出元素的输出迭 代器的类型，以及要从传递给该函数对象的 `any` 参数中取出的类型来参数化。

```cpp
template <typename OutIt,typename Type> class extractor {
  OutIt it_;
public:
  extractor(OutIt it) : it_(it) {}

  void operator()(boost::any& a) {
    Type* t(boost::any_cast<Type>(&a));
    if (t) {
      *it_++ = *t;
    }
  }
}; 
```

为了更方便地创建一个取出器, 这里有一个函数，它可以推断出输出迭代器的类型，并返回一个相应的取出器.

```cpp
template <typename Type, typename OutIt>
  extractor<OutIt,Type> make_extractor(OutIt it) {
    return extractor<OutIt,Type>(it);
  } 
```

### 使用谓词和取出器

现在该用一个例程来测试一下我们新的 `any` 同伴了。

```cpp
int main() {
  std::cout << "Example of using predicates and the "
   "function object any_counter\n";

  std::vector<boost::any> vec;
  vec.push_back(boost::any());
  for(int i=0;i<10;++i) {
    vec.push_back(i);
  }
  vec.push_back(boost::any()); 
```

我们把 12 个 `any` 对象加入到 `vec`, 现在我们想找出有多少个元素包含有值。为了计算含值元素的数量，我们使用前面创建的函数对象 `any_counter`。

```cpp
// 计算含有值的 any 实例的数量
int i=std::for_each(
  vec.begin(),
  vec.end(),
  any_counter()).count();
std::cout
  << "There are " << i << " non-empty any's in vec\n\n"; 
```

下面看操作一个 `any` 容器的取出器函数对象如何工作，它用来自源容器的特定类型的元素组成一个新的容器。

```cpp
 // 从 vec 中取出所有 int
std::list<int> lst;
std::for_each(vec.begin(),vec.end(),
  make_extractor<int>(std::back_inserter(lst)));
std::cout << "Found " << lst.size() << " ints in vec\n\n"; 
```

让我们清除容器 `vec` 中的内容，再加一些新的值。

```cpp
vec.clear();

vec.push_back(std::string("This is a string"));
vec.push_back(42);
vec.push_back(3.14); 
```

现在，我们试用一下已创建的谓词。首先，我们分别用两个谓词来显示 `any` 是否包含一个 `string` 或一个 `int` 。

```cpp
if (is_string(vec[0])) {
  std::cout << "Found me a string!\n";
}

if (is_int(vec[1])) {
  std::cout << "Found me an int!\n";
} 
```

正如我们前面指出的，为每一种我们要用到的类型定义一个谓词是乏味的，也是不必要的，我们只要简单地使用我们的语言优势。

```cpp
 if (contains<double>(vec[2])) {
    std::cout <<
      "The generic tool is sweeter, found me a double!\n";
  }
} 
```

运行这个例子，有如下输出。

```cpp
Example of using predicates and the function object any_counter
There are 10 non-empty any's in vec

Found 10 ints in vec

Found me a string!
Found me an int!
The generic tool is sweeter, found me a double! 
```

象以上这些小而简单的工具已经被证实是非常有用的。当然，不仅对 `any` 是这样；它是标准库容器和算法的设计中的一个特点。这些例子示范了如何与 `any` 一起使用组合函数。提供过滤、计数、操作特定类型等等，是隐藏实现细节的有效方法，并简单化了对 `any` 的使用。

### 遵守标准库适配器的要求

如果你觉得谓词 `contains` 很有用，你可能要注意它并不是到处都能用。它不能和标准库的适配器一起使用。下面的例子稍稍超出了本章的范围，但由于 `any` 是那么地适用于容器类，所以留下 `contains` 谓词的这点缺陷是不应该的。问题在于标准库的适配器(`bind1st`, `bind2nd`, `not1`, 和 `not2`)利用了它们所适配的谓词的一些必要条件。参数类型和结果类型必须用`typedef`暴露出来，这意味着我们需要的是函数对象而不是函数。

先来定义一个新的函数对象，`contains_t`. 它可以派生自辅助类 `std::unary_function` (它是 C++标准库的组成部分，以便创建正确的 `typedef`)，自动地定义参数类型和结果类型，但为了让事情更清楚，我们自己来提供所需的 `typedef` 。参数类型由 `const boost::any&` 改为 `boost::any`, 以避免产生到引用的引用，那是非法的。实现和前面的一样，这里只给出函数调用操作符。

```cpp
template <typename T> struct contains_t {
  typedef boost::any argument_type;
  typedef bool result_type;
  bool operator()(boost::any a) const {
    return typeid(T)==a.type();
  }
}; 
```

为了保留名字 `contains` 以用在稍后的辅助函数，我们用 `contains_t` 作这个函数对象的名字。这里有一个辅助函数用来创建并返回一个自动设置为相应类型的 `contains_t` 实例。原因是我们想重载 `contains` ，以便我们还可以提供我们原来创建的谓词。

```cpp
template <typename T> contains_t<T> contains() {
  return contains_t<T>();
} 
```

最后，旧的谓词被改为利用 `contains_t` 来实现。现在，如果我们为了某些原因要改变 `contains_t` 的实现，`contains` 可以反应出这些修改而无须更多的改进。

```cpp
template <typename T> bool contains(const boost::any& a) {
  return contains_t<T>()(a);
} 
```

下面这个例程示范了我们已经得到的东西，包括新的函数对象和前例中的谓词。

```cpp
int main() {
  std::cout << "Example of using the improved is_type\n";

  std::vector<boost::any> vec;

  vec.push_back(std::string("This is a string"));
  vec.push_back(42);
  vec.push_back(3.14); 
```

使用的谓词与前面没有什么不同。测试一个 `any` 是否某种类型仍然很容易。

```cpp
if (contains<double>(vec[2])) {
  std::cout << "The generic tool has become sweeter! \n";
}

vec.push_back(2.52f);
vec.push_back(std::string("Another string")); 
```

另一个使用 `contains` 的例子是，在一个容器中查找某种类型。这个例子查找第一个 `float`.

```cpp
std::vector<boost::any>::iterator
  it=std::find_if(vec.begin(),vec.end(),contains<float>()); 
```

现在，从一个中取回所含值的两种方法都被示范出来。通过 `const` 引用传递 `any` 给 `any_cast` 的是异常抛出版本。传递 `any` 地址的版本则返回一个所存值的指针。

```cpp
if (it!=vec.end()) {
  std::cout << "\nPrint the float twice!\n";
  std::cout << boost::any_cast<float>(*it) << "\n";
  std::cout << *boost::any_cast<float>(&*it) << "\n";
}
std::cout <<
  "There are " << vec.size() << " elements in vec\n"; 
```

我还没有给出一个好的例子来说明为什么 `contains` 应该是一个发育完全的函数对象。在很多情形下，原因可能无法预先知道，因为我们不能预见我们的实现将会面对的每一种情形。一个强烈的原因是遵守标准库的要求，更适用于我们已知的用例以外的情形。然而，我还是给你一个例子：任务是从一个容器 `vec` 中删除所有不含 `string` 的元素。当然，另写一个谓词来做与 `contains` 相反的事情是一种方法，但这样会很快导致维护的恶梦，因为类似作用的函数对象要不断增生。标准库提供给我们一个名为 `not1` 的适配器，它对一个函数对象的结果取反，它可以轻易地从我们的 `vector vec` 中清除所有非 `string` 元素。

```cpp
 vec.erase(std::remove_if(vec.begin(),vec.end(),
    std::not1(contains<std::string>())),vec.end());

  std::cout << "Now, there are only " << vec.size()
    << " elements left in vec!\n";
} 
```

本节的例子示范了如何有效地使用 `any`. 因为所存值的类型不是 `any` 的类型的组成部分，要在不对所存类型强加要求(包括从同一基类派生)的前提下提供存储，`any` 是一个基本工具。我们已经看到这个类型隐藏有某种价值。`any` 不允许在对值的类型不了解的情况下访问所保存的值，限制了对所存值进行操作的机会。作为大的扩展，通过创建一些辅助类——谓词和函数对象——提供所需的逻辑来访问所存值，可以进行补偿。

# Any 总结

## Any 总结

这个类型可以包含不同类型的值，而且与无类类型(如 `void*`)有很大不同。我们总是严重地依赖 C++中的类型安全，只有在极少数情形下我们会愿意没有它来干活。

这是有很好的原因的：类型安全防止我们犯错，并改善了我们代码的性能。因此，我们应该避免无类类型。还有，发现自己需要异类存储的情形很少见，或者为了将使用者隔离于类型的细节，或者为了在更低的层次获得极度的灵活性。`any` 提供了这些功能，同时维护了类型安全，它是我们的工具箱的最好扩充！

在以下情形时使用 Any 库：

*   你需要在容器中存放不同类型的值

*   需要保存未知类型

*   类型被传递到无须知晓任何有关该类型信息的层次

Any 的设计同时也是一门很有价值的课程，关于如何封装一个类型而不影响到该类型的封套类。这种设计可以用于创建泛型函数对象、泛型迭代器等等。它是一个展示封装的威力以及与模板相关的多态性的例子。

在标准库中，有很好的工具来存放多个元素。当需要存储异类的元素时，我们想避免使用新的集合类型。`any` 提供了一种方法，在大多数情况下它可以与已有容器一起使用。在某种程度上，模板类 `any` 扩展了标准库容器的能力，把不同的类型封入一个同类型的包装中，就可以把它们放入前述容器中了。

把 Boost.Any 加到已有代码中是很简单的。它不需要修改设计，并且立即就增加了灵活性。接口非常小，这使得它成为一个很容易理解的工具。

Any 库由 Kevlin Henney 创建，与所有 Boost 库一样，它由 Boost 社区复审、改进和强化。