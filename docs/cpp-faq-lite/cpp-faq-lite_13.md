# [15] 通过 <iostream class="calibre21">和 <cstdio class="calibre21">输入／输出</cstdio></iostream>

# [15] 通过 `<iostream>` 和 `<cstdio>`输入／输出

## FAQs in section [15]:

*   [15.1] 为什么应该用 `<iostream>` 而不是传统的 `<cstdio>`?
*   [15.2] 当键入非法字符时，为何我的程序进入死循环？
*   [15.3] 那个古怪的`while (std::cin >> foo)`语法如何工作？
*   [15.4] 为什么我的输入处理会超过文件末尾？
*   [15.5] 为什么我的程序在第一个循环后，会忽略输入请求呢？
*   [15.6] 如何为 `class` `Fred` 提供打印？
*   [15.7] 但我可以总是使用 `printOn()` 方法而不是一个友元函数吗？
*   [15.8] 如何为 `class` `Fred` 供输入？
*   [15.9] 如何为完整继承层次的类提供打印？
*   [15.10] 在 DOS 和／或 OS/2 环境下，如何以二进制模式“重打开” `std::cin` 和 `std::cout` ？
*   [15.11] 为何我不能在如“`..\test.dat`”这样的不同的目录中打开文件？
*   [15.12] 如何将一个值（如，一个数字）转换为 `std::string`?
*   [15.13] 如何将 `std::string` 转换为数值？

## 15.1 为什么应该 `<iostream>` 而不是传统的 `<cstdio>`?

因为`<iostream>`加强了类型安全，减少了错误，提升了性能，可扩展，并且提供继承。

`printf()` 不错，`scanf()` 不管其可能导致错误，也还是有价值的，然而对于 C++ I／O（译注：I／O 即输入／输出） 所能做的来说，它们的功能都是非常有限的。相对于 C （使用 `printf()` 和 `scanf()`）来说，C++ I／O （使用 `<<` 和 `>>`）是：

*   *更好的类型安全：*使用 `<iostream>`，编译器静态地知道被 I／O 的对象的类型。相反，`<cstdio>`使用“`%`”域来动态地指出类型。
*   *更少的错误倾向：*使用 `<iostream>，`没有多余的必须与实际被 I／O 的对象相一致的“`%`”。去除多余的，意味着去除了一类错误。
*   _ 可扩展：_C++ `<iostream>` 机制允许在不破坏现有代码的情况下，新的用户定义类型能够被 I／O。（可以想象一下，每个人同事添加新的不相容的“`%`” 域到`printf()` 和 `scanf()`，是怎样的混乱场面？！）。
*   _ 可继承：_C++ `<iostream>` 机制是建立在真正的类上的，如 `std::ostream` 和 `std::istream`。不象 `<cstdio>`的 `FILE*`，有真正的类，因此可继承。这意味着你可以你可以拥有其他的用户定义的看上去以及其效果都类似流的东西，而它可以做任何你需要的奇怪的和有趣的事情。你将自动的得到无数行的你所不认识的用户写的 I／O 代码，并且，他们不需要认识你写的“extended stream”类。

## 15.2 当键入非法字符时，为何我的程序进入死循环？

举个例子，假设你有如下的代码，从`std::cin`读取一个整数：

```cpp
 #include <iostream>

 int main()
 {
   std::cout << "Enter numbers separated by whitespace (use -1 to quit): ";
   int i = 0;
   while (i != -1) {
     std::cin >> i;        // 不良的形式 — 见如下注释
     std::cout << "You entered " << i << '\n';
   }
 } 
```

该程序没有检查键入的是否是合法字符。尤其是，如果某人键入的不是整数（如“x”），`std::cin`流进入“失败状态”，并且其后所有的输入尝试都不作任何事情而立即返回。换句话说，程序进入了死循环；如果`42`是最后成功读到的数字，程序会反复打印“`You entered 42`”消息。

检查合法输入的一个简单方法是将输入请求从 `while` 循环体中移到 `while` 循环的控制表达式，如：

```cpp
 #include <iostream>

 int main()
 {
   std::cout << "Enter a number, or -1 to quit: ";
   int i = 0;
   while (std::cin >> i) {    // 良好的形式
     if (i == -1) break;
     std::cout << "You entered " << i << '\n';
   }
 } 
```

这样的结果就是当你敲击 end-of-file，或键入一个非整数，或键入 `-1`时， while 循环会退出。

（自然，你也可以不用`break`，而将`while`循环表达式`while (std::cin >> i)`改为`((std::cin >> i) && (i != -1))`，但这不是本 FAQ 的重点，本 FAQ 处理 iostream，而不是一般的结构化编程指南。）

## 15.3 那个古怪的`while (std::cin >> foo)`语法如何工作？

“古怪的 `while (std::cin >> foo)`语法”的例子见前一个 FAQ。

