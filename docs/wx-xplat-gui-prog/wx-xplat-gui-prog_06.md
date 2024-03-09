# 第六章处理用户输入

所有的 GUI 程序都要以某种方式响应用户的输入，这一章我们来介绍以下在 wxWidgets 中怎样处理来自用户的鼠标，键盘以及游戏手柄的输入.

# 6.1 鼠标输入

总的说来，有两类的鼠标事件。基本的鼠标事件使用 wxMouseEvent 作为参数，不加任何翻译的发送给响应的窗口事件处理函数，而窗口事件处理函数则通常把它们翻译成对应的命令事件（wxCommandEvent）。

举例来说，当你在你的事件表中增加 EVT_BUTTON 事件映射宏的时候，它的处理函数的参数是一个由按钮产生的 wxCommandEvent 类型。而在控件内部，这个 wxCommandEvent 类型是按钮控件对 EVT_LEFT_DOWN 事件宏进行处理并将对应的鼠标事件翻译成 wxCommandEvent 事件的结果。（当然，在大多数平台上，按钮都是使用本地控件实现的，不需要自己处理底层的鼠标事件，但是对于别的定制的类来说，确是如此）

因为我们在前面已经介绍过处理命令事件的方法，我们将主要介绍怎样处理基本的鼠标事件。

你可以分别拦截鼠标左键，中键或者右键的鼠标按下，鼠标释放或者鼠标双击事件。你还可以拦截鼠标的移动事件（无论有没有鼠标按下）。你还可以拦截那些用来告诉你鼠标正在移入或者移出某个窗口的事件，最后，如果这个鼠标有滚轮，你还可以拦截鼠标的滚轮事件。

当你收到一个鼠标事件时，你可以获得鼠标按钮的状态信息，以及象 Shift，Alt 等等这些状态键的信息，你还可以获得鼠标指针相对于当前窗口客户区域左上角的座标值。

下表列出了所有对应的鼠标事件映射宏。需要注意的是，wxMouseEvent 事件是不会传递给父窗口处理的，所以，为了处理这个事件，你必须重载一个新的窗口类，或者重载一个新的 wxEvtHandler，然后将其挂载在某个窗口上，当然你还可以使用动态事件处理函数 Connect，相关内容参见第三章。

