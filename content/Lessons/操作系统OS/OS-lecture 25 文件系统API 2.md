# Review & Comments
**文件系统 API**
- 目录树管理（mount, umount）
- 增删改查
	- 目录树：`mkdir, rmdir, getdents`
	- 链接：`link, symlink`
	- 元数据: `mode, xattr, ACL`
- 回忆数组和差分数组，差分数组使得区间修改问题变得很简单，这提示我们数据结构的设计可以极大改善时间复杂度
- **文件系统**作为一个 Abstract Data Type 可以支持怎样的操作
---
**今天主要做的**：
- 监控
- 快照
- 覆盖
- 终极的虚拟化

# 监控
> [!Note] “监控”的需求
> **在文件系统改变之后通知我**
> - 什么文件变了
> - For example: *Web server*
> 	- Flask, next. js,...

还记得软工课做 web server 的时候，flask 确实每次保存都会刷新
**用 CRUD**实现：获取一个 `last modifed` 列表
- 最简单的来说，一行 `diff` 脚本就可以解决

- 目录如果几百万个文件，**这个方法是遍历的可能太慢**。

## 实现文件系统的监控 
`inotify`
- NAME
       `inotify` - monitoring filesystem events

- DESCRIPTION
       The  `inotify` API provides a mechanism for monitoring **filesystem events.**
       `Inotify` can be used to monitor individual files, or to monitor directories.   When  a  directory is monitored, `inotify` will return events for the directory itself, and for files inside the directory.
**文件管理器自动刷新**
**编辑器自动重载**
- 你的印象：VS Code 可以 monitor 到 claude code 对文件的编辑，但是在 vim 中，你需要重新关闭再打开才能载入


**任何监控都是可以实现的**
> [!Note] Everything is a state-machine
> 操作系统是一个巨大的状态机，是一个在解释器 (比如 `NMEU`) 上运行的程序。代码有函数调用，构成状态机的转移。
> 所以操作系统（OS）是可以监控的，所有状态都可以
> - strace, ltrace, inotify

- **启发**：我们可以从“操作系统内核执行”提取信息
	- 插入一个 programmable 的 probe
	- 我们追踪文件修改相关的 api 即可
- Congrats! 你发明了 eBPF (extended Berkeley Packet Filter)

-  **eBPM VM**: 轻量级 in-kernel “只读”**虚拟机**
	- RISC like, r0-r10 11 个寄存器
	- `BPF_CALL <helper>: bpf_get_current_pid_tgit(), bpf_map_lookup_elem(),`
	 - 严格控制副作用，禁止破坏内核地址空间

eBPM 的**动机**：更好的检测和监控操作系统

## 一些反思
```
diff <(stat -c '%n %y' **) <(sleep 2; data>a.txt; stat-c '%n %y' **)
```

**机制与策略的彻底分离**
- Specification is all you need

# 快照
> [!Note] 在文件系统上实现快照 (snapshot)
> **Random read + append-only write = 持久化数据结构**
> - **Git**：我就是一个**持久化数据结构**。*Tree*
> - 持久化数据结构的好处：
> 	- git reflog 包含所有的历史，trees, commits, etc.
> 	- 除了 `git reset --hard` 清除的工作区，只要不 `git gc` 都可以恢复


Git **的三大类对象**（存在 `.git/objects` 中）
- blob: `blob [length] \0[content]`
- tree: `[mode] [filename]\0[hash]\n...`
- commit: `tree [hash]\nparrent [hash]...`
	- **压缩存储**

Git **的快照**一切都是存在文件系统中的。
- 很干净的设计
## Git：数据结构操作
**分支和提交**
- `refs/heads/[branch]` 是指向 commit object 的指针
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260529145256.png)
- `git checkout -b new` ->新建一个 `new` 的文件（指针）
- `git commit` ->创建 tree/commit object，更新 `refs/heads/[branch]`

