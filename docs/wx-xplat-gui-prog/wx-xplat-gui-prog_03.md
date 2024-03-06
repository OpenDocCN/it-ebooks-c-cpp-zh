# 第三章 事件处理

# 第三章 事件处理

# 3.1 事件驱动编程

# 3.1 事件驱动编程

当程序员们首次面对苹果公司的第一个图形界面个人电脑的时候，他们为这种和以前所有的经验都不同的电脑操作方法感到惊奇。鼠标指针在一个个的窗口之间移来移去，滚动条，菜单，文本编辑框等等等等，真的很难以想象，这么多让人眼花缭乱的东西，其背后的代码该是多么复杂和不可思议。似乎所有这一切都是以完全并行的方式运行的，虽然这只是一个假象。对于很多人来说，苹果个人电脑是他们对事件驱动编程的第一印象。

所有的 GUI 程序都是事件驱动的。换句话说，应用程序一直停留在一个循环中，等待着来自用户或者其他地方（比如窗口刷新或网络连接）的事件，一旦收到某种事件，应用程序就将其扔给处理这个事件的函数。虽然看上去不同的窗口是同时被刷新的，但实际上，绝大多数的 GUI 程序都是单线程的，因此窗口的刷新是依次按顺序进行的。如果由于某种意外你的电脑变得很慢导致窗口刷新的过程变的很明显，你就会注意到这一点。

不同的 GUI 编程架构用不同的方法将它内部的事件处理机制展现给程序开发者。对于 wxWidgets 来说，事件表机制是最主要的方法。在下一小节我们会对此进行进一步的解释。

# 3.2 事件表和事件处理过程

# 3.2 事件表和事件处理过程

wxWidgets 事件处理系统比起通常的虚方法机制来说要稍微复杂一点，但它的一个好处是可以避免需要实现基类中所有的虚方法，因为实现所有的基类虚方法有时候是不切实际的或者是效率很低的。

每一个 wxEvtHandler 的派生类，例如 frame，按钮，菜单以及文档等，都会在其内部维护一个事件表，用来告诉 wxWidgets 事件和事件处理过程的对应关系。所有继承自 wxWindow 的窗口类，以及应用程序类都是 wxEvtHandler 的派生类.

要创建一个静态的事件表(意味着它是在编译期间创建的)，你需要下面几个步骤：

1.  定义一个直接或者间接继承自 wxEvtHandler 的类.
2.  为每一个你想要处理的事件定义一个处理函数。
3.  在这个类中使用 DECLARE_EVENT_TABLE 声明事件表。
4.  在.cpp 文件中使用使用 BEGIN_EVENT_TABLE 和 END_EVENT_TABLE 实现一个事件表。
5.  在事件表的实现中增加事件宏，来实现从事件到事件处理过程的映射。

所有的事件处理函数拥有相同的形式。他们的返回值都是 void，他们都不是虚函数，他们都只有一个事件对象作为参数。（如果你熟悉 MFC，这可能会让你觉得轻松，因为在 MFC 中消息处理函数并没有一个统一的形式。）这个事件对象的类型是随这个处理函数要处理的事件的变化而变化的。例如简单控件（比如按钮）的命令处理函数和菜单命令的处理函数的参数都是 wxCommandEvent 类型，而 size 事件（这个事件通常是由用户改变窗口的客户区尺寸而引起的）处理函数的参数则是 wxSizeEvent 的类型。不同的事件参数类型可以调用的方法也不相同，通过这些方法，你可以获得事件产生的原因以及产生这个事件的控件的值的改变情况（比如，文本框中的值的改变）。当然最简单的情形是你完全不需要访问这个参数的任何方法，比如按钮点击事件。

让我们来扩展一下前一章中的例子，来增加一个窗口大小改变事件的处理和一个确定按钮的处理。下面是扩展以后的 MyFrame 的定义：

```cpp
class MyFrame : public wxFrame
{
public:
    MyFrame(const wxString& title);
    void OnQuit(wxCommandEvent& event);
    void OnAbout(wxCommandEvent& event);
    void OnSize(wxSizeEvent& event);
    void OnButtonOK(wxCommandEvent& event);
private:
    DECLARE_EVENT_TABLE()
}; 
```

