# 第四章 窗口的基础知识

# 第四章 窗口的基础知识

在这一章中，在详细描述应用程序中大量使用的各种窗口类之前，我们首先来看看一个窗口的主要元素。这些要素中的大部分相信你以前在各种平台上的编程经验或者使用经验足以让你对它们非常的熟悉。虽然总会有一些平台相关的差异，但是你会发现，单就功能上来说，各个平台之间有着惊人的相似，而 wxWidgets 已经屏蔽了几乎所有这些平台上的差异，还剩下的一小部分差异，主要体现在各个平台可选的窗口类型上的不同。

# 4.1 窗口解析

# 4.1 窗口解析

你当然大略的知道一个窗口指的是什么，但是为了更好的理解 wxWidgets 的 API,你应该更精通 wxWidgets 使用的窗口模型的细节。它可能和你在某个特定平台上的窗口概念有些许的不同。下图演示了一个窗口中的各个基本元素：

![](img/mht9FB6%281%29.tmp)

窗口的概念

一个窗口指的是屏幕上的任何一个拥有以下特征的规则区域：它可以被改变大小，可以自我刷新，可以被显示和隐藏等等。它可以包含别的窗口(比如 frame 窗口就可以包含菜单条窗口，工具条窗口以及状态条窗口)，也可以子窗口(比如一个静态的文本或者一副静态图片)。通常你在使用 wxWidgets 编写的程序运行的屏幕上看到的窗口，都和一个 wxWindow 类或者它的派生类对应，当然也不总是这样。比如本地原生下拉框就不总是用 wxWindow 建模的。

客户区和非客户区

当我们谈到窗口的大小，我们通常指的是它整个的大小，包括一些用于修饰的边框和标题栏等。而当我们谈到一个窗口的客户区大小，通常都只意味着窗口里面那些能被绘制或者它的子窗口能被放置的位置的大小。例如一个 frame 窗口的客户区大小就不包括那些菜单栏，状态栏和工具栏所占用的地方。

滚动条

大多数窗口都有显示滚动条的能力，这些滚动条通常是窗口自己增加的而不是有应用程序手动增加的。在这种情况下，客户区的大小还应该减去滚动条所占用的空间。为了优化性能，只有那些拥有 wxHSCROLL 和 wxVSCROLL 类型的窗口才会自动生成它们自己的滚动条。关于滚动条更多的情形我们会在本章稍候讨论 wxScrolledWindow 的时候讨论。

光标和鼠标指针

一个窗口可以拥有一个光标(wxCaret,用来显示当前的文本位置)和一个鼠标指针(wxCursor,用来显示当前鼠标指针的位置).当鼠标移入某个窗口时，wxWidgets 会自动显示这个窗口的鼠标指针。当一个窗口变为当前焦点窗口时，如果可以的话，光标将会显示在当前文本的插入位置，或者如果这个焦点是由于鼠标点击造成的，光标将会显示在鼠标对应的位置。

顶层窗口

窗口通常分为象 wxFrame,wxDialog,wxPopup 这样的顶层窗口和其它窗口。只有顶层窗口创建的时候可以使用 NULL 作为其父窗口，也指有顶层窗口是延迟删除的(所谓延迟删除的意思是说，它们只有在系统空闲的时候才会被删除，也就是说只有当所有的事件都被处理完以后才会被删除)。而且出了 Popup 窗口以外，顶层窗口通常都有一个标题栏和一个关闭按钮，只要应用程序允许，它们就可以被拽着满屏幕的跑或者被改变大小。

座标体系

窗口的座标体系通常是左上角为原点(0,0),单位是象素。在某个用于窗口绘制的特定的上下文中，原点和比例允许被改变。这方面详细的可以参考第五章,"窗口绘制和打印".

窗口绘制

当一个窗口需要重绘的时候，它将收到两个事件，wxEVT_ERASE_BACKGROUND 事件用于通知应用程序重新绘制背景， wxEVT_PAINT 则用于通知重新绘制前景。那些已经准备好使用的窗口空间比如 wxButton(按钮)通常知道怎么处理这两个事件，但是如果你是要创建自己的窗口控件，你就需要自己处理这两个事件了。通过获取窗口的变动区域你可以优化你的绘制代码。

颜色和字体

每一个窗口都有一个前景色和一个背景色。默认的背景擦除函数会使用背景色来清除窗口背景，如果没有设置背景色，则会使用当前的系统皮肤推荐的颜色进行背景的清除。前景色则相对来说很少被用到。每一个窗口也拥有一个字体设置，是否用到这个字体设置要取决于这个窗口本身的类型。

窗口变体

在 Mac OS X 上，一个窗口有一个窗口变体的概念，通过这个概念窗口可以被以不同级别的大小显示：wxWINDOW_VARIANT_NORMAL(默认显示级别), wxWINDOW_VARIANT_SMALL, wxWINDOW_VARIANT_MINI, 或者 wxWINDOW_VARIANT_LARGE.当你有很多信息要展示而屏幕空间不够的时候，你可以使用相对较小的级别，但是这个显示级别的使用应该适可而止。有些程序总是喜欢使用小的显示级别。

改变大小

当一个窗口的大小，无论是来自用户还是应用程序本身的原因，发生变化时，它将收到一个 wxEVT_SIZE 事件。如果这个窗口拥有子窗口，它们可能需要被重新放置和重新计算大小。处理这种情况推荐的方法是使用 sizer 类，关于这个类的细节将在第七章，"使用 Sizer 确定窗口的布局" 这一章详细介绍。大多数已经确定的窗口类都有一个默认的大小和位置，这需要你在创建这些窗口的时候使用 wxDefaultSize 和 wxDefaultPosition 这两个特殊的值。到目前为止，每一个控件都实现了 DoGetBestSize 函数，这个函数会返回一个基于控件的内容，当前字体以及其它各方面来说最合理的大小。

输入

任何窗口在任何时候都可以接收到鼠标事件，除非某个窗口已经临时捕获了鼠标或者这个窗口已经被禁止使用了，而对于键盘事件来说，只有当前处于活动状态的窗口才可以收到。应用程序自己可以设置自己为活动状态，wxWidgets 也会在用户点击某个窗口的时候将其设置为活动状态。正变成活动状态的窗口会收到 wxEVT_SET_FOCUS 事件，而正失去焦点的窗口会收到 wxEVT_KILL_FOCUS 事件。请参考第六章：处理输入。

空闲事件处理和用户界面更新

所有的窗口(除非特殊声明)都将收到空闲事件 wxEVT_IDLE,这个事件是在所有其它的事件都已经被处理完以后发出的。使用 EVT_IDLE 事件映射宏来处理。更多的信息请参卡第十七章,"多线程编程"中的"空闲时间处理"小节.

其中一个特殊的空闲时间操作就是进行用户界面更新，在这个操作中所有的窗口都可以定义一个函数来更新自己的状态。这个函数将会被周期性的在系统空闲时调用。而 EVT_UPDATE_UI(id, func)这个宏则通常不需要作什么事情。更多关于用户界面更新的细节请参考第九章,"创建自定义对话框".

窗口的创建和删除

一般来说，窗口都是在堆上使用 new 方法创建的，第十五章，"内存管理，调试和错误检查"这一章对此有详细的描述，也会提到一些例外情况。大多数的窗口类都可以通过两种方法被创建：单步创建和两步创建。比如 wxButton 的两种构造函数如下:

```cpp
wxButton();
wxButton(wxWindow* parent,
    wxWindowID id,
    const wxString& label = wxEmptyString,
    const wxPoint& pos = wxDefaultPosition,
    const wxSize& size = wxDefaultSize,
    long style = 0,
    const wxValidator& validator = wxDefaultValidator,
    const wxString& name = wxT("button")); 
```

下面演示了使用一步创建的方法创建一个 wxButton 的实例(其中多数参数采用默认值)：

```cpp
wxButton* button  = new wxButton(parent, wxID_OK); 
```

除非是 frame 或者 dialog 窗口，对于别的窗口，都必须在构造函数中传入一个非空的父窗口。这会自动把这个新窗口作为这个父窗口的子窗口。当父窗口被释放的时候，它的所有的子窗口也将被释放。正象我们前面提到的那样，你必须输入一个自定义的或者系统内建的标识符给这个窗口以便唯一标识这个窗口。你也可以使用 wxID_ANY 宏让 wxWidgets 帮你生成一个。你可以传递位置和大小参数给这个窗口，一个校验类(参考第九章)，一个类型 (接下来会提到)，和一个字符串的名字。这个字符串的名字可以是任意的值或者干脆不填也可以。只有在 Xt 和 Motif 系统上这个参数才有意义，因为在这些系统上控件是通过它们的名字来区分的,平常情况下则很少用到。

两步创建的意思是说，你先使用默认的构造函数创建一个实例，然后再使用这个实例 Create 方法实际创建这个对象。Create 的参数和前面使用的构造函数的参数完全相同,还是用 wxButton 作为例子:

```cpp
wxButton* button  = new wxButton;
button->Create(parent, wxID_OK); 
```

窗口在你调用 Create 函数的时候会收到 wxEVT_CREATE 事件，你可以对这个事件进行进一步的处理。

使用两步创建的原因是什么呢？第一个原因是有时侯你可能想在晚些时候，在真正需要的时候才完整的创建窗口。另外一个原因是你希望在调用 Create 函数之前设置窗口的某些属性值。尤其是这些属性值被 Create 函数使用的时候就显的更有意义。例如你可能想在窗口被创建之前设置 wxWS_EX_VALIDATE_RECURSIVELY 扩展类型，而这个类型必须通过 SetExtraStyle 函数才可以设置。在这种情况下，对某些对话框类而言，validation 必须在 Create 函数被调用之前被初始化。所以如果你需要这个功能，就必须在调用 Create 之前初始化这个值。

当你创建一个窗口类，或者其它任何非顶层窗口的派生类的时候，如果它的父窗口是可见的，那么它也总是可见的，你可以通过 Show(false)来使它不可见。这和 wxDialog 或者 wxFrame 这样的顶层窗口是不一样的。顶层窗口在创建时通常是不可见的，这样可以避免绘制那些子窗口和排列子控件的时候发生闪烁。你需要通过 Show 或者(对于模式对话框来说)ShowModal 的调用让它可见。

窗口是通过调用其 Destroy 函数(对于顶层窗口来说)或者 delete 函数(对于其子窗口来说)来释放的。wxEVT_DESTROY 事件会在窗口刚刚要被释放之前被调用。实际上，子窗口是被自动释放的，所以 delete 函数是很少直接被手动调用的。

窗口类型

窗口拥有一个类型和一个扩展类型。窗口类型是设置窗口创建时的行为和外观的一种简洁的方法。这些类型的值被设置成可以使用类似比特位的方法操作，例如下面的例子:

```cpp
wxCAPTION | wxMINIMIZE_BOX | wxMAXIMIZE_BOX | wxTHICK_FRAME 
```

wxWindow 类有一组基本的类型值，例如边框的类型等，每一个派生类可以增加它们自己的类型。需要特别指出的是，扩展类型的值是不可以拿来给类型用的。

# 4.2 窗口类概览

# 4.2 窗口类概览

在接下来的章节中，我们会介绍最常用的那些窗口类以便你可以在你的应用程序中使用它们。然而如果你是第一次阅读这本书，你可以直接跳到第五章阅读后面的内容，而在晚些时候你需要使用的时候再回过头来阅读本章的内容。

为了让你先大致浏览一下本章的内容，我们把本章将会讨论的窗口类列举如下。对于其他一些窗口类，请参考第十二章,"高级窗口类"以及附录 E,"wxWidgets 中的第三方工具".

基本窗口类

下面的这些基本的窗口类实现了一些最基本的功能。这些类主要是用来作为别的类型的基类以生成更实用的派生类。

*   wxWindow. 这是所有窗口类的基类。
*   wxControl. 所有控件(比如 wxButton)的基类.
*   wxControlWithItems. 是那些拥有多个子项目的控件的基类.

顶层窗口类

顶层窗口类通常指那些独立的位于桌面上的类。

*   wxFrame. 一个可以包含其他窗口，并且大小可变的窗口类。
*   wxMDIParentFrame. 是一个可以管理其他 Frame 类的类.
*   wxMDIChildFrame. 是一个可以被其父窗口管理的 frame 类.
*   wxDialog. 是一种可变大小的用于给用户提供选项的窗口类.
*   wxPopupWindow. 是一种暂态的只有很少修饰的顶层窗口.

容器窗口类

容器窗口类可以管理其他窗口

*   wxPanel. 这是一个给其它窗口提供布局的窗口.
*   wxNotebook. 可以实用 TAB 页面进行切换的窗口.
*   wxScrolledWindow. 可以有滚动条的窗口.
*   wxSplitterWindow. 可以管理两个子窗口的一种特殊窗口类.

非静态控件窗口类

这些控件是用户可以操作或者编辑的。

*   wxButton. 一种拥有一个标签的按钮控件.
*   wxBitmapButton. 一种拥有图片和标签的按钮控件.
*   wxChoice. 拥有一个下拉列表的选择控件.
*   wxComboBox. 拥有一组选项的可编辑的选择控件.
*   wxCheckBox. 拥有一个复选框的控件，复选框有选中和未选中两种状态.
*   wxListBox. 拥有一组可选择的字符串项目的列表框.
*   wxRadioBox. 拥有一组选项的单选框.
*   wxRadioButton. 单选框.
*   wxScrollBar. 滚动条控件。
*   wxSpinButton. 一个拥有增加和减小两个选项的按钮.
*   wxSpinCtrl. 拥有一个文本编辑框和一个 wxSpinButton 用来编辑整数.
*   wxSlider. 这个控件用来在一个固定的范围内选择一个整数.
*   wxTextCtrl. 单行或者多行的文本编辑框.
*   wxToggleButton. 两态按钮.

静态控件

这些控件提供不能被最终用户编辑的静态信息

*   wxGauge. 用来显式数量的控件.
*   wxStaticText. 文字标签控件.
*   wxStaticBitmap. 用来显示一幅静态图片.
*   wxStaticLine. 用来显式静态的一行.
*   wxStaticBox. 用来在别的控件周围显示一个静态的方框.

菜单

菜单是一种包含一组命令列表的窗口

控件条

控件条通常在 Frame 窗口中使用，用来为信息或者命令的访问提供快捷操作

*   wxMenuBar. wxFrame 上的菜单条.
*   wxToolBar. 工具条.
*   wxStatusBar. 状态条用来在程序运行过程中显示运行期信息.

# 4.3 基础窗口类

# 4.3 基础窗口类

虽然你不一定有机会直接使用基础窗口类(wxWindow)，但是由于这个类是很多窗口控件的基类，它实现的很多方法在它的子类型中都可以直接拿来使用，所以有必要介绍一下这个基础窗口类。

窗口类 wxWindow

wxWindow 窗口类既是一个重要的基类，也是一个你可以直接在代码中使用的类。当然，前者使用的频度要比后者大很多。

和前面提到的一样，wxWindow 也可以使用单步创建或者两步创建两种方式。单步创建的构造函数原型如下：

```cpp
wxWindow(wxWindow* parent,
    wxWindowID id,
    const wxPoint& pos = wxDefaultPosition,
    const wxSize& size = wxDefaultSize,
    long style = 0,
    const wxString& name = wxT("panel")); 
```

可以象下面这样使用:

```cpp
wxWindow* win  = new wxWindow(parent, wxID_ANY,
    wxPoint(100, 100), wxSize(200, 200)); 
```

窗口类型

每一个窗口类都可以使用定义在下表中的这些窗口基类中的窗口类型。这些类型中不是所有的类些都被所有的控件所支持。例如对于边框的类型。如果在创建窗口的时候你没有指定窗口的边框类型，那么在不同的平台上将会有不同的边框类型的缺省值。在 windows 平台上，控件边框的缺省值为 wxSUNKEN_BORDER,意为使用当前系统风格的边框。你可以使用类似 wxNO_BORDER 这样的值来覆盖系统的默认值。

| wxSIMPLE_BORDER | 在窗口周围显示一个瘦边框. |
| --- | --- |
| wxDOUBLE_BORDER | 显示一个双层边框. |
| wxSUNKEN_BORDER | 显示一个凹陷的边框，或者使用当前窗口风格设置. |
| wxRAISED_BORDER | 显示一个凸起的边框. |
| wxSTATIC_BORDER | 显示一个适合静态控件的边框. 只支持 Windows 平台. |
| wxNO_BORDER | 不显示任何边框. |
| wxTRANSPARENT_WINDOW | 定义一个透明窗口 (意思是这个窗口不接收 paint 事件).只支持 windows 平台. |
| wxTAB_TRAVERSAL | 使用这个类型允许非 Dialog 窗口支持使用 TAB 进行遍历. |
| wxWANTS_CHARS | 使用这个类型来允许窗口接收包括回车和 TAB 在内的所有的键盘事件。TAB 用来在 Dialog 类型的窗口中遍历各控件。如果没有设置这个类型，这些特殊的按键事件将不会被产生。 |
| wxFULL_REPAINT_ON_RESIZE | 在默认情况下，在窗口客户区大小发生改变时，wxWidgets 并不会重画整个客户区。设置这个类型将使得 wxWidgets 改变这种默认的作法，而保持整个客户区的刷新 |
| wxVSCROLL | 显示垂直滚动条. |
| wxHSCROLL | 显示水平滚动条. |
| wxALWAYS_SHOW_SB | 如果一个窗口有滚动条，那么在不需要滚动条的时候（当窗口足够大不需要使用滚动条的时候），禁止滚条而不隐藏滚动条。这个类型目前只支持 Windows 平台和 wxWidgets 的 wxUniversal 版本. |
| wxCLIP_CHILDREN | 只支持 Windows 平台,用于消除由于擦除子窗口的背景而引起的闪铄. |

下表列出了窗口的扩展类型，这些扩展类型不能直接和类型混用，而要使用 wxWindow::SetExtraStyle 函数来进行设置。

| wxWS_EX_VALIDATE_RECURSIVELY | 在默认情况下，Validate,transferDataToWindow,和 transferDataFromWindow 只在窗口的直接子窗口上才可以使用。如果设置了这个扩展类型，那么将可以递归的在各个子窗口上使用。 |
| --- | --- |
| wxWS_EX_BLOCK_EVENTS | wxCommandEvents 事件将会在无法在当前事件表中找到匹配的时候在其父窗口中尝试匹配，设置这个扩展属性可以阻止这个行为。Dialog 类型的窗口默认设置了这个类型，但是如果 SetExtraStyle 被应用程序类调用过的话，默认设置可能被覆盖. |
| wxWS_EX_TRANSIENT | 不要使用这个窗口作为其它窗口的父窗口.这个类型一定只能用于瞬间窗口;否则，如果使用它作为一个 dialog 或者 frame 类型窗口的父窗口，如果父窗口在子窗口之前释放，可能导致系统崩溃。 |
| wxWS_EX_PROCESS_IDLE | 这个窗口应该处理所有的 idle 事件，包括那些设置了 wxIDLE_PROCESS_SPECIFIED 模式的 idle 事件。 |
| wxWS_EX_PROCESS_UI_UPDATES | 这个窗口将处理所有的 Ui 刷新事件，包括那些设置了 wxUPDATE_UI_PROCESS_SPECIFIED 的 UI 刷新事件。参考第九章获得和界面刷新有关的更多的内容. |

窗口事件

窗口类和它的派生类可以产生下面的事件（鼠标，键盘和游戏手柄产生的事件将会在第六章描述）。

| EVT_WINDOW_CREATE(func) | 用于处理 wxEVT_CREATE 事件, 这个事件在窗口刚刚被产生的时候生成，处理函数的参数类型是 wxWindowCreateEvent. |
| --- | --- |
| EVT_WINDOW_DESTROY(func) | 用于处理 wxEVT_DELETE 事件,在这个窗口即将被删除的时候产生，处理函数的参数类型是 wxWindowDestroyEvent. |
| EVT_PAINT(func) | 用于处理 wxEVT_PAINT 事件,在窗口需要被刷新的时候产生.处理函数的参数类型是 wxPaintEvent. |
| EVT_ERASE_BACKGROUND(func) | 用于处理 wxEVT_ERASE_BACKGROUND 事件,在窗口背景需要被更新的时候产生. 处理函数的参数是 wxEraseEvent. |
| EVT_MOVE(func) | 用于处理 wxEVT_MOVE 事件, 在窗口移动的时候产生.处理函数的参数类型是 wxMoveEvent. |
| EVT_SIZE(func) | 用于处理 wxEVT_SIZE 事件, 在窗口大小发生变化以后产生.处理函数的参数类型是 wxSizeEvent. |
| EVT_SET_FOCUS(func)EVT_KILL_FOCUS(func) | 用于处理 wxEVT_SET_FOCUS 和 wxEVT_KILL_FOCUS 事件,在窗口得到或者失去键盘焦点的时候产生. 处理函数参数类型是 wxFocusEvent. |
| EVT_SYS_COLOUR_CHANGED(func) | 用于处理 wxEVT_SYS_COLOUR_CHANGED 事件,当用户在控制面板里更改了系统颜色的时候产生(只支持 windows 平台). 处理函数参数类型为 wxSysColourChangedEvent. |
| EVT_IDLE(func) | 用于处理 wxEVT_IDLE 事件,在空闲事件产生.处理函数参数类型位 wxIdleEvent. |
| EVT_UPDATE_UI(func) | 用于处理 wxEVT_UPDATE_UI 事件,在系统空闲时间产生用来给窗口一个更新自己的机会. |

