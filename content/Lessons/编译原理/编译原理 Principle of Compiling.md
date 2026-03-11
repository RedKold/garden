- 教师-许畅

> task number: 17



## 第三章词法分析

### 词法分析器的作用
编译器的第一阶段：划分 token

- 为什么要设立独立的词法分析器？
	- 简化编译器的设计
	- 首先完成一些简单的处理工作
	- 提高编译器的效率
		- 相对于语法分析，词法分析过程简单，可高效实现
	- 增强编译器的 *可移植性*

- 词法单元 (token)
- 模式 (pattern)
- 词素 (lexeme)
	- 源程序中的字符序列
	- 它与某个词法单元的模式匹配，被词法分析器识别为该词法单元的实例

### 词法单元的规约（正则表达式）

字母表 $\Sigma$ 上的 **正则表达式** 定义
- 基本部分
	- $\varepsilon$ 是一个正则表达式，$L(\varepsilon)=\{ \varepsilon \}$
	- 如果 $a$ 是 $\Sigma$ 上的一个符号，那么 $a$ 是正则表达式，$L(a)=\{ a \}$
- 归纳步骤
	- 选择：$(r)|(s), L((r)|(s)) =L(r)\cup L(s)$
	- 连接：$(r)(s),L((r)(s))=L(r)L(s)$
	- 闭包：$(r)^{*},L((r)^{*})=(L(r))^{*}$
		- 0 个或多个
	- 括号：$(r),L((r))=L(r)$
- **运算优先级**：`* > 连接 > |`
- **正则集合**(regular set): 可用一个正则表达式定义的语言

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260304102932.png)

为了书写方便，可以给正则表达式命名，**形成正则定义**(regular definition)

> [!Definition] 正则定义
> 是如下定义的定义序列：
> ```
> d1 -> r1
> d2 -> r2
> dn -> rn
> ```
> where
> - di 不在 $\Sigma$ 中，且各不相同
> - 每个 $r_{i}$ 是字母表 $\Sigma\cup \{ d_{1},d_{2},\dots,d_{i-1} \}$ 上的正则表达式，保证不会出现递归定义

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260304103536.png)


- 正则表达式的扩展
	- 基本运算符：选择、连接、Kleene 闭包
	- 扩展的运算符
		- 一个或多个实例：单目后缀 $^{+}$
		- 零个或一个实例：$?$
			- $r?\equiv\varepsilon|r$
		- 字符类
			- $[a_{1}a_{2}\dots a_{n}]$ 等价于 $a_{1}|a_{2}|\dots|a_{n}$
			- 使用 $-$ 符号，如：$[a-e]$ 等价于 $a|b|c|d|e$


### 词法单元的识别

词法分析器需要检查输入字符串，在其前缀中找出和某个模式匹配的词素

定义 $ws\to(\text{blank} |\text{tab}|\text{newline})^{+}$ 消除空白
- 当词法分析器识别出这个模式时，**不返回**词法单元，继续识别其他模式

- **状态转换图**(transition diagram)
	- 状态 (state)：表示在识别语素时可能出现的情况
		- state 是已处理部分的总结
		- 某些 state 为接受状态或最终状态，表明已找到词素
		- 加上 $^{*}$ 的接受状态表示最后读入的符号不在词素中
		- **开始状态**(初始状态) ：用 Start 边表示
	- 边 (edge)：从一个状态指向另一个状态
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260304104935.png)

- 有时候有相同模式，不同属性的，这时候就设置独立的、高优先级的状态转移图
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260304105249.png)

- **词法分析器的体系结构**：
- 从转换图构造词法分析器的方法
	- 变量 state 记录当前状态
	- 一个 switch 语句根据 state 的值转到相应的代码
	- 每个状态对应于一段代码
		- 这段代码根据读入的符号，确定下一个状态
		- 如果找不到相应的边，则调用 `fail()` 进行错误修复
	- 进入某个接受状态时，返回相应的词法单元
		- 注意状态有 $^{*}$ 标记时，需要回退 `forward` 指针

- 如何处理多个模式？
	- 按照优先级，顺序地尝试各个状态转换图，如果引发 `fail()` 则回退并尝试下一个
	- better：**并行**地运行各个状态转换图；通过 greedy 策略，识别与某个模式匹配的输入前缀
	- 实际使用的方法：预先把各个状态转换图合成一个状态转换图，然后运行这个状态转换图（后面介绍）


### 词法分析工具 Lex
- Lex/Flex 是一个有用的词法分析器生成工具
- 通常和 Yacc 一起使用，生成编译器的前端

