# Library 12\. Signals

*   Signals 库如何改进你的程序？
*   Signals 如何适用于标准库？
*   Signals
*   用法
*   Signals 总结

# Signals 库如何改进你的程序？

## Signals 库如何改进你的程序？

*   函数和函数对象的灵活多点回调

*   健壮的触发器及事件处理的机制

*   兼容于函数对象工厂，如 Boost.Bind 和 Boost.Lambda

Boost.Signals 库具体化了信号(signals)和 插槽(slots)，信号指的是某种可被"抛出"的 东西，而插槽是接收该信号的连接者。这是一种著名的设计模式，它还有另外一些名字 *Observer, signals/slots, publisher/subscriber, events* (和 event targets)，这些名字指的都是同一个东西，指的是一些信 息源和某些对这些信息的变化感兴趣的实例之间的一对多关系。这种设计模式的使用有多种情况；最常见的是在 GUI 代 码中，用于使特定动作(例如，用户单击了一个按钮)与其它动作(按 钮改变它的外观，执行某个商业逻辑)松散连接。信号与插槽 在许多场合都很有用，解耦动作的触发条件(信号)和处理它的代码(一 个或多个插槽)。它可用于动态改变处理代码的行为，允许同 一信号对应多个处理，或者通过一个信号及插槽的类型间的抽象关联来降低类型依赖性。通过使用 Boost.Signals, 可 以创建一些信号来接受任意给定的函数特征的插槽，即插槽接受任意类型的参数。这种方法使得该库非常灵活；它适用于任意范围的信号需求。通过对信号源和处理 者的解耦，系统无论在物理和逻辑依赖上都变得更为健壮。它可以让信号类型对插槽类型完全一无所知，反之亦然。这对于更高层次的可复用性是很有必要的，它有 助于打破依赖性的循环。因此，一个信号与插槽的库不仅仅关系到面向对象的回调，它也关系到使用它的整个系统的健壮性。

# Signals 如何适用于标准库？

## Signals 如何适用于标准库？

C++标 准库中没有用于回调的工具，而这种工具显然是需要的。Boost.Signals 使 用了与标准库相同的态度进行设计，它是标准库工具箱的一个杰出的扩展。

# Signals

## Signals

### 头文件: `"boost/signals.hpp"`

通过单个头文件包含了整个库。

```cpp
"boost/signals/signal.hpp" 
```

包含了 `signal` 的定义。

```cpp
"boost/signals/slot.hpp" 
```

包含了 `slot` 类的定义。

```cpp
"boost/signals/connection.hpp" 
```

包含了类 `connection` 和 `scoped_connection` 的定义。

要使用这个库，可以包含头文件 `"boost/signals.hpp"`，这样可以确保整个库可用，或者可以按照你的需要包含单独的头文件。Boost.Signals 库的核心部分定义在名字空间 `boost` 中，高级特性则定义在 `boost::signals` 中。

以下是 `signal` 的部分内容，其后将对其中最主要的成员函数进行了简要的介绍。如果你需要完整的参考，请见 Signals 的在线文档。

```cpp
namespace boost {

  template<typename Signature,
  // Function type R(T1, T2, ..., TN)
    typename Combiner = last_value<R>,
    typename Group = int,
    typename GroupCompare = std::less<Group>,
    typename SlotFunction = function<Signature> >
  class signal : public signals::trackable,
                 private noncopyable {
  public:
    signal(const Combiner&=Combiner(),
           const GroupCompare&=GroupCompare());

    ~signal();

    signals::connection connect(const slot_type&);
    signals::connection connect(
      const Group&,
      const slot_type&);

    void disconnect(const Group&);

    std::size_t num_slots() const;

    result_type operator()
      (T1, T2, ..., TN);
  };
} 
```

### 类型

我们先来看看 `signal` 的模板参数。除了第一个参数，其它参数都有相应的缺省值，这有助于理解这些参数的基本意思。第一个模板参数是被调用的函数的签名。在这种 `signal`s 的情况下，`signal` 本身就是被调用的实体。声明这个签名时，使用与普通函数签名相同的语法[1]。例如，一个返回 `double` 且接受一个类型 `int` 的参数的函数签名应该象这样：

> [1] 细心的读者可以已经注意到 `boost::function` 也是这样用的。

```cpp
signal<double(int)> 
```

`Combiner` 参数表示一个函数对象，它负责逐个对该 `signal` 所有已连接的插槽(slot)进行调用。它同时也决定如何将组合这些调用返回的结果。缺省的类型是 `last_value`, 它只是简单地返回最后一个插槽的调用结果。

`Groups` 参数是用于组合所有连接到 `signal` 的插槽的一种类型。通过连接到不同的插槽组，你可以预设调用插槽的顺序，同时也可以断开插槽组。

`GroupCompare` 参数决定了如何排序 `Groups`，缺省值为 `std::less&lt;Group&gt;`, 通常它都是正确的。如果 `Groups` 使用了定制的类型，就有可能需要其它的排序方法。

最后，`SlotFunction` 参数表示插槽函数的类型，缺省值为 `boost::function`. 我想不出有什么理由改变这个缺省值。这个模板参数用于定义插槽的类型，定义的方法是一个公有的 `typedef slot&lt;SlotFunction&gt; slot_type`.

### 成员函数

```cpp
signal(const Combiner&=Combiner(),
  const GroupCompare&=GroupCompare()); 
```

在构造一个 `signal` 时，可以传入一个 `Combiner`，它是一个负责在信号到达时调用相应插槽并对返回值进行处理的对象。

```cpp
~signal(); 
```

析构函数在析构时断开所有已连接的插槽。

```cpp
signals::connection connect(const slot_type& s); 
```

`connect` 函数把插槽 `s` 连接到 `signal`. 函数指针、函数对象、`bind` 表达式或者 lambda 表达式都可以用作插槽。`connect` 返回一个 `signals::connection`, 它是代表被创建的连接的句柄。通过使用这个句柄，插槽可以从 `signal` 断开，或者你也可以测试该插槽是否还有连接。

```cpp
signals::connection connect(const Group& g, const slot_type& s); 
```

