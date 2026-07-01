# Review & Comments (2)

## 输入/输出设备

- 设备 = Canonical Device Model (总线上的寄存器)
    - 从 LED 灯到打印机、GPU，都遵循这个模型
- 设备驱动程序：把 file operations 翻译成设备能听懂的命令
    - 读、写、控制 (`ioctl`)

## 我们终于进入 “持久化”

- “持久化” 的数据保存在存储设备上
    - 存储设备就是 “字节序列” 的抽象
    - 今天展开这部分内容


# 数据的持久化
> Persistence: "A firm or obstinate continuance in a course of action in spite of difficulty or opposition"
## 正 vs 反
**持久化可能没有那么困难？**
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260521141152.png)
- 需要一个“能反复改写的状态”
	- 为了让计算机能访问，一个 bit 必须能用**电路读写**
### 磁带

**电磁感应：物理和数字世界的桥梁**
- 1D 存储设备：把 Bits“卷起来”（磁带：1928）
- 只需要一个机械部件 (转动) 定位
    - 读取：磁通量变化产生电动势，放大感应电流
    - 写入：强磁场下电子自旋方向翻转 (不是磁针物理翻转)

---
**密度可以高到离谱！**
- 和集成电路同级别的制程，但是结构要简单的多（存储部分：结晶）
- 离谱的 `300 Gb/in^2`


> [!Note] **磁带**：作为存储设备的分析
> - 存储特性
> 	- 价格低，容量**高**，可靠性**高**
> - 读写性能
> 	- 顺序读写**勉强**，随机读写**几乎完全不行**
> 		- 致命缺陷。RAM 很快也很需要
> - 应用场景
> 	- 冷数据的存档和备份；只需要顺序读（写）的场景：音频/视频

结构决定性质，性质决定用途了
- **读写头**慢慢转，慢慢放音乐
- **磁带**：AB 面音乐


### 磁鼓 (Magnetic Drum, 1932)
1D -> 1.5 D (1D x n)
- 用旋转的二维平面存储数据（无法内卷，容量变小）
- 读写延迟不会超过旋转周期（随机读写大幅提升）
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260521142251.png)
- **读写头在旋转**。最坏情况也是等一个旋转周期 $T$

### 疯狂内卷：磁盘（Hard Disk，1956）
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260529164858.png)

> In particular, it must just wait for the desired sector to rotate under the disk head. This wait happens often enough in modern drives, and is an important enough component of I/O service time.
> It has a special name: **rotational delay**

**对于一传感器版本**

- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260529165248.png)

  磁盘读取一个扇区需要经历三个阶段：
  1. 寻道（Seek） — 将磁盘臂移动到正确的磁道。寻道过程包括：加速 → 巡航 → 减速 →
   精确定位（settling，耗时 0.5~2 ms）。这是最昂贵的操作之一。
  2. 旋转延迟（Rotational Delay） — 等待目标扇区旋转到磁头下方。寻道完成后，盘片
  可能已经转过了若干扇区，需要等待目标扇区到来。
  3. 传输（Transfer） — 当目标扇区经过磁头下方时，实际读写数据。

  核心洞察： I/O 总时间 = 寻道时间 + 旋转延迟 + 传输时间。其中寻道和旋转延迟是主
  要的性能瓶颈，因为它们涉及机械运动，远比电子传输慢得多。

Doing the math

$$
T_{I/O}=T_{seek}+T_{rotation}+T_{transfer}
$$






#### 构造
1.5D -> 2.5D (2D x n)
- 在二维平面上放置许多磁带
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260521142649.png)
- 我们换了个折叠方式
- **扇区**编号
	- 每个盘的上表面和下表面都有读写头
	- 旋转速度很快
		- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260521142900.png)

#### 磁盘：存储设备的分析
> [!Note] **磁盘**：作为存储设备的分析
> **存储特性**
> - 价格**低**：高密度、低成本
> - 容量**高**：2.5D，上万磁道
> - 可靠性**高**：（高速运转的机械部件，潜在威胁）
> **读写性能**
> - 顺序读写：较高
> - 随机读写：勉强
> **应用场景**
> - 计算机系统的主力数据存储（便宜；坏了还能修）
> - 机械原理上容易修
> 	- 跌落划伤扇区... 总不会全部 goodbye?





**SSD**实在是效果太好了
- 所以正在侵占机械硬盘空间

#### 磁盘：性能调优
**为了读/写一个扇区**
1. 读写头需要到对应的磁道
	- `7200rpm -> 120 rps -> "寻道时间" 8.3ms `
2. 转轴将盘片旋转到读写头的位置
	- 读写头移动时间通常也需要几个 ms
**通过缓存/调度等环节**

> [!Note] Disk Scheduling
> Because of the high cost of I/O, the OS has historically played a role in deciding the order of I/Os issued to the disk.
> **With dis scheduling, we can make a good guess at how long a "job"(i.e., disk, request) will tkae**

**The disk scheduler** will try to follow the *principle of SJF* (shortest job first) in its operation

**SSTF: Shortest Seek Time First**
- But a pure SSTF approach may cause **starvation**
	- 如果有平稳的在 inner track 的 request，那么 pure SSTF 会忽略其他 track 的 request

**SPTF: Shortest Positioning Time First**
- also called **shortest access time first** or **SAFT**
> `it depends` is always the answer, reflecting that trade-offs are part of the life of the engineer






