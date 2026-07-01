**并发编程**：back to sum++
```c
void T_sum(){
	mutex_lock(&lk);
	sum++;
	mutex_unlock(&lk);
}


```

`buf[len++] = elem;`
`mapping[key] = value`

**性能观测**：
- 完全的 serializability（release -> acquire, 且确保可见）
- 导致很差的 scalability
	- 性能无法随 CPU/线程/机器增加而增长
	- 如何 scale up & scale out


# 并行算法

## Scale Up & Scale Out：分解问题
**任何并行计算**，本质都是 **计算图**

```
mutex_lock(&lk);
job = get();
mutex_unlock(&lk);

job->run();	// mostly thread-local computation
job->done(); // enable other jobs
```

- `job->run()` 的同步条件：等待所有 predecessors 执行完毕
	- 条件变量：直接等待这个条件
	- 信号量：需要 predecessor 那么多把🔑

**Scale up & scale out**
- 只要 `job->run()` 的时间>>互斥的时间，就可以 scale




## （经典）高性能计算
> "A technology that harnesses the power of supercomputers or computer clusters to solve complex problems requiring massive computation." (IBM)
[The world's most expensive love-seat](https://dl.acm.org/doi/10.1145/359327.359336)

- 源自数值密集型科学计算任务
- Today's AI ([chat jimmy](chatjimmy.ai))

## 高性能计算程序：特点
**可以大规模并行**
- 物理世界具有“空间局部性”
	- 划分网格，除去边界都可以独立计算
- 细节：HPC-China 100: 测试基准 Linpack
	- 求解稠密矩阵的 $Ax=b$
	- 可以分块计算
- Newton's Method 把非线性系统转化为稀疏线性系统求解 (FEM)

- **Embarrassingly parallel**（几乎不需要同步/通信）
	- Tree search
	- Monte Carlo estimation
	- 视频逐帧处理


-  [Mondelbrot set](https://cn.mathigon.org/course/fractals/mandelbrot)
	- 每一个像素的计算都是完全独立的


**通常计算图容易静态切分（机器-线程两级任务分解）**
- 生产者-消费者就能实现“按轮同步”（每次计算一个 $\Delta t$）
	- MPI: "message passing libraries"
	- OpenMP: "multi-platform shared-memory parallel programming" (C/C++ and fortan)


# 并行数据结构

如果有很多的并行数据结构操作（`sum++`）呢？

## Aside: thread_local

我们都不喜欢 `int sum_local[]`
- 语言机制的设计者也不喜欢
- 于是我们有了 `thread_local` keyword (C++11/C23)
	- 每个线程“自动”得到拷贝
	- 允许是任何类型（可以赋初值）

```c
thread_local int sum_local;
void T_sum(){
	thread_lcoal int t;	// compiler error
	sum_local++;
}
```
编译 thread-local storage
- 不同的线程&sum_local 必须得到不同的地址


## 并行数据结构
数据结构 (Abstract Data Type)
- array, list, tree, graph,...
- 天生是“分散”存储的
- 于是又有访问的**局部性**了
	- 读写数据结构的一部分未必要锁住整个数据结构
	- **我们天生就有局部性**
		- **树**和**图**关心周围结点的性质
		- 集合关心元素是否属于他
		- 

**Key Idea**：锁的拆分
- 能用原子指令就不用锁 (Atom inst)
	- (mymalloc 的 online judge)
- Reader/writer locker
	- 读是可以不止一个线程做的
	- 读写会冲突（不对称锁）
	- 多个线程持有读锁，一个线程持有写锁
- Segment/element-wise lock

### 例子：Hash Table
- `hash(key)` 将转化为数组下标
- 在该位置存储对应的 value
	- O (1) 插入、删除、查找
	- per-bucket lock

