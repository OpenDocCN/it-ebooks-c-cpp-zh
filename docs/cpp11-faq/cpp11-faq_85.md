# metaprogramming（元编程）and type traits

（译注：原作者还没有完成这个小节，等原作者完成后中文版随后奉上。这里补充一点关于元编程(metaprogramming)）的基础知识，供大家参考

1、何谓“元编程(Metaprogramming)”?

具备如下特征之一的程序编写称为元编程：

利用或者编写其它语言程序来作为所编写程序的数据

在程序运行时完成一些编程工作，而不是在编译时—具备这种特征的语言，我们也常成为动态语言

2、元编程的优势

元编程为程序开发人员提供了在与手动编写所有代码相同时间内，完成更多工作的可能，同时，由于

在面对新情况时，不需要重新编译，因此元编程为程序提供了更大的灵活性和可用性。

3、元编程语言

用于元编程的语言，称作元语言，被元语言所利用的语言通常称作对象语言。元语言具有自反性。所谓

自反性就是自己描述自己的特性。自反性是元编程中非常重要特点。有些语言中，将自身作为 First-class object

也是非常有用的。对于有些可以调用元编成机制的程序语言，也具备自反性。

4、元编程方式

一般来说，实现元编程有两种方式。

第一种方法是调用 API 将运行时引擎中的代码展示为程序代码

第二种方法是动态执行包含在程序中的字符表达式，也就是我们常说的“程序生成程序”。

虽然，两种方法都可以实现元编程，但是大多数语言都只是支持其中的一种方法。

5、实例

1）bash script

```cpp
#!/bin/bash # metaprogram echo ‘#!/bin/bash’ >program for ((I=1; I< =992; I++)) do echo "echo $I" >>program done chmod +x program 
```

该脚本产生 993 行代码，这便是用程序生成程序的一个例子

2）像 Lisp， Python 和 Javascript 等语言可以在程序运行期间修改或者增量编译，这些程序语言也可以在不产生新代码的

情况下进行元编程。

3）我们常见的编译器其实也是元编码的代表，编译器通常是用相对简约的高级语言来产生汇编或者机器代码。

（翻译：Yibo Zhu）