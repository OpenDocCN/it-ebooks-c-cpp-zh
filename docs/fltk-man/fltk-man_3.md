# 第三章：FLTK 的画图函数

# 第三章：FLTK 的画图函数

这章涵盖了 FLTK 提供的所有画图函数

# 3.1 何时可以画图

## 3.1 何时可以画图

什么时候可以画图，只有在几个地方可以执行画图代码。在其他地方调用该函数会出现未定义该行为的错误：

1.  最常出现的地方是在虚拟函数 Fl_Widget::draw()中。你的类需要继承一个 Fl_Widget 类，然后在自己的类中写 draw()函数。
2.  在写 boxtype 和 labeltype 函数中用到。
3.  你可以调用 Fl_Window::make_current()来增加控件的更新。用 Fl_Widget::window()找到要更新的窗口

# 3.2 FLTK 的画图函数

## 3.2 FLTK 的画图函数

调用这些画图函数之前，要包含头文件`<FL/fl_draw.H>`

FLTK 提供以下画图函数：

*   Boxes
*   Clipping
*   Colors
*   Line dashes and thickness
*   Fase Shapes
*   Complex Shapes
*   Text
*   Images
*   Overlay

### Boxes

FLTK 提供了三个函数来画 box，主要用于画按钮和其他的 UI 控件。每一个函数都提供了 box 的左上角，宽, 高等参数。

```cpp
void fl_draw_box(Fl_Boxtype b, int x, int y, int w, int h, Fl_Color c); 
```

该函数画了一个标准的 box,box 类行为 b,颜色是 c

```cpp
void fl_frame(const char *s, int x, int y, int w, int h); 
```

该函数画了一个边框，s 是 4 个字母，A 代表黑色，X 代表白色，顺序是上，左，下，右。

```cpp
void fl_frame2(const char *s, int x, int y, int w, int h); 
```

与 fl_frame 不同时 s 代表的颜色的顺序，分别是下，右，上，左。

# 3.3 剪切

## 3.3 剪切

你可以限制你的画图行为在一个矩形之内，应用 fl_push_clip(x,y,w,h),释放用 fl_pop_clip.

该矩形用象素未单位，不会受变换矩阵的影响

另外，系统会提供更新窗口的剪切域，但是比一个简单的矩形要复杂的多

```cpp
void fl_clip(int x, int y, int w, int h)
void fl_push_clip(int x, int y, int w, int h) 
```

用一个矩形剪切一个区域，并把这个区域压入堆栈。Fl_clip()不提倡，并将在以后的版本中去除该函数

```cpp
void fl_push_no_clip() 
```

压入一个空的剪切域到堆栈

```cpp
void fl_pop_clip() 
```

恢复剪切域，画图范围不再受矩形限制，fl_push_clip()一定要调用该函数。

```cpp
int fl_clip_box(int x, int y, int w, int h, int &X, int &Y, int &W, int &H) 
```

新的剪切域与旧的剪切域相交，相交的矩形位置保存在 X,Y,W,H,如果完全没有相交，则 W,H 为 0；

# 3.4 颜色

## 3.4 颜色

FLTK 将颜色处理为 32 位的整形。0-255 分别代表不同的颜色。Fl_color 枚举类型定义了前 256 个基本的颜色。

颜色值大于 255 的被认为是 24 位的 RGB 值。显示的是最接近该值的颜色。

```cpp
void fl_color(Fl_Color)  设置当前使用的颜色
Fl_Color fl_color()               返回最后设定的颜色
void fl_color(uchar r, uchar g, uchar b) 设置 rgb 颜色。 
```

# 3.5 设置线条的属性

## 3.5 设置线条的属性

FLTK 支持设定线条的宽度和类型。

```cpp
void fl_line_style(int style, int width=0, char* dashes=0) 
```

style 是以下几种类型之一，默认的是 FL_SOLID。

| FL_SOLID | ------- |
| --- | --- |
| FL_DASH | - - - - |
| FL_DOT | ....... |
| FL_DASHDOT | - . - . |
| FL_DASHDOTDOT | - .. - |
| FL_CAP_FLAT |  |
| FL_CAP_ROUND |  |
| FL_CAP_SQUARE | (extends past end point 1/2 line width) |
| FL_JOIN_MITER | (pointed) |
| FL_JOIN_ROUND |  |
| FL_JOIN_BEVEL | (flat) |

宽度是以象素值为单位，默认的 0

# 3.6 画一般的图形函数

## 3.6 画一般的图形函数

下面的函数几乎可以用来画所有的控件，这些函数画图非常精确，也非常快。他们可以在任何支持 FLTK 的平台上使用。

```cpp
void fl_point(int x,int y) //画点函数
void fl_rectf(int x,int y,int w,int h) //画一个矩形并填充内部
void fl_rectf(int x,int y,int w,int h,uchar r,uchar g,uchar b) //自定义颜色填充矩形
void fl_line(int x, int y, int x1,int y1) //画一条直线，起点为 x,y,终点为 x1,y1
void fl_line(int x int y,int x1,int y1,int x2,int y2) //画两条直线
void fl_loop(int x, int y, int x1, int y1, int x2, int y2)
void fl_loop(int x, int y, int x1, int y1, int x2, int y2, int x3, int y3) 
```

Outline a 3 or 4-sided polygon with lines.

# 3.7 画封闭的线，一次连接个顶点

## 3.7 画封闭的线，一次连接个顶点

```cpp
void fl_polygon(int x, int y, int x1, int y1, int x2, int y2)
void fl_polygon(int x, int y, int x1, int y1, int x2, int y2, int x3, int y3) 
```

Fill a 3 or 4-sided polygon. The polygon must be convex.

