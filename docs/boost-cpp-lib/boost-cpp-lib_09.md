# 第八章 进程间通讯

### 目录

*   8.1 概述
*   8.2 共享内存
*   8.3 托管共享内存
*   8.4 同步
*   8.5 练习

![](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 该书采用 [Creative Commons License](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 授权

## 8.1\. 概述

进程间通讯描述的是同一台计算机的不同应用程序之间的数据交换机制。 但不包括网络通讯方式。 如果需要经由网络，在彼此运行在不同计算机上的应用程序之间交换数据，请看第七章 *异步输入输出*，该章讲述了 Boost.Asio 库。

本章展示了 [Boost.Interprocess](http://www.boost.org/libs/interprocess/) 库，它包括众多的类，这些类提供了操作系统相关的进程间通讯接口的抽象层。 虽然不同操作系统的进程间通讯概念非常相近，但接口的变化却很大。 Boost.Interprocess 库使通过 C++使用这些功能成为可能。

虽然 Boost.Asio 也可以用来在同一台计算机的应用程序间交换数据，但是使用 Boost.Interprocess 库通常性能更好。 Boost.Interprocess 库实际上是使用操作系统的功能优化了同一台计算机的应用程序之间数据交换，所以它应该是任何不需要网络时应用程序间数据交换的首选。

## 8.2\. 共享内存

共享内存通常是进程间通讯最快的形式。 它提供一块在应用程序间共享的内存区域。 一个应用能够在另一个应用读取数据时写数据。

这样一块内存区用 Boost.Interprocess 的 `boost::interprocess::shared_memory_object` 类表示。 为使用这个类，需要包含 `boost/interprocess/shared_memory_object.hpp` 头文件。

```cpp
#include <boost/interprocess/shared_memory_object.hpp> 
#include <iostream> 

int main() 
{ 
  boost::interprocess::shared_memory_object shdmem(boost::interprocess::open_or_create, "Highscore", boost::interprocess::read_write); 
  shdmem.truncate(1024); 
  std::cout << shdmem.get_name() << std::endl; 
  boost::interprocess::offset_t size; 
  if (shdmem.get_size(size)) 
    std::cout << size << std::endl; 
} 
```

*   下载源代码

`boost::interprocess::shared_memory_object` 的构造函数需要三个参数。 第一个参数指定共享内存是要创建或打开。 上面的例子实际上是指定了两种方式：用 `boost::interprocess::open_or_create` 作为参数，共享内存如果存在就将其打开，否则创建之。

假设之前已经创建了共享内存，现打开前面已经创建的共享内存。 为了唯一标识一块共享内存，需要为其指定一个名称，传递给 `boost::interprocess::shared_memory_object` 构造函数的第二个参数指定了这个名称。

第三个，也就是最后一个参数指示应用程序如何访问共享内存。 例子应用程序能够读写共享内存，这是因为最后的一个参数是 `boost::interprocess::read_write`。

在创建一个 `boost::interprocess::shared_memory_object` 类型的对象后，相应的共享内存就在操作系统中建立了。 可是此共享内存区域的大小被初始化为 0.为了使用这块区域，需要调用 `truncate()` 函数，以字节为单位传递请求的共享内存的大小。 对于上面的例子，共享内存提供了 1,024 字节的空间。

请注意，`truncate()` 函数只能在共享内存以 `boost::interprocess::read_write` 方式打开时调用。 如果不是以此方式打开，将抛出 `boost::interprocess::interprocess_exception` 异常。

为了调整共享内存的大小，`truncate()` 函数可以被重复调用。

在创建共享内存后，`get_name()` 和 `get_size()` 函数可以分别用来查询共享内存的名称和大小。

由于共享内存被用于应用程序之间交换数据，所以每个应用程序需要映射共享内存到自己的地址空间上，这是通过 `boost::interprocess::mapped_region` 类实现的。

你也许有些奇怪，为了访问共享内存，要使用两个类。 是的，`boost::interprocess::mapped_region` 还能映射不同的对象到具体应用的地址空间。 如 Boost.Interprocess 提供 `boost::interprocess::file_mapping` 类实际上代表特定文件的共享内存。 所以 `boost::interprocess::file_mapping` 类型的对象对应一个文件。 向这个对象写入的数据将自动保存关联的物理文件上。 由于 `boost::interprocess::file_mapping` 不必加载整个文件，但却可以使用 `boost::interprocess::mapped_region` 将任意部分映射到地址空间，所以就能处理几个 GB 的文件，而这个文件在 32 位系统上是不能全部加载到内存上的。

```cpp
#include <boost/interprocess/shared_memory_object.hpp> 
#include <boost/interprocess/mapped_region.hpp> 
#include <iostream> 

int main() 
{ 
  boost::interprocess::shared_memory_object shdmem(boost::interprocess::open_or_create, "Highscore", boost::interprocess::read_write); 
  shdmem.truncate(1024); 
  boost::interprocess::mapped_region region(shdmem, boost::interprocess::read_write); 
  std::cout << std::hex << "0x" << region.get_address() << std::endl; 
  std::cout << std::dec << region.get_size() << std::endl; 
  boost::interprocess::mapped_region region2(shdmem, boost::interprocess::read_only); 
  std::cout << std::hex << "0x" << region2.get_address() << std::endl; 
  std::cout << std::dec << region2.get_size() << std::endl; 
} 
```

*   下载源代码

为了使用 `boost::interprocess::mapped_region` 类，需要包含 `boost/interprocess/mapped_region.hpp` 头文件。 `boost::interprocess::mapped_region` 的构造函数的第一个参数必须是 `boost::interprocess::shared_memory_object` 类型的对象。 第二个参数指示此内存区域对应用程序来说，是只读或是可写的。

上面的例子创建了两个 `boost::interprocess::mapped_region` 类型的对象。 名为"Highscore"的共享内存，被映射到进程的地址空间两次。 通过 `get_address()` 和 `get_size()` 这两个函数获得共享内存的地址和大小写到标准标准输出流中。 在这两种情况下，`get_size()` 的返回值都是`1024`，而 `get_address()` 的返回值是不同的。

下面的例子使用共享内存写入并读取一个数字。

```cpp
#include <boost/interprocess/shared_memory_object.hpp> 
#include <boost/interprocess/mapped_region.hpp> 
#include <iostream> 

int main() 
{ 
  boost::interprocess::shared_memory_object shdmem(boost::interprocess::open_or_create, "Highscore", boost::interprocess::read_write); 
  shdmem.truncate(1024); 
  boost::interprocess::mapped_region region(shdmem, boost::interprocess::read_write); 
  int *i1 = static_cast<int*>(region.get_address()); 
  *i1 = 99; 
  boost::interprocess::mapped_region region2(shdmem, boost::interprocess::read_only); 
  int *i2 = static_cast<int*>(region2.get_address()); 
  std::cout << *i2 << std::endl; 
} 
```

*   下载源代码

通过变量 `region`, 数值 99 被写到共享内存的开始处。 然后变量 `region2` 访问共享内存的同一个位置，并将数值写入到标准输出流中。 正如前面例子的 `get_address()` 函数的返回值所见，虽然变量 `region` 和 `region2` 表示的是该进程内不同的内存区域，但由于两个内存区域底层实际访问的是同一块共享内存，所以程序打印出`99`。

通常，不会在同一个应用程序内使用多个 `boost::interprocess::mapped_region` 访问同一块共享内存。 实际上在同一个应用程序内将同一个共享内存映射到不同的内存区域上没有多大的意义，上面的例子只用于说明的目的。

为了删除指定的共享内存，`boost::interprocess::shared_memory_object` 对象提供了静态的 `remove()` 函数，此函数带有一个要被删除的共享内存名称的参数。

Boost.Interprocess 类的 RAII 概念支持，明显来自关于智能指针的章节，并使用了另外的一个类名称 `boost::interprocess::remove_shared_memory_on_destroy`。 它的构造函数需要一个已经存在的共享内存的名称。 如果这个类的对象被销毁了，那么在析构函数中会自动删除共享内存的容器。

请注意构造函数并不创建或打开共享内存，所以，这个类并不是典型 RAII 概念的代表。

```cpp
#include <boost/interprocess/shared_memory_object.hpp> 
#include <iostream> 

int main() 
{ 
  bool removed = boost::interprocess::shared_memory_object::remove("Highscore"); 
  std::cout << removed << std::endl; 
} 
```

*   下载源代码

如果 `remove()` 没有被调用, 那么，即使进程终止，共享内存还会一直存在，而不论共享内存的删除是否依赖底层操作系统。 多数 Unix 操作系统，包括 Linux，一旦系统重新启动，都会自动删除共享内存，在 Windows 或 Mac OS X 上，`remove()` 必须调用，这两种系统实际上将共享内存存储在持久化的文件上，此文件在系统重启后还是存在的。

Windows 提供了一种特别的共享内存，它可以在最后一个使用它的应用程序终止后自动删除。 为了使用它，提供了 `boost::interprocess::windows_shared_memory` 类，定义在 `boost/interprocess/windows_shared_memory.hpp` 文件中。

```cpp
#include <boost/interprocess/windows_shared_memory.hpp> 
#include <boost/interprocess/mapped_region.hpp> 
#include <iostream> 

int main() 
{ 
  boost::interprocess::windows_shared_memory shdmem(boost::interprocess::open_or_create, "Highscore", boost::interprocess::read_write, 1024); 
  boost::interprocess::mapped_region region(shdmem, boost::interprocess::read_write); 
  int *i1 = static_cast<int*>(region.get_address()); 
  *i1 = 99; 
  boost::interprocess::mapped_region region2(shdmem, boost::interprocess::read_only); 
  int *i2 = static_cast<int*>(region2.get_address()); 
  std::cout << *i2 << std::endl; 
} 
```

*   下载源代码

请注意，`boost::interprocess::windows_shared_memory` 类没有提供 `truncate()` 函数，而是在构造函数的第四个参数传递共享内存的大小。

即使 `boost::interprocess::windows_shared_memory` 类是不可移植的，且只能用于 Windows 系统，但使用这种特别的共享内存在不同应用之间交换数据，它还是非常有用的。

## 8.3\. 托管共享内存

上一节介绍了用来创建和管理共享的 `boost::interprocess::shared_memory_object` 类。 实际上，由于这个类需要按单个字节的方式读写共享内存，所以这个类几乎不用。 概念上来讲，C++改善了类对象的创建并隐藏了它们存储在内存中哪里，是怎们存储的这些细节。

Boost.Interprocess 提供了一个名为“托管共享内存”的概念，通过定义在 `boost/interprocess/managed_shared_memory.hpp` 文件中的 `boost::interprocess::managed_shared_memory` 类提供。 这个类的目的是，对于需要分配到共享内存上的对象，它能够以内存申请的方式初始化，并使其自动为使用同一个共享内存的其他应用程序可用。

```cpp
#include <boost/interprocess/managed_shared_memory.hpp> 
#include <iostream> 

int main() 
{ 
  boost::interprocess::shared_memory_object::remove("Highscore"); 
  boost::interprocess::managed_shared_memory managed_shm(boost::interprocess::open_or_create, "Highscore", 1024); 
  int *i = managed_shm.construct<int>("Integer")(99); 
  std::cout << *i << std::endl; 
  std::pair<int*, std::size_t> p = managed_shm.find<int>("Integer"); 
  if (p.first) 
    std::cout << *p.first << std::endl; 
} 
```

*   下载源代码

上面的例子打开名为 "Highscore" 大小为 1,024 字节的共享内存，如果它不存在，它会被自动地创建。

在常规的共享内存中，为了读写数据，单个字节被直接访问，托管共享内存使用诸如 `construct()` 函数，此函数要求一个数据类型作为模板参数，此例中声明的是 `int` 类型，函数缺省要求一个名称来表示在共享内存中创建的对象。 此例中使用的名称是 "Integer"。

由于 `construct()` 函数返回一个代理对象，为了初始化创建的对象，可以传递参数给此函数。 语法看上去像调用一个构造函数。 这就确保了对象不仅能在共享内存上创建，还能够按需要的方式初始化它。

为了访问托管共享内存上的一个特定对象，用 `find()` 函数。 通过传递要查找对象的名称，返回或者是一个指向这个特定对象的指针，或者是 0 表示给定名称的对象没有找到。

正如前面例子中所见，`find()` 实际返回的是 `std::pair` 类型的对象，`first` 属性提供的是指向对象的指针，那么 `second` 属性提供的是什么呢？

```cpp
#include <boost/interprocess/managed_shared_memory.hpp> 
#include <iostream> 

int main() 
{ 
  boost::interprocess::shared_memory_object::remove("Highscore"); 
  boost::interprocess::managed_shared_memory managed_shm(boost::interprocess::open_or_create, "Highscore", 1024); 
  int *i = managed_shm.construct<int>("Integer")10; 
  std::cout << *i << std::endl; 
  std::pair<int*, std::size_t> p = managed_shm.find<int>("Integer"); 
  if (p.first) 
  { 
    std::cout << *p.first << std::endl; 
    std::cout << p.second << std::endl; 
  } 
} 
```

*   下载源代码

这次，通过在 `construct()` 函数后面给以用方括号括住的数字 10，创建了一个 10 个元素的 `int` 类型的数组。 将 `second` 属性写到标准输出流，同样是这个数字`10`。 使用这个属性，`find()` 函数返回的对象能够区分是单个对象还是数组对象。 对于前者，`second` 的值是 1，而对于后者，它的值是数组元素的个数。

请注意数组中的所有元素被初始化为数值 99。 不可能每个元素初始化为不同的值。

如果给定名称的对象已经在托管的共享内存中存在，那么 `construct()` 将会失败。 在这种情况下，`construct()` 返回值是 0。 如果存在的对象即使存在也可以被重用，`find_or_construct()` 函数可以调用，此函数返回一个指向它找到的对象的指针。 此时没有初始化动作发生。

其他可以导致 `construct()` 失败的情况如下例所示。

```cpp
#include <boost/interprocess/managed_shared_memory.hpp> 
#include <iostream> 

int main() 
{ 
  try 
  { 
    boost::interprocess::shared_memory_object::remove("Highscore"); 
    boost::interprocess::managed_shared_memory managed_shm(boost::interprocess::open_or_create, "Highscore", 1024); 
    int *i = managed_shm.construct<int>("Integer")4096; 
  } 
  catch (boost::interprocess::bad_alloc &ex) 
  { 
    std::cerr << ex.what() << std::endl; 
  } 
} 
```

*   下载源代码

应用程序尝试创建一个 `int` 类型的，包含 4,096 个元素的数组。 然而，共享内存只有 1,024 字节，于是由于共享内存不能提供请求的内存，而抛出 `boost::interprocess::bad_alloc` 类型的异常。

一旦对象已经在共享内存中创建，它们可以用 `destroy()` 函数删除。

```cpp
#include <boost/interprocess/managed_shared_memory.hpp> 
#include <iostream> 

int main() 
{ 
  boost::interprocess::shared_memory_object::remove("Highscore"); 
  boost::interprocess::managed_shared_memory managed_shm(boost::interprocess::open_or_create, "Highscore", 1024); 
  int *i = managed_shm.find_or_construct<int>("Integer")(99); 
  std::cout << *i << std::endl; 
  managed_shm.destroy<int>("Integer"); 
  std::pair<int*, std::size_t> p = managed_shm.find<int>("Integer"); 
  std::cout << p.first << std::endl; 
} 
```

*   下载源代码

由于它是一个参数的，要被删除对象的名称传递给 `destroy()` 函数。 如果需要，可以检查此函数的 `bool` 类型的返回值，以确定是否给定的对象被找到并成功删除。 由于对象如果被找到总是被删除，所以返回值 `false` 表示给定名称的对象没有找到。

除了 `destroy()` 函数，还提供了另外一个函数 `destroy_ptr()`，它能够传递一个托管共享内存中的对象的指针，它也能用来删除数组。

由于托管内存很容易用来存储在不同应用程序之间共享的对象，那么很自然就会使用到来自 C++标准模板库的容器了。 这些容器需要用 `new` 这种方式来分配各自需要的内存。 为了在托管共享内存上使用这些容器，这就需要更加仔细地在共享内存上分配内存。

可惜的是，许多 C++标准模板库的实现并不太灵活，不能够提供 Boost.Interprocess 使用 `std::string` 或 `std::list` 的容器。 移植到 Microsoft Visual Studio 2008 的标准模板库实现就是一个例子。

为了允许开发人员可以使用这些有名的来自 C++标准的容器，Boost.Interprocess 在命名空间 `boost::interprocess` 下，提供了它们的更灵活的实现方式。 如，`boost::interprocess::string` 的行为实际上对应的是 `std::string`，优点是它的对象能够安全地存储在托管共享内存上。

```cpp
#include <boost/interprocess/managed_shared_memory.hpp> 
#include <boost/interprocess/allocators/allocator.hpp> 
#include <boost/interprocess/containers/string.hpp> 
#include <iostream> 

int main() 
{ 
  boost::interprocess::shared_memory_object::remove("Highscore"); 
  boost::interprocess::managed_shared_memory managed_shm(boost::interprocess::open_or_create, "Highscore", 1024); 
  typedef boost::interprocess::allocator<char, boost::interprocess::managed_shared_memory::segment_manager> CharAllocator; 
  typedef boost::interprocess::basic_string<char, std::char_traits<char>, CharAllocator> string; 
  string *s = managed_shm.find_or_construct<string>("String")("Hello!", managed_shm.get_segment_manager()); 
  s->insert(5, ", world"); 
  std::cout << *s << std::endl; 
} 
```

*   下载源代码

为了创建在托管共享内存上申请内存的字符串，必须为 Boost.Interprocess 提供的另外一个分配器定义对应的数据类型，而不是使用 C++标准提供的缺省分配器。

为了这个目的，Boost.Interprocess 在 `boost/interprocess/allocators/allocator.hpp` 文件中提供了 `boost::interprocess::allocator` 类的定义。 使用这个类，可以创建一个分配器，此分配器的内部使用的是“托管共享内存段管理器”。 段管理器负责管理位于托管共享内存之内的内存。 使用新建的分配器，与 string 相应的数据类型被定义了。 如上面所示，它使用 `boost::interprocess::basic_string` 而不是 `std::basic_string`。 上面例子中的新数据类型简单地命名为 `string`，它是基于 `boost::interprocess::basic_string` 并经过分配器访问段管理器。 为了让通过 `find_or_construct()` 创建的 `string` 特定实例，知道哪个段管理器应该被访问，相应的段管理器的指针传递给构造函数的第二个参数。

与 `boost::interprocess::string` 一起, Boost.Interprocess 还提供了许多其他 C++标准中已知的容器。 如, `boost::interprocess::vector` 和 `boost::interprocess::map`，分别定义在 `boost/interprocess/containers/vector.hpp` 和 `boost/interprocess/containers/map.hpp`文件中

无论何时同一个托管共享内存被不同的应用程序访问，诸如创建，查找和销毁对象的操作是自动同步的。 如果两个应用程序尝试在托管共享内存上创建不同名称的对象，访问相应地被串行化了。 为了立刻执行多个操作而不被其他应用程序的操作打断，可以使用 `atomic_func()` 函数。

```cpp
#include <boost/interprocess/managed_shared_memory.hpp> 
#include <boost/bind.hpp> 
#include <iostream> 

void construct_objects(boost::interprocess::managed_shared_memory &managed_shm) 
{ 
  managed_shm.construct<int>("Integer")(99); 
  managed_shm.construct<float>("Float")(3.14); 
} 

int main() 
{ 
  boost::interprocess::shared_memory_object::remove("Highscore"); 
  boost::interprocess::managed_shared_memory managed_shm(boost::interprocess::open_or_create, "Highscore", 1024); 
  managed_shm.atomic_func(boost::bind(construct_objects, boost::ref(managed_shm))); 
  std::cout << *managed_shm.find<int>("Integer").first << std::endl; 
  std::cout << *managed_shm.find<float>("Float").first << std::endl; 
} 
```

*   下载源代码

`atomic_func()` 需要一个无参数，无返回值的函数作为它的参数。 被传递的函数将以以一种确保排他访问托管共享内存的方式被调用，但仅限于对象的创建，查找和销毁操作。 如果另一个应用程序有一个指向托管内存中对象的指针，它还是可以使用这个指针修改该对象的。

Boost.Interprocess 也可以用来同步对象的访问。 由于 Boost.Interprocess 不知道在任意一个时间点谁可以访问某个对象，所以同步需要明确的状态标志，下一节介绍这些类提供的同步方式。

## 8.4\. 同步

Boost.Interprocess 允许多个应用程序并发使用共享内存。 由于共享内存被定义为在应用程序之间“共享”，所以 Boost.Interprocess 需要支持一些同步方式。

当考虑到同步的时候，Boost.Thread 当然浮现在脑海里。 正如在 第六章 *多线程* 所见，Boost.Thread 确实提供了各种概念，如互斥对象和条件变量来同步线程。 可惜的是，这些类只能用来同步同一个应用程序内的线程，它们不支持同步不同的应用程序。 由于二者面临的问题相同，所以在概念上没有什么差别。

当诸如互斥对象和条件变量等同步对象位于一个多线程的应用程序的同一地址空间内时，当然它们对于所有线程都是可以访问的，而在共享内存方面的问题是不同的应用程序需要在彼此之间正确地共享这些对象。 例如，如果一个应用程序创建一个互斥对象，它有时候需要从另外一个应用程序访问此对象。

Boost.Interprocess 提供了两种同步对象，匿名对象被直接存储在共享内存上，这使得他们自动对所有应用程序可用。 命名对象由操作系统管理，所以它们不存储在共享内存上，它们可以被应用程序通过名称访问。

接下来的例子通过 `boost::interprocess::named_mutex` 创建并使用一个命名互斥对象，此类定义在 `boost/interprocess/sync/named_mutex.hpp` 文件中。

```cpp
#include <boost/interprocess/managed_shared_memory.hpp> 
#include <boost/interprocess/sync/named_mutex.hpp> 
#include <iostream> 

int main() 
{ 
  boost::interprocess::managed_shared_memory managed_shm(boost::interprocess::open_or_create, "shm", 1024); 
  int *i = managed_shm.find_or_construct<int>("Integer")(); 
  boost::interprocess::named_mutex named_mtx(boost::interprocess::open_or_create, "mtx"); 
  named_mtx.lock(); 
  ++(*i); 
  std::cout << *i << std::endl; 
  named_mtx.unlock(); 
} 
```

*   下载源代码

除了一个参数用来指定互斥对象是被创建或者打开之外，`boost::interprocess::named_mutex` 的构造函数还需要一个名称参数。 每个知道此名称的应用程序能够访问这同一个对象。 为了获得对位于共享内存中数据的访问，应用程序需要通过调用 `lock()` 函数获得互斥对象的拥有关系。 由于互斥对象在任意时刻只能被一个应用程序拥有，其他应用程序需要等待，直到互斥对象被第一个应用程序使用 `lock()` 函数释放。 一旦应用程序获得互斥对象的所有权，它可以获得互斥对象保护的资源的排他访问。 在上面例子中，资源是`int`类的变量被递增并写到标准输出流中。

如果应用程序被启动多次，每个实例都会打印出和前一个值比较递增 1 的值。 感谢互斥对象，访问共享内存和变量本身在多个应用程序之间是同步的。

接下来的应用程序使用了定义在 `boost/interprocess/sync/interprocess_mutex.hpp` 文件中的 `boost::interprocess::interprocess_mutex` 类的匿名对象。 为了可以被所有应用程序访问，它存储在共享内存中。

```cpp
#include <boost/interprocess/managed_shared_memory.hpp> 
#include <boost/interprocess/sync/interprocess_mutex.hpp> 
#include <iostream> 

int main() 
{ 
  boost::interprocess::managed_shared_memory managed_shm(boost::interprocess::open_or_create, "shm", 1024); 
  int *i = managed_shm.find_or_construct<int>("Integer")(); 
  boost::interprocess::interprocess_mutex *mtx = managed_shm.find_or_construct<boost::interprocess::interprocess_mutex>("mtx")(); 
  mtx->lock(); 
  ++(*i); 
  std::cout << *i << std::endl; 
  mtx->unlock(); 
} 
```

*   下载源代码

这个应用程序的行为确实和前一个有点像。 唯一的不同是这次互斥对象通过用 `boost::interprocess::managed_shared_memory` 类的 `construct()` 或 `find_or_construct()` 函数被直接存储在共享内存中。

除了 `lock()` 函数，`boost::interprocess::named_mutex` 和 `boost::interprocess::interprocess_mutex` 还提供了 `try_lock()` 和 `timed_lock()` 函数。 它们的行为和 Boost.Thread 提供的互斥对象相对应。

在需要递归互斥对象的时候，Boost.Interprocess 提供 `boost::interprocess::named_recursive_mutex` 和 `boost::interprocess::interprocess_mutex` 两个对象可供使用。

在互斥对象保证共享资源的排他访问的时候，条件变量控制了在什么时候，谁必须具有排他访问权。 一般来讲，Boost.Interprocess 和 Boost.Thread 提供的条件变量工作方式相同。 它们有非常相似的接口，使熟悉 Boost.Thread 的用户在使用 Boost.Interprocess 的这些条件变量时立刻有一种自在的感觉。

```cpp
#include <boost/interprocess/managed_shared_memory.hpp> 
#include <boost/interprocess/sync/named_mutex.hpp> 
#include <boost/interprocess/sync/named_condition.hpp> 
#include <boost/interprocess/sync/scoped_lock.hpp> 
#include <iostream> 

int main() 
{ 
  boost::interprocess::managed_shared_memory managed_shm(boost::interprocess::open_or_create, "shm", 1024); 
  int *i = managed_shm.find_or_construct<int>("Integer")(0); 
  boost::interprocess::named_mutex named_mtx(boost::interprocess::open_or_create, "mtx"); 
  boost::interprocess::named_condition named_cnd(boost::interprocess::open_or_create, "cnd"); 
  boost::interprocess::scoped_lock<boost::interprocess::named_mutex> lock(named_mtx); 
  while (*i < 10) 
  { 
    if (*i % 2 == 0) 
    { 
      ++(*i); 
      named_cnd.notify_all(); 
      named_cnd.wait(lock); 
    } 
    else 
    { 
      std::cout << *i << std::endl; 
      ++(*i); 
      named_cnd.notify_all(); 
      named_cnd.wait(lock); 
    } 
  } 
  named_cnd.notify_all(); 
  boost::interprocess::shared_memory_object::remove("shm"); 
  boost::interprocess::named_mutex::remove("mtx"); 
  boost::interprocess::named_condition::remove("cnd"); 
} 
```

*   下载源代码

例子中使用的条件变量的类型 `boost::interprocess::named_condition`，定义在 `boost/interprocess/sync/named_condition.hpp` 文件中。 由于它是命名变量，所以它不需要存储在共享内存。

应用程序使用 `while` 循环递增一个存储在共享内存中的 `int` 类型变量而变量是在每个循环内重复递增，而它只在每两个循环时写出到标准输出中：写出的只能是奇数。

每次，在变量递增 1 之后，条件变量 `named_cnd` 的 `wait ()`函数被调用。 一个称作锁，在此例中是变量 `lock` 被传递给此函数。 这个锁和 Boost.Thread 中的锁含义相同：基于 RAII 概念的在构造函数中获得互斥对象的所有权,并在析构函数中释放它。

在 `while` 之前创建的锁因而在整个应用程序执行期间拥有互斥对象的所有权。 可是，如果作为一个参数传递给 `wait()` 函数，它会被自动释放。

条件变量常常用来等待一个信号，此信号会指示等待现在到了。 同步是通过 `wait()` 和 `notify_all()` 函数控制的。 如果一个应用程序调用 `wait()` 函数，一直到对应的条件变量的 `notify_all()` 函数被调用，相应的互斥对象的所有权才会被被释放。

如果启动此程序，它看上去什么也没做：而只是变量在 `while` 循环内从 0 递增到 1，然后应用程序使用 `wait()` 等待信号。 为了提供这个信号，应用程序需要再启动第二个实例。

应用程序的第二个实例将会在进入 `while` 循环之前，尝试获得同一个互斥对象的所有权。 这肯定是成功的，由于应用程序的第一个实例通过调用 `wait()` 释放了互斥对象的所有权。 因为变量已经递增了一次，第二个实例现在会执行 `if` 表达式的 `else` 分支，这使得在递增 1 之前将当前值写到标准输出流。

现在，第二个实例也调用了 `wait()` 函数，可是，在调用之前,它调用了 `notify_all()` 函数，这对于两个实例正确协作是非常重要的顺序。 第一个实例被通知并再次尝试获得互斥对象的所有权，虽然现在它还被第二个实例所拥有。 由于第二个实例在调用 `notify_all()` 之后调用了 `wait()`，这自动释放了所有权，第一个实例此时能够获得所有权。

两个实例交替地递增共享内存中的变量。 仅有一个实例将变量值写到标准输出流。 只要变量值到达 10，`while` 循环结束。 为了让其他实例不必永远等待信号， `notify_all()` 函数在循环之后又被调用了一次。 在终止之前，共享内存，互斥对象和条件变量都被销毁。

就像有两种互斥对象，即必须存储在共享内存中匿名类型和命名类型，也存在两种类型的条件变量。 现在用匿名条件变量重写上面的例子。

```cpp
#include <boost/interprocess/managed_shared_memory.hpp> 
#include <boost/interprocess/sync/interprocess_mutex.hpp> 
#include <boost/interprocess/sync/interprocess_condition.hpp> 
#include <boost/interprocess/sync/scoped_lock.hpp> 
#include <iostream> 

int main() 
{ 
  try 
  { 
    boost::interprocess::managed_shared_memory managed_shm(boost::interprocess::open_or_create, "shm", 1024); 
    int *i = managed_shm.find_or_construct<int>("Integer")(0); 
    boost::interprocess::interprocess_mutex *mtx = managed_shm.find_or_construct<boost::interprocess::interprocess_mutex>("mtx")(); 
    boost::interprocess::interprocess_condition *cnd = managed_shm.find_or_construct<boost::interprocess::interprocess_condition>("cnd")(); 
    boost::interprocess::scoped_lock<boost::interprocess::interprocess_mutex> lock(*mtx); 
    while (*i < 10) 
    { 
      if (*i % 2 == 0) 
      { 
        ++(*i); 
        cnd->notify_all(); 
        cnd->wait(lock); 
      } 
      else 
      { 
        std::cout << *i << std::endl; 
        ++(*i); 
        cnd->notify_all(); 
        cnd->wait(lock); 
      } 
    } 
    cnd->notify_all(); 
  } 
  catch (...) 
  { 
  } 
  boost::interprocess::shared_memory_object::remove("shm"); 
} 
```

*   下载源代码

这个应用程序的工作完全和前一个例子一样，为了递增变量 10 次，因而也需要启动两个实例。 两个例子之间的差别很小。 与是否使用匿名或命名条件变量根本没有什么关系。

处理互斥对象和条件变量，Boost.Interprocess 还提供了叫做信号量和文件锁。 信号量的行为和条件变量相似，除了它不能区别两种状态，但它确是基于计数器的。 文件锁有些像互斥对象，虽然它们不是关于内存的对象，但它们确是文件系统上关于文件的对象。

就像 Boost.Thread 能够区分不同的互斥对象和锁，Boost.Interprocess 也提供了几个互斥对象和锁。 例如，互斥对象不仅能被排他地拥有，也可以不排他地所有。 这在多个应用程序需要同时读而排他写的时候非常有用。 对于不同的互斥对象，可以使用不同的具有 RAII 概念的锁类。

请注意如果不使用匿名同步对象，那么名称应该是唯一的。 虽然互斥对象和条件变量是基于不同类的对象，但也不必总是认为操作系统独立的接口是由 Boost.Interprocess 区别对待的。 在 Windows 系统上，互斥对象和条件变量使用同样的系统函数。 如果这两种对象使用相同的名称，那么应用程序在 Windows 上将不会正确地址执行。

## 8.5\. 练习

You can buy [solutions to all exercises](http://en.highscore.de/shop/index.php?p=boost-solution) in this book as a ZIP file.

1.  创建一个通过共享内存通讯的客户端/服务器应用程序。 文件名称应该作为命令行参数传递给客户端应用程序。 这个文件存储在服务器端应用程序启动的目录下，这个文件应经由共享内存发送给服务器端应用程序。