wxWindow 类的成员函数

因为 wxWindow 类是其它所有窗口类的基类，它拥有很多的成员函数。我们不可能在这里作一一的说明，只能拣其中最重要的一些作简要的说明。不过我们还是推荐你浏览一下 wxWidgets 手册中的相关部分，以便能够彻底了解 wxWindow 类提供的所有功能，以及要使用这个功能你需要提供的参数等。

CaptureMouse 函数可以捕获所有的鼠标输入（将其限制在本窗口以内）,ReleaseMouse 则可以释放前一次的捕获.在绘图程序中，这两个函数是很有用的。它可以让你在鼠标移动到窗口边缘的时候来滚动窗口的客户区，而不是任由鼠标跑到别的窗口并且激活别的窗口。另外的两个函数 GetCapture 用来获取当前正在使用的捕获设备(如果是被当前的应用程序设置的话),而 HasCapture 可以用来检测是否鼠标正被这个窗口捕获。

Centre, CentreOnParent 和 CentreOnScreen 三个函数可以让窗口调整自己的位置使其位于屏幕或者是其父窗口的正中间位置。

ClearBackground 函数将使用当前的背景色清除当前窗口.

ClientToScreen 和 ScreenToClient 可以将座标在相对于屏幕左上角和相对于客户区左上角之间进行互相转换.

Close 函数产生一个 wxCloseEvent 事件，这个事件通常的处理过程将关闭窗口，释放内存。当然如果应用程序为这个事件定义了特殊的处理函数，那么窗口也有可能不被关闭和释放。

ConvertDialogToPixels 和 ConvertPixelsToDialog 函数可以对数值进行对话框单位和象素单位之间的转换。这在基于字体大小以便应用程序的显示更合理的操作中是很有用的.

Destroy 函数将安全的释放窗口.使用这个函数代替直接调用 delete 操作符因为这个函数下不同的窗口类型的表现是不一样的。对于对话框和 frame 这样的顶层窗口来说，这个函数会将窗口放入一个等待删除的额窗口列表中，等到这个窗口的所有的事件都处理完毕的时候才会被删除，这可以避免一些事件在已经不存在的窗口上被执行.

Enable 允许或者禁止窗口和它的子窗口处理输入事件。在禁止状态下一些窗口会有不同的颜色和外观。Disable 函数和 Enable 函数使用 false 作为参数的效果是完全一样的。

FindFocus 函数是一个静态函数，用它可以找到当前拥有键盘焦点的窗口。

FindWindow 函数可以通过标识符或者名字在它的窗口关系树中寻找某个窗口. 返回值可能是它的一个子窗口或者它自己.如果你可以确定你要找的窗口的类型，可以安全的使用 wxDynamicCast 进行类型强制转换，转换的结果将是一个指向那个类型的指针或者是 NULL:

```cpp
MyWindow* window = wxDynamicCast(FindWindow(ID_MYWINDOW), MyWindow); 
```

Fit 函数会自动改变窗口的大小以便刚好可以容纳它的所有子窗口.这个函数应用被应用在使用基于 sizer 的布局的时候。FitInside 函数是一个类似的函数，区别在于它使用的是虚大小(应用在那些包含滚动条的窗口).

Freeze 和 Thaw，这两个函数的作用是告诉 wxWidgets，在这两个函数之间进行的刷新界面的操作是允许被优化的。举例来说，如果你要在一个文本框控件逐行中加入多行文本，则可以用这个方法来优化显示效果，避免闪烁，这两个函数已经在 GTK+版本的 wxTextCtrl 控件上实现，也适用于 Windows 和 Max Os X 平台的所有类.

GetAcceleratorTable 和 SetAcceleratorTable 用来获取和设置某个窗口的加速键表.

GetBackgroundColour 和 SetBackgroundColour 用来访问窗口的背景颜色属性。这个属性被 wxEVT_ERASE_BACKGROUND 事件用来绘制窗口背景.如果你更改了背景颜色设置，你应该调用 Refresh 或者 ClearBackground 函数来刷新背景. SetOwnBackgroundColour 的作用和 SetBackgroundColour 基本相同，但是前者不会更改当前窗口的子窗口的背景属性.

GetBackgroundStyle 和 SetBackgroundStyle 用来设置窗口的背景类型. 默认的窗口背景类型是 wxBG_STYLE_SYSTEM,它的含义是 wxWidgets 按照系统默认设置来进行背景绘制。系统默认的背景绘制方法根据控件的不同而不同，比如 wxDialog 的默认背景绘制方法是什么纹理绘制的方法，而 wxListBox 则是使用固定颜色的绘制方法。如果你设置了窗口的背景绘制方法类型是 wxBG_STYLE_COLOUR,那么 wxWidgets 会全部用单一颜色的方法绘制背景。而如果你将其设置为 wxBG_STYLE_CUSTOM, wxWidgets 将不进行任何的背景绘制工作，你可以自己在擦除背景事件中或者重画事件中自己绘制背景.如果你希望自己绘制背景，请一定设置 wxBG_STYLE_CUSTOM 为背景类型，否则很容易引起画面的闪烁.

GetBestSize 函数以象素为单位返回窗口最合适的大小(因为每个窗口类都需要实现 DoGetBestSize 函数).这个函数用来提示 Sizer 系统不要让这个窗口的尺寸太小以致不能正确的显示和使用。举例来说，对于静态文本控件来说，不要让它的字符只能显示一半。对于包含其它子窗口的窗口比如 wxPanel 来说，这个尺寸等于对这个窗口调用 Fit 函数以后的尺寸的大小。

GetCaret 和 SetCaret 函数用来访问或者设置窗口的光标.

GetClientSize 和 SetClientSize 用来访问和设置窗口的客户区大小，单位是象素. 客户区大小指的是不包括边框和修饰的区域，或者说你用户可以绘制或者用来放置子窗口的区域.

GetCursor 和 SetCursor 函数用来访问和设置窗口的鼠标指针.

GetDefaultItem 函数返回一个指向这个窗口默认的子按钮的指针或者返回 NULL。默认子按钮是当用户按回车键的时候默认激活的按钮，可以通过 wxButton::SetDefault 函数进行设置.

GetDropTarget 和 SetDropTarget 函数用来取得或者设置和窗口关联的 wxDropTarget 对象，这个对象用来处理和控制窗口的拖放操作. 我们将在第十一章"剪贴板和拖放"中详细介绍拖放有关的操作。

GetEventHandler 和 SetEventHandler 函数用来访问和设置窗口的第一事件表.默认情况下，窗口的事件表就是窗口自己，但是你可以指定不同的事件表，也可以通过 PushEventHandler 和 PopEventHandler 函数设置一个事件表链。然后让不同的事件表处理不同的事件. wxWidgets 将搜索整个事件表链来寻找匹配的事件处理函数处理新收到的事件，详情请参阅第三章，“事件处理”.

GetExtraStyle 和 SetExtraStyle 函数用来获取和设置窗口的扩展类型位。扩展类型宏通常以 wxWS*EX*开头.

GetFont 和 SetFont 函数获取和设置和窗口相关的字体. SetOwnFont 和 SetFont 的功能相似,唯一的区别在于前者设置的字体不会被子窗口继承.

GetForegroundColour 和 SetForegroundColour 函数用来操作窗口的前景颜色属性. 和 SetOwnForegroundColour 函数的区别仅在于是否改变子窗口的这个属性。

GetHelpText 和 SetHelpText 用来获取和设置窗口相关的上下文帮助.这个文本属性实际上存储在当前的 wxHelpProvider 实现中，而不是存在于窗口类中.

GetId 和 SetId 用来操作窗口标识符.

GetLabel 函数返回窗口相关联的标签.具体的含义取决的特定的窗口类.

GetName 和 SetName 用来操作窗口名称,这个名字不需要是唯一的。这个名称对 wxWidgets 来说没有什么意义,但是在 Motif 系统中被用于窗口资源的名称.

GetParent 返回窗口的父窗口.

GetPosition 以象素单位返回相对于父窗口的窗口左上角座标。

GetRect 返回一个 wxRect 对象 (参考第十三章, "数据结构和类型"),其中包含了象素单位的这个窗口的大小和位置信息.

GetSize 和 SetSize 函数获取和设置窗口象素单位的窗口长宽.

GetSizer 和 SetSizer 函数用来操作这个窗口绑定的最顶级的窗口布局对象.

GetTextExtent 用于返回当前字体下某个字符串的象素长度.

GetToolTip 和 SetToolTip 用来操作这个窗口的 tooltip 对象.

GetUpdateRegion 函数返回窗口自上次 Paint 事件以来需要刷新的区域.

GetValidator 和 SetValidator 函数用来操作这个窗口可选的 wxValidator 对象。详情请参考第九章.

GetVirtualSize 返回窗口的虚大小，通常就是和滚动条绑定的那个大小.

GetWindowStyle 和 SetWindowStyle 用来操作窗口类型比特位。

InitDialog 函数发送一个 wxEVT_INIT_DIALOG 事件来来开始对话框数据传送.

IsEnabled 用来检测当前窗口的使能状态.

IsExposed 用来检测一个点或者一个矩形区域是否位于需要刷新的范围。

IsShown 用来指示窗口是否可见.

IsTopLevel 用来指示窗口是否是顶层窗口(仅用于 wxFrame 或者 wxDialog).

Layout 函数用来在窗口已经指定一个布局对象的情况下更新窗口布局。参考第七章.

Lower 函数用来将窗口移到窗口树的最低层，而 Raise 则把窗口移动到窗口树的最顶层.这两个函数既可以用于顶层窗口，也可以用于子窗口.

MakeModal 函数禁用其它所有的顶层窗口，以便用户只能和当前这个顶层窗口交互.

Move 函数用来移动窗口.

MoveAfterInTabOrder 更改窗口的 TAB 顺序到作为参数的窗口的后面, 而 MoveBeforeInTabOrder 则将其挪到参数窗口的前面.

PushEventHandler 压入一个事件表到当前的事件表栈, PopEventHandler 函数弹出并且返回事件表栈最顶层的事件表. RemoveEventHandler 则查找事件表栈中的一个事件表并且将其出栈.

PopupMenu 函数在某个位置弹出一个菜单.

Refresh 和 RefreshRect 函数导致窗口收到一个重画事件(和一个可选的擦除背景事件).

SetFocus 函数让本窗口收到键盘焦点.

SetScrollbar 函数用来设置窗口内建的滚动条的属性.

SetSizeHints 函数用来定义窗口的最小最大尺寸，依次窗口尺寸增量的大小，仅对顶层窗口适用.

Show 函数用来显示和隐藏窗口; Hide 函数的作用相当于适用 false 作为参数调用 Show 函数.

transferDataFromWindow 和 transferDataToWindow 获取和传输数据到窗口对象，并且在没有被重载的情况下会调用验证函数.

Update 立即重画窗口已经过期的区域.

UpdateWindowUI 函数发送 wxUpdateUIEvents 事件到窗口，以便给窗口一个更新窗口元素（比如工具条和菜单）的机会.

Validate 使用当前的验证对象验证窗口数据.

wxControl 类

wxControl 是一个虚类。它继承自 wxWindow，用来作为控件的基类: 所谓控件指的是那些可以显示数据项并且通常需要响应鼠标或者键盘事件的那些窗口类.

wxControlWithItems 类

wxControlWithItems 也是一个虚类，用来作为 wxWidgets 的一些包含多个数据项的控件（比如 wxListBox, wxCheckListBox,wxChoice 和 wxComboBox 等）的基类。使用这个基类的目的为了给这些具有相似功能的控件提供一致的 API 函数。

wxControlWithItems 的数据项拥有一个字符串和一个和这个字符串绑定的可选的客户数据。客户数据可以是两种类型，要么是无类型指针(void*),这意味这这个控件只帮忙存储客户数据但是永远不会使用客户数据。另外一种是有类型（wxClientData）指针,对于后一种情况，客户数据会在控件被释放或者数据项被清除的时候被自动释放。同一个控件的所有数据项必须拥有同样的客户区数据类型：要么是前者，要么是后者。客户区数据的类型是在第一次调用 Append 函数或者，SetClientData 函数或者 SetClientObject 函数的时候被确定的。如果要使用有类型指针客户数据，你应该自定义一个继承自 wxClientData 的类，然后将它的实例指针传递给 Append 函数或者 SetClientObject 函数。

wxControlWithItems 的成员函数

Append 函数给这个控件增加一个或者一组数据项. 如果是增加一个数据项，你可以用第二个参数指定有类型客户数据指针或者无类型客户数据指针。比如:

```cpp
wxArrayString strArr;
strArr.Add(wxT("First string"));
strArr.Add(wxT("Second string"));
controlA->Append(strArr);
controlA->Append(wxT("Third string"));
controlB->Append(wxT("First string"), (void *) myPtr);
controlC->Append(wxT("First string"), new MyTypedData(1)); 
```

Clear 函数清除控件所有数据项并且清除所有的客户数据.

Delete 函数使用基于 0 的索引清除一个数据项以及它的客户数据。

FindString 返回一个和某个字符串基于 0 的数据项的索引。如果找不到则返回 wxNOT_FOUND.

GetClientData 和 GetClientObject 返回某个数据项的客户区数据指针(如果有的话). SetClientData 和 SetClientObject 则可以用来设置这个指针.

GetCount 数据项的总数.

GetSelection 返回选中的数据项或 wxNOT_FOUND. SetSelection 则用来设置某个数据项为选中状态.

GetString 用来获取某个数据项的字符串; SetString 则用来设置它.

GetStringSelection 用来返回选中的数据项的字符串或者一个空的字符串; SetStringSelection 则用来设置选中的字符串。你应该先调用 FindString 函数保证这个字符串是存在于某个数据项的，否则可能引发断言失败.

Insert 在控件数据项的某个位置插入一个数据项(可以包含也可以不包含客户数据).

IsEmpty 则用来检测一个控件的数据项是否为空.

# 4.4 顶层窗口

# 4.4 顶层窗口

顶层窗口直接被放置在桌面上而不是包含在其它窗口之内。如果应用程序允许，他们可以被移动或者重新改变大小。总共有三种基础的顶层窗口类型。 wxFrame 和 wxDialog 都是从一个虚类 wxTopLevelWindow 继承来的。另外一个 wxPopupWindow 功能相对简单，是直接从 wxWindow 继承过来的。一个 dialog 既可以是模式的也可以是非模式的，而 frame 通常都是非模式的。模式对话框的意思是说当这个对话框弹出时，应用程序除了等待用户关闭这个对话框以外不再作别的事情。对于那些要等待用户响应以后才能继续的操作来说。这是比较合适的。但是寻找替代的解决方案通常也不是一件坏事。举例来说，在工具条上设置一个字体选项可能比弹出一个模式的对话框让用户选择字体更能是整个应用程序看上去更流畅。

顶层窗口通常都拥有一个标题栏，这个标题栏上有一些按钮或者菜单或者别的修饰用来关闭，或者最小化，或者恢复这个窗口。而 frame 窗口则通常还会拥有菜单条，工具条和状态条。但是通常对话框则没有这些。在 Mac OS X 上，frame 窗口的菜单条通常不是显示在 frame 窗口的顶部而是显示在整个屏幕的顶部。

不要被"顶层窗口"的叫法和 wxApp::GetTopWindow 函数给搞糊涂了。wxWidgets 使用后者来得到应用程序的主窗口，这个主窗口通常是你在应用程序里创建的第一个 frame 窗口或者 dialog 窗口。

如果需要，你可以使用全局变量 wxTopLevelWindows 来访问所有的顶层窗口，这个全局变量是一个 wxWindowList 类型.

wxFrame

wxFrame 是主应用程序窗口的一个通用的选择。下图演示了常见的各个 frame 窗口元素。frame 窗口拥有一个可选的标题栏（上面有一些类似关闭功能的按钮），一个菜单条，一个工具栏，一个状态栏。剩下的区域则称为客户区，如果拥有超过一个子窗口，那么他们的尺寸和位置是由应用程序决定的。你应该通过第七章中会讲到的窗口布局控件来作相关的事情。或者如果只有两个子窗口的话，你可以考虑使用一个分割窗口。后者我们稍后就会讲到。

![](img/mht6E9(1).tmp)

在某些平台上不允许直接绘制 frame 窗口，因此你应该增加一个 wxPanel 作为容器来放置别的子窗口控件，以便可以通过键盘来遍历各个控件。

frame 窗口是允许包含多个工具条的，但是它只会自动放置第一个工具条。因此你需要自己明确的布局另外的工具条。

你很应该基于 wxFrame 实现你自己的 frame 类而不是直接使用 wxFrame 类，并且在其构造函数中创建自己的菜单条和子窗口。这会让你更方便处理类似 wxEVT_CLOSE 事件和别的命令事件。

你可以给 frame 窗口指定一个图标以便让系统显示在任务栏上或者文件管理器中。在 windows 平台上，你最好指定一组 16x16 和 32x32 并且拥有不同颜色深度的图标。在 linux 平台上也一样，windows 系统上的图标已经足够。而在 Max OsX 上，你需要更多的不同颜色和深度的图标。关于图标和图标列表的更多信息，可以参考第十章，ï¿½在应用程序中使用图片ï¿½。

除了默认的构造函数以外，wxFrame 还拥有下面的构造函数

```cpp
wxFrame(wxWindow* parent, wxWindowID id, const wxString& title,
    const wxPoint& pos = wxDefaultPosition,
    const wxSize& size = wxDefaultSize,
    long style = wxDEFAULT_FRAME_STYLE,
    const wxString& name = wxT("frame")); 
```

例如：

```cpp
wxFrame* frame = new wxFrame(NULL, ID_MYFRAME,
    wxT("Hello wxWidgets"), wxDefaultPosition,
    wxSize(500, 300));
frame->Show(true); 
```

注意在你显式的调用 Show 函数之前，frame 是不会被显示的。以便应用程序有机会在所有的子窗口还没有被显示的时候进行窗口布局。

没有必要给这个新建的 frame 窗口指定父窗口。如果指定了父窗口并且指定了 wxFRAME_FLOAT_ON_PARENT 类型位，这个新建的窗口将会显示在它的父窗口的上层。

要删除一个 frame 窗口，应该使用 Destroy 方法而不是使用 delete 操作符，因为 Destroy 会在处理完这个窗口的所有事件以后才真正释放这个窗口。调用 Close 函数会导致这个窗口收到 wxEVT_CLOSE 事件，这个事件默认的行为是调用 Destroy 方法。当一个 frame 窗口被释放时。它的所有的子窗口都将被释放，谁叫他们自己不是顶层窗口呢。

当最后一个顶层窗口被释放的时候，应用程序就退出了（这种默认的行为可以通过调用 wxApp:: SetExitOnFrameDelete 改变）。你应该在你的主窗口的 wxEVT_CLSE 事件处理函数中调用 Destroy 释放其它的顶层窗口（比如一个查找对话框之类），否则可能出现主窗口已经关闭但是应用程序却不退出的情况。

frame 窗口没有类似 dialog 窗口那样的 wxDialog::ShowModal 方法可以禁止其它顶层窗口处理消息，然后，还是可以通过别的方法达到类似的目的。一个方法是通过创建一个 wxWindowDisabler 对象，另外一个方法是通过 wxModalEventLoop 对象，传递这个 frame 窗口的指针作为参数，然后调用 Run 函数开始一个本地的事件循环，然后调用 Exit 函数（通常是在这个 Frame 窗口的某个事件处理函数里）退出这个循环。

下图演示了一个用户应用程序的 frame 窗口在 windows 上运行的样子。这个 frame 窗口拥有一个标题栏，一个菜单条，一个五彩的工具栏，在 frame 窗口的客户区有个分割窗口，底端有一个状态条，正显示着当用户的鼠标划过工具条上的按钮或者菜单项的时候显示的提示。

![](img/mht6FB(1).tmp)

wxFrame 的窗口类型比特位

Frame 窗口比起基本窗口类，增加了下面一些类型：

| wxDEFAULT_FRAME_STYLE | 其值为 wxMINIMIZE_BOX &#124; wxMAXIMIZE_BOX &#124; wxRESIZE_BORDER &#124; wxSYSTEM_MENU &#124; wxCAPTION &#124; wxCLOSE_BOX. |
| --- | --- |
| wxICONIZE | 以最小化的方式显示窗口。目前只适用于 Windows 平台. |
| wxCAPTION | 在窗口上显示标题. |
| wxMINIMIZE | 和 wxICONIZE 的意义相同. 也只适用于 Windows 平台. |
| wxMINIMIZE_BOX | 显示一个最小化按钮. |
| wxMAXIMIZE | 以最大化方式显示窗口. 仅适用于 Windows. |
| wxMAXIMIZE_BOX | 在窗口上显示最大化按钮. |
| wxCLOSE_BOX | 在窗口上显示关闭按钮. |
| wxSTAY_ON_TOP | 这个窗口显示在其它所有顶层窗口之上.仅适用于 Windows. |
| wxSYSTEM_MENU | 显示系统菜单. |
| wxRESIZE_BORDER | 边框可改变大小. |
| wxFRAME_TOOL_WINDOW | 窗口的标题栏比正常情况要小，而且在 windows 平台上，这个窗口不会在任务栏上显示. |
| wxFRAME_NO_TASKBAR | 创建一个标题栏是正常大小，但是不在任务栏显示的窗口。目前支持 windows 和 linux。需要注意在 windows 平台上，这个窗口最小化时将会最小化到桌面上，这有时候会让用户觉得不习惯，因此这种类型的窗口最好不要使用 wxMINIMIZE_BOX 类型。 |
| wxFRAME_FLOAT_ON_PARENT | 拥有这种类型的窗口总会显示在它的父窗口的上层。使用这种类型的窗口必须拥有非空父窗口，否则可能引发断言错误. |
| wxFRAME_SHAPED | 拥有这种类型的窗口可以通过调用 SetShape 函数来使其呈现不规则窗口外貌. |

