# 高级事件循环

# Advanced event loops

libuv 提供了非常多的控制 event-loop 的方法，你能通过使用多 loop 来实现很多有趣的功能。你还可以将 libuv 的 event loop 嵌入到其它基于 event-loop 的库中。比如，想象着一个基于 Qt 的 UI，然后 Qt 的 event-loop 是由 libuv 驱动的，做着加强级的系统任务。

## Stopping an event loop

`uv_stop()`用来终止 event loop。loop 会停止的最早时间点是在下次循环的时候，或者稍晚些的时候。这也就意味着在本次循环中已经准备被处理的事件，依然会被处理，`uv_stop`不会起到作用。当`uv_stop`被调用，在当前的循环中，loop 不会被 IO 操作阻塞。上面这些说得有点玄乎，还是让我们看下`uv_run()`的代码：

#### src/unix/core.c - uv_run

```cpp
int uv_run(uv_loop_t* loop, uv_run_mode mode) {
  int timeout;
  int r;
  int ran_pending;

  r = uv__loop_alive(loop);
  if (!r)
    uv__update_time(loop);

  while (r != 0 && loop->stop_flag == 0) {
    uv__update_time(loop);
    uv__run_timers(loop);
    ran_pending = uv__run_pending(loop);
    uv__run_idle(loop);
    uv__run_prepare(loop);

    timeout = 0;
    if ((mode == UV_RUN_ONCE && !ran_pending) || mode == UV_RUN_DEFAULT)
      timeout = uv_backend_timeout(loop);

    uv__io_poll(loop, timeout); 
```

`stop_flag`由`uv_stop`设置。现在所有的 libuv 回调函数都是在一次 loop 循环中被调用的，因此调用`uv_stop`并不能中止本次循环。首先，libuv 会更新定时器，然后运行接下来的定时器，空转和准备回调，调用任何准备好的 IO 回调函数。如果你在它们之间的任何一个时间里，调用`uv_stop()`，`stop_flag`会被设置为 1。这会导致`uv_backend_timeout()`返回 0，这也就是为什么 loop 不会阻塞在 I／O 上。从另外的角度来说，你在任何一个检查 handler 中调用`uv_stop`，此时 I/O 已经完成，所以也没有影响。

在已经得到结果，或是发生错误的时候，`uv_stop()`可以用来关闭一个 loop，而且不需要保证 handler 停止的顺序。

下面是一个简单的例子，它演示了 loop 的停止，以及当前的循环依旧在执行。

#### uvstop/main.c

```cpp
#include <stdio.h>
#include <uv.h>

int64_t counter = 0;

void idle_cb(uv_idle_t *handle) {
    printf("Idle callback\n");
    counter++;

    if (counter >= 5) {
        uv_stop(uv_default_loop());
        printf("uv_stop() called\n");
    }
}

void prep_cb(uv_prepare_t *handle) {
    printf("Prep callback\n");
}

int main() {
    uv_idle_t idler;
    uv_prepare_t prep;

    uv_idle_init(uv_default_loop(), &idler);
    uv_idle_start(&idler, idle_cb);

    uv_prepare_init(uv_default_loop(), &prep);
    uv_prepare_start(&prep, prep_cb);

    uv_run(uv_default_loop(), UV_RUN_DEFAULT);

    return 0;
} 
```