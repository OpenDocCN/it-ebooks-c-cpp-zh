# Library 8\. Tuple

*   Tuple 库如何改进你的程序？
*   Tuple 库如何适用于标准库？
*   Tuple
*   用法
*   Tuple 总结

# Tuple 库如何改进你的程序？

## Tuple 库如何改进你的程序？

*   从函数返回多个返回值

*   相关类型的组合

*   将数值组合起来

与许多其它的编程语言一样，C++允许函数返回一个数值。但是，这一个数值可以是任意的类型，你可以用一个 `struct` 或 `class` 把多个数值组合起来作为结果。虽然可以，但是用这样的结构来组合相关的返回值通常都是很不方便的，因为这意味着要为对一种返回类型进行定义。为了避免在返回值中拷贝大量的对象，同时也为了避免创建一个特殊的类型用于从函数返回多个数值，我们常常使用非 `const` 引用参数或者指针参数，从而允许函数通过这些参数设置调用者的变量。在多数情况下这样做都工作良好，但也有人不愿意使用输出参数。还有，输出参数不能明确指出返回值就是返回值。有些时候，`std::pair` 可以满足要求，但在需要返回两个以上数值时，它就不能满足要求了。

为了提供多个返回值，我们需要一个 tuple 结构。一个 tuple 是一个固定大小的、多个指定类型的数值的聚集。相应的例子包括有：pairs, triples, quadruples, 等等。有些语言本身就内建有这样的 tuple 类型，但 C++没有。借助 C++本身的强大功能，这一缺点可以通过库来弥补，如你所想， Boost.Tuple 正是这样的一个库。

Tuple 库提供了 tuple 结构，它可以方便地用于返回多个数值，也可以组合任意的类型并以泛型代码来操作它们。

# Tuple 库如何适用于标准库？

## Tuple 库如何适用于标准库？

标准库提供了一个 tuple 的特例，一个 2-tuple, 名为 `std::pair`. 这个结构被用于标准库的容器，你可能在操作 `std::map` 的元素时已经留意到了。你也可以在容器类中存储 `pair`。当然，`std::pair` 不仅是为了给容器类使用的，它还有它自己的用途，它附带有一个方便的函数 `std::make_pair`, 可以自动地进行类型推断，还有一组操作符用于 `pair` 的比较。一个 tuple 的通常解决方案，而不仅仅是 2-tuples，会更加有用。Tuple 库所提供的还不是完全通用的，它最多可以允许 10 个元素的 tuple (如果需要更多的，看起来不常见但也不是没有可能的，这个限制可以放松)。还有，这些 tuples 的效率与使用 `struct` 的手工解决方案同样高！

# Tuple

## Tuple

### 头文件: `"boost/tuple/tuple.hpp"`

它包含了 `tuple` 类模板及库的核心部分。

```cpp
Header: "boost/tuple/tuple_io.hpp" 
```

包含了对 `tuple` 的输入输出操作符。

```cpp
Header: "boost/tuple/tuple_comparison.hpp" 
```

包含了 `tuple` 的关系操作符。

Tuple 库位于 `boost` 里的嵌套名字空间 `boost::tuples` 中。要使用 tuples, 需要包含 "`boost/tuple/tuple.hpp"`, 它包含了核心库。要进行输入输出操作，就包含 `"boost/tuple/tuple_io.hpp"`, 要支持 `tuple` 的比较，就包含 `"boost/tuple/tuple_comparison.hpp"`. 有些 Boost 库提供一个包含了所有相关库的头文件以方便使用；但 Boost.Tuple 没有。原因是把库分到各个不同的头文件中可以减少编译时间；如果你不使用关系操作符，你就无须为此付出时间和依赖性的代价。为了方便使用，Tuple 库中有些名字位于名字空间 `boost`：如 `tuple`, `make_tuple`, `tie`, 和 `get`. 以下是 Boost.Tuple 的部分摘要，列出并简要讨论了最主要的一些函数。

```cpp
namespace boost {

  template <class T1,class T2,...,class TM> class tuple {
  public:
    tuple();

    template <class P1,class P2...,class PM> 
      tuple(class P1,class P2,...,PN); 

    template <class U1,class U2,...,class UN>
    tuple(const tuple<U1,U2,...,UN>&);

    tuple& operator=(const tuple&);
  };

  template<class T1,class T2,...,class TN> tuple<V1,V2,...,VN> 
    make_tuple(const T1& t1,const T2& t2,...,const TN& tn);

  template<class T1,class T2,...,class TN> tuple<T1&,T2&,...,TN> 
    tie(T1& t1,T2& t2,...,TN& tn);

  template <int I,class T1,class T2,...,class TN> 
    RI get(tuple<T1,T2,...,TN>& t);

  template <int I,class T1,class T2,...,class TN> 
    PI get(const tuple<T1,T2,...,TN>& t);

  template <class T1,class T2,...,class TM,
           class U1,class U2,...,class UM>
    bool operator==(const tuple<T1,T2,...,TM>& t,
                    const tuple<U1,U2,...,UM>& u);

  template <class T1,class T2,...,class TM,
           class U1,class U2,...,class UM>
    bool operator!=(const tuple<T1,T2,...,TM>& t,
                   const tuple<U1,U2,...,UM>& u);

  template <class T1,class T2,...,class TN,
           class U1,class U2,...,class UN>
    bool operator<(const tuple<T1,T2,...,TN>&, 
                  const tuple<U1,U2,...,UN>&);
} 
```

### 成员函数

```cpp
tuple(); 
```

`tuple` 的缺省构造函数初始化所有元素，这意味着这些元素必须是可以缺省构造的，它们必须有一个公有的缺省构造函数。任何从这些所含元素的构造函数抛出的异常都会被传播。

```cpp
template <class P1,class P2...,class PM> 
  tuple(class P1,class P2,...,PN); 
