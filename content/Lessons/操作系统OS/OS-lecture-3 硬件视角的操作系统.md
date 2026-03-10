
- Use your agent
- "intelligence is cheap"
- 一个改变世界的 prompt：
	- I'm doing xxx, if you are an expert, what would you do ...
	

- Review:
# Aside: Before the Lesson
## 软件：C (SimpleC), Python, …

- 初始状态：`stack=[main(argc, argv), globals]`
- 迁移：执行一条语句

## 硬件：x86, RISC-V, ARM…

- 初始状态：CPU Reset
- 迁移：执行一条指令/响应中断

## 操作系统：使软件能运行在硬件上的程序

- 系统调用 API (syscall/ecall/svc 指令)，将控制权交给操作系统

# 什么是计算机硬件

==硬件不知道有没有操作系统==

- Hardware run the instruction one by one， pure state machine

- **计算机是一个抽象层**
	- 隔离复杂性的根本手段
	- `OS -> printf -> (syscall)write`
- How to design an abstract layer is an important capacity in the generation of *Agentic AI*
	- Plugin?
	


## 计算机系统的状态机模型

## 状态
- 内存、寄存器的数值
## 初始状态
- 由系统设计者规定（CPU Reset）
## 状态转移
- 从 PC 取指令执行
## 准确，但忽略了一个重要的细节
- "`jmp .`" will forever loop?
	- 有一个物理的线，可以打破状态机模型的死局
- Some details make OS can be implemented

```c
struct CPUState { 
	uint32_t regs[32], csrs[CSR_COUNT];
	uint8_t *mem;
	uint32_t mem_offset, mem_size; 
};
```


**我们还有外部的状态**： 世界へ
- 设备上的寄存器 (memory-mapped I/O 可以访问)
	- 对外界影响的主要方式

- Example

## 计算机系统：初始状态
`CPU Reset` 是一个明确的行为
- 老旧电脑配一个 `Reset` 按钮

```asm
.globl _start _start: 
# SBI system shutdown (sbi_ext_base = 0x10, sbi_ext_base_shutdown = 0x08) 
# a7 = SBI extension ID (0x10 for base)
# a6 = function ID (0x08 for shutdown) 
li a7, 0x10 
li a6, 0x08 
j go 
go: ecall 
# Should never reach here 
1: wfi j 1b
```


- **执行指令**
	- 如果有多个处理器？
		- 可以想象成“每次选*一个处理器*执行一条指令”
		- 在并发部分回到这个问题
- **响应中断**
	- `if(intr) goto vec;`
- 输入输出
	- 与“计算机系统外”交换数据
		- 读取电信号（input）或把电信号传递给设备（output）

操作系统在内存中，但是只有在系统调用、中断发生的时候运行自己，是一个 **服务的提供商**

# 固件：硬件和操作系统之间的桥梁

CPU 从 CPU Reset 开始执行，从 `Mem[PC]` 取指令开始译码、执行，如此往复。这里必须要是合法的代码
- 是系统厂商的代码
-  把一个特殊的存储器（只读的）memory-map -> CPU Reset 后的代码
	- 合法的代码，有完整的机器控制权
	- Called `Firmware`

## Firmware （固件）
- 厂商“固定”在计算机系统中的代码
	- 早期：固件是 ROM
	- 升级只能换芯片（今天可以直接升级了）
		- 耳机？词典笔？手环？你并不陌生固件升级
## Firmware 的功能
- 完成硬件扫描、初始化和配置
    - (这些配置要生效可能需要重启计算机)
- 不严格地说，**加载操作系统**
    - QEMU：可以绕过 Firmware 直接加载操作系统 ([Manual](https://www.qemu.org/docs/master/system/invocation.html#hxtool-8))


- 配置计算机系统
- 按一个特殊的组合键，进入 BIOS![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260310112235.png)
- 直接看到了系统硬件

## Firmware： 就是一段代码
- A Small "OS"
	- After CPU Reset， we initialize the hardware； and dock to OS's Boot Loader
- Legacy BIOS (Basic I/O System) 
	- IBM PC
		- 16 bit DOS, BIOS is *always* in memory, provide I/O etc.
- UEFI (Unified Extensible Firmware Interface)
	- More support, as device driver-
	- 指纹锁
	- USB 蓝牙转接器连接的蓝牙键盘

## 小插曲-梦回 1998
Firmware is usually read-only (of-course)

**But Firmware** is also need to update
- Intel 430TX (Pentium): 允许写入（更新）PROM
	- 主板提供写保护的跳线
- 防止 Bug 损坏 Firmware
	- 只要向 BIOS 写入特定序列，写保护就可以打开
	- 序列就在手册里... 😊


- **病毒**成为了可能

- 如何预防病毒破坏固件？
	- 炫技搞破坏的意义越来越小
		- Safer OS， AppStore， Cloud backup
	- 获取权限的意义越来越大
		- 偷取隐私、LLM token、移动支付 

# 从固件到操作系统
# Back to 43 years ago

## BM PC/PC-DOS 2.0 (1983)

- Firmware (BIOS) 会加载磁盘的前 512 字节到 0x7c00
    - (如果这 512 字节最后是 0x55, 0xAA)
    - 为什么是 0b01010101 和 0b10101010？

## 让我们试一试

`(printf "\xeb\xfe"; cat /dev/zero | head -c 508; printf "\x55\xaa") > a.img`

- 还记得 eb 是什么吗？

如果 Firmware 也是代码，则 `0x7c00` 附近的内容是被 Firmware 的代码扫描磁盘、加载了它

#TODO Try QEMU to run this code by yourself
