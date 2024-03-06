# 第十五章内存管理,调试和错误处理

# 第十五章内存管理,调试和错误处理

找到应用程序中的错误是任何开发过程中一个最基本的部分,虽然,也许它并不是那么有诱惑力.本章用来介绍 wxWidgets 提供的用来检测内存问题的机制.同时也涉及了一些构建更可靠和更容易调试的应用程序的内容,比如怎样构建具有自防御功能的程序.我们还将解释什么时候应该在堆上创建对象,什么时候则应该在栈上创建对象,以及怎样使用运行期类型信息机制,模块机制以及 wxWidgets 提供的 C++异常支持等.最后,我们会提供一些通用的调试方法提示.

# 15.1 内存管理基础

# 15.1 内存管理基础

和大多数的 C++程序一样,你可以在栈上创建对象,也可以在堆上通过 new 来创建对象.前者的生命周期仅限于其作用范围,当代码离开其作用范围时,对象的析构函数被调用,对象被释放.而堆上的对象则相反,它将一直存在除非你使用 delete 操作释放对象或者整个应用程序退出.

创建和释放窗口对象

一个通用的规则是:想 frame 或者按钮这样的窗口对象总是应该在堆上使用 new 方法创建.因为窗口对象通常有一个不确定的生命周期.或者说,窗口对象通常要等用户决定什么时候关闭并且释放自己.注意 wxWidgets 会自动释放子窗口,因此你只需要调用顶层窗口的 Destroy 函数,而不需要依次调用窗口中其它控件的 Destroy 函数.类似的,在被释放的时候,frame 窗口也会自动释放它的所有的子窗口.不过如果你创建了一个作为 frame 窗口子窗口的顶层窗口(比如另外一个 frame),这个子窗口将不会被自动释放.这又有一个例外,在 MDI(多文档界面)中,子文档窗口将被自动释放,因为它们其实是主文档窗口的一部分,是不应该独立存在的.

你可以在栈上创建对话框窗口,但是它必须是模式对话框.换句话说,你必须使用 ShowModal 函数使其进入一个事件循环,所有的和用户的交互都将在这个事件循环中被处理,并且这些交互都发生在对话框离开其作用域从而被释放之前.

释放 frame 窗口和对话框窗口的方法也许让你感到困惑,我们进一步解释一下.要释放 frame 窗口或者非模式对话框,应用程序应该调用 Destroy 函数,这个函数在相关对话框的事件队列变空以后才会释放窗口,否则可能将事件发送到不存在的窗口导致程序异常.然而对于模式对话框来说,应该首先调用 EndModal 函数退出窗口的事件循环.这个函数通常是在对话框的事件处理函数中被调用的,比如默认的 OK 按钮事件处理函数.在事件处理函数中只能调用 EndModal 而不能调用 Destroy,因为模式对话框很可能是在栈上创建的,如果在事件处理函数中调用 Destroy,很可能导致这个对话框被重复释放(一次是在事件处理函数中,一次是在栈变量退出其作用域时)导致应用程序表现异常.因此,正确的处理顺序是:当用户点击关闭按钮时,产生 wxEVT_CLOSE_WINDOW 事件,这个事件的处理函数调用 EndModal 函数(而不是 Destroy 函数).而对话框则可以在其退出作用域的时候被自动释放,这也是所有 wxWidgets 提供的标准对话框采用的方式.这种方式的另外一个好处在于,当调用 EndModal 函数退出事件循环以后, 你还可以访问对话框变量的公共成员以便获取相应的数据.当然你也可以设计直接在其事件处理函数中释放自己的对话框,不过对于这种对话框,你就不能够在栈上创建它并且也不可以在它被关闭以后还可以访问它的公共成员了.

下面演示的两种使用 wxMessageDialog 的方法都是允许的:

```cpp
// 1) 在栈上创建模式对话框,没有显式的释放函数
wxMessageDialog dialog(NULL, _("Press OK"), _("App"), wxOK|wxCANCEL);
if (dialog.ShowModal() == wxID_OK)
{
    // 2) 创建在堆上的模式对话框,必须使用 Destroy 显式删除
    wxMessageDialog* dialog = new wxMessageDialog(NULL,
        _("Thank you! "), _("App"), wxOK);
    dialog->ShowModal();
    dialog->Destroy();
} 
```

非模式对话框和 frame 窗口通常需要在关闭的时候释放自己,关闭动作可以通过各种方式产生.它们不能在栈上创建,因为它们通常会很快离开自己的作用范围,将被立即释放.

如果你使用了指向窗口对象的指针,需要确定在窗口被释放以后,指针被置为 NULL,这可以在窗口的析构函数或者关闭事件处理函数中完成,如下所示:

```cpp
void MyFindReplaceDialog::OnCloseWindow(wxCloseEvent& event)
{
    wxGetApp().SetFindReplaceDialog(NULL);
    Destroy();
} 
```

创建和复制绘画对象

绘画过程中使用的对象,比如 wxBrush, wxPen, wxColour, wxBitmap 和 wxImage 等,既可以在堆上创建也可以在栈上创建.这些对象在绘画函数中使用时通常在栈上创建(局部变量).这些对象内部使用引用记数机制,因此直接使用对象(而不是对象指针)的系统开销也是非常小的.这种机制的具体作法是:在进行赋值或复制操作的时候,只复制对象内部数据的一个引用而不是整个的内部数据.当然这种机制有时候也会导致一些奇怪的问题,比如某个对象的修改导致所有引用该对象的对象的修改.为了避免这种情况,在需要的时候,你可以通过在构造函数中传递具体的参数来构造一个全新的对象.下面各自举一些例子:

```cpp
// 引用记数
wxBitmap newBitmap = oldBitmap;
// 全新拷贝
wxBitmap newBitmap = oldBitmap.GetSubBitmap(
    wxRect(0, 0, oldBitmap.GetWidth(), oldBitmap.GetHeight()));
// 引用记数
wxFont newFont = oldFont;
// 全新拷贝
wxFont newFont(oldFont.GetPointSize(), oldFont.GetFamily(),
    oldFont.GetStyle(), oldFont.GetWeight(),
    oldFont.GetUnderlined(), oldFont.GetFaceName()); 
```

初始化你的应用程序中的对象

因为你的自定义对象可能先于 wxWidgets 自己的初始化而被创建,有可能在你的自定义对象初始化的时候,wxWidgets 的内部数据包括颜色数据库,字体数据库等都还没有被初始化,因此,不要在你的自定义应用程序类的构造函数中使用 wxwidgets 的预定义值初始化你的对象,如果你需要使用这些值,可以在 OnInit 函数中进行.举例如下:

```cpp
MyApp::MyApp()
{
    // 不要在这个时候作初始化
    // m_font = *wxNORMAL_FONT;
}
bool MyApp::OnInit()
{
    m_font = *wxNORMAL_FONT;
    ...
    return true;
} 
```

在应用程序退出时执行清理

你可以在重载的 wxApp::OnExit 函数中执行大多数的清理工作,这个函数是在所有的窗口都已经被释放,wxWidgets 正准备释放自己的内部数据的时候被调用的.不过有些清理工作还是需要在主窗口的关闭事件处理函数中执行,比如关闭那些可能往主窗口发送数据的进程.

# 15.2 检测内存泄漏和其它错误

# 15.2 检测内存泄漏和其它错误

在理想状态下,当你的应用程序退出时,所有的对象都应该被你的应用程序或者 wxWidgets 本身释放,不会有残留的内存需要操作系统自己去释放.不用关心内存的释放听上去似乎非常的诱人,但是,我们还是要说,你应该自己控制所有对象的释放,而不要把它留给操作系统,因为通常,内存泄漏都是一些其它重大问题的前兆,它可能导致你的系统在一段时间内出现严重的内存问题.而当你已经转而关注程序的其它方面的时候,会过头来研究哪里出现了泄漏是一件非常令人沮丧的事情,因此,让你的程序尽量保持对内存泄漏的零容忍度不是一个坏事.

那么,怎样才可能让你的应用程序拥有这种检测内存泄漏的能力呢?各种各样的第三方工具提供了这种能力甚至更多其它的能力,而且, wxWidgets 也提供了一个自己的简单的内建的内存检测器.如果你想使用它,在 windows 平台上,你需要打开 setup.h 中的几个开关,而在 linux 平台上,configure 程序需要一些特殊的开关:

在 windows 平台上,你需要:

```cpp
#define wxUSE_DEBUG_CONTEXT 1
#define wxUSE_MEMORY_TRACING 1
#define wxUSE_GLOBAL_MEMORY_OPERATORS 1
#define wxUSE_DEBUG_NEW_ALWAYS 1 
```

而对于 configure 程序,需要传递这样的参数:

```cpp
--enable-debug --enable-mem_tracing --enable-debug_cntxt 
```

另外,使用这个系统还有一个限制:到作者停笔之前,它还不支持 MinGW 或 Cygwin 编译器,并且如果你的代码中使用了 STL 或者使用 CodeWarrior 编译器,你将不能使用 wxUSE_DEBUG_NEW_ALWAYS 选项.

如果打开了 wxUSE*DEBUG*NEW*ALWAYS 选项,那么所有使用 new 操作分配对象的地方都将被 new (*_TFILE**,__LINE**)代替,后者已经被重新使用调试版本的定制内存分配和释放机制进行实现.如果想不重新定义 new 而显式使用调试版本的 new 过程,你可以在使用 new 的地方用 WXDEBUG_NEW 代替.

最简单的使用这个系统的方法是:什么都不做,就只运行你的程序,作该做的事情,然后退出应用程序,然后检查是否有任何内存泄漏被报告.下面的内存泄漏报告的一个例子:

```cpp
There were memory leaks.

- Memory dump -
.\memcheck.cpp(89): wxBrush at 0xBE44B8, size 12
..\..\src\msw\brush.cpp(233): non-object data at 0xBE55A8, size 44
.\memcheck.cpp(90): wxBitmap at 0xBE5088, size 12
..\..\src\msw\bitmap.cpp(524): non-object data at 0xBE6FB8, size 52
.\memcheck.cpp(93): non-object data at 0xBB8410, size 1000
.\memcheck.cpp(95): non-object data at 0xBE6F58, size 4
.\memcheck.cpp(98): non-object data at 0xBE6EF8, size 8

- Memory statistics -
1 objects of class wxBitmap, total size 12
5 objects of class nonobject, total size 1108
1 objects of class wxBrush, total size 12

Number of object items: 2
Number of non-object items: 5
Total allocated size: 1132 
```

上面的例子显示有一个 wxBrush 对象和一个 wxBitmap 对象被分配了但是没有被释放,还有一些其它的未知的对象没有被释放,因为它们没有提供 wxWidgets 的运行期类型信息,因此无法确定对象的类型.在某些集成开发环境中,你可以通过双击报告行显示相应的代码中分配这个对象的位置,这通常是检查内存泄漏问题一个很好的开始.当然,为了最好的报告效果,最好给任何继承自 wxObject 都提供运行期类型信息,方法是,在类声明的部分增加 DECLARE_CLASS(class)宏,在类实现的地方增加 IMPLEMENT_CLASS(class, parentClass)宏.