增加菜单项的代码和前一章的代码类似，而在 frame 窗口增加一个按钮的代码也只需要在 MyFrame 的构造函数中增加下面的代码：

```cpp
wxButton* button = new wxButton(this, wxID_OK, wxT("OK"),
                                   wxPoint(200, 200)); 
```

类似的，在事件表中也需要相应的增加事件映射宏：

```cpp
BEGIN_EVENT_TABLE(MyFrame, wxFrame)
    EVT_MENU     (wxID_ABOUT,     MyFrame::OnAbout)
    EVT_MENU     (wxID_EXIT,      MyFrame::OnQuit)
    EVT_SIZE     (                MyFrame::OnSize)
    EVT_BUTTON   (wxID_OK,        MyFrame::OnButtonOK)
END_EVENT_TABLE() 
```

当用户点击关于菜单或者退出菜单的时候，这个事件被发送到 frame 窗口，而 MyFrame 的事件表告诉 wxWidgets，对于标识符为 wxID_ABOUT 的菜单事件应该发送到 MyFrame::OnAbout 函数，而标识符为 wxID_EXIT 的菜单事件应该发送到 MyFrame:: OnQuit 函数。换句话说：当事件处理循环处理这两个事件的时候，相应的函数将会以一个的 wxCommandEvent 类型的参数被调用。

EVT_SIZE 事件映射宏不需要标识符参数因为这个事件只会被产生这个事件的控件所处理。

EVT_BUTTON 这一行将导致当 frame 窗口及其子窗口中标识符为 wxID_OK 的按钮被点击的时候，OnButtonOK 函数被调用。这个例子表明，事件可以不必被产生这个事件的控件所处理。让我们假定这个按钮是 MyFrame 的子窗口。当按钮被点击的时候，wxWidgets 会首先检查 wxButton 类是否指定了相应的处理函数，如果找不到，则在其父亲的类所属的事件表中进行查找，在这个例子中，按钮的父亲是 MyFrame 类型的一个实例，在其事件表中指明了一个对应的处理函数，因此 MyFrame::OnButtonOK 函数就被调用了。类似的搜索过程不光发生在窗口控件的父子继承树中，也发生在普通的类继承关系中，这意味着你可以选择在哪里定义事件的处理函数。举例来说，如果你设计了一个对话框，这个对话框需要响应的类似标识符为 wxID_OK 的 command 事件。但是你可能需要把这个控件的创建工作留给使用你的代码的其他的程序员，只要他在创建这个控件的时候使用同样的标识符，你仍然可以给这个控件为这个事件定义默认的处理函数。

下图中演示了一次普通的按钮点击事件发生以后，wxWidgets 搜索所有事件表的顺序。图中只演示了 wxButton 和 MyFrame 两次继承关系。当用户点击了确认按钮的时候，一个新的 wxCommandEvent 事件被创建，其中包含标识符 wxID_OK 和事件类型 wxEVT_COMMAND_BUTTON_CLICKED，然后这个按钮的事件表开始通过 wxEvtHandler::ProcessEvent 函数进行匹配，事件表中的每一个条目都会去尝试匹配，然后是其父类 wxControl 的事件表，然后是 wxWindow 的。如果都没有匹配到， wxWidgets 会搜索其父亲的类事件表，然后就找到了一条匹配条目：

```cpp
EVT_BUTTON(wxID_OK,MyFrame::OnButtonOK) 
```

因此 MyFrame::OnButtonOK 被调用了。

![](img/mht8108%281%29.tmp)

需要注意的事：只有 Command 事件(其事件类型直接或者间接的继承自 wxCommandEvent)才会被递归的应用到其父亲的事件表。通常这是 wxWidgets 的用户经常会感到困惑的地方，因此我们把那些不会传递给其父亲的事件表的事件列举如下：wxActivate, wxCloseEvent, wxEraseEvent, wxFocusEvent, wxKeyEvent, wxIdleEvent, wxInitDialogEvent, wxJoystickEvent, wxMenuEvent, wxMouseEvent, wxMoveEvent, wxPaintEvent, wxQueryLayoutInfoEvent, wxSizeEvent, wxScrollWinEvent, 和 wxSysColourChangedEvent，这些事件都不会传给事件源控件的父亲.

