# [22] 继承 — 抽象基类(ABCs)

# [22] 继承 — 抽象基类(ABCs)

## FAQs in section [22]:

*   [22.1] 将接口和实现分离的作用是什么？
*   [22.2] 在 C++中如何分离接口和实现（就象 Modula-2）？
*   [22.3] 什么是 ABC？
*   [22.4] 什么是“纯虚”成员函数？
*   [[22.5] 如何为包含指向（抽象）基类的指针的类定义拷贝构造函数或赋值操作符？](abcs.html#[22.5])

## 22.1 将接口和实现分离的作用是什么？

接口是公司最有价值的资源。设计接口比用一堆类来实现这个接口更费时间。而且接口需要更昂贵的人力的时间。

既然接口如此有价值，它们应该被保护，以免因为数据结构和其他实现的改变而被破坏。因此，应该将接口和实现分离。

## 22.2 在 C++中如何分离接口和实现（就象 Modula-2）？

使用 ABC。（译注：即抽象基类 abstract base class）

## 22.3 什么是 ABC?

抽象基类（abstract base class）。

在设计层次，抽象基类（ABC）对应于抽象概念。如果你问一个机修工他是否修理交通工具，他可能想知道你所说的是哪种交通工具。他不修理航天飞机、远洋轮、自行车或核潜艇。问题在于“交通工具”是一个抽象概念（例如，除非你知道你要建造哪种交通工具，否则你无法建造一个“交通工具”）。在 C++中，`Vehicle`（交通工具）类是一个 ABC，而`Bicycle`（自行车），`SpaceShuttle`（航天飞机）等则是派生类（`OceanLiner(远洋轮）`是一种`Vehicle`）。在真实世界的 OO 中，ABC 无处不在。

在程序语言层次上，抽象基类（ABC）是有一个或多个纯虚成员函数的类。无法建立抽象基类的对象（实例）。

## 22.4 什么是“纯虚”成员函数？

将普通类变成抽象基类（也就是 ABC）的成员函数。通常只在派生类中实现它。

某些成员函数只在概念中存在，而没有合理的定义。例如，假设我让你在坐标`(x,y)`处画一个图形，大小为 7。你会问我：“我应该画哪种图形？”（圆，矩形，六边形等，画法都不同）。在 C++中，我们必须指出 `draw()` 成员函数的实在物（由此用户才能在有一个`Shape*`或者 `Shape&` 的时候调用它），但我们认识到，在逻辑上，它只能在子类中被定义：

```cpp
 class Shape {
 public:
   virtual void draw() const = 0;  // = 0 表示它是 "纯虚" 的
   // ...
 }; 
```

这个纯虚函数使 `Shape` 成为了 ABC（抽象基类）。如果你愿意，你可以将“`= 0;`”语法看作为代码位于 NULL 指针处。因此 `Shape` 向它的用户承诺了一个服务，然而 `Shape` 无法提供任何代码来实现这个承诺。这样做使得即使基类没有足够的信息来实际定义成员函数时，也强制了任何由 `Shape` 派生的具体类的对象须给出成员函数。

注意，为纯虚函数提供一个实现是可能的，但是这样通常会使初学者糊涂，并且最好避免这样，直到熟练之后。

## 22.5 如何为包含指向（抽象）基类的指针的类定义拷贝构造函数或赋值操作符？

如果类拥有被（抽象）基类指针指向的对象，则在（抽象）基类中使用虚构造函数用法](virtual-functions.html#[20.5])。就如同一般用法一样，在基类中声明一个[纯虚方法 `clone()` ：

```cpp
 class Shape {
 public:
   // ...
   virtual Shape* clone() const = 0;   // 虚拟（拷贝）构造函数
   // ...
 }; 
```

然后在每个派生类中实现 `clone()`方法：

```cpp
 class Circle : public Shape {
 public:
   // ...
   virtual Shape* clone() const { return new Circle(*this); }
   // ...
 };

 class Square : public Shape {
 public:
   // ...
   virtual Shape* clone() const { return new Square(*this); }
   // ...
 }; 
```

现在假设每个 `Fred` 对象有一个 `Shape`对象。`Fred`对象自然不知道 `Shape`是圆还是矩形还是……。`Fred`的拷贝构造函数和赋值操作符将调用`Shape`的`clone()`方法来拷贝对象：

```cpp
 class Fred {
 public:
   Fred(Shape* p) : p_(p) { assert(p != NULL); }   // p must not be NULL
  ~Fred() { delete p_; }
   Fred(const Fred& f) : p_(f.p_->clone()) { }
   Fred& operator= (const Fred& f)
     {
       if (this != &f) {              // 检查自赋值 _
         Shape* p2 = f.p_->clone();   // Create the new one FIRST...
         delete p_;                   // ...THEN delete the old one
         p_ = p2;
       }
       return *this;
     }
   // ...
 private:
   Shape* p_;
 }; 
```