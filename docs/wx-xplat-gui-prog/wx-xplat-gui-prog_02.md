# 第二章 开始使用

在这一章里，我们将使用一个很小的例子，来建立你对用 wxWidgets 编程的一个大致的印象。我们将会来看看一个用 wxWidgets 编写的程序是怎么开始的又是怎样结束的，怎样创建主窗口，怎样响应用户的命令。正如 wxWidgets 所追求的短小精悍的哲学一样，这就是所有这一章我们要涉及的内容。不过在这之前，你可能先要参考附录 A 来安装 wxWidgets.

# 2.1 一个小例子

下图演示了我们的小例子在 Windows 上运行的样子

![](img/mht8E0B%281%29.tmp)

这个小例子创建了一个主窗口(是一个 wxFrame 类的实例),这个主窗口有一个菜单条和一个状态条。菜单条上的菜单按照你的命令显示一个关于窗口或者退出这个小程序。显然这算不上什么杀手级的大程序，但是足以给你展示 wxWodgets 的一些基本原则,并且让你可以相信，随着知识的慢慢积累，有一天你将作出更复杂的（那些杀手级的）程序，

# 2.2 应用程序类

每一个 wxWidgets 程序都需要定义一个 wxApp 类的子类，并且需要并且只能构造一个这个类的实例，这个实例控制着整个程序的执行。你的这个继承自 wxApp 的子类至少需要定义一个 OnInit 函数，当 wxWidgets 准备好运行你写的代码的时候，它将会调用这个函数（和一个典型的 Win32 程序中的 main 函数或者 WinMain 函数类似）。

你定义这个子类的代码可能和下面的代码类似：

```cpp
class MyApp : public wxApp
{
  public:
      virtual bool OnInit();
}; 
```

在这个 OnInit 函数中，你通常应该创建至少一个窗口，对传入的命令行参数进行解析，为应用程序进行数据设置和其它的一些初始化的操作.如果这个函数返回真，wxWidgets 将开始事件循环用来处理用户输入并且在必要的情况下处理这些输入。如果 OnInit 函数返回假， wxWidgets 将会释放它内部已经分配的资源，然后结束整个程序的运行。

接下来我们看一个最简单的 OnInit 函数的实现：

```cpp
bool MyApp::OnInit()
{
   MyFrame *frame = new MyFrame(wxT("Minimal wxWidgets App"));
   frame->Show(true);
   return true;
} 
```

你可能还会注意到上面例子中的 wxT 这个宏，在接下来的例子中，这个宏还会被频繁用到。它的作用是让你的代码兼容 Unicode 模式。这个宏和另外一个*T 宏的作用是完全一样的。使用这个宏也不会带来运行期的性能损失。（你可能还会遇到另外一个类似的"*()"标记，这个标记是用来告诉 wxWidgets 将其中的字符串翻译成其它语言的版本，参见第十六章“编写国际化程序”）。

那么创建 MyApp 的实例的代码在哪里呢？实际上，这是在 wxWidgets 内部实现的，不过你仍然需要告诉 wxWidgets 需要创建哪一个 App 类的实例，所以你还需要增加下面的一个宏:

```cpp
IMPLEMENT_APP(MyApp) 
```

如果没有实现这个类，wxWidgets 就不知道怎样创建一个新的应用程序对象。这个宏除了上述的功能以外，还会检查编译应用程序使用的库文件是否和当前的库文件的版本相匹配，如果没有这种检查，由此而产生的一些运行期的错误可能很难被查出原因。

当 wxWidgets 创建这个 MyApp 类的实例的时候，会将创建的结果赋值给一个全局变量 wxTheApp.你当然可以在你的程序中使用这个变量，但是你可能不得不一遍又一遍的进行从 wxApp 到 MyApp 的类型强制转换。增加下面的这一行声明以后，你就可以调用 wxGetApp()函数，这个函数会返回一个到这个 MyApp 实例的引用，这样用起来就方便多了。

```cpp
DECLARE_APP(MyApp) 
```

一点提示：

即使没有声明 DECLARE_APP,你仍然可以不用进行类型强制转化就直接对 wxTheApp 变量调用 wxApp 的方法.这可以避免在所有的头文件中包含 MyApp 的头文件，对于那些库文件而不是应用程序的代码来说也更有意义，而且还可以缩短编译的时间。

# 2.3 Frame 窗口类