这个 `connect` 的重载版本与前一个作用相似，但是它还把插槽 `s` 连接到组 `g`. 把一个插槽连接到一个组意味着当一个 `signal` 产生时，属于较前面的组的插槽会先被调用(即按组的顺序来调用，`signal` 模板的 `GroupCompare` 参数定义了组的顺序)，而且属于组的所有插槽会在不属于组的插槽之前被调用(可能只有部分插槽是在组中的)。

```cpp
void disconnect(const Group& g); 
```

断开所有属于组 `g` 的已连接插槽。

```cpp
std::size_t num_slots() const; 
```

返回当前连接到 `signal` 的插槽数量。要测试插槽是否为空，应该调用函数 `empty`，而不要调用 `num_slots` 并测试其返回是否为 0，因为 `empty` 的效率更高。

```cpp
result_type operator()(T1, T2, ..., TN); 
```

`signal`s 使用调用操作符来调用。当信号产生时，必须传递适当的参数给调用操作符，必须符合 `signal` 的签名(即声明 `signal` 类型时的第一个模板参数)。参数的类型必须可以隐式转换为信号所需的类型，只有这样调用才可以成功。

Boost.Signals 中还有其它的类型，我们将在本章剩余部分讨论它们。我们还将讨论 `signal` 类中有用的 `typedef`s。

# 用法

## 用法

当你面对需要用多段代码来处理一个事件的情况时，典型的解决方案有：用函数指针进行回调，或者直接对 产生事件的子系统与处理事件的子系统之间的依赖性进行编码。这种设计常常会导致循环的依赖性。通过使用 Boost.Signals, 你将获得灵活性和解耦。要开始使用这个库，首先要包含头文件 `"boost/signals.hpp"`.[2]

<small class="calibre23"></small><small class="calibre23"> [2]</small> <small class="calibre23"></small><small class="calibre23"></small><small class="calibre23">Boost.Signals</small> 库和 <small class="calibre40"></small><small class="calibre40">Boost.Regex</small> 库是本书所讨论的库中仅有的需要编译和链接才能使用的库。编译的过程很简单，在线文档中已有详尽的描述，这里我不再复述。

以下例子示范了 `signal`s 和插槽(slots)的基本特性，包括如何连接它们以及如何产生一个 `signal`. 注意，插槽指的是由你提供的一个兼容于 `signal` 的函数签名的函数或函数对象。在以下代码中，我们既创建了一个普通函数，`my_first_slot`, 也创建了一个函数对象，`my_second_slot`; 它们两个都将连接到我们创建的一个 `signal` 上。

```cpp
#include <iostream>
#include "boost/signals.hpp"

void my_first_slot() {
  std::cout << "void my_first_slot()\n";
}

class my_second_slot {
public:
  void operator()() const {
    std::cout <<
      "void my_second_slot::operator()() const\n";
  }
};

int main() {
  boost::signal<void ()> sig;

  sig.connect(&my_first_slot);
  sig.connect(my_second_slot());

  std::cout << "Emitting a signal...\n";
  sig();
} 
```

我们首先声明一个 `signal`, 它所需的插槽为返回 `void` 且不带参数。然后，我们把两个兼容的插槽类型连接到该 `signal`. 对于第一个插槽，我们用普通函数 `my_first_slot` 的地址调用 `connect`。对于另一个插槽，我们缺省构造一个函数对象 `my_second_slot` 的实例并把它传给 `connect`。这些连接意味着当我们产生一个 `signal` (通过调用 `sig`)时，这两个插槽将被立即调用。

```cpp
sig(); 
```

运行这个程序，输出信息如下：

```cpp
Emitting a signal...
void my_first_slot()
void my_second_slot::operator()() const 
```

但是，后两行的顺序不一定是这样的，因为属于同一个组的插槽会以不确定的顺序执行。没有办法确定哪一个插槽会先被调用。如果插槽的调用顺序事关紧要，你就必须把它们放入不同的组。

### 插槽分组

有时候，某些插槽需要在其它插槽之前调用，例如某些插槽会产生一些副作用而别的插槽需要依赖于这些副作用。分组就是支持这种需求的方法。`signal` 有一个模板参数，名为 `Group`，其缺省值为 `int`. Groups 缺省以 `std::less&lt;Group&gt;` 为排序标准，对于 `int` 就是 `operator&lt;` 。换句话说，属于 group 0 的插槽会在 group 1 的插槽之前调用，等等。但是请注意，同一个组中的插槽的调用顺序是不确定的。要严格控制所有插槽的调用顺序，唯一的办法就是把每个插槽都安排到各自的组中。

把插槽指定到一个组的方法是，传递一个 `Group` 给 `signal::connect`. 一个已连接插槽不能改变其所属的组；要改变一个插槽所属的组，必须先断开它的连接，然后重新把它连接到 `signal` 上并同时指定新组。

作为例子，我们考虑两个插槽，它们带一个类型为 `int&` 的参数；第一个插槽将参数加倍，第二个插槽则把当前值加 3。我们要求正确的语义是，先把该值加倍，然后再加 3。如果不指定顺序，我们就不能确保按该语义执行。以下方法只能在某些系统的某些时候正确执行(可能是周一或周三而且月圆的时候)。

```cpp
#include <iostream>
#include "boost/signals.hpp"

class double_slot {
public:
  void operator()(int& i) const {
    i*=2;
  }
};

class plus_slot {
public:
  void operator()(int& i) const {
    i+=3;
  }
};

int main() {
  boost::signal<void (int&)> sig;
  sig.connect(double_slot());
  sig.connect(plus_slot());

  int result=12;
  sig(result);
  std::cout << "The result is: " << result << '\n';
} 
```

运行这段程序，可能产生以下输出：

```cpp
The result is: 30 
```

或者产生以下输出：

```cpp
The result is: 27 
```

不使用分组的方法就无法保证正确的行为。我们需要确保 `double_slot` 总是在 `plus_slot` 之前被调用。这就要求我们要指定 `double_slot` 属于一个顺序在 `plus_slot` 所属的组之前的组，即：

```cpp
sig.connect(0,double_slot());
sig.connect(1,plus_slot()); 
```

这样可以确保得到我们想要的(即 27)。再次提醒，对于同一个组中的插槽，它们被调用的顺序是不确定的。只要你需要插槽以特定的顺序来执行，就必须确保它们使用不同的组。

