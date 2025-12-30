# 作业

- [[ICS-1]]
- [[ICS-2]]
- [[ICS-3]]
- [[ICS-5]]
- [[ics-6]]
- [[ics-7]]
- [[ics-8]]
- [[ics-9]]

# pa
- [[ICS-PA1-report]]
- [[ICS-PA1 note]]
- [[ICS-PA2 report]]
- [[ICS-PA2 note]]
- [[ics-pa3-note]] 
- [[ics-pa3-report]]
- [[ICS-lab-cachesim]]

# exam
- [[ics-exam-example]]
- [[ics-exam-2018]]

#TODO 
- 可执行程序的存储器映像
- 

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251108123433.png)

# 授课
office: 1008
QQ
## [[ICS - Section 2 ：数据的机器级表示与处理]]
## [[ICS - Section 3 程序的转换和指令系统]]
## [[ICS - Section 4 程序的机器级表示]]


##  [[ICS - Section 5 程序的链接和加载执行]]

##  [[ics - section 6 内存层次结构 memory hierarchy]]



##  [[draft/ics - section 7 虚拟存储器]]




##  [[ics - section 8 异常处理控制流]]

##  [[ics - section 9 IO|ics - section 9 I/O]]


## About
- **浮点的表示**
	- convert to 单精度浮点
- ALU 整数运算的加减
	- 4 个标志位
	- CF
		- 管的无符号数
	- OF
		- 管的是带符号数
		- 表示的是带符号数，结果是不是在内
- **对齐**
	- 第一个是个 ` char`，第二个是个 `int`，如何对齐
		- `int` 从第四个开始
	- 4-align


 **栈帧必考**
 `%ebp - 8 ` 是谁？（first param）
 代码都是 IA-32
 会有一些 x86-64 的简单问题
 - 主要区别：**传参**

- not test
7.7
8.4
8.5
5.6
3.3.5
3.3.6
3.3.7

- 以下哪些东西需要重定位？
![eb1d65b14804121654774ac07421b012_720.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/eb1d65b14804121654774ac07421b012_720.png)
- function names
	- `fopen`
	- `fread` 
	- `fclose` (老师挡住了)	
	- `exit(1)`
	- 外部 `sort`
- 全局变量
	- 需要编译时候确定-到底在 `.o` 的哪里
	- `rec_idx`
	- `const int rec_num`
	- `fopen` 的 `"rb"` 是一个格式符，**需要重定位到代码段**。也需要重定位
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20251227141918.png)

![IMG_0270.jpeg|400](https://kold.oss-cn-shanghai.aliyuncs.com/IMG_0270.jpeg)

- 缓存区溢出是怎么让程序崩溃的？


- 整理栈图到课本 #TODO 