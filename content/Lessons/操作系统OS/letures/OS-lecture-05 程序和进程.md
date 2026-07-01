## A Simple Review
- 应用视角的操作系统
	- 操作系统 = 对象 + API
- 硬件视角的操作系统
	- 操作系统 = 程序
- **明确的数学模型**
	- 应用程序从 `main(argc, argv)` 开始执行语句
	- 及其从 `CPU Reset` 开始执行固件指令

# OS 正片开始

我们是如何把程序的状态机模型抽象成一个系统的？好像每个程序都独占计算机运行？
- Physical computer -> Virtual computer

- One of the most fundamental abstractions that the OS provides to users: the **process**

# 程序和进程

## 虚拟化: A Crazy Idea
- Tower of Hanoi's non-**recursive** version:
	- 可以在程序中模拟任何“另一个程序”运行
	- 像是实现了一个“**解释器**”
	- **操作系统**？！
- **CrazyOS**
```c
while(1){
	p = pickup_one();
	p->single_step();
}
```
- **这就是操作系统的主循环的样式**

- read `crazy-os` for more info
```c
struct proc {
    // Process "virtual machine" state:
    // Register & memory
    struct CPUState cpu;
    uint8_t mem[MEM_SIZE];

    // Operating-system internal state
    char buf[256];
    int buf_len;
};
```

一个进程必须的一些信息


## 程序 v.s. 进程

程序是语义（状态机）的静态描述
- 描述了初始状态和迁移规则
- 程序运行起来就成了进程 (running state-machine )

## 操作系统上的进程
**进程**：程序的运行时状态随时间的演进
 Key OS Concepts Demonstrated

- Claude read the proc-info program
  1. Process is more than just code - The OS maintains extensive metadata: IDs, credentials, resource limits, memory maps, open files, scheduling state, security contexts, etc.
  2. Kernel data structures - All this information lives in kernel memory (task_struct in Linux), exposed through controlled interfaces.
  3. Virtualization - Each process has its own view via namespaces (PID, mount, network, etc.) and cgroups for resource isolation.
  4. Security layers - Capabilities, seccomp, personality flags, and credentials provide fine-grained access control.


## \[OS API\] 查询进程状态
- `procfs`
	- `/proc/[pid]/`
		- 通过 readdir, open, read 访问进程信息
		- claude code to read
- `syscalls`
	- `getpid(), getppid(), etc..`


关于 `procfs`:
claude: 很好的问题！让我解释一下 /proc 目录的结构：
```
  /proc 目录的两类内容

  /proc/
  ├── 1/              ← 数字目录 = 进程目录 (PID 1 = init/systemd)
  ├── 2/              ← PID 2
  ├── 334/            ← PID 334 (比如 bash)
  ├── ...
  ├── cpuinfo         ← 字符文件/目录 = 系统信息
  ├── meminfo
  ├── mounts
  ├── self -> 3179/   ← 符号链接，指向当前进程
  ├── net/
  └── sys/

  ┌──────────┬──────┬────────────────┬──────────────────────────────┐
  │   类型   │ 命名 │      内容      │             例子             │
  ├──────────┼──────┼────────────────┼──────────────────────────────┤
  │ 进程目录 │ 数字 │ 单个进程的信息 │ /proc/1/, /proc/334/         │
  ├──────────┼──────┼────────────────┼──────────────────────────────┤
  │ 系统信息 │ 字符 │ 全局系统状态   │ /proc/cpuinfo, /proc/meminfo │
  └──────────┴──────┴────────────────┴──────────────────────────────┘

```



> [!Note] How to know the 1st process in OS?
> ```bash
> cat /proc/1/comm
> ```

res:

```bash
kasumi@iZuf6d880j0fyz9px7feiaZ:~$ cat /proc/1/comm
systemd
```





# 进程管理
## 创建进程
**进程管理系统调用**
操作系统是状态机的管理者
 - 进程管理 = 状态机管理
- Straight-forward idea:
	- using two syscall to do it
	- Create state-machine: `spawn(path, argv)`
	- Delete it `_exit()` 
	- make sense (e.g. windows)
- UNIX
	- copy state-machine `fork()`
	- reset（复位）state-machine: `execve()`

## \[OS API\] 创建状态机
```c
pid_t fork(void);
```
- UNIX's answer `fork`
	- 状态机的完整的复制

## fork () 的行为
man fork
> fork () creates a new process by duplicating the calling process.  The new process is *referred* to as the child process.  The calling process is referred to as the *parent* process.

- 如何区分两个状态机
- 新创建进程返回 0
- 执行 fork 的进程返回子进程的进程号——“父子关系”
	- a if-else can differ out.