# 3.8 画三边形或四边形，并填充内部

## 3.8 画三边形或四边形，并填充内部

```cpp
void fl_xyline(int x, int y, int x1)
void fl_xyline(int x, int y, int x1, int y2)
void fl_xyline(int x, int y, int x1, int y2, int x3) 
```

先画一条水平的线，再画一条垂直的线条，最后画一条水平线

```cpp
void fl_yxline(int x, int y, int y1)
void fl_yxline(int x, int y, int y1, int x2)
void fl_yxline(int x, int y, int y1, int x2, int y3) 
```

首先画垂直线条，接着是水平线，最后是垂直线

```cpp
void fl_arc(int x, int y, int w, int h, double a1, double a2)
void fl_pie(int x, int y, int w, int h, double a1, double a2) 
```

画弧形线，两个角度是以三点处为 0 度，逆时针旋转，a2 必须大于或等于 a1

`fl_pie()`填充弧形内部

# 3.9 复杂图形函数

## 3.9 复杂图形函数

复杂的画图函数利用 2-D 线性转换能让你画出任意图形。这个功能与 Adobe? PostScript 语言实现的功能很相似，在 X 和 Win32 上，在画线段之前所有的转换顶点都是用整数表示，这就限制了画图的精确性。如果要画比较精确的图形，最好用 OpenGL 来画。

```cpp
void fl_push_matrix()
void fl_pop_matrix() 
```

保存和恢复当前的转换，堆栈的最大深度为 4

```cpp
void fl_scale(float x, float y)
void fl_scale(float x)
void fl_translate(float x, float y)
void fl_rotate(float d)
void fl_mult_matrix(float a, float b, float c, float d, float x, float y) 
```

在当前的转换基础上连接另外一个转换。旋转角度是度数不是弧度，逆时针旋转。

```cpp
void fl_begin_line()
void fl_end_line() 
```

开始和结束画线

```cpp
void fl_begin_loop()
void fl_end_loop() 
```

开始和结束画一系列封闭的线

```cpp
void fl_begin_polygon()
void fl_end_polygon() 
```

开始和结束画多边形并填充

```cpp
void fl_begin_complex_polygon()
void fl_gap()
void fl_end_complex_polygon() 
```

开始和结束画一个复杂的多边形并填充。这个多边形可以是凹凸不同的，不连贯的，甚至中间有空心的。调用 fl_gap()分开路径。不必也是有害的如果在第一个顶点之前或最后一个顶点之后调用 fl_gap()函数，在一行中多次调用也是不行的。

Fl_gap()只能用在 fl_begin_complex_polygon()和 fl_end_complex_polygon()之间。画多边形的轮廓，使用 fl_begin_loop 并用 fl_end_loop 和 fl_begin_loop 代替 fl_gap();

```cpp
void fl_vertex(float x, float y) 
```

在当前路径中增加一个顶点

```cpp
void fl_curve(float x, float y, float x1, float y1, float x2, float y2, float x3, float y3) 
```

在路径中增加一系列的点画 Bezier 曲线。该曲线的末端是 x,y 和 x3,y3。

```cpp
void fl_arc(float x, float y, float r, float start, float end) 
```

增加一系列的点在当前圆环的弧线上。在调用 fl_arc()之前应用 scale 和 rotate 可以得到椭圆的路径。X,y 是圆的中心，r 是半径。Fl_arc()从 start 角度画弧直到 end,按逆时针旋转。如果 end 大于 start 则它是按照顺时针转

```cpp
void fl_circle(float x, float y, float r) 
```

fl_circle 等于 fl_arc(…,0,360)，但是更快，如果你在画多边形的时候用到 圆,则必须用 fl_arc(). 文本的画法

所有的文本都字体都是适用当前字体。现在还不明确在转换情况下，位置或字符是否会改变

```cpp
void fl_draw(const char *, int x, int y)
void fl_draw(const char *, int n, int x, int y) 
```

在窗口中画出字符串，位置是靠左，接近底线

```cpp
void fl_draw(const char *, int x, int y, int w, int h, Fl_Align align, Fl_Image *img = 0, int draw_symbols = 1)
void fl_measure(const char *, int &w, int &h, int draw_symbols = 1)
int fl_height() 
```

得到当前字体的高度

```cpp
int fl_descent()
float fl_width(const char*)
float fl_width(const char*,int n)
float fl_width(uchar)
const char *fl_shortcut_label(ulong) 
```

返回按钮或菜单的快捷键字符串

# 3.10 字体

## 3.10 字体

FLTK 支持很多标准的字体，比如 Times, Helvetica/Arial, Courier, and Symbol typefaces，用户也可以自定义字体，每一个字体都有自己的索引列表

初始化只安装了 16 种字体，他们的名字是 FL_HELVETICA, FL_TIMES, FL_COURIER,另外有二个修饰体 FL_BOLD,FL_ITATIC。加上 FL_SYMBOL,FL_ZAPF_DINGBATS

不能超过 255 种字体，因为 Fl_Widget 是以一个字节来存储的。

```cpp
void fl_font(int face , int size)设置字体和大小
int fl_font()
int fl_size() 
```

得到字体和大小

# 3.11 覆盖画图函数

## 3.11 覆盖画图函数

```cpp
void fl_overlay_rect(int x, int y, int w, int h);
void fl_overlay_clear(); 
```

前者与先前颜色异或操作，后者清楚异或操作

使用该函数非常的巧妙，你应该在控件中有 handle()和 draw()函数，draw()应该调用 fl_overlay_clear()在做任何事情之前。Handle()函数应该调用 window()->make_current()然后在 FL_DRAG 事件中调用 fl_overlay_rect()，在 FL_RELEASE 事件中调用 fl_overlay_clear().