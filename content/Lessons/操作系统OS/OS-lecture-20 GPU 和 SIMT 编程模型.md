# CPU 是什么

你可以在 linux 系统查看 cpu info
`cat /proc/cpuinfo`

以我的阿里云的节选为例
```
bogomips	: 5000.00
clflush size	: 64
cache_alignment	: 64
address sizes	: 46 bits physical, 48 bits virtual
power management:

processor	: 1
vendor_id	: GenuineIntel
cpu family	: 6
model		: 85
model name	: Intel(R) Xeon(R) Platinum
stepping	: 4
microcode	: 0x1
cpu MHz		: 2500.000
cache size	: 33792 KB
physical id	: 0
siblings	: 2
core id		: 0
cpu cores	: 1
apicid		: 1
initial apicid	: 1
fpu		: yes
fpu_exception	: yes
cpuid level	: 22
wp		: yes
```


无情的执行指令的机器
- `rv32ima_step`, "顺序执行"
- 顺序执行只是精心设计的假象
- CPU 天生可以并行执行
	- 指令级别的并行
	- 怎么检测？
	- 我们发现 bogomips > cpu MHz

- **本质上**CPU 内置了一个编译器，来处理任务

## Instruction-level Parallelism 意味着什么？
在单线程能效和性能之间，CPU 选择了后者
- 跑得越快，浪费的越多
	- 每一个门电路的翻转都会产生 **热量**
	- CPU 里的“**编译器**”（指令调度）会消耗巨大的能量
		- 这些能量使计算“尽快完成”
		- 但不等同于“单位时间完成尽可能多的计算”

**直到进入 Dark Silicon** “暗硅时代”
- $P(热功耗)=CV^{2}f$
	- $C$：电容
	- $f$：频率
	- $V$：电压
	- 制程决定 $C$ 和 $f$
- **功耗墙**：散热峰值是有限度的，电路做的更大也无法发挥全部性能
- **墙** = 极限： 频率墙、内存墙、I/O 墙......
- **税** = overhead：存储税、网络税、计算税......

##  面对功耗墙
**单位面积上能反转的逻辑门是有限的**
- 如何压榨出更多的 performance/watt?

### Approach I：让一条指令能处理更多的数据
- SIMD (Single Instruction, Multiple Data)
	- “调度一条指令”浪费的能量大概是定数
		- 做一次 add 消耗甚至远不如 schedule
		- 如果能一条指令多做一些事，就稀释了调度代价

#### Aside: 编程中的"SIMD"
**Bitset**
- `(arr[x/8] >> (x%8)) & 1`
- lowbit: `(x & -x) == (x & (~x + 1))`
	- 提取出一个整数在二进制表示下，最低位的那个 $1$ 以及它后面的所有 $0$。

```python
def count_ones(n):
    count = 0
    while n > 0:
        n -= (n & -n) # 减去最低位的 1
        count += 1
    return count
```

#### 引入 Packed Register
在计算机体系结构（尤其是 SIMD 指令集如 x86 的 AVX/SSE 或 Arm 的 Neon）中，**Packed Register**（打包寄存器）是一种能够同时存储并处理多个独立数据元素的寄存器。

与传统的通用寄存器（每次只能处理一个标量数据）不同，Packed Register 将一个超宽的寄存器（如 128 位、256 位或 512 位）划分成若干个等长的通道，从而实现 **SIMD（单指令多数据流）** 并行计算。


#### 寄存器越长，调度代价就摊薄得越多
**MMX -> SSE -> AVX -> AVX-512**
- AVX-512 已经达到热密度极限 


#### SIMD: 没能完全解决问题
SIMD 指令仍然是在 CPU 里调度的
- 参与到缓存和动态流水线中，和其他类型的指令格格不入
- 抢缓存（这也是动态调度）、抢功耗

- **失败的尝试**：Very Long Instruction Word (VLIW)
	-  消灭动态流水线，编译器扛下所有，静态指令调度
- **成功的尝试**：简化指令调度，改成横向扩展
	- 单个大 CPU -> 多个并行小 CPU
	- 还可以更小吗？


### Approach II:


# 改变人类命运的另一条世界线
## 游戏机遇到的性能问题
CPU 无法根据“图形场景描述”完成图形绘制

```c
for (int y = 0; y < H; ++y){
	for(int x = 0; x < W; ++x)
		putchar(f(x,y) ? '*' : ' ');
	putchar('\n');
}
```
- 对 `f(i,j)` 的计算是 embarrassingly parallel

- return to 1983
- Nintendo Entertainment System (NES)
	- MOS 6502 @ 1.79Mhz； IPC=0.43
	- 屏幕共有 256 x 240 = 61k 像素（64 色）
	- 60fps->10k 指令完成 61k 像素的渲染

#### NES 的图形描述

“场景描述”：数据结构
- 以 8x8 的“贴块”（tile）做基本着色单位
- 有各种各样的 trick

## 图形渲染
我们对每一个像素做的事情是一样的
- 依次计算出背景像素、Sprite 像素的颜色，然后合成
	- 一直用到了今天...
- 这就是早期的"fixed-function pipeline GPU"
- 每个 GPU“线程”都在做完全相同的动作

