# 第二章 动态链接和泛函数

> 来源：[`blog.csdn.net/besidemyself/article/details/6387915`](http://blog.csdn.net/besidemyself/article/details/6387915)
> 
> 译者：[besidemyself](http://my.csdn.net/besidemyself)

## 2.1 构造器和析构器

让我们来实现一个简单的字符串数据类型，这个数据类型将在接下来的集合中用到。对于新的字符串，我们分配一个动态缓存来保存字文本。当这个字符串被删除时，我们将回收其所占用的内存缓冲。

`new()` 负责创建一个对象，`delete()` 必须回收这个对象所占用的资源。 `new()` 预先知道它所创建的资源的类型，因为它的第一个参数将传递对这个对象的描述。基于这个参数，我们可以使用一系列的判断语句`if` 来处理单个不同的要创建的对象。这样做的缺点是 `new()` 得完全包含对每个支持的对象要处理的代码。

现在 `new()` 拥有一个更大的问题。它负责创建对象并且返回对象的指针，这个指针被传递到`delete()` ，也就是说，`new()` 必须在每个对象中安装特定的析构器信息。最明显的应用是使用一个指针，指向特定的析构器，这个析构器是类型描述符的一部分，被传进 `new()` 。到目前为止，我们需要像如下的声明:

```cpp
struct type{
    size_t size;          /*size of an object*/
    void (*dtor)(void*);  /*destructor*/
};
struct String{
    char *text;           /*dynamic string*/
    const void* destroy;  /*locate destructor*/
};
struct Set{
    ...information...
    const void * destroy; /*locate destructor*/
}; 
```

似乎我们又有另外一个问题：在新的对象中，有人需要把析构器指针 `dtor`从类型描述中拷贝到 `destroy` 中。而且这样的拷贝在不同的对象的类中会放到不同的位置。

初始化工作是 `new()` 的一部分，不同的类型会要求`new()` 干不同的工作——`new()` 可能需要不同的参数来处理不同的类型：

```cpp
new(Set);           /*make a set*/
new(String,"text"); /*make a string*/ 
```

对于初始化我们使用另外一个指定类型的功能函数，这个函数我们称它为构造器。因为构造器和析构器都是类型指定的，不需要改变，我们把他们当成类型描述的一部分传递给 `new()` 函数。

注意，对于一个对象自身来说构造器和析构器并不是用来担当获取和释放内存的责任——这是`new()` 和 `delete()` 的工作。构造器被 `new()` 调用仅仅用来初始化 `new()` 所分配的内存。对于一个字符串，这需要涉及需要另外一块内存存储文本，但是对于`struct String` 自身的内存却是使用 `new()` 来分配的。这个空间接下来会被`delete(`) 所释放。然而 `delete()`首先 会调用析构器，析构器是对构造器所做的初始化进行反向操作。这步完成后才调用`delete()` 释放`new()` 所分配的内存。

## 2.2 方法，消息，类和对象

`delete()` 必须能够在不知道对象类型的情况下定位析构器。因此修正了 2.1 部分的声明，我们必须坚持指针被用来定位析构器，这个指针必须放到所有对象的开始处传进`delete()` 中，而不管他们的类型是什么。

这个指针应该指向什么呢 ？如果所有我们所拥有的是一个对象的地址，那么对于一个对象来说，指针给了我们访问指定类型的对象信息，就像对象的析构函数一样。似乎很可能我们将要创造一个类型指定的功能如一个函数用来显示对象，或者一个对象的比较函数`differ()` ，或一个函数`clone()` 去创建一个对象的完全拷贝。因此我们将使用一个指针指向函数指针表。

仔细看，我们会意识到这个表必须是类型描述的一部分，被传进 `new()` , 显然的解决问题的答案就是让对象指向一个类型的完全描述：

```cpp
struct Class {
       size_t size;
       void * (* ctor) (void * self, va_list * app);
       void * (* dtor) (void * self);
       void * (* clone) (const void * self);
       int (* differ) (const void * self, const void * b);
};
struct String {
       const void * class;  /* must be first */
       char * text;
};
struct Set{
    const void* class   /*must be first*/
        ...
}; 
```

每个对象将以一个指针开始，这个指针向对象的类型描述表，通过这个类型描述表，我们就可以定位一个对象的类型指定信息：`.size` 是`new()` 所分配的对象所占用内存的大小；`.dtor` 指向被`delete()`（`delete` 用来销毁对象） 所调用的析构器；而 `.differ` 指向一个函数，这个函数用于比较对象。

继续往下看，我们会注意到，每个功能都是以对象而存在的，通过对象来选择这些功能。只有构造函数要处理部分初始化内存区域工作。我们都叫这些功能为一个对象的方法。调用一个对象的方法就是处理一则消息，我们使用参数`self` 来标记消息接收的对象。因为我们使用基本的Ｃ函数功能，`self` 是不需要作为第一个参数而传进的。

多个对象共享相同的类型描述符，也就是说，他们需要相同数量大小的内存空间，可用于相同的方法。我们称所有拥有相同的类型描述符的对象为一类；单独的对象被称为类的实例。到目前为止，一个类，一个抽象数据类型，可能的值与操作结合的集合，即，一个数据类型，这些是极其相似的。

一个对象是一个类的实例，也就是说，它拥有一个描述，这个描述被`new()` 所分配的内存所指示，并且这个描述被类的方法操作。普遍来说，一个对象是特殊数据类型的值。

## 2.3 选择器，动态链接，多态

谁来邮递消息呢？ 构造器被 `new()` 所调用，对于大多数内存区域是不被初始化的:

```cpp
void * new (const void * _class, ...) {
    const struct Class * class = _class;
       void * p = calloc(1, class -> size);

       assert(p);
       * (const struct Class **) p = class;

       if (class -> ctor)
       {    
        va_list ap;

              va_start(ap, _class);
              p = class -> ctor(p, & ap);
              va_end(ap);
       }
       return p;
} 
```

在一个对象的起始地方，`struct Class` 指针的存在是极其重要的。这也是我们在 `new()` 中初始化它的原因：

如上图右边的类型描述 `class` 在编译的时候已经被初始化。对象是在运行时被创建的，接下来图中的虚线关联才被插入。在语句：

```cpp
* (const struct Class **) p = class; 
```

中，`p`指向对象的内存区域的起始位置。我们对`p`进行了强制类型转换，`p`把对象的起始位置当成一个指针，指向`struct Class`，即把参数`class` 设置为这个指针的值。

接下来，若构造器是类型描述的一部分，我们调用它，并把其返回值做为 `new()` 的结果，即作为一个新的对象返回。2.6 部分列出一个很聪明的构造器，由于它聪明，所以能够对它自己的内存管理作出决策。

注意啦，只有明确的可见函数如 new() 能拥有可变的参数列表。参数列表被 `va_list` 的变量 `ap` 所访问，`ap` 被一个宏 `va_start()` 初始化，这个宏在`stdarg.h` 头文件。`new()`仅仅能够把整个参数列表传进构造器中；因此，`.ctor` 也被声明成拥有 `va_list` 的参数，而不是它私有的参数列表。由于我们接下来要在好多函数中共享源参数列表，因此我们只传递 `ap`的地址到构造器中——当它返回时，`ap` 指向参数列表的第一个参数，而参数列表本身不会被改变。

`delete()` 假设每个对象，也就是说，每个非空指针，指向一个类型描述。如果类型描述的析构器存在，则调用它。这里，`self` 扮演前面 `p` 的角色。我们使用局部变量`cp` 来进行强制类型转换，并从`self` 中获得我们所需要的信息。

```cpp
void delete (void * self) {    
    const struct Class ** cp = self;

       if (self && * cp && (* cp) -> dtor){
              self = (* cp) -> dtor(self);
       }
       free(self);
} 
```

析构器，在上述`delete()`中，也会获得一次把他的返回值传进`free()` 的机会，如果构造器试着去欺骗，则析构器会有更改的机会，参看 2.6 部分。如果一个对象在调用`delete()` 的时候不想被删除，则可在他的析构器中返回一个空指针。

所有其他的方法都存储在类型描述中，并以相似的方式被调用。在每个例子中，我们有一个单独的接收对象`self` 且我们通过它来路由我们的方法调用。

```cpp
int differ (const void * self, const void * b) {    
    const struct Class * const * cp = self;

       assert(self && * cp && (* cp) -> differ);
       return (* cp) -> differ(self, b);
} 
```

最关键的部分，当然是一个假设，假设我们能够找到一个类型描述指针 `*self` ，而这个`*self` 会隐藏在任意的指针`self`下面。此时此刻，至少，我们会对空指针很警惕。在每个类型描述的起始，我们将存放一个“魔法数字”，或甚至把地址或所有已知类型的地址范围与 `*self` 相比较，但是，在第八章会看到，我们将做更严格的检查。

不管怎么说，`differ()` 列举出了函数调用技术怎么被动态链接或后期链接调用的原因：即只要我们能够在一开始拥有一个正确的类型描述指针，那么我们就可以对任意的对象使用`differ()` 调用。这个函数实际上被调用的时机是尽可能的晚的——即仅仅在实际执行期间调用，而不是之前调用。

我们可以称`differ()` 为一个选择器。它是多态功能的一个例子，也就是说，一个函数能够接受不同的参数类型，且表现不同，并且这种现象是基于他们的参数类型。一旦我们实现了更多的类时，这些类在他们的描述符中都包含 `.differ` ，则可称 `differ()` 为一个泛函数，且在这些类中能够被应用于任何对象。

我们可以把这个选择器当成方法，方法自己本身不会动态链接，但仍然能够像多态函数一样的表现，因为它能让动态的连接的函数做他们真实的事情。

多态机制实际已经嵌入到很多编程语言中，例如：如在 Pascal（一种编程语言）中，`write()` 函数会根据参数类型不同进行不同的处理。在 C++中，操作符 + 如果被不同的类型值如整型，指针，浮点指针调用，将产生不同的结果。这个现象被称作重载，即：参数类型和操作符名结合起来决定操作结果。相同的操作符与不同的参数类型结合将产生不同的响应。

这里并没有明显的差异。因为动态连接，`differ()` 的表现更像一个重载函数，而且 C 的编译器也能够使得 + 看起来像多态函数——至少对于内嵌的数据类型来说。然而，C 编译器能够根据对 + 操作符的不同使用而产生不同的返回类型，但是函数`differ()` 依靠它的参数类型只能返回相同的类型。

很多方法在不需要动态连接的情况下能够实现多态。例如，函数 `sizeOf()` 返回任意类型的对象的大小。

```cpp
size_t sizeOf (const void * self)
{    
    const struct Class * const * cp = self;

       assert(self && * cp);
       return (* cp) -> size;
} 
```

所有的对象都携带它们的描述符，我们可以使用描述符来获得对象的大小。注意如下的不同之处：

```cpp
void* s=new(string, "text");
assert(sizeof s!=sizeOf(s)); 
```

`sizeof` 是 C 语言的操作符，用于在运行时以字节的个数返回参数的大小。而 `sizeOf()` 是我们实现的多态函数，它的参数指向一个对象，返回在运行时对象所占用的字节大小。

## 2.4 应用

然而我们还没有实现一个字符串类，我们仍然做好了一个简单的测试程序的准备。`String.h` 定义了抽象数据类型：

```cpp
extern const void * String; 
```

对于所有的对象，我们的方法都是相似的。我们向内存管理头文件`new.h` 中增加在 1.4 部分介绍的声明：

```cpp
void * (* clone) (const void * self);
int (* differ) (const void * self, const void * b);

size_t sizeOf(const void* self); 
```

前两个源型声明称为选择器，它们在相关的`struct Class` 中声明。下面是其应用：

```cpp
int main () {
    void * a = new(String, "a"), * aa = clone(a);
       void * b = new(String, "b");

       printf("sizeOf(a) == %lu/n", (unsigned long)sizeOf(a));
       if (differ(a, b)){
              puts("ok");
       }
       if (differ(a, aa)){
              puts("differ?");
       }
       if (a == aa){
              puts("clone?");
       }
       delete(a), delete(aa), delete(b);
       return 0;
} 
```

我们创建了两个字符串，并且拷贝了其中一份。我们打印出`String` 对象所占用的大小——并不是对象的控制文本所占用的大小。最终，检查拷贝的对象与对象本身相等，但并不相同，最后再次删除字符串对象。如果所有的程序均以实现，程序的运行结果如下：

```cpp
sizeOf(a)==8
ok 
```

## 2.5 实现——`String`

我们通过写这些方法实现字符串，这些方法需要被放入类型描述`String`中。对于实现一个新的数据类型，动态连接使我们清晰的确定出那些功能函数需要实现。

构造器从新获得文本，传递给 `new()` ，并把这些动态拷贝存储进通过 `new()` 创建的`struct String` 中。

```cpp
struct String {
       const void * class;  /* must be first */
       char * text;
};

static void * String_ctor (void * _self, va_list * app) {     struct String * self = _self;
       const char * text = va_arg(* app, const char *);

       self -> text = malloc(strlen(text) + 1);
       assert(self -> text);
       strcpy(self -> text, text);
       return self;
} 
```

在构造器中，我们紧紧需要初始化 .text 因为 new() 已经建立了 .class 。

析构器释放被字符串控制的动态内存。由于 delete() 只在 self 为非空的情况下调用 析构器，所以我们不需要做其他参数检查，代码如下：

```cpp
static void * String_dtor (void * _self) {     struct String * self = _self;

       free(self -> text), self -> text = 0;
       return self;
} 
```

`String_clone()` 是对字符串的一个拷贝。接下来，源和源的拷贝都将被传进`delete()` 中，因此我们必须对字符串的文本做一个动态内存的拷贝。这个工作通过调用`new()` 很容易实现。

```cpp
static void * String_clone (const void * _self) {     const struct String * self = _self;

       return new(String, self -> text);
} 
```

毫无疑问，对于`String_differ` 如果我们比较同一个字符串对象，则返回假，若果我们比较两个不同的字符串对象，返回真，如果我们想比较字符串文本的差异可试着使用`strcmp()`:

```cpp
static int String_differ (const void * _self, const void * _b) {     const struct String * self = _self;
       const struct String * b = _b;

       if (self == b)
              return 0;
       if (! b || b -> class != String)
              return 1;
       return strcmp(self -> text, b -> text);
} 
```

类型描述符是独一无二的——这里我们要确定一个因素，即：我们的第二个参数是否为字符串文本。

所有这些方法都应该使用关键字`static` 来修饰。因为这些方法只能通过 `new()` 和`delete()` ，或者选择器调用。对于通过类型描述符的方式指定的选择器都是可用的方法。

```cpp
#include “new.r”
static const struct Class _String = {
       sizeof(struct String),
       String_ctor, String_dtor,
       String_clone, String_differ
};

const void * String = & _String; 
```

在 String.h 中声明 String.c 中包含的公有方法，new.h 中声明 new.c 中包含的公有方法。以便于正确的初始化类型描述符，这里也包含了一个私有的头文件 new.r ，此文件中包含了 2.2 部分定义的 struct Class 类型描述。

## 2.6 另一种实现——原子

为了列举我们通过构造器和析构器到底能够做什么，我们实现了原子，所谓原子就是一个唯一的字符窜对象；如果两个原子包含相同的字符窜，则他们是相等的。原子是很容易比较的：如果两个参数的指针不同，则 `differ()` 返回真。原子的构造和销毁要付出一定的代价；我们为所有的原子维持了一个循环链表，并计数原子被克隆的次数，如下：

```cpp
struct String {
       const void * class;                /* must be first */
       char * text;
       struct String * next;
       unsigned count;
};

static struct String * ring;      /* of all strings */

static void * String_clone (const void * _self) {     struct String * self = (void *) _self;

       ++ self -> count;
       return self;
} 
```

所有的原子的循环链表被`ring` 所标记，通过它的成员 `.next` 来扩展，并使用构造器和析构器来维持。在构造器保存文本之前，首先会遍历链表是否有相同的文本已经存在，如下的代码插入到`String_ctor()` 之前：

```cpp
if (ring)
{    
struct String * p = ring;
       do{
              if (strcmp(p -> text, text) == 0)
              {    
++ p -> count;
                     free(self);
                     return p;
              }
       }while ((p = p -> next) != ring);
}
else{
       ring = self;
}
self -> next = ring -> next, ring -> next = self;
self -> count = 1; 
```

如果我们找到了相同文本的原子，则增加它的引用计数`count`值，释放新的对象`self` 返回当前找的的原子指针`p` 。否则我们向循环链表中插入一个新字符串对象并设置其引用计数为`count` 为 1。

析构器防止删除引用计数为非零的原子。如下的代码被插入到`String_dtor()` 之前：

```cpp
if (-- self -> count > 0){
       return 0;
}
assert(ring);
if (ring == self){
       ring = self -> next;
}
if (ring == self){
       ring = 0;
}
else{      
struct String * p = ring;
       while (p -> next != self){     
p = p -> next;
              assert(p != ring);
       }
       p -> next = self -> next;
} 
```

如果对引用计数的减 1 操作计数扔为正数，则返回一个空指针，以便于 `delete()` 手下留情。否则如果我们的字符串对象是最后一个对象我们清除循环链表标记符，否则从链表中删除我们的字符串。

把上述的实现加入到 2.4 的程序中，注意，对一个字符串对象的克隆此时为源字符串对象本身，运行结果如下：

```cpp
sizeOf(a)==16
ok
clone? 
```

## 2.7 总结

给一个指针指定一个对象，动态连接使我们找到了类型指定的函数功能：每个对象都会以一个描述符开始，这个描述符包含了指针，指向对象的可用函数指针表。尤其是，一个描述符包含一个指向构造器的指针，这个构造器用来初始化对象所关联的内存区域，另外这个指针还指向一个析构器，析构器会在删除对象之前回收对象所拥有的资源。

我们称所有的对象所共享的描述符为一个类。而对象是类的实例，对于对象指定类型的功能被称作对象的方法，而消息被这些功能函数所调用。对于一个对象，我们使用选择器功能去定位和调用动态连接的方法。

通过选择器和动态连接使得相同函数名对于不同的类而产生不同的结果。这样的函数被称为是多态的。

多态功能是非常有用的。他们提供了一种概念上的抽象：`differ()` 可比较任何两个对象——我们不需要铭记`differ()` 针对具体的情形是否可用。一个很容易并非常方便的调试工具就是多态函数 `store()` ，可在一个文件描述符上显示任何对象。

## 2.8 练习

了解了多态的功能后，我们需要使用动态连接来实现`Object` 和 `Set`。这对于 `Set` 来说是比较困难的。我们不再记录一个元素属于哪个集合。

对于字符串来说，似乎有更多的方法去实现。我们需要知道字符串的长度，我们更想为一个对象从新设置它的字符串文本，我们应该能打印字符串文本。如果我们乐意去处理子串，将会更加有趣味。

原子是如此有效的，我们可使用一个哈希表来跟踪它，那么一个原子的值能否被改变呢？

`String_clone()` 呈现出一个微妙的问题：在这个函数中，`String` 的值似乎应该与`self->class` 相同。我们向 `new()` 中传递的参数会有任何变化吗？