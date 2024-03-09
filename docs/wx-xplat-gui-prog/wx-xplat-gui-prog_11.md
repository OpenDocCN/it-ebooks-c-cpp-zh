# 第十一章剪贴板和拖放操作

大多数应用程序通常都会从剪贴板通过拷贝,剪切和粘贴的动作来实现数据的传输.这是你的应用程序和别的应用程序进行交互的一种最基本的方式,大多数老道的应用程序还会允许使用者在一个程序内部的窗口之间或者是不同的应用程序之间进行拖放操作.比如把一个文件从资源管理器拖放到应用程序窗口,以便这个窗口直接填充这个文件相关的数据.或者是将这个文件名添加到某个列表中,或者执行别的什么行为.这种操作比起通过菜单打开一个对话框来选择文件的方式要快捷的多,你的用户将会很喜欢这样的操作方式.

剪贴板和拖放操作在 wxWidgets 中共享了一些类,这主要是因为他们的主要任务都是实现数据传输,因此本章把这两个操作放在一起操作.我们将会看到怎样使用 wxWidgets 提供的标准数据传输控件,同时也会学习到怎样实现自己的数据传输控件.

# 11.1 数据对象

wxDataObject 类是剪贴板操作和拖放操作的核心,这个类的实例代表了拖放操作中鼠标拖拽的事物,以及剪贴板操作中拷贝和粘贴的事物. wxDataObject 是一块聪明的数据,因为它知道哪种格式它可以支持(通过 GetFormatCount 和 GetAllFormats),也知道怎样来支持它们(通过 GetdataHere).如果实现了对应的 SetData 函数,它也可以从应用程序的外部接受不同格式的数据.我们将在本章的后边对此进行介绍.

标准的数据格式,比如 wxDF_TEXT 是用整数来区分的,而定制的数据格式则是通过字符串来区分的.wxDataFormat 类支持使用这两种参数的构造函数.下表列出了标准的数据格式.

| wxDF_INVALID | 无效格式,用于缺省的 wxDataFormat 的构造函数. |
| --- | --- |
| wxDF_TEXT | 文本数据格式,对应的数据类型为 wxTexTDataObject. |
| wxDF_BITMAP | 图片数据格式,对应的数据类型为 wxBitmapDataObject. |
| wxDF_METAFILE | 元文件数据格式,对应的数据类型为 wxMetafileDataObject(仅支持 Windows) |
| wxDF_FILENAME | 文件名列表数据格式,对应的数据类型为 wxFileDataObject. |

你也可以创建定制的数据格式,在这种情况下,你需要给 wxDataFormat 构造函数传递一个定制的字符串,来标识你的定制的数据类型,这个数据类型将在首次使用的时候被登记.

剪贴板操作和拖放操作都需要处理数据源(数据提供者)和数据目标(数据接收者).它们可以位于同一个应用程序内,甚至是位于同一个窗口内,比如,你在同一个窗口内把一段文本从一个位置拖到另一个位置,我们来分别描述一下这两个角色.

数据源的职责

数据源负责创建要包含要传输的数据的数据对象,在创建数据对象以后,数据源还负责通过 SetData 函数将其传递给剪贴板,或者在拖放操作开始时,通过 DoDragDrop 函数将其传递给一个 wxDropSource 对象.

在这种情况下,剪贴板操作和拖放操作的最大的不同在于剪贴板传输的数据必须使用 new 函数,在堆上创建,而且只能被剪贴板在其不被需要的时候释放,事实上,我们根本不知道它是在什么时候被释放的,我们甚至连原始的数据是什么时候被放到剪贴板上去的也不知道.而另一方面,用于拖放操作的数据对象只需要在 DoDragDrop 执行期间存在,执行完以后就可以被安全地释放了,因此,这种数据对象,即可以在堆上创建,也可以在栈上创建(意思就是一个局部变量).

另一个细微的差别在于:对于剪贴板操作应用程序通常很清楚它正在进行的操作的整个过程.当进行了剪切操作的时候,数据被首先拷贝到剪切板,然后从当前操作的对象中移除.这通常是由于用户对菜单项的选择来触发的.但是对于拖放操作来说,应用程序只有在 DoDragDrop 函数执行以后,才能了解这些信息.

数据目标的职责

要从剪贴板接收数据(意味着一个粘贴操作),你应该首先创建一个支持你想要获取的数据格式的 wxDataObject 的派生类,以便将其传递给 wxClipboard::GetData 函数.如果这个函数返回失败,表明剪贴板上没有你想要的类型的数据.如果返回成功,则表明剪贴板上的数据已经被成功地传输到你创建的 wxDataObject 的派生类中.

对于拖放操作,当一个数据对象被放置的时候,wxDropTarget::OnData 虚函数将会被调用.如果数据类型合适,应用程序可以调用 wxDropTarget::OnData 函数来获取相应的数据.

# 11.2 使用剪贴板

要使用剪贴板,你主要是在调用全局指针 wxTheClipboard 的成员函数. 在进行拷贝或者粘贴的动作之前,你必须先通过 wxClipboard::Open 获得剪贴板的控制权,如果这个函数返回成功,你将已经获得了剪贴板的控制权,可以调用 wxClipboard::SetData 来将数据拷贝到剪贴板上,或者调用 wxClipboard::GetData 函数从剪贴板上获取数据.最后,你需要调用 wxClipboard::Close 函数来释放剪贴板的控制权.一旦你不使用剪贴板了,就应该尽快释放掉剪贴板的控制权.

wxClipboardLocker 类可以在其构造函数中获得剪贴板的控制权(如果可以的话),并且在其析构函数中释放剪贴板的控制权,因此,你可以使用下面这样的代码:

```cpp
wxClipboardLocker locker;
if (!locker)
{
   ... 报告错误然后返回 ...
}
... 使用剪贴板 ...

下边的代码演示了怎样将文本拷贝到剪贴板以及怎样从剪贴板读取文本数据:

// 拷贝一些文本到剪贴板
if (wxTheClipboard->Open())
{
    // 数据对象将被剪贴板释放,
    // 因此不在要你的应用程序中释放它们.
    wxTheClipboard->SetData(new wxTextDataObject(wxT("Some text")));
    wxTheClipboard->Close();
}

// 从剪贴板获取一些文本
if (wxTheClipboard->Open())
{
    if (wxTheClipboard->IsSupported(wxDF_TEXT))
    {
        wxTextDataObject data;
        wxTheClipboard->GetData(data);
        wxMessageBox(data.GetText());
    }
    wxTheClipboard->Close();
}

下边是一个操作图片的例子:

// 将一副图片拷贝到剪贴板
wxImage image(wxT("splash.png"), wxBITMAP_TYPE_PNG);
wxBitmap bitmap(image.ConvertToBitmap());
if (wxTheClipboard->Open())
{
    // 数据对象将被剪贴板释放,
    // 因此不在要你的应用程序中释放它们.
    wxTheClipboard->SetData(new wxBitmapDataObject(bitmap));
    wxTheClipboard->Close();
}

// 从剪贴板读取一幅图片
if (wxTheClipboard->Open())
{
    if (wxTheClipboard->IsSupported(wxDF_BITMAP))
    {
        wxBitmapDataObject data;
        wxTheClipboard->GetData( data );
        bitmap = data.GetBitmap();
    }
    wxTheClipboard->Close();
} 
```

如果你使用了剪贴板操作你可能需要即时更新你的用户界面,比如在剪贴板拥有或者失去相关数据的时候,允许或者禁用相关的菜单项,工具条上的按钮以及一些普通的按钮.这个工作是通过 wxWidgets 的界面更新机制来完成的.在合适的时候 wxWidgets 将会给你的应用程序发送 wxUpdateUIEvent 事件,详情请参考第九章"创建自己自定义的对话框".这个事件允许你在系统空闲的时候根据剪贴板的数据来更新你的用户界面.

某些控件,比如 wxTextCtrl 已经实现了用户界面的自动更新.如果你的菜单项或者工具条使用了标准的标识符 wxID_CUT, wxID_COPY, 和 wxID_PASTE,并且指定了命令事件将首先被活动的控件处理,那么对应的控件将会自动按照用户的预期来进行界面更新.参考第二十章"优化你的应用程序"来学习怎样通过重载 wxFrame::ProcessEvent 函数将命令事件传递到当前激活的控件.

# 11.3 实现拖放操作

你可以在你的应用程序中实现拖放源,拖放目标或者两者同时实现.

实现拖放源

要实现一个拖放源,也就是说要提供用户用于拖放操作的数据,你需要使用一个 wxDropSource 类的实例.要注意下面描述的事情都是在你的应用程序已经认定一个拖放操作已经开始以后发生的.决定拖放是否开始的逻辑,是完全需要由应用程序自己决定的,一些控件会通过产生一个拖放开始事件来通知应用程序, 在这种情况下,你不需要自己关心这一部分的逻辑(对这一部分的逻辑的关心,可能会让你的鼠标事件处理代码变得混乱).在本章中,也会提供一个何时 wxWidgets 通知你拖放操作开始的大概描述.

一个拖放源需要采取的动作包括下面几步:

1 准备工作

首先,必须先创建和初始化一个将被拖动的数据对象,如下所示:

```cpp
wxTextDataObject myData(wxT("This text will be dragged.")); 
```

2 开始拖动要开始拖动操作,最典型的方式是响应一个鼠标单击事件,创建一个 wxDropSource 对象,然后调用它的 wxDropSource::DoDragDrop 函数,如下所示:

```cpp
wxDropSource dragSource(this);
dragSource.SetData(myData);
wxDragResult result = dragSource.DoDragDrop(wxDrag_AllowMove); 
```

下表列出的标记,可以作为 DoDragDrop 函数的参数:

| wxDrag_CopyOnly | 只允许进行拷贝操作. |
| --- | --- |
| wxDrag_AllowMove | 允许进行移动操作. |
| wxDrag_DefaultMove | 默认操作是移动数据. |

当创建 wxDropSource 对象的时候,你还可以指定发起拖动操作的窗口,并且可以选择拖动使用的光标,可选的范围包括拷贝,移动以及不能释放等.这些光标在 GTK+上是图标,而在别的平台上是光标,因此你需要使用 wxDROP_ICON 来屏蔽这种区别,正如我们很快就将看到的拖动文本的例子中演示的那样.

3 拖动

对 DoDragDrop 函数的调用将会阻止应用程序进行其他处理,直到用户释放鼠标按钮(除非你重载了 GiveFeedback 函数以便进行其他的特殊操作)当鼠标在应用程序的一个窗口上移动时,如果这个窗口可以识别这个拖动操作协议,对应的 wxDropTarget 函数就会被调用,参考接下来的小节"实现一个拖放目的"

4 处理拖放结果

DoDragDrop 函数返回一个拖放操作的结果,这个返回值的类型为 wxDragResult,它的枚举值如下表所示:

| wxDragError | 在拖动操作的执行过程中出现了错误. |
| --- | --- |
| wxDragNone | 数据源不被拖动目的接受. |
| wxDragCopy | 数据已经被成功拷贝. |
| wxDragMove | 数据已经被成功移动(仅适用于 Windows). |
| wxDragLink | 以完全一个链接操作. |
| wxDragCancel | 用户已经取消了拖放操作. |

你的应用程序可以针对不同的返回值进行自己的操作,如果返回值是 wxDragMove,通常你需要删除绑定在数据源中的数据,然后更新屏幕显示.而如果返回值是 wxDragNone,则表示拖动操作已经被取消了.下面举例说明:

```cpp
switch (result)
{
    case wxDragCopy: /* 数据被拷贝或者被链接:
                          无需特别操作 */
    case wxDragLink:
        break;
    case wxDragMove: /* 数据被移动,删除原始数据 */
        DeleteMyDraggedData();
        break;
    default:         /* 操作被取消或者数据不被接受
                         或者发生了错误:
                         不做任何操作 */
        break;
} 
```

下面的例子演示了怎样实现一个文本数据拖放源.DnDWindow 包含一个 m_strText 成员变量,当鼠标左键按下的时候,针对 m_strText 的拖放操作开始,拖放操作的结果通过一个消息框显示.另外,拖放操作将会在鼠标已经拖动了一小段距离后才会开始,因此单击鼠标动作并不会导致一个拖放操作.

