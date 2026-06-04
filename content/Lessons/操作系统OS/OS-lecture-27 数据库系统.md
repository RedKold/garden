
## 什么是应用程序?
### Some insights
PPT skills
- `.xml` is a data structure file.
	- you can read this and modify this to generate *ppt* you wanna make.

> 软件是物理世界在信息世界的投影。

**软件天生**需要 persistent data
- 文件系统

应用程序也有持久化存储的需求。如何做？

### Everything is a File.
用文件，目录存储。
- 好处：UNIX 世界的工具都能用了：find， grep，...
	- you can even use *symlink* to implement "reference" (enroll/id/student -> ...)
- **自动**：agent friendly
e.g. jyy's OS wiki.
- **用目录**管理了学生的作业提交
	- 然后统计提交数量，和 reject（删除提交的 result）。很友好。

#### 优点和缺点 
Tool friendly 和 Agent friendly 的代价
- Crash consistency 实现需要小心
	- 最好是 append-only
- 性能稍差一些。
	- 每次都需要去文件系统 API 逛一圈，还有隐藏的读/写放大


### 有趣的 hack
直接在文件（虚拟磁盘）上构建数据结构
- 就像 ELF，BMP，
```
struct superblock {
	struct student *s;
	struct course *c;
}
struct student { char stuid[16]; ...};
struct course {char cid[16]; ...}
```

实现这个数据结构的 `CRUD`
- 你可以构建一个 `superblock` 结构体，完全模拟这个链表

```c
typedef struct super block{
	unsigned magic;
	...
}
```

直接在文件系统上做一个数据库。
- 甚至直接把**数据结构**mmap 到内存
	- 但是要自己维护 write back


> [!Note] Implement WAL Log
> WAL: Write-Ahead Logging
> 先写日志，在写数据
> - 对于任何数据结构操作，先把所有的 side effects 写入 wal
> 	- crash 时数据是一致的
> - 然后执行操作，清除 wal

invention of *single-level os*


# 数据库系统
> [!Note] **需求**：一个非常非常好的数据结构
> **好用**
> - 可以处理几乎任意类型的数据结构
> - 对任何人来说都比较好上手
> 
> **性能**
> - 全校一起选课，**系统不能崩溃**
> 
> **持久化**(Crash Consistent) 和**一致性**
> - 系统故障，数据不能丢
> - 数据结构需要支持 multi-write
> 	- all or nothing
> 	- 真的有同时满足这些需求的东西吗？


> [!Tip] **Reflection**: How we implement data structure?
> 我们的世界，本质是两两之间的二元关系
> **Our computer** is pointer machine
> - implement data base don't need *array*. 
> - `malloc(constant)`: alloc a fixed size memory: can implement any data structure.
> 	- array is a linear table.
> **Pointer**是对象之间的关系
> ```c
> struct node {
> 	char value[32];
> 	struct node *left, *right; // Pointer 	
> }
> ```

**离散数学**：整个计算机世界都是在二元关系上 formalize 的

## Relational Database 关系型数据库
- A relational model of data for large shared data banks
	- Edgar F. Codd: 1981 *Turing Award Winner*
	- **关系代数**
### 1. 关系代数 (Relational Algebra) - 查询的理论基础

你笔记里提到了，Codd 提出关系模型时也定义了**关系代数**，这是所有 SQL 查询的数学基础。基本操作只有几个：

|操作|含义|SQL 对应|
|---|---|---|
|**选择 (Selection)**|选出满足条件的**行**| `WHERE` |
|**投影 (Projection)**|选出指定的**列**| `SELECT` |
|**并 (Union)**|合并两个关系| `UNION` |
|**差 (Difference)**|在一个关系中不在另一个的元组| `EXCEPT` |
|**笛卡尔积 (Product)**|所有组合| `CROSS JOIN` / `FROM a, b` |
|**重命名 (Rename)**|给关系或属性改名| `AS` |
|**连接 (Join)**|按条件组合两个表| `JOIN ... ON` |

> **革命性在于**：这些操作是**封闭的** — 输入是表，输出也是表。所以可以任意嵌套组合。