这些事件不会传递给其父亲，是因为这些事件仅对产生这个事件的窗口才有意义，举例来说，把一个子窗口的重绘事件发送给它的父亲，其实是没有任何意义的。

# 3.3 过滤某个事件

# 3.3 过滤某个事件

wxWidgets 事件处理系统实现了一些和 C++中的虚方法非常类似的机制，通过这种机制，你可以通过重载某种基类的事件表的方法来改变基类的默认的事件处理过程。在多数情况下，通过这种方法，你甚至可以改变本地原生控件的默认行为。举例来说，你可以过滤某些按键事件以便本地原生的编辑框控件不处理这些按键。要达到这个目的，你需要实现一个继承自 wxTextCtrl 的新的类，然后在其事件表中使用 EVT_KEY_DOWN 事件映射宏。过滤所有的按键事件也许不是你想要的，这时候，你可以通过调用 wxEvent::Skip 函数来提示事件处理过程对于其中的某些按键事件应该继续寻找其父类的事件表。

总的来说，在 wxWidgets 中，你应该通过调用事件的 Skip 方法，而不是通过显式直接调用其父类对应函数的方法来实现对特殊事件的过滤。

下面的这个例子演示怎样让你自己的文本框控件只接受"a"到"z"和"A"到"Z"的按键，而忽略其它按键的方法：

```cpp
void MyTextCtrl::OnChar(wxKeyEvent& event)
{
    if ( wxIsalpha( event.KeyCode() ) )
    {
       //这些按键在可以接受的范围，所以按照正常的流程处理
       event.Skip();
    }
    else
    {
       // 这些事件不在我们可以接受的范围，所以我们不调用 Skip 函数
       // 由于事件表已经匹配并且没有调用 Skip 函数，所以事件处理过程不会
       // 再继续匹配别的事件表，而是认为事件处理已经结束
       wxBell();
    }
} 
```

# 3.4 挂载事件表

# 3.4 挂载事件表

你并不一定非要实现继承自某个类的新类，才可以改变它的事件表。对于那些继承自 wxWindow 的类来说，有另外一种可取代的方法。你可以通过实现一个新的直接继承自 wxEvtHandler 的新类，然后定义这个新类事件表，然后使用 wxWindow::PushEventHandler 函数将这个事件表压入到某个窗口类的事件表栈中。最后压入的那个事件表在事件匹配过程中将会被最先匹配，如果在其中没有匹配到对应的事件处理过程，那么栈中以前的事件表仍将被匹配，如此类推。你还可以用 wxWindow::PopEventHandler 函数来弹出最顶层的事件表，如果你给 wxWindow:: PopEventHandler 函数传递的是 True 的参数，那么这个弹出的事件表将被删除。

这种方法可以避免大量的类重载，也使不同的类的实例共享同一个事件表成为可能。

有时候，你需要手动调用窗口类的事件表，这时候你应该使用 wxWindow::GetEventHandler 方法，而不是直接使用调用这个窗口类的成员函数。虽然 wxWindow::GetEventHandler 通常返回这个窗口类本身。但是如果你之前曾经使用 PushEventHandler 压入另外一个事件表，那么函数将会返回处于最顶层的事件表。因此使用 wxWindow:: GetEventHandler 函数才可以保证事件被正确的处理。

PushEventHandler 的方法通常用来临时的或者永久的改变图形界面的行为。举例来说，加入你想在你的应用程序实现对话框编辑的功能。你可以捕获这个对话框和它的内部控件的所有的鼠标事件，先使用你自己的事件表处理这些事件，来进行类似拖拽控件，改变控件大小以及移动控件动作。这在联机教学中也是很有用技术。你可以在你自己的事件表中过滤收到的事件，如果是可以接受的，则调用 wxEvent::Skip 函数正常处理。

# 3.5 动态事件处理方法

# 3.5 动态事件处理方法

