# 互斥

# 互斥

互斥是多线程系统中用于控制访问的一个原对象（primitive object）。下面的例子给出了它最基本的用法：

```cpp
std::mutex  m;
int sh; //共享数据
// …
m.lock();
// 对共享数据进行操作：
sh += 1;
m.unlock(); 
```

在任何时刻，最多只能有一个线程执行到 lock()和 unlock()之间的区域（通常称为临界区）。当第一个线程正在临界区执行时，后续执行到 m.lock()的线程将会被阻塞直到第一个进程执行到 m.unlock()。这个过程比较简单，但是如何正确使用互斥并不简单。错误地使用互斥将会导致一系列严重后果。大家可以设想以下情形所导致的后果：一个线程只进行了 lock()而没有执行相应 unlock()； 一个线程对同一个 mutex 对象执行了两次 lock()操作；一个线程在等待 unlock()操作时被阻塞了很久；一个线程需要对两个 mutex 对象执行 lock()操作后才能执行后续任务。可以在很多书（译者注：通常操作系统相关书籍中会讲到）中找到这些问题的答案。在这里（包括 Locks section 一节）所给出的都是一些入门级别的。

除了 lock()，mutex 还提供了 try_lock()操作。线程可以借助该操作来尝试进入临界区，这样一来该线程不会在失败的情况下被阻塞。下面例子给出了 try_lock()的用法：

```cpp
std::mutex m;
int sh; //共享数据
// …
if (m.try_lock()) {
    //操作共享数据
    sh += 1;
    m.unlock();
}
else {
    //可能在试图进入临界区失败后执行其它代码
} 
```

recursive_mutex 是一种能够被同一线程连续锁定多次的 mutex。下面是 recursive_mutex 的一个实例：

```cpp
std::recursive_mutex m;
 int sh; //共享数据
 //..
 void f(int i)
 {
     //…
     m.lock();
     //对共享数据进行操作
     sh += 1;
     if (–i>0) f(i);  //注意：这里对 f(i)进行了递归调用，
     //将导致在 m.unlock()之前多次执行 m.lock()
     m.unlock();
     //…
 } 
```

对于这点，我曾经夸耀过并且用 f()调用它自身。一般地，代码会更加微妙。这是因为代码中经常会有间接递归调用。比如 f()调用 g()，而 g()又调用了 h()，最后 h()又调用了 f()，这样就形成了一个间接递归。

如果我想在未来的 10 秒内进入到一个 mutex 所划定的临界区，该如果实现？ timed_mutex 类可以解决这个问题。事实上，关于它的使用可以被看做是关联了时间限制的 try_lock()的一个特例。

```cpp
std::timed_mutex m;
int sh; //共享数据
//…
if ( m.try_lock_for(std::chrono::seconds(10))) {
//对共享数据进行操作
sh += 1；
m.unlock();
}
else {
    //进入临界区失败，在此执行其它代码
} 
```

try_lock_for()的参数是一个用相对时间表示的 duration。如果你不想这么做而是想等到一个固定的时间点：一个 time_point，你可以使用 try_lock_until()：

```cpp
std::timed_mutex m;
int sh; //共享数据
// …
if ( m.try_lock_until(midnight)) {
//对共享数据进行操作
sh += 1;
m.unlock();
}
else {
    //进入临界区失败，在此执行其它代码
} 
```

这里使用 midnight 是一个冷笑话：对于 mutex 级别的操作，相应的时间是毫秒级别的而不是小时。

当然地，C++0x 中也有 recursive_timed_mutex。

mutex 可以被看做是一个资源（因为它经常被用来代表一种真实的资源），并且当它对至少两个线程可见时它才是有用的。必然地，mutex 不能被复制或者移动（正如你不能复制一个硬件的输入寄存器）。

令人惊讶地，实际中经常很难做到 lock()s 与 unlock()s 的匹配。设想一下那些复杂的控制结构，错误以及异常，要做到匹配的确比较困难。如果你可以选择使用 locks 去管理你的互斥，这将为你和你的用户节省大量的时间，再也不用熬夜通宵彻夜无眠了。（that will save you and your users a lot of sleep？？）。

同时可参考：

*   Standard: 30.4 Mutual exclusion [thread.mutex]
*   H. Hinnant, L. Crowl, B. Dawes, A. Williams, J. Garland, et al.:

    [Multi-threading Library for Standard C++ (Revision 1)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2320.html)

*   ???

（翻译：Yibo Zhu）