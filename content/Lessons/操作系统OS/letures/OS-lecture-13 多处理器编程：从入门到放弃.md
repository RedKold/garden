# Review


# 并发编程入门

### 共享内存的并发编程：动机

- 动机 1: `syscall` 可能执行很长时间
	- 在执行系统调用时，程序也许可以“不闲着”
	- 也许我们可以同时执行若干个 handle_request
```c
void http_server(int fd){
	while(1){
		buf = alloc_buf();
		nread = read(fd, buf, 1024);
		handle_request(buf, nread);
	}
}
```


- 动机 2: 多处理器系统是共享内存的 (memory-shared), 不用白不用
	- 一旦分叉 (fork) 就形同陌路了，无法直接用内存通信

### 实现共享内存的“进程”？
- 加一个操作系统 API 就可以了
	- C 语言的状态机模型
		- 初始状态：`main(argc, argv, envp)`
		- 状态迁移：执行一条语句（指令）
	- 扩展这个模型就行

-  多线程程序的状态机模型
	- add a special system call: `spawn()`
		- 增加一个“状态机”(线程, thread)，有独立的栈，但共享全局变量
	- **状态迁移**：选择一个状态机执行一条语句（指令）
- 恭喜你已经了解了并发的全部

## 概念：并发和并行
  **并发**
- 逻辑上的 “同时执行”
    - 可以由操作系统/运行库模拟出的 “轮流执行”
    - （也可以是真正同时执行）

---

 **并行**

- 真正意义上的 “同时执行”
- 有 (共享内存的) 多个处理器
    - 同时执行指令 (load/store 访问共享内存)
 对应到概念模型
- SimpleC 选一个线程执行一条语句 (并发模型)
- SimpleC 的每个线程同时执行一条语句 (并行模型)


## 迷你线程库

### Spawn & Join

**简化的线程 API(thread. h)**
- 给操作系统 pthread (7) 线程库做了一些减法


### 证明“共享内存并发”

```c
#include <thread.h>

int x = 0, y = 0;

void inc_x() { while (1) { x++; sleep(1); } }
void inc_y() { while (1) { y++; sleep(2); } }

int main() {
    spawn(inc_x);
    spawn(inc_y);
    while (1) {
        printf("\\033[2J\\033[H");
        printf("x = %d, y = %d", x, y);
        fflush(stdout);
    }
}
```

- `thread.h` 是 jyy 做的简化的线程库

### More:
#### 多线程程序真的利用了多处理器吗？
- 并发确定了，是不是真并行？

#### 线程是否具有独立堆栈？
- if so, what's the range of stack?

#### gdb 单步调试多线程程序

`set scheduler-locking on`
- 把其他线程的调度锁住，可以观察其他线程行为


# 放弃（1）：状态迁移的确定性

### 什么是不确定性？

- Math (determinism)
	- $f:X\to Y$, same input, one output
- Program (determinism)
	- SimpleC; NEMU: 迁移都是函数
	- $s'=f(s)$, $f$ 是确定的
	- Everything is a state machine
		- 理解并发程序仍然是好用的视角
- **确定性**给我们“可控”的感觉
	- 同一件事总是 reproducible


### 不确定性(non-determinism)的魔鬼

程序“自身”是完全确定的
- 初始状态不确定
- 系统调用不确定
- 内存和计算指令确定

共享内存的并发打破了这一点
- 线程执行的*速度*没有保证
	- load 可能读到别的线程的 store，也可能不
- 理解起来好困难..



## 非确定性带来的并发控制难题

### 你怎么计算 1+1?

