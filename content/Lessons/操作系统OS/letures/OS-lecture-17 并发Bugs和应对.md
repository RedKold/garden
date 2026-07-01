## 并发编程
- 任何物理世界：spawn, join
- 桌子和钥匙：lock, unlock
- 等待全局同步条件：wait, broadcast
- 桌子和多把钥匙：P/V (acquire/release)

**并发编程很难**
- 人类是 "sequential creature"
	- 出生就接触顺序执行的程序
	- 并发编程容易产生顺序执行的幻觉.
	- 理解所有行为是 NP-Hard 的
- Learn from mistakes


# Deadlock

## AA-Deadlock
同一个线程，pthread_mutex_lock 同一个锁锁两次，会怎么样？

「真实系统的复杂性」会使你很容易犯控制流上的错误
- 多层函数调用（递归）
- 隐藏的控制流
```c
void T_lock(void *arg) {
    mutex_lock(&lock);
    printf("Thread %ld acquired lock\n", (long)arg);

    // 同一个线程再次获取同一把锁 → 死锁
    mutex_lock(&lock);
    printf("Thread %ld acquired lock again (never reached)\n", (long)arg);

    mutex_unlock(&lock);
    mutex_unlock(&lock);
}

int main() {
    spawn(T_lock);
}
```
- 由于锁不可重入，第二次阻塞永远无法成功

##  ABBA-Deadlock
```c
void T_philosopher(){
	mutex_lock(&avail[lhs]);
	mutex_lock(&avail[rhs]);
	// ...
}
```
- 有一个交叉形..

## 死锁产生的必要条件
System deadlocks: 锁理解为桌子上有一把钥匙，有钥匙的人可以进入
1. Mutual-exclusion - 一把钥匙只能被一个人拿到，拿到钥匙才能继续
2. Wait-for - 拿到钥匙的人还想要更多的钥匙
3. No-preemption - 不能抢别人的钥匙
4. Circular-chain - 形成循环等待

**必要条件**的阐释：
- 打破以上任意一个条件，就不会等待了。

### 解析
**Mutual-exclusion**：NOT (钥匙只能被一个人拿到，拿到钥匙才能继续)
- 把“锁”的定义彻底就改掉了
- 代码设计的彻底重构 (e.g.: 引入 send/receive 向 T_waiter 发送信息)

**Wait-for**: NOT (拿到钥匙的人还想要更多的钥匙)
- 一把大锁保平安（性能低）
- Transaction memory (极难实现！)

**No-preemption**: NOT (不能抢别人的钥匙)
- 要能**回滚持有锁线程**执行过的操作（又是 transaction memory）
	- 把某一个线程踢出去，**就要回滚操作**
	
**Circular-chain**: NOT (形成循环等待)
- Lock ordering: （回忆 Philosophy 问题的规定先拿左再拿右）
	- 如果我给世界上的锁都编上号，规定获取锁的顺序..
- 好像很实用

### 在实际系统中避免死锁
**Lock ordering**
- 任意时刻系统中的锁都是有限的
- 给所有锁编号（Lock Ordering）
	- 严格按照从小到大顺序获得锁
	- **容易检查**
- **Proof**
	- 任意时刻总有一个线程获得编号最大的锁，它总是可以继续

**我们总会有一个线程**在继续运行着

## 软件工程的本质：Harness Engineering
**做最坏的假设，最防御性的编程**
- 例子：项目规定必须使用 RAII（C++ and rust 的核心反噬）
- 例子：项目规定必须明确 lock ordering
**做更坏的假设**：没有程序员是信得过的
- 每次 acquire/release 都用 printf 打一个日志
	- if A->B and B->A, error
	- 谨慎的对待 `return` (回收资源)
- `LD_PRELOAD=./locktrace.so ./a.out | python3 check.py`
	- lock 和 unlock 都做检查
	- 一边 trace 一边检查


- QEMU User trace memory...
	- 把程序当分析对象。(少数人的看家本领 -> AI  agent 直接搞定了)
	- 模拟解释执行任何一个程序
	- 未来有一天 AI 可以做好 Harness Engineering

