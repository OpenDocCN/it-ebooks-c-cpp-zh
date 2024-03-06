# 第一章 介绍

# 第一章 介绍

在这一章中，我们会回答这样一些基本的问题：wxWidgets 是什么，它和别的类似的开发库有什么不同。我们还会大概说一下这个项目的历史，以及 wxWidgets 社区的工作，它采用的许可协议，它的体系架构以及目前拥有的各种版本等。

什么是 wxWidgets

wxWidgets 是一个程序员的开发工具包，这个工具包用来开发用于桌面或者移动设备的图形界面应用程序。或者说它提供了一个框架，它作了很多底层的管家婆似的工作以便给应用程序提供一些默认的行为。wxWidgets 库给程序员提供了大量的类以及类的方法，以供其使用和定制。一个典型图形界面应用程序所作的事情包括：显示一个包含各种标准控件的窗口，也可能需要在窗口中绘制某种特别的图形或者图像，并且还要响应来自鼠标，键盘以及其它输入设备的输入。很可能这个应用程序还要和其它的进程通信，甚至还要驱动别的应用程序，换句话说，wxWidgets 可以让程序员编写一个拥有所有通用特性的时髦应用程序的工作变的相对容易。

虽然 wxWidgets 经常被打上图形界面程序开发的标签，但是它在很多其它的应用程序开发方面也提供了很多特性支持。这样作的目的是为了让使用 wxWidgets 编写的程序的各个部分都可以是跨平台的，而不仅仅是图形界面的部分。这些部分包括：文件和流操作，多线程，程序设置，进程间通讯，在线帮助，数据库访问等等。

# 1.1 为什么要使用 wxWidgets?

# 1.1 为什么要使用 wxWidgets?

wxWidgets 和其它类似的 GUI（图形用户界面，下同）库比如 MFC 或者 OWL 一个最本质的区别在于，它是跨平台的。wxWidgets 提供的 API 函数在它支持的所有平台上是想同的或者是非常相似的。这意味着你可以编写一个 windows 上运行的程序，这个程序不需要经过任何改动，或者只需要 很少的改动（这种情况并不常见），只需要通过重新编译，就可以在 Linux 或者 Max OSX 上运行。比起为另外的平台从头编写代码，这显然有很大的好处，另外一个附带的好处是，你不需要重新学习那个平台的 API。 而且，你的程序可能在将来很长时间仍然可以使用。因为随着计算机科技的演进，wxWidgets 将会随之一起演进，这样你的程序将会很方便的移植到最新的 操作系统以支持最新的特性。

另外一个与众不同的地方在于，wxWidgets 可以给你的应用程序提供本地观感。一些其它的可以跨平台的开发框架在不同的平台使用同 样的窗口组件代码（译者注：难到他指的是 JAVA？），也许它会通过类似窗口主题这样的方式来模拟本地观感。而 wxWidgets 则尽可能的使用本地的窗 口控件（当然 wxWidgets 也提供自己的控件集，这是另外一个话题了），所以 wxWidgets 的程序不只是看上去象是本操作系统上的原生程序，它实 际上就是原生程序。对于使用应用程序的用户来说，这是非常重要的，因为和本地操作系统标准的任何一点细微的甚至是几乎难以察觉的不同，都会让他们产生避而 远之的想法。

让我们来举例说明。下图演示了一个叫做 StoryLines 的小程序运行在 Windows XP 上的样子：

![](img/mhtAB07%281%29.tmp)

正象大家看到的那样，这是一个典型的 Windows 应用程序，有典型的 Windows 的 GUI 控件例如标签页，滚动条以及下拉列表。类似了，下图演示了这个程序在 Max OSX 上的样子，正象我们期待的那样，它有着水晶外形图标，没有菜单条（因为按照苹果的风格，当前窗口的菜单条应该显示在屏幕的最顶层。）

![](img/mhtAB0A%281%29.tmp)

最后，我们还将演示一下同样的程序在小红帽 Linux 上作为一个 GTK+程序的样子:

![](img/mhtAB2C%281%29.tmp)

为什么不直接使用 JAVA 呢？对于基于 Web 的应用来说，JAVA 的确很不错，但是对于桌面应用程序来说，JAVA 有时候并不是一个很好的选择。一般来 讲，基于 C++的 wxWidgets 程序会运行更快，感观上更象本地原生程序并且更容易安装，因为它并不依赖于你的机器一定要有 JAVA 虚拟机。C++也 更容易访问操作系统提供的底层函数并且更容易和已有的 C++或者 C 代码集成。基于以上原因，您现在经常用到的桌面程序中，很少有全部基于 JAVA 开发的。 而 wxWidgets 则可以让你开发高性能的，本地原生的应用程序。而这可能正是你的用户所期待的。

wxWidgets 是一个开放源代码的项目。毫无疑问，这意味着使用 wxWidgets 是免费，它不需要额外花费你 1 分钱（除非您愿意大方的向这个项目进 行捐助），但是，开放源代码并不仅仅意味着免费，它有着更重要的意义。开源项目通常可以持续比它的创建团队或者通常意义上的拥有者更长久的时间。使用 wxWidgets 开发程序，你的代码永远不会过时，你的代码所依赖的开发平台永远不会消失。你可以通过直接修改源代码来修正基础库中的问题（译者注：使 用 Delphi 的开发者对此可能有更深的体会，由于众所周知的原因，很多开发工具慢慢的被淘汰了）。你甚至可以自己抽点时间加入到 wxWidgets 的开 发团队中来，维护其中的一部分代码，这也是一件非常有趣的事情。开源项目的团队成员之所以加入某个团队是因为他们热爱他们正在作的事情，并且迫不及待的想 把他们的知识和别人分享，而商业项目的客服支持人员通常不具有这种理想主义的情节，当你使用 wxWidgets 开始编程时，你其实是把自己放入一个令人惊 讶的有艺术天赋的一堆天才中间（译者注：我只是按照字面意思翻译，虽然我自己用 wxWidgets 开发程序，但是这样的话还是让我觉的有一点点善意的恶 心。可能是我的英文太差了，没有理解原话的意思，原话是这样的：When you use wxWidgets, you tap into an astonishing talent pool, with contributors from a wide range of backgrounds.），这些天才来自世界的各个角落，有着各种各样的背景。开发应用程序需要考虑的很多细节都被这些天才封装在了你可以直接拿来很简 单就可以使用的类中，如果不是这些天才的劳动，你可能要花费很大的精力才能应付。一个开放和活跃的社区将会通过邮件列表对你提供帮助，在这里，你会享受到 讨论的乐趣。这些讨论并不全是和 wxWidgets 相关的。更多情形下，你是和社区那些有经验的或者没有经验的开发者进行心灵的交流.也许有一天，你会发 现自己成为 wxWidgets 之所以成功的一分子。

wxWidgets 已经被广泛的应用在各种工业领域。它的用户包含了象 AOL,AMD,CALTECH, Lockheed Martin, NASA, the Open Source Applications Foundation, Xerox 等等这些大的商业和团体机构。wxWidgets 拥有很广泛的使用者，从个体的软件开发者到大的商业团体，从计算机科学领域到医疗研究领域，从社会生态学到电信领域。当然，还有数不清的开源项目在使用它，例如 Audacity 声音编辑项目和 pgAdmin III 数据库设计和维护项目等。

人们出于各种各样的目的而使用 wxWidgets,一些人只是把它作为单平台开发上 MFC 的优雅的替代者，一些则是为了让他们的程序可以方便的从微软的 Windows 移植到 Linux 或者是苹果的 OSX。 wxWidgets 还正致力于移动终端的支持，包括嵌入式 linux，微软的 Pocket PC，在不久的将来还会支持 Palm OS。

# 1.2 wxWidgets 的历史

# 1.2 wxWidgets 的历史

