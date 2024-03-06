# [13] 运算符重载

# [13] 运算符重载

## FAQs in section [13]:

*   [13.1] 运算符重载的作用是什么？
*   [13.2] 运算符重载的好处是什么？
*   [13.3] 有什么运算符重载的实例？
*   [13.4] 但是运算符重载使得我的类很丑陋；难道它不是应该使我的类更清晰吗？
*   [13.5] 什么运算符能／不能被重载？
*   [13.6] 我能重载 `operator==` 以便比较两个 `char[]` 来进行字符串比较吗？
*   [13.7] 我能为“幂”运算创建一个 `operator**` 吗？
*   [13.8] 如何为`Matrix`（矩阵）类创建下标运 运算符？
*   [13.9] 为什么`Matrix`（矩阵）类的接口不应该象数组的数组？
*   [13.10] 该从外（接口优先）还是从内（数据优先）设计类？

## 13.1 运算符重载的作用是什么？

它允许你为类的用户提供一个直觉的接口。

运算符重载允许 C/C++的运算符在用户定义类型（类）上拥有一个用户定义的意义。重载的运算符是函数调用的语法修饰：

```cpp
 class Fred {
 public:
   // ...
 };

 #if 0

   // 没有运算符重载：
   Fred add(Fred, Fred);
   Fred mul(Fred, Fred);

   Fred f(Fred a, Fred b, Fred c)
   {
     return add(add(mul(a,b), mul(b,c)), mul(c,a));    // 哈哈，多可笑...
   }

 #else

   // 有运算符重载：
   Fred operator+ (Fred, Fred);
   Fred operator* (Fred, Fred);

   Fred f(Fred a, Fred b, Fred c)
   {
     return a*b + b*c + c*a;
   }

 #endif 
```

## 13.2 运算符重载的好处是什么？

通过重载类上的标准运算符，你可以发掘类的用户的直觉。使得用户程序所用的语言是面向问题的，而不是面向机器的。

最终目标是降低学习曲线并减少错误率。

## 13.3 有什么运算符重载的实例？

这里有一些运算符重载的实例：

*   `myString + yourString` 可以连接两个 `std::string` 对象
*   `myDate++` 可以增加一个 `Date` 对象
*   `a * b` 可以将两个 `Number` 对象相乘
*   `a[i]` 可以访问 `Array` 对象的某个元素
*   `x = *p` 可以反引用一个实际“指向”一个磁盘记录的 "smart pointer" —— 它实际上在磁盘上定位到 `p` 所指向的记录并返回给`x`。

## 13.4 但是运算符重载使得我的类很丑陋；难道它不是应该使我的类更清晰吗？

运算符重载使得类的用户的工作更简易，而不是为类的开发者服务的！

考虑一下如下的例子：

```cpp
 class Array {
 public:
   int& operator[] (unsigned i);      // 有些人不喜欢这种语法
   // ...
 };

 inline
 int& Array::operator[] (unsigned i)  // 有些人不喜欢这种语法
 {
   // ...
 } 
```

有些人不喜欢`operator`关键字或类体内的有些古怪的语法。但是运算符重载语法不是被期望用来使得类的开发者的工作更简易。它被期望用来使得类的用户的工作更简易：

```cpp
 int main()
 {
   Array a;
   a[3] = 4;   // 用户代码应该明显而且易懂...
 } 
```

记住:在一个面向重用的世界中，使用你的类的人有很多，而建造它的人只有一个（你自己）；因此你做任何事都应该照顾多数而不是少数。

## 13.5 什么运算符能／不能被重载？

大多数都可以被重载。C 的运算符中只有 `.`和 `? :`（以及`sizeof`，技术上可以看作一个运算符）。C++增加了一些自己的运算符，除了`::`和`.*`，大多数都可以被重载。

这是一个下标 运算符的示例（它返回一个引用）。先没有运算符重载：

