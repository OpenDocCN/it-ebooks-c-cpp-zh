# [21] 继承 — 适当的继承和可置换性

# [21] 继承 — 适当的继承和可置换性

## FAQs in section [21]:

*   [21.1] 我应该隐藏基类的公有成员函数吗？
*   [21.2] `Derived* —> Base*` 可以很好地工作; 为什么 `Derived** —> Base**` 不行？
*   [21.3] parking-lot-of-Car（停车场）是一种 parking-lot-of-Vehicle（交通工具停泊场）吗？
*   [21.4] `Derived`数组是一种 `Base`数组吗？
*   [21.5] 派生类数组(array-of-`Derived`)“不是一种”基类数组(array-of-`Base`)是否意味着数组不好？
*   [21.6] `Circle`（圆）是一种 `Ellipse`（椭圆）吗？
*   [21.7] 对于“圆是/不是一种椭圆”这个两难问题，有其它说法吗？
*   [21.8] 但我是数学博士，我相信圆是一种椭圆！这是否意味着 Marshall Cline 是傻瓜？或者 C++是傻瓜？或者 OO 是傻瓜？
*   [21.9] 也许椭圆应该从圆继承？
*   [21.10] 但我的问题与圆和椭圆无关，这种无聊的例子对我有什么好处？

## 21.1 我应该隐藏基类的公有成员函数吗？

不要，不要，不要这样做。永远不要！

试图隐藏（消除、废除、私有化）继承而来的公有成员函数是非常常见的设计错误。通常这产生于浆糊脑袋。

(注意: 本 FAQ 的论述仅与公有继承（`public` inheritance）有关; 私有和保护继承并不相同)

## 21.2 `Derived* —> Base*` 可以很好地工作; 为什么 `Derived** —> Base**` 不行？

由于`Derived`对象是一种`Base`对象，C++允许`Derived*` 转换成 `Base*`。然而，将 `Derived**`转换成 `Base**` 将产生错误。尽管这个错误不是显而易见的，这未尝不是件好事。例如，如果你能够将`Car**`转换成 `Vehicle**`（译注：Vehicle 意为交通工具），并且如果你能同样的将`NuclearSubmarine**`（译注：NuclearSubmarine 意为核潜艇） 转换成`Vehicle**`，那么你可能给这两个指针赋值，并最终使 `Car*` 指针指向 `NuclearSubmarine`：

```cpp
 class Vehicle {
 public:
   virtual ~Vehicle() { }
   virtual void startEngine() = 0;
 };

 class Car : public Vehicle {
 public:
   virtual void startEngine();
   virtual void openGasCap();
 };

 class NuclearSubmarine : public Vehicle {
 public:
   virtual void startEngine();
   virtual void fireNuclearMissle();
 };

 int main()
 {
   Car   car;
   Car*  carPtr = &car;
   Car** carPtrPtr = &carPtr;
   Vehicle** vehiclePtrPtr = carPtrPtr;  // 这在 C++中是一个错误
   NuclearSubmarine  sub;
   NuclearSubmarine* subPtr = &sub;
   *vehiclePtrPtr = subPtr;
   // 最后这行将导致 carPtr 指向 sub !
   carPtr->openGasCap();  // 这将调用 fireNuclearMissle()! （译注：也就是发射核弹）
 } 
```

换句话说，如果从`Derived**` 到`Base**`的转换是合法的，那么`Base**`将可能被解除引用（易变的 `Base*`），并且 `Base*`可能被指向不同的派生类对象，这将导致严重的国家安全问题（天知道如果你调用了`NuclearSubmarine`（核潜艇）对象的 `openGasCap()`成员函数会发生什么!!而你却认为这是一个`Car`对象!!——试一下以上的代码，看看会发生什么——大多数的编译器会调用`NuclearSubmarine::fireNuclearMissle()`!

(注意: 本 FAQ 的论述仅与公有继承（`public` inheritance）有关; 私有和保护继承并不相同)

## 21.3 parking-lot-of-Car（停车场）是一种 parking-lot-of-Vehicle（交通工具停泊场）吗？

不。

