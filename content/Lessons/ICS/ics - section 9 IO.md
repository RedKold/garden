## ics - section 9 I/O

### `stdio.h` 
系统调用开销很大。
- 系统级别 I/O 函数对文件的标识是**文件描述符**，C 标准 I/O 库函数对文件的标识是 **指向 `FILE*`** 结构的指针
- FILE 中定义了 1024 字节的缓存区
	- `stdio`: store in  ` buf ` then output
	- `stderr`: no buffer, directly **out**;
- `_fillbuf()` 一个在输出前填充缓冲区的函数

我们最终是通过 `syscall` 的 `write` 和 `read` 来实现的。
回忆一下 PA [[ics-pa3-report#必答题(需要在实验报告中回答) - hello程序是什么, 它从而何来, 要到哪里去|hello程序发生了什么]]


![b20008c96653e09f01d1a8bd9d15d8fb_720.png|500](https://kold.oss-cn-shanghai.aliyuncs.com/b20008c96653e09f01d1a8bd9d15d8fb_720.png)



**内核空间**
- **三种方式**
	- CPU 直接控制
		- **轮询状态**
		- 100%时间，监督状态
		- 巨大的开销
	- 中断方式
- **会算效率**


- **查询方式和中断方式的比较**
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251222091839.png)
	- OBR： 数据缓冲器
	- CPU
		- do I/O instructions to send data to OBR
	- I/O
		- automatically send data to device
---


- **DMA 方式**
- DMA 方式下 **CPU** 的工作
- `sys_write`  string output 的 program segment
```
copy_string_to_kernel(strbuf, kernelbuf, n); // copy str to kernel buf
initialize_DMA();		// initialize the DMA controller ( ready to transfer param)
*DMA_control_port=START;	// send "START DMA transfer" command
scheduler();			// block user process P, schedule to other process to execute
```
- DMA 结束“中断服务程序”

```
acknowledge_interrupt();	
unblock_user();
return_from_interrupt();
```



- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251222092501.png)
	- 在高速外设和主存间**直接传送**数据
	- 由专门硬件（即：DMA 控制器）控制总线进行传输
- DMA 方式适合场合
	- 告诉设备
	- 成批数据交换，且数据间间隔时间短，**一旦启动，数据连续读写**
	- [[ics-9#16|DMA计算]]


主机-北桥- **I/O**总线---南桥（带连接器设备控制器）---电缆---外设

**显卡**：
- 现在的显卡已经有自己的操作系统、指令集了，很进步
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251222102440.png)
- `typec` 是一个插座，不是设备控制器（I/O）接口，不是一个协议
	- 就是一个 `24引脚` 的插座
	- 什么是控制器接口？e.g USB3.0

- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251222102859.png)

- **IO 端口**的寻址方式
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251222104144.png)
	- RISC-V 统一编址
		- PA 设计。
	- IA-32 独立编址

- **多重中断**
	- 中断优先级没考
- 多重中断：
	- 被新进来的中断中断掉
- **默认**：linux 不支持多重中断


 - **重点了解**
	 - [hello 程序运行过程综述](https://ysyx.oscc.cc/slides/hello-x86.html)


---