我们来看一看自定义的 Frame 窗口类 MyFrame.一个 Frame 窗口是一个可以容纳别的窗口的顶级窗口，通常拥有一个标题栏和一个菜单栏。下面是我们的例子中这个类的定义，可以将其放在 MyApp 的定义之后：

```cpp
class MyFrame : public wxFrame
{
public:
    MyFrame(const wxString& title);
    void OnQuit(wxCommandEvent& event);
    void OnAbout(wxCommandEvent& event);
private:
    DECLARE_EVENT_TABLE()
}; 
```

这个窗口类的定义有一个构造函数，两个用来把菜单命令和 C++代码相连的事件处理函数，还有一个宏来告诉 wxWidgets 这个类想要自己处理某些事件。

# 2.4 事件处理函数

你也许已经注意到了，事件处理函数在 MyFrame 类中不是虚函数。如果不是虚函数，他们是怎样被调用的呢？答案就在下面的事件表里：

```cpp
BEGIN_EVENT_TABLE(MyFrame, wxFrame)
    EVT_MENU(wxID_ABOUT, MyFrame::OnAbout)
    EVT_MENU(wxID_EXIT,  MyFrame::OnQuit)
END_EVENT_TABLE() 
```

所谓事件表，是一组位于类的实现文件（.cpp 文件）中的宏，用来告诉 wxWidgets 来自用户或者其它地方的事件应该怎样和类的成员函数对应起来。

前面展示的事件表表明，要把鼠标点击标识分别为 wxID_EXIT 和 wxID_ABOUT 的菜单项的事件和 MyFrame 的成员函数 OnAbout 和 OnQuit 关联起来。这里的 EVT_MENU 宏只是很多中事件宏的其中之一，事件宏的作用是告诉 wxWidgets 哪种事件应该被关联到哪个成员函数。这里的两个标识 wxID_ABOUT 和 wxID_EXIT 是 wxWidgets 预定义的宏，通常你应该通过枚举，常量或者预编译宏的方式定义你自己的标识。

用上面的方法定义的时间表是一种静态的事件表，它不可以在运行期改变，在下一章里，我们将描述怎样定义可以在运行期改变的动态事件表。

现在，让我们来看看这两个事件处理函数.

```cpp
void MyFrame::OnAbout(wxCommandEvent& event)
{
    wxString msg;
    msg.Printf(wxT("Hello and welcome to %s"),
               wxVERSION_STRING);
    wxMessageBox(msg, wxT("About Minimal"),
                 wxOK | wxICON_INFORMATION, this);
}

void MyFrame::OnQuit(wxCommandEvent& event)
{
    Close();
} 
```

当用户点击关于菜单项的时候，MyFrame::OnAbout 函数弹出一个消息框。这用到了 wxWidgets 提供的 API wxMessageBox,它的四个参数分别代表消息内容，标题，窗口类型以及父窗口。

当用户点击退出菜单项的时候，MyFrame::OnQuit 函数被调用（你已经意识到了，这是事件表的功劳）。它调用 wxFrame 类的 Close 函数来释放 frame 窗口。因为没有别的窗口存在了，这触发了应用程序的退出，实际上，wxFrame 类的 Close 函数并不直接关闭 frame 窗口，而是产生一个 wxEVT_CLOSE_WINDOW 事件，这个事件默认的处理函数调用 wxWindow::Destroy 函数释放了 frame 窗口。

用户还可以通过别的方法关掉应用程序，比如通过点击标题栏上的关闭按钮或者是通过系统菜单中的关闭菜单，在这种情况下，OnQuit 函数是怎样被调用的呢？事实上，在这种情况，OnQuit 函数并没有被调用。这时，wxWidgets 会通过 Close 函数（象 OnQuit 中的那样），给 frame 窗口发送一个 wxEVT_CLOSE_WINDOW 事件，这个事件默认的处理函数会释放掉 frame 窗口。在你的应用程序中，可以通过重载这个处理函数来增加改变这种默认的行为，比如：如果你想问一问你的用户是不是真的要关闭窗口。关于这种处理的细节，可以参见第四章,"窗口基础"。

另外，大多数的应用程序类还应该重载一个 OnExit 函数，以便在任何时候程序退出时，执行一下清理和资源回收的动作。需要注意的是，这个函数只有在 OnInit 函数返回真的时候才会被执行。当然，我们这个小例子程序就用不着定义这个函数了。

# 2.5 Frame 窗口的构造函数

