# 第四章-语法分析

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
### 推导
> [!Definition] 推导
> - 将待处理的串中的某个非终结符号替换为这个非终结符号的某个产生式的体 
> - 从开始符号出发，不断进行上面的替换，就可以得到文法的不同句型

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260311104523.png)

**推导**的正式定义：
- 如果 $A\to\gamma$ 是一个产生式，那么 $\alpha A\beta\implies\alpha\gamma \beta$
- **最左/右推导**(leftmost/rightmost derivation): 在上面推导的定义基础上, $\alpha/\beta$ 中不包含非终结符号
	- **最左推导**(leftmost derivation)：总是选择最左侧的非终结符号替换                                                                                                                                                                                                                                                                                                                                                                                             
	- **最右推导**（rightmost derivation）：总是选择最右侧的非终结符号替换
	- 符号：
$$
\implies_{lm}^{*},\implies_{rm}^{*}
$$
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260311104824.png)
$\alpha,\beta$ 是任意的文法符号串。
### 句型/句子/语言
- 句型（sentential form）
	- if $S\implies^{*}\alpha$, then $\alpha$ is the sentential form of grammar $S$
- 句子 (sentence)
	- the *sentence* of *grammar* is the sentential form without *non-terminal sign*
- 语言
	- the *language* of grammar $G$ is the *set* of sentences of $G$, that is $L(G)$
	- $w\in L(G)\iff w\text{ is sentence of }G,$，that is $S\implies^{*}w$


### 语法分析树
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

### 二义性
> [!Definition] 二义性 (ambiguity)
> 如果一个文法可以为某个句子生成 *多棵* 语法分析树，这个 *文法* 就是二义的

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260311111526.png)
- 现在没有优先级的**信息**
- 二义性和程序设计的文法原则矛盾（通常无二义），所以我们需要规定计算的优先级
	- 避免一个程序有多种“正确”的解释
- 我们用 $term,expression,factor$ 的层次来解决

### 词法分析和语法分析的比较
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260311112350.png)

### 上下文无关文法和正则表达式
- 上下文无关文法比正则表达式的能力 *更强*
	- 正则语言是文法描述语言的子集

> [!Question] Proof to 一些用文法描述的语言不能用正则表达式描述
> 首先 $S\to aSb |ab$ 描述了语言 $L\;\{ a^{n}b^{n}|n>0 \}$，但这个语言无法用 DFA 识别
> - 假设有 DFA 识别此语言 $L$，且这个 DFA 有 $k$ 个状态。那么在识别 $a^{k+1}\dots$ 的输入串时，必然两次到达同一个状态（**鸽巢原理**）。假设自动机在第 $i$ 个 $a$ 和第 $j$ 个 $a$ 时进入同一个状态，那么：因为 DFA 识别 $L$，$a^{j}b^{j}$ 必然到达接受状态，因此 $a^{i}b^{j}$ 必然也到达接受状态
> - 直观地说：**有穷自动机**不能计数

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260311114057.png)


### 文法及其生成的语言
- **语言**是从文法的开始符号出发，能推导到的所有句子的集合
	- 文法 $G: S\to aS\;|\;a\;|\;b,\;L(G)=\{ a^{i}(a|b),i\ge 0 \}$
	- 文法 $G$：$S\to aSb\;|\;ab,L(G)=\{ a^{n}b^{n},n\geq 1 \}$
	- 文法 $G$: $S\to(S)S\;|\;\varepsilon$，$L(G)=\{ 所有 \}$
- 如何 *验证* 文法 $G$ 确定的语言 $L$？
	- 证明 $G$ 生成的每个串都在 $L$ 中
	- 证明 $L$ 的每个串都能被 $L$ 生成


