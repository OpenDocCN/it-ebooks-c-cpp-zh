# 第六章 多线程

# 第六章 多线程

### 目录

*   6.1 概述
*   6.2 线程管理
*   6.3 同步
*   6.4 线程本地存储
*   6.5 练习

![](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 该书采用 [Creative Commons License](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 授权

## 6.1\. 概述

线程就是，在同一程序同一时间内允许执行不同函数的离散处理队列。 这使得一个长时间去进行某种特殊运算的函数在执行时不阻碍其他的函数变得十分重要。 线程实际上允许同时执行两种函数，而这两个函数不必相互等待。

一旦一个应用程序启动，它仅包含一个默认线程。 此线程执行 `main()` 函数。 在 `main()`中被调用的函数则按这个线程的上下文顺序地执行。 这样的程序称为单线程程序。

反之，那些创建新的线程的程序就是多线程程序。 他们不仅可以在同一时间执行多个函数，而且这在如今多核盛行的时代显得尤为重要。 既然多核允许同时执行多个函数，这就使得对开发人员相应地使用这种处理能力提出了要求。 然而线程一直被用来当并发地执行多个函数，开发人员现在不得不仔细地构建应用来支持这种并发。 多线程编程知识也因此在多核系统时代变得越来越重要。

本章将介绍 C++ Boost 库 [Boost.Thread](http://www.boost.org/libs/thread/)，它可以开发独立于平台的多线程应用程序。

## 6.2\. 线程管理

在这个库最重要的一个类就是 `boost::thread`，它是在 `boost/thread.hpp` 里定义的，用来创建一个新线程。下面的示例来说明如何运用它。

```cpp
#include <boost/thread.hpp> 
#include <iostream> 

void wait(int seconds) 
{ 
  boost::this_thread::sleep(boost::posix_time::seconds(seconds)); 
} 

void thread() 
{ 
  for (int i = 0; i < 5; ++i) 
  { 
    wait(1); 
    std::cout << i << std::endl; 
  } 
} 

int main() 
{ 
  boost::thread t(thread); 
  t.join(); 
} 
```

*   下载源代码

新建线程里执行的那个函数的名称被传递到 `boost::thread` 的构造函数。 一旦上述示例中的变量 `t` 被创建，该 `thread()` 函数就在其所在线程中被立即执行。 同时在 `main()` 里也并发地执行该 `thread()` 。

为了防止程序终止，就需要对新建线程调用 `join()` 方法。 `join()` 方法是一个阻塞调用：它可以暂停当前线程，直到调用 `join()` 的线程运行结束。 这就使得 `main()` 函数一直会等待到 `thread()` 运行结束。

正如在上面的例子中看到，一个特定的线程可以通过诸如 `t` 的变量访问，通过这个变量等待着它的使用 `join()` 方法终止。 但是，即使 `t` 越界或者析构了，该线程也将继续执行。 一个线程总是在一开始就绑定到一个类型为 `boost::thread` 的变量，但是一旦创建，就不在取决于它。 甚至还存在着一个叫 `detach()` 的方法，允许类型为 `boost::thread` 的变量从它对应的线程里分离。 当然了，像 `join()` 的方法之后也就不能被调用，因为这个变量不再是一个有效的线程。

任何一个函数内可以做的事情也可以在一个线程内完成。 归根结底，一个线程只不过是一个函数，除了它是同时执行的。 在上述例子中，使用一个循环把 5 个数字写入标准输出流。 为了减缓输出，每一个循环中调用 `wait()` 函数让执行延迟了一秒。 `wait()` 可以调用一个名为 `sleep()` 的函数，这个函数也来自于 Boost.Thread，位于 `boost::this_thread` 名空间内。

`sleep()` 要么在预计的一段时间或一个特定的时间点后时才让线程继续执行。 通过传递一个类型为 `boost::posix_time::seconds` 的对象，在这个例子里我们指定了一段时间。 `boost::posix_time::seconds` 来自于 Boost.DateTime 库，它被 Boost.Thread 用来管理和处理时间的数据。

虽然前面的例子说明了如何等待一个不同的线程，但下面的例子演示了如何通过所谓的中断点让一个线程中断。

```cpp
#include <boost/thread.hpp> 
#include <iostream> 

void wait(int seconds) 
{ 
  boost::this_thread::sleep(boost::posix_time::seconds(seconds)); 
} 

void thread() 
{ 
  try 
  { 
    for (int i = 0; i < 5; ++i) 
    { 
      wait(1); 
      std::cout << i << std::endl; 
    } 
  } 
  catch (boost::thread_interrupted&) 
  { 
  } 
} 

int main() 
{ 
  boost::thread t(thread); 
  wait(3); 
  t.interrupt(); 
  t.join(); 
} 
```

*   下载源代码

在一个线程对象上调用 `interrupt()` 会中断相应的线程。 在这方面，中断意味着一个类型为 `boost::thread_interrupted` 的异常，它会在这个线程中抛出。 然后这只有在线程达到中断点时才会发生。

如果给定的线程不包含任何中断点，简单调用 `interrupt()` 就不会起作用。 每当一个线程中断点，它就会检查 `interrupt()` 是否被调用过。 只有被调用过了， `boost::thread_interrupted` 异常才会相应地抛出。

Boost.Thread 定义了一系列的中断点，例如 `sleep()` 函数。 由于 `sleep()` 在这个例子里被调用了五次，该线程就检查了五次它是否应该被中断。 然而 `sleep()` 之间的调用，却不能使线程中断。

一旦该程序被执行，它只会打印三个数字到标准输出流。 这是由于在 main 里 3 秒后调用 `interrupt()`方法。 因此，相应的线程被中断，并抛出一个 `boost::thread_interrupted` 异常。 这个异常在线程内也被正确地捕获， `catch` 处理虽然是空的。 由于 `thread()` 函数在处理程序后返回，线程也被终止。 这反过来也将终止整个程序，因为 `main()` 等待该线程使用 join（）终止该线程。

Boost.Thread 定义包括上述 `sleep()`函数十个中断。 有了这些中断点，线程可以很容易及时中断。 然而，他们并不总是最佳的选择，因为中断点必须事前读入以检查 `boost::thread_interrupted` 异常。

为了提供一个对 Boost.Thread 里提供的多种函数的整体概述，下面的例子将会再介绍两个。

```cpp
#include <boost/thread.hpp> 
#include <iostream> 

int main() 
{ 
  std::cout << boost::this_thread::get_id() << std::endl; 
  std::cout << boost::thread::hardware_concurrency() << std::endl; 
} 
```

*   下载源代码

使用 `boost::this_thread`命名空间，能提供独立的函数应用于当前线程，比如前面出现的 `sleep()` 。 另一个是 `get_id()`：它会返回一个当前线程的 ID 号。 它也是由 `boost::thread` 提供的。

`boost::thread` 类提供了一个静态方法 `hardware_concurrency()` ，它能够返回基于 CPU 数目或者 CPU 内核数目的刻在同时在物理机器上运行的线程数。 在常用的双核机器上调用这个方法，返回值为 2。 这样的话就可以确定在一个多核程序可以同时运行的理论最大线程数。

## 6.3\. 同步

虽然多线程的使用可以提高应用程序的性能，但也增加了复杂性。 如果使用线程在同一时间执行几个函数，访问共享资源时必须相应地同步。 一旦应用达到了一定规模，这涉及相当一些工作。 本段介绍了 Boost.Thread 提供同步线程的类。

```cpp
#include <boost/thread.hpp> 
#include <iostream> 

void wait(int seconds) 
{ 
  boost::this_thread::sleep(boost::posix_time::seconds(seconds)); 
} 

boost::mutex mutex; 

void thread() 
{ 
  for (int i = 0; i < 5; ++i) 
  { 
    wait(1); 
    mutex.lock(); 
    std::cout << "Thread " << boost::this_thread::get_id() << ": " << i << std::endl; 
    mutex.unlock(); 
  } 
} 

int main() 
{ 
  boost::thread t1(thread); 
  boost::thread t2(thread); 
  t1.join(); 
  t2.join(); 
} 
```

*   下载源代码

多线程程序使用所谓的互斥对象来同步。 Boost.Thread 提供多个的互斥类，`boost::mutex`是最简单的一个。 互斥的基本原则是当一个特定的线程拥有资源的时候防止其他线程夺取其所有权。 一旦释放，其他的线程可以取得所有权。 这将导致线程等待至另一个线程完成处理一些操作，从而相应地释放互斥对象的所有权。

上面的示例使用一个类型为 `boost::mutex` 的 `mutex` 全局互斥对象。 `thread()` 函数获取此对象的所有权才在 `for` 循环内使用 `lock()` 方法写入到标准输出流的。 一旦信息被写入，使用 `unlock()` 方法释放所有权。

`main()` 创建两个线程，同时执行 `thread ()`函数。 利用 `for` 循环，每个线程数到 5，用一个迭代器写一条消息到标准输出流。 不幸的是，标准输出流是一个全局性的被所有线程共享的对象。 该标准不提供任何保证 `std::cout` 可以安全地从多个线程访问。 因此，访问标准输出流必须同步：在任何时候，只有一个线程可以访问 `std::cout`。

由于两个线程试图在写入标准输出流前获得互斥体，实际上只能保证一次只有一个线程访问 `std::cout`。 不管哪个线程成功调用 `lock()` 方法，其他所有线程必须等待，直到 `unlock()` 被调用。

获取和释放互斥体是一个典型的模式，是由 Boost.Thread 通过不同的数据类型支持。 例如，不直接地调用 `lock()` 和 `unlock()`，使用 `boost::lock_guard` 类也是可以的。

```cpp
#include <boost/thread.hpp> 
#include <iostream> 

void wait(int seconds) 
{ 
  boost::this_thread::sleep(boost::posix_time::seconds(seconds)); 
} 

boost::mutex mutex; 

void thread() 
{ 
  for (int i = 0; i < 5; ++i) 
  { 
    wait(1); 
    boost::lock_guard<boost::mutex> lock(mutex); 
    std::cout << "Thread " << boost::this_thread::get_id() << ": " << i << std::endl; 
  } 
} 

int main() 
{ 
  boost::thread t1(thread); 
  boost::thread t2(thread); 
  t1.join(); 
  t2.join(); 
} 
```

*   下载源代码

`boost::lock_guard` 在其内部构造和析构函数分别自动调用 `lock()` 和 `unlock()` 。 访问共享资源是需要同步的，因为它显示地被两个方法调用。 `boost::lock_guard` 类是另一个出现在 第二章 *智能指针* 的 RAII 用语。

除了`boost::mutex` 和 `boost::lock_guard` 之外，Boost.Thread 也提供其他的类支持各种同步。 其中一个重要的就是 `boost::unique_lock` ，相比较 `boost::lock_guard` 而言，它提供许多有用的方法。

```cpp
#include <boost/thread.hpp> 
#include <iostream> 

void wait(int seconds) 
{ 
  boost::this_thread::sleep(boost::posix_time::seconds(seconds)); 
} 

boost::timed_mutex mutex; 

void thread() 
{ 
  for (int i = 0; i < 5; ++i) 
  { 
    wait(1); 
    boost::unique_lock<boost::timed_mutex> lock(mutex, boost::try_to_lock); 
    if (!lock.owns_lock()) 
      lock.timed_lock(boost::get_system_time() + boost::posix_time::seconds(1)); 
    std::cout << "Thread " << boost::this_thread::get_id() << ": " << i << std::endl; 
    boost::timed_mutex *m = lock.release(); 
    m->unlock(); 
  } 
} 

int main() 
{ 
  boost::thread t1(thread); 
  boost::thread t2(thread); 
  t1.join(); 
  t2.join(); 
} 
```

*   下载源代码

上面的例子用不同的方法来演示 `boost::unique_lock` 的功能。 当然了，这些功能的用法对给定的情景不一定适用；`boost::lock_guard` 在上个例子的用法还是挺合理的。 这个例子就是为了演示 `boost::unique_lock` 提供的功能。

`boost::unique_lock` 通过多个构造函数来提供不同的方式获得互斥体。 这个期望获得互斥体的函数简单地调用了 `lock()` 方法，一直等到获得这个互斥体。 所以它的行为跟 `boost::lock_guard` 的那个是一样的。

如果第二个参数传入一个 `boost::try_to_lock` 类型的值，对应的构造函数就会调用 `try_lock()` 方法。 这个方法返回 `bool` 型的值：如果能够获得互斥体则返回`true`，否则返回 `false` 。 相比 `lock()` 函数，`try_lock()` 会立即返回，而且在获得互斥体之前不会被阻塞。

上面的程序向 `boost::unique_lock` 的构造函数的第二个参数传入`boost::try_to_lock`。 然后通过 `owns_lock()` 可以检查是否可获得互斥体。 如果不能， `owns_lock()` 返回 `false`。 这也用到 `boost::unique_lock` 提供的另外一个函数： `timed_lock()` 等待一定的时间以获得互斥体。 给定的程序等待长达 1 秒，应较足够的时间来获取更多的互斥。

其实这个例子显示了三个方法获取一个互斥体：`lock()` 会一直等待，直到获得一个互斥体。 `try_lock()` 则不会等待，但如果它只会在互斥体可用的时候才能获得，否则返回 `false` 。 最后，`timed_lock()` 试图获得在一定的时间内获取互斥体。 和 `try_lock()` 一样，返回`bool` 类型的值意味着成功是否。

虽然 `boost::mutex` 提供了 `lock()` 和 `try_lock()` 两个方法，但是 `boost::timed_mutex` 只支持 `timed_lock()` ，这就是上面示例那么使用的原因。 如果不用 `timed_lock()` 的话，也可以像以前的例子那样用 `boost::mutex`。

就像 `boost::lock_guard` 一样， `boost::unique_lock` 的析构函数也会相应地释放互斥量。此外，可以手动地用 `unlock()` 释放互斥量。也可以像上面的例子那样，通过调用 `release()` 解除`boost::unique_lock` 和互斥量之间的关联。然而在这种情况下，必须显式地调用 `unlock()` 方法来释放互斥量，因为 `boost::unique_lock` 的析构函数不再做这件事情。

`boost::unique_lock` 这个所谓的独占锁意味着一个互斥量同时只能被一个线程获取。 其他线程必须等待，直到互斥体再次被释放。 除了独占锁，还有非独占锁。 Boost.Thread 里有个 `boost::shared_lock` 的类提供了非独占锁。 正如下面的例子，这个类必须和 `boost::shared_mutex` 型的互斥量一起使用。

```cpp
#include <boost/thread.hpp> 
#include <iostream> 
#include <vector> 
#include <cstdlib> 
#include <ctime> 

void wait(int seconds) 
{ 
  boost::this_thread::sleep(boost::posix_time::seconds(seconds)); 
} 

boost::shared_mutex mutex; 
std::vector<int> random_numbers; 

void fill() 
{ 
  std::srand(static_cast<unsigned int>(std::time(0))); 
  for (int i = 0; i < 3; ++i) 
  { 
    boost::unique_lock<boost::shared_mutex> lock(mutex); 
    random_numbers.push_back(std::rand()); 
    lock.unlock(); 
    wait(1); 
  } 
} 

void print() 
{ 
  for (int i = 0; i < 3; ++i) 
  { 
    wait(1); 
    boost::shared_lock<boost::shared_mutex> lock(mutex); 
    std::cout << random_numbers.back() << std::endl; 
  } 
} 

int sum = 0; 

void count() 
{ 
  for (int i = 0; i < 3; ++i) 
  { 
    wait(1); 
    boost::shared_lock<boost::shared_mutex> lock(mutex); 
    sum += random_numbers.back(); 
  } 
} 

int main() 
{ 
  boost::thread t1(fill); 
  boost::thread t2(print); 
  boost::thread t3(count); 
  t1.join(); 
  t2.join(); 
  t3.join(); 
  std::cout << "Sum: " << sum << std::endl; 
} 
```

*   下载源代码

`boost::shared_lock` 类型的非独占锁可以在线程只对某个资源读访问的情况下使用。 一个线程修改的资源需要写访问，因此需要一个独占锁。 这样做也很明显：只需要读访问的线程不需要知道同一时间其他线程是否访问。 因此非独占锁可以共享一个互斥体。

在给定的例子， `print()` 和 `count()` 都可以只读访问 `random_numbers` 。 虽然 `print()` 函数把 `random_numbers` 里的最后一个数写到标准输出，`count()` 函数把它统计到 `sum` 变量。 由于没有函数修改 `random_numbers`，所有的都可以在同一时间用 `boost::shared_lock` 类型的非独占锁访问它。

在 `fill()` 函数里，需要用一个 `boost::unique_lock` 类型的非独占锁，因为它插入了一个新的随机数到 `random_numbers`。 在 `unlock()` 显式地调用 `unlock()` 来释放互斥量之后， `fill()` 等待了一秒。 相比于之前的那个样子， 在 `for` 循环的尾部调用 `wait()` 以保证容器里至少存在一个随机数，可以被`print()` 或者 `count()` 访问。 对应地，这两个函数在 `for` 循环的开始调用了 `wait()` 。

考虑到在不同的地方每个单独地调用 `wait()` ，一个潜在的问题变得很明显:函数调用的顺序直接受 CPU 执行每个独立进程的顺序决定。 利用所谓的条件变量，可以同步哪些独立的线程，使数组的每个元素都被不同的线程立即添加到 `random_numbers` 。

```cpp
#include <boost/thread.hpp> 
#include <iostream> 
#include <vector> 
#include <cstdlib> 
#include <ctime> 

boost::mutex mutex; 
boost::condition_variable_any cond; 
std::vector<int> random_numbers; 

void fill() 
{ 
  std::srand(static_cast<unsigned int>(std::time(0))); 
  for (int i = 0; i < 3; ++i) 
  { 
    boost::unique_lock<boost::mutex> lock(mutex); 
    random_numbers.push_back(std::rand()); 
    cond.notify_all(); 
    cond.wait(mutex); 
  } 
} 

void print() 
{ 
  std::size_t next_size = 1; 
  for (int i = 0; i < 3; ++i) 
  { 
    boost::unique_lock<boost::mutex> lock(mutex); 
    while (random_numbers.size() != next_size) 
      cond.wait(mutex); 
    std::cout << random_numbers.back() << std::endl; 
    ++next_size; 
    cond.notify_all(); 
  } 
} 

int main() 
{ 
  boost::thread t1(fill); 
  boost::thread t2(print); 
  t1.join(); 
  t2.join(); 
} 
```

*   下载源代码

这个例子的程序删除了 `wait()` 和 `count()` 。线程不用在每个循环迭代中等待一秒，而是尽可能快地执行。此外，没有计算总额；数字完全写入标准输出流。

为确保正确地处理随机数，需要一个允许检查多个线程之间特定条件的条件变量来同步不每个独立的线程。

正如上面所说， `fill()` 函数用在每个迭代产生一个随机数，然后放在 `random_numbers` 容器中。 为了防止其他线程同时访问这个容器，就要相应得使用一个排它锁。 不是等待一秒，实际上这个例子却用了一个条件变量。 调用 `notify_all()` 会唤醒每个哪些正在分别通过调用`wait()` 等待此通知的线程。

通过查看 `print()` 函数里的 `for` 循环，可以看到相同的条件变量被 `wait()` 函数调用了。 如果这个线程被 `notify_all()` 唤醒，它就会试图这个互斥量，但只有在 `fill()` 函数完全释放之后才能成功。

这里的窍门就是调用 `wait()` 会释放相应的被参数传入的互斥量。 在调用 `notify_all()`后， `fill()` 函数会通过 `wait()` 相应地释放线程。 然后它会阻止和等待其他的线程调用 `notify_all()` ，一旦随机数已写入标准输出流，这就会在 `print()` 里发生。

注意到在 `print()` 函数里调用 `wait()` 事实上发生在一个单独 `while` 循环里。 这样做的目的是为了处理在 `print()` 函数里第一次调用 `wait()` 函数之前随机数已经放到容器里。 通过比较 `random_numbers` 里元素的数目与预期值，发现这成功地处理了把随机数写入到标准输出流。

## 6.4\. 线程本地存储

线程本地存储（TLS）是一个只能由一个线程访问的专门的存储区域。 TLS 的变量可以被看作是一个只对某个特定线程而非整个程序可见的全局变量。 下面的例子显示了这些变量的好处。

```cpp
#include <boost/thread.hpp> 
#include <iostream> 
#include <cstdlib> 
#include <ctime> 

void init_number_generator() 
{ 
  static bool done = false; 
  if (!done) 
  { 
    done = true; 
    std::srand(static_cast<unsigned int>(std::time(0))); 
  } 
} 

boost::mutex mutex; 

void random_number_generator() 
{ 
  init_number_generator(); 
  int i = std::rand(); 
  boost::lock_guard<boost::mutex> lock(mutex); 
  std::cout << i << std::endl; 
} 

int main() 
{ 
  boost::thread t[3]; 

  for (int i = 0; i < 3; ++i) 
    t[i] = boost::thread(random_number_generator); 

  for (int i = 0; i < 3; ++i) 
    t[i].join(); 
} 
```

*   下载源代码

该示例创建三个线程，每个线程写一个随机数到标准输出流。 `random_number_generator()` 函数将会利用在 C++标准里定义的 `std::rand()` 函数创建一个随机数。 但是用于 `std::rand()` 的随机数产生器必须先用 `std::srand()` 正确地初始化。 如果没做，程序始终打印同一个随机数。

随机数产生器，通过 `std::time()` 返回当前时间， 在 `init_number_generator()` 函数里完成初始化。 由于这个值每次都不同，可以保证产生器总是用不同的值初始化，从而产生不同的随机数。 因为产生器只要初始化一次， `init_number_generator()` 用了一个静态变量 `done` 作为条件量。

如果程序运行了多次，写入的三分之二的随机数显然就会相同。 事实上这个程序有个缺陷：`std::rand()` 所用的产生器必须被各个线程初始化。 因此 `init_number_generator()` 的实现实际上是不对的，因为它只调用了一次 `std::srand()` 。使用 TLS，这一缺陷可以得到纠正。

```cpp
#include <boost/thread.hpp> 
#include <iostream> 
#include <cstdlib> 
#include <ctime> 

void init_number_generator() 
{ 
  static boost::thread_specific_ptr<bool> tls; 
  if (!tls.get()) 
    tls.reset(new bool(false)); 
  if (!*tls) 
  { 
    *tls = true; 
    std::srand(static_cast<unsigned int>(std::time(0))); 
  } 
} 

boost::mutex mutex; 

void random_number_generator() 
{ 
  init_number_generator(); 
  int i = std::rand(); 
  boost::lock_guard<boost::mutex> lock(mutex); 
  std::cout << i << std::endl; 
} 

int main() 
{ 
  boost::thread t[3]; 

  for (int i = 0; i < 3; ++i) 
    t[i] = boost::thread(random_number_generator); 

  for (int i = 0; i < 3; ++i) 
    t[i].join(); 
} 
```

*   下载源代码

用一个 TLS 变量 `tls` 代替静态变量 `done`，是基于用 `bool` 类型实例化的 `boost::thread_specific_ptr` 。 原则上， `tls` 工作起来就像 `done` ：它可以作为一个条件指明随机数发生器是否被初始化。 但是关键的区别，就是 `tls` 存储的值只对相应的线程可见和可用。

一旦一个 `boost::thread_specific_ptr` 型的变量被创建，它可以相应地设置。 不过，它期望得到一个 `bool` 型变量的地址，而非它本身。使用 `reset()` 方法，可以把它的地址保存到 `tls` 里面。 在给出的例子中，会动态地分配一个 `bool` 型的变量，由 `new` 返回它的地址，并保存到 `tls` 里。 为了避免每次调用 `init_number_generator()` 都设置 `tls` ，它会通过 `get()` 函数检查是否已经保存了一个地址。

由于 `boost::thread_specific_ptr` 保存了一个地址，它的行为就像一个普通的指针。 因此，`operator*()` 和 `operator-&gt;()` 都被被重载以方便使用。 这个例子用 `*tls` 检查这个条件当前是 `true` 还是 `false`。 再根据当前的条件，随机数生成器决定是否初始化。

正如所见， `boost::thread_specific_ptr` 允许为当前进程保存一个对象的地址，然后只允许当前进程获得这个地址。 然而，当一个线程已经成功保存这个地址，其他的线程就会可能就失败。

如果程序正在执行时，它可能会令人感到奇怪：尽管有了 TLS 的变量，生成的随机数仍然相等。 这是因为，三个线程在同一时间被创建，从而造成随机数生成器在同一时间初始化。 如果该程序执行了几次，随机数就会改变，这就表明生成器初始化正确了。

## 6.5\. 练习

You can buy [solutions to all exercises](http://en.highscore.de/shop/index.php?p=boost-solution) in this book as a ZIP file.

1.  重构下面的程序用两个线程来计算总和。由于现在许多处理器有两个内核，应利用线程减少执行时间。

    ```cpp
    #include &lt;boost/date_time/posix_time/posix_time.hpp&gt; 
    #include &lt;boost/cstdint.hpp&gt; 
    #include &lt;iostream&gt; 

    int main() 
    { 
      boost::posix_time::ptime start = boost::posix_time::microsec_clock::local_time(); 

      boost::uint64_t sum = 0; 
      for (int i = 0; i &lt; 1000000000; ++i) 
        sum += i; 

      boost::posix_time::ptime end = boost::posix_time::microsec_clock::local_time(); 
      std::cout &lt;&lt; end - start &lt;&lt; std::endl; 

      std::cout &lt;&lt; sum &lt;&lt; std::endl; 
    } 
    ```

    *   下载源代码
2.  通过利用处理器尽可能同时执行多的线程，把例 1 一般化。 例如，如果处理器有四个内核，就应该利用四个线程。

3.  修改下面的程序，在 `main()`中自己的线程中执行 `thread()` 。 程序应该能够计算总和，然后把结果输入到标准输出两次。 但可以更改 `calculate()`，`print()` 和 `thread()` 的实现，每个函数的接口仍需保持一致。 也就是说每个函数应该仍然没有任何参数，也不需要返回一个值。

    ```cpp
    #include &lt;iostream&gt; 

    int sum = 0; 

    void calculate() 
    { 
      for (int i = 0; i &lt; 1000; ++i) 
        sum += i; 
    } 

    void print() 
    { 
      std::cout &lt;&lt; sum &lt;&lt; std::endl; 
    } 

    void thread() 
    { 
      calculate(); 
      print(); 
    } 

    int main() 
    { 
      thread(); 
    } 
    ```

    *   下载源代码