这个内存检测系统还试图检测那些内存越界或重复释放的错误.在分配内存的时候,它自动在已经分配的内存块上设置一个"good"标记, 在释放的时候将会检查这个标记,如果释放的内存块没有这个标记,则报告一个内存释放错误.这将帮助你发现那些隐藏的,也许在多次运行以后才会导致系统异常的内存错误.

wxDebugContext 类的一些静态函数也很有用处,你可以通过 PrintClasses 函数获取当前系统中分配的对象的列表, PrintStatistics 函数可以打印出当前系统中的已知类型对象和未知类型对象的个数.使用 SetCheckpoint 函数,你可以告诉 wxDebugContext 忽略这个函数调用以前的内存分配动作,而仅关注这以后的内存分配.详情请参考 samples/memcheck 例子或者 wxDebugContext 的相关手册.

除了 wxWidgets 内建的基本系统,你还可以使用别的商业的或者免费的内存检测软件,商业软件比如:BoundsChecker, Purify 或 AQtime 等.自由软件比如:StackWalker,ValGrind,Electric Fence 或来自 Fluid Studios 的 MMGR 等.如果你使用的是 Visual C++,wxWidgets 使用的是编译器内置的内存检测机制,它不会给出类的名字但是会给出行号.最好是打开 wxUSE_DEBUG_NEW_ALWAYS 选项,因为它将重定义 new 过程,除非打开这个选项导致别的第三方的库出现兼容方面的问题.

# 15.3 构建自防御的程序

# 15.3 构建自防御的程序

通常一个缺陷在导致其产生的逻辑错误产生很久以后才会表现出来.如果一个异常的或者不正确的值没有被及时检测到,那么通常在应用程序又执行了几千行代码以后,应用程序才会崩溃或者作出令人费解的表现.为解决这样的问题你可能要花上很长的时间.但是,如果你在你的代码中增加一些普通的检查,来检测函数的某个地方出现了错误的值,那么你的代码将最终变得非常强壮,你将可能避免你和你的用户陷入到大麻烦中去.这就是所谓的自防御程序.你的代码可以防止自己被别的代码或者自己内部的某些错误逻辑搞的一团糟.因为大多数的这些检测在正式发布版中都将被移除,因此,它们占用的系统开销几乎可以忽略不计.

正如你期待的那样,wxWidgets 在其内部使用了大量的错误检测代码,你可以在你的代码中直接使用这些宏.这些宏主要分为三类,每一类都有多种形式:第一类是 wxASSERT,如果它的参数不等于 True,它将显示一个错误信息.这种检测仅存在于调试版本中.wxFAIL 则将使用产生一个错误信息,其作用相当于 wxASSERT(false).它也仅会存在于调试版本中.wxCHECK 则判断条件是否成立,如果不成立则返回某个值并显示错误信息.和前面两种不同的是,在正式版本中,wxCHECK 的代码仍然有效,但是将不显示任何消息.

下面的例子演示了怎样使用这些宏:

```cpp
// 两个正数相加
int AddPositive(int a, int b)
{
    // 检查是否为一个正数
    wxASSERT(a &gt; 0);
    // 检查是否位一个正数,显示定制的消息
    wxASSERT_MSG(b &gt; 0, wxT("The second number must be positive!"));
    int c = a + b;
    // 如果相加的结果不为正数,显示一个错误的消息并且返回-1.
    wxCHECK_MSG(c &gt; 0, -1, wxT("Result must be positive!"));
    return c;
} 
```

你还可以使用 wxCHECK2 和 wxCHECK2_MSG 宏,这两个宏在条件不满足的时候可以执行任意的操作而不仅仅是设置一个返回值. 而 wxCHECK_RET 则可以被用在没有返回值(void)的函数中.另外一些不太常用的宏包括 wxCOMPILE_TIME_ASSERT 和 wxASSERT_MIN_BITSIZE 等,请参考 wxWidgets 的相关手册.

下图演示了一个断言不满足时显示消息的对话框的样子,在这个对话框上有三个按钮,Yes 按钮停止当前程序的运行,No 按钮则忽略这个断言失败,而 Cancel 按钮则表示以后的断言失败都不需要显示.如果程序正运行在某个调试器中,则程序终止运行将返回到调试器中,你可以打印出当前的函数调用堆栈,进而可以知道断言失败的准确位置以及当时各个参数的值.

![](img/mht31B7%281%29.tmp)

# 15.4 错误报告

# 15.4 错误报告

有时候你需要在控制台或者一个对话框中显示一条消息以帮助调试或者用来提示那些不能被你的代码正常处理的行为.wxWidgets 提供了很多用于记录错误的函数,这些函数工作方式各不相同,你可以使用它们来进行运行情况的报告和记录.比如,当你正在分配一个很大的图片时,可能由于这个图片太大了,系统无法分配足够的资源,系统将使用 wxLogError 函数显示一个对话框来报告这个错误(如下图所示).又或者说,你想将某个参数的值打印在调试窗口中以便于调试,你可以使用 wxLogDebug 函数.究竟这些错误信息或者调试信息是显示在终端上,对话框中还是别的什么地方,取决于你所使用的函数,以及当前激活的 wxLog 目标对象,我们将在稍后的部分描述相关内容.

![](img/mht4639%281%29.tmp)

所有的这种记录函数都拥有类似于 printf 或 vprintf 的语法,也就是:第一个参数是格式化文本参数,后面是不定类型的变量或者一组指向变量的指针,如下所示:

