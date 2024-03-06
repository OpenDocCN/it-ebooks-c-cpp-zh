# 第十二章高级窗口控件

# 第十二章高级窗口控件

显然我们不能在这里列举 wxWidgets 提供的所有的控件,但是还是有必要对其中几个更高级一点的控件作一些介绍,以便在需要的时候你可以更好的使用它们.本章覆盖的内容包括下面的主题:

*   wxTReeCtrl; 这个控件用来帮助你为分等级的数据建模.
*   wxListCtrl; 这个控件让你以灵活的方式显示一组文本标签和图标.
*   wxWizard; 这个控件使用多个页面对某个特定的任务提供向导机制.
*   wxHtmlWindow; 你可以在"关于"对话框和报告对话框(以及其他你可以想到的对话框)中使用的轻量级的 HTML 显示控件.
*   wxGrid; 网格控件用是一个支持多种特性的标状数据显示控件.
*   wxTaskBarIcon; 这个控件让你的程序可以很容易的访问系统托盘区或者类似的区域.
*   编写自定义的控件. 介绍了制作一个专业级的自定义控件必须的几个步骤

# 12.1 wxTreeCtrl

# 12.1 wxTreeCtrl

树状控件以层的形式展示信息,它的子项可以展开也可以合并.下图演示了 wxWidgets 的树状控件例子,它正以不同的字体和风格以及颜色进行展示.每一个树状控件的子项都代表一个 wxtreeItemId 对象,它拥有一个文本标签和一个可选图标,并且文本和图标的内容都可以动态修改.树状控件可以以单选或者多选的形式创建.如果你希望在 wxtreeItemId 上绑定一些数据,你需要实现自己的 wxTreeItemData 派生类,然后调用 wxTreeCtrl::SetItemData 函数以及 wxTreeCtrl::GetItemData 函数.这个数据在子项被释放的时候将会被一并释放(delete 调用),如果你将其指向你实际的数据,需要注意避免重复释放.

![](img/mht8FFD%281%29.tmp)

因为应用程序可以检测到树状控件的子项被单击的事件,你可以用这个特点通过更改子项的图片来达到模拟其他的控件的目的.比如说,你可以很容易用树状控件的子项来模拟一个复选框.

下面的代码演示了怎样创建一个树状控件,定义其子项的绑定数据以及图片:

```cpp
#include "wx/treectrl.h"
// 声明一个代表和子项绑定的数据的类
class MyTreeItemData : public wxTreeItemData
{
public:
    MyTreeItemData(const wxString& desc) : m_desc(desc) { }
    const wxString& GetDesc() const { return m_desc; }
private:
    wxString m_desc;
};
// 子项相关的图片
#include "file.xpm"
#include "folder.xpm"
// 创建一个树状控件
wxTreeCtrl* treeCtrl = new wxTreeCtrl(
    this, wxID_ANY, wxPoint(0, 0), wxSize(400, 400),
    wxTR_HAS_BUTTONS|wxTR_SINGLE);
wxImageList* imageList = new wxImageList(16, 16);
imageList->Add(wxIcon(folder_xpm);
imageList->Add(wxIcon(file_xpm);
treeCtrl->AssignImageList(imageList);
// 根节点使用文件夹图标,而两个字节点使用文件图标
wxTreeItemId rootId = treeCtrl->AddRoot(wxT("Root"), 0, 0,
                          new MyTreeItemData(wxT("Root item")));
wxTreeItemId itemId1 = treeCtrl->AppendItem(rootId,
                          wxT("File 1"), 1, 1,
                          new MyTreeItemData(wxT("File item 1")));
wxTreeItemId itemId2 = treeCtrl->AppendItem(rootId,
                          wxT("File 2"), 1, 1,
                          new MyTreeItemData(wxT("File item 2"))); 
```

wxTreeCtrl 的窗口类型

wxTreeCtrl 有如下表所示的额外的窗口类型:

| wxtr_DEFAULT_STYLE | 这个值是各个平台上树状控件实现和默认值最接近的值 |
| --- | --- |
| wxtr_EDIT_LABELS | 是否子项文本可编辑 |
| wxtr_NO_BUTTONS | 不必显示用于展开或者合并子项的按钮 |
| wxtr_HAS_BUTTONS | 显示用于展开或者合并子项的按钮 |
| wxTR_NO_LINES | 不必显示用于表示层级关系的垂直虚线 |
| wxtr_FULL_ROW_HIGHLIGHT | 当选中某个子项的时候高亮显示整行(在 windows 平台上,除非设置了 wxtr_NO_LINES,否则这个类型将被忽略) |
| wxtr_LINES_AT_ROOT | 不必显示根节点之间的连线.这个类型只有在设置 wxtr_HIDE_ROOT 并且没有设置 wxtr_NO_LINES 的时候有效 |
| wxtr_HIDE_ROOT | 不显示根节点,这将导致第一层的字节点成为一系列根节点 |
| wxtr_ROW_LINES | 使用这个类型在已显示的行之间绘制一个高对比的边界 |
| wxTR_HAS_VARIABLE_ROW_HEIGHT | 设置这个类型允许各行采用不同的高度,否则各行都将采用和最大的行高同样的高度.这个类仅适用于树状控件的标准实现(而非各个平台的原生实现) |
| wxtr_SINGLE | 单选模式 |
| wxtr_MULTIPLE | 多选模式 |
| wxtr_EXTENDED | 允许多选非连续的子项(该功能仅是部分实现) |

wxTreeCtrl 的事件

树状控件产生 wxtreeEvent 类型的事件,这种事件可以在父子关系的窗口之间传递.

| EVT_TREE_BEGIN_DRAG(id, func)EVT_TREE_BEGIN_RDRAG(id, func) | 在用户开始拖放操作的时候产生,这个事件的使用细节请参考第十一章,"剪贴板和拖放操作" |
| --- | --- |
| EVT_TREE_BEGIN_LABEL_EDIT(id, func) EVT_TREE_END_LABEL_EDIT(id, func) | 当用户开始编辑或者刚刚完成编辑子项标签的时候产生 |
| EVT_TREE_DELETE_ITEM(id, func) | 当某个子项被删除的时候产生 |
| EVT_TREE_GET_INFO(id, func) | 当某个子项的数据被请求的时候产生 |
| EVT_TREE_SET_INFO(id, func) | 当某个子项的数据被设置的时候产生 |
| EVT_TREE_ITEM_ACTIVATED(id, func) | 当某个子项被激活(双击或者使用键盘选择)的时候产生 |
| EVT_TREE_ITEM_COLLAPSED(id, func) | 给定的子项已被收缩(合并)的时候产生 |
| EVT_trEE_ITEM_COLLAPSING(id, func) | 给定的子项即将收缩(合并)的时候产生,这个事件可以被 Veto 以阻止收缩. |
| EVT_TREE_ITEM_EXPANDED(id, func) | 给定子项已被展开的时候产生 |
| EVT_TREE_ITEM_EXPANDING(id, func) | 给定子项即将展开的时候产生,这个事件可以被 Veto 以阻止展开 |
| EVT_TREE_SEL_CHANGED(id, func) | 选中的子项发生变化以后(新的子项被选中或者旧的选中项不被选中的时候)产生 |
| EVT_TREE_SEL_CHANGING(id, func) | 选中的子项即将发生变化的时候产生,该事件可以被 Veto 以阻止变化产生 |
| EVT_TREE_KEY_DOWN(id, func) | 检测针对该树状控件的键盘事件 |
| EVT_TREE_ITEM_GET_TOOLTIP(id, func) | 这个事件仅支持 windows 平台,它使得你可以给某个子项设置单独的工具提示 |

wxTreeCtrl 的成员函数

下面列出了 wxTreeCtrl 控件的一些重要的成员函数.

使用 AddRoot 函数增加第一个子项,然后使用 AppendItem, InsertItem 或 PrependItem 来增加随后的子项.使用 Delete 移除一个子项,使用 DeleteAllItems 删除某个子项所有的子项,或者使用 DeleteChildren 删除某个子项的所有直接子项.

使用 SetItemText 设置某个子项的标签,使用 SetItemTextColour,SetItemBackgroundColour,SetItemBold 和 SetItemFont 来设置标签的外观.

如果你想给某个子项指定一幅图片,首先需要使用 SetImageList 函数将某个图片列表和这个树状控件绑定.每个子项可以指定四个状态的图片,分别是 wxTReeItemIcon_Normal,wxTReeItemIcon_Selected, wxtreeItemIcon_Expanded 和 wxTReeItemIcon_SelectedExpanded,你可以使用 SetItemImage 函数给每个状态指定一个图片列表中图片索引.如果你只给 wxTReeItemIcon_Normal 状态指定了一个索引,那么别的状态也将都使用这个图片.

使用 Scroll 函数以便将某个子项移动到可见区域,使用 EnsureVisible 使得这个子项在需要的时候展开以便其可以位于可见区域.使用 Expand 函数展开某个子项,Collapse 和 CollapseAndReset 函数合并某个子项,后者还将移除其所有的子项,如果你正在使用的树状控件有很多子项,你可能希望只增加可见部分的子项以便提高性能.在这种情况下,你可以处理 EVT_TREE_ITEM_EXPANDING 事件,在需要的时候才增加子项,在收缩的时候则移除所有子项.而且你还需要调用 SetItemHasChildren 函数以便没有子项的子项也可以显示一个可扩展按钮,即使它真的没有.

使用 SelectItem 选择或者去选择某个子项.如果是单选类型,你可以使用 GetSelection 函数得到正被选中的子项,如果当前没有子项被选中,则返回一个未初始化的 wxTReeItemId,你可以调用 wxTreeItemId::IsOk 函数来判断其有效性.而对于多选类型,你可以使用 GetSelections 函数获取当前选中的子项,你需要传递一个 wxArrayTreeItemItemIds 类型的引用作为参数. Unselect 函数在单选情况下去选中当前的子项,而 UnselectAll 函数则用在多选情况下去选中所有正被选中的子项,UnselectItem 函数可以用来在多选情况下去选中某一个子项.

遍历某个树状控件的所有子项也有多种方法:你可以先使用 GetRootItem 函数获得根节点,然后使用 GetFirstChild 和 GetNextChild 遍历所有子项.使用 GetNextSibling 和 GetPrevSibling 获取某个子项后一个和前一个兄弟节点.使用 ItemHasChildren 函数判断某个子项是否有字节点,使用 GetParent 函数获取某个子项的父节点.GetCount 函数则用来返回树状控件中所有子项的个数,而 GetChildrenCount 则返回某个子项的字节点的数目.

HitTest 函数在实现你自己拖放的时候是很有用的,它使得你可以通过鼠标位置找到这个位置对应的子项以及子项的某个特定部分.具体返回值请参考相关手册中的内容.使用 GetBoundingRect 函数可以得到某个子项对应的矩形区域.

更多关于树状控件的信息请参考使用手册以及 samples/treectrl 中的 wxTreeCtrl 例子.

# 12.2 wxListCtrl

# 12.2 wxListCtrl

列表控件使用四种形式中的一种来显示子项:多列视图,多列报告视图,大图标方式以及小图标方式.下图分别对齐进行了演示.每一个子项使用一个长整型的索引来表示的,随着子项的增加,删除以及排序,这个索引可能发生变化.和树状控件不同,列表框默认采用允许多选的方式,不过你还是可以在创建窗口的时候通过类型指定只允许单选.如果你需要对所有子项进行排序,你可以提供一个排序函数.在多列报告视图中,可以给每一列增加一个标题,点过拦截标题单击事件可以实现一些附加操作比如按当前列进行排序.每一列的宽度既可以通过代码改变,也可以通过用户使用鼠标拖拽来改变.

![](img/mht11AB%281%29.tmp)

![](img/mht11AE%281%29.tmp)

![](img/mht11B1%281%29.tmp)

![](img/mht11C4%281%29.tmp)

列表控件的每个子项同样可以绑定一些客户区数据,但是和树状控件不同,每一个列表框的子项只能绑定一个长整型数据.如果你希望给每一个子项绑定一个特定对象,你需要自己实现一个从长整型到特定数据对象之间的隐射,并且自己负责那些数据对象的创建和释放.

wxListCtrl 的窗口类型

| wxLC_LIST | 使用可选的小图标进行多列显示.列数是自动计算的,不需要设置象 wxLC_REPORT 那样设置列数,换句话说,这只是一个自动换行的排列. |
| --- | --- |
| wxLC_REPORT | 单列或者多列报告方式,并且可以设置可选的标题. |
| wxLC_VIRTUAL | 指定显示的文本由应用程序动态提供; 只能用于 wxLC_REPORT 方式. |
| wxLC_ICON | 大图标方式显示,可选显示文本标签. |
| wxLC_SMALL_ICON | 小图标方式显示,可选显示文本标签 |
| wxLC_ALIGN_TOP | 图标顶端对齐. 仅适用于 Windows. |
| wxLC_ALIGN_LEFT | 图标左对齐. |
| wxLC_AUTO_ARRANGE | 图标自动排列.仅适用于 Windows. |
| wxLC_EDIT_LABELS | 标签可编辑; 当编辑动作开始时应用程序将收到通知. |
| wxLC_NO_HEADER | 在报告模式下不显示标题. |
| wxLC_SINGLE_SEL | 指定单选模式; 默认为多选模式. |
| wxLC_SORT_ASCENDING | 从小到大排序. 应用程序需要在 SortItems 中提供自己的排序函数. |
| wxLC_SORT_DESCENDING | 从大到小排序. 应用程序需要在 SortItems 中提供自己的排序函数. |
| wxLC_HRULES | 在报告模式中显示每行之间的标尺. |
| wxLC_VRULES | 在报告模式中显示每列之间的标尺. |