`Groups` 的类型是类 `signal` 的一个模板参数，所以它可以使用别的类型，如 `std::string` 。

```cpp
#include <iostream>
#include <string>
#include "boost/signals.hpp"

class some_slot {
  std::string s_;
public:
  some_slot(const std::string& s) : s_(s) {}
  void operator()() const {
    std::cout << s_ << '\n';
  }
};

int main() {
  boost::signal<void (),
    boost::last_value<void>,std::string> sig;

  some_slot s1("I must be called first, you see!");
  some_slot s2("I don't care when you call me, not at all. \
It'll be after those belonging to groups, anyway.");
  some_slot s3("I'd like to be called second, please.");

  sig.connect(s2);
  sig.connect("Last group",s3);
  sig.connect("First group",s1);

  sig();
} 
```

首先我们定义一个插槽类型，它在执行时输出一个 `std::string` 到 `std::cout` 。然后，我们声明 `signal`. 因为 `Groups` 参数是在 `Combiner` 类型之后的，所以我们必须同时指定 `Combiner` (我们只是按缺省值来声明)。我们把 `Groups` 类型设为 `std::string` 。

```cpp
boost::signal<void (),boost::last_value<void>,std::string> sig; 
```

对于剩下的模板参数，我们接受缺省值就可以了。在连接到插槽 `s1`, `s2`, 和 `s3` 时，所创建的组是以字母顺序排序的(因为这是 `std::less&lt;std::string&gt;` 的行为)，因此 `"First group"` 先于 `"Last group"`. 注意，由于字符串常量可以隐式转换为 `std::string`, 所以我们可以把它们直接传递给 `signal` 的 `connect` 函数。运行该程序可以告诉我们正确的结果。

```cpp
I must be called first, you see!
I'd like to be called second, please.
I don't care when you call me, not at all.
It'll be after those belonging to groups, anyway. 
```

我们也可以在声明 `signal` 类型时选择别的排序方法，例如 `std::greater`.

```cpp
boost::signal<void (),boost::last_value<void>,
 std::string,std::greater<std::string> > sig; 
```

如果我们把它用于前面的例子，输出将变为：

```cpp
I'd like to be called second, please.
I must be called first, you see!
I don't care when you call me, not at all.
It'll be after those belonging to groups, anyway. 
```

当然，在这个例子中，`std::greater` 产生的顺序导致了错误的输出，但这是另一回事。分组非常有用，绝对必要，但是给组赋以正确的值并不总是那么简单的事，因为被连接的插槽并不需要在代码的同 一个地方执行。弄清楚某个插槽所应该使用什么组号可能是个问题。有时，这个问题可以用规定来解决，即在代码中增加注释，确保每个人都能看到这些注释，但是 这也只能在代码中不是很多地方要进行组号的赋值以及程序员不偷懒时有用。换句话说，这种方法也不一定管用。所以，你需要一个集中的产生组号的地方，它可以 依据某个给定的值为每个插槽产生唯一的组号，或者如果相关的插槽相互了解，那么也可以由插槽提供它们自己的组号。

现在你已经知道如何解决按顺序调用插槽的问题了，让我们来看看如何让你的 `signal`s 使用不同的签名。你常常需要传递额外的信息给你系统中的重要事件。

### 带参数的 Signals

通常会有一些额外的数据要传递给 `signal`. 例如，想象一个温度保护器，它报告温度的急剧变化。仅仅知道保护器发现了问题是不够的；插槽可能需要知道当前的温度。虽然保护器(一个 `signal`)和插槽都可以从一个公用的传感器去获取温度值，但是最简单的方式还是让保护器在调用插槽时把当前温度传递给插槽。还有一个例子，想象有多个插槽连接到多个 `signal` 上：插槽很可能需要知道是哪一个 `signal` 调用了它。有很多用例都需要从 `signal` 传递一些信息给插槽。插槽接受的参数是 `signal` 声明中的一部分。`signal` 类模板的第一个参数就是调用 `signal` 的函数签名，而且这个签名也用于 `signal` 调用那些被连接的插槽。如果我们想这个参数可以修改，我们就要确保它是通过非`const` 引用或指针来进行传递的，否则我们就可以通过值或 `const` 引用来传递它。注意，这个原始参数除了是可修改或不可修改这么明显的差异之外，对于 `signal` 本身以及插槽可以接受的参数类型还有一些隐喻，如果 `signal` 接受一个传值或传 `const` 引用的参数，那么所有可以隐式转换为该参数类型的类型都可以用于产生一个 `signal`. 对于插槽也一样，如果插槽是通过传值或传 `const` 引用来接受参数的话，这就意味着允许从 `signal` 的真正参数类型隐式转换到这个类型。我们后面将讨论如果在处理信号时正确地传递参数，届时我们将看到更多关于这一点的详细讨论。

想象一个自动停车场监视器，一旦有车进入或离开停车场，监视器将收到一个通知。它需要知道一些关于这 辆车的唯一信息，例如车的登记号码，这样它才可以跟踪每辆车的进入和离开。这个监视器有一个它自己的 `signal`，能够在有人试图进行欺骗时触发警报。这样就需要一些警卫监听这个 `signal`, 我们用一个名为 `security_guard` 来对它们进行建模。最后，我们再增加一个 `gate` 类，它包含一个 `signal` 用于在一辆车进入或离开停车场时产生。( `parking_lot_guard` 显然需要知道这一点)。我们先来看看这个 `parking_lot_guard` 的声明。

```cpp
class parking_lot_guard {
  typedef
    boost::signal<void (const std::string&)> alarm_type;
  typedef alarm_type::slot_type slot_type;

  boost::shared_ptr<alarm_type> alarm_;

  typedef std::vector<std::string> cars;
  typedef cars::iterator iterator;

  boost::shared_ptr<cars> cars_;
public:

  parking_lot_guard();
  boost::signals::connection
    connect_to_alarm(const slot_type& a);
  void operator()(bool is_entering,const std::string& car_id);

private:
  void enter(const std::string& car_id);
  void leave(const std::string& car_id);
}; 
```

这里有三个特别重要的地方要认真看一下；第一个是警报，即一个返回 `void` 且接受一个 `std::string` (它用于标识一辆车)的 `boost::signal` 。这个 `signal` 的声明值得再好好看一次。