```

这个构造函数接受一些参数，用于初始化 `tuple` 相应元素。对于一些带有非缺省构造类型的 `tuple` ，就需要用这种构造方式；不能缺省构造一个 `tuple` 而不构造它的所有元素。例如，引用类型的元素必须在构造时初始化。注意，参数的数量不必与 tuple 类型中的元素数量一致。可以仅给出部分元素的值，而让剩余元素初始化为缺省值。任何从元素的构造函数抛出的异常都会被传播。

```cpp
template <class U1,class U2,...,class UN>
  tuple(const tuple<U1,U2,...,UN>&); 
```

这个构造函数用来自另一个 tuple 的元素来进行初始化，要求被构造的 tuple (`T1`, `T2`,…,`TM`) 的每一个元素都必须可以从 (`U1`,`U2`,…,`UN`) 构造。任何从元素的构造函数抛出的异常都会被传播。

```cpp
TIndex & get<int Index>();
const TIndex & get<int Index>() const; 
```

返回位于给定的 `Index` 处的元素的引用。`Index` 必须是一个常量整型表达式；如果索引大于或等于 `tuple` 中的元素数量，将产生一个编译期错误。结果的类型通过相应的模板参数 `TIndex` 给出。

```cpp
tuple& operator=(const tuple& other); 
```

tuple 的赋值要求两个 tuples 具有相同的长度和元素类型。`*this` 中的每一个元素被赋值为 `other` 的对应元素。元素赋值中的任何异常都会被传播。

### 普通函数

```cpp
template<class T1,class T2,...,class TN> tuple<V1,V2,...,VN> 
  make_tuple(const T1& t1,const T2& t2,...,const TN& tn); 
```

函数模板 `make_tuple` 是 `tuple` 版本的 `std::make_pair`. 它使用函数模板参数推断来决定一个包含这些参数的 `tuple` 的元素类型。创建这个 `tuple` 的元素类型时不使用这些参数的高级 cv-限定符。要控制对引用类型的类型推断，可以使用 Boost.Ref 的工具 `ref` 和 `cref` 来包装这些参数，从而影响返回的 tuple 结果类型。(稍后我们将看到关于 `ref` 和 `cref` 的更多内容)

```cpp
template<class T1,class T2,...,class TN> tuple<T1&,T2&,...,TN> 
  tie(T1& t1,T2& t2,...,TN& tn); 
```

函数模板 `tie` 类似于 `make_tuple`. 调用 `tie(t1,t2,...,tn)` 等同于调用 `make_tuple(ref(t1),ref(t2)... ref(tn))`，即它创建一个由函数参数的引用组成的 `tuple` 。实际结果是把一个 `tuple` 赋值为由 `tie` 创建的对象，拷贝源 `tuple` 的元素到 `tie` 的参数。这样，`tie` 可以很容易地从一个由函数返回的 `tuple` 拷贝值到一个已有变量中。你也可以让 `tie` 从一个 `std::pair` 创建一个 2-tuple 。

```cpp
template <int I,class T1,class T2,...,class TN> 
  RI get(tuple<T1,T2,...,TN>& t); 
```

这个函数 `get` 的重载版本用于取出 `tuple t` 的一个元素。索引 `I` 必须位于范围 [0..N), N 为 `tuple` 中的元素数量。如果 `TI` 是一个引用类型，则`RI` 为 `TI`; 否则, `RI` 为 `TI&`.

```cpp
template <int I,class T1,class T2,...,class TN> 
  RI get(const tuple<T1,T2,...,TN>& t); 
```

这个函数 `get` 用于取出 `tuple t` 的一个元素。索引 `I` 必须位于范围 [0..N), N 为 `tuple` 中的元素数量。如果 `TI` 是一个引用类型，则`RI` 为 `TI`; 否则, `RI` 为 `const TI&`.

### 关系操作符

```cpp
bool operator==(
  const tuple<T1,T2,...,TN>& lhs, 
  const tuple<U1,U2,...,UN>& rhs); 
```

如果对于所有位于范围[0..N)的 `i`，都有 `get&lt;i&gt;(lhs)==get&lt;i&gt;(rhs)` ，N 为元素数量，则相等操作符返回 `true` 。这两个 `tuple`s 必须具有相同数量的元素。对于 N=0 的空 `tuple` ，总是返回 `true` 。

```cpp
bool operator!=(
  const tuple<T1,T2,...,TN>& lhs, 
  const tuple<U1,U2...,...,>& rhs); 
```

如果对于任意一个位于范围[0..N)的 `i`，有 `get&lt;i&gt;(lhs)!=get&lt;i&gt;(rhs)` ，N 为元素数量，则不等操作符返回 `true` 。这两个 `tuple`s 必须具有相同数量的元素。对于 N=0 的空 `tuple` ，总是返回 `false` 。

```cpp
bool operator<(
  const tuple<T1,T2,...,TN>& lhs, 
  const tuple<U1,U2,...,UN>& rhs); 