```cpp
wxString name(wxT("Calculation"));
int nGoes = 3;
wxLogError(wxT("%s does not compute! You have %d more goes."),
           name.c_str(), nGoes); 
```

下面我们逐个描述一下这些函数:

wxLogError 函数用来显示那些必须显示给用户的错误消息.其默认行为是弹出一个对话框来通知用户相应的错误.那为什么不直接使用 wxMessageBox 呢?原因是,首先 wxLogError 提示的错误消息是可以通过创建一个 wxLogNull 的 Log 目标来屏蔽让其不显示出来的,而且这些消息也是排在系统队列中,在系统空闲的时候显示的.因此如果有一系列的错误同时出现,它们将显示在同一个对话框中,而如果使用 wxMessageBox,你的用户可能不得不不停的点击 OK 按钮.

wxLogFatalError 和 wxLogError 类似,不过除了显示错误消息,它还使用标准的系统调用 abort,以错误码 3 结束整个程序的运行.和其它类似的函数不同,这个函数显示的消息不能通过设置空的打印目标的方法来屏蔽.

wxLogWarning 也和 wxLogError 类似,不过显示的信息将作为警告而不是错误.

wxLogMessage 则用来显示所有正常的,信息类型的消息,默认也是显示在对话框中.

wxLogVerbose 则用来显示那些冗长的详细信息.通常情况下,这种信息是不显示的,但是如果用户想显示它以便了解程序运行的更详细的情况,可以通过使用 wxLog::SetVerbose 函数改变这种默认的行为.

wxLogStatus 则用来显示状态条消息,如果当前的 frame 窗口拥有一个状态条,那么这个消息将显示在那里.

wxLogSysError 通常主要被 wxWidgets 自己使用,它用来报告那些系统错误,同时会显示由 errno 或者 GetLastError(依平台的不同)指示的错误码和错误消息.它的另外一种形式允许你的第一个参数的位置显式的指示系统错误码.