### 设计文法
语法分析器接受的语言是程序设计语言的[[集合#超集|超集]]；必须通过语义分析来剔除一些符合文法、但不合法的程序

- **文法处理**
	- 消除 *二义性*
		- 二义性：文法可以为句子生成多棵不同的分析树
		- `if, else, other`
	- 消除 *左递归*
		- **左递归**：文法中的一个非终结符号 $A$ 使得对某个串 $\alpha$，存在一个推导 $A\implies^{+}A\alpha$，则称这个文法是左递归的
		- Note: $+$ is on the $\implies$
		- 无限递归否则
	- 提取 *左公因子*

#### 左递归 (left recursive) 的消除
> [!Definition] 左递归 (left recursive)
>  **左递归**：文法中的一个非终结符号 $A$ 使得对某个串 $\alpha$，存在一个推导 $A\implies^{+}A\alpha$，则称这个文法是左递归的

> [!Note] 立即左递归
> 文法中存在一个形如 $A\to A\alpha$ 的产生式


自顶向下无法解决左递归，需要消除。但是自底向上的技术可以处理左递归

假设非终结符号 $A$ 存在 ==立即左递归==的情况
$$
A\to A\alpha_{1}|\dots|A\alpha_{m}|\beta_{1}|\dots|\beta_{n}
$$
可替换为
$$
\begin{align}
A\to \beta_{1}A'\;|\dots\;|\beta_{n}A' \\
A'\to \alpha_{1}A'|\dots|\;\alpha_{m}A'\;|\varepsilon
\end{align}
$$
- 本质是从后往前拼接改成了从前往后拼接
- 利用了中间 $A'$

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260316103743.png)


- 多做题不是坏事
```
p105 3.7.1 NFA->DFA,(2)
chapter 4
p130 4.2.1 给最左推导最右推导,去掉第四小节二义性的证明，第五小题可以做一下(去掉123)
```

### 预测分析法
- 假设文法有一个很好的性质：每一个产生式的第一个终结符号都不一样
- 试图从开始符号推导出输入符号串
- 文法：$E\to +EE\;|-EE\;|\;id$,
- 输入: $+id-id \;id$
- 如果两个产生式具有相同前缀时无法预测，我们可以提取公因子
	- old: `stmt -> if expr then stmt else stmt | if expr then stmt`
	- new:
		- `stmt -> if expr then stmt elsePart`
		- `elsePart -> else stmt | :e`

### 提取公因子的文法变换
- 算法
	- input: 文法 $G$
	- output: 等价的提取了左公因子的文法
	- 方法：对于每个非终结符号 $A$，找出它的两个或者多个可选产生式体之间的 **最长公共前缀**

## 自顶向下的语法分析
- 为输入串构造语法分析树
	- 从分析树的根结点开始，按照 *先根次序* ，深度优先地创建各个节点
	- 对应于最左推导
- **基本步骤**
	- 确定对句型中 **最左边** 的非终结符号应用哪个产生式
	- 然后对该产生式与输入符号进行匹配
### 递归下降的语法分析
- 每个非终结符号对应于一个**过程**（某个函数？），该过程负责扫描此非终结符号对应的**结构**
- 程序执行从开始符号对应的过程开始
	- 当扫描整个输入串时宣布分析成功完成


```c
void A(){
	选择一个A产生式, A->X1X2...Xk
	for( i = 1 to k){
		if(A is a non-terminal sign)
			call Xi();
		else if (Xi equal to current input sign a) 
			read-in next input sign;
		else /* a error occurs */
	}
}
```


递归下降分析技术存在**回溯**技术。
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260316111754.png)

- 如果没有足够的信息来唯一确定可能的产生式
- **都加回溯**，可以穷尽所有的可能吗？
	- **不可以**。（是不完备的）
	- 只能发现当前的**匹配错误**。如果是之前表面上匹配上了（但其实没有！），想要回溯到多步之前是无法实现的

### FIRST 和 FOLLOW
- 在自顶向下的分析技术中使用向前看几个符号确定产生式

- 当前句型是 $xA\beta$，而输入是 $xa\dots$，那么选择产生式 $A\to\alpha$ 的必要条件是下列之一：
	- $\alpha\implies^{*}a\dots$
	- $a\implies^{*}\varepsilon$, 且 $\beta$ 以 $a$ 开头，即在某个句型中，$a$ 跟在 $A$ 之后
	- 如果按照这两个条件选择时能保证唯一性，就可以避免回溯
