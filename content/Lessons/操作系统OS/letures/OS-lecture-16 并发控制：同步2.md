# 信号量
**计算图和一个奇妙的想法**
- 任意一个有向无环图
	- `u -> v` 代表 `u` 的计算 happen-before `v` 的计算
		- v: `foreach (u->v) while(!u_done)`
		- `v`: `while(pred_done != n_pred)`;


**奇妙的想法：用互斥锁实现同步**
- Dependency e: `u->v` 对应一把锁
	- T_main: `lock(e); spawn_threads();`
	- T_u: `work(u); unlock(e);`
	- T_v: `lock(e); work(v)`;
- Release-Acquire 天然实现了 happens-before
	- 代码实现
	- **为每一条边**分配一把互斥锁
	- 通过在 main 函数 acquire(lock of u -> v)，u release -> v acquire 来实现 happens-before。注意虽然这个程序可以正常运行，但这是 pthread mutex 不允许的 undefined behavior。

**Release-Acquire**天然实现了 happens-before 同步
- Acquire = 等待信号（拿走桌上的🔑）
	- `while(!(k == 1)); k=0;`
- Release = 发出信号（在桌上放🔑）
	- `k=1;`
- 信号/🔑可以理解为现实生活中的“进入凭证”

**可以允许桌上有多个钥匙存在**
- Acquire: 
	- `while (!(c > 0)); c--;`
- Release:
	- `c++;`

## 发明信号量 (Semaphore)

带一个计数器 `count`（初始化时指定）的互斥锁

- Acquire (Prolaag - try + decrease/down/wait)
    - `while (!(count > 0)) ; count--;`
    - “拿走一把 🔑”、“吃掉一个车位”
- Release (Verhoog - increase/up/post/signal):
    - `count++;`
    - “放回一把 🔑”、“变出一个车位”
- Caveat: 信号量只管出入口，停车位还需要其他机制管理

