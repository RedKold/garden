教材第九章后习题中的第4、5、6、12、13、16题。



## 4
以下是在 IA-32+Linux 系统中执行的用户程序 P 的汇编代码
```asm
# hello.s
# display a string "Hello, world."

.section .rodata

msg:
.ascii "Hello, world. \n"

.section .text
.globl _start
_start:

movl	$4,	%eax		# syscall no. 4 (sys_write)
movl	$1,	%ebx		# file descriptor(param 1): stdout
movl	$msg,	%ecx	# string address (param 2): string to be display
movl 	$14,	%edx	# string length
int 	$0x80			# call kernel function

movl	$1,	%eax		# sys_exit
movl	$0,	%ebx		# param 1: exit the code
int		$0x80			# call kernel function

```

1. 程序的功能是什么：
	1. 向终端输出一行字符 `"Hello, world.\n"`
2. 执行到哪些指令时发生从用户态到内核态执行的情况？
	1. 两个 `int $0x80` 指令
3. 该用户程序调用了哪些系统调用？
	1. `sys_write`: 是系统调用 `write` 的封装，用于把字符写到 `stdout`
	2. `sys_exit` ：是系统调用 `exit` 的封装，用于退出系统进程，转移到用户进程继续执行

## 5
第 4 题中用户程序的功能可以用以下 C 语言代码实现
```c
int main(){
	
	write(1, "Hello world.\n", 14);
	exit(0);
}
```
回答下列问题
1. 执行 `write()` 函数时，传递给 `write()` 的实参在 `main` 的栈帧中存放情况怎样？要求画图说明
2. 从执行 `write()` 函数到开始调出 `write` 系统调用服务例程 `sys_write()` 执行的过程中，其函数调用关系是怎样的？要求画图说明
3. 就程序设计的便捷性和灵活性以及程序执行性能等方面，与第 4 题中的实现方式进行比较

### 栈帧
执行 `write` 函数，会把 `1`（file descriptor），`"Hello world.\n"` 和 `14`（string length）传参。
main 会把他们压栈。第一个参数放在 `%esp`
栈帧：
```
高位
^
|
+-----------------------+
|	14					|
+-----------------------+	-> %ebp+8 0xf
|	"Hello world.\n"	|
+-----------------------+	-> %ebp + 0xc
|	1					|
+-----------------------+	-> %ebp + 0x8
|	return address		|
+-----------------------+	-> %ebp + 0x4
```

