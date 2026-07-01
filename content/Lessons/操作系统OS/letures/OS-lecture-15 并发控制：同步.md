## 代码表示同步的直觉

### 同步电路
```c
while(!posedge(clk));		// await posedge(clk)
ff_out = ff_in;
```

### 线程 Join

```c
spawn(T_1); spawn(T_2); ... spawn(T_n);
join
```


## 同步-Thought

这些案例的共性：实现“握手”
- 等到某一个条件发生，然后继续分叉执行
- 如果现在条件不发生，一定有一个操作使它发生

**我收的瞬间就有了一个“全局同步的状态”**-同步点 (synchronized point)


## 同步的乐团

> Everything is Code, so AI may generate everything.

In the model of **orchestra**, one possible *synchronize* way is to **wait_for_beat**

```c
void release_beat(){
	mutex_lock;
	// do something
	mutex_unlock;
}
```


## 发明条件变量
如果不希望自旋？
```c
do{
	mutex_lock(&lk);
	cond = check_condition();
	mutex_unlock(&lk);
} while(cond)
```

- **条件满足**的时候，我们唤醒
- 不满足的时候，我们睡觉

- **条件变量要带一把锁**
	- `cond_wait(&cv, &lk)` 
		- **行为**：释放锁，然后立即等待（两件事瞬间同时完成）
		- `cond_wait` 等待的线程，通过 `signal(&cv)` 或 `broadcast(&cv)` 唤醒
	- `cv`: condition variable. `lk`: lock


### **条件变量**模版 #必考 
```c
mutex_lock(&lk);
while(!check_condition()){
	cond_wait(&cv, &lk);
}
mutex_unlock(&lk);
```

- **我在持有锁的时候**，同步条件是不满足的。
- 在一个操作下，完成“我要睡眠”和“释放锁”的事情。


**万能同步的实现方法**

在要修改的共享状态前后都上好锁

```c
void Tlcs(int i, int j){
 if(i==0 || j==0) return;
 // TODO
 dp[i][j] = max3(dp[i-1][j],dp[i][j-1],dp[i-1][j-1]+(a[i]==b[j]))
 // TODO
}
int main(){
 for(int i=1;i<=n;++i){
 for(int j=1;j<=m;++j){
 spawn(Tlcs,i,j);
 }
 }
 join();
 printf("%d\n",dp[n][m]);
 return 0;
}
```

这是一个 max-common-sequence 问题，请你把它并行化

- 我们可以看到依赖关系：左边，上边，和左上需要计算完毕。


```c
void Tlcs(int i, int j) {
    if (i == 0 || j == 0) return;

    // Wait until all dependencies (i-1,j), (i,j-1), (i-1,j-1) are ready
	// using our modle:
	/*
		mutex_lock(&lk);
		while(!check_condition()){
			cond_wait(&cv, &lk);
		}
		mutex_unlock(&lk);
	*/

    mutex_lock(&lk);
    while (!(ready[i - 1][j] && ready[i][j - 1] && ready[i - 1][j - 1])) {
        cond_wait(&cv, &lk);
    }
    mutex_unlock(&lk);

    // Compute LCS value for this cell
    // Dependencies are guaranteed to be computed (ready flags were set)
    dp[i][j] = max3(
        dp[i - 1][j],
        dp[i][j - 1],
        dp[i - 1][j - 1] + (a[i] == b[j])
    );

    // Mark this cell as ready and signal waiting threads
    mutex_lock(&lk);
    ready[i][j] = 1;
	// Broadcast signal to waiting threads
    cond_broadcast(&cv);
    mutex_unlock(&lk);
}

```


## 经典同步问题：生产者-消费者问题
- 99%的实际并发问题都可以用生产者-消费者解决
	- Master-slave (scheduler-worker) 模式
- Producer and Consumer share **one** buffer
	- Producer: if buffer has empty place, *place* in; else *wait*
	- Consumer: if buffer has data, *take* out; else *wait*
- **Synchronization**: The same object's production must **happens-before** consumption


### 简化描述
```c
void T_producer(){ printf("("); }
void T_consumer(){ printf(")"); }
```
- Produce: print left bracket (push into buffer)
- Consume: print right bracket (pop from buffer)

在 `printf` 前后增加代码，使得打印的括号序列满足
- 不能输出错误的括号序列
- 括号嵌套的深度不超过 $n$（buffer size）


### Caveat：小心并发

- Producer 如果唤醒了等待的 producer 就糟了
```c
#define CAN_PRODUCE(depth < n)
#define CAN_CONSUME(depth > 0)

void T_producer(){
	while(1){
		mutex_lock(&lk);
		
		if(!CAN_PRODUCE){
			cond_wait(&cv, &lk);
			
		}
		
		printf("(");
		depth++;
		
		cond_signal(&cv);
		mutex_unlock(&lk);
	}
}
```

- 这里把 broadcast 改成了 `cond_signal()`,
	- 在只有一对 Producer & Consumer 情况下其实是对的
	- 但是你可能会发生 producer 发给 producer 的情况


### 条件变量：万能的同步方法
我们希望在多线程下实现**鱼**
- 打印 `<><_` 和 `><>_` 的组合
**有三种线程**
- `T_a` 若干：死循环打印 `<`
- `T_b` 若干：死循环打印 `>`
- `T_c` 若干：死循环打印 `_`


- 我们规定了每个状态下，我们能打印的东西。
- **条件**：
	- 当前状态的下一个字符是我这个线程打印的字符，并且没有别人在打印
- **条件变量回答的问题**：
	- 我什么时候被唤醒？



## 计算图模型
`G(V, E)`：有向无环的 Dependency Graph
- Edge: 计算的 Dependency
- 几乎总是可以以这个视角去理解并行计算


### 无处不在的计算图
- 神经网络 (`Pytorch autograd`; ` fxgraph`)
- `Makefile` (GNU Make 是可以自动并行化的)
	- 不同的 `a.out` 也有依赖，我们也需要一个计算图


## 同步：实现任意计算图
- **思路**：为每个计算节点设置一个线程和条件变量

```c
void T_u(){
	... // u's calculation
	mutex_lock(v->lock);
	v->n_done++;
	cond_signal(v->cv);		// can signal
	mutex_unlock(v->lock);
}

void T_v(){
	mutex_lock(v->lock);
	while(!(v->n_done == v->n_predecessors)){
	
	}
}
```

---
- **思路**：实现一个任务的调度器
	- 一个生产者（scheduler），许多消费者 (workers) 循环：
		- 也叫做 `Executor Pool`
```c
mutex_lock(&lk);
```

 为每一个 Task Spawn 一个线程。

## 同步的关键：理解同步条件
- 方法 1：为每个计算节点分配线程
- 方法 2:

