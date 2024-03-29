# 四、CGDB 配置命令

上一节：TTY 模式中的命令，下一章：CGDB 高亮组，目录：目录

* * *

# 4 CGDB 配置命令

您可能会觉得 CGDB 中的一些特性比较有用。CGDB 可以通过一个叫做 *cgdbrc* 的文件自动加载 CGDB 的命令。CGDB 将会在 *$HOME/.cgdb/* 目录下查找这个文件。如果这个文件存在的话，CGDB 将会依次运行这个文件中的每一行。这和用户在 tui 初始化后在状态栏输入所有的命令是一样的。

下列变量改变了 CGDB 一些方面的行为。其中有些命令是缩写，所有的布尔命令都可能通过在命令前添加 *'no'* 而变成一个否定的命令。例如： `:set ignorecase` 将搜索设置为大小写不敏感；而设置 `:set noignorecase` 则将搜索设置为大小写敏感。

`:set as=`*style*
`:set arrowstyle=`*style*
设置箭头的风格。可能的取值为 *'short'*， *'long'*，以及 *'highlight'* 。被改变风格的箭头是指向当前运行的源代码行的箭头。默认值是 *'short'*。为了更容易阅读，CGDB 提供了更长的箭头。最后， *'highlight'* 选项不绘制出箭头，而是将整行反色。

`:set asr`
`:set autosourcereload`
如果这个选项被打开了，CGDB 将会在源代码文件被 CGDB 打开后又被修改的时候自动地重新加载源代码文件。如果这个选项被关闭了，源代码文件将不会被重新加载，直到您重启 CGDB。这个选项默认是开启的。这个特性在您调试程序时，修改源代码，重新编译以及在 GDB 的命令行窗口中输入 *r* 时非常有用。在这种情况下，这个文件将会被更新到最新的版本。请注意，CGDB 只通过检查源代码文件的时间戳来判断它是否被改变。因此如果您修改了代码，但是没有重新编译它，CGDB 将会依然重新加载改动后的源代码。

`:set cgdbmodekey=`*key*
这个选项用来设置将 CGDB 切换至 CGDB 模式的快捷键。默认的情况下， *ESC* 键是 CGDB 模式键。通过使用键码，CGDB 模式键还可以被设置为任意的其他键。这个选项在用户想使用 readline 的 vi 模式时会特别有用。如果用户输入 `:set cgdbmodekey=<PageUp>` 然后 *PageUp* 键将会让 CGDB 进入 CGDB 模式， 而 *ESC* 键将会被 readline 接收。

`:set ic`
`:set ignorecase`
将搜索设置为大小写不敏感。默认情况下，这个选项是关闭的（默认大小写敏感）。

`:set stc`
`:set showtgdbcommands`
当这个选项被开启时，CGDB 将会显示所有它发送给 GDB 的命令。如果这个选项关闭，CGDB 将不会显示它发送给 GDB 的命令。这个选项默认是关闭的。

`:set syn=`*style*
`:set syntax=`*style*
将当前的源代码设置为 *style* 类型的高亮模式。可取的值有 *'c'* ， *'ada'* 以及 *'off'* 。通常的情况下，用户不会使用到这个命令，因为 CGDB 会自动得通过检测源代码文件的扩展名来选择高亮模式。但是这个特性目前可以被用于调试 CGDB。

`:set to`
`:set timeout`
这个选项是与 *ttimeout* 选项一起使用的，它用来决定 CGDB 在接收到一部分被映射的按键或者是一部分的虚拟键码后的行为。如果这个选项被开启了，CGDB 将会在一段时间后将这些按键序列或是虚拟键码的一部分判断为超时。如果这个选项是关闭的，用户定义的映射将不会被判断为超时。CGDB 将会通过检验 *ttimeout* 选项的值来决定是否将虚拟键码的部分输入判断为超时。想要确定 CGDB 会如何处理被映射的按键与虚拟键码的超时和超时时长，请参见第六章中的列表。这个选项默认是被开启的。

`:set tm=`*delay*
`:set timeoutlen=`*delay*
这个选项被用来与 *ttimeoutlen* 选项配合使用。它用毫秒数值表示了 CGDB 将会等待一个按键映射序列完成输入的时间长度。如果 *delay* 为 0，CGDB 将会立即接收每个收到的字符输入。这样会阻止任何按键映射或是虚拟键码的输入。*delay* 可以在 0 至 10000 之间取值，包括 0 和 10000。*delay* 的默认值为 1000（一秒）。

`:set ttimeout`
这个选项是与 *timeout* 一起使用的，它用来决定 CGDB 在接收到一部分虚拟键码后的行为。如果这个选项被开启了，CGDB 对从键盘输入的虚拟键码设置超时。如果这个选项关闭，则 CGDB 将会根据 *timeout* 选项来决定是否需要对虚拟键码设置超时。想要确定 CGDB 会如何处理虚拟键码的超时和超时时长，请参见第六章中的列表。这个选项默认是被开启的。

`:set ttm=`*delay*
`:set ttimeoutlen=`*delay*
这个选项是与 *timeoutlen* 选项一起使用的。它用毫秒数值表示了 CGDB 将会等待一个虚拟键码被完成输入的时间长度。如果 *delay* 为 0，CGDB 将会立即接收每个收到的字符输入。这样将会阻止任何虚拟键码的输入。*delay* 可以在 0 至 10000 之间取值，包括 0 和 10000。*delay* 的默认值为 100（十分之一秒）。

`:set ts=`*number*
`:set tabstop=`*number*
设置一个 TAB 键在屏幕上显示的对应空格的个数。默认的 *number* 值为 8。

`:set wmh=`*number*
`:set winminheight=`*number*
窗口的最小高度。CGDB 中的所有的窗口的高度都不会小于这个值。默认的 *number* 值为 0。

`:set winsplit=`*style*
设置代码窗口和 GDB 窗口分界的位置。这个选项在被写入 cgdbrc 时会非常有用。参见第四章。*style* 选项可取的值有 *top_full* ， *top_big* ， *even* ， *bottom_big* 以及 *bottom_full*。

`:set ws`
`:set wrapscan`
当搜索到文件的末尾时从头开始继续搜索。这个选项默认是被开启的。

`:c`
`:continue`
向 GDB 发送一个`continue`命令。

`:down`
向 GDB 发送一个`down`命令。

`:e`
`:edit`
重新加载代码窗口中的文件。这个选项在文件被 cgdb 打开后被改变的时候有用。

`:f`
`:finish`
向 GDB 发送一个`finish`命令。

`:help`
这个命令将在代码窗口中以文本形式显示本手册。

`:hi` *group* `cterm=`*attributes* `ctermfg=`*color* `ctermbg=`*color* `term=`*attributes*
`:highlight` *group* `cterm=`*attributes* `ctermfg=`*color* `ctermbg=`*color* `term=`*attributes*
为特定的高亮组设置颜色和属性。命令的格式模仿了 vim 中的“highlight”命令。`group`、`attributes`和`color`的可以选择的值请参见第五章。
您可以给出任意顺序、任意数量的名字-值对。'*ctermfg*' 和 '*ctermbg*' 分别设置前景色和背景色。可以通过数字或者 vim 中颜色的名字来选择特定的颜色。当 CGDB 与 ncurses 链接时，使用数字表达颜色的数值可以设置为-1 至 COLORS 之间的值。当 CGDB 与 curses 链接时，数值必须在 0 至 COLORS 之间。
'*cterms*' 设置彩色终端的视频属性。'*term*' 设置黑白终端的视频属性。以下是一些这个命令的例子：
`:highlight Logo cterm=bold,underline ctermfg=Red ctermbg=Black`
`:highlight Normal cterm=reverse ctermfg=White ctermbg=Black`
`:hi Normal term=bold`

`:insert`
激活 GDB 窗口。

`:n`
`:next`
向 GDB 发送一个 next 命令。

`:q`
`:quit`
退出 CGDB。

`:r`
`:run`
向 GDB 发送一个 run 命令。

`:start`
向 GDB 发送一个 start 命令。

`:k`
`:kill`
向 GDB 发送一个 kill 命令。

`:s`
`:step`
向 GDB 发送一个 step 命令。

`:syntax`
打开或关闭代码高亮（使用`:syntax on`与`:syntax off`打开和关闭代码高亮，译者注）

`:up`
向 GDB 发送一个 up 命令。

`:map` *lhs* *rhs*
创建一个在 CGDB 模式下的新的键盘映射，或是替换一个已有的 CGDB 模式下的键盘映射。在命令被执行后，当 *lhs* 被输入时，CGDB 将会收到 *rhs* 而不是 *lhs*。更多关于如何使用 map 命令的细节请参见 6.2 节。

`:unm lhs`
`:unmap lhs`
删除一个 CGDB 模式下的已有的键盘映射。*lhs* 是用户在创建键盘映射时左边输入的字符组合。例如，如果用户输入`:map a<Space>b foo`，那么用户可以通过输入`:unmap a<Space>b`来取消这个已经存在的键盘映射。

`:im lhs rhs`
`:imap lhs rhs`
创建一个 GDB 模式下的新的键盘映射，或是替换一个已有的 GDB 模式下的键盘映射。在命令被执行后，当 *lhs* 被输入时，CGDB 将会收到 *rhs* 而不是 *lhs*。更多关于如何使用 map 命令的细节请参见 6.2 节。

`:iu lhs`
`:iunmap lhs`
删除一个 GDB 模式下的已有的键盘映射。*lhs* 是用户在创建键盘映射时左边输入的字符组合。例如，如果用户输入`:imap a<Space>b foo`，那么用户可以通过输入`:iunmap a<Space>b`来取消这个已经存在的键盘映射。