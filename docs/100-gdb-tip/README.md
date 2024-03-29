# 《100 个 gdb 小技巧》

> 作者：[hellogcc](https://github.com/hellogcc)
> 
> 来源：[100-gdb-tips](https://github.com/hellogcc/100-gdb-tips)

一个关于 gdb 使用小技巧的文档。100，在这里可能只是表明很多；具体的数目取决于您的参与和贡献。

## 如何参与

直接发 PULL REQUEST，或与我们联系。

增加一个小技巧的步骤：

1.  在 src 目录下新增一个 md 文件，参照现有文件的格式风格，编写一个小技巧
    markdown 语法参见 [`wowubuntu.com/markdown/`](http://wowubuntu.com/markdown/)
    md 文件编写可以使用在线所见即所得编辑器 [`www.zybuluo.com/mdeditor`](https://www.zybuluo.com/mdeditor)
2.  在 index.md 中为新 md 文件增加一个索引，可以放到已有分类中，或增加一个分类
3.  如果预览下没有问题，OK!

本地生成 html 的步骤：

1.  确保[go](http://code.google.com/p/go)和[md2min](https://github.com/fairlyblank/md2min)已经安装并可用
2.  直接运行 build.sh
3.  如果顺利，会在 html 目录下生成所有的 html 文件

## 联系方式

*   [博客网站](http://www.hellogcc.org)
*   在线讨论问题：IRC, freenode, #hellogcc 房间
*   [邮件列表](http://www.freelists.org/list/hellogcc) (发信需要先订阅)

## 版权

本文档版权归贡献者所有。

## 授权许可

本文档使用的是[GNU Free Documentation License](http://www.gnu.org/licenses/fdl.html)。

## 致谢

*   各位参与者

## 其它资源

*   [GDB 在线手册](https://sourceware.org/gdb/onlinedocs/gdb)
*   [GDB 命令卡片](https://github.com/hellogcc/100-gdb-tips/blob/master/refcard.pdf)
*   [GDB dashboard](https://github.com/cyrus-and/gdb-dashboard)
*   [Gdbinit for OS X, iOS and others - x86, x86_64 and ARM](https://github.com/gdbinit/Gdbinit)
*   [dotgdb：关于底层调试和反向工程的 gdb 脚本集](https://github.com/dholm/dotgdb)