```cpp
boost::signal<void (const std::string&)> 
```

它就象是一个函数的声明，只是没有了函数名。如果有怀疑，请记住除此以外没有别的东西了！你可以从外部使用成员函数 `connect_to_alarm` 连接这个 `signal` 。(我们将看到在实现这个类时，如何以及为何我们要发出警报)。下一个要留意的地方是，这个警报以及容纳车辆标识的容器(一个容纳 `std::string`s 的 `std::vector` )两者均保存于 `boost::shared_ptr` 中。这样做的原因是，尽管我们只是打算声明一个 `parking_lot_guard` 实例，但是也可能变成多份拷贝；因为这个监视器类稍后还会连接到其它的 `signal` 上，这样就会创建多份拷贝(Boost.Signals 会复制插槽，所以需要正确地管理生存期)；而我们希望所有的数据都可用，因此我们就要共享它。虽然我们可以避免拷贝，例如通过使用指针或者把插槽的行为外 部化，但是这样做可以发现一些容易掉进去的陷阱。最后还要留意的是，我们声明了一个调用操作符，其原因是我们将要在 `gate` 类(待会定义)中把 `parking_lot_guard` 连接到一个 `signal` in the class .

现在让我们把注意力放到 `security_guard` 类。

```cpp
class security_guard {
  std::string name_;
public:
  security_guard (const char* name);

  void do_whatever_it_takes_to_stop_that_car() const;
  void nah_dont_bother() const;

  void operator()(const std::string& car_id) const;
}; 
```

`security_guard`s 并不需要做太多事情。这个类有一个调用操作符，用作来自于 `parking_lot_guard` 的警报的一个插槽，另外还有两个函数：一个用于停住引发警报的车辆，另一个不做任何事。下面带来我们的 `gate` 类，它用于在有车辆到达停车场以及车辆离开时进行检查。

```cpp
class gate {
  typedef
    boost::signal<void (bool,const std::string&)> signal_type;
  typedef signal_type::slot_type slot_type;

  signal_type enter_or_leave_;
public:
  boost::signals::connection
    connect_to_gate(const slot_type& s);
  void enter(const std::string& car_id);
  void leave(const std::string& car_id);
}; 
```

你将留意到，`gate` 类包含一个 `signal`，它在有车辆进入或离开停车场时被触发。有一个公用成员函数(`connect_to_gate`)用于连接这个 `signal`, 另两个成员函数(`enter` 和 `leave`)用于在车辆进入或离开时被调用。

现在是时候来实现它们了。让我们从 `gate` 类开始。

```cpp
class gate {
  typedef
    boost::signal<void (bool,const std::string&)> signal_type;
  typedef signal_type::slot_type slot_type;

  signal_type enter_or_leave_;
public:
  boost::signals::connection
    connect_to_gate(const slot_type& s) {
    return enter_or_leave_.connect(s);
  }

  void enter(const std::string& car_id) {
    enter_or_leave_(true,car_id);
  }

  void leave(const std::string& car_id) {
    enter_or_leave_(false,car_id);
  }
}; 
```

这个实现很简单。多数工作都前转到其它对象。函数 `connect_to_gate` 简单地把调用转为对 `signal enter_or_leave_` 的 `connect` 的调用。函数 `enter` 产生 `signal`, 传入一个 `true` (代表有车辆进入)和车辆的标识。`leave` 完成同样的工作，但是传入的是 `false`, 代表有车辆离开。简单的类做简单的事。`security_guard` 类也不太复杂。

```cpp
class security_guard {
  std::string name_;
public:
  security_guard (const char* name) : name_(name) {}

  void do_whatever_it_takes_to_stop_that_car() const {
    std::cout <<
      "Stop in the name of...eh..." << name_ << '\n';
  }

  void nah_dont_bother() const {
    std::cout << name_ <<
      " says: Man, that coffee tastes f i n e fine!\n";
  }

  void operator()(const std::string& car_id) const {
    if (car_id.size() && car_id[0]=='N')
      do_whatever_it_takes_to_stop_that_car();
    else
      nah_dont_bother();
  }
}; 
```

`security_guard`s 知道它们自己的名字，并且可以决定在警报发出时是否要做些事情(如果 `car_id` 以字母 N 打头，它们就会有所动作)。调用操作符就是被调用的插槽函数，`security_guard` 对象是一个函数对象，并且符合 `parking_lot_guard` 的 `alarm_type` 信号的要求。`parking_lot_guard` 稍微复杂一些，但也不是很复杂。

```cpp
class parking_lot_guard {
  typedef
    boost::signal<void (const std::string&)> alarm_type;
  typedef alarm_type::slot_type slot_type;

  boost::shared_ptr<alarm_type> alarm_;

  typedef std::vector<std::string> cars;
  typedef cars::iterator iterator;

  boost::shared_ptr<cars> cars_;
public:

  parking_lot_guard()
    : alarm_(new alarm_type), cars_(new cars) {}

  boost::signals::connection
    connect_to_alarm(const slot_type& a) {
    return alarm_->connect(a);
  }

  void operator()
    (bool is_entering,const std::string& car_id) {
    if (is_entering)
      enter(car_id);
    else
      leave(car_id);
  }

private:
  void enter(const std::string& car_id) {
    std::cout <<
      "parking_lot_guard::enter(" << car_id << ")\n";

    // 如果车辆已经在这，就触发警报
    if (std::binary_search(cars_->begin(),cars_->end(),car_id))
      (*alarm_)(car_id);
    else // Insert the car_id
      cars_->insert(
        std::lower_bound(
          cars_->begin(),
          cars_->end(),car_id),car_id);
  }

  void leave(const std::string& car_id) {
    std::cout <<
      "parking_lot_guard::leave(" << car_id << ")\n";

    // 如果是未登记的车辆，就触发警报
    std::pair<iterator,iterator> p=
      std::equal_range(cars_->begin(),cars_->end(),car_id);
    if (p.first==cars_->end() || *(p.first)!=car_id)
      (*alarm_)(car_id);
    else
      cars_->erase(p.first);
  }
}; 
```

