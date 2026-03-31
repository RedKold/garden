# Review: Process on OS
- A asm/C/python
	- 只有寄存器和内存是可以通过指令不受监督“随意访问”
	- 其他所有功能都需要系统调用实现
	- **进程管理 `API`**: `fork`, `execve`, `exit`
	- 内存管理 API：`mmap`, `munmap`, `mprotect`
	- 访问对象 API：`open`, `read`, `write`, `lseek`

# C 和 libc

## The C Programming Language
- SimpleC's grammar and semantic meaning
	- 指针、数组、结构体、函数调用
	- 内存和寄存器的直接操作
- C 语言的*完全体*
	- Foreign Function Interface, FFI
	- C 可以和汇编实现的函数**链接**(ABI)
	- C 可以使用 inline assembly

```c
void start(){
	__asm__("mov $60,	 %eax\n"
			"xor %edi, 	 %edi\n"
			"syscall");
}
```

## Aside: Inline Assembly Demystified
- 设计糟糕的 DSL

## 构建应用生态：组合、复用、分层
**在抽象层上积累抽象层**
- 1950s: 只有汇编语言
- 1970s: 用 asm =行为艺术
	- 1960s 高级语言普及
	- C, Pascal 和结构化程序设计；UNIX
- 2010s：C 语言=古法编程
	- Python, Java, JavaScript
- 2026: 用高级语言编程=行为艺术
	- 自然语言=编程语言

**标准化的力量**
- IOS C
- POSIX C 的子集 (unistd. h)


## The C Standard Library
调试 glibc
- 没必要。优化很多

学习 musl 吧！

# 探索 libc
## 指令集体系结构和计算的封装
### Freestanding
- Freestanding 环境
	- C 代码直接翻译成指令*在机器上执行*
		- 有些标准库功能依赖操作系统 (e.g. putchar, exit)
		- But, OS can be implemented by C.所以一定有不依赖操作系统的库功能，作为实现 OS 的基础 
		- Freestanding: 不依赖任何 Host OS 功能（`syscall`）
- 机器/平台相关的常数和定义
	- `stddef.h`, `float.h`, `limits.h`, `inttypes.h`, `stdint.h`
	- 你所有不知道的定义都在里面
	- 有趣的 `offsetof(T,m)`，计算结构体的 offsetof
		- [理解 C/C++中的 offsetof 宏原理 - Zhihu](https://zhuanlan.zhihu.com/p/677486774)
```cpp
#define offsetof(s, m) (reinterpret_cast<size_t>(&reinterpret_cast<const volatile char&>(static_cast<s*>(nullptr)->m)))
```
	
- 指令集的语义和 ABI 相关的参数解析
	- `stdarg.h`
	- 寄存器传参复杂。
	- In PA, you try to read the man and use it

### 一些“随手可以实现的函数”
- `string.h`
	- `memcpy`, `memmove`, `strcpy`
- 



### `setjmp` / `longjmp`: 长跳转

和 GOTO 类似. 一个更大号的 GOTO

-  还记得 SimpleC 的模拟器 (形式语义) 吗？
`
```c
pc = stack[-1].PC
stack[-1].PC.next()
inst[pc].execute()
```
-  栈上的 “标记” 和 “长跳转”
```c
match inst[pc]:     
	case "call setjmp":         
		buf.depth = len(stack)        
		buf.pc = stack[-1].PC         
		stack[-1].retval = 0     
	case "call longjmp":         
		stack = stack[:buf.depth]         
		stack[-1].PC = buf.pc        
		stack[-1].retval = (x if x != 0 else 1)     
	case _:         
		inst[pc].execute()
```

## 系统调用的封装

### 文件描述符
#### Standard I/O: stdio. h
- `FILE*` 背后是一个文件描述符
	- C and UNIX 密不可分
- gdb can check FILE*
	- as stdout
- 封装了文件描述符上的系统调用 (`fseek`, `fgetpos`, `ftell`, `feof`...)

#### The `printf()` family
- **可以复用**
- 理应没有 `code clones`
- PA 中自己实现标准库： [[ics-pa3-report]]

```c
int vfprintf(FILE* const char*, va_list);
```


### 进程管理 
#### 退出管理
- abort (send a SIGABRT to myself), core dump
- exit : (exit normal), include flush stio buffers
- atexit: 注册 exit handler (normal exit 时被 call)

### err, error,  perror

`lipc` can tell to `perror` in which language.

使用 ls：
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260331114321.png)

- `perror` function in GNU has different translation version.

## 进程运行环境的封装
### 进程的运行环境
- 环境 (envp) 会影响应用的行为
	- UNIX use **环境变量**和**默认子进程继承机制**实现
	- e.g. LANGUAGE=zh_CN command
	- man 3 exec
- `int main(argc,char*argv[], char* envp[]);`

### man 7 environ
- global variable *environ*: who set value for it?
	- environ 是一个**变量**（符号）
- System V ABI "Initial Process Stack"

### C Runtime
终究程序从_start 开始运行
（其实并不是从 main 开始运行，我们需要准备一些给 main 用的东西）
- `_start` 也是 libc 的一部分
- 二进制文件链接了 crt1. o
- C runtime 最终把 control 交给了 `__libc_start_main`

调试 crtbegin. o, crtend. o, ctn. o




