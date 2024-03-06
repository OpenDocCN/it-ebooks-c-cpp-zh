# 五、CGDB 高亮组

上一章：CGDB 配置命令，下一节：不同的高亮组，目录：目录

* * *

# 5 CGDB 高亮组

如果终端支持彩色显示，那么 CGDB 是能够使用彩色的。直到 0.6.1 版本，CGDB 不允许用户以任何方式配置色彩高亮。目前 CGDB 中的色彩高亮是完全可配置的。

CGDB 在色彩高亮上模仿了 vim 中的色彩高亮。任何在终端中被高亮的数据都是被一个高亮组所表示的。一个高亮组表示了数据需要被高亮的前景色、背景色和属性。目前 CGDB 中有多种不同类型的高亮组。有语法高亮组，代表着对源代码的语法高亮。也有 UI 高亮组，代表 CGDB 的 logo 与状态栏等等。

每个高亮组都有一组默认的属性和颜色与其相关联。您可以通过 highlight 命令来修改一个高亮组的属性。参见第四章。

请注意，CGDB 目前支持使用和启动 CGDB 前终端的颜色相同的背景颜色。但是，这只在 CGDB 与 ncurses 链接时有效。如果您将 CGDB 与 curses 链接，则 CGDB 将会将背景色强制设置成黑色。

*   可使用的高亮组：不同的高亮组。
*   可使用的属性：不同的属性。
*   可使用的颜色：不同的颜色。

# 5.1 不同的高亮组

上一节：CGDB 高亮组，下一节：不同的属性，目录：目录

* * *

## 5.1 不同的高亮组

下面的列表列出了 CGDB 在高亮代码文件时会使用的全部高亮组。

`Statement`
表示了一种语言定义的关键字

`Type`
表示了一种语言定义的类型

`Constant`
表示了字符串或者数字

`Comment`
表示了源代码中的注释

`PreProc`
表示了 C/C++中的预处理指令

`Normal`
表示了普通文本

下面的列表列出了 CGDB 在显示其 UI 时会使用的全部高亮组。

`StatusLine`
表示了 CGDB 中的状态栏。文件对话框的状态栏也会是用这个高亮组。

`IncSearch`
表示了当用户在文件对话框窗口或是代码窗口中搜索时使用的高亮组。

`Arrow`
表示了 CGDB 绘制的指向当前浏览代码行的箭头

`LineHighlight`
表示了当用户设置`arrowstyle`选项为`highlight`时，高亮行使用的高亮组

`Breakpoint`
表示了 CGDB 显示被设置了断点的行所使用的高亮组

`DisabledBreakpoint`
表示了 CGDB 显示断点被禁用的行所使用的高亮组

`SelectedLineNr`
表示了 CGDB 显示当前被选择的行。也就是光标所在的行。

`Logo`
这个高亮组是 CGDB 在没有源代码被自动检测到的时候所显示的 logo 的高亮组。

# 5.2 不同的属性

上一节：不同的高亮组，下一节：不同的颜色，目录：目录

* * *

## 5.2 不同的属性

CGDB 支持 curses 提供的部分属性。它会将这些属性应用至输出窗口，但是这取决于您使用的终端是否支持这些特性。

下面列出了 CGDB 目前支持的一系列属性。

`normal`
`NONE`
这将会让文本保留不同样式。使用 curses 中的 A_NORMAL 属性。

`bold`
这将会让文本加粗显示。使用 curses 中的 A_BOLD 选项。

`underline`
这将会让文本带下划线显示。使用 curses 中的 A_UNDERLINE 选线。

`reverse`
`inverse`
这会将前景色与背景色调换。使用 curses 中的 A_REVERSE 选项。

`standout`
这是最适合的终端高亮模式。使用 curses 中的 A_STANDOUT 选项。

`blink`
这会使文本闪烁。使用 curses 中的 A_BLINK 选项。

`dim`
这会使文本亮度减少 1/2。使用 curses 中的 A_DIM 选项。

# 5.3 不同的颜色

上一节：不同的属性，下一章：CGDB 键盘用户接口，目录：目录

* * *

## 5.3 不同的颜色

CGDB 支持一些颜色，取决于您的终端支持多少种颜色。下表是一个 CGDB 所提供的颜色的表格。标题为 NR-16 的列表示终端至少支持 16 种颜色。标题为 NR-8 的列表示终端至少支持 8 种颜色。每种颜色对应的整数数值表示了被传入 curse 函数`init_pair()`的数值，该函数用来使 curse 创建一种新的颜色。

| `COLOR NAME` | `NR-16` | `NR-8` | `NR-8 bold attribute`  |
| Black | 0 | 0 | No  |
| DarkBlue | 1 | 4 | No  |
| DarkGreen | 2 | 2 | No  |
| DarkCyan | 3 | 6 | No  |
| DarkRed | 4 | 1 | No  |
| DarkMagenta | 5 | 5 | No  |
| Brown, DarkYellow | 6 | 3 | No  |
| LightGray, LightGrey, Gray, Grey | 7 | 7 | No  |
| DarkGray, DarkGrey | 8 | 0 | Yes  |
| Blue, LightBlue | 9 | 4 | Yes  |
| Green, LightGreen | 10 | 2 | Yes  |
| Cyan, LightCyan | 11 | 6 | Yes  |
| Red, LightRed | 12 | 1 | Yes  |
| Magenta, LightMagenta | 13 | 5 | Yes  |
| Yellow, LightYellow | 14 | 3 | Yes  |
| White | 15 | 7 | Yes  |