就是这样了！(当然，我们还没有把插槽连接到 `signal` 上，还要做一些事情。但是这些类对于所要做的事情而言还是非常地简单的)。 为了让警报和车辆标识的 `shared_ptr` 有正确的行为，我们实现了缺省构造函数，在其中适当地分配了 `signal` 和 `vector` 。隐式创建的复制构造函数、析构函数以及赋值操作符都可以正确工作(这要归功于智能指针)。函数 `connect_to_alarm` 把调用转到所含的 `signal` 的 `connect`. 调用操作符则检查其布尔参数的值来看是否有车辆进入或离开，并且调用相应的函数 `enter` 或 `leave`. 在函数 `enter` 中，首先做的是在车辆标识的 `vector` 中进行查找。如果找到该标识则说明有问题；可能有人偷了车号牌。查找采用的是算法 `binary_search`,[3] 它要求容器是有序的(我们必须要确保它总是有序的)。如果我们发现标识已存在，就立即触发警报，即调用 `signal` 。

<small class="calibre23"></small><small class="calibre23"> [3] </small><small class="calibre40"></small><small class="calibre40">`binary_search`</small> 的复杂度为 <small class="calibre23"></small><small class="calibre23"></small><small class="calibre23">`O(logN)`.</small>

```cpp
(*alarm_)(car_id); 
```

首先我们需要解引用 `alarm_` ，因为 `alarm_` 是一个 `boost::shared_ptr`, 而在调用它时，我们传给它一个表示车辆标识的参数。如果我们没有找到该标识，则一切正常， 我们就把这个车辆标识插入到 `cars_` 的正确位置中。记住我们必须保证容器随时有序，最好的办法就是把元素插入到一个不会影响顺序的位置上。算法 `lower_bound` 可以给我们指出这个位置(该算法同样要求有序序列)。最后一个是函数 `leave`, 它在有车辆离开停车场时被调用。`leave` 先确认车辆的标识是否已登记在我们的容器中。这是通过调用算法 `equal_range` 来实现的，该算法返回一对迭代器，表示了一个元素可以插入且不影响有序性的范围。这意味着我们必须解引用这个返回的迭代器并确认它的值是否等于我们要查找的那个。如果我们没有找到，我们就要再一次触发警报，而如果我们找到了，就只需要简单地把它从 `vector` 中删掉。你也许留意到我们没有给出停车者交费的代码；这种有害的代码超出了本书的范围。

我们的停车场所需的各个参与者都已经定义好了，我们必须连接这些 `signal`s 和这些插槽，否则不会发生任何事情！`gate` 类不知道任何关于 `parking_lot_guard` 类的东西，同样后者也不知道任何关于 `security_guard` 类的东西。这就是本库的一个特性：产生事件的类型不需要对接收事件的类型有任何了解。回到这个例子上，我们来看看是否可以让这个停车场运作起来。

```cpp
int main() {
  // 创建一些警卫
  std::vector<security_guard> security_guards;
  security_guards.push_back("Bill");
  security_guards.push_back("Bob");
  security_guards.push_back("Bull");
  // 创建两个门
  gate gate1;
  gate gate2;

  // 创建自动监视器
  parking_lot_guard plg;

  // 把自动监视器连接到门上
  gate1.connect_to_gate(plg);
  gate2.connect_to_gate(plg);

  // 把警卫连接到自动监视器上
  for (unsigned int i=0;i<security_guards.size();++i) {
    plg.connect_to_alarm(security_guards[i]);
  }

  std::cout << "A couple of cars enter...\n";
  gate1.enter("SLN 123");
  gate2.enter("RFD 444");
  gate2.enter("IUY 897");

  std::cout << "\nA couple of cars leave...\n";
  gate1.leave("IUY 897");
  gate1.leave("SLN 123");

  std::cout << "\nSomeone is entering twice - \
or is it a stolen license plate?\n";
  gate1.enter("RFD 444");
} 
```

这就是你要的，一个具有完整功能的停车场。我们创建了三个 `security_guard`s, 两个 `gate`s, 和一个 `parking_lot_guard`. 它们相互之间一无所知，但我们还是要通过正确的架构把它们联系起来，停车场中发生的重要事件才得以相互传递。这意味着要把 `parking_lot_guard` 连接到两个 `gate`s 上。

```cpp
gate1.connect_to_gate(plg);
gate2.connect_to_gate(plg); 
```

这样就确保了无论何时 `gate` 实例中产生了 `signal enter_or_leave_` 信号，`parking_lot_guard` 都可以收到这个事件通知。接着，我们再将 `security_guard`s 连接到 `parking_lot_guard` 中的警报 `signal` 上。

```cpp
plg.connect_to_alarm(security_guards[i]); 
```

我们已经设法将这些类型相互之间进行了解耦，它们还是得到了执行它们的职责所需的适量的信息。在前面的代码中，我们让少量的车辆进入和离开，来测试这个停车场。这个真实世界的模拟显示了我们已经让各个模块按要求相互通信了。

```cpp
A couple of cars enter...
parking_lot_guard::enter(SLN 123)
parking_lot_guard::enter(RFD 444)
parking_lot_guard::enter(IUY 897)

A couple of cars leave...
parking_lot_guard::leave(IUY 897)
parking_lot_guard::leave(SLN 123)

Someone is entering twice - or is it a stolen license plate?
parking_lot_guard::enter(RFD 444)
Bill says: Man, that coffee tastes f.i.n.e fine!
Bob says: Man, that coffee tastes f.i.n.e fine!
Bull says: Man, that coffee tastes f.i.n.e fine! 
```

可惜的是，拿着车牌 RFD 444 的骗子跑掉了，但是你能做的就是这些。

关于 `signal`s 的参数已经讨论了很长一段篇幅，事实上我们更多是在讨论 Signals 的基本用法，即对产生 `signal`s 的类型和监听它的插槽进行解耦。记住，任何类型的参数都可以传递，而 `signal` 类型的声明决定了插槽函数的签名，该声明看起来就象一个不带函数名的函数声明。我们根本没有提到返回类型，虽然它也是签名的一部分。这个疏忽的原因是返回类型可以有多种不同的处理方法，接下来我们将看到为什么会这样以及如何去做。

### 对结果进行组合

