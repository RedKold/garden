# Review & Comments
**文件系统** = 数据结构
- 任何实现了 `struct fuse_operations` 的东西都是文件系统
	- 例子：
	- `procfs`, `gGalgame FS`
需要在**块设备**（按块访问的字节序列）**上实现数据结构**

> [!Warning]
> Crash safe is another box of evil
# 块设备上的数据结构
## 真实的块设备
- **可以是一个实际的设备**
	- GPT (GUID Partition) $\in$ UEFI
	- 也是一个*数据结构*，帮助你分出 `/dev/nvme0n1p1`
- **也可以是虚拟化的设备**
	- Physical Volume -> Volume Group -> Logical Volume (可以一对多、多对一、多对多)
		-  if you use `strace` to trace system call, you will find `ioctl` is significant
	- NVMe namespace: “硬件级”的“LVM”

## 实现文件系统的代价
**读、写放大**
- `bread`, `bwrite` 必须按块读写
- update one byte, what happened?
	- `bread() x 1`
	- `bwrite() x 1`
	- **需要整个块**
	- **数据结构更新**。写放大（更新 metadata）

**缓解办法：缓存**
- 和 memory hierarchy 一样，在更快的存储器（内存）中留一份数据
	- `bread/bwrite` 总是访问 cache
	- 如果 cache miss 再去取存储设备
- locality 原理

## 进入文件系统设计
> [!Note] 需求分析
> - 我们要 persist 什么？
> 	- 文件数据 (bytes)
> 	- 文件 metadata (name, size, permissions, timestamps, ..) 
> 	- 目录结构 (name -> 文件的映射)
> - 观察一些统计规律
> 	- Most files are small; a few big files use most sapce
> 	- Directories are typically small (<20 entries)


**核心数据结构问题**
- 如何存储文件的 metadata? (size, mode, time, ...)
- 文件的 virtual to physical map table 什么数据结构？怎么存储？
- 目录如何实现


## Super Block
数据结构的“根结构体”
- 所有的数据结构都有一个 root (as a *entrance*) 入口
	- 链表 head，树结构的 root 各种文件的 Header
- 文件系统把“入口”放在 *Superblock*(fixed position)

**描述文件系统布局的“元信息”**
- Magic number, block size
- Total/free blocks, total/free inodes
- 数据结构的起始位置、大小、...
	生命周期
	- **创建**：`mkfs`（格式化）时写入 Super Block
	- **读取**：`mount` 时内核读取 Super Block，确定文件系统类型和布局

**This is the most important block**
- 损坏=修复起来麻烦

## 小文件系统和 FAT
*Drive*: 25''软盘很小，单面 160KiB。任何复杂的数据结构都显得奢侈


**核心数据结构**: File Allocation Table (**FAT**)
- 把磁盘分成 clusters (sector 的 `2^k` 倍)，`BPB_SecPerClus`
	- `cluster_t_fat[4096]` ，代表每个 cluster 的 next
		- 0 = free, -1 = EOF
	- next **指针**：对于小文件完全没有问题
- **缺点**：会产生碎片。空闲的 cluster 反复读写，一个文件的 cluster 会分散各处。随机访问碎片
	- tip: 老 windows 的磁盘碎片整理

本身没有 inode，而是存储关于文件的元数据的题目录条目，并且直接指向所述文件的第一个块。**这导致不可能创建硬链接**
- [[链接的语义]]

### 工作原理

FAT 本质上是一个**链表**，但不是用指针，而是用"跳转表"实现的：

```
文件 A:  cluster 3 → cluster 7 → cluster 9 → EOF
         fat[3] = 7, fat[7] = 9, fat[9] = -1

文件 B:  cluster 2 → cluster 5 → EOF
         fat[2] = 5, fat[5] = -1
```

**目录项**中只需存文件的**起始 cluster 号**，后续通过查 FAT 表即可遍历整个文件。

### 特点

|优点|缺点|
|---|---|
|数据结构极其简单，适合小容量设备|**随机访问性能差**——要读第 100 个块，必须沿着链表走 100 步|
|只用数组就能实现，开销小|大文件会产生很长的链，访问延迟大|
|易于理解实现|文件大小受限（需乘以 cluster size）|