我们定义 FIRST 和 FOLLOW

### FIRST
- $\mathrm{FIRST}(a)$
	- 可以从 $\alpha$ 推导得到串的 *首符号* 的集合
	- 如果 $\alpha \implies^{*}\varepsilon$，那么 $\varepsilon$ 也在 $\mathrm{FIRST}(\alpha)$ 中
- $\mathrm{FIRST}$ 函数的意义
	- $A$ 的产生式 $A\to\alpha|\beta$，且 $\mathrm{FIRST(\alpha)}$ 和 $\mathrm{FIRST}(\beta)$ *不相交*
	- 下一个输入符号是 $a$，且 $a\in \mathrm{FIRST}(\alpha)$, 则选择 $A\to\alpha$; 若 $a\in \mathrm{FIRST}(\beta)$, 则选择 $A\to \beta$
	

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260316114041.png)
- 如果是非终结符号，就逐层迭代

### FOLLOW
- $\mathrm{FOLLOW}(A)$
	- *可能*在某些句型中紧跟在 $A$ 右边的终结符号的集合
	- 例如：$S\to\alpha Aa\beta$，终结符号 $a\in \mathrm{FOLLOW}(A)$
	- $\mathrm{FOLLOW}()$ 是不存在 $\varepsilon$ 的。
- $\mathrm{FOLLOW}$ 函数的意义
	- 如果 $A\to\alpha$, 当 $\alpha\to\varepsilon$ 或 $\alpha\implies\varepsilon$ 的时候, $\mathrm{FOLLOW}(A)$ 可以帮助我们选择恰当的产生式
	- 例如：$A\to\alpha$, 而 $b\in \mathrm{FOLLOW}(A)$, 如果 $a\implies\varepsilon$, 而当前输入符号是 $b$，则可以选择 $A\to a$，因为 $A$ 最终到达了 $\varepsilon$，而且后面跟着 $b$
- 计算方法
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260316114631.png)
- 首先要加入**右端结束标记**`$`

### LL (1) 文法
- **定义**：

举例：文法

$A\to\alpha,A\to \beta$
- if $a\not=> \varepsilon,\beta\not=>\varepsilon$
	- $a\in \mathrm{FIRST}(\alpha)$, then $\alpha$
	- $a \in \mathrm{FIRST}(\beta)$, then $\beta$
 - if $a\not= >\varepsilon,\beta\implies\varepsilon$
	 - $a\in \mathrm{FIRST}(\alpha)$, then $\alpha$
	 - $a\in \mathrm{FIRST}(\beta)or{\{ \varepsilon \}\cup \mathrm{FOLLOW}(A)}$ then $\alpha$
- **直观理解**：
	- 如果能推出空串，则可消失，所以可以选择 $A$ 的 FOLLOW
	- 不能两个文法都推出空串，否则不知道选谁了
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260316115944.png)

- 不存在终结符号 $a$ 使得 $\alpha$ 和 $\beta$ 都可推导出以 $a$ 开头的串 (condition 1)
- $\alpha$ 和 $\beta$ 最多只有一个可推导出空串 (condition 2)
- 如果 $\beta$ 可推导出空串，那么 $\alpha$ 不能推导出以 $\mathrm{FOLLOW}(A)$ 中任何终结符号开头的串 (condition 3)

等价于
- $\mathrm{FIRST}(\alpha)\cap \mathrm{FIRST}(\beta)=\emptyset$ (涵盖了 1 和 2)
- 如果 $\varepsilon \in \mathrm{FIRST}(\beta)$, 那么 $\mathrm{FIRST}(\alpha)\cap \mathrm{FOLLOW}(A)=\emptyset$ ；反之亦然（条件三）

**预测分析法例子**
一个 LL（1）文法应用的例子。
可以通过下一个输入符号确定选择的产生式。即如果是输入符号属于 $\mathrm{FOLLOW}(a)$ 集合的, 选择 $\varepsilon$

