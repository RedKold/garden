## 指令选择算法
**线性 LR**
- 我们有了 CFG 之后，可以确定**指令转移指令**的目的地址
### 宏扩展
- 指令选择器遍历 `IR`，宏扩展器根据输入的 IR 种类，选择对应的**生成模版**
- 宏匹配成功我们使用匹配的字符串作为参数来执行相应的宏或者函数
	- maybe useful: 窥孔优化（peephole optimization）
- 不成功，则匹配失败，并报告错误


：

# 中间代码与 MIPS32 指令映射及翻译优化说明

本节主要展示了常见中间代码（如三地址代码）向 MIPS32 汇编指令逐条翻译的映射关系，并指出这种简单翻译方式在实际代码生成中所面临的效率缺陷。

## 1. 中间代码与 MIPS32 指令映射表

下表展示了各种运算、控制流、函数调用以及指针/内存访问操作的翻译规则。其中 `reg(x)` 表示变量 `x` 分配到的寄存器，`#k` 表示常数。

|**类别**|**中间代码**|**MIPS32 指令**|**说明 / 映射逻辑**|
|---|---|---|---|
|**标签**| `LABEL x :` | `x:` |定义汇编标签|
|**赋值 & 运算**| `x := #k` | `li reg(x), k` |加载立即数（Load Immediate）|
|| `x := y` | `move reg(x), reg(y)` |寄存器间数据复制|
|| `x := y + #k` | `addi reg(x), reg(y), k` |寄存器与立即数相加|
|| `x := y + z` | `add reg(x), reg(y), reg(z)` |两寄存器有符号相加|
|| `x := y - #k` | `addi reg(x), reg(y), -k` |寄存器与负立即数相加实现减法|
|| `x := y - z` | `sub reg(x), reg(y), reg(z)` |两寄存器相减|
|| `x := y * z` | `mul reg(x), reg(y), reg(z)` |两寄存器相乘，结果存入目标寄存器|
|| `x := y / z` | `div reg(y), reg(z)`<br><br>  <br><br>`mflo reg(x)` |两寄存器相除，并将商从 `lo` 寄存器移至目标寄存器|
|**内存/指针访问**| `x := *y` | `lw reg(x), 0(reg(y))` |加载字（Load Word）：以 `y` 寄存器的值为地址读取内存|
|| `*x = y` | `sw reg(y), 0(reg(x))` |存储字（Store Word）：将 `y` 寄存器的值写入 `x` 所指向的内存|
|**控制流 (跳转)**| `GOTO x` | `j x` |无条件跳转（Jump）|
|**函数调用**| `x := CALL f` | `jal f`<br><br>  <br><br>`move reg(x), $v0` |跳转并链接（Jump and Link），并将返回值从 `$v0` 复制到 `x` |
|| `RETURN x` | `move $v0, reg(x)`<br><br>  <br><br>`jr $ra` |将返回值存入 `$v0`，并跳转回返回地址寄存器 `$ra` |
|**条件跳转**| `IF x == y GOTO z` | `beq reg(x), reg(y), z` |若相等则跳转（Branch if Equal）|
|| `IF x != y GOTO z` | `bne reg(x), reg(y), z` |若不相等则跳转（Branch if Not Equal）|
|| `IF x > y GOTO z` | `bgt reg(x), reg(y), z` |若大于则跳转（Branch if Greater Than）|
|| `IF x < y GOTO z` | `blt reg(x), reg(y), z` |若小于则跳转（Branch if Less Than）|
|| `IF x >= y GOTO z` | `bge reg(x), reg(y), z` |若大于等于则跳转（Branch if Greater Equal）|
|| `IF x <= y GOTO z` | `ble reg(x), reg(y), z` |若小于等于则跳转（Branch if Less Equal）|

## 2. 逐条翻译方式的局限性分析

图片下方的文字指出，这种逐条翻译（Naive Translation）的方式往往无法得到高效的目标代码。

### 典型问题示例：数组元素访问

- **已知条件：**
    
    - 假设程序需要访问某个数组元素 `a[3]`。
        
    - 变量 `a` 的基地址（首地址）已经被保存到了寄存器 `$t1` 中。
        
    - 我们的最终目标是将保存在内存中的 `a[3]` 的值放到寄存器 `$t2` 里。
        
- **简单翻译的缺陷：**
    
    - 如果严格按照表 5.7 的逐条翻译规则，这段功能对应的中间代码里**至少需要两条**。
        
    - 因此，最终翻译出来的 MIPS32 代码也**同样需要两条指令**（例如：先计算 `a + 12` 的地址，再进行 `lw` 加载）。
        
    - 这种方式忽略了 MIPS32 架构自身的指令集特性—— `lw` 指令本身就支持变址寻址（如 `lw $t2, 12($t1)`），如果进行指令合并与优化，实际上仅用**一条指令**就能完成该操作。



# Test

```bash
Examples

 # Run base tests for lab4 with your parser
 ./run.sh -r ../../lab1/Code/parser -e base -l 4

 # Run base tests + advanced tests, don't stop on failure
 ./run.sh -r ../../lab1/Code/parser -e base -l 4 -a -c

 # Run with a 60-second timeout per test
 ./run.sh -r ../../lab1/Code/parser -e base -l 4 -t 60
```