如果一个 `signal` 的签名以及它的插槽具有非`void` 的返回类型，显然对于插槽的返回值会有事发生，事实上，那个对 `signal` 的调用将产生某种结果。但是结果是什么呢？`signal` 类模板有一个参数名为 Combiner, 它就是负责组合并返回结果的一个类型。缺省的 Combiner 是 `boost::last_value`, 它是一个类，只负责简单地返回所调用的最后一个插槽的返回值。那么，究竟是哪一个插槽呢？我们真的不知道，因为调用同一个组内的插槽的顺序是不确定的[4]。我们从一个小例子来示范一下缺省的 Combiner 。

> [4] 所以，假设最后一个组中只有一个插槽，我们就可以知道。

```cpp
#include <iostream>
#include "boost/signals.hpp"

bool always_return_true() {
  return true;
}

bool always_return_false() {
  return false;
}

int main() {
  boost::signal<bool ()> sig;

  sig.connect(&always_return_true);
  sig.connect(&always_return_false);

  std::cout << std::boolalpha << "True or false? " << sig();
} 
```

有两个插槽，`always_return_true` 和 `always_return_false`, 被连接到 `signal sig`, 每个都返回一个 `bool` 且不带参数。调用 `sig` 的结果被输出到 `cout`. 它会是 `true` 还是 `false`? 不经测试的话，我们无法知道(我试了一上，结果是 `false`)。在实践中，你要么不关心调用 `signal` 所返回的值，要么你就要创建你自己的 Combiner 来提供有意义的、客户化的行为。例如，可能是对所有插槽返回的结果进行处理后得到调用 `signal` 的最终结果。另一种情况，也可能是在某一个插槽返回 `false` 后就不再调用其它的插槽。一个定制的 Combiner 可以做到这些，甚至更多。这是因为 Combiner 可以对插槽进行逐个调用，并根据返回值来决定做什么。

想象一个初始化序列，其中任何失败都将中止整个序列。插槽可以根据它们被调用的次序来指定到组中。没有一个定制的 Combiner 的话，它看起来就象这样：

```cpp
#include <iostream>
#include "boost/signals.hpp"

bool step0() {
  std::cout << "step0 is ok\n";
  return true;
}

bool step1() {
  std::cout << "step1 is not ok. This won't do at all!\n";
  return false;
}

bool step2() {
  std::cout << "step2 is ok\n";
  return true;
}

int main() {
  boost::signal<bool ()> sig;
  sig.connect(0,&step0);
  sig.connect(1,&step1);
  sig.connect(2,&step2);

  bool ok=sig();

  if (ok)
    std::cout << "All system tests clear\n";
  else
    std::cout << "At least one test failed. Aborting.\n";
} 
```

以上这段代码没有办法让代码知道其中有一个测试是失败的。你也记得，缺省的 combiner 是 `boost::last_value`, 它只是简单地返回最后一个插槽的返回值，即调用 `step2` 的返回值。运行这个例子会给出一个令人失望的输出：

```cpp
step0 is ok
step1 is not ok. This won't do at all!
step2 is ok
All system tests clear 
```

显然这不是正确的结果。我们需要一个 Combiner ，它应该在某个插槽返回 `false` 时中止处理，并把结果传回给 `signal`. 一个 Combiner 就是一个具有某些额外要求的函数对象。它必须有一个名为 `result_type` 的 `typedef`，用于指定其调用操作符的返回类型。此外，调用操作符必须以它被调用的迭代器类型泛化。我们这里需要的 Combiner 非常简单，因此它恰好是一个好的例子。

```cpp
class stop_on_failure {
public:
  typedef bool result_type;

  template <typename InputIterator>
  bool operator()(InputIterator begin,InputIterator end) const
  {
    while (begin!=end) {
      if (!*begin)
        return false;
      ++begin;
    }
    return true;
  }
}; 
```

注意，公有的 `typedef result_type`, 它定义为 `bool`. `result_type` 的类型无需与插槽的返回类型相关。(在声明 `signal` 时，你指定了插槽的签名以及 `signal` 的调用操作符的参数。但是，Combiner 的返回类型决定了 `signal` 的调用操作符的返回类型。缺省情况下，它与插槽的返回类型相同，但这不是必须的)。`stop_on_failure` 的调用操作符以一个插槽迭代器类型所泛化，它对插槽进行逐个迭代并调用；直到我们遇到一个错误为止。对于 `stop_on_failure`, 我们不想在遇到错误的返回值后再继续调用插槽，因此我们对于每次调用都检查其返回值。如果返回值为 `false`, 该函数说立即返回，否则它继续调用下一个插槽。要使用这个 `stop_on_failure`, 我们只需在声明 `signal` 类型时指出即可：

```cpp
boost::signal<bool (),stop_on_failure> sig; 
```

如果我们在前面的例子中使用它，则输出的结果就会符合我们的要求了。

```cpp
step0 is ok
step1 is not ok. This won't do at all!
At least one test failed. Aborting. 
```

Combiner 的另一个常用类型是，返回所有被调用插槽的返回值中的最大或最小值。还有其它很多有趣的 Combiners，包括：将所有结果保存在一个容器中。本库的(优秀的)在线文档就有这么一个 Combiner 的例子，你应该去读一下！你并不是每天都需要编写自己的 Combiner 类，但偶尔在为一个复杂的问题给出一个漂亮的解决方案时可能会用到。

### Signals 决不能复制

我已经提到过，`signal`s 不能被复制，但是值得留意的是，应该怎样实现一个包含 `signal` 的类。这些类也都必须是不可复制的吗？不，它们不必，但必须手工实现其复制构造函数和赋值操作符。因为 `signal` 类将其复制构造函数和赋值操作符声明为私有的，所以一个聚合了 `signal`s 的类必须实现其所需的语义。正确处理复制的一个方法是，在类的多个实例间共享 `signal`s，我们在停车场的例子中就是这么做的。在那个例子中，每一个 `parking_lot_guard` 实例通过 `boost::shared_ptr` 引向同一个 `signal`。对于其它类，可以在拷贝中缺省构造 `signal`，因为该复制语义不包含对插槽的连接。另一种情况是，复制一个含有 `signal` 的类是没有意义的，这种情况下你可以依赖所含 `signal` 的不可复制语义来确保复制与赋值是被禁止的。为了看得更清楚一点，考虑一个类 `some_class`, 它的定义是：

```cpp
class some_class {
  boost::signal<void (int)> some_signal;
}; 
```

