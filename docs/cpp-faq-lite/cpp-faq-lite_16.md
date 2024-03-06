# [18] const 正确性

# [18] const 正确性

## FAQs in section [18]:

*   [18.1] 什么是“const 正确性”？
*   [18.2] “const 正确性”是如何与普通的类型安全有何联系？
*   [18.3] 我应该“尽早”还是“推迟”确定 const 正确性？
*   [18.4] “const Fred* p”是什么意思？
*   [18.5] “const Fred *p”、“Fred* const p”和“const Fred* const p”有什么不同？
*   [18.6] “const Fred& x”是什么意思？
*   [18.7] “Fred& const x”有意义吗？
*   [18.8] “Fred const& x”是什么意思？
*   [18.9] “Fred const* x”是什么意思？
*   [18.10] 什么是“const 成员函数”？
*   [18.11] 返回引用的成员函数和 const 成员函数之间有什么联系？
*   [18.12] “const 重载”是做什么用的？
*   [18.13] 如果我想让一个 const 成员函数对数据成员做“不可见”的修改，应该怎么办？
*   [18.14] const_cast 会导致无法优化么？
*   [18.15] 当我用 const int*指向一个 int 后，为什么编译器还允许我修改这个 int？
*   [18.16] “const Fred *p”的意思是*p 不会改变么？
*   [18.17] 当把 Foo**转换成 const Foo**时为什么会出错？

## 18.1 什么是“`const`正确性”？

这是个好东西。意思是用`const`关键字来阻止`const`对象被修改。

例如，如果你要编写一个函数`f()`，它接收`std::string`类型的参数，并且想要对调用者保证不会修改调用者传过来的`std::string`参数，可以按以下方法声明`f()`

*   `void f1(const std::string& s); //传 const 引用`
*   `void f2(const std::string* sptr); //传 const 指针`
*   `void f3(std::string s); //传值`

在*传`const`引用*和*传`const`指针*的情况下，任何试图在`f()`内部修改`std::string`的行为都会在编译时被编译器标记为错误。这完全是在编译时做的，所以使用`const`没有运行时的空间或速度损失。在*传值*时（`f3()`），被调用函数获得了调用者`std::string`的一份拷贝。也就是说，`f3()`可以修改这个拷贝，但返回时这个拷贝会被销毁。尤其是`f3()`无法修改调用者的`std::string`对象。

举个反例，如果想要编写一个函数`g()`，也是接收`std::string`，但想要告知调用者 g()有可能会修改调用者的`std::string`对象。这时，可以按以下方法声明`g()`:

*   `void g1(std::string& s); //传非 const 引用`
*   `void g2(std::string* sptr); //传非 const 指针`

在这些函数中省去`const`，就是告诉编译器允许（但不强制）它们修改调用者的`std::string`对象。因此，这些`g()`函数可以把它们的`std::string`传递给任何`f()`函数，但只有`f3()`（通过传值接收参数）能够将其参数传递给`g1()`或`g2()`。如果`f1()`或`f2()`需要调用`g()`函数，必须给`g()`传递一份`std::string`的本地拷贝。`f1()`或`f2()`的参数不能直接传递给`g()`函数。例如：

```cpp
 void g1(std::string& s);

 void f1(const std::string& s)
 {
   g1(s);          // 编译错误，因为 s 是 const 的

   std::string localCopy = s;
   g1(localCopy);  // 正确，因为 localCopy 不是 const 的
 } 
```

当然，在上面的例子中，任何`g1()`所做的修改都会反映到`f1()`函数内的`localCopy`对象。特别是，通过`const`引用传递给`f1()`的参数不会被修改。

## 18.2 “`const`正确性”是如何与普通的类型安全有何联系？

将参数声明为`const`正是另外一种形式的类型安全。这就好像`const std::string`是与`std::string`不同的类一样。因为`const`变量缺少一些非`const`变量所具有的一些变更性操作（例如，可以想象以下，`const std::string`没有赋值操作符）。

如果你发现普通的类型安全有助于构建正确的系统（的确有帮助，尤其是对于大型系统来说），你会发现`const`正确性也有帮助。

## 18.3 我应该“尽早”还是“推迟”确定`const`正确性？

应该在最最最开始。

事后保证`const`正确性会导致一种滚雪球效应：每次你在一个地方添加了`const`会需要在四个更多的地方也添加`const`。

## 18.4 “`const Fred* p`”是什么意思？

