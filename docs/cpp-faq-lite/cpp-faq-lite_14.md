# [16] 自由存储（Freestore）管理

# [16] 自由存储（Freestore）管理

## FAQs in section [16]:

*   [16.1] `delete p` 删除指针 `p`，还是删除指针所指向的数据 `*p`?
*   [16.2] 可以 `free()` 一个由 `new` 分配的指针吗？可以 `delete` 一个由 `malloc()` 分配的指针吗？
*   [16.3] 为什么要用 `new` 取代原来的值得信赖的 `malloc()`？
*   [16.4] 可以在一个由 `new` 分配的指针上使用 `realloc()` 吗？
*   [16.5] 需要在 `p = new Fred()`之后检查`NULL` 吗？
*   [16.6] 我如何确信我的（古老的）编译器会自动检查 `new` 是否返回`NULL`？
*   [16.7] 在`delete p`之前需要检查 `NULL`吗？
*   [16.8] `delete p` 执行了哪两个步骤？
*   [16.9] 在 `p = new Fred()`中，如果`Fred`构造函数抛出异常，是否会内存“泄漏”？
*   [16.10] 如何分配／释放一个对象的数组？
*   [16.11] 如果 `delete` 一个由`new T[n]`分配的数组，漏了 `[]` 会如何？
*   [16.12] 当`delete`一个内建类型(`char`, `int`, 等) 的数组时，能去掉 `[]` 吗？
*   [16.13] `p = new Fred[n]`之后，编译器在`delete[] p`的时候如何知道有 `n` 个对象被析构？
*   [16.14] 成员函数调用`delete this`合法吗？
*   [16.15] 如何用 `new` 分配多维数组？
*   [16.16] 但前一个 FAQ 的代码太技巧而容易出错，有更简单的方法吗？
*   [16.17] 但上面的 `Matrix` 类是针对 `Fred`的！有办法使它通用吗？
*   [16.18] 还有其他方法建立 `Matrix` 模板吗？
*   [16.19] C++ 有能够在运行期指定长度的数组吗？
*   [16.20] 如何使类的对象总是通过 `new` 来创建而不是局部的或者全局的／静态的对象？
*   [16.21] 如何进行简单的引用计数？
*   [16.22] 如何用写时拷贝（copy-on-write）语义提供引用计数？
*   [16.23] 如何为派生类提供写时拷贝（copy-on-write）语义的引用计数？
*   [16.24] 你能绝对地防止别人破坏引用计数机制吗？如果能的话，你会这么做吗？
*   [16.25] 在 C++中能使用垃圾收集吗？
*   [16.26] C++的两种垃圾收集器是什么？
*   [16.27] 还有哪里能得到更多的 C++垃圾收集信息？

## 16.1 `delete p` 删除指针 `p`，还是删除指针所指向的数据 `*p`?

指针指向的数据。

关键字应该是 `delete_the_thing_pointed_to_by`。同样的情况也发生在 C 中释放指针所指的内存： `free(p)`实际上是指`free_the_stuff_pointed_to_by(p)`。

## 16.2 可以 `free()` 一个由 `new` 分配的指针吗？可以 `delete` 一个由 `malloc()` 分配的指针吗？

*不！*

在一个程序中同时使用 `malloc()` 和 `delete` 或者同时使用 `new` 和 `free()` 是合情合理合法的。但是，对由 `new` 分配的指针调用 `free()`，或对由 `malloc()` 分配的指针调用 `delete`，是无理的、非法的、卑劣的。

当心！我偶尔收到一些人的 e-mail，他们告诉我在他们的机器 X 上和编译器 Y 上工作正常。但这并不能使得它成为正确的！有时他们说：“但我只是用一下字符数组而已”。即便虽然如此，也不要在同一个指针上混合`malloc()` 和 `delete`，或在同一个指针上混合`new` 和 `free()`。如果通过`p = new char[n]`分配，则必须使用`delete[] p`；不可以使用`free(p)`。如果通过分配`p = malloc(n)`，则必须使用`free(p)`；不可以使用`delete[] p` 或 `delete p`！将它们混合，如果将代码放到新的机器上，新的编译器上，或只是同样编译器的新版本上，都可能导致运行时灾难性的失败。

记住这个警告。

## 16.3 为什么要用 `new` 取代原来的值得信赖的 `malloc()`？

构造函数／析构函数，类型安全，可覆盖性（Overridability）。

*   构造函数／析构函数：与 `malloc(sizeof(Fred))`不一样，`new Fred()` 调用 `Fred` 的构造函数。同样，`delete p` 调用 `*p` 的析构函数。
*   类型安全：`malloc()` 返回一个没有类型安全的 `void*` 。`new Fred()` 返回一个正确类型（一个 `Fred*`）的指针。
*   可覆盖性：`new` 是一个可被类重写／覆盖的算符（`operator`），而 `malloc()` 在类上没有可覆盖性。

## 16.4 可以在一个由 `new` 分配的指针上使用 `realloc()` 吗？

不可！

`realloc()` 拷贝时，使用的是位拷贝（*bitwise* copy ）算符，这会打碎许多 C++ 对象。C++对象应该被允许拷贝它们自己。它们使用自己的拷贝构造函数或者赋值算符。

除此之外，`new` 使用的堆可能和 `malloc()` 和 `realloc()` 使用的堆不同！

## 16.5 需要在 `p = new Fred()`之后检查`NULL`吗？

不！（但如果你只有旧的编译器，你可能不得不强制 new 算符在内存溢出时抛出一个异常。）

总是在每一个`new` 调用之后写显式的 `NULL` 测试实在是非常痛苦的.如下的代码是非常单调乏味的：

```cpp
 Fred* p = new Fred();
 if (p == NULL)
   throw std::bad_alloc(); 
```

如果你的编译器不支持（或如果你拒绝使用）异常， 你的代码可能会更单调乏味：

```cpp
 Fred* p = new Fred();
 if (p == NULL) {
   std::cerr << "Couldn't allocate memory for a Fred" << endl;
   abort();
 } 
```

