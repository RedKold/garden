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

我们说，过程调用和返回由一个称为 *控制栈*(control stack) 的运行时刻栈管理。





Activation Records form a stack, but not quite complex like User stack you learnt at ICS.

- 激活记录 AR 是连续的存放在内存地址中的


> [!Note] **The advantage of placing the return value 1 st in a frame...**
> **The caller can find it at a fixed offset from its own frame**

- There's nothing magic about this organization
	- Can rearrange order of frame elements
	- Can divide caller/callee responsibilities differently
	- An organization is better if it improves execution speed or simplifies code generation
	- 好用的就是对的


**真实场景**: Real compilers hold as much of the frame as possible in registers
- Especially the method result and arguments


编译器在编译时就要确定活动记录的布局，并生成可以正确的在活动记录中访问 location 的代码

- AR layout and code generator must be designed together!


# Globals & Heap
- All references to a global variable point to the same object
	- Can't store a global in an **activation record**
	- 会导致全局变量被释放
- Globals are assigned a fixed **address once**
	- Variables with fixed address are **"statically allocated"**
	- 编译器决定了他们的位置，在程序的整个执行过程中它都在那
	- 
- Depending on the language, there may be other statically allocated values


Now we take a new look to memory

```
Memory:

+---------------------------+--- Low Address
|		Code				|
+---------------------------+
|		Static Data			|
+---------------------------+
|		Stack				|
+---------------------------+
|							|
|							|
+---------------------------+--- High Address

```


- A value that **outlives** the procedure that creates it cannot **be kept in the AR**


**总结一下内存层次**：
- The code area contains obejct **code**.
- The static area contains data (not code) with fixed addresses (e.g. global data)
	- Fixed size, may be readable or writable
- The stack contains an AR for each currently active procedure
	- Each AR usually fixed size, contains locals
- Heap contains all other data
	- In C, heap is managed by *malloc* and *free*
	- In Java, new...

> [!Note]  When we have heap and stack...
> Must take care that they don't grow into each other!
> **Solution:** start heap and stack at opposite ends of memory and let them grow towards each other


```
Memory:

+---------------------------+--- Low Address
|		Code				|
+---------------------------+
|		Static Data			|
+---------------------------+
|		Stack				|
+---------------------------+
|							|
|	...						|  ^
+---------------------------|--|
|		Heap				|
+---------------------------+--- High Address

```

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260505225441.png)



# Alignment



# 存储空间

**程序中的局部性**(locality)
- 时间局部性
- 空间局部性
- 充分利用计算机的存储层次结构

**堆空间分配方法**
- Best-fit
	- 总是将请求的内存分配在满足请求的**最小的窗口**中
	- 好处：可以将大的窗口保留下来，应对更大的请求
	- 坏处：慢+碎片
- First-fit
	- 总是将对象放置在 **第一个** 能够容纳请求的窗口中
	- 放置对象时花费时间较少，总体性能较差
	- **数据局部性好**

---


## **处理手工存储管理**
- 两大问题
	- **内存泄漏**(memory-leak): 未能删除不可能再被引用的数据
	- **悬空指针饮用**(dangling-pointer-dereference)：引用已经被删除的数据
- **其他问题**：访问非法地址
	- 空指针/数组越界

## **垃圾回收**
- 垃圾 (garbage)
	- 广义：**不需要**再被引用的数据
	- 狭义：**不能被**引用（不可达）的数据
- 垃圾回收：自动回收不可达数据的机制，解除了程序员的负担

> [!Note] 垃圾回收器的设计目标
> - 基本要求（静态/动态确定数据的类型）
> 	- 语言必须为 **类型安全**(typesafe)：保证回收器知道数据元素是否为一个指向某内存块的指针
> 	- 不安全的：C/C++
> - 性能目标
> 	- 总体运行时间：不显著增加应用程序的总运行时间
> 	- 停顿时间：当垃圾回收机制启动时，可能引起应用程序的停顿，这个停顿应该比较短
> 	- 空间使用：最大限度地利用可用内存，避免内存碎片
> 	- 程序局部性：改善空间局部性和时间局部性