意思是`p`是一个指向`Fred`类的指针，但不能通过`p`来修改`Fred`对象（当然`p`也可以是`NULL`指针）。

例如，假设`Fred`类有一个叫做`inspect()`的`const`成员函数，那么写`p->inspect()`是可以的。但如果`Fred`类有一个非`const`成员函数`mutate()`，那么写`p->mutate()`就是个错误（编译器会捕获这种错误；不会在运行时测试；因此`const`不会降低运行速度）。

## 18.5 “`const Fred* p`”、“`Fred* const p`”和“`const Fred* const p`”有什么不同？

应该从右往左读指针声明。

*   `const Fred* p`表明`p`指向一个`const`的`Fred`对象——`Fred`对象不能通过`p`修改。
*   `Fred* const p`表明`p`是一个指向`Fred`对象的`const`指针——可以通过`p`修改`Fred`对象，但不能修改`p`本身。
*   `cosnt Fred* const p`表明“`p`是一个指向`const Fred`对象的`const`指针”——不能修改`p`，也不能通过`p`修改`Fred`对象。

## 18.6 “`const Fred& x`”是什么意思？

意思是`x`是`Fred`对象的一个别名，但不能通过`x`来修改`Fred`对象。

例如，假设`Fred`类有一个叫做`inspect()`的`const`成员函数，那么写`x.inspect()`是可以的。但如果`Fred`类有一个非`const`成员函数`mutate()`，那么写`x.mutate()`就是个错误（编译器会捕获这种错误；不会在运行时检查；因此`const`不会降低运行速度）。

## 18.7 “`Fred& const x`”有意义吗？

没意义。

为了理解这个声明，需要从右往左读这个声明。因此“`Fred& const x`”的意思是“`x`是一个指向`Fred`的`const`引用”。但这是多余的，因为引用本来就是`const`的。你不能重新绑定一个引用。不管有没有`const`，都不行。

换句话说，“`Fred& cosnt x`”在功能上与“`Fred& x`”是一样的。因为在`&`后面加上`const`没什么用，因此为了避免迷惑就不应该多此一举。有人可能会认为这里加上`const`后指向的`Fred`对象就是`const`的了，就好像“`const Fred& x`”一样。

## 18.8 “`Fred const& x`”是什么意思？

“`Fred cosnt& x`”在功能上与`const Fred& x`相同。然而，真正的问题是*应该*用哪一种。

答案：绝*没有任何人*能够为你所在的机构做决定，除非他们了解你的机构。没有放之四海而皆准的规则。没有对所有机构都“正确”的答案。所以不要让*任何人*做仓促的选择。“思考（Think）”并非一个四字母的单词。

例如，一些机构看重的是一致性，并且已经有大量代码使用“`const Fred&`”了。对于他们来说，不管是否有优点，“`Fred const&`”都不是个好选择。还有很多其它的商业环境，一些倾向于“`Fred const&`”，其它则倾向于“`const Fred&`”。

采用适合你机构中*普通维护程序员*的写法。不是专家，不是傻瓜，而是维护代码的普通程序员。除非你决定解雇他们并雇佣新人，否则就要确保*他们*能够理解你的代码。根据实际情况做商业决定，而不是根据其它什么人的假设。

使用“`Fred const&`”需要克服一些惯性。现在大多数 C++书籍都使用`const Fred&`，大多数程序员学 C++时接触的就是这种语法，并且仍然这么用。这并非是说`const Fred&`一定对你的机构好。但在更改（这种风格）期间，和/或在招收新人时，的确可能会引起一些混乱。一些机构认为用`Fred const&`带来的好处更大，其它机构则不这么认为。

另一个警告：如果决定用`Fred const&`，确保采取措施使人们不会误写成没意义的“`Fred& const x`”。

## 18.9 “`Fred const* x`”是什么意思？

“`Fred cosnt* x`”在功能上与`const Fred* x`相同。然而，真正的问题是*应该*用哪一种。

答案：绝*没有任何人*能够为你所在的机构做决定，除非他们了解你的机构。没有放之四海而皆准的规则。没有对所有机构都“正确”的答案。所以不要让*任何人*做仓促的选择。“思考（Think）”并非一个四字母的单词。

例如，一些机构看重的是一致性，并且已经有大量代码使用“`const Fred*`”了。对于他们来说，不管是否有优点，“`Fred const*`”都不是个好选择。还有很多其它的商业环境，一些倾向于“`Fred const*`”，其它则倾向于“`const Fred*`”。

