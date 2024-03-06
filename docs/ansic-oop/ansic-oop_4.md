# 第三章 编程的悟性——算术表达式

> 来源：[`blog.csdn.net/besidemyself/article/details/6423491`](http://blog.csdn.net/besidemyself/article/details/6423491)
> 
> 译者：[besidemyself](http://my.csdn.net/besidemyself)

动态连接就其本身而言是一项强大的编程技术，并不是去写一些带有庞大的`switch` 语句去处理很多特例的函数。我们可以写很多小的函数，对于每个`case` 语句，安排适当的函数被动态连接调用。这样做通常简化了编程工作并且会使得代码容易扩展。

作为一个例子，我们将写一个小的程序去读并评估由浮点数字，括号，常用操作符，减号，等组成的算术表达式。正常情况下我们宁愿使用编译器产生器工具`lex`和`yacc` 去建立这部分的程序去剖析算术表达式。这不是一本关于编译器建立的书，然而，就仅仅此次我们将自己写这次的代码。

## 3.1 主循环

程序的主循环从标准输入读取一行数据，初始化以便数字和操作符能被提取出来，空格被忽略，调用一个函数去确认正确的算术表达式并存储之，最终处理所存储的表达式。如果出错了，我们简单的读取下一行数据。如下为主循环：

```cpp
#include <setjmp.h>
int main (void)
{    
    volatile int errors = 0;
       char buf [BUFSIZ];

       if (setjmp(onError)){
              ++ errors;
       }

       while (fgets(buf, sizeof buf, stdin)){
              if (scan(buf))
              {     void * e = sum();

                     if (token){
                            error("trash after sum");
                     }
                     process(e);
                     delete(e);
              }
}
       return errors > 0;
}
void error (const char * fmt, ...)
{    
    va_list ap;

       va_start(ap, fmt);
       vfprintf(stderr, fmt, ap), putc('/n', stderr);
       va_end(ap);
       longjmp(onError, 1);
} 
```

错误恢复点被使用`setjmp()` 所定义。如果`error()` 在程序中的某个位置被调用，`longjmp()` 伴随着从 `setjmp()` 另外一个返回而继续执行。在这种情况下，结果是一个值被传进 `longjmp()` ，错误累加，而且下一个输入行被读取。如果遇到错误，程序的出口代码将报告错误。

## 3.2 扫描器

在主循环中，一旦一个输入行被读入到`buf[]` 中，它将被传进 `scan()`, 此函数对于每一个调用把下一个输入符号放入变量`token` 。在最后一行，`token` 的值为 0：

```cpp
#include <ctype.h>
#include <errno.h>
#include <stdlib.h>
#include “parse.h”

static double number;            /* if NUMBER: numerical value */
static enum tokens scan (const char * buf) {    
    static const char * bp;

       if (buf){
              bp = buf;               /* new input line */
       }
       while (isspace(* bp & 0xff)){
              ++ bp;
       }
       if (isdigit(* bp & 0xff) || * bp == '.')
       {
        errno = 0;
              token = NUMBER, number = strtod(bp, (char **) & bp);
              if (errno == ERANGE){
                     error("bad value: %s", strerror(errno));
              }
       }
       else{
              token = * bp ? * bp ++ : 0;
       }
       return token;
} 
```

我们调用 `scan()` ，可传递输入行缓冲的地址，或传进一个空指针得以继续工作在当前的行。空格被忽略，并且遇到第一个为数字或小数点，我们就是用一个 ANSI-C 的函数 `strtod()` 开始提取出浮点数字。若为其他的任何字符将被返回，并且我们不会预先在输入缓冲传递一个空字节。

`scan()` 的结果被存储在全局变量`token` ——这样简化了识别程序（识别器）。如果我们侦测出一个数字，我们将返回唯一的值 `NUMBER` 并使得在全局变量`number` 中实际的值有效。

## 3.3 识别器

在最高水平,表达式通过函数`sum()` 被识别，`sum()` 函数内部调用`scan()` 并返回一个表示，这个表示可通过调用 `process()` 被处理并通过`delete()` 被回收。

如果我们不使用`yacc`(是 Unix/Linux 上一个用来生成编译器的编译器（编译器代码生成器）），我们将通过递归下降的方法识别表达式，合乎语义的规则被翻译成等价的 C 函数。例如：一个`sum` 是一个产物，接下来被 0 跟随，或更多的组，每个由额外的操作符和另外的产物组成，一个语义规则如下：

```cpp
sum：product {+|- product}… 
```

被翻译成 C 函数如下：

```cpp
static void * sum (void) {    
    void * result = product();
       const void * type;

       for (;;)
       {     switch (token) {
              case '+':
              case '-':
                     scan(0),product();continue;
              return;
       }
} 
```

对于每一个语义规则有一个 C 函数，以便于这些规则能够相互调用，这些不同的分支被转换成`switch` 或 `if` 语句，迭代的语法将在 C 中翻译成循环。仅仅一个问题就是我们必须避免无限的递归。

`token` 总是包含下一个输入的符号。如果我们识别出它，我们必须调用`scan(0)`

## 3.4 处理器

我们如何来处理表达式呢？如果我们仅仅想用一些用数字表示的值执行简单的算术。我们可以扩展识别函数并且一旦识别出操作符和操作码就计算出结果如：`sum()` 应该会期望从每一个对 `product()` 的调用期望一个`double` 类型的结果，尽可能的执行加或减法，并且返回结果，再次作为一个`double` 类型函数的值。

如果我们想要建立一个系统用来处理更加复杂的表达式，我们需要存储表达式以便于后续处理。在这种情况下，我们能够不仅仅执行算术，而且可以允许决定并且有条件的评估一个表达式的一部分，且可用存储的表达式作为用户的函数包含在其他表达式中。我们所需要的是一个合理通用的方式代表一个表达式。比较常规的技术是使用一个二叉树在每一个节点上存储 `token`.

```cpp
struct Node {
    enum tokens token;
    struct Node * left, * right;
}; 
```

然而，这样并不是很灵活。我们需要介绍一个`union` 去创建一个节点，在这个节点上我们可存储一个数，并且我们在这些节点代表的一元操作符上浪费了空间。此外，`process()` 和 `delete()` 将包含`witch` 分支，并`witch` 分支会随着我们增加的符号而增多。

## 3.5 信息隐藏

应用迄今为止我们学到的，我们绝不去揭示节点结构。相反，我们先在头文件 `value.h`中放置一些声明如下：

```cpp
const void * Add;
    …
void * new (const void * type, ...);
void process (const void * tree);
void delete (void * tree); 
```

现在我们可以编写代码 `sum()` 如下：

```cpp
#include "value.h"
static void * sum (void) {    
    void * result = product();
       const void * type;

       for (;;)
       {    
        switch (token) {
                  case '+':
                         type = Add;
                         break;
                  case '-':
                         type = Sub;
                         break;
                  default:
                         return result;
              }
              scan(0);
              result = new(type, result, product());
       }
} 
```

`product()` 与 `sum()` 有相同的结构，并且调用 一个函数 `factor()` 去识别数字，符号，且`sum`被赋予了括号：

```cpp
static void * factor (void) {    
    void * result;

       switch (token) {
       case '+':
              scan(0);
              return factor();
       case '-':
              scan(0);
              return new(Minus, factor());
       default:
              error("bad factor: '%c' 0x%x", token, token);
       case NUMBER:
              result = new(Value, number);
              break;
       case '(':
              scan(0);
              result = sum();
              if (token != ')')
                     error("expecting )");
       }
       scan(0);
       return result;
} 
```

尤其在 `factor()` 中，我们需要特别小心的保持扫描器（scanner）是不变的：`token` 必须总是包含下一个输入的符号。一旦`token` 被使用，我们需要调用 `scan(0)`。

## 3.6 动态连接

识别器是完善的。`value.h` 对于算术表达式完全隐藏了求值程序，且与此同时指定了我们必须所实现的。 `new()` 携带描述符，如`Add` 和合适的参数如指针对加的操作且返回一个表示和的指针。

```cpp
struct Type {
       void * (* new) (va_list ap);
       double (* exec) (const void * tree);
       void (* delete) (void * tree);
};

void * new (const void * type, ...) {    
va_list ap;
       void * result;

       assert(type && ((struct Type *) type) -> new);

       va_start(ap, type);
       result = ((struct Type *) type) -> new(ap);
       * (const struct Type **) result = type;
       va_end(ap);
       return result;
} 
```

我们使用动态连接并传递一个对指定节点例程的调用，在例程中的`Add` 分支处，必须常见一个节点，并且传进两个指针。

```cpp
truct Bin {
       const void * type;
       void * left, * right;
};   

static void * mkBin (va_list ap) {    
struct Bin * node = malloc(sizeof(struct Bin));

       assert(node);
       node -> left = va_arg(ap, void *);
       node -> right = va_arg(ap, void *);
       return node;
} 
```

注意，只有 `mkBin()` 知道它创建的是什么。所有我们要求的是各个节点对于动态连接是以一个指针开始。这个指针被 `new()` 传进一遍于`delete()` 能够调用到它指定节点的函数：

```cpp
void delete (void * tree) {
   assert(tree && * (struct Type **) tree
           && (* (struct Type **) tree) -> delete);

   (* (struct Type **) tree) -> delete(tree);
} 
```

动态连接很优雅的避免了复杂难解的节点。`.new()` 精确的创建了每个类型描述符的右节点：二元操作符拥有两个子孙。一元操作符拥有一个子孙，且值节点仅仅包含了值。`delete()`是一个非常简单的函数因为每个节点处理它自己的销毁过程：二元操作符删除两个子树并且释放他们自己的节点，一元操作符仅仅删除一个子树，且值节点仅仅释放自己。变量和常量甚至可以留到后面——对于`delete()` 的回应他们简单的什么也不做。

## 3.7 A Postfix Writer

到目前为止我们还没有真正的决定 `process()` 将要真正做什么。如果我们想要发布一个表达式的后缀版，我们将要对 `struct Type` 增加一个字符串以便于显示出实际的操作符，且 `process()` 将要安排一个单独的被`tab` 键缩进的行：

```cpp
void process (const void * tree)
{
       putchar('/t');
       exec(tree, (* (struct Type **) tree) -> rank, 0);
       putchar('/n');
} 
```

`exec()` 处理动态连接

```cpp
static void exec (const void * tree, int rank, int par) {
       assert(tree && * (struct Type **) tree
              && (* (struct Type **) tree) -> exec);

       (* (struct Type **) tree) -> exec(tree, rank, par);
} 
```

每一个二元操作符被使用如下函数发出：

```cpp
static void doBin(const void *tree) {
    exec(((struct Bin *) tree) —> left);
    exec(((struct Bin *) tree) —> right);
    printf(" %s", (* (struct Type **) tree) —> name);
} 
```

类型描述符如下绑定：

```cpp
static struct Type _Add = { "+", mkBin, doBin, freeBin };
static struct Type _Sub = { "—", mkBin, doBin, freeBin };
const void * Add = & _Add;
const void * Sub = & _Sub; 
```

应该很容易猜测一个数值是怎样被实现的。它被代表作为一个结构体携带`double` 类型的信：

```cpp
struct Val {
    const void * type;
    double value;
};
static void * mkVal (va_list ap) {
    struct Val * node = malloc(sizeof(struct Val));
    assert(node);
    node —> value = va_arg(ap, double);
    return node;
} 
```

处理组成的打印值：

```cpp
static void doVal (const void * tree) {
    printf(" %g", ((struct Val *) tree) —> value);
} 
```

我们已经做了——没有子树要删除，因此我们可以使用库函数 `free()` 直接的删除值节点：

```cpp
static struct Type _Value = { "", mkVal, doVal, free };
const void * Value = & _Value; 
```

一元操作符如`Minus` 将留作练习。

## 3.8 算术

如果我们想做算术运算，我们让执行的函数返回一个`double` 类型的值，然后让`process()` 打印这个值：

```cpp
static double exec (const void * tree) {
    return (* (struct Type **) tree) —> exec(tree);
}
void process (const void * tree) {
    printf("/t%g/n", exec(tree));
} 
```

对于每个节点的类型，我们需要一个执行函数来计算和返回这个节点的值。这里有两个实例：

```cpp
static double doVal (const void * tree) {
    return ((struct Val *) tree) —> value;
}
static double doAdd (const void * tree) {
    return exec(((struct Bin *) tree) —> left) +
    exec(((struct Bin *) tree) —> right);
}
static struct Type _Add = { mkBin, doAdd, freeBin };
static struct Type _Value = { mkVal, doVal, free };
const void * Add = & _Add;
const void * Value = & _Value; 
```

## 3.9 插入输出

也许对于处理算术表达式的突出点是带小括号的形式打印。这通常是有点滑稽的，依照谁来负责发出括号。此外对于操作符的名字用于前缀输出，我们增加了两个数值到`struct Type`中。

```cpp
struct Type {
    const char * name; /* node’s name */
    char rank, rpar;
    void * (* new) (va_list ap);
    void (* exec) (const void * tree, int rank, int par);
    void (* delete) (void * tree);
}; 
```

`.rank` 是优先的操作符，以 1 开始，此外 `.rpar` 被设置用于操作符，如减操作，此操作如果用于相等的优先级的操作就要求他们的右操作被附上括号。

```cpp
$ infix
1 + (2 — 3)
1 + 2 — 3
1 — (2 — 3)
1 — (2 — 3) 
```

这个证实了我们需要如下的初始化：

```cpp
static struct Type _Add = {"+", 1, 0, mkBin, doBin, freeBin};
static struct Type _Sub = {"—", 1, 1, mkBin, doBin, freeBin}; 
```

滑稽的部分是对于二元节点得去决定它是否必须要增加括号。一个二元节点如加法，被给予它自己较高的优先级并且一个标记指示在相等的优先级中括号是否是必须的。`doBin()` 去判别是否使用括号:

```cpp
static void doBin (const void * tree, int rank, int par) {
    const struct Type * type = * (struct Type **) tree;
    par = type —> rank < rank
    || (par && type —> rank == rank);
    if (par)
    putchar(’(’);
    exec(((struct Bin *) tree) —> left, type —> rank, 0);
    printf(" %s ", type —> name);
    exec(((struct Bin *) tree) —> right,
    type —> rank, type —> rpar);
    if (par)
    putchar(’)’);
} 
```

与高优先级的操作符比若我们有一个较低优先级，或者如果我们被要求在相等的优先级情况下输出括号，我们就打印括号。在任何情况下，如果我们的描述有 `.rpar` 的设置，我们要求仅仅我们的所有操作输出额外的括号如上：

保持打印的实例程序是较容易写的。

## 3.10 总结

三种不同的处理器证实了信息隐藏的优越性。动态连接帮助我们把一个问题分解成很简单的函数功能点。最终的程序是很容易扩展的——试着去增加 C 语言中的比较和如`?:`的操作符吧。