前面我们讨论的事件处理方法，都是静态的事件表，这也是我们处理事件最常用的方式。接下来，我们来讨论一下事件表的动态处理，也就是说在运行期改变事件表的映射关系。使用这种事件映射方法的原因，可能是你想在程序运行的不同时刻使用不同的映射关系，或者因为你使用的那种语言(例如 python)不支持静态映射，或者仅仅是因为你更喜欢动态映射。因为动态映射的方法可以使你更精确的控制事件表的细节，你甚至可以单独的将事件表中的某一个条目在运行期打开或者关闭，而前面说的 PushEventHandler 和 PopEventHandler 的方法只能针对整个事件表进行处理。除此以外，动态事件处理还允许你在不同的类之间共享事件函数。

和动态事件处理相关的 API 有两个:wxEvtHandler::Connect 和 wxEvtHandler::Disconnect。大多数情况下你不需要手动调用 wxEvtHandler::Disconnect 函数，这个函数将在窗口类被释放的时候自动。

下面我们还用前面的 MyFrame 类来举例说明：

```cpp
class MyFrame : public wxFrame
{
public:
    MyFrame(const wxString& title);
    void OnQuit(wxCommandEvent& event);
    void OnAbout(wxCommandEvent& event);
private:
}; 
```

你可能已经注意到，这次我们没有使用 DECLARE_EVENT_TABLE 来声明一个事件表。为了动态进行事件映射，我们需要在 OnInit 函数中增加下面的代码：

```cpp
frame->Connect( wxID_EXIT,
    wxEVT_COMMAND_MENU_SELECTED,
    wxCommandEventHandler(MyFrame::OnQuit) );
frame->Connect( wxID_ABOUT,
    wxEVT_COMMAND_MENU_SELECTED,
    wxCommandEventHandler(MyFrame::OnAbout) ); 
```

我们传递给 Connect 函数的三个参数分别为菜单标识符，事件标识符和事件处理函数指针。要注意这里的事件标识符 wxEVT_COMMAND_MENU_SELECTED 不同于前面在静态事件表中用于表示事件映射的宏 EVT_MENU,实际上 EVT_MENU 内部也使用了 wxEVT_COMMAND_MENU_SELECTED.EVT_MENU 其实也自动包含了用于对事件处理指针类型强制转换的宏 wxCommandEventHandler()。一般说来，如果事件处理函数的参数类型是 wxXYZEvent,那么其处理函数的类型就应该用 wxXYZEventHandler 宏进行强制转换.

如果我们希望在某个时候中止上面的事件映射，可以使用下面的方法：

```cpp
frame->Disconnect( wxID_EXIT,
    wxEVT_COMMAND_MENU_SELECTED,
    wxCommandEventHandler(MyFrame::OnQuit) );
frame->Disconnect( wxID_ABOUT,
    wxEVT_COMMAND_MENU_SELECTED,
    wxCommandEventFunction(MyFrame::OnAbout) ); 
```

# 3.6 窗口标识符

# 3.6 窗口标识符

窗口标识符是在事件系统中用来唯一确定窗口的整数。事实上，在整个应用程序的范围内，窗口标识符不必一定是唯一的，而只要在某个固定的上下文(比如说，在一个 frame 窗口和它的所有子窗口)内是唯一的就可以了。举例来说：你可以在无数个对话框中使用 wxID_OK 这个标识符，只要在某个对话框内不要重复使用就可以了。

如果在窗口的构造函数中使用 wxID_ANY 作为其标识符，则意味着你希望 wxWidgets 自动为你生成一个标识符。这或者是因为你不关心这个标识符的值，或者是因为这个窗口不需要处理任何事件，或者是因为你将在同一个地方处理所有的事件。如果是最后一种情况，在使用 wxEvtHandler::Connect 函数或者在静态事件表中，你应该使用 wxID_ANY 作为窗口的标识符。wxWidgets 自动创建的标识符是总是一个负数，所以永远不会和用户定义的窗口标识符重复，用户定义的窗口标识符只能是正整数。