- Lex **源程序的结构**
	- 声明部分
		- 常量： 表示常数的标识符
		- 正则定义
	- 转换规则
		- 模式 {动作}
			- 模式是正则表达式
			- 动作表示识别到相应模式时应采取的处理方式
			- 处理方式：C 语言代码表示
	- **辅助函数**
		- 各个动作中使用的函数


![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260304111633.png)
- `%{ * }%` 是注释

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260304111838.png)
- `yylval` 是一个把识别好的标识符加入标识符表的方案，标记一个属性


![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260304112056.png)

- Lex 中的冲突解决办法
	- **冲突**：多个输入前缀与某个模式相匹配，或者一个前缀与多个模式相匹配
	- Lex 解决冲突的办法
		- 多个前缀可能匹配时，选择**最长的**的前缀
			- 比如，词法分析器把<=当作一个词法单元识别
		- 最长的前缀与多个模式匹配时，选择列在**前面的**模式
			- 如果保留字的规则在标识符的规则之前，词法分析器将识别出保留字


### 有穷自动机 (finite automata)
- 本质上和状态转换图相同，但有穷自动机 (finite automata) 只回答 Yes/No
	- **不确定的有穷自动机**(Nondeterministic Finite Automata / NFA)
	- **确定的有穷自动机**(Deterministic Finite Automata / DFA)
- 都识别**正则语言**(regular language)
	- 对于每个可以用正则表达式描述的语言，均可用某个 NFA 或 DFA 来识别；反之亦然