下表列出的是 frame 窗口的扩展类型，需要使用 wxWindow::SetExtraStyle 函数才可以设置扩展类型

| wxFRAME_EX_CONTEXTHELP | 在 windows 平台上，这个扩展类型导致标题栏增加一个帮助按钮。当这个按钮被点击以后，窗口会进入一种上下文帮助模式。在这种模式下，任何应用程序被点击时，将会发送一个 wxEVT_HELP 事件。 |
| --- | --- |
| wxFRAME_EX_METAL | 在 Mac OS X 平台上,这个扩展类型将会导致窗口使用金属外观。请小心使用这个扩展类型，因为它可能会假定你的应用程序用户默认安装类声卡之类的多媒体设备。 |

wxFrame 的事件

下表列出了 frame 窗口比基本窗口类增加的事件类型：

| EVT_ACTIVATE(func) | 用来处理 wxEVT_ACTIVATE 事件, 在 frame 窗口被激活或者去激活的时候产生.处理函数的参数类型为 wxActivateEvent. |
| --- | --- |
| EVT_CLOSE(func) | 用来处理 wxEVT_CLOSE 事件, 在应用程序准备关闭窗口的时候产生. 处理函数的参数类型是 wxCloseEvent。这个类型支持 Veto 函数调用以阻止这个事件的进一步处理. |
| EVT_ICONIZE(func) | 用来处理 wxEVT_ICONIZE 事件, 当窗口被最小化或者被恢复普通大小的时候产生.处理函数的参数类型为 wxIconizeEvent. 通过使用 IsIconized 函数来判断究竟是最小化事件还是恢复事件. |
| EVT_MAXIMIZE(func) | 用来处理 wxEVT_MAXIMIZE 事件, 当窗口被最大化或者从最大化恢复的时候产生. 处理函数的参数类型为 wxMaximizeEvent. 通过 IsMaximized 函数判断究竟是最大化还是从最大化状态恢复. |

wxFrame 的成员函数

下面将介绍 wxFrame 类的主要的成员函数。因为 wxFrame 类是从 wxTopLevelWindow 和 wxWindow 继承过来的，请同样参考这两个类的成员函数。

CreateStatusBar 函数在 frame 窗口的底部创建一个拥有一个或多个域的状态栏. 可以使用 SetStatusText 函数来设置状态栏上的文字,使用 SetStatusWidths 函数来设置状态栏每个域的宽度(参考本章稍后对 wxStatusBar 的介绍). 使用举例:

```cpp
frame->CreateStatusBar(2, wxST_SIZEGRIP);
int widths[3] = { 100, 100, -1 };
frame->SetStatusWidths(3, widths);
frame->SetStatusText(wxT("Ready"), 0); 
```

CreateToolBar 函数在 frame 窗口的菜单条下创建一个工具栏. 当然你也可以直接创建一个 wxToolBar 实例，然后调用 wxFrame::SetToolBar 函数以便让 frame 类来管理你刚创建的 toolbar.

GetMenuBar 用来 SetMenuBar 操作 frame 和绑定的菜单条.一个 frame 窗口只可以有一个菜单条。如果你新创建了一个，那么就的那个将被删除和释放。

GetTitle 和 SetTitle 函数用来访问窗口标题栏上的文本.

Iconize 函数使得 frame 窗口最小化或者从最小化状态恢复.你可以使用 IsIconized 函数检查当前的最小化状态.

Maximize 函数使得 frame 窗口最大化或者从最大化状态恢复. 你可以用 IsMaximized 函数来检查当前的最大化状态.

SetIcon 函数可以设置在 frame 窗口最小化的时候显示的图标。各个平台的窗口管理器也可能把这个图标用于别的用途，比如显示在任务条上或者在显示在窗口浏览器中。你还可以通过 SetIcons 函数指定一组不同颜色和深度的图标列表.

SetShape 函数用来给 frame 窗口设置特定的显示区域。目前支持这个函数的平台有 Windows, Mac OS X, 和 GTK+以及打开某种 X11 扩展功能的 X11 版本.

ShowFullScreen 函数使用窗口在全屏和正常状态下切换，所谓全屏状态指的是 frame 窗口将隐藏尽可能多的修饰元素，而尽可能把客户区以最大化的方式显示。可以使用 IsFullScreen 函数来检测当前的全屏状态.

不规则的 Frame 窗口

如果你希望制作一个外貌奇特的应用程序，比如一个时钟程序或者一个媒体播放器，你可以给 frame 窗口指定一个不规则的区域，只有这个区域范围内的部分才会被显示出来。下图演示了一个没有任何标题栏，状态栏，菜单条的 frame 窗口，它的重画函数显示了一个企鹅，这个窗口被设置了一个区域使得只有这个企鹅才会显示。

![](img/mht6FE(1).tmp)

显示一个有着奇特外观的程序的代码并不复杂，你可以在附带光盘的 samples/shaped 目录里找到一个使用不同图片的完整的例子。总的来说，当窗口被创建时，它加载了一幅图片，并且使用这幅图片的轮廓创建了一个区域。在 GTK+版本上，设置窗口区域的动作必须在发送了窗口创建事件之后，所以你需要使用宏定义区别对待 GTK+的版本。下面的例子演示了你需要怎样对事件表，窗口的构造函数和窗口的创建事件处理函数进行修改：

```cpp
BEGIN_EVENT_TABLE(ShapedFrame, wxFrame)
    EVT_MOTION(ShapedFrame::OnMouseMove)
    EVT_PAINT(ShapedFrame::OnPaint)
#ifdef __WXGTK__
    EVT_WINDOW_CREATE(ShapedFrame::OnWindowCreate)
#endif
END_EVENT_TABLE()
ShapedFrame::ShapedFrame()
       : wxFrame((wxFrame *)NULL, wxID_ANY, wxEmptyString,
                  wxDefaultPosition, wxSize(250, 300),
                  | wxFRAME_SHAPED
                  | wxSIMPLE_BORDER
                  | wxFRAME_NO_TASKBAR
                  | wxSTAY_ON_TOP
            )
{
    m_hasShape = false;
    m_bmp = wxBitmap(wxT("penguin.png"), wxBITMAP_TYPE_PNG);
    SetSize(wxSize(m_bmp.GetWidth(), m_bmp.GetHeight()));
#ifndef __WXGTK__
    // On wxGTK we can't do this yet because the window hasn't
    // been created yet so we wait until the EVT_WINDOW_CREATE
    // event happens. On wxMSW and wxMac the window has been created
    // at this point so we go ahead and set the shape now.
    SetWindowShape();
#endif
}
// Used on GTK+ only
void ShapedFrame::OnWindowCreate(wxWindowCreateEvent& WXUNUSED(evt))
{
    SetWindowShape();
} 
```

为了创建一个区域模板，我们从一幅图片的创建了一个区域，其中的颜色参数用来指定需要透明显示的颜色，然后调用 frame 窗口的 SetShape 函数设置这个区域模板。

```cpp
void ShapedFrame::SetWindowShape()
{
    wxRegion region(m_bmp, *wxWHITE);
    m_hasShape = SetShape(region);
} 
```

为了让这个窗口可以通过鼠标拖拽来实现在窗口上的移动，我们可以通过下面的鼠标移动事件处理函数：

```cpp
void ShapedFrame::OnMouseMove(wxMouseEvent& evt)
{
    wxPoint pt = evt.GetPosition();
    if (evt.Dragging() && evt.LeftIsDown())
    {
        wxPoint pos = ClientToScreen(pt);
        Move(wxPoint(pos.x - m_delta.x, pos.y - m_delta.y));
    }
} 
```

而重画事件处理函数就比较简单了，当然在你的真实的应用程序里，你可以在其中放置更多的图形元素。

```cpp
void ShapedFrame::OnPaint(wxPaintEvent& evt)
{
    wxPaintDC dc(this);
    dc.DrawBitmap(m_bmp, 0, 0, true);
} 
```

你也可以参考 wxWidgets 的发行版中的 samples/shaped 目录中的例子。

小型 frame 窗口

在 Window 平台和 GTK+平台上，你可以使用 wxMiniFrame 来实现那些必须使用小的标题栏的窗口，例如，来实现一个调色板工具。下图(译者注：这个图片应该是搞错了，是下一个例子中的图片，不过作者的翻译源使用的就是这个图片，没有办法了。)演示了 windows 平台上的这种小窗口的样子。而在 Max Os X 平台上，这个小窗口则是直接使用的普通的 frame 窗口代替。其它方面 wxMiniFrame 和 wxFrame 都是一样的。

![](img/mht711(1).tmp)

wxMDIParentFrame

这个窗口类继承自 wxFrame,是 wxWidgets 的 MDI（多文档界面）体系的组成部分。MDI 的意思是由一个父窗口管理零个或多个子窗口的一种界面架构。这种 MDI 界面结构依平台的不同而有不同的外观，下图演示了两种主要的外观。在 windows 平台上，子文档窗口是排列在其父窗口内的一组 frame 窗口，这些文档窗口可以被平铺，层叠或者把其中的某一个在父窗口的范围内最大化以便在某个时刻仅显示一个文档窗口。 wxWidgets 会自动在主菜单上增加一组菜单用来控制文档窗口的这些操作。MDI 界面的一个优点是使得整个应用程序界面相对来说显得不那么零乱。这一方面是由于因为所有的文档窗口被限制在父窗口以内，另外一个方面，父窗口的菜单条会被活动的文档窗口的菜单条替代，这使得多个菜单条的杂乱性也有所减轻。

![](img/mht724(1).tmp)

而在 GTK+平台上，wxWidgets 则通过 TAB 页面控件来模拟多文档界面。在某个时刻只能有一个窗口被显示，但是用户可以通过 TAB 页面在窗口之间进行切换。在 Mac Os 上，MDI 的父窗口和文档窗口则都使用和普通的 Frame 窗口一样的外观，这符合这样的一个事实就是在 Mac Os 上，文档总是在一个新的窗口中被打开。

在那些 MDI 的文档窗口包含在父窗口之中的平台上，父窗口将把它的所有的文档窗口排列在一个子窗口中，而这个窗口可以和 frame 窗口中的其它控件子窗口和平共处。在上图中，父窗口将一个文本框控件和那些文档窗口进行了这样的排列。更详细的情形请参考 wxWidgets 发行版本的 samples/mdi 目录中的例子。

除了父窗口的菜单条外，每一个文档窗口都可以有自己的菜单条。当某个文档窗口被激活时，它的菜单条将显示在父窗口上，而没有子文档窗口被激活时，父窗口显示自己的菜单条。在构建子文档窗口的菜单条时，你应该主要要包含那些同样命令的父窗口的菜单项，再加上文档窗口自己的命令菜单项。父窗口和文档窗口也可以拥有各自的工具条和状态栏，但是这两者并没有类似菜单条这样的机制。

wxMDIParentFrame 类的构造函数和 wxFrame 类的构造函数是完全一样的.

wxMDIParentFrame 的窗口类型

wxMDIParentFrame 类额外的窗口类型列举如下:

| --- | --- | | wxFRAME_NO_WINDOW_MENU | Under Windows, removes the Window menu that is normally added automatically. |

wxMDIParentFrame 的成员函数

下面列出了 wxMDIParentFrame 类除了 wxFrame 的函数以外的主要成员函数。

ActivateNext 和 ActivatePrevious 函数激活前一个或者后一个子文档窗口。

Cascade 和 Tile 层叠或者平铺所有的子窗口. ArrangeIcons 函数以图标的方式平铺所有最小化的文档窗口.这三个函数都只适用于 Windows 平台.

GetActiveChild 获取当前活动窗口的指针(如果有的话).

GetClientWindow 函数返回那个包含所有文档窗口的子窗口的指针.这个窗口是自动创建的，但是你还是可以通过重载 OnCreateClient 函数来返回一个你自己的继承自 wxMDIClientWindow 的类的实例。如果你这样作，你就需要使用两步创建的方法创建这个多文档父窗口类.

wxMDIChildFrame

wxMDIChildFrame 窗口应该总被创建为一个 wxMDIParentFrame 类型窗口的子窗口. 正如我们前面已经提到的那样，依平台的不同，这种类型的窗口既可以是在其父窗口范围内的一组窗口列表，也有可能是在桌面上自由飞翔的普通的 frame 窗口。

除了父窗口必须不能为空，它的构造函数和普通的 frame 的构造函数是一样的。

```cpp
#include "wx/mdi.h"
wxMDIParentFrame* parentFrame = new wxMDIParentFrame(
     NULL, ID_MYFRAME, wxT("Hello wxWidgets"));
wxMDIChildFrame* childFrame = new wxMDIChildFrame(
     parentFrame, ID_MYCHILD, wxT("Child 1"));
childFrame->Show(true);
parentFrame->Show(true); 
```

wxMDIChildFrame 的窗口类型

wxMDIChildFrame 和 wxFrame 的窗口类型是一样的。尽管如此，不是所有的窗口类型的值在各个平台上都是有效的。

wxMDIChildFrame 的成员函数

wxMDIChildFrame 的除其基类 wxFrame 以外的主要成员函数列举如下:

Activate 函数激活本窗口，将其带到前台并且使得其父窗口显示它的菜单条。

Maximize 函数将其在父窗口范围内最大化(仅适用于 windows 平台).

Restore 函数使其从最大化的状态恢复为普通状态(仅适用于 windows 平台)。

wxDialog

对话框是一种用来提供信息或者选项的顶层窗口。他可以有一个可选的标题栏，标题栏上可以有可选的关闭或者最小化按钮，和 frame 窗口一样，你也可以给它指定一个图标以便显示在任务栏或者其它的地方。一个对话框可以组合任何多个非顶层窗口或者控件，举例来说，可以包含一个 wxNoteBook 控件和位于底部的两个确定和取消按钮。正如它的名字所指示的那样，对话框的目的相比较于整个应用程序的主窗口，只是为了给用户提示一些信息，提供一些选项等。

有两种类型的对话框：模式的或者非模式的。一个模式的对话框在应用程序关闭自己之前阻止其它的窗口处理来自应用程序的消息。而一个非模式的对话框的行为则更像一个 frame 窗口。它不会阻止应用程序中的别的窗口继续处理消息和事件。要使用模式对话框，你应该调用 ShowModal 函数来显示对话框，否则就应该象 frame 窗口那样，使用 Show 函数来显示对话框。

模式对话框是 wxWindow 派生类中极少数允许在堆栈上创建的对象之一，换句话说，你既可以使用下面的方法使用模式对话框:

```cpp
void AskUser()
{
    MyAskDialog *dlg = new MyAskDialog(...);
    if ( dlg->ShowModal() == wxID_OK )
        ...
    dlg->Destroy();
} 
```

也可以直接使用下面的方法:

```cpp
void AskUser()
{
    MyAskDialog dlg(...);
    if ( dlg.ShowModal() == wxID_OK )
        ...
    //这里不需要调用 Destroy()函数
} 
```

通常你应该从 wxDialog 类派生一个你自己的类，以便你可以更方便的处理类似 wxEVT_CLOSE 这样的事件以及其它命令类型的事件。通常你应该在派生类的构造函数中创建你的对话框需要的其它控件。

和 wxFrame 类一样，如果你的对话框只有一个子窗口，那么 wxWidgets 会自动为你布局这个窗口，但是如果有超过一个子窗口，应用程序应该自己负责窗口的布局（参考第七章）

当你调用 Show 函数时，wxWidgets 会调用 InitDialog 函数来发送一个 wxInitDialog 事件到这个窗口，以便开始对话框数据传输和验证以及其它的事情。

除了默认的构造函数，wxDialog 还拥有下面的构造函数:

```cpp
wxDialog(wxWindow* parent, wxWindowID id, const wxString& title,
    const wxPoint& pos = wxDefaultPosition,
    const wxSize& size = wxDefaultSize,
    long style = wxDEFAULT_DIALOG_STYLE,
    const wxString& name = wxT("dialog")); 
```

例如:

```cpp
wxDialog* dialog = new wxDialog(NULL, ID_MYDIALOG,
    wxT("Hello wxWidgets"), wxDefaultPosition,
    wxSize(500, 300));
dialog->Show(true); 
```

在调用 Show(true)函数或者 ShowModal 函数之前，对话框都是不可见的，以便其可以以不可见的方式进行窗口布局。

默认情况下，如果对话框的父窗口为 NULL,应用程序会自动指定其主窗口为其父窗口，你可以通过指定 wxDIALOG_NO_PARENT 类型来创建一个无父无母的孤儿 dialog 类，不过对于模式对话框来说，最好不要这样作。

和 wxFrame 类一样，最好不要直接使用 delete 操作符删除一个对话框，取而代之使用 Destroy 或者 Close。以便在对话框的所有事件处理完毕以后才释放对话框，当你调用 Cloes 函数时，默认发送的 wxEVT_CLOSE 事件的处理函数其实也是调用了 Destroy 函数.

需要注意，当一个模式的对话框被释放的时候，它的某个事件处理函数需要调用 EndModal 函数，这个函数的参数是导致这次释放的那个事件所属窗口的窗口标识符（例如 wxID_OK 或者 wxID_CANCEL).这样才能使得应用程序退出这个模式对话框的事件处理循环，从而使得调用 ShowModal 的代码能够释放这个窗口。而这个标识符则作为 ShowModal 函数的返回值。我们举下面一个 OnCancel 函数为例：

```cpp
//wxID_CANCEL 的命令事件处理函数
void MyDialog::OnCancel(wxCommandEvent& event)
{
    EndModal(wxID_CANCEL);
}
//显示一个模式对话框
void ShowDialog()
{
    //创建这个对话框
    MyDialog dialog;
    // OnCancel 函数将会在 ShowModal 执行的过程中被调用。
    if (dialog.ShowModal() == wxID_CANCEL)
    {
        ...
    }
} 
```

下图展示了几个典型的对话框，我们会在第九章阐述它们的创建过程。当然，对话框可以远比图中的那些更复杂，例如下面第二个图中的那样，它包含一个分割窗口，一个树形控件以便显示不同的面板(panels)，以及一个扮演属性编辑器的网格控件

![](img/mht736(1).tmp)

![](img/mht739(1).tmp)

wxDialog 的窗口类型

除了基本窗口类型以外，wxDialog 还有下面一些窗口类型可以使用：

| wxDEFAULT_DIALOG_STYLE | 其值等于 wxSYSTEM_MENU &#124; wxCAPTION &#124; wxCLOSE_BOX. |
| --- | --- |
| wxCAPTION | 在对话框上显示标题. |
| wxMINIMIZE_BOX | 在标题栏显示最小化按钮. |
| wxMAXIMIZE_BOX | 在标题栏显示最大化按钮. |
| wxCLOSE_BOX | 在标题栏显示关闭按钮. |
| wxSTAY_ON_TOP | 对呼框总在最前. 仅支持 windows 平台. |
| wxSYSTEM_MENU | 显示系统菜单. |
| wxRESIZE_BORDER | 显示可变大小边框. |
| wxDIALOG_NO_PARENT | 如果创建对话框的时候父窗口为 NULL，则应用程序会使用其主窗口作为对话框的父窗口，使用这个类型可以使得在这种情况下，对话框强制使用 NULL 作为其父窗口。模式窗口不推荐强制使用 NULL 作为父窗口。 |

下表解释了对话框类额外的扩展窗口类型。虽然 wxWS_EX_BLOCK_EVENTS 类型是窗口基类的窗口类型，但是由于它是对话框类的默认扩展类型，我们在这里也进行了描述。

| wxDIALOG_EX_CONTEXTHELP | 在 Windows 平台上,这个扩展类型使得对话框增加一个查询按钮.如果这个按钮被按下，对话框将进入一种帮助模式，在这种模式下，无论用户点击哪个子窗口，都将发送一个 wxEVT_HELP 事件. 这个扩展类型不可以和 wxMAXIMIZE_BOX 和 wxMINIMIZE_BOX 类型混用. |
| --- | --- |
| wxWS_EX_BLOCK_EVENTS | 这是一个默认被设置的扩展类型。意思是阻止命令类型事件向更上一级的窗口传播.要注意调用 SetExtraStyle 函数可以重置这个扩展类型. |
| wxDIALOG_EX_METAL | 在 Mac OS X 平台上,这个扩展类型导致对话框使用金属外观.不要滥用这个类型，因为它默认用户的机器有各种多媒体硬件 |

wxDialog 事件

下表解释了相对于窗口基类来说额外的对话框事件:

| EVT_ACTIVATE(func) | 用于处理 wxEVT_ACTIVATE 事件,在对话框即将激活或者去激活的时候产生。处理函数的参数类型是 wxActivateEvent。 |
| --- | --- |
| EVT_CLOSE(func) | 用户处理 wxEVT_CLOSE 事件,在应用程序或者操作系统即将关闭窗口的时候产生.处理函数的参数类型是 wxCloseEvent，这个事件可以调用 Veto 函数来放弃事件的后续处理. |
| EVT_ICONIZE(func) | 用来处理 wxEVT_ICONIZE 事件, 在窗口被最小化或者从最小化状态恢复的时候产生.处理函数的参数类型是 wxIconizeEvent. 调用 IsIconized 来检测到底是最小化还是从最小化恢复. |
| EVT_MAXIMIZE(func) | 用来处理 wxEVT_MAXIMIZE 事件,这个事件在窗口被最大化或者从最大化状态恢复的时候产生，处理函数的参数类型是 wxMaximizeEvent.调用 IsMaximized 函数确定具体的状态类型. |
| EVT_INIT_DIALOG(func) | 用于处理 wxEVT_INIT_DIALOG 事件,在对话框初始化自己之前被产生。处理函数的参数类型是 wxInitDialogEvent.wxPanel 也会产生这个事件.默认执行的操作是调用 TransferDataToWindow. |

# 4.5 容器窗口

# 4.5 容器窗口

容器窗口是用来装载别的可见元素的窗口，所谓可见元素指的是子窗口或者是画在这个窗口上的图案。

wxPanel

wxPanel 是一个在某些方面有点象 dialog 窗口的窗口。这个窗口通常被用来摆放那些除了对话框或者 frame 窗口以外的其它控件窗口。它也常被用来作为 wxNoteBook 控件的页面。它通常使用系统默认的颜色。

和对话框一样，可以使用它的 InitDialog 方法来产生一个 wxInitDialogEvent 事件。如果设置了 wxTAB_TRAVERSAL 类型，那么它通常可以通过使用类似 TAB 键的导航键遍历所有它上面的子控件。

除了默认的构造函数以外，wxPanel 还拥有下面的构造函数：

```cpp
wxPanel(wxWindow* parent, wxWindowID id,
    const wxPoint& pos = wxDefaultPosition,
    const wxSize& size = wxDefaultSize,
    long style = wxTAB_TRAVERSAL|wxNO_BORDER,
    const wxString& name = wxT("panel")); 
```

用法如下:

```cpp
wxPanel* panel = new wxPanel(frame, wxID_ANY,
    wxDefaultPosition, (500, 300)); 
```

wxPanel 的窗口类型

wxPanel 没有额外的窗口类型。

wxPanel 的成员函数

wxPanel 也没有额外的成员函数。

wxNotebook

这个类提供了一个有多个页面的窗口，页面之间可以通过边上的 TAB 按钮来切换。每个页面通常是一个普通的 wxPanel 窗口或者其派生类，当然你完全可以使用别的窗口。

NoteBook 的 TAB 按钮可以包含一个图片，也可以包含一个文本标签。图片是由 wxImageList（参考第十章）提供的，是通过在列表中的位置和页面对应的。

使用 notebook 的方法是，创建一个 wxNotebook 对象，然后调用其 AddPage 方法或者 InserPage 方法，传递一个用来作为页面的窗口指针。不要手动释放那些已经被 wxNoteBook 作为页面的窗口，你应该使用 DeletePage 来删除某个页面或者干脆等到 notebook 释放它自己的时候。notebook 会一并释放那些页面.

下面举例说明怎样创建一个有三个页面，包含文本和图片的 TAB 标签的 notebook：

```cpp
#include "wx/notebook.h"
#include "copy.xpm"
#include "cut.xpm"
#include "paste.xpm"
//创建 notebook
wxNotebook* notebook = new wxNotebook(parent, wxID_ANY,
  wxDefaultPosition, wxSize(300, 200));
// 创建图片列表
wxImageList* imageList = new wxImageList(16, 16, true, 3);
imageList->Add(wxIcon(copy_xpm));
imageList->Add(wxIcon(paste_xpm));
imageList->Add(wxIcon(cut_xpm));
// 创建页面
wxPanel1* window1 = new wxPanel(notebook, wxID_ANY);
wxPanel2* window2 = new wxPanel(notebook, wxID_ANY);
wxPanel3* window3 = new wxPanel(notebook, wxID_ANY);
notebook->AddPage(window1, wxT("Tab one"), true, 0);
notebook->AddPage(window2, wxT("Tab two"), false, 1);
notebook->AddPage(window3, wxT("Tab three"), false 2); 
```

下图演示了上述代码在 windows 平台上的结果：

![](img/mht1CCA%281%29.tmp)

在大多数的平台上，当 TAB 页面数量太多不能被完全显示的时候，都会自动出现导航按钮。但是在 Mac OS 上，这个导航按钮是不会出现的，因此在这个系统上，可以使用的页面数目将受到窗口大小和 TAB 标签大小的限制。

如果你在每个页面上都使用布局控件，并且在创建 notebook 的时候使用默认的大小 wxDefaultSize,那么 wxNotebook 将会自动调整大小以适合它的页面。

Notebook 窗口主题管理

在 Windows Xp 系统中，默认的窗口主题会给 notebook 的页面添加过渡色。虽然这是预期的本地行为，但是它可能会降低性能。出于审美方面的原因，你可能想直接使用单一的色彩，尤其是当 notebook 不是位于一个对话框以内的时候。如果你想阻止这种默认的行为，可以采用以下三种办法：第一：你可以给你的 notebook 指定 wxNB_NOPAGETHEM 类型来禁止某个特定的 notebook 的这种效果，或者你可以使用 wxSystemOptions::SetOption 来在应用程序范围内禁止它，或者你可以使用 SetBackgroundColour 函数来给某个单独的页面禁止它。要在应用程序范围内禁止它，你可以使用下面的代码： wxSystemOptions::SetOption(wxT("msw.notebook.themed-background"), 0); 将它的值设置成 1 可以再次允许这种效果。要让某个单独的页面禁用这种效果可以使用下面的方法：

```cpp
wxColour col = notebook->GetThemeBackgroundColour();
if (col.Ok())
{
    page->SetBackgroundColour(col);
} 
```

在 windows 系统以外的平台，或者如果一个应用程序没有使用主题风格，GetThemeBackgroundColour 都将返回一个未初始化的颜色，因此上面的代码在各个平台都是可以使用的。另外就是上述的这部分代码的语法和行为是有可能在将来的 wxWidgets 版本中发生变化，请参考你本地 wxWidgets 发行版中的 wxNotebook 类的文档来获取最新的信息。

wxNotebook 的窗口类型

wxNotebook 可以拥有下面的额外窗口类型:

| wxNB_TOP | 标签放在顶部. |
| --- | --- |
| wxNB_LEFT | 标签放在左面. 不是所有的 WindowsXp 的主题都支持这个选项. |
| wxNB_RIGHT | 标签在右面. 不是所有的 WindowsXp 的主题都支持这个选项. |
| wxNB_BOTTOM | 标签在底部. 不是所有的 WindowsXp 的主题都支持这个选项. |
| wxNB_FIXEDWIDTH | 所有标签宽度相同.仅适用于 windows. |
| wxNB_MULTILINE | 可以有多行的标签. 仅适用于 windows |
| wxNB_NOPAGETHEME | 在 windos 上禁用主题风格.以提高性能和提供另外一种审美选择. |

wxNotebook 的事件

wxNotebook 可以产生 wxNotebookEvent 事件，这个事件可以被它和它的继承者处理.

| EVT_NOTEBOOK_PAGE_CHANGED(id,func) | 当前页面已经改变. |
| --- | --- |
| EVT_NOTEBOOK_PAGE_CHANGING(id,func) | 当前页面即将改变。你可以使用 Veto 函数阻止这种改变. |

wxNotebook 的成员函数

AddPage 增加一个页面,InsertPage 在某个固定位置插入一个页面.你可以使用文本标签或者图片标签或者两者都有，用法如下:

```cpp
//增加一个使用了文本和图片两种标签都有的页面，并且当前处于未选中状态。
// (其中图片使用的是图片列表中的索引为 2 的那个).
notebook->AddPage(page, wxT("My tab"), false, 2); 
```

DeletePage 函数移除并且释放某个特定的页面，而 RemovePage 函数则仅仅是移除这个页面。DeleteAllPages 函数用来删除所有的页面。当 wxNoteBook 被释放时，它也会自动删除所有的页面。

AdvanceSelection 函数循环选择页面。 SetSelection 函数以基于 0 的索引选择特定的页面. GetSelection 取得当前选中页面的索引或者返回 wxNOT_FOUND.

SetImageList 函数用来给 Notebook 设置一个图标列表，这个函数仅是设置而不绑定图片列表，这意味着在 notebook 控件被删除的时候，这个图片列表控件并不会被删除，如果你希望使用绑定，则可以使用 AssignImageList 函数. GetImageList 函数返回 notebook 相关的图片列表对象.图片列表对象用来提供每个页面标签中使用的图片，详情请参考第十章。

GetPage 函数用来返回和某个索引对应的页面窗口指针, GetPageCount 函数则返回页面总数.

SetPageText 和 GetPageText 用来操作页面标签上的文本.

SetPageImage 和 GetPageImage 用来操作页面标签上的图片在图片列表中的索引，

wxNotebook 的替代选择

wxNotebook 是 wxBookCtrlBase 的派生类, 这个基类是用来提供用于管理一组页面的所有数据和方法的抽象类。和 wxNotebook 有着相似 API 的类还有两个，一个是 wxListbook，一个是 wxChoicebook,你也可以实现你自己的类，比如你可以实现一个 wxTreebook.

wxListbook 使用一个 wxListCtrl 变量来控制页面;ListCtrl 控件是一种在内部显示一组带有标签的图片的控件. 在 wxListbook 中，相关的 ListCtrl 控件可以显示在上下左右四个方向，默认是在左边。这是 wxNotebook 的一个很有吸引力的替代者。因为即使在 Mac Os X 平台上，ListCtrl 可以管理的项目数量也几乎没有限制，因此 wxListbook 可以管理的页面数也不受窗口大小和标签长度的限制。

wxChoicebook 则使用一个选择控件（一个下拉列表）来管理页面。比较适用于窗口空间比较小的场合。这个控件不会在页面标签处显示图片，而且默认情况下，选择控件显示在整个控件的上方。

上述这两个控件的头文件分别为 wx/listbook.h 和 wx/choicebk.h.它们的事件处理函数的参数类型分别为 wxListbookEvent 和 wxChoicebookEvent 类型，事件影射宏则分别为 EVT_XXX_PAGE_CHANGED(id, func)和 EVT_XXX_PAGE_CHANGING(id,func),其中 XXX 代表 LISTBOOK 或者 CHOICEBOOK.

你可以使用类似 wxNotebook 定义的那些窗口类型，也可以类似用 wxCHB_TOP 或者 wxLB_TOP 取代 wxNB_TOP 这样的窗口类型，它们的值都是一样的，

wxScrolledWindow

尽管所有的窗口都可以拥有滚动条，但是为了让滚动条工作，还需要一些额外的代码。这个类则为让不同类型的窗口类中的滚动条正确工作提供了足够的灵活性。 wxScrolledWindow 主要实现了那些让窗口可以以一定的单位连续的滚动（而不是不定大小的跳跃），也可以定义在使用翻页键进行翻页的时候的大小。这种实现通常仅适用于类似画图程序中的那种翻页功能，对于一些复杂功能的文本编辑程序则不一定适合。因为富文本编辑器中每一行的高度和宽度都有可能不同。wxGrid 网格控件就是这样的一个例子（在网格控件中，每一行和每一列的宽度都有可能不同），在这种情况下，你应该自己直接从 wxWindow 类实现自己的派生类，从而实现自己的窗口滚动机制。

要适用滚动窗口，你需要提供每次逻辑移动的单位（也就是当处理上滚一行和下滚一行时窗口应该移动的大小）以及整个窗口的虚拟逻辑大小。然后 wxScrollWindow 类就会自己关注是否显示滚动条，以及滚动条上的滑钮应该用多大显示等等这些细节问题.

下面演示了怎样创建一个滚动窗口:

```cpp
#include "wx/scrolwin.h"
wxScrolledWindow* scrolledWindow = new wxScrolledWindow(
    this, wxID_ANY, wxPoint(0, 0), wxSize(400, 400),
    wxVSCROLL|wxHSCROLL);
// 设置窗口的虚拟逻辑大小： 1000x1000
// 每次滚动 10 个象素
int pixelsPerUnixX = 10;
int pixelsPerUnixY = 10;
int noUnitsX = 1000;
int noUnitsY = 1000;
scrolledWindow->SetScrollbars(pixelsPerUnitX, pixelsPerUnitY,
    noUnitsX, noUnitsY); 
```

第二种设置虚拟大小的方法是使用 SetVirtualSize 函数，它的参数是以象素为单位的虚拟大小。然后再用 SetScrollRate 函数来设置水平和垂直方向上的滚动增量。第三种方法是使用布局控件来布局窗口，滚动窗口会自动计算所有子窗口需要的窗口大小作为窗口的虚大小，你同样需要调用 SetScrollRate 函数来设置滚动增量。

你可以想普通窗口那样使用重画事件，但是在进行任何窗口重画动作之前你应该调用 DoPrepareDC 函数来保证重画动作以当前的窗口原点作为原点，就象下面演示的那样：

```cpp
void MyScrolledWindow::OnPaint(wxPaintEvent& event)
{
    wxPaintDC dc(this);
    DoPrepareDC(dc);
    dc.SetPen(*wxBLACK_PEN);
    dc.DrawLine(0, 0, 100, 100);
} 
```

你也可以直接重载 OnDraw 虚函数，wxScrolledWindow 在调用这个函数之前，会首先调用 DoPrepareDC 函数，因此你只需要象下面演示的那样作：

```cpp
void MyScrolledWindow::OnDraw(wxDC& dc)
{
    dc.SetPen(*wxBLACK_PEN);
    dc.DrawLine(0, 0, 100, 100);
} 
```

需要注意的是，在别的任何事件处理函数中如果要重画窗口，你同样需要调用 DoPrepareDC 函数。

你也可以象下面这样提供你自己的 DoPrepareDC 函数，这个函数默认的行为只是把设备操作原点移动到当前滚动条开始的位置：

```cpp
void wxScrolledWindow::DoPrepareDC(wxDC& dc)
{
    int ppuX, ppuY, startX, startY;
    GetScrollPixelsPerUnit(& ppuX, & ppuY);
    GetViewStart(& startX, & startY);
    dc.SetDeviceOrigin( - startX * ppuX, - startY * ppuY );
} 
```

关于在 wxScrollWindow 上进行作图的详细情形，包括怎样使用双缓冲区，请参考第五章，ï¿½重画和打印ï¿½中的 wxPaintDC 小节。

wxScrolledWindow 的窗口类型

wxScrolledWindow 类并没有特别的窗口类型，但是通常需要设置 wxVSCROLL|wxHSCROLL 类型，这也是 wxScrolledWindow 的默认类型。在某些平台上因为效率的原因可能不支持这两个类型。

wxScrolledWindow 的事件

wxScrolledWindow 会产生 wxScrollWinEvent 事件（参见下表）.这些事件不会在父子窗口关系中传播，因此要处理这种事件，你必须定义自己的滚动窗口派生类或者挂载你自己的事件表。不过在通常情况下，你并不需要覆盖默认的处理函数来自己处理这些事件。

| EVT_SCROLLWIN(func) | 处理所有滚动事件. |
| --- | --- |
| EVT_SCROLLWIN_TOP(func) | 处理事件 wxEVT_SCROLLWIN_TOP，滚动到最顶端. |
| EVT_SCROLLWIN_BOTTOM(func) | 处理事件 wxEVT_SCROLLWIN_BOTTOM 滚动到最底端事件. |
| EVT_SCROLLWIN_LINEUP(func) | 处理事件 wxEVT_SCROLLWIN_LINEUP 上滚一行. |
| EVT_SCROLLWIN_LINEDOWN(func) | 处理事件 wxEVT_SCROLLWIN_LINEDOWN 下滚一行. |
| EVT_SCROLLWIN_PAGEUP(func) | 处理事件 wxEVT_SCROLLWIN_PAGEUP 上滚一页. |
| EVT_SCROLLWIN_PAGEDOWN(func) | 处理事件 wxEVT_SCROLLWIN_PAGEDOWN 下滚一页. |

wxScrolledWindow 的成员函数介绍

CalcScrolledPosition 和 CalcUnscrolledPosition 函数都需要四个参数，前两个整数参数是输入需要计算的点，后两个则是指向整数的指针用来放置计算结果。第一个函数用来计算实际位置到逻辑位置的映射。如果当前的滚动条下滚了 10 个象素，则 0 这个数字作为输入将得到对应输出-10,第二个函数则实现相反的功能。

EnableScrolling 函数用来允许或者禁止垂直或者水平方向的物理滚动。物理滚动的含义是说在收到滚动事件的时候对窗口进行物理上的平移。但是如果应用程序需要不等量的移动（比如，由于字体的不同为了避免滚动的时候显示半行字的情况出现），则需要禁止物理移动，在这种情况下，应用程序需要自己移动对应的子窗口。物理移动在所有支持物理移动的平台上都是默认打开的。当然不是所有的平台都支持物理移动。

GetScrollPixelsPerUnit 函数在两个指向整数的指针中返回水平和垂直方向上的移动单位。返回 0 表示在那个方向上不能滚动。

GetViewStart 函数返回窗口可视部分左上角的座标，单位是逻辑单位，你需要乘以 GetScrollPixelsPerUnit 的返回值以便将结果转换成象素单位。

GetVirtualSize 返回当前设定的以象素为单位的虚拟窗口大小。

DoPrepareDC 将画画设备的原点设置到当前的可见原点。

Scroll 函数将窗口滚动到一个特定的逻辑单位位置（注意这里的单位不是象素）.

SetScrollbars 用来设置移动单位的象素值,各个方向的移动单位总数,水平或者垂直方向的当前滚动位置(可选)以及是否立即刷新窗口（默认否）等。

SetScrollRate 用来设置滚动单位，相当于单独设置 SetScrollbars 中的移动单位参数.

SetTargetWindow 用来滚动非 wxScrolledWindow 类型的其它窗口.

滚动非 wxScrolledWindow 类型的窗口

如果你想自己实现窗口的滚动行为，你可以直接从 wxWindow 派生你的窗口类，然后使用 SetScrollbar 函数来设置这个窗口的滚动条。

SetScrollbar 函数的参数如下表的说明：

| int orientation | 滚动条的类型: wxVERTICAL or wxHORIZONTAL. |
| --- | --- |
| int position | 滚动条滑块的位置，逻辑滚动单位. |
| int visible | 可见部分大小，逻辑滚动单位. 通常会决定滑块的长度. |
| int range | 滚动条的最大长度，逻辑滚动单位. |
| bool refresh | 是否立即刷新窗口. |

举例来说，如果你想显示一个 50 行文本的文本窗口，使用同样的字体，而窗口的大小只够显示 16 行，你可以使用下面的代码：

```cpp
SetScrollbar(wxVERTICAL, 0, 16, 50) 
```

注意在上面的例子中，滑块的位置永远不可能大于 50-16=34.

你可以通过用当前窗口除以当前字体下文本的高度来得到当前窗口可以显示多少行这个值。

如果你是自己实现滚动行为，你总是需要在窗口大小发生改变的时候更改滚动条的设置。因此你可以在首次计算滚动条参数的代码中使用 AdjustScrollbars 函数，然后在 wxSizeEvent 的处理事件中使用 AdjustScrollbars 函数。

你可以参考 wxGrid 控件的代码来获得实现自定义滚动窗口的更多灵感。

你也可以参考 wxWidgets 手册中的 wxVScrolledWindow 类，它可以用来建立一个可以在垂直方向进行不固定单位滚动的窗口类。

wxSplitterWindow

这个类用来管理最多两个窗口，如果你想在更多窗口中实现分割，你可以使用多个分割窗口。当前的窗口可以被应用程序分割成两个窗口，比如通过一个菜单命令，也可以通过应用程序命令或者通过分割窗口的用户操作界面（双击分割条或者拖拽分割条使得其中一个窗口的大小变为 0）重新变为一个窗口.其中拖拽分割条的方法将会受到后面会提到的 SetMinimumPaneSize 方法的限制。

在大多数平台上，当分割条被拖拽时，会有一个和背景颜色相反的竖条随之移动以显示分割条的最终位置，你可以通过使用 wxSP_LIVE_UPDATE 窗口类型来使得分割条以实时方式通过直接改变两个窗口的大小来代替那种默认方式。实时方式是 Mac OS X 上默认的也是唯一的方式。

下面的代码演示了怎样创建一个分割窗口来操作两个窗口并且隐藏其中的一个：

```cpp
#include "wx/splitter.h"
wxSplitterWindow* splitter = new wxSplitterWindow(this, wxID_ANY,
    wxPoint(0, 0), wxSize(400, 400), wxSP_3D);
leftWindow = new MyWindow(splitter);
leftWindow->SetScrollbars(20, 20, 50, 50);
rightWindow = new MyWindow(splitter);
rightWindow->SetScrollbars(20, 20, 50, 50);
rightWindow->Show(false);
splitter->Initialize(leftWindow);
//去掉下面的注释以便禁止窗口隐藏
//    splitter->SetMinimumPaneSize(20);
下面的代码代码演示了创建分割窗口以后怎样使用它：
void MyFrame::OnSplitVertical(wxCommandEvent& event)
{
    if ( splitter->IsSplit() )
        splitter->Unsplit();
    leftWindow->Show(true);
    rightWindow->Show(true);
    splitter->SplitVertically( leftWindow, rightWindow );
}
void MyFrame::OnSplitHorizontal(wxCommandEvent& event)
{
    if ( splitter->IsSplit() )
        splitter->Unsplit();
    leftWindow->Show(true);
    rightWindow->Show(true);
    splitter->SplitHorizontally( leftWindow, rightWindow );
}
void MyFrame::OnUnsplit(wxCommandEvent& event)
{
    if ( splitter->IsSplit() )
        splitter->Unsplit();
} 
```

