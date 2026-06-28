## 课程简介
### 成绩组成
- 期末考试 50%
- 随堂期中测验 10%
	- 5.1? take it easy
	- 如果期中不来，以期末为准
- 实验 40%
	- MiniLab Only

## 反思
Computer can solve *mathematically well-formed* 问题
- *软件工程师*将现实世界的问题建模求解，并提供给人访问的借口（App, GUI, ...）
	- **软件是物理世界过程在信息世界中的投影**
	- AI 很擅长这种投影


学 CS 仍然是有意义的。

- **复杂系统的实现**仍然需要抽象，《操作系统》传递的 takeaway message 是抽象层的设计与实现

> 行こう、次の舞台へ
- 操作系统是最后一门**编程课**
	- Why program can do, and why cannot?
- Stack Overflow is dead, AI is alive (but based on material on stack overflow)

> Talk is cheap, show me the code -> Code is cheap, show me the talk


## 操作系统概述
- No need for *accurate* description
	- 操作系统是帮我们更好地开发程序的
	- 很多事办起来很复杂，所以需要操作系统

- **理解操作系统**：


> [!Question] 复习：理解计算机硬件（电路）
> *数字逻辑电路*学了什么？
> - 极简的公理系统（导线、时钟、逻辑门）
> - 支撑复杂的处理系统

>  You can even implement a simple Logisim system using C. It's rather simple

Content as Code
- 没什么东西不能用代码表示
	- Pic/Anime: SVG, Python
	- Slides: Markdown, HTML, CSS,

CLI is a fun design: text to run

讨论狭义的操作系统
- 一个状态机


- The Birth of new Era: ENIAC (1946.2.14)
- Turin's implement try:
	- logistic circuit, obey state-machine model
		- has *PC*
			- but next PC using physical wire, "hard-wire"
			- Re-programming need to re-design the lines


- 1940s computer hard wire
	- logistic gate: 布尔电路
	- memory: 水银延迟线存储器
		- **延迟线存储器 (Delay Line Memory)：** 利用声波在充满水银的管子中传播的时间差来存数。
	- program: 打孔纸带 (Punched Cards)

- 1940s OS
	- No OS
	- 甚至没有编程语言
		- 画流程图，写机制代码，戳纸带
	- 程序跑起来就很厉害了

- 1950s-1960s 的计算机软件
	- 高级语言和 API 诞生 (Fortran, 1957)：一行代码，一张纸片
	- 作业调度
	- **库函数** + 管理程序排队运行的调度代码
		- 写程序（戳纸带）非常费事
		- 计算机很贵

- 1960s-1970s 的计算机硬件
	- 集成电路、总线出现
	- 可以同时载入程序了
		- 有了 switch 和 recover
		- 之前只有加载
	- 更丰富的 IO，异常/中断处理机制
- 1960s OS
	- **有了资源虚拟化**
		- 几乎就是今天的计算机了
		- 内存和磁盘的空分复用非常自然 (MMU -> 进程)
		- 借助中断的 CPU 时分复用
- 1970s+ 的 OS 和应用生态
	- UNIX 奠定了 pre-AI 时代计算机世界的应用生态
	- 信号 API、管道（对象）、grep
	- socket
	- procfs
	- **大家族**
- wonder: 2030s+ OS?
	- 不只是给“人”用了
	- Model context 也可以和 OS 交互

## 乐趣和动机
- 理解人类文明的高光时刻
	- 讲操作系统 API，如何支撑了不起的软件

- 实现童年梦想的途径
	- 每一节课都让你“能力增长”



- NEXT [[OS-lecture-02 应用视角的操作系统]]