#TODO 用真机探索 libc 的实现

## Debug Info 的作用
```sh
gcc main.c -g
```

你可以通过 `-g` 获得一个可以调试的程序

问题是，debug info 中都有什么？
- 二进制文件除了指令和数据，**还有额外的信息**
	- 低级程序状态 (PC, register, memory)
	- 高级程序状态 (栈帧，变量，代码行)


- Stack unwinding (backtrace)
	- `backtrace`
	- 应该使用过 gdb 的 bt 功能 (backtrace)
- Trace/profiler (Perfetto, flame, graph, ...)
- Crash dump 调试
- AddressSanitizer 诊断报告
**可以做逆向**

- what if you free a p then use it?


## 调试 setjmp/longjmp
比较复杂
- 一个有趣的 trick
	- setjmp 前写入所有寄存器
	- longjmp 在写入一次
	- 分析其行为
- 逆向出 aarch64 的 calling convention
	- 哪些恢复了，就是 callee-saved
	- 没恢复的都是 caller-saved


## 调试 gettimeofday
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260402152556.png)
`#ifdef VDSO_CGT_SYM`

**不陷入系统内核的系统调用**
- 在代码里面嵌入的，只需要跳进去即可。
- 特殊的 DSO 机制

遍历符号表，有一些辅助向量

栈中有一些信息
- `vvar`, 
	- cpu 进程号
	- clock（时钟）
		- 如果我不需要那么精确的时间，可以用时钟算
- `vdso`
	- 

## malloc 和 free
非常简单直观的 API

```
ptr = malloc(n);
free(ptr)
```

- 内存从哪里来的？
- 操作系统**不支持**分配一小段内存
	- 应用程序自己维护，每次多要一点，要的足够多就分配一个新的 page
	- [[ICS-PA4 note]] 中你对 malloc 的虚页机制是接触过的

特性
> The malloc() function allocates size bytes of memory and returns a pointer to the allocated memory.

内存背后的系统调用:
`mmap/sbrk`
- 大段内存，要多少有多少...


### malloc/free 犯下的错误
> I call it my billion-dollar mistake. It was the invention of the null reference in 1965. (Tony Hoare, 1935-2026)

- malloc 和 free 引入了额外的要求，在任何可能路径上必须有一次配对的 free，且该指针之后非法，不再使用

```c
free(p);
*p = 1;
// 符合C语言语法，但是不符合malloc free规范
```
- C 语言充满了内存漏洞

所以我们希望阻止这一点
```c
free(p);
p=NULL;
*p = 1; // not valid, can be found by C program
// 符合C语言语法，但是不符合malloc free规范
```


- **锁**
```c
void foo(){
	LockedObject lk(mutex);
	
	ptr = malloc(4);
	
	for(i = 0; i ... ){
		if(...)	{
		return;	// Release?
		}
	}
	
	free(ptr);
}
```

C++的思想：用构造器和析构器分析来处理内存
- 智能指针


### If you implement malloc?
- 如果 n 足够大
	- 直接用 mmap
	- In PA we do it

- 如果 n 不够大


#TODO 
read, 什么是真正严肃的科研