FAT 至今仍广泛用于 **U 盘、SD 卡**等嵌入式存储设备（FAT32），主要优势就是**简单和兼容性好**。

### 目录
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260604133207.png)
- `struct dent entries[0];` "目录文件"
	- DOS "8+3"文件名，`AUTOEXEC.BAT`
	- 不支持硬链接，长文件名需要打补丁
	- 目录也是一个 FAT，也是一个 linear table
		-  有很多 dent (directory entry)


- `DIR_FstClusHI`
	- High word of first data lcuster number for file/directory descrbed by this entry.
- `DIR_FstClusLO`
	- Low word of first data lcuster number for file/directory descrbed by this entry.


## UNIX 的选择：分离
**G(V, E)** 顶点和边分开存储
- `inode` (Index Node): Node, Links, User/Group, Size, ...
	- fstat 直接返回 inode 中的信息即可
- 通过 bitmap 分配（have better *locality* than Fat's）
-  `dent`（边）

```
using dent = map<string, inode_t>
```

目录就是一个**文件名到 inode 编号的映射表**。每个 dent 是一条边：

```
"/home/user/a.txt" ——inode_num—→ inode_42
```

inode = Index Node，是**文件/目录的元数据节点**，包含了：
```c
inode {
    mode      // 文件类型 + 权限（rwx）
    links     // 硬链接计数
    uid/gid   // 属主和属组
    size      // 文件大小
    timestamps // atime, mtime, ctime
    block pointers  // 指向数据块的指针
}
```

`fstat` 系统调用几乎就是从 inode 里直接查数据返回。

### 索引：联合数据结构 (Fast/Slow Path ) ext2
- 12 个直接指针+1 indirect + 1 double indirect + 1 triple indirect
- 多级索引


### 目录 (dent)。边
我们在一个目录的结构体，存一个 `using dent = map<string, inode_t>`, 天然支持硬[[链接的语义|链接]]
- 因为 inode 和 dent 是分离的，**一个 inode 可以被多个 dent 引用：**



# 存储系统：应对崩溃
Crash: 内存中的一切都瞬间丢失
- Power loss, bug (kernel panic)
	- but persistent storage data cannot be **lost**

> [!Note] Crash Consistency
> Move the file system from one consistent state (e.g., before the file got appended) to another atomically (e.g., after the inode, bitmap, and new data block have been written to disk.)

## 暗藏杀机的数据结构

数据结构更新涉及 **multiple-location writes**
来举一个例子：`insert(x, y)`：在 `x` 后面插入 `y`
- a program sequence
```c
insert(x,y){
	n = x->next;
	x->next = y;
	y->prev = x;
	n->prev = y;
}
```
我们必须保证这是一个**原子化**做完的事情。

- `write(fd, buf, 4096)` // append
考虑一个 bwrite（在一块上写的）
```
分配数据块: bwrite(d-bitmap)
写入数据块: bwrite(data)
更新元数据(size, time, index, ...): bwrite(iblock)
```

1. `balloc() = 40`
2. `write(40, data)`
3. `update(inode)`


## 层次化存储结构带来的问题


`Block cache` -> `queue(DMA)` ->存储系统上的计算机->queue
- 计算机系统会按照他认为的“最佳”顺序写入（乱序执行）
	- e.g. HDD 的磁头运动规划；SDD FTL
- 于是 `bread, bwrite` 就成了 `relaxed memory model` 了
	- 魔鬼的盒子😭

**可能发生**：（以上面的三行程序为例子）
- `write` 和 `update` 都干了，但是 `balloc` 没有做。所以这 40 块在 FS 看来还是 free 的。

> [!Tip] Systems: “先挣钱，后还债”
> - **送去修电脑**(File System Check)
> - “快速格式化”的文件是能恢复的。


### File System Checking (FSCK)
我们可以根据磁盘上已有的 $G(V,E)$，恢复出“最可能”的数据结构

我们可以根据文件系统的 `inode` 的状态，猜测 `balloc, write, update` 谁没有被做好。恢复文件系统。