## 进程树 process tree
- 进程的创建关系形成了进程树
	- A->B->C, if B terminal... what's ppid of C?
	- **托孤**
		- 子进程结束会通知父进程
		- 通过 SIGCHLD 信号
		- 父进程可以捕获这个信号


> [!Question] 什么是 ppid?
> PPID stands for **Parent Process ID**. It's the process ID of the parent process that created the current process.
> 
>   In Unix-like systems, every process (except the init process) is created by another process. The PPID identifies which process spawned the current one, forming a process tree hierarchy.
> 
>   For example:
>   - When you run a command in your shell, the shell's PID becomes the PPID of that command
>   - If a process's parent terminates, the process typically gets re-parented to init (PID 1)
>   - You can view PPID using commands like ps -ef or by reading /proc/[pid]/status


### fork () 的用处
- Fork Bomb
- `while(1) fork()`
- 1 变 2，2 变 4，指数增长
```shell
:(){ : | : &} ; :
```

- : is a name, a shell function
-  we use tunnel `|` to link them
- then we call this function `:`
- a fork bomb is formed

- fork () 的全量内存快照：应用
	- 共享信息预处理
		- calculate `prime_table`, then fork process to deal
		- cooler e.g: Android Zygote Process
		- 冷启动
	- 并行搜索
		- depth-first search, fork process for each branch
	- 沙盒隔离
		- checkpoint timely. recover if crash


### 软件系统的复杂性
- 理解 fork：习题
> [!Note] read the program and write the output of run
> ```c
> for(int i=0;i<2;i++){
> 	fork();
> 	printf("Hello\n");
> }
> ```

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260317114959.png)


In this example, we will find  the amount of lines output by execute `./a.out`  and by execute `./a.out | wc -l` is *different*. 
- 机器永远是对的
- *Assumption*: libc buffer
- *strace*
```
// Author: doubao-seed-2.0-code
why this program print 6 hello, but I use `./fork-demo | cat |wc -l` get a answer 8?

⏺ Let me look at the program first.

⏺ Searched for 1 pattern, read 1 file (ctrl+o to expand)

⏺ 这是一个经典的 stdio 缓冲区 问题。

  当你直接运行 ./fork-demo 时，stdout 连接到终端（TTY），使用行缓冲模式。每次 printf("Hello\n") 遇到换行符就立即刷新缓冲区，所以 fork() 时缓冲区是空的。

  但当你使用管道 ./fork-demo | cat | wc -l 时，stdout 变成全缓冲模式。这时：

  第一次循环 (i=0):
  - P0 调用 fork() → 创建 P1
  - P0 和 P1 各自 printf("Hello\n") → 数据写入各自的缓冲区但未刷新
  - P0 缓冲区: "Hello\n"
  - P1 缓冲区: "Hello\n"

  第二次循环 (i=1):
  - P0 调用 fork() → 创建 P2，P2 继承了 P0 缓冲区中的 "Hello\n"
  - P1 调用 fork() → 创建 P3，P3 继承了 P1 缓冲区中的 "Hello\n"
  - 4 个进程各自再 printf("Hello\n")

  程序退出时刷新缓冲区:
  - P0: "Hello\nHello\n" → 2 行
  - P1: "Hello\nHello\n" → 2 行
  - P2: "Hello\nHello\n" → 2 行
  - P3: "Hello\nHello\n" → 2 行

  总共 8 行。

  解决方法： 在 fork() 之前调用 fflush(stdout) 清空缓冲区：

  for (int i = 0; i < 2; i++) {
      fflush(stdout);  // 添加这行
      fork();
      printf("Hello\n");
  }
```

**这个案例最深刻的启示就是，** 进程在 fork 的时候，*会复制操作系统中所有的状态*。包括缓存区。

Another things
```c
int main()
{
	// without a '\n'
	printf("Hello");
	while(1)
	{
	}
}
```

this program will fall into dead-loop, if you use `ctrl+c` to terminal this program, you will not see the terminal print "Hello". That's because *buffer* strategy. 
- a  `'\n'` or `fflush` can solve this

## 重置进程
## \[OS API\]复位状态机
```c
int execve(cosnt char *filename, char*const argv[], char * const envp[]);
```

UNIX: only this one api to reset state-machine
`execve` 是唯一可以执行程序的系统调用

execve 设置了进程的初始状态
- argc & argv
- envp (environment parameter)
	 - `export:` tell shell to set envp, when create sub-process
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260317120006.png)

machine is always right.

## 销毁进程
## \[OS API\]销毁状态机
- 没有争议的
	- 立即销毁状态机，允许有一个返回值









