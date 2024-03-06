# 第二章：常用的控件和属性

这章将描述 FLTK 提供的控件，并介绍如何得到和设置控件的标准属性。

# 2.1 按钮

## 2.1 按钮

FLTK 提供了很多类型的按钮

| Fl_Button | 普通按钮 |
| --- | --- |
| Fl_Check_Button | 带有选择框的按钮 |
| Fl_Light_Button | 带有指示灯的按钮 |
| Fl_Repeat_Button |  |
| Fl_Return_Button | 能被 Enter 激活的按钮 |
| Fl_Round_Button | 带有圆形选择框的按钮 |

每一个按钮都需要相应的`<FL/Fl_xyz_Button.H>`头文件。

构造函数包含了控件的位置，大小和可选的标签

```cpp
Fl_Button *button = new Fl_Button(x, y, width, height, "label");
Fl_Light_Button *lbutton = new Fl_Light_Button(x, y, width, height);
Fl_Round_Button *rbutton = new Fl_Round_Button(x, y, width, height, "label"); 
```

每一个按钮可以设置自己的类型用 type(),通过这个设置，可以让一个按钮为 push button, toggle button, or radio button:

```cpp
button->type(FL_NORMAL_BUTTON);
lbutton->type(FL_TOGGLE_BUTTON);
rbutton->type(FL_RADIO_BUTTON); 
```

对于 toggle 和 radio 按钮，value()函数返回当前的状态，开/关（0 代表关，1 代表开），set()和 clear()分别用来设置和清除 togglebutton 的状态。Radio Button 可以用 setonly()打开，同组中的其他 Radio button 按钮将关闭。

# 2.2 文本

## 2.2 文本

FLTK 提供了几种文本控件来显示和接收文本信息

| Fl_Input | 输入单行的文本 |
| --- | --- |
| Fl_Output | 输出单行的文本 |
| Fl_Multiline_Input | 多行文本输入框 |
| Fl_Multiline_Output | 多行文本输出框 |
| Fl_Text_Display | 显示多行文本控件 |
| Fl_Text_Editor | 多行文本编辑控件 |
| Fl_Help_View | 显示 HTML 文本控件 |

Fl_Output and Fl_Multiline_Output 控件允许互相 copy，但是不能改变

Value()函数用来设置和得到显示的字符串

```cpp
Fl_Input *input = new Fl_Input(x, y, width, height, "label");
input->value("Now is the time for all good men..."); 
```

这个字符串将被拷贝到该控件的存储空间内，当用 value()设置后

Fl_Text_Display and Fl_Text_Editor 用 Fl_Text_Buffer 来设置他的值，而不是一个简单的字符串。

### Valuators

| Valuators | 用来显示数字轨迹信息 |
| --- | --- |
| Fl_Counter | 带有箭头按钮的控件显示当前值 |
| Fl_Dial | 圆形手柄 |
| Fl_Roller |  |
| Fl_Scrollbar | 滚动条控件 |
| Fl_Slider | 带有手柄的滑块 |
| Fl_Value_Slider | 显示当前值的滑块 |

value()函数得到和设置控件的当前值，minimum()和 maximum()设置了控件的范围

### 群 Groups

Fl_Group 控件被用来做一般的容器控件。除了单选按钮群以外，还被用来形成 windows,tabs,scrolled windows 等控件。一下是 FLTK 提供的群类。

| Fl_Double_Window | 一个双缓冲的窗口 |
| --- | --- |
| Fl_Gl_Window | 一个 OpenGL 的窗口类 |
| Fl_Group | 容器类的基类。能被用来包含所有的控件 |
| Fl_Pack | 将控件收集到一个群区域中 |
| Fl_Scroll | 滚动窗口区域 |
| Fl_Tabs |  |
| Fl_Tile |  |
| Fl_Window |

### 设置控件的位置和大小

控件的位置和大小在你创建的时候就已经设置了，你可以通过 x(),y(),w(),h(),来得到。

改变大小和位置用 position(),resize(),size()函数。

```cpp
button->position(x, y);
group->resize(x, y, width, height);
window->size(width, height); 
```

# 2.3 颜色

## 2.3 颜色

FLTK 用一个 32 位的无符号整形存储颜色。它可能是 256 种颜色一个索引，也可能是一个 24 位的 RGB 颜色。调色板不是 X 或 WIN32 的 colormap,它是有对应固定内容的调色板

以下是一些常用的颜色的符号定义：

*   FL_BLACK
*   FL_RED
*   FL_GREEN
*   FL_YELLOW
*   FL_BLUE
*   FL_MAGENTA
*   FL_CYAN
*   FL_WHITE

这些符号是 FLTK 控件默认的颜色，详细情况请参考 Enumerations

*   FL_FOREGROUND_COLOR
*   FL_BACKGROUND_COLOR
*   FL_INACTIVE_COLOR
*   FL_SELECTION_COLOR

RGB 颜色可以用 fl_rgb_color()函数设置。

```cpp
Fl_Color c = fl_rgb_color(80,170,255); 
```

控件的颜色用 color()函数设置

```cpp
button->color(FL_RED); 
```

类似的，标签的颜色用 labelcolor()函数设置

```cpp
button->labelcolor(FL_WHITE); 
```

# 2.4Box 类型

## 2.4Box 类型

Fl_Boxtype 的类型在`<Enumeration.H>`中定义，可以用 Fl_Widget::box()设置和得到。

FL_NO_BOX 意思是任何东西都不要画，但仍然是留在窗口上。Fl_FRAME 类型只是画边框，中间不做任何改变。如图中蓝色的部分。

### 制作你自己的 Boxtypes

你可以自己制作个性风格的 boxtype.通过一个小函数，并将其加到 boxtypes 的列表中画图函数

画图函数传递的参数控件的是 box 的边界和背景颜色

```cpp
void xyz_draw(int x, int y, int w, int h, Fl_Color c)
{

} 
```

如一个简单的画图函数填充一个矩形，给定颜色并画一个黑色的外框

```cpp
void xyz_draw(int x, int y, int w, int h, Fl_Color c)
{  
    fl_color(c);  
    fl_rectf(x, y, w, h);  
    fl_color(FL_BLACK);  
    fl_rect(x, y, w, h);
} 
```

### 加入自定义的 box 类型

Fl::set_boxtype 函数添加或取代特定的 box 类型

```cpp
#define XYZ_BOX FL_FREE_BOXTYPEFl::set_boxtype(XYZ_BOX, xyz_draw, 1, 1, 2, 2); 
```

最后 4 个参数是偏移量，当画该 box 时，x,y,w,h 会减去相应的偏移量