**随机化**
**每个字节都有一定的规定**。
- 每个进程内存的每个字节是什么怎么来的？

## 地址空间和指针
- 进程 `execve` 后的进程地址空间

Linux 进程的地址空间

- A detail
	- *process* is has *memory*

```c
// crazy-os
struct proc {
     // Process "virtual machine" state:
     // Register & memory
     struct CPUState cpu;
     uint8_t mem[MEM_SIZE];

     // Operating-system internal state
     char buf[256];
     int buf_len;
};
```

**地址空间** = CrazyOS "struct proc 中的 mem"
- pointer can access *address space*
```c
volatile char *ptr = input();
*ptr;	// read
*prt = 1 // store
```
- Why we need `volatile`?
	-  volatile 告诉编译器：这个内存访问不能被优化掉。
	- 没有 volatile 时，编译器可能会：
```c
  char *ptr = input ();
  *ptr;        // 这行被优化掉 —— "读了但没用，删掉"
  *ptr = 1;    // 如果后面又 *ptr = 2，前一行也可能被删掉
```
  加了 volatile 后，编译器必须：
  - 每次都真正去访问内存，不能用寄存器缓存的值替代
  - 不能重排、合并或删除对该地址的读写
  在操作系统/地址空间场景下，volatile 的意义在于：ptr 指向的内存可能被外部改变（其他进程、硬件、内核映射等 ），编译器无法感知，所以必须强制每次都去读真实的内存。

### Re-Consider Pointer
事实上你甚至可以给 main 函数赋值给一个函数指针。
By playing pointer you will learn more than you think

- malloc (size) 的内存从哪里来？

### 计算机的世界没有魔法
- **每个字节有一定的规定**
- **字节**“是什么”？
	- 由 CPU 的 ISA 决定如何解释
- **字节**“放在哪”？
	- 由链接和可执行文件格式决定
	- compile/link will generate types of segment (. text, .rodata...)
- **字节**“初始时什么样？”：由 OS loader 和 ABI 约定决定
	- Load segment, set 0 to .bss, init the CPU state (PC, SP), prepare the stack (aargc, argv, enpv, auxv)
	- 划定动态区域 (Heap, Stack)

`execve` 之后地址空间的探索...

- 从哪里开始到哪里结束，每一页的访问权限，都有规定，you can check [[ICS-PA4 note]] for more infomation
## **MMU**
visit [[ICS-PA4 note]] for your own `MMU`
- 操作系统给进程戴上了 VR 眼镜
- **系统调用**配置它 
	- UNIX: brk/sbrk
- change data segment size

我们抽象了一个新的系统调用：`Memory Map` system call

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260324100546.png)
- 在 `struct proc` 上增加/删除/修改一段可访问的内存





**所有游戏生效的金手指**
- cheat engine
- 不断的扫描进程内存，**找到变化的量**。
	- 你可以回忆你改 pvz 阳光数值的过程
	- **基于内存的游戏修改器**
- 反汇编可以做到这样的软件
- 理论上我可以调试任何进程
	- 所以我也可以修改任何游戏

 