振作一下。在 C++中，如果运行时系统无法为`p = new Fred()`分配 `sizeof(Fred)` 字节的内存，会抛出一个 `std::bad_alloc` 异常。与 `malloc()`不同，`new` *永远不会*返回 `NULL`！

因此你只要简单地写：

```cpp
Fred* p = new Fred();   // 不需要检查 `p` 是否为 `NULL` 
```

然而，如果你的编译器很古老，它可能还不支持这个。查阅你的编译器的文档找到“`new`”。如果你只有古老的编译器，就必须强制编译器拥有这种行为。

## 16.6 我如何确信我的（古老的）编译器会自动检查 `new` 是否返回 `NULL` ？

最终你的编译器会支持的。

如果你只有古老的不自动执行`NULL` 测试的编译器的话，你可以安装一个“new handler”函数来强制运行时系统来测试。你的“new handler”函数可以作任何你想做的事情，诸如抛出一个异常， `delete` 一些对象并返回（在`operator new`会试图再分配的情况下），打印一个消息或者从程序中 `abort()` 等等。

这里有一个“new handler”的例子，它打印消息并抛出一个异常。它使用 `std::set_new_handler()` 被安装：

```cpp
 #include <new>       // 得到 std::set_new_handler
 #include <cstdlib>   // 得到 abort()
 #include <iostream>  // 得到 std::cerr

 class alloc_error : public std::exception {
 public:
   alloc_error() : exception() { }
 };

 void myNewHandler()
 {
   // 这是你自己的 handler。它可以做任何你想要做的事情。
   throw alloc_error();
 }

 int main()
 {
   std::set_new_handler(myNewHandler);   // 安装你的 "new handler"
   // ...
 } 
```

在`std::set_new_handler()`被执行后，如果／当内存不足时，`operator new`将调用你的`myNewHandler()`。这意味着`new` 不会返回`NULL`：

```cpp
Fred* p = new Fred();   // 不需要检查 `p` 是否为 `NULL` 
```

注意：如果你的编译器不支持异常处理，作为最后的诉求，你可以将 `throw` ...`;` 这一行改为：

```cpp
std::cerr << "Attempt to allocate memory failed!" << std::endl;
abort(); 
```

注意：如果某些全局的／静态的对象的构造函数使用了`new`，由于它们的构造函数在`main()`开始之前被调用，因此它不会使用`myNewHandler()`函数。不幸的是，没有简便的方法确保`std::set_new_handler()` 在第一次使用 `new` 之前被调用。例如，即使你将`std::set_new_handler()`的调用放在全局对象的构造函数中，你仍然无法知道包含该全局对象的模块（“编译单元”）被首先还是最后还是还是中间某个位置被解释。因此，你仍然无法保证`std::set_new_handler()` 的调用会在任何其他全局对象的构造函数调用之前。

## 16.7 在`delete p`之前需要检查`NULL`吗？

不需要！

C++语言担保，如果`p`等于`NULL`，则`delete p`不作任何事情。由于之后可以得到测试，并且大多数的测试方法论都强制显式测试每个分支点，因此你不应该加上多余的 `if` 测试。

错误的：

```cpp
if (p != NULL)
  delete p; 
```

正确的：

```cpp
delete p; 
```

## 16.8 `delete p` 执行了哪两个步骤？

`delete p` 是一个两步的过程：调用析构函数，然后释放内存。`delete p`产生的代码看上去是这样的（假设是`Fred*`类型的）：

```cpp
 // 原始码：delete p;
 if (p != NULL) {
   p->~Fred();
   operator delete(p);
 } 
```

`p->~Fred()` 语句调用 `p` 指向的`Fred` 对象的析构函数。

`operator delete(p)` 语句调用内存释放原语 `void operator delete(void* p)`。该原语类似`free(void* p)`。（然而注意，它们两个不能互换；举例来说，没有谁担保这两个内存释放原语会使用同一个堆！）。

## 16.9 在 `p = new Fred()` 中，如果`Fred` 构造函数抛出异常，是否会内存“泄漏”？

不会。

如果异常发生在`p = new Fred()`的 `Fred`构造函数中， C++语言确保已分配的 `sizeof(Fred)`字节的内存会自动从堆中回收。

这里有两个细节：`new Fred()`是一个两步的过程：

1.  `sizeof(Fred)` 字节的内存使用`void* operator new(size_t nbytes)`原语被分配。该原语类似于`malloc(size_t nbytes)`。（然而注意，他们两个不能互换；举例来说，没有谁担保这两个内存分配原语会使用同一个堆！）。
2.  它通过调用`Fred`构造函数在内存中建立对象。第一步返回的指针被作为 `this` 参数传递给构造函数。这一步被包裹在一个块中以处理这步中抛出异常的情况。

因此实际产生的代码可能是象这样的：

```cpp
 // 原始代码：Fred* p = new Fred();
 Fred* p = (Fred*) operator new(sizeof(Fred));
 try {
   new(p) Fred();       // Placement new
 } catch (...) {
   operator delete(p);  // 释放内存 _
   throw;               // 重新抛出异常
 } 
```

标记为“Placement `new`”的这句语句调用了 `Fred` 构造函数。指针 `p` 成了构占函数 `Fred::Fred()`内部的`this`指针。

## 16.10 如何分配／释放一个对象的数组？

使用 `p = new T[n]` 和 `delete[] p`:

```cpp
 Fred* p = new Fred[100];
 // ...
 delete[] p; 
```

任何时候你通过`new` 来分配一个对象的数组（通常在表达式中有`[`*n*`]`），则在 `delete` 语句中必须使用`[]`。该语法是必须的，因为没有什么语法可以区分指向一个对象的指针和指向一个对象数组的指针（从 C 派生出的某些东西）。

## 16.11 如果 `delete` 一个由`new T[n]`分配的数组，漏了`[]`会如何？

所有生命毁灭性地终止。

正确地连接`new T[n]`和`delete[] p`是程序员的——不是编译器的——责任。如果你弄错了，编译器会在编译时或运行时给出错误消息。堆（Heap）被破坏是可能的结果，或者更糟糕，你的程序可能会死亡。