```cpp
void DnDWindow::OnLeftDown(wxMouseEvent& event )
{
    if ( !m_strText.IsEmpty() )
    {
        // 开始拖动操作
        wxTextDataObject textData(m_strText);
        wxDropSource source(textData, this,
                            wxDROP_ICON(dnd_copy),
                            wxDROP_ICON(dnd_move),
                            wxDROP_ICON(dnd_none));
        int flags = 0;
        if ( m_moveByDefault )
            flags |= wxDrag_DefaultMove;
        else if ( m_moveAllow )
            flags |= wxDrag_AllowMove;
        wxDragResult result = source.DoDragDrop(flags);
        const wxChar *pc;
        switch ( result )
        {
            case wxDragError:   pc = wxT("Error!");    break;
            case wxDragNone:    pc = wxT("Nothing");   break;
            case wxDragCopy:    pc = wxT("Copied");    break;
            case wxDragMove:    pc = wxT("Moved");     break;
            case wxDragCancel:  pc = wxT("Cancelled"); break;
            default:            pc = wxT("Huh?");      break;
        }
        wxMessageBox(wxString(wxT("Drag result: ")) + pc);
    }
} 
```

实现一个拖放目的

要实现一个拖放目的,也就是说要接收用户拖动的数据,你需要使用 wxWindow::SetDropTarget 函数,将某个窗口和一个 wxDropTarget 绑定在一起,你需要实现一个 wxDropTarget 的派生类,并且重载它的所有纯虚函数.另外还需要重载 OnDragOver 函数,以便返回一个 wxDragResult 类型的返回码,以说明当鼠标指针移过这个窗口的时候,光标应该怎样显示,并且重载 OnData 函数来实现放置操作.你还可以通过继承 wxTexTDropTarget 或者 wxFileDropTarget,或者重载它们的 OnDropText 或者 OnDropFiles 函数来实现一个拖放目的.

下面的步骤将发生在拖放操作过程当中的拖放目的对象上.

1 初始化

wxWindow::SetDropTarget 函数在窗口创建期间被调用,以便将其和一个拖放目的对象绑定.在窗口创建或者应用程序的其他某个部分,通过函数 wxDropTarget::SetDataObject,拖放目的对象和某一种数据类型绑定,这种数据类型将用来作为拖放源和播放目的进行协商的依据.

2 拖动

当鼠标在拖放目的上以拖动的方式移动时,wxDropTarget::OnEnter,wxDropTarget:: OnDragOver 和 wxDropTarget::OnLeave 函数将在适当的时候被调用,它们都将返回一个对应的 wxDragResult 值.以便拖放操作可以对其进行合适的用户界面反馈.

3 放置

当用户释放鼠标按钮的时候,wxWidgets 通过调用函数 wxDataObject::GetAllFormats 询问窗口绑定的 wxDropTarget 对象是否接受正在拖动的数据.如果数据类型是可接受的,那么 wxDropTarget::OnData 将被调用.拖放对象绑定的 wxDataObject 对象将进行对应的数据填充动作.wxDropTarget::OnData 函数将返回一个 wxDragResult 类型的值,这个值将作为 wxDropSource::DoDragDrop 函数的返回值.

使用标准的拖放目的对象

wxWidgets 提供了了标准的 wxDropTarget 的派生类,因此你不必在任何时候都需要实现自己的拖放对象.你只需要实现重载这些类的一个虚函数,以便在拖放的时候得到提示.

wxTextdropTarget 对象可以接收被拖动的文本数据,你只需要重载 OnDropText 函数以便告诉 wxWidgets 当有文本数据被放置的时候做什么事情就可以了.下面的例子演示了当有文本数据被放置的时候,怎样将其添加列表框内.

```cpp
// 一个拖放目的用来将文本填加到列表框
class DnDText : public wxTextDropTarget
{
public:
    DnDText(wxListBox *owner) { m_owner = owner; }
    virtual bool OnDropText(wxCoord x, wxCoord y,
                              const wxString& text)
    {
        m_owner->Append(text);
        return true;
    }
private:
    wxListBox *m_owner;
};
// 设置拖放目的对象
wxListBox* listBox = new wxListBox(parent, wxID_ANY);
listBox->SetDropTarget(new DnDText(listBox)); 
```

下面的例子展示了怎样使用 wxFileDropTarget,这个对象可接收从资源管理器里拖动的文件对象,并且报告拖动文件的数目以及它们的名称.

```cpp
// 一个拖放目的类用来将拖动的文件名添加到列表框
class DnDFile : public wxFileDropTarget
{
public:
    DnDFile(wxListBox *owner) { m_owner = owner; }
    virtual bool OnDropFiles(wxCoord x, wxCoord y,
                               const wxArrayString& filenames)
    {
        size_t nFiles = filenames.GetCount();
        wxString str;
        str.Printf( wxT("%d files dropped"), (int) nFiles);
        m_owner->Append(str);
        for ( size_t n = 0; n &lt; nFiles; n++ ) {
            m_owner->Append(filenames[n]);
        }
        return true;
    }
private:
    wxListBox *m_owner;
};
// 设置拖放目的类
wxListBox* listBox = new wxListBox(parent, wxID_ANY);
listBox->SetDropTarget(new DnDFile(listBox)); 
```

创建一个自定义的拖放目的

现在我们来创建一个自定义的拖放目的,它可以接受 URLs(网址).这一次我们需要重载 OnData 和 OnDragOver 函数,我们还将实现一个可以被重载的虚函数 OnDropURL.

```cpp
// 一个自定义的拖放目的对象,可以拖放 URL 对象
class URLDropTarget : public wxDropTarget
{
public:
    URLDropTarget() { SetDataObject(new wxURLDataObject); }
    void OnDropURL(wxCoord x, wxCoord y, const wxString& text)
    {
        // 当然, 一个真正的应用程序在这里应该做些更有意义的事情
        wxMessageBox(text, wxT("URLDropTarget: got URL"),
                      wxICON_INFORMATION | wxOK);
    }
    // URLs 不能被移动,只能被拷贝
    virtual wxDragResult OnDragOver(wxCoord x, wxCoord y,
                                      wxDragResult def)
        {
            return wxDragLink;
        }
    // 这个函数调用了 OnDropURL 函数,以便它的派生类可以更方便的使用
    virtual wxDragResult OnData(wxCoord x, wxCoord y,
                                  wxDragResult def)
    {
        if ( !GetData() )
            return wxDragNone;
        OnDropURL(x, y, ((wxURLDataObject *)m_dataObject)->GetURL());
        return def;
    }
};
// 设置拖放目的对象
wxListBox* listBox = new wxListBox(parent, wxID_ANY);
listBox->SetDropTarget(new URLDropTarget); 
```

更多关于 wxDataObject 的知识