最后，让我们来看看 Frame 窗口的构造函数，正是它实现了 frame 窗口的图标，菜单条和状态条。

```cpp
#include "mondrian.xpm"

MyFrame::MyFrame(const wxString& title)
       : wxFrame(NULL, wxID_ANY, title)
{
    SetIcon(wxIcon(mondrian_xpm));
    wxMenu *fileMenu = new wxMenu;
    wxMenu *helpMenu = new wxMenu;
    helpMenu->Append(wxID_ABOUT, wxT("&About...\tF1"),
                     wxT("Show about dialog"));
    fileMenu->Append(wxID_EXIT, wxT("E&xit\tAlt-X"),
                     wxT("Quit this program"));
    wxMenuBar *menuBar = new wxMenuBar();
    menuBar->Append(fileMenu, wxT("&File"));
    menuBar->Append(helpMenu, wxT("&Help"));
    SetMenuBar(menuBar);
    CreateStatusBar(2);
    SetStatusText(wxT("Welcome to wxWidgets!"));
} 
```

这个构造函数首先调用它的基类（wxFrame）的构造函数，使用的参数是父窗口(还没有父窗口，所以用 NULL),窗口标识(wxID_ANY 标识让 wxWidgets 自己选择一个)和标题。这个基类的构造函数才真正创建了一个窗口的实例。除了这样的调用方法，还有另外一种方法是直接在构造函数里面显式调用基类默认的构造函数，然后调用 wxFrame::Create 函数来创建一个 frame 窗口的实例。

小图片或者是图标在所有的平台上都可以用 XPM 格式来表示。XPM 文件其实是一个 ASCII 编码的完全符合 C++语法的文本文件，所以可以直接用 C++的方式包含到代码中（译者注：显然这样的包含方式在分发软件的时候是不需要分发这个图片文件的）。SetIcon 那一行代码使用 mondrian_xpm 变量在堆栈上创建了一个图标（这个 mondrian 变量是在 mondrian.xpm 文件里定义的）。然后将这个图标和 frame 窗口关联在一起。

接下来创建了菜单条。增加菜单项的 Append 函数的三个参数的意义分别为：菜单项标识，菜单上的文本以及一个稍微长一些的帮助字符串。这个帮助字符串会自动在菜单项被高亮显示的时候自动显示在状态栏上。菜单上的文本中由"&"符号前导的字符将成为菜单的快捷操作符，在实际的显示中用下划线表示。而"\t"符号则前导一个全局的快捷键，这个快捷键甚至可以在菜单项没有显示的时候触发菜单功能。

这个构造函数所做的最后一件事是创建一个由两个区域组成的状态条并且在状态条的第一个区域写上欢迎的字样。

# 2.6 完整的例子

现在是时候把所有的代码放在一起了，通常，我们应该把头文件和实现文件分开，但是对于这样小的一个程序，就没有这个必要了。

```cpp
// Name:        minimal.cpp
// Purpose:     Minimal wxWidgets sample
// Author:      Julian Smart

#include "wx/wx.h"

// 定义应用程序类
class MyApp : public wxApp
{
public:
    // 这个函数将会在程序启动的时候被调用
    virtual bool OnInit();
};

// 定义主窗口类
class MyFrame : public wxFrame
{
public:
    // 主窗口类的构造函数
    MyFrame(const wxString& title);

    // 事件处理函数
    void OnQuit(wxCommandEvent& event);
    void OnAbout(wxCommandEvent& event);

private:
    // 声明事件表
    DECLARE_EVENT_TABLE()
};

// 有了这一行就可以使用 MyApp& wxGetApp()了
DECLARE_APP(MyApp)

// 告诉 wxWidgets 主应用程序是哪个类
IMPLEMENT_APP(MyApp)

// 初始化程序
bool MyApp::OnInit()
{
    // 创建主窗口
    MyFrame *frame = new MyFrame(wxT("Minimal wxWidgets App"));

    // 显示主窗口
    frame->Show(true);

    // 开始事件处理循环
    return true;
}

// MyFrame 类的事件表
BEGIN_EVENT_TABLE(MyFrame, wxFrame)
    EVT_MENU(wxID_ABOUT, MyFrame::OnAbout)
    EVT_MENU(wxID_EXIT,  MyFrame::OnQuit)
END_EVENT_TABLE()

void MyFrame::OnAbout(wxCommandEvent& event)
{
    wxString msg;
    msg.Printf(wxT("Hello and welcome to %s"),
               wxVERSION_STRING);

    wxMessageBox(msg, wxT("About Minimal"),
                 wxOK | wxICON_INFORMATION, this);
}

void MyFrame::OnQuit(wxCommandEvent& event)
{
    // 释放主窗口
    Close();
}

#include "mondrian.xpm"

MyFrame::MyFrame(const wxString& title)
       : wxFrame(NULL, wxID_ANY, title)
{
    // 设置窗口图标
    SetIcon(wxIcon(mondrian_xpm));

    // 创建菜单条
    wxMenu *fileMenu = new wxMenu;

    // 添加“关于”菜单项
    wxMenu *helpMenu = new wxMenu;
    helpMenu->Append(wxID_ABOUT, wxT("&About...\tF1"),
                     wxT("Show about dialog"));

    fileMenu->Append(wxID_EXIT, wxT("E&xit\tAlt-X"),
                     wxT("Quit this program"));

    // 将菜单项添加到菜单条中
    wxMenuBar *menuBar = new wxMenuBar();
    menuBar->Append(fileMenu, wxT("&File"));
    menuBar->Append(helpMenu, wxT("&Help"));

    // ...然后将菜单条放置在主窗口上
    SetMenuBar(menuBar);

    // 创建一个状态条来让一切更有趣些。
    CreateStatusBar(2);
    SetStatusText(wxT("Welcome to wxWidgets!"));
} 
```