```cpp
 class Array {
 public:
   int& elem(unsigned i)        { if (i > 99) error(); return data[i]; }
 private:
   int data[100];
 };

 int main()
 {
   Array a;
   a.elem(10) = 42;
   a.elem(12) += a.elem(13);
 } 
```

现在用运算符重载给出同样的逻辑：

```cpp
 class Array {
 public:
   int& operator[] (unsigned i) { if (i > 99) error(); return data[i]; }
 private:
   int data[100];
 };

 int main()
 {
   Array a;
   a[10] = 42;
   a[12] += a[13];
 } 
```

## 13.6 我能重载 `operator==` 以便比较两个 `char[]` 来进行字符串比较吗？

不行：被重载的运算符，至少一个操作数必须是用户定义类型（大多数时候是类）。

但即使 C++允许，也不要这样做。因为在此处你应该使用类似 `std::string`的类而不是字符数组，因为数组是有害的。因此无论如何你都不会想那样做的。

## 13.7 我能为“幂”运算创建一个 `operator**` 吗？

不行。

运算符的名称、优先级、结合性以及元数都是由语言固定的。在 C++中没有`operator**`，因此你不能为类类型创建它。

如果还有疑问，考虑一下`x ** y`与`x * (*y)`等同（换句话说，编译器假定 `y` 是一个指针）。此外，运算符重载只不过是函数调用的语法修饰。虽然这种特殊的语法修饰非常美妙，但它没有增加任何本质的东西。我建议你重载`pow(base,exponent)`（双精度版本在`<cmath>`中）。

顺便提一下，`operator^`可以成为幂运算，只是优先级和结合性是错误的。

## 13.8 如何为`Matrix`（矩阵）类创建下标运算符？

用 `operator()`而不是`operator[]`。

当有多个下标时,最清晰的方式是使用`operator()`而不是`operator[]`。原因是`operator[]`总是带一个参数，而`operator()`可以带任何数目的参数（在矩形的矩阵情况下，需要两个参数）。

如：

```cpp
 class Matrix {
 public:
   Matrix(unsigned rows, unsigned cols);
   double& operator() (unsigned row, unsigned col);
   double  operator() (unsigned row, unsigned col) const;
   _// ..._
  ~Matrix();                              // 析构函数
   Matrix(const Matrix& m);               // 拷贝构造函数
   Matrix& operator= (const Matrix& m);   // 赋值运算符
   // ...
 private:
   unsigned rows_, cols_;
   double* data_;
 };

 inline
 Matrix::Matrix(unsigned rows, unsigned cols)
   : rows_ (rows),
     cols_ (cols),
     data_ (new double[rows * cols])
 {
   if (rows == 0 || cols == 0)
     throw BadIndex("Matrix constructor has 0 size");
 }

 inline
 Matrix::~Matrix()
 {
   delete[] data_;
 }

 inline
 double& Matrix::operator() (unsigned row, unsigned col)
 {
   if (row >= rows_ || col >= cols_)
     throw BadIndex("Matrix subscript out of bounds");
   return data_[cols_*row + col];
 }

 inline
 double Matrix::operator() (unsigned row, unsigned col) const
 {
   if (row >= rows_ || col >= cols_)
     throw BadIndex("const Matrix subscript out of bounds");
   return data_[cols_*row + col];
 } 
```

然后，你可以使用`m(i,j)`来访问`Matrix` `m` 的元素，而不是`m[i][j]：`

```cpp
 int main()
 {
   Matrix m(10,10);
   m(5,8) = 106.15;
   std::cout << m(5,8);
   // ...
 } 
```

## 13.9 为什么`Matrix`（矩阵）类的接口不应该象数组的数组？