## 乱序执行：后果
> [!Note] 磁盘掉电时，写入请求的顺序是**没有任何保证的**
> - multi-write 会产生怎么样的后果?
> 	- `bwrite(inode)`, `bwrite(d-bitmap)`, `bwrite(data)`
> 	- 裸奔...
> - 早期 SSD 的更严重的问题！
> 	- FTL crash -> corrupted data structure!


## 实现崩溃一致性：重新理解“数据结构”
理解数据结构有*两个视角*
- **View 1：存储实际数据结构**
	- 链表、二叉树、......
		- 文件系统的“直观”表示
		- Multi-write (crash unsafe)
- **View 2: Append-only 记录所有历史操作**
	- 容易实现崩溃一致性
	- 一旦有历史操作，就可以随时重构 redo 实际的数据结构了
		- Write-ahead log ("Redo log")
实现一个 **journal**
- `bwrite(data)`
- 在 `bwrite` 完成之前，journal 总是把这个记录 on disk 存储。


## Append-only + Lazy Update
**Store buffer**
- Store 写入 CPU 本地缓存，慢慢传递给其他处理器
**LSM(log-structured Merge Trees)**
- Only `MemTable` 可写（例如 Skip List），带 WAL (crash consistency)
	- MemTable 写满，触发 Flush 向磁盘持久化
	- **分布式友好的**
- 磁盘上全是 Immutable SSTable
	- 如果结构不够好就 Merge（创建新 SSTable）
	- 类比 Memory Hierarchy。小的树会 *override* 大的树



![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260605143818.png)

## 文件系统中的 Write-ahead Log
**写入 TXBegin; operations**
- 写完后 *flush* 等待数据落盘
**写入 TxEnd**（代表事务结束）
- 在此之前写完后 *flush* 等待数据落盘
- **数据落盘**"*happens*-before" **TxEnd 落盘**
	- 因而可以确定
**写入文件系统(Checkpointing) & 崩溃恢复**
- replay (operations) 写入文件系统、回收日志空间
- 如果写入时崩溃，fsck 将为 replay (operations) 恢复数据

## Some Journaling Tricks
- 批处理
- Checksum
- Metadata journaling

### Data Journaling vs. Metadata Journaling

**Data Journaling (全量日志)**：把 data block、inode、bitmap **全都写进 journal**，然后再 checkpoint 到磁盘。崩溃时可以通过 journal 完整恢复数据。但**性能差**——每次写文件都要把数据写两遍（journal 一次 + 最终位置一次）。

**Metadata Journaling (元数据日志)**：只把 inode、bitmap、dent 等**元数据**记录到 journal，**数据块直接写到最终位置**。

```
写入流程 (Metadata Journaling):
1. write(data) → 直接写到数据块位置（不经过 journal）
2. write(journal, inode + bitmap) → 元数据写到 journal
3. fsync(journal) → 确保 journal 落盘 (TxEnd)
4. checkpoint: 把 inode/bitmap 写到最终位置
5. 回收 journal 空间
```

**为什么可以接受？**
- data 占磁盘写入的绝大部分。跳过 data 的 journal 写入能大幅减少写放大
- 保证文件系统的目录结构是一致的（inode、bitmap、dent 之间的一致性）
- **但数据可能丢失**：如果 data 落盘后、metadata journal 完成前崩溃，文件数据已经写了但 inode 没更新，文件系统 fsck 时不会知道这些块属于某个文件（可能变成孤儿数据块）

### Linux ext3 的三种 Journaling 模式

| 模式 | journal 记录 | 一致性保证 | 性能 |
|---|---|---|---|
| `journal` (全量) | data + metadata | 最强：数据和元数据都一致 | 最慢 |
| `ordered` (默认) | 仅 metadata，但**先写 data，后写 metadata** | 保证 data 不会指向垃圾（data 已落盘才更新 inode） | 中等 |
| `writeback` | 仅 metadata，data 写入顺序无保证 | 最弱：data 可能指向已崩溃时未写入的垃圾数据 | 最快 |

**ordered 模式**是实际中最常用的权衡：它保证了不会出现 "inode 指向了垃圾数据" 的情形，因为 data 落盘 happens-before metadata journal 写入。




