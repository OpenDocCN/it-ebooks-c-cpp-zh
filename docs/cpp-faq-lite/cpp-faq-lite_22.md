# [24] 继承 — 私有继承和保护继承

# [24] 继承 — 私有继承和保护继承

## FAQs in section [24]:

*   [24.1] 如何表示“私有继承”？
*   [24.2] 私有继承和组合(composition)有什么类似？
*   [24.3] 我应该选谁：组合还是私有继承？
*   [24.4] 从私有继承类到父类需要指针类型转换吗？
*   [24.5] 保护继承和私有继承的关系是什么？
*   [24.6] 私有继承和保护继承的访问规则是什么？

## 24.1 如何表示“私有继承”？

用 `: private` 代替 `: public`，例如

```cpp
 class Foo : private Bar {
 public:
   // ...
 }; 
```

## 24.2 私有继承和组合(composition)有什么类似？

私有继承是组合的一种语法上的变形（聚合或者 “有一个”）

例如，“汽车有一个(has-a)引擎”关系可以用单一组合表示为：

```cpp
 class Engine {
 public:
   Engine(int numCylinders);
   void start();                 // Starts this Engine
 };

 class Car {
 public:
   Car() : e_(8) { }             // Initializes this Car with 8 cylinders
   void start() { e_.start(); }  // Start this Car by starting its Engine
 private:
   Engine e_;                    // Car has-a Engine
 }; 
```

同样的“有一个”关系也能用私有继承表示：

```cpp
 class Car : private Engine {    // Car has-a Engine
 public:
   Car() : Engine(8) { }         // Initializes this Car with 8 cylinders
   using Engine::start;          // Start this Car by starting its Engine
 }; 
```

两种形式有很多类似的地方：

*   两种情况中，都只有一个 Engine 被确切地包含于 Car 中
*   两种情况中，在外部都不能将 `Car*` 转换为 `Engine*`
*   两种情况中，`Car`类都有一个`start()`方法，并且都在包含的`Engine`对象中调用`start()`方法。

也有一些区别：

*   如果你想让每个 `Car`都包含若干 `Engine`，那么只能用单一组合的形式
*   私有继承形式可能引入不必要的多重继承
*   私有继承形式允许 `Car` 的成员将 `Car*` 转换成`Engine*`
*   私有继承形式允许访问基类的保护（`protected`）成员
*   私有继承形式允许 `Car` 重写 `Engine` 的虚函数
*   私有继承形式赋予`Car`一个更简洁（20 个字符比 28 个字符）的仅通过 `Engine`调用的`start()`方法

注意，私有继承通常用来获得对基类的 protected: 成员的访问，但这只是短期的解决方案（权宜之计）

## 24.3 我应该选谁：组合还是私有继承？

尽可能用组合，万不得已才用私有继承

通常你不会想访问其他类的内部，而私有继承给你这样的一些的特权（和责任）。但是私有继承并不有害。只是由于它增加了别人更改某些东西时，破坏你的代码的可能性，从而使维护的花费更昂贵。

当你要创建一个 `Fred` 类，它使用了 `Wilma` 类的代码，并且`Wilma` 类的这些代码需要调用你新建的 `Fred` 类的成员函数。在这种情况下，`Fred` 调用 `Wilma` 的非虚函数，而`Wilma` 调用（通常是纯虚函数）被`Fred`重写的这些函数。这种情况，用组合是很难完成的。

```cpp
 class Wilma {
 protected:
   void fredCallsWilma()
     {
       std::cout << "Wilma::fredCallsWilma()\n";
       wilmaCallsFred();
     }
   virtual void wilmaCallsFred() = 0;   // 纯虚函数
 };

 class Fred : private Wilma {
 public:
   void barney()
     {
       std::cout << "Fred::barney()\n";
       Wilma::fredCallsWilma();
     }
 protected:
   virtual void wilmaCallsFred()
     {
       std::cout << "Fred::wilmaCallsFred()\n";
     }
 }; 
```

## 24.4 从私有继承类到父类需要指针类型转换吗？

一般来说，不。

对于该私有继承类的成员函数或者友元来说，和基类的关系是已知的，并且这种从`PrivatelyDer*` 到 `Base*` （或 `PrivatelyDer&` 到 `Base&`）的向上转换是安全的，不需要也不推荐进行类型转换。

然而，对于该私有继承类（`PrivatelyDer`）的用户来说，应该避免这种不安全的转换。因为它基于`PrivatelyDer`的私有实现，它可以自行改变。

## 24.5 保护继承和私有继承的关系是什么？

相同点：都允许重写私有/保护基类的虚函数，都不表明派生类“是一种（a kind-of）”基类。

不同点：保护继承允许派生类的派生类知道继承关系。如此，子孙类可以有效的得知祖先类的实现细节。这样既有好处（它允许保护继承的子类使用它和保护基类的关联）也有代价（保护派生类不能在无潜在破坏更深派生类的情况下改变这种关联）。

保护继承使用 `: protected` 语法：

```cpp
 class Car : protected Engine {
 public:
   // ...
 }; 
```

## 24.6 私有继承和保护继承的访问规则是什么？

以这些类为例：

```cpp
 class B                    { /*...*/ };
 class D_priv : private   B { /*...*/ };
 class D_prot : protected B { /*...*/ };
 class D_publ : public    B { /*...*/ };
 class UserClass            { B b; /*...*/ }; 
```

子类都不能访问 `B` 的私有部分。在 `D_priv`中，B 的公有和保护部分都是私有的。在 `D_prot`中，`B` 的公有和保护部分都是保护的。在 `D_publ` 中，`B` 的公有部分是公有的，`B` 的保护部分是保护的（`D_publ` 是一种 `B`）。`UserClass` 类仅仅可以访问 `B` 的公有部分。

要使 `B` 的公有成员在 `D_priv` 或 `D_prot`中也是公有的，则使用 `B::` 前缀声明成员的名称。例如，要使 `B::f(int,float)`成员在 `D_prot`中公有，应该这样写：

```cpp
 class D_prot : protected B {
 public:
   using B::f;  // 注意: 不是 using B::f(int,float)
 }; 
```