![](https://jyywiki.cn/OS/2026/static/img/parking.jpg)


## 信号量 API
```c
void P(sem_t *sem) {  // Acquire
    mutex_lock(&sem->lk);
    while (!(sem->count > 0)) {
        cond_wait(&sem->cv, &sem->lk);
    }
    sem->count--;  // 消耗一个 token (信号)
    mutex_unlock(&sem->lk);
}

void V(sem_t *sem) {  // Release
    mutex_lock(&sem->lk);
    sem->count++;  // 创建一个 token (信号)
    cond_broadcast(&sem->cv);
    mutex_unlock(&sem->lk);
}
```

- 初始时的 `count` 相当于执行了 `count` 次 release
    - 桌上 🔑 的数量、停车场车位的数量……

---
# 信号量实现同步

## 用信号量实现互斥锁

```c
sem_t sem = SEM_INIT(1);

void lock(){
	P(&sem);	// acquire: get🔑
}

void unlock(){
	V(&sem);	// release: put🔑
}
```

- 互斥锁是信号量的特例
- 信号量是互斥锁的扩展

## 典型应用-信号量
- 实现一次临时的 happens-before: A->B
	- `s=0`
	- A -> release: V (s) -> acquire: P (s) ->B
		- 最开始的“用互斥锁实现同步”


## 实现计算图的两种方式

- 考虑线程 join
	- 为**每条边计数**
		- worker_t: `V(done_t)`
		- main: `P(done_1); P(done_2); ... P(done_n)`
	- 为 **每个节点计数**
		- worker_t: `V(done)` 
		- main: `P(done); P(done); ... P(done)`;
**推广到任意计算图**
- 为每条边 e : u->v 计数
	- T_u: `work(u); V(e);` 做完任务放回🔑
	- T_v: `P(e); work(v);` 需要🔑才能开始
- 为每个节点 count


# 信号量、条件变量与同步

“或”这个条件是单个信号量无法表达的。

## 哲学家吃饭问题
- 哲学家 (线程) 有时思考，有时吃饭
- 吃饭需要同时得到左手和右手的叉子![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260423150709.png)

通过一个额外的信号量，我们限制上桌吃饭的人数不超过 4 人。上桌的 4 人之中至少有一人可以获得左右手的叉子，然后释放后退出临界区。这个协议的正确性并不是显然的：我们必须非常小心地对待任何并发问题

- **并发编程**的两种办法
	- 互斥锁
	- 信号量

- **条件变量**
	- 同步条件： `avail[lhs] && avail[rhs]`
	- 背模版 #必考 
		- [[OS-lecture-15 并发控制：同步#条件变量模版|条件变量模版]]

**解决它的有趣函数**-信号量版本

```c
void T_philosopher(int id) {
    int lhs = (id + N - 1) % N;
    int rhs = id % N;

    // if (lhs > rhs) {
    //     int tmp = lhs;
    //     lhs = rhs;
    //     rhs = tmp;
    // }

    while (1) {
        // Come to table
        // P(&table);

        P(&avail[lhs]);
        printf("+ %d by T%d\n", lhs, id);
        P(&avail[rhs]);
        printf("+ %d by T%d\n", rhs, id);

        // Eat.
        // Philosophers are allowed to eat in parallel.

        printf("- %d by T%d\n", lhs, id);
        printf("- %d by T%d\n", rhs, id);
        V(&avail[lhs]);
        V(&avail[rhs]);

        // Leave table
        // V(&table);
    }
}
```

### 成功的尝试：信号量
**如果 5 个哲学家同时举起左手的叉子...**
- **死锁**，需要禁止

Workaround 1: 从桌子上赶走一个人
- 直观理解：大家先从桌上退出
	- 想吃饭，先得拿到桌上的🔑
	- 吃完饭，把🔑还回去
	- 实现最多只有 4 个人可以上桌，就不会循环等待了

Workaround 2：Lock Ordering


## 用信号量实现条件变量
Implementing condition variables out of a simple primitive like semaphores is surprisingly *tricky*.

```c
void wait(cond_t *cv, mutex_t *mutex) {
    cv->nwait++;
    mutex_unlock(mutex);
    P(&cv->sleep);
    mutex_lock(mutex);
}

void broadcast(cond_t *cv) {
    mutex_lock(&cv->lock);
    for (int i = 0; i < cv->nwait; i++)
        V(&cv->sleep);
    cv->nwait = 0;
    mutex_unlock(&cv->lock);
}
```



### 引入一个“调度线程”
- 引入一个发叉子的服务员（Waiter）
	- 根据 Global State: 决定谁可以继续
	- **最好的方式**：把条件一起送给 T_waiter，由 T_waiter 决定谁可以继续
	- **好处**：甚至我们可以实现不同的调度策略，实现公平性
- **把任何问题**，改造成 Producer-Consumer 模型



## Exam: 石头剪刀布

```c
void play_game(int tid, int type) {
    assert(type == ROCK || type == SCISSORS || type == PAPER);
    while (1) {
        int result = play(tid, type);
        if (result == TIE) {
            printf("%d TIE\n", tid);
        } else if (result == WIN) {
            printf("%d WIN\n", tid);
        } else {
            printf("%d LOSE\n", tid);
        }
    }
}
```

用并发编程 API 实现 
```c
int play(int tid, int type);
```

函数。

使用 conditional variable
then, play should as least know whether know the `count` of `type`

one possible calculation *order* is left to right.

```c
int res[n];
int type[n];

int play(int tid, int type){
	// 如果左边的计算完了
	mutex_lock(&lk);
	while(!done[tid-1]){
		cond_wait(&cv);
		/* TODO */
		if(isNew(type)){
			RSP[rsp_idx++] = type;
		}
	}
	mutex_unlock(&lk)
}
```


```c
int play(int tid, int type) {
    mutex_lock(&lock);

    int my_round = round_;

    /* Record this thread's type */
    types[arrived] = type;
    arrived_ids[arrived] = tid;
    arrived++;

    if (arrived < n) {
        /* Not everyone is here yet — wait */
        while (round_ == my_round) {
            cond_wait(&cv, &lock);
        }
        int result = results[tid];
        mutex_unlock(&lock);
        return result;
    } else {
		// one  thread to summarize
        /* ── All n threads have arrived: resolve the round ── */

        /* Count each gesture */
        int cnt[3] = {0, 0, 0};
        for (int i = 0; i < n; i++) {
            cnt[types[i]]++;
        }

		// 计算有多少种
        int present = 0;
        for (int t = 0; t < 3; t++) {
            if (cnt[t] > 0) {
                present++;
            }
        }

        /* Determine winner/loser types */
        int won_type = -1;
        if (present == 2) {
            /* Exactly two gestures present: one beats the other */
            for (int t = 0; t < 3; t++) {
                if (cnt[t] > 0 && cnt[(t + 1) % 3] > 0) {
                    /* t beats (t+1)%3 → t wins */
                    won_type = t;
                    break;
                }
            }
        }
        /* present == 1 or present == 3: all TIE (won_type stays -1) */

        for (int i = 0; i < n; i++) {
            int id = arrived_ids[i];
            int t = types[i];
            if (won_type == -1) {
                results[id] = TIE;
            } else if (t == won_type) {
                results[id] = WIN;
            } else {
                results[id] = LOSE;
            }
        }

        /* Advance the round and wake everyone */
        round_++;
        arrived = 0;

		// 唤醒大家
		// 别忘了唤醒大家啊
        cond_broadcast(&cv);

        int result = results[tid];
        mutex_unlock(&lock);
        return result;
    }
}
```