```

如果对于任意一个位于范围[0..N)的 `i`，有 `get&lt;i&gt;(lhs)&lt;get&lt;i&gt;(rhs)` ，N 为元素数量，则小于操作符返回 `true` ；假如对每个比较都返回 `false`，则表达式 `!(get&lt;i&gt;(rhs)&lt;get&lt;i&gt;(lhs))` 为 `true`；否则表达式为 `false`。这两个 `tuple`s 必须具有相同数量的元素。对于 N=0 的空 `tuple` ，总是返回 `true` 。

值得注意的是，对于所有支持的关系操作符(operators `==`, `!=`, `&lt;`, `&gt;`, `&lt;=`, 和 `&gt;=`), 两个 `tuple`s 必须有相同的约束。首先，它们必须有相同的长度。其次，两个 `tuple` 间的每对元素(第一个对第一个，第二个对第二个，等等)必须支持同一个关系操作符。当这些约束被满足时，`tuple` 的操作符才可以实现，它按顺序比较每一对元素，即关系操作符是短路(short-circuited)的，一旦有了明确结果就马上返回。操作符 `&lt;`, `&gt;`, `&lt;=`, 和 `&gt;=` 执行字典序的比较，并要求元素对执行同样的操作。元素对的比较操作符产生的任何异常都会被传播，但 `tuple` 操作符本身不抛出异常。

# 用法

## 用法

Tuples 位于名字空间 `tuples`, 后者又位于名字空间 `boost`. 使用这个库要包含头文件 `"boost/tuple/tuple.hpp"`。关系操作符的定义在头文件 `"boost/tuple/tuple_comparison.hpp"`中。tuples 的输入输出定义在头文件 `"boost/tuple/tuple_io.hpp"`中。`tuple` 的一些关键部件(`tie` 和 `make_tuple`)也可以直接在名字空间 `boost` 中使用。在本节中，我们将讨论如何在一些常见情形下使用 tuples ，以及如何可以扩展这个库的功能以最好地符合我们的意图。我们将从构造 tuples 开始，并逐渐转移到其它主题，包括如何利用 tuples 的细节。

### 构造 Tuples

构造一个 `tuple` 包括声明各种类型，并可选地提供一组兼容类型的初始值。[1]

> [1] 在特化时 `tuple` ，构造函数的参数不必与元素的类型精确相同，只要它们可以隐式地转换为元素的类型就可以了。

```cpp
boost::tuple<int,double,std::string> 
  triple(42,3.14,"My first tuple!"); 
```

类模板 `tuple` 模板参数指定了元素的类型。前面这个例子示范了一个带有三个类型的 `tuple` 的创建：一个 `int`, 一个 `double`, 和一个 `std::string`. 并向构造函数提供了三个参数来初始化所有三个元素的值。也可以传递少于元素数量的参数，那样的话剩下的元素将被缺省初始化。

```cpp
boost::tuple<short,int,long> another; 
```

在这个例子中，`another` 有类型为 `short`, `int`, 和 `long` 的元素，并且它们都被初始化为 0.[2] 不管你的 `tuple` 是什么类型，这就是它如何定义和构造的方式。所以，如果你的 `tuple` 有一个元素类型不能缺省构造，你就需要自己初始化它。与定义 `struct` 相比，`tuple` 更容易声明、定义和使用。还有一个便于使用的函数，`make_tuple`，它使得创建 `tuple`s 更加容易。它自动推断元素的类型，不用你来重复指定(这也会是出错的机会！)。

> [2] 在一个模板上下文中，`T()` 对于一个内建类型而言意味着初始化为零。

```cpp
boost::tuples::tuple<int,double> get_values() {
  return boost::make_tuple(6,12.0);
} 
```

函数 `make_tuple` 类似于 `std::make_pair`. 缺省情况下，`make_tuple` 设置元素类型为非`const`, 非引用的，即是最简单的、根本的参数类型。例如，考虑以下变量：

```cpp
int plain=42;
int& ref=plain;
const int& cref=ref; 
```

这三个变量根据它们的 cv 限定符(常量性)以及是否引用来命名。通过调用以下 `make_tuple` 创建的 `tuple` 都带有一个 `int` 元素。

```cpp
boost::make_tuple(plain);
boost::make_tuple(ref);
boost::make_tuple(cref); 
```

这种行为不总是正确的，但通常是，这正是为什么它是缺省行为的原因。为了使一个 `tuple` 的元素设为引用类型，你要使用函数 `boost::ref`, 它来自另一个名为 Boost.Ref 的 Boost 库。以下三行代码使用了我们前面定义的三个变量，但这次 `tuple` 带有一个 `int&` 元素，除了最后一个，它带的是一个 `const int&` 元素 (我们不能去掉 `cref` 的常量性)：

```cpp
boost::make_tuple(boost::ref(plain));
boost::make_tuple(boost::ref(ref));
boost::make_tuple(boost::ref(cref)); 
```

如果元素需要是 `const` 引用的，就使用来自 Boost.Ref 的 `boost::cref`。下面三个 tuples 带有一个 `const int&` 元素：

```cpp
boost::make_tuple(boost::cref(plain));
boost::make_tuple(boost::cref(ref));
boost::make_tuple(boost::cref(cref)); 
```

`ref` 和 `cref` 在其它地方也经常使用。事实上，它们原先是作为 Boost.Tuple 库的一部分而建立的，但后来因为它们的通用性而移出去成为一个独立的库。

### 访问 tuple 元素

一个 `tuple` 的元素可以通过 `tuple` 成员函数 `get` 或普通函数 `get` 来访问。它们都要求用一个常量整型表达式来指定要取出的元素的索引。

```cpp
#include <iostream>
#include <string>

#include "boost/tuple/tuple.hpp"