- Everything is a *table*
	- 每行一个对象；对象可以用 `id` 索引其他对象
	- struct = table 的一行
	- **可以构成**一个 3 重甚至更多的循环，来处理这个简单的数据模型
	- 但是如果我们能更快，即时有更多循环也很快呢？
		- **革命性的**

## 从模型到实现：数据库系统
“ACID”数据库开启软件的新时代
- A (Atomicity), C (Consistency), I (Isolation), D (Durability)
	- Serializability: 并发执行<=>某种串行之行
	- Strong crash consistency: 系统 crash 也不会损坏或者丢失

### ACID 数据库
- 并发执行的效果=一把大锁保平安
- 性能>>粗粒度大锁
- 完全自动的崩溃恢复
**关系数据库：海量的实现优化**
- 查询优化 (rewriting)
- 索引 (B+ Tree 等数据结构)
- 缓存、分库分表、并发控制、读写分离

SQL **是一个非常复杂的并发程序**

- T_1: `tx_begin(); x = 1; may_crash(); y=2; tx_commit()`
- T_2: `tx_begin(); y = 1; sleep(100000); x=2; tx_commit()`
	- 2 Phase Locking: Tx 可以执行任何代码->边执行边上锁
	- deadlock detection
		- met deadlock ->  rollback, commit -> release
	- MVCC (Multi-Version Concurrency Control)
		- 不是修改数据而是追加新版本
		- **读不阻塞写，写不阻塞读**。

### 关系型数据库查询总结

**本质**：用关系代数 + 声明式 SQL，让数据库自动找到最优方式从表中获取数据。

**SQL 执行流程**：
```
SQL → Parser → Rewriter → Optimizer → Executor → Result
```
优化器利用关系代数等价变换（如下推选择、结合律重排）生成最优执行计划。

**核心查询类别**：
- 单表查询：`SELECT ... WHERE`（选择+投影）
- 多表连接：`JOIN ... ON`（$\bowtie$）
- 聚合查询：`GROUP BY ... HAVING`（分组+聚合函数）
- 子查询：嵌套的 SELECT

**MVCC 要点**：
- 每行保留多版本 `(value, created_by_tx, expired_by_tx)`
- 事务看到的是开始时刻的**快照**（Snapshot Isolation）
- 读不阻塞写，写不阻塞读
- 写写冲突仍需解决（通常用锁+回滚）
- 代价：需要更多存储 + 后台版本回收（VACUUM/GC）

### SQL Lite
> TODO


## 实现国民级的应用
“透明”的 SQL 就有些困难了

> [!Note] e.g. 微信
> 回复消息：
> ```
> <message>
> ------
> <your reply>
> ```
> 客户端会渲染这个消息支持跳转。

- SQL 依然正确，**但是数据库引擎**压力很大。
- You can't never predict what **query** will a programmer write
- 提供了功能=被滥用
- 任何 Systems 都无法避免的 tech debt
	- `fork()`


### 解决方案：做减法
- 只支持一个“够用”但可以 scale out的 subset

- Key-value store
- 全靠 key 管理数据
	- `user: {uid}`
	- `user:{uid}:like_history (a list)`
	- List: support append/pop/range
- **关键的性能优化**

- 简化了，可以分布式部署了。

### Redis: Everything is in In-memory
Core: key-value store (GET, SET)
- support store data structure: Stirng, List, Set, Hash, Sorted Set, ..., Stream
- support **query**


### NoSQL & NewSQL
**我们发现**：key-value 还是不如 sql 好用
提供 SQL 的一个子集
- MongoDB (Document
	- Key->BSON (Binary JSON)
	- e.g.: create a new document (message), then append it to `user:{uid}.messages`
- Cassandra (Colume)
	- CQL (Table model)
### Vector Database
> [!Note] **向量数据库**
> 之前讲的都是数据结构。
> - Embedding model: 将 context 映射为高维向量（距离越近，语义越近）
> - Vector database：存储和检索向量的数据库
> 	- ANN: Approximate Nearest Neighbor 返回“最近”的文档



