# Review & Comments
- thread. h: `spawn (fn), join ()`
- 1+1 不会求了，1，2，3 也不会数了

**Why we teach Concurrency in OS lesson?**
- UNIX's fork-execve model: processes don't *share memory*
	- But system call *shared*
		- syscall 指令会跳转到操作系统代码执行
		- 操作系统是同一个 C 程序（共享内存）
- 每个进程的系统调用就成了多个 *线程*
	- P1: `read(fd, buf, size);`
	- P2: `write(fd, buf, size);`
- OS is the first *serious?* and large concurrency program

# 互斥：概念与 API
## 遗留问题：1+1
- Hope introduce a API to make sure *sum*'s answer is always *correct*(no matter how to execute)
- **互斥**：阻止一些代码的并发/并行执行
	- 就可以解决

```c
logn sum = 0;

void T_sum(){
	make_it_work{
		sum++;
	}
}
```

- **发明了**: "Transactional Memory"

- How to implement make it work?

- **单处理器模型**：关闭中断

```c
disable_interrupt();
sum++;
enable_interrupt();
```

## 互斥：API
**所有被 lock 标记的代码块**都"mutually exclusive"
- mutex lock

```c
lock();
// 任意代码, e.g. sum++
unlock();
```

- 有点像锁房间
	- lock: 锁住别人就进不来
	- unlock: 解锁别人才进来
- 或者说有钥匙
	- lock (acquire)：有钥匙就继续，没有钥匙需要等待。
	- unlock (release)
- 关于线程的锁，有点想到被计网的 server 和 client 线程支配的痛苦了
	- [[Lessons/计算机网络/exps/exp2/README|计算机网络lab2 note]]


**可以创建多个桌子（多把钥匙）**
- 我们只需要保护同一块内存区域不被冲突就可以了

```c
mutex_t lock_a = MUTEX_INIT();
mutex_t lock_b = MUTEX_INIT();

mutex_lock(&lock_a);
x++;
mutex_unlock(&lock_a);

mutex_lock(&lock_b);
y++;
mutex_unlock(&lock_b);
```

Finally can write 1+1 right.
- Example
- 
```c
#include <thread.h>
#include <thread-sync.h>

#define N 10000000
int sum;

mutex_t lk = MUTEX_INIT();

void T_sum(int tid) {
    for (int i = 0; i < N / 10; i++) {
        mutex_lock(&lk);
        for (int i = 0; i < 10; i++) {
            sum++;
            asm volatile("" : : : "memory");
        }
        mutex_unlock(&lk);
    }
}

int main() {
    spawn(T_sum);
    spawn(T_sum);
    join();

    printf("sum  = %ld\n", sum);
    printf("2*n = %ld\n", 2 * N);
}
```

**使用互斥锁**；


## 互斥锁的使用方法

假设正常使用锁（其他进程的读写和 mutex 哪的读写互不干扰）
- 程序行为和 lock (); unlock (); 完全等价
- `stop_the_world()`;

## 你高兴的太早了
- 人类的神经网络训练，天生不太擅长处理多个头绪
- 大家做好自己就好了
- 可是线程会互相影响，脑子就炸了

- 非常痛苦的 **API**
	- lock 和 unlock 都是*程序员*负责的
	- 很复杂.

- **两把锁是可以并发的**。
	- 锁错了！
- 没有人能确信自己记得什么锁锁了什么

- **从一把大锁保平安开始**
	- 充足的压力测试
	- 更细粒度的锁会提高性能，但也容易出错
	- Easy Test v.s. Hard Test, who will win?
- *Premature optimization is the root of all evil*

## FAQ: 都不并发了，我们还要线程吗？

**悲观的 Amdahl's Law**
- 如果你有 $\frac{1}{k}$ 的代码是不能并行的，那么
$$
T_\infty=\frac{T_{1}}{k}
$$
- 哪怕一万个 CPU，加速效果都一般，因为有 $\frac{1}{k}$ 不可加速。
- 下标表示并行数量

**乐观的 Gustafson's Law**的更细致版本
- 能并行的并行计算总是能实现的
$$
T_{p}<T_{\infty}+\frac{T_{1}}{p}
$$

**物理世界具有局部性**-局部性原理
- 物理对相邻物体的影响需要时间
	- 很好的近似
- **推论**：任何物理世界模拟皆可以大规模进行

- **独立完成**。
	- e.g. 图书馆借一本书，和同学同时只有一个人能够到书架上的书（**关键数据结构**），是不可并行的。但是借到手里看，就是一个 private memory，即使还在图书馆的地址空间，但已经是可以并行的了。
	- 一个关于并行的乐观启示

# 在操作系统上实现互斥
## Dekker's Algorithm
A process P can enter the critical section if the other does not want to enter, otherwise it may enter only if it is its turn.


## Peterson's Algorithm

A process P can enter the **critical section** if the other *does not want to enter*, or it has indicated its desire to enter and has given the other process the turn.

### **厕所的故事**

> 三个变量：你的手、他的手、厕所门。

#### 如果希望进入厕所，按顺序执行以下操作：

1. 举起自己的旗子 (store 手)
2. 把写有对方名字的字条贴在厕所门上 (store 门)
	1. **贴标签**

#### 然后进入持续的观察模式：