对于这个类，编译器生成的复制构造函数和赋值操作符都是不能使用的。如果代码企图去使用它们，编译器就会抗议。例如，以下例子试图从 `sc1` 复制构造 `some_class sc2` ：

```cpp
int main() {
  some_class sc1;
  some_class sc2(sc1);
} 
```

编译这段程序时，编译器生成的复制构造函数试图对 `some_class` 的成员进行逐个成员的复制。由于 `signal` 的私有复制构造函数，编译器会输出以下信息：

```cpp
c:/boost_cvs/boost/boost/noncopyable.hpp: In copy constructor `
 boost::signals::detail::signal_base::signal_base(const
 boost::signals::detail::signal_base&)':
c:/boost_cvs/boost/boost/noncopyable.hpp:27: error: `
 boost::noncopyable::noncopyable(
 const boost::noncopyable&)' is private
noncopyable_example.cpp:10: error: within this context 
```

所以，无论你的含有 `signal` 的类需要哪一种复制和赋值，你都必须确保其中不会有对 `signal` 的复制！

### 管理连接

我们已经讨论了如何连接插槽到 `signal`s, 但我们还没有看到如何断开它们。有许多原因让一个插槽不应该永久地连接到一个 `signal` 上。到现在为止，我们都忽略了它，其实 `boost::signal::connect` 会返回一个 `boost::signals::connection` 实例。通过使用这个 `connection` 对象，就可以从 `signal` 断开一个插槽，也可以测试一个插槽是否已连接到 `signal`. `connection` 是到 `signal` 和插槽间的实际链接的一个句柄。由于 `signal` 和插槽间的连接的信息是由它们两者分别跟踪的，所以插槽并不知道它本身是否被连接。如果一个插槽不想与 `signal` 断开，它只要忽略掉 `signal::connect` 所返回的 `connection` 即可。还有，对一个插槽所属的组调用 `disconnect`，或者调用 `disconnect_all_slots` 都会断开插槽而无需提供插槽的 `connection`. 如果检查插槽是否还连接着 `signal` 的能力非常重要，你就只能保存 `connection` 并用它来询问 `signal`，别无它法。

`connection` 类提供了 `operator&lt;`, 这使得你可以把连接保存在标准库的容器中。为了完备性，它也提供了 `operator==` 。最后，这个类提供了一个 `swap` 成员函数，用于与另一个 `connection` 交换各自的 `signal`/slot 连接信息。以下例子示范了如何使用 `signals::connection` 类：

```cpp
#include <iostream>
#include <string>
#include "boost/signals.hpp"

class some_slot_type {
  std::string s_;
public:
  some_slot_type(const char* s) : s_(s) {}

  void operator()(const std::string& s) const {
    std::cout << s_ << ": " << s << '\n';
  }
};

int main() {
  boost::signal<void (const std::string&)> sig;

  some_slot_type sc1("sc1");
  some_slot_type sc2("sc2");

  boost::signals::connection c1=sig.connect(sc1);
  boost::signals::connection c2=sig.connect(sc2);

  // 比较
  std::cout << "c1==c2: " << (c1==c2) << '\n';
  std::cout << "c1<c2: " << (c1<c2) << '\n';

  // 检查连接
  if (c1.connected())
    std::cout << "c1 is connected to a signal\n";

  // 交换并断开
  sig("Hello there");
  c1.swap(c2);
  sig("We've swapped the connections");
  c1.disconnect();
  sig("Disconnected c1, which referred to sc2 after the swap");
} 
```

在这个例子中有两个 `connection` 对象，我们看到它们可以用 `operator&lt;` 和 `operator==` 来比较。`operator&lt;` 所实现的顺序关系是不确定的；它的存在是为了支持把 `connection`s 保存到标准库的容器中。而 `operator==` 所表示的等价关系则是有定义的。如果两个 `connection`s 引向同一个物理连接，它们就是等价的。如果两个 `connection`s 不引向任何连接，它们也是等价的。其它的 `connection`s 对都不等价。在这个例子中，我们还断开了一个 `connection`.

```cpp
c1.disconnect(); 
```

虽然 `c1` 原先是引向 `sc1` 的 `connection`，但是在断开的时候它是引向 `sc2` 的，因为我们用成员函数 `swap` 交换了这两个连接的内容。断开连接意味着在 `signal` 产生时，该插槽不再被通知。以下是该程序的运行结果：

```cpp
c1==c2: 0
c1<c2: 1
c1 is connected to a signal
sc1: Hello there
sc2: Hello there
sc1: We've swapped the connections
sc2: We've swapped the connections
sc1: Disconnected c1, which referred to sc2 after the swap 
```

如你所见，最后一次的 `signal sig` 只调用了插槽 `sc1`.

有些时候，一个插槽的 `connection` 的生存期只限于某一段特定代码的范围。这种情况类似于其它资源要求仅限于某个特定范围时，通常可以使用智能指针或其它作用域机制来处理。Boost.Signals 提供了 `connection` 的一个作用域版本，名为 `scoped_connection`. `scoped_connection` 确保该 `connection` 在 `scoped_connection` 被销毁时断开连接。`scoped_connection` 的构造函数用一个 `connection` 对象作参数，它以此方式接受其所有权。

```cpp
#include <iostream>
#include "boost/signals.hpp"

class slot {
public:
  void operator()() const {
    std::cout << "Something important just happened!\n";
  }
};

int main() {
  boost::signal<void ()> sig;
  {
    boost::signals::scoped_connection s=sig.connect(slot());
  }
  sig();
} 
```

`boost::signals::scoped_connection s` 被限定在 `main` 内的一个小范围中，在离开该范围后，`signal sig` 被调用。这里不会产生输出，因为 `scoped_connection` 已经断开了插槽与 `signal` 间的连接。使用这样的带作用域的资源可以简化代码及其维护工作。

### 用 Bind 和 Lambda 创建插槽

你已经看到 Signals 多么有用以及么灵活。但是，当你把 Boost.Signals 与 Boost.Bind 和 Boost.Lambda 结合使用时，你会发现更大的威力。这两个库，它们的详细讨论请见 "Library 9: Bind 9" 和 "Library 10: Lambda 10"，它们有助于就地创建函数对象。这意味着你可以在需要连接到 `signal` 的地方就地创建插槽(以及插槽类型)，不再需要为插槽编写一个特定的、功能单一的类，然后再创建一个实例并连接它。这样做还可以把插槽的逻辑就放在使用它 们的地方，而不是放在源代码的别的地方。最后，这些库甚至可以用于改编一些已有的库，这些已有的库不提供调用操作符，但是有别的合适的方法来处理 `signal`。

