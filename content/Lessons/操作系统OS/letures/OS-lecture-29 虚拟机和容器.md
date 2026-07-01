# 全系统虚拟机
## 一个进程，一台机器
**Full System Emulation**
- 比如 NEMU：取指令、译码、执行
	- 致命的**性能**：不足 native 的 10%
- PA4 x86 JIT
	- 什么是 JIT?（Just-In-Time Compilation）
	- 即时编译，介于**解释执行**和**提前编译**
	- **频繁执行的代码块动态编译成宿主机的机器码并缓存**，下次执行时直接运行编译后的代码，而不是逐条解释。

## QEMU dyngen (dynamic generator)
- 借用一个有优化能力的编译器，为每条指令生成优化后的本机代码
- 运行时把连续的非跳转指令用 `memcpy` 组装起来+ `relocate`

- **example**: 解释执行 `addi r1, r1, -16`
```c
// 手写的汇编 (T0 通常是寄存器)
// op_movl_T0_r1
// op_addl_T0_im
// op_movl_r1_T0

void op_addl_T0_im(void) {
	// T0是某一个寄存器
    T0 = T0 + ((long)(&__op_param1));
	// __op_param1: 一个我现在还不知道的立即数
}
```


- Just-in-time: 把连续的指令代码交给编译器优化

## 黄金时代
Disco
- SOSP'97; Mendel Rosenblum 是 LFS 的作者
- Virtual machine monitors 重现江湖

- **ISP**(Internet Service Provider) 提供的是物理机
	- 绝大部分应用的 CPU 的空闲率达到 90%
- **虚拟机**：和物理机用起来完全一样，但一台能当 $n$ 台卖
	- **黑心商人**: 我们都是 oversubscribe 的💰
	- 赌：不是所有人都会同时访问服务

## 大学的使命
技术有了->产品投机家
产品投机死了->技术永存

**不学技能**，**谈不上品味**
- [Live updating operating systems using virtualization](https://dl.acm.org/doi/10.1145/1134760.1134767) (VEE‘06); 2007 年，VMware 在纽交所上市

**我们还在**Reward Hacking

> 保研结束后的一瞬间，分数不再有意义的时候，你是否会失去人生的动力？


- 第一手的经验是不会骗你的。

## 关键技术：直接执行特权代码
操作系统和进程，**看到的都是虚拟地址空间**
- 只不过操作系统可以修改“VR 眼镜”（CR3，页映射）
**VMM**
- 模拟所有“有系统级副作用”的指令，例如关中断、I/O
	- 对 CR3 和叶表的修改
	- 改写一份页表到 CR3
		- **CR3** 是 x86 架构中的一个**控制寄存器**，也叫做 **页目录基址寄存器 (Page Directory Base Register, PDBR)**。
		- Shadow page table
- 可以在 Ring 3 模拟执行 Ring 0

VMM：
- 让 Guest OS 以为自己掌控一切（完整的硬件、特权级）


# 操作系统级虚拟化
## 虚拟化的另一个方法
**操作系统**：我自己就能虚拟化自己啊

让 `pid` 不再是整个操作系统唯一的
- 给每个进程增加一个 `osid`，增加系统调用 `vos(fs_root)`
	- 创建新的 osid
		- 有非常大好处
	- pid 从 1 开始分配
	- fork 继承父进程的 osid

## Aside: pidfs
**pid**是会回收利用的

## 发明 Linux Namespace
`osid` 需要管理：
- pid
- user
- mnt
- ipc
- net
- time
- uts
- Linux namespaces


**实现资源的控制**
- “圈一些进程”，设定资源使用策略
- 你发明了 cgroups!
	- `cat /proc/*/cgroup`; `/sys/fs/cgroup`


## 云时代的虚拟机
- 如果只需要 Linux
	- 那么容器和虚拟机**完全一样**
	- 开销比虚拟机低很多，安全性略低
		- 可以在一台物理机上部署更多的服务了！
- Kubernetes (K8s)：“**容器编排**”
	- 动态、透明管理的“无状态”容器 Pod
		- 容器出错、网络异常、节点崩溃都可以随时在另一个地方重启
	- 声明式的访问和路由
		- 配置好后，只要访问 Gateway API，自动路由到正确的容器
		- 自动的负载均衡、健康检查......


> [!Tip] 结语
> 技术是可以改变世界的


