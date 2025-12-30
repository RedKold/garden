Chapter 8 Assignment:
3、4、5、6、11、13、14、17、18

### 3 
计算机字长 16bit， Flag 中的 ZF, SF, OF。双字节定长指令字。假定有一条 Bgt 指令，指令格式如下：第一个字节指明操作码和寻址方式，第二个字节为偏移地址 Imm8。功能：若 `ZF + (SF^OF)==0` 则 `PC=PC+2+imm8`，否则 `PC=PC+2`, 回答：


1. 该计算机的编址单位是多少位
字长 16bit，计算机一次取一条指令为 2 字节（PC=PC+2），每个字节 8bit，按字节编址。所以编址单位是 8bit，一个字节

2. 画出实现 Bgt 指令的数据通路
![[Drawing 2025-12-21 11.49.42.excalidraw]]


### 4
![df8d6192618c2b1c1d6fec6512a2461e_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/df8d6192618c2b1c1d6fec6512a2461e_720.png)
假定图给出的单周期数据通路的控制逻辑发生错误，使得控制信号 `RegWr, ALUASrc, Branch, Jump, MemWr, MemtoReg` 中的某一个在任何情况下总是 0，则该控制信号为 0 时哪些指令不能正确执行？
- `RegWr`：
	- 若 `RegWr=0`，则发生寄存器写入的指令不能执行，如运算类 `I-type`，`R-type` 和一些 `load` 指令
- `ALUASrc`
	- 若 `ALUASrc=0`, 则一些 `B-type` 指令和 `J` 指令可能无法执行。因为无法将 PC 的值送入 ALU
- `Branch`
	- 如果 `Branch=0`，则分支类指令 `B-type` 可能无法运行
- `Jump`
	- 如果 `Jump=0`，则 `J-type` 指令可能无法运行
- `MemWr`
	- 如果 `MemWr=0`，则无法向内存写入数据，则 `sw, sb, sh` 等 store 指令可能无法运行
- `MemtoReg`
	- 如果 `MemtoReg=0`, 则 `load` 指令发生错误，因为无法读内存数据写入寄存器。


