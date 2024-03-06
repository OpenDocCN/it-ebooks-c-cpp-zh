# noexcept – 阻止异常的传播与扩散

# noexcept – 阻止异常的传播与扩散

如果一个函数不能抛出异常，或者一个程序并没有接获某个函数所抛出的异常并进行处理，那么这个函数可以用新的 noexcept 关键字对其进行修饰，表示这个函数不会抛出异常或者抛出的异常不会被接获并处理。例如：

```cpp
extern "C" double sqrt(double) noexcept;    // 永远不会抛出异常

vector my_computation(const vector& v) noexcept // 在这里，我不准备处理内存耗尽的异常，所以我只是简单地将函数声明为 noexcept
{
    vector res(v.size());    // 可能会抛出异常
    for(int i; i         return res;
} 
```

如果一个经过 noexcept 修饰的函数抛出异常（异常会尝试逃出这个函数（？）），程序会通过调用 terminate()来结束执行。通过 terminate()的调用来结束程序的执行会带来很多问题，例如，无法保证对象的析构函数的正常调用，无法保证栈的自动释放，同时也无法在没有遇到任 何问题的情况下重新启动程序。所以，它是不可靠的。

我们这样写是故意的，它使得成为一种简单、粗暴但是非常有效的机制（比那种旧的动态地抛出异常的机制要有效得多）。

同时，我们还可以让一个函数根据不同的条件实现 noexcept 修饰或者是无 noexcept 修饰。例如，一个算法可以根据它用作模板参数所使用的操作是否抛出异常，来决定自己是否抛出异常。例如：

```cpp
template
void do_f(vector& v) noexcept(noexcept(f(v.at(0)))); //如果 f(v.at(0))可以抛出异常，则这个函数也可以抛出异常
{
    for(int i; i             v.at(i) = f(v.at(i));
} 
```

在这里，第一个 noexcept 被用作操作符（operator）：如果 if f(v.at(0))不能够抛出异常，noexcept(f(v.at(0)))则返回 true，也即意味着 f()和 at()是无法抛出异常（noexcept）。

noexcept()操作符是一个常量表达式，并且不计算表达式的值，只是判断这个表达式是否会产生并抛出异常。

声明的通常形式是 noexcept(expression)，并且单独的一个“noexcept”关键字实际上就是的一个 noexcept(true)的简化。一个函数的所有声明都必须与 noexcept 声明保持 兼容。

一个析构函数不应该抛出异常；通常，如果一个类的所有成员都拥有 noexcept 修饰的析构函数，那么这个类的析构函数就自动地隐式地 noexcept 声明，而与函数体内的代码没有关系。

通常，将某个抛出的异常进行移动操作是一个很坏的主意，所以，在任何可能的地方都用 noexcept 进行声明。如果某个类的所有成员都有使用 noexcept 声明的析构函数，那么这个类默认生成的复制或者移动操作（类的复制构造函数，移动构造函数等）都是隐式的 noexcept 声明。（？）

noexcept 被广泛地系统地应用在 C++11 的标准库中，以此来提供标准库的性能和满足标准库对于简洁性的需求。

参考 :

*   Standard: 15.4 Exception specifications [except.spec].
*   Standard: 5.3.7 noexcept operator [expr.unary.noexcept].
*   [N3103==10-0093] D. Kohlbrenner, D. Svoboda, and A. Wesie: Security impact of noexcept. (Noexcept must terminate, as it does).
*   [N3167==10-0157] David Svoboda: Delete operators default to noexcept .
*   [N3204==10-0194] Jens Maurer: Deducing "noexcept" for destructors
*   [N3050==10-0040] D. Abrahams, R. Sharoni, and D. Gregor: Allowing Move Constructors to Throw (Rev. 1) .