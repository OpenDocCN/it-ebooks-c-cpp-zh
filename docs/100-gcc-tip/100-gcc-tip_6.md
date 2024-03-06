# 函数属性

# 函数属性

# 禁止函数被优化掉

# 禁止函数被优化掉

## 例子

```cpp
#if (GCC_VERSION > 4000)
#define DEBUG_FUNCTION __attribute__ ((__used__))
#define DEBUG_VARIABLE __attribute__ ((__used__))
#else 
#define DEBUG_FUNCTION
#define DEBUG_VARIABLE
#endif

DEBUG_FUNCTION void
debug_bb (basic_block bb)
{
  dump_bb (bb, stderr, 0);
} 
```

## 技巧

上面的例子是 gcc 的源码。使用 gcc 的扩展功能——函数属性`__attribute__ ((__used__))`，可以指定该函数是有用的，不能被优化掉。

详情参见[gcc 手册](https://gcc.gnu.org/onlinedocs/gcc/Function-Attributes.html#Function-Attributes)

## 贡献者

xmj

# 强制函数 inline

# 强制函数永远以 inline 的形式调用

## 例子

```cpp
#if defined(__GNUC__)
#define FORCEDINLINE  __attribute__((always_inline))
#else 
#define FORCEDINLINE
#endif

FORCEDINLINE int add(int a,int b)
{
  return a+b;
} 
```

## 技巧

上面的例子是 gcc 的源码。使用 gcc 的扩展功能——函数属性`__attribute__ ((always_inline))`，可以指定该函数永远以 inline 的形式调用

详情参见[gcc 手册](https://gcc.gnu.org/onlinedocs/gcc/Inline.html)

## 贡献者

mengke