# 2.7 wxWidgets 程序一般执行过程

下面大概的描述一下整个程序的执行过程：

1.  依照系统平台的不同，不同的 main 函数或者 winmain 函数或者其它类似的函数被调用(这个函数是由 wxWidgets 提供的,而不是由应用程序提供的).wxWidgets 初始化它自己的数据结构并且创建一个 MyApp 的实例.
2.  wxWidgets 调用 MyApp::OnInit 函数, 这个函数会创建一个 MyFrame 的实例.
3.  MyFrame 的构造函数通过它的基类 wxFrame 的构造函数创建一个窗口，然后给这个窗口增加图标，菜单栏和状态栏.
4.  MyApp::OnInit 函数显示主窗口并且返回真.
5.  wxWidgets 开始事件循环，等待事件发生并且将事件分发给相应的处理过程.

就目前我们所知道的，应用程序会在以下情况下退出：主窗口被关闭，用户选择退出菜单或者系统按钮和系统菜单中的关闭选项（这些系统菜单和系统按钮在不同的系统中就往往千差万别了）。

# 2.8 编译和运行程序

你可以在附带光盘的 examples/chap02 里找到这个例子。你可能需要把它拷贝到你硬盘上的某个目录中才能进行编译。因为要提供适合所有读者的软件环境的 Makefile 几乎是不可能的，我们提供了一个 DialogBlocks 的项目文件，并且把我们能想到的大多数系统平台进行了配置。你可以参考附录 C，“使用 DialogBlocks 创建应用程序”来找到适合你自己的编译器的编译方法。在附录 B 中，我们也大概的描述了直接用 wxWidgets 编译的方法。

你可以从 CDRom 上安装 wxWidgets 和 DialogBlocks，如果你使用的是 windows 操作系统并且你的系统中还没有可以用的编译器，你还需要安装一个编译器。然后运行 DialogBlocks，并且在其设置叶面设置你的 wxWidgets 和编译器的路径，然后打开 examples/chap02/minimal.pj，选择适合你的编译器和编译平台，比如 Mingw Debug 或者 VC++ Debug(Windows),GCC Debug GTK+(Linux),或者 GCC DEBUG Max(Max Os X)等，然后点击绿色的编译和运行工程按钮。如果你的 wxWidgets 库还没有被编译，它将首先编译 wxWidgets 库。

你也可以在 wxWidgets 的 sample/minimal 目录中找到一个类似的例子。如果你不想使用 DialogBlocks 编译，你也可以试着编译这个例子。你可以在附录 A“安装 wxWidgets”中找到编译这个例子的方法，

# 第二章小结

本章向你展示了一个非常简单的 wxWidgets 小程序是怎样工作的。我们已经接触到了 wxFrame 类，事件处理，应用程序初始化，并且创建了一个菜单条和状态条。虽然代码看上去有些复杂，但是任何其它更复杂的程序设计和我们展示在这个小例子中的一些公共的原则都是相同的。在下一章里，我们会更近距离的看一看事件表机制以及我们的应用程序是怎样处理事件表的。