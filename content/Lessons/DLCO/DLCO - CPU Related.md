# CPU Related
## CPU  中央处理器
- 指令执行过程
- CPU 的基本组成
	- **操作元件**(组合逻辑元件)
	- 状态/存储元件（时序逻辑元件）
- 数据通路与时序控制
- 计算机性能和 CPU 时间
### CPU 基本组成
![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251114103116.png)


### CPU 基本结构
- 计算机的五大组成部分
- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251114103435.png)

- Control Unit （控制器）
	- 指令的控制部件
- Datapath （**数据通路**）
	- 指令的执行部件
- Memory
- Input
- Output

 **除了存储元件**，**都是组合逻辑电路**。
#### 数据通路
- **两类元件**
	- 组合逻辑（操作元件）
	- 时序逻辑（状态元件、存储元件）
- 元件间的连接方式
	- 总线连接方式
	- 分散链接方式
- 数据通路的具体工作
	- 进行数据存储、处理、传送

- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251114103855.png)

- **数据通路**是由 **操作元件** 和 **存储元件** 通过总线方式或分散方式连接而成的进行数据存储、处理、**传送的路径**。

#### 存储元件：寄存器和寄存器组
- 寄存器 (Register)
	- has a `Write Enable-WE` signal
	- `0`: when clock edge come, output stay the same
	- `1`: when clock edge come, output is becoming the input
- 寄存器组 (Register File)
	- **Two** read port (组合逻辑)
		- busA, busB
		- address given by `RA`, `RB`
		- After a Access Time, busA busB becoming valid
	- **One** write port (时序逻辑)
		- Write Enable == 1:
			- when clock edge come, write the value from busW to the register assigned by RW


#### 时序控制
![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251114112426.png)
- **锁存延迟**(Latch Prop, a.k.a Clk-to-Q)，即触发器的锁存延迟
- **时钟偏移**(Clock Skew): 由于工艺、走线延迟等原因造成的时钟信号的偏差。
- Longest Delay Path: **关键路径**。组合逻辑的最长路径。


- **现代时钟周期**


#### CPU 性能
- CPU **执行时间**
	- CPI: Cycles Per Instructions
- Time to do the **task**
	- response time
	- execution time
	- latency
- Tasks per day, hour, sec, ns...
	- 吞吐率 (throughput)
	- 带宽（bandwidth）

- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251114112601.png)

CPU 性能 (CPU performance): User CPU time
系统性能 (System performance): 一般指没有其他负载时的响应时间


$$
吞吐率=单位时间内运行的作业(指令)数(有或无负载/干扰)
$$


- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251114115921.png)


**产品宣称指标**：Marketing Metrics

- MIPS
- MFLOPS




### 设计处理器的步骤
确定 ISA 后，我们进行处理器设计的大致步骤：

- **分析每条指令的功能**，并用 RTL（Register Transfer Language）来表示
- 根据指令的功能给出所需的元件，并考虑如何将他们互连
- 确定每个元件所需控制信号的取值
- 汇总所有指令所涉及到的控制信号，生成一张反映指令与控制信号之间关系的表
- 根据表得到每个控制信号的逻辑表达式，据此设计控制器思路


我们用 RISC-V 所例子

## 数据通路


指令执行结果总是在下一个时钟到来时开始保存在**寄存器**或 **存储器** 或 **PC** 中


### 指令开始时，取指部件中的动作
Fetch instruction: `Instruction <- M[PC]`
- All instruction is same
- When clock signal come, PC is updated to `s->dnpc` (dynamic next pc)
![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251121114200.png)
- previous: 上一条指令遗留下的指令
![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20251121114600.png)