int main() {
  boost::tuple<int,double,std::string> 
    triple(42,3.14,"The amazing tuple!"); 

  int i=boost::tuples::get<0>(triple);
  double d=triple.get<1>();
  std::string s=boost::get<2>(triple);
} 
```

这个例子中，一个三元素的 `tuple` 取名为 `triple` 。`triple` 含有一个 `int`, 一个 `double`, 和一个 `string`, 它们可以用 `get` 函数取出。

```cpp
int i=boost::tuples::get<0>(triple); 
```

这里，你看到的是普通函数 `get` 。它把 `tuple` 作为一个参数。注意，给出一个无效的索引会导致一个编译期错误。这个函数的前提是索引值对于给定的 `tuple` 类型必须有效。

```cpp
double d=triple.get<1>(); 
```

这段代码使用的是成员函数 `get`. 它也可以写成这样：

```cpp
double& d=triple.get<1>(); 
```

这个绑定到一个引用的方式可以使用，因为 `get` 总是返回一个到元素的引用。如果 `tuple`, 或者其类型，是 `const` 的， 则返回一个 `const` 引用。这两个函数是等价的，但在某些编译器上，只有普通函数可以正确工作。普通函数有一个优点，它提供了与 `tuple` 之外的其它类型一致的提取元素的风格。通过索引来访问 `tuple` 的元素而不是通过名字来访问，这样做的一个优点是它可以支持泛型的解决方法，因为这样做不依赖于某个特定的名字，仅仅是一个索引值。稍后对此有更多介绍。

### Tuple 赋值及复制构造

`tuple`s 可以被赋值和被复制构造，可以在两个 `tuple` 间进行，只要它们的元素类型可以相互转换。要赋值或复制 `tuple`s, 就是执行成员间的赋值或复制，因此这两个 `tuple`s 必须具有相同数量的元素。源 `tuple` 的元素必须可以转换为目标 `tuple` 的元素。以下例子示范了如何使用。

```cpp
#include <iostream>
#include <string>

#include "boost/tuple/tuple.hpp"

class base {
public:
  virtual ~base() {};
  virtual void test() {
    std::cout << "base::test()\n";
  }
};

class derived : public base {
public:
  virtual void test() {
    std::cout << "derived::test()\n";
  }
};

int main() {
  boost::tuple<int,std::string,derived> tup1(-5,"Tuples");
  boost::tuple<unsigned int,std::string,base> tup2;
  tup2=tup1;

  tup2.get<2>().test();
  std::cout << "Interesting value: " 
    << tup2.get<0>() << '\n';

  const boost::tuple<double,std::string,base> tup3(tup2);
  tup3.get<0>()=3.14;
} 
```

这个例子开始时定义两个类，`base` 和 `derived`, 它们被用作两个 `tuple` 类型的元素。第一个 `tuple` 有三个元素，类型为 `int`, `std::string`, 和 `derived`. 第二个 `tuple` 有三个兼容类型的元素，分别为 `int`, `std::string`, 和 `base`. 因此，这两个 `tuple`s 符合赋值的要求，这就是为什么 `tup2=tup1` 有效的原因。在这个赋值中，`tup1` 的第三个元素类型为 `derived`, 被赋值给 `tup2` 的第三个元素，后者类型为 `base`. 赋值可以成功，但 `derived` 对象被切割，因此这样会破坏多态性。

```cpp
tup2.get<2>().test(); 
```

这一行取出一个 `base&`, 但 `tup2` 中的对象类型为 `base`, 因此它将调用 `base::test`. 我们可以通过把 `tuple`s 修改为分别包含 `base` 和 `derived` 的引用或指针来获得多态行为。注意数值转换的危害(精度损失、正负溢出)也会出现在 `tuple`s 间的转换。这种危险的转换可以通过 Boost.Conversion 库的帮助来变得安全，请参见"Library 2: Conversion"。

下一行是复制构造一个新的 `tuple`, `tup3`, 它的类型不同但还是兼容于 `tup2`.

```cpp
const boost::tuple<double,std::string,base> tup3(tup2); 
```

注意，`tup3` 被声明为 `const`. 这意味着本例中有一个错误。看你能否找到它。我等一下你...，你看出来了吗？就是这里：

```cpp
tup3.get<0>()=3.14; 
```

因为 `tup3` 是 `const`, `get` 将返回一个 `const double&`. 这意味着这个赋值语句是非法的，这个例子不能编译。`tuple`s 间的赋值与复制构造是符合直觉的，因为它的语义与单个元素是相同的。通过下面这个例子，我们来看看如何在 `tuple`s 间的派生类至基类的赋值中获得多态行为。

```cpp
derived d;
boost::tuple<int,std::string,derived*> 
  tup4(-5,"Tuples",&d);
boost::tuple<unsigned int,std::string,base*> tup5;
tup5=tup4;

tup5.get<2>()->test();

boost::tuple<int,std::string,derived&>
  tup6(12,"Example",d); 

boost::tuple<unsigned int,std::string,base&> tup7(tup6);

tup7.get<2>()->test(); 
```

在这两种情况下，都会调用 `derived::test` ，这正是我们想要的。`tup6` 和 `tup7` 间不能赋值，因为你不能对一个引用进行赋值，这就是为什么 `tup7` 要从 `tup6` 复制构造，以及 `tup6` 要用 `d` 进行初始化的原因。因为 `tup4` 和 `tup5` 对它们的第三个元素使用的是指针，因此它们可以支持赋值。注意，通常在 tuple 中最好使用智能指针(与裸指针相比)，因为它们可以减轻对指针所指向的资源的生存期管理的压力。但是，正如 `tup4` 和 `tup5` 所示，在 `tuple`s 中，指针不总是指向需要进行内存管理的东西的。(参考 "Library 1: Smart_ptr 1"，可获得 Boost 强大的智能指针的更多信息)

### 比较 Tuples

要比较 `tuple`s, 你必须包含头文件 `"boost/tuple/tuple_comparison.hpp"`. `tuple` 的关系操作符有 `==`,`!=`,`&lt;`,`&gt;`,`&lt;=` 和 `&gt;=`, 它们按顺序地对要比较的 `tuple`s 中的每一对元素调用相应的操作符。这些比较是短路的，即只比较到可以得到正确结果为止。只有具有相同数量元素的 `tuple`s 可以进行比较，并且显然两个 `tuple`s 的对应的元素必须是可比较的。如果两个 `tuple`s 的所有元素对都相等，则相等比较返回 `true` 。如果任意一对元素的相等比较返回 `false`, `operator==` 也将返回 `false`。不等比较也是类似的，但返回的是相反的结果。其它关系操作符按字典序进行比较。

以下例程示范比较操作符的行为。

```cpp
#include <iostream>
#include <string>

