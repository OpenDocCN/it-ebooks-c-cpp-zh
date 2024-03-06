# 第四章：在 FLTK 中自定义控件

# 4.1 定制图形控件说明

## 4.1 定制图形控件说明

新控件的创建是通过继承已经存在的控件来得到的,一般控件继承 Fl_Widget 得到，组合控件继承 Fl_Group 得到

一个普通控件一般通过接收和显示一个值来与用户交互

一个组合控件包含一组子控件并处理子控件的移动，改变大小，显示或隐藏事件。Fl_Group 是所有组合控件的基类，其他组合控件比如 Fl_Pack, Fl_Scroll, Fl_Tabs , Fl_Tile, Fl_Window 都是他的子类

你也可以通过继承其他的已存在控件来得到你要的控件，通过提供不同的外观和接口。比如 Button 控件都是 Fl_Button 类的子类。他们的共同点是都是通过鼠标点击事件与用户交互。唯一不同的是按钮的外观。

# 4.2 如何开发一个控件的子类

## 4.2 如何开发一个控件的子类

你的子类可以直接继承 Fl_Widget 类，也可以继承任何 Fl_Widget 类的子类。Fl_Widget 只有四个虚拟函数，子类必须重载所有的或部分的这些函数。

### 构造函数

构造函数应该有以下参数

```cpp
MyClass(int x, int y, int w, int h, const char *label = 0); 
```

这就允许该类能很好的应用于 FLUID 中

这个构造函数必须调用基类的构造函数并传递相同的参数

```cpp
MyClass::MyClass(int x, int y, int w, int h, const char *label):
    Fl_Widget(x, y, w, h, label) {// do initialization stuff...} 
```

Fl_Widget 的保护构造函数通过传递的参数 x,y,w,h,label 分别设置 x(),y(),w(),h()和 label()并初始化其他的属性如：

```cpp
type(0);
box(FL_NO_BOX);
color(FL_BACKGROUND_COLOR);
selection_color(FL_BACKGROUND_COLOR);
labeltype(FL_NORMAL_LABEL);
labelstyle(FL_NORMAL_STYLE);
labelsize(FL_NORMAL_SIZE);
labelcolor(FL_FOREGROUND_COLOR);
align(FL_ALIGN_CENTER);
callback(default_callback,0);
flags(ACTIVE|VISIBLE);
image(0);
deimage(0); 
```

### Fl_Widget 的保护成员函数

以下的成员函数是 Fl_Widget 提供给子类的：

```cpp
Fl_Widget::clear_visible
Fl_Widget::damage
Fl_Widget::draw_box
Fl_Widget::draw_focus
Fl_Widget::draw_label
Fl_Widget::set_flag
Fl_Widget::set_visible
Fl_Widget::test_shortcut
Fl_Widget::type
void Fl_Widget::damage(uchar mask)
void Fl_Widget::damage(uchar mask, int x, int y, int w, int h)
uchar Fl_Widget::damage() 
```

第一个函数是指对象的部分需要更新。参数 mask 中的位设置传递给 damage().draw()函数能根据该值得到哪些需要重画。公共成员函数 Fl_Widget::redraw()只是简单的做 Fl_Widget::damage(FL_DAMAGE_ALL),即所有的都重画，但是你的控件真正执行的时候会调用私有成员函数 damage(n).

第二个函数指某个区域无效，需要重画。

第三个函数返回所有 damage(n)的调用所产生的位。

当重新画一个控件时，你应该先看看无效位，再决定你的控件的哪部分需要重新画。Handle()函数能够设置单独的无效位限制需要重画的数量。

```cpp
MyClass::handle(int event)
{
    if (change_to_part1) damage(1);
    if (change_to_part2) damage(2);
    if (change_to_part3) damage(4);
}

MyClass::draw()
{

    if(damage() & FL_DAMAGE_ALL)
    {

        ... draw frame/box and other static stuff ...

    }
    if (damage() & (FL_DAMAGE_ALL | 1)) draw_part1();
    if (damage() & (FL_DAMAGE_ALL | 2)) draw_part2();
    if (damage() & (FL_DAMAGE_ALL | 4)) draw_part3();

}
void Fl_Widget::draw_box() const 
```

第一个函数根据该控件的尺度画他的 box().第二个函数根据 box 的类型 b ,颜色 c 画 box.

```cpp
void Fl_Widget::draw_focus() const
void Fl_Widget::draw_focus(Fl_Boxtype b, int x, int y, int w, int h) const 
```

在一个空间的限制 box 中画出焦点筐。第二个函数允许指定另一个不同 box 来画焦点

