# 静态链接和加载
- Review：什么是可执行文件？
	- ~~click then come a  window ~~
- After learning **OS**
	- A file (object) in OS
	- a byte series (edit like string)
- 一个描述了进程初始内存布局的*数据结构*


## 真正的可执行文件
UNIX a.out "assembler output"

```c
struct exec {
    uint32_t  a_midmag;  // Machine ID & Magic
    uint32_t  a_text;    // Text segment size
    uint32_t  a_data;    // Data segment size
    uint32_t  a_bss;     // BSS segment size
    uint32_t  a_syms;    // Symbol table size
    uint32_t  a_entry;   // Entry point
    uint32_t  a_trsize;  // Text reloc table size
    uint32_t  a_drsize;  // Data reloc table size
};
```

## 和ELF 搏斗...

ELF 为机器效率设计的：高效、紧凑
- bitfiled, offset, 交叉引用
- 人类不擅长...


## Funny Little Executable
- ELF 本质是一个*描述*
	- 基本信息
	- 内存布局 (memory layout)，哪部分是什么数据
	- 其他（调试信息，符号表，重定位）

- *总是可以在等价的描述形式转换*
	- 可以做一个更易读的转换工具！
	- By AI

## Linux 内核的加载器
`execve(path,argv,envp)`
- 操作系统内核解析path、完成加载

## Shebang 运行程序
UNIX 对注释的妙用（滥用）

存在跨系统不一致。
- 对于 `#! A B C`, Linux 会将 `B C` 作为一个参数，而macos 会依次传


优先使用 `#!` 作为解释器

我们创建一个 `a.sh`

```sh
#! /bin/bash
echo "Hello!"
```

when you type `./a.sh`, it will translate to `/bin/bash  ./a.sh` 
- 对注释的滥用


> [!Note] What is Shebang
> 在 UNIX 的早期，为了能更方便地将脚本作为可执行文件，实现了 `#!` 开头的 “可执行文件”，并沿用至今。Shebang 会调用第一行中执行的命令和参数，并把这个脚本文件作为命令行参数传入。


![image.png|500](https://kold.oss-cn-shanghai.aliyuncs.com/20260407111132.png)


# 动态链接和加载
- **Why We need dynamic link?**
静态链接的世界是——
```c
struct exec {
    uint32_t  a_midmag;  // Machine ID & Magic
    uint32_t  a_text;    // Text segment size
    uint32_t  a_data;    // Data segment size
    uint32_t  a_bss;     // BSS segment size
};
```

- **游戏**：给运行依赖的 `.dll` 复制很多份好像并不是好主意...

- **我们希望“拆解”应用程序**
	- 运行库和代码分离

## 探索动态链接的行为

怎么探索呢？
- 我们开一个包含 `10M` 的程序（由 `nop` 组成），然后运行 100 个实例（采取动态链接），如果每个进程都有一个副本，那么内存占用将到达 `1GB`


## 动态链接程序的加载
- 先有鸡还是先有蛋？
	- `musl-gcc-static` 编译出的 `_start` 在 `crt1.o` 中
		- If libc is *dynamic-link*, where is `_start`?
		- `__libc_start_main` where?

> [!Note] 动态链接的第一条指令
> 相比于静态链接，*动态链接*的第一条指令不在自己的 `_start` 里面。
> `crt1.o` 还是静态链接的，而动态链接 a.out 的第一条指令不是程序的_start
> "ld-linux. so": “写死”在ELF 文件的 INTERP (interpreter) 段里的
> - 我们可以调试，甚至直接编辑它
> - glibc 是用 ld-linux. so 调用 mmap 系统调用加载的


## Aside: 阅读 ld. so 手册

### man 8 ld. so
- 打开系统世界大门的金矿
**记录了**glibc 里面的动态链接的有趣的事情。


### LD_PRELOAD
一个神奇的"hook"机制
- ld. so: 谁先被加载并 **首次满足未定义** 符号，谁就生效
	- 如果在任何库加载之前 load 一个. so，就可以“覆盖”任何符号
	- LD_PRELOAD: 这里可以玩的花活就多了
**实现变速齿轮**
- 覆盖 time-related functions?

> [!Note] 可以用来实现外挂？
> - 我只是让 AI 覆盖 time-related functions，他就可以给你惊喜
> - AI 还知道 LD_PRELOAD 可以玩的花活
>     - glXSwapBuffers 实现透视挂
>     - 劫持 rand() 实现 “随机”
>     - 非侵入性的日志 (例如 malloc/free trace)

## Aside：实现动态链接
有更多的细节需要追问


### PLT: 没能解决数据的问题

#### 数据不能像代码一样 “两级跳转”

`extern int x; x = 1;`

- 如果在同一个 .so: `mov $1, x(%rip)`
- 如果在另一个 .so: `mov GOT[x], %rdi; mov $1, (%rdi)`
    - main 访问 stdout: 高效
    - libwheel. so 访问 stdout: 低效

#### 不优雅的解决方法

- -fPIC 默认会为所有 extern 数据增加一层间接访问
    - 可以通过 **attribute**((visibility (“hidden”))) “告诉” 编译器