#include "boost/tuple/tuple.hpp"
#include "boost/tuple/tuple_comparison.hpp"

int main() {
  boost::tuple<int,std::string> tup1(11,"Match?");
  boost::tuple<short,std::string> tup2(12,"Match?");

  std::cout << std::boolalpha;

  std::cout << "Comparison: tup1 is less than tup2\n";

  std::cout << "tup1==tup2: " << (tup1==tup2) << '\n'; 
  std::cout << "tup1!=tup2: " << (tup1!=tup2) << '\n';
  std::cout << "tup1<tup2: " << (tup1<tup2) << '\n';
  std::cout << "tup1>tup2: " << (tup1>tup2) << '\n';
  std::cout << "tup1<=tup2: " << (tup1<=tup2) << '\n';
  std::cout << "tup1>=tup2: " << (tup1>=tup2) << '\n';

  tup2.get<0>()=boost::get<0>(tup1); //tup2=tup1 also works

  std::cout << "\nComparison: tup1 equals tup2\n"; 

  std::cout << "tup1==tup2: " << (tup1==tup2) << '\n'; 
  std::cout << "tup1!=tup2: " << (tup1!=tup2) << '\n';
  std::cout << "tup1<tup2: " << (tup1<tup2) << '\n';
  std::cout << "tup1>tup2: " << (tup1>tup2) << '\n';
  std::cout << "tup1<=tup2: " << (tup1<=tup2) << '\n';
  std::cout << "tup1>=tup2: " << (tup1>=tup2) << '\n';
} 
```

如你所见，这两个 `tuple`s, `tup1` 和 `tup2`, 并不是严格相同的类型，但它们的类型是可比较的。在第一组比较中，`tuple`s 的第一个元素值不同，而在第二组中，`tuple`s 是相同的。以下是程序的运行输出。

```cpp
Comparison: tup1 is less than tup2
tup1==tup2: false
tup1!=tup2: true
tup1<tup2: true
tup1>tup2: false
tup1<=tup2: true
tup1>=tup2: false

Comparison: tup1 equals tup2
tup1==tup2: true
tup1!=tup2: false
tup1<tup2: false
tup1>tup2: false
tup1<=tup2: true
tup1>=tup2: true 
```

支持比较的一个重要方面是，`tuple`s 可以被排序，这意味着它们可以在关联容器中被排序。有些时候，我们需要按 `tuple` 中的某一个元素进行排序(建立一个弱序)，我们可以用一个简单的泛型方法来实现。

```cpp
template <int Index> class element_less {
public:
  template <typename Tuple> 
  bool operator()(const Tuple& lhs,const Tuple& rhs) const {
    return boost::get<Index>(lhs)<boost::get<Index>(rhs); 
  } 
}; 
```

这显示了使用索引而不是用名字来访问元素的优势；它可以很容易地创建泛型的构造来执行强大的操作。我们的 `element_less` 可以这样用：

```cpp
#include <iostream>
#include <vector> 
#include "boost/tuple/tuple.hpp"
#include "boost/tuple/tuple_comparison.hpp"

template <int Index> class element_less {
public:
  template <typename Tuple> 
  bool operator()(const Tuple& lhs,const Tuple& rhs) const {
    return boost::get<Index>(lhs)<boost::get<Index>(rhs); 
  } 
};

int main() {
  typedef boost::tuple<short,int,long,float,double,long double> 
    num_tuple;

  std::vector<num_tuple> vec;

  vec.push_back(num_tuple(6,2));
  vec.push_back(num_tuple(7,1));
  vec.push_back(num_tuple(5));

  std::sort(vec.begin(),vec.end(),element_less<1>());

  std::cout << "After sorting: " << 
    vec[0].get<0>() << '\n' <<
    vec[1].get<0>() << '\n' <<
    vec[2].get<0>() << '\n';
} 
```

`vec` 由三个元素组成。使用从我们前面创建的模板所特化的 `element_less&lt;1&gt;` 函数对象，来执行基于 `tuple`s 的第二个元素的排序。这类函数对象还有更多的应用，如用于查找指定的 `tuple` 元素。

### 绑定 Tuple 元素到变量

Boost.Tuple 库的一个方便的特性是"绑定" `tuple`s 到变量。绑定者就是用重载函数模板 `boost::tie` 所创建的 `tuple`s，它的所有元素都是非`const` 引用类型。因此，`tie`s 必须使用左值进行初始化，从而 `tie` 的参数也必须是非 `const` 引用类型。由于结果 `tuple`s 具有非 `const` 引用类型，对这样一个 `tuple` 的元素进行赋值，就会通过非 `const` 引用赋值给调用 `tie` 时的左值。这样就绑定了一个已有变量给 `tuple`, tie 的名字由此而来！

以下例子首先示范了一个通过返回 `tuple` 获得值的明显的方法。然后，它示范了通过一个 `tie`d `tuple` 直接赋值给变量以完成相同操作的方法。为了让这个例子更为有趣，我们开始时定义了一个返回两个数值的最大公约数和最小公倍数的函数。当然，这两个结果值被组合成一个 `tuple` 返回类型。你将发现计算最大公约数和最小公倍数的函数来自于另一个 Boost 库——Boost.Math.

```cpp
#include <iostream>
#include "boost/tuple/tuple.hpp"
#include "boost/math/common_factor.hpp"