采用适合你机构中*普通维护程序员*的写法。不是专家，不是傻瓜，而是维护代码的普通程序员。除非你决定解雇他们并雇佣新人，否则就要确保*他们*能够理解你的代码。根据实际情况做商业决定，而不是根据其它什么人的假设。

使用“`Fred const*`”需要克服一些惯性。现在大多数 C++书籍都使用`const Fred*`，大多数程序员学 C++时接触的就是这种语法，并且仍然这么用。这并非是说`const Fred*`一定对你的机构好。但在更改（这种风格）期间，和/或在招收新人时，的确可能会引起一些混乱。一些机构认为用`Fred const*`带来的好处更大，其它机构则不这么认为。

另一个警告：如果决定用`Fred const*`，确保采取措施使人们不会误写成语义不同但语法相似的“`Fred* const x`”。这两者虽然第一眼看上去非常相似，但含义完全不同。

## 18.10 什么是“`const`成员函数”？

是指仅查看（而不改变）对象的成员函数。

`const`成员函数会在紧跟函数参数列表的后面跟一个`const`关键字。有`const`后缀的成员函数被称作“`const`成员函数”或者是“查看函数”（inspector）。没有`const`后缀的成员函数被称作“非`const`函数”或“变更函数”（mutator）。

```cpp
 class Fred {
 public:
   void inspect() const;   // 该成员保证不修改*this
   void mutate();          // 该成员可能会修改*this
 };

 void userCode(Fred& changeable, const Fred& unchangeable)
 {
   changeable.inspect();   // 正确：没有修改一个可修改对象
   changeable.mutate();    // 正确：修改一个可修改对象

   unchangeable.inspect(); // 正确：没有修改一个不可修改对象。
   unchangeable.mutate();  // 错误：试图修改一个不可修改对象。
 } 
```

`unchangeable.mutate()`这个错误在编译期被发现。`const`不会有运行时的时空效率损失。

在`inspect()`成员函数后面的`const`后缀表示不会改变对象的（调用方可见的）*抽象*状态。这并非保证不改变对象的“底层二进制位”。C++编译器不允许将其解释为“按位”（不变），除非能解决别名问题，而别名问题一般无法解决（即可能存在会修改对象状态的非`const`别名）。另外一个对这种别名问题的（重要）认识是：用一根“指向`const`对象的指针”并不能保证对象不改变，它只是保证对象不会*通过该指针*被改变。

## 18.11 返回引用的成员函数和`const`成员函数之间有什么联系？

如果想要从一个查看函数中返回`this`对象的引用，那么应该返回指向`cosnst`对象的引用，即`const X&`。

```cpp
 class Person {
 public:
   const std::string& name_good() const;  ← 正确：调用者不能修改 name
   std::string& name_evil() const;        ← 错误：调用者能够修改 name...
 };

 void myCode(const Person& p)  ← 这里保证不会修改 Person 对象...
 {
   p.name_evil() = "Igor";     ← ...但还是修改了！！
 } 
```

好消息是当你犯这种错误时，编译器*通常*能够发现。尤其是如果不小心返回了`this`对象的非`const`引用，例如上面的`Person::name_evil()`，编译器在编译这个成员函数时，*通常*能够发现并给出一条编译错误。

坏消息是编译器并不能发现所有这种错误：在一些情况下编译器无法产生一条错误消息。

最后：你需要思考，并记住本 FAQ 所述的原则。如果你通过引用返回的对象在*逻辑上*是`this`对象的一部分，而不管其是否在物理上放在了`this` 对象内，那么`const`方法应该返回`const`引用或直接按值返回。（`this`对象的“逻辑”部分与对象的“抽象状态”相关。请参阅前一个 FAQ。）

## 18.12 “`const`重载”是做什么用的？

当一个查看函数和一个变更函数名字相同，且参数个数与类型也相同时就有用了——即两者的不同之处仅在于一个有`const`另一个没有`const`。

`const`重载的一个常见应用是下标运算符。通常应该尽量使用标准模板容器，例如`std::vector`，但有时会需要在自己的类中支持下标运算符。一个经验法则是：**下标运算符通常成对出现**。

```cpp
 class Fred { ... };

 class MyFredList {
 public:
   const Fred& operator[] (unsigned index) const;  ←下标运算符通常成对出现
   Fred&       operator[] (unsigned index);        ←下标运算符通常成对出现...
 }; 
```

