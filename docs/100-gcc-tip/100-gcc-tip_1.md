# 信息显示

# 打印 gcc 预定义的宏信息

## 例子

```cpp
[root@linux:~]$ gcc -dM -E - < /dev/null
#define __DBL_MIN_EXP__ (-1021)
#define __FLT_MIN__ 1.17549435e-38F
#define __CHAR_BIT__ 8
#define __WCHAR_MAX__ 2147483647
#define __GCC_HAVE_SYNC_COMPARE_AND_SWAP_1 1
#define __GCC_HAVE_SYNC_COMPARE_AND_SWAP_2 1
#define __GCC_HAVE_SYNC_COMPARE_AND_SWAP_4 1
#define __DBL_DENORM_MIN__ 4.9406564584124654e-324
#define __GCC_HAVE_SYNC_COMPARE_AND_SWAP_8 1
#define __FLT_EVAL_METHOD__ 0
#define __unix__ 1
#define __x86_64 1
#define __DBL_MIN_10_EXP__ (-307)
#define __FINITE_MATH_ONLY__ 0
#define __GNUC_PATCHLEVEL__ 7 
```

## 技巧

如上所示，使用“`gcc -dM -E - < /dev/null`”命令就可以显示出 gcc 预定义的宏信息。“`-dM`”生成预定义的宏信息，“`-E`”表示预处理操作完成后就停止，不再进行下面的操作。此外，也可以使用这个命令：“`echo | gcc -dM -E -`”。

