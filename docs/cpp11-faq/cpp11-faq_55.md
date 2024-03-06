# nullptr——空指针标识

# nullptr——空指针标识

空指针标识(nullptr)（其本质是一个内定的常量）是一个表示空指针的标识，它不是一个整数。（译注：这里应该与我们常用的 NULL 宏相区别，虽然它们都是用来表示空置针，但 NULL 只是一个定义为常整数 0 的宏，而 nullptr 是 C++0x 的一个关键字，一个内建的标识符。下面我们还将看到 nullptr 与 NULL 之间更多的区别。）

```cpp
char* p = nullptr;
int* q = nullptr;
char* p2 = 0;           //这里 0 的赋值还是有效的，并且 p=p2

void f(int);
void f(char*);

f(0);         //调用 f(int)
f(nullptr);   //调用 f(char*)

void g(int);
g(nullptr);       //错误：nullptr 并不是一个整型常量
int i = nullptr;  //错误：nullptr 并不是一个整型常量 
```

（译注：实际上，我们这里可以看到 nullptr 和 NULL 两者本质的差别，NULL 是一个整型数 0，而 nullptr 可以看成是一个 char *。）

参考：

*   the C++ draft section ???
*   [N1488==/03-0071] Herb Sutter and Bjarne Stroustrup:[A name for the null pointer: nullptr](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2003/n1488.pdf) .
*   [N2214 = 07-0074 ] Herb Sutter and Bjarne Stroustrup:[A name for the null pointer: nullptr (revision 4)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2007/n2431.pdf) .

（翻译：张潇）