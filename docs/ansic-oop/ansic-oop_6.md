# 第五章 编程经验——符号表

> 来源：[`github.com/oxnz/ooc/blob/master/tex/chapter0x05.tex`](https://github.com/oxnz/ooc/blob/master/tex/chapter0x05.tex)
> 
> 译者：[oxnz](https://github.com/oxnz)

一个结构体明智的加长，以此来共享基本结构的功能，可以帮助省去笨重的`union`的使用。特别在具有动态绑定，我们得到了一种统一的且完美健壮的方式处理消息传递。一旦基本机制就位，一个新的扩展类就可以重用基本代码容易的添加。

作为一个例子，我们将会添加关键字、常量、变量和数学函数到第三章开始的小计算器中。所有这些对象存在一个符号表中并且共享相同的名称搜索技术。

## 扫瞄标识符

在 3.2 节中我们实现了函数`scan()`，从主程序获取一行输入并在每次调用返回一个输入符号。如果我们想引进关键字，命名常量等等，我们需要扩展`scan()`。像浮点数一样，我们提取字符数字串以供深入分析：

```cpp
 #define ALNUM    "ABCDEFGHIJKLMNOPQRSTUVWXYZ" \
                    "abcdefghijklmnopqrstuvwxyz" \
                    "_" "0123456789"

    static enum tokens scan (const char * buf) {
        static const char * bp;
        …
        if (isdigit(* bp) || * bp == '.')
            …
        else if (isalpha( *bp) || *bp == '_') {
            char buf[BUFSIZ];
            int len = strspn(bp, ALNUM);

            if (len >= BUFSIZ)
                error("name too long: %-.10s…", bp);

            strncpy(buf, bp, len), buf[len] = '\0', bp += len;
            token = screen(buf);
        }
        ... 
```

一旦我们有了一个标识符，我们让新函数`screen()`来决定它的`token`值应当是什么。如果有必要，`screen()`将会存放一个解析器可以识别的符号描述到全局变量`symbol`中。

## 使用变量

一个变量参与两个操作：它的值被用作一个表达式的操作数，或者表达式的赋值对象。第一中操作是对 3.5 节中一个简单的`factor()`扩展。

```cpp
static void * factor (void) {
    void * result;
    ...
    switch (token) {
        case VAR:
            result = symbol;
            break;
        ... 
```

`VAR`是一个当`screen()`发现适当的标识符的时候放到`token`中的唯一值。有关标识符的附加信息被放到全局变量 symbol 中。在这种情况下，`symbol`包含一个节点来标示变量作为表达式树中的一个叶子。`screen()`要么找到变量再符号表中或者使用描述`Var`去创建它。

识别一个赋值有点复杂。如果我们的计算器允许如下两种语法的声明，它将会是舒适的：

```cpp
asgn : sum
     | VAR = asgn 
```

不幸的是，`VAR`也可以出现在`sum`的左端，也就是说，使用我们递归下降的技术如何识别 C 风格嵌入赋值不是立即就能清楚的。因为我们想要学到如何操作关键字，我们设置如下的语法：

> 这里有一个技巧：对`sum`做简单尝试。如果返回是下一个输入符号是`=`，`sum`必定是一个变量叶子节点，我们就可以创建这个赋值。

```cpp
stmt  : sum
    | LET VAR = sum 
```

这被翻译成如下函数：

```cpp
static void * stmt (void) {
    void * result;

    switch (token) {
        case LET:
            if (scan(0) != VAR)
                error("bad assignment");
            result = symbol;
            if (scan(0) != '=')
                error("expecting =");
            scan(0);
            return new(Assign, result, sum());
        default:
            return sum();
    } /* this i s comet */
} 
```

在主程序中我们调用`stmt()`来替代`sum()`，并且我们的识别器已经准备好操作变量。`Assign`是一个新的类型描述，来计算一个`sum`的值并赋值给一个变量。

## 筛子--`Name`

赋值具有如下语法：

```cpp
stmt : sum
     | LET VAR = sum 
```

`LET`是关键字的一个例子。在构建筛子的过程中我们仍然可以决定什么标识符将标示`LET: scan()`从输入行提取一个标识符并传递给`screen()`，是它在符号表中查找并返回`token`的适当值，至少一个变量，一个节点在`symbol`中。

识别器丢弃`LET`但是插入变量作为叶子节点在树中。对于另一个符号，例如一个算数函数的名称，我们可能想要适用`new()`到`screener`返回的符号来获取一个新的节点。因此，我们的符号表入口应当对与大部分具有相同的函数动态绑定与我们树节点。

对于一个关键字，一个`Name`需要包含输入字符串和`token`值。稍后我们想要继承`Name`；因此，我们定义结构在`Name.r`中：

```cpp
struct Name {            /* base structure */
    const void * type;    /* for dynamic linkage */
    const char * name;    /* may be malloc-ed */
    int token;
}; 
```

我们的符号从不死亡：他们的名字是预定义的常量字符串还是存储的用户自定义变量动态字符串是没有关系的--我们将不会回收他们。

在我们可以定义一个符号之前，我们需要输入它到符号表。这不能通过调用`new(Name, ...)`来处理，因为我们想要支持更多比`Name`复杂的符号，并且我们想要隐藏符号表的实现。相反的，我们提供一个函数`install()`，它需要一个`Name`对象并把它插入到符号表中。这里给出符号表接口文件`Name.h`：

```cpp
extern void * symbol;    /* -> last Name found by screen() */
void install (const void * symbol);
int screen (const char * name); 
```

识别器必须插入像`LET`的关键字到符号表中，在他们被`screener`发现之前。这些关键字可以被定义进一个常量表结构中--它对`install()`没有影响。下面的函数被用来初始化识别：

```cpp
#include "Name.h"
#include "Name.r"

static void initName (void) {
    static const struct Name names [] = {
        { 0, "let", LET },
        0
    };
    const struct Name * np;

    for (np = names; np->name; ++np)
        install(np);
} 
```

注意`names[]`，关键字表，不需要被存储。我们适用`Name`的标示来定义`names[]`，也就是说，我们包含`Name.r`。由于关键字`LET`被丢弃，我们不提供动态绑定的方法。

## 父类的实现--`Name`

通过名字搜索符号是一个标准问题。不幸的是，ANSI 标准没有定义一个合适的库函数来解决它。`bsearch()`--有序表中的二分查找--比较接近，但是如果我们想要插入一个单独的新符号，我们不得不调用`qsort()`来设置阶段给后续搜索。

UNIX 系统很可能提供两三个函数家族来处理动态增长表。`lsearch()`--线性搜索一个数组并在`end(!)`添加--不是完全高校的。`hsearch()`--一个结构体哈希表由一个文本和一个信息指针组成--维护一个固定大小的表并且强制一个尴尬的结构体入口。`tsearch()`--一个二叉树具有任意比较和删除--是最常用的家族但是很没有效率，如果初始符号从一个有序序列中安装。

在一个 UNIX 系统上，`tsearch()`有可能是最好的折衷。对于一个可移植的实现具有二元线程树可以在`[Sch87]`找到。然而，如果这个家族不可用，或者如果我们不能保证一个随机的初始化，我们应当查看一个简单的设备来实现。一个小心实现的`bsearch()`可以很容易的被扩展来支持存储的数组插入：

```cpp
void * binary (const void * key,
    void * _base, size_t * help, size_t width,
    int (* cmp) (const void * key, const void * elt)) {
    size_t nel = * nelp;
#define base    (* (char **) & _base)
    char * lim = base + nel * width, * high;

    if (nel > 0) {
        for (high = lim - width; base <= high; nel >>= 1) {
            char * mid = base + (nel >> 1) * width;
            int c = cmp(key, mid);

            if (c < 0)
                high = mid - width;
            else if (c > 0)
                base = mid + width, --nel;
            else
                return (void *) mid;
        } 
```

到这里为止，这是一个任意数组的二元搜索。`key`只想要找的对象；`base`初始值是一个具有`*nelp`个元素的表的开始位置，每个元素`width`字节；并且`cmp`是一个函数用来比较`key`和一个表中的元素。在此我们要么找到一个元素并返回它的位置，要么`base`现在是`key`应当出现在表中的位置地址。我们继续如下：

```cpp
 memmove(base + wdith, base, lim - base);
    }
    ++ * nelp;
    return memcpy(base, key, width);
#undef base
} 
```

`memmove()`移动数组的末尾，`memcpy()`插入`key`。我们假定数组之外还有空间并且我们通过`nelp`纪录我们已经加入了一个元素--`binary()`和标准函数`bsearch()`只需要地址而不是变量的值包含饿了表中元素的个数。

> `memmove()`复制字节即使源和目标区域重叠；`memcpy()`并不如此，但是它更具效率

给出一个通用搜索和入口的方式，我们可以很轻易的管理我们的符号表。首先我们需要比较一个 key 和表中的元素：

```cpp
static int cmp (const void * _key, const void * _elt) {
    const char * const * key = _key;
    const struct Name * const * elt = _elt;

    return strcmp(* key, (* elt) -> name);
} 
```

作为一个键值，我们只传递一个指向输入符号文本的指针的地址。表中的元素当然是`Name` 结构体，并且我们只查看他们的`.name`成员。

搜索和入口通过适用适当的参数对`binary()`进行调用来实现。由于我们事先不知道符号个数，我们确保一直有空间让我们扩招表：

```cpp
static struct Name ** search (const char ** name) {
    static const struct Name ** names;    /* dynamic table */
    static size_t used, max;

    if (used >= max) {
        names = names
            ? realloc(names, (max *= 2) * sizeof * names)
            : malloc((max = NAMES) * sizeof * names);
        assert(names);
    }
    return binary(name, names, & used, sizeof * names, cmp);
} 
```

`NAMES`是一个定义的常量具有初始分配的表项；每次我们用完，我们都是表的大小加倍。

`search()`适用指向要查找的文本的地址指针作为参数并且返回表项的地址。如果文本未找到，`binary()`就插入`key`--也就是说，只有指向文本的指针，而不是一个`struct Name`--到表中。这个策略是为了`screen()`的利益，它只新建一个表元素，如果一个输入中的标识符是未知的：

```cpp
int screen (const char * name) {
    struct Name ** pp = search(& name);

    if (* pp == (void *) name)    /* entered name */
        * pp = new(Var, name);
    symbol = * pp;
    return (* pp) -> token;
} 
```

`screen()`让`search()`查找要显示的输入符号。如果指符号文本的指针被插入符号表，我们需要使用一个新标识符的项目描述替换它。

对于`screen()`，一个新的标识符必须是一个变量。我们假定这里有一个类型描述`Var`知道如何构建`Name`结构体来描述变量并且我们让`new()`做剩下的工作。其他的情况，我们让`symbol`指向符号表项并且返回它的`.token`值。

```cpp
void install (const void * np) {
    const char * name = ((struct Name *) np) -> name;
    struct Name ** pp = search(& name);

    if (* pp != (void *) name)
        error("cannot install name twice: %s", name);
    * pp = (struct Name *) np;
} 
```

`install()`比较简单。我们接受一个`Name`对象并且让`search()`在符号表中找到它。`install()`被假定来只处理新符号，所以我们应当总是能够插入对象替换它的名字。否则，如果`search()`真的找到一个符号，我们就有麻烦了。

## 子类的事先--Var

`screen()`调用`new()`来创建一个新的变量符号并且返回它到识别器，并插入它到一个表达式树中。因此，`Var`必须创建可以项节点行为的符号表项，也就是说，当定义`struct Var`的时候，我们需要扩展一个`struct Name`来继承在符号表中存在的能力并且我们必须支持动态绑定的函数可以适用于表达式节点。我们描述接口在`Var.h`中:

```cpp
const void * Var;
const void * Assign; 
```

一个变量具有一个名字和一个值。如果我们计算一个算术表达式的值，我们需要返回`.value`成员。如果我们删除一个表达式，我们一定不能删除变量节点，因为它存活在符号表中：

```cpp
struct Var { struct Name _; double value; };
#define value(tree) (((struct Var *) tree) -> value)

static double doVar (const void * tree) {
    return value(tree);
}

static void freeVar (void * tree) {
} 
```

就如在 4.6 节中讨论的，通过提供一个值的访问函数来简化代码。

创建一个变量需要分配一个`struct Var`，插入一个变量名的动态副本，并且标识值`VAR`被识别器规定：

```cpp
static void * mkVar (va_list ap) {
    struct Var * node = calloc(1, sizeof(struct Var));
    const char * name = va_arg(ap, const char *);
    size_t len = strlen(name);

    assert(node);
    node-> _.name = malloc(len + 1);
    assert(node -> _.name);
    strcpy((void *) node-> _.name, name);
    node -> _.token = VAR;
    return node;
}

static struct Type _Var = { mkVar, doVar, freeVar };

const void * Var = & _Var; 
```

`new()`照料插入`Var`类型描述到节点中，在符号被`screen()`返回之前或者任何的使用。

就技术而言，`mkVar()`是`Name`的构建子。然而，只有变量名需要被动态存储。因为饿哦我们决定在我们的计算器中构建子负责分配一个对象，我们不能让`Var`构建子调用一个`Name`构建子来维护`.name`和`.token`成员--一个`Name`构建子将会分配一个`struct Name`而不是一个`struct Var`。

## 赋值

赋值是一个二元操作。识别器保证我们具有一个变量作为做操作数和`sum`作为右操作数。因此，我们世纪需要实现的是实际赋值操作，也就是说，动态绑定进类型描述的`.exec`成员：

```cpp
#include "value.h"
#include "value.r"

static double doAssign (const void * tree) {
    return value(left(tree)) = exec(right(tree));
}

static struct Type _Assign = { mkBin, doAssign, freeBin };
const void * Assign = & _Assign; 
```

我们共享`Bin`的构建子和析构子，因此，在算数操作的实现中必须是全局的。我们也共享`struct Bin`和访问函数`left()`和`right()`。所有这些使用`value.h`导出并且实现文件`value.r`。我们自己的访问函数`value()`对于`struct Var`故意的允许修改，如此赋值就可以被很优雅的实现。

## 另一个子类--常量

谁会喜欢输入`pi`或者其他数学常量的值呢？我们从 Kernighan 和 Pike's hoc [K&P84]得到线索并且预定义一些常量给我们的计算器。下面的函数需要被调用在初始化识别器期间：

```cpp
void initConst (void) {
    static const struct Var constants [] = {    /* like hoc */
        { &_Var, "PI", CONST, 3.14159265358979323846 },
        ...
        0 };
    const struct Var * vp;

    for (vp = constants; vp -> _.name; ++ vp)
        install(vp);
} 
```

变量和常量几乎是一样的：都具有名称和值并且存活在符号表中；都返回他们的值在一个算数表达式的使用中；并且都不应当被删除，当我们删除一个算数表达式的时候。然而，我们不应当给常量赋值，所以我们需要同意一个新的标识符值`CONST`，识别器在`factor()`中接受就像`VAR`一样，但是不允许在`stmt()`的赋值的左边。

## 数学函数--Math

ANSI-C 定义了许多数学函数例如`sin()`,`sqrt()`,`exp()`等等。作为另一个继承的练习，我们将添加库函数使用一个单个`double`参数并且具有一个`double`结构到我们的计算器。

这些函数工作的就如同一元运算符一样。我们可以定义一个新的类型给节点给每个函数并且收集大多数功能从`Minus`和`Name`类，但是这里有一个更简单的方法。我们扩展`struct Name`到`struct Math`如下：

```cpp
struct Math { struct Name _;
    double (* funct) (double);
};
#define funct(tree) (((struct Math *) left(tree)) -> funct) 
```

额外的给函数名称用于输入和标识符给识别，我们存储像`sin()`的库函数的地址在符号表项中。

在初始化期间我们调用下面的函数来输入所有的函数描述到符号表中：

```cpp
#include <math.h>

void initMath (void) {
    static const struct Math functions [] = {
        { &_Math, "sqrt", MATH, sqrt },
        ...
        0 };
    const struct Math * mp;

    for (mp = functions; mp -> _.name; ++ mp)
        install(mp);
} 
```

一个函数调用是一个因子就好像使用一个减号标记一样。对于识别我们需要扩展我们的语法对因子：

```cpp
factor : NUMBER
     | - factor
     | ...
     | MATH ( sum ) 
```

`MATH`是公共标识符对所有函数输入通过`initMath()`。这个翻译到下面的附加`factor()`在识别器中：

```cpp
static void * factor (void) {
    void * result;
    ...
    switch (token) {
        case MATH:
        {
            const struct Name * fp = symbol;

            if (scan(0) != '(')
                error("expecting (");
            scan(0);
            result = new(Math, fp, sum());
            if (token != ')')
                error("expecting )");
            break;
        } 
```

`symbol`首先包含符号表元素对一个函数例如`sin()`。我们保存这个指针并且构建表达式树对于函数参数通过调用`sum()`。然后我们使用`Math`，类型描述给函数，并且让`new()`构建下面的节点给表达式树：

我们让一个二元节点的左边只想符号表元素给函数并且我们附加参数树在右边。这个二元节点具有`Math`作为类型描述，也就是说，方法`doMath()`和`freeMath()`将会被调用来分别执行和删除节点。

Math 节点仍然使用`mkBin()`构建，因为这个函数不关心指针的后代。`freeMath()`，然而，可能只会删除右子树：

```cpp
static void freeMath (void * tree) {
    delete(right(tree));
    free(tree);
} 
```

如果我们仔细看上图，我们可以看到一个`Math`节点的执行是非常容易的。`doMath()`需要调用存储在符号表中元素可以被访问的作为左后代二元节点从之被调用：

```cpp
static double doMath (const void * tree) {
    double result = exec(right(tree));

    errno = 0;
    result = funct(tree)(result);
    if (errno)
        error("error in %s: %s",
            ((struct Math *) left(tree)) -> _.name,
            strerror(errno));
    return result;
} 
```

唯一的问题是抓住数字错误通过检测`errno`变量在 ANSI-C 头文件`errno.h`中声明。这个完成了数学函数的实现给计算器。

## 总结

机遇一个函数`binary()`来搜索和插入一个有序数组，我们实现了一个符号表包含了就诶够体具有名称和标识符值。继承允许我们插入其他结构体到表中而不需要改变函数搜索和插入。这种方式的高雅变得明显一旦我们考虑一个传统的定义一个符号表元素出于我们的目的：

```cpp
struct {
    const char * name;
    int token;
    union {            /* based on token */
        double value;
        double (* funct) (double);
    } u;
}; 
```

对于关键字，`union`是没有必要的。用户定义的函数讲会要求一个更详细的描述，并且引用`union`的部分是讨厌的。

继承允许我们适用符号表功能到新的项而不改变已经存在的代码。动态绑定在许多方式帮助保持实现的简单性：常量符号表元素，变量和函数可以被绑定进表达式树中而不用担心我们不慎删除他们；一个执行函数参考自身值有它自己安排节点。

## 练习

新关键字是必须的用来实现例如`while`或者`repeat`循环，`if`语句等等。识别被`stmt()`处理，但是这个对于大部分情况，只有一个问题编译器构建，而不是继承。一旦我们决定语句描述，我们将创建节点类型例如`While`，`Repeat`或者`IfElse`，并且关键字在符号表中不需要知道他们的存在。

更有趣的是函数具有两个参数的例如`atan2()`在 ANSI-C 数学库中。从这里看出符号表，这个函数被处理仅仅类似简单函数，但是对于表达式树我们需要开发一个新的节点类型使用三个后代。

用户定义的函数具有一个现实的有意思的问题。如果我们表示单独的参数通过`$`并且我们使用一个节点类型`Parm`来只想函数项在符号表中从那里我们可以暂时存储参数值只要我们不允许递归，就是简单的。函数具有参数名和几个参数是比较困难的，当然了。然而，这是一个好的练习`inverstigate`继承的好处和动态绑定的好处。我们将在第十一章中返回到这个问题。