在下面的第一个例子中，我们将看到 lambda 表达式如何漂亮地创建出一些插槽类型。这些插槽可以在调用 `connect` 的地方创建。第一个插槽在调用时简单地输出一个信息到 `std::cout` 。第二个插槽检查 `signal` 传入的字符串值。如果它等于 `"Signal"`, 则输出一个信息；否则它输出另一个信息。(这些例子确实有点做作，但这种表达式可以完成任何有用的计算)。该例子中创建的最后两个插槽完成了本章前面的例子中的 `double_slot` 和 `plus_slot` 所做的工作。你会发现这个 lambda 版本更具可读性。

```cpp
#include <iostream>
#include <string>
#include <cassert>
#include "boost/signals.hpp"
#include "boost/lambda/lambda.hpp"
#include "boost/lambda/if.hpp"

int main() {
  using namespace boost::lambda;

  boost::signal<void (std::string)> sig;

  sig.connect(var(std::cout)
    << "Something happened: " << _1 << '\n');
  sig.connect(
    if_(_1=="Signal") [
      var(std::cout) << "Ok, I've got it\n"]
    .else_[
      std::cout << constant("Yeah, whatever\n")]);

  sig("Signal");
  sig("Another signal");

  boost::signal<void (int&)> sig2;
  sig2.connect(0,_1*=2); // 加倍
  sig2.connect(1,_1+=3); // 加 3
  int i=12;
  sig2(i);
  assert(i==27);
} 
```

如果你还不熟悉 C++(或其它)中的 lambda 表达式，不要为前面这段代码看起来有点糊涂而着急，你可以先看看 Bind 和 Lambda 那两章，然后再回到这个例子上来。如果你已经了解了 lambda 表达式，我可以肯定你一定会认为使用 lambda 表达式可以带来简洁的代码；而且它避免了把代码分割成多个小的函数对象。

现在让我们来看看使用绑定器来创建插槽类型。插槽必须实现一个调用操作符，但不是所有的类都适合作为 插槽。另一方面，通常可以使用一些已有的类成员函数，用绑定器重新包装它们以用作插槽。绑定器也有助于可读性，它允许处理某个事件的函数(而不是函数对 象)具有一个有意义的名字。最后，有时同一个对象需要对不同的事件作出反应，每一个都有相同的插槽签名，但是反应各有不同。因此，这种对象需要不同的成员 函数来为不同的事件所调用。在这些情形下，没有一个调用操作符适用于连接到一个 `signal`. 因此，需要一个可配置的函数对象，而 Boost.Bind 正好提供了 (就象 Boost.Lambda 中的 `bind` 工具一样) 需要的方法。

考虑一个 `signal`，它接受一个返回 `bool` 且接受一个类型 `double` 的参数的插槽类型。假设类 `some_class` 有一个成员函数 `some_function` ，它具有相符的签名，你如何把 `some_class::some_function` 连接到 `signal` 呢？一个方法是给 `some_class` 增加一个调用操作符，而该调用操作符把调用前转到 `some_function`. 这意味着要修改类的接口，而且它不好扩展。而绑定器可以做得更好。

```cpp
#include <iostream>
#include "boost/signals.hpp"
#include "boost/bind.hpp"

class some_class {
public:
  bool some_function(double d) {
    return d>3.14;
  }

  bool another_function(double d) {
    return d<0.0;
  }
};

int main() {
  boost::signal<bool (double)> sig0;
  boost::signal<bool (double)> sig1;

  some_class sc;

  sig0.connect(
    boost::bind(&some_class::some_function,&sc,_1));
  sig1.connect(
    boost::bind(&some_class::another_function,&sc,_1));

  sig0(3.1);
  sig1(-12.78);
} 
```

绑定这种方法有一个有趣的副作用：它避免了不必要的 `some_class` 实例的拷贝。绑定器持有对 `some_class` 实例的指针，而 `signal` 复制的是绑定器。不幸的是，这种方法有一个潜在的生存期管理问题：如果 `sc` 被销毁而后一个 `signal` 被调用，将导致未定义行为。这是因为绑定器将持有一个到 `sc` 的悬空指针。为了避免复制，我们必须负责保证插槽的生存期与(间接)引向它们的 `connection` 的存在一样长。当然，这正是引用计数智能指针的功能，所以这个问题很容易解决。

在使用 Boost.Signals 时，象这样使用绑定器是很常见的。无论你是使用 lambda 表达式来创建插槽，还是使用绑定器来把已有类改编为插槽类型使用，你都可以很快看到 Boost.Signals, Boost.Lambda, 与 Boost.Bind 相互配合的价值所在。它可以节省你的时间，并让你的代码更加美观和简洁。

# Signals 总结

## Signals 总结

以下情形时使用 Signals ：

*   你需要健壮的回调时

*   事件具有多个处理者时

*   `signal` 与插槽之间的连接需要在运行时可配置时

Boost.Signals 取代旧有风格的回调现在已经是很清楚了，这个库是当前可用的、最好的 signals/slots 实现之一。这个库所代表的设计模式非常著名，并且已经被研究了很长一段时间，所以这个领域已经非常成熟。一些编程语言已经在语言中直接实现了这种机制，如 .NET 中的 delegates 和 events。在 C++中，这个问题被库优美地解决了。Signals 和 slots 用于把事件的触发器机制从处理它的代码中分离出去。这种分离解耦了子系统，使它们更易于理解。它还解决了当重要事件发生时更新多个关注方的问题。在典型的程序或库中，有很多地方需要用到 signals 和 slots 。无论你是在编写一个 GUI 框架，或是一个发电站的入侵检测系统，Signals 都可以满足你的需要。它的用法很容易学习，它还提供了复杂任务所需的高级功能。例如，定制的 Combiners 可用于编写特定领域的事件处理机制。

Boost.Signals 由 Douglas Gregor 编写(他还编写了 Boost.Function)。这是一个伟大的库；谢谢你，Doug！