我知道这听起来很奇怪，但这是事实。你可以将这看作为以上 FAQ 的直接结论，或者你可以这样来理解：如果这个“是一种”关系成立的话，那么就可以将 parking-lot-of-Vehicle 类型的指针指向一个 parking-lot-of-Car。但是，parking-lot-of-Vehicle 有 `addNewVehicleToParkingLot(Vehicle&)`成员函数用来向停泊场添加任何 `Vehicle`（交通工具）对象。这样将允许你在 parking-lot-of-Car（停车场）停泊一个`NuclearSubmarine`（核潜艇）。当然，当某人认为从 parking-lot-of-Car 删除一个`Car`对象，而实际是一个`NuclearSubmarine`时，他会非常惊讶。

用另一种方法阐述这个事实：一种事物的容器不是一种任何事物的容器。也许很难接受，但这是事实。

你可以不喜欢它，但必须接受它。

我们在 OO/C++训练课程使用的最后一个例子：“一袋苹果不是一袋水果”。如果一袋苹果能够被传递给一袋水果的话，就可以把香蕉放入袋中，即使它被认为里面只能放苹果！

(注意: 本 FAQ 的论述仅与公有继承（`public` inheritance）有关; 私有和保护继承并不相同)

## 21.4 `Derived`数组是一种 `Base`数组吗？

不。

这是以上 FAQ 的结论。不幸的是它会把你带入困境，考虑一下这个：

```cpp
 class Base {
 public:
   virtual void f();             // 1
 };

 class Derived : public Base {
 public:
   // ...
 private:
   int i_;                       // 2
 };

 void userCode(Base* arrayOfBase)
 {
   arrayOfBase[1].f();           // 3
 }

 int main()
 {
   Derived arrayOfDerived[10];   // 4
   userCode(arrayOfDerived);     // 5
 } 
```