## 16.12 当`delete`一个内建类型 (`char`, `int`, 等)的数组时，能去掉 `[]` 吗？

*不行！*

有时程序员会认为在`delete[] p` 中存在`[]` 仅仅是为了编译器为数组中的每个元素调用适当的析构函数。由于这个原因，他们认为一些内建类型的数组，如 `char`或`int`可以不需要`[]`。举例来说，他们认为以下是合法的代码：

```cpp
 void userCode(int n)
 {
   char* p = new char[n];
   // ...
   delete p;     // <— 错！应该是 delete[] p ！
 } 
```

但以上代码是错误的，并且会导致一个运行时的灾难。更详细地来说，`delete p`调用的是`operator delete(void*)`，而`delete[] p`调用的是`operator delete[](void*)`。虽然后者的默认行为是调用前者，但将后者用不同的行为取代是被允许的（这种情况下通常也会将相应的`operator new[](size_t)`中的 `new` 取代）。如果被取代的`delete[]` 代码与`delete` 代码不兼容，并且调用错误的那个（例如，你写了`delete p`而不是`delete[] p`），在运行时可能完蛋。

## 16.13 `p = new Fred[n]`之后，编译器在`delete[] p`的时候如何知道有个对象被析构？

精简的回答：魔法。

详细的回答：运行时系统将对象的数量 `n` 保存在某个通过指针 `p` 可以获取的地方。有两种普遍的技术来实现。这些技术都在商业编译器中使用，各有权衡，都不完美。这些技术是：

*   超额分配数组并将 `n` 放在第一个`Fred`对象的左边。
*   使用关联数组， `p` 作为键， `n` 作为值。

## 16.14 成员函数调用`delete this`合法吗？

只要你小心，一个对象请求自杀(`delete` `this`).是可以的。

以下是我对“小心”的定义：