当对一个非`const`的`MyFredList`对象使用下标运算符时，编译器会调用非`const`的下表运算符。因为返回的是一个普通`Fred&`，所以能够查看或修改对应的`Fred`对象。例如，假设`Fred`类有一个查看函数`Fred::inspect() const`和一个变更函数`Fred::mutate()`：

```cpp
 void f(MyFredList& a)  ← MyFredList 是非 const 的
 {
   // 可以调用不修改 a[3]处 Fred 对象的方法：
   Fred x = a[3];
   a[3].inspect();

   // 可以调用修改 a[3]处 Fred 对象的方法:
   Fred y;
   a[3] = y;
   a[3].mutate();
 } 
```

但是，当对一个`const`的`MyFredList`对象使用下标运算符时，编译器会调用`const`的下标运算符。因为会返回`const Fred&`，所以可以查看对应的`Fred`对象而不能修改它。

```cpp
 void f(const MyFredList& a)  ← MyFredList 是 const 的
 {
   // 可以调用不修改 a[3]处 Fred 对象的方法:
   Fred x = a[3];
   a[3].inspect();

   // 错误（很幸运！）：试图改变 a[3]出的 Fred 对象:
   Fred y;
   a[3] = y;       ← 幸运的是编译器在编译时发现了这个错误。
   a[3].mutate();  ← 幸运的是编译器在编译时发现了这个错误。
 } 
```

在以下 FAQ 中演示了针对下标运算符和函数调用运算符的`const`重载：[13.10], [16.17], [16.18], [16.19]和[35.2]

当然除了下标运算符，其它函数也可以进行`const`重载。

## 18.13 如果我想让一个`const`成员函数对数据成员做“不可见”的修改，应该怎么办？

用`mutable`（或者实在没办法了，用最后一招`const_cast`）

少数查看函数需要对数据成员做适当的修改（例如，一个`Set`对象可能想要缓存上一次查看的内容，以便下一次查看时能够提高性能）。这里“适当”的意思是，所做的修改不会从对象的接口上反映到外部（否则该成员函数就应该是一个变更函数，而不是查看函数了）。

这时，需要修改的数据成员应标记为`mutable`（把`mutable`关键字放在数据成员的声明前；即和`const`的位置一样）。这就通知编译器说这个数据成员允许在`const`成员函数中被修改。如果编译器不支持`mutable`关键字，那么可以通过`const_cast`去除掉`this`的`const`（但是记着读下面的**注意事项**）。例如在`Set::lookup() const`中，可以这么写：

```cpp
 Set* self = const_cast<Set*>(this);
   // 在这么做之前，记着读下面的**注意事项** 
```

然后，`self`和`this`内容一样（即`self == this`为真），但`self`类型是`Set*`而不是`const Set*`（技术上来讲，是`const Set* const`，不过最右边的`const`与这里的问题无关）。因此可以使用`self`来修改`this`所指向的对象。

**注意：**`const_cast`可能会导致一种非常罕见的错误。这个错误仅在三件很少见的事情同时发生时出现：数据成员本应该是`mutable`（例如上面所说的），编译器不支持`mutable`，并且对象原本就定义为`const`（不是通过一根指向`const`对象的指针来访问的普通`const`对象）。虽然这种组合非常罕见，甚至永远不会发生，但如果真的发生了，那么这种代码可能就不能正常运行（标准说这种行为是未定义的）。

如果想要用`const_cast`，那么应该用`mutable`替代。换句话说，如果需要修改一个对象的成员，而又是通过指向`const`对象的指针来访问这个对象，那么最安全和最简单的做法就是给该成员的声明前加上`mutable`。如果你确信实际对象不是`const`的（例如能够确定对象是像这样声明的：`Set s;`），那么也可以用`const_cast`。但如果对象本身就是`const`的（例如可能声明为：`const Set s;`），那么就应该用`mutable`而不是`const_cast`。

请不要告诉我说 Y 编译器的 X 版本在 Z 机器上允许修改`const`对象的非`mutable`成员。我不管这个——根据标准这是错误的，如果换一个编译器，甚至是同一编译器的不同版本（升级版），你的代码就可能会失败。不要这么做。用`mutable`吧。

## 18.14 `const_cast`会导致无法优化么？

在理论上是的；在实际中不会。

