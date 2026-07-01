- M1 已经发布

# 操作系统中的对象
## 操作系统对象和 API
- **程序员**希望：
	- 操作系统帮我做一件事情
	- 本质都是访问操作系统中的对象

- **操作系统**设计者
	- 提供一套简单的稳定的通用 API

Windows: 开发者要什么我就给什么
- CreateFile, WriteFile, SetFilePointerEx, OpenProcess, VirtualBox
## Everything is a File
Unix: 我的用户都是 hacker
- Everything is a file
- 一个有名字的 **数据对象**(字节序列)
- 可以对任意位置读取/写入

一个非常普适的抽象

e.g. OS's wiki website
- 用目录来管理名字
- 一个 crazy idea: `dict[Path, bytes]`
- 数据库很 fantastic，但是 file system 恒久不衰也有理由

**到底有什么？**
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260324104558.png)

## UNIX Philosophy
### Keep It Simple, Stupid
- Do one thing and do it well.
- Work together
- Handle text streams as a universal interface
	- 非常具有前瞻性！
	- **机器和人都很容易理解**
	- **文本**是一个很适合 AI 理解的东西 as well...
	- The basic for today's agentic AI

	文本接口机器可读，人也可读，损失一部分性能（相比于数据库）
	- 1970s 做出这样的 trade-off 相当具有前瞻性
	- Compared to Windows: windows return a json table

### 操作系统还会“假装”一些文件
- `/proc, /dev, ...`
	- real device `/dev/sda, /dev/tty`
	- fake device `/dev/urandom, /dev/null`
- 操作系统把它们都当作“可读写的对象”

# 访问操作系统中的对象

## 文件描述符：访问操作系统对象的指针
```c
struct FILE{
	char *data;
	size_t offset;
}
```

OS provide API to access object in OS.
- based on idea: [[#Everything is a File]]
- `open`: `malloc(sizeof(struct FILE));`
- `close`: `free(f);`
- `read/write `: `*(f->data++);`
- `lseek`: `f->+=offset;`
- `dup`: `f_new = f;`

应用程序如何访问操作系统中的 struct FILE 呢？
- 持有一个由 OS 解读的“指针”


**另一个地址空间**
- 0 (stdin), 1 (stdout), 2 (stderr) 
- open () always allocate the minimum unused `fd`
	- new file allocate from `3`
	- when file closed, `fd` can be reused

**Windows Handle**
- （句柄）错误翻译的保留.

## 操作系统的真正复杂性
API will inflect each other
- fork () will default  inherit **all resources**
	- interactions... complex...


Key findings:
  1. Memory is COPIED: Child modifying global_var/stack_var didn't affect parent
  2. File descriptors are SHARED: File position was shared between processes (writes didn't overwrite each other)
  3. PIDs are different: Child gets new PID, PPID is parent's PID

as a Comparison

**Windows Handle API**
- 默认 handle 不继承
- 最小权限原则





## 小 TIP：如何安全地保存一个文件？给出系统调用顺序

可以阅读 OSTEP for some tips (Chapter 39, directory and file)

- `rename()` 通常是一个 atomic call
- `fsync()` 可以强制写入磁盘
所以可以


```c
	int fd = open("foo.txt.tmp", O_WORNLY|O_CREAT|O_TRUNC);
	write(fd, buffer, size);	// write the new version
	fsycn(fd);	// update
	close(fd);
	
	// if above is done, we can rename it
	rename("foo.txt.tmp", "foo.txt");
```


