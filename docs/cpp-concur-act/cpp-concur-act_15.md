# 附录 B 并发库简要对比

# 附录 B 并发库的简单比较

虽然，C++11 才开始正式支持并发，不过，高级编程语言都支持并发和多线程已经不是什么新鲜事了。例如，Java 在第一个发布版本中就支持多线程编程，在某些平台上也提供符合 POSIX C 标准的多线程接口，还有[Erlang](http://www.erlang.org/)支持消息的同步传递(有点类似于 MPI)。当然还有使用 C++类的库，比如 Boost，其将底层多线程接口进行包装，适用于任何给定的平台(不论是使用 POSIX C 的接口，或其他接口)，其对支持的平台会提供可移植接口。

这些库或者变成语言，已经写了很多多线程应用，并且在使用这些库写多线程代码的经验，可以借鉴到 C++中，本附录就对 Java，POSIX C，使用 Boost 线程库的 C++，以及 C++11 中的多线程工具进行简单的比较，当然也会交叉引用本书的相关章节。

| 特性 | 启动线程 | 互斥量 | 监视/等待谓词 | 原子操作和并发感知内存模型 | 线程安全容器 | Futures(期望) | 线程池 | 线程中断 |
| 章节引用 | 第二章 | 第三章 | 第四章 | 第五章 | 第六章和第七章 | 第四章 | 第九章 | 第九章 |
| C++11 | std::thread 和其成员函数 | std::mutex 类和其成员函数 | std::condition_variable | std::atomic_xxx 类型 | N/A | std::future<> | N/A | N/A |
| std::lock_guard<>模板 | std::condition_variable_any 类和其成员函数 | std::atomic<>类模板 | std::shared_future<> |
| std::unique_lock<>模板 | std::atomic_thread_fence()函数 | std::atomic_future<>类模板 |
| Boost 线程库 | boost::thread 类和成员函数 | boost::mutex 类和其成员函数 | boost::condition_variable 类和其成员函数 | N/A | N/A | boost::unique_future<>类模板 | N/A | boost::thread 类的 interrupt()成员函数 |
| boost::lock_guard<>类模板 | boost::condition_variable_any 类和其成员函数 | boost::shared_future<>类模板 |
| boost::unique_lock<>类模板 |
| POSIX C | pthread_t 类型相关的 API 函数 | pthread_mutex_t 类型相关的 API 函数 | pthread_cond_t 类型相关的 API 函数 | N/A | N/A | N/A | N/A | pthread_cancel() |
| pthread_create() | pthread_mutex_lock() | pthread_cond_wait() |
| pthread_detach() | pthread_mutex_unlock() | pthread_cond_timed_wait() |
| pthread_join() | 等等 | 等等 |
| Java | java.lang.thread 类 | synchronized 块 | java.lang.Object 类的 wait()和 notify()函数，用在内部 synchronized 块中 | java.util.concurrent.atomic 包中的 volatile 类型变量 | java.util.concurrent 包中的容器 | 与 java.util.concurrent.future 接口相关的类 | java.util.concurrent.ThreadPoolExecutor 类 | java.lang.Thread 类的 interrupt()函数 |