`(std::cin >> foo)`表达式调用了适当的`operator>>`（例如，它调用了左边带有`std::istream`参数以及，如果的类型是`int`，并且右边有一个`int&`的`operator>>`）。`std::istream` `operator>>`函数按惯例地返回左边的参数，在这里，它返回`std::cin`。下一步编译器注意到返回的 `std::istream`处于一个布尔型的上下文中，因此编译器将`std::istream`转换为一个布尔值。

编译器调用一个称为`std::istream::operator void*()`的成员函数来将`std::istream`转换成布尔。它返回一个被转换成布尔的`void*`指针（`NULL`成为`false`，任何其他的指针成为`true`）。因此在这里，编译器产生了`std::cin.operator void*()`的调用，就如同你象`(void*) std::cin`这样显式地强制类型转换。

如果 stream 处于良好状态，那么转换算符`operator void*()`返回非指针，如果处于失败状态，则返回 `NULL`。例如，如果读了太多次（也就是说，已经处于 end-of-file），或实际输入到流的信息不是`foo`的合法类型（如，如果 `foo`是一个`int`，而数据是一个“x”字符），流会进入失败状态并且转换算符会返回`NULL`。

`operator>>`不是简单地返回一个`bool`（或 `void*`）以支出是否成功或失败的原因是为了支持“级联”语法：

```cpp
std::cin >> foo >> bar; 
```

`operator>>`是向左结合的，意味着如上的代码会解释为：

```cpp
(std::cin >> foo) >> bar; 
```

换句话说，如果我们将`operator>>`变为一个普通的函数名称，如`readFrom()`，将变为这样的表达式：

```cpp
readFrom( readFrom(std::cin, foo), bar); 
```

我们总是从最内部开始计算表达式。因为 `operator>>`的左结合性，就成了最左边表达式`std::cin >> foo`。该表达式返回`std::cin` （更合适的，他返回一个它左边参数的引用）给下一个表达式。下一个表达式也返回（一个引用）给`std::cin`，但第二个引用被忽略了，因为它是这个“表达式语句”的最外边的表达式了。

## 15.4 为何我的输入处理会超过文件末尾？

因为只有在试图超过文件末尾后，eof 标记才会被设置。也就是，在从文件读最后一个字节时，还没有设置 eof 标记。例如，假设输入流映射到键盘——在这种情况下，理论上来说，C++库不可能预知到用户所键入的字符是否是最后一个字符。

如，如下的代码对于计数器 `i` 会有“超出 1”的错误：

```cpp
 int i = 0;
 while (! std::cin.eof()) {   // 错误！（不可靠）
   std::cin >> x;
   ++i;
   // Work with x ...
 } 
```

你实际需要的是：

```cpp
 int i = 0;
 while (std::cin >> x) {      // 正确！（可靠）
   ++i;
   // Work with x ...
 } 
```

## 15.5 为什么我的程序在第一个循环后，会忽略输入请求呢？

因为数字的提取器将非数字留在了输入缓冲器之后。

如果你的代码看上去象这样：

```cpp
 char name[1000];
 int age;

 for (;;) {
   std::cout << "Name: ";
   std::cin >> name;
   std::cout << "Age: ";
   std::cin >> age;
 } 
```

而你实际需要的是：

```cpp
 for (;;) {
   std::cout << "Name: ";
   std::cin >> name;
   std::cout << "Age: ";
   std::cin >> age;
   std::cin.ignore(INT_MAX, '\n');
 } 
```

当然，你也许想将`for (;;)`语句变为`while (std::cin)`，但不要搞错，在循环末尾通过`std::cin.ignore(...);`这一行跳过非数字字符。

## 15.6 如何为`class` `Fred`提供打印？

用算符重载提供一个友元的左切换的算符 `operator<<`。

```cpp
 #include <iostream>

 class Fred {
 public:
   friend std::ostream& operator<< (std::ostream& o, const Fred& fred);
   // ...
 private:
   int i_;    // 只是为了说明
 };

 std::ostream& operator<< (std::ostream& o, const Fred& fred)
 {
   return o << fred.i_;
 }

 int main()
 {
   Fred f;
   std::cout << "My Fred object: " << f << "\n";
 } 
```

由于 `Fred` 对象是 `<<` 算符的右边的操作数，我们使用非成员函数（在这里是一个友元）。如果 `Fred` 对象被期望为在`<<`的左边（那就是 `myFred << std::cout`而不是 `std::cout << myFred`），则就会有一个命名为`operator<<`的成员函数。

注意，`operator<<`返回流。这就使得输出算符能够被级联。

## 15.7 但我可以总是使用 `printOn()` 方法而不是一个友元函数吗？

不。