正如我们已经看到的那样,wxDataObject 用来表示所有可以被拖放.以及可以被剪贴板操作的数据.wxDataObject 最重要的特性之一,是在于它是一个"聪明"的数据块,和普通的包含一段内存缓冲,或者一些文件的哑数据不同.所谓"聪明"指的是数据对像自己可以知道它内部的数据可以支持什么数据格式,以及怎样将它的内部数据表现为那种数据格式.

所谓支持的数据格式,意思是说,这种格式可以从一个数据对象的内部数据或者将被设置的内部数据产生.通常情况下,一个数据对象可以支持多种数据格式作为输入或者输出.因此,一个数据对象可以支持从一种格式创建内部数据,并将其转换为另外一种数据格式,反之亦然.

当你需要使用某种数据对象的时候,有下面几种可选方案:

1.  使用一种内建的数据对象.当你只需要支持文本数据,图片数据,或者是文件列表数据的时候,你可以使用 wxTextdataObject,wxBitmapDataObject,或 wxFileDataObject.
2.  使用 wxDataObjectSimple.从 wxDataObjectSimple 产生一个派生类,是实现自定义数据格式的最简便的方法,不过,这种派生类只能支持一种数据格式,因此,你将不能够使用它和别的应用程序进行交互.不过,你可以用它在你的应用程序以及你应用程序的不同拷贝之间进行数据传输.
3.  使用 wxCustomDataObject 的派生类(它是 wxDataObjectSimple 的一个子类)来实现自定义数据对象.
4.  使用 wxDataObjectComposite.这是一个简单而强大的解决方案,它允许你支持任意多种数据格式(如果你和前面的方法结合使用的话,可以实现同时支持标准数据和自定义数据).
5.  直接使用 wxDataObject.这种方案可以实现最大的灵活度和效率,但也是最难的一种实现方案.

在拖放操作和剪贴板操作中,使用多重数据格式最简单的方法是使用 wxDataObjectComposite 对象,但是,这种使用方法的效率是很低的,因为每一个 wxDataObjectSimple 都使用自己定义的格式来保存所有的数据.试想一下,你将从剪贴板上以你自己的格式粘贴超过两百页的文本,其中包含 Word,RTF,HTML,Unicode,和普通文本,虽然现在计算机的能力已经足可以应付这样的任务,但是从性能方面考虑, 你最好还是直接从 wxDataObject 实现一个派生类,用它来定义你的数据格式,然后在程序中指定这种类型的数据.

剪贴板操作和拖放操作,潜在的数据传输机制将在某个应用程序真正请求数据的时候,才会进行数据拷贝.因此,尽管用户可能认为在自己点击了应用程序的拷贝按钮以后数据就已存在于剪贴板了,但实际上这时候仅仅是告诉剪贴板有数据存在了.

实现 wxDataObject 的派生类

我们来看一下实现一个新的 wxDataObject 的派生类要用到哪些东西.至于怎样实现的过程,我们前面已经介绍过了,它是非常简单的.因此,我们在这里不多说了.

每一个 wxDataObject 的派生类,都必须重载和实现它的纯虚成员函数,那些只能用来输出数据或者保存数据(意味着只能进行单向操作)的数据对象的 GetFormatCount 函数在其不支持的方向上应该总是返回零.

GetAllFormats 函数的参考为一个 wxDataFormat 类型的列表,以及一个方向(获取或设置).它将所有自己在这个方向上支持的数据格式填入这个列表.GetFormatCount 函数则用来检测列表中元素的个数.

GetdataHere 函数的参数是一个 wxDataFormat 参数,以及一个 void*缓冲区.如果操作成功,返回 TRue,否则返回 false.这个函数必须将数据以给定的格式填入这个缓冲区,数据可以是任意的二进制数据,或者是文本数据,只要 SetData 函数可以识别就行了.

GetdataSize 函数则返回在某种给定的数据格式下数据的大小.

GetFormatCount 函数返回用于转换或者设置的当前支持数据类型的个数.

GetPreferredFormat 函数则返回某个指定方向上优选的数据类型.

SetData 函数的参考包括一个数据类型,一个整数格式的缓冲区大小,以及一个 void*类型的缓冲区指针.你可以在适当的时候(比如将其拷贝到自己的内部结构的时候)对其进行适当的解释.这个函数在成功的时候返回 TRue,失败的时候返回 false.

wxWidgets 的拖放操作例子

我们通过使用位于 samples/dnd 目录的 wxWidgets 的拖放操作的例子,来演示一下怎样制作一个自定义的拥有自定义数据类型的数据对象.这个例子演示了一个简单的绘图软件,可以用来绘制矩形,三角形,或者椭圆形,并且允许你对其进行编辑,拖放到一个新的位置,拷贝到剪贴板以及从新粘贴到应用程序等操作.你可以通过选择文件菜单中的新建命令来创建一个应用程序的主窗口.这个窗口的外观如下图所示:

![](img/mht5C51%281%29.tmp)

这些图形是用继承字 DnDShape 的类来建模的,数据对象被称为 DnDShapeDataObject.在学习怎样实现这个数据对象之前,我们先看一下应用程序是怎样使用它们的.

当一个剪贴板拷贝操作被请求的时候,一个 DnDShapeDataObject 对象将被增加到剪贴板,这个对象包含当前正在操作的图形的拷贝,如果剪贴板上已经有了一个对象,那么旧的对象将被释放.代码如下所示:

```cpp
void DnDShapeFrame::OnCopyShape(wxCommandEvent& event)
{
    if ( m_shape )
    {
        wxClipboardLocker clipLocker;
        if ( !clipLocker )
        {
            wxLogError(wxT("Can't open the clipboard"));
            return;
        }
        wxTheClipboard->AddData(new DnDShapeDataObject(m_shape));
    }
} 
```

剪贴板的粘贴操作也很容易理解,调用 wxClipboard::GetData 函数来获取位于剪贴板的图形数据对象,然后从其中获取图形数据.前面我们已经介绍过怎样在剪贴板拥有对应数据的时候允许 paste 菜单.shapeFormatId 是一个全局变量,其中包含了自定义的数据格式名称:wxShape.