wxListCtrl 事件

wxListCtrl 产生 wxListEvent 类型的事件,如下表所示,事件可以父子窗口之间传递,wxListEvent::GetIndex 可以用来返回针对单个子项的事件中的子项索引.

| EVT_LIST_BEGIN_DRAG(id, func) EVT_LIST_BEGIN_RDRAG(id, func) | 这个事件在拖放开始的时候产生,如果要实现拖放,你需要提供拖放操作的剩下的部分.通过 wxListEvent::GetPoint 函数获取当前的鼠标位置. |
| --- | --- |
| EVT_LIST_BEGIN_LABEL_EDIT(id, func) EVT_LIST_END_LABEL_EDIT(id, func) | 用户正准备编辑标签或者结束编辑的时候产生,使用 Veto 函数可禁止这种编辑行为.wxListEvent::GetText 则返回当前的标签. |
| EVT_LIST_DELETE_ITEM(id, func) EVT_LIST_DELETE_ALL_ITEMS(id, func) | 事件在某个子项被删除或者所有的子项都被删除的时候产生. |
| EVT_LIST_ITEM_SELECTED(id, func) EVT_LIST_ITEM_DESELECTED(id, func) | 在子项的选中状态发生改变的时候产生. |
| EVT_LIST_ITEM_ACTIVATED(id, func) | 在某个子项被激活(鼠标双击或者通过键盘)的时候产生. |
| EVT_LIST_ITEM_FOCUSED(id, func) | 当当前焦点子项发生改变的时候产生. |
| EVT_LIST_ITEM_MIDDLE_CLICK(id, func) EVT_LIST_ITEM_RIGHT_CLICK(id, func) | 在子项被鼠标以中键或者右键单击的时候产生. |
| EVT_LIST_KEY_DOWN(id, func) | 在有针对列表控件的按键事件的时候产生. 使用 wxListEvent::GetKeyCode 函数来得到当前按键的编码. |
| EVT_LIST_INSERT_ITEM(id, func) | 新的子项插入的时候产生. |
| EVT_LIST_COL_CLICK(id, func) EVT_LIST_COL_RIGHT_CLICK(id, func) | 某一列被单击的时候产生.使用 wxListEvent::GetColumn 函数获得被单击的列索引. |
| EVT_LIST_COL_BEGIN_DRAG(id, func) EVT_LIST_COL_DRAGGING(id, func) EVT_LIST_COL_END_DRAG(id, func) | 列大小发生改变的时候或者结束改变的时候产生.你可以使用 Veto 函数来禁止这种改变.使用 wxListEvent::GetColumn 获得关联的列索引. |
| EVT_LIST_CACHE_HINT(id, func) | 如果你正在实现一个续列表控件,你可能想在某一组子项被显示之前更新其内部数据.这个事件通知你进行这个操作的最合适的时机.使用 wxListEvent::GetCacheFrom 函数和 wxListEvent::GetCacheTo 函数来获得即将更新的子项的索引范围. |

wxListItem

你需要使用 wxListItem 这个类在列表控件中进行插入,设置子项属性或者获取子项属性的操作.

SetMask 函数用来指示你希望使用的列表子项属性,如下表所示:

| wxLIST_MASK_STATE | state 属性有效. |
| --- | --- |
| wxLIST_MASK_TEXT | text 属性有效. |
| wxLIST_MASK_IMAGE | image 属性有效. |
| wxLIST_MASK_DATA | data 属性有效. |
| wxLIST_MASK_WIDTH | width 属性有效. |
| wxLIST_MASK_FORMAT | format 属性有效. |

使用 SetId 函数设置子项基于 0 的索引值,使用 SetColumn 函数设置当控件在报告模式时基于 0 的列索引.

SetState 函数则用来设置下表所示的子项状态:

| wxLIST_STATE_DONTCARE | 不关心子项状态. |
| --- | --- |
| wxLIST_STATE_DROPHILITED | 子项正被高亮显示以便接受一个拖放中的放置事件(仅适用于 Windows). |
| wxLIST_STATE_FOCUSED | 子项正拥有焦点. |
| wxLIST_STATE_SELECTED | 子项正被显示. |
| wxLIST_STATE_CUT | 子项已被剪切(仅适用于 Windows). |

SetStateMask 则用来设置当前正在更改的状态,参数和上表中的值相同.

SetText 函数用来设置标签或者标题文本.SetImage 函数用来设置子项的图片在图片列表中的索引.

SetData 函数则用来设置和子项绑定的长整型客户区数据.

SetFormat 函数用来设置列表模式时的对齐模式,其值为 wxLIST_FORMAT_LEFT, wxLIST_FORMAT_RIGHT 或 wxLIST_FORMAT_CENTRE(或者 wxLIST_FORMAT_CENTER), SetColumnWidth 函数则可以用来设置列宽.

还有一些其它的可视属性可以通过下面的这些函数设置:SetAlign, SetBackgroundColour, SetTextColour 和 SetFont.这些属性不需要设置掩码标记.并且所有 Set 函数都拥有一个对应的 Get 函数来获取相应的设置.

下面的代码用 wxListItem 来选择第二个子项,并且设置其文本标签以及字体颜色.

```cpp
wxListItem item;
item.SetId(1);
item.SetMask(wxLIST_MASK_STATE|wxLIST_MASK_TEXT);
item.SetStateMask(wxLIST_STATE_SELECTED);
item.SetState(wxLIST_STATE_SELECTED);
item.SetTextColour(*wxRED);
item.SetText(wxT("Red thing"));
listCtrl->SetItem(item); 
```

你也可以直接通过另外的函数来设置这些属性.比如 SetItemText, SetItemImage, SetItemState, GetItemText, GetItemImage, GetItemState 等等,我们马上就会谈到这些函数.

wxListCtrl 成员函数

在 windows 平台上可以调用 Arrange 函数在大图标或者小图标的方式下进行排列图标.

AssignImageList 给列表控件绑定一个图标列表,图形列表的释放由列表控件负责,SetImageList 也可以实现同样的功能,不过图片列表的释放由应用程序自己负责,使用 wxIMAGE_LIST_NORMAL 或 wxIMAGE_LIST_SMALL 来指定用于哪种显示形式,GetImageList 则用来获取对应的图片列表指针.

InsertItem 函数用来在控件的特定位置中插入一个子项.可以传递的参数包括几种形式:直接传递已经设置了成员变量的 wxListItem 对象.或者一个索引和一个字符串,一个索引和一个图片索引以及一个索引,一个标签和一个图片索引等. InsertColumn 函数则用来在报告视图中插入一列.

ClearAll 函数删除所有的子项以及报告视图中的列信息,并产生一个所有子项被删除的事件.DeleteAllItems 删除所有的子项并产生所有的子项被删除的事件.DeleteItem 删除某一个子项并产生子项被删除事件.DeleteColumn 则用来删除报告视图中的某一列.

SetItem 函数通过传递一个 wxListItem 变量来设置某个子项的属性,就象前面例子中的那样,或者你还可以传递一个索引,一个列,一个标签以及图片索引参数.GetItem 则用来查询那些经由 wxListItem::SetId 函数设置的子项索引的信息.

SetItemData 函数用来给某个子项绑定一个长整数.在绝大多数的平台上,都可以将一个指针强制转换成长整数,以便这里可以设置一个指针,但是某些平台上,这样作是不适合的,这时候你可以通过一个哈希映射来将一个长整数和某个对象对应.GetItemData 则返回某个子项绑定的长整数.需要注意的是,子项的位置随着其它子项的插入或者删除以及排序操作,将会发生改变,因此不适合用来索引一个子项,但是子项绑定的客户数据是不会随子项位置的变化而变化的,可以用来唯一标识一个子项.

SetItemImage 函数采用子项索引和图片索引两个参数给某个子项指定一个图标.

SetItemState 用来设置子项的某个状态值,必须通过一个掩码来指定哪个状态的值将改变,参考前面介绍 wxListItem 的部分.GetItemState 则用来返回某个给定子项的状态.

SetItemText 函数和 GetItemText 函数用来设置和获取某个子项的文本标签.

SetTextColour 用来设置所有子项的文本标签的颜色,SetBackgroundColour 则用来设置控件的背景颜色. SetItemTextColour 和 SetItemBackgroundColour 用来设置在报告模式下某个单独子项的文本前景色和背景色.以上函数都由对应的 Get 函数.

使用 EditLabel 函数开始编辑某个子项的标签,这将会导致一个 wxEVT_LIST_BEGIN_LABEL_EDIT 事件.你可以通过这个事件的 GetEditControl 函数获得对应编辑控件的指针(仅适用于 windows).

EnsureVisible 将使得指定的子项处于可见区域.ScrollList 用来在某个方向上滚动指定的象素(仅适用于 windows),RefreshItem 用来刷新某个子项的显示,RefreshItems 则用来刷新一系列子项的显示,这两个函数主要用在使用虚列表控件而子项的内部数据已经改变时.GetTopItem 用于在列表或者报告视图中返回第一个可见的子项.

FindItem 可以被用来查找指定标签,客户区数据或者位置的子项.GetNextItem 则用来查找某些指定状态的子项(比如所有正被选中的子项).HitTest 函数用来返回给定位置的子项.GetItemPosition 返回大图标或者小图标视图中某个子项的相对于客户区的起始座标.GetItemRect 则返回相对于客户区的座标以及子项所占的大小.

你可以动态改变列表控件的显示类型而不需要先释放再重新创建它:使用 SetSingleStyle 函数指定一个类型,比如 wxLC_REPORT.指定 false 参数来移除某一个显示类型.

在报告模式中使用 SetColumn 函数来设置某列的信息,比如标题或者宽度,参考前面的 wxListItem 的相关介绍,作为一个简单的替代版本,SetColumnWidth 用来直接设置某个列的宽度.使用 GetColumn 和 GetColumnWidth 来获取对应的设置.而 GetColumnCount 则用来获取当前的列数.

GetItemCount 函数用来返回列表控件中子项的数目,GetSelectedItemCount 函数则用来返回选中的子项的数目.GetCountPerPage 函数在大小图标方式中返回所有子项的数目,而在列表和报告模式中返回可见区域可以容纳的子项的数目.

最后,SortItems 函数可以被用来对列表控件中的子项进行排序.这个函数的参数为比较函数 wxListCtrlCompare 的地址,这个比较函数使用的参数包括两个子项对应的客户区数据,另外一个更深入的整数,然会另外一个整数,如果两个子项比较的结果为相等,则返回 0,前一个大于后一个则返回正数,前一个小于后一个则返回负数.当然,为了排序功能更好的工作,你需要给每个子项指定一个长整型的客户区数据(可以通过 wxListItem::SetData 函数).这些客户区数据被传递给比较函数以实现比较.

使用 wxListCtrl

下面的代码创建和显示了一个报告模式的列表控件,这个列表控件公有三列 10 个子项,每一行开始的地方都会显示一个 16x16 的文件图标:

```cpp
#include "wx/listctrl.h"
// 报告模式中的图片
#include "file.xpm"
#include "folder.xpm"
// 创建一个报告模式的列表控件
wxListCtrl* listCtrlReport = new wxListCtrl(
    this, wxID_ANY, wxDefaultPosition, wxSize(400, 400),
    wxLC_REPORT|wxLC_SINGLE_SEL);
// 绑定一个图片列表
wxImageList* imageList = new wxImageList(16, 16);
imageList->Add(wxIcon(folder_xpm));
imageList->Add(wxIcon(file_xpm));
listCtrlReport->AssignImageList(imageList, wxIMAGE_LIST_SMALL);
// 插入三列
wxListItem itemCol;
itemCol.SetText(wxT("Column 1"));
itemCol.SetImage(-1);
listCtrlReport->InsertColumn(0, itemCol);
listCtrlReport->SetColumnWidth(0, wxLIST_AUTOSIZE );
itemCol.SetText(wxT("Column 2"));
itemCol.SetAlign(wxLIST_FORMAT_CENTRE);
listCtrlReport->InsertColumn(1, itemCol);
listCtrlReport->SetColumnWidth(1, wxLIST_AUTOSIZE );
itemCol.SetText(wxT("Column 3"));
itemCol.SetAlign(wxLIST_FORMAT_RIGHT);
listCtrlReport->InsertColumn(2, itemCol);
listCtrlReport->SetColumnWidth(2, wxLIST_AUTOSIZE );
// 插入 10 个子项
for ( int i = 0; i &lt; 10; i++ )
{
    int imageIndex = 0;
    wxString buf;
    // 插入一个子项,字符串用于第 1 栏,
    // 图片索引为 0
    buf.Printf(wxT("This is item %d"), i);
    listCtrlReport->InsertItem(i, buf, imageIndex);
    // 子项可能由于各种原因(比如:排序)改变索引,
    // 因此将现在的索引保存为客户区数据
    listCtrlReport->SetItemData(i, i);
    // 为第 2 栏设置一个文本
    buf.Printf(wxT("Col 1, item %d"), i);
    listCtrlReport->SetItem(i, 1, buf);
    // 为第三栏设置一个文本
    buf.Printf(wxT("Item %d in column 2"), i);
    listCtrlReport->SetItem(i, 2, buf);
} 
```