> [!Note] jyy 对学生这样讲
> 你们从模型存在的第一天就开始用，你们会比别人站得更前，你们第一天享受到 AI 能力的红利，你们比别人更知道 AI 的边界在哪里，你能为它的下一次发布做好准备，你就能在这个时代找到自己的位置


## 'Killed by a machine'
The Therac-25
并发 bug
- `assert mode_in [Electron(Low), XRay(High)]`
- `assert beam_flattener in [On, Off]`
- `assert not (mode == XRay(High) and beam_flattener == Off)`

有反射镜控制照射剂量，但是并发 bug 导致无法控制。

 - **问题修复后**
- 还有 bug？！
	- If the operator sent a command at the exact moment hte counter overflowed, the machine would skip setting up some of the beam accessories

- 最终解决办法
	- 独立的硬件安全方案，监测到大计量照射直接停机（硬件保平安）


# 不上锁就不会死锁吗
## 数据竞争 (Data Race)
不同的线程同时访问同一内存，至少有一个是写
- T_alipay: `if(balance >= balance) {balance -= amount;}`
- T_sum: `sum++;`
- Peterson's protocol

**Race 的快慢导致截然不同的运行结果**

- **内存**可能是地理空间的任何内存
	- 可以是全部变量
	- 可以是堆区分配的变量
	- 可以是栈

### 数据竞争的例子

Case 1：上错了锁
```c
void T_1() { mutex_lock(&A); sum++; mutex_unlock(&A); } 
void T_2() { mutex_lock(&B); sum++; mutex_unlock(&B); }
```

Case 2：忘了上锁
```c
void T_1() { mutex_lock(&A); sum++; mutex_unlock(&A); }
void T_2() { sum++; }
```

- Data race 比 lock 复杂的多
	- 我们可以记录 lock/unlock 和所有 load/store
	- 检测"happens-before" race



# 都上锁就没有数据竞争了吗？
- 人类本质上是 sequential creature
	- **函数返回时天然同步的**。
	- 自带"All or nothing"的原子性 (atom)
- Therac-25: 如果模式切换瞬间完成，就没有问题了
	- 可惜并不是，机械装置需要时间。
	- 要是加个锁就不会杀人了吧
	- 我们形成了“立即生效”的肌肉记忆
		- 没有副作用 (pure functions)，就没有并发问题
	- 副作用有很好的用处，trade-off 了

**那么，程序员用的对不对呢？**
- **实证研究**
- 97%的非死锁并发 bug 都是原子性或顺序错误
	- 人类的确是 sequential creature

## 原子性违反 (Atomicity Violation)
**“ABA”： 代码被别人“强势插入”**
- 即使上锁也是 AV
- Therac-25 中“移动 Mirror + 设置状态”
- 比如，你可能有一个指针，被别的进程 free 掉了，违背了程序的原子性，即使上锁也不行。

---

**操作系统的状态也是共享状态**
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260508143230.png)

`/etc/password`：攻破
- 通过 symbolic link, 把邮件写进去
- 如果攻击者，通过 OS 的办法，删除/home/abc/mainbox, 创建符号连接，指向/etc/passwd 呢？
	- 攻击者虽然没有权限，但是这个邮件软件有权限

```
if (cond) {
	assert(cond);
	open(...); // 打开这个文件
}
```

## 顺序违反 (Order Vi)

**"BA"：事件未按预定的顺序发生**
- 例子：concurrent use-after-free

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260508144135.png)
- 如果 S4 发生在 S2 之前，则 S3 的的 while 不会停机

Back to Harness Engineering..
- 加强版的 LockDep
- 把整个程序执行对齐道时间轴上，为每个函数调用都标注“做了什么”
- Query AI 寻找疑似不该被打断/顺序错误的事件

Agentic AI 时代
- 让程序的 trace“可审计”是非常重要的
- **你做的**每件事情，都是可解释的，有一个动机的。
	- 程序是物理世界的事情在信息世界的投影
	- 程序更是可以解释的
	- 让每个函数在调用的时候都可以输出自己“做什么”
