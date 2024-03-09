# 时间工具程序

在编写程序时，我们常常需要定时执行一些任务。例如，标准库 mutexes 和 locks 提供了的一些选项就需要这一定时功能：线程等待一段时间(duration)或者等到某一给定时刻(time_point)。

如果你需要得到当前时刻，你可以调用 system_clock、monotonic_clock、high_resolution_clock 中任何一个时钟的 now()方法。例如：

```cpp
monotonic_clock::time_point t = monotonic_clock::now();
// 执行一些代码
monotonic_clock::duration d = monotonic_clock::now() – t;
// 一些需要 d 个单位时间的任务 
```

在上面例子中，一个时钟返回一个 time_point 和一个 duration。其中 duration 是该时钟它返回的两个 time_point 的差值。如果你对细节不感兴趣，你可以使用 auto 类型。

```cpp
auto t = monotonic_clock::now();
// 执行一些代码
auto d = monotonic_clock::now() – t;
// 一些需要 d 个单位时间的任务 
```

这里提供的时间工具是为了高效支持系统内部的应用。它们不会提供便捷的工具来帮助你维护你的社交日历。事实上，这些时间工具源自于高能物理对时间度量的高精度要求。为了能够表达所有的时间尺度（比如世纪和皮秒），同时避免单位、打字排版以及舍入时的混淆，使用编译时的有理数包来表示 duration 和 time_point。一个 duration 由两部分组成：一个数字时钟”tick”（滴答）和能够表示一个 tick 期望(一秒还是一毫秒？)的事物（一个 period）。这里的 period 是 duration 类型的一部分。下面的表格摘自标准头文件中，它定义了国际单位系统中的 period。这或许会帮助你明白它们的使用范围。

```cpp
// 为方便起见，对国际单位做的 typedef:
typedef ratio<1, 1000000000000000000000000> yocto;  // 有条件的支持
typedef ratio<1,    1000000000000000000000> zepto;  // 有条件的支持
typedef ratio<1,       1000000000000000000> atto;
typedef ratio<1,          1000000000000000> femto;
typedef ratio<1,             1000000000000> pico;
typedef ratio<1,                1000000000> nano;
typedef ratio<1,                   1000000> micro;
typedef ratio<1,                      1000> milli;
typedef ratio<1,                       100> centi;
typedef ratio<1,                        10> deci;
typedef ratio<                       10, 1> deca;
typedef ratio<                      100, 1> hecto;
typedef ratio<                     1000, 1> kilo;
typedef ratio<                  1000000, 1> mega;    
typedef ratio<               1000000000, 1> giga;
typedef ratio<            1000000000000, 1> tera;
typedef ratio<         1000000000000000, 1> peta;
typedef ratio<      1000000000000000000, 1> exa;    
typedef ratio<   1000000000000000000000, 1> zetta;  //有条件的支持
typedef ratio<1000000000000000000000000, 1> yotta;  //有条件的支持 
```

编译时有理数提供的常用算术操作符(`+`, `-`, `*`, and `/`)和比较操作符 (`==`, `!=`, `<` , `<=`, `>`, `>=`)适用于合理的 duration 和 time_point 组合（比如你不能对两个 time_point 进行加法运算）。系统会对这些运算进行溢出以及除数为 0 的检查。由于这是编译时的工具，所以不用担心它在的运行时的性能。另外，你还可以使用++、–、+=、-=以及/=来操作 duration，同时可以使用 tp+=d 和 tp-=d 来操作 time_point tp 和 duration d。

下面是一些使用定义在中的 duration 类型的例子：

```cpp
microseconds mms = 12345;
milliseconds ms = 123;
seconds s = 10;
minutes m = 30;
hours h = 34;

auto x = std::chrono::hours(3);    // 显式使用命名空间
auto x = hours(2)+minutes(35)+seconds(9); // 假设合适的”using” 
```

你不能用一个分数来初始化 duration。比如，不要用 2.5 秒，而应该用 2500 毫秒。这是因为 duration 被解释为若干个 tick，而每个 tick 表示一个 duration 时间段的单位，比如上面例子中所定义的 milli 和 kilo。所以必须用整数来初始化。Duration 的默认单位是秒。也就是说，时间段为 1 的 tick 被解释为 1 秒。在程序中，我们也可以指明 duration 的单位。

```cpp
duration d0 = 5;        // 秒 (默认值)
duration d1 = 99;        // 千秒
duration > d2 = 100;    // d1 和 d2 的类型相同 
```

如果我们想利用 duration 来做一些事（比如打印出这个 duration 的值），那么我们必须给出它的单位。如：分钟或者微妙。例如：

```cpp
auto t = monotonic_clock::now();
// 执行一些代码
nanoseconds d = monotonic_clock::now() – t;    // 我们希望结果的单位是纳秒
cout << “something took ” << d << “nanosecondsn”; 
```

或者，我们可以将 duration 转换成一个浮点数

```cpp
auto t = monotonic_clock::now();
//执行一些代码
auto d = monotonic_clock::now() – t;
cout << “something took ” << duration(d).count() << “secondsn”; 
```

这里的 count()返回 tick 的数量。

同时可参考：

*   Standard: 20.9 Time utilities [time]
*   Howard E. Hinnant,

    Walter E. Brown,

    Jeff Garland,

    and Marc Paterno:

    [A Foundation to Sleep On.](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2661.htm)

    [N2661=08-0171.](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2661.htm)

    [Including “A Brief History of Time” (With apologies to Stephen Hawking).](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2661.htm)

(翻译：Yibo Zhu)