| EVT_LEFT_DOWN(func | 用来处理 wxEVT_LEFT_DOWN 事件, 在鼠标左键按下的时候产生. |
| --- | --- |
| EVT_LEFT_UP(func) | 用来处理 wxEVT_LEFT_UP 事件, 在鼠标左键被释放的时候产生. |
| EVT_LEFT_DCLICK(func) | 用来处理 wxEVT_LEFT_DCLICK 事件,在鼠标左键被双击的时候产生. |
| EVT_MIDDLE_DOWN(func) | 用来处理 wxEVT_MIDDLE_DOWN 事件, 在鼠标中键被按下的时候产生. |
| EVT_MIDDLE_UP(func) | 用来处理 wxEVT_MIDDLE_UP 事件,当鼠标中键被释放的时候产生. |
| EVT_MIDDLE_DCLICK(func) | 用来处理 wxEVT_MIDDLE_DCLICK 事件,在鼠标中键被双击的时候产生. |
| EVT_RIGHT_DOWN(func) | 用来处理 wxEVT_RIGHT_DOWN 事件,鼠标右键被按下的时候产生. |
| EVT_RIGHT_UP(func) | 用来处理 wxEVT_RIGHT_UP 事件,鼠标右键被释放的时候产生. |
| EVT_RIGHT_DCLICK(func) | 用来处理 wxEVT_RIGHT_DCLICK 事件,鼠标右键被双击的时候产生. |
| EVT_MOTION(func) | 用来处理 wxEVT_MOTION 事件,鼠标指针移动的时候产生. |
| EVT_ENTER_WINDOW(func) | 用来处理 wxEVT_ENTER_WINDOW 事件,鼠标指针移入某个窗口的时候产生. |
| EVT_LEAVE_WINDOW(func) | 用来处理 wxEVT_LEAVE_WINDOW 事件,鼠标移出某个窗口的时候产生. |
| EVT_MOUSEWHEEL(func) | 用来处理 wxEVT_MOUSEWHEEL 事件,鼠标滚轮滚动的时候产生. |
| EVT_MOUSE_EVENTS(func) | 用来处理所有的鼠标事件. |

处理按钮和鼠标指针移动事件

按钮和指针移动事件是你想要处理的最主要的鼠标事件。

要检测当产生某个事件时状态键的状态，可以使用 AltDown,MetaDown,ControlDown 或者 ShiftDown 等函数.使用 CmdDown 函数来检测 Mac OS X 平台上的 Meta 键或者别的平台上的 Control 键的状态.本章稍后的"状态键变量"小节会对这些函数进行更详细的介绍.

要检测那个鼠标按钮正被按下,你可以使用 LeftIsDown, MiddleIsDown 和 RightIsDown 函数,或者你可以使用 wxMOUSE_BTN_LEFT, wxMOUSE_BTN_MIDDLE, wxMOUSE_BTN_RIGHT 或 wxMOUSE_BTN_ANY 参数来调用 Button 函数.要注意这些函数通常只是反应在事件产生那个时刻鼠标的状态,而不是反应鼠标的状态改变.(译者注:换句话说,两续同样按钮的两个事件中的按钮状态可能是一样的).

在 Mac OS X 上,Command 键被翻译成 Meta,Option 键是 Alt.因为在 Mac 系统上通常使用的是一键鼠标,当用户按下 Control 键点击鼠标的时候将产生右键单击事件.因此在 MacOS 上没有按下 Control 键时进行右键单击这样的事件,除非你正在使用的是一个两键或者三键的鼠标.

你还可以用下面的函数来或者鼠标事件的类型:Dragging (某个键正按下时鼠标移动), Moving (鼠标正在移动而没有鼠标键被按下), Entering, Leaving, ButtonDown, ButtonUp, ButtonDClick, LeftClick, LeftDClick, LeftUp, RightClick, RightDClick, RightUp, ButtonUp 和 IsButton 等.

你可以使用 GetPosition 函数或者 GetX 和 GetY 函数获取鼠标指针当前的设备单位位置,也可以给 GetLogicalPosition 函数传递某个设备上下文参数以便得到对应的逻辑位置.

下面的例子演示了一个涂鸦程序中的鼠标处理过程:

```cpp
BEGIN_EVENT_TABLE(DoodleCanvas, wxWindow)
    EVT_MOUSE_EVENTS(DoodleCanvas::OnMouseEvent)
END_EVENT_TABLE()
void DoodleCanvas::OnMouseEvent(wxMouseEvent& event)
{
    static DoodleSegment *s_currentSegment = NULL;
    wxPoint pt(event.GetPosition());
    if (s_currentSegment && event.LeftUp())
    {
        // 鼠标按钮释放的时候停止当前线段
        if (s_currentSegment->GetLines().GetCount() == 0)
        {
            // 释放线段记录并且释放指针
            delete s_currentSegment;
            s_currentSegment = (DoodleSegment *) NULL;
        }
        else
        {
            // 已经得到一个有效的线段,把它存下来
            DrawingDocument *doc = GetDocument();
            doc->GetCommandProcessor()->Submit(
            new DrawingCommand(wxT("Add Segment"), DOODLE_ADD,
                                    doc, s_currentSegment));
            doc->Modify(true);
            s_currentSegment = NULL;
        }
    }
    else if (m_lastX &gt; -1 && m_lastY &gt; -1 && event.Dragging())
    {
        //正在拖动鼠标,增加一行到当前的线段中
        if (!s_currentSegment)
            s_currentSegment = new DoodleSegment;
        DoodleLine *newLine = new DoodleLine(m_lastX, m_lastY, pt.x, pt.y);
        s_currentSegment->GetLines().Append(newLine);
        wxClientDC dc(this);
        DoPrepareDC(dc);
        dc.SetPen(*wxBLACK_PEN);
        dc.DrawLine( m_lastX, m_lastY, pt.x, pt.y);
    }
    m_lastX = pt.x;
    m_lastY = pt.y;
} 
```

在上面的应用程序中,线段被存在文档类型.当用户使用鼠标左键在窗口上拖拽时,上面的函数增加一个线条到当前的线段中,并且把它画出来, 当用户释放左键的时候,当前的线段被提交到文档类进行处理(文档类是 wxWidgets 的文档视图框架的一部分) ,以便进一步实现文档的重做或者撤消动作,而在窗口的 OnPaint 函数(代码没有被展示) 中,整个文档被重绘.在第十九章"使用文档和视图"中,我们会完整的介绍这个例子.

如果想让这个程序更专业一点,可以在鼠标按下的时候捕获鼠标并且在鼠标释放的时候释放捕获,以便当鼠标左键按下并且移出窗口的时候仍然可以收到鼠标事件.

处理鼠标滚轮事件

当处理鼠标滚轮事件的时候,你可以使用 GetWheelRotation 函数获得滚轮滚过的位置的大小(可能为负数).用这个数除以 GetWheelDelta 以便得到实际滚动行数的值.多数的设备每个 GetWheelDelta 发送一个滚轮事件,但是将来的设备也许会以更快的频率发送事件,因此你需要进行这种计算以便只有在滚轮滚过一整行的时候才滚动窗口,或者如果你可以滚动半行也可以.你还要把用户在控制面板中设置的滚轮每次滚动数量计算进去,这个数目可以通过 GetLinesPerAction 函数获得,要乘以这个值来得到实际用户希望滚动的数量.

另外,鼠标滚轮还可以被设置为每次滚动一页,你需要调用 IsPageScroll 函数来判断是否属于这种情况.

我们来举个例子,下面的代码是 wxScrolledWindow 的默认滚轮处理事件处理函数,其中的变量 m_wheelRotation 对已经滚动的位置进行累加,只有在滚动超过一行的时候才进行滚动.

```cpp
void wxScrollHelper::HandleOnMouseWheel(wxMouseEvent& event)
{
    m_wheelRotation += event.GetWheelRotation();
    int lines = m_wheelRotation / event.GetWheelDelta();
    m_wheelRotation -= lines * event.GetWheelDelta();
    if (lines != 0)
    {
        wxScrollWinEvent newEvent;
        newEvent.SetPosition(0);
        newEvent.SetOrientation(wxVERTICAL);
        newEvent.m_eventObject = m_win;
        if (event.IsPageScroll())
        {
            if (lines &gt; 0)
                newEvent.m_eventType = wxEVT_SCROLLWIN_PAGEUP;
            else
                newEvent.m_eventType = wxEVT_SCROLLWIN_PAGEDOWN;
            m_win->GetEventHandler()->ProcessEvent(newEvent);
        }
        else
        {
            lines *= event.GetLinesPerAction();
            if (lines &gt; 0)
                newEvent.m_eventType = wxEVT_SCROLLWIN_LINEUP;
            else
                newEvent.m_eventType = wxEVT_SCROLLWIN_LINEDOWN;
            int times = abs(lines);
            for (; times &gt; 0; times)
                m_win->GetEventHandler()->ProcessEvent(newEvent);
        }
    }
} 
```

# 6.2 处理键盘事件

键盘事件是由 wxKeyEvent 类表示的.总共有三种不同类型的键盘事件,分别为:键按下,键释放和字符事件.键按下和键释放事件是原始事件,而字符事件是翻译事件,我们马上会描述字符事件,不过在这之前先要清楚,如果一个按键被长时间按下,你通常就收到很多个键按下事件,而只收到一个键释放事件,,因此不要以为一个键释放事件一定对应一个键按下事件,这种想法是错误的.

要想接收到键盘事件,你的窗口必须拥有键盘焦点,这可以通过函数 wxWindow::SetFocus 来设置,比如当鼠标点击窗口的时候可以调用这个函数.

下表列出了对应的事件映射宏

| EVT_KEY_DOWN(func) | 用来处理 wxEVT_KEY_DOWN 事件 (原始按键按下事件). |
| --- | --- |
| EVT_KEY_UP(func) | 用来处理 wxEVT_KEY_UP 事件 (原始的按键释放). |
| EVT_CHAR(func) | 用来处理 wxEVT_CHAR 事件 (已经翻译的按键按下事件). |

接下来描述一下你可以在事件处理函数中使用的处理函数.

要获得按键编码,你可以使用 GetKeyCode 函数(在 Unicode 版本中,你还可以使用 GetUnicodeKeyCode 函数).下表列出了所有的按键编码:

| WXK_BACK | WXK_RIGHT |
| --- | --- |
| WXK_TAB | WXK_DOWN |
| WXK_RETURN | WXK_SELECT |
| WXK_ESCAPE | WXK_PRINT |
| WXK_SPACE | WXK_EXECUTE |
| WXK_DELETE | WXK_SNAPSHOT |
| WXK_INSERT | WXK_START |
| WXK_HELP | WXK_LBUTTON |
| WXK_RBUTTON | WXK_NUMPAD0 |
| WXK_CANCEL | WXK_NUMPAD1 |
| WXK_MBUTTON | WXK_NUMPAD2 |
| WXK_CLEAR | WXK_NUMPAD3 |
| WXK_SHIFT | WXK_NUMPAD4 |
| WXK_CONTROL | WXK_NUMPAD5 |
| WXK_MENU | WXK_NUMPAD6 |
| WXK_PAUSE | WXK_NUMPAD7 |
| WXK_CAPITAL | WXK_NUMPAD8 |
| WXK_PRIOR | WXK_NUMPAD9 |
| WXK_NEXT | WXK_END |
| WXK_MULTIPLY | WXK_HOME |
| WXK_ADD | WXK_LEFT |
| WXK_SEPARATOR | WXK_UP |
| WXK_SUBTRACT | WXK_DECIMAL |
| WXK_PAGEDOWN | WXK_DIVIDE |
| WXK_NUMPAD_SPACE | WXK_F1 |
| WXK_NUMPAD_TAB | WXK_F2 |
| WXK_NUMPAD_ENTER | WXK_F3 WXK_F4 |
| WXK_NUMPAD_F1 | WXK_F5 |
| WXK_NUMPAD_F2 | WXK_F6 |
| WXK_NUMPAD_F3 | WXK_F7 |
| WXK_NUMPAD_F4 | WXK_F8 |
| WXK_NUMPAD_HOME | WXK_F9 |
| WXK_NUMPAD_LEFT | WXK_F10 |
| WXK_NUMPAD_UP | WXK_F11 |
| WXK_NUMPAD_RIGHT | WXK_F12 |
| WXK_NUMPAD_DOWN | WXK_F13 |
| WXK_NUMPAD_PRIOR | WXK_F14 |
| WXK_NUMPAD_PAGEUP | WXK_F15 |
| WXK_NUMPAD_NEXT | WXK_F16 |
| WXK_NUMPAD_PAGEDOWN | WXK_F17 |
| WXK_NUMPAD_END | WXK_F18 |
| WXK_NUMPAD_BEGIN | WXK_F19 |
| WXK_NUMPAD_INSERT | WXK_F20 |
| WXK_NUMPAD_DELETE | WXK_F21 |
| WXK_NUMPAD_EQUAL | WXK_F22 |
| WXK_NUMPAD_MULTIPLY | WXK_F23 |
| WXK_NUMPAD_ADD | WXK_F24 |
| WXK_NUMPAD_SEPARATOR | WXK_NUMPAD_SUBTRACT |
| WXK_NUMLOCK | WXK_NUMPAD_DECIMAL |
| WXK_SCROLL | WXK_NUMPAD_DIVIDE |
| WXK_PAGEUP |

要判断在按键的时候是否有状态键按下,可以使用 AltDown, MetaDown, ControlDown 或者 ShiftDown 函数. HasModifiers 函数在有 Control 或者 Alt 键按下的时候返回 True(不包括 Shift 和 Meta 键).

你可以使用 CmdDown 函数来代替 ControlDown 或者 MetaDown 函数,它在 Mac OSX 上调用 MetaDown 而在别的平台上调用 ControlDown.在接下来的部分会对此进行解释.

GetPosition 函数返回按键的时候的鼠标指针相对于窗口客户区原点的位置.

提示:如果你在键盘事件处理函数中没有调用 event.Skip()函数,对应的字符事件将不会产生.在某些平台上,全局的快捷键也会不起作用.

字符事件处理的例子

下面列出的代码是随书光盘 examples/chap12/thumbnail 目录中 wxThumbnailCtrl 类的事件处理函数:

```cpp
BEGIN_EVENT_TABLE( wxThumbnailCtrl, wxScrolledWindow )
    EVT_CHAR(wxThumbnailCtrl::OnChar)
END_EVENT_TABLE()
void wxThumbnailCtrl::OnChar(wxKeyEvent& event)
{
    int flags = 0;
    if (event.ControlDown())
        flags |= wxTHUMBNAIL_CTRL_DOWN;
    if (event.ShiftDown())
        flags |= wxTHUMBNAIL_SHIFT_DOWN;
    if (event.AltDown())
        flags |= wxTHUMBNAIL_ALT_DOWN;
    if (event.GetKeyCode() == WXK_LEFT ||
        event.GetKeyCode() == WXK_RIGHT ||
        event.GetKeyCode() == WXK_UP ||
        event.GetKeyCode() == WXK_DOWN ||
        event.GetKeyCode() == WXK_HOME ||
        event.GetKeyCode() == WXK_PAGEUP ||
        event.GetKeyCode() == WXK_PAGEDOWN ||
        event.GetKeyCode() == WXK_PRIOR ||
        event.GetKeyCode() == WXK_NEXT ||
        event.GetKeyCode() == WXK_END)
    {
        Navigate(event.GetKeyCode(), flags);
    }
    else if (event.GetKeyCode() == WXK_RETURN)
    {
        wxThumbnailEvent cmdEvent(
            wxEVT_COMMAND_THUMBNAIL_RETURN,
            GetId());
        cmdEvent.SetEventObject(this);
        cmdEvent.SetFlags(flags);
        GetEventHandler()->ProcessEvent(cmdEvent);
    }
    else
        event.Skip();
} 
```

为了代码更简洁,方向键处理时候调用了另外一个单独的函数 Navigate,而回车键则产生了一个更高一级的事件,这个事件可以被使用这个类的应用程序捕获并且处理,对于所有其它的按键,调用 Skip 函数以便应用程序的其它部分可以继续处理.

按键编码翻译

键盘事件提供的是未翻译的按键编码,而字符事件提供的是翻译以后的字符编码,对于未翻译的按键编码来说,子母永远是大些字符,其它字符则是在 WXK_XXX 中定义的字符.而对于已经翻译的按键编码来说,字符的值和同样的按键在一个文本编辑框中被按下以后在编辑框中产生的字符相同.

举个简单的例子,当一个单独的 A 键按下的时候,在 KEY_DOWN 事件中的字符编码是大字子母 A 的 ASCII 码 65,而在相应的字符事件中的字符编码是小写的 ASCII 的 a,编码为 97.换句话说,当 Shift 和 A 键同时被按下时,上述两个事件中的编码是一样的,都是大写的 A(65).

从这个小例子中我们可以清晰的看到,我们可以从键盘按下事件中的键盘编码和 Shift 键状态计算出相应的 ASCII 码,但是通常来说,如果你希望处理的是 ASCII 码,你应该使用字符事件 EVT_CHAR,因为对于非子母按键来说,如何翻译是和键盘布局有关的,只有系统本身才能对按键事件进行很好的翻译.

另外一种自动完成的按键翻译是那种带有 Control 键的翻译:比如说 Ctrl+A,在 KeyDown 事件中,字符编码仍然是 A,但是在字符事件中则为 ASCII 的 1,因为 ASCII 中定义这个组合键的编码为 1.

如果你对在你的系统中这种系统相关的键盘行为感兴趣,可以编译和运行键盘例子程序(在 samples/keyboard 目录中)然后按每个键试一下.

修饰键变量

在 windows 平台上,有 Control 和 Alt 两个修饰键,那个特殊的 window 键表现 Meta 键的行为.在 Unix 平台上,表现 Meta 键的按键是可以配置的(通过运行 xmodmap 来查看和改变现有配置).有时 Numlock 键也会被配置成 Meta 键,这是为什么在 Numlock 键被按下时,按下 Meta 键再按下别的键的时候,HasModifiers 却返回 False 的原因.

在 Mac OSX 平台上,Command 键(上面有一个苹果的标识)被翻译成 Meta 键,而 Option 键被翻译成 Alt 键.

各个平台上修饰键的不同如下表所示,其中 wxWidgets 采用的修饰键名放在第一栏,三个主要平台上对应的键放在后面三栏.

| Modifier | Key on Windows | Key on Unix | Key on Mac |
| --- | --- | --- | --- |
| Shift | Shift | Shift | Shift |
| Control | Control | Control | Control |
| Alt | Alt | Alt | Option | ![](img/mht493A%281%29.tmp) |
| Meta | Windows | (Configurable) | Command | ![](img/mht493D%281%29.tmp) |

因为在 Mac OSX 上使用 Command 键而在别的平台上使用 Control 键,你可以使用 wxKeyEvent 的 CmdDown 函数来判断这个键在不同的平台上是否被按下.

另外除了在键盘事件处理函数中判断一个修饰键是否被按下以外,你还可以使用 wxGetKeyState 函数加上对应的键盘编码作为参数来判断某个键是否被按下,

加速键

加速键是为了实现通过某种组合键来快速执行菜单命令.加速键的处理是在所有的键盘事件(包括字符事件)之后.标准的加速键包括 Ctrl+ O 用来打开一个文件,Ctrl+V 用来把剪贴板上的数据粘贴到应用程序中等.最简单的定义加速键的方法是在菜单项定义函数中使用下面的代码:

```cpp
menu->Append(wxID_COPY, wxT("Copy\tCtrl+C")); 
```

wxWidgets 把"\t"后面的内容翻译为加速键增加到菜单的加速键表中.在上面的例子中,用户按 Ctrl+C 组合键的效果和用户选择这个菜单的效果是完全一样的.

你可以使用 Ctrl,Alt 和 Shift 以及它们的各种组合,然后加一个+号或者-号再跟一个字符或者功能键,比如下面的这些加速键都是合法的加速键: Ctrl+B, G, Shift-Alt-K, F9, Ctrl+F3, Esc 和 Del. 在你的加速键定义中可以使用下面的名字: Del, Back, Ins, Insert, Enter, Return, PgUp, PgDn, Left, Right, Up, Down, Home, End, Space, Tab, Esc 和 Escape. 这些命令是大小写无关的(你想怎样使用大小写都可以).

注意在 Mac OSX 平台上,一个定义为 Ctrl 的加速键实际上代表的是 Command 键.

另外一种设置加速键的方法是使用 wxAcceleratorEntry 对象定义一个加速键表,然后使用 wxWindow:: SetAcceleratorTable 函数将其和某个窗口绑定.每一个 wxAcceleratorEntry 的记录是由一个修饰键比特位值和一个字符或者功能键以及一个窗口标识符组成的,如下所示:

```cpp
wxAcceleratorEntry entries[4];
entries[0].Set(wxACCEL_CTRL,  (int) 'N',     wxID_NEW);
entries[1].Set(wxACCEL_CTRL,  (int) 'X',     wxID_EXIT);
entries[2].Set(wxACCEL_SHIFT, (int) 'A',     wxID_ABOUT);
entries[3].Set(wxACCEL_NORMAL, WXK_DELETE,   wxID_CUT);
wxAcceleratorTable accel(4, entries);
frame->SetAcceleratorTable(accel); 
```

你可以同时使用多个加速键表,也可以混合使用菜单项加速键和加速键表,如果你想给一个菜单项指定多个加速键,这将是非常有用的,因为你不可能在一个菜单项中指定多个加速键.

# 6.3 处理游戏手柄事件

wxJoystick 类让你可以在 windows 平台或者 linux 平台上使用一到两个游戏手柄.典型的使用方法是,你创建 wxJoystick 的一个实例,并且传递给它 wxJOYSTICK1 或者 wxJOYSTICK2 的参数,并且以全局指针保持这个实例.当你需要处理手柄事件的时候,使用 SetCapture 函数和一个窗口指针作为参数,来使的这个窗口收到游戏手柄事件,当你不需要使用手柄的时候,调用 ReleaseCapture 函数移出手柄事件.当然,你完全可以在应用程序初始化的时候调用 SetCapture 函数而在应用程序退出的时候调用 ReleaseCapture 函数,以便在整个应用程序生命周期内处理游戏手柄事件.

在开始描述详细的函数和事件之前,让我们先看一下 wxWidgets 的发行版中 samples/joystick 目录中的例子.在这个例子中,用户可以使用游戏手柄上的某个按钮画线,并且在按下按钮的时候播放一个声音片断.

下面大概列出了应用程序的起始代码,首先,应用程序通过创建一个临时的游戏手柄对象来检查系统是否有安装手柄,如果没有则退出应用程序.然后装载声音文件,并且获取手柄的最大活动范围以便在窗口上画画的时候实现合理的缩放使得画画的区域充满整个绘画窗口.

```cpp
#include "wx/wx.h"
#include "wx/sound.h"
#include "wx/joystick.h"
bool MyApp::OnInit()
{
    wxJoystick stick(wxJOYSTICK1);
    if (!stick.IsOk())
    {
        wxMessageBox(wxT("No joystick detected!"));
        return false;
    }
    m_fire.Create(wxT("buttonpress.wav"));
    m_minX = stick.GetXMin();
    m_minY = stick.GetYMin();
    m_maxX = stick.GetXMax();
    m_maxY = stick.GetYMax();
    // Create the main frame window
    ...
    return true;
} 
```

MyCanvas 是那个用来接收和处理手柄事件的窗口,下面的 MyCanvas 类的实现部分:

```cpp
BEGIN_EVENT_TABLE(MyCanvas, wxScrolledWindow)
    EVT_JOYSTICK_EVENTS(MyCanvas::OnJoystickEvent)
END_EVENT_TABLE()
MyCanvas::MyCanvas(wxWindow *parent, const wxPoint& pos,
    const wxSize& size):
    wxScrolledWindow(parent, wxID_ANY, pos, size, wxSUNKEN_BORDER)
{
    m_stick = new wxJoystick(wxJOYSTICK1);
    m_stick->SetCapture(this, 10);
}
MyCanvas::~MyCanvas()
{
    m_stick->ReleaseCapture();
    delete m_stick;
}
void MyCanvas::OnJoystickEvent(wxJoystickEvent& event)
{
    static long xpos = -1;
    static long ypos = -1;
    wxClientDC dc(this);
    wxPoint pt(event.GetPosition());
    // if negative positions are possible then shift everything up
    int xmin = wxGetApp().m_minX;
    int xmax = wxGetApp().m_maxX;
    int ymin = wxGetApp().m_minY;
    int ymax = wxGetApp().m_maxY;
    if (xmin &lt; 0) {
        xmax += abs(xmin);
        pt.x += abs(xmin);
    }
    if (ymin &lt; 0) {
        ymax += abs(ymin);
        pt.y += abs(ymin);
    }
    // Scale to canvas size
    int cw, ch;
    GetSize(&cw, &ch);
    pt.x = (long) (((double)pt.x/(double)xmax) * cw);
    pt.y = (long) (((double)pt.y/(double)ymax) * ch);
    if (xpos &gt; -1 && ypos &gt; -1 && event.IsMove() && event.ButtonIsDown())
    {
        dc.SetPen(*wxBLACK_PEN);
        dc.DrawLine(xpos, ypos, pt.x, pt.y);
    }
    xpos = pt.x;
    ypos = pt.y;
    wxString buf;
    if (event.ButtonDown())
        buf.Printf(wxT("Joystick (%d, %d) Fire!"), pt.x, pt.y);
    else
        buf.Printf(wxT("Joystick (%d, %d)"), pt.x, pt.y);
    frame->SetStatusText(buf);
    if (event.ButtonDown() && wxGetApp().m_fire.IsOk())
    {
        wxGetApp().m_fire.Play();
    }
} 
```

wxJoystick 的事件

wxJoystick 类产生 wxJoystickEvent 类型的事件,下表列出了所有相关的事件映射宏:

| EVT_JOY_BUTTON(func) | 用来处理 wxEVT_JOY_BUTTON_DOWN 事件,手柄上的某个按钮被按下的时候产生. |
| --- | --- |
| EVT_JOY_BUTTON(func) | 用来处理 wxEVT_JOY_BUTTON_UP 事件,某个按钮被释放的时候产生. |
| EVT_JOY_MOVE(func) | 用来处理 wxEVT_JOY_MOVE 事件,手柄在 X-Y 平面上产生移动的时候产生. |
| EVT_JOY_ZMOVE(func) | 用来处理 wxEVT_JOY_ZMOVE 事件,手柄在 Z 方向上产生移动的时候产生. |
| EVT_JOYSTICK_EVENTS(func) | 处理所有的手柄事件. |

wxJoystickEvent 的成员函数

下面列出了所有 wxJoystickEvent 的成员函数,你可以使用它们获取关于手柄事件更详细的信息,当然,首先还是 GetEventType 函数,这个函数在你使用 EVT_JOYSTICK_EVENTS 宏的时候告诉你你正在处理哪种类型的手柄事件.

调用 ButtonDown 函数来检测是否是按钮被按下事件,你可以传递可选的 wxJOY_BUTTONn(n 代表 1,2,3 或 4)来测试是否某个特定的按钮正被按下,或者使用 wxJOY_BUTTON_ANY 如果你不关心具体是哪个按钮被按下的话.ButtonUp 则用相似的方法来检测是按钮被释放事件.而 IsButton 则相当于调用 ButtonDown() || ButtonUp().

要判断是否某个按钮被按下的状态而不是事件本身,你可以使用 ButtonIsDown 函数,它和 ButtonDown 的参数相似.或者你可以直接使用 GetButtonState 函数和类似的参数来返回指定按钮的按下状态的比特位.

使用 IsMove 函数来判断是否是一个 XY 平面的移动事件,使用 IsZMove 判断是否是一个 Z 平面的移动事件.

GetPosition 返回游戏手柄当前 XY 平面的座标,而 GetZPosition 返回 Z 方向的深度值(当然,如果手柄支持的话).

最后,你可以使用 GetJoystick 来判断这个事件是哪个手柄(wxJOYSTICK1 还是 wxJOYSTICK2)产生的.

wxJoystick 成员函数

我们没有列出游戏手柄类所有的成员函数,你可以参考 wxWidgets 的手册,不过下面是其中最有趣的几个:

正象我们在前面的例子中看到的那样,SetCapture 函数需要被调用如果你想让某个窗口处理手柄事件,而一个匹配的 ReleaseCapture 函数调用用来释放这次捕获,以便允许别的应用程序使用手柄.为避免别的程序正在使用手柄,或者手柄不能正常工作,你应该在调用 SetCapture 之前先调用 IsOK 函数来判断手柄的状态.你还可以通过这些函数来获取手柄的能力集:GetNumberButtons, GetNumberJoysticks,GetNumberAxes,HasRudder 等等.

GetPosition 函数和 GetButtonState 函数可以让你在手柄事件处理函数以外的地方获取手柄的状态.

你的应用程序还可能需要调用 GetXMin,GetXMax 等这些类似的函数来判断一个手柄支持的范围.

# 第六章小结

在这一章里,我们介绍了怎样处理来自鼠标,键盘以及游戏手柄的输入事件,现在你可以给你的应用程序增加复杂的交互性代码了.你可以参考 wxWidgets 自带的例子比如: samples/keyboard, samples/joytest 和 samples/dragimag,也可以参考光盘上的 examples/chap12 目录里的 wxThumbnailCtrl 类来了解更具体的信息.

在接下来的一章里,我们会介绍一下怎样对窗口里的控件进行可变大小的,轻便的,可以友善转换的布局.之所以能支持这些诱人的特性,是因为我们有强大而灵活的布局控件.