**逐行扫描**


## 从 PPU 到 2D 图形渲染管线
**扩展“场景描述”的数据结构**

- 2D 图形：动态拼凑图片 (texture) 实现场景的绘制
	- 依然是“固定循环”，执行完成相同的逻辑
	- 我们发现，我们可以不用**块块(tile)了**，而是用贴图


## 从 2D 到 3D
**继续拓展“场景描述”的数据结构**
- Vertex, Face, Texture, Material, Light, Camera
- 三角剖分定理，**因此看到的一切都是三角形**
- 3D 空间中的各种变换 (translation, camera projection, ...), 在 4D 齐次坐标下是线性的


**矩阵的运算**：平移、旋转
- 一个四维的矩阵。

从线性代数的角度来看，从 2D 到 3D 的跨越不仅仅是多了一个 $z$ 轴，更是一次从**平面几何**向**空间解析几何**的升维。

以下是实现这一转变时，在线性代数层面上最核心的几个变化：

---

### 1. 从 $3 \times 3$ 到 $4 \times 4$ 的齐次坐标变换

在 2D 游戏中，我们通常使用 $3 \times 3$ 的矩阵来表示平移、旋转和缩放。而在 3D 空间中，为了能够用统一的矩阵乘法处理平移（Translation），我们需要引入**齐次坐标（Homogeneous Coordinates）**，将维度提升到 4D。

- **2D 向量**：$(x, y, 1)$
    
- **3D 向量**：$(x, y, z, 1)$
    

所有的变换矩阵变为 $4 \times 4$。这多出来的一维不仅解决了平移问题，还是实现透视投影（Perspective Projection）的关键。

---

### 2. 旋转：从“一个角”到“一个轴”

在 2D 中，旋转非常简单，因为只能绕着垂直于屏幕的 $z$ 轴转，旋转只需一个标量角度 $\theta$。

但在 3D 中，旋转变得异常复杂：

- **欧拉角（Euler Angles）**：通过绕 $X, Y, Z$ 三个轴的旋转序列（Pitch, Yaw, Roll）来描述。虽然直观，但会遇到万向锁（Gimbal Lock）问题。
    
- **四元数（Quaternions）**：这是 3D 游戏开发的核心。它通过一个 4D 向量 $q = (x, y, z, w)$ 来表示旋转，能有效避免万向锁，且在两个旋转状态之间做球面线性插值（Slerp）时更加平滑。

## 从数据结构到编程语言
**可编程的“着色器”**（shader）
- Vertext shader: 可以编程修改**几何形状**
	- Killer application: 实现“扭曲”，例如水面的波纹、毛发的摆动
- Fragment (Pixel) shader: 可以编程修改颜色
	- Normal mapping (**法线贴图**)：远距离观察时“骗过”光照计算
	- 回忆 Raw 格式照片（记录原始传感器的三色光子信息）
		- 可以调整曝光等-即调整每个像素的信息（编程实现的）
		- `pixels(x,y,r,g,b)` -> we have a function...

> [!Note] 简单介绍一下法线贴图
> **法线贴图（Normal Mapping）** 是 3D 游戏和图形学中一种极其巧妙的“欺骗”技术。它的核心目的在于：**让只有少量多边形的低模（Low-Poly），在视觉上呈现出高模（High-Poly）才有的复杂细节和光影效果。**
> 在 3D 渲染中，光照效果主要取决于法线（Normal）的方向。法线是垂直于物体表面的一根虚拟箭头，光线打在物体上时，系统会计算光线与法线的夹角来决定这个点的亮度。
> 
> - **传统模型**：每个面的法线是固定的。如果平面是平的，光照看起来就是平整的。
>     
> - **法线贴图**：它并不是改变物体的实际形状，而是通过一张特殊的纹理图，**在像素级别上重新定义法线的方向**。
>     
> 
> 即使模型表面在几何上是完全平坦的，通过法线贴图改变了光照计算时的“法线角度”，大脑就会被光影误导，认为表面有凹凸起伏。

# PPGPU->CUDA

## Hacking “可编程” Shader
- "Fast matrix multiples using graphics hardware" (SC'01)
	- 图片意义上可能不 make sense，但是我们可以用图形技术作计算
	- 外积分解: $AB=\sum A_{{,k}}B_{{k,}}$
	- 把各种计算问题转换为“图像”的计算（物理模拟、天气预报...）
- **显卡**作高性能计算


## 什么是 Shader Program?
> [!Note] Shader Program
> 为一大堆东西 (vertex, pixel, ...) 执行同一段代码

- Back to first lecture of OS's concurrency part...
- 从绘图出发来想，并发执行呢？

- 在 GPU 上启动 2073600 个线程
	- 并行执行，允许访问 shard memory
	- 发明了 CUDA（的 programming model）



## CUDA-改变人类命运的时间线：复盘
- 这个“程序”有什么特别的地方？


## 尝试“单步执行”SIMT 程序
**一个 PC(program counter)** 管多个线程

- SIMT (Single Instruction Multiple Thread)
- CUDA 把这些线程捆起来，让他们共享一个 **PC**
	- 这样只需要一个译码单元

