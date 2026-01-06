## 多道程序

### 上下文切换

Basic **Idea**:
- Context switching is the process stack switching
How to find *Context* of other processes?
- Use a `cp`  pointer (Context pointer) to record the *Context* structure's position.
操作系统使用的是 `PCB` 结构 (process, control block)。每一个进程维护一个 PCB

我们首先来做内核线程。

创建内核线程上下文的函数是 `kcontext()`，在 `abstract-machine/am/src/$ISA/nemu/cte.c` 中定义

```c
Context *kcontext(Area kstack, void (*entry)(void *), void *arg) {
  return NULL;
}
```

- `kstack` 是栈的范围
- `entry` 是内核线程的入口
- `arg` 是内核线程的参数

在 `kstack` 的底部创建一个以 `entry` 为入口的上下文结构 `Context c`。
- `return value`: 返回 `&c`，该结构的指针。