编译器会认为这是完美的类型安全。编号 5 的这一行将 `Derived*` 转换为 `Base*`。但实际上这样做是可怕的：由于 `Derived`比`Base` 大，在编号 3 的这一行的指针运算是错误的：当编译器计算 `arrayOfBase1]`的地址时使用 `sizeof(Base)`，而数组其实是一个`Derived`数组，这意味着在编号 3 的这一行的所计算的地址（以及之后的成员函数 `f()` 的调用）并不在任何对象的起始位置！而在`Derived`对象的中间。假设你的编译器使用通常的方法寻找[虚函数，那么将导致第一个`Derived`对象的 `int i_` 被重新解释，将它看作指向虚函数表的指针，跟随着这个“指针”（意味着我们正在访问一个随机的内存位置），并将内存中那个位置的前几个字节解释为 C++成员函数的地址，然后将它们（随机的内存地址）装载到指令寄存器并开始从那个内存区产生机器指令。发生这样情况的几率相当高。

根本问题是 C++无法区别指向事物的指针和指向事物数组的指针。自然的，C++是从 C 继承了这一特征。

注意：如果我们使用类似数组（array-like）的类（例如，标准库中的`std::vector<Derived>`）来代替原始的数组，这个问题将会被作为编译时错误找出而不是运行时的灾难。

(注意: 本 FAQ 的论述仅与公有继承（`public` inheritance）有关; 私有和保护继承并不相同)

## 21.5 派生类数组（array-of-`Derived）“不是一种”基类数组（array-of-`Base）是否意味着数组不好？

是的，数组很差劲。（开个玩笑）。

真诚的来说，数组和指针非常接近，并且指针很难处理。但是如果我们完全掌握了为什么从设计角度来看，以上 FAQ 所说的会是一个问题（例如，如果你真的知道为什么事物的容器不是一种任何事物的容器），并且你认为将维护你的代码的其他人都完全掌握这些 OO 的设计事实的话，那么你可以自由使用数组。但是如果你象大多数人一样的话，你应该使用诸如标准库的`std::vector<T>`这样的模板容器类而不是原始的数组。

(注意: 本 FAQ 的论述仅与公有继承（`public` inheritance）有关; 私有和保护继承并不相同)

## 21.6 `Circle`（圆）是一种 `Ellipse`（椭圆）吗？

如果椭圆允许改变圆率，则不是。

例如，假设椭圆有一个`setSize(x,y)`成员函数，并且这个成员函数允许椭圆的 `width()`是`x`，`height()` 是`y`。在这种情况下，圆无法是一种椭圆。很简单，如果椭圆能做某些圆不能做的事，则圆不是一种椭圆。

据此推出圆和椭圆的两种（合法的）关系：

*   使圆类和椭圆类完全无关
*   使圆和椭圆都从一个基类派生，该基类是“不能执行不对称`setSize()`运算的椭圆”

在第一种情况下，椭圆可以从`AsymmetricShape`（不对称图形）类派生，`setSize(x,y)`可以在`AsymmetricShape`类中声明。而圆可以从有`setSize(size)`成员函数的`SymmetricShape`（对称图形）类派生。

在第二种情况下，`Oval`（卵形）类可以只有`setSize(size)`来同时设置 `width()`和`height()`的大小。椭圆和圆都继承自`Oval`。椭圆（但不是圆）可以增加`setSize(x,y)`运算（但如果`setSize()`成员函数名称重复，当心隐藏规则）

(注意: 本 FAQ 的论述仅与公有继承（`public` inheritance）有关; 私有和保护继承并不相同)

(注意: `setSize(x,y)`并不是神圣的。依赖于你的目标，防止用户改变椭圆的尺寸也是可以的。在某些情况下，椭圆没有`setSize(x,y)`方法是有效的设计选择。然而这个系列的讨论是当你想为一个已存在的类建立一个派生类并且基类含有一个“无法接受”的方法时，该如何做。当然理想情形是在基类不存在时就发现这个问题。但生活并不总是理想的……)

## 21.7 对于“圆是/不是一种椭圆”这个两难问题，有其它说法吗？

如果你主张所有椭圆是可以被压成不对称的，并且你主张圆是一种椭圆，并且你主张圆不能被压成不对称的。无疑你必须调整（实际上是撤回）你的主张之一。由此，你要么去掉`Ellipse::setSize(x,y)`，去掉圆和椭圆的继承关系，要么承认你的 `Circle`s（圆）不必是正圆。

这里有两个 OO/C++编程新手通常会陷入的陷阱。他们会试图用代码的技巧来弥补设计的缺陷（他们会重定义`Circle::setSize(x,y)`来抛出异常，调用`abort()`，取两个参数的平均数，或者什么都不做）。不幸的是，由于用户期望 `width() == x`并且 `height() == y`，所以这些技巧会使用户惊讶。而让用户惊讶是不允许的。

如果保持“圆是一种椭圆”的继承关系对你来说非常重要，那么你只能削弱椭圆的`setSize(x,y)`所做的承诺。例如，你可以改变承诺为，“该城圆函数可以把 `width()`设置为`x`并且/或把 `height()`设置为`y`，或不做什么事情”。不幸的是由于用户没有任何意义的行为可以倚靠，这样会冲淡契约。因此整个层次都变得没有价值（如果某人问你到对象能做什么，而你只能耸耸肩膀的话，你很难说服他取使用这个对象）

(注意: 本 FAQ 的论述仅与公有继承（`public` inheritance）有关; 私有和保护继承并不相同)

(注意: `setSize(x,y)`并不是神圣的。依赖于你的目标，防止用户改变椭圆的尺寸也是可以的。在某些情况下，椭圆没有`setSize(x,y)`方法是有效的设计选择。然而这个系列的讨论是当你想为一个已存在的类建立一个派生类并且基类含有一个“无法接受”的方法时，该如何做。当然理想情形是在基类不存在时就发现这个问题。但生活并不总是理想的……)

## 21.8 但我是数学博士，我相信圆是一种椭圆！这是否意味着 Marshall Cline 是傻瓜？或者 C++是傻瓜？或者 OO 是傻瓜？

事实上，这并不意味着这些。而是意味着你的直觉是错误的。

看，我收到并回复了大量的关于这个主题的热情的 e-mail。我已经给各地上千个软件专家讲授了数百次。我知道它违背了你的直觉。但相信我，你的直觉是错误的。

真正的问题是你的直觉中的“是一种（kind of）”的概念不符合 OO 中的适当的继承（学术上称为“子类型(subtyping)”）概念。派生类对象最起码必须是可以取代基类对象的。在圆/椭圆的情况下，`setSize(x,y)`成员函数违背了这个可置换性。

你有三个选择：[1]从`Ellipse`（椭圆）类中删除 `setSize(x,y)`成员函数（从而废弃调用`setSize(x,y)`成员函数的已存在代码），[2]允许`Circle`（圆）的高和宽不同（一个不对称的圆），或者[3]去掉继承关系。抱歉，但没有其他选择。有人提过另一个选项，让圆和椭圆都从第三个通用基类派生，但这只不过是以上选项[3]的变种罢了。

换一种说法就是，你要么使基类弱一些（在这里就是说你不能为椭圆的高和宽设置不同的值），要么使派生类强一些（在这里就是使圆同时具有对称的和不对称的能力）。当这些都无法令人满意（就如圆/椭圆例子），通常就简单的消除继承关系。如果继承关系必须存在，你只能从基类中删除变形成员函数（`setHeight(y)`，`setWidth(x)，` 和`setSize(x,y)`）

(注意: 本 FAQ 的论述仅与公有继承（`public` inheritance）有关; 私有和保护继承并不相同)

(注意: `setSize(x,y)`并不是神圣的。依赖于你的目标，防止用户改变椭圆的尺寸也是可以的。在某些情况下，椭圆没有`setSize(x,y)`方法是有效的设计选择。然而这个系列的讨论是当你想为一个已存在的类建立一个派生类并且基类含有一个“无法接受”的方法时，该如何做。当然理想情形是在基类不存在时就发现这个问题。但生活并不总是理想的……)

## 21.9 也许椭圆应该从圆继承？

如果圆是基类，椭圆是派生类的话，那么你会面临许多新的问题。例如，假设圆有`radius()`方法（译注：设置半径的成员函数）。那么椭圆也会有`radius()`方法，但那没有意义：一个椭圆（可能不对称）的半径是什么意思？

如果你克服这个障碍（也就是使得`Ellipse::radius()`返回主轴和辅轴的平均值或其它办法），那么`radius()`和 `area()`（译注：得到面积的成员函数）之间的关联就会有问题。比如，假设圆有`area()`方法返回的是 3.14159 乘以`radius()`返回值的平方。而`Ellipse::area()`将不会返回椭圆的真实面积，否则你必须记住让`radius()`返回符合上述公式的某个值。

即使你克服了这个问题（也就是使得`Ellipse::radius()`返回了椭圆的面积除以 pi 的平方根），你还要应付`circumference()`方法（译注：计算周长的成员函数）。比如，假设圆有`circumference()`方法返回 2 乘以 pi 乘以`radius()`的返回值。现在你的麻烦是：对于椭圆没有办法两碗水端平了：椭圆类不得不在面积，或者周长，或者两者的计算上撒谎。（译注：对于椭圆，面积和周长的计算无法同时得到正确答案，因为它们都使用了`radius()`的返回值，而它们对于`radius()`的返回值的要求却不相同，`radius()`无法同时满足它们的需要）

底线：只要派生类遵守基类的承诺，你就可以使用继承。而不能仅仅因为你感觉上象继承或仅仅因为你想使得代码被重用就使用继承。只有在(a)派生类的方法能遵守基类所做的所有承诺，并且(b)用户不会被你搞糊涂，并且(c)使用继承能明显获得实在的时间上的，金钱上的或风险上的改进时，才应该使用继承。

## 21.10 但我的问题与圆和椭圆无关，这种无聊的例子对我有什么好处？

啊，有点小误会。你认为圆/椭圆例子是无聊的，但实际上，你的问题和它是同性质的。

我不在意你的继承问题是什么，但所有（是的，所有）不良的继承都可以归结为“圆不是一种椭圆”的例子。

这就是为什么：不良的继承总有一个有额外能力（经常是一个或两个额外的成员函数；有时是一个或多个成员函数给出的承诺）的基类，而派生类却无法满足它。你要么使基类弱一些，派生类强一些，要么消除继承关系。我见过很多很多很多不良的继承方案，相信我，它们都可以归结为圆/椭圆的例子。

因此，如果你真的理解了圆/椭圆的例子，你就能找出所有的不良继承。如果你没有理解圆/椭圆问题，那么你很可能犯一些严重的并且昂贵的继承错误。

令人忧伤，但是真的。

(注意: 本 FAQ 的论述仅与公有继承（`public` inheritance）有关; 私有和保护继承并不相同)