下图演示了分割窗口在 windows 平台上的例子，在这个例子中，分割窗口没有使用 wxSP_NO_XP_THEME 类型。如果使用了这个类型，分割窗口将会拥有更传统的下沉边框和 3 维外观。

![](img/mht1CCD%281%29.tmp)

wxSplitterWindow 的窗口类型

| wxSP_3D | 使用三维效果的边框和分割条 . |
| --- | --- |
| wxSP_3DSASH | 使用三维效果分割条 . |
| wxSP_3DBORDER | 和 xSP_BORDER 效果相同 |
| wxSP_BORDER | 使用标准边框 . |
| wxSP_NOBORDER | 无边框(默认值). |
| wxSP_NO_XP_THEME | 在 Xp 操作系统上，如果你不喜欢默认的效果，使用三维边框和分割条效果。 |
| wxSP_PERMIT_UNSPLIT | 即使设置了最小值也允许窗口被隐藏. |
| wxSP_LIVE_UPDATE | 在分割条移动的时候实时更新窗口. |

wxSplitterWindow 事件

wxSplitterWindow 使用 wxSplitterEvent 类型的事件处理函数

| EVT_SPLITTER_SASH_POS_CHANGING(id,func) | 用于处理 wxEVT*COMMAND_SPLITTER_SASH* POS_CHANGING 事件,在分割条位置即将改变的时候产生。调用 Veto 函数阻止分割条移动，或者调用事件的 SetSashPosition 函数来更改分割条的位置. |
| --- | --- |
| EVT_SPLITTER_SASH(id,func) | 用于处理 wxEVT*COMMAND_SPLITTER* SASH_POS_CHANGED 事件,分割条的位置已经改变你可以通过事件的 SetSashPosition 函数阻止这种改变或者更改变化的幅度. |
| EVT_SPLITTER_UNSPLIT(id,func) | 用于处理 wxEVT_COMMAND_SPLITTER_UNSPLIT 事件,在有某个窗口被隐藏的时候产生. |

EVT_SPLITTER_DCLICK(id,func) | 用于处理 wxEVT_COMMAND_SPLITTER_DOUBLECLICKED 事件,在分割条被双击的时候产生. |

wxSplitterWindow 的成员函数

GetMinimumPaneSize 和 SetMinimumPaneSize 函数用于操作分割窗口的最小窗格大小。默认为 0,意味着分割窗口的每一侧的大小都可以被缩减至 0。相当于移除某一部分分割窗口.要阻止这种情形，也为了阻止拖拽分割条的时候超出边界，可以将其设置成一个大于 0 的值比如 20 个象素。虽然这样，如果设置了 wxSP_PERMIT_UNSPLIT 窗口类型，即使最小值设置为大于 0 的数，也还是可以将某个分割窗口隐藏。

GetSashPosition 和 SetSashPosition 用来操作分割条的位置，传递 true 参数给 SetSashPosition 函数导致分割条被立刻刷新。

GetSplitMode 和 SetSplitMode 函数用来设置分割的方向 wxSPLIT_VERTICAL 或者 wxSPLIT_HORIZONTAL.

GetWindow1 和 GetWindow2 用来获取两个分割窗口的指针。

Initialize 函数使用一个窗口指针参数调用，用来显示两个分割窗口中的某一个窗口。

IsSplit 函数用来判断窗口是否处于分割状态.

ReplaceWindow 用来替换被分割窗口控制的两个窗口中的一个。使用这个函数要好过先使用 Unsplit 函数然后再增加另外一个窗口.

SetSashGravity 用一个浮点小数来设置分割比例。0.0 表示只显示右边或者下边的窗口，1.0 表示只显示左边或者上边的窗口。中间的值则表示按照某种比例分配两个窗口的大小。使用 GetSashGravity 函数来获取当前的分割比例。

SplitHorizontally 函数 SplitVertically 使用一个可选的分割尺寸来初始化分割窗口。

Unsplit 去掉指定的那个分割窗口.

UpdateSize 用来使得分割窗口立即刷新(通常情况下，这是在系统空闲的时候完成的).

布局控件中使用 wxSplitterWindow 的说明

在布局控件中使用分割窗口有一点细微之处需要说明一下。如果你不需要这个分割窗口的分割条是可移动的，你可以在创建两个子窗口的时候指定绝对大小。这将固定这两个子窗口的最小大小，从而使得分割条不能自由移动。如果你希望分割条能正常移动，在创建两个子窗口的时候你就需要使用默认的大小，然后在分割窗口的构造函数中指定其最小大小。然后在将这个分割窗口增加到布局控件的时候，在 Add 函数中使用 wxFIXED_MINSIZE 标记来告诉 wxWidgets 将分割窗口控件当前的大小作为其最小大小。

另外一种情况是，分割窗口没有设定分割条的位置，也没有设定它的子窗口的大小，当布局控件完成布局时，分割窗口不愿意过早的设置分割条的位置，而要等到系统空闲的时候（这是它的默认行为）。在这种情况下，我们可能会看到当窗口刚刚可见的时候分割条复位自己的位置。为了避免出现这种情景，我们应该在对布局控件调用 Fit 之后，马上调用分割窗口的 UpdateSize 函数让其作立即更新。

默认情况下，当用户或者应用程序改变分割窗口的大小时，只有底端（或者右端）的子窗口改变自己的大小以便获取或减小额外的空间，要改变这种默认的行为，你可以使用前面说过的 SetSashGravity 函数指定一个固定的分割比例。

wxSplitterWindow 的替代者

如果在应用程序中你需要使用很多的分割窗口，不妨考虑使用 wxSashWindow.这个窗口允许它的任何边成为一个分割条，以便将其分割成多个窗口。而新分的这些窗口通常是用户创建的 wxSashWindow 的子窗口。

当 wxSashWindow 的分割条被拖动时，会向应用程序发送 wxSashEvent 事件以便事件处理函数可以相应的进行窗口布局。布局是通过一个称为 wxLayoutAlgorithm 的类来完成的，这个类可以根据父窗口的不同提供 LayoutWindow，LayoutFrame， LayoutMDIFrame 等多种不同的排列方法。

你还可以使用 wxSashLayoutWindow 类，这个类通过 wxQueryLayoutInfoEvent 事件来给 wxLayoutAlgorithm 提供布局方向和尺寸方面的信息。

关于 wxSashWindow，wxLayoutAlgorithm 和 wxSashLayoutWindow 更多的信息请参考手册中的相关内容。wxSashWindow 不允许子窗口被移动或者分离，因此在不久的将来，这个类可能被一个通用的支持合并和分离的布局框架体系所代替。

下图演示了 samples/sashtest 目录中的例子在 windows 上执行的效果：

![](img/mht1CE0%281%29.tmp)

# 4.6 非静态控件

# 4.6 非静态控件

非静态控件指的是那些可以响应鼠标和键盘事件的类似 wxButton 和 wxListBox 之类的控件。这里我们会对其中最基本的那些进行一些描述。更高级的内容被安排在第十二章，你可以参考附录 E 中的方法从网上下载或者创建你自己的更高级的控件。

wxButton

wxButton 控件看上去象是一个物理上可以按下的按钮，它拥有一个文本标签，是用户界面中最常用的一个元素，它可以被放置在对话框，或者面板(wxPanel)或者几乎任何其它的窗口中。当用户点击按钮的时候，会产生一个命令类型的事件。

下面是创建按钮的一个简单的例子:

```cpp
#include "wx/button.h"
wxButton* button = new wxButton(panel, wxID_OK, wxT("OK"),
    wxPoint(10, 10), wxDefaultSize); 
```

下图演示了按钮在 Windows Xp 系统上的默认外观：

![](img/mhtCB59%281%29.tmp)

wxWidgets 创建的按钮的默认大小是依靠静态函数 wxButton::GetDefaultSize 指定的，这个函数在不同的平台上返回不同的值。不过你可以通过给按钮指定 wxBU_EXACTFIT 类型来使其产生刚好符合其标签尺寸的大小。

wxButton 的窗口类型

| wxBU_LEFT | 标签文本作对齐. 仅适用于 Windows 和 GTK+平台. |
| --- | --- |
| wxBU_TOP | 标签文本上对齐. 仅适用于 Windows 和 GTK+平台. |
| wxBU_RIGHT | 标签文本右对齐。仅适用于 Windows 和 GTK+. |
| wxBU_BOTTOM | 标签文本对齐按钮底部. 仅适用于 Windows 和 GTK+平台. |
| wxBU_EXACTFIT | 不使用默认大小创建按钮，而是按照其标签文本的大小来创建按钮. |
| wxNO_BORDER | 创建一个平面按钮。仅适用于 Windows 和 GTK+平台. |

wxButton 的事件

wxButton 可以创建类型为 wxCommandEvent 的事件，如下表所示：

|  |  |
| --- | --- |
| EVT_BUTTON(id,func) | 用于处理 wxEVT_COMMAND_BUTTON_CLICKED 事件,在用户用左键单击按钮的时候产生. |

wxButton 的成员函数

wxButton 的成员函数

SetLabel 和 GetLabel 函数用来操作按钮标签文本.你可以使用"&"前导符来指示用于这个按钮的快捷键，仅适用于 Windows 和 GTK+平台. SetDefault 将按钮设置为其父窗口的默认按钮，这样用户按回车键就相当于用左键点击了这个按钮.

wxButton 的标签

你可以使用一个前导符"&"来指定其后的字符为这个按钮的快捷键，不过这个操作仅适用于 Windows 和 GTK+的版本，在其它的平台上，这个前导符将被简单的忽略。

在某些系统，尤其是 GTK+系统中，例如确定或者新建这样的标准按钮的标签以及附带显示的一些小图片都被进行了默认的定义。在 wxWidgets 中，通常 你只需要指定这个按钮的标识符为系统预定义的标识符，那么这些预定义的标签和小图片将被显示，不过，指定和默认值不同的标签文本以及小图片的操作都是允许 的。

推荐的对于这些预定义按钮的使用方法如下，你只需要指定预定义的标识符，不需要指定标签文本，或者指定标签文本为空字符串：

```cpp
wxButton* button = new wxButton(this, wxID_OK); 
```

wxWidgets 会各种平台提供合适的标签，在上面的例子中，在 windows 或者 Mac OSX 平台上，将使用字符串"&OK",而在 GTK+系统中，会使用当前语言下的 OK 按钮标签。然后如果你提供了自己的标签文本， wxWidgets 将会按照你的指示办事：

```cpp
wxButton* button = new wxButton(this, wxID_OK, wxT("&Apply")); 
```

这会导致无论在哪种平台上，这个按钮都将覆盖标准标识符的定义，使用"Apply"作为这个按钮的标签。

你可以使用 wxGetStockLabel 函数(需要包含 wx/stockitem.h 这个头文件)来获得某个预定义窗口标识符的标签，使用窗口标识符和 true（如果你希望在返回的结果中包含前导符(&)）作为参数.

下表列出了预定义标识符和其默认标签的对应关系。

| 预定义标识符 | 预定义标签 |
| --- | --- |
| wxID_ADD | "Add" |
| wxID_APPLY | "&Apply" |
| wxID_BOLD | "&Bold" |
| wxID_CANCEL | "&Cancel" |
| wxID_CLEAR | "&Clear" |
| wxID_CLOSE | "&Close" |
| wxID_COPY | "&Copy" |
| wxID_CUT | "Cu&t" |
| wxID_DELETE | "&Delete" |
| wxID_FIND | "&Find" |
| wxID_REPLACE | "Rep&lace" |
| wxID_BACKWARD | "&Back" |
| wxID_DOWN | "&Down" |
| wxID_FORWARD | "&Forward" |
| wxID_UP | "&Up" |
| wxID_HELP | "&Help" |
| wxID_HOME | "&Home" |
| wxID_INDENT | "Indent" |
| wxID_INDEX | "&Index" |
| wxID_ITALIC | "&Italic" |
| wxID_JUSTIFY_CENTER | "Centered" |
| wxID_JUSTIFY_FILL | "Justified" |
| wxID_JUSTIFY_LEFT | "Align Left" |
| wxID_JUSTIFY_RIGHT | "Align Right" |
| wxID_NEW | "&New" |
| wxID_NO | "&No" |
| wxID_OK | "&OK" |
| wxID_OPEN | "&Open" |
| wxID_PASTE | "&Paste" |
| wxID_PREFERENCES | "&Preferences" |
| wxID_PRINT | "&Print" |
| wxID_PREVIEW | "Print previe&w" |
| wxID_PROPERTIES | "&Properties" |
| wxID_EXIT | "&Quit" |
| wxID_REDO | "&Redo" |
| wxID_REFRESH | "Refresh" |
| wxID_REMOVE | "Remove" |
| wxID_REVERT_TO_SAVED | "Revert to Saved" |
| wxID_SAVE | "&Save" |
| wxID_SAVEAS | "Save &As..." |
| wxID_STOP | "&Stop" |
| wxID_UNDELETE | "Undelete" |
| wxID_UNDERLINE | "&Underline" |
| wxID_UNDO | "&Undo" |
| wxID_UNINDENT | "&Unindent" |
| wxID_YES | "&Yes" |
| wxID_ZOOM_100 | "&Actual Size" |
| wxID_ZOOM_FIT | "Zoom to &Fit" |
| wxID_ZOOM_IN | "Zoom &In" |
| wxID_ZOOM_OUT | "Zoom &Out" |

wxBitmapButton

和普通按钮唯一不同的地方在于，它将显示一个小图片而不是标签文本。

下面演示了创建一个图片按钮的方法：

```cpp
#include "wx/bmpbuttn.h"
wxBitmap bitmap(wxT("print.xpm"), wxBITMAP_TYPE_XPM);
wxBitmapButton* button = new wxBitmapButton(panel, wxID_OK,
    bitmap, wxDefaultPosition, wxDefaultSize, wxBU_AUTODRAW); 
```

以及在 Windows 平台上的显示效果:

![](img/mhtCB6C%281%29.tmp)

图片按钮通常只需要指定一个拥有透明背景的图片，wxWidgets 将会在任何状态的时候都使用这个图片，不过如果你愿意，你可以给不同的按钮状态指定不同的图片，比如选中状态，释放按钮状态以及不可用状态等。

XPM 图片格式是推荐的图片格式，因为它可以很容易的提供透明相关的信息以及可以被包含在 C++代码中。不过加载别的格式的图片也是可以的，比如常见的 JPEG,PNG,GIF 以及 BMP 格式。

wxBitmapButton 的窗口类型

| wxBU_AUTODRAW | 如果指定了这个类型，图片按钮将只会使用标签图片以系统默认的方式自动被绘制，它将包含边框和 3D 的外观。如果没有指定这个类型，图片按钮将按照不同状态提供的不同的图片来进行绘制. 这个类型仅适用于 Windows 和 Mac OS. |
| --- | --- |
| wxBU_LEFT | 图片左对齐. 在 Mac OS 上忽略这个类型. |
| wxBU_TOP | 图片顶端对齐. Mac OS 不适用. |
| wxBU_RIGHT | 图片右对齐. Mac OS 不适用. |
| wxBU_BOTTOM | 图片底端对齐. Mac OS 不适用. |

wxBitmapButton 事件

图片按钮的事件和 wxButton 完全相同.

wxBitmapButton 的成员函数

SetBitmapLabel 和 GetBitmapLabel 用来操作按钮的主要标签图片. 你还可以使用 SetBitmapFocus, SetBitmapSelected, 和 SetBitmapDisabled 函数来设置按钮被激活，被按下以及被禁用状态下对应的图片。

SetDefault 将按钮设置为其父窗口的默认按钮，这样用户按回车键就相当于用左键点击了这个按钮.

wxChoice

选择控件由一个只读的文本区域组成，这个区域的文本通过对一个附属的下拉列表框中值进行选择来进行赋值。这个下拉列表框是默认不可见的，只有在用户用鼠标点击下拉按钮以后才会显示。

创建一个选择控件的代码如下，其中的参数含义依次为：父窗口，标识符，位置，大小，窗口类型以及文本选项。

```cpp
#include "wx/choice.h"
wxArrayString strings;
strings.Add(wxT("One"));
strings.Add(wxT("Two"));
strings.Add(wxT("Three"));
wxChoice* choice = new wxChoice(panel, ID_COMBOBOX,
    wxDefaultPosition, wxDefaultSize, strings); 
```

在大多数平台上，除了文本区域的文本是只读的以外，选择控件和 wxComboBox(如下图所示)是非常相似的。在 GTK+平台上，选择控件是一个拥有一 个下拉菜单的按钮。你可以用过设置 wxComboBox 窗口的只读属性来模拟选择控件，以便能够使得在选项过多的时候使用滚动条。

![](img/mhtCB6F%281%29.tmp)

wxChoice 的窗口类型

wxChoice 控件没有特别的窗口类型.

wxChoice 的事件

wxChoice 控件产生 wxCommandEvent 类型的事件，如下表所示：

|  |  |
| --- | --- |
| VT_CHOICE(id,func) | 用于处理 wxEVT_COMMAND_CHOICE_SELECTED 事件,当有用户通过列表选择某个选项的时候产生. |

wxChoice 的成员函数

wxChoice 相关的成员函数都是前面介绍过的 wxControlWithItems 类的函数，如： Clear, Delete, FindString, GetClientData, GetClientObject, SetClientData, SetClientObject, GetCount, GetSelection, SetSelection, GetString, SetString, GetStringSelection, SetStringSelection, Insert, 和 IsEmpty.

wxComboBox

ComboBox 是一个单行编辑框和一个列表框的组合，用来允许用户以和下拉框中的文本没有关系的方式设置或者获取编辑框中的文本。其中 的单行文本域可以是只读的，在这种情况下就和选择控件非常相似了。和选择控件一样，通常列表框是隐藏的，除非用户单击了编辑框旁边的小按钮。这个控件的目 的在提供一种紧凑的方法给用户，使他们既可以自己输入文本，也可以很方便的选择预定义好的文本。

创建一个 ComboBox 的方法和创建选择控件的方法很类似，如下所示：

```cpp
#include "wx/combobox.h"
wxArrayString strings;
strings.Add(wxT("Apple"));
strings.Add(wxT("Orange"));
strings.Add(wxT("Pear"));
strings.Add(wxT("Grapefruit"));
wxComboBox* combo = new wxComboBox(panel, ID_COMBOBOX,
    wxT("Apple"), wxDefaultPosition, wxDefaultSize,
    strings, wxCB_DROPDOWN); 
```

wxComboBox 的窗口类型

| wxCB_SIMPLE | 列表框永远显示. 仅适用于 Windows 平台. | \ |
| --- | --- | --- |
| wxCB_DROPDOWN | 列表框下拉显示. |
| wxCB_READONLY | 和 wxCB_DROPDOWN 的含义相同，只不过编辑框的文本只能通过列表框进行选择，即使在应用程序的代码里，也不允许编辑框中的文本不存在于列表框内。 |
| wxCB_SORT | 列表框中的选项自动按照子母顺序排序. |

wxComboBox 的事件

wxComboBox 产生的是 wxCommandEvent 类型的事件，如下表所示：

| EVT_TEXT(id, func) | 用来处理 wxEVT_COMMAND_TEXT_UPDATED 事件,当编辑框内文本变化时产生. |
| --- | --- |
| EVT_COMBOBOX(id, func) | 用来处理 wxEVT_COMMAND_COMBOBOX_SELECTED 事件，当用户通过列表框选择一个选项的时候产生. |

wxComboBox 的成员函数

除了 wxControlWithItems 的成员函数以外，额外的成员函数还包括：

Copy 函数将编辑框中的文本拷贝到剪贴板， Cut 函数除了作 Copy 函数的动作，还将编辑框中的文本清空， Paste 函数则把剪贴板内的文本复制到编辑框中。

GetInsertionPoint 函数返回当前编辑框中插入点的位置(长整型),SetInsertionPoint 则用来设置这个位置.SetInsertionPointEnd 用来将其设置到编辑框末尾。

GetLastPosition 返回编辑框中的最后位置。

GetValue 函数用来返回编辑框中的文本,SetValue 则用来设置它.如果 combobox 拥有 wxCB_READONLY 类型，则参数中的文本必须在列表框内，否则在发表版本中，这条语句将被忽略，而在调试版本中，则会出现一个告警。

SetSelection 用来设置文本框中的一部分为选中状态，Replace 则将文本框中的某一部分由给定的参数取代。Remove 则移除文本框中的给定部分.

请同时参考 wxControlWithItems 类的这些成员函数: Clear, Delete, FindString, GetClientData, GetClientObject, SetClientData, SetClientObject, GetCount, GetSelection, SetSelection, GetString, SetString, GetStringSelection, SetStringSelection, Insert, 和 IsEmpty.

wxCheckBox