- 预测分析表制作时候，可能出现 *冲突*。这是因为 LL (1) 文法要求无二义性，但是该文法存在二义性。

 - Algorithm
	 - ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260318104158.png)


### 非递归的预测分析
相比于[[#递归下降的语法分析]]，我们在这部分通过查预测分析表，实现了无递归的分析。

- 分析时的处理过程
	- 初始化时，栈中仅包含开始符号 $S$ 和 $\$$
	- 如果栈顶元素是终结符号，那么进行匹配
	- 如果栈顶元素是非终结符号
		- 使用预测分析表选择适当的产生式
		- 在栈顶用产生式右部替换产生式左部
- 所有文法的预测分析都可以用 *同样* 的驱动程序

![image.png|500](https://kold.oss-cn-shanghai.aliyuncs.com/20260318105133.png)
预测分析算法
- 输入：串 $w$，预测分析表 $M$
- 输出：如果 $w$ 是句子，输出 $w$ 的*最左推导*；否则报错
	- **初始化**：输入缓冲区中为 $w\$$，栈中为 $S\$$，$ip$, 指向 $w$ 的第一个符号
	- 令 $X=$ 栈顶符号，$ip$ 指向输入符号 $a$
		- if $(X ==a)$，$X$ 出栈，$ip$ 向前移动 // 与终结符号匹配成功
		- else if $(X是终结符号)$ error () // 失配
		- else if $(M[X,a])是报错条目$ error () // 无适当的产生式
		- else if $(M[X,a]=X\to Y_{1}Y_{2}\dots Y_{k})${
			输出产生式 $X\to Y_{1}Y_{2}\dots Y_{k}$
			弹出栈顶符号 $X$，并将 $Y_{k},Y_{k-1},\dots,Y_{1}$ 压入栈中
		- }
	- 不断执行第二步，直到要么报错，要么栈中为空

## 自底向上的语法分析
- 为一个输入串构造语法分析树的过程
- 重要的自底向上语法分析的通用框架
	- *移入*-*归约*（shift-reduce）
- 简单 LR 技术（SLR），LR 技术（LR）


#### 移入和归约
- 自底向上的语法分析过程可以看成是从串 $w$ 归约为文法开始符号 $S$ 的过程
- 归约 (reduction) 步骤 
	- 一个与某产生式体相匹配的特定子串被替换为该产生式头部的非终结符号

- **移入**是压栈，**归约**是弹出
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260318113708.png)


- How do we decide when to shift or reduce?
- Not always *reduce* when we see the stack-top element that is can be reduced
	- A Simple and correct **insight** is to do *reduce*, when the result of *reduce* can be reduced, too
	![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260324152406.png)

	- Then $\alpha \beta$ is a *handle* of $\alpha \beta \omega$
	- means it's OK to reduce beta to X, we can still somehow get to the start symbol by such a *reduction*


#### 句柄（handle）

> [!Question] What do handle do?
> 句柄的作用是将关于在何处进行规约的直觉概念加以形式化
> So handles formalize the intuition about where it is okay to do a reduction


- 如果一个符号串是从**开始符号**出发，通过一系列“最右推导”得到的，那么这个符号串就叫最右句型
- 最右句型 (Right-sentential form) 中和**某个产生式体**相匹配的子串，对它的归约代表了该最右句型的最右推导的最后一步
- **正式定义**：如果 $S\implies_{rm}^{*}\alpha Aw\implies_{rm}\alpha \beta w$，那么紧跟 $\alpha$ 之后的 $\beta$ 是 $A\to \beta$ 的一个句柄
	- 一个最右句型中，句柄右边只有终结符号
- A handle is a *reduction* that also allows further reductions back to the start symbol
	- We only want to reduce at handles
	- 一个自然的想法是，handles 总是也只是在栈顶出现



> [!Quesition] *为什么句柄总是在栈顶*
>  ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260318113548.png)
**有两种情况**：
- 内嵌的情况
- 并列的情况

- In shift-reduce parsing, handles always appear at the top of the *stack*
	- 在初始状态下，栈为空，所以我们规约也一定是栈顶元素
	- 完成规约之后，栈顶一定是一个*非终结符*（non-terminal）
	- **在最右推导**中，我们每次总是替换最右边的非终结符，shift-reduce 作为最右推导的逆过程，下一个**句柄**handle 一定是在 right-most 的 non-terminal 的 right。
	- 经过一系列的 shift  moves 操作，我们到达下一个**句柄**handle
- 采用栈这个结构是合理的。一路规约，句柄总是在栈顶
- Handles are never to the left of the rightmost non-terminal
	- Therefore, shift-reduce moves are sufficient; the `|` need never move left


Bottom-up **parsing** is based on the recognition of *handles*, but it's hard.

## LR 语法分析技术
- $\mathrm{LR}(k)$ 的语法分析概念
	- L 表示从左到右扫描输入，R 代表构造 *最右*推导的逆过程

- It can help **Recognizing Handles**

### LR (0) Parsing: Assume
- stack contains $\alpha$
- next input is $t$
- DFA on input $\alpha$ terminates in state $s$

- **Reduce by $X\to \beta$ if**
	- s contains item $X\to \beta$
- **Shift if**
	- s contains item $X\to \beta.t\omega$
		- it's ok to shift $t$ in
	- equivalent to saying s has a transition labeled $t$



### LR (0) 项
- 项（item）：文法的一个产生式加上在其中某处的一个**点** $\cdot$
	- e.g. The items for $T\to(E)$ are
		- $T\to.(E)$
		- $T\to(.E)$
		- $T\to(E.)$
		- $T\to(E).$
	- extra. The only item for $X\to\varepsilon$ is $X\to.$
- An item is production with a "`.`" somewhere on the *rhs*
- Marker, express where we have read and what we are expecting

- 每一个带有点的产生式称为**项**（Item）
- The problem in recognizing viable prefixes is that the stack has only bits and pieces of the rhs of productions
	- If it had a complete rhs, we could reduce
- 在任何成功的分析中，**堆栈**上的内容必须是某些产生式右侧的前缀
- Item $T\to(E.)$ says that so far we have seen $(E$ of  this production and hope to see $)$

### LR (0) 项集规范族的构造
- 三个相关定义
	- 增广文法
		- *我们需要一个开始和结束*
		- Add a dummy production $S'\to \cdot S$ to $G$
	- 项集闭包 CLOSURE
		- *设计状态*
	- GOTO 函数
		- *设计状态转移*
- **增广文法**（augmented grammar）
	- 在文法 $G$ 的基础上，增加新开始符号 $S'$，并加入产生式 $S'\to S$
	

### 项集闭包 CLOSURE
- 如果 $I$ 是文法 $G$ 的一个项集，$\mathrm{CLOSURE(I)}$ 就是这么构造的：
	- 将 $I$ 的各项加入 $\mathrm{CLOSURE}(I)$
	- 如果 $A\to \alpha \cdot B\beta$ 在 $\mathrm{CLOSURE}(I)$ 中，而 $B\to \gamma$ 是一个产生式，且项 $B\to \cdot \gamma$ 不在 $\mathrm{CLOSURE}(I)$ 中，就将该项加入其中，不断应用该规则直到没有新项可加入
- **意义**
	- $A\to\alpha \cdot B\beta$，表示希望看到由 $B\beta$ 推导出的串，那要先看到由 $B$ 推导出的串，因此加上 $B$ 的各个产生式对应的项
	- Closure is for, except what we have seen's generation expression, what else maybe matched
	

e.g
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260323103958.png)

