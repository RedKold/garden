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
- 