通常人们*总是*愿意使用`printOn()`方法而不是一个友元函数的原因是因为他们错误地相信友元破坏了封装并且／或者友元是不良的。这些信仰是天真的和错误的：适当的使用，友元实际上可以增强封装。

这也不是说`printOn()` 方法没用。例如：为一个完整的继承层次的类提供打印时就是有用的。但如果你看到一个`printOn()` 方法，它通常应该是`protected`的，而不是`public`的。

为完整，这里给出“`printOn()` 方法”。想法是有一个成员函数（通常被称为`printOn()`，来完成实际的打印，然后有一个`operator<<`来调用`rintOn()`方法）。当错误地完成它时，`printOn()`方法是`public` 的，因此`operator<<`不需要成为友元——它成为一个简单的顶级函数，即不是类的友元，也不是类的成员函数。这是一些示例代码：

```cpp
 #include <iostream>

 class Fred {
 public:
   void printOn(std::ostream& o) const;
   // ...
 };

 // operator<< 可以被声明为非友元 [不推荐！]
 std::ostream& operator<< (std::ostream& o, const Fred& fred);

 // 实际打印由内部的 printOn() 方法完成 [不推荐！]
 void Fred::printOn(std::ostream& o) const
 {
   // ...
 }

 // operator<< 调用 printOn() [不推荐！]
 std::ostream& operator<< (std::ostream& o, const Fred& fred)
 {
   fred.printOn(o);
   return o;
 } 
```

人们错误地假定“由于避免了出现一个友元函数”而减少了维护成本。这个假定是错误的，因为：

1.  **在维护成本上，“顶级函数调用成员”方法不会带来任何好处。**我们假设 *N* 行代码来完成实际的打印。在使用友元函数的情况下，那 *N* 行代码将直接访问类的 `private`/`protected` 部分，这意味着某人无论何时改变了类的 `private`/`protected` 部分，那 *N* 行代码将需要被扫描并且可能被修改，这增加了维护成本。然而，使用 `printOn()` 方法并没有改变：我们仍然有 *N* 行代码直接访问类的 `private`/`protected` 部分。因此将代码从友元函数移到成员函数根本就并不减少维护成本。没有减少。在维护成本上没有好处。（如果有的话，`printOn()`方法更差一点，因为你有了一个额外的原先没有的函数，现在有更多行的代码需要被维护）
2.  **“顶级函数调用成员”方法使得类更难被使用，尤其是程序员不是类的设计者时。**这种方法将一个并不期望被调用的`public`方法暴露给程序员。当程序员阅读类的`public`方法时，他们会看见两种方法做同一件事情。文档需要象这样说明：“这个和那个并不完全一样，但不要用这个；而应该用那个”。并且通常的程序员会说：“唔？如果我不应该使用它，为什么它是 `public`的？”事实上`printOn()`方法是`public`的唯一理由是避免将友元授权给 `operator<<`，这个主张对于某些仅仅想使用这个类的程序员来说，是微妙的并且难以理解的。

总之，“顶级函数调用成员”方法有成本，没有收益。因此，通常，不是好主意。

注意：如果 `printOn()`方法是`protected`或`private`的，第二个异议将不成立。有些情况这方法是合理的，如为一个完整的继承层次的类提供打印时。同样要注意，当`printOn()`方法是非`public`的时， `operator<<` 需要成为友元。

## 15.8 如何为 `class` `Fred`提供输入？

