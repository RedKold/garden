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