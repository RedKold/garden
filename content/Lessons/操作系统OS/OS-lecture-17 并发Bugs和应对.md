## 并发编程
- 任何物理世界：spawn, join
- 桌子和钥匙：lock, unlock
- 等待全局同步条件：wait, broadcast
- 桌子和多把钥匙：P/V (acquire/release)

**并发编程很难**
- 人类是 "sequential creature"
	- 出生就接触顺序执行的程序
	- 并发编程容易产生顺序执行的幻觉.
	- 理解所有行为是 NP-Hard 的
- Learn from mistakes
- 