### 可达性
- **可达性**：一个存储块可以被程序访问到
- **根集**(root set)
	- 不需要指针解引用就可以直接访问的数据
- 可达性
	- 根集的成员都是可达的
	- 对于任意一个对象，如果指向它的一个指针被保存在可达对象的某字段/数组元素中，那么这个对象也是可达的
- **性质**
	- 一单一个对象变得不可达，则它就不会再变成可达的

### **改变可达对象集合的操作**
- 对象分配
- 参数传递/返回值
- 引用赋值：`u=v`
- 过程返回


### 垃圾回收方法
- 引用计数垃圾回收
	- **跟踪相关操作**，捕获对象变得不可达的时刻，回收对象占用的空间
- 传递地跟踪所有的引用
	- **在需要时**，标记出所有可达对象，回收其他对象

### 基于引用计数的垃圾回收器
- 为每个对象有一个用于存放引用次数的字段，按如下方式维护
	- 对象分配：引用次数设为 1
	- 参数传递：引用次数+1
	- 引用赋值：`u=v`，`u` 指向的对象引用减 1，`v` 指向的对象引用加 1
		- 
	- 过程返回：局部变量指向对象的引用计数-1
- 如果引用次数为 0，在删除对象之前，此对象中各个指针所指对象的引用次数减 1

- eg
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260513103513.png)
	- $Y=X$ 这样的引用赋值，$Y$ 原来指向的 $B$ 悬空了，故减少引用次数

- **循环垃圾的例子**
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260513104048.png)
	- 这是一个连通的大垃圾块


### 基于跟踪的垃圾回收
- 不在垃圾产生时回收，而是周期性地运行

- 标记-清扫式垃圾回收
- 标记并压缩垃圾回收
- 拷贝垃圾回收

#### 标记-清扫式垃圾回收
- 标记-清扫式（mark-and-sweep）
	- 一种直接的全面停顿的算法
- 分成两个阶段
	- **标记**：从根集开始，跟踪并标记出所有的可达对象
	- **清扫**：遍历整个堆区，释放**不可达**对象
- 如果把数据对象看作顶点，引用看作有向边。那么标记的过程实际上是从根集开始的**图遍历**过程

- 回收算法
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260513104616.png)
	- 有点像一个BFS

- **例子**
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260513105232.png)


- **基本抽象分类**
	- 每个存储块处于四种状态之一
		- 空闲、未被访问、待扫描、已扫描
	- 对存储块的操作会改变存储块的状态
		- 应用程序分配
		- 垃圾回收器扫描
		- 回收


### 标记并压缩垃圾回收
- **标记并压缩回收器**（mark-and-compact collector）
	- 对可达对象进行**重新定位**(relocating) 并消除存储碎片
		- 把可达对象移动到堆区的一端，另一端是空闲空间
		- 空闲空间合并成**单一块**，提高分配内存时的效率
- 整个过程
	- 标记
	- 计算新位置
	- 移动并设置新的引用

> [!Thought] 这个...
> 类似于磁盘整理？


- 回收算法
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260513111250.png)
	- `free = free + sizeof(o)` 是一个遍历过程, 将空闲空间组织成尽量连续的
### 拷贝垃圾回收
- 堆空间被分为两个 **半空间**(semispace)
	- 应用程序在某个半空间内分配存储，当充满这个半空间时，开始垃圾回收
	- 回收时，**可达对象被拷贝到另一个半空间**
	- 回收完成后，**两个半空间角色对调**
- **优点**：不涉及任何不可达对象
- **缺点**：必须移动所有可达对象



### 开销的比较
- 标记-清扫式垃圾回收
	- 与堆区中存储块的数目成正比
- 标记并压缩垃圾回收
	- 与堆区中存储块的数目和可达对象的总大小成正比
- 拷贝垃圾回收
	- 与可达对象的总大小成正比