详情参见[gcc 手册](https://gcc.gnu.org/onlinedocs/gcc/Preprocessor-Options.html#index-dM-908)

## 贡献者

nanxiao

# 打印 gcc 执行的子命令

## 例子

```cpp
$ gcc -### foo.c
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/lib/gcc/x86_64-linux-gnu/4.6/lto-wrapper
Target: x86_64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu/Linaro 4.6.3-1ubuntu5' --with-bugurl=file:///usr/share/doc/gcc-4.6/README.Bugs --enable-languages=c,c++,fortran,objc,obj-c++ --prefix=/usr --program-suffix=-4.6 --enable-shared --enable-linker-build-id --with-system-zlib --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --with-gxx-include-dir=/usr/include/c++/4.6 --libdir=/usr/lib --enable-nls --with-sysroot=/ --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --enable-gnu-unique-object --enable-plugin --enable-objc-gc --disable-werror --with-arch-32=i686 --with-tune=generic --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu
Thread model: posix
gcc version 4.6.3 (Ubuntu/Linaro 4.6.3-1ubuntu5) 
COLLECT_GCC_OPTIONS='-mtune=generic' '-march=x86-64'
 /usr/lib/gcc/x86_64-linux-gnu/4.6/cc1 -quiet -imultilib . -imultiarch x86_64-linux-gnu foo.c -quiet -dumpbase foo.c "-mtune=generic" "-march=x86-64" -auxbase foo -fstack-protector -o /tmp/ccezMraJ.s
COLLECT_GCC_OPTIONS='-mtune=generic' '-march=x86-64'
 as --64 -o /tmp/cc9Ce7IE.o /tmp/ccezMraJ.s
COMPILER_PATH=/usr/lib/gcc/x86_64-linux-gnu/4.6/:/usr/lib/gcc/x86_64-linux-gnu/4.6/:/usr/lib/gcc/x86_64-linux-gnu/:/usr/lib/gcc/x86_64-linux-gnu/4.6/:/usr/lib/gcc/x86_64-linux-gnu/
LIBRARY_PATH=/home/xmj/install/cap-llvm-3.4/lib/../lib/:/usr/lib/gcc/x86_64-linux-gnu/4.6/:/usr/lib/gcc/x86_64-linux-gnu/4.6/../../../x86_64-linux-gnu/:/usr/lib/gcc/x86_64-linux-gnu/4.6/../../../../lib/:/lib/x86_64-linux-gnu/:/lib/../lib/:/usr/lib/x86_64-linux-gnu/:/usr/lib/../lib/:/home/xmj/install/cap-llvm-3.4/lib/:/usr/lib/gcc/x86_64-linux-gnu/4.6/../../../:/lib/:/usr/lib/
COLLECT_GCC_OPTIONS='-mtune=generic' '-march=x86-64'
 /usr/lib/gcc/x86_64-linux-gnu/4.6/collect2 "--sysroot=/" --build-id --no-add-needed --as-needed --eh-frame-hdr -m elf_x86_64 "--hash-style=gnu" -dynamic-linker /lib64/ld-linux-x86-64.so.2 -z relro /usr/lib/gcc/x86_64-linux-gnu/4.6/../../../x86_64-linux-gnu/crt1.o /usr/lib/gcc/x86_64-linux-gnu/4.6/../../../x86_64-linux-gnu/crti.o /usr/lib/gcc/x86_64-linux-gnu/4.6/crtbegin.o -L/home/xmj/install/cap-llvm-3.4/lib/../lib -L/usr/lib/gcc/x86_64-linux-gnu/4.6 -L/usr/lib/gcc/x86_64-linux-gnu/4.6/../../../x86_64-linux-gnu -L/usr/lib/gcc/x86_64-linux-gnu/4.6/../../../../lib -L/lib/x86_64-linux-gnu -L/lib/../lib -L/usr/lib/x86_64-linux-gnu -L/usr/lib/../lib -L/home/xmj/install/cap-llvm-3.4/lib -L/usr/lib/gcc/x86_64-linux-gnu/4.6/../../.. /tmp/cc9Ce7IE.o -lgcc --as-needed -lgcc_s --no-as-needed -lc -lgcc --as-needed -lgcc_s --no-as-needed /usr/lib/gcc/x86_64-linux-gnu/4.6/crtend.o /usr/lib/gcc/x86_64-linux-gnu/4.6/../../../x86_64-linux-gnu/crtn.o 
```

## 技巧

如上所示，使用`-###`选项可以打印出 gcc 所执行的各个子命令，分别为，

cc1：

```cpp
 /usr/lib/gcc/x86_64-linux-gnu/4.6/cc1 -quiet -imultilib . -imultiarch x86_64-linux-gnu foo.c -quiet -dumpbase foo.c "-mtune=generic" "-march=x86-64" -auxbase foo -fstack-protector -o /tmp/ccezMraJ.s 
```

as：

```cpp
 as --64 -o /tmp/cc9Ce7IE.o /tmp/ccezMraJ.s 
```

collect2：

```cpp
 /usr/lib/gcc/x86_64-linux-gnu/4.6/collect2 "--sysroot=/" --build-id --no-add-needed --as-needed --eh-frame-hdr -m elf_x86_64 "--hash-style=gnu" -dynamic-linker /lib64/ld-linux-x86-64.so.2 -z relro /usr/lib/gcc/x86_64-linux-gnu/4.6/../../../x86_64-linux-gnu/crt1.o /usr/lib/gcc/x86_64-linux-gnu/4.6/../../../x86_64-linux-gnu/crti.o /usr/lib/gcc/x86_64-linux-gnu/4.6/crtbegin.o -L/home/xmj/install/cap-llvm-3.4/lib/../lib -L/usr/lib/gcc/x86_64-linux-gnu/4.6 -L/usr/lib/gcc/x86_64-linux-gnu/4.6/../../../x86_64-linux-gnu -L/usr/lib/gcc/x86_64-linux-gnu/4.6/../../../../lib -L/lib/x86_64-linux-gnu -L/lib/../lib -L/usr/lib/x86_64-linux-gnu -L/usr/lib/../lib -L/home/xmj/install/cap-llvm-3.4/lib -L/usr/lib/gcc/x86_64-linux-gnu/4.6/../../.. /tmp/cc9Ce7IE.o -lgcc --as-needed -lgcc_s --no-as-needed -lc -lgcc --as-needed -lgcc_s --no-as-needed /usr/lib/gcc/x86_64-linux-gnu/4.6/crtend.o /usr/lib/gcc/x86_64-linux-gnu/4.6/../../../x86_64-linux-gnu/crtn.o 
```

这个跟使用`-v`所显示的内容差不多，区别在于使用`-###`是只打印，不实际执行具体的命令。手册里提到，它的一种用法，就是在脚本里使用这个选项，来获得 gcc 所调用的各个子命令行。

详情参见[gcc 手册](https://gcc.gnu.org/onlinedocs/gcc/Overall-Options.html#Overall-Options)

## 贡献者

xmj

# 打印优化级别的对应选项

## 例子

```cpp
$ gcc -Q --help=optimizers
The following options control optimizations:
  -O<number>                          
  -Ofast                              
  -Os                                 
  -falign-functions                   [disabled]
  -falign-jumps                       [disabled]
  -falign-labels                      [disabled]
  -falign-loops                       [disabled]
  -fasynchronous-unwind-tables         [enabled]
  -fbranch-count-reg                  [enabled]
  -fbranch-probabilities              [disabled]
  -fbranch-target-load-optimize     [disabled]
  -fbranch-target-load-optimize2     [disabled]
  -fbtr-bb-exclusive                  [disabled]
  -fcaller-saves                      [disabled]
  -fcombine-stack-adjustments         [disabled]
  -fcommon                            [enabled]
  -fcompare-elim                      [disabled]
  -fconserve-stack                    [disabled]
  -fcprop-registers                   [disabled]
  -fcrossjumping                      [disabled]
  -fcse-follow-jumps                  [disabled]
  -fcx-fortran-rules                  [disabled]
  -fcx-limited-range                  [disabled]
  -fdata-sections                     [disabled]
  -fdce                               [enabled]
  -fdefer-pop                         [disabled]
  -fdelayed-branch                    [disabled]
  -fdelete-null-pointer-checks         [enabled]
  -fdevirtualize                      [disabled]
  -fdse                               [enabled]
  -fearly-inlining                    [enabled]
  -fexceptions                        [disabled]
  -fexpensive-optimizations           [disabled]
  -ffinite-math-only                  [disabled]
  -ffloat-store                       [disabled]
  -fforward-propagate                 [disabled]
  -fgcse                              [disabled]
  -fgcse-after-reload                 [disabled]
  -fgcse-las                          [disabled]
  -fgcse-lm                           [enabled]
  -fgcse-sm                           [disabled]
  -fgraphite-identity                 [disabled]
  -fguess-branch-probability          [disabled]
  -fhandle-exceptions                 
  -fif-conversion                     [disabled]
  -fif-conversion2                    [disabled]
  -finline-functions                  [disabled]
  -finline-functions-called-once     [enabled]
  -finline-small-functions            [disabled]
  -fipa-cp                            [disabled]
  -fipa-cp-clone                      [disabled]
  -fipa-matrix-reorg                  [disabled]
  -fipa-profile                       [disabled]
  -fipa-pta                           [disabled]
  -fipa-pure-const                    [disabled]
  -fipa-reference                     [disabled]
  -fipa-sra                           [disabled]
  -fivopts                            [enabled]
  -fjump-tables                       [enabled]
  -floop-block                        [disabled]
  -floop-flatten                      [disabled]
  -floop-interchange                  [disabled]
  -floop-parallelize-all              [disabled]
  -floop-strip-mine                   [disabled]
  -flto-report                        [disabled]
  -fltrans                            [disabled]
  -fmath-errno                        [enabled]
  -fmerge-all-constants               [disabled]
  -fmerge-constants                   [disabled]
  -fmodulo-sched                      [disabled]
  -fmove-loop-invariants              [enabled]
  -fnon-call-exceptions               [disabled]
  -fnothrow-opt                       [disabled]
  -fomit-frame-pointer                [disabled]
  -foptimize-register-move            [disabled]
  -foptimize-sibling-calls            [disabled]
  -fpack-struct                       [disabled]
  -fpack-struct=<number>              
  -fpeel-loops                        [disabled]
  -fpeephole                          [enabled]
  -fpeephole2                         [disabled]
  -fpredictive-commoning              [disabled]
  -fprefetch-loop-arrays              [enabled]
  -freg-struct-return                 [disabled]
  -fregmove                           [disabled]
  -frename-registers                  [enabled]
  -freorder-blocks                    [disabled]
  -freorder-blocks-and-partition     [disabled]
  -freorder-functions                 [disabled]
  -frerun-cse-after-loop              [disabled]
  -freschedule-modulo-scheduled-loops     [disabled]
  -frounding-math                     [disabled]
  -frtti                              [enabled]
  -fsched-critical-path-heuristic     [enabled]
  -fsched-dep-count-heuristic         [enabled]
  -fsched-group-heuristic             [enabled]
  -fsched-interblock                  [enabled]
  -fsched-last-insn-heuristic         [enabled]
  -fsched-pressure                    [disabled]
  -fsched-rank-heuristic              [enabled]
  -fsched-spec                        [enabled]
  -fsched-spec-insn-heuristic         [enabled]
  -fsched-spec-load                   [disabled]
  -fsched-spec-load-dangerous         [disabled]
  -fsched-stalled-insns               [disabled]
  -fsched-stalled-insns-dep           [enabled]
  -fsched2-use-superblocks            [disabled]
  -fschedule-insns                    [disabled]
  -fschedule-insns2                   [disabled]
  -fsection-anchors                   [disabled]
  -fsel-sched-pipelining              [disabled]
  -fsel-sched-pipelining-outer-loops     [disabled]
  -fsel-sched-reschedule-pipelined     [disabled]
  -fselective-scheduling              [disabled]
  -fselective-scheduling2             [disabled]
  -fshort-double                      [disabled]
  -fshort-enums                       [enabled]
  -fshort-wchar                       [disabled]
  -fsignaling-nans                    [disabled]
  -fsigned-zeros                      [enabled]
  -fsingle-precision-constant         [disabled]
  -fsplit-ivs-in-unroller             [enabled]
  -fsplit-wide-types                  [disabled]
  -fstrict-aliasing                   [disabled]
  -fstrict-enums                      [disabled]
  -fthread-jumps                      [disabled]
  -fno-threadsafe-statics             [enabled]
  -ftoplevel-reorder                  [enabled]
  -ftrapping-math                     [enabled]
  -ftrapv                             [disabled]
  -ftree-bit-ccp                      [disabled]
  -ftree-builtin-call-dce             [disabled]
  -ftree-ccp                          [disabled]
  -ftree-ch                           [disabled]
  -ftree-copy-prop                    [disabled]
  -ftree-copyrename                   [disabled]
  -ftree-cselim                       [enabled]
  -ftree-dce                          [disabled]
  -ftree-dominator-opts               [disabled]
  -ftree-dse                          [disabled]
  -ftree-forwprop                     [enabled]
  -ftree-fre                          [disabled]
  -ftree-loop-distribute-patterns     [disabled]
  -ftree-loop-distribution            [disabled]
  -ftree-loop-if-convert              [enabled]
  -ftree-loop-if-convert-stores     [disabled]
  -ftree-loop-im                      [enabled]
  -ftree-loop-ivcanon                 [enabled]
  -ftree-loop-optimize                [enabled]
  -ftree-lrs                          [disabled]
  -ftree-phiprop                      [enabled]
  -ftree-pre                          [disabled]
  -ftree-pta                          [enabled]
  -ftree-reassoc                      [enabled]
  -ftree-scev-cprop                   [enabled]
  -ftree-sink                         [disabled]
  -ftree-slp-vectorize                [enabled]
  -ftree-sra                          [disabled]
  -ftree-switch-conversion            [disabled]
  -ftree-ter                          [disabled]
  -ftree-vect-loop-version            [enabled]
  -ftree-vectorize                    [disabled]
  -ftree-vrp                          [disabled]
  -funit-at-a-time                    [enabled]
  -funroll-all-loops                  [disabled]
  -funroll-loops                      [disabled]
  -funsafe-loop-optimizations         [disabled]
  -funsafe-math-optimizations         [disabled]
  -funswitch-loops                    [disabled]
  -funwind-tables                     [disabled]
  -fvar-tracking                      [enabled]
  -fvar-tracking-assignments          [enabled]
  -fvar-tracking-assignments-toggle     [disabled]
  -fvar-tracking-uninit               [disabled]
  -fvariable-expansion-in-unroller     [disabled]
  -fvect-cost-model                   [enabled]
  -fvpt                               [disabled]
  -fweb                               [enabled]
  -fwhole-program                     [disabled]
  -fwpa                               [disabled]
  -fwrapv                             [disabled] 
```

## 技巧

如上所示，使用`-Q --help=optimizers`选项可以打印出 gcc 的所有优化（相关的）选项，以及缺省情况下它们是否打开。类似的，你也可以查看不同优化级别下，这些优化选项是否打开：

```cpp
$ gcc -Q --help=optimizers -O
$ gcc -Q --help=optimizers -O1
$ gcc -Q --help=optimizers -O2
$ gcc -Q --help=optimizers -O3
$ gcc -Q --help=optimizers -Og
$ gcc -Q --help=optimizers -Os
$ gcc -Q --help=optimizers -Ofast 
```

详情参见[gcc 手册](https://gcc.gnu.org/onlinedocs/gcc/Overall-Options.html#Overall-Options)

## 贡献者

xmj

# 打印彩色诊断信息

## 技巧

这是 gcc-4.9 新增的功能，可以通过定义环境变量`GCC_COLORS`来彩色打印诊断信息。

也可以使用选项`-fdiagnostics-color`来设定。

详情参见[gcc 手册](https://gcc.gnu.org/onlinedocs/gcc/Language-Independent-Options.html#Language-Independent-Options)

## 贡献者

xmj

# 打印头文件搜索路径

## 例子

```cpp
$ gcc -v foo.c
...
ignoring nonexistent directory "/usr/local/include/x86_64-linux-gnu"
ignoring nonexistent directory "/usr/lib/gcc/x86_64-linux-gnu/4.6/../../../../x86_64-linux-gnu/include"
#include "..." search starts here:
#include <...> search starts here:
 /usr/lib/gcc/x86_64-linux-gnu/4.6/include
 /usr/local/include
 /usr/lib/gcc/x86_64-linux-gnu/4.6/include-fixed
 /usr/include/x86_64-linux-gnu
 /usr/include
End of search list.
... 
```

## 技巧

如上所示，使用`-v`选项可以打印出 gcc 搜索头文件的路径和顺序。当然，也可以使用`-###`选项

## 贡献者

xmj

# 打印连接库的具体路径

## 例子

```cpp
$ gcc -print-file-name=libc.a
/usr/lib/gcc/x86_64-linux-gnu/4.6/../../../x86_64-linux-gnu/libc.a 
```

## 技巧

如上所示，使用`-print-file-name`选项就可以显示出 gcc 究竟会连接哪个 libc 库了。

详情参见[gcc 手册](https://gcc.gnu.org/onlinedocs/gcc/Debugging-Options.html#index-print-file-name-777)

## 贡献者

xmj