```cpp
void DnDShapeFrame::OnPasteShape(wxCommandEvent& event)
{
    wxClipboardLocker clipLocker;
    if ( !clipLocker )
    {
        wxLogError(wxT("Can't open the clipboard"));
        return;
    }
    DnDShapeDataObject shapeDataObject(NULL);
    if ( wxTheClipboard->GetData(shapeDataObject) )
    {
        SetShape(shapeDataObject.GetShape());
    }
    else
    {
        wxLogStatus(wxT("No shape on the clipboard"));
    }
}
void DnDShapeFrame::OnUpdateUIPaste(wxUpdateUIEvent& event)
{
    event.Enable( wxTheClipboard->
                      IsSupported(wxDataFormat(shapeFormatId)) );
} 
```

为了实现拖放操作,还需要一个拖放目标对象,以便在图片数据被放置的时候通知应用程序.DnDShapeDropTarget 类包含一个 DnDShapeDataObject 数据对象,用来扮演这个角色,当它的 OnData 函数被调用的时候,则表明正在放置一个图形数据对象,下面的代码演示了 DnDShapeDropTarget 的声明及实现:

```cpp
class DnDShapeDropTarget : public wxDropTarget
{
public:
    DnDShapeDropTarget(DnDShapeFrame *frame)
        : wxDropTarget(new DnDShapeDataObject)
    {
        m_frame = frame;
    }
    // 重载基类的(纯)虚函数
    virtual wxDragResult OnEnter(wxCoord x, wxCoord y, wxDragResult def)
    {
        m_frame->SetStatusText(_T("Mouse entered the frame"));
        return OnDragOver(x, y, def);
    }
    virtual void OnLeave()
    {
        m_frame->SetStatusText(_T("Mouse left the frame"));
    }
    virtual wxDragResult OnData(wxCoord x, wxCoord y, wxDragResult def)
    {
        if ( !GetData() )
        {
            wxLogError(wxT("Failed to get drag and drop data"));
            return wxDragNone;
        }
        // 通知主窗口正在进行放置
        m_frame->OnDrop(x, y,
                ((DnDShapeDataObject *)GetDataObject())->GetShape());
        return def;
    }
private:
    DnDShapeFrame *m_frame;
}; 
```

在应用程序初始化函数里,主窗口被创建的时候设置这个拖放目标:

```cpp
DnDShapeFrame::DnDShapeFrame(wxFrame *parent)
             : wxFrame(parent, wxID_ANY, _T("Shape Frame"))
{
    ...
    SetDropTarget(new DnDShapeDropTarget(this));
    ...
} 
```

当鼠标左键单击的时候,拖放操作开始,其处理函数将创建一个 wxDropSource 对象,并且给 DoDragDrop 函数传递一个 DnDShapeDataObject 对象,以便初始化拖放操作.DndShapeFrame::OnDrag 函数如下所示:

```cpp
void DnDShapeFrame::OnDrag(wxMouseEvent& event)
{
    if ( !m_shape )
    {
        event.Skip();
        return;
    }
    // 开始拖放操作
    DnDShapeDataObject shapeData(m_shape);
    wxDropSource source(shapeData, this);
    const wxChar *pc = NULL;
    switch ( source.DoDragDrop(true) )
    {
        default:
        case wxDragError:
            wxLogError(wxT("An error occured during drag and drop"));
            break;
        case wxDragNone:
            SetStatusText(_T("Nothing happened"));
            break;
        case wxDragCopy:
            pc = _T("copied");
            break;
        case wxDragMove:
            pc = _T("moved");
            if ( ms_lastDropTarget != this )
            {
                // 如果这个图形被放置在自己的窗口上
                // 不要删除它
                SetShape(NULL);
            }
            break;
        case wxDragCancel:
            SetStatusText(_T("Drag and drop operation cancelled"));
            break;
    }
    if ( pc )
    {
        SetStatusText(wxString(_T("Shape successfully ")) + pc);
    }
    //在其他情况下,状态文本已经被设置了
} 
```

当用户释放鼠标以表明正在执行放置操作的时候,wxWidgets 调用 DnDShapeDropTarget::OnData 函数,这个函数将以一个新的 DndShape 对象来调用 DndShapeFrame::OnDrop 函数,以便给 DndShape 对象设置一个新的位置.这样,拖放操作就完成了.

```cpp
void DnDShapeFrame::OnDrop(wxCoord x, wxCoord y, DnDShape *shape)
{
    ms_lastDropTarget = this;
    wxPoint pt(x, y);
    wxString s;
    s.Printf(wxT("Shape dropped at (%d, %d)"), pt.x, pt.y);
    SetStatusText(s);
    shape->Move(pt);
    SetShape(shape);
} 
```

现在,唯一剩下的事情,就是实现自定义的 wxDataObject 对象了.为了说明得更清楚,我们将对整个实现进行逐项说明.首先,我们来看一下自定义数据类型标识符声明,以及 DndShapeDataObject 类的声明,它的构造函数和析构函数,和它的数据成员.

数据类型标识符是 shapeFormatId,它是一个全局变量,在整个例子中都有使用.构造函数通过 GeTDataHere 函数获得当前图形(如果有的话)的一个拷贝作为参数.这个拷贝也可以通过 DndShape::Clone 函数产生.DnDShapeDataObject 的析构函数将会释放这个拷贝.

DndShapeDataObject 可以提供位图和(在支持的平台上)源文件来表示它的内部数据.因此,它还拥有 wxBitmapDataObject 和 wxMetaFileDataObject 两个类型的数据成员(以及一个标记用来指示当前正在使用哪种类型)来缓存内部数据以便在需要的时候提供这种格式.

