# 异步编程模型

## Review & Comments
计算图
- 理解并行的关键
并行算法
- 大规模数值计算问题
- Mandelbrot set 那样的"embarrassingly parallel"

并行数据结构
- 数据结构的访问局部性
- 尽可能把计算留在线程本地 (thread-local)

## 例子：实现 malloc/free

> Premature optimization is the root of all evil.
					——D.E Knuth

- **脱离 workload 做优化**就是耍流氓
	- 实际系统通常不考虑 adversarial worst case
	- 而是在真实分布下“表现好”
	- 在开始考虑性能之前，理解你需要考虑什么样的性能
- `malloc()` 的观察
	- 大对象分配后，读写数量应当远大于它的大小
		- 否则就是 performance bug
	- **推论**：越小的对象创建/分配越频繁
		- **小对象**：字符串、临时对象
		- **中对象**：容器、复杂的对象；更长的生存周期
		- **大对象**：巨大的容器、分配器；很长的生存周期
	- **实验**：几乎你只需要管好小对象就好了

- **核心**：让小对象的分配和回收快？
	- 尽量在本地完成
	- **分配**：Segregated Lists (Slab)
	- 每个 slab 中的对象都**一样大**
		- 每个线程本地都持有一些 slabs
		- Fast path-> 立即在线程本地分配完成
			- 类比 sloppy counter
		- Slow path -> `mmap()`
			- Fast path 不够用了，向系统要一个新的
	- **还有一些思路**：
		- 把每个 slab 切成小块



# 线程的代价

- **线程需要占用资源**
	- 首先，需要内存（线程栈、操作系统中的各种数据结构......）
		- `/proc/[pid]` 下的文件：操作系统内数据结构的投射
	- 还有一些分配的“资源”，例如 pid
		- 线程的切换需要进入 OS kernel 代码（开销）
- Linux 的一个线程占用多少资源？
	- 可以通过程序计算。
	- 找到可以计数的“资源”集合
	- 重复实验，统计资源的变化

**线程其实是开销很大的**

## 我们想要“随时随地” 的并行计算
- **启动线程的代价远大于函数调用**
	- Approach I： 轻量化线程
		- 让 spawn 和 join 的代价接近函数调用
		- Coroutine, goroutine,...
		- 未来可以在 CPU 上做 100w 个上下文来做吗？
	- Approach II：**异步模型**
		- 改变程序的执行模型，允许描述并行/并发的计算的计算图
		- Promise/Future, async/await

- C 语言本身是一个顺序的语言


# 方案一：线程的轻量化

- Python generator
```python
import random
import resource

THREADS = 1_000_000

def T_worker(name):
    i = 0
    while (i := i + 1):
        yield f"[{name}] i = {i}"

threads = [T_worker(i) for i in range(THREADS)]

count = 0
while count < 10_000_000:
    current = random.choice(threads)
    res = current.send(None)
    print(res)

    count += 1
```

- 我们有多个栈帧，不过不是 OS 分配的而是程序模拟的，类似于 coroutine
	- `yield` 可以切换
- 你也可以在 C 和 CPP 实现这一点 
	- `setjmp, longjump` 来换栈帧
```c
static const char *coro_yield(struct coroutine *c) {
    c->state = YIELD;
    if (setjmp(c->coro_ctx) == 0) {
        longjmp(c->ctx, 1);
    }
    return c->buf; /* resumed here */
}
```

> [!Note] 为什么 setjmp 和 longjmp 可以做到协程?
> setjmp 和 longjmp 是 C 标准库中的函数，用于非局部跳转（non-local goto）。让我解释它们的作用和协程实现原理：
> setjmp 和 longjmp 的基本概念：
> 1. setjmp(jmp_buf env)：
>     - 保存当前的执行上下文（程序计数器、栈指针、寄存器状态等）到 jmp_buf 中
>     - 第一次调用时返回 0
>     - 当通过 longjmp 跳转回来时，返回 longjmp 传入的值
>  2. longjmp(jmp_buf env, int val)：
>     - 恢复之前 setjmp 保存的上下文
>     - 程序从 setjmp 调用处继续执行，但 setjmp 会返回 val
>   协程实现原理（看代码）：
>   这个协程实现的核心在于：
>  3. **两个 jmp_buf：ctx（调用者上下文）和 coro_ctx（协程上下文）**
>  4. 手动切换栈：coro_create 中通过汇编修改 rsp/sp 寄存器，让协程在自己分配的栈上运行
>  5. yield/resume 机制：
> 	- coro_yield：用 setjmp 保存协程上下文，然后 longjmp 回到调用者
> 	- coro_resume：用 setjmp 保存调用者上下文，然后 longjmp 回到协程
>   执行流程：
> ``` 
>   main → coro_create → setjmp(ctx) → 切换栈 → coro_entry → coro_yield
>                                                             ↓
>   main ← longjmp(ctx, 1) ← setjmp(coro_ctx) ←────────────┘
>     ↓
>   coro_resume → setjmp(ctx) → longjmp(coro_ctx, 1) → 继续执行
> ``` 
>   这种方式实现的协程是协作式多任务，需要显式调用 yield 让出控制权。

## 协程：缺陷和解决办法
一个协程等待，1,000,000 个都等待
- 操作系统中只有**一个执行流**
	- 进程的话可以并行
	- 一旦协程 R1 使用 `read()` 进入操作系统内核，其他协程只能等待，而无法计算
		- 这一整个程序还是一个用户栈，模拟的协程栈无法再进入内核的同时并行工作
- `mutex_lock` + `yield` = AA 型死锁
	- **在协程中使用互斥锁**，但由于只有一个线程，则会死锁。（是操作系统为线程设计的）