下表列举了 wxWidgets 提供的一些标准的标识符。你应该尽可能的使用这些标识符，这是由于下面一些原因。某些系统会给特定的标识符提供一些小图片(例如 GTK+系统上的 OK 和取消按钮)或者提供默认的处理函数(例如自动产生 wxID_CANCEL 事件来响应 Escape 键)。在 Mac OS X 系统上，wxID_ABOUT, wxID_PREFERENCES 和 wxID_EXIT 菜单项也有特别的处理。另外一些 wxWidgets 的控件也会自动处理标识符为 wxID_COPY, wxID_PASTE 或 wxID_UNDO 等的一些菜单或者按钮的命令。

| 标识符名称 | 描述 |
| --- | --- |
| wxID_ANY | 让 wxWidgets 自动产生一个标识符 |
| wxID_LOWEST | 最小的系统标识符值 (4999) |
| wxID_HIGHEST | 最大的系统标识符值 (5999) |
| wxID_OPEN | 打开文件 |
| wxID_CLOSE | 关闭窗口 |
| wxID_NEW | 新建窗口文件或者文档 |
| wxID_SAVE | 保存文件 |
| wxID_SAVEAS | 文件另存为(应该弹出文件位置对话框) |
| wxID_REVERT | 恢复文件在磁盘上的状态 |
| wxID_EXIT | 退出应用程序 |
| wxID_UNDO | 撤消最近一次操作 |
| wxID_REDO | 重复最近一次操作 |
| wxID_HELP | 帮助 (例如对话框上的帮助按钮可以用这个标识符) |
| wxID_PRINT | 打印 |
| wxID_PRINT_SETUP | 打印设置 |
| wxID_PREVIEW | 打印预览 |
| wxID_ABOUT | 显示一个用来描述整个程序的对话框 |
| wxID_HELP_CONTENTS | 显示上下文帮助 |
| wxID_HELP_COMMANDS | 显示应用程序命令 |
| wxID_HELP_PROCEDURES | 显示应用程序过程 |
| wxID_HELP_CONTEXT | 未使用 |
| wxID_CUT | 剪切 |
| wxID_COPY | 复制到剪贴板 |
| wxID_PASTE | 粘贴 |
| wxID_CLEAR | 清除 |
| wxID_FIND | 查找 |
| wxID_DUPLICATE | 复制 |
| wxID_SELECTALL | 全选 |
| wxID_DELETE | 删除 |
| wxID_REPLACE | 覆盖 |
| wxID_REPLACE_ALL | 全部覆盖 |
| wxID_PROPERTIES | 查看属性 |
| wxID_VIEW_DETAILS | 列表框中的按照详细信息方式显示 |
| wxID_VIEW_LARGEICONS | 列表框按照大图标的方式显示 |
| wxID_VIEW_SMALLICONS | 列表框中按照小图标的方式显示 |
| wxID_VIEW_LIST | 列表框中按照列表的的方式显示 |
| wxID_VIEW_SORTDATE | 按照日期排序 |
| wxID_VIEW_SORTNAME | 按照名称排序 |
| wxID_VIEW_SORTSIZE | 按照大小排序 |
| wxID_VIEW_SORTTYPE | 按照类型排序 |
| wxID_FILE1 to wxID_FILE9 | 显示最近使用的文件 |
| wxID_OK | 确定 |
| wxID_CANCEL | 取消 |
| wxID_APPLY | 应用变更 |
| wxID_YES | YES |
| wxID_NO | No |
| wxID_STATIC | 静态文本或者静态图片可以用这个标识符 |
| wxID_FORWARD | 向前 |
| wxID_BACKWARD | 向后 |
| wxID_DEFAULT | 恢复默认设置 |
| wxID_MORE | 显示更多选项 |
| wxID_SETUP | 显示一个设置对话框 |
| wxID_RESET | 重置所有选项 |
| wxID_CONTEXT_HELP | 显示上下文帮助 |
| wxID_YESTOALL | 全部选是 |
| wxID_NOTOALL | 全部选否 |
| wxID_ABORT | 中止当前操作 |
| wxID_RETRY | 重试 |
| wxID_IGNORE | 忽略错误 |
| wxID_UP | 向上 |
| wxID_DOWN | 向下 |
| wxID_HOME | 首页 |
| wxID_REFRESH | 刷新 |
| wxID_STOP | 停止正在进行的操作 |
| wxID_INDEX | 显示一个索引 |
| wxID_BOLD | 加粗显示 |
| wxID_ITALIC | 斜体显示 |
| wxID_JUSTIFY_CENTER | 居中 |
| wxID_JUSTIFY_FILL | 格式 |
| wxID_JUSTIFY_RIGHT | 右对齐 |
| wxID_JUSTIFY_LEFT | 左对齐 |
| wxID_UNDERLINE | 下划线 |
| wxID_INDENT | 缩进 |
| wxID_UNINDENT | 反缩进 |
| wxID_ZOOM_100 | 放大到 100% |
| wxID_ZOOM_FIT | 缩放到整页 |
| wxID_ZOOM_IN | 放大 |
| wxID_ZOOM_OUT | 缩小 |
| wxID_UNDELETE | 反删除 |
| wxID_REVERT_TO_SAVED | 恢复到上次保存的状态 |

