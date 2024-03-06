# （广义的）联合体

# （广义的）联合体

在 C++98（或更早）版本中，union 的成员类型，必须不含自定义构造/析构函数或者赋值操作符。

```cpp
union U {
  int m1;
  complex m2;    //错误（明显的）：complex 拥有构造函数
  //错误（不那么明显）：string 的内部数据只能严格地由其构造函数，
  // 拷贝构造函数，和析构函数去维护
  string m3;
}; 
```

亦即：

```cpp
U u;        // 使用哪个成员的构造函数呢？
u.m1 = 1;     // 给整型成员赋值
string s = u.m3;    //灾难：从 string 成员拷贝 
```

显而易见，把值写入一个成员，之后又读取另外一个成员的做法是给自己找麻烦，然而人们往往并非刻意而为（多因失误导致） 。

C++11 对 union 的限制条件进行了放宽，以允许更灵活广泛的成员类型。其中值得特别指出的是，带有自定义构造函数/析构函数的类型现在也可作为 union 的成员了。此外，为使“灵活”不至成为脱缰野马，新标准又特别引入了“第四条军规”（译注：参见下文），并倡导使用“可识别 union”（译注：参见下文 Widget 样例以及 Boost::Any 和 Boost::Variant）。

C++11 中的对 union 的限制条件重新定义如下：

*   不含虚函数（与 C++98 相同）
*   不含引用成员（与 C++98 相同）
*   没有基类（与 C++98 相同）
*   若 union 的某个成员的类型含有自定义构造/拷贝/析构函数，那么该 union 的相应构造/拷贝/析构函数将会被自动“禁用”（译注：在 C++11 中我们可以使用 delete 关键字来“禁用”构造/析构函数），随之而来的后果是：该 union 不能被实例化成对象。（新增的所谓“第四条军规”）

例如：

```cpp
union U1 {
  int m1;
  complex m2;    // ok
};

union U2 {
  int m1;
  string m3;    // ok
}; 
```

上述代码看起来容易出问题（译注：例如有人实例化了 U2 类型的对象并给其 m1 成员赋值之后，当 union 析构时，m3 的析构函数可能会 crash），但是有了第四条军规，刚才的隐含问题便迎刃而解（译注：通过编译错误）。即：

```cpp
U1 u;          // ok
u.m2 = {1,2};    // ok：给 complex 成员赋值
U2 u2;   // 编译错误: string 类含有析构函数，因而 U2 的析构函数已被自动禁用
    //（译注：析构函数被禁用意味着不允许在栈上实例化 U2 对象，否则无法析构）
U2 u3 = u2;    // 编译错误：string 类含有拷贝构造函数，
    // 因而 U2 类型同样也不能被拷贝构造 
```

这样看来，先前定义的 U2 几乎没什么实际用途了。唯一可能用到这种奇葩 union 的地方是，把它嵌到结构体里并且额外记录其“当值”成员类型——也就是所谓的“可识别 union”。样例如下：

```cpp
class Widget {  // 用 union 存储的“三态”Widget
  private:
    // 用以实现“可识别 union”的“当值”类型标记
    enum class Tag { point, number, text } type;
    union {    // 三态的具体存储形式
      point p;    // point 类含有构造函数
      int i;
      string s;   // string 类含有默认的构造/拷贝/析构函数
    };
    // ...
    // 由于 string 中存在拷贝函数，所以需要“手工拷贝”union
    Widget& operator=(const Widget& w)
    {
      // 译注：从 text 态到 text 态
      if (type==Tag::text && w.type==Tag::text) {
        s = w.s;    // 直接使用 string 的赋值运算符
        return *this;
      }
      // 译注：从 text 态到其他态，意味着 union 不再用于存放 string
      //    由于 union 的拷贝函数已被自动禁用，
      //    所以需要有人手工释放 string 原先所占资源
      if (type==Tag::text) s.~string();  // 此处需要显式析构

      switch (w.type) {
      case Tag::point: p = w.p; break;  // 普通的拷贝
      case Tag::number: i = w.i; break;
      // 译注：C++98/03 标准中的的原地拷贝构造语法
      case Tag::text: new(&s)(w.s); break;  // placement new
      }
      type = w.type;
      return *this;
    }
  };
}; 
```

其他参考文献：

*   [N2544=08-0054] Alan Talbot, Lois Goldthwaite, Lawrence Crowl, and Jens Maurer: [Unrestricted unions (Revison 2)](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2544.pdf)

（翻译：张潇）