[A Game: Verify Sequential Consistency](https://jyywiki.cn/OS/2026/vsc.html)

## 失去确定性的后果

### 并发执行三个 T_sum，sum 的最小值是多少？

```c
void T_sum() {
    for (int i = 0; i < 3; i++) {
        int t = load(sum);  // 假设单行语句的执行是原子的
        t += 1;
        store(sum, t);
    }
}
```

> [!Answer] 答案
> `min_sum = 2`



### 这个问题本质上就很难：[Trace recovery is NP-Complete](https://epubs.siam.org/doi/10.1137/S0097539794279614); [vibe code 的游戏](https://jyywiki.cn/OS/2026/vsc.html)

- 是一个 NP 问题。

### 确定性丧失的后果

- 并发影响了计算机系统中的一切


# 放弃（2）: 顺序执行的幻想

- **性能**=**金钱**
	- Generation of *tokens*: 还真是

- Alipay
```c
#include "thread.h"

unsigned long balance = 100;

void Alipay_withdraw(int amount) {
    if (balance >= amount) {
        // Bugs may only manifest on specific timings. Sometimes
        // we reproduce bugs by inserting sleep()s.

        usleep(1);

        balance -= amount;
    }
}

void T_alipay() {
    Alipay_withdraw(100);
}

int main() {
    spawn(T_alipay);
    spawn(T_alipay);
    join();
    printf("balance = %lu\n", balance);
}
```

如果加入 `usleep(1)`, 就会产生很大的正数余额。其实是 `unsigned long` 溢出了。

这可能是线程强制等待了一下（在 `balance-=amount` 之前），B 线程没有获取 A 进程已经减过了，所以认为可以继续扣钱。



### 一个聪明的例子

```c
while(!flag);
```
- 等另一个线程举起 flag，我就可以继续。

- But, **编译器**更聪明，会优化掉...
	- 编译器认为永远不会有人改变 flag，直接优化成死循环

```c
void T_sum() {
    for (int i = 0; i < N; i++) sum++;
}

int main() {
    create(T_sum);
    create(T_sum);
    join();
    printf("sum = %ld\n", sum);
}
```
#### 如果添加编译优化？
- -O1: 100000000 (n) 😱😱
	- **优化了**：一直在寄存器做+1+1+1...，最后再 store 一次
	- 线程 A 读走 `sum=0`, 线程 B 也读走 `sum=0`, 各自谁后写会就覆盖谁
- -O2: 200000000 (2n) 😱😱😱
	- sum -o2 (编译器更加激进)
	- 直接执行 `sum+=N`, 然后写会
	- A 先做存之后，B 再读到 `sum=100000000` 再做一次，得到了看似正确但是危机四伏的答案

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260414125907.png)


- #TODO  read the asm code
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260414130303.png)
	- 这个是 `sum-o1` 的 asm code，其中 `%rdx` 是 sum 对应的寄存器（对的，其没有直接往内存操作，而是用寄存器）。然后最后把结果写回到寄存器
	- 还有一些别的有趣的优化：we don't need `i` anymore.

> [!Note] doubao-seed-code-2.0 said:
> Here's what GCC did with -O1:
> 
>   1. Register Promotion / Load/Store Hoisting
> 
>   - Original code: sum++ in a loop → N memory loads + N memory stores
>   - Optimized:
>     - Load sum once from memory into a register at the start (mov 0x2e51(%rip),%rdx)
>     - Perform all increments in registers
>     - Store sum once back to memory at the end (mov %rcx,0x2e33(%rip))
> 
>   2. Loop Transformation
> 
>   The compiler replaced the original memory-based increment loop with a register-based counting loop:
>   - rdx = initial_sum + N + 1 (loop bound)
>   - rax counts from initial_sum + 1 up to rdx
>   - rcx tracks the value before increment, ending at initial_sum + N


- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260414130731.png)
	- 这个是 `sum-o2` 的 asm code，很极端了，编译器直接把除了 call ret 的指令优化成了一条 `addq` 指令。
### 控制编译器优化的行为
- Way1: insert `volatile` code block
	- `while(!flag){ asm volatile ("" ::: "memory");}`
- Way2: mark load/store `volatile`
	 - `int volatile falg; while(!flag);`

> [!Note] 一句忠告
> Don't play with memory.

# 放弃（3）: 指令顺序执行

## 两层“编译”
- `.c` -> `.s`
	- 编译器“大杀四方”
	- 调换语句的顺序，删除死代码；......
- `.s` ->CPU 的那不装啊题
	- 处理器也是个*编译器*
	- 不只有一条 on-the-fly 的指令
	- 动态数据分析，“擅自调整”指令执行的顺序

## 观测“无序”带来的后果

```c
int x = 0, y = 0;

void T_1() {
  x = 1; // Store(x);
  int t = y; // Load(y);
  printf("%d", t);
}

void T_2() {
  y = 1;  // Store(y);
  int t = x; // Load(x)
  printf("%d", t);
}
```