NFA 的定义
-  一个有穷的状态集合 $S$
- 一个输入符号集合 $\Sigma$，即输入字母表 (input alphabet)
- 转换函数 (transition function) 对于每个状态和 $\Sigma\cup \{ \varepsilon \}$ 中的符号，给出相应的后继状态 (*next state*) 集合
- S 中的某个状态 $s_{0}$ 被指定为**开始状态**/**初始状态** (有些定义中可以有多个开始状态)
- S 的一个子集 $F$ 被指定为接受状态集合


![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260304113843.png)

- **输入字符串的接受**
	- 一个 NFA 接受 (accept) 输入字符串 $x$
		- 当且仅当对应的转换图中，**存在**一条从开始状态到某个接受状态的路径，且该路径各条边上的标号按顺序组成 $x$（不含 $\varepsilon$ 标号）
		- **NFA 接受的语言**：从开始状态到达接受状态的所有路径的标号串的集合
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260309102623.png)


- 确定的有穷自动机
	- 一个 NFA 被称为 DFA，如果
		- 没有标号为 $\varepsilon$ 的转换($\varepsilon$ 其实就意味着不确定性)，**并且**
		- 对于每个状态 $s$ 和每个输入符号 $a$，有且仅有一条标号为 $a$ 的离开 $s$ 的边
	- 可以高效判断一个串是否能被一个 DFA 接受
	- 每个 NFA 都有一个等价的 DFA


- DFA 的模拟运行
	- 假设输入符号是字符
	- $nextChar$ 读入 next character
	- $move$ 给出了离开状态 $s$ 且标号为 $c$ 的边的目标状态
```c
s = s0;
c = nextChar();
while( c != eof){
	s = move(s, c);
	c = nextChar();
}
if( s in F) return "yes";
else return "no";
```

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260304115002.png)

### 从正则表达式到自动机的转换
- 正则表达式可简洁、**精确**地描述词法单元的模式
- 模拟 DFA 的执行可高效地进行模式匹配

- 正则表达式->NFA
- NFA-DFA

#### NFA 到 DFA（子集构造法）
- **子集构造法**(subset construction)
	- 构造得到的 DFA 的每个状态和 NFA 的状态子集对应
	- DFA 读入 $a_{1},a_{2},\dots,a_{n}$ 后到达的状态对于从 NFA 开始状态出发沿着 $a_{1},a_{2},\dots,a_{n}$ 可能到达的状态集合
- 并行模拟

- 理论上最坏情况下的 DFA 的状态个数是 NFA 状态个数的**指数**多个
	- 但是对于大部分应用，NFA 和相应的 DFA 的状态数量大致相同

- **常用操作**
	- $\varepsilon-\text{closure}(s)$: 从 NFA 状态 $s$ 开始，只通过 $\varepsilon$ 转换可以到达的 NFA 状态集合
	- 求闭包：$\varepsilon-\text{closure}(T)$: 从 $T$ 中某个状态 $s$ 开始，只通过 $\varepsilon$ 转换能到达的 NFA 状态集合
	- $move(T,a)$: 从 $T$ 中某个状态 $s$ 出发，通过一个标号为 $a$ 的转换能到达的 NFA 状态集合


![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260309104822.png)

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260309104908.png)


- **基本思想**：
	- 根据正则表达式的递归定义，按照正则表达式的结构递归地构造出相应的 NFA
	- 算法分两个部分
		- 基本规则处理 $\varepsilon$ 和单符号
		- 对于每个正则表达式的运算，建立组合相应 NFA 的方法


- Thompson 构造法（Thompson's Construction）
	- Regular Expression to NFA
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260309111151.png)
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260309111230.png)
- You can check the slides  [课件](https://cs.nju.edu.cn/changxu/2_compiler/slides/Chapter_3.pdf) page 65 for more tips
- $N(s)$ 代表 $s$ 的已经构造好的小型自动机

- **NFA**合并的方法
	- 引入新的开始状态，并引入从该开始状态到各个原开始状态的 $\varepsilon$ **转换**
	- 得到的 NFA 所接受的语言是原来各个 NFA 语言的**并集**
	- 不同的接受状态代表不同的模式


#### DFA 状态数量的最小化
- 一个正则语言课对应多个识别此语言的 DFA
- 通过 DFA 的最小化可以得到 **状态数量最少的** DFA（不计同构，这样的 DFA 是唯一的）
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260309112420.png)

#### 状态的区分
- 状态的 **可区分**(distinguishable)
	- 如果存在串 $x$，使得从状态 $s_{1}$ 和 $s_{2}$ 一个到达接受状态另一个到达非接受状态，那么 $x$ 就区分了 $s_{1}$ 和 $s_{2}$
	- 如果存在某个串区分了 $s$ 和 $t$，我们说 $s$ 和 $t$ 是可区分的，否则它们是不可区分的
- **不可区分**的两个状态就是等价的，可以合并。

#### DFA 最小化算法

- 我们可以一步步的识别不可区分的组，然后识别出几个分支，他们内部是不可区分的（*找一个代表元素*），彼此之间是可区分的，就找到了最小的状态集合。


![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260309113405.png)

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260309113849.png)
- 在 $\Pi_{final}$ 的每个组中选择一个状态作代表，作为最小化 DFA 中的状态
	- 开始状态就是包含原开始状态的组的代表 
	- 接受状态就是包含了原接受状态的组的代表 (这个组一定只包含接受状态）
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260309114536.png)


## 第四章-语法分析

### 语法分析器的作用
- 基本作用
	- 从词法分析器获得词法单元的序列，确认该序列是否可以由语言的文法生成
	- 对于语法错误的程序，报告错误信息
	- 对于语法正确的程序，生成**语法分析树（简称语法树）**

### 语法分析树的分类
- General
- Top-down (Deal with *LL*)
- Down-Top (Deal with LR)
- 后两种方法通常从左到右、逐个扫描词法单元
- 智能处理特定类型的文法

### 程序设计语言构造的描述
- 上下文无关文法 (CFG)
- BNF
#### 上下文无关文法
one CFG contains 4 parts:
#### 终结符号 (terminal): 
组成串的基本符号（词法单元名字）
#### 非终结符号：
表示串的集合的语法变量
- 在程序设计语言中通常对应于某个程序构造，例如 *stmt*（语句）
#### **开始符号**：
某个被指定的 **非终结符号**
#### **产生式**：
- 描述将终结符号和非终结符号组成串的方法（一些规则）
	- 形式：头（左）部 -> 体（右）部
	- 头部是一个非终结符号，右部是一个符号串
	- 例子： $expression \to expression +term$
	![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260309120013.png)


- $term$ is a sequence of factors joined by "multiplying" operators (`*` or `/`)
	- 上面的产生式集合，描述了 $term$ 的产生
- $factor$ is the most basic building block. It has the highest precedence. (cannot be broken up by operators outside of it)
	- name ($id$)? or an entire expression wrapped by a parentheses
- $expression$: is the top-level construct, representing a sequence of terms joined by "adding" operators (`+` or `-`).
	- the final mathematical statement

	![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260311104106.png)
## 推导
> [!Definition] 推导
> - 将待处理的串中的某个非终结符号替换为这个非终结符号的某个产生式的体 
> - 从开始符号出发，不断进行上面的替换，就可以得到文法的不同句型

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260311104523.png)

**推导**的正式定义：
- 如果 $A\to\gamma$ 是一个产生式，那么 $\alpha A\beta\implies\alpha\gamma \beta$
- **最左/右推导**(leftmost/rightmost derivation): $\alpha/\beta$ 中不包含非终结符号
	- 符号：
$$
\implies_{lm}^{*},\implies_{rm}^{*}
$$
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260311104824.png)

## 句型/句子/语言
- 句型（sentential form）
	- if $S\implies^{*}\alpha$, then $\alpha$ is the sentential form of grammar $S$
- 句子 (sentence)
	- the *sentence* of *grammar* is the sentential form without *non-terminal sign*
- 语言
	- the *language* of grammar $G$ is the *set* of sentences of $G$, that is $L(G)$
	- $w\in L(G)\iff w\text{ is sentence of }G,$，that is $S\implies^{*}w$


## 语法分析树
- 是什么？
	- **推导**的图形表示形式
	- 根结点的标号是文法的 *开始符号*
	- 每个 *叶子结点* 的标号是非终结符号、中间符号或 $\varepsilon$
	- 每个 *内部结点* 的标号是非终结符号
	- 每个 *内部结点* 表示某个产生式的一次**应用**
		- 结点的标号为产生式头，其子结点从左到右是产生式的体
- 树的叶子组成的 *序列* 是根的文法符号的一个句型
- 一棵语法分析树可对应多个推导序列
	- 但只有 *唯一的最左推导及最右推导*

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260311110025.png)

推导仅仅是字符串替换，不要想更多的信息（知识的诅咒）

在这个意义下产生了二义性

## 二义性
> [!Definition] 二义性 (ambiguity)
> 如果一个文法可以为某个句子生成 *多棵* 语法分析树，这个 *文法* 就是二义的

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260311111526.png)
- 现在没有优先级的**信息**
- 二义性和程序设计的文法原则矛盾（通常无二义），所以我们需要规定计算的优先级
	- 避免一个程序有多种“正确”的解释
- 我们用 $term,expression,factor$ 的层次来解决

## 词法分析和语法分析的比较
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260311112350.png)

## 上下文无关文法和正则表达式
- 上下文无关文法比正则表达式的能力 *更强*
	- 正则语言是文法描述语言的子集

> [!Question] Proof to 一些用文法描述的语言不能用正则表达式描述
> 首先 $S\to aSb |ab$ 描述了语言 $L\;\{ a^{n}b^{n}|n>0 \}$，但这个语言无法用 DFA 识别
> - 假设有 DFA 识别此语言 $L$，且这个 DFA 有 $k$ 个状态。那么在识别 $a^{k+1}\dots$ 的输入串时，必然两次到达同一个状态（**鸽巢原理**）。假设自动机在第 $i$ 个 $a$ 和第 $j$ 个 $a$ 时进入同一个状态，那么：因为 DFA 识别 $L$，$a^{j}b^{j}$ 必然到达接受状态，因此 $a^{i}b^{j}$ 必然也到达接受状态
> - 直观地说：**有穷自动机**不能计数

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260311114057.png)


## 文法及其生成的语言
- **语言**是从文法的开始符号出发，能推导到的所有句子的集合
	- 文法 $G: S\to aS\;|\;a\;|\;b,\;L(G)=\{ a^{i}(a|b),i\ge 0 \}$
	- 文法 $G$：$S\to aSb\;|\;ab,L(G)=\{ a^{n}b^{n},n\geq 1 \}$
	- 文法 $G$: $S\to(S)S\;|\;\varepsilon$，$L(G)=\{ 所有 \}$
- 如何 *验证* 文法 $G$ 确定的语言 $L$？
	- 证明 $G$ 生成的每个串都在 $L$ 中
	- 证明 $L$ 的每个串都能被 $L$ 生成


## 设计文法
语法分析器接受的语言是程序设计语言的**[[集合#超集|超集]]**；必须通过语义分析来剔除一些符合文法、但不合法的程序

- **文法处理**
	- 消除 *二义性*
		- 二义性：文法可以为句子生成多棵不同的分析树
	- 消除 *左递归*
		- **左递归**：文法中的一个非终结符号 $A$ 使得对某个串 $\alpha$，存在一个推导 $A\implies^{+}A\alpha$，则称这个文法是左递归的
		- Note: $+$ is on the $\implies$
		- 无限递归否则
	- 提取 *左公因子*

### 左递归(left recursive)的消除
> [!Definition] 左递归 (left recursive)
>  **左递归**：文法中的一个非终结符号 $A$ 使得对某个串 $\alpha$，存在一个推导 $A\implies^{+}A\alpha$，则称这个文法是左递归的

> [!Note] 立即左递归
> 文法中存在一个形如 $A\to A\alpha$ 的产生式


自顶向下无法解决左递归，需要消除。但是自底向上的技术可以处理左递归

- 多做题不是坏事
```
p105 3.7.1 NFA->DFA,(2)
chapter 4
p130 4.2.1 给最左推导最右推导,去掉第四小节二义性的证明，第五小题可以做一下(去掉123)
```