## `sync()` 系列系统调用
- sync - Synchronize cached writes to persistent storage
- 把所有东西都落盘再做

**全局同步**
- 同步所有文件系统中的数据
- 等于 performance bug (很卡)
	- 只在关机/命令行使用
**文件描述符同步**
- `syncfs(fd)`：同步 fd 对应的文件系统
- `fsync(fd)`：同步文件 data+全部 metadata
- `fdatasync(fd)`：同步文件 data
	- 仅同步改变了的相关 metadata
	- 最弱，但依旧可以控制 data loss
	- **例子**：实现应用程序中的 WAL (bflush 的效果)

### 例：用 `fsync` 实现 WAL

```c
write(wal_fd, log_entry, len);   // 写入日志
fsync(wal_fd);                    // 保证日志落盘（TxEnd 语义）
write(data_fd, data, len);        // 写入实际数据
// 崩溃后可通过 wal 恢复
```

这对应笔记中的 **Write-ahead Log 协议**：TxEnd（`fsync`）之前的所有写操作必须在 TxEnd 落盘之前已经落盘（happens-before 关系），从而保证事务的原子性和持久性。

# 应用程序的崩溃一致性
**应用程序的多个写入操作也会被乱序吗？**
- **YES**
	- because metadata *journaling*. the operations blow is *crash unsafe*
	- `Path("a.txt.tmp").write_text(...)`
	- `unlink("a.txt")`
	- `rename("a.txt.tmp", "a.txt")`

### 为什么应用层的多个系统调用会被乱序？

**1. 单个系统调用本身就不是原子的**

`write(fd, buf, 4096)` 在文件系统层面拆成多个 `bwrite`：
```
bwrite(bitmap)   // 分配数据块
bwrite(data)     // 写入数据
bwrite(inode)    // 更新 size 和 block pointer
```
这些 `bwrite` 可能被块设备的**写队列乱序执行**（类似 relaxed memory model）。

**2. 系统调用之间没有事务边界**

考虑"安全保存"模式：
```c
write(tmp_fd, data, len);    // ① 写入临时文件
fsync(tmp_fd);                // ② 确保 tmp 落盘
unlink("a.txt");              // ③ 删除原文件
rename("a.txt.tmp", "a.txt"); // ④ 原子重命名
```
如果在 `unlink` 之后、`rename` 之前崩溃：
```
磁盘状态：a.txt 已被删除，a.txt.tmp 存在
→ 文件彻底丢失了！
```

**3. Metadata journaling 加剧了问题**

Metadata journaling 保证的是**文件系统内部数据结构的一致性**（inode/bitmap/dent 之间是对的），但不保证**应用程序语义的一致性**。

在 `ordered` 模式下，data 总是先于 metadata 落盘，所以 FS 自身不会出问题（不会有 inode 指向垃圾块）。但从应用角度看：
```
时间 →
write(tmp)       → 数据落盘了
unlink(a.txt)    → dent 更新 journal 了，落盘了
                  ↳ 崩溃在这里！！！
rename(...)      → 还没执行
```
文件系统的视角：inode 和 dent 一致，没问题。
应用的视角：**文件丢了**。

### 如何保证应用层的崩溃一致性？

应用层需要自己实现类似事务的机制，经典做法是**带 complete flag 的 WAL**：

```c
// 1. 写 WAL 条目，包含完整操作 + complete flag = 0
write(wal_fd, log_entry);
fsync(wal_fd);

// 2. 执行实际操作
write(tmp_fd, data, len);
fsync(tmp_fd);
unlink("a.txt");
rename("a.txt.tmp", "a.txt");
fsync_dir(".");   // 确保 dent 落盘

// 3. 标记 WAL 完成 (complete flag = 1)
//    崩溃恢复时：若 complete = 1 则已做完，否则回滚
mark_complete(wal_fd);
fsync(wal_fd);
```

这就是**数据库**的做法——通过 WAL 把多个操作包装成一个事务，崩溃后通过 redo/undo 恢复。

- *2016*, `Node.js`：*无法安全地保存文件*