为了避免你自己定义的标识符和这些预定义的标识符重复，你可以使用大于 wxID_HIGHEST 的标识符或者小于 wxID_LOWEST 的标识符。

# 3.7 自定义事件

# 3.7 自定义事件

如果你要使用自定义的事件，你需要下面的几个步骤:

1.  从一个合适的事件类派生一个你自己的事件类，声明动态类型信息并且实现一个 Clone 函数，按照你自己的意愿增加新的数据成员和函数成员.如果你希望这个事件在窗口继承关系之间传递，你应该使用的 wxCommandEvent 派生类，如果你希望这个事件的处理函数可以调用 Veto(译者注:某些事件可以调用这个函数来阻止后续可能对这个事件进行的任何操作(如果有的话)，最典型的例子是关闭窗口事件 wxEVT_CLOSE)，你应该使用 wxNotifyEvent 的派生类.
2.  为这个事件类的处理函数定义类型.
3.  定义一个你的事件类支持的事件类型的表。这个表应该定义在你的头文件中。用 BEGIN_DECLARE_EVENT_TYPES()宏和 END_DECLARE_EVENT_TYPES()宏包含起来。其中的每一个支持的事件的声明应该使用 DECLARE_EVENT_TABLE (name, integer)格式的宏.然后在你的.cpp 文件中使用 DEFINE_EVENT_TYPE(name)来实现这个事件类.
4.  为每个你的事件类支持的事件定义一个事件映射宏。

我们还是通过例子来让上面这段绕口的话显的更生动一些。假如我们要实现一个新的控件 wxFontSelectorCtrl,这个控件将可以显示字体的预览。用户通过点击字体的预览来弹出一个对话框让用户可以更改字体。应用程序也许想拦截这个字体改变事件，因此我们在我们的底层鼠标消息处理过程中将会给应用程序发送一个自定义的字体改变事件。

因此我们需要定义一个新的事件 wxFontSelectorCtrlEvent.应用程序可以通过事件映射宏 EVT_FONT_SELECTION_CHANGED(id, func)来增加对这个事件的处理。我们还需要给这个事件定义一个事件类型: wxEVT_COMMAND_FONT_SELECTION_CHANGED. 这样，我们的头文件看上去就象下面的样子:

```cpp
/*!
 * Font selector event class
 */
class wxFontSelectorCtrlEvent : public wxNotifyEvent
{
public:
    wxFontSelectorCtrlEvent(wxEventType commandType = wxEVT_NULL,
      int id = 0): wxNotifyEvent(commandType, id)
    {}
    wxFontSelectorCtrlEvent(const wxFontSelectorCtrlEvent& event): wxNotifyEvent(event)
    {}
    virtual wxEvent *Clone() const
                  { return new wxFontSelectorCtrlEvent(*this); }
DECLARE_DYNAMIC_CLASS(wxFontSelectorCtrlEvent);
};
typedef void (wxEvtHandler::*wxFontSelectorCtrlEventFunction)
                                        (wxFontSelectorCtrlEvent&);
/*!
 * Font selector control events and macros for handling them
 */
BEGIN_DECLARE_EVENT_TYPES()
    DECLARE_EVENT_TYPE(wxEVT_COMMAND_FONT_SELECTION_CHANGED, 801)
END_DECLARE_EVENT_TYPES()
#define EVT_FONT_SELECTION_CHANGED(id, fn) DECLARE_EVENT_TABLE_ENTRY(
 wxEVT_COMMAND_FONT_SELECTION_CHANGED, id, -1, (wxObjectEventFunction) (wxEventFunction)
(wxFontSelectorCtrlEventFunction) & fn,
(wxObject *) NULL ), 
```

