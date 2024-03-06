# 枚举类——具有类域和强类型的枚举

# 枚举类——具有类域和强类型的枚举

枚举类（“新的枚举”/“强类型的枚举”）主要用来解决传统的 C++枚举的三个问题：

*   传统 C++枚举会被隐式转换为 int，这在那些不应被转换为 int 的情况下可能导致错误
*   传统 C++枚举的每一枚举值在其作用域范围内都是可见的，容易导致名称冲突(同名冲突)
*   不可以指定枚举的底层数据类型，这可能会导致代码不容易理解、兼容性问题以及不可以进行前向声明

枚举类（enum）（“强类型枚举”）是强类型的，并且具有类域：

```cpp
enum Alert { green, yellow, election, red }; // 传统枚举
 enum class Color { red, blue };   // 新的具有类域和强类型的枚举类
 // 它的枚举值在类的外部是不可直接访问的，需加“类名::”
 //  不会被隐式转换成 int
 enum class TrafficLight { red, yellow, green };
 Alert a = 7;   //  错误，传统枚举不是强类型的，a 没有数据类型
 Color c = 7;   // 错误，没有 int 到 Color 的隐式转换
 int a2 = red;           // 正确，Alert 被隐式转换成 int
// 在 C++98 中是错误的，但是在 C++11 中正确的
int a3 = Alert::red;
int a4 = blue;            // 错误，blue 并没有在类域中
int a5 = Color::blue; // 错误，没有 Color 到 int 的默认转换
Color a6 = Color::blue;   // 正确 
```

正如上面的代码所展示的那样，传统的枚举可以照常工作，但是你现在可以通过提供一个类名来改善枚举的使用，使其成为一个强类型的具有类域的枚举。

因为新的具有类名的枚举具有传统的枚举的功能（命名的枚举值），同时又具有了类的一些特点（枚举值作用域处于类域之内且不会被隐式类型转换成 int），所以我们将其称为枚举类（enum class）。

因为可以指定枚举的底层数据类型，所以可以进行简单的互通操作以及保证枚举值所占的字节大小：

```cpp
enum class Color : char { red, blue }; // 紧凑型表示(一个字节)
// 默认情况下，枚举值的底层数据类型为 int
enum class TrafficLight { red, yellow, green };
// E 占几个字节呢？旧规则只能告诉你：取决于编译器实现
enum E { E1 = 1, E2 = 2, Ebig = 0xFFFFFFF0U };

// C11 中我们可以指定枚举值的底层数据类型大小
enum EE : unsigned long { EE1 = 1, EE2 = 2, EEbig = 0xFFFFFFF0U }; 
```

同时，由于能够指定枚举值的底层数据类型，所以前向声明得以成为可能：

（译注：就是在枚举类定义之前就使用这个枚举类的名字声明指针或引用变量）

```cpp
enum class Color_code : char;     // (前向) 声明
void foobar(Color_code* p);   // 使用
// ...
// 定义
enum class Color_code : char { red, yellow, green, blue }; 
```

枚举类的底层数据类型必须是有符号或无符号的整型，默认情况下是 int。

标准库中也使用了枚举类。

*   系统特定的错误代码，定义在`<system_error>: enum class errc</system_error>`
*   指针安全指示，定义在`<memory>: enum class pointer_safety { relaxed, preferred, strict }</memory>`
*   I/O 流错误，定义在`<iosfwd>: enum class io_errc { stream = 1 }</iosfwd>`
*   异步通信错误，定义在`<future>: enum class future_errc { broken_promise, future_already_retrieved, promise_already_satisfied }</future>`

其中的某些枚举类还定义了操作符重载，比如“==”等。

参考：

the C++ draft section 7.2

[N1513=03-0096] David E. Miller: Improving Enumeration Types (original enum proposal).

[N2347 = J16/07-0207] David E. Miller, Herb Sutter, and Bjarne Stroustrup: Strongly Typed Enums (revision 3).

[N2499=08-0009] Alberto Ganesh Barbati: Forward declaration of enumerations.