boost::tuple<int,int> gcd_lcm(int val1,int val2) {
  return boost::make_tuple(
    boost::math::gcd(val1,val2),
    boost::math::lcm(val1,val2));
}

int main() {
  //"老"方法
  boost::tuple<int,int> tup;
  tup=gcd_lcm(12,18);
  int gcd=tup.get<0>();  // 译注：原文为 int gcd=tup.get<0>()); 明显有误
  int lcm=tup.get<1>();  // 译注：原文为 int gcd=tup.get<1>()); 明显有误

  std::cout << "Greatest common divisor: " << gcd << '\n';
  std::cout << "Least common multiple: " << lcm << '\n';

  //"新"方法
  boost::tie(gcd,lcm)=gcd_lcm(15,20);

  std::cout << "Greatest common divisor: " << gcd << '\n';
  std::cout << "Least common multiple: " << lcm << '\n';
} 
```

有时我们并不是对返回的 tuple 中所有的元素感兴趣，`tie` 也可以支持这种情况。有一个特殊的对象，`boost:: tuples::ignore`，它忽略一个 `tuple` 元素的值。如果前例中我们只对最大公约数感兴趣，我们可以这样写：

```cpp
boost::tie(gcd,boost::tuples::ignore)=gcd_lcm(15,20); 
```

另一种方法是创建一个变量，传递给 `tie`, 然后在后面的处理中忽略它。这样做会令维护人员弄不清楚这个变量为什么存在。使用 `ignore` 可以清楚地表明代码将不使用 `tuple` 的那个值。

注意，`tie` 也支持 `std::pair`. 用法与从 `boost::tuple`s 绑定值一样。

```cpp
std::pair<short,double> p(3,0.141592);
short s;
double d;

boost::tie(s,d)=p; 
```

绑定 `tuple`s 不仅仅是方便使用；它有助于使代码更为清晰。

### Tuples 的流操作

在本章的每一个例子中，取出 `tuple`s 的元素都只是为了能够把它们输出到 `std::cout`. 可以象前面那样做，但还有更容易的方法。`tuple` 库支持输入和输出流操作；`tuple` 重载了`operator&gt;&gt;` 和 `operator&lt;&lt;`。还有一些操纵器用于改变输入输出的缺省分隔符。对输入操作改变分隔符改变 `operator&gt;&gt;` 查找元素值的结果。我们用一个简单的读写 `tuple`s 的程序来测试一下这些情况。注意，要使用 `tuple` 的流操作，你必须包含头文件 `"boost/tuple/tuple_io.hpp"`.

```cpp
#include <iostream>
#include "boost/tuple/tuple.hpp"
#include "boost/tuple/tuple_io.hpp"

int main() {
  boost::tuple<int,double> tup1;
  boost::tuple<long,long,long> tup2;

  std::cout << "Enter an int and a double as (1 2.3):\n";
  std::cin >> tup1;

  std::cout << "Enter three ints as |1.2.3|:\n";
  std::cin >> boost::tuples::set_open('|') >>
  boost::tuples::set_close('|') >>
  boost::tuples::set_delimiter('.') >> tup2;

  std::cout << "Here they are:\n"
    << tup1 << '\n'
    << boost::tuples::set_open('\"') <<
  boost::tuples::set_close('\"') <<
  boost::tuples::set_delimiter('-');

  std::cout << tup2 << '\n';
} 
```

上面这个例子示范了如何对 `tuple`s 使用流操作符。`tuple`s 的缺省分隔符是：`(` (左括号) 作为开始分隔符，`)` (右括号) 作为结束分隔符，空格用于分隔各个 `tuple` 元素值。这意味着我们的程序要正确运行的话，我们需要象这样输入：`(12 54.1)` 和 `|4.5.3|`. 以下是运行的例子。

```cpp
Enter an int and a double as (1 2.3):
(12 54.1)
Enter three ints as |1.2.3|:
|4.5.3|
Here they are:
(12 54.1)
"4-5-3" 
```

对流操作的支持是很方便的，通过对分隔符操纵器的支持，可以很容易让使用 `tuple` 的代码的流操作兼容于已有代码。

### 关于 Tuples 的更多

还有很多我们没有看到的工具可用于 `tuple`s。这些更为先进的特性对于创建使用 `tuple` 的泛型结构至为重要。例如，你可以获得一个 `tuple` 的长度(即元素的数量)，取出某个元素的类型，使用 `null_type tuple` 哨兵来终结递归模板实例化。

不可能用一个 `for` 循环来迭代一个 `tuple` 里的元素，因为 `get` 要求提供一个常量整型表达式。但是，使用模板元编程，我们可以打印一个 tuple 的所有元素。

```cpp
#include <iostream>
#include <string>
#include "boost/tuple/tuple.hpp"

template <typename Tuple,int Index> struct print_helper {
  static void print(const Tuple& t) {
    std::cout << boost::tuples::get<Index>(t) << '\n';
    print_helper<Tuple,Index-1>::print(t);
  }
};

template<typename Tuple> struct print_helper<Tuple,0> {
  static void print(const Tuple& t) {
    std::cout << boost::tuples::get<0>(t) << '\n';
  }
};

template <typename Tuple> void print_all(const Tuple& t) {
  print_helper<
    Tuple,boost::tuples::length<Tuple>::value-1>::print(t);
}