```cpp
// 自定义的数据格式标识符
static const wxChar *shapeFormatId = wxT("wxShape");

class DnDShapeDataObject : public wxDataObject
{
public:
    // 构造函数没有直接拷贝指针
    // 这样在原来的图形对象被释放以后,这里的图形对象是有效的
    DnDShapeDataObject(DnDShape *shape = (DnDShape *)NULL)
    {
        if ( shape )
        {
            // 我们需要拷贝真正的图形对象,而不是只拷贝指针
            // 这是因为图形对象有可能在任何时候被删除,在这种情况下
            // 剪贴板上的数据仍然应该是有效的
            // 因此我们使用下边的方法来实现图形拷贝
            void *buf = malloc(shape->DnDShape::GetDataSize());
            shape->GetDataHere(buf);
            m_shape = DnDShape::New(buf);
            free(buf);
        }
        else
        {
            // 不需要拷贝任何东西
            m_shape = NULL;
        }
        // 这个字符串应该用来唯一标识我们的数据格式类型
        // 除此以外,它可以是任意的字符串
        m_formatShape.SetId(shapeFormatId);
        // 我们直到需要的(也就是数据被第一次请求)时候才产生图片或者元文件数据
        m_hasBitmap = false;
        m_hasMetaFile = false;
    }
    virtual ~DnDShapeDataObject() { delete m_shape; }
    // 在这个函数被调用以后,图形数据归调用者所有
    // 调用者将负责释放相关内存
    DnDShape *GetShape()
    {
        DnDShape *shape = m_shape;
        m_shape = (DnDShape *)NULL;
        m_hasBitmap = false;
        m_hasMetaFile = false;
        return shape;
    }
    // 其他成员函数省略
    ...
    // 数据成员
private:
    wxDataFormat         m_formatShape; // 我们的自定义格式
    wxBitmapDataObject   m_dobjBitmap;  // 用来响应位图格式请求
    bool                 m_hasBitmap;   // 如果 m_dobjBitmap 有效为真
    wxMetaFileDataObject m_dobjMetaFile;// 用来响应元数据格式请求
    bool                 m_hasMetaFile;// 如果 m_dobjMetaFile 有效为真
    DnDShape             *m_shape;       // 原始数据
}; 
```

接下来我们来看一下那些用于回答和我们内部存储的数据相关的问题的函数.GetPreferredFormat 只简单的返回 m_formatShape 数据绑定的本地的数据格式,它是在我们的构造函数中使用 wxShape 类型初始化的.GetFormatCount 函数用来检测某种特定的格式是否可以被用来获取或者设置数据.在获取数据的时候,只有位图和元文件格式是可以被处理的.GetDataSize 函数依据请求的数据格式的不同返回合适的数据大小,如果必要的话,为了得到这个大小,你可以在这个时候创建位图成员或者元文件成员.

```cpp
virtual wxDataFormat GetPreferredFormat(Direction dir) const
{
    return m_formatShape;
}
virtual size_t GetFormatCount(Direction dir) const
{
    // 我们自定义的数据格式类型即可以支持 GetData()
    // 也可以支持 SetData()
    size_t nFormats = 1;
    if ( dir == Get )
    {
        // 但是,位图格式只支持输出
        nFormats += m_dobjBitmap.GetFormatCount(dir);
        nFormats += m_dobjMetaFile.GetFormatCount(dir);
    }
    return nFormats;
}
virtual void GetAllFormats(wxDataFormat *formats, Direction dir) const
{
    formats[0] = m_formatShape;
    if ( dir == Get )
    {
        // 在获取方向上我们增加位图和元文件两种格式的支持
        //在 Windows 平台上
        m_dobjBitmap.GetAllFormats(&formats[1], dir);
        // 不要认为 m_dobjBitmap 只有一种格式
        m_dobjMetaFile.GetAllFormats(&formats[1 +
                m_dobjBitmap.GetFormatCount(dir)], dir);
    }
}
virtual size_t GetDataSize(const wxDataFormat& format) const
{
    if ( format == m_formatShape )
    {
        return m_shape->GetDataSize();
    }
    else if ( m_dobjMetaFile.IsSupported(format) )
    {
        if ( !m_hasMetaFile )
            CreateMetaFile();
        return m_dobjMetaFile.GetDataSize(format);
    }
    else
    {
        wxASSERT_MSG( m_dobjBitmap.IsSupported(format),
                       wxT("unexpected format") );
        if ( !m_hasBitmap )
            CreateBitmap();
        return m_dobjBitmap.GetDataSize();
    }
} 
```

GetdataHere 函数按照请求的数据格式类型将数据拷贝到 void*类型的缓冲区:

```cpp
virtual bool GetDataHere(const wxDataFormat& format, void *pBuf) const
{
    if ( format == m_formatShape )
    {
        // 使用 ShapeDump 结构将其转换为 void*流
        m_shape->GetDataHere(pBuf);
        return true;
    }
    else if ( m_dobjMetaFile.IsSupported(format) )
    {
        if ( !m_hasMetaFile )
            CreateMetaFile();
        return m_dobjMetaFile.GetDataHere(format, pBuf);
    }
    else
    {
        wxASSERT_MSG( m_dobjBitmap.IsSupported(format),
                      wxT("unexpected format") );
        if ( !m_hasBitmap )
            CreateBitmap();
        return m_dobjBitmap.GetDataHere(pBuf);
    }
} 
```

SetData 函数只需要处理本地格式,因此,它需要做的所有事情就是使用 DndShape::New 函数来制作一个参数图形的拷贝:

```cpp
virtual bool SetData(const wxDataFormat& format,
                       size_t len, const void *buf)
{
    wxCHECK_MSG( format == m_formatShape, false,
                  wxT( "unsupported format") );
    delete m_shape;
    m_shape = DnDShape::New(buf);
    // the shape has changed
    m_hasBitmap = false;
    m_hasMetaFile = false;
    return true;
} 
```

实现 DndShape 和 void*类型的互相转换的方法是非常直接的.它使用了一个 ShapeDump 的结构来保存图形的详细信息.下面是其实现方法:

```cpp
// 静态函数用来从一个 void*缓冲区中创建一个图形
DnDShape *DnDShape::New(const void *buf)
{
    const ShapeDump& dump = *(const ShapeDump *)buf;
    switch ( dump.k )
    {
        case Triangle:
            return new DnDTriangularShape(
                             wxPoint(dump.x, dump.y),
                             wxSize(dump.w, dump.h),
                             wxColour(dump.r, dump.g, dump.b));
        case Rectangle:
            return new DnDRectangularShape(
                             wxPoint(dump.x, dump.y),
                             wxSize(dump.w, dump.h),
                             wxColour(dump.r, dump.g, dump.b));
        case Ellipse:
            return new DnDEllipticShape(
                             wxPoint(dump.x, dump.y),
                             wxSize(dump.w, dump.h),
                             wxColour(dump.r, dump.g, dump.b));
        default:
            wxFAIL_MSG(wxT("invalid shape!"));
            return NULL;
    }
}
// 返回内部数据大小
size_t DndShape::GetDataSize() const
{
    return sizeof(ShapeDump);
}
// 将自己填入一个 void*缓冲区
void DndShape::GetDataHere(void *buf) const
{
    ShapeDump& dump = *(ShapeDump *)buf;
    dump.x = m_pos.x;
    dump.y = m_pos.y;
    dump.w = m_size.x;
    dump.h = m_size.y;
    dump.r = m_col.Red();
    dump.g = m_col.Green();
    dump.b = m_col.Blue();
    dump.k = GetKind();
} 
```