**HEAD “指针的指针”**
- `.git/HEAD`：文本文件 `ref: refs/heads/main`
- `git checkout` 就是改 `HEAD`

**Stash**
- 有两个 parent：HEAD 和 next stash
- 压入暂存区，而不 commit
	- `git stash push`
	- `git status list`

### 处理分叉的 Commits
**Common Ancestor**上的分叉
- Local: CA -> A -> B
- Remote: CA -> C -> D
	- Merge：增加一个 E 节点，合并 B&D
		- merge 是一个新结点
	- Rebase：CA->C->D->A'->B'
	- 如果本地没有 A 和 B，可以 fast-forward: CA->C->D

**Rebase**: A'和 B'是怎么来的？ 
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260529151822.png)
- 这个图很清楚。
我们本质是
```bash
git reset --hard D
git cherry-pick A # git diff CA A | git apply
git cherry-pick B # git diff A B | git apply
```

- **我们变了基**，很危险
	- 把 branch 签到 D 上
	- 我们先对照 `A` 和 `B`，提取出 `diff` 变化（类似一个编辑序列），然后像一个补丁，作用在 `C->D` 上
	- 即使成功，也可能隐藏逻辑冲突，需要 review 代码

**Git 是为单线程设计的**
- `git stash` 地狱
- 多次被紧急要求切换到优先级更高的 feature 分支上修 bug？

Worktree (Since 2015, Git 2.5)
- Persistent data structure 管理的是**快照**？

## 文件系统级的快照
干脆用 persistent data structure 实现文件系统
- btrfs: `ioctl(fd, BTRFS_IOC_SNAP_CREATE,...);`

对**持久化数据结构**，实现快照如此自然
> 只需要指针就好了


# 覆盖
## 目录的“拼凑”
一个神奇的 idea：你看到的目录可以是“拼凑”出来的
- 把 `upper/` 和 `lower/` 拼起来，形成一个“假”（虚拟化的）目录
	- 写入都会写到 `upper/`
	- 重名的，优先看到 `upper/` 的版本
- Kill Application：光盘打包
	- 刻盘工具 `burn /path-to-dir/`, `/dev/cdrw0`
	- 对每一个写入不同的 CD-Key
		- 16 个 `cdrw drives`，同一个 lower, 但 verify-key. exe 是不同的

## OverlayFS ("UnionFS", 联合挂载)

```bash
mount -t overlay overlay \
-o lowerdir=L1:L2:..., upperdir=U, workdir=W \
path_to_merged
```

**有趣的行为**
- `workdir` 是文件系统内部使用的临时空间
- 只允许一个 upper，但可以有多个 lower
- 在 `merged` 中删除 `lower` 的文件，会在 `upper` 里创建一个 `witeout` 文件。
	- 网吧管理
	- 考试机-系统一键还原

- **实现了新的增删改查**。
	- lower 是不会变的


## Docker 的多层 Overlay
- 每一个 RUN 都会创建一个 layer
	- 从文件系统角度，RUN 工作在 `upper` 上
	- 这个 `upper` 会“增加”到下一次 RUN 的 lower stack 顶部

> Overlay 不是增删改查实现的


# 文件系统和数据结构
- 可以写一个 `fs.c` 实现文件系统
- `Filesystem in User Space(FUSE)`：内核把 `lookup, read, write, ...` 转发到 FUSE driver
- 只要实现 `struct fuse_operations` 就行
	- 从此文件系统不再是*CRUD*
	- 你可以不受约束地实现“任意数据结构”
	- 比如 `/proc` 里的真正隐藏目录

你可以 hack 文件系统：因为 OS kernel 会转发 `read, write` 给你的 api

## FUSE Hacks
- 把任何远端变成文件系统
	- sshfs：远程目录变成本地目录
	- aitfs：远程 git 仓库变成本地目录
- 把任何数据改造为文件系统接口
	- `ffs`：把“数据库”(json,...) 变成文化系统

- **非常规的**文件系统操作
	- 真正隐藏 `学习资料`

