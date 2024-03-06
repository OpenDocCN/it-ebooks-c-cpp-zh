# 第二章 智能指针

# 第二章 智能指针

### 目录

*   2.1 概述
*   2.2 RAII
*   2.3 作用域指针
*   2.4 作用域数组
*   2.5 共享指针
*   2.6 共享数组
*   2.7 弱指针
*   2.8 介入式指针
*   2.9 指针容器
*   2.10 练习

![](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 该书采用 [Creative Commons License](http://creativecommons.org/licenses/by-nc-nd/3.0/de/deed.zh) 授权

## 2.1\. 概述

1998 年修订的第一版 C++标准只提供了一种智能指针： `std::auto_ptr` 。 它基本上就像是个普通的指针： 通过地址来访问一个动态分配的对象。 `std::auto_ptr` 之所以被看作是智能指针，是因为它会在析构的时候调用 `delete` 操作符来自动释放所包含的对象。 当然这要求在初始化的时候，传给它一个由 `new` 操作符返回的对象的地址。 既然 `std::auto_ptr` 的析构函数会调用 `delete` 操作符，它所包含的对象的内存会确保释放掉。 这是智能指针的一个优点。

当和异常联系起来时这就更加重要了：没有 `std::auto_ptr` 这样的智能指针，每一个动态分配内存的函数都需要捕捉所有可能的异常，以确保在异常传递给函数的调用者之前将内存释放掉。 Boost C++ 库 [Smart Pointers](http://www.boost.org/libs/smart_ptr/) 提供了许多可以用在各种场合的智能指针。

## 2.2\. RAII

智能指针的原理基于一个常见的习语叫做 RAII ：资源申请即初始化。 智能指针只是这个习语的其中一例——当然是相当重要的一例。 智能指针确保在任何情况下，动态分配的内存都能得到正确释放，从而将开发人员从这项任务中解放了出来。 这包括程序因为异常而中断，原本用于释放内存的代码被跳过的场景。 用一个动态分配的对象的地址来初始化智能指针，在析构的时候释放内存，就确保了这一点。 因为析构函数总是会被执行的，这样所包含的内存也将总是会被释放。

无论何时，一定得有第二条指令来释放之前另一条指令所分配的资源时，RAII 都是适用的。 许多的 C++ 应用程序都需要动态管理内存，因而智能指针是一种很重要的 RAII 类型。 不过 RAII 本身是适用于许多其它场景的。

```cpp
#include <windows.h> 

class windows_handle 
{ 
  public: 
    windows_handle(HANDLE h) 
      : handle_(h) 
    { 
    } 

    ~windows_handle() 
    { 
      CloseHandle(handle_); 
    } 

    HANDLE handle() const 
    { 
      return handle_; 
    } 

  private: 
    HANDLE handle_; 
}; 

int main() 
{ 
  windows_handle h(OpenProcess(PROCESS_SET_INFORMATION, FALSE, GetCurrentProcessId())); 
  SetPriorityClass(h.handle(), HIGH_PRIORITY_CLASS); 
} 
```

*   下载源代码

上面的例子中定义了一个名为 `windows_handle` 的类，它的析构函数调用了 `CloseHandle()` 函数。 这是一个 Windows API 函数，因而这个程序只能在 Windows 上运行。 在 Windows 上，许多资源在使用之前都要求打开。 这暗示着一旦资源不再使用之后就应该关闭。 `windows_handle` 类的机制能确保这一点。

`windows_handle` 类的实例以一个句柄来初始化。 Windows 使用句柄来唯一的标识资源。 比如说，`OpenProcess()` 函数返回一个 `HANDLE` 类型的句柄，通过该句柄可以访问当前系统中的进程。 在示例代码中，访问的是进程自己——换句话说就是应用程序本身。

我们通过这个返回的句柄提升了进程的优先级，这样它就能从调度器那里获得更多的 CPU 时间。 这里只是用于演示目的，并没什么实际的效应。 重要的一点是：通过 `OpenProcess()` 打开的资源不需要显示的调用 `CloseHandle()` 来关闭。 当然，应用程序终止时资源也会随之关闭。 然而，在更加复杂的应用程序里， `windows_handle` 类确保当一个资源不再使用时就能正确的关闭。 某个资源一旦离开了它的作用域——上例中 `h` 的作用域在 `main()` 函数的末尾——它的析构函数会被自动的调用，相应的资源也就释放掉了。

## 2.3\. 作用域指针

一个作用域指针独占一个动态分配的对象。 对应的类名为 `boost::scoped_ptr`，它的定义在 `boost/scoped_ptr.hpp` 中。 不像 `std::auto_ptr`，一个作用域指针不能传递它所包含的对象的所有权到另一个作用域指针。 一旦用一个地址来初始化，这个动态分配的对象将在析构阶段释放。

因为一个作用域指针只是简单保存和独占一个内存地址，所以 `boost::scoped_ptr` 的实现就要比 `std::auto_ptr` 简单。 在不需要所有权传递的时候应该优先使用 `boost::scoped_ptr` 。 在这些情况下，比起 `std::auto_ptr` 它是一个更好的选择，因为可以避免不经意间的所有权传递。

```cpp
#include <boost/scoped_ptr.hpp> 

int main() 
{ 
  boost::scoped_ptr<int> i(new int); 
  *i = 1; 
  *i.get() = 2; 
  i.reset(new int); 
} 
```

*   下载源代码

一经初始化，智能指针 `boost::scoped_ptr` 所包含的对象，可以通过类似于普通指针的接口来访问。 这是因为重载了相关的操作符 `operator*()`，`operator-&gt;()` 和 `operator bool()` 。 此外，还有 `get()` 和 `reset()` 方法。 前者返回所含对象的地址，后者用一个新的对象来重新初始化智能指针。 在这种情况下，新创建的对象赋值之前会先自动释放所包含的对象。

`boost::scoped_ptr` 的析构函数中使用 `delete` 操作符来释放所包含的对象。 这对 `boost::scoped_ptr` 所包含的类型加上了一条重要的限制。 `boost::scoped_ptr` 不能用动态分配的数组来做初始化，因为这需要调用 `delete[]` 来释放。 在这种情况下，可以使用下面将要介绍的 `boost:scoped_array` 类。

## 2.4\. 作用域数组

作用域数组的使用方式与作用域指针相似。 关键不同在于，作用域数组的析构函数使用 `delete[]` 操作符来释放所包含的对象。 因为该操作符只能用于数组对象，所以作用域数组必须通过动态分配的数组来初始化。

对应的作用域数组类名为 `boost::scoped_array`，它的定义在 `boost/scoped_array.hpp` 里。

```cpp
#include <boost/scoped_array.hpp> 

int main() 
{ 
  boost::scoped_array<int> i(new int[2]); 
  *i.get() = 1; 
  i[1] = 2; 
  i.reset(new int[3]); 
} 
```

*   下载源代码

`boost:scoped_array` 类重载了操作符 `operator[]()` 和 `operator bool()`。 可以通过 `operator[]()` 操作符访问数组中特定的元素，于是 `boost::scoped_array` 类型对象的行为就酷似它所含的数组。

正如 `boost::scoped_ptr` 那样, `boost:scoped_array` 也提供了 `get()` 和 `reset()` 方法，用来返回和重新初始化所含对象的地址。

## 2.5\. 共享指针

这是使用率最高的智能指针，但是 C++ 标准的第一版中缺少这种指针。 它已经作为技术报告 1（TR 1）的一部分被添加到标准里了。 如果开发环境支持的话，可以使用 `memory` 中定义的 `std::shared_ptr`。 在 Boost C++ 库里，这个智能指针命名为 `boost::shared_ptr`，定义在 `boost/shared_ptr.hpp` 里。

智能指针 `boost::shared_ptr` 基本上类似于 `boost::scoped_ptr`。 关键不同之处在于 `boost::shared_ptr` 不一定要独占一个对象。 它可以和其他 `boost::shared_ptr` 类型的智能指针共享所有权。 在这种情况下，当引用对象的最后一个智能指针销毁后，对象才会被释放。

因为所有权可以在 `boost::shared_ptr` 之间共享，任何一个共享指针都可以被复制，这跟 `boost::scoped_ptr` 是不同的。 这样就可以在标准容器里存储智能指针了——你不能在标准容器中存储 `std::auto_ptr`，因为它们在拷贝的时候传递了所有权。

```cpp
#include <boost/shared_ptr.hpp> 
#include <vector> 

int main() 
{ 
  std::vector<boost::shared_ptr<int> > v; 
  v.push_back(boost::shared_ptr<int>(new int(1))); 
  v.push_back(boost::shared_ptr<int>(new int(2))); 
} 
```

*   下载源代码

多亏了有 `boost::shared_ptr`，我们才能像上例中展示的那样，在标准容器中安全的使用动态分配的对象。 因为 `boost::shared_ptr` 能够共享它所含对象的所有权，所以保存在容器中的拷贝（包括容器在需要时额外创建的拷贝）都是和原件相同的。如前所述，`std::auto_ptr`做不到这一点，所以绝对不应该在容器中保存它们。

类似于 `boost::scoped_ptr`， `boost::shared_ptr` 类重载了以下这些操作符：`operator*()`，`operator-&gt;()` 和 `operator bool()`。另外还有 `get()` 和 `reset()` 函数来获取和重新初始化所包含的对象的地址。

```cpp
#include <boost/shared_ptr.hpp> 

int main() 
{ 
  boost::shared_ptr<int> i1(new int(1)); 
  boost::shared_ptr<int> i2(i1); 
  i1.reset(new int(2)); 
} 
```

*   下载源代码

本例中定义了 2 个共享指针 `i1` 和 `i2`，它们都引用到同一个 `int` 类型的对象。`i1` 通过 `new` 操作符返回的地址显示的初始化，`i2` 通过 `i1` 拷贝构造而来。 `i1` 接着调用 `reset()`，它所包含的整数的地址被重新初始化。不过它之前所包含的对象并没有被释放，因为 `i2` 仍然引用着它。 智能指针 `boost::shared_ptr` 记录了有多少个共享指针在引用同一个对象，只有在最后一个共享指针销毁时才会释放这个对象。

默认情况下，`boost::shared_ptr` 使用 `delete` 操作符来销毁所含的对象。 然而，具体通过什么方法来销毁，是可以指定的，就像下面的例子里所展示的：

```cpp
#include <boost/shared_ptr.hpp> 
#include <windows.h> 

int main() 
{ 
  boost::shared_ptr<void> h(OpenProcess(PROCESS_SET_INFORMATION, FALSE, GetCurrentProcessId()), CloseHandle); 
  SetPriorityClass(h.get(), HIGH_PRIORITY_CLASS); 
} 
```

*   下载源代码

`boost::shared_ptr` 的构造函数的第二个参数是一个普通函数或者函数对象，该参数用来销毁所含的对象。 在本例中，这个参数是 Windows API 函数 `CloseHandle()`。 当变量 `h` 超出它的作用域时，调用的是这个函数而不是 `delete` 操作符来销毁所含的对象。 为了避免编译错误，该函数只能带一个 `HANDLE` 类型的参数， `CloseHandle()` 正好符合要求。

该例和本章稍早讲述 RAII 习语时所用的例子的运行是一样的。 然而，本例没有单独定义一个 `windows_handle` 类，而是利用了 `boost::shared_ptr` 的特性，给它的构造函数传递一个方法，这个方法会在共享指针超出它的作用域时自动调用。

## 2.6\. 共享数组

共享数组的行为类似于共享指针。 关键不同在于共享数组在析构时，默认使用 `delete[]` 操作符来释放所含的对象。 因为这个操作符只能用于数组对象，共享数组必须通过动态分配的数组的地址来初始化。

共享数组对应的类型是 `boost::shared_array`，它的定义在 `boost/shared_array.hpp` 里。

```cpp
#include <boost/shared_array.hpp> 
#include <iostream> 

int main() 
{ 
  boost::shared_array<int> i1(new int[2]); 
  boost::shared_array<int> i2(i1); 
  i1[0] = 1; 
  std::cout << i2[0] << std::endl; 
} 
```

*   下载源代码

就像共享指针那样，所含对象的所有权可以跟其他共享数组来共享。 这个例子中定义了 2 个变量 `i1` 和 `i2`，它们引用到同一个动态分配的数组。`i1` 通过 `operator[]()` 操作符保存了一个整数 1——这个整数可以被 `i2` 引用，比如打印到标准输出。

和本章中所有的智能指针一样，`boost::shared_array` 也同样提供了 `get()` 和 `reset()` 方法。 另外还重载了 `operator bool()`。

## 2.7\. 弱指针

到目前为止介绍的各种智能指针都能在不同的场合下独立使用。 相反，弱指针只有在配合共享指针一起使用时才有意义。 弱指针 `boost::weak_ptr` 的定义在 `boost/weak_ptr.hpp` 里。

```cpp
#include <windows.h> 
#include <boost/shared_ptr.hpp> 
#include <boost/weak_ptr.hpp> 
#include <iostream> 

DWORD WINAPI reset(LPVOID p) 
{ 
  boost::shared_ptr<int> *sh = static_cast<boost::shared_ptr<int>*>(p); 
  sh->reset(); 
  return 0; 
} 

DWORD WINAPI print(LPVOID p) 
{ 
  boost::weak_ptr<int> *w = static_cast<boost::weak_ptr<int>*>(p); 
  boost::shared_ptr<int> sh = w->lock(); 
  if (sh) 
    std::cout << *sh << std::endl; 
  return 0; 
} 

int main() 
{ 
  boost::shared_ptr<int> sh(new int(99)); 
  boost::weak_ptr<int> w(sh); 
  HANDLE threads[2]; 
  threads[0] = CreateThread(0, 0, reset, &sh, 0, 0); 
  threads[1] = CreateThread(0, 0, print, &w, 0, 0); 
  WaitForMultipleObjects(2, threads, TRUE, INFINITE); 
} 
```

*   下载源代码

`boost::weak_ptr` 必定总是通过 `boost::shared_ptr` 来初始化的。一旦初始化之后，它基本上只提供一个有用的方法: `lock()`。此方法返回的`boost::shared_ptr` 与用来初始化弱指针的共享指针共享所有权。 如果这个共享指针不含有任何对象，返回的共享指针也将是空的。

当函数需要一个由共享指针所管理的对象，而这个对象的生存期又不依赖于这个函数时，就可以使用弱指针。 只要程序中还有一个共享指针掌管着这个对象，函数就可以使用该对象。 如果共享指针复位了，就算函数里能得到一个共享指针，对象也不存在了。

上例的 `main()` 函数中，通过 Windows API 创建了 2 个线程。 于是乎，该例只能在 Windows 平台上编译运行。

第一个线程函数 `reset()` 的参数是一个共享指针的地址。 第二个线程函数 `print()` 的参数是一个弱指针的地址。 这个弱指针是之前通过共享指针初始化的。

一旦程序启动之后，`reset()` 和 `print()` 就都开始执行了。 不过执行顺序是不确定的。 这就导致了一个潜在的问题：`reset()` 线程在销毁对象的时候`print()` 线程可能正在访问它。

通过调用弱指针的 `lock()` 函数可以解决这个问题：如果对象存在，那么 `lock()` 函数返回的共享指针指向这个合法的对象。否则，返回的共享指针被设置为 0，这等价于标准的 null 指针。

弱指针本身对于对象的生存期没有任何影响。 `lock()` 返回一个共享指针，`print()` 函数就可以安全的访问对象了。 这就保证了——即使另一个线程要释放对象——由于我们有返回的共享指针，对象依然存在。

## 2.8\. 介入式指针

大体上，介入式指针的工作方式和共享指针完全一样。 `boost::shared_ptr` 在内部记录着引用到某个对象的共享指针的数量，可是对介入式指针来说，程序员就得自己来做记录。 对于框架对象来说这就特别有用，因为它们记录着自身被引用的次数。

介入式指针 `boost::intrusive_ptr` 定义在 `boost/intrusive_ptr.hpp` 里。

```cpp
#include <boost/intrusive_ptr.hpp> 
#include <atlbase.h> 
#include <iostream> 

void intrusive_ptr_add_ref(IDispatch *p) 
{ 
  p->AddRef(); 
} 

void intrusive_ptr_release(IDispatch *p) 
{ 
  p->Release(); 
} 

void check_windows_folder() 
{ 
  CLSID clsid; 
  CLSIDFromProgID(CComBSTR("Scripting.FileSystemObject"), &clsid); 
  void *p; 
  CoCreateInstance(clsid, 0, CLSCTX_INPROC_SERVER, __uuidof(IDispatch), &p); 
  boost::intrusive_ptr<IDispatch> disp(static_cast<IDispatch*>(p)); 
  CComDispatchDriver dd(disp.get()); 
  CComVariant arg("C:\\Windows"); 
  CComVariant ret(false); 
  dd.Invoke1(CComBSTR("FolderExists"), &arg, &ret); 
  std::cout << (ret.boolVal != 0) << std::endl; 
} 

void main() 
{ 
  CoInitialize(0); 
  check_windows_folder(); 
  CoUninitialize(); 
} 
```

*   下载源代码

上面的例子中使用了 COM（组件对象模型）提供的函数，于是乎只能在 Windows 平台上编译运行。 COM 对象是使用 `boost::intrusive_ptr` 的绝佳范例，因为 COM 对象需要记录当前有多少指针引用着它。 通过调用 `AddRef()` 和 `Release()` 函数，内部的引用计数分别增 1 或者减 1。当引用计数为 0 时，COM 对象自动销毁。

在 `intrusive_ptr_add_ref()` 和 `intrusive_ptr_release()` 内部调用 `AddRef()` 和 `Release()` 这两个函数，来增加或减少相应 COM 对象的引用计数。 这个例子中用到的 COM 对象名为 'FileSystemObject'，在 Windows 上它是默认可用的。通过这个对象可以访问底层的文件系统，比如检查一个给定的目录是否存在。 在上例中，我们检查 `C:\Windows` 目录是否存在。 具体它在内部是怎么实现的，跟 `boost::intrusive_ptr` 的功能无关，完全取决于 COM。 关键点在于一旦介入式指针 `disp` 离开了它的作用域——`check_windows_folder()` 函数的末尾，函数 `intrusive_ptr_release()` 将会被自动调用。 这将减少 COM 对象 'FileSystemObject' 的内部引用计数到 0，于是该对象就销毁了。

## 2.9\. 指针容器

在你见过 Boost C++ 库的各种智能指针之后，应该能够编写安全的代码，来使用动态分配的对象和数组。多数时候，这些对象要存储在容器里——如上所述——使用 `boost::shared_ptr` 和 `boost::shared_array` 这就相当简单了。

```cpp
#include <boost/shared_ptr.hpp> 
#include <vector> 

int main() 
{ 
  std::vector<boost::shared_ptr<int> > v; 
  v.push_back(boost::shared_ptr<int>(new int(1))); 
  v.push_back(boost::shared_ptr<int>(new int(2))); 
} 
```

*   下载源代码

上面例子中的代码当然是正确的，智能指针确实可以这样用，然而因为某些原因，实际情况中并不这么用。 第一，反复声明 `boost::shared_ptr` 需要更多的输入。 其次，将 `boost::shared_ptr` 拷进，拷出，或者在容器内部做拷贝，需要频繁的增加或者减少内部引用计数，这肯定效率不高。 由于这些原因，Boost C++ 库提供了 [指针容器](http://www.boost.org/libs/ptr_container/) 专门用来管理动态分配的对象。

```cpp
#include <boost/ptr_container/ptr_vector.hpp> 

int main() 
{ 
  boost::ptr_vector<int> v; 
  v.push_back(new int(1)); 
  v.push_back(new int(2)); 
} 
```

*   下载源代码

`boost::ptr_vector` 类的定义在 `boost/ptr_container/ptr_vector.hpp` 里，它跟前一个例子中用 `boost::shared_ptr` 模板参数来初始化的容器具有相同的工作方式。 `boost::ptr_vector` 专门用于动态分配的对象，它使用起来更容易也更高效。 `boost::ptr_vector` 独占它所包含的对象，因而容器之外的共享指针不能共享所有权，这跟 `std::vector&lt;boost::shared_ptr&lt;int&gt; &gt;` 相反。

除了 `boost::ptr_vector` 之外，专门用于管理动态分配对象的容器还包括：`boost::ptr_deque`， `boost::ptr_list`， `boost::ptr_set`， `boost::ptr_map`， `boost::ptr_unordered_set` 和 `boost::ptr_unordered_map`。这些容器等价于 C++标准里提供的那些。最后两个容器对应于`std::unordered_set` 和 `std::unordered_map`，它们作为技术报告 1 的一部分加入 C++ 标准。 如果所使用的 C++ 标准实现不支持技术报告 1 的话，还可以使用 Boost C++ 库里实现的 `boost::unordered_set` 和 `boost::unordered_map`。

## 2.10\. 练习

You can buy [solutions to all exercises](http://en.highscore.de/shop/index.php?p=boost-solution) in this book as a ZIP file.

1.  使用适当的智能指针优化下面的程序：

    ```cpp
    #include &lt;iostream&gt; 
    #include &lt;cstring&gt; 

    char *get(const char *s) 
    { 
      int size = std::strlen(s); 
      char *text = new char[size + 1]; 
      std::strncpy(text, s, size + 1); 
      return text; 
    } 

    void print(char *text) 
    { 
      std::cout &lt;&lt; text &lt;&lt; std::endl; 
    } 

    int main(int argc, char *argv[]) 
    { 
      if (argc &lt; 2) 
      { 
        std::cerr &lt;&lt; argv[0] &lt;&lt; " &lt;data&gt;" &lt;&lt; std::endl; 
        return 1; 
      } 

      char *text = get(argv[1]); 
      print(text); 
      delete[] text; 
    } 
    ```

    *   下载源代码
2.  优化下面的程序：

    ```cpp
    #include &lt;vector&gt; 

    template &lt;typename T&gt; 
    T *create() 
    { 
      return new T; 
    } 

    int main() 
    { 
      std::vector&lt;int*&gt; v; 
      v.push_back(create&lt;int&gt;()); 
    } 
    ```

    *   下载源代码