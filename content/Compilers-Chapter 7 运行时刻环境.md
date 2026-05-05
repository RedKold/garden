# Runtime Organization
- We have covered the front-end phases
	- Lexical analysis
	- Parsing
	- Semantic analysis
- Next are teh back-end phases
	- Optimization
	- Code generation

 ---

First question: 
- What we are trying to generate
三个主要议题:
- **Management of run-time resources**
- Difference: **静态结构(compile-time)** and **动态结构(run-time)**
	- Static structures are things that **exist to compile time.**
		- *global constant*, *global variable*
	- Dynamic structures are those are the things that exist or **happen at runtime.**
		- Stack storage
			- 和过程的调用/返回同步进行分配集合回首
		- Heap storage
			- 数据对象更长寿
			- malloc, free
				- 手动回收
- Storage organization


- Execution of a program is initially under the control of the operating **system**
- When a program is invoked
	- OS allocates *space* for the program
	- The code is loaded into part of the *space*
	- The OS jumps to the **entry point** (i.e., "main")


**回忆一下内存体系结构**。[[ics - section 6 内存层次结构 memory hierarchy]]

- Other space = Data space
```
+-----------------------+
|		Code			|
+-----------------------+
|		Data			|
+-----------------------+
```

- **Compilers** is responsible for 
	- Generating **Code**
	- Orchestrating **use of the data area**

代码 Code 及其数据 Data 的布局需要协同设计
- The code and the layout of the data, need to be designed together


# Activations

代码生成的总体目标-Two goals:
- Correctness
- Speed
- 做到两者是复杂的 trade-off 和考虑，需要提出良好的运行时结构

- Complications in code generation come from trying to be **fast** as well as **correct**


We have **Two assumption**
- Execution is sequential; control moves from one point in a program to another in a well-defined order
	- *Concorrency* program do not obey this!
	- Because next move could be in a totally different thread
- When a procedure is called, control always returns to the point immediately after the call
	- **高级控制结构**违背这个假设，**比如异常处理**
	- Catch and Throw exception



- An invocation of procedure $P$ is an activation（调用） of $P$
- The *lifetime* of an activation $P$ is 
	- All the steps to execute $P$
	- Including all the steps in procedures $P$ calls

> [!Note] 变量的生命周期 (lifetime)
> The lifetime of a variable $x$ is the portion of execution in which $x$ is defined
- Note that
	- Lifetime is a **dynamic** (run-time) concept
	- Scope is a **static** concept


 为什么用栈处理调用？这其实基于一个很自然的观察:
 - Observation:
	 - When $P$ calls $Q$, then $Q$ returns before $P$ returns
这正是后进先出的栈结构。

**我们可以用树表示激活的过程（激活树）**
- Since activations are properly nested, a **stack** can track currently active procedures


---

来看看内存结构，现在我们加入了**活动记录栈**
```
Memory:

+---------------------------+--- Low Address
|		Code				|
+---------------------------+
|		Stack				|
+---------------------------+
|							|
+---------------------------+--- High Address

```


# Activation Records（活动记录）
- The information needed to manage one procedure actiavtion is called an *activation record (AR)* or ***frame***
	- Frame （帧）只是另一种叫法
- If procedure $F$ calls $G$, then $G$ 's activation record contains a mix of **info** about $F$ and $G$
	- 调用者和被调用者的信息都有

> [!Question] 我们为什么要保存活动信息？
> The reason is that there is some states associated with each procedure activation, that is needed in order to properly execute the procedure. And we have to track that somewhere
> 保存调用和被调用需要的信息

$F$ is "suspended" until $G$ completes, at which point $F$ resumes（恢复执行）
- 我们必须把 $F$ 的 Activation Record 保存在某处，以保证正确执行

$G$ 's AR contains information needed to
- Complete execution of $G$
- Resume execution of $F$


- A example frame
```
+-------------------+
|	result			|
+-------------------+
|	argument		|
+-------------------+
|	control link	|-----> to the activation for caller's function
+-------------------+
|	return address	|-----> the address of instruction/memory that we are supposed to jump after the execution of F completes
+-------------------+
```




Activation Records form a stack, but not quite complex like User stack you learnt at ICS.

- 激活记录 AR 是连续的存放在内存地址中的
- 