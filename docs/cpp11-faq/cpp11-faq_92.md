# 线程（thread）

线程（译注：大约是 C++11 中最激动人心的特性了）是一种对程序中的执行或者计算的表述。跟许多现代计算一样，C++11 中的线程之间能够共享地址空间。从这点上来看，它不同于进程：进程一般不会直接跟其它进程共享数据。在过去，C++针对不同的硬件和操作系统有着不同的线程实现版本。如今，C++将线程加入到了标准件库中：一个标准线程 ABI。

许多大部头书籍以及成千上万的论文都曾涉及到并发、并行以及线程。在这一条 FAQ 里几乎不涉及这些内容。事实上，要做到清楚地思考并发非常难。如果你想编写并发程序，请至少看一本书。不要依赖于一本手册、一个标准或者一条 FAQ。

在用一个函数或者函数对象（包括 lambda）构造 std::thread 时，一个线程便启动了。

```cpp
 #include <thread>

void f();

struct F {
    void operator()();
};

int main()
{
    std::thread t1{f};    // f() 在一个单独的线程中执行
    std::thread t2{F()};    // F()() 在一个单独的线程中执行
} 
```

然而，无论 f()和 F()执行任何功能，都不能给出有用的结果。这是因为程序可能会在 t1 执行 f()之前或之后以及 t2 执行 F()之前或之后终结。我们所期望的是能够等到两个任务都完成，这可以通过下述方法来实现：

```cpp
int main()
{
    std::thread t1{f};    // f() 在一个单独的线程中执行
    std::thread t2{F()};    // F()()在一个单独的线程中执行

    t1.join();    // 等待 t1
    t2.join();    // 等待 t2
} 
```

上面例子中的 join()保证了在 t1 和 t2 完成后程序才会终结。这里”join”的意思是等待线程返回后再终结。

通常我们需要传递一些参数给要执行的任务。例如：

```cpp
void f(vector<double>&);

struct F {
vector<double>& v;
F(vector<double>& vv) :v{vv} { }
void operator()();
};

int main()
{
    // f(some_vec) 在一个单独的线程中执行
    std::thread t1{std::bind(f,some_vec)};   

    // F(some_vec)() 在一个单独的线程中执行
    std::thread t2{F(some_vec)};        

    t1.join();
    t2.join();
} 
```

上例中的标准库函数 bind 会将一个函数对象作为它的参数。

通常我们需要在执行完一个任务后得到返回的结果。对于那些简单的对返回值没有概念的，我建议使用 std::future。另一种方法是，我们可以给任务传递一个参数，从而这个任务可以把结果存在这个参数中。例如：

```cpp
void f(vector<double>&, double* res);    // 将结果存在 res 中

struct F {
    vector<double>& v;
    double* res;
    F(vector<double>& vv, double* p) :v{vv}, res{p} { }
    void operator()();    //将结果存在 res 中
};

int main()
{
    double res1;
    double res2;

    // f(some_vec,&res1) 在一个单独的线程中执行
    std::thread t1{std::bind(f,some_vec,&res1)};    
    // F(some_vec,&res2)() 在一个单独的线程中执行
    std::thread t2{F(some_vec,&res2)};        

    t1.join();
    t2.join();

    std::cout << res1 << " " << res2 << ‘\n’;
} 
```

但是关于错误呢？如果一个任务抛出了异常应该怎么办？如果一个任务抛出一个异常并且它没有捕获到这个异常，这个任务将会调用 std::terminate()。调用这个函数一般意味着程序的结束。我们常常会为避免这个问题做诸多尝试。std::future 可以将异常传送给父线程（这正是我喜欢 future 的原因之一）。否则，返回错误代码。

除非一个线程的任务已经完成了，当一个线程超出所在的域的时候，程序会结束。很明显，我们应该避免这一点。

没有办法来请求（也就是说尽量文雅地请求它尽可能早的退出）一个线程结束或者是强制（也就是说杀死这个线程）它结束。下面是可供我们选择的操作：

*   设计我们自己的协作的中断机制（通过使用共享数据来实现。父线程设置这个数据，子线程检查这个数据（子线程将会在该数据被设置后很快退出））。
*   使用 thread::native_handle()来访问线程在操作系统中的符号
*   杀死进程（std::quick_exit()）
*   杀死程序(std::terminate())

这些是委员会能够统一的所有的规则。特别地，来自 POSIX 的代表强烈地反对任何形式的“线程取消”。然而许多 C++的资源模型都依赖于析构器。对于每种系统和每种可能的应有并没有完美的解决方案。

线程中的一个基本问题是数据竞争。也就是当在统一地址空间的两个线程独立访问一个对象时将会导致没有定义的结果。如果一个（或者两个）对对象执行写操作，而另一个（或者两个）对该对象执行读操作，两个线程将在谁先完成操作方面进行竞争。这样得到的结果不仅仅是没定义的，而且常常无法预测最后的结果。为解决这个问题，C++0x 提供了一些规则和保证从而能够让程序员避免数据竞争。

*   C++标准库函数不能直接或间接地访问正在被其它线程访问的对象。一种例外是该函数通过参数（包括 this）来直接或间接访问这个对象。
*   C++标准库函数不能直接或间接修改正在被其它线程访问的对象。一种例外是该函数通过非 const 参数（包括 this）来直接或间接访问这个对象。
*   C++标准函数库的实现需要避免在同时修改统一序列中的不同成员时的数据竞争。

除非已使用别的方式做了声明，多个线程同时访问一个流对象、流缓冲区对象，或者 C 库中的流可能会导致数据竞争。因此除非你能够控制，绝不要让两个线程来共享一个输出流。

你可以

*   等待一个线程[一定的时间](http://chenlq.net/chinese-version-of-c-0-x-faq-time-utility.html)
*   通过[互斥](http://chenlq.net/cpp11-faq-chinese-version-series-exclusive.html)来控制对数据的访问
*   通过[锁来控制对数据的访问](http://chenlq.net/c-0-x-faq-chinese-version-lock.html)
*   使用[条件变量](http://chenlq.net/chinese-version-of-c-11-faq-condition-variable.html)来等待另一个线程的行为
*   通过[future](http://chenlq.net/c-0-x-faq-chinese-version-std-future-and-the-the-std-promise.html)来从线程中返回值

同时可参考：

*   Standard: 30 Thread support library [thread]
*   17.6.4.7 Data race avoidance [res.on.data.races]
*   ???
*   H. Hinnant, L. Crowl, B. Dawes, A. Williams, J. Garland, et al.:

    [Multi-threading Library for Standard C++ (Revision 1)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2320.html)

    N2497==08-0007

*   H.-J. Boehm, L. Crowl:

    C++ object lifetime interactions with the threads API

    N2880==09-0070.

*   L. Crowl, P. Plauger, N. Stoughton:

    Thread Unsafe Standard Functions

    N2864==09-0054.

*   WG14:

    [Thread Cancellation](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2455.pdf) N2455=070325.

（翻译：Yibo Zhu）