wxLogDebug 用来显式调试信息.这些信息只在调试版本中(定义了**WXDEBUG**宏)才会出现,在正式版本中将被移除.在 windows 平台上,只有当程序在一个调试器中运行或者使用第三方工具比如来自[`www.sysinternals.com 的`](http://www.sysinternals.com 的) DebugView 工具运行的时候才会显示出来.

wxLogTrace 和 wxLogDebug 的功能几乎完全一样,也是只在调试模式才会显示信息.之所以有这个函数,是为了提供一个和普通的调试模式不同的级别以便区分普通的调试信息和用于跟踪的调试信息.它的另外一种形式允许你指定一个掩码,通过 wxLog:: AddTraceMask 函数设置了掩码以后,只有掩码符合的跟踪消息才会被显示出来以实现跟踪消息的过滤.比如在 wxWidgets 内部使用了 mousecapture 掩码.如果你设置了这个掩码,在鼠标移动的时候你将看到跟踪信息.

```cpp
void wxWindowBase::CaptureMouse()
{
    wxLogTrace(wxT("mousecapture"), wxT("CaptureMouse(%p) "), this);
...
}
void MyApp::OnInit()
{
    // Add mousecapture to the list of trace masks
    wxLog::AddTraceMask(wxT("mousecapture"));
    ...
} 
```

你可能会疑惑,为什么不直接使用 C 的标准输入输出函数或者 C++的流呢?简短的回答是,它们都是很不错的机制,但是并不一定适用于 wxWidgets.wxLog 机制主要有以下三个优点:

首先,wxLog 是可移植的.常用的 printf 语句或者 C++的 cout 流和 cerr 流在 unix 系统下工作是没有问题的,但是在 windows 系统中,对于图形化界面的应用程序,这些函数或流可能不能正常显示需要的内容.因此,你可以使用 wxLogMessage 作为 printf 的一个简单的替代品.

你也可以通过下面的方法将所有的 log 信息转向标准的 cout 流:

```cpp
wxLog *logger=new wxLogStream(&cout);
wxLog::SetActiveTarget(logger); 
```

另外,将发送往 cout 的输出重定向到一个 wxTextCtrl 控件也是可行的,这需要使用到 wxStreamToTextRedirector 类.

其次:wxLog 更灵活.使用 wxLog 机制的输出可以被分情况重定向或者隐藏,比如只显示错误消息和告警消息,忽略所有正常的信息.而如果使用标准的函数或流,这是不可能的或者说是很难作到的.

最后:wxLog 机制也是更完善的机制.通常,当有错误发生的时候,应该给用户显示一些信息.让我们来举一个简单的例子,假如你正在进行写文件操作,这时候发生了磁盘空间不足的情况,这种错误是被 wxWidgets 内部(wxFile::Write)处理的,因此,调用这个函数只能知道写动作发生了异常,至于是什么类型的异常则很难得到,如果在这种情况下使用 wxLogError 函数,正确的错误码和相应的错误信息都将显示给用户.

现在我们来描述以下 wxWidgets 的 Log 机制是怎样工作的,以便你处理那些默认没有提供的行为.

wxWidgets 有一个 log 目标的概念:它其实就是一个 wxLog 的派生类.它需要实现 wxLog 定义的那些虚函数,这些函数将在相应的 Log 函数被调用的时候使用.任何时候都只有一个 log 目标是活动的.log 目标通常的使用方法就是调用 wxLog:: SetActiveTarget 来安装这个目标,安装以后的目标将在相应的 log 函数被调用的时候自动使用.

要创建一个自定义的 log 目标,你只需要创建一个 wxLog 的派生类,并实现其虚函数 DoLogString 和(或)DoLog.如果你对 wxWidgets 默认的增加时间戳和信息类型的格式化方法感到满意,只是想更改信息的目的地,那么实现 DoLogString 函数就足够了,而重载 DoLog 函数则使得你可以任意的定制输出信息的格式,不过同时你也需要自己区分信息的各种类型.你可以参考 src/common/log.cpp 文件看看 wxWidgets 是怎么作到这一点的.

wxWidgets 自己实现了几个 wxLog 的派生类,你也可以读一读它们的代码,这对你创建自定义的 log 目标也是有好处的.这些预定义的 log 目标包括:

wxLogStderr 将所有的信息输出到 FILE*作为参数的文件中,如果 FILE*为空,则输出到标准错误输出.

wxLogStream 和 wxLogStderr 功能相同,不过它使用标准 C++的 ostream 类和 cerr 流来代替 FILE*和 stderr.

wxLogGui 则是 wxWidgets 所有 wxWidgets 程序默认使用的 log 目标,依平台的不同它实现了不同的输出处理.

wxLogWindow 则提供了一个类似"跟踪终端"之类的窗口,这个窗口将显示所有的输出信息,同时这些信息也将显示在之前的 log 目标上.这个跟踪终端窗口提供了清除信息,关闭窗口以及将所有信息保存到文件中的功能.

wxLogNull 则被用来临时阻止某些错误信息的输出,比如你打开不存在的文件的时候将显示一个错误信息,有时候你不希望显示这个信息,可以在栈上创建一个 wxLogNull 变量,在这个变量的作用域范围内,没有任何错误信息将被显示,而离开了其作用域,则所有的信息又可以正常显示了.

```cpp
wxFile file;
// wxFile.Open()在打开一个不存在的文件时通常会显示错误信息,但是在这里我们不想看到这个信息.
{
    wxLogNull logNo;
    if ( !file.Open("bar") )
      ... process error ourselves ...
} // wxLogNull 的析构函数被调用,旧的 log 目标被恢复.
wxLogMessage("..."); // 可以被显示 
```

有时候你也许希望将信息输出到多个地方,比如,你可以希望所有的信息在正常显示的同时被保存在某个文件中,这时候你可以使用 wxLogChain 和 wxLogPassThrough,如下所示:

```cpp
// 这将隐式的设置当前 log 目标
wxLogChain *logChain = new wxLogChain(new wxLogStderr);
// 所有的输出将被同时显示在 stderr 和通常的地方
// 不要直接删除 logChain 指针,这会导致当前活动 log 目标为一个不确定的指针.
// 应该使用 SetActiveTarget.
delete wxLog::SetActiveTarget(new wxLogGui); 
```

wxMessageOutput VS wxLog

有时候,使用 wxLog 不太合适,这主要是因为 wxLog 对输出的信息作了过多的处理,并且会等待空闲的时候才会显示这些信息.而 wxMessageOutput 和它的派生类则可以作为你的底层 printf 的替代品,来在 GUI 和命令行程序中使用.你可以象使用 printf 函数那样使用 wxMessageOutput::Printf 函数,比如,如果你想把信息打印在标准错误输入:

```cpp
#include "wx/msgout.h"
wxMessageOutputStderr err;
err.Printf(wxT("Error in app %s.\n"), appName.c_str()); 
```

wxMessageOutputDebug 将信息显示在调试器终端或者是标准错误输出中,这主要看程序是以什么方式运行的,和 wxLogDebug 不同,wxMessageOutputDebug 输出的信息在正式版本中将不会被移除.GUI 应用程序还可以使用 wxMessageOutputMessageBox 来即时显示消息,而不比象 wxLog 那样需要搜集(其它 Log 信息)和等待(系统空闲),同样的还存在一个 wxMessageOutputLog 类,它将消息输出到 wxLogMessage.

和 wxLog 类似,wxMessageOutput 也有一个当前的目标,这个目标可以通过 wxMessageOutput::Set 设置,通过 wxMessageOutput::Get 获取.默认的目标是系统初始化的时候由 wxWidgets 设置的,在命令行程序中使用的是 wxMessageOutputStderr,在 GUI 程序中使用的是 wxMessageOutputMessageBox.wxWidgets 内部经常使用这个对象,比如在 wxCmdLineParser 类中,使用了下面的代码:

```cpp
wxMessageOutput* msgOut = wxMessageOutput::Get();
if ( msgOut )
{
    wxString usage = GetUsageString();
    msgOut->Printf( wxT("%s%s"), usage.c_str(), errorMsg.c_str() );
}
else
{
    wxFAIL_MSG( _T("no wxMessageOutput object?") );
} 
```

# 15.5 提供运行期类型信息

# 15.5 提供运行期类型信息

提供运行期类型信息

和大多数编程框架一样,wxWidgets 提供了比标准 C++更多的运行期类型信息.这对于在运行期依对象类型决定行为,或者在上节提到的错误报告中都是很有用处的.并且它还允许你通过对象名来创建一个那种类型的变量.需要注意的是:只有派生自 wxObject 的类才可以使用 wxWidgets 的 RTTI (运行期类型信息).

如果你不需要动态创建功能,你需要在类声明中使用 DECLARE*CLASS(class)宏,而在类实现中使用 IMPLEMENT_CLASS(class, baseClass)宏,反之,则需要使用 DECLARE_DYNAMIC_CLASS(class)宏和 IMPLEMENT_DYNAMIC* CLASS(class, baseClass)宏.另外,如果你希望使用动态创建功能,你还需要保证你的类拥有一个默认的构造函数,否则在编译那些用来动态产生某个对象的代码时, 编译器可能会报错.

下面是定义和动态创建一个对象的例子:

```cpp
class MyRecord: public wxObject
{
DECLARE_DYNAMIC_CLASS(MyRecord)
public:
    MyRecord() {}
    MyRecord(const wxString& name) { m_name = name; }
    void SetName(const wxString& name) { m_name = name; }
    const wxString& GetName() const { return m_name; }
private:
    wxString m_name;
};
IMPLEMENT_DYNAMIC_CLASS(MyRecord, wxObject)
MyRecord* CreateMyRecord(const wxString& name)
{
    MyRecord* rec = wxDynamicCast(wxCreateDynamicObject(wxT("MyRecord")), MyRecord);
    if (rec)
        rec->SetName(name);
    return rec;
} 
```

当 CreateMyRecord 被调用的时候,wxCreateDynamicObject 负责创建所需的对象,而 wxDynamicCast 则负责进行类型检查,如果检查失败则返回 NULL.也许你觉得这个代码看上去并没有什么实际的用处,但是在从文件加载一堆不同类型的对象的时候,这是非常有用的.对象的数据和它的名称被一起存放在文件中,通过名字创建一个对象的实例,然后再使用这个实例加载相应的数据.

下面我们来介绍以下运行期类型信息的一起其它相关宏:

CLASSINFO(class)返回一个指向 wxClassInfo 类型的指针.你可以使用 wxObject::IsKindOf 函数来判断某个对象是否是对应的类型:

```cpp
if (obj->IsKindOf(CLASSINFO(MyRecord)))
{
    ...
} 
```

使用 DECLARE_ABSTRACT_CLASS(class)和 IMPLEMENT_ABSTRACT_CLASS(class, baseClass)来定义虚类.

使用 DECLARE_CLASS2(class)和 IMPLEMENT_CLASS2(class, baseClass1, baseClass2)来定义有两个父类的类.

使用 DECLARE_APP(class)和 IMPLEMENT_APP(class)来使得 wxWidgets 知道整个应用程序类的运行期类型信息.

wxConstCast(ptr, class)用来代替 const_cast<class *>(ptr),如果编译器不支持 const_cast,则使用旧的 C 语言的类型强制转换.

wxDynamicCastThis(class)相当于 wxDynamicCast(this, class), 但是后者在某些编译器上会导致永真比较告警,因为它将测试是否指针为空,而 this 指针永远不可能为空,前者可以避免这个告警.

wxStaticCast(ptr, class)在调试模式将检查强制类型转换的有效性(如果 wxDynamicCast(ptr, class) == NULL 将引发断言错误)然后返回 static_cast<class*>(ptr)的等同结果.

wx_const_cast(T, x)在编译器支持的情况下等同于 const_cast<T>(x),否则等同于(T)x(C 语言类型强制转换语法).和 wxConstCast 不同的是,它强制转换的是 T 本身而不是指向 T 的指针,而且参数的顺序也和标准强制转换的顺序相同.

wx_reinterpret_cast(T, x)则相当于 reinterpret_cast<T>(x),如果编译器不支持则等同于(T)x.

wx_static_cast(T, x)等同于 static_cast<T>(x),如果编译器不支持则等同于(T)x. 和 wxStaticCast 不同的是,它不做类型检查,并且采用和标准的静态转换相同的参数顺序,而且转换的也是 T 而不是指向 T 的指针.

# 15.6 使用 wxModule

# 15.6 使用 wxModule

wxWidgets 的模块管理系统是一个很简单的系统,它允许应用程序(以及 wxWidgets 自己)可以定义将被 wxWidgets 自动在开始和退出时执行的初始化和资源清理代码.这有助于避免应用程序在 OnInit 函数和 OnExit 函数中依它们功能的需要添加过多的代码.

要定义一个这样的模块,你需要实现一个 wxModule 的派生类,重载其 OnInit 和 OnExit 函数,然后在其声明部分使用 DECLARE_DYNAMIC_CLASS 宏,在其实现部分使用 IMPLEMENT_DYNAMIC_CLASS 宏(它们可以位于同一个文件内). 在系统初始化的时候,wxWidgets 会找到所有 wxModule 的派生类,创建一个它的实例然后执行其 OnInit 函数,而在系统退出时执行其 OnExit 函数.

举例如下:

```cpp
// 下面这个模块用来自动进行 DDE 的初始化和清除动作.
class wxDDEModule: public wxModule
{
DECLARE_DYNAMIC_CLASS(wxDDEModule)
public:
    wxDDEModule() {}
    bool OnInit() { wxDDEInitialize(); return true; };
    void OnExit() { wxDDECleanUp(); };
};
IMPLEMENT_DYNAMIC_CLASS(wxDDEModule, wxModule) 
```

# 15.7 加载动态链接库

# 15.7 加载动态链接库

如果你想要使用位于动态链接库中的函数,你需要使用 wxDynamicLibrary 类.使用方法是:在其构造函数或者 Load 函数中传递动态链接库的文件名,如果你不希望 wxWidgets 自动增加类似.dll(windows)或者.so(linux)这样的扩展名,还需要传递 wxDL_VERBATIM 参数.如果加载成功,就可以使用 GetSymbol 函数通过函数名使用其中的函数了.下面的例子演示了怎样加载和使用 windows 的标准控件动态链接库:

```cpp
#include "wx/dynlib.h"
INITCOMMONCONTROLSEX icex;
icex.dwSize = sizeof(icex);
icex.dwICC = ICC_DATE_CLASSES;
// 加载 comctl32.dll
wxDynamicLibrary dllComCtl32(wxT("comctl32.dll"), wxDL_VERBATIM);
// 定义 ICCEx_t 类型
typedef BOOL (WINAPI *ICCEx_t)(INITCOMMONCONTROLSEX *);
// 获取 InitCommonControlsEx 符号表
ICCEx_t pfnInitCommonControlsEx =
    (ICCEx_t) dllComCtrl32.GetSymbol(wxT("InitCommonControlsEx"));
// 调用获取的函数.
if ( pfnInitCommonControlsEx )
{
    (*pfnInitCommonControlsEx)(&icex);
} 
```

wxDYNLIB_FUNCTION 宏使得上面代码中的 GetSymbol 行更为简洁:

```cpp
wxDYNLIB_FUNCTION( ICCEx_t, InitCommonControlsEx, dllComCtl32 ); 
```

wxDYNLIB_FUNCTION 使得你只需要使用一次返回值类型作为其第一个参数,它将创建一个对应的变量,变量名由函数名和 pfn 前缀组成.

如果动态链接库被成功加载,它将在对应的 wxDynamicLibrary 对象被释放的时候,在其析构函数中自动被卸载,如果你希望在 wxDynamicLibrary 被释放以后继续使用相应的函数,你应该调用首先调用 wxDynamicLibrary 的 Detach 函数.

# 15.8 异常处理

# 15.8 异常处理

wxWidget 创立的时间比起"异常"的概念引入 C++要早的多,因此在其代码中已经花费了大量的力气来应付各种各样的异常,因此,可以说,整个 wxWidgets 框架内部都没有使用 C++的异常机制.当然,这并不意味着你不可以在你的代码中使用 C++的异常机制,相反,你在你的代码中使用它是安全的,而且 wxWidgets 也会帮助你这样作.

要在你的程序中使用异常处理机制,最简单的方法就是根本忽略它的存在,既然 wxWidgets 不会抛出任何异常,你又何必去处理异常呢?除非你的代码自己抛出了一些异常.这是最简单的方法,但是有时候,对于处理各种可能遇到的错误来说,这种方法是不够的.

另外一个策略是你只用异常机制来处理那些非常致命的系统错误.这种情况下,你不寄希望于你的程序可以从这种致命错误中恢复,它所作的事情只是让你的程序以一种更绅士的方式结束.这种情况下,你只需要重载你的 wxApp 派生类的 OnUnhandledException 函数来执行资源清除工作,注意这时候所有和异常有关的信息已经被清除了.如果你需要这些信息,你需要在 OnRun 函数中针对调用基类函数的语句使用 try/catch 语句块. 这将使得你可以捕获在应用程序主循环中引发的异常.如果你还希望处理在应用程序初始化和退出时候引发的异常,你需要在你的 OnInit 和 OnExit 函数中使用 try/catch 语句.

最后,如果你希望在异常发生的时候,你的应用程序可以从异常中恢复并且继续运行,那么:如果你程序的异常主要集中在某个类(或其派生类)的事件处理函数中,你可以你可以在这个类的 ProcessEvent 函数中统一处理这些异常,如果这是不切实际的,你还可以考虑重载 wxApp:: HandleEvent 函数,它将允许你拦截并处理任何由事件处理函数引发的异常.

wxWidgets 的异常处理机制默认是打开的,它取决于 wxUSE_EXCEPTIONS 标记被设置为 1.但是如果它被设置为 0,在 windows 版本中,你需要修改 include/wx/msw/setup.h 将其更改为 1,或者在别的平台上运行 configure 时增加-- enable-exceptions 开关.而将其设置为 0 或者使用--disable-exceptions 开关将会产生更为小巧和相对快速的 wxWidgets 库.另外,在 windows 平台下,如果你使用的是 Visual C++,你希望使用 wxApp::OnFatalException 函数来处理异常而不是引发一个 GPF(一般保护性错误),你可以在你的 setup.h 中将 wxUSE_ON_FATAL_EXCEPTION 设置为 1.相反的,如果你宁愿将这种错误扔给你的调试器,将它设置成 0.

wxWidgets 自带一个使用异常的例子,位于 samples/except 目录中.

# 15.9 调试提示

# 15.9 调试提示

自我保护的程序,错误报告何其它的编码技术只能帮助你这么多了,要单步跟踪你的代码,检查每个变量的值,准确的告诉你你的程序有什么异常行为或者从代码的什么地方退出,你还需要使用一个调试工具.因此,针对你的代码,你至少需要维护两套配置文件,一套调试版本何一套正式版本.调试版本将包含更多的错误检测,将关闭编译器的优化开关并且将包含调试程序需要的文件名,行号等调试信息.在调试模式,宏**WXDEBUG**总是被定义了,因此你可以使用这个宏来编写那些仅存在于调试版本中的代码,不过类似 wxLogDebug 这样的函数,即使你没有使用**WXDEBUG**宏将其包含,它仍然将在正式版本中被移除.

确实有很多人从来没有使用过调试器,但是在你熟悉了这些工具以后,它将大大降低你的工作量.在 windows 平台上,VC 自带了一个很不错的调试器.如果你使用的是 GCC,你可以使用 GDB 工具包,它工作在命令行模式下,你也可以使用一些集成了 GDB 的 IDE 环境.更多的信息可以参考附录 E,"wxWidgets 的第三方工具".

wxWidgets 支持同时编译多个版本.在 windows 平台上,你可以给对应的 Makefile 传递 BUILD=debug 或 BUILD=release 这样的开关.如果你使用的是 configure 程序,你可以配置不同的版本编译在不同的目录,然后在各个版本中使用类似-- enable-debug 或--disable-debug 这样不同的开关.某些集成开发环境出于各种各样的原因不允许你的应用程序同时使用不同的配置文件,对于这样的集成开发环境,作者的忠告是:不要使用它们.

调试 X11 错误

极少的情形下,wxGTK 程序会由于 X11 的错误而崩溃,这时候你的应用程序将立即退出而不给你任何栈调用情况的打印,这种问题是非常难以跟踪的,在这种情况下,你需要象下面代码展示的那样,增加一个错误处理函数:

```cpp
#if defined(__WXGTK__)
#include &lt;X11/Xlib.h&gt;
typedef int (*XErrorHandlerFunc)(Display *, XErrorEvent *);
XErrorHandlerFunc gs_pfnXErrorHandler = 0;
int wxXErrorHandler(Display *display, XErrorEvent *error)
{
    if (error->error_code)
    {
          char buf[64];
          XGetErrorText (display, error->error_code, buf, 63);
          printf ("** X11 error in wxWidgets for GTK+: %s\n  serial %ld error_code %d
 request_code %d minor_code %d\n",
           buf,
           error->serial,
           error->error_code,
           error->request_code,
                   error->minor_code);
    }
    // 去掉下面的注释以便将错误重定向道你的处理函数.
#if 0
    if (gs_pfnXErrorHandler)
        return gs_pfnXErrorHandler(display, error);
#endif
    return 0;
}
#endif
  // __WXGTK__
bool MyApp::OnInit(void)
{
#if defined(__WXGTK__)
    // 安装错误处理函数
    gs_pfnXErrorHandler = XSetErrorHandler( wxXErrorHandler );
#endif
    ...
    return true;
} 
```

现在,你的应用程序在遇到这样的错误的时候,将会产生一个普通的段错误.如果你在启动你的应用程序的时候传递了--sync 参数,这个段错误将正好显示在被传递了错误参数的 X11 函数的地方.

一个简单有效的定位问题方法

如果你确实碰到了一个很难定位的问题,一个好的方法是使用尽量少的代码来重现这个问题,你可以修改 wxWidgets 自带的任何一个例子,增加一些代码来重现你的问题,或者把你的代码制作一份拷贝,逐段的去掉那些不影响这个错误的代码,直至你可以准确的定位出是那些代码导致了这个错误的产生.如果你认为这是 wxWidgets 本身的错误,把你修改后的导致问题出现的 wxWidgets 例子发送给 wxWidgets 社区,相信这个问题将被很快修正.

调试一个发布版本

某些情况下,你的应用程序可能在调试版本工作正常,而在正式版本中则工作不正常,这可能是由于编译器使用的不同运行期库文件的细微差别导致的.如果你正在使用的是 VC,你可以创建一个和调试版本一模一样的配置,但是却定义了 NDEBUG 宏,这将使得你的代码和 wxWidgets 和调试信息都具备,运行的时候却使用的是发布版本的运行库.这至少可以让你确定是不是由于运行期库的原因导致了这个问题.

通常当你发布你的应用程序的时候都将使用去除了所有调试信息的版本,但是有时候你的客户会遇到一些在你的机器上很难重现的问题,,这时候你可能想给你的用户发送一份调试版本的程序(在 windows 平台上,通常你需要使用静态编译的方法以避免同时发送那些调试模式的动态链接库).然后在你的客户的机器上,可以使用一个叫做 Dr.Watson 的程序来运行你的包含调试信息的程序,在程序异常退出的时候,将会产生一个文件记录当时的情况.参考你的编译器的信息以及网上的教程来了解怎样使用由 Dr.Watson 产生的这个文件来定位你的代码中的异常.

如果你使用的是 MinGW,你可以使用一个叫做 Dr.MinGW 的工具来在程序异常退出时候打印出一个有用的当前函数调用堆栈(当然, 如果你的代码包含调试信息(打开了-g 开关)的话).你可以从 [`www.mingw.org`](http://www.mingw.org) 下载这个工具,如果你的客户有耐心并且很合作,你可以把这个工具发送给他们然后让他们把产生的报告发送给你.

在 Unix 平台上,调试版本的可执行文件可以产生一个 core 文件(这依赖于系统的设置,参考 Unix 命令 ulimit 的 man 手册).你可以象你平时调试可执行文件那样使用这个 core 文件,以便观察程序退出时候的现场情况.然后,这个 core 文件可能会很大,你的客户可能不大愿意发送给你这样的一个 core dump 文件.

另外的一个替代方案是,你可以在你的程序中自己记录程序的重要的执行情况.这方面你可以参考 wxWidgets 手册中 wxDebugReport 类相关的内容,这个类可以产生一个合适 email 给程序开发商的报告.类似的,你还可以使用来自 http: //wxcode.sf.net 的 wxCrashPrint(for linux)或者来自 [`www.codeproject.com/tools/blackbox.asp`](http://www.codeproject.com/tools/blackbox.asp) 的 BlackBox(for windows).

# 第十五章小结

# 第十五章小结

在这一章里,我们讨论了内存管理和错误检测各个方面的内容.你现在应该了解到,什么时候使用 new 来创建对象,什么时候直接使用栈变量,你的应用程序应该怎样进行资源清除工作,怎么识别内存泄漏,怎么使用宏来创建有自我保护意识的程序.你也看到了怎样动态创建对象,也知道了何时使用 wxLogDebug 何时使用 wxLogError.也看到了怎样在 wxWidgets 中使用 C++异常机制,我们还对怎样调试应用程序给出了一些提示. 接下来我们来看看怎样让你的应用程序支持多种语言.