### GOTO 函数

- GOTO 函数
	- $I$ 是一个项集，$X$ 是一个*文法符号*，则 $\mathrm{GOTO}(I,X)$ 定义为 $I$ 中所有形如 $\left[ A\to\alpha \cdot X\beta \right]$ 所对应的项 $\left[ A\to\alpha X\cdot \beta \right]$ 的集合的闭包（CLOSURE）


### LR (0) 自动机的构造
- **构造方法**
	- 基于规范 LR (0) 项集族可以构造 $\mathrm{LR(0)}$ 自动机
	- 规范 LR（0）项集族中的每个 *项集* 对应于 LR（0）自动机的一个状态
	- **状态转换**：如果 $\mathrm{GOTO}(I,X)=J$, 则从 $I$ 到 $J$ 有一个标号为 $X$ 的转换
	- **开始状态**为 $\mathrm{CLOSURE}(\{ [S'\to \cdot S] \})$ 对应的项集

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260323111150.png)

### LR 语法分析表的结构
- **两个部分**：动作 ACTION 、转换 GOTO
- ACTION 表项有两个参数：状态 $i$，终结符号 $a$
	- 移入 $j$：$j$ 是新状态，把 $j$ 压入栈（同时移入 $a$）
	- 归约 $A\to \beta$：把栈顶的 $\beta$ 归约为 $A$（并根据 GOTO 表项压入新状态）
	- 接受：接受输入，完成分析
	- 报错：在输入中发现语法错误
- GOTO 表项
	- 如果 $\mathrm{GOTO}[I_{i},A]=I_{j}$, then $\mathrm{GOTO}[i,A]=j$
	- GOTO 函数和 GOTO 表项
	- $I_{i}$ 是一个项集，$i$ 代表一个状态

### LR 语法分析器的格局
- LR 语法分析器的*格局*(configuration) 包含了*栈中内容*和*余下输入* $(s_{0}s_{1}\dots s_{m},a_{i}a_{i+1}\dots a_{n}\$)$
	- 第一个分量是栈中的内容 (right is stack top)
	- 第二个分量是余下的输入
- LR 语法分析器的每一个状态都对应一个文法符号（$s_{0}$ 除外）
	- 如果进入状态 $s$ 的边的标号为符号 $X$，那么 $s$ 就对应于 $X$
	- 令 $X_{i}$ 为 $s_{i}$ 对应的符号，那么 $X_{1}X_{2}\dots X_{m}a_{i}a_{i+1}\dots a_{n}$ **对应于一个最右句型**(right-most sentential)

### LR 语法分析算法
- input： 文法 $G$ 的 LR 语法分析表，输入串 $w$
- output:

### LR 分析表的例子

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260323112832.png)


表中的标号是根据文法的序号的意思。
GOTO 查询下一个状态，维护栈
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260323113242.png)