最后,我们回到 DnDShapeDataObject 数据对象,下边的这些函数用来在需要的时候将内部数据转换为位图或者元数据:

```cpp
void DnDShapeDataObject::CreateMetaFile() const
{
    wxPoint pos = m_shape->GetPosition();
    wxSize size = m_shape->GetSize();
    wxMetaFileDC dcMF(wxEmptyString, pos.x + size.x, pos.y + size.y);
    m_shape->Draw(dcMF);
    wxMetafile *mf = dcMF.Close();
    DnDShapeDataObject *self = (DnDShapeDataObject *)this;
    self->m_dobjMetaFile.SetMetafile(*mf);
    self->m_hasMetaFile = true;
    delete mf;
}
void DnDShapeDataObject::CreateBitmap() const
{
    wxPoint pos = m_shape->GetPosition();
    wxSize size = m_shape->GetSize();
    int x = pos.x + size.x,
        y = pos.y + size.y;
    wxBitmap bitmap(x, y);
    wxMemoryDC dc;
    dc.SelectObject(bitmap);
    dc.SetBrush(wxBrush(wxT("white"), wxSOLID));
    dc.Clear();
    m_shape->Draw(dc);
    dc.SelectObject(wxNullBitmap);
    DnDShapeDataObject *self = (DnDShapeDataObject *)this;
    self->m_dobjBitmap.SetBitmap(bitmap);
    self->m_hasBitmap = true;
} 
```

我们自定义的数据对象的实现到此为止就全部完成了,部分细节(比如图形怎样把自己绘制到用户界面上)没有在此列出,你可以参考 wxWidgets 自带的 samples/dnd 中的代码.

wxWidgets 中的拖放相关的一些帮助

下面我们来描述一些在实现拖放操作时可以给你帮助的控件.

wxTreeCtrl

你可以使用 EVT_TREE_BEGIN_DRAG 或 EVT_TREE_BEGIN_RDRAG 事件映射宏来增加对鼠标左键或右键开始的拖放操作的处理,这是这个控件内部的鼠标事件处理函数实现的.在你的事件处理函数中,你可调用 wxtreeEvent::Allow 来允许 wxtreeCtrl 使用它自己的拖放实现来发送一个 EVT_TREE_END_DRAG 事件.如果你选择了使用 tree 控件自己的拖放代码,那么随着拖放鼠标指针的移动,将会有一个小的拖动图片被创建,并随之移动,整个放置的操作则完全需要在应用程序的结束放置事件处理函数中实现.

下面的例子演示了怎样使用树状控件提供的拖放事件,来实现当用户把树状控件中的一个子项拖到另外一个子项上的时候,产生一个被拖动子项的拷贝.

```cpp
BEGIN_EVENT_TABLE(MyTreeCtrl, wxTreeCtrl)
    EVT_TREE_BEGIN_DRAG(TreeTest_Ctrl, MyTreeCtrl::OnBeginDrag)
    EVT_TREE_END_DRAG(TreeTest_Ctrl, MyTreeCtrl::OnEndDrag)
END_EVENT_TABLE()
void MyTreeCtrl::OnBeginDrag(wxTreeEvent& event)
{
    // 需要显式的指明允许拖动
    if ( event.GetItem() != GetRootItem() )
    {
        m_draggedItem = event.GetItem();
        wxLogMessage(wxT("OnBeginDrag: started dragging %s"),
                      GetItemText(m_draggedItem).c_str());
        event.Allow();
    }
    else
    {
        wxLogMessage(wxT("OnBeginDrag: this item can't be dragged."));
    }
}
void MyTreeCtrl::OnEndDrag(wxTreeEvent& event)
{
    wxTreeItemId itemSrc = m_draggedItem,
                  itemDst = event.GetItem();
    m_draggedItem = (wxTreeItemId)0l;
    // 在哪里拷贝这个子项呢?
    if ( itemDst.IsOk() && !ItemHasChildren(itemDst) )
    {
        // 这种情况下拷贝到它的父项内
        itemDst = GetItemParent(itemDst);
    }
    if ( !itemDst.IsOk() )
    {
        wxLogMessage(wxT("OnEndDrag: can't drop here."));
        return;
    }
    wxString text = GetItemText(itemSrc);
    wxLogMessage(wxT("OnEndDrag: '%s' copied to '%s'."),
                  text.c_str(), GetItemText(itemDst).c_str());
    //  增加新的子项
    int image = wxGetApp().ShowImages() ? TreeCtrlIcon_File : -1;
    AppendItem(itemDst, text, image);
} 
```

如果你想自己处理拖放操作,比如使用 wxDropSource 来实现,你可以在拖放开始事件处理函数中使用 wxtreeEvent:: Allow 函数来禁止默认的拖放动作,并且开始你自己的拖放动作.这种情况下拖放结束事件将不会被发送,因为你已经决定用自己的方式来处理拖放(如果使用 wxDropSource::DoDragDrop 函数,你需要自己检测何时拖放结束).

wxListCtrl

这个类没有提供默认的拖动图片,或者拖放结束事件,但是,它可以让你知道什么时候开始一个拖放操作.使用 EVT_LIST_BEGIN_DRAG 或 EVT_LIST_BEGIN_RDRAG 事件映射宏来实现你自己的拖放代码.你也可以使用 EVT_LIST_COL_BEGIN_DRAG,EVT_LIST_COL_DRAGGING 和 EVT_LIST_COL_END_DRAG 来检测何时某一个单独的列正在被拖动.

wxDragImage

在你实现自己的拖放操作的时候,可以很方便地使用 wxDragImage 类.它可以在顶层窗口上绘制一副图片,还可以移动这个图片,并且不损坏它后面的窗口.这通常是通过在移动之前保存一份背景窗口,并且在需要的时候,重绘背景窗口来实现的.

下图演示了 wxDragImage 例子中的主窗口,你可以在 wxWidgets 的 samples/dragimag 中找到这个例子.当主窗口上的三个拼图块被拖动时,将会采用不同的拖动图片,分别为图片本身,一个图标,或者一个动态产生的包含一串文本的图片.如果你选择使用整个屏幕这个选项,那么这个图片可以被拖动到窗口以外的地方,在 Windows 平台上,这个例子即可以使用标准的 wxDragImage 的实现(默认情形)来编译,也可以使用本地原生控件来编译,后者需要你在 dragimag.cpp 中将 wxUSE_GENERIC_DRAGIMAGE 置为 1.

