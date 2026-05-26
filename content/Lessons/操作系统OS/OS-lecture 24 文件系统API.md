# Review & Comments
**存储（block）设备的抽象**
- `read_block(id)`
- `write_block(id, data)`


**进程直接访问设备文件？**
- `open("/dev/mmcblok0p1", O_RDWR);`, then lseek
- **抽象层**：文件系统


# 从块设备到文件系统

## Abstract Data Type
**地址空间上的数据结构抽象**
- 物理内存 random access `char[]` -> 虚拟内存和 `mmap -> malloc(), free()` -> `list, set, map`

**存储设备应该抽象成什么？**
- 内存->虚拟内存; 磁盘->虚拟磁盘 (字节序列 `vector<char>`)
	- 回到来虚拟化 *virtualization*
	- 支持 `read, write, lseek, ftruncate, ...`
- **需求**：怎么**管理**系统中众多的文件
	- `pid` 已经是很糟糕的抽象了
	- 游戏修改器：遍历了 `/proc/[pid]/mem` 来找寻要修改的进程。（很低效！）人类不友好

## 设备的抽象：目录树
**管理文件方法**：图书馆
- **背后原理**：信息的局部性
- 组织成**层次索引**结构

### 树状分层索引：利用信息的局部性
```

└── 学习资料
    ├── .
    ├── ..
    ├── .学习资料(隐藏)
    ├── ……
    └── 操作系统
```

- **很强大的抽象**。
- 所有文件皆文本，名字皆目录

### 设备=目录树
系统中不止一个存储设备
- 每个设备、每个 `partition` 都是一个目录树
- `/dev/nvme0n1p1`, 优盘，网络 (WebDAV)

**Windows: 很复杂**
- 所有操作系统对象的根 `\`, 例如 `\Device\HarddiskVolume1`, `\Driver\Ntfs`
    - `\\`: Universal Naming Convention 路径 (网络)
    - `\\server\share` → `\Device\LanmanRedirector\server\share`
    - 这个 `\` 不能 cd，但可以用 API 访问
- **盘符的历史遗留问题**
	- A 盘、B 盘：软盘（虽然现在没有软盘了）
	- **盘符**：实际上是 `\??\C:\`, `\??` 是 `per-user` 的用户空间

---
我们需要一个 API：把设备的目录树**放**到世界里

### 从无到有的目录树
OS need to provide a API, to put directory-tree into the **world**
- *最小 Linux*

> [!Note] 关于 `mount`
> The _mount_ command in Linux is used to attach file systems, devices, or partitions to a directory in the file system hierarchy. This process is called "mounting," and the directory where the file system is attached is referred to as the "mount point." To access files on external devices or partitions, they must first be mounted.
> 
> Syntax and Options
> 
> The basic syntax of the _mount_ command is:
> 
> mount [-t vfstype] [-o options] device dir
> 
> - **_device_**: Specifies the device or partition to be mounted (e.g., _/dev/sda1_).
>     
> - **_dir_**: The directory where the device will be mounted (e.g., _/mnt_).
>     
> - **_-t vfstype_**: Specifies the file system type (e.g., _ext4_, _xfs_, _ntfs_). If omitted, the system attempts to auto-detect the type.
>     
> - **_-o options_**: Specifies mount options, such as read-only (_ro_), read-write (_rw_), or others.

我们可以用 `strace` 追踪 `mount`

```
struct iso9660_primary_descriptor {
    uint8_t  type;              /* ISO9660_VD_PRIMARY */
    char     standard_id[5];    /* 必须是 "CD001" */
    ...
    char     system_id[32];     /* 系统标识符 (如 "Win32", "LINUX") */
    char     volume_id[32];     /* 卷标识符 (光盘名称) */ ...
```

- mount 可以挂载一个文件


## Aside：挂载一个文件
`mount` 必须要求一个块设备
- `fs.img`？
- **答案**：创建一个 `loopback`（回环）设备
	- 设备驱动把设备的 `read/write` 翻译成文件的 `read/write`
	- `drivers/block/loop.c`
		- 实现了 `loop_mq_ops` 不是 `file_operations`
-

**观察挂载文件的 `strace`**
- `lsblk` 查看系统中的 block devices (strace)
- strace 观察挂载的流程
	- `ioctl(3, LOOP_CTL_GET_FREE)`
	- `ioctl(4, LOOP_SET_FD, 3)`

## Filesystem Hierarchy Standard
FHS enables software and user to predict the location of installed files and directories
- macOS-UNIX's kernel, but not obey the Linux FHS

# 目录树 API
**增删改查**：
- `mkdir`: 创建目录
- `rmdir`：删除目录
- `getdents`: 读取目录
```c
int mkdirat(int dirfd, const char *pathname, mode_t mode);
int unlinkat(int fd, const char *path, int flag);
ssize_t getdents64(int fd, void *dirp, size_t count);
```

`globbing`

- `read, write, grep, glob` 几乎可以完成几乎所有运维工作了

### 硬(hard)链接 
> [!Note] 硬链接：允许一个文件被多个目录引用
> `fid` 其实也是存在的
> - 删除文件的系统调用称为 `unlink`


我用 `link a.txt b.txt`，然后 `cp a.txt c.txt`：

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260526111818.png)
- 可以看到 `a.txt` 和 `b.txt` 的 `fid` 完全一样

- `unlink` 可以删除
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260526111905.png)

### 软 (symbolic) 链接
> [!Note] 软链接：在文件里存储一个“跳转提示”
> 软链接也是一个文件
> - 当引用这个文件时，去找另一个文件
> - 另一个文件的绝对/相对路径以文本形式存储在文件里
> - 可以跨文件系统、可以链接目录...


**几乎没有限制**
- 类似于**快捷方式**
	- 比你想象的复杂：重命名、移动、U 盘驱动器号改变都能找到?!
- 制作 *Galgame*
- fun


### 软链接：伪造文件系统
#### [Nix](https://nixos.org/): 这个题我会
- **把所有软件包的所有版本都集中存储**
- `/nix/store/b6gvzjyb2pg0kjfwrjmg1vfhh54ad73z-firefox-33.1`
    - 然后用符号链接构建一个完全虚拟的环境
        - 完全的 deterministic: 由软件包的 hash 决定
- 可以随时随地构建 “任意” 环境
    - `nix-shell -p python3 nodejs`

#### 这是一个 persistent data structure 啊
- 还记得吗？random read + append-only write = 任意数据结构
    - /nix/store 是 “只增不减” 的，符号链接就是 random read
    - 对比 apt: 所有的修改都是 in-place 的
- 随时回滚 `nix-shell -p $(nix-env --list-generations | grep "3 days ago")`


# 文件的元数据
## 问文件添加属性
**软件=物理世界在信息世界的投影**
`ls -l`：查看对象的属性

## 更多的元数据
**Extended Attributes(xattr)**
```c
ssize_t fgetxattr(int fd, const char *name, void value[.size], size_t size);
int fsetxattr(int fd, const char *name, const void value[.size], size_t size, int flags);
```
- 每个文件维护一个任意的 `key-value dictionary`
- e.g. macOS 的 com. apple. metadata 会保存每个互联网下载文件的 `url`
	- macOS 下载：**下载了多少**？的信息。也是存在 `xattr` 中的
- **拷贝的时候可能会丢**
	- `cp -preserve=xattr`
	

**好用不火的操作系统特性**
- 文件系统的向量索引: `vectorfs`
- `vfs search cat static/Photos/ | claude -p -summary`


# Access Control List（访问控制列表）

