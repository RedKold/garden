# Review
**Virtualization & Concurrency**
- 进程和系统调用上的应用世界
- `spawn(T_worker)`：魔鬼的盒子
	- 并行计算、异步编程、异构计算

**操作系统中的对象**
- 文件描述符：**指向操作系统对象的“指针”**
	- "Everything is a file"
	- 通过指针可以访问“一切”
- 我们直接选择了操作系统对象 (tty, disk,...)
	- **正式展开**


# I/O 设备
- 我们实际上“看到”**的计算机**
- 我们讨论了很多 CPU、内存都是看不见的

## I/O 设备：“计算”和“物理世界”之间的桥梁
> [!Note] I/O 设备
> =一个能与 CPU 交换数据的接口/控制器
> 

- 使计算机能**感知外部状态**，对外实施动作
	- 眼睛、耳朵、收
- “**几组约定好功能的线**”（寄存器）
	- 通过握手信号从线上读出/写入数据
- 给寄存器赋予一个内存地址 (Address Decoder)
	- "Everything is a file", [[ICS-PA2 note]]
	- CPU 可以直接使用指令（In/out/MMIO）和设备交换数据
	- 也可以向计算机发送阶段（这部分实现在操作系统内部）

## 一些简单的设备
**实现核弹发射**：也是一根线I/O 设备

```
[CPU] --|mem+addr|---O/I---> Your device
```

> [!Note] GPIO (General Purpose Input/Output)
> - 极简的模型：Memory-mapped I/O 直接读取/写入电平信号 `Logic 1 = 3.3V`
> - 我们甚至可以直接控制 Raspberry Pi 上的灯


### 串口 (UART)

#### "COM1"（Communication 1）
- Universal Asynchronous Receiver/Transmitter
- 映射到 **UART**（可以接真正的终端使用）

```c
#define COM1 0x3f8
static int uart_init() {
  outb(COM1 + 2, 0);   // 控制器相关细节
  outb(COM1 + 3, 0x80);
  outb(COM1 + 0, 115200 / 9600);
  ...
}
static void uart_tx(AM_UART_TX_T *send) {
  outb(COM1, send->data); // 写寄存器
}
static void uart_rx(AM_UART_RX_T *recv) {
  recv->data = (inb(COM1 + 5) & 0x1) ? inb(COM1) : -1;  // 读寄存器
}
```

### 键盘控制器
IBM PC/AT 8042 PS/2 (Keyboard) Controller
- **老式接线** PS/2
	- 6 根线
	- Data, Clock, VCC, GND, 两个预留
	- 映射到 Port 0x60 (data), 0x64 (status/command)
		- command = 0xED ->LED 灯控
		- command = 0xF# -> 设置重复速度和重复延迟
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260521194315.png)

- I/O 设备没什么了不起的，只是个寄存器和线.


### 磁盘控制器
**ATA(Advanced Technology Attachment)**
- IDE 接口磁盘 (40pin data+4pin 电源)
	- primary: 0x1f0 - 0x1f7; secondary: 0x170 - 0x177

## 有趣的设备

### 打印机
任何能 input/output 的东西，都可以是 Canonical Device
- 打印机：将字节流描述的文字/图形打印到纸张上

- 一个简单的想法

### PostScript 和打印机
> [!Note] PostScript: 一种描述页面布局的 DSL
> - 类似于汇编语言（由编译器，如 latex，生成）
> 	- PDF 是 PostScript 的“改进版”
> 	- “图形状态机”：路径构造、路径着色、文本渲染、外部对象

> [!Note] 打印机理解
> 一个**汇编语言**解释器
> - PCL, PostScript, IPP (AirPrint) -> 打印机机械动作

```
<ESC>*t300R          // PCL: Set resolution to 300 DPI
<ESC>*r1A            // Start raster graphics
<ESC>*b100W          // Set width of raster data (100 bytes)
<ESC>*b0M            // Set compression mode (0 = uncompressed)
<ESC>*b100V          // Send 100 bytes of raster data
<binary raster data> // Actual image data
```



### 摄像头（WebCam）
**你买到的所有摄像头都是“免驱”的**
- 感谢 UVC (USB Video Class)
- **标准化的协议**：枚举设备->协商格式/分辨率/帧率->传输开始->逐帧读取 MJPEG/H.264
	- 仍然是交换字节流（canonical device model）
	- **大量的衍生产品**