虚列表控件

通常,列表控件自己保存所有子项相关的文本标签,图片以及其它可见属性的信息,对于小量数据来说,这是不成问题的,但是如果你有成千上万的子项,你可能需要实现一个虚的列表控件. 虚列表控件由应用程序保存子项的数据,你需要重载它的三个虚函数 OnGetItemLabel,OnGetItemImage 和 OnGetItemAttr,列表控件会在需要的时候调用它们.你必须调用 SetItemCount 函数来设置当前的子项个数,因为你将不会实际增加任何子项.你还可以处理 EVT_LIST_CACHE_HINT 事件以便在某一部分子项即将被显示直接更新相关的内部数据,下面是重载三个函数的一个简单的例子:

```cpp
wxString MyListCtrl::OnGetItemText(long item, long column) const
{
    return wxString::Format(wxT("Column %ld of item %ld"), column, item);
}
int MyListCtrl::OnGetItemImage(long WXUNUSED(item)) const
{
    // 全部的子项都返回 0 号索引的图片
    return 0;
}
wxListItemAttr *MyListCtrl::OnGetItemAttr(long item) const
{
    // 这个成员是内部使用用来为每一个子项保存相关属性信息
    return item % 2 ? NULL : (wxListItemAttr *)&m_attr;
} 
```

下面演示怎样创建一个虚列表控件,我们不用增加任何子项,并且故意将它的子项个数设置为一个略显夸张的值:

```cpp
virtualListCtrl = new MyListCtrl(parent, wxID_ANY,
      wxDefaultPosition, wxDefaultSize, wxLC_REPORT|wxLC_VIRTUAL);
virtualListCtrl->SetImageList(imageListSmall, wxIMAGE_LIST_SMALL);
virtualListCtrl->InsertColumn(0, wxT("First Column"));
virtualListCtrl->InsertColumn(1, wxT("Second Column"));
virtualListCtrl->SetColumnWidth(0, 150);
virtualListCtrl->SetColumnWidth(1, 150);
virtualListCtrl->SetItemCount(1000000); 
```

当控件内部数据改变时,如果子项总数改变,你需要重新设置其数目,然后调用 wxListCtrl::RefreshItem 或者 wxListCtrl::RefreshItems 来刷新子项的显示.

参考 samples/listctrl 目录中的完整的例子.

# 12.3 wxWizard

# 12.3 wxWizard

使用向导是将一堆复杂的选项变成一系列相对简单的对话框的一个好方法.它通常用来帮助应用程序的使用新手们开始使用某个特定的功能,比如,搜集建立新工程需要的信息,导出数据等等.向导中的选项通常都在应用程序别的用户界面上有体现,但是提供一个向导以便用户可以专注于那些为了完成某个任务必须要设置的选项.

向导控件通常在一个窗口内提供一系列的类似对话框的窗口,这些窗口的左边都拥有一副图片(可以是一样的也可以是不一样的),而底部则通常用一系列导航按钮,随着用户的选择进入预先设置好的下一页面,下一页面的索引是随用户的选择而变化的,因此不是所有的页面都会在一次向导执行过程中全部显示.

当标准的向导导航按钮被按下时,将会产生对应的事件,向导类或者其派生类可以捕获相应的事件.

要显示一个向导,你需要先创建一个向导(或者其派生类),然后创建子页面(作为向导的子窗口).你可以使用 wxWizardPageSimple 类(或其派生类)来创建向导页面,然后使用 wxWizardPageSimple::Chain 函数将其和一个向导绑定.或者,如果你需要动态调整页面的顺序,你可以使用 wxWizardPage 的派生类,重载其 GetPrev 和 GetNext 函数. 然后将每一个页面增加到 GetPageAreaSizer 返回的布局控件中,以便向导控件自动将自己的大小调整到最大页面的大小.

向导控件只额外定义了 wxWIZARD_EX_HELPBUTTON 扩展类型,以便在标准向导导航按钮中增加帮助按钮.注意这是一个扩展类型,需要在 Create 之前使用 SetExtraStyle 来进行设置.

wxWizard 事件

wxWizard 产生 wxWizardEvent 类型的事件,如下表所示,这些事件将首先被发送到当前页面,如果页面没有定义处理函数, 则发送到向导本身.除了 EVT_WIZARD_FINISHED 事件以外,别的事件都可以调用 wxWizardEvent::GetPage 函数返回当前页面.

| EVT_WIZARD_PAGE_CHANGED(id, func) | 当导航页面已发生变化的时候产生,使用 wxWizardEvent::GetDirection 函数判断方向(true 为向前). |
| --- | --- |
| EVT_WIZARD_PAGE_CHANGING(id, func) | 当导航页面即将变化的时候产生,这个事件可以被 Veto 以便取消这个事件.同样可以使用 wxWizardEvent::GetDirection 判断方向(true 为向前). |
| EVT_WIZARD_CANCEL(id, func) | 用户点击取消按钮的时候产生,这个事件可以被 Veto(使得事件导致的操作无效). |
| EVT_WIZARD_HELP(id, func) | 用户点击帮助按钮的时候产生. |
| EVT_WIZARD_FINISHED(id, func) | 用户点击完成按钮的时候产生.这个事件产生的时间是在向导对话框刚刚关闭以后. |

wxWizard 的成员函数

GetPageAreaSizer 函数返回用户管理所有页面的布局控件.你需要将所有的页面增加到这个布局控件中,或者将某一个可以通过 GetNext 函数访问到其它所有页面的页面增加到布局控件中,以便向导控件可以知道最大的页面的大小.如果你没有这样作,你需要在显示向导时在第一个页面显示之前调用其 FitToPage 函数,如果 wxWizardPage::GetNext 不能访问到所有的页面,你需要对每个页面调用 FitToPage 函数.

GetCurrentPage 函数返回当前活动的页面,如果 RunWizard 函数还没有被执行则返回 NULL.

GetPageSize 当前设置的页面大小. SetPageSize 则用来设置所有页面使用的页面大小,不过最好还是将页面增加到 GetPageAreaSizer 布局控件中来决定页面大小比较合适.

调用 RunWizard,传递要显示的第一个页面作为参数,以便将向导置于执行状态.如果向导执行成功这个函数返回 True,如果用户取消了向导则返回 False.

可以用 SetBorder 函数设置向导边界的大小,默认为 0.

wxWizard 使用举例

我们来看看 wxWidgets 自带的向导例子.它包含四个页面,如下图所示(页面索引并没有显示在对话框上,这样说只是为了清晰).

![](img/mhtEEF1%281%29.tmp)

![](img/mhtEF03%281%29.tmp)

![](img/mhtEF06%281%29.tmp)

![](img/mhtEF09%281%29.tmp)

第一个页面非常简单,它不需要实现任何派生类,只是简单的创建了一个 wxWizardPageSimple 类的实例,然后在其中增加了一个静态文本标签,如下所示:

```cpp
#include "wx/wizard.h"
wxWizard *wizard = new wxWizard(this, wxID_ANY,
                  wxT("Absolutely Useless Wizard"),
                  wxBitmap(wiztest_xpm),
                  wxDefaultPosition,
                  wxDEFAULT_DIALOG_STYLE | wxRESIZE_BORDER);
// 第一页
wxWizardPageSimple *page1 = new wxWizardPageSimple(wizard);
wxStaticText *text = new wxStaticText(page1, wxID_ANY,
    wxT("This wizard doesn't help you\nto do anything at all.\n")
    wxT("\n")
    wxT("The next pages will present you\nwith more useless controls."),
         wxPoint(5,5)); 
```

第二页则实现了一个 wxWizardPage 的派生类,重载了其 GetPrev 和 GetNext 函数.前者总是返回第一页,而后者则可以根据用户的选择返回下一页或者最后一页.其声明和实现如下所示:

```cpp
// 演示怎样动态改变页面顺序
// 第二页
class wxCheckboxPage : public wxWizardPage
{
public:
    wxCheckboxPage(wxWizard *parent,
                   wxWizardPage *prev,
                   wxWizardPage *next)
        : wxWizardPage(parent)
    {
        m_prev = prev;
        m_next = next;
        wxBoxSizer *mainSizer = new wxBoxSizer(wxVERTICAL);
        mainSizer->Add(
            new wxStaticText(this, wxID_ANY, wxT("Try checking the box below and\n")
                                   wxT("then going back and clearing it")),
            0, // 不需要垂直拉伸
            wxALL,
            5 // 边界宽度
        );
        m_checkbox = new wxCheckBox(this, wxID_ANY,
                         wxT("&Skip the next page"));
        mainSizer->Add(
            m_checkbox,
            0, // 不需要垂直拉伸
            wxALL,
            5 // 边界宽度
        );
        SetSizer(mainSizer);
        mainSizer->Fit(this);
    }
    // 重载 wxWizardPage 函数
    virtual wxWizardPage *GetPrev() const { return m_prev; }
    virtual wxWizardPage *GetNext() const
    {
        return m_checkbox->GetValue() ? m_next->GetNext() : m_next;
    }
private:
    wxWizardPage *m_prev,
                 *m_next;
    wxCheckBox *m_checkbox;
}; 
```

第三页实现了一个 wxRadioboxPage 类,它拦截取消向导和页面改变事件.如果你视图在这个页面取消向导,它会询问你是否真的要取消,如果你选择否,则它将使用事件的 Veto 函数来取消这个操作.OnWizardPageChanging 函数则拦截所有的页面改变事件,并根据当前单选框的选项来确定是否允许页面改变.在实际应用程序中,你可以使用这种技术来确保向导在某一页的时候必须填充某些必须的域,否则不可以前进到下一页或者你可以出于某种原因阻止用户返回以前的页面.代码列举如下:

```cpp
// 我们演示了另外一个稍微复杂一些的例子,通过拦截相应事件阻止用户向前或者向后翻页
// 或者让用户确认取消操作.
// 第三页
class wxRadioboxPage : public wxWizardPageSimple
{
public:
    // 方向枚举值
    enum
    {
        Forward, Backward, Both, Neither
    };
    wxRadioboxPage(wxWizard *parent) : wxWizardPageSimple(parent)
    {
        // 应该和上面的枚举值对应
        static wxString choices[] = { wxT("forward"), wxT("backward"), wxT("both"), wxT
("neither") };

        m_radio = new wxRadioBox(this, wxID_ANY, wxT("Allow to proceed:"),
                                 wxDefaultPosition, wxDefaultSize,
                                 WXSIZEOF(choices), choices,
                                 1, wxRA_SPECIFY_COLS);
        m_radio->SetSelection(Both);
        wxBoxSizer *mainSizer = new wxBoxSizer(wxVERTICAL);
        mainSizer->Add(
            m_radio,
            0, // 不伸缩
            wxALL,
            5 // 边界
        );
        SetSizer(mainSizer);
        mainSizer->Fit(this);
    }  
    // 事件处理函数
    void OnWizardCancel(wxWizardEvent& event)
    {
        if ( wxMessageBox(wxT("Do you really want to cancel?"), wxT("Question"),
                          wxICON_QUESTION | wxYES_NO, this) != wxYES )
        {
            // 不确认,取消
            event.Veto();
        }
    }
    void OnWizardPageChanging(wxWizardEvent& event)
    {
        int sel = m_radio->GetSelection();

        if ( sel == Both )
            return;
        if ( event.GetDirection() && sel == Forward )
            return;
        if ( !event.GetDirection() && sel == Backward )
            return;
        wxMessageBox(wxT("You can't go there"), wxT("Not allowed"),
                     wxICON_WARNING | wxOK, this);
        event.Veto();
    }
private:
    wxRadioBox *m_radio;
    DECLARE_EVENT_TABLE()
}; 
```

第四页也是最后一页,wxValidationPage,演示了重载 transferDataFromWindow 函数以便对复选框控件进行数据校验的方法.transferDataFromWindow 在无论向前或者向后按钮被点击的时候都会被调用,而且如果这个函数返回失败,将会取消向前或者向后操作.和所有的对话框用法一样,你可以不必重载 transferDataFromWindow 函数而是给对应的控件设置一个验证器.这个页面还演示了怎样更改作为向导构造函数的一个参数的默认的左图片.下面是相关的代码:

```cpp
// 第四页
class wxValidationPage : public wxWizardPageSimple
{
public:
    wxValidationPage(wxWizard *parent) : wxWizardPageSimple(parent)
    {
        m_bitmap = wxBitmap(wiztest2_xpm);
        m_checkbox = new wxCheckBox(this, wxID_ANY,
                          wxT("&Check me"));
        wxBoxSizer *mainSizer = new wxBoxSizer(wxVERTICAL);
        mainSizer->Add(
            new wxStaticText(this, wxID_ANY,
                     wxT("You need to check the checkbox\n")
                     wxT("below before going to the next page\n")),
            0,
            wxALL,
            5
        );
        mainSizer->Add(
            m_checkbox,
            0,
            wxALL,
            5
        );
        SetSizer(mainSizer);
        mainSizer->Fit(this);
    }
    virtual bool TransferDataFromWindow()
    {
        if ( !m_checkbox->GetValue() )
        {
            wxMessageBox(wxT("Check the checkbox first!"),
                         wxT("No way"),
                         wxICON_WARNING | wxOK, this);
            return false;
        }
        return true;
    }
private:
    wxCheckBox *m_checkbox;
}; 
```