### 5
[[#4]] 的**镜像问题**：

假定图给出的单周期数据通路的控制逻辑发生错误，使得控制信号 `RegWr, ALUASrc, Branch, Jump, MemWr, MemtoReg` 中的某一个在任何情况下总是 1，则该控制信号为 1 时哪些指令不能正确执行？

- `RegWr`
	- if `RegWr` is always 1, then all instructions that don't need to write to the GPRs will error. such as `B-type`, `store-type`
- `ALUASrc`
	- if `ALUASrc` is always 1, then in ALU, the first operand is always the `PC`, then I-Arithmetic type instruction, R type and all instructions that may use **ALU** would error
- `Branch`
	- if `Branch` is always 1, then all instructions except `B-type` may error. (Unnecessary jump)
- `Jump`
	- if `Jump` is always 1, then all instruction except `J-type` may error. (Unnecessary jump)
- `MemWr`
	- if `MemWr` is always 1, then all instruction except `store type` may error, because you may store unnecessary data into memory and trigger bugs hard to understand
- `MemtoReg`
	- if `MemtoReg` is always 1, then all instruction except `load` may error, because you may load unnecessary data to register from memory, and lead to weird bugs
### 6 实现 RV32I swap
若要在 RV32I 指令集中增加一个 swap 指令，（实现两个寄存器功能的互换），可以用两种方式：
- 伪指令方式（即软件）
	- 用若干条已有的指令序列交换
- 直接改动硬件来实现 `swap` 指令
讨论一下问题：

1. 写出用伪指令方式实现 `swap rs, rt` 时的指令序列（提示，伪指令对应的指令序列中不能使用额外寄存器）
回忆一下：c 语言怎么不利用额外空间实现交换？利用异或！
```cpp
void swap(int&x, int&y){
	x=x^y;
	y=x^y; 	// y= x^y^y =x;
	x=x^y	// x= x^y^x	=y;
}
```


翻译成汇编指令。
```asm
# x: rt, y: rs
xor rt, rs, rt
xor rs, rs, rt
xor rt, rs, rt
```

注意 RISC-V 指令，第一个参数是 Destination Reg

2. 加入硬件实现 swap 会使得每条指令的执行时间增加 10%，则 swap 指令在程序中占多大比例才值得用硬件而不是用 1 中的方式 ？
$$
\alpha \times W\times 3+ (1-\alpha)\times W \geq 1.1\times W
$$
列一个方程，`W` 代表总指令执行时间，`alpha` 代表比例。
解方程：
$\alpha\geq 5 \%$

3. 采用硬件实现，不对 GPRs 进行修改的情况下，能否在单周期数据通路完成 swap？多周期呢？
单周期无法实现，因为单周期无法同时完成两个 Reg 数的修改。
多周期可以。分为两个时钟周期完成。


### 11 流水线 Pipeline
![](https://kold.oss-cn-shanghai.aliyuncs.com/Pipeline%20datapath.jpg)
假定在一个 5 级流水线处理器中，各主要功能单元的操作时间为：存储单元，200 ps; ALU 和加法器, 150ps; 通用寄存器的读口或写口，50ps。请问：
1. 若 EX 阶段所用的 ALU 操作时间缩短 20%，能否加快流水线执行速度？如果能，加快多少？不能，为什么？
不能。因为缩短 20%之后，仍然慢于 `200ps` 的存储单元。而流水线执行速度取决于最慢的单元
2. ALU 操作时间增加 20%，有影响吗？
	没有影响。`150 * 1.2 = 180 < 200`
3. ALU 操作时间增加 40%，有何影响？
	`150 * 1.4 = 210 > 200` 有影响。流水线慢了 10ps，降低了效率 `(210-200)/200` =5%
### 13
假定最复杂的一条指令所用的组合逻辑分为 6 个部分，依次为 A～F。延迟分别为 
`A,   B,   C,    D,    E,    F`
`80ps,30ps,60ps, 50ps, 70ps, 10ps`.
在这些组合逻辑块插入必要的流水段寄存器就可以实现相应的指令流水线，寄存器延迟为 20ps。
理想情况下，以下各种方式所得到的时钟周期、指令吞吐率和指令执行时间各是多少？应该在哪里插入流水段寄存器？
1. 插入一个，得到两级流水线
	插入在 C-D 之间。ABC=170， DEF=130. 加上寄存器延迟 20，时钟周期为 190ps，吞吐率为 `1/190ps = 5.26 G条指令/s` 指令执行时间 `2*190 =380 ps`
2. 插入两个
	B-C, D-E
	AB=110, CD=110, EF =80
	时钟周期 `max(AB,CD,EF)+20` =130, 
	吞吐率 `1/130ps = 7.69 G条指令/s`
	指令执行时间：`130 * 3 = 390 ps`
3. 插入三个
	A-B, C-D, D-E
	A=80, BC=90, D=50, EF=80
	时钟周期 `max(A, BC, D, EF)+20 = 110`
	吞吐率 `1/110ps = 9.09 G条指令`
	 指令执行时间：`110*4=440ps`
4. 吞吐量最大的流水线：
	最长 delay 为 80ps，所以最大吞吐量应该使得时钟延迟达到 `100ps`
	划分：插入五个
	A-B, B-C, C-D, D-E, 
	A=80ps, B=30ps, C=60ps, D=50ps, EF=80ps
	时钟周期 `100ps`
	吞吐量：`1/100ps = 10G条指令`
	指令执行时间：`100ps * 5 = 500ps`
### 14
以下指令序列中，哪些指令对之间发生数据相关？假定采用 IF, ID, EX, M, WB 5 段流水线方式，如果不用转发技术，需要在发生数据相关的指令前加入几条 nop 指令才能避免数据冒险？如果转发，是否可以完全解决数据冒险？不行的话，需要在发生数据相关的指令前加入几条 nop 指令才能使这段 RV32I 程序不发生数据冒险？
```asm
1 add	s3,	s1,	s0
2 add	t2, s3, s3
3 lw	t1, 0(t2)
4 add t3,	t1, t2
```

inst 1: `R[s3] = R[s1] + R[s0]`
而 inst 2 需要取出 s3 的值。但是 1 还没有完成写入：发生数据相关
inst 3 要 load `0(t2)`，但是 t2 在 inst 2 add 中还没有完成 WB，发生数据相关
inst 4 需要访问寄存器 t1 和 t2，但是 lw 还没有完成 WB，发生数据相关, 且 inst 2 也没有完成 WB，发生数据相关

总结：
`(1,2), (2,3), (3,4), (2,4)`
- `(1,2)`
```
# old
1		IF	ID	EX	M	WB
2			IF	ID	EX	M	WB

# new
1		IF	ID	EX	M	WB
nop
nop
nop
2						IF	ID	EX	M	WB

```
- 需要在 2 前面加三条 nop，即可使得不发生冒险。`(2,3)` 和 `(3,4)`, `(2,4)` 同理
- 如果考虑 WB 的前半周期写，调整触发沿，让 ID 可以在 WB 的后半周期读取，则可减少到两个 `nop`

如果采用转发，我们可以在 EX 阶段通过额外的数据通路直接发送 ID 阶段取出的值。比如送到 EX 阶段的 ALU

则（1，2）的数据冒险可以解决，(2,3) 的可以解决，（2，4）的可以解决。因为它们都是利用 `ALU` 计算，只要 ALU 正确接收到即可。而它们 EX 阶段，前面依赖的指令已经经历过 EX 阶段了，可以把寄存器的值送过来

但是 `(3,4)` 无法解决，因为 `（3，4）` 是一个 `load-use` 的数据冒险，内存中没有完成 WB，是无法通过转发解决问题的。

```
3		IF	ID	EX	M	WB -> write back
4			IF	ID	EX	M	WB
						|	
						+-> need data!	but 3 not finish write back!
# solution: insert a nop
3		IF	ID	EX	M	WB -> write back
nop
4				IF	ID	EX	M	WB
							|	
							+-> need data, and 3 write back done, OK!
```

为了解决这个问题，可以插入一个 nop
### 17 调整序列-性能最优

假定在一个带转发的 5 段流水线中执行以下RV32I 程序段，应该怎么样调整指令序列以使其性能最优？
```asm
1 lw	s2,	100(s6)
2 add	s2,	s2,	s3
3 lw	s3,	200(s7)
4 add	s6,	s4,	s7
5 sub	s3,	s4,	s6
6 lw	s2,	300(s8)
7 beq	s2,	s8	Loop
```

采用转发，只存在 load-use 数据冒险
(1,2) 存在 load-use 数据冒险
(3,5) 存在 (load-use) 数据冒险

所以我们可以把和 12 无关的 3 放到 1 和 2 之间执行。
把和 35 无关的 6 放到 45 之间执行
```asm
	1 lw	s2,	100(s6)
++	3 lw	s3,	200(s7)
	2 add	s2,	s2,	s3
--	
	4 add	s6,	s4,	s7
++	6 lw	s2,	300(s8)
	5 sub	s3,	s4,	s6
--
	7 beq	s2,	s8	Loop
```
### 18 5 段流水线
在一个采用 `IF, ID, EX, M, WB` 的 5 段流水线中，若检测相减结果是否为 0 的操作在执行阶段进行，则分支延迟损失时间片（即分支延迟槽）为多少？以下一段 RV32I 指令序列中，在考虑数据转发的情况下，哪些指令执行时会发生流水线阻塞？各需要阻塞几个时钟周期？

```asm
loop:	add	t1,	s3,	s3
		add	t1, t1, t1
		add	t1, t1, s6
		add	t0,	0(t1)
		bne	t0,	s5,	Exit
		add	s3,	s3,	s4
		j	Loop
Exit:
```



- ![image.png|600](https://kold.oss-cn-shanghai.aliyuncs.com/20251219111920.png)

**分支延迟损失时间片**：
- [[DLCO-0 数字逻辑与计算机组成#流水线的三种冒险]]
- 如果 EX 阶段计算出了是否需要跳转，而根据 PC 取指令发生在 IF 阶段，和下一条指令至少需要阻塞 2 个。
- 则分支延迟损失时间片为 `2`
---
**阻塞**
- `(4,5)`
	第 4 条和第 5 条指令之间有 load-use 冒险
	
	`bne` 需要判断 t0 和 s5 是否不 equal
	t0 在 `add t0 0(t1)` 计算, **发生了数据冒险**，需要流水线阻塞
	
	如果采用转发
	```
	add		IF	ID	EX	M	WB
						|
						+-> 可以开始forwarding data: t0			
	bne			IF	ID	EX	M	WB
						|
						+-> 需要 t0参与ALU计算
						
	# insert a nop
	add		IF	ID	EX	M	WB
						|
				+-------+-> 可以开始forwarding data: t0			
	bne			|	IF	ID	EX	M	WB
				+-----------+
							+-> 需要 t0参与ALU计算,可以拿到数据
	```
	
	阻塞一个时钟周期即可。

- `(7)` J
**发生跳转**。

J 是无条件跳转，所以获得 `imm`（在 ID）之后都可以知道跳转到哪里，关键是硬件实现决定在什么阶段更新 PC
- 如果 `ID` 阶段决定跳转（并转发），则阻塞 1
- 如果 `EX` 阶段决定跳转，阻塞 2

this ends the DLCO-7


> 永远进步