1.  你必须 100%的确定，`this`对象是用 `new`分配的（不是用`new]`，也不是用[定位放置 `new`，也不是一个栈上的局部对象，也不是全局的，也不是另一个对象的成员，而是明白的普通的`new`）。
2.  你必须 100%的确定，该成员函数是`this`对象最后调用的的成员函数。
3.  你必须 100%的确定，剩下的成员函数（`delete` `this`之后的）不接触到 `this`对象任何一块（包括调用任何其他成员函数或访问任何数据成员）。
4.  你必须 100%的确定，在`delete` `this`之后不再去访问`this`指针。换句话说，你不能去检查它，将它和其他指针比较，和 `NULL`比较，打印它，转换它，对它做任何事。

自然，对于这种情况还要习惯性地告诫：当你的指针是一个指向基类类型的指针，而没有虚析构函数时（也不可以 `delete` `this`）。

## 16.15 如何用`new`分配多维数组？

有许多方法，取决于你想要让数组有多大的灵活性。一个极端是，如果你在编译时就知道数组的所有的维数，则可以静态地（就如同在 C 中）分配多维数组：

```cpp
 class Fred { /*...*/ };
 void someFunction(Fred& fred);

 void manipulateArray()
 {
   const unsigned nrows = 10;  // 行数是编译期常量
   const unsigned ncols = 20;  // 列数是编译期常量
   Fred matrix[nrows][ncols];

   for (unsigned i = 0; i < nrows; ++i) {
     for (unsigned j = 0; j < ncols; ++j) {
       // 访问(i,j)元素的方法：
       someFunction( matrix[i][j] );

       // 可以安全地“返回”，不需要特别的 delete 代码：
       if (today == "Tuesday" && moon.isFull())
         return;     // 月圆的星期二赶紧退出
     }
   }

   // 在函数末尾也没有显式的 delete 代码
 } 
```

更一般的，矩阵的大小只有到运行时才知道，但确定它是一个矩形。这种情况下，你需要使用堆（“自由存储”）（heap，freestore），但至少你可以把所有元素非胚在自由存储块中。

```cpp
 void manipulateArray(unsigned nrows, unsigned ncols)
 {
   Fred* matrix = new Fred[nrows * ncols];

   // 由于我们上面使用了简单的指针，因此我们需要非常
   // 小心避免漏过 delete 代码。
   // 这就是为什么要捕获所有异常：
   try {

     // 访问(i,j) 元素的方法：
     for (unsigned i = 0; i < nrows; ++i) {
       for (unsigned j = 0; j < ncols; ++j) {
         someFunction( matrix[i*ncols + j] );
       }
     }

     // 如果你想在月圆的星期二早点退出，
     // 就要确保在返回的所有途径上做 delete ：
     if (today == "Tuesday" && moon.isFull()) {
       delete[] matrix;
       return;
     }

     // ...

   }
   catch (...) {
     // 确保在异常抛出后 delete ：
     delete[] matrix;
     throw;    // 重新抛出当前异常
   }

   // 确保在函数末尾也做了 delete ：
   delete[] matrix;
 } 
```

最后是另一个极端，你可能甚至不确定矩阵是矩形的。例如，如果每行可以有不同的长度，你就需要为个别地分配每一行。在如下的函数中，`ncols[i]` 是第 `i` 行的列数，`i` 的可变范围是 `0` 到 `nrows-1`。

```cpp
 void manipulateArray(unsigned nrows, unsigned ncols[])
 {
   typedef Fred* FredPtr;

   // 如果后面抛出异常，不要成为漏洞：
   FredPtr* matrix = new FredPtr[nrows];

   // 以防万一稍后会有异常，将每个元素设置为 NULL：
   // (见 try 块顶端的注释。)
   for (unsigned i = 0; i < nrows; ++i)
     matrix[i] = NULL;

   // 由于我们上面使用了简单的指针，我们需要
   // 非常小心地避免漏过 delete 代码。
   // 这就是为什么我们要捕获所有的异常：
   try {

     // 接着我们组装数组。如果其中之一抛出异常，所有的
     // 已分配的元素都会被释放 (见如下的 catch )。
     for (unsigned i = 0; i < nrows; ++i)
       matrix[i] = new Fred[ ncols[i] ];

     // 访问(i,j) 元素的方法：
     for (unsigned i = 0; i < nrows; ++i) {
       for (unsigned j = 0; j < ncols[i]; ++j) {
         someFunction( matrix[i][j] );
       }
     }

     // 如果你想在月圆的星期二早些退出，
     // 确保在返回的所有途径上做 delete：
     if (today == "Tuesday" && moon.isFull()) {
       for (unsigned i = nrows; i > 0; --i)
         delete[] matrix[i-1];
       delete[] matrix;
       return;
     }

     // ...

   }
   catch (...) {
     // 确保当有异常抛出时做 delete ：
     // 注意 matrix[...] 中的一些指针可能是
     // NULL, 但由于 delete NULL 是合法的，所以没问题。
     for (unsigned i = nrows; i > 0; --i)
       delete[] matrix[i-1];
     delete[] matrix;
     throw;    // 重新抛出当前异常
   }

   // 确保在函数末尾也做 delete ：
   // 注意释放与分配反向：
   for (unsigned i = nrows; i > 0; --i)
     delete[] matrix[i-1];
   delete[] matrix;
 } 
```

注意释放过程中 `matrix[i-1]`的使用。这样可以防止无符号值 `i` 的步进为小于 0 的回绕。

最后，注意指针和数组是会带来麻烦的](containers-and-templates.html#[31.1])。通常，最好将你的指针封装在一个有着安全的和简单的接口的类中。下一个 FAQ 告诉你如何这样做。

## 16.16 但前一个 FAQ 的代码太技巧容易出错！有更简单的方法吗？

有。

前一个 FAQ 之所以太过技巧而容易出错是因为它使用了指针，我们知道指针和数组会带来麻烦。解决办法是将指针封装到一个有着安全的和简单的接口的类中。例如，我们可以定义一个 `Matrix` 类来处理矩形的矩阵，用户代码将比[前一个 FAQ 中的矩形矩阵的代码简单得多：

```cpp
 // Matrix 类的代码在下面显示...
 void someFunction(Fred& fred);

 void manipulateArray(unsigned nrows, unsigned ncols)
 {
   Matrix matrix(nrows, ncols);   // 构造一个 matrix

   for (unsigned i = 0; i < nrows; ++i) {
     for (unsigned j = 0; j < ncols; ++j) {
       _// 访问(i,j) 元素的方法：_
       someFunction( matrix(i,j) );

       _// 你可以不用写任何的 delete 代码安全地“返回”：
       if (today == "Tuesday" && moon.isFull())
         return;     // 月圆的星期二早些退出
     }
   }

   // 在函数末尾也没有显式的 delete 代码
 } 
```

需要注意的主要是整理后的代码的短小。例如，再如上的代码中没有任何 `delete` 语句，也不会有内存泄漏，这个假设仅仅是基于析构函数正确地完成它的工作。

以下就是使得以上成为可能的`Matrix`的代码：

```cpp
 class Matrix {
 public:
   Matrix(unsigned nrows, unsigned ncols);
   // 如果任何一个尺寸为 0，则抛出 BadSize 对象的异常：
   class BadSize { };

   // 基于大三法则（译注：即三者须同时存在）：
  ~Matrix();
   Matrix(const Matrix& m);
   Matrix& operator= (const Matrix& m);

   // 取得 (i,j) 元素的访问方法：
   Fred&       operator() (unsigned i, unsigned j);
   const Fred& operator() (unsigned i, unsigned j) const;
   // 如果 i 或 j 太大，抛出 BoundsViolation 对象
   class BoundsViolation { };

 private:
   Fred* data_;
   unsigned nrows_, ncols_;
 };

 inline Fred& Matrix::operator() (unsigned row, unsigned col)
 {
   if (row >= nrows_ || col >= ncols_) throw BoundsViolation();
   return data_[row*ncols_ + col];
 }

 inline const Fred& Matrix::operator() (unsigned row, unsigned col) const
 {
   if (row >= nrows_ || col >= ncols_) throw BoundsViolation();
   return data_[row*ncols_ + col];
 }

 Matrix::Matrix(unsigned nrows, unsigned ncols)
   : data_  (new Fred[nrows * ncols]),
     nrows_ (nrows),
     ncols_ (ncols)
 {
   if (nrows == 0 || ncols == 0)
     throw BadSize();
 }

 Matrix::~Matrix()
 {
   delete[] data_;
 } 
```

注意以上的`Matrix`类完成两件事：将技巧性的内存管理代码从客户代码（例如，`main()`）移到类中，并且总体上减少了编程。这第二点很重要。例如，假设 `Matrix`有略微的可重用性，将复杂性从`Matrix`的用户们［复数］处移到了`Matrix`自身［单数］就等于将复杂性从多的方面移到少的方面。任何看过星际旅行 2 的人都知道多数的利益高于少数或者个体的利益。

## 16.17 但上面的`Matrix`类是针对`Fred`的！有办法使它通用吗？

有；那就是使用模板：

以下就是如何能用模板：

```cpp
 #include "Fred.hpp"     // 得到 Fred 类的定义
 // Matrix<T> 的代码在后面显示...
 void someFunction(Fred& fred);

 void manipulateArray(unsigned nrows, unsigned ncols)
 {
   Matrix<Fred> matrix(nrows, ncols);   // 构造一个称为 matrix 的 Matrix<Fred> 

   for (unsigned i = 0; i < nrows; ++i) {
     for (unsigned j = 0; j < ncols; ++j) {
       // 访问 (i,j) 元素的方法：
       someFunction( matrix(i,j) );

       // 你可以不用任何的 delete 的代码安全地“返回”：
       if (today == "Tuesday" && moon.isFull())
         return;     // 月圆的星期二早些退出
     }
   }

   // 函数末尾也没有显式的 delete 代码

 } 
```

现在很容易为非 `Fred` 的类使用 `Matrix<T>`。例如，以下为`std::string` 使用一个 `Matrix` （`std::string` 是标准字符串类）：

```cpp
 #include <string>

 void someFunction(std::string& s);

 void manipulateArray(unsigned nrows, unsigned ncols)
 {
   Matrix<std::string> matrix(nrows, ncols);   // 构造一个 Matrix<std::string>

   for (unsigned i = 0; i < nrows; ++i) {
     for (unsigned j = 0; j < ncols; ++j) {
       // 访问 (i,j) 元素的方法：
       someFunction( matrix(i,j) );

       // 你可以不用任何的 delete 的代码安全地“返回”：
       if (today == "Tuesday" && moon.isFull())
         return;     // 月圆的星期二早些退出
     }
   }

   // 函数末尾也没有显式的 delete 代码
 } 
```

因此，你可以从模板得到类的完整家族。例如， `Matrix<Fred>`, `Matrix<std::string>`, `Matrix< Matrix<std::string> >`等等。

以下是实现该模板的一种方法：

```cpp
 template<class T>  // 详见模板一节
 class Matrix {
 public:
   Matrix(unsigned nrows, unsigned ncols);
   // 如果任何一个尺寸为 0，则抛出 BadSize 对象
   class BadSize { };

   // 基于大三法则（译注：即三者须同时存在）：
  ~Matrix();
   Matrix(const Matrix<T>& m);
   Matrix<T>& operator= (const Matrix<T>& m);

   // 获取 (i,j) 元素的访问方法：
   T&       operator() (unsigned i, unsigned j);
   const T& operator() (unsigned i, unsigned j) const;
   // 如果 i 或 j 太大，则抛出 BoundsViolation 对象
   class BoundsViolation { };

 private:
   T* data_;
   unsigned nrows_, ncols_;
 };

 template<class T>
 inline T& Matrix<T>::operator() (unsigned row, unsigned col)
 {
   if (row >= nrows_ || col >= ncols_) throw BoundsViolation();
   return data_[row*ncols_ + col];
 }

 template<class T>
 inline const T& Matrix<T>::operator() (unsigned row, unsigned col) const
 {
   if (row >= nrows_ || col >= ncols_) throw BoundsViolation();
   return data_[row*ncols_ + col];
 }

 template<class T>
 inline Matrix<T>::Matrix(unsigned nrows, unsigned ncols)
   : data_  (new T[nrows * ncols])
   , nrows_ (nrows)
   , ncols_ (ncols)
 {
   if (nrows == 0 || ncols == 0)
     throw BadSize();
 }

 template<class T>
 inline Matrix<T>::~Matrix()
 {
   delete[] data_;
 } 
```

## 16.18 还有其他方法建立 `Matrix` 模板吗？

用标准的`vector` 模板，制作一个向量的向量。

以下代码使用了一个`vector<vector<T> >`（注意两个 `>` 符号之间的空格）。

```cpp
 #include <vector>

 template<class T>  // 详见模板一节
 class Matrix {
 public:
   Matrix(unsigned nrows, unsigned ncols);
   // 如果任何的尺寸为 0，抛出 BadSize 对象
   class BadSize { };

   // 不需要大三法则！
   // 得到 (i,j) 元素的访问方法：
   T&       operator() (unsigned i, unsigned j);
   const T& operator() (unsigned i, unsigned j) const;
   // 如果 i 或 j 太大，则抛出 BoundsViolation 对象
   class BoundsViolation { };

 private:
   vector<vector<T> > data_;
 };

 template<class T>
 inline T& Matrix<T>::operator() (unsigned row, unsigned col)
 {
   if (row >= nrows_ || col >= ncols_) throw BoundsViolation();
   return data_[row][col];
 }

 template<class T>
 inline const T& Matrix<T>::operator() (unsigned row, unsigned col) const
 {
   if (row >= nrows_ || col >= ncols_) throw BoundsViolation();
   return data_[row][col];
 }

 template<class T>
 Matrix<T>::Matrix(unsigned nrows, unsigned ncols)
   : data_ (nrows)
 {
   if (nrows == 0 || ncols == 0)
     throw BadSize();
   for (unsigned i = 0; i < nrows; ++i)
     data_[i].resize(ncols);
 } 
```

## 16.19 C++ 有能够在运行期指定长度的数组吗？

有，是基于标准库有一个 `std::vector` 模板可以提供这种行为的认识。

没有，是基于内建数组类型需要在编译期指定其长度的认识。

有，是基于即使对于内建数组类型也可以在运行期指定第一维索引边界的认识。例如，看一下前一个 FAQ，如果你只需要数组的第一维的维数具有灵活性，你可以申请一个新的数组的数组，而不是一个指向多个数组的指针数组：

```cpp
 const unsigned ncols = 100;           // ncols = 数组的列数

 class Fred { /*...*/ };

 void manipulateArray(unsigned nrows)  // nrows = 数组的行数
 {
   Fred (*matrix)[ncols] = new Fred[nrows][ncols];
   // ...
   delete[] matrix;
 } 
```

如果你所需要的不是在运行期改变数组的第一维维数，则不能这么做。

但非万不得已，不要用数组。因为数组是会带来麻烦的。如果可以的话，使用某些类的对象。万不得已才用数组。

## 16.20 如何使类的对象总是通过 `new` 来创建而不是局部的或者全局的／静态的对象？

使用命名的构造函数用法。

就如命名的构造函数用法的通常做法，所有构造函数是`private:` 或`protected:`，且有一个或多个`public` `static` `create()`方法（因此称为“命名的构造函数，named constructors”），每个构造函数对应一个。此时， `create()` 方法通过 `new` 来分配对象。由于构造函数本身都不是`public`，因此没有其他方法来创建该类的对象。

```cpp
 class Fred {
 public:
   // create() 方法就是 "命名的构造函数，named constructors":
   static Fred* create()                 { return new Fred();     }
   static Fred* create(int i)            { return new Fred(i);    }
   static Fred* create(const Fred& fred) { return new Fred(fred); }
   // ...

 private:
   // 构造函数本身是 private 或 protected:
   Fred();
   Fred(int i);
   Fred(const Fred& fred);
   // ...
 }; 
```

这样，创建 `Fred` 对象的唯一方法就是通过 `Fred::create()`：

```cpp
 int main()
 {
   Fred* p = Fred::create(5);
   // ...
   delete p;
 } 
```

如果你希望 `Fred`有派生类，则须确认构造函数在 `protected:` 节中。

注意，如果你想允许`Fred`类的对象成为`Wilma`类的成员，可以把`Wilma` 作为 `Fred` 的友元。当然，这样会软化最初的目标，也就是强迫 `Fred` 对象总是通过 `new` 来分配。

## 16.21 如何进行简单的引用计数？

如果你所需要的只是分发指向同一个对象的多个指针，并且当最后一个指针消失的时候能自动释放该对象的能力的话，你可以使用类似如下的“只能指针（smart pointer）”类：

```cpp
 // Fred.h

 class FredPtr;

 class Fred {
 public:
   Fred() : count_(0) /*...*/ { }  // 所有的构造函数都要设置 count to 0 !
   // ...
 private:
   friend FredPtr;     // 友元类
   unsigned count_;
   // count_ 必须被所有构造函数初始化
   // count_ 就是指向 this 的对 FredPtr 象数目
 };

 class FredPtr {
 public:
   Fred* operator-> () { return p_; }
   Fred& operator* ()  { return *p_; }
   FredPtr(Fred* p)    : p_(p) { ++p_->count_; }  // p 不能为 NULL
  ~FredPtr()           { if (--p_->count_ == 0) delete p_; }
   FredPtr(const FredPtr& p) : p_(p.p_) { ++p_->count_; }
   FredPtr& operator= (const FredPtr& p)
         { // 不要改变这些语句的顺序！
           // (如此的顺序适当的处理了自赋值)
           ++p.p_->count_;
           if (--p_->count_ == 0) delete p_;
           p_ = p.p_;
           return *this;
         }
 private:
   Fred* p_;    // p_ 永远不为 NULL
 }; 
```

自然，你可以使用嵌套类，将`FredPtr`改名为`Fred::Ptr`。

注意，在构造函数，拷贝构造函数，赋值算符和析构函数中增加一点检查，就可以软化上面的“不远不为 NULL”的规则。如果你这样做的话，可能倒不如在“`*`”和“`->`”算符中放入一个`p_ != NULL`检查（至少是一个 `assert()`）。我不推荐`operator Fred*()` ，因为它可能让人们意外地取得`Fred*`。

`FredPtr`的隐含约束之一是它可能指向通过 `new`分配的`Fred`对象。如果要真正的安全，可以使所有的`Fred`构造函数成为`private`，为每个构造函数加一个用`new`来分配`Fred` 对象且返回一个`FredPtr` （不是`Fred*`）的`public` (`static`) `create()` 方法来加强这个约束。这种办法是创建`Fred`对象而得到一个`FredPtr`的唯一办法（“`Fred* p = new Fred()`”会被“`FredPtr p = Fred::create()`”取代）。这样就没人会意外破坏引用计数的机制了。

例如，如果`Fred`有一个`Fred::Fred()` 和一个`Fred::Fred(int i, int j)`，`class` `Fred` 会变成：

```cpp
 class Fred {
 public:
   static FredPtr create();              // 定义如下的 class FredPtr {...}
   static FredPtr create(int i, int j);  // 定义如下的 class FredPtr {...}
   // ...
 private:
   Fred();
   Fred(int i, int j);
   // ...
 };

 class FredPtr { /* ... */ };

 inline FredPtr Fred::create()             { return new Fred(); }
 inline FredPtr Fred::create(int i, int j) { return new Fred(i,j); } 
```

最终结果是你现在有了一种办法来使用简单的引用计数为给出的对象提供“指针语义（pointer semantics）”。`Fred`类的用户明确地使用`FredPtr` 对象，它或多或少的类似`Fred*`指针。这样做的好处是用户可以建立多个`FredPtr`“智能指针”对象的拷贝，当最后一个`FredPtr`对象消失时，它所指向的 `Fred` 对象会被自动释放。

如果你希望给用户以“引用语义”而不是“指针语义”的话，可以使用引用计数提供“写时拷贝（copy on write）”。

## 16.22 如何用写时拷贝（copy-on-write）语义提供引用计数？

引用计数可以由指针语义或引用语义完成。前一个 FAQ 显示了如何使用指针语义进行引用计数。本 FAQ 将显示如何使用引用语义进行引用计数。

基本思想是允许用户认为他们在复制`Fred`对象，但实际上真正的实现并不进行复制，直到一些用户试图修改隐含的`Fred` 对象才进行真正的复制。

`Fred::Data`类装载了`Fred` 类所有的数据。 `Fred::Data`也有一个额外的成员`count_`，来管理引用计数。`Fred` 类最后成了一个指向`Fred::Data`的“智能指针”（内部的）。

```cpp
 class Fred {
 public:

   Fred();                               // 默认构造函数
   Fred(int i, int j);                   // 普通的构在函数

   Fred(const Fred& f);
   Fred& operator= (const Fred& f);
  ~Fred();

   void sampleInspectorMethod() const;   // this 对象不会变
   void sampleMutatorMethod();           // 会改变 this o 对象
   // ...

 private:

   class Data {
   public:
     Data();
     Data(int i, int j);
     Data(const Data& d);

     // 由于只有 Fred 能访问 Fred::Data 对象，
     // 只要你愿意，你可以使得 Fred::Data 的数据为 public，
     // 但如果那样使你不爽，就把数据作为 private
     // 还要用 friend Fred;使 Fred 成为友元类
     // ...

     unsigned count_;
     // count_ 是指向的 this 的 Fred 对象的数目
     // count_ m 必须被所有的构造函数初始化为 1
     // (从 1 开始是因为它被创建它的 Fred 对象所指)
   };

   Data* data_;
 };

 Fred::Data::Data()              : count_(1) /*初始化其他数据*/ { }
 Fred::Data::Data(int i, int j)  : count_(1) /*初始化其他数据*/ { }
 Fred::Data::Data(const Data& d) : count_(1) /*初始化其他数据*/ { }

 Fred::Fred()             : data_(new Data()) { }
 Fred::Fred(int i, int j) : data_(new Data(i, j)) { }

 Fred::Fred(const Fred& f)
   : data_(f.data_)
 {
   ++ data_->count_;
 }

 Fred& Fred::operator= (const Fred& f)
 {
   // 不要更该这些语句的顺序！
   // (如此的顺序适当地处理了自赋值)
   ++ f.data_->count_;
   if (--data_->count_ == 0) delete data_;
   data_ = f.data_;
   return *this;
 }

 Fred::~Fred()
 {
   if (--data_->count_ == 0) delete data_;
 }

 void Fred::sampleInspectorMethod() const
 {
   // 该方法承诺 (“const”) 不改变 *data_ 中的任何东西
   // 除此以外，任何数据访问将简单地使用“data_->...”
 }

 void Fred::sampleMutatorMethod()
 {
   // 该方法可能需要改变 *data_ 中的数据
   // 因此首先检查 this 是否唯一的指向 *data_
   if (data_->count_ > 1) {
     Data* d = new Data(*data_);    // 调用 Fred::Data 的拷贝构造函数
     -- data_->count_;
     data_ = d;
   }
   assert(data_->count_ == 1);

   // 现在该方法如常进行“data_->...”的访问
 } 
```

如果非常经常地调用 `Fred` 的默认构造函数，你可以为所有通过`Fred::Fred()`构造的`Fred` 共享一个公共的`Fred::Data` 对象来消除那些 `new`调用。为避免静态初始化顺序问题，该共享的 `Fred::Data` 对象在一个函数内“首次使用”时才创建。如下就是对以上的代码做的改变（注意，该共享的`Fred::Data`对象的析构函数永远不会被调用；如果这成问题的话，要么解决静态初始化顺序的问题，要么索性返回到如上描述的方法）：

```cpp
 class Fred {
 public:
   // ...
 private:
   // ...
   static Data* defaultData();
 };

 Fred::Fred()
 : data_(defaultData())
 {
   ++ data_->count_;
 }

 Fred::Data* Fred::defaultData()
 {
   static Data* p = NULL;
   if (p == NULL) {
     p = new Data();
     ++ p->count_;    // 确保它不会成为 0
   }
   return p;
 } 
```

注意：如果 `Fred` 通常作为基类的话，也可以为类层次提供引用计数。

## 16.23 如何为派生类提供写时拷贝（copy-on-write）语义的引用计数？

前一个 FAQ 给出了引用语义的引用计数策略，但迄今为止都针对单个类而不是分层次的类。本 FAQ 扩展之前的技术以允许为类层次提供引用计数。基本不同之处在于现在`Fred::Data`是类层次的根，着可能使得它有一些虚函数。注意 `Fred` 类本身仍然没有任何的虚函数。

虚构造函数用法用来建立 `Fred::Data` 对象的拷贝。要选择创建哪个派生类，如下的示例代码使用了命名构造函数用法，但还有其它技术（构造函数中加一个`switch`语句等）。示例代码假设了两个派生类：`Der1`和`Der2`。派生类的方法并不查觉引用计数。

```cpp
 class Fred {
 public:

   static Fred create1(const std::string& s, int i);
   static Fred create2(float x, float y);

   Fred(const Fred& f);
   Fred& operator= (const Fred& f);
  ~Fred();

   void sampleInspectorMethod() const;   // this 对象不会被改变
   void sampleMutatorMethod();           // 会改变 this 对象
   // ...

 private:

   class Data {
   public:
     Data() : count_(1) { }
     Data(const Data& d) : count_(1) { }              // 不要拷贝 'count_' 成员！
     Data& operator= (const Data&) { return *this; }  // 不要拷贝 'count_' 成员！
     virtual ~Data() { assert(count_ == 0); }         // 虚析构函数
     virtual Data* clone() const = 0;                 // 虚构造函数
     virtual void sampleInspectorMethod() const = 0;  // 纯虚函数
     virtual void sampleMutatorMethod() = 0;
   private:
     unsigned count_;   // count_ 不需要是 protected 的
     friend Fred;       // 允许 Fred 访问 count_
   };

   class Der1 : public Data {
   public:
     Der1(const std::string& s, int i);
     virtual void sampleInspectorMethod() const;
     virtual void sampleMutatorMethod();
     virtual Data* clone() const;
     // ...
   };

   class Der2 : public Data {
   public:
     Der2(float x, float y);
     virtual void sampleInspectorMethod() const;
     virtual void sampleMutatorMethod();
     virtual Data* clone() const;
     // ...
   };

   Fred(Data* data);
   // 创建一个拥有 *data 的 Fred 智能引用
   // 它是 private 的以迫使用户使用 createXXX() 方法
   // 要求：data 必能为 NULL

   Data* data_;   // Invariant: data_ is never NULL
 };

 Fred::Fred(Data* data) : data_(data)  { assert(data != NULL); }

 Fred Fred::create1(const std::string& s, int i) { return Fred(new Der1(s, i)); }
 Fred Fred::create2(float x, float y)            { return Fred(new Der2(x, y)); }

 Fred::Data* Fred::Der1::clone() const { return new Der1(*this); }
 Fred::Data* Fred::Der2::clone() const { return new Der2(*this); }

 Fred::Fred(const Fred& f)
   : data_(f.data_)
 {
   ++ data_->count_;
 }

 Fred& Fred::operator= (const Fred& f)
 {
   // 不要更该这些语句的顺序！
   // (如此的顺序适当地处理了自赋值)
   ++ f.data_->count_;
   if (--data_->count_ == 0) delete data_;
   data_ = f.data_;
   return *this;
 }

 Fred::~Fred()
 {
   if (--data_->count_ == 0) delete data_;
 }

 void Fred::sampleInspectorMethod() const
 {
   // 该方法承诺 ("const") 不改变*data_ 中的任何东西
   // 因此我们只要“直接把方法传递”给 *data_：
   data_->sampleInspectorMethod();
 }

 void Fred::sampleMutatorMethod()
 {
   // 该方法可能需要更该 *data_ 中的数据
   // 因此首先检查 this 是否唯一的指向*data_
   if (data_->count_ > 1) {
     Data* d = data_->clone();   // 虚构造函数用法
     -- data_->count_;
     data_ = d;
   }
   assert(data_->count_ == 1);

   // 现在“直接把方法传递给” *data_：
   data_->sampleInspectorMethod();
 } 
```

自然，`Fred::Der1` 和`Fred::Der2` 的构造函数和`sampleXXX`方法将需要被以某种途径适当的实现。

## 16.24 你能绝对地防止别人破坏引用计数机制吗？如果能的话，你会这么做吗？

不能，（通常）不会。

有两个基本的办法破坏引用计数机制：

1.  如果某人获得了`Fred*` （而不是别强制使用的`FredPtr`），该策略就会被破坏。如果`FredPtr`类有返回一个 `Fred&`的`operator*()`的话，就可能得到`Fred*`：`FredPtr p = Fred::create(); Fred* p2 = &*p;`。是的，那是奇异的、不被预期的，但它可能发生。该漏洞有两个方法弥补：重载`Fred::operator&()`使它返回一个`FredPtr`，或改变`FredPtr::operator*()`的返回类型，使它返回一个`FredRef`（`FredRef`是一个模拟引用的类；它需要拥有`Fred`所拥有的所有方法，并且需要将这些方法的调用转送给隐含的`Fred`对象；第二种选择可能成为性能瓶颈，这取决于编译器在内联方法中的表现）。另一个方法是消除 `FredPtr::operator*()` ——相应的会失去取得和使用 `Fred&` 的能力。但即使你这样做了，某些人仍然可以通过显式的调用 `operator->()`: `FredPtr p = Fred::create(); Fred* p2 = p.operator->();`来取得一个`Fred*` 。
2.  如果某人有一个泄漏的和／或悬空的`FredPtr`指针的话，该策略会被破坏。基本上我们说`Fred`是安全的，但我们无法阻止别人对`FredPtr` 对象做傻事。（并且如果我们可以通过`FredPtrPtr`对象来解决的话，则对于`FredPtrPtr`仍然有相同的问题）。这里的一个漏洞是如果某人使用 `new` 创建了一个`FredPtr` ，然后`FredPtr`就可能有泄漏（这里最糟的情况是有泄漏，但通常还是比悬空指针要好一点点）。该漏洞可以通过将`FredPtr::operator new()` 声明为`private`来弥补，从而防止 `new FredPtr()`。此处另一个漏洞是如果某人创建了一个局部的`FredPtr`对象，则可取得`FredPtr`的地址并传递给`FredPtr*`。如果`FredPtr*`生存期比`FredPtr`更长，就可能成为悬空指针——颤抖的指针。该漏洞可以通过防止取得 `FredPtr`的地址来弥补（重载`FredPtr::operator&()`为`private`），相应的会损失一些功能。但即使你这样做了，他们只要这样做：`FredPtr p; ... FredPtr& q = p;`（或者将`FredPtr&`传递其它什么），仍然可以创建 `FredPtr*`与一样危险的`FredPtr&`。

并且，即使我们弥补了*所有*那些漏洞，C++ 还有奇妙的称为指针转换（pointer cast）的语法。使用一两个指针转换，一个有意的程序员可以创造一个大得足以穿过一辆卡车的漏洞。

此处的教训是：(a) 无论你多么的智者千虑，也不可能防止间谍，(b) 你可以简单的防止错误。

因此我建议：用易建易用的机制来防止错误，不要操心试图去防止间谍。即使你殚精竭力做了，也不会成功，得不偿失。

如果不能使用 C++语言本身来防止间谍，还有其它办法吗？有。我为它亲自用旧式风格的代码检视。由于间谍技巧通常包括一些奇异的语法和／或指针转换的使用和联合（union），你可以使用工具来指出大多数的“是非之地”。

## 16.25 在 C++中能使用垃圾收集吗？

能。

相比于前面所述的“智能指针”技术，垃圾收集技术：

*   更轻便
*   通常更有效 （尤其当平均的对象尺寸较小时或多线程环境中）
*   能处理数据中的“循环（cycles）”（如果数据结构能形成循环，引用计数技术通常会有“泄漏”）
*   有时会泄漏其它对象（由于垃圾收集器必要的保守性，有时会进入一个看上去象是指针的随机位模式的分配单元，尤其是如果分配单元较大时，可能导致该分配单元有泄漏）。
*   与现存的库工作得更好（由于智能指针需要显式使用，可能很难集成到现存的库中）

## 16.26 C++的两种垃圾收集器是什么？

通常，好像有两种风味的 C++垃圾收集器：

1.  *保守的垃圾收集器。*这些垃圾收集器对于栈和 C++对象的分布知之甚少或一无所知，只是寻找看上去象指针的位模式。实践中与 C 以及 C++ 代码共同工作，尤其是平均的对象尺寸较小时，这里有一些例子，按字母顺序：

    *   [Boehm-Demers-Weiser collector](http://www.hpl.hp.com/personal/Hans_Boehm/gc)
    *   [Geodesic Systems collector](http://www.geodesic.com/solutions/greatcircle.html)
2.  *混合的垃圾收集器。*这些垃圾收集器通常适当地扫描栈，但需要程序员提供堆对象的布局信息。这需要程序员方面做更多工作，但结果是提高性能。这里有一些例子，按字母顺序：

    *   Bartlett's mostly copying collector
    *   Attardi and Flagella's CMM (如果谁有 URL，请发给我）。

由于 C++垃圾收集器通常是保守的，如果一个位模式“看上去”象是有可能是指向另外一个未使用块的指针，就会有泄漏。当指向某块的指针实际超出了块（这是非法的，但一些程序员会越过该限制；唉）以及（很少）当一个指针被编译器的优化所隐藏，也会使它困惑。在实践中，这些问题通常不严重，然而倘若收集器有一些关于对象布局的提示的话，可能会改善这些情况。

## 16.27 还有哪里能得到更多的 C++垃圾收集信息？

更多信息，详见[垃圾收集 FAQ](http://www.iecc.com/gclist/GC-faq.html)。