### 函数调用关系分析
执行 `write()` 函数，
![3892abc42312c4e3ba387a6dedcc6cd9_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/3892abc42312c4e3ba387a6dedcc6cd9_720.png)
笔记中 [[ICS#ics - section 9 I/O]]

### 就程序设计的便捷性和灵活性以及程序执行性能等方面，与第 4 题中的实现方式进行比较
第四题采用汇编指令实现，更麻烦，不方便书写，而且直接对底层操作，容易出现安全应还。不灵活
本题采用封装后的系统调用函数实现，更便捷灵活。

从执行性能来看，第 4 题是直接执行，本题封装的 IO 函数会有额外的开销，性能会差一些。
## 6
第 4 题和第 5 题的用户程序的功能可以用如下的 C 语言代码实现
```c
#include<stdio.h>
int main(){
	printf("Hello, world.\n");
	exit(0);
}
```

假定源程序名为 `hello.c` ，可重定位目标文件名为 `hello.o`，可执行目标文件名为 `hello`，程序用 GCC 编译驱动程序处理，在 IA-32+Linux 系统中执行。回答如下问题


### 为什么在 hello. c 的开头要加 `#include<stdio.h>`? 为什么 hello. c 中没有定义 `printf()` 函数，也没有它的原型声明，但 `main()` 函数引用它没有发生错误？

因为 `hello.c` 使用了 `stdio.h` 中的函数 `printf` .
在包含该头文件之后，预处理阶段编译器会把 `stdio.h` 中的所有内容“粘贴到”本代码编译单元。
而 `stdio.h` 中有一行 `extern int printf (const char *__restrict __format, ...);`，所以不会发生错误（编译器知道这是个外部的函数名，可以通过符号解析、链接引入）

### 需要经过哪些步骤才能在机器上执行 `hello` 程序？要求详细说明各个环节的处理过程

- **预处理**
	- 编译器展开宏，把 `#include` 包含的头文件展开到对应编译单元的位置, 去掉注释
- **编译**
	- 编译器编译各个 unit，形成汇编语言程序
- **汇编**
	- 将汇编文件转化为可重定向的机器语言文件 (`.o`)
- **链接**
	- 将多个 `.o` 文件链接，形成可执行文件

### 为什么 `printf()` 函数没有指定字符串的输出目的地，但执行 `hello` 程序后会在屏幕上显示字符串？
因为 `printf` 的默认输出地是 `stdout`，他在调用 `write` 函数的时候第一个参数 `fd=1`，即 `stdout`。而最终显示屏幕是经由 `write` 调用底层系统函数 `sys_write()` 的，故可以在屏幕上显示字符串

### 字符串 `"Hello, world.\n"` 在机器中对应的 0/1 序列（机器码）是什么？这个 0/1 序列存放在 `hello.o` 文件中的哪个节中？这个 0/1 序列在可执行文件 hello 的哪个段中？
0/1 序列：机器码如下
// 课本看不太出中间有没有空格，我按有空格算
`48 65 6c 6c 6f 2c 20 77 6f 72 6c 64 2e 0a 00`
`H  e  l  l  o  ,  SP w  o  r  l  d  .  \n  0`

放在 `hello.o` 的 `.rodata` 节

在 `.data` 段 
[[ICS - Section 4 程序的机器级表示#内存：x86-64 Linux Memory Layout|Linux Memory Layout]]
- data segment: global vars, `static` vars, **string constants**

### 若采用静态链接，则需要用到 `printf.o` 模块来解析 `hello.o` 中的外部引用符号 `printf`，`printf.o` 模块在哪个静态库中？静态链接后, `printf.o` 的代码部分 (. text section) 被映射到虚拟地址空间的哪个段中？若采用动态链接，则 `printf()` 函数的代码在虚拟地址空间的何处

`printf.o` 在静态库中，即 `libc.so`
静态链接后，`printf.o` 的代码部分被映射到虚拟地址空间的 `text` 段即只读代码段，因为静态链接我们会把 `printf` 运行的所有依赖都拷贝到虚拟地址空间的代码部分.

如果采用动态链接，则 `printf()` 的代码在虚拟地址空间的**共享库映射区域（Shared Libraries / Memory Mapping Segment）**。我们通过 `GOT`（Global Offset Table）来动态的运行 `printf`

[[ICS - Section 5 程序的链接和加载执行]]
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251226211919.png)

- Lazy Binding:
- 延迟到第一次执行来动态绑定
- 需要用GOT和PLT（Procedure linkage Table, 过程链接表）
## 12 打印机
某台打印机每分钟最快打印 6 个页面，页面规格为 50 行 x 80 字符。已知某台计算机的主频为 500MHz，若采用中断方式进行字符打印，则每个字符申请一次中断且中断响应和中断处理时间合起来为 1000 个时钟周期。请问该计算机系统能否采用中断控制 I/O 方式进行字符打印输出？为什么？
主频 500MHz，1000 个时钟周期需要 `1000 / 500MHz = 2us`

中断方式打印：每分钟打印 `6 * 50 * 80 = 24000` 个字符，需要 `2us * 24000 = 480ms`  < `1 min`

该计算机系统能采用中断方式进行字符打印输出，且负载很小。
## 13
假定某计算机系统的 CPU 主频为 `500 MHz`，所链接的某个外设的最大数据传输速率为 20KB/s，该外设接口中有一个 16 位的数据缓存器，相应的中断服务程序的执行时间为 500 个时钟周期，是否可以用中断控制 I/O 方式进行该外设的输入/输出？假定该外设的最大数据传输速率为 2MB/s，是否可以用中断控制 I/O 方式进行该外设的输入/输出？
16bit 即 2B
20KB/s, 16bit，每秒需要中断 `20KB / 16bit = 20 * 2^10 / 2 = 20 * 2 ^ 9 = 10240` 次
中断时间：`10240 * 500 /500MHz =10.24 ms`
20KB，可以满足。

如果是 2MB/s，则中断时间为 `1024ms` > `1s=1000ms`，且 CPU 还有其他开销，则这时候不能用中断方式进行 IO 了。



## 16 DMA
假设某计算机系统的所有指令都在两个总线周期内完成，一个总线周期用来取指令，另一个总线周期用来存取数据。总线周期为 `250ns`，因而每条指令的执行时间为 `500ns`。若该计算机中配置的磁盘上每个磁道有 16 个 512 字节的扇区，磁盘旋转一圈的时间是 8.192ms，总线宽度 16 位，采用 DMA 控制 I/O 方式传送磁盘数据，则在进行 DMA 传送时该计算机指令执行速度降低了百分之几？


每个磁道的数据量：`16 * 512 = 2^4 * 2^9 = 8192 B`
需要的 DMA 传输次数（总线宽度 `16bit = 2B`）: `8192 / 2B = 4096 times`
单位 DMA 传输占用的时间：`8.192ms / 4096 / 0.002 ms = 2us = 2000ns`

以 `2000ns` **为单位考虑**：
- 本来可以执行 4 条指令, 8 个总线周期 
- 现在每隔 2000ns 要被 DMA 占用一次
- 即只能执行 7 个总线周期给指令，1 个给 DMA

降低了：
`(8-7)/8 * 100% = 12.5%`

> this ends the ics-9 assignment

