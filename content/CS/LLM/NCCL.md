Objective:
- 理解通信原语与集合通信


# NCCL Collective Operations
*Collective Operations* have to be called for each **rank** (hence CUDA device), using the same count and the same datatype, to from a complete *collective* operation. Failure to do so will result in undefined behavior, including hangs, crashes, or data corruption.


### 1. 八个操作的语义

记 `k` = rank 数，`N` = 每 rank 的元素数。

|操作|每个 rank 输入|每个 rank 输出|一句话|
|---|---|---|---|
|**AllReduce**|N|N（所有 rank 相同）|求和后**人人有份**：`out[i] = Σ_r in_r[i]` |
|**Broadcast**|root 有 N|所有 rank 各 N|从 root **复制**到所有人|
|**Reduce**|N|仅 root 有 N|求和后**只有 root 拿到**|
|**AllGather**|N|k·N（所有 rank 相同）|把各家的 N 个**拼起来**，人人一份|
|**ReduceScatter**|k·N|N（各拿一块）|先求和，再**按 rank 把结果切开分掉**|
|**AlltoAll**|k·N|k·N|**各发各的**：第 j 块发给 rank j，第 i 块来自 rank i|
|**Gather**|N|仅 root 有 k·N|拼起来，但**只有 root 拿到**|
|**Scatter**|仅 root 有 k·N|N|root 把数据**切开分发**给各 rank|

注意 `Broadcast`、`Reduce`、`Gather`、`Scatter` 这四个是**非对称**的（有 root），另外四个是**对称**的（人人出、人人得）。


### AllReduce
- performs reductions on data
	- reduction: for example, sum, min, max
	- reduction: a series data -> one data

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20261027150446.png)

e.g. *sum* allreduce operation between *k* ranks
- each *rank* will provide an array in of $N$ values, and receive identical results in array out of $N$ values, where `out[i] = in0[i] + in1[i] + ... + in(k-1)[i]`.

- 对每个 rank 做 reduce，然后结果 broadcast 到所有 rank


### Broadcast
The **Broadcast** operation copies an N-element buffer from the root rank to all the ranks.

![image](https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/_images/broadcast.png)

> [!note] Broadcast operation
> all ranks receive data from a "root" rank
> 
> **Important**: The root argument is **one** of the *ranks*, not a device number, and is therefore impacted by a different rank to device mapping.
> > **`root` 参数是 rank 编号，不是 device 编号；因此它会受"rank 到 device 映射关系"的影响。**

### Reduce
和 AllReduce 的功能一样，**但是只在某个特定的 root rank**的接收缓存区 (receive buffer) 存储结果
（stores the result only in the receive buffer of a specified root rank）

![image](https://docs.nvidia.com/deeplearning/nccl/user-guide/docs/_images/reduce.png)

> [!note] Reduce operation
> one rank receives the *reduction* of input values across ranks
> **note**: The root argument is one of the ranks (not a device number), and is therefore impacted by a different rank to device mapping.


> [!tip] link with AllReduce and Broadcast
> $\mathrm{\mathrm{Re}duce}+\mathrm{Broadcast}\equiv \mathrm{All\mathrm{Re}duce}$


### AllGather