CheckBox 是一个通常拥有两种状态：选中或者未选中的控件。通常情况下，如果是选中的状态，则控件上显示一个小的叉号或者对号。通 常这个控件还包含一个标签，这个标签可以显示在控件的左边，也可以显示在控件的右边。CheckBox 控件甚至还可以有第三个状态，姑且称之为混合状态或 者不确定状态。一个典型的用法是在安装程序中，对于某个组件来说除了要安装和不要安装两种状态以后，还可以有必须安装这种状态。

下面是创建 CheckBox 的例子代码：

```cpp
#include "wx/checkbox.h"
wxCheckBox* checkbox = new wxCheckBox(panel, ID_CHECKBOX,
    wxT("&Check me"), wxDefaultPosition, wxDefaultSize);
checkBox->SetValue(true); 
```

以及在 windows 平台上控件的样子：

![](img/mhtCB81%281%29.tmp)

以及另外的一个三态的 wxCheckBox 第三态的样子：

![](img/mhtCB84%281%29.tmp)

wxCheckBox 的窗口类型

| wxCHK_2STATE | 创建一个二态选择框，这是默认类型. |
| --- | --- |
| wxCHK_3STATE | 创建一个三态选择框. |
| wxCHK_ALLOW_3RD_STATE_FOR_USER | 默认情况下，用户是不能设置第三种状态的，第三种状态只能在程序中通过代码来设置，使用这个类型可使得用户也可以通过鼠标设置第三态. |
| wxALIGN_RIGHT | 使标签显示在选择框的左边. |

wxCheckBox 的事件

wxCheckBox 产生 wxCommandEvent 类型的事件，如下表所示：

|  |  |
| --- | --- |
| EVT_CHECKBOX(id, func) | 用于处理 wxEVT_COMMAND_CHECKBOX_CLICKED 事件, 当 wxCheckBox 的选择状态改变时产生. |

wxCheckBox 的成员函数

SetLabel 和 GetLabel 用来设置选择框的标签文本. 在 Windows 和 GTK+平台上，可以通过"&"前导字符设置快捷键.

GetValue 和 SetValue 用来操作 Bool 型的当前选择状态. 使用 Get3StateValue 和 Set3StateValue 来操作三种状态 wxCHK_UNCHECKED, wxCHK_CHECKED,或 wxCHK_UNDETERMINED.

Is3State 用于检测是否是三态选择框。

IsChecked 用于检测当前是否为选中状态。

wxListBox 和 wxCheckListBox

wxListBox 用来从一组基于 0 索引的字符串列表中选择一个或者多个。这一组备选的字符串列表显示在一个滚动窗口中，选中的文本被高 亮显示.列表框可以是单选的，这种情况下，如果一个选项被选中，以前被选中的选项就自动变为未选中状态，也可以是多选框，这种状态下，对某个选项的点击只 会导致这个选项的选中状态进行切换。

使用下面的代码创建一个列表框：

```cpp
#include "wx/listbox.h"
wxArrayString strings;
strings.Add(wxT("First string"));
strings.Add(wxT("Second string"));
strings.Add(wxT("Third string"));
strings.Add(wxT("Fourth string"));
strings.Add(wxT("Fifth string"));
strings.Add(wxT("Sixth string"));
wxListBox* listBox = new wxListBox(panel, ID_LISTBOX,
    wxDefaultPosition, wxSize(180, 80), strings, wxLB_SINGLE); 
```

在 windows 下的样子：

![](img/mhtCB87%281%29.tmp)

wxCheckListBox 是 wxListBox 的派生类，继承了它的功能，另外它还在每个选项上额外显示一个复选框. 这个类包含在头文件 wx/checklst.h 中,下图演示了 wxCheckListBox 控件在 windows 上的样子：

![](img/mhtCB9A%281%29.tmp)

如果有很多选项要显示，你可以考虑使用 wxVListBox。这是一个虚的列表框类，它用来显示每个选项的方法是你在继承自这个类的派生类中实现的 OnDrawItem 函数和 OnMeasureItem 函数，而它的事件映射宏的格式则和 wxListBox 的一模一样。

wxHtmlListBox 就是一个 wxVListBox 的派生类，它提供了显示复杂选项的一种简单的方法。你需要定义一个 wxHtmlListBox 的派生类，然后在其 OnGetItem 函数中，为每个选项定义一段 HTML 文本，然后 wxHtmlListBox 则按照这段 HTML 文本来显示各个选项。下图演示了光盘例子 samples/htlbox 在 Windows Xp 上的效果，在这个例子中，通过重载 OnDrawSeparator 函数实现不同选项的不同显示。

![](img/mhtCB9D%281%29.tmp)

wxListBox 和 wxCheckListBox 的窗口类型

| wxLB_SINGLE | 单选列表. |
| --- | --- |
| wxLB_MULTIPLE | 多选列表. |
| wxLB_EXTENDED | 扩展选择选项，用户可以通过 Shift 键或者鼠标以及其它一些键盘绑定进行快速多选. |
| wxLB_HSCROLL | 创建水平滚动条如果选项的值过长. Windows only. |
| wxLB_ALWAYS_SB | 总显示垂直滚动条. |
| wxLB_NEEDED_SB | 只在需要的时候显示垂直滚动条. |
| wxLB_SORT | 所有选项自动按照子母顺序排序. |

wxListBox 的 wxCheckListBox 事件

wxListBox 和 wxCheckListBox 产生的事件类型 wxCommandEvent 类型的事件.

| EVT_LISTBOX(id, func) | 用于处理 wxEVT_COMMAND_LISTBOX_SELECTED 事件,当用户选择某个选项的时候产生. |
| --- | --- |
| EVT_LISTBOX_DCLICK(id, func) | 用于处理 wxEVT_COMMAND_LISTBOX_DOUBLECLICKED 事件, 当某个选项被双击的时候产生. |
| EVT_CHECKLISTBOX (id, func) | 用于处理 wxEVT_COMMAND_CHECKLISTBOX_TOGGLED 事件,当 wxCheckListBox 的某个选项的选中状态发生改变的时候产生。 |

wxListBox 成员函数

Deselect 不选则某个选项。 GetSelections 使用 wxArrayInt 类型返回一组选中的索引。 InsertItems 用来插入一组选项，参数可以是个数和 C++的 wxString 数组，以及插入点的组合，也可以是 wxArrayString 对象和插入点的组合.

Selected 用来判断某个选项是否被选中。

Set 用来清除选项并且用参数中的选项组重置选项。参数可以是个数和 C++的 wxString 数组，以及插入点的组合，也可以是 wxArrayString 对象和插入点的组合.

SetFirstItem 将某个选项设置为第一个可见的选项。

SetSelection 和 SetStringSelection 用索引或者字符创值的方式指定某个选项的选中状态。

请同时参考 wxControlWithItems 的下列成员函数: Clear, Delete, FindString, GetClientData, GetClientObject, SetClientData, SetClientObject, GetCount, GetSelection, GetString, SetString, GetStringSelection, Insert, 和 IsEmpty.

wxCheckListBox 的成员函数

除了 wxListBox 的成员函数以外，wxCheckListBox 还另外拥有下面的函数：

Check 函数使用选项索引和一个 bool 值作为参数也控制选项的选中状态。

IsChecked 用来判断某个选项是否为选中状态。

wxRadioBox

Radio Box 用来在一组相关但是互斥的选项中进行选择。通常显示为一个可以拥有一个文本标签的静态框中的一组垂直的或者水平的选项按钮。

这些选项按钮的排列方式取决于构造函数中的两个参数，栏数和方向窗口有关的类型，方向有关的窗口类型包括 wxRA_SPECIFY_COLS（默认值）和 wxRA_SPECIFY_ROWS，举例来说，如果你的 radio box 共有 8 个选项，栏数为 2,方向为 wxRA_SPECIFY_COLS，那么这些选项将会以两列四行的方式显示，而假如方向为 wxRA_SPECIFY_ROWS，则显示为四列两行。

下面演示了怎样创建一个有三栏的 RadioBox:

```cpp
#include "wx/radiobox.h"
wxArrayString strings;
strings.Add(wxT("&One"));
strings.Add(wxT("&Two"));
strings.Add(wxT("T&hree"));
strings.Add(wxT("&Four "));
strings.Add(wxT("F&ive "));
strings.Add(wxT("&Six "));
wxRadioBox* radioBox = new wxRadioBox(panel, ID_RADIOBOX,
    wxT("Radiobox"), wxDefaultPosition, wxDefaultSize,
    strings, 3, wxRA_SPECIFY_COLS); 
```

以及它在 windows 系统中的样子：

![](img/mhtCBB0%281%29.tmp)

wxRadioBox 的窗口类型

wxRadioBox 额外的窗口类型如下表所示：

| wxRA_SPECIFY_ROWS | 横向分栏. |
| --- | --- |
| wxRA_SPECIFIY_COLS | 纵向分栏. |

wxRadioBox 事件

wxRadioBox 产生 wxCommandEvent 类型的事件，如下表所示：

|  |  |
| --- | --- |
| EVT_RADIOBOX(id, func) | 用来处理 wxEVT_COMMAND_RADIOBOX_SELECTED 事件,当用户点击选项的时候产生. |

wxRadioBox 成员函数

Enable 选中（或不选）某个选项.

FindString 查找匹配的选项，返回这个选项的索引或者 wxNOT_FOUND.

GetCount 返回选项的总数。

GetString 和 SetString 用来操作某个选项的标签文本. GetLabel 和 SetLabel 则用来操作整个 radio box 的标签.

GetSelection 返回基于 0 的选中的选项的索引. GetStringSelection 则返回选中的选项的标签文本. SetSelection 和 SetStringSelection 则用来设置对应的选项，通过这两个函数进行设置时不会发送任何事件。

Show 函数用来显示或者隐藏某个特定的选项或者整个 radio box 控件。

wxRadioButton

一个 radio 按钮通常用来表示一组相关但是互斥的选项中的一个，它拥有一个按钮和一个文本标签，这个按钮的外观通常是一个圆形。

radio 按钮有两种状态：选中和未选中。你可以创建一组 radio 按钮来代替前面介绍的 radiobox 控件，这样作的方法是，给第一个 radio 按钮 指定 wxRB_GROUP 类型，然后直到另外一个组被创建，或者没有同级的子控件的时候，这个组才结束，组中的控件既可以是 radio 按钮，也可以是别的 控件。

为什么要代替 radiobox 呢，其中一个原因是有时候你需要使用更复杂的布局，例如，你可能希望给每一个 radio 按钮增加一个额外的描述或者增加一个额外的窗口控件，又或者你不希望在一组 radio 周围显示一个静态的边框等。

![](img/mhtCBB3%281%29.tmp)

下面是创建一组（两个按钮的代码）：

```cpp
#include "wx/radiobut.h"
wxRadioButton* radioButton1 = new wxRadioButton (panel,
    ID_RADIOBUTTON1, wxT("&Male"), wxDefaultPosition,
    wxDefaultSize, wxRB_GROUP);
radioButton1->SetValue(true);
wxRadioButton* radioButton2 = new wxRadioButton (panel,
    ID_RADIOBUTTON2, wxT("&Female"));
//用于水平放置两个按钮的布局控件使用的代码
wxBoxSizer* sizer = new wxBoxSizer(wxHORIZONTAL);
sizer->Add(radioButton1, 0, wxALIGN_CENTER_VERTICAL|wxALL, 5);
sizer->Add(radioButton2, 0, wxALIGN_CENTER_VERTICAL|wxALL, 5);
parentSizer->Add(sizer, 0, wxALIGN_CENTER_VERTICAL|wxALL, 5); 
```

wxRadioButton 的窗口类型

| wxRB_GROUP | 用来标识一组 radio 按钮的开始. |
| --- | --- |
| wxRB_USE_CHECKBOX | 使用 checkbox 的按钮取代 radio 按钮(仅适用于 Palm OS). |

wxRadioButton 的事件

wxRadioButton 产生 wxCommandEvent 类型的事件，如下表所示：

|  |  |
| --- | --- |
| EVT_RADIOBUTTON(id, func) | 用于处理 wxEVT_COMMAND_RADIOBUTTON_SELECTED 事件，当用户点击某个 radio 按钮的时候产生。 |

wxRadioButton 的成员函数

GetValue 和 SetValue 使用 bool 类型来操作 radio 按钮的状态.

wxScrollBar

wxScrollBar 用来单独的增加一个滚动条，这个滚动条和某些窗口自动增加的两个滚动条是有区别的，但是它们处理事件的方式是一样的。滚动条主要有下面四个属性：范围(range),滑块大小，页大小和当前位置。

范围的含义指的是和这个滚动条绑定的窗口的逻辑单位的大小。比如一个表格有 15 行，那么和这个表格绑定的滚动条的范围就可以设置为 15。

滑块大小通常用来反映当前可视部分的大小，还用表格来作为例子，如果因为窗口大小的原因表格只能显示 5 行，那么滚动条的滑块大小就可以设置成 5。如果滑块大小大于或者等于范围，在多数平台上，这个滚动条将自动隐藏。

页大小指的是当滚动条执行翻页命令时需要滚动的单位数目。

当前位置指的是滑块当前所处的位置。

使用下面的代码来创建一个滚动条，其中构造函数的参数含义依次为父窗口，标识符，位置，大小和窗口类型：

```cpp
#include "wx/scrolbar.h"
wxScrollBar* scrollBar = new wxScrollBar(panel, ID_SCROLLBAR,
    wxDefaultPosition, wxSize(200, 20), wxSB_HORIZONTAL); 
```

在 windows 平台上，显示结果如下图所示：

![](img/mhtCBB6%281%29.tmp)

创建一个滚动条以后，可以使用 SetScrollbar 函数来设置它的属性。前面在介绍 wxScrolledWindow 的时候已经介绍这个函数的用法。

wxScrollBar 的窗口类型

| wxSB_HORIZONTAL | 指定滚动条为水平方向. |
| --- | --- |
| wxSB_VERTICAL | 指定滚动条为垂直方向. |

wxScrollBar 的事件

wxScrollBar 产生 wxScrollEvent 类型的事件。你可以使用 EVT_COMMAND_SCROLL...事件映射宏加 窗口标识符来拦截由特定滚动条产生的相关事件，或者使用 EVT_SCROLL...不带窗口标识符的事件映射宏来拦截除了本窗口以外来自其他的窗口的滚动 条事件。使用 EVT_SCROLL(func)可以响应所有的滚动条事件。在附录 I�事件类型和相关宏�中详细列举了所有的滚动事件，你也可以参考手册中 的相关内容。

wxScrollBar 的成员函数

Getrange 返回范围大小.

GetPageSize 返回页大小.通常这个大小和滑块大小相同。

GetThumbPosition 和 SetThumbPosition 用来操作滑块当前位置.

GetThumbLength 返回滑块或者当前可是区域的大小.

SetScrollbar 用来设置滚动条的所有属性.比如滑块位置（逻辑单位）,滑块大小，范围，页大小以及一个可选的 bool 参数用来指示是否立即更新滚动条的显示。

wxSpinButton

wxSpinButton 拥有两个小的按钮用来表示上下或者左右.这个控件通常和一个文本编辑框控件一起使用，以用来增加或者减少某个值。为了方便移植， 应该尽可能使用 wxSpinCtrl 来代替 wxSpinButton,因为不是所有的平台都支持 wxSpinButton.

这个控件（以及 wxSpinCtrl）所支持的值的范围是平台相关的，但是至少在-32768 到 32768 之间，

使用下面的代码来创建一个 wxSpinButton,其中构造函数的参数的含义分别为：父窗口，标识符，位置，大小以及窗口类型：

```cpp
#include "wx/spinbutt.h"
wxSpinButton* spinButton = new wxSpinButton(panel, ID_SPINBUTTON,
    wxDefaultPosition, wxDefaultSize, wxSP_VERTICAL); 
```

在 windows 平台上，显示的结果如下图所示：

![](img/mhtCBC8%281%29.tmp)

wxSpinButton 的窗口类型

下表列出了 wxSpinButton 的窗口类型

| wxSP_HORIZONTAL | spin 按钮为左右方向. 不支持 wxGTK. |
| --- | --- |
| wxSP_VERTICAL | spin 按钮为上下垂直方向. |
| wxSP_ARROW_KEYS | 用户可以使用方向键来改变相关值. |
| wxSP_WRAP | 将最大值和最小值首尾相连，比如最小值的更小的下一个值将是最大值. |

wxSpinButton 的事件

wxSpinButton 产生 wxSpinEvent 类型的事件, 如下表所示:

| EVT_SPIN(id, func) | 用来处理 wxEVT_SCROLL_THUMBTRACK 事件, 当上下（或左右）按钮被点击的时候产生. |
| --- | --- |
| EVT_SPIN_UP(id, func) | 用来处理 wxEVT_SCROLL_LINEUP 事件,当向上（或者向左）的按钮被点击的时候产生. |
| EVT_SPIN_DOWN(id, func) | 用来处理 wxEVT_SCROLL_LINEDOWN 事件,当向下（或者向右）按钮被点击的额时候产生. |

wxSpinButton 的成员函数

GetMax 返回设置的最大值. GetMin 返回设置的最小值. GetValue 和 SetValue 用来操作当前值. SetRange 用来设置最大和最小值.

wxSpinCtrl

wxSpinCtrl 是 wxTextCtrl 和 wxSpinButton 控件的组合。当用户点击 wxSpinButton 的向上或者向下按钮的时候，wxTextCtrl 中的值将会随之变化。用户也可以直接在 wxTextCtrl 中输入他想要的值。

下面的代码用来创建一个值范围在 0 到 100 之间，初始值为 5 的 wxSpinCtrl。

```cpp
#include "wx/spinctrl.h"
wxSpinCtrl* spinCtrl = new wxSpinCtrl(panel, ID_SPINCTRL,
    wxT("5"), wxDefaultPosition, wxDefaultSize, wxSP_ARROW_KEYS,
    0, 100, 5); 
```

在 windows 平台上的显示效果如下图所示：

![](img/mhtCBCB%281%29.tmp)

wxSpinCtrl 的窗口类型

| wxSP_ARROW_KEYS | 用户可以通过方向键改变相关值. |
| --- | --- |
| wxSP_WRAP | 将最大值和最小值首尾相连. |

wxSpinCtrl 事件

wxSpinCtrl 产生 wxSpinEvent 类型的事件，如下表所示. 你也可以使用 EVT_TEXT 事件映射宏来处理其文本框的文本更新事件，处理函数的额参数类型为 wxCommandEvent.

| EVT_SPIN(id, func) | Handles a wxEVT_SCROLL_THUMBTRACK event, generated whenever the up or down arrow is clicked. |
| --- | --- |
| EVT_SPIN_UP(id, func) | Handles a wxEVT_SCROLL_LINEUP event, generated when the up arrow is clicked. |
| EVT_SPIN_DOWN(id, func) | Handles a wxEVT_SCROLL_LINEDOWN event, generated when the down arrow is clicked. |
| EVT_SPINCTRL(id, func) | Handles all events generated for the wxSpinCtrl. |

wxSpinCtrl 成员函数

GetMax 返回设置的最大值. GetMin 返回设置的最小值. GetValue 和 SetValue 用来操作当前值. SetRange 用来设置最大值和最小值.

wxSlider

slider 控件拥有一个滑条和一个滑块，可以通过移动滑条上的滑块来改变控件的当前值.

使用下面的代码创建一个取值范围为 0 到 40,初始位置为 16 的 wxSlider.

```cpp
#include "wx/slider.h"
wxSlider* slider = new wxSlider(panel, ID_SLIDER, 16, 0, 40,
    wxDefaultPosition, wxSize(200, -1),
    wxSL_HORIZONTAL|wxSL_AUTOTICKS|wxSL_LABELS); 
```

在 windows 平台上显示的外观如下所示：

![](img/mhtCBCE%281%29.tmp)

wxSlider 的窗口类型

| wxSL_HORIZONTAL | 显示水平方向的滑条. |
| --- | --- |
| wxSL_VERTICAL | 显示垂直方向的滑条. |
| wxSL_AUTOTICKS | 显示刻度标记. |
| wxSL_LABELS | 显示最大、最小以及当前值的标签. |
| wxSL_LEFT | 对于垂直滑条，将刻度显示在左面. |
| wxSL_RIGHT | 对于垂直滑条，将刻度显示在右面. |
| wxSL_TOP | 对于水平滑条，刻度显示在上面. 默认值为显示在底部. |
| wxSL_SELRANGE | 允许用户通过滑条选择一个范围.仅适用于 Windows. |

wxSlider 的事件

wxSlider 产生 wxCommandEvent 类型的事件,如下表所示,但是如果你希望更好的控制，你可以使用 EVT*COMMAND_SCROLL*...宏，其处理函数的参数类型为 wxScrollEvent，参见附录 I.

|  |  |
| --- | --- |
| EVT_SLIDER(id, func) | 用于处理 wxEVT_COMMAND_SLIDER_UPDATED 事件,当用户移动滑块的时候产生. |

wxSlider 的成员函数

ClearSel 用来在设置了 wxSL_SELRANGE 类型的滑条上清除选择区域. ClearTicks 则用来清除刻度，仅适用于 windows 系统.

GetLineSize 和 SetLineSize 用来操作移动单位，这个移动单位用来在用户使用方向键操作滑块的时候使用. GetPageSize 和 SetPageSize 也用来操作移动单位，这个单位在用户用鼠标点击滑条的任一边的时候使用.

GetMax 返回当前设置的最大值.

GetMin 返回当前设置的最小值.

GetSelEnd 和 GetSelStart 用来返回选择区域的起点和终点; SetSelection 用来设置选择区域的起点和终点. 这些函数都只适用于 Windows.