本 FAQ 其实是关于：某些人建立的 Matrix 类，带有一个返回 `Array` 对象的引用的`operator]`。而该`Array` 对象也带有一个 `operator[]` ，它返回 Matrix 的一个元素（例如，一个`double`的引用）。因此，他们使用类似`m[i][j]` 的语法来访问矩阵的元素，而不是[象`m(i,j)`的语法。

数组的数组方案显然可以工作，但相对于`operator()`方法来说，缺乏灵活性。尤其是，用`[][]`方法很难表现的时候，用`operator()`方法可以很简单的完成，因此`[][]`方法很可能导致差劲的表现，至少某些情况细是这样的。

例如，实现`[][]`方法的最简单途径就是使用作为密集矩阵的，以以行为主的形式保存（或以列为主，我记不清了）的物理布局。相反，`operator()` 方法完全隐藏了矩阵的物理布局，在这种情况下，它可能带来更好的表现。

可以这么认为：`operator()`方法永远不比`[][]`方法差，有时更好。

*   `operator()` 永远不差，是因为用`operator()`方法实现以行为主的密集矩阵的物理布局非常容易。因此，当从性能观点出发，那样的结构正好是最佳布局时，`operator()`方法也和`[][]`方法一样简单（也许`operator()`方法更容易一点点，但我不想夸大其词）。
*   `operator()`方法有时更好，是因为当对于给定的应用，有其它比以行为主的密集矩阵更好的布局时，用 `operator()` 方法比`[][]`方法实现会容易得多。

作为一个物理布局使得实现困难的例子，最近的项目发生在以列访问矩阵元素（也就是，算法访问一列中的所有元素，然后是另一列等），如果物理布局是以行为主的，对矩阵的访问可能会“cache 失效”。例如，如果行的大小几乎和处理器的 cache 大小相当，那么对每个元素的访问，都会发生“cache 不命中”。在这个特殊的项目中，我们通过将映射从逻辑布局（行，列）变为物理布局（列，行），性能得到了 20%的提升。

当然，还有很多这类事情的例子，而稀疏矩阵在这个问题中则是又一类例子。通常，使用`operator()`方法实现一个稀疏矩阵或交换行／列顺序更容易，`operator()`方法不会损失什么，而可能获得一些东西——它不会更差，却可能更好。

使用 `operator()` 方法。

## 13.10 该从外（接口优先）还是从内（数据优先）设计类？

从外部！

良好的接口提供了一个简化的，以用户词汇表达的视图。在面向对象软件的情况下，接口通常是单个类或一组紧密结合的类的 public 方法的集合.

首先考虑对象的逻辑特征是什么，而不是打算如何创建它。例如，假设要创建一个`Stack`（栈）类，其包含一个 `LinkedList`：

```cpp
 class Stack {
 public:
   _// ..._
 private:
   LinkedList list_;
 }; 
```

Stack 是否应该有一个返回`LinkedList`的`get()`方法？或者一个带有`LinkedList`的`set()`方法？或者一个带有`LinkedList`的构造函数？显然，答案是“不”，因为应该从外向里设计接口。也就是说，`Stack`对象的用户并不关心 `LinkedList`；他们只关心 pushing 和 popping。

现在看另一个更微妙的例子。假设 `LinkedList`类使用`Node`对象的链表来创建，每一个`Node`对象有一个指向下一个`Node`的指针：

```cpp
 class Node { /*...*/ };

 class LinkedList {
 public:
   // ...
 private:
   Node* first_;
 }; 
```

`LinkedList`类是否应该有一个让用户访问第一个`Node`的`get()`方法？`Node`对象是否应该有一个让用户访问链中下一个 `Node`的 `get()`方法？换句话说，从外部看，`LinkedList`应该是什么样的？`LinkedList` 是否实际上就是一个 `Node` 对象的链？或者这些只是实现的细节？如果只是实现的细节，`LinkedList` 将如何让用户在某时刻访问 `LinkedList` 中的每一个元素？

某人的回答：`LinkedList` 不是的 `Node` 链。它可能的确是用 `Node` 创建的，但这不是本质。它的本质是元素的序列。因此，`LinkedList` 抽象应该提供一个“LinkedListIterator”，并且“LinkedListIterator”应该有一个`operator++` 来访问下一个元素，并且有一对`get()`/`set()`来访问存储于`Node` 的值（`Node` 元素中的值只由`LinkedList`用户负责，因此有一对`get()`/`set()`以允许用户自由地维护该值）。

从用户的观点出发，我们可能希望 `LinkedList`类支持看上去类似使用指针算法访问数组的 运算符：

```cpp
 void userCode(LinkedList& a)
 {
   for (LinkedListIterator p = a.begin(); p != a.end(); ++p)
     std::cout << *p << '\n';
 } 
```

实现这个接口，`LinkedList`需要一个 `begin()`方法和 `end()`方法。它们返回一个“LinkedListIterator”对象。该“LinkedListIterator”需要一个前进的方法，`++p` ；访问当前元素的方法，`*p`；和一个比较运算符，`p != a.end()`。

如下的代码，关键在于 `LinkedList` 类没有任何让用户访问 `Node` 的方法。`Node` 作为实现技术被完全地隐藏了。 `LinkedList` 类内部可能用双重链表取代，甚至是一个数组，区别仅仅在于一些诸如`prepend(elem)` 和 `append(elem)`方法的性能上。

```cpp
 #include <cassert>    // Poor man's exception handling

 class LinkedListIterator;
 class LinkedList;

 class Node {
   // No public members; this is a "private class"_
   friend LinkedListIterator;   // 友员类
   friend LinkedList;
   Node* next_;
   int elem_;
 };

 class LinkedListIterator {
 public:
   bool operator== (LinkedListIterator i) const;
   bool operator!= (LinkedListIterator i) const;
   void operator++ ();   // Go to the next element
   int& operator*  ();   // Access the current element
 private:
   LinkedListIterator(Node* p);
   Node* p_;
   friend LinkedList;  // so LinkedList can construct a LinkedListIterator
 };

 class LinkedList {
 public:
   void append(int elem);    // Adds elem after the end_
   void prepend(int elem);   // Adds elem before the beginning
   // ...
   LinkedListIterator begin();
   LinkedListIterator end();
   // ...
 private:
   Node* first_;
 }; 
```

这些是显然可以内联的方法（可能在同一个头文件中）：

```cpp
 inline bool LinkedListIterator::operator== (LinkedListIterator i) const
 {
   return p_ == i.p_;
 }

 inline bool LinkedListIterator::operator!= (LinkedListIterator i) const
 {
   return p_ != i.p_;
 }

 inline void LinkedListIterator::operator++()
 {
   assert(p_ != NULL);  // or if (p_==NULL) throw ...
   p_ = p_->next_;
 }

 inline int& LinkedListIterator::operator*()
 {
   assert(p_ != NULL);  // or if (p_==NULL) throw ...
   return p_->elem_;
 }

 inline LinkedListIterator::LinkedListIterator(Node* p)
   : p_(p)
 { }

 inline LinkedListIterator LinkedList::begin()
 {
   return first_;
 }

 inline LinkedListIterator LinkedList::end()
 {
   return NULL;
 } 
```

结论：链表有两种不同的数据。存储于链表中的元素的值由链表的用户负责（并且只有用户负责，链表本身不阻止用户将第三个元素变成第五个），而链表底层结构的数据（如 `next` 指针等）值由链表负责（并且只有链表负责，也就是说链表不让用户改变（甚至看到！）可变的`next` 指针）。

因此 `get()`/`set()` 方法只获取和设置链表的元素，而不是链表的底层结构。由于链表隐藏了底层的指针等结构，因此它能够作非常严格的承诺（例如，如果它是双重链表，它可以保证每一个后向指针都被下一个 `Node` 的前向指针匹配）。

我们看了这个例子，类的一些数据的值由用户负责（这种情况下需要有针对数据的`get()`/`set()`方法），但对于类所控制的数据则不必有`get()`/`set()`方法。

注意：这个例子的目的不是为了告诉你如何写一个链表类。实际上不要自己做链表类，而应该使用编译器所提供的“容器类”的一种。理论上来说，要使用标准容器类之一，如：`std::list<T>` 模板。