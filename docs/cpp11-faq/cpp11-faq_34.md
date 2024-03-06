# 常量表达式（constexpr）

# 常量表达式（constexpr）

常量表达式机制是为了：

*   提供一种更加通用的常量表达式
*   允许用户自定义的类型成为常量表达式
*   提供了一种保证在编译期完成初始化的方法（可以在编译时期执行某些函数调用）

考虑下面这段代码：

```cpp
enum Flags { good=0, fail=1, bad=2, eof=4 };
constexpr int operator|(Flags f1, Flags f2)
{ return Flags(int(f1)|int(f2)); }
void f(Flags x)
{
    switch (x) {
    case bad:         /* … */ break;
    case eof:         /* … */ break;
    case bad|eof:     /* … */ break;
    default:          /* … */ break;
    }
} 
```

在这里，常量表达式关键字 constexpr 表示这个重载的操作符“|”只应包含形式简单的运算，如果它的参数本身就是常量 ，那么这个操作符应该在编译时期就应该计算出它的结果来。（译注： 我们都知道，switch 的分支条件要求常量，而使用 constexpr 关键字重载操作符“|”之后，虽然“bad|eof”是一个表达式，但是因为这两个参数都是常量，在编译时期，就可以计算出它的结果，因而也可以作为常量对待。）

除了可以在编译时期被动地计算表达式的值之外，我们希望能够强制要求表达式在编译时期计算其结果值，从而用作其它用途，比如对某个变量进行赋值。当我们在变量声明前加上 constexpr 关键字之后，可以实现这一功能，当然，它也同时会让这个变量成为常量。

```cpp
 constexpr int x1 = bad|eof;    // ok
    void f(Flags f3)
    {
        // 错误：因为 f3 不是常量，所以无法在编译时期计算这个表达式的结果值
        constexpr int x2 = bad|f3;
        int x3 = bad|f3;     // ok，可以在运行时计算
    } 
```

使用 constexpr 强制在运行期求值，一般用于全局对象（或 namespace 内的对象），尤其是那些放在只读区的对象。

除了基本类型外，对于那些构造函数比较简单的对象和由其构成的表达式，也可以成为常量表达式

```cpp
 struct Point {
        int x,y;
        constexpr Point(int xx, int yy) : x(xx), y(yy){}
    };
    constexpr Point origo(0,0);
    constexpr int z = origo.x;
    constexpr Point a[] = {Point(0,0), Point(1,1), Point(2,2) };
    constexpr int x = a[1].x;    // x 变成常量 1 
```

需要注意的是，constexpr 并不是 const 的通用版，反之亦然：

*   const 主要用于表达“对接口的写权限控制”，即“对于被 const 修饰的量名(例如 const 指针变量)，不得通过它对所指对象作任何修改”。(但是可以通过其他接口修改该对象)。另外，把对象声明为 const 也为编译器提供了潜在的优化可能。具体来说就是，如果把一个量声明为 const，并且没有其他地方对该量作取址运算，那么编译器通常(取决于编译期实现)会用该量的实际常量值直接替换掉代码中所有引用该量的地方，而不用在最终编译结果中生成对该量的存取指令。
*   constexpr 的主要功能则是让更多的运算可以在编译期完成，并能保证表达式在语义上是类型安全的。(译注：相比之下，C 语言中#define 只能提供简单的文本替换，而不具任何类型检查能力)。与 const 相比，被 constexpr 修饰的对象则强制要求其初始化表达式能够在编译期完成计算。之后所有引用该常量对象的地方，若非必要，一律用计算出来的常量值替换。

（译注：zwvista 的一段评论，有助于我们理解 constexpr 的意义，感谢 zwvista。constexpr 将编译期常量概念延伸至括用户自定义常量以及常量函数，其值的不可修改性由编译器保证，因而 constexpr 表达式是一般化的，受保证的常量表达式。）

参考：

*   the C++ draft 3.6.2 Initialization of non-local objects, 3.9 Types [12], 5.19 Constant expressions, 7.1.5 The constexpr specifier
*   [N1521=03-0104] Gabriel Dos Reis: [Generalized Constant Expressions](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2003/n1521.pdf) (original proposal).
*   [N2235=07-0095] Gabriel Dos Reis, Bjarne Stroustrup, and Jens Maurer: [Generalized Constant Expressions — Revision 5](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2235.pdf) .

(翻译：陈良乔，感谢：zwvista)