1992 年，Julian Smart 在 Edinburgh 大学开始制作一个叫做 Hardy 的图表工具的时候，为了避免其发行版本在 Sun 的工作站和各种 PC 之间作选择，他决定使用跨平台的编程框架。但是当时可选的跨平台的编程框架不多，而他的部门也不可能给他很多的预算，所以他只能自己创建一个自己的跨平台编程框架。这样， wxWidgets 1.0 诞生了。 1992 年 9 月，学校允许他把他的 wxWidgets 1.0 上传到部门的 FTP 服务器，因此别的一些开发者也开始使用他的代码。最开始的时候，wxWidgets 是面向 XView 和 MFC 1.0 的，由于 Borland C++的适用者抱怨其对 MFC 的依赖，所以 Julian Smart 用纯 Win32 的代码重写了 wxWidgets。又因为 XView 很快被 Motif 取代，很快，Widgets 提供了对 Motif 的支持。

不久以后，一个很小但是却很付有激情的 wxWidgets 用户社区成立了并且拥有了自己的邮件列表。大量的新代码和补丁开始融入到 wxWidgets 中，其中包括 Markus Holzem 提供的 Xt 的支持。wxWidgets 也自然的拥有了越来越多的来自世界各地的使用者：独立工作者，学术机构，政府机构以及很多企业用户等，他们认为 wxWidgets 提供的产品质量和产品支持甚至好过他们见过的或者用过的其它商业的产品。

1997 年，在 Markus Holzem 的帮助下，新版的 wxWidgets 2 API 问世。此时，Wolfram Gloger 建议应该提供 GTK+的支持。GTK+是被 GNOME 桌面系统采纳的一套窗口控件。于是，Robert Roebling 开始领导 GTK 版本的 wxWidgets 的开发，现在 wxWidgets 的 GTK 版本已经成为其在 UNIX/LINUX 下的最主要的版本。到了 1998 年，Windows 和 GTK+的版本被合入版本控制工具 CVS。Vadim Zeitlin 加入到项目中来帮助管理和维护如此大量的设计和代码，同年，Stefan Csomor 开始着手增加对 Mac OS 的支持。

1999 年，Vaclav Slavik 的令人印象深刻的 wxHTML 类和 HTML 帮助文件显示控件被加入进来。2000 年，SciTech 公司开始开发 wxUniversal 版本，这个版本提供属于 wxWidgets 自己的不依赖于任何其它图形库的窗口控件，以便支持那些没有原生窗口控件库的操作系统。wxUniversal 最初被用于 SciTech 公司的 MGL 产品，这个产品为图形用户界面提供了底层支持。

到了 2002 年，Julian Smart 和 Robert Roebling 在 wxUniversal 的基础上提供了 wxX11 版本，这个版本仅依赖于 Unix 和 X11,因此它几乎适用于任何的类 Unix 环境，所以，它可以被用在相当底层的系统中。

2003 年，wxWidgets 开始了对 Windows CE 的支持，同年 Robert Roebling 在 GPE 嵌入式 Linux 平台上演示了使用 wxGTK 编写的程序。

2004 年，因为收到微软的商标方面的威胁，wxWidgets 被迫从它原来的名字"wxWindows"改名。

同样是在 2004 年，Stefan Csomor 和一大群热心的参与者彻底的修改了 wxMac OSX 版本，OSX 版本的功能和性能都得到了极大的提升。而 David Elliot 领导的小组正在稳步的开发一个基于 Cocoa 的版本，William Osborne 也着手开发一个可以支持 wxWidgets 的"minimal"例子的 Palm OS 6 的版本。 2005 年 4 月，2.6 版的 wxWidgets 发布了，几乎所有的平台版本在这个版本都有了大幅的改进和提高。

wxWidgets 将来的计划包括：

*   一个包管理工具，使得集成第三方工具变得容易。
*   更好的嵌入式支持。
*   更好的事件处理机制。
*   增强型控件支持：比如一种捆绑了树形控件和列表控件的控件。
*   wxHTML 2 提供在各种平台下的完整的 Web 能力支持。
*   STL 标准兼容
*   完整的 Palm OS 支持

# 1.3 wxWidgets 社区