## 协处理器和总线
> [!Note] “独立于计算机”的设备之外
> 我的 Device 能不能和 CPU 接到一个内存上？
> 如果允许设备访问**计算机内的资源**？

**DMA**：一个专门执行 `memcpy` 的协处理器
- 共享 CPU 内存，也共享 I/O 端口
	- 则 DMA 相当于一个协处理器。只能执行 hard-wired 程序
- 早期的“多处理器系统”
- 可以和 I/O 端口通信，大大减轻了 CPU 的压力


## 异构加速器：GPU 和 NPU
如果 I/O 设备可以直接访问内存？-**更有趣的事情**

GPU 看起来就是“一台完整的计算机”
- 可以执行 `<<<>>>` 中定义的 kernel
- 代码和数据由 CPU 送过去的 (`cudaMalloc`, `cudaMemcpy`)
- **送数据**是 DMA 完成的
- 控制指令则是 canonical device model
```c
void T_worker(dim3 threadIdx, dim3 blockIdx, dim3 gridDim, dim3 blockDim){
	...
}
```


## 连接万千设备：总线
计算机硬件生态的“扩展性”：无穷无尽的 I/O 设备
> [!Note] **总线**：提供设备的注册和转发
> - 把收到的地址（总线地址）和数据转发到相应的设备上
> - **例子**：port I/O 的端口就是总线上的地址
> 	- IBM PC 的 CPU 其实只有这一个 I/O 设备

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260521201158.png)
- jyy 完成了显卡和总线的链接
	- 寄存器对齐了

**用的更多的**：USB 总线

## PCIe and CXL
获得 CPU **直连**特殊待遇的特殊设备
- PC: "PCIe lanes"，CPU 负责内存一致性
- ARM SoC: 内存、PCIe 都挂在告诉总线上
高速设备都是直插 PCIe 上的：
- FPGA
- 显卡
- 网卡
- `NVMe`
	- USB Bridge
	**更进一步，基于 PCIe 物理层的 CXL**（Computer Express Link）
	- CXL. io, CXL. cache, CXL. memory
	- CXL 3.0 允许设备直接共享内存
		- **甚至另一个计算机上的内存**
		- Memory hierarchy 发生了变化

# 程序视角的 I/O 设备
我们理论上不应该让程序可以直接访问总线/设备寄存器
- 一个程序可以，就有很多程序可以
- **并发**的梦魇？
- 访问->共享；共享->同步; 同步->bug
	- 完犊子
- CPU 和内存也是 I/O 设备啊！这启示我们回到**虚拟化**

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260521202252.png)

## Everything is a File
**File = 实现了文件操作的[Anything]**

```c
struct file_operations {
    struct module *owner;
    loff_t (*llseek) (struct file *, loff_t, int);
    ssize_t (*read) (struct file *, char __user *, size_t, loff_t *);
    ssize_t (*write) (struct file *, const char __user *, size_t, loff_t *);
```

- **一个巨大的结构体**，定义了设备虚拟化之后的接口
	- 无法虚拟化的设备，就提供设备寄存器的访问
- 可以“二次开发”让设备更好用
	- 把系统调用“翻译成”**设备能听懂的数据**
	- 一段普通的 linux 内核代码...
	- `mmap`, `read`, `write`

## ioctl - I/O control
**设备不仅仅是数据，还有配置**
- 打印机的卡纸、清洁、自动装订......
- 键盘的跑马灯、重复速度、宏编程
- 磁盘的健康状况、缓存控制......

**操作系统必须提供设备相关的接口**
> The ioctl() system call manipulates the underlying device parameters of special files. In particular, many operating characteristics of character special files (e.g., terminals) may be controlled with ioctl() requests. The argument fd must be an open file descriptor.


- 设备的复杂度无法降低
	- 功能就那么多
	- hidden specification

**一整个虚拟化的子系统**

# 终于，理解了“操作系统的对象”
**程序是如何访问它的**

**更多的例子**
- GPU=一个有自己内存的协处理器
	- 通过 `mmap` 把内存交给设备驱动/设备
	- 通过 `ioctl` 提交命令 (doorbell)
- Linux DRM (Direct Rendering Manager) 模块
	- libdrm: 用户态库，封装 ioctl
	- Mesa：用户态 OpenGL/Vulkan 实现

---
> 学习了这节课
> 可以玩
> - **游戏外挂**
> - 假装自己是什么设备