## SLR 语法分析技术
SLR = "Simple LR"
- SLR improves on LR (0) shift/reduce heuristics
	-  Fewer states have conflicts


Compared with [[#LR (0) Parsing Assume]], we add a condition to decide when to reduce:

- stack contains $\alpha$
- next input is $t$
- DFA on input $\alpha$ terminates in state $s$

- **Reduce by $X\to \beta$ if**
	- s contains item $X\to \beta$
	- **$t \in Follow(X)$**

- **Shift if**
	- s contains item $X\to \beta.t\omega$
		- it's ok to shift $t$ in
	- equivalent to saying s has a transition labeled $t$
### SLR 语法分析表的构造
- 以 LR (0) 自动机为基础的 SLR 语法分析表构造算法
	- 构造增广文法 $G'$ 的 LR (0) 项集规范族 $\{ I_{0},I_{1},\dots,I_{n} \}$
	- 状态 $i$ 对应项集 $I_{i}$, ACTION 和 GOTO 条目如下：
		- $[A\to\alpha \cdot a\beta]$ 在 $I_{i}$ 中，且 $\mathrm{GOTO}(I_{i},a)=I_{j}$，则 $\mathrm{ACTION}[i,a]=\text{移入}j$
			- **点在终结符之前**
		- $[A\to\alpha \cdot]$ 在 $I_{i}$ 中，那么对 $\mathrm{FOLLOW}(A)$ 中所有 $a$，$\mathrm{ACTION}[i,a]=\text{按}A\to\alpha归约$
			- **点在末尾**
		- 如果 $[S'\to S\cdot]$ 在 $I_{i}$ 中，那么将 $\mathrm{ACTION}[i,\$]$ 设为“**接受**”
	- 空白的条目设为"error"

移入-归约冲突
归约-归约冲突

### 可行前缀 viable prefix
- LR (0) 自动机刻画了可能出现在 shift-reduce 语法分析栈中的文法符号串
- **可行前缀**(viable prefix)
	- 可以出现在语法分析器栈中的**最右句型**的前缀，且没有越过该句型的句柄的*右端*
### 有效项
- 有效项 (valid item)
	- 如果存在一个 right-most 推导过程 $S$ 到 $\alpha Aw\implies\alpha \beta_{1}\beta_{2}w$ 那么我们说项 $A\to \beta_{1}\cdot \beta_{2}$ 是可行前缀 $\alpha \beta_{1}$ 的有效项
	- After parsing $\alpha \beta_{1}$，the valid items are the possible tops of the stack of items
	- **一个更直观的说法**：An item $I$ is valid for a viable prefix $\alpha$ if the DFA recognizing viable prefixes terminates on input $\alpha$ in a state $s$ containing $I$
	- The items in $s$ describe what the top of the item stack might be after reading input $\alpha$

***

## LR (1) 语法分析技术
### LR （1）
包含更多信息来消除一些归约动作。
- LR (1) 项的形式：$[A\to\alpha \cdot \beta,a]$
	- $a$ 称为向前看符号，可以是终结符号或者 $\$$
	- $a$ 表示如果将来要按照 $A\to\alpha \beta$ 进行归约，归约的下一个符号是 $a$
	- 可以理解为跟在上一步归约的**非终结符**后面的东西，而 $\cdot$ 表示了我们的归约看到**哪**里了
- **增广文法**的改造
	- $\left[ S'\to \cdot S,\$ \right]$
- CLOSURE 的改造
	- if have $\left[ A\to\alpha \cdot B\beta,a \right]$, and we have reduction $B\to \theta$
	- then $\left[ B\to \cdot \theta,\mathrm{FIRST}(\beta a) \right]$
		- 当 $\mathrm{FIRST}(\beta)= \{ \varepsilon \}$, 则选择 $a$ 作为向前看符号
		- 否则选择 $\mathrm{FISRT(\beta)}$

### LR (1) 的 GOTO 算法
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260325104846.png)
### LR (1) 项集族的构造算法
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260325104907.png)

### LR (1) 语法分析表的构造

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260325105749.png)

## LALR (向前看 ) 语法分析
- **基本思想**
	- 寻找具有**相同核心的 LR (1) 项集**，并把它们合并称为一个项集
> [!Question] 什么是核心?
> - **核心**(core): 项的第一分量的集合
- 一个 $\mathrm{LR(1)}$ 项集的**核心**(core) 是一个 $\mathrm{LR(0)}$ 项集
- $\mathrm{GOTO}(I,X)$ 的核心只由 $I$ 的核心决定，因此呗合并项集的 $\mathrm{GOTO}$ 目标也可以合并
	- 核心怎么找呢?
- **合并不会引起** shift/reduce conflict
	- if so, you can prove before merge, there's conflict
- 会引起 reduce/reduce conflict


- **LALR**技术本质
	- 对 LR（1）项集规范族中的同核心项集进行合并
	- 保持向前看信息
	- 状态数减少

### LALR (Look-Ahead LR) 分析表构造算法

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260325112119.png)


## 二义性文法的使用
### 优先性/结合性消除冲突
- **二义性文法**
	- $E\to E+E\; |E*E\; |(E)\; |id$

- **结合性**是在同优先级情况下，从哪往哪算的问题。

- 二义性文法容易修改算符的优先级和结合性
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260330111531.png)
- When stack top state is 7, it shows:
	- 栈中状态序列对应的文法符号序列为: $\dots E+E$
	- if next sign is `+` or `*` shift or reduce?
		- if next sign is `*`, we shift `*` (higher previlege)
		- if next sign is `+`, we reduce `E+E` to `E`
- 通过规定优先级决定规约移入规则