![](img/mht5C64%281%29.tmp)

当检测到开始拖动操作的时候,创建一个 wxDragImage 对象,并且把它存放在任何你可以在整个拖动过程中访问的地方,然后调用 BeginDrag 来开始拖动,调用 EndDrag 来结束拖动.要移动这个图片,第一次要使用 Show 函数,后面则需要使用 Move 函数.如果你需要在拖动过程当中刷新屏幕内容(比如在 dragimag 的例子中高量显示某个项目),你需要先调用 Hide 函数,然后更新你的窗口,然后调用 Move 函数,然后调用 Show 函数.

你可以在一个窗口内拖动,也可以在整个屏幕或者屏幕的任何一部分内拖动,以节省资源.如果你希望用户可以在两个拥有不同的顶层父窗口的窗口之间拖动,你就必须使用全屏拖动的方式.全屏拖动的效果不一定完美,因为它在开始拖动的时候获取了一副整个屏幕的快照,屏幕后续的改动则不会进行相应的更新.如果在你的拖动过程当中,别的应用程序对屏幕内容进行了改动,将会影响到拖动的效果.

在接下来的例子中,基于上面的那个例子,MyCanvas 窗口显示了很多 DragShap 类的图片,它们中的每一个都和一副图片绑定. 当针对某个 DragShap 的拖动操作开始时,一个使用其绑定的图片的 wxDragImage 对象被创建,并且 BeginDrag 被调用.当检测到鼠标移动的时候,调用 wxDragImage::Move 函数来移动来将这个对象进行相应的移动.最后,当鼠标左键被释放的时候,用于指示拖动的图片被释放,被拖动的图片则在其新的位置被重绘.

```cpp
void MyCanvas::OnMouseEvent(wxMouseEvent& event)
{
    if (event.LeftDown())
    {
        DragShape* shape = FindShape(event.GetPosition());
        if (shape)
        {
            // 我们姑且认为拖动操作已经开始
            // 不过最好等待鼠标移动一段时间再真正开始.
            m_dragMode = TEST_DRAG_START;
            m_dragStartPos = event.GetPosition();
            m_draggedShape = shape;
        }
    }
    else if (event.LeftUp() && m_dragMode != TEST_DRAG_NONE)
    {
        // 拖动操作结束
        m_dragMode = TEST_DRAG_NONE;
        if (!m_draggedShape || !m_dragImage)
            return;
        m_draggedShape->SetPosition(m_draggedShape->GetPosition()
                            + event.GetPosition() - m_dragStartPos);
        m_dragImage->Hide();
        m_dragImage->EndDrag();
        delete m_dragImage;
        m_dragImage = NULL;
        m_draggedShape->SetShow(true);
        m_draggedShape->Draw(dc);
        m_draggedShape = NULL;
    }
    else if (event.Dragging() && m_dragMode != TEST_DRAG_NONE)
    {
        if (m_dragMode == TEST_DRAG_START)
        {
            // 我们将在鼠标已经移动了一小段距离以后开始真正的拖动
            int tolerance = 2;
            int dx = abs(event.GetPosition().x - m_dragStartPos.x);
            int dy = abs(event.GetPosition().y - m_dragStartPos.y);
            if (dx &lt;= tolerance && dy &lt;= tolerance)
                return;
            // 开始拖动.
            m_dragMode = TEST_DRAG_DRAGGING;
            if (m_dragImage)
                delete m_dragImage;
            // 从画布上清除拖动图片
            m_draggedShape->SetShow(false);
            wxClientDC dc(this);
            EraseShape(m_draggedShape, dc);
            DrawShapes(dc);
            m_dragImage = new wxDragImage(
                                           m_draggedShape->GetBitmap());
            // 被拖动图片的左上角到目前位置的偏移量
            wxPoint beginDragHotSpot = m_dragStartPos
                                         m_draggedShape->GetPosition();
            // 总认为坐标系为被捕获窗口的客户区坐标系
            if (!m_dragImage->BeginDrag(beginDragHotSpot, this))
            {
                delete m_dragImage;
                m_dragImage = NULL;
                m_dragMode = TEST_DRAG_NONE;
            } else
            {
                m_dragImage->Move(event.GetPosition());
                m_dragImage->Show();
            }
        }
        else if (m_dragMode == TEST_DRAG_DRAGGING)
        {
            // 移动这个图片
            m_dragImage->Move(event.GetPosition());
        }
    }
} 
```

如果你希望自己绘制用于拖动的图片而不是使用一个位图,你可以实现一个 wxGenericDragImage 的派生类,重载其 wxDragImage::DoDrawImage 函数和 wxDragImage::GetImageRect 函数.在非 windows 的平台上, wxDragImage 是 wxGenericDragImage 的一个别名而已,而 windows 平台上实现的 wxDragImage 不支持 DoDrawImage 函数,也限制只能绘制有时候显得有点恐怖的半透明图片,因此,你可以考虑在所有的平台上都使用 wxGenericDragImage 类.

当你开始拖动操作的时候,就在正准备调用 wxDragImage::Show 函数之前,通常你需要现在屏幕上擦除你要拖动的对象,这可以使得 wxDragImage 保存的背景中没有正在拖动的对象,因此整个的拖动过程看上去也更合理,不过这将导致屏幕在开始拖动的时候会有一点点的闪烁.要避免这种闪烁(仅适用于使用 wxGenericDragImage 的情况),你可以重载 wxGenericDragImage 的 UpdateBackingFromWindow 函数,使用传递给你的设备上下文绘制一个不包含正在拖动对象的背景,然后你就不需要在调用 wxDragImage::Show 函数之前擦除你要拖动的对象了,整个拖动过程的屏幕就将会是平滑而无闪烁的了.

# 第十一章小结

在这一章里,我们看到了怎样将数据传输到剪贴板上或者怎样从剪贴板获取数据.我们也了解了怎样从拖放源的角度以及拖放目的的角度实现拖放操作.还了解了 wxWidgets 中和拖放相关的一些其他领域的知识.更深入的了解请参考 wxWidgets 的 samples/dnd, samples/dragimag 和 samples/treectrl 目录中的例子.

在下一章里,我们将回到窗口类相关的主题,介绍一些高级的窗口类以及怎样通过它们让你的应用程序进入一个更新的层级.