# 1.3 wxWidgets 社区

wxWidgets 社区是非常活跃的。它拥有两个邮件列表: wx-users (用于普通用户)和 wx-dev (用于 wxWidgets 的开发者).一个网站，网站上有最新消息,一些文章以及和 wxWidgets 有关的链接，还拥有一个"Wiki," 所谓"Wiki"是一个网页的集合，这些网页可以被任何人修改和增加信息.还有一个论坛可以用来就某一话题发起讨论. 这些网络资源的网址列举在下面：

*   [`www.wxwidgets.org`](http://www.wxwidgets.org): 这是 wxWidgets 的官方网站
*   [`lists.wxwidgets.org`](http://lists.wxwidgets.org): wxWidgets 的邮件列表
*   [`wiki.wxwidgets.org`](http://wiki.wxwidgets.org): wxWidgets 的 Wiki
*   [`www.wxforum.org`](http://www.wxforum.org): wxWidgets 的论坛

和大多数开放源代码的项目一样，wxWidgets 采用 CVS 来进行代码的管理和修改记录的跟踪.为了保证开发的有序进行，只有少数几个开发者拥有对 CVS 库的修改权限，其它的开发者可以通过提交补丁和提交缺陷报告的方式参与开发，wxWidgets 目前使用的补丁和缺陷管理系统是由 SourceForge 提供的。CVS 库主要有两个分枝，稳定版分支和开发分支。CVS 的稳定版分支只允许为了修改缺陷而进行修改。新功能的开发需要在开发分支进行。稳定版的版本号都是双数的，例如 2.4.x 等，而开发分支的版本好都是单数的，比如 2.5.x 等。对于单数版本，使用者最好等待新的稳定版本的发布再使用。当然直接从 CVS 下载最新的开发版本也是可以的。

wxWidgets 的 API 的修改通常是由开发者在 wx-dev 邮件列表里讨论以后决定的。

除了以上这些，很多其它的 wxWidget 相关的项目都拥有他们自己的社区。例如 wxPython 社区和 wxPerl（参见附录 E：wxWidgets 的第三方工具）社区.

# 1.4 wxWidgets 和面向对象编程

# 1.4 wxWidgets 和面向对象编程

和大多数现代的 GUI 编程框架一样，wxWidgets 大量使用了面向对象编程的概念。每一个窗口都是一个 C++的对象。这些对象已经被预置了很好的处理机制，可以接收事件并对事件作出相应的反应。用户所看到的，就是这个对象的交互系统中可视化那一部分。作为一个程序开发人员，你所要作的事情就是合理的安排这些可视的行为集来让它们作出的反应看上去更合理。wxWidgets 已经实现了很多默认的行为来让这个工作变的更容易。

当然，面向对象的思想和 GUI 编程并不是同时产生的。但是在 20 世纪 70 年代，由 Alan Kay 和其它一些人设计的面向对象的语言 SmallTalk 却是 GUI 编程历史上的一个重要的里程碑。无论对于用户界面设计来说，还是对计算机程序语言来说，它都是一个创新。虽然 wxWidgets 使用不同的语言和不同的 API，但是就面向对象的原理来说，本质上都是一样的。

# 1.5 wxWidgets 的体系结构

# 1.5 wxWidgets 的体系结构

下表展示的 wxWidgets 的四层体系结构: wxWidgets 公用 API 层，各个平台发行版，用于各个平台的 API 和操作系统层。

| wxWidgets API |   |   |   |   |   |   |   |   |   |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| wxWidgets Port | wxMSW | wxGTK | wxX11 | wxMotif | wxMac | wxCocoa | wxOS2 | wxPalmOS | wxMGL |
| Platform API | Win32 | GTK+ | Xlib | Motif/Lesstif | Carbon | Cocoa | PM | Palm OS Protein APIs | MGL |
| Operating System | Windows/Windows CE | Unix/Linux |   |   | Mac OS 9/ Mac OS X | Mac OS X | OS/2 | Palm OS | Unix/DOS |

下面依次说明目前已有的各个平台发行版本：

wxMSW

这个版本编译和运行在各个版本的微软的 Windows 操作系统上，包括：Windows95,Windows98,WinMe,Windows NT,Windows 2000,Windows Xp 以及 Windows 2003.在 linux 平台上，这个版本也可以使用 Wine 的库进行编译，并且可以被配置成在 WinCE 上运行。除了使用本地原生窗口控件，这个版本也可以配置成使用 wxWidgets 自己的窗口控件。

wxGTK

wxWidgets 的 GTK+版本可以使用 GTK 的 1.x 或者 2.x 版本,支持所有可以运行 X11 和 GTK 的类 Unix 平台 (比如： Linux, Solaris, HP-UX, IRIX, FreeBSD, OpenBSD, AIX 等). 它也可以运行在那些有足够资源的嵌入式平台，比如 GPE Palmtop 环境 (如下图所示). wxWidgets 的 GTK 版本是类 Unix 系统的推荐版本.

![](img/mht3AA9%281%29.tmp)

wxX11

wxWidgets 的 X11 版本使用了 wxUniversal 的窗口控件集，直接运行在 Xlib 上。这使得它很适合嵌入式系统,当然它也可以运行在那些不喜欢 GTK+的桌面系统上。它支持所有可以运行 X11 的 Unix 系统，当然 wxX11 并不像 wxGTK 那样完善。下图演示了 Life 程序使用 wxX11 版本编译运行在一个 iPAQ PDA 的类似 Linux/TinyX 环境下的样子。

![](img/mht3ABB%281%29.tmp)

wxMotif

这个版本的 wxWidgets 可以在大多数拥有 Motif, OpenMotif, 或者 Lesstif 的 Unix 系统上. 既然连 Sun 自己都正准备把它的窗口控件集转向 GNOME 和 GTK+,对于大多数开发者来说，Motif 并不是一个很可靠的选择.

wxMac

wxMac 是为 Mac OS 9 (9.1 以后的版本)和 Mac OS X (10.2.8 以后的版本)准备的. 如果在 Mac OS 9 上编译,你需要 Metrowerks CodeWarrior 的工具包, 如果在 Mac OS X 上编译, 你可以选择 Metrowerks CodeWarrior 工具包或者苹果公司的工具包. 如果使用苹果公司的工具包，你应该使用 Xcode 1.5 或者更高的版本,或者你可以考虑直接使用命令行工具 toolsGCC 3.3 或者其后续的版本.

wxCocoa

这是一个正在进行中的版本, 它使用 Mac OS X 的 Cocoa API. 虽然 Carbon 和 Cocoa 的功能很相似,但是这个版本有可能会支持除 Mac 以外的其它支持 GNUStep 的操作系统。

wxWinCE

Windows CE 版本的 wxWidgets 封装了 WindowsCE 平台上的各种不同的开发包,包括 Pocket PC 和 Smartphone 等. 这个版本包含在 wxMSW 的 Win32 版本中。下面第一副图演示了 Life 程序在 Pocket PC 2003 模拟器上运行的样子。第二副图则演示了 wxWidgets 中的对话框例子运行在一个拥有四个屏幕，分辨率为 176x220 的 SmartPhone2003 上的样子。wxWidgets 作了大量的用户界面适配方面的工作，比如因为 Smartphone 只支持两个菜单按钮，所以在通常显示菜单的地方 wxWidgets 构建了可以折叠的菜单。尽管如此，一些地方仍然需要依靠编程者使用不同的代码，比如应该使用 SetLeftMenu 和 SetRightMenu 函数来代替直接在对话框上增加两个确定和取消按钮。

![](img/mht3ACE%281%29.tmp)

![](img/mht3AD1%281%29.tmp)

wxPalmOS

这个版本是为 Palm OS 6 准备的。到作者写这本书的时候为止，这个版本还处在很初级的阶段，但是已经可以在 Palm OS 6 的模拟器中运行一个很简单的小程序了。参见下图：

![](img/mht3AE4%281%29.tmp)

wxOS2

wxOS2 是一个由别人维护的用于 OS/2 或者 eComStation 的版本(is a Presentation Manager port for OS/2 or eComStation).

wxMGL

这个版本使用了 SciTech 公司的底层图形库,窗口控件使用的是 wxUniversal 中的版本。

内部组织

在内部，wxWidgets 的代码大致分为 6 层：

1.  通用代码被所有的版本使用，包括类的数据结构，运行期类型信息，和一些公共基类比如 wxWindowBase 等，这些基类的代码将被所有它的子类所继承。
2.  一般代码用来实现独立于各个平台的高级窗口控件，在某个平台不具有某种控件的时候将使用这部分代码，比如 wxWizard 和 wxCalendarCtrl。
3.  通用组件基本的窗口控件集，这套控件可以在某个平台（比如 X11 或者 MGL）不具有它自己的窗口控件的时候使用。
4.  平台相关代码调用特定平台的 API 来实现某个类的代码。比如在 wxMSW 中的 wxTextCtrl 控件的实现是封装了 Win32 的 edit 控件。
5.  外来代码存放在一个单独的 contrib 目录中，提供一些非必要但是很有用的类实现。比如 wxStyledTextCtrl
6.  第三方代码不是由 wxWidgets 开发维护但是被 wxWidgets 使用以提供一些很重要的特性的代码，比如 JPEG, Zlib, PNG 和 Expat 库。

每一个平台版本所需要作的事情就是提取它需要的那些层的代码，然后用它那个平台的底层的 API 来实现 wxWidgets 的 API。

当你编译你的代码的时候，wxWidgets 怎么知道要编译哪一个平台的类呢？当你包含一个 wxWidgets 的头文件（比如 wx/textctrl.h）的时候,由于使用的不同的宏定义，实际上你包含的是一个特定平台的头文件(wx/msw/textctrl.h)。然后，当你链接 wxWidgets 的库文件的时候，当然这个库文件也需要是用同样的宏定义编译的。你可以同时拥有多套宏定义，例如你可以有调试版本和发布版本两套宏定义。通过这些宏定义的不同，你可以控制编译器链接 wxWidgets 的动态或者是静态版本，也可以禁止编译某个特定组件，或者是决定你要编译的是 Unicode 版本还是 ANSI 版本。为了达到这个目的，你需要修改 setup.h 或者增加不同的编译选项，这取决于你的编译器。有关这些问题更详细的描述，请参见附录 A，???安装 wxWidgets???

另外一点提示是：虽然 wxWidgets 封装了本地的平台相关的 API，并不意味着在你的 wxWidgets 程序中不可以使用这些 API。只是，在绝大多数情况下，你用不着使用这些 API。

# 1.6 许可协议

# 1.6 许可协议

wxWidgets 采用的是 L-GPL 的许可协议外加一个附加条款。你可以从 wxWidgets 的网站上或者 wxWidgets 的分发包的 docs 目录里看到这份许可协议。这份协议总体上来说就是：你既可以使用 wxWidgets 开发自由软件，也可以使用它开发商业软件，并且不需要支付任何版权费用。你既可以动态链接 wxWidgets 的运行期库文件，也可以使用静态链接。如果你对 wxWidgets 本身进行了任何改动，你必须公开这一部分的源代码。而完全属于你自己的那部分代码或者库文件则不需要公开。另外，在发布使用 wxWidgets 编写的软件的时候，你还需要考虑 wxWidgets 的一些可选组件自己的许可协议，比如 PNG 和 JPEG 图形库的许可协议，它们和 wxWidgets 的许可协议可能并不完全相同。

本书中所有的例子和源代码同样使用 wxWidgets 的许可协议。

# 第一章小结

# 第一章小结

在这一章里，我们试图向你解释 wxWidgets 是什么，简单的描述了它的历史，大概支持哪些平台以及大概说了一下它的内部的组织结构。

在下一章 "开始使用"里，我们会通过几个例子来展示一下用 wxWidgets 编写应用程序大概是怎样一个样子。