- R-type 控制信号：
	- `RegWr = 1`: 我们需要写入寄存器
	- `ALUASrc = 0`: 
		- 我们不需要 `imm` ,自然也不需要立即数
	- `ALUctr = add/slt/sltu`
	- `Jump` 和 `Branch` 都是 `0`，只有 `J/B` 才为 1
	- `ExtOp = x` (任意)，即 R 不需要拓展立即数
	- `ALUBSrc = 0`
		- 我们不需要 `imm` ,自然也不需要立即数
	- `MemWr =0`
		- R 型不需要读写内存
	- `MemtoReg = 0`
		- 同理


- U-type 控制信号
![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20251126141030.png)


## 多周期处理器

思想：
- 把每条指令的执行，**分为多个阶段**。每个阶段在一个或多个时钟周期内完成；
- 每个时钟周期称为一个**状态**，期间最多完成一次访存或一次寄存器读写或一次 ALU 操作。
- 每个时钟内的执行结果在下个时钟到来前，保存到相应存储元件或者稳定地保持在组合电路中
- **时钟周期的宽度以最复杂**阶段的所用的时间为准。

**思考**：指令有几个阶段？
1. fetch inst  `IF`
	- read storage
	- read inst due to pc. put it in `IR`
	- IR will not be update at every clock. So it need a write-enable
	- when fetch-inst ended, `alu` output is `PC+4`, send it to input of PC. But, you can't update `pc` at every clock. so pc need a write-enable
2. decode / read register `ID`
	- after control-logic-delay, update control signal
	- do read register, and decode
	- `alu` is free at this period. You can use it 
3. alu `EX`
4.  read/write store `M`
5. write the result back `WB`

**多周期处理器的好处**
- 时钟周期更短
- **不同指令所用的周期数可以不同**
	- 如 `Load: 4? 5? cycles`
	- Others: `2? 3? 4? cycles`


简单指令系统-对应的多周期 CPU：

- **控制器**：提供一个 of，zf 的寄存器
- **指令寄存器**：
- **MAR**
	- 给出数据地址
![image.png|800](https://kold.oss-cn-shanghai.aliyuncs.com/20251128102308.png)
- **有个三态门**：`PCout, MARout,` **控制能否向总线** 写出
- 我们把地址总线、控制总线、数据总线简化画成了一个大总线。
	- `PCout, MARout` 走地址总线。**数据送到主存**

#### 各类指令执行过程
- 取指令并计算下一条指令地址：`IFetch`
- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251128104812.png)
- 译码并取数（公共操作）记为 `Rfetch/ID`
- **投机计算**：当前时钟结束，下个时钟来之前可以投机计算
	- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251128104914.png)
	- **所有控制信号不是同时生成的**。
- R-型指令的执行。两个时钟周期：`RExec, RFinisih`
	- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251128105456.png)

- I-型指令的执行，需要两个时钟周期。`IExec, IFinish`
- Load 指令：地址已经投机计算，还需两个时钟周期。`lwExec, lwFinish`
	- 根据投机计算好的地址（MAR中）到主存中取数，送MDR，再将MDR内容写入Rt。
- Store 指令：地址已经投机计算。还需要两个时钟周期 `swExec, swFinish`
- Jump 指令：计算转移目标地址。送 PC。需一个时钟周期
- 


- 状态转换图
- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251128112928.png)


**多周期控制器的实现**
**回忆单周期**：控制信号在指令执行过程是不变的。用真值表可以反映指令和控制信号的关系。

但是多周期
- 每个指令周期不同
- 控制信号取值不同
前面我们已经讨论了指令执行的不同阶段，受此启发，**可以构造状态转移图**。
![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251128114515.png)
![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251128114530.png)
- 和状态转移图完全对应。