```cpp
Fl_Widget::draw_label() const
void Fl_Widget::draw_label(int x, int y, int w, int h) const
void Fl_Widget::draw_label(int x, int y, int w, int h, Fl_Align align) const 
```

Draw()函数调用该函数来画一个控件的 label,如果标签出了该控件的 box 范围，将不会被画出。

第二种形式自定义一个 box 来画标签，比如用于移动的滑块

第三种形式可以将标签画在任意的地方

```cpp
void Fl_Widget::set_visible()
void Fl_Widget::clear_visible() 
```

与 Fl_Widget::show() Fl_Widget::hide()作用相同，但不发送 FL_SHOW,FL_HIDE 事件。

```cpp
int Fl_Widget::test_shortcut() const
static int Fl_Widget::test_shortcut(const char *s)
uchar Fl_Widget::type() const
void Fl_Widget::type(uchar t) 
```

返回一个 8 位的标示符，用于与 Forms 兼容，你也可以同于其他任何目的，设置的值应该小于 100，以免与系统的保留值冲突

# 4.3 处理事件

## 4.3 处理事件

虚拟函数 int handle(int event)被用来处理任何发送给控件的事件.他能改变控件的状态

调用 Fl_Widget::redraw()如果该控件需要重新显示

调用 Fl_Widget::damage(n)当控件需要部分更新时(假如你在 Fl_Widget::draw()函数中提供了对该函数的支持)

调用 Fl_Widget::do_callback()如果一个回调函数产生时.

调用 Fl_Widget::handle()对子控件

事件用一个整数来标识.最近事件产生的其他消息静态存储在本地,调用 Fl::event_*()可以得到.

以下是一个利用 handle()处理事件的例子，该控件的行为类似按钮同时接收 x 按键并调用回调函数

```cpp
int MyClass::handle(int event)
{
    switch(event)
    {
        case FL_PUSH:
        highlight = 1;
        redraw();
        return 1;
        case FL_DRAG:
        {
            int t = Fl::event_inside(this);
            if (t != highlight)
            {
                highlight = t;
                redraw();
            }
        }
        return 1;
        case FL_RELEASE:
        if(highlight)
        {
            highlight = 0;
            redraw();
            do_callback();
            // never do anything after a callback, as the callback
            // may delete the widget!
        }
        return 1;
        case FL_SHORTCUT:
        if(Fl::event_key() == 'x')
        {
            do_callback();
            return 1;
        }
        return 0;
        default: return Fl_Widget::handle(event);
    }
} 
```

当你的 handle()函数处理某事件后不能返回 0，若是返回 0，父控件将会把该事件发送给其他控件。

# 4.4 画控件

## 4.4 画控件

当 FLTK 需要重画控件时将调用虚拟函数 draw().只有在 damage()返回非 0 值时调用该函数，draw()返回后，damage()被清 0。Draw()应该被声明为保护成员函数，避免在不需要写画图代码时用到。

Damage()将包含从最后一次调用 draw()后 damage(n)调用产生的所有与或位信息，根据该信息只重画需要重画的位置，只有 FLTK 认为需要全部重画时才打开 FL_DAMAGE_ALL 位，比如收到 expose 事件。

### 修改控件的尺寸

resize(int x,int y,int w,int h)在控件被移动和改变大小时被调用，这些参数分别是新位置，宽度和高度。但是 x(),y(),w(),h(),还是以前的值，若要改变这些值，必须在基类中也调用 resize()函数

不需要调用 redraw()函数，至少只改变 x(),y()时不需要，因为一个组合控件有一套更有效的方法来画新的位置

### 如何制作一个组合控件

一个组合控件包括一个或多个子控件。制作组合控件必须继承 Fl_Group 类.不继承 Fl_Group 类当然也可能可以制作一个组合控件，但是你还是要重新写 Fl_Group 类里面的工作

子控件可能在类里面声明

```cpp
class MyClass : public Fl_Group
{
    Fl_Button the_button;
    Fl_Slider the_slider;
    ...
}; 
```

构造函数要初始化这些子控件。他们将被自动的 add()到 group 中。因为 Fl_Group 构造函数调用了 begin().在构造函数中不要忘记调用 end()函数

```cpp
MyClass::MyClass(int x, int y, int w, int h) :
Fl_Group(x, y, w, h),
the_button(x + 5, y + 5, 100, 20),
the_slider(x, y + 50, w, 20)
{
    ...(you could add dynamically created child widgets here)...
    end(); // don't forget to do this!
} 
```