int main() {
  boost::tuple<int,std::string,double> 
    tup(42,"A four and a two",42.424242);

  print_all(tup);
} 
```

在这个例子中有一个辅助类模板，`print_helper`, 它是一个元程序，访问 `tuple` 的所有索引，并对每个索引打印出相应元素。偏特化版本用于结束模板递归。函数 `print_all` 用它的 `tuple` 参数的长度以及这个 `tuple` 来调用 `print_helper` 构造函数。tuple 的长度可以这样来取得：

```cpp
boost::tuples::length<Tuple>::value 
```

这是一个常量整型表达式，这意味着它可以作为第二个模板参数传递给 `print_helper`. 但是，我们的解决方案中有一个警告，我们看看这个程序的输出结果就清楚了。

```cpp
42.4242
A four and a two
42 
```

我们按反序来打印元素了！虽然有些情况下可能会需要这种用法(他狡猾地辩解说)，但在这里不是。问题在于 `print_helper` 先打印 `boost::tuples::length&lt;Tuple&gt;::value-1` 元素的值，然后才到前一个元素，直到偏特化版本打印第一个元素值为止。我们不应该使用第一个元素作为特化条件以及从最后一个元素开始，而是需要从第一个元素开始以及使用最后一个元素作为特化条件。这怎么可能？在你明白到 `tuple` 是以一个特殊的类型 `boost::tuples:: null_type` 作为结束以后，解决的方法就很明显了。我们可以确保一个 `tuple` 中的最后一个类型是 `null_type`, 这也意味着我们的解决方法应该是对 `null_type` 进行特化或函数重载。

剩下的问题就是取出第一个元素的值，然后继续，并在列表尾部结束。`tuple`s 提供了成员函数 `get_head` 和 `get_tail` 来访问其中的元素。顾名思义，`get_head` 返回数值序列的头，即第一个元素的值。`get_tail` 返回一个由该 `tuple` 中除了第一个值以外的其它值组成的 `tuple`. 这就引出了 如下 `print_all` 的解决方案。

```cpp
void print_all(const boost::tuples::null_type&) {}

template <typename Tuple> void print_all(const Tuple& t) {
  std::cout << t.get_head() << '\n';
  print_all(t.get_tail());
} 
```

这个解决方案比原先的更短，并且按正确的顺序打印元素的数值。每一次函数模板 `print_all` 执行时，它打印 `tuple` 的第一个元素，然后用一个由 `t` 中除第一个值以外的其它值所组成的 `tuple` 递归调用它自己。当 `tuple` 没有数值时，结尾是一个 `null_type`, 将调用重载函数 `print_all` ，递归结束。

可以知道某个元素的类型有时是有用的，例如当你要在泛型代码中声明一个由 `tuple` 的元素初始化的变量时。考虑一个返回 `tuple` 中前两个元素的和的函数，它有一个额外的要求，即返回值的类型必须是两个中较大的那个类型(例如，考虑整数类型)。如果不清楚元素的类型，就不可能创建一个通用的解决方案。这正是辅助模板 `element&lt;N,Tuple&gt;::type` 要做的，如下例所示。我们所面对的问题不仅仅是计算哪个元素具有较大的类型，还有如何声明这个函数的返回值类型。这有点复杂，但我们可以通过增加一个间接层来解决。这个间接层以一个额外的辅助模板形式出现，它有一个责任：提供一个 `typedef` 以定义两种类型中的较大者。代码可能看起来有点多，但它的确可以完成任务。

```cpp
#include <iostream>
#include "boost/tuple/tuple.hpp"
#include <cassert>
#include <typeinfo>  //译注：原文没有这行，不能通过编译

template <bool B,typename Tuple> struct largest_type_helper {
  typedef typename boost::tuples::element<1,Tuple>::type type;
};

template<typename Tuple> struct largest_type_helper<true,Tuple> {
  typedef typename boost::tuples::element<0,Tuple>::type type;
};

template<typename Tuple> struct largest_type {
  typedef typename largest_type_helper<
    (sizeof(boost::tuples::element<0,Tuple>)>
    sizeof(boost::tuples::element<1,Tuple>)),Tuple>::type type; 
};

template <typename Tuple> 
typename largest_type<Tuple>::type sum(const Tuple& t) {
  typename largest_type<Tuple>::type
    result=boost::tuples::get<0>(t)+
      boost::tuples::get<1>(t);

  return result;
}

int main() {
  typedef boost::tuple<short,int,long> my_tuple;

  boost::tuples::element<0,my_tuple>::type first=14;
  assert(typeid(first) == typeid(short));  
  //译注：原文为 assert(type_id(first) == typeid(short)); 明显有误
  boost::tuples::element<1,my_tuple>::type second=27;
  assert(typeid(second) == typeid(int));
  //译注：原文为 assert(type_id(second) == typeid(int)); 明显有误
  boost::tuples::element<
    boost::tuples::length<my_tuple>::value-1,my_tuple>::type 
    last;

  my_tuple t(first,second,last);

  std::cout << "Type is int? " << 
    (typeid(int)==typeid(largest_type<my_tuple>::type)) << '\n';

  int s=sum(t);
} 
```

如果你不太清楚模板元编程的运用，不用担心，对于使用 Tuple 库这不是必需的。虽然这类代码有时会用到，它的思想其实也很简单。`largest_type` 从两个辅助类模板 `largest_type_helper` 中的一个获得 `typedef` ，具体使用哪一个就要靠那个布尔参数来特化了。这个参数通过比较 `tuple` (第二个模板参数) 的头两个元素的大小来决定。这样的结果是，`typedef` 会表现为两个类型中较大的那一个。我们的函数 `sum` 使用这个类型来作为返回值的类型，剩下的部分就是简单地对两个元素进行相加了。

这个例子的剩余部分示范了如何使用函数 `sum`, 还有如何从某个 `tuple` 元素的类型来声明变量。头两个对 `tuple` 中的索引使用了硬编码。

```cpp
boost::tuples::element<0,my_tuple>::type first=14;
boost::tuples::element<1,my_tuple>::type second=27; 
```

最后一个声明取出 `tuple` 的最后一个元素的索引，并用它作为辅助模板的输入来(通用地)声明这个类型。

```cpp
boost::tuples::element<
  boost::tuples::length<my_tuple>::value-1,my_tuple>::type last; 