GetThumbLength 和 SetThumbLength 用来操作滑块的大小.

GetTickFreq 和 SetTickFreq 用来操作滑条上刻度的密度.SetTick 用来设置刻度的位置。这些函数仅适用于 Windows.

GetValue 返回滑条的当前值, SetValue 用来设置滑条的当前值.

SetRange 用来设置滑条的最大值和最小值.

wxTextCtrl

文本控件是用来显示和编辑文本的控件，它支持单行和多行的文本编辑。在某些平台上，支持给文本控件中的文本设置一些简单的格式和风格。在 windows 平台，GTK+平台以及 Mac OS X 平台上通过使用 wxTextAttr 类来设置和获取文本的当前格式。

使用下面的代码创建一个支持多行文本的文本框控件：

```cpp
#include "wx/textctrl.h"
wxTextCtrl* textCtrl = new wxTextCtrl(panel, ID_TEXTCTRL,
    wxEmptyString, wxDefaultPosition, wxSize(240, 100),
    wxTE_MULTILINE); 
```

在 windows 平台上，多行文本框控件的外观如下:

![](img/mhtCBE1%281%29.tmp)

多行文本框允许使用以"\n"分割的一组文本行的方式来处理一段文本，即使在非 Unix 的平台上，使用"\n"作为分割符也是允许的。这使得你可以忽略平 台之间换行符的差异。不过，作为代价，你将不能直接使用那些 GetInsertionPoint 函数或者 GetSelection 函数返回的索引，作为 GetValue 返回的字符串中的索引，因为在前者返回的索引中，操作系统可能会进行了一点点的偏移以便对应上平台使用的"\r\n"换行符，就象 windows 平台上的那样。

如果你想从上面例子的函数的返回值中得到一个子字符串应该怎么办呢？你可以使用 GetRange 函数。它们返回的索引可以被用于它们自 己别的成员函数，比如 SetInsertionPoint 或者 SetSelection。总而言之，不要将用多行文本框的成员函数返回的索引直接用于它的 内容字符串的索引，但是可以将其用于其它成员函数作为参数。

多行文本框支持设置文本格式：你可以为部分文本设置单独的文本颜色和字体。在 windows 平台上，这需要窗口使用 wxTE_RICH 窗口类型。你可以在插入文本之前使用 SetDefaultStyle 函数，或者在插入文本以后使用 SetStyle 来改变已经存在于文本框中的文本的格 式。前者更有效率。

无论在哪种情况下，如果指定的格式中的部分格式是没有被指定的，则默认格式中相应的值将被使用，如果没有默认的格式，那个文本控件自己的属性将会被使用。

在下面的代码中，第二次调用 SetDefaultStyle 不会改变文本的前景颜色（仍然是红色），最后一次调用 SetDefaultStyle 则不会改变文本的背景颜色（仍然是灰色的）。

```cpp
text->SetDefaultStyle(wxTextAttr(*wxRED));
text->AppendText(wxT("Red text\n"));
text->SetDefaultStyle(wxTextAttr(wxNullColour, *wxLIGHT_GREY));
text->AppendText(wxT("Red on gray text\n"));
text->SetDefaultStyle(wxTextAttr(*wxBLUE));
text->AppendText(wxT("Blue on gray text\n")); 
```

wxTextCtrl 的窗口类型

| wxTE_PROCESS_ENTER | 这个控件将会产生 wxEVT_COMMAND_TEXT_ENTER 事件.如果没有设置这个类型，回车键将会或者被空家内部处理，或者被 dialog 窗口用来遍历所有子窗口. |
| --- | --- |
| wxTE_PROCESS_TAB | 这个控件将会在 TAB 键被按下的时候处理 wxEVT_CHAR 事件,否则 TAB 键用来在 dialog 的所有子窗口之间遍历. |
| wxTE_MULTILINE | 支持多行文本. |
| wxTE_PASSWORD | 文本将会以"*"显示. |
| wxTE_READONLY | 文本只读. |
| wxTE_RICH | 在 Windows 下使用富文本编辑控件. 这将允许控件存储超过 64KB 的文本;并且垂直滚动条自动在需要的时候显示.这个类型在别的平台上将被忽略. |
| wxTE_RICH2 | 在 Windows 下使用富文本编辑控件的 2.0 或者 3.0 版本; 垂直滚动条将始终被显示. 在其它平台忽略这个类型. |
| wxTE_AUTO_URL | 高亮显示 URL 字符串，并且在其被点击的时候产生 wxTextUrlEvents 事件. 在 windows 平台上需要设置 wxTE_RICH 类型.仅适用于 Windows 和 GTK+平台. |
| wxTE_NOHIDESEL | 默认情况下，在 windows 平台上，如果文本框当前没有得到焦点，将不会高亮显示文本框中文本的选中部分，使用这个类型可以使其即使在文本框没有获得焦点的时候，被选部分也会高亮显示。其它平台则忽略这个类型。 |
| wxHSCROLL | 显示水平滚动条，这样就不需要文本自动换行了. 在 GTK+平台上无效. |
| wxTE_LEFT | 文本框中的文本左对齐 (默认). |
| wxTE_CENTRE | 文本框中的文本居中对齐. |
| wxTE_RIGHT | 文本框中的文本右对齐. |
| wxTE_DONTWRAP | 等同于 wxHSCROLL 类型. |
| wxTE_LINEWRAP | 如果文本行的长度超出可以显示的部分，则允许在文本行的任何位置换行，目前只支持 wxUniversal 版本. |
| wxTE_WORDWRAP | 如果文本行的长度超出可以显示的部分，则允许在文本行的单词边界位置换行，目前只支持 wxUniversal 版本. |
| wxTE_NO_VSCROLL | Removes the vertical scrollbar. No effect on GTK+. |

wxTextCtrl 的事件

wxTextCtrl 主要产生 wxCommandEvent 类型的事件，如下表所示：

| EVT_TEXT(id, func) | 用于处理 wxEVT_COMMAND_TEXT_UPDATED 事件,文本框内文本值改变的时候产生. |
| --- | --- |
| EVT_TEXT_ENTER(id, func) | 用于处理 wxEVT_COMMAND_TEXT_ENTER 事件,在用户按下回车键的时候产生并且文本框设置了 wxTE_PROCESS_ENTER 窗口类型. |
| EVT_TEXT_MAXLEN(id, func) | 用于处理 wxEVT_COMMAND_TEXT_MAXLEN 事件,当用户试图输入的文本长度超过 SetMaxLength 设置的长度的时候产生.仅适用于 Windows 和 GTK+平台. |

wxTextCtrl 的成员函数

AppendText 将文本添加在文本框最后, WriteText 在当前的插入点插入文本. SetValue 清除文本框中的当前文本然后赋值,赋值后 IsModified 函数返回 False.在所有这些函数中，如果文本框支持多行文本，则可以使 用换行符。需要注意这些函数都会产生文本更新事件.

GetValue 函数返回文本框的所有文本，如果是多行文本类型，则其中可以包含换行符. GetLineText 返回其中一行. GetRange 返回某个位置范围内的文本。

Copy 函数拷贝选中的文本到剪贴板. Cut 除了 Copy 以外还清除选中的文本. Paste 则将剪贴板上的文本替换当前的选中文本，在用户界面刷新事件处理函数中，你还可以使用 CanCopy, CanCut 和 CanPaste 函数。

Clear 清除所有文本.产生文本更新事件。

DiscardEdits 复位内部的�已修改�标记，就好像文本框的文本已经被保存了那样。

EmulateKeyPress 用来模拟按键输入，以便在文本框中进行某些修改.

GetdefaultStyle 和 SetDefaultStyle 用来操作当前的默认文本格式. GetStyle 返回某个位置文本的当前格式,SetStyle 则用来设置某个范围内的文本格式 e.

GetInsertionPoint 和 SetInsertionPoint 函数用来操作新文本的插入点. GetLastPosition 用来返回当前文本的最末位置,SetInsertionPointEnd 用来将插入点设置在文本最末。

GetLineLength 返回某个特定行的字符创长度。

GetNumberOfLines 返回总行数。

GetStringSelection 返回当前选中的文本。如果当前没有选中任何文本，则返回空字符串。GetSelection 返回当前选中部分的索引. SetSelection 则用两个整数参数来设置当前的选中部分。

IsEditable 返回当前控件是否可以被编辑. SetEditable 用来设置控件的可编辑状态以便让其只读或者可编辑. IsModified 当文本框内的文本已经被编辑过的时候返回 True. IsMultiline 用来检测当前文本框是否是多行文本框。

LoadFile 将文件内容读入文本框, SaveFile 将文本框内容存入文件.

PositionToXY 将象素值转换成文本的行号和位置, 而 XYToPosition 则刚好相反。

Remove 删除给定区域的文本，. Replace 则替换给定区域的文本。

ShowPosition 使得文本控件显示包含给定位置的部分。

Undo 撤消最近一次编辑, Redo 重复最近一次编辑. 在某些平台上，这些操作可能什么也不作. 你可以用 CanUndo 和 CanRedo 来测试当前的平台是否支持撤消和重做动作。

wxToggleButton

wxToggleButton 在用户点击以后保持按下状态.换句话说，除了长的象按钮，它其实更可以说是一个 wxCheckBox.

创建 wxToggleButton 的代码如下:

```cpp
#include "wx/tglbtn.h"
wxToggleButton* toggleButton = new wxToggleButton(panel, ID_TOGGLE,
    wxT("&Toggle label"), wxDefaultPosition, wxDefaultSize);
toggleButton->SetValue(true); 
```

下图则显示了其在 windows 平台上的样子：

![](img/mhtCBE4%281%29.tmp)

wxToggleButton 的窗口类型

wxToggleButton 没有特别的窗口类型.

wxToggleButton 事件

wxToggleButton 产生 wxCommandEvent 类型的事件。

|  |  |
| --- | --- |
| EVT_TOGGLEBUTTON(id, func) | 用于处理 wxEVT_COMMAND_TOGGLEBUTTON_CLICKED 事件, 用户点击该按钮的时候产生. |

wxToggleButton 的成员函数

SetLabel 和 GetLabel 用来操作按钮上的标签. 在 windows 和 GTK+平台上你可以使用"&"前导符来指定一个加速键.

GetValue 和 SetValue 用来获取和设置按钮的状态。

# 4.7 静态控件

# 4.7 静态控件

静态控件不响应任何用户输入，只用来显示一些信息或者增加应用程序的美感。

进度条

这是一个水平或者垂直的用来显示进度（通常是时间的进度）的控件。它不产生任何命令事件。下面的代码用来创建一个进度条：

```cpp
#include "wx/gauge.h"
wxGauge* gauge = new wxGauge(panel, ID_GAUGE,
  200, wxDefaultPosition, wxDefaultSize, wxGA_HORIZONTAL);
gauge->SetValue(50); 
```

在 windows 平台上的外观：

![](img/mhtF224%281%29.tmp)

wxGauge 的窗口类型

| wxGA_HORIZONTAL | 水平进度条. |
| --- | --- |
| wxGA_VERTICAL | 垂直进度条. |
| wxGA_SMOOTH | 创建一个光滑的进度条，进度条的每一段之间没有空格. 仅适用于 Windows. |

wxGauge 事件

因为进度条只是用来显示信息，因此不产生任何事件。

wxGauge 成员函数

GetRange 和 SetRange 用来设置进度条的最大值。

GetValue 和 SetValue 用来获取和设置进度条的当前值。

IsVertical 用来检测是否是垂直进度条（否则就是水平的）。

wxStaticText

静态文本控件用来显示一行或者多行的静态文本。

下面的例子创建了一个静态文本控件：

```cpp
#include "wx/stattext.h"
wxStaticText* staticText = new wxStaticText(panel, wxID_STATIC,
  wxT("This is my &static label"),
  wxDefaultPosition, wxDefaultSize, wxALIGN_LEFT); 
```

以及它在 windows 平台上的外观：

![](img/mhtF227%281%29.tmp)

在静态文本控件标签中的前导符"&"，在某些平台（比如 Windows 和 GTK+）上用来定义一个快捷键，通过这个快捷键可以直接访问到下一个非静态的控件。

wxStaticText 的窗口类型

| wxALIGN_LEFT | 标签左对齐. |
| --- | --- |
| wxALIGN_RIGHT | 标签右对齐. |
| wxALIGN_CENTRE | 标签在水平方向上居中对齐. |
| wxST_NO_AUTORESIZE | 默认情况下，静态文本控件会在调用 SetLabel 以后自动改变大小以使得其大小刚好满足标签文本的需要，如果设置了这个类型，则标签不会改变自己的大小。通常这个类型应该和上面的对齐类型一起使用因为如果没有设置这个类型，自动调整大小使得对齐没有任何意义。 |

wxStaticText 的成员函数

GetLabel 和 SetLabel 用户获取和设置文本标签。

wxStaticBitmap

静态图片控件显示一个图片。

使用下面的代码创建静态图片控件。

```cpp
#include "wx/statbmp.h"
#include "print.xpm"
wxBitmap bitmap(print_xpm);
wxStaticBitmap* staticBitmap = new wxStaticBitmap(panel, wxID_STATIC,
  bitmap); 
```

这会在作为父窗口的面板或者对话框上显示一个图片，如下图所示：

![](img/mhtF23A%281%29.tmp)

wxStaticBitmap 的窗口类型

没有特别的窗口类型.

wxStaticBitmap 的成员函数

GetBitmap 和 SetBitmap 用来获取和设置其显示的图片。

wxStaticLine

这个控件用来在其父窗口上显示一个水平或者垂直的长条，以便作为子窗口的静态分割条。

下面是创建 wxStaticLine 的代码：

```cpp
#include "wx/statline.h"
wxStaticLine* staticLine = new wxStaticLine(panel, wxID_STATIC,
    wxDefaultPosition, wxSize(150, -1), wxLI_HORIZONTAL); 
```

以及其在 windows 平台上的外观：

![](img/mhtF23D%281%29.tmp)

wxStaticLine 的窗口类型

| wxLI_HORIZONTAL | 水平长条. |
| --- | --- |
| wxLI_VERTICAL | 垂直长条. |

wxStaticLine 的成员函数

IsVertical 用来检测是否为垂直长条.

wxStaticBox

这个控件用来在一组控件周围显示一个静态的拥有一个可选标签的矩形方框。到目前为止，这个控件不可以作为其它控件的父窗口。它围绕的那些控件是它的的兄弟窗口而非子窗口。它们应该在它后面创建，但是它们拥有同样的父窗口。在将来的版本中，也许会更改这个限制以便它可以同时容纳兄弟窗口和子窗口。

下面是创建一个 wxStaticBox 的例子代码：

```cpp
#include "wx/statbox.h"
wxStaticBox* staticBox = new wxStaticBox(panel, wxID_STATIC,
  wxT("&Static box"), wxDefaultPosition, wxSize(100, 100)); 
```

以及它在 windows 平台上的样子：

![](img/mhtF24F%281%29.tmp)

wxStaticBox 的窗口类型

没有特别的窗口类型

wxStaticBox 的成员函数

GetLabel 和 SetLabel 用来获取和设置其静态标签。

# 4.8 菜单

# 4.8 菜单

在这一小节中，我们来介绍一下怎样使用 wxMenu,它用一种相对简单的办法来提供一组命令但是却不占用大量的空间。在下一小节，我们会描述一下怎样在菜单条上使用菜单。

wxMenu

菜单是指的一串命令，它可以从菜单条弹出，也可以从任何一个窗口上作为关联菜单，通过通常是右键单击来弹出。菜单项可以是一个普通的命令，也可以是一个复选框或者是一个单选框。菜单项可以被禁用，这是它将不能触发任何命令。某些菜单项可以通过一下特殊的三角符号来带出一个新的菜单，菜单中有可以有新的特殊菜单项，这种循环可以使用任意多次。另外一种特殊的菜单项是一个分割条，它只是简单的显示一行或者一段空白以便把两组菜单项进行分割。

下图显示了一个典型的拥有普通菜单项，复选框，单选框以及子菜单的菜单：

![](img/mhtB552%281%29.tmp)

上面的例子还演示了两种快捷操作方法：加速键和快捷键。加速键是通过前导符"&"指定的，使用下划线表示，当菜单显示的时候可以通过这个键来执行相应的命令。而快捷键则是一个组合键，它可以在菜单没有显示的时候执行菜单命令，它通过一个 TAB 加一个组合键被定义。例如，上图中的 New 菜单是通过下面的代码实现的：

```cpp
menu->Append(wxID_NEW, wxT("&New...\tCtrl+N")); 
```

更多通过菜单或者 wxAcceleratorTable 创建快捷键的方法将会在第六章说明。

菜单中的复选框和单选框的状态是由菜单类自动维护的。它们会在单击的时候自动将自己的状态改变，并且在下一次菜单展示的时候以新的状态展示。对于单选框来说，在改变自己状态的同时，它还会将其它同一组中的单选框更改为未选中的状态。你也可以在自己的代码中设置它们的状态（参考第九章）。

你可以通过 wxWindow::PopupMenu 函数将某个菜单在一个窗口的特定位置显示，比如下面的代码：

```cpp
void wxWindow::OnRightClick(wxMouseEvent& event)
{
    if (!m_menu)
    {
        m_menu = new wxMenu;
        m_menu->Append(wxID_OPEN, wxT("&Open"));
        m_menu->AppendSeparator();
        m_menu->Append(wxID_EXIT, wxT("E&xit"));
    }
    PopupMenu(m_menu, event.GetPosition());
} 
```

菜单事件在发送给它的父窗口以及进行其它的事件表匹配之前会首先发送给菜单自己。PopupMenu 函数会导致程序短暂堵塞，直到用户关闭这个菜单。如果你愿意，你可以每次都释放旧的并且重新创建新的菜单，也可以每次都使用同一个菜单。

你应该尽可能的使用系统预定义的菜单标识符，比如 wxID_OPEN, wxID_ABOUT, wxID_PRINT 等等，在第三章中有一个完整的列表。需要特别指出的是，在 Mac OS X 上，标识符为 wxID_ABOUT, wxID_PREFERENCES 和 wxID_EXIT 的菜单项不是显示在你定义的菜单中，而是显示在系统菜单中，wxWidgets 自动为你的菜单作了这个调整。但是你需要注意在调整以后不要留下类似空的帮助菜单，或者是两个菜单分割条连在一起这样的不专业的情况出现。

在 wxWidgets 的发行版的例子中的 samples/menu 例子演示了所有菜单的功能，而在另外一个 samples/ownerdrw 例子中，则演示了怎样在菜单中使用定制的字体和图片。

wxMenu 的事件

wxMenu 相关的事件类型总共有四种: wxCommandEvent, wxUpdateUIEvent, wxMenuEvent, 和 wxContextMenuEvent.

下表列出了处理函数使用 wxCommandEvent 作为参数类型的事件映射宏。可用其来处理菜单命令，无论是弹出菜单命令还是主菜单（来自类似 Frame 窗口的菜单条上的菜单）命令。这种事件宏和工具条上的事件映射宏是一致的，这使得菜单和工具条上的按钮产生的事件可以通过同一个处理函数处理。

| EVT_MENU(id, func) | 用来处理 wxEVT_COMMAND_MENU_SELECTED 事件,某个菜单项被选中的时候产生. |
| --- | --- |
| EVT_MENU_RANGE(id1, id2, func) | 用来处理 wxEVT_COMMAND_MENU_RANGE 事件,在某个范围内的菜单项被选中的时候产生. |

下表列出了处理函数使用 wxUpdateUIEvent 作为参数类型的事件映射宏。这种宏对应的事件是在系统空闲的时候产生的，以便给应用程序一个更新用户界面的机会。例如，允许或者禁止一个菜单项等。尽管 wxUpdateUIEvent 可以被任何窗口产生，但是菜单产生的 wxUpdateUIEvent 还是有一些不同的地方，在于在这个事件中可以调用 Check 函数，SetText 函数和 Enable 函数等。Check 函数用来选中或者不选某个菜单项，而 SetText 函数用来设置菜单项的标签。如果菜单项的标签需要根据某种条件动态改变的话会比较有用。例如：

```cpp
BEGIN_EVENT_TABLE(MyFrame, wxFrame)
    EVT_UPDATE_UI(ID_TOGGLE_TOOLBAR, MyFrame::OnUpdateToggleToolbar)
END_EVENT_TABLE()
void MyFrame::OnUpdateToggleToolbar(wxUpdateUIEvent& event)
{
    event.Enable(true);
    event.Check(m_showToolBar);
    event.SetText(m_showToolBar ?
                  wxT("Show &Toolbar (shown)") :
                  wxT("Show &Toolbar (hidden)"));
} 
```

| EVT_UPDATE_UI(id, func) | 用来处理 wxEVT_UPDATE_UI 事件. 处理函数可以调用 Enable, Check, 和 SetText 以及其它函数. |
| --- | --- |
| EVT_UPDATE_UI_RANGE(id1, id2, func) | 用来处理 wxEVT_UPDATE_UI 事件，以便同时处理一组标识符. |

关于界面更新的更多详情请参考第九章。

下表列出另外一些事件映射宏，其中 EVT_CONTEXT_MENU 的参数类型为 wxContextMenuEvent，这是一个从 wxCommandEvent 继承的事件类型，因此这个事件可以在父子窗口继承关系中传播。使用这个宏来拦截类似鼠标右键单击以产生关联菜单的事件，然后调用事件的 GetPosition 函数来获得菜单应该显示的准确位置。剩下的事件映射宏的处理函数采用 wxMenuEvent 对象作为参数，这些事件只从菜单条发送给其 frame 窗口，来告诉应用程序一个菜单已经被打开或者关闭了，或者某个菜单项正被高亮显示，默认的 EVT_MENU_HIGHLIGHT 的处理函数是在主程序的状态栏显示这个菜单项的帮助信息，不过你可以提供你自己的处理函数来作一些不同的事情。

| EVT_CONTEXT_MENU(func) | 用来处理由于用户或者通过某个特殊按键（在 windows 平台上），或者通过单击鼠标右键产生的弹出一个上下文菜单的请求。处理函数的参数类型为 wxContextMenuEvent. |
| --- | --- |
| EVT_COMMAND_CONTEXT_MENU(id, func) | 和 EVT_CONTEXT_MENU 相似,不过多了一个窗口标识符参数. |
| EVT_MENU_OPEN(func) | 用来处理 wxEVT_MENU_OPEN 事件,在一个菜单即将被打开的时候产生。在 windows 平台上主菜单上每次遍历只会导致一次这个事件产生. |
| EVT_MENU_CLOSE(func) | 用来处理 wxEVT_MENU_CLOSE 事件,它在某个菜单被关闭的时候产生. |
| EVT_MENU_HIGHLIGHT(id, func) | 用来处理 wxEVT_MENU_HIGHLIGHT 事件,当某个菜单项被高亮显示的时候产生，通常用来在主程序的状态栏上产生关于这个菜单项的帮助信息。 |
| EVT_MENU_HIGHLIGHT_ALL(func) | 用来处理 wxEVT_MENU_HIGHLIGHT 事件，在任何一个菜单项被高亮显示的时候产生。 |

wxMenu 的成员函数

Append 增加一个菜单项: 参数为标识符,菜单项标签, 帮助信息和菜单项类型(wxITEM*NORMAL, wxITEM_SEPARATOR, wxITEM_CHECK 或 wxITEM* RADIO). 你也可以使用 AppendCheckItem 和 AppendRadioItem 来避免手动指定 wxITEM_CHECK 或 wxITEM_RADIO 参数类型.例如:

```cpp
// 增加一个普通菜单项
menu->Append(wxID_NEW, wxT("&New...\tCtrl+N"));
// 增加一个复选框菜单项
menu->AppendCheckItem(ID_SHOW_STATUS, wxT("&Show Status"));
// 增加一个单选框菜单项
menu->AppendRadioItem(ID_PAGE_MODE, wxT("&Page Mode"));
Another overload of Append enables you to append a submenu, for example:
// 增加一个子菜单
menu->Append(ID_SUBMENU, wxT("&More options..."), subMenu); 
```

还有一种 Append 的重载函数允许你直接使用一个 wxMenuItem 来增加一个菜单项，这是在菜单中增加图片或者使用自定义字体唯一的方法：

```cpp
// 初始化图片和字体
wxBitmap bmpEnabled, bmpDisabled;
wxFont fontLarge;
// 创建一个菜单项
wxMenuItem* pItem = new wxMenuItem(menu, wxID_OPEN, wxT("&Open..."));
// 设置图片和字体
pItem->SetBitmaps(bmpEnabled, bmpDisabled);
pItem->SetFont(fontLarge);
// 增加到菜单中
menu->Append(pItem); 
```

使用 Insert 函数来在特定的位置插入一个菜单项。使用 Prepend, PrependCheckItem, PrependRadioItem 和 PrependSeparator 在菜单最开始的地方插入一个菜单项。

AppendSeparator 插入一个分割条, InsertSeparator 在特定位置插入一个分割条.

```cpp
// 插入一个分割条
menu->AppendSeparator(); 
```

Break 函数在菜单里插入一个中断点，导致下一个插入的菜单项出现在另外一栏。

使用 Check 函数是标记复选框或者单选框的状态，参数为菜单项标识符和一个 bool 值。使用 IsChecked 函数来获取当前状态.

Delete 函数删除并且释放一个菜单项使用菜单项标识符或者 wxMenuItem 指针。如果这个菜单项是一个子菜单，则子菜单将不被删除。如果你想删除并且释放一个子菜单，使用 Destroy 函数。Remove 函数移除一个菜单项但是并不释放它，而是返回指向它的指针。

使用 Enable 函数来允许或者禁止一个菜单项。但是比直接调用这个方法更好的作法是处理用户界面更新事件（参考第九章）。IsEnabled 函数返回当前的可用状态。

使用 FindItem 函数根据标签或者标识符找到一个菜单项，使用 FindItemByPosition 函数通过位置找到一个菜单项。

GetHelpString 和 SetHelpString 函数用来访问菜单项的帮助信息.当菜单是菜单条的一部分时，高亮显示的菜单项的帮助信息将会显示在状态栏上。

GetLabel 和 SetLabel 用来获取或者设置菜单项的标签.

GetMenuCount 返回菜单项的个数。

GetMenuItems 返回一个 wxMenuItemList 类型的菜单项的列表。

GetTitle 和 SetTitle 用来获取或者设置菜单的可选标题，这个标题通常只在弹出菜单中有意义。

UpdateUI 发送用户界面更新事件到其参数只是的事件处理对象，如果参数为 NULL 则发往菜单的父窗口.这个函数在菜单即将显示之前会被调用，但是应用程序也可以在自己认为需要的时候调用它。

# 4.9 控制条

# 4.9 控制条

控制条提供了一个比较直观而方便的方法来排列多个控件。目前有三种类型的控制条，分别是：菜单条，工具条和状态条，其中菜单条只能在 frame 窗口上使用，工具条和状态条通常也是在 frame 窗口上使用，不过它们也可以作为别的窗口的子窗口。

wxMenuBar

菜单条包含一系列的菜单，显示在 frame 窗口顶部标题栏的下方。你可以通过调用 SetMenuBar 函数覆盖 frame 窗口当前的菜单条，使用下面的代码创建一个菜单条：

```cpp
wxMenuBar* menuBar = new wxMenuBar;
wxMenu* fileMenu = new wxMenu;
fileMenu->Append(wxID_OPEN, wxT("&Open..."), wxT("Opens a file"));
fileMenu->AppendSeparator();
fileMenu->Append(wxID_EXIT, wxT("E&xit"), wxT("Quits the program"));
menuBar->Append(fileMenu);
frame->SetMenuBar(menuBar, wxT("&File")); 
```

上述代码创建了一个只有一个菜单的菜单条，如下图所示：

![](img/mht4E8D%281%29.tmp)

你可以给菜单增加子菜单，也可以在菜单上增加单选框和复选框，还可以给菜单项增加快捷键和加速键（请参考本章前一章节的内容）。

如果你提供了一个帮助字符创，那么默认的 EVT_MENU_HIGHLIGHT 函数会将其显示到 frame 的窗口的状态栏上（如果有的话）。

wxMenuBar 的窗口类型

wxMenuBar 拥有一个 wxMB_DOCKABLE 类型，在 GTK+平台上允许菜单条从主窗口分离出来。

wxMenuBar 事件

参考本章前一小节中的相关内容。

wxMenuBar 成员函数

Append 函数在菜单条的末尾增加一个菜单项，一旦菜单项被成功增加，那么它将被菜单条管理并且在合适的时候被释放。这个函数参数为要增加的菜单以及一个标签。Insert 函数则在给定的位置插入一个菜单。

Enable 函数允许或者禁止给定标识符的菜单项. 使用 IsEnabled 函数判断菜单项是否被允许.

Check 函数选中或者去选中一个复选框或者单选框菜单项. 使用 IsChecked 函数来获得当前的选择状态。

EnableTop 函数允许或者禁止一整个菜单。参数为基于 0 的菜单索引。

FindMenu 使用给定的标签字符串查找某个菜单，其中参数字符串可以带有前导字符也可以不带有前导字符.如果找不到则返回 wxNOT_FOUND.

FindMenuItem 通过菜单名和菜单项返回菜单项的索引。

FindItem 通过给定的菜单项标识符返回 wxMenuItem 类型的菜单项，如果这个菜单项是一个子菜单，那么第二个参数将会返回这个子菜单的指针。

GetHelpString 和 SetHelpString 用来获取或者设置某个菜单项的帮助信息。

GetLabel 和 SetLabel 用来设置某个菜单项的标签，

GetLabelTop 和 SetLabelTop 用来获取和设置某个菜单在菜单条上的标签，参数为基于 0 的索引。

GetMenu 根据索引返回对应菜单指针，

GetMenuCount 返回菜单条上所有菜单的个数，

Refresh 重画菜单条。

Remove 移除一个菜单项并且返回对应的菜单指针，然后用户应该自己释放这个已经移除的菜单。

Replace 函数则将某个位置的菜单使用新的菜单代替并且返回老的菜单的指针。 用户应该自己释放这个老的菜单，

wxToolBar

工具条用来放置各种按钮和控件。工具条可以是水平的也可以是垂直的，其上的按钮可以是单选，复选以及 push 按钮。这些按钮可以显示标签也可以显示图片。如果你使用 wxFrame::CreateToolBar 函数创建工具条，或者使用 wxFrame::SetToolBar 函数将你创建的工具条和 frame 窗口绑定，那么 frame 窗口将会管理这个工具条的位置和大小以及释放，并且工具条的大小也不算作 frame 的客户区大小。。任何其它方法创建的工具条，你都需要自己使用布局控件或者其它方法来来负责这个工具条的位置和大小，

下面是创建工具条以及和 frame 窗口绑定的例子代码：

```cpp
#include "wx/toolbar.h"
#include "open.xpm"
#include "save.xpm"
wxToolBar* toolBar = new wxToolBar(frame, wxID_ANY,
    wxDefaultPosition, wxDefaultSize, wxTB_HORIZONTAL|wxNO_BORDER);
wxBitmap bmpOpen(open_xpm);
wxBitmap bmpSave(save_xpm);
toolBar->AddTool(wxID_OPEN, bmpOpen, wxT("Open"));
toolBar->AddTool(wxID_SAVE, bmpSave, wxT("Save"));
toolBar->AddSeparator();
wxComboBox* comboBox = new wxComboBox(toolBar, ID_COMBOBOX);
toolBar->AddControl(comboBox);
toolBar->Realize();
frame->SetToolBar(toolBar); 
```

在 windows 平台上的外观如下图所示：

![](img/mht4E90%281%29.tmp)

注意调用 Realize 之前，所有位于其上的按钮和控件应该已经被增加到了工具条中，否则工具条上将什么也不会显示。

你可以查看 wxWidgets 的 samples/toolbar 中的例子，来了解怎样更改工具条的方向，增加显示按钮上的标签以及更改按钮的大小等等其它方面的内容。

Windows 平台上工具按钮上的图片的颜色

在 windows 平台上，wxWidgets 试图将工具按钮上的图片的颜色映射到当前桌面风格下的标准颜色。通常来说，亮灰色（light gray）用来表示透明颜色。下表列出了所有这些颜色。事实上，工具按钮上的图片的颜色只需要接近于标准颜色。接近意味着 RGB 三原色的值和标准颜色的值差别在 10 个单位范围内。

| 颜色值 | 颜色名 | 用于 |
| --- | --- | --- |
| wxColour(0, 0, 0) | 黑色 | 深色阴影 |
| wxColour(128, 128, 128) | 深灰 | 亮物体的 3 维边缘的阴面 |
| wxColour(192, 192, 192) | 亮灰 | 3 维物体的表面(按钮背景), 表示透明区域 |
| wxColour(255, 255, 255) | 白色 | 亮物体的 3 维边缘的高亮面 |

对于 16 色的图片来说，这中映射是没有问题的，但是如果你使用的是颜色更丰富的按钮图片甚至真彩图片，那么这种映射可能会让你的按钮上的图片变的非常丑陋，在这种情况下，你可以使用下面的代码禁止这种映射：

```cpp
wxSystemOptions::SetOption(wxT("msw.remap"), 0); 
```

要使用上面的代码，你需要包含"wx/sysopt.h"头文件。

wxToolBar 的窗口类型

| wxTB_HORIZONTAL | 创建水平工具条. |
| --- | --- |
| wxTB_VERTICAL | 创建垂直工具条. |
| wxTB_FLAT | 给工具条一个浮动外观. Windows and GTK+ only. |
| wxTB_DOCKABLE | 使得工具条可以支持浮动和放置. GTK+ only. |
| wxTB_TEXT | 显示按钮上的标签;默认情况下只显示图标. |
| wxTB_NOICONS | 不显示按钮上的图标;默认情况下是显示的. |
| wxTB_NODIVIDER | 指示工具条上没有分割线. Windows only. |
| wxTB_HORZ_LAYOUT | 指示文本显示在图片的旁边而不是下面. 仅适用于 Windows 和 GTK+. 这个类型必须和 wxTB_TEXT 共同使用. |
| wxTB_HORZ_TEXT | 是 wxTB_HORZ_LAYOUT 和 wxTB_TEXT 的组合. |

wxToolBar 的事件

工具条事件映射宏如下表所示。象前面说过的那样，工具条的事件宏和菜单的事件宏是完全一样的，你既可以使用 EVT_MENU 宏，也可以使用 EVT_TOOL 宏。它们的事件处理函数的参数类型都是 wxCommandEvent。对于它们中的大多数来说，窗口标识符指的都是工具按钮的窗口标识符，只有 EVT_TOOL_ENTER 事件宏的窗口标识符指的是工具条的窗口标识符，而对应的工具按钮的窗口标识符要从 wxCommandEvent 事件中获取，这是因为在表示鼠标离开一个工具按钮的时候，窗口标识符可能为-1,而在 wxWidgets 的事件体系中是不允许标识符为-1 的。

| EVT_TOOL(id, func) | 用于处理 wxEVT_COMMAND_TOOL_CLICKED 事件 (和 wxEVT_COMMAND_MENU_SELECTED 的值相同), 在用户点击工具按钮的时候产生. 在宏中使用工具按钮的标识符. |
| --- | --- |
| EVT_TOOL_RANGE(id1, id2, func) | 用于处理一组工具按钮标识符的 wxEVT_COMMAND_TOOL_CLICKED 事件. |
| EVT_TOOL_RCLICKED(id, func) | 用于处理 wxEVT_COMMAND_TOOL_RCLICKED 事件,用户在工具按钮上点击右键的时候产生.使用工具按钮的窗口标识符. |
| EVT_TOOL_RCLICKED_RANGE (id1, id2, func) | 用于处理一组工具按钮标识符的 wxEVT_COMMAND_TOOL_RCLICKED 事件. |
| EVT_TOOL_ENTER(id, func) | 用于处理 wxEVT_COMMAND_TOOL_ENTER 事件, 用户的鼠标移入或者移出某个工具按钮的额时候产生，宏中使用工具条的窗口标识符，使用 wxCommandEvent::GetSelection 函数来获得移入的工具按钮的标识符，如果是移出则这个值是-1 |

wxToolBar 的成员函数

AddTool 增加一个工具按钮:指定按钮的标识符,一个可选的标签,一个图片,一个帮助信息,以及按钮的类型(wxITEM_NORMAL, wxITEM_CHECK, 或者 wxITEM_RADIO). 使用 InsertTool 在某个特定的位置插入一个工具按钮. 也可以使用 AddCheckTool 和 AddRadioTool 来避免指定 wxITEM_CHECK 或 wxITEM_RADIO 类型. AddSeparator 增加一个分割线,基于实现的不同，它可能显示为一条线或者一段空白. 使用 InsertSeparator 在某个特定的位置插入一个分割线,例如下面的代码将增加一个复选框工具按钮，它包含一个标签("Save"), 一个图片，一个帮助字符串 ("Toggle button 1"):

```cpp
toolBar->AddTool(wxID_SAVE, wxT("Save"), bitmap,
                 wxT("Toggle button 1"), wxITEM_CHECK); 
```

AddControl 增加一个控件，比如 combo 框. InsertControl 在某个位置插入一个控件，

DeleteTool 删除给定标识符的工具按钮. DeleteToolByPos 删除指定位置的按钮. RemoveTool 移除给定位置的那个按钮但是并不释放它，而是将它的指针返回给.

EnableTool 用来允许或者禁止某个工具按钮,你可能想参考第九章中的方法，使用用户界面更新事件来更合理的作这个动作。 GetToolEnabled 函数返回某个工具按钮的当前可用状态。

FindById 和 FindControl 来通过窗口标识符寻找某个按钮或者某个控件。

如果你想使用非默认大小 16x15 的图标。你可以使用 SetToolBitmapSize 函数. 使用 GetToolBitmapSize 函数获得当前设置的图片尺寸. GetToolSize 返回当前工具按钮整个的大小。

GetMargins 和 SetMargins 用来获取和设置工具条的左右和上下的边界。

GetToolClientData 和 SetToolClientData 通过窗口标识符获取和设置工具按钮绑定的某个继承自 wxObject 的类.

GetToolLongHelp 和 SetToolLongHelp 用来获取或者设置和工具按钮绑定的长的帮助信息。这个信息可以被显示在状态条或者别的什么位置。GetToolShortHelp 则用来获取或者设置工具按钮的短的帮助信息。

GetToolPacking 和 SetToolPacking 用来获取和设置两排工具按钮之间的间隔。如果是垂直工具条，则为水平工具按钮之间的间隔，如果为水平工具条，则为垂直工具按钮之间的间隔。

GetToolPosition 返回某个由窗口标识符指定的工具按钮在工具条上的位置。

GetToolSeparation 和 SetToolSeparation 用来获取或者设置工具条上分割线的大小。

GetToolState 和 SetToolState 用来获取或者设置某个单选框或者复选框的状态。

Realize 必须在任何按钮被增加以后调用。

ToggleTool 反选某个单选或者复选按钮的状态。

wxStatusBar

状态条是一个狭长的窗口，这个窗口通常被放置在一个 Frame 窗口的底部，用来提供一些状态信息。状态条可以包含一个或多个区域，区域可以拥有固定的或者可变的宽度。如果你使用 wxFrame::CreateStatusBar 或者 wxFrame::SetStatusBar 创建或者将某个状态条和 frame 窗口绑定，那么这个状态条的大小，位置以及其释放都由这个 frame 窗口负责，并且这个状态条的大小也不包含在 frame 窗口的客户区域内，反之，任何其它的方式创建的状态条，这些事情都要由你自己的代码去做。

下面是创建一个状态条的例子代码，这个状态条有三个区域，前两个的大小固定是 60 象素，第三个将使用剩余的区域：

```cpp
#include "wx/statusbr.h"
wxStatusBar* statusBar = new wxStatusBar(frame, wxID_ANY,
    wxST_SIZEGRIP);
frame->SetStatusBar(statusBar);
int widths[] = { 60, 60, -1 };
statusBar->SetFieldWidths(WXSIZEOF(widths), widths);
statusBar->SetStatusText(wxT("Ready"), 0); 
```

这段代码产生的结果如下图所示：

![](img/mht4EA3%281%29.tmp)

如果你想，你甚至可以在状态条的区域里面放置一些小的控件。这需要你自己管理它们的尺寸和大小，例如在你继承自 wxStatusBar 的新类的 size 事件的处理函数中。

wxStatusBar 的窗口类型,注意你可以使用 SetStatusStyles 函数设置某个区域的类型

|:--- |:--- | | wxST_SIZEGRIP | 在状态条的右边显示一个小的修饰. |

wxStatusBar 的事件

没有特别的事件。

wxStatusBar 的成员函数

GetFieldRect 返回某个区域的内部的大小和位置.

GetFieldsCount 返回当前区域的个数. SetFieldsCount 用来设置区域的个数。

GetStatusText 返回当前某个区域的文本, SetStatusText 用来设置某个区域的文本.

PushStatusText 保存当前的区域的文本到一个堆栈中，并且把参数的字符串显示在状态条上. PopStatusText 则将堆栈中最顶层的字符串显示在状态条上。

SetMinHeight 设置状态条最小的合理高度。

SetStatusWidths 采用一个整数数组来设置各个区域的宽度。其中整数代表绝对值，而负数则代表比例，比如说，如果你希望创建一个有三个区域的状态条，其中最右边的区域固定为 100 个象素，左边的两个区域按照 2/3 和 1/3 的比例瓜分剩下的区域，则你应该使用一个包含-2,- 1,100 三个整数的数组作为这个函数的参数。

SetStatusStyles 使用区域个数和一组整数的类型数组来给各个区域设置不同的类型，这些整数的类型决定了区域的外观。其中 wxSB_NORMAL 用来显示标准的拥有三维边界的下沉区域，而 wxSB_FLAT 显示一个没有边框的区域，而 wxSB_RAISED 则显示一个鼓起的拥有三维边框的区域。

# 第四章小结

# 第四章小结

这一章中，我们介绍了足够的关于各种窗口类和控件的知识，相信这足以是你可以开始构建你的各种应用程序。如果你还希望了解这个窗口或者控件的更多的知识，请参考 wxWidgets 的相关手册。对于那些更深入的窗口类，以及怎样创建自己的控件，我们将在第十二章介绍。阅读 wxWidgets 发行版中的各个例子也将是非常有帮助的，比如 samples/widgets, samples/toolbar, samples/text 和 samples/listbox 中的那些例子。

在接下来的一章里，我们将介绍一下应用程序怎样在不同的表面上作画，包括在窗口上，图片上或者是打印机上。