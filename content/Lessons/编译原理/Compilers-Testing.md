## 实际执行和符号执行

### 动态符号执行 (dynamic symbolic execution/concolic testing)

# 编译器测试
- **如何测试**？
- *测试预言问题*：给定任意的输入（源语言程序），需要能知道所期望的输出（目标语言程序）
	- 编译器测试难以做到这点
- **转化为**：给定输入（源语言程序）的**变换** -> 知道输出（目标语言程序）的**变换**
	- 编译器测试有机会做到
## 蜕变测试: metamorphic testing
> [!Note] Testing Attempts: Equivalence
> - *Mettoc*: equivalent programs should behave identically
> 	- revealed real bugs in GCC

e.g.:
`x=b*2 -> x=b+b`
`gcd(z,x) -> gcd(6*z, 6*x)`



# 测试
如果生成一个合理的**源程序**？
- **最好的工具**RCGen
- RCGen 甚至可以发现 GCC 的真实问题


**测试生成**（充分性问题）
**测试语言**（可能性问题）
