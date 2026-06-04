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
- 损坏=修复起来笔记哦啊麻烦

## 小文件系统和 FAT
*Drive*: 25''软盘很小，单面 160KiB。任何复杂的数据结构都显得奢侈


**核心数据结构**: File Allocation Table (**FAT**)
- 把磁盘分成 clusters (sector 的 `2^k` 倍)，`BPB_SecPerClus`
	- `cluster_t_fat[4096]` ，代表每个 cluster 的 next
		- 0 = free, -1 = EOF
	- next **指针**：对于小文件完全没有问题
- **缺点**：会产生碎片。空闲的 cluster 反复读写，一个文件的 cluster 会分散各处。随机访问碎片
	- tip: 老 windows 的磁盘碎片整理

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
我们在一个目录的结构体，存一个 `using dent = map<string, inode_t>`, 天然支持硬链接
- 因为 inode 和 dent 是分离的，**一个 inode 可以被多个 dent 引用：**


# 存储系统：应对崩溃
Crash: 内存中的一切都瞬间丢失
- Power loss, bug (kernel panic)
	- but persistent storage data cannot be **lost**

> [!Note] Crash Consistency
> Move the file system from one consistent state (e.g., before the file got appended) to another atomically (e.g., after the inode, bitmap, and new data block have been written to disk.)

## 暗藏杀机的数据结构