1. 观察对方是否举旗 (load 手)
2. 观察厕所门上的名字 (load 门)
3. **对方不举旗或名字是自己，进入厕所，否则继续观察**

#### 出厕所后，放下自己的旗子 (不用管门上的字条)
#### 进入厕所的情况

- 如果只有一个人举旗，他就可以直接进入
- 如果两个人同时举旗，由厕所门上的标签决定谁进
    - 看似 “谦让”，实则手快有 (被另一个人的标签覆盖)、手慢无

#### 一些更细节情况（ 直觉）

- A 看到 B 没有举旗
    - B 一定不在厕所
    - B 可能想进但还没来得及把 “A 正在使用” 贴在门上
- A 看到 B 举旗子
    - A 一定已经把旗子举起来了 (_!@^ #_ &!%^(&^!@%#
### Code's View

```c
void T_A() {
    while (1) {
        a = 1;                    BARRIER;
        turn = B;                 BARRIER; // <- this is critcal for x86
        while (1) {
            if (!b) break;        BARRIER;
            if (turn != B) break; BARRIER;
        }

        // T_B can't execute critical_section now.
        critical_section();

        a = 0;                    BARRIER;
    }
}

void T_B() {
    while (1) {
        b = 1;                    BARRIER;
        turn = A;                 BARRIER;
        while (1) {
            if (!a) break;        BARRIER;
            if (turn != A) break; BARRIER;
        }

        // T_A can't execute critical_section now.
        critical_section();

        b = 0;                    BARRIER;
    }
}

```

### 如何理解 Peterson 算法？
- Prove by brute force
	- Peterson 算法状态空间是*有限的*
		- `(x, y, turn, PC1, PC2)`
	- 计算机可以做

	- 可以模拟一下。

### Peterson 算法的路线错误
- 试图在 **load/store**上互斥
	- 计算机系统是我们造的
	- 我们当然可以改造它成容易实现互斥的样子
	- 这是 Computer Science 和自然科学的很大不同
		- 相当多的游戏规则是我们定的
- 软件不好解决的，硬件帮忙
## 实现线程互斥：分析
- 不正确的：
	- Peterson 假设 load 和 store 都可以瞬间完成
	- load 时候手一定背在后面...

- **不正确的例子**；
```c
void lock(){
retry:
	if(can_go == ✅){
		can_go = ❌; // return
	}
	else{
		goto retry;
	}
}

void unlock(){
	can_go = ✅;
}
```

我看到 can_go是✅的，但是当我想改的时候，`can_go` 已经不在我线程控制了（可能被别人改）。

- 只需要一小段时间的 `stop-the-world` 就好了
- 总线锁一下，不让别人访问。(x86: Bus Lock (locked instruction))

## 甚至我们又可以实现 1+1 了

x86 asm:
```asm
lock addq $1, (sum)
```
就可以了！
- 硬件真伟大啊...
- 但也带来了麻烦（都是 trade-off）


## 自旋锁

不停的 exchange

```c
void spin_lock(spinlock_t *lk) {
    while (1) {
        int status = __atomic_exchange_n(lk, LOCKED, __ATOMIC_ACQ_REL);
        if (status == FREE) {
            break;
        }
    }
}
void spin_unlock(spinlock_t *lk) {
    __atomic_exchange_n(lk, FREE, __ATOMIC_ACQ_REL);
}

```
 2. How it works & Why it works

  Key mechanism:

  3. Initialization: The lock starts in FREE state (value = 0).
  4. Lock acquisition:
    - __atomic_exchange_n is a compiler built-in atomic operation that:
        - Atomically sets the lock value to LOCKED (1)
      - Returns the previous value of the lock
    - If the previous value was FREE, this thread has successfully acquired the lock → exit loop
    - If the previous value was LOCKED, another thread holds the lock → keep looping (spinning) waiting for it to be released
  5. Lock release: Simply atomically set the lock back to FREE,
  allowing a waiting thread (in the while loop) to acquire it.

  Why it guarantees mutual exclusion:

  - The __atomic_exchange_n operation is atomic — it executes as a
  single, indivisible step from the perspective of other threads.
  - Only one thread can successfully exchange FREE → LOCKED and get
  FREE back as the old value. All other threads will see LOCKED and
  keep spinning.
  - The __ATOMIC_ACQ_REL memory ordering provides acquire-release
  semantics:
    - Acquire on lock: All subsequent memory accesses by this thread
  cannot be reordered before the lock acquisition
    - Release on unlock: All prior memory accesses by this thread
  cannot be reordered after the lock release
    - This ensures proper memory visibility between threads.


 ### 自旋锁-严重的性能问题
 - 除了获得锁的线程，其他处理器上的线程都在空转
	 - 都在死循环
	 - 如果代码执行很久不如把处理器给其他线程。

- **应用程序不能关中断**
	- 持有自旋锁的线程被切换
- 如果 Application 可以告诉 OS 就好了

## 操作系统中的锁
**把**锁的实现放在 OS 中实现

- `syscall(SYSCALL_acquier, &lk);`
- `syscall(SYSCALL_release, &lk)`
- **复杂的工作交给内核**
	- 关中断+自旋

- **系统调用**实现 
- `man 7 futex | claude explain`


---

做实验时候，一定要记得严谨～
做实验之前，阅读一下

[Systems Benchmarking Crimes](https://gernot-heiser.org/benchmarking-crimes.html)

