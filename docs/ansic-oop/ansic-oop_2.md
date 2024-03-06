# 第一章 抽象数据类型——信息隐藏

# 第一章 抽象数据类型——信息隐藏

> 来源：[`blog.csdn.net/besidemyself/article/details/6376408`](http://blog.csdn.net/besidemyself/article/details/6376408)
> 
> 译者：[besidemyself](http://my.csdn.net/besidemyself)

## 1.1 数据类型

数据类型是每种编程语言不可或缺的一部分。ANSI-C（标准化 C）拥有一些基本数据类型：`int` ，`double`和`char`。有限的数据类型几乎不能满足程序员的要求，所以编程语言会提供一种机制来使得程序员使用这些基本的预定义数据类型构造新的数据类型。一个简单的应用就是构造集合，如数组，结构体和联合体。而指针集，依照 C.A.R Hare 的话：“从这一步起，我们也许永远不会复苏”允许我们描述和操作本质上无限复杂的数据。

什么才是真正的数据类型呢？我们可以发表不同的观点。数据类型是一系列值的集合——`char`(字符型)数据类型拥有 256 个不同的值，`int`（整型）数据类型拥有更多不同值， 他们之间的间隔相等，表现形式多少有点像数学中的自然数或整数，而`double`(浮点数据类型)类型拥有更多的可能的值，类似数学中的带小数部分的数。

有选择性地，我们能够定义一种数据类型作为一个值的集合加上一系列操作来做一些事情。典型地，定义的这些值计算机能够表示，这样的操作过程能够翻译成机器指令。在这方面，`int` 型在标准 C 语言中做的并不是很好。这些数据的集合的值的范围可能随着不同机器而不同，操作方式就像算术中的右移操作，表现形式可能不尽相同。

太复杂的例子往往不能得到有效的说明。我们可以典型地把一个线性列表中的元素定义成一个结构体如下：

```cpp
typedef  struct node {
    struct node *Next;
    …information…
}node ; 
```

并且，对于这个列表的操作，我们指定列表的头，如下：

```cpp
node * head (node * elt , const  node *  tail); 
```

然而，这样的应用是非常冗余的，好的编程规则指示我们隐藏数据项的表示，仅仅声明操作方法。

## 1.2 抽象数据类型

如果我们不把对这个数据类型的表示呈现给用户，则我们称这个数据类型为抽象数据类型。从理论的角度，要求我们通过包含可能性操作的数学表达式中指定数据类型的属性。例如，我们从队列中删除一个我们先前增加的元素，并且可以从队列中以相同的次序检索我们增加的元素。

抽象数据类型为程序员提供了很强的便利性。因为表达式不是定义的一部分。我们可以自由的选择更简单，更有效的方式去实现。如果我们能够正确的分离出必要的信息，那么对数据类型的使用和实现将完全的独立开来。

抽象数据类型满足了“信息隐藏”和“分而治之”的良好编程规则。例如数据项表达式—— 只给需要知道的人提供，给抽象数据类型的实现者，而不是用户。通过使用抽象数据类型，我们可以清晰的分离出实现者和使用者的不同任务。并且可很好的把一个大的系统分解成各个小的模块。

## 1.3 举例——集合

由此，我们怎样的实现一个抽象数据类型呢？我们以对一个集合中的元素操作为例，使用操作方法，`add`（增加），`find`（查找），和`drop`（删除）。这些方法均用于集合和集合中的元素。`add` 方法向集合中添加一个元素，并返回要添加的元素，`find`方法从集合中查找指定的元素，可被用于实现来判断一个指定的元素是否在集合中，`drop`方法从集合中删除一个元素。

使用这种方式，即可看到`set`(集合)是一个抽象的数据类型。现在声明一下我们想要做的，以一个头文件`Set.h` 开始：

```cpp
#ifndef __USR_SET_H__
#define __USR_SET_H__
extern const void * Object;
void* add(void* set,const void* element);
void* find(const void* set,const void *element);
void* drop(void *set,const void * element);
int contains(const void* set,const void* element);
#endif 
```

前两句的作用可使编译器对这段声明加以保护处理。无论头文件`Set.h` 被包含多少次，C 编译器只对这个声明编译一次。这样的声明头文件的方法是很标准的，GNU C 预处理器能够识别，而且当保护符号（如上的`__USR_SET_H__`） 被定义，则保证不会再进入保护区声明的代码。

`Set.h` 很完整，但它真的很有用吗？ 我们几乎不可能发觉和想象出它的不足：`Set`（集合）理所当然代表一个实例，我们可以使用这个实例来做很多事情。`Add()` 方法传递一个元素，并把它添加到集合，并且返回所添加的元素或集合中已经存在的元素；`find()` 在指定的集合中查找元素，并且返回找到的元素，若没有找到，则返回`NULL`（空）；`drop()` 定位一个元素，把这个元素从集合中删除，并且返回删除的元素；`contains()` 的本质就是把`find()` 方法所查找的结果转换为“真”值。

通用指针类型 `void*` 的应用贯穿全文。一方面它使得我们想发现集合到底是什么东西成为不可能。但是另一方面它允许我们向如`add()` 和其他方法中传递任意类型的数据。并不是每件事都会拥有像集合和集合中的元素一样的表现形式——在信息隐藏的乐趣中我们牺牲了类型的安全性。然而，我们可以在第八章看到这样的应用会非常之安全。

## 1.4 内存管理

也许我们已经瞥见了某些东西：怎样的获得一个集合呢？`Set`（集合）是一个指针，并不是被`typedef` 关键字定义的类型；因此我们不能把`Set`定义成一个局部或全局的类型。相反的我们只是使用指针来引用集合和集合中的元素，并且建立一个文件`new.h`，并声明如下：

```cpp
void * new (const void * type, ...);
void delete (void * item); 
```

就像`Set.h` 文件一样的做法，文件被预处理器符号`NEW_H`保护起来。以后只列出感兴趣的部分，所有的源代码和所有实例的代码均能在光碟中找到。

`new()` 接收一个像`Set`的描述符,传递更多可能的参数用于初始化操作，返回一个指向新数据项的携带描述符信息的指针。`delete()` 接受一个由`new()` 所原先产生的指针，并回收关联的资源。

`new()` 和 `delete()` 可看成类似于标准Ｃ函数`calloc()` 和`free()` 。如果的确是，描述符得能够指示出至少需要申请多大的内存空间。

## 1.5 （`Object`）对象

如果我们想搜集在集合感兴趣的东西，我们需要另外一个抽象数据类型`Object` ,在头文件`Object.h` 中有如下描述：

```cpp
extern const void * Object; /* new(Object); */
int differ (const void * a, const void * b); 
```

`differ()` 是用来做对象比较的：即若两个对象不相等则返回真，否则返回假。这样的描述为 C 语言函数`strcmp()` 留有余地：因为在某些比较中我们也许选择返回一个整数或负数的值来指示排列的次序（正序或倒叙）。

现实生活的对象需要更多的功能去做有用的事情。此刻，我们约束我们自己只对集合中的成员（必须品）操作而已。如果我们建立一个更大的类库，我们将看到所谓的集合 — 实际，包括其他所有东西 — 均是一个对象。从这个观点出发，很多的对象包含的功能其实都是无条件存在的。

## 1.6 应用

包含头文件，和库信息，抽象数据类型，我们能够写一个`main.c` 的应用程序如下所示：

```cpp
#include <stdio.h>
#include "New.h"
#include "Object.h"
#include "Set.h"

int main() {
       void* s=new(Set);
       void* a=add(s,new(Object));
       void* b=add(s,new(Object));
       void* c=new(Object);

       if(contains(s,a)&&contains(s,b)){
              puts("ok");
       }

       if(contains(s,c)){
              puts("contains?");
       }

       if(differ(a,add(s,a))){
              puts("differ?");
       }

       if(contains(s,drop(s,a))){
              puts("drop?");
       }

       delete(drop(s,a));
       delete(drop(s,a));

       return 0;
} 
```

我们创建了一个结合并给集合中添加了两个新建的对象。如果不出意外，我们可以发现对象会在集合中，并且我们不会再发现其他的新对象。程序的运行会简单得打印出 ok 。

对 `differ()` 的调用会证明出这样的语义：数学上的集合只包含集合 `a` 的一份拷贝；对一个元素的重复添加必须返回已经加入的对象，因此上述程序的 `differ()` 为 假。相似的，一旦我们从一个结合删除一个元素，它将不会再存在于这个集合中。

从一个集合中删除一个不存在的元素将返回一个空的指针传递给 `delete()` 。现在，我们已经指示出 `free()` 的语义且必须是合情合理可接受的。

## 1.7 一种实现机制——`Set`（集合）

`main.c` 会编译成功，但在连接和执行之前，我们必须实现其中的抽象数据类型和内存管理。如果一个对象不存储信息，且每个对象最多属于一个结合，我们可以把每个对象和集合当成，小的，独立的，正整数的值。可在 `heap[]` 中通过数组下标来索引到。如果一个对象（这里的对象为数组元素的地址）是一个集合的成员，则数组元素的值代表这个集合。对象指向包含它的集合。

首先的解决方案是非常简单的，即我们把其他模块与`Set.c` 相结合。集合对象集有相似的呈现方式。因此，对于 `new()` 无需关注类型描述。它仅仅从数据 `heap[]` 中返回非零元素。

```cpp
void * new (const void * type, ...) {
    int *p ;    /*&heap[1...]*/
    for(p=heap+1;p<heap+MANY;++p){
        if(!*p){/*若 heap 中的某个元素为 0，则返回这个指针，并把值设为 MANY*/
            break;
        }
    }
    assert(p<heap+MANY);
    *p=MANY;   
    return p;
} 
```

我们是用`0`来标记 `heap[]` 中有效的元素；因此不能返回`heap[0]` 的引用——如果它是一个集合，而集合的元素可以是索引值为`0`的对象。

在一个对象被添加到集合当中之前， 我们让它包含无效的索引`MANY`，以便于`new()` 不会再次返回它，请不能误解 `MANY` 是集合的一个成员。

`new()` 能够使用完内存。这是很多“致命性错误”的其中一个。我们可以简单的使用标准化Ｃ语言宏的宏 `assert()` 来标记这些错误。一个更理想的实现方式是至少会打印合理的错误信息或使用用户可重写的错误处理机制的通用功能。这也是我们的目的中，编码技术完整性的一部分。在第十三章，我们会介绍一种通用异常处理的技术。

`delete()` 必须得严加防范空指针的传入。通过设置其元素的值为`0` 来进行 `heap[]` 中元素的回收。

```cpp
void delete (void * _item) {
    int* item=_item;
    if(item){
        assert(item>heap && item,heap+MANY);
        *item=0;
    }
} 
```

我们需要统一的处理通用指针；因此，给每个通用指针的变量的前面加上下划线前缀，然后仅仅使用它初始化指定类型的局部变量。

一个集合被它的对象所表示。集合中的每个元素指向它的集合。如果元素包含 `MANY` ，则它可以被添加到一个集合中。否则它已经属于一个集合的元素了。因为我们不允许一个对象属于多个集合。

```cpp
void* add(void* _set,const void* _element);
{
    int * set=_set;
    const int *element=_element;

    assert(set>heap && set<heap+MANY);
    assert(*set==MANY);
    assert(element>heap&& element<heap+MANY);

    if(*element==MANY){
        *(int*)element=set-heap;
    }
    else{
        assert(*element==set-heap);
    }
    return (void*) element;
} 
```

`assert()` 在这里稍微显得逊色：我们只关注在 `heap[]` 内的指针和集合不属于其他部分的集合，等等，数组元素的值应该为 `MANY`。

其他的功能都是很简单的。`find()` 只查找元素的值为集合索引的元素。若找到，返回元素，否则返回`NULL`。

```cpp
void* find(const void* _set,const void * _element) {
    const int* set=_set;
    const int* *element=_element;
    assert(set>heap && set<heap+MANY);
    assert(*set==MANY);
    assert(element>heap && element<heap+MANY);
    assert(*element);
    return *element==set-heap?(void*)element:0;
} 
```

`contains()` 把 `find()` 的结果转换为真值：

```cpp
int contains(const void* _set,const void* _element) {
    return find(_set,_element)!=0;
} 
```

`drop()` 依赖于`find()` 的结果，若在集合中查找到，则把此元素的值标记为`MANY`，并返回此元素：

```cpp
void* drop(void * _set,const void * _element) {
    int* element=find(_set,_element);
    if(element){
        *element=MANY;
    }
    return element;
} 
```

如果我们深入挖掘，一定会坚持被删除的元素要不包含于其他集合中。在这种情况下，毫无疑问会在 `drop()` 中复制更多 `find()` 的代码。

我们的实现是很非传统的。在实现一个集合时似乎不需要 `differ()` 。我们仍然提供它，因为我们的程序要使用这个函数。

```cpp
int differ (const void * a, const void * b) {
    return a!=b;
} 
```

当数组中对象的索引不同时，这个对象必然是不同的，也就是索引值就能区分它们的不同，但一个简单的指针比较已经足够了。

我们已经做完了——对于这个问题的解决我们还没有使用描述符 `Set` 和 `Object` ，但是不得不定义它以使我们的编译器能通过。

```cpp
const void * Set;
const void * Object; 
```

我们在 main() 函数中使用上述指针来创建集合和对象。

## 1.8 另一种实现——包

不需要改变`Set.h` 中的接口，我们来改变接口的实现方式。这次使用动态内存分配，使用结构体来表示集合和对象：

```cpp
struct Set{
       unsigned count;
};
struct Object{
       unsigned count;
       struct Set* in;
}; 
```

`count` 用于跟踪集合中的元素的计数个数。对于一个元素来说，`count` 记录这个元素被集合添加的次数。如果我们想递减`count` 值，可调用 `drop()` 方法。一旦一个元素的`count` 值为`0`，我们就可以删除它，我们拥有一个包，即，一个集合，集合中的元素拥有一个对`count` 的引用。

因为我们使用动态内存分配机制去表示集合集和对象集，所以需要初始化`Set` 和 `Object` 描述符，以便于`new()` 能够知道需要分配多少内存：

```cpp
static const size_t _Set=sizeof(struct Set);
static const size_t _Object=sizeof(struct Object);

const void * Set=&_Set;
const void * Object=&_Object; 
```

`new()` 方法现在更加简单：

```cpp
void * new (const void * type,...) {
       const size_t size= *(const size_t*)type;
       void* p=calloc(1,size);
       assert(p);
       return p;
} 
```

`delete()` 可直接把参数传递给 `free()`——标准化 C 语言中 一个空的指针可以传进 `free()` 。如下：（如意调用）

```cpp
void delete (void * _item) {
       free(_item);
} 
```

`add()` 方法多多少少对它的指针自变量比较信任。它会增加元素的引用计数和集合的引用计数。

```cpp
void* add(void* _set,const void* _element) {
       struct Set *set=_set;
       struct Object* element=(void*)_element;

       assert(set);
       assert(element);

       if(!element->in){
              element->in=set;
       }
       else{
              assert(element->in==set);
       }

       ++element->count;
       ++set->count;

       return element;
} 
```

`find()` 方法仍然会检查，一个元素是否指向一个适当的集合：

```cpp
void* find(const void* _set,const void * _element) {
       const struct Object* element=_element;

       assert(element);

       return element->in==_set?(void*)element:0;
} 
```

`contains()` 方法基于`find()` 方法来实现，仍然保持不变。

若 `drop()` 在集合中找到它要操作的元素，它将递减元素的引用计数和元素在集合中的计数。如果引用计数减为 0，这个元素即被从集合中删除：

```cpp
void* drop(void * _set,const void * _element) {
       struct Set* set=_set;
       struct Object* element=find(set,_element);

       if(element){
              if(--element->count==0){
                     element->in=0;
              }
              --set->count;
       }
       return element;
} 
```

现在我们可以提供一个新的方法，用来获取集合中的元素个数：

```cpp
unsigned count(const void* _set) {
       const struct Set* set=_set;

       assert(set);
       return set->count;
} 
```

当然啦，直接让程序通过读 `对象 .count` 显得比较简单，但是我会坚持不去披露集这样的实现。与应用程序重写临界值的危险性相比上述功能的调用的开销是可忽视的。

包的表现与集合是不同的。一个元素可被添加多次；当一个元素的删除次数等于其被添加的次数时，这个元素被从集合中删除，`contains()` 方法仍然能够找着它。测试程序的运行结果如下：

```cpp
ok
drop? 
```

## 1.9 总结

对于抽象数据类型，我们完全隐藏了其实现的细节，例如应用程序代码中数据项的描述。

程序代码只访问头文件，在头文件中描述符指针表示数据类型，对数据类型的操作作为一种方法被声明，此方法接收和返回通用指针。

描述符指针被传进通用方法 `new()` 中去获得一个指向数据项的指针，这个指针被传进通用方法 `delete()` 中去回收关联的资源。

通常情况，每个抽象数据类型被在单独的源文件中实现。理想情况下，它不对其他数据类型描述。这个描述符指针正常情况下至少指向一个固定大小的值来指示需要的数据项空间大小。

## 1.10 练习

略。