```

### Tuples 与 for_each

我们前面用于创建 `print_all` 函数的方法可以被推广至创建象 `std::for_each` 那 样的更通用的机制。例如，如果我们不想打印元素，而是想对它们取和或是复制它们，又或者我们只想打印它们中的一部分，要怎么做呢？对 tuple 的元素进行顺序访问并不简单，正如我们在前面的例子所见的一样。创建一个通用性的解决方案来接受一个函数或函数对象参数来调用 tuple 的元素是很有意义的。这样就不仅可以实现(有点限制的) `print_all` 函数的功能，还可以执行任何函数，只要这个函数可以接受 `tuple` 中的元素的类型。下面的例子创建了一个名为 `for_each_element` 的函数模板来实现这个功能。这个例子用两个函数对象作为参数来示范如何使用 `for_each_element` 。

```cpp
#include <iostream>
#include <string>
#include <functional>
#include "boost/tuple/tuple.hpp"

template <typename Function> void for_each_element(
  const boost::tuples::null_type&, Function) {}

template <typename Tuple, typename Function> void 
for_each_element(Tuple& t, Function func) {
  func(t.get_head());
  for_each_element(t.get_tail(),func);
}

struct print {
  template <typename T> void operator()(const T& t) {
    std::cout << t << '\n';
  }
};

template <typename T> struct print_type {
  void operator()(const T& t) {
    std::cout << t << '\n';
  }

  template <typename U> void operator()(const U& u) {}
};

int main() {
  typedef boost::tuple<short,int,long> my_tuple;

  boost::tuple<int,short,double> nums(1,2,3.01);

  for_each_element(nums, print());
  for_each_element(nums, print_type<double>());
} 
```

函数 `for_each_element` 重用了前面例子中的策略，通过重载函数的一个版本来接受类型为 `null_type` 的参数，表示已来到 `tuple` 元素的结尾，从而不做任何事。让我们来看看完成实际工作的那个函数。

```cpp
template <typename Tuple, typename Function> void 
for_each_element(Tuple& t, Function func) {
  func(t.get_head());
  for_each_element(t.get_tail(),func);
} 
```

第二个模板的函数参数指定了以 `tuple` 元素为参数进行调用的函数(或函数对象)。`for_each_element` 首先用 `get_head` 返回的元素来调用这个函数(对象)。要注意的是，`get_head` 返回的是 `tuple` 的当前元素。然后，它递归地用 `tuple` 的剩余元素来调用自已本身。第二次调用同样取出头一个元素并用它调用给定的函数(对象)，然后再次递归，一直下去。最后，`get_tail` 发现没有元素了，就返回一个 `null_type` 实例，它匹配了那个非递归的 `for_each_element` 重载版本，从而结束了递归。这就是 `for_each_element` 的全部！

接下来，这个例子给出了两个示例函数对象，它们所用的技术可以在其它情况下重用。一个是 `print` 函数对象。

```cpp
struct print {
  template <typename T> void operator()(const T& t) {
    std::cout << t << '\n';
  }
}; 
```

这个 `print` 函数对象没什么奇怪的地方，但正如它所做的那样，许多程序员不知道调用操作符可以是模板的！通常，函数对象可以对一个或多个类型进行特化，但对于 `tuples` 这样不行，因为它的元素可以是完全不同的类型。因此，不是对于函数对象本身进行特化，而是对调用操作符进行特化，这样做的另一个好处是它更简单，如下。

```cpp
for_each_element(nums, print()); 
```

不需要给出类型，而如果使用特化函数对象则需要。把模板参数推给成员函数有些时候是很有用的，而且通常用户也更容易使用。

第二个函数对象打印指定类型的所有元素。这种过滤方法也可以用于取出相容类型的元素。

```cpp
template <typename T> struct print_type {
  void operator()(const T& t) {
    std::cout << t << '\n';
  }

  template <typename U> void operator()(const U& u) {}
}; 
```

这个函数对象显示了另一个有用的技术，我称之为忽略重载(discarding overload)。它用于忽略传给它的除了 `T` 类型以外的所有元素。窍门就是除了指定类型外的其它类型的重载匹配。这可能会让你想到这种技术与另一个有密切的联系，即 `sizeof` 窍门和省略号(`...`)结构，后者被也用于在编译期进行决议，但它在这里不能使用，在这里，函数确实被调用了，只是没有做任何事情而已。这个函数对象可以这样用：

```cpp
for_each_element(print_type<double>(),nums); 
```

很容易用，也容易写，并且它有更多的价值。实际上 Tuple 库使用的函数对象可能没有这么多特性，它们也可以用于这种技术或其它的用法。

# Tuple 总结

## Tuple 总结

Tuple 库为 C++带来了 tuples 的概念。它是符合直觉和简单明了的，虽然它的主要用途看起来就是用于从函数返回多个返回值，但它也可以用于创建各种逻辑组合，就象在标准库容器中保存一组元素一样。这种方法和为各个不同的返回类型创建一个 `struct` 是相同的，但后者不仅沉闷，而且不可能作出递归的泛型解决方案。使用 Boost.Tuple 就可以解决这些问题。

在本章中，我们看到了如何使用 Tuple 库，以及如何用函数对象扩充它，还有如何对 tuple 使用算法。通过索引来访问元素，以及 `get_head`/`get_tail` 成员函数，提供了使用 `tuple`s 的一致性，使得许多解决方案可以实现，而使用用户定义类型(UDTs)时是不可能的。

Boost.Tuple 的创建者，Jaakko Järvi，应该为这个杰出的库获得荣誉。他的努力有力地证明了，几乎所有 C++中缺少的东西都可以由天才的设计者通过库增加进来。