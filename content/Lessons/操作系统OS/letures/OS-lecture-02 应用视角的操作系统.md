## 程序：解释与编译
Operating System: A body of software, in fact, that is responsible for _making it easy to run programs_ (even allowing you to seemingly run many at the same time), allowing programs to share memory, enabling programs to interact with devices, and other fun stuff like that. (OSTEP)

- 《操作系统》课会彻底讲清楚什么是程序
	- as accurate as each bit is well-defined
	- 充当形式语义的第一课

### 单步解释执行 C 程序

- **理解计算机程序**
C -> simpleC
- each C program can be written as pattern of "each line only do a simple thing"
- and if we "step" a C program like we did in *NEMU*([[ICS-PA2 note]]), we can see the simple connection between C and asm code
	- C 像一个高级的汇编语言
	- 一旦把 C 语言和汇编一条条映射起来后，基于计算机程序是一个状态机的信念，你也可以相信 C 也是个状态机 (state-machine)



- **理解计算机程序**：函数调用

Tower of Hanoi: nightmare of C++ lesson

由于汉诺塔，`printf` 打印的副作用，交换 `return ` 时候的顺序，是困难的。
- 需要用递归仔细的**模拟**函数调用栈


- 非递归的汉诺塔
```python
def hanoi_iterative(n, source, auxiliary, target):
    # 栈中存放任务：(当前圆盘数, 起点, 辅助, 终点)
    stack = [(n, source, auxiliary, target)]
    
    steps = 0
    while stack:
        count, src, aux, tgt = stack.pop()
        
        if count == 1:
            # 状态1：只剩一个圆盘，直接移动
            print(f"第 {steps + 1} 步: 将圆盘 1 从 {src} 移到 {tgt}")
            steps += 1
        else:
            # 状态2：拆分任务。注意：栈是后进先出，所以步骤要反着压入
            
            # 步骤 3: 将 n-1 个圆盘从 aux 移到 tgt (最后执行)
            stack.append((count - 1, aux, src, tgt))
            
            # 步骤 2: 将最大的圆盘从 src 移到 tgt (中间执行)
            stack.append((1, src, aux, tgt))
            
            # 步骤 1: 将 n-1 个圆盘从 src 移到 aux (最先执行)
            stack.append((count - 1, src, tgt, aux))

# 调用示例
hanoi_iterative(3, 'A', 'B', 'C')
```

- 理解函数调用的终极考题
```c
int f(int n){ return (n<=1) ? 1 : f(n-1) + g(n-2);}
int g(int n){ return (n<=1) ? 1 : f(n+1) + g(n-1);}
```


- 理解函数调用：**程序状态机**
	- state-machine **是有严格数学定义的**。意味着你可以把定义写出来，并且用**数学严格**的方法理解它——形式化方法
	- **状态**（State）
		- `[StackFrame, StackFrame, ...] + 全局变量`
		- `状态 != "变量 + 栈 + PC"`
	- **初始状态**
		- C 从 main 开始执行
		- 仅有一个 `StackFrame (main, argc, argv, PC=0)`
			- `PC` 不是一个孤立的，一个比较典型的理解方式是每一个栈帧 (stack frame) 上有一个 PC
		- 全局变量均为初始值
	- **状态迁移**
		- 执行 `frames[-1].PC` 初的语句 (simple C)
		- `f(x): stack[-1].next_pc(); stack.push(x=x, PC=f)`
			- **完全机械**的操作
			- 许多体系结构都提供完成这个操作的 call 指令
			- 取 PC，执行指令..

### 编译器

- 什么是编译后的程序？
```c
struct CPUState { 
	uint32_t regs[32], csrs[CSR_COUNT]; 
	uint8_t *mem; 
	uint32_t mem_offset, mem_size; 
};
```

*编译正确*的可执行文件是所有编译结果 `a.out` 的一个子集，
- 编译器的优化等级和编译选项都影响了这个子集里的内容
- *编译器* 唯一***不能优化***的是一些比较特殊的 *语句*
	- 编译器看不到的，向外界调用的，例如 `printf()`，是稍后链接的，编译器不清楚它的行为，不能冒险。

- **思考**：最小的 `hello.c` 程序，打印一个 `"hello world"` 该如何做？
先考虑没有 `printf()`
如果你写一个 `hello1.c`

- 《计算机系统基础》说，程序是从 ` _start` 开始执行的
- 但是也不尽然![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260701155511.png)

```c
// hello1.c
void _start()
{
	while(1){
	}
}
```
这个程序 `hello1.c` ,进入这个空函数，是可以编译运行的，单步调试发现进入死循环

但如果，去掉死循环，让其返回

```
// hello2.c
void _start()
{

}
```

则会报 `signal SIGSEGV(address boudary error)`

- 因为状态机没有一个正常的停机状态，空函数 `ret` 下一条 pc 没有有效指令了

## 操作系统上的最小程序

- **话又说回来了**，如何写一个最小的 hello 呢？`minimal.S`
	- 可以用汇编代码来写最小的。参考 `syscall`, 调用 `SYS_write` 来做。
	- 程序不能关闭计算机，**只能**借助操作系统
```asm
movq $SYS_exit, %rax		# exit(
movq $1			%rdi		# 	status=1
syscall						# );
```

- 把**系统调用**的参数放到寄存器中
- 执行 syscall，操作系统接管程序
    - 操作系统可以任意改变程序状态 (甚至终止程序)

> [!Question] 最小可执行文件
>  为了理解操作系统上的程序，我们的目标是构造一个能直接被操作系统加载且打印 Hello World 的指令序列。如果你能想到这一点，剩下的一切都可以让 AI 帮助你。



```asm
    movq $SYS_##id, %rax; \ 
    movq $a1, %rdi; \ 
    movq $a2, %rsi; \ 
    movq $a3, %rdx; \ 
    syscall

```
## 操作系统上的应用世界
最小的可执行文件，和庞大的应用程序，在操作系统看来是一样的

- **操作系统**，我们其实是感知不到操作系统的存在的，只能感知到操作系统上运行的程序（进程）
- 操作系统提供了一系列 API
	- linux 上的很多工具本质也都是程序
	- 守护进程 daemon
- **AI** 时代我们有感兴趣的东西可以立刻去考虑

**一切程序都是一样的**
- 任何程序 = minimal.S = 状态机
- **可执行文件** 是操作系统中的对象，他们都是 ELF 可执行文件
	- 可执行文件是“*状态机初始状态的描述*”
		- 执行信息 (类型，入口地址，...)
		- 指令序列（普通指令/系统调用）、数据......


### 用正确的工具“打开”应用程序
- 工具体系思维 from *OS*
- 打开程序的执行：Trace (追踪)
	- In general, trace refers to the process of following _anything_ from the beginning to the end. For example, the traceroute command follows each of the network hops as your computer connects to another computer.
-  **System call trace (`strace`)**
- 理解程序是如何与操作系统交互的
    - (观测状态机执行里的 syscalls)
    - Demo: 试一试最小的 Hello World


用合适的工具，是对调试文本的一步步的提纯。

- NEXT [[OS-lecture-03 硬件视角的操作系统]]