而在我们的.cpp 文件中，看上去则象下面的样子:

```cpp
DEFINE_EVENT_TYPE(wxEVT_COMMAND_FONT_SELECTION_CHANGED)
IMPLEMENT_DYNAMIC_CLASS(wxFontSelectorCtrlEvent, wxNotifyEvent) 
```

然后，在我们的新控件的鼠标处理函数中，可以通过下面的方法在检测到用户选择了一个新的字体的时候，发送一个自定义的事件：

```cpp
wxFontSelectorCtrlEvent event(
                    wxEVT_COMMAND_FONT_SELECTION_CHANGED, GetId());
event.SetEventObject(this);
GetEventHandler()->ProcessEvent(event); 
```

现在，在使用我们的新控件的应用程序的代码中就可以通过下面代码来处理我们定义的这个新事件了:

```cpp
BEGIN_EVENT_TABLE(MyDialog, wxDialog)
  EVT_FONT_SELECTION_CHANGED(ID_FONTSEL, MyDialog::OnChangeFont)
END_EVENT_TABLE()
void MyDialog::OnChangeFont(wxFontSelectorCtrlEvent& event)
{
    // 字体已经更改了，可以作一些必要的处理。
    ...
} 
```

上面用到的事件标识符 801 在最新的版本中已经没有用处了，之所以这样写只是为了兼容 wxWidgets2.4 的版本。

让我们再来看一眼事件映射宏的定义：

```cpp
#define EVT_FONT_SELECTION_CHANGED(id, fn) DECLARE_EVENT_TABLE_ENTRY(
 wxEVT_COMMAND_FONT_SELECTION_CHANGED, id, -1, (wxObjectEventFunction) (wxEventFunction)
(wxFontSelectorCtrlEventFunction) & fn,
(wxObject *) NULL ), 
```

这个宏的作用其实是把一个五元组放入到一个数组中，所以这段代码的语法看上去是奇怪了一些，这个五元组的意义分别解释如下：

*   事件类型:一个事件类可以处理多种事件类型，但是在我们的例子中，只处理了一个事件类型，所以就只有一个事件映射宏的记录。这个事件类型必须要和事件处理函数所有处理的事件的类型一致。
*   传递给事件映射宏的标识符:只有当事件的标识符和事件表中的标识符一致的时候，相应的事件处理函数才会被调用。
*   第二标识符。在这里-1 表示没有第二标识符。
*   事件处理函数。如果没有类型的强制转换，一些编译器会报错，这也就是我们要定义事件处理函数类型的原因。
*   用户数据，通常都是 NULL;

随书附带光盘中的 examples/chap03 目录中有一个完整的自定义事件的例子，其中包括了一个字体选择控件和一个简单验证类，你可以在你的应用程序中使用她们。你还可以阅读你的 wxWidgets 发行版目录中的 include/wx/event.h 来获得更多的灵感。

# 第三章小结

# 第三章小结

在这一章里，我们讨论了 wxWidgets 中的事件分发机制，以及怎样进行动态事件处理，还谈到了窗口标识符以及怎样使用自定义的事件。更多关于事件处理的内容，你可以参考附录 H,"wxWidgets 怎样处理事件"以及附录 I"事件处理类和宏",那里列举了主要的事件处理的类和宏。你还可以参考大量的 wxWidgets 的例子来学习事件的用法,尤其是 wxWidgets 发行版目录 samples/event 这个例子。

在下一章里，我们将讨论一系列重要的 GUI 控件以及如何在你的程序中使用这些控件.