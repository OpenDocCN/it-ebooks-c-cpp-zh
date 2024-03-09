# 复制和重新抛出异常

你怎么捕获一个异常，之后在另外一个线程上重新抛出？使用在标准文档 18.8.5 中描述的异常传递中的方法吧，那将显示标准库的魔力。

exception_ptr current_exception(); 返回一个 exception_ptr 变量，它将指向现在正在处理的异常（15.3）或者现在正在处理的异常的副本（拷贝），或者有的时候在当前没有遇到异常的时候，返回值为一个空的 exception_ptr 变量。只要 exception_ptr 指向一个异常，那么至少在 exception_ptr 的生存期内，运行时能够保证被指向的异常是有效的。

```cpp
void rethrow_exception(exception_ptr p); 
template exception_ptr copy_exception(E e); 
```

它的作用如同：

```cpp
try {
    throw e;
} catch(...) {
    return current_exception();
} 
```

当我们需要将异常从一个线程传递到另外一个线程时，这个方法十分有用.