即使语言本身禁止了`const_cast`，要想在调用`const`成员函数时避免读写寄存器的唯一办法是解决别名问题（即要证明没有其它指向该对象的非`const`指针）。这只有在极少数情况下才能办到（当在调用`const`成员函数时构造对象，所有在构造对象和调用`const`成员函数之间调用的非`const`成员函数是静态绑定的，并且所有这些调用包括构造函数都是内联的，同时构造函数调用的任何成员函数也要是内联的）。

## 18.15 当我用`const int*`指向一个`int`后，为什么编译器还允许我修改这个`int`？

因为“`const int* p`”意思是“`p`保证不会修改`*p`”，而*不是说*“`*p`保证不变”。

用`const int*`指向一个`int`，不会使这个`int`变为`const`。`int`不会通过`const int*`被修改，但如果有另外一个`int*`（注意没有`const`）指向该`int`（这就是“别名”），那么这个`int*`指针可以用来修改`int`。例如：

```cpp
 void f(const int* p1, int* p2)
 {
   int i = *p1;         // 获得*p1 的（原始）值*p1
   *p2 = 7;             // 如果 p1 == p2，这就会修改*p1。
   int j = *p1;         // 获取*p1（可能是更新后）的值。
   if (i != j) {
     std::cout << "*p1 changed, but it didn't change via pointer p1!\n";
     assert(p1 == p2);  // 这是*p1 可能会变化的唯一办法。
   }
 }

 int main()
 {
   int x = 5;
   f(&x, &x);           // 这完全合法（甚至是合情理的！）...
 } 
```

注意`main()`和`f(const int*, int*)`可能是在不同的编译单元中，并且不是在同一天编译的。因此，编译器就无法在编译时发现别名。因此无法在语言中禁止这种事情。实际上，我们甚至不想添加这样一个规则。因为一般来说，允许很多指针指向同一个对象，这是一个功能。当指针保证说不去修改所指内容时，这只是该*指针*作出的保证，而*不是*内容所做的保证。

## 18.16 “`const Fred* p`”的意思是`*p`不会改变么？

不是！（这个与 int 指针的别名问题相关）

“`const Fred* p`”意思是不能通过指针`p`来修改`Fred`，但有可能不经过`const`（例如一个非`const`指针`Fred*`），而是通过其它途径来访问`object`。例如，如果有两根指针“`const Fred* p`”和“`Fred* q`”都指向同一个`Fred`对象（别名），那么指针`q`可以用来修改`Fred`对象，但指针`p`不能。

```cpp
 class Fred {
 public:
   void inspect() const;   // const 成员函数
   void mutate();          // 非 const 成员函数
 };

 int main()
 {
   Fred f;
   const Fred* p = &f;
         Fred* q = &f;

   p->inspect();    // 可以：不改变*p_
   p->mutate();     // 错误：不能通过 p 来修改*p

   q->inspect();    // 可以：允许使用 q 来查看对象
   q->mutate();     // 可以：允许使用 q 来修改对象

   f.inspect();     // 可以：允许使用 f 来查看对象
   f.mutate();      _// 可以：允许使用 f 来修改对象... 
```

## 18.17 当把`Foo**`转换成`const Foo**`时为什么会出错？

因为把`Foo**`转换成`const Foo**`是非法且危险的。

C++允许`Foo*`到`const Foo*`的转换（这是安全的）。但如果想要将`Foo**`隐式转换成`const Foo**`则会报错。

这么做的原因如下所示。但首先，这里有个最普通的解决办法：只要把`const Foo**`改成`const Foo* const*`就可以了。

```cpp
 class Foo { /* ... */ };

 void f(const Foo** p);
 void g(const Foo* const* p);

 int main()
 {
   Foo** p = /*...*/;
   ...
   f(p);  // 错误：将 Foo**转换成 const Foo**是非法且邪恶的
   g(p);  // 可以：将 Foo**转换成 const Foo* const*是合法且合理的...
 } 
```

之所以`Foo**`到`const Foo**`的转换是危险的，是因为这会使你没有经过转换就在不经意间修改了`const Foo`对象。

```cpp
 class Foo {
 public:
   void modify();  // 修改 this 对象
 };

 int main()
 {
   const Foo x;
   Foo* p;
   const Foo** q = &p;  // 这时 q 指向 p；（幸亏）这是个错误。
   *q = &x;             // 这时 p 指向 x
   p->modify();         // 啊：修改了 const Foo！！...
 } 
```

记住：*请不要*用指针转换绕过这里。别这么做就是了！