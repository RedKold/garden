> [!Note] Terminal 到底在做什么
> terminal 只有把字符传输给操作系统是不变的，根本的，其余行为都是程序员可以定义的
> 「你好，我只是一个打字机」


## 从键盘到终端
键盘的老祖宗：keyboard
- 管风琴 (pipe organ)
- 1500s 击奏弦鸣乐器

### 打字机时代的遗产
- Shift/Caps Lock
- CR & LF
	- \r, CR (Carriage Return): 回车，将打印头移回行首
	- \n LF (Line Feed) 换行，将纸张向上移动一行
	- UNIX: \n include CR and LF both


### 电传打字机 (Teletypewriter)
实现了键盘打字、远端打印

### Video Teletypewriter
- milestone: VT100 (DEC, 1978)
- putchar：事实上的行业标准
	- 首个完整实现 ANSI Escapde Sequence 的终端
	- `80*24` 显示称为标号尊布局


### Thanks to Unicode
- ANSI Escape Code and toybox
- After *Unicode*
	- 可以画出很多 unicode art
- e.g.![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260326142744.png)

### 终端: 作为输入设备
- 输入字节流，直接送入操作系统
- quite like a Typewriter


### 伪终端和终端模拟器
- Pseudo Terminal: 想要多少有多少
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260326143209.png)
- 一对“管道”提供双向通信通道
	- 主设备 (PTY Master)：连接到终端模拟器
	- 从设备 (PTY Slave)：连接到 shell 或其他程序
- ` minitty`
	- unix 系统保留了对 tty 的称呼，抽象成了一种终端（teletypewriter）

**有趣的应用**：这种转发字符和时间，实际可以改造成“录屏”

**如果不满足于字符终端**：
- Sixel (six pixel)
	- 一个字符编码一个 6x1 的 Yes/No
	- 今天的终端大多支持
- 今天的终端设备多数都是模拟


## 终端和操作系统

### 终端：人机交互的第一个设备
- **是用户登录的起点**
	- 系统启动（kernel -> init ->getty）
	- 远程登录 (sshd->fork->openpty)
		- stdin, stdout, stderr 都会指向分配的终端
- 调用 login 程序


### Ctrl-C 发生了什么
***WAIT! TUI 是多进程的！***
- 本质还是字节流（terminal is typewriter）
	- Ctrl-C: End of Text (ETX), 03
	- Ctrl-D: End of Transmission (EOT), 04
	- `stty -a`
	- **操作系统**收到了这个字符，就可以采取行动


### 多进程的 TUI
作为操作系统的设计者，需要在 Ctrl-C 的时候找到一个当前进程


- **信号机制**
	- signal: 注册一个信号处理程序
	- kill: 终端程序执行，强行跳转到信号处理程序
	- 今天 sigaction

> [!Question] 更麻烦的问题：**信号发给谁**？
> 完整的 process tree, 有的在前台有的在后台，给谁发信号？
> fork () 的树状结构
> Ctrl-C 终止所有前台的进程


### 会话和进程组：机制
- 给进程引入一个额外编号 (Session ID, 大分组)
	- 子进程回继承父进程 Session ID
	- 一个 Session 关联一个 controlling terminal
	- Leader 退出时，全体进程收到 Hang Up (SIGHUP)
		- **tmux** will ignore this signal so **tmux** can run your program
- 再引入另一个编号（Process Group ID, 小分组）
	- 只能有一个前台进程组
	- 操作系统收到 Ctrl-C，向前台**所有进程发送**
- **并不优雅的设计**
	- But it's hard for us to find a clever way...

- **API**
	- setsid/getsid
	- setpgit/getpgid
	- tcsetpgrp/tcgetpgrp

### Job Control
实现类似窗口管理器的 Job Control

- 终端可以有一个“前台进程组”
	- 最小化 = Ctrl- Z（SIGTSTP）
		- 默认行为暂停进程，收到 SIGCONT 后恢复
	- 切换 = fg/bg (tcsetpgrp)
- 实现窗口栏上的按钮... 大费周章
	- maybe `tmux` is more clever

## UNIX Shell : 一门编程语言

UNIX Philosophy 背后的编程语言
- Keep It Simple, Stupid
背后是 UNIX Shell 编程语言支撑的
- 基于文本替换的极简编程语言 (Only `String`)
- 算数不支持
- 命令翻译成系统调用 (open, dup, pipe, fork, execve, waitpid,...)
	- **预处理**: $(), <()
	- **重定向**: cmd > file < file 2> /dev/null
	- **顺序结构**: cmd1; cmd2, cmd1 && cmd2, cmd1 || cmd2
	- **管道**: cmd1 | cmd2

> [!Note] 阅读 shell 手册
> `man sh: dash`
> `man -P cat sh | claude` 

- 什么事情是能干的，什么事情是办不到的
	- Broaden your horizen