### 软盘 (Floppy Disk, 1971)
**把读写头和盘片分开——实现数据移动**
- 计算机上的软盘驱动器（drive）+可移动的盘片
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260521143836.png)
- 最初的软盘成本很低——一个纸壳子
- 软盘甚至有写保护功能，一个小孔决定
#### 软盘：作为存储设备的分析
**存储特性**
- 价格**低**：极低成本
- 容量**低**：裸露介质，密度受限
- 可靠性**低**：不要抱有太大的期望
**读写性能**
- 顺序读写：**低**
- 随机读写：**低**
**应用场景**
- 当年最主力的**传递数据**的方案
	- 可以把整个 OS 带着走（几十 KB 的 BASIC 代码就可以很复杂了）
- 今天的保存按钮。


## 坑 v.s.平
**坑**：天然容易“阅读”的数据存储
- 把字刻在石头上
**现代工业**：我们可以挖出更精细的坑

**黑胶**
- **模拟唱片**
- 声波刻录在黑胶盘片上，通过杠杆原理+探针，放大振动得到音乐

### Compact Disk (CD, 1980)
**在反射表面（1）上挖上粗糙的坑**
- 激光扫过表面，就能读出坑的信息来
	- 44.1kHz, 16-bit, 2 声道贝多芬第九交响曲 (74 分钟, ~700MB)，飞利浦 (碟片) 和索尼 (数字音频) 发明
	- Eight-to-Fourteen Modulation 编码 (8 bit 编码到 14 bit
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260521150626.png)

#### 光盘最有趣的特性：容易复制
光盘的坑是挖在透明塑料上的
- “压盘”后镀上反射膜
- 3s 生产一张 Blue Ray 100GB (33,000MB/s write speed)

分享信息变得很容易和廉价

#### 光盘：作为存储设备的分析
> [!Note] 光盘-存储设备的分析
> **存储特性**
> - 价格极低
> - 容量高（当然，也没有那么高）
> - 可靠性高
> **读写性能**
> - 顺序读：一般
> **应用场景**
> - 数字内容分发
> - （逐渐被互联网的速度优势取代）


### 挖坑：Other Thoughts
**Project Silica: 回归 Rosetta Stone**
- Random read + Append-only write = 任何数据结构
- 用激光在玻璃上改变晶体结构，写入 1 个 bit，
- 实现数据结构

## 充电 v.s. 放电
### Solid State Drive (1991)
磁和坑都不太完美
- 磁：机械部件（无法避免的 ms 延迟）
- 坑（光）：挖坑效率低，填坑困难
“完美”=**电子的密度、电路的死都**
- Flash Memory“闪存”
- 如何在电路中持久 1-bit
	- 挖个坑
	- 电子填进去=一个状态
	- 把电子放跑=另一个状态
- 甚至可以 MLC/TLC/QLC

### 1-Bit Flash Memory
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260521152611.png)
- Floating Gate 的充电、放电

### 闪存：作为存储设备的分析
**存储特性**
- 价格**低**：大规模集成电路
- 容量**高**
- 可靠性**高**：集成电路封装，不怕摔

**读写性能**
- **极高**，而且极高拓展性（电路天然并行）
	- 极为离谱的优点：**容量越大**，**速度越快**

---
开启了 U盘时代

### Flash Memory 致命缺陷
放电 (erase) 做不到 100% 放干净
- 放电**数千/数万次**以后，就好像是 “充电” 状态了 (Erase Saturation)
- Dead cell; “wear out”
    - QLC: 大约只有 1,000 次写入寿命
 有没有感到很害怕？
```Java
for i in range(1000):     Path("a.txt").write_text(str(i))
```
- 这个文件就该要损坏了？

### 软件定义磁盘
**你的 SDD、优盘，甚至是 TF 卡里都藏了完整的计算机系统**
- 更好的服务，随机写入，负载均衡
- **FTL**: Flash Translation Layer
- **"Wear Leveling"**: 用软件使写入变得 **“均匀”**
	- 维护一个 Logical-to-Physical Table ( L2P Table )
	- 再一次的
> Random Read + Append-only Write = **Any Data Structure**

#### FTL: Flash Translation Layer
![](https://jyywiki.cn/OS/2026/static/img/ftl-slide.jpg)

---
### Flash Disk 和 NAND Flash
优盘，SD 卡，SSD 都是 NAND Flash
- 但是复杂程度不同，效率/手 ing 也不同


> [!Warning] 不要买过度便宜的 U 盘
> - 设备 = 一组寄存器 -> 设备可以“伪造”
> 	- 联系前面 I/O 知识：OS 链接 U 盘会发一个信息给 U 盘，二者通过协议确定-存储协议、容量等信息
> 	- U 盘的回答来自其内置的计算机系统的寄存器，存储的信息
> 	- 中控信息的“容量”是可以修改的！


# 操作系统视角的存储设备
## **存储设备的抽象**
Random Access 的代价：**寻址**
- 磁盘/磁带/光盘：物理旋转
- 电路：选通信号

**减少寻址的代价：按块访问**
- SSD Page 内部的独立选通信号：浪费电路
	- SSD Page 同步并行读写
- **存储设备**=随机读写的 block array
	- `struct block disk[NUM_BLOCKS]`

**Block devices**
- 按块访问的字节序列（可以直接 mmap 道进程的地址空间）

## Linux Bio
### 一个应用程序 “看不见” 的接口
- 为你的存储设备实现 `struct block_device_operations` 和 `struct request_queue`，剩下的 read/write/mmap/… 都是文件系统的事了
- (凭什么不能看到看不见的接口？)

![](https://jyywiki.cn/OS/2026/static/img/blk-mq.jpg)


# Takeaway
[[OS-OSTEP-security-access]]