**解决办法**：异步 I/O
- `man 2 open`:  `O_NONBLOCK` “非阻塞”
	- 系统调用不与 I/O 操作完成同步
	- read 可能返回 nread/EAGAIN -> 可以 yield
- 基于文件描述符和 epoll 的同步
	- epoll 是 Linux 上高效的 I/O 多路复用机制，监控大量文件描述符
	- `OS` 提供了 `eventfd`，`timerfd`，`io_uring `
	- main loop *`epoll`*, create coroutine, *` eventfd `* synchronize
- 不会考


## A Programming-Language Trick

- 线程+blocking I/O
```c
sleep(1);				// wait_until(T >= cur + 1s);
read(fd, buf, size);	// wait_until(fd has data);
```

- hack
```c
put_my_self_into_sleep(1); yield();
while (read_async(fd, buf, size) == -EAGAIN){
	yield();
}
```

- **我们在一个线程(Process)上**，有很多个进程 (Thread)
	- 共享内存的
- 协程(Coroutine)组成了一个队列 
	- 用 `put_my_self_into_sleep` 维护这个队列
	- 用 `yield` 调度
- 这样类似于在一个进程上做了一个小的**操作系统**
- 每个 CPU 上有一个 Go worker Thread

## Go 语言中的同步和通信

> Do not communicate by sharing memory; instead, share memory by communicating.
> ——Effective Go

共享内存=万恶之源
- 信号量/条件变量：实现了同步，但没有实现“通信”
	- 数据传递靠手工
- UNIX 时代就有很棒的并发编程机制了
	- **计算图**、生产者/消费者同步
	- 管道 (Channels in Go) 实现线程间的同步+通信
		- 管道天生是一个计算图
- Golang: 继承 UNIX Philosophy


# 方案二：描述计算图

## 另一条平行的世界线

1995， Brendan Eich，加入 Netscape
- 设计网页脚本语言
	- 函数式变成
- 糅合 C，Java， Scheme， Self


## From Web 1.0 to Web 2.0
Asynchronous JavaScript and XML
- no JSON
- webpage now can "flush at backstage"
	- 随时请求后端
	- 任何“应用”可以做的，网页都可以做了

jQuery $(2006): A DOM Query Language
```javascript
$(document).ready(function()){ /*code*/}
$("#myElement").text("新内容").css("color", "red")
```
非常优雅
- 现代浏览器：$ 就是 `document.querySelector`

## 从此，做“任何事”都只要浏览器
**一个梦想**：ChromeOS
- HTML  + CSS 构建应用的方便程度超过 GUI 编程
- GTK, Qt, MFC...
	- 微信小程序继承遗志

## JavaScript 并发编程
- 零门槛的编程语言，让大家并发编程
	- Data race, atomicity violation,...
- 更简单的**描述计算图的模型** is strongly needed
- Event-based concurrency in JavaScript
	- 事件的并发模型
	- 没有任何 Blocking I/O
	- **每个事件都是原子的**。
		- 按顺序执行所有 event handlers; run to completion
		- 网络请求，sleep 都在队列中插入一个新的事件（计算图中的节点）
- 

## 事件编程、计算图和回调
## 异步回调 (callback)

- 在事件完成后排队调用 (success, error)
- 创建了计算图中的**动态节点**
```javascript
$.ajax({
    url: '/api/user',
    success: function(user) {
        $.ajax({
            url: `/api/user/${user.id}/friends`,
            success: function(friends) {
                $.ajax({url: `/api/friend/${friends[0].id}`, ...});
            },
            error: function(err) { ... }
        });
    }, ...
});
```
- callback **地狱**

## 回归"描述计算图"

`Promise("promise", "contract")`, generate calculate-graph node "future to be complete"

- Promise. then 还是没有解决顺序逻辑断裂的问题

## Async/Await 语法糖
- 还能再简单一点吗？
- 编译器/编程语言设计在我，有什么办不到的呢？
    - **同步的写法表达异步的流程**，剩下的让编译器去头疼吧
```javascript
async function fetchData(token) {
  const response = await fetch(
    `/api/submissions/?token={token}`
  )
  return response.json()
}
await Promise.all([fetchData('1234'), fetchData('5678')])
```

- async function f () → function f () { return new Promise (…) }
- await f () → return Promise.resolve (f ()). then (…)
- **一个语法糖**

 Promise
-  简单来说，Promise 是一个"承诺"，表示将来某个时间点会返回一个结果（可能是成功的结果，也可能是失败的原因）。
Promise 有三种状态：
- pending：初始状态，既不是成功，也不是失败状态
- fulfilled：意味着操作成功完成
- rejected：意味着操作失败
  - Purpose: Async operation container, represents a value that may be available now/later/never
  - States: pending → fulfilled (resolved) OR rejected
  - Methods: .then () for success, .catch () for errors, .finally () for cleanup
  - Composition: Promise.all (), Promise.race (), Promise.allSettled (), Promise.any ()
  async/await
  - Purpose: Syntactic sugar over Promises, makes async code look synchronous
  - async function: Always returns a Promise
  - await keyword: Pauses execution until Promise settles, only inside async functions
  - Error handling: Uses try/catch instead of .catch () chains
  - Flow: Linear, easier to read and debug than nested .then () chains
  Both enable non-blocking async code while maintaining readability and composability.

## 从前端到全栈

### 操作系统上的另一个应用生态
- Angular, React, Vue, Bootstrap, Tailwindcss
- Express, Next, Nest
- Electron and vscode; link and claude code; React Native
- asm. js
- Mermaid, TensorFlow, Three...

> 如果对 javascript 的编写感到疑惑，不妨重新看看并发的路



