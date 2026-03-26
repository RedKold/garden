## 从键盘到终端
键盘的老祖宗：keyboard
- 管风琴 (pipe organ)
- 1500s 击奏弦鸣乐器

### 打字机时代的遗产
- Shift/Caps Lock
- CR & LF
	- \r, CR (Carriage Return): 回车，将打印头移回行首
	- \n LF (Line Feed) 换行，将纸张向上移动一行
	- UNIX: \n include CR and LF both


### 电传打字机 (Teletypewriter)
实现了键盘打字、远端打印

### Video Teletypewriter
- milestone: VT100 (DEC, 1978)
- putchar：事实上的行业标准
	- 首个完整实现 ANSI Escapde Sequence 的终端
	- `80*24` 显示称为标号尊布局


### Thanks to Unicode
- ANSI Escape Code and toybox
- After *Unicode*
	- 可以画出很多 unicode art
- e.g.![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260326142744.png)

### 终端: 作为输入设备
- 输入字节流，直接送入操作系统
- quite like a Typewriter


### 伪终端和终端模拟器
- Pseudo Terminal: 想要多少有多少
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260326143209.png)
- 一对“管道”提供双向通信通道
	- 主设备 (PTY Master)：连接到终端模拟器
	- 从设备 (PTY Slave)：连接到 shell 或其他程序
- ` minitty`
	- unix 系统保留了对 tty 的称呼，抽象成了一种终端（teletypewriter）

**有趣的应用**：这种转发字符和时间，实际可以改造成“录屏”

**如果不满足于字符终端**：
- Sixel (six pixel)
	- 一个字符编码一个 6x1 的 Yes/No
	- 今天的终端大多支持
- 今天的终端设备多数都是模拟


## 终端和操作系统

### 终端：人机交互的第一个设备