使用算符重载](operator-overloading.html)提供一个友元的右切换的算符`operator>>`。除了参数没有一个`const`：“`Fred&`”而不是“`const Fred&`”，其他和[输出算符类似。

```cpp
 #include <iostream>

 class Fred {
 public:
   friend std::istream& operator>> (std::istream& i, Fred& fred);
   // ...
 private:
   int i_;    // 只是为了说明
 };

 std::istream& operator>> (std::istream& i, Fred& fred)
 {
   return i >> fred.i_;
 }

 int main()
 {
   Fred f;
   std::cout << "Enter a Fred object: ";
   std::cin >> f;
   // ...
 } 
```

注意`operator>>`返回流。这就使得输入算符能被级联和／或在循环或语句中使用。

## 15.9 如何为完整继承层次的类提供打印？

提供一个友元调用一个`protected` `virtual`函数：

```cpp
 class Base {
 public:
   friend std::ostream& operator<< (std::ostream& o, const Base& b);
   // ...
 protected:
   virtual void printOn(std::ostream& o) const;
 };

 inline std::ostream& operator<< (std::ostream& o, const Base& b)
 {
   b.printOn(o);
   return o;
 }

 class Derived : public Base {
 protected:
   virtual void printOn(std::ostream& o) const;
 }; 
```

最终结果是`operator<<` 就象是动态绑定，即使它是一个友元函数。这被称为“虚友元函数用法”。

注意派生类重写了`printOn(std::ostream&)` `const`。尤其是，它们不提供他们自己的 `operator<<`。

自然的，如果 `Base`是一个 ABC（抽象基类），`Base::printOn(std::ostream&)` `const`可以用“`= 0`”语法被声明为纯虚函数。

## 15.10 在 DOS 和／或 OS/2 环境下，如何以二进制模式“重打开” `std::cin` 和 `std::cout` ？

这依赖于实现，请查看你的编译器的文档。

例如，假设你想使用`std::cin`和 `std::cout`进行二进制 I／O。更假设你的操作系统（如 DOS 或 OS/2）坚持将从`std::cin`输入的“`\r\n`”翻译为“`\n`”，将从 `std::cout`或`std::cerr`输出的“`\n`”翻译为“`\r\n`”。

不行的是没有标准方法使得`std::cin`，`std::cout`和／或`std::cerr`以二进制模式被打开。关闭流并且试图以二进制方式重打开它们，可能会得到非期望的或不合需要的结果。

在系统的区别处，实现可能提供了一种方法使它们成为二进制流，但你必须查看手册来找到。

## 15.11 为何我不能在如“`..\test.dat`”这样的不同的目录打开文件？

因为“`\t`”是一个 tab 字符。

你应该在文件中使用正斜杠，即使在使用反斜杠的操作系统，如 DOS, Windows, OS/2 等中。例如：

```cpp
 #include <iostream>
 #include <fstream>

 int main()
 {
   #if 1
     std::ifstream file("../test.dat");  _// 正确！_
   #else
     std::ifstream file("..\test.dat");  _// 错误！_
   #endif

   _// ..._
 } 
```

记住，反斜杠（“`\`”）被用来在字符串中建立特殊字符：“`\n`”是换行，“`\b`”是退格，以及“`\t`”是一个 tab，“`\a`”是一个警告（alert），“`\v`”是一个 vertical-tab 等。因此文件名“`\version\next\alpha\beta\test.dat`”被解释为一堆有区的字符；应该用“`/version/next/alpha/beta/test.dat`”来替代，即使系统中使用“`\`”作为目录分隔符，如 DOS, Windows, OS/2 等。这是因为操作系统中的库例程是可交换地处理“/”和“\”的。

## 15.12 如何将一个值（如，一个数字）转换为 `std::string`?

有两种方法：可以使用`<stdio>`工具或`<iostream>`库。通常，你应该使用`<iostream>`库。

`<iostream>` 库允许你使用如下的语法（转换一个`double`的示例，但你可以替换美妙的多的任何使用`<<`算符的东西）将任何美妙得多的东西转换为 `std::string`：

```cpp
 #include <iostream>
 #include <sstream>
 #include <string>

 std::string convertToString(double x)
 {
   std::ostringstream o;
   if (o << x)
     return o.str();
   // 这儿进行一些错误处理...
   return "conversion error";
 } 
```

`std::ostringstream` 对象 `o` 提供了类似`std::cout`提供的格式化工具。你可以使用操纵器和格式化标志来控制格式化的结果，就如同你用`std::cout`可以做到的。

在这个例子中，我们通过被重载了的插入运算符`<<`，将 `x` *插入*到 `o`。它调用了 iostream 的格式化工具将 `x` 转换为一个`std::string`。 `if` 测试保证转换正确工作——对于内建／固有类型，总是成功的，但 `if` 测试是良好的风格。

表达式`os.str()`返回包含了被插入到流 `o` 中的任何东西的`std::string` ，在这里，是 x 的值的字符串。

## 15.13 如何将 `std::string` 转换为数值？

有两种方法：可以使用`<stdio>`工具或`<iostream>`库。通常，你应该使用`<iostream>`库。

`<iostream>` 库允许你使用如下的语法（转换一个 `double`的示例，但你可以替换美妙的多的任何能使用 `>>` 算符被读取的东西）将一个`std::string`转换为美妙得多的任何东西：

```cpp
 #include <iostream>
 #include <sstream>
 #include <string>

 double convertFromString(const std::string& s)
 {
   std::istringstream i(s);
   double x;
   if (i >> x)
     return x;
   // 这儿进行一些错误处理...
   return 0.0;
 } 
```

`std::istringstream` 对象 `i` 提供了类似`std::cin`提供的格式化工具。你可以使用操纵器和格式化标志来控制格式化的结果，就如同你用`std::cin`能做到的。

在这个示例中，我们传递了`td::string` `s`来初始化`std::istringstream` `i` （例如，`s` 可能是字符串“`123.456`”），然后我们通过被重载了的抽取运算符 `>>`，将 `i` *抽取*到 `x`。它调用了 iostream 的格式化工具对字符串进行尽可能的／适当的基于 x 的类型的转换。

`if` 测试保证了转换正确地工作。例如，如果字符串包含不适合 x 类型的字符，`if` 测试将失败。