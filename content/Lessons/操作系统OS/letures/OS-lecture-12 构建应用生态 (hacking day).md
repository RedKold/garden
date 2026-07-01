## Review & Comments
**链接和加载**
- 本质讲的是 `execve` 的行为
	- 继承文件描述符、singal hander 等
	- 加载 ELF 文件（数据结构）和 INTERP 的所有 PT_LOAD, PT_GNU_STACK, PT_TLS...
	- 设置进程的初始状态 (argc, argv, envp, auxv, regs)
	- **加上库函数的行为**。就理解了一切
	- Shebang (#!)


> [!Note] 课程目标：
> **阅读手册、指导 AI 写代码来理解任何系统调用**，并且弄清楚**为什么要这样设计**

# 见证历史
## 这些系统调用是哪里来的

- 最早的版本甚至没有 fork()
	- Shell 关闭所有打开的文件，然后为 0,1fd 打开终端文件
	- 从终端读取命令行
	- 打开文件，把加载器代码复制到内存执行（相当于 exec）
	- exit 会重新加载 shell
- **微内核**的思想，但在那个时代很超前

- *We ONLY need a syscall(send/recv)*
	- 靠消息来传送

**Andrew S. Tanenbaum**: Minix
- Minix1 (1987): UNIXv7 兼容，也是 Linus 实现 Linux 的起点
- Minix2 (1997): POSIX 兼容，随书送代码
- Minix3 (2006): POSIX/NetBSD 兼容，全功能，一度是世界上应用最广的操作系统 (Intel ME)

？

Minix 是 UNIX 之后的经典教学操作系统。
[Minix](https://jyywiki.cn/OS/demos/virtualization/minix/)
- 16bit
- 致命的缺点：慢

## Linux 的诞生 (1991.8.25)
- “我写了一个加强版的 Minix，快来试试吧”

- 最早的 Linux 依赖 Minix 的工具链，运行 GNU gcc，bash，...没有自己的磁盘

CUDA 编程模型很烦人类...

**名场面**：
- "The single worst company we've ever dealt with..." fxxk you, NVIDIA

## "Just for fun"
> [!Note] The story of an accidental revolutionary
>  Revolutionaries aren't born. Revolutions can't be panned. Revolutions can't be managed. Revolutions happen...
> ——David Diamond

## 质疑、回应与时代的车轮

在 comp. os. minix 上关于 Linux 的讨论


## 反复发生

- **AI 时代**从 0 到 0.1 变得前所未有的容易
	- 好奇就好了
	- CrazyOS
	- Claude Mythous: finding bugs is easy
	- caveman: why use many token when few do trick
- **阅读文档**：养 claude-code 的 skill



# 回顾：初始状态

- **进程的初始状态**
	- `execve(path, argv, envp)`
	- SP points to `[argc, argv, 0, envp, 0, auxv]`
	- path 和 interp 被加载到内存，PC 是 interp 的 ELF entry
- **计算机系统的初始状态**
	- CPU Reset
	-  运行 Firmware 代码加载操作系统
	-  终究，操作系统会执行一个 execve，启动**第一个进程**，变成 “服务提供者”
-  **操作系统的第一个进程？**
	- 这个 “长出” 了你看到操作系统世界全部的进程，它到底在哪里，它做了什么？

## initramfs
> Can we control the first process that Linux Load?

- 制作我们的 initramfs
    - 可以只有一个 init 文件
        - Linux 会按照一个硬编码的 /sbin/init, /etc/init, … 逐个尝试 execve
    - (系统启动后，Linux 还会增加 /dev 和 /dev/console)
        - 需要给 stdin/stdout/stderr 一个 “地方”

> [!Note] Unpack initramfs
> You can even unpack your OS's initramfs


## 构建“真正”应用世界的系统调用

在 initramfs（初始化内存文化系统）之后，我们会执行

switch_root 命令背后的系统调用：
```c
int pivot_root(const char* new_root, const char* put_old);
```

- Changes the root mount in the mount namespace of the calling process.
    - 我们也可以在 “最小 Linux 上” 复现这个行为
    - 真实的 Linux: 驱动加载、NetworkManager、tty 字体变化……都是在 switch_root 之后 systemd 拉起的
        - 例子：[NOILinux Lite](https://zhuanlan.zhihu.com/p/619237809)
- 可以 umount 把 put_old 释放

# 故事的结尾

## 操作系统会到达一个确定的初始状态
- initramfs + /dev/console + execve (init)

- **在**到达确定的初始状态之后，操作系统只是不断地提供 API 给你使用
## 操作系统 = 对象 + API
- 所有的其他对象 (`procfs`, `devfs`, …) 都是系统调用创建和管理的
    - 进程管理: `fork`, `execve`, `exit`, `waitpid`, `getpid`, …
    - 操作系统对象和访问: `open`, `close`, `read`, `write`, `pipe`, `mount`, `mkfifo`, `mknod`, `stat`, `socket`, …
    - 地址空间管理: `mmap`, `munmap`, `mprotect`, `msync`, …
    - 以及一些其他的机制: `pivot`_`root`, `chmod`, `chown`, …

## Unix → Minix → Linux，到达成熟稳定的状态
- 精彩的故事在这个 API 抽象层上延续
- 在 Linux API 上，**没有什么东西是不能做的**

# 应用生态
- **应用生态**成就了操作系统的繁荣
	- 有一套核心工具集来支撑
	- 基本的运行库，coreutils，安装工具，版本公里，系统管理....
- **前互联网时代**
	- DOS/Windows 3. X/95 软盘/光盘
- **进入互联网**
	- `AppStore`, `apt`, `rpm`, `PyPI`, `npm`, `HuggingFace`, `ollama`


## e.g. Debian
**长出完整的英勇事迹**


