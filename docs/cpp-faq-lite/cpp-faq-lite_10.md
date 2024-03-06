# [12] 赋值算符

# [12] 赋值算符

## FAQs in section [12]:

*   [12.1] 什么是“自赋值”？
*   [12.2] 为什么应该当心“自赋值”？
*   [12.3] 好，好；我会处理自赋值的。但如何做呢？

## 12.1 什么是“自赋值”？

自赋值就是将对象赋值给本身。例如，

```cpp
 #include "Fred.hpp"    // 声明 Fred 类 

 void userCode(Fred& x)
 {
   x = x;   // 自赋值
 } 
```

很明显，以上代码进行了显式的自赋值。但既然多个指针或引用可以指向相同对象（别名），那么进行了自赋值而自己却不知道的情况也是可能的：

```cpp
 #include "Fred.hpp"    // 声明 Fred 类

 void userCode(Fred& x, Fred& y)
 {
   x = y;   // 如果&x == &y 就可能是自赋值
 }

 int main()
 {
   Fred z;
   userCode(z, z);
 } 
```

## 12.2 为什么应该当心“自赋值”？

如果不注意自赋值，将会使你的用户遭受非常微妙的并且一般来说非常严重的 bug。例如，如下的类在自赋值的情况下将导致灾难：

```cpp
 class Wilma { };

 class Fred {
 public:
   Fred()                : p_(new Wilma())      { }
   Fred(const Fred& f)   : p_(new Wilma(*f.p_)) { }
  ~Fred()                { delete p_; }
   Fred& operator= (const Fred& f)
     {
       // 差劲的代码：没有处理自赋值！
       delete p_;                // Line #1
       p_ = new Wilma(*f.p_);    // Line #2
       return *this;
     }
 private:
   Wilma* p_;
 }; 
```

如果有人将 `Fred` 对象赋给其本身，由于`*this`和 `f` 是同一个对象，line #1 同时删除了`this->p_`和`f.p_`。而 line #2 使用了已经不存在的对象`*f.p_`，这样很可能导致严重的灾难。

作为 `Fred`类的作者，你最起码有责任确信在`Fred`对象上自赋值是无害的。不要假设用户不会在对象上这样做。如果对象由于自赋值而崩溃，那是*你的*过失。

> 另外：上述的`Fred::operator= (const Fred&)`还有第二个问题：如果在执行`new Wilma(*f.p_)`时，抛出了异常或者`Wilma`的拷贝构造函数中的异常）， `this->p_`将成为悬空指针——它所指向的内存不再是可用的。这可以通过在删除就对象前创建对象来解决。

## 12.3 好，好；我会处理自赋值的。但如何做呢？

在你创建类的每时每刻，都应该当心自赋值。这并不意味着需要为你所有的类都增加额外的代码：只要对象优雅地处理自赋值，而不管是否必须增加额外的代码。

如果不需要为赋值算符增加额外代码，这里有一个简单而有效的技巧：

```cpp
 Fred& Fred::operator= (const Fred& f)
 {
   if (this == &f) return *this;   // 优雅地处理自赋值
   // 此处写正常赋值的代码...

   return *this;
 } 
```

显式的测试并不总是必要的。例如，如果修正前一个 FAQ 中的赋值算符使之处理`new`抛出的异常和／或`Wilma`类的拷贝构造函数抛出的异常，可能会写出如下的代码。注意这段代码有（令人高兴的）自动处理自赋值的附带效果：

```cpp
 Fred& Fred::operator= (const Fred& f)
 {
   // 这段代码优雅地（但隐含的）处理自赋值
   Wilma* tmp = new Wilma(*f.p_);   // 如果异常在此处被抛出也没有问题
   delete p_;
   p_ = tmp;
   return *this;
 } 
```

在象这个例子的情况下（自赋值是无害的但是低效），一些程序员想通过增加另外的不必要的测试，如“`if (this == &f) return *this;`”来改善自赋值时的效率。通常来说，使自赋值情况更高效而使得非自赋值情况更低效的折衷是错误的。例如，为`Fred`类的赋值算符增加如上的`if`测试会使得非自赋值情况更低效（一个额外的(而且不必要的)条件分支）。如果自赋值实际上一千次才发生一次，那么 `if`将浪费 99.9%的时间周期。