下面的代码用于将所有的页面放在一起并且开始执行这个向导:

```cpp
void MyFrame::OnRunWizard(wxCommandEvent& event)
{
    wxWizard *wizard = new wxWizard(this, wxID_ANY,
                    wxT("Absolutely Useless Wizard"),
                    wxBitmap(wiztest_xpm),
                    wxDefaultPosition,
                    wxDEFAULT_DIALOG_STYLE | wxRESIZE_BORDER);
    // 向导页面既可以是一个预定义对象的实例
    wxWizardPageSimple *page1 = new wxWizardPageSimple(wizard);
    wxStaticText *text = new wxStaticText(page1, wxID_ANY,
         wxT("This wizard doesn't help you\nto do anything at all.\n")
         wxT("\n")
         wxT("The next pages will present you\nwith more useless controls."),
         wxPoint(5,5)
        );

    // ... 也可以是一个派生类的实例
    wxRadioboxPage *page3 = new wxRadioboxPage(wizard);
    wxValidationPage *page4 = new wxValidationPage(wizard);
    // 一种方便的设置页面顺序的方法
    wxWizardPageSimple::Chain(page3, page4);
    // 另外一种设置页面顺序的方法
    wxCheckboxPage *page2 = new wxCheckboxPage(wizard, page1, page3);
    page1->SetNext(page2);
    page3->SetPrev(page2);
    // 允许向导设置自适应的大小.
    wizard->GetPageAreaSizer()->Add(page1);
    if ( wizard->RunWizard(page1) )
    {
        wxMessageBox(wxT("The wizard successfully completed"),
         wxT("That's all"), wxICON_INFORMATION | wxOK);
    }
    wizard->Destroy();
} 
```

当向导被完成或者取消的时候,MyFrame 拦截了相关的事件,在这个例子中,只是简单的将其结果显示在 frame 窗口的状态条上.当然你也可以在向导类中拦截相应的事件.

完整版本的代码可以在附录 J,"代码列表"或者随书光盘的 examples/chap12 目录中找到.

# 12.4 wxHtmlWindow

# 12.4 wxHtmlWindow

wxWidgets 在其内建帮助系统中使用了 wxHtmlWindow 控件,如果你希望你的程序可以显示格式化文件,图片等(比如在生成报表的时候),你可以使用这个控件.这个控件可以支持标准 HTML 标记的一个子集,包括表格(但是不支持框架),GIF 动画,高亮链接显示,字体显示,背景色,列表,居中或者右对齐,水平条以及字符编码等.虽然它不支持 CSS,你还是可以通过已经的或者自定义的标记来达到同样的效果.其中的 HTML 文本也支持拷贝到剪贴板以及从剪贴板以普通文本的方式拷贝回应用程序.

下图演示了自带的 samples/html/test 例子编译运行以后的样子:

![](img/mht100%281%29.tmp)

和一个完整的浏览器不同,wxHtmlWindow 是小而快的,因此你可以在你的程序中大方的使用它,下图演示了在一个关于对话框中使用 wxHtmlWindow 的例子:

![](img/mht112%281%29.tmp)

下面的代码用来创建上面的例子,在这个例子中,HTML 控件首先使自己的大小满足其内部的 HTML 文本的需要,然后对话框的布局控件在调整自己的大小已满足 HTML 控件的需要.

```cpp
#include "wx/html/htmlwin.h"
void MyFrame::OnAbout(wxCommandEvent& WXUNUSED(event))
{
    wxBoxSizer *topsizer;
    wxHtmlWindow *html;
    wxDialog dlg(this, wxID_ANY, wxString(_("About")));
    topsizer = new wxBoxSizer(wxVERTICAL);
    html = new wxHtmlWindow(&dlg, wxID_ANY, wxDefaultPosition,
            wxSize(380, 160), wxHW_SCROLLBAR_NEVER);
    html->SetBorders(0);
    html->LoadPage(wxT("data/about.htm"));
    // 让 HTML 控件的大小满足其内部 HTML 文本需要的大小
    html->SetSize(html->GetInternalRepresentation()->GetWidth(),
                  html->GetInternalRepresentation()->GetHeight());
    topsizer->Add(html, 1, wxALL, 10);
    topsizer->Add(new wxStaticLine(&dlg, wxID_ANY), 0, wxEXPAND | wxLEFT | wxRIGHT, 10);
    wxButton *but = new wxButton(&dlg, wxID_OK, _("OK"));
    but->SetDefault();
    topsizer->Add(but, 0, wxALL | wxALIGN_RIGHT, 15);
    dlg.SetSizer(topsizer);
    topsizer->Fit(&dlg);
    dlg.ShowModal();
} 
```

下面列出的是例子中的 HTML 文本:

```cpp
<html>
<body bgcolor="#FFFFFF">
<table cellspacing=3 cellpadding=4 width="100%">
  <tr>
    <td bgcolor="#101010">
    <center>
    <font size=+2 color="#FFFFFF"><b><br>wxHTML Library Sample 0.2.0<br></b>
    </font>
    </center>
    </td>
  </tr>
  <tr>
    <td bgcolor="#73A183">
    <b><font size=+1>Copyright (C) 1999 Vaclav Slavik</font></b><p>
    <font size=-1>
      <table cellpadding=0 cellspacing=0 width="100%">
        <tr>
          <td width="65%">
            Vaclav Slavik<p>
          </td>
          <td valign=top>
            <img src="logo.png">
          </td>
        </tr>
      </table>
    <font size=1>
    Licenced under wxWindows Library Licence, Version 3.
    </font>
    </font>
    </td>
  </tr>
</table>
</body>
</html> 
```

请参考第四章,"基础窗口类"中,"wxListBox 和 wxCheckListBox"小节中关于 wxHtmlListBox 的内容.

wxHtmlWindow 窗口类型

| wxHW_SCROLLBAR_NEVER | 不要显示滚动条. |
| --- | --- |
| wxHW_SCROLLBAR_AUTO | 只在需要的时候显示滚动条. |
| wxHW_NO_SELECTION | 用户不可以选择其中的文本(默认可以). |

wxHtmlWindow 成员函数

GetInternalRepresentation 函数返回最顶层的 wxHtmlContainerCell 控件,使用它的 GetWidth 和 GetHeight 函数可以得到整个 HTML 区域的大致大小.

LoadFile 加载一个 HTML 文件然后显示它. LoadPage 则可以传递一个 URL. 有效的 URL 包括:

[`www.wxwindows.org/front.htm`](http://www.wxwindows.org/front.htm) # 一个 URL file:myapp.zip#zip:html/index.htm # 一个 zip 文件中特定的文件

SetPage 则直接传递要显示的 HTML 字符串.

OnCellClicked 函数在有鼠标单击某个 HTML 元素的时候被调用.它的参数包括 wxHtmlCell 指针, X 和 Y 座标, 一个 wxMouseEvent 引用.其默认行为为:如果这个元素是一个超链接,则调用 OnLinkClicked 函数.

OnLinkClicked 函数的参数为 wxHtmlLinkInfo 类型,它的默认行为是调用 LoadPage 函数加载当前链接.你可以重载这种行为,比如,在你的关于对话框中,你可以在用户单击某个链接的时候使用默认的浏览器打开你的主页.

其它可以重载的函数包括 OnOpeningURL,它在某个 URL 正被打开的时候调用, OnCellMouseHover 函数,当鼠标移过某个 HTML 元素的时候被调用.

ReadCustomization 和 WriteCustomization 函数则用来保存字体和边界信息,它们的参数包括一个 wxConfig*指针以及一个可选的位于配置中的路径.

你可以使用 SelectAll,SelectLine 和 SelectWord 函数来进行文本选择,SelectionToText 将当前选择区域以纯文本的方式返回,ToText 函数则将整页以纯文本的方式返回.

SetBorders 函数用来设置 HTML 周围的边框,SetFonts 函数则用来设置字体名称,你还可以给七个预定义的字体大小指定整数类型点单位的具体大小.

AppendToPage 函数在当前的 HTML 文本中添加内容并且刷新窗口.

你可以编写一个自定义的 wxHtmlFilter 以用来读取特定的文件,你可以使用 AddFilter 函数将其在 wxHtmlWindow 类中登记.比如,你可以写一个自定义的过滤器用来解密并显示加密的 HTML 电子杂志.

GetOpenedAnchor, GetOpenedPage 和 GetOpenedPageTitle 函数用来返回当前网页的一些相关信息.

wxHtmlWindow 有自己的访问历史机制,你可以通过 HistoryBack, HistoryForward, HistoryCanBack, HistoryCanForward 和 HistoryClear 函数来使用它.

在网页中集成窗口控件

你可以在网页中集成你自己的窗口控件,甚至包括那些你自定义的控件,如下图所示.这是通过制作一个定制的标签处理函数来实现的,这个定制的标签处理函数用来处理某些特定的标签并相应的在网页窗口中插入自己的窗口.

![](img/mht115%281%29.tmp)

上图演示了某个 HTML 窗口的一部分,是由下面的网页产生的:

```cpp
<mybind name="(small one)" x=150 y=30>
<hr>
<mybind name="a widget with floating width" float=y x="50" y=50>
<hr>
Here you can find multiple widgets at the same line:<br>
 here
<mybind name="widget_1" x="100" y=30>
...and here:
<mybind name="widget_2" x="150" y=30> 
```

用于实现特定的 HTML 标记 mybind 的代码如下所示:

```cpp
#include "wx/html/m_templ.h"
TAG_HANDLER_BEGIN(MYBIND, "MYBIND")
TAG_HANDLER_PROC(tag)
{
    wxWindow *wnd;
    int ax, ay;
    int fl = 0;
    tag.ScanParam(wxT("X"), wxT("%i"), &ax);
    tag.ScanParam(wxT("Y"), wxT("%i"), &ay);
    if (tag.HasParam(wxT("FLOAT"))) fl = ax;
    wnd = new wxTextCtrl(m_WParser->GetWindow(), wxID_ANY, tag.GetParam(wxT("NAME")),
        wxPoint(0,0), wxSize(ax, ay), wxTE_MULTILINE);
    wnd->Show(true);
    m_WParser->GetContainer()->InsertCell(new wxHtmlWidgetCell(wnd, fl));
    return false;
}
TAG_HANDLER_END(MYBIND)
TAGS_MODULE_BEGIN(MyBind)
    TAGS_MODULE_ADD(MYBIND)
TAGS_MODULE_END(MyBind) 
```

这种技术在你想使用 wxHtmlWindow 来创建整个应用程序界面的时候是比较有用的,你可以用一些教本来产生 HTML 文件以响应用户输入,就象一个网页表单一样.另外一个例子是当你需要搜集用户的注册信息的时候,你可以提供这样一个界面,其中包含文本框,用户可以输入相关的信息然后点击"注册"按钮以便将他输入的信息发送到你的组织.或者你也可以使用这种技术来给用户产生一个报告,报告中包含一些复选框,以便可以可以针对他感兴趣的内容进行详细查看.

关于制作自定义 HTML 标记的更详细的内容,请参考 samples/html/widget 目录中的例子,或者 wxWidgets 参考手册.

HTML 打印

通常如果你的应用程序中使用了 wxHtmlWindow,那么你也希望能够打印这些 HTML 文件.wxWidgets 提供了一个 wxHtmlEasyPrinting 类来用一种简单的方法打印 HTML 文件.你需要作的是创建一个这个类的实例,然后使用本地 HTML 文件调用 PreviewFile 和 PrintFile 函数.你还可以调用 PageSetup 函数来显示打印设置对话框,使用 GetPrintData 函数和 GetPageSetupData 获取用户的打印设置和页面设置.可以通过 SetHeader 和 SetFooter 函数定制页眉和页脚,其中可以包含预定义的宏@PAGENUM@(当前页码)和@PAGESCNT@(总页数).

下面的代码取自 samples/html/printing,演示了上述控件的基本使用方法以及怎样更改默认的字体:

```cpp
#include "wx/html/htmlwin.h"
#include "wx/html/htmprint.h"
MyFrame::MyFrame(const wxString& title,
                 const wxPoint& pos, const wxSize& size)
        : wxFrame((wxFrame *)NULL, wxID_ANY, title, pos, size)
{
    ...
    m_Name = wxT("testfile.htm");
    m_Prn = new wxHtmlEasyPrinting(_("Easy Printing Demo"), this);
    m_Prn->SetHeader(m_Name + wxT("(@PAGENUM@/@PAGESCNT@)&lt;hr&gt;"),
                     wxPAGE_ALL);
}
MyFrame::~MyFrame()
{
    delete m_Prn;
}
void MyFrame::OnPageSetup(wxCommandEvent& event)
{
    m_Prn->PageSetup();
}
void MyFrame::OnPrint(wxCommandEvent& event)
{
    m_Prn->PrintFile(m_Name);
}
void MyFrame::OnPreview(wxCommandEvent& event)
{
    m_Prn->PreviewFile(m_Name);
}
void MyFrame::OnPrintSmall(wxCommandEvent& event)
{
    int fontsizes[] = { 4, 6, 8, 10, 12, 20, 24 };
    m_Prn->SetFonts(wxEmptyString, wxEmptyString, fontsizes);
}
void MyFrame::OnPrintNormal(wxCommandEvent& event)
{
    m_Prn->SetFonts(wxEmptyString, wxEmptyString, 0);
}
void MyFrame::OnPrintHuge(wxCommandEvent& event)
{
    int fontsizes[] = { 20, 26, 28, 30, 32, 40, 44 };
    m_Prn->SetFonts(wxEmptyString, wxEmptyString, fontsizes);
}

wxWidgets 自带的 samples/html 例子演示了上述所有的知识,请参考. 
```

# 12.5 wxGrid

# 12.5 wxGrid

wxGrid 是一个功能强大的但是又稍微有一些复杂的窗口类用来显示表格类型的数据.你还可以使用它来作为一个包含名称和值两栏的属性编辑器.或者是通过你自己的代码使其作为一个一般意义上的表格,用来显示一个数据库或者是你自己应用程序产生的特定统计数据.在某种情况你,你还可以用它来代替列表控件中的报告显示模式,尤其是你希望在某一个特定的表格位置显示图片的时候.

网格控件拥有给电子表格增加行标题或者列标题的能力,用户可以通过拖拽行或者列之间的分割线来改变行列的大小,选择某个或者某几个特定的表格,以及通过点击某一格对其中的值进行编辑.每一个表格都有自己单独的属性包括字体,颜色,对齐方式以及自己的渲染方法(用于绘制表格)和编辑器(用于编辑表格的值).你也可以制作你自己的渲染器和编辑器:参考 include/wx/generic/grid.h 和 src/generic/grid.cpp 中的代码.默认情况下,表格将使用简单文本渲染器和编辑器.如果你使用的表格拥有不同于预定义属性的属性,你可以创建一个"属性提供者"对象,以便在程序运行期动态返回需要的表格的属性.

你也可以使用包含大量数据的虚表格,这种表格的数据由应用程序自己保管,不过不是通过 wxGrid 类.你需要使用 wxGridTableBase 的派生类并且重载其中的 GetValue, GetNumberRows 和 GetNumberCols 函数.这些函数将和你的应用程序数据(也许是数据库)打交道.然后通过 SetTable 函数给网格控件增加一份数据以便网格控件可以进行对应的显示,这种更深入的技巧在你的 wxWidgets 发行版的 samples/grid 目录中进行了演示,如下图所示:

![](img/mht5805%281%29.tmp)

下面的代码创建了一个简单的 8 行 10 列的表格:

```cpp
#include "wx/grid.h"
// 创建一个 wxGrid 对象
wxGrid* grid = new wxGrid(frame, wxID_ANY,
                          wxDefaultPosition, wxSize(400, 300));
// 然后调用 CreateGrid 设置表格的大小
grid->CreateGrid(8, 10);
// 我们可以单独给某一行或者某一列设置象素单位的宽度或高度.
grid->SetRowSize(0, 60);
grid->SetColSize(0, 120 );
// 然后设置表格内的文本内容
grid->SetCellValue(0, 0, wxT("wxGrid is good"));
// 可以指定某些表格是只读的
grid->SetCellValue(0, 3, wxT("This is read-only"));
grid->SetReadOnly(0, 3);
// 还可以指定某个表格的文本颜色
grid->SetCellValue(3, 3, wxT("green on grey"));
grid->SetCellTextColour(3, 3, *wxGREEN);
grid->SetCellBackgroundColour(3, 3, *wxLIGHT_GREY);
// 还可以指定某一列的数据采用数字的格式
// 这里我们设置第 5 列的数字格式为宽度为 6,小数点后保留 2 位的浮点数.
grid->SetColFormatFloat(5, 6, 2);
grid->SetCellValue(0, 6, wxT("3.1415"));
// 设置网格的大小为可以显示所有内容的最小大小.
grid->Fit();
// 设置其父窗口的客户区大小以便放的下整个网格.
frame->SetClientSize(grid->GetSize()); 
```

wxGrid 系统中的类

正如你已经理解到的那样,wxGrid 类是许多的类交互作用的结果.下表展示了 wxGrid 系统中的类以及它们之间的相互关系:

| wxGrid | 最主要的网格类,用来存放别的用于管理表格,行和列等的其它的窗口类. |
| --- | --- |
| wxGridTableBase | 这个类允许应用程序想虚拟的网格提供数据.SetTable 函数将其派生类的一个实例挂载入网格类. |
| wxGridCellAttr | 保存用于渲染表格的属性数据.你可以显式的通过类似 SetCellTextColour 这样的函数更改表格的属性.你也可以通过 SetAttr 函数设置某个单独的表格的属性或者是通过 SetRowAttr 和 SetColAttr 函数设置某一列或者某一行的属性. 你也可以在你自定义的表格类中通过 GetAttr 函数返回指定表格的属性. |
| wxGridCellRenderer | 这个类负责对表格进行渲染和绘制.你可以通过改变 wxGridCellAttr(或者通过 wxGrid::SetCellRenderer 函数)中对应的类的实例来改变某个表格的数据格式,你也可以直接通过 wxGrid::SetDefaultRenderer 函数更改整个表格的显示方式.这是一个虚类, 你通常需要使用一个预定义的派生类或者自己实现一个派生类.预定义的派生类包括 wxGridCellStringRenderer, wxGridCellNumberRenderer,wxGridCellFloatRenderer 和 wxGridCellBoolRenderer 等. |
| wxGridCellEditor | 这个类负责实现对表格数据的即时编辑功能.这个虚类的派生类的实例可以和某个表格,某行,某列甚至整个表格绑定. 比如说,使用 wxGrid::SetCellEditor 函数来给某个表格中的一格设置编辑器. 预定义的派生类包括 wxGridCellTextEditor, wxGridCellFloatEditor, wxGridCellBoolEditor, wxGridCellNumberEditor 和 wxGridCellChoiceEditor. |
| wxGridEvent | 这个类包含了各种网格相关事件的信息,比如鼠标在表格上单击事件,表格数据改变事件,表格被选中事件,表格编辑器被显示或者隐藏事件等. |
| wxGridRangeSelectEvent | 当用户选择一组表格以后将产生这个事件. |
| wxGridSizeEvent | 当某一行或者某一列的大小发生变化的时候产生这个事件. |
| wxGridEditorCreatedEvent | 当创建某个编辑器的时候产生这个事件. |
| wxGridCellCoords | 这个类用来代表表格中的某一格. 使用 GetRow 和 GetCol 函数获取具体的位置. |
| wxGridCellCoordsArray | 这是一个 wxGridCellCoords 类型的数组,用在函数 GetSelectedCell, GetSelectionBlockTopLeft 和 GetSelectionBlockBottomRight 的返回值中. |

wxGrid 的事件

下面列出了 wxGrid 的主要的事件映射宏.注意对于其中任何一个 EVT*GRID*...宏,都对应的还有一个 EVT*GRID_CMD*...宏,后者比前者多一个标识符参数,可以用来避免仅仅出于这个原因而重新定义新的类.

| EVT_GRID_CELL_LEFT_CLICK(func) | 用户用左键单击某个表格. |
| --- | --- |
| EVT_GRID_CELL_RIGHT_CLICK(func) | 用户用邮件单击某个表格. |
| EVT_GRID_CELL_LEFT_DCLICK(func) | 用户用左键双击某个表格. |
| EVT_GRID_CELL_RIGHT_DCLICK(func) | 用户用右键双击某个表格. |
| EVT_GRID_LABEL_LEFT_CLICK(func) | 用户用左键单击某个标题. |
| EVT_GRID_LABEL_RIGHT_CLICK(func) | 用户用右键单击某个标题. |
| EVT_GRID_LABEL_LEFT_DCLICK(func) | 用户用左键双击某个标题. |
| EVT_GRID_LABEL_RIGHT_DCLICK(func) | 用户用右键双击某个标题. |
| EVT_GRID_CELL_CHANGE(func) | 用户更改了某个表格的数据. |
| EVT_GRID_SELECT_CELL(func) | 用户选中了某个表格. |
| EVT_GRID_EDITOR_HIDDEN(func) | 某个表格的编辑器已隐藏. |
| EVT_GRID_EDITOR_SHOWN(func) | 某个表格的编辑器已显示. |
| EVT_GRID_COL_SIZE(func) | 用户通过拖拽改变了某列大小. |
| EVT_GRID_ROW_SIZE(func) | 用户通过拖拽改变某行大小. |
| EVT_GRID_RANGE_SELECT(func) | 用户选取了一组连续的单元格. |
| EVT_GRID_EDITOR_CREATED(func) | 某个单元格的编辑器已被创建. |

wxGrid 的成员函数

下面按功能列出了比较重要的 wxGrid 的成员函数.完整的成员函数列表以及其它相关类的成员函数请参考相关手册.

用于创建,删除和数据交互的函数

AppendCols 和 AppendRows 用来在最下边或者最右边增加行或者列,或者你还可以使用 InsertCols 和 InsertRows 在某个特定位置插入行或者列. 如果你使用了自定义的表格,你需要重载同名的这些函数.

GetNumberCols 和 GetNumberRows 函数用来获取和网格绑定的表格数据的列数和行数.

CreateGrid 这个函数用来以指定的行数和列数初始化网格的数据.你应该在创建网格实例以后马上调用这个函数.这个函数会为网格创建一个用于操作简单文本数据的表格,表格的所有相关数据都将保存在内存中.如果你的应用程序要处理更负责的数据类型或者更复杂的依赖关系,或者说要处理很大量的数据,你可以实现自己的表格派生类,然后使用 wxGrid::SetTable 函数将其和某个网格对象绑定.

ClearGrid 函数清除所有的网格绑定表格中的数据并且刷新网格的显示.表格本身并不会被释放.如果你使用了自定义的表格类,记得重载其 wxGridTableBase::Clear 成员函数以实现对应功能. ClearSelection 函数用来去选择所有当前选择的单元格.

DeleteCols 和 DeleteRows 函数分别用来删除列和行.

GetColLabelValue 函数返回某个指定列的标签.默认的表格类提供的列标签是从 A, B�Z, AA, AB�ZZ, AAA�等,如果你使用自定义的表格类,你可以重定义 wxGridTableBase::GetColLabelValue 函数以返回相应的标签. 类似的, GetrowLabelValue 函数用来返回指定行的标签. 默认的行标签为数字.如果你使用自定义的表格,可以重载其 wxGridTableBase::GetRowLabelValue 函数来提供自定义的默认行标签.你还可以使用 SetColLabelValue 和 SetRowLabelValue 函数来指定某个特定行列的标签.

GetCellValue 用来返回特定单元格内的文本.对于那些使用简单网格类的应用程序,你可以使用这个函数和对应的 SetCellValue 函数来操作单元格内的文本数据.对于更复杂的使用了自定义表格的程序,这个函数只能用于返回那些包含文本内容的单元格的内容.

界面相关函数

一下这些函数将会影响到网格控件的界面更新

BeginBatch 和 EndBatch 函数阻止它们之间的对网格对象的操作引起的界面更新. 界面更新将在 GetBatchCount 返回 0 的情况下更新(译者注:BeginBatch 增加这个值而 EndBatch 减少这个值).

EnableGridLines 设置允许或者禁止绘制网格线. GridLinesEnabled 则返回当前设置.

ForceRefresh 用来强制立即更新网格显示. 你应该使用这个函数来代替 wxWindow::Refresh 函数.

Fit 函数用来使得网格控件将自己的大小更改为当前行数和列数所要求的最小大小.

GetCellAlignment 返回指定单元格在垂直和水平方向上的对齐方式. GetColLabel 返回列标签的对齐方式, GetRowLabelAlignment 返回行标签的对齐方式. GetDefaultCellAlignment 返回默认单元格的对齐方式. 这些函数都有对应的 Set 开始的设置函数. 水平对齐的值可以为 wxALIGN_LEFT, wxALIGN_CENTRE (wxALIGN_CENTER)或 wxALIGN_RIGHT. 垂直对齐的枚举值可以为 wxALIGN_TOP, wxALIGN_CENTRE (wxALIGN_CENTER)或 wxALIGN_BOTTOM 之一.

GetCellBackgroundColour 返回指定单元格背景颜色. GetdefaultCellBackgroundColour 返回默认单元格背景颜色.GetLabelBackgroundColour 返回标签背景颜色.这些函数也都有对应的设置函数.

GetCellFont 函数返回指定单元格字体, GetdefaultCellFont 返回默认单元格字体. GetLabelFont 则返回标签文本字体.这些函数也都有对应的设置函数.

GetCellTextColour 返回指定单元格的文本颜色. GetdefaultCellTextColour 返回默认单元格的文本颜色,GetLabelTextColour 则返回标签文本颜色.这些函数同样拥有对应的设置函数

SetGridLineColour 和 GetGridLineColour 函数用来更改和获取网格线的颜色.

SetColAttr 和 SetRowAttr 用来设置某一行或者某一列中所有单元格的属性.

SetColFormatBool, SetColFormatNumber, SetColFormatFloat 和 SetColFormatCustom 函数可以用来更改某一列的单元格内容格式.

和 wxGrid 大小相关的函数

下面的这些函数的参数都以象素作为单位.

AutoSize 函数自动将所有单元格的大小按照其内容进行调整.对应的还有 AutoSizeColumn, AutoSizeColumns, AutoSizeRow 和 AutoSizeRows 函数.

CellToRect 返回指定单元格的大小和位置,以 wxRect 表示.

SetColMinimalWidth 和 SetRowMinimalHeight 用来设置最小行高和最小列宽, 对应的获取函数为 GetColMinimalWidth 和 GetrowMinimalHeight.

下面这些函数用来获取各种尺寸大小: GetColLabelSize, GeTDefaultColLabelSize, GeTDefaultColSize, GetColSize, GetdefaultRowLabelSize, GeTRowSize, GeTDefaultRowSize 和 GeTRowLabelSize. 它们都有对应的设置函数.

如果你想在表格周围设置额外的边距,可以使用 SetMargins 函数.

XToCol 和 YToRow 函数用来将 X 和 Y 座标转换成对应的行数或列数.要找到距离其右边线最近的对应于 X 座标值的列号,使用 XToEdgeOfCol 函数. 要找到具体其底边线最近的对应于 Y 座标的行号,使用 YToEdgeOfRow 函数.

选择和游标函数

下面的这些函数用于控制网格的游标和选择操作

GetGridCursorCol 和 GetGridCursorRow 函数返回当前游标所处的行号和列号.相应的设置函数为 SetGridCursor.

你可以用下面的函数以每次一格的方式移动游标:MoveCursorDown, MoveCursorLeft, MoveCursorRight 和 MoveCursorUp.如果希望在移动的时候跳到第一个非空单元格,则对应的使用 MoveCursorDownBlock, MoveCursorLeftBlock, MoveCursorRightBlock 和 MoveCursorUpBlock 函数.

如果想一次移动一页,可以使用 MovePageDown 和 MovePageUp 函数,页大小由网格窗口的大小决定.

GetSelectionMode 返回当前设置的选择模式,它的值为下列之一: wxGrid::wxGridSelectCells (默认模式,以单元格为单位选择), wxGrid::wxGridSelectRows (以行为单位选择), and wxGrid::wxGridSelectColumns (以列为单位进行选择). 对应的设置函数为 SetSelectionMode.

GetSelectedCells 用来获取当前选中的单元格列表,它的返回值是 wxGridCellCoordsArray 类型,其中包含所有被选中的单元格.GetSelectedCols 和 GetSelectedRows 则返回当前选中的所有行和列.因为用户可以选择多个不连续的单元格块,所以 GetSelectionBlockTopLeft 和 GetSelectionBlockBottomRight 返回的也是一个 wxGridCellCoordsArray 类型的类表. 你需要自己群举这些列表以遍历所有选中的单元格.

IsInSelection 用于查询指定的单元格是否被选中,参数为行号列号或者 wxGridCellCoords 类型. IsSelection 则返回整个网格是否有任何一格单元格被选中.

使用 SelectAll 选择整个表格, SelectCol 函数用于选择某列, SelectRow 函数用于选择某行. SelectBlock 用于选择连续的一块区域,参数为四个代表左上角及右下角的座标的整数,或者两个 wxGridCellCoords 类型的参数.

其它 wxGrid 函数

GetTable 函数返回网格用于保存内部数据的绑定表格对象.如果你使用了 CreateGrid 函数,则已经创建了一个用户保存简单文本数据的表格,或者,你已经使用 SetTable 函数设置了一个你自定义的表格类型.

GetCellEditor 和 SetCellEditor 用来获取或者设置指定单元格绑定的编辑器指针. GetdefaultEditor 和 SetDefaultEditor 用来获取和设置所有单元格使用的默认编辑器.

GetCellRenderer 和 SetCellRenderer 获取或者设置指定单元格绑定的渲染器的指针. GeTDefaultRenderer 和 SetDefaultRenderer 用来获取和设置所有单元格使用的渲染器.

ShowCellEditControl 和 HideCellEditControl 用来在相应的位置显示和隐藏当前单元格指定的编辑控件.这两个函数通常是在用户点击某个单元格来编辑单元格的值或者按下回车键和取消键(或者单击另外一个窗口)以关闭编辑器的时候自动调用的. SaveEditControlValue 函数用来将编辑器中的值传输到单元格中,这个函数在关闭网格或者从网格获取数据的时候你可以考虑调用以便网格的值反应最新编辑的值.

EnableCellEditControl 函数允许或者禁止对网格数据进行编辑. 这个函数调用的时候,网格类将会产生 wxEVT_GRID_EDITOR_SHOWN 或 wxEVT_GRID_EDITOR_HIDDEN 事件. IsCellEditControlEnabled 函数用来检测是否当前单元格可以被编辑. IsCurrentCellReadOnly 则返回是否当前单元格是只读的.

EnableDragColSize 允许或者禁止通过拖拽更改列宽. EnableDragGridSize 允许或者禁止通过拖拽改变单元格大小. EnableDragRowSize 允许或者禁止通过拖拽改变行高.

EnableEditing 的参数如果为 false,则设置整个网格为只读状态. 如果为 true,则网格恢复到默认状态.在默认状态,单元格和行以及列都可以有自己的是否可编辑标记,这个标记可以通过 wxGridCellAttribute::SetReadOnly 改变,针对单元格的只读设置可以直接通过 wxGridCellAttribute:: SetReadOnly 函数改变.IsEditable 函数返回是否整个网格是可编辑的.

你也可以通过调用 SetReadOnly 函数改变某个单元格的只读状态,IsReadOnly 则用来获取这个设置.

IsVisible 函数在单元格部分或者全部可见的时候返回 true,MakeCellVisible 则确保某个单元格出于可见区域.

# 12.6 wxTaskBarIcon

# 12.6 wxTaskBarIcon

这个类的功能是在系统托盘区(Windows,Gnome 或者 KDE)或者停靠区(Mac OS X)安装一个图标.点击这个图标将会弹出一个应用程序提供的菜单,并且在当鼠标划过图标的时候会显示一个可选的工具提示.这种技术提供了一种不必通过正规的用户界面快速访问某些重要功能的方法.应用程序可以通过更换图标来提供某些状态信息,比如用来提示电池的剩余电量或者 windows 系统上的网络连接提示.

下图演示了 wxWidgets 自带的 samples/taskbar 例子在 windows 平台上运行时的样子.它首先显示一个 wxWidgets 的图标,当鼠标移过这个图标的时候显示"wxTaskBarIconSample."工具提示,右键单击这个图标将会显示一个菜单,菜单有三个选项,选择设置新图标选项将会将图标设置为一个笑脸,并且把工具提示更改为一个新的文本.

![](img/mht809C%281%29.tmp)

以 wxTaskBarIcon 方式驱动的应用程序并不十分复杂,如下所示.自定义的类型 MyTaskBarIcon 重载了 wxTaskBarIcon 的 CreatePopupMenu 函数,并且拦截了左键双击事件,并实现了三个菜单项.

```cpp
class MyTaskBarIcon: public wxTaskBarIcon
{
public:
    MyTaskBarIcon() {};
    void OnLeftButtonDClick(wxTaskBarIconEvent&);
    void OnMenuRestore(wxCommandEvent&);
    void OnMenuExit(wxCommandEvent&);
    void OnMenuSetNewIcon(wxCommandEvent&);
    virtual wxMenu *CreatePopupMenu();
DECLARE_EVENT_TABLE()
};
enum {
    PU_RESTORE = 10001,
    PU_NEW_ICON,
    PU_EXIT,
};
BEGIN_EVENT_TABLE(MyTaskBarIcon, wxTaskBarIcon)
    EVT_MENU(PU_RESTORE, MyTaskBarIcon::OnMenuRestore)
    EVT_MENU(PU_EXIT,    MyTaskBarIcon::OnMenuExit)
    EVT_MENU(PU_NEW_ICON,MyTaskBarIcon::OnMenuSetNewIcon)
    EVT_TASKBAR_LEFT_DCLICK  (MyTaskBarIcon::OnLeftButtonDClick)
END_EVENT_TABLE()
void MyTaskBarIcon::OnMenuRestore(wxCommandEvent& )
{
    dialog->Show(true);
}
void MyTaskBarIcon::OnMenuExit(wxCommandEvent& )
{
    dialog->Close(true);
}
void MyTaskBarIcon::OnMenuSetNewIcon(wxCommandEvent&)
{
    wxIcon icon(smile_xpm);

    if (!SetIcon(icon, wxT("wxTaskBarIcon Sample - a different icon")))
        wxMessageBox(wxT("Could not set new icon."));
}
// 重载
wxMenu *MyTaskBarIcon::CreatePopupMenu()
{
    wxMenu *menu = new wxMenu;
    menu->Append(PU_RESTORE, wxT("&Restore TBTest"));
    menu->Append(PU_NEW_ICON,wxT("&Set New Icon"));
    menu->Append(PU_EXIT,    wxT("E&xit"));
    return menu;
}
void MyTaskBarIcon::OnLeftButtonDClick(wxTaskBarIconEvent&)
{
    dialog->Show(true);
} 
```

下面的代码则用来显示一个对话框并且安装初始化图标.

```cpp
#include "wx/wx.h"
#include "wx/taskbar.h"
// 定义一个新的应用程序
class MyApp: public wxApp
{
public:
    bool OnInit(void);
};
class MyDialog: public wxDialog
{
public:
    MyDialog(wxWindow* parent, const wxWindowID id, const wxString& title,
        const wxPoint& pos, const wxSize& size, const long windowStyle =
 wxDEFAULT_DIALOG_STYLE);
    ~MyDialog();
    void OnOK(wxCommandEvent& event);
    void OnExit(wxCommandEvent& event);
    void OnCloseWindow(wxCloseEvent& event);
    void Init(void);
protected:
    MyTaskBarIcon   *m_taskBarIcon;
DECLARE_EVENT_TABLE()
};
#include "../sample.xpm"
#include "smile.xpm"
MyDialog   *dialog = NULL;
IMPLEMENT_APP(MyApp)
bool MyApp::OnInit(void)
{
    // 创建主窗口
    dialog = new MyDialog(NULL, wxID_ANY, wxT("wxTaskBarIcon Test Dialog"),
 wxDefaultPosition, wxSize(365, 290));
    dialog->Show(true);
    return true;
}
BEGIN_EVENT_TABLE(MyDialog, wxDialog)
    EVT_BUTTON(wxID_OK, MyDialog::OnOK)
    EVT_BUTTON(wxID_EXIT, MyDialog::OnExit)
    EVT_CLOSE(MyDialog::OnCloseWindow)
END_EVENT_TABLE()
MyDialog::MyDialog(wxWindow* parent, const wxWindowID id, const wxString& title,
    const wxPoint& pos, const wxSize& size, const long windowStyle):
  wxDialog(parent, id, title, pos, size, windowStyle)
{
    Init();
}
MyDialog::~MyDialog()
{
    delete m_taskBarIcon;
}
void MyDialog::OnOK(wxCommandEvent& WXUNUSED(event))
{
    Show(false);
}
void MyDialog::OnExit(wxCommandEvent& WXUNUSED(event))
{
    Close(true);
}
void MyDialog::OnCloseWindow(wxCloseEvent& WXUNUSED(event))
{
    Destroy();
}
void MyDialog::Init(void)
{
  (void)new wxStaticText(this, wxID_ANY, wxT("Press 'Hide me' to hide me, Exit to quit."),
                         wxPoint(10, 20));
  (void)new wxStaticText(this, wxID_ANY, wxT("Double-click on the taskbar icon to show me
 again."),
                         wxPoint(10, 40));
  (void)new wxButton(this, wxID_EXIT, wxT("Exit"), wxPoint(185, 230), wxSize(80, 25));
  (new wxButton(this, wxID_OK, wxT("Hide me"), wxPoint(100, 230), wxSize(80,
 25)))->SetDefault();
  Centre(wxBOTH);
  m_taskBarIcon = new MyTaskBarIcon();
  if (!m_taskBarIcon->SetIcon(wxIcon(sample_xpm), wxT("wxTaskBarIcon Sample")))
        wxMessageBox(wxT("Could not set icon."));
} 
```

wxTaskBarIcon 的事件

下表列出的事件宏用来拦截 wxTaskBarIcon 相关的事件.注意不是所有的发行版都产生这些事件,因此如果你想再鼠标点击的时候弹出菜单,你应该使用重载 CreatePopupMenu 函数的方法.还要注意 wxTaskBarIconEvent 事件不会提供任何鼠标指针状态信息,比如鼠标位置之类.

| EVT_TASKBAR_MOVE(func) | 鼠标正在图标上移动. |
| --- | --- |
| EVT_TASKBAR_LEFT_DOWN(func) | 左键按下. |
| EVT_TASKBAR_LEFT_UP(func) | 左键释放. |
| EVT_TASKBAR_RIGHT_DOWN(func) | 右键按下. |
| EVT_TASKBAR_RIGHT_UP(func) | 右键释放. |
| EVT_TASKBAR_LEFT_DCLICK(func) | 左键双击. |
| EVT_TASKBAR_RIGHT_DCLICK(func) | 右键双击. |

wxTaskBarIcon 成员函数

wxTaskBarIcon 的成员函数是非常简单的,下面列出的就是它所有的成员函数.

CreatePopupMenu 是一个虚函数,应用程序已经重载这个函数以返回一个 wxMenu 指针.这个函数在对应的 wxEVT_TASKBAR_RIGHT_DOWN 事件中被调用(在 Mac OS X 系统上模拟了这个事件). wxWidgets 也会在菜单被关闭的时候自动释放这个菜单所占用的内存.

IsIconInstalled 返回是否 SetIcon 已经被成功调用.

IsOk 在 wxTaskBarIcon 对象已经被成功初始化的时候返回 True.

PopupMenu 在当前位置显示一个菜单.最好不要调用这个函数,而应该重载 CreatePopupMenu 函数,然后让 wxWidgets 帮你显示对应的菜单.

RemoveIcon 移除前一次使用 SetIcon 函数设置的图标.

SetIcon 设置一个图标(wxIcon)以及一个可选的工具提示.这个函数可以被多次调用.

# 12.7 编写自定义的控件

# 12.7 编写自定义的控件

这一小节,我们来介绍一下怎样创建自定义的控件.实际上,wxWidgets 并不具有象别的应用程序开发平台上的二进制的,支持鼠标拖入应用程序窗口的这种控件.第三方控件通常都和 wxWidgets 自带的控件比如 wxCalendarCtrl 和 wxGrid 一样,是通过源代码的方式提供的.我们这里用的 "控件"一词,含义是比较松散的,你不一定非要从"wxControl"进行派生,比如有时候,你可能更愿意从 wxScrolledWindow 产生派生类.

制作一个自定义的控件通常需要经过下面 10 个步骤:

1.  编写类声明,它应该拥有一个默认构造函数,一个完整构造函数,一个 Create 函数用于两步创建, 最好还有一个 Init 函数用于初始化内部数据.
2.  增加一个函数 DoGetBestSize,这个函数应该根据内部控件的情况(比如标签尺寸)返回该控件最合适的大小.
3.  如果已有的事件类不能满足需要,为你的控件增加新的事件类. 比如对于内部的一个按钮被按下的事件,可能使用已有的 wxCommandEvent 就可以了,但是更复杂的控件需要更复杂的事件类.并且如果你增加了新的事件类,也应改增加相应的事件映射宏.
4.  编写代码在你的新控件上显示信息.
5.  编写底层鼠标和键盘控制代码,并在其处理函数中产生你自定义的新的事件,以便应用程序可以作出相应处理.
6.  编写默认事件处理函数,以便控件可以处理那些标准事件(比如处理 wxID_COPY 或 wxID_UNDO 等标准命令的事件)或者默认的用户界面更新事件.
7.  可选的增加一个验证器类,以便应用程序可以用它使得数据和控件之间的传输变得容易,并且增加数据校验功能.
8.  可选的增加一个资源处理类,以便可以在 XRC 文件中使用你自定义的控件.
9.  在你准备使用的所有平台上测试你的自定义控件.
10.  编写文档

我们来举一个简单的例子,这个例子我们在第三章"事件处理"中曾经使用过,当时我们用来讨论自定义的事件: wxFontSelectorCtrl,你可以在随书光盘的 examples/chap03 中找到这个例子.这个类提供了一个字体预览窗口,当用户在这个窗口上点击鼠标的时候会弹出一个标准的字体选择窗口用于更改当前字体.这将导致一个新的事件 wxFontSelectorCtrlEvent,应用程序可以使用 EVT_FONT_SELECTION_CHANGED(id, func)宏来拦截这个事件.

这个控件的运行效果如下图所示,下图中静态文本下方即为我们自定义的控件.

![](img/mht6DF0%281%29.tmp)

自定义控件的类声明

下面的代码展示了自定义控件 wxFontSelectorCtrl 的类声明.其中 DoGetBestSize 只简单的返回固定值 200x40 象素,如果构造函数中没有指定大小,我们会默认使用这个大小.

```cpp
/*!
 * 一个显示显示字体预览的自定义控件.
 */
class wxFontSelectorCtrl: public wxControl
{
    DECLARE_DYNAMIC_CLASS(wxFontSelectorCtrl)
    DECLARE_EVENT_TABLE()
public:
    // 默认构造函数
    wxFontSelectorCtrl() { Init(); }
    wxFontSelectorCtrl(wxWindow* parent, wxWindowID id,
        const wxPoint& pos = wxDefaultPosition,
        const wxSize& size = wxDefaultSize,
        long style = wxSUNKEN_BORDER,
        const wxValidator& validator = wxDefaultValidator)
    {
        Init();
        Create(parent, id, pos, size, style, validator);
    }
    // Create 函数
    bool Create(wxWindow* parent, wxWindowID id,
        const wxPoint& pos = wxDefaultPosition,
        const wxSize& size = wxDefaultSize,
        long style = wxSUNKEN_BORDER,
        const wxValidator& validator = wxDefaultValidator);
    // 通用初始化函数
    void Init() { m_sampleText = wxT("abcdeABCDE"); }
    // 重载函数
    wxSize DoGetBestSize() const { return wxSize(200, 40); }
    // 事件处理函数
    void OnPaint(wxPaintEvent& event);
    void OnMouseEvent(wxMouseEvent& event);
    // 操作函数
    void SetFontData(const wxFontData& fontData) { m_fontData = fontData; };
    const wxFontData& GetFontData() const { return m_fontData; };
    wxFontData& GetFontData() { return m_fontData; };
    void SetSampleText(const wxString& sample);
    const wxString& GetSampleText() const { return m_sampleText; };
protected:
    wxFontData  m_fontData;
    wxString    m_sampleText;
}; 
```

象 wxFontDialog 中的那样,我们使用 wxFontData 类型来保存字体数据,这个类型可以额外的保存字体颜色信息.

这个控件的 RTTI(运行期类型标识)事件表和创建函数的代码列举如下:

```cpp
BEGIN_EVENT_TABLE(wxFontSelectorCtrl, wxControl)
    EVT_PAINT(wxFontSelectorCtrl::OnPaint)
    EVT_MOUSE_EVENTS(wxFontSelectorCtrl::OnMouseEvent)
END_EVENT_TABLE()
IMPLEMENT_DYNAMIC_CLASS(wxFontSelectorCtrl, wxControl)
bool wxFontSelectorCtrl::Create(wxWindow* parent, wxWindowID id,
             const wxPoint& pos, const wxSize& size, long style,
             const wxValidator& validator)
{
    if (!wxControl::Create(parent, id, pos, size, style, validator))
        return false;
    SetBackgroundColour(wxSystemSettings::GetColour(
                                          wxSYS_COLOUR_WINDOW));
    m_fontData.SetInitialFont(GetFont());
    m_fontData.SetChosenFont(GetFont());
    m_fontData.SetColour(GetForegroundColour());
    // 这个函数告诉相应的布局控件使用指定的最佳大小.
    SetBestFittingSize(size);
    return true;
} 
```

对于函数 SetBestFittingSize 的调用告诉布局控件或者使用构造函数中指定的大小,或者使用 DoGetBestSize 函数返回的大小作为这个控件的最小尺寸.当控件被增加到一个布局控件中时,根据增加函数的参数不同,这个控件的尺寸有可能被放大.

增加 DoGetBestSize 函数

实现 DoGetBestSize 函数的目的是为了让 wxWidgets 可以以此作为控件的最小尺寸以便更好的布局.如果你提供了这个函数,用户就可以在创建控件的时候使用默认的大小(wxDefaultSize)以便控件自己决定自己的大小.在这里我们只是选择一个固定值 200x40 象素,虽然是固定的,但是是合理的.当然,应用程序可以通过在构造函数中传递不同的大小来覆盖它.类似按钮或者标签这样的控件,我们可以提供一个合理的大小,当然,你的控件也可能不能够决定自己的大小,比如一个没有子窗口的滚动窗口通常无法知道自己最合适的大小,在这种情况下,你可以不理会 wxWindow::DoGetBestSize.在这种情况下,你的控件大小将取决于用户在构造函数中指定的非默认大小或者应用程序需要通过一个布局控件来自动觉得你的控件的大小.

如果你的控件包含拥有可感知大小的子窗口,你可以通过所有子窗口的大小来决定你自己控件的大小,子窗口的合适大小可以通过 GetAdjustedBestSize 函数获得.比如如果你的控件包含水平平铺的两个子窗口,你可以用下面的代码来实现 DoGetBestSize 函数:

```cpp
wxSize ContainerCtrl::DoGetBestSize() const
{
    // 获取子窗口的最佳大小
    wxSize size1, size2;
    if ( m_windowOne )
        size1 = m_windowOne->GetAdjustedBestSize();
    if ( m_windowTwo )
        size2 = m_windowTwo->GetAdjustedBestSize();
    // 因为子窗口是水平平铺的,因此
    // 通过下面的方法计算控件的最佳大小.   
    wxSize bestSize;
    bestSize.x = size1.x + size2.x;
    bestSize.y = wxMax(size1.y, size2.y);
    return bestSize;
} 
```

定义一个新的事件类

我们在第三章中详细介绍了怎样创建一个新的事件类(wxFontSelectorCtrlEvent)及其事件映射宏 (EVT_FONT_SELECTION_CHANGED).使用这个控件的应用程序可能并不需要拦截这个事件,因为可以直接使用数据传送机制会更方便. 不过在一个更复杂的例子中,事件类可以提供特别的函数,以便应用程序的事件处理函数可以从事件中获取更有用的信息,比如在这个例子中,我们可以增加 wxFontSelectorCtrlEvent::GetFont 函数以返回用户当前选择的字体.

显示控件信息

我们的自定义控件使用一个简单的重绘函数显示控件信息(一个居中放置的使用指定字体的文本),如下所示:

```cpp
void wxFontSelectorCtrl::OnPaint(wxPaintEvent& event)
{
    wxPaintDC dc(this);
    wxRect rect = GetClientRect();
    int topMargin = 2;
    int leftMargin = 2;
    dc.SetFont(m_fontData.GetChosenFont());
    wxCoord width, height;
    dc.GetTextExtent(m_sampleText, & width, & height);
    int x = wxMax(leftMargin, ((rect.GetWidth() - width) / 2)) ;
    int y = wxMax(topMargin, ((rect.GetHeight() - height) / 2)) ;
    dc.SetBackgroundMode(wxTRANSPARENT);
    dc.SetTextForeground(m_fontData.GetColour());
    dc.DrawText(m_sampleText, x, y);
    dc.SetFont(wxNullFont);
} 
```

如果你需要绘制标准元素,比如分割窗口的分割条或者一个边框,你可以考虑使用 wxNativeRenderer 类(更多信息请参考使用手册).

处理输入

我们的控件会拦截左键单击事件来显示一个字体对话框,如果用户选择了新的字体,则一个新的事件会使用 ProcessEvent 函数发送给这个控件.这个事件可以被 wxFontSelectorCtrl 的派生类处理,也可以被包含我们自定义控件的对话框或者窗口处理.

```cpp
void wxFontSelectorCtrl::OnMouseEvent(wxMouseEvent& event)
{
    if (event.LeftDown())
    {
        // 获取父窗口
        wxWindow* parent = GetParent();
        while (parent != NULL &&
               !parent->IsKindOf(CLASSINFO(wxDialog)) &&
               !parent->IsKindOf(CLASSINFO(wxFrame)))
            parent = parent->GetParent();
        wxFontDialog dialog(parent, m_fontData);
        dialog.SetTitle(_("Please choose a font"));
        if (dialog.ShowModal() == wxID_OK)
        {
            m_fontData = dialog.GetFontData();
            m_fontData.SetInitialFont(
                          dialog.GetFontData().GetChosenFont());
            Refresh();
            wxFontSelectorCtrlEvent event(
                  wxEVT_COMMAND_FONT_SELECTION_CHANGED, GetId());
            event.SetEventObject(this);
            GetEventHandler()->ProcessEvent(event);
        }
    }
} 
```

这个控件没有拦截键盘事件,不过你还是可以通过拦截它们来实现和左键单击相同的动作.你也可以在你的控件上绘制一个虚线框来表明是否其当前拥有焦点,这可以通过 wxWindow::FindFocus 函数判断,如果你决定这样作,你就需要拦截 EVT_SET_FOCUS 和 EVT_KILL_FOCUS 事件来在合式的时候进行窗口刷新.

定义默认事件处理函数

如果你看过了 wxTextCtrl 的实现代码,比如 src/msw/textctrl.cpp 中的代码,你就会发现一些标准的标识符比如 wxID_COPY,wxID_PASTE, wxID_UNDO 和 wxID_REDO 以及用户界面更新事件都有默认的处理函数.这意味着如果你的应用程序设置了将事件首先交给活动控件处理(参考第二十章:"优化你的程序"),这些标准菜单项事件或者工具按钮事件将会拥有自动处理这些事件的能力.当然我们自定义的控件还没有复杂到这种程度,不过你还是可以通过这种机制实现撤消/重做操作以及剪贴板相关操作.我们来看看 wxTextCtrl 的例子:

```cpp
BEGIN_EVENT_TABLE(wxTextCtrl, wxControl)
    ...
    EVT_MENU(wxID_COPY, wxTextCtrl::OnCopy)
    EVT_MENU(wxID_PASTE, wxTextCtrl::OnPaste)
    EVT_MENU(wxID_SELECTALL, wxTextCtrl::OnSelectAll)

    EVT_UPDATE_UI(wxID_COPY, wxTextCtrl::OnUpdateCopy)
    EVT_UPDATE_UI(wxID_PASTE, wxTextCtrl::OnUpdatePaste)
    EVT_UPDATE_UI(wxID_SELECTALL, wxTextCtrl::OnUpdateSelectAll)
    ...
END_EVENT_TABLE()
void wxTextCtrl::OnCopy(wxCommandEvent& event)
{
    Copy();
}
void wxTextCtrl::OnPaste(wxCommandEvent& event)
{
    Paste();
}
void wxTextCtrl::OnSelectAll(wxCommandEvent& event)
{
    SetSelection(-1, -1);
}
void wxTextCtrl::OnUpdateCopy(wxUpdateUIEvent& event)
{
    event.Enable( CanCopy() );
}
void wxTextCtrl::OnUpdatePaste(wxUpdateUIEvent& event)
{
    event.Enable( CanPaste() );
}
void wxTextCtrl::OnUpdateSelectAll(wxUpdateUIEvent& event)
{
    event.Enable( GetLastPosition() &gt; 0 );
} 
```

实现验证器

正如我们在第九章,"创建自定义的对话框"中看到的那样,验证器是数据在变量和控件之间传输的一种很方便地手段.当你创建自定义控件的时候,你可以考虑创建一个特殊的验证器以便应用程序可以使用它来和你的控件进行数据传输.

wxFontSelectorValidator 是我们为 wxFontSelectorCtrl 控件事件的验证器,你可以将其和一个字体指针和颜色指针或者一个 wxFontData 对象绑定.这些变量通常是在对话框的成员变量中声明的,以便对话框持续跟踪控件改变并且在对话框被关闭以后可以通过其成员返回相应的值.注意验证器的使用方式,不需要使用 new 函数创建验证器,SetValidator 函数将创建一份验证器的拷贝并且在需要的时候自动释放它.如下所示:

```cpp
wxFontSelectorCtrl* fontCtrl =
   new wxFontSelectorCtrl( this, ID_FONTCTRL,
               wxDefaultPosition, wxSize(100, 40), wxSIMPLE_BORDER );
// 或者使用字体指针和颜色指针作为参数
fontCtrl->SetValidator( wxFontSelectorValidator(& m_font,
                                                & m_fontColor) );
// ...或者使用 wxFontData 指针作为参数
fontCtrl->SetValidator( wxFontSelectorValidator(& m_fontData) ); 
```

m_font 和 m_fontColor 变量(或者 m_fontData 变量)将反应用户对字体预览控件所做的任何改变.数据传输在控件所在的对话框的 transferDataFromWindow 函数被调用的时候发生(这个函数将被默认的 wxID_OK 处理函数调用).

实现验证器必须的函数包括默认构造函数,带参数的构造函数,一个 Clone 函数用于复制自己.Validate 函数用于校验数据并在数据非法的时候显示相关信息.transferToWindow 和 transferFromWindow 则实现具体的数据传输.

下面列出了 wxFontSelectorValidator 的声明:

```cpp
/*!
 * wxFontSelectorCtrl 验证器
 */
class wxFontSelectorValidator: public wxValidator
{
DECLARE_DYNAMIC_CLASS(wxFontSelectorValidator)
public:
    // 构造函数
    wxFontSelectorValidator(wxFontData *val = NULL);
    wxFontSelectorValidator(wxFont *fontVal,
                            wxColour* colourVal = NULL);
    wxFontSelectorValidator(const wxFontSelectorValidator& val);
    // 析构函数
    ~wxFontSelectorValidator();
    // 复制自己
    virtual wxObject *Clone() const
    { return new wxFontSelectorValidator(*this); }
    // 拷贝数据到变量
    bool Copy(const wxFontSelectorValidator& val);
    // 在需要校验的时候被调用
    // 此函数应该弹出错误信息
    virtual bool Validate(wxWindow *parent);
    // 传输数据到窗口
    virtual bool TransferToWindow();
    // 传输数据到变量
    virtual bool TransferFromWindow();
    wxFontData* GetFontData() { return m_fontDataValue; }
DECLARE_EVENT_TABLE()
protected:
    wxFontData*     m_fontDataValue;
    wxFont*         m_fontValue;
    wxColour*       m_colourValue;
    // 检测是否验证器已经被正确设置
    bool CheckValidator() const;
}; 
```

建议阅读 fontctrl.cpp 文件中相关的函数实现以便对齐有进一步的了解.

实现资源处理器

如果你希望你自定义的控件可以在 XRC 文件中使用,你可以提供一个对应的资源处理器.我们在这里不对此作过多的介绍,请参考第九章,介绍 XRC 体系时候的相关介绍.同时也可以参考 wxWidgets 发行目录 include/wx/xrc 和 src/xrc 中已经的实现.

你的资源处理器在应用程序中登记以后,XRC 文件可以包含你的自定义控件了,它们也可以被你的应用程序自动加载.不过如果制作这个 XRC 文件也称为一个麻烦事.因为通常对话框设计工具都不支持动态加载自定义控件.不过通过 DialogBlocks 的简单的自定义控件定义机制,你可以指定你的自定义控件的名称和属性,以便生成正确的 XRC 文件,虽然,DialogBlocks 只能作到在其设计界面上显示一个近似的图形以代替你的自定义控件.

检测控件显示效果

当创建自定义控件的时候,你需要给 wxWidgets 一些关于控件外观的小提示.记住,wxWidgets 总会尽可能的使用系统默认的颜色和字体,不过也允许应用程序或者自定义控件在平台允许的时候自己决定这些属性.wxWidgets 也会让应用程序或者控件自己决定是否这些属性应该被其子对象继承.控制这些属性的体系确实有一些复杂,不过,除非要大量定制控件的颜色(这是不推荐的)或者实现完全属于自己的控件,程序员很少需要关心这些细节.

除非显式指明,应用程序通常会允许子窗口(其中可能包含你自定义的控件)继承它们父窗口的前景颜色和字体.然后这种行为可以通过使用 SetOwnFont 函数设置字体或者使用 SetOwnForegroundColour 函数设置前景颜色来改变.你的控件也可以通过调用 ShouldInheritColours 函数来决定是否要继承父窗口的颜色(wxControl 默认需要,而 wxWindow 则默认不需要).背景颜色通常不需要显式继承,你的控件应该通过不绘制不需要的区域的方法来保持和它的父窗口一致的背景.

为了实现属性继承,你的控件应该在构造函数中调用 InheritAttributes 函数,依平台的不同,这个函数通常可以在构造函数调用 wxControl::Create 函数的时候被调用.

某些类型实现了静态函数 GetClassDefaultAttributes,这个函数返回一个 wxVisualAttributes 对象,包含前景色,背景色以及字体设定.这个函数包含一个只有在 Mac OS X 平台上才有意义的参数 wxWindowVariant.这个函数指定的相关属性被用在类似 GetBackgroundColour 这样的函数中作为应用未指定值时候的返回值.如果你不希望默认的值被返回,你可以在你的控件中重新实现这个函数.同时你也需要重载 GetDefaultAttributes 虚函数,在其中调用 GetClassDefaultAttributes 函数以便允许针对某个特定的对象返回默认属性.如果你的控件包含一个标准控件的类似属性,你可以直接使用它,如下所示:

```cpp
// 静态函数,用于全局访问
static wxVisualAttributes GetClassDefaultAttributes(
                wxWindowVariant variant = wxWINDOW_VARIANT_NORMAL)
{
    return wxListBox::GetClassDefaultAttributes(variant);
}
// 虚函数,用户对象返回访问
virtual wxVisualAttributes GetDefaultAttributes() const
{
    return GetClassDefaultAttributes(GetWindowVariant());
}

wxVisualAttributes 的结构定义如下:

struct wxVisualAttributes
{
    // 内部标签或者文本使用的字体
    wxFont font;
    // 前景色
    wxColour colFg;
    // 背景色; 背景不为纯色背景时可能为 wxNullColour
    wxColour colBg;
}; 
```

如果你的自定义控件使用了透明背景,比如说,它是一个类似静态文本标签的控件,你应该提供一个函数 HasTransparentBackground 以便 wxWidgets 了解这个情况(目前仅支持 windows).

最后需要说明的是,如果某些操作需要在某些属性已经确定或者需要在最后的步骤才可以运行.你可以使用空闲时间来作这种处理,在第十七章,"多线程编程"的"多线程替代方案"中对此有进一步的描述.

一个更复杂一点的例子:wxThumbnailCtrl

前面我们演示的例子 wxFontSelectorCtrl 是一个非常简单的例子,很方便我们逐一介绍自定义控件的一些基本原则,比如事件,验证器等.然后,对于显示以及处理输入方面则显得有些不足.在随书光盘的 examples/chap12/thumbnail 目录中演示了一个更复杂的例子 wxThumbnailCtrl,这个控件用来显示缩略图.你可以在任何应用程序中使用它来显示图片的缩略图.(事实上它也可以显示一些别的缩略图,你可以实现自己的 wxThumbnailItem 的派生类以便使其可以支持显示某种文件类型的缩略图,比如显示那些包含图片的文档中的缩略图).

下图演示的 wxThumbnailBrowserDialog 对话框使用了 wxGenericDirCtrl 控件,这个控件使用了 wxThumbnailCtrl 控件.出于演示的目的,正在显示的这个目录放置了一些图片.

![](img/mht6DF3%281%29.tmp)

这个控件演示了下面的一些主题,当然列出的并不是全部:

*   鼠标输入: 缩略图子项可以通过单击鼠标左键进行选择或者通过按着 Ctrl 键单击鼠标左键进行多选.
*   键盘输入: 缩略图网格可以通过键盘导航,也可以通过方向键翻页,子项可以通过按住 Shift 键进行选择.
*   焦点处理: 设置和丢失焦点将导致当前的活动缩略图的显示被更新.
*   优化绘图: 通过 wxBufferedPaintDC 以及仅更新需要更新的区域的方法实现无闪烁更新当前显示.
*   滚动窗口: 这个控件继承自 wxScrolledWindow 窗口,可以根据子项的数目自动调整滚动条.
*   自定义事件: 在选择,去选择以及右键单击的时候产生 wxThumbnailEvent 类型的事件.

为了灵活处理,wxThumbnailCtrl 并不会自动加载一个目录中所有的图片,而是需要通过下面的代码显式的增加 wxThumbnailItem 对象.如下所示:

```cpp
// 创建一个多选的缩略图控件
wxThumbnailCtrl* imageBrowser =
   new wxThumbnailCtrl(parent, wxID_ANY,
         wxDefaultPosition, wxSize(300, 400),
         wxSUNKEN_BORDER|wxHSCROLL|wxVSCROLL|wxTH_TEXT_LABEL|
         wxTH_IMAGE_LABEL|wxTH_EXTENSION_LABEL|wxTH_MULTIPLE_SELECT);
// 设置一个漂亮的大的缩略图大小
imageBrowser->SetThumbnailImageSize(wxSize(200, 200));
// 在填充的时候不要显示
imageBrowser->Freeze();
// 设置一些高对比的颜色
imageBrowser->SetUnselectedThumbnailBackgroundColour(*wxRED);
imageBrowser->SetSelectedThumbnailBackgroundColour(*wxGREEN);
// 从'path'路径查找图片并且增加
wxDir dir;
if (dir.Open(path))
{
    wxString filename;
    bool cont = dir.GetFirst(&filename, wxT("*.*"), wxDIR_FILES);
    while ( cont )
    {
        wxString file = path + wxFILE_SEP_PATH + filename;
        if (wxFileExists(file) && DetermineImageType(file) != -1)
        {
            imageBrowser->Append(new wxImageThumbnailItem(file));
        }
        cont = dir.GetNext(&filename);
    }
}
// 按照名称排序
imageBrowser->Sort(wxTHUMBNAIL_SORT_NAME_DOWN);
// 标记和选择第一个缩略图
imageBrowser->Tag(0);
imageBrowser->Select(0);
// 删除第二个缩略图
imageBrowser->Delete(1);
// 显示图片
imageBrowser->Thaw(); 
```

如果你完整的阅读 thumbnailctrl.h 和 thumbnail.cpp 中的代码,你一定会得到关于自定义控件的足够的知识.另外,你也可以在你的应用程序中直接使用 wxThumbnailCtrl 控件,不要客气.

# 第十二章小结

# 第十二章小结

本章介绍了一些复杂一点的可视控件,这些控件可能你在第一次使用 wxWidgets 的时候并不会用到,但是随着你的应用程序复杂程序的增加,你几乎肯定会在你的代码中使用它们中的一个或者多个.最后一小节介绍了怎样实现自定义的控件,结合本章介绍的这些控件的源代码,你可以获得足够多这方面的提示.

同时,你可以考虑参考附录 D,"wxWidgets 提供的其它特性",里面介绍了更多 wxWidgets 自带的复杂控件.以及附录 E,"wxWidgets 的第三方工具",其中介绍了一些第三方的控件.

接下来,我们将介绍 wxWidgets 中使用的一些数据结构及其类型.