- 流水线数据通路
![Pipeline datapath.jpg|400](https://kold.oss-cn-shanghai.aliyuncs.com/Pipeline%20datapath.jpg)

- **WB**主要是逆向数据通路，用于往寄存器回写数据。
- 只要RegWr=1，产生逆向数据通路，就可能产生冒险！
### 流水线的三种冒险
1. **数据冒险**：当指令在流水线中重叠执行时，后面的指令需要用到前面的指令的执行结果，而前面的指令尚未写回导致的冲突，称为数据冒险（也称为**数据相关性**）。
2. **结构冒险**：当一条指令需要的硬件部件还在为之前的指令工作，而无法为这条指令提供服务，那就导致了结构冒险。（这里结构是指硬件当中的某个部件、也称为**资源冲突**）。
3. **控制冒险**：如果现在想要执行哪条指令，是由之前指令的运行结果决定，而现在那条之前指令的结果还没产生，就导致了控制冒险（实际上就是 risc-v的跳转指令引起的，跳转指令要经过2个周期后才会出现**跳转结果**）
具体的冒险可以看后面：[[#延迟]]

##### 结构冒险  
- 同一个部件可能被不同阶段同时使用，比如寄存器可能同时被 ID 读，被 WR 写  
- 修改寄存器，实现**上半周期写，后半周期读**  
- 内存也可能被同时使用（读取指令和数据），可以对**指令和数据划分区域**，使得互不干扰  
##### 数据冒险  
- 流水线执行时，前面指令执行完成之前后面指令就开始执行（比如后面的指令依赖 load 的结果，但是此时 load 还没完成 wr）  
    - ![image.png|500](https://thdlrt.oss-cn-beijing.aliyuncs.com/20240101192049.png)  
- 基本流水线只会遇到 RAW **写后读**问题  
- 最小化冲突条数  
    - 如上图最初会冲突 3 条  
    - 使用上半周期写下半周期读能减少一条（Wr、ID 可以在同一阶段）  
    - 转发技术：数据计算出来要早于存储，希望计算出来之后尽快使用结果，把数据从流水段寄存器中**直接取到 ALU 的输入端**  
    - ![image.png](https://thdlrt.oss-cn-beijing.aliyuncs.com/20240101200437.png)  
    - 途中给出了三种转发形式（将数据向前供 ex 使用），要注意的是 load 只有在 ME 之后才得到值，因此 load 无法做到完美转发，中间可能还是需要**等待一条指令**，称为 load-use  
- 阻塞指令执行  
    - 硬件阻塞  
    - 软件插入 NOP  
##### 控制冒险  
- beq/jmp 等跳转指令造成的延迟，会在真正跳转执行在原先的 PC 继续错误执行几条指令  
    - ![image.png|500](https://thdlrt.oss-cn-beijing.aliyuncs.com/20240101192256.png)  
- 未优化之前需要等待三条指令，使用分支预测进行优化  
- 静态预测  
    - 总是简单的预测条件不满足，或者使用一些简单的启发式规则  
    - 为了尽可能减少等待，将预测提前到 ID 阶段，如果预测失败也**只需要等待一个时钟周期**  
- 动态预测  
    - 利用**最近转移发生的情况**，来预测下一次可能转移还是不转移  
    - 从 **BHT 表**中寻找，看这条分支指令以前是否执行过，如果没有则插入新的表项到 BHT **(默认设置为顺序取**)，如果找到了则直接使用预测表中的转移目标地址进行转移（预测发生时，选择“转移取“；预测不发生时，选择“顺序取”）  
- 预测位  
    - ![image.png|450](https://thdlrt.oss-cn-beijing.aliyuncs.com/20240101211516.png)  
    - ![image.png|450](https://thdlrt.oss-cn-beijing.aliyuncs.com/20240101212036.png)  
- JMP 指令会回造成**一个时钟周期的延迟**，即在第二个时钟周期跳转（不能放在第一个周期，不然耗时太长，拖慢总体上的时钟）  
- 编译优化：延迟分支  
    - 将与分支无关的指令转移到分支指令后面执行，填充延迟时间，不够用时再使用 nop 进行填充  
    - ![image.png](https://thdlrt.oss-cn-beijing.aliyuncs.com/20240101212514.png)



### 流水线控制器
**流水线控制器的实现**
- ID 段生成所有控制信号，并随指令执行过程信息同步向后续阶段流
- 与单周期处理器的控制器的实现方法一样，无需采用有限状态机


我们有一些流水线段寄存器，比如 `IF/ID`, `ID/EX`, `M/WB`, 来存储周期执行过程中的一些信息



- `M/WB`
	- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251219104114.png)
	- 



#### 1. IF/ID 寄存器（取指/译码间）

用于暂存从指令存储器中取出的原始数据，以便在下一周期进行解析。
- **指令内容 (Instruction)**：从存储器中读出的 32 位机器码。 111
- **PC+4**：当前指令的下一条地址，用于分支跳转计算或返回地址。 2

#### 2. ID/EX 寄存器（译码/执行间）

存放译码后产生的所有控制信号和操作数，为 ALU 运算做准备。

- **操作数 (Read Data)**：从寄存器堆中读出的 $Rs$ 和 $Rt$ 寄存器的数值。 333
    
- **符号扩展后的立即数 (Immediate)**：经过位扩展后的 32 位立即数。 4444
    
- **控制信号**：如 $ALUOp$（运算类型）、$RegDst$（目标寄存器选择）等。 5
    
- **寄存器索引**：记录目标寄存器的编号（$Rt$ 或 $Rd$）。
    

#### 3. EX/MEM 寄存器（执行/访存间）

存放运算结果和待写入存储器的数据。

- **ALU 运算结果**：作为内存地址或直接作为计算结果。 6
    
- **零标志位 (Zero Flag)**：用于分支指令判断是否发生跳转。 7
    
- **待写内存数据**：原本在 $Rt$ 寄存器中的值（用于 $Store$ 指令）。
    
- **控制信号**：如 $MemRead$、$MemWrite$、$RegWrite$ 等。
    

#### 4. MEM/WB 寄存器（访存/回写间）
存放最终准备写回寄存器堆的数据。
- **内存读取结果 (Read Data)**：从数据存储器中读出的数据（针对 $Load$ 指令）。 888
    
- **ALU 结果**：非访存指令的计算结果。
    
- **目标寄存器编号**：确定数据最终写回到哪个寄存器。 9
    
- **控制信号**：$RegWrite$（写使能）和 $MemtoReg$（写回数据来源选择）。

### 流水线各个阶段


- **第八周期**
	- 需要新的地址回送 PC，出现**反向数据流**


#### 延迟
- **数据冒险 (Data Hazard)**
- 描述数据冒险，**一定要说清楚指令之间**
- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251219111920.png)
- 第一条 `load` 指令在第 5 周期才能完成 `Write`，但这涉及到数据冒险（Data Hazard）
- `load` 的延迟效应。中间要延迟三个周期，才能在下一个 `R-type` 指令到来时，其取数取到正确的值
	- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251219112041.png)
[[DLCO-8 CPU#14|数据冒险习题]]


- **转移指令的延迟**
- **控制冒险**(Control Hazard)
	- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251219112119.png)
	- Branch: 我们在 M 阶段计算出是否需要跳转，但是下一个 Ifetch 不知道这个信息，可能取错！
	- **取错了几个指令**！
	- **分支延迟损失时间片**$C$
		- 我们通常把由于流水线阻塞而带来的延迟执行周期称为 ***延迟损失时间片* $C$***
	- 习题：[[DLCO-8 CPU#18 5 段流水线|控制冒险]]

#### 数据冒险的解决

- 硬件阻塞 (stall)
- 软件插入 `NOP` 指令
- 合理实现寄存器堆的读/写顺序 
	- 同一周期内寄存器先写后读
- **转发 (Forwarding** 或 **Bypassing 旁路)** 技术（不能解决所有的数据冒险）
- 编译优化，调整指令顺序
	- 编译优化思路可以看这个[[DLCO-8 CPU#17 调整序列-性能最优|调整指令序列-习题]]

1. **硬件阻塞**：stall, 插入气泡
	1. 比如：`add r1, r2, r3` -> `stall` -> `stall` -> `stall` -> `sub r4, r1, r3`


2. 合理实现寄存器堆的读/写顺序 
	1. 我们需要写口读口是独立的
	2. 同一周期内寄存器先写后读。**间隔两条以上的，可以避免冒险。**
3. 利用 DataPath 的中间数据：转发 (Forwarding)
	1. 我们使用了原有数据通路额外的通路，将数据提前转发给需要的指令。
	2. `EX/M` 寄存器记录了 `R-type` 指令的中间结果，可以开一个转发通路


![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251224143247.png)

#### 条件跳转冒险解决
- ***简单（静态）分支预测方法***
	- **基本做法**
		- 总预测条件不满足（not taken），即不跳转
			- 可加启发式规则
			- 在特定情况下总是预测满足 (taken)
		- 预测失败时，需把**流水线中三条错误预测指令（C=3）丢弃掉**
	- **预测**错误的检测和处理 (冲刷， -- Flush)
		- **如果原来预测不转移**，
		- 但是发现 Branch=1, 并且 Zero=1
			- beq 预测失败！
		- 此时需要完成以下两件事（延迟损失时间片 C=1 时）
			1. 将转移目标地址->PC
			2. 清除 IF 段取出的指令，即将 IF/ID 中的指令字清 0，转变为 NOP 指令
		- 预测错误，需要停止之前做的。用气泡代替


- ***动态预测的基本思想***
	- 利用**最近转移发生的情况**，来预测下一次可能转移还是 **不转移**
		- Dynamic Programming?
	- 根据实际情况来调整预测
	- 转移发生的历史情况记录在 BHT 中
		- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251224152638.png)
		- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251224153529.png)
		- **按预测**来决定下一条 IF 如何做。
			- **思想**：bne 会发生转移很多次，直到最后一次才会顺序执行下去。
			- **假设**我们初始设置为 `0`（0 是不跳），则第一次和最后一次都错。
		- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251224153937.png)
		- **注意内循环**，会从外层进来 $N$ 次，然后判断 $N+1$ 次要不要跳出。
			- 若初始预测为 0，则外循环只有最后一次预测错；跳出内循环时预测为 1，第一次总是预测错误，并且任何一次循环的最后一次总是预测错误，因此总共有 $1+2\times(N-1)$ 次预测错误
	- **两位预测状态图**
		- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251224154255.png)
		- 举个例子: Double-loop 2bit dynamic prediction
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251224154422.png)

- ***延迟分支***
	- **静态调度技术**。由编译程序重排指令顺序来实现
	- 基本思想
		- 把分支指令前面的与***分支指令***无关的指令调到分支指令后面执行，以填充延迟时间片（也称分支延迟槽 Branch Delay slot）
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251224154843.png)




## 提高性能措施
最理想的串行流水线，CPI=1

**实现指令流内部的并行流水线**：指令级并行（ILP）
- 两种指令级并行策略：
	- **超流水线**(Super-pipelining)
	- **多发射流水线**(Multiple issue pipelining
		- 如何多发射？

- **多发射**的两种实现方法
	- 静态多发射
		- **compiler** statically do the instruction package and risk deal
		- **指令打包**：将同时发射的多条指令合并到同一个长指令
			- 将同一个时钟周期发射的多个指令看成是一条多个操作的长指令，称为一个发射包
			- 静态多发射，aka **超长指令字**(VLIW - Very Long Instruction Word)
			- 同一个周期发射的指令类型受**硬件**限制
			- 
	- 动态多发射
		- Hardware dynamically do the instruction package in the runtime
		- **可以**一边执行，一边由硬件到换顺序
		- **三种执行模式**
			- 按序发射按序完成 (Pentium)
			- 按序发射无序完成 (Pentium II & Pentium III)
			- 无序发射无序完成 (Pentium IV)
				- 一般窗口是有限的。
				- **取指和译码按顺序进行**。发射前进行相关性检测，无关指令可先行发射和先行完成



