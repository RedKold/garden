# 0. Intro
基于南京大学课件整理。

主要领域：
- 逻辑与证明
	- 命题逻辑、谓词逻辑、推理与证明方式
- 计数技术与离散概率
	- 组合计数、递推、离散概率
- 离散结构
	- 集合、图、树
- 代数系统
	- 群论、格、布尔代数

# 1.逻辑和证明

## 命题逻辑
> [!Note] **命题**
> 命题是一个陈述语句，即一个陈述事实的句子
> - 要么真，要么假
> - 不能既真又假

- ‘你会说英语吗’ -> 并非命题

> [!Note] 命题变元
> - 常用小写字母表示， $p,q,r$
> - **取值范围**： $\{ T,F \},\{ 1,0 \}$


- 原子命题和复合命题
	- 复合命题是否为真取决于作为复合成分的子命题的真假。

> [!Note] 命题表达式（命题逻辑公式）
> - 命题变元是命题表达式；
> - 若 $p$ 是命题表达式，则 $(\lnot p)$ 也是
> - 若 $p$ 和 $q$ 是命题表达式，则 $(p\land q),(p\lor q),(p\leftrightarrow q)$ 也是
> - 只有有限次应用上述规则形成的符号串才是命题表达式
> - **优先级**: $\lnot,\land,\lor,\to,\leftrightarrow$

- 将*自然语言*翻译为命题表达式
	- 只有你主修计算机科学或不是新生，才可以从校园网访问因特网
		- $a$: 你可以从校园网访问因特网
		- $c$：你主修计算机科学
		- $f$: 你是新生
	- $a\to(c\lor \lnot f)$

> [!Note] **逻辑等价**
> $p$ 和 $q$ 逻辑等假：在所有可能下 $p$ 和 $q$ 真值相同
> - that is $p\leftrightarrow q$  is always true
> - $p\equiv q$


> [!Question] 戴帽子
> - A，B，C 3 个人，$C$ 蒙眼，3 个人各带一个帽子，帽子颜色 $\in \{ 白，黑 \}$，但是不是全是白色的。$A$ 看了看 $B$ 和 $C$ 说无法确定自己帽子颜色，$B$ 看了看 $A$ 和 $C$ 说也不能确定自己头上帽子的颜色，这时候 $C$ 说他知道自己帽子的颜色了。请问 $C$ 帽子的颜色
> -  ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260805212550.png)


- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260805213028.png)

- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260805212954.png)

a useful equivalent logic:
$$
\lnot(p\to q)\implies \lnot(\lnot p \lor q)\implies p \land(\lnot q)
$$
- only `1->0` is false



## 命题逻辑公式的范式
- some definition
	- 命题**变元**或命题**变元的否定**称为**文字**
	- 有限个文字的析取式称为简单析取式（基本和）
	- 有限个文字的合取式称为简单合取式（基本积）
- 由有限个简单合取式构成的析取式称为**析取范式**
	- (DNF, Disjunctive Normal From)
- 由有限个简单析取式构成的合取式称为**合取范式**
	- (CNF, Conjunctive Normal From)
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260805214457.png)


- **性质**
	- 一个文字既是 DNF 又是 CNF

> [!Note] 极小项和极大项
> - 包含所有命题变元或其否定一次仅一次的简单合取式，称为**极小项**
> - 包含所有命题变元或其否定一次仅一次的简单析取式，称为**极大项**
> - 由有限个极小项组合的析取范式称为*主析取范式*
> - 由有限个极大项组合的合取范式称为*主合取范式*

- 没有任意两个极小项、**极大项是等价的**！
	- 可以唯一的用二进制编码！

- **可以用真值表**求出主析取范式和主合取范式
	- 本质是在不同 binary input 下映射出的真值 value，每一个 binary input are related with a smallest *items*

**推理问题**
- 语义蕴含 (Semantic entailment)
	- Given premises $\mathrm{\phi_{1},\phi_{2}},\dots,\phi_{n}$, we hold conclusion $\phi$ 
- $\phi_{1},\phi_{2},\dots,\phi _n |=\phi$ 
- **可满足**？
	- $\phi$ is satisfiable iff $\lnot\phi$ is not *valid*

> [!Note] **论证**
> An *argument* in propositional logic is a sequence of propositions, All but the final proposition in the argument are called *premises* and the final proposition is called the *conclusion*. An argument is *valid* if the truth of all its premises implies that teh conclusion is true

论证形式：
```
p -> q
p
----
so q
```

## **命题逻辑的自然演绎规则**
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260805215608.png)

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260805224209.png)


## 谓词逻辑
> [!Note] **谓词**（Predicate）
> - if $x$ is integer, $x >2$ is not a *proposition*, its true value depends on the value of $x$
> - you can express $x>2$ as $P (x)$
> **一元谓词**$P$：陈述 $P(x)$ 看作命题函数 $P$ 在 $x$ 的一个值
> - $P$ 的定义域是整数集
> - $P(3)$ 是一个取值为 $T$ 的命题
> - $P(1)$ 是一个取值为 $F$ 的命题


> [!Note] **量词**(Quantifier)
> - 全称量词 $\forall$, Universal Quantifier
> - 存在量词 $\exists$, Existential Quantifier


- 否定式
	- $\lnot \forall xP(x)\equiv \exists x \lnot P(x)$
	- $\lnot \exists xP(x)\equiv \forall x \lnot P(x)$

- **多量词**
	- $\forall x\forall yP(x,y)\equiv \forall y\forall xP(x,y)$
	- $\exists x\exists yP(x,y)\equiv \exists y\exists xP(x,y)$
	- no others

> [!Note] 重言式
> **重言式（tautology，也叫永真式）**：一个命题公式，无论其中的命题变元取什么真值，它都恒为真。
> 
> 比如：
> 
> - $p \vee \neg p$ —— 无论 $p=0$ 还是 $p=1$，结果都是 1；
> - $p \to p$；
> - $(p \wedge q) \to p$。
> 
> **怎么判断**：列真值表，看公式在变元所有可能的取值组合下是否全部为 1。有 $n$ 个变元就有 $2^n$ 行。
> 
> **和它对应的两个概念**：
> 
> - 矛盾式（永假式）：恒为假，如 $p \wedge \neg p$；
> - 可能式（可满足式但不是重言式）：有时真有时假，如 $p$、$\neg p$。
> 
> **在考试里的用处**（课件 01–03 的重点）：
> 
> 1. 判断逻辑等价：$p \equiv q$ 当且仅当 $p \leftrightarrow q$ 是重言式；


## 与量词有关的“自然演绎”规则
- **全称例示** UI
	- $\forall xP(x)$, so $P(c)$
- **全程生成** UG
	- $P(c)$ 对任意的 $c$
	- so $\forall xP(x)$
- **存在例示** EI
	- $\exists xP(x)$
	- so $P(c)$ for some $c$
- **存在生成** EG
	- $P(c)$ 对某个 $c$
	- $\exists xP(x)$

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260806205819.png)

---
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260806210725.png)

## 证明方法
> [!Note] 定理（Theorem）
> - 能够证明为真的陈述，通常是比较重要的陈述


> [!Note] **证明**(proof)
> - 表明陈述（定理）为真的有效论证

- 定理证明中可以使用的*陈述*：
	- （当前）定理的前提
	- 已经证明的定理
	- 公理
	- 术语的定义

**反证法**：
- 原理
	- $p\to q\equiv \lnot q\to \lnot p$

**归谬法**：
- 原理
	- $q\equiv \lnot q\to F$


# 2.集合论
> [!Note] What will we talk about?
> - **基本概念**
> 	- 集合及其描述
> 	- 集合相等、子集关系
> 	- 幂集、笛卡尔乘积
> - **集合运算**
> 	- 交并补、广义交、广义并
> 	- 集合恒等式
> 	- 集合相关命题的证明方式
> - **自然数的构造**



## 集合及其运算
G. Cantor-朴素集合论
- **外延法**：罗列、枚举
	- $V=\{ a,e,i,o,u \}$
- 概括法

### 集合相等、子集关系
- **定义**：集合相等当且仅当它们有同样的元素
	- $A=B,\iff \forall x(x \in A \leftrightarrow x \in B)$
- **定义**：集合 $A$ 称为集合 $B$ 的**子集**，记作 $A \subseteq B$
	- $\forall x(x \in A \to x \in B)$
- $A\subseteq B\land B\subseteq A \iff A=B$

> [!Note] 集合的大小
> - **有限集合**及其**基数**
> 	- 若 $S$ 恰有 $n$ 个不同的元素，$n$ 是自然数，就说 $S$ 是有限集合，而 $n$ 是 $S$ 的基数
> 	- 记作 $|S|=n$
> - 无限集合
> - **空集**$\emptyset$
> 	- 空集本身可以是一个 Object，或者是某个元素的 Element
> 	- 从空集开始构造集合世界


### 幂集
**幂集 (power set)** — the set of all subsets of a given set.
**Definition:** For a set $S$, its power set, written $P(S)$ (or $2^S$), is the set whose elements are all the subsets of $S$:

$$
 P (S) = \{X \mid X \subseteq S\}
$$

**Example:** If $S = {a, b}$, then the subsets are $\emptyset$, ${a}$, ${b}$, ${a,b}$, so
$$ P (S) = \{\emptyset,\ \{a\},\ \{b\},\ \{a, b\}\} $$

**Key facts (from your handout 05):**

- If $|S| = n$, then $|P(S)| = 2^n$. Each element of $S$ independently either is or isn't in a subset, giving $2 \cdot 2 \cdots 2 = 2^n$ choices.
- $P(\emptyset) = {\emptyset}$ — note this has size 1, not 0.
- $\emptyset \in P(S)$ always, and $S \in P(S)$ always.
- The empty set is a subset of every set: $\emptyset \subseteq S$.

> [!Tip] 一个 power set 的性质
> if $\rho(A)\subseteq \rho(B)$, then $A\subseteq B$

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260806233720.png)

### 集合运算定义
- 集合运算的结果仍然是集合
- **并**：$A \cup B=\{ x \in A \lor x \in B\}$
- **交**：$A \cap B=\{ x| x \in A \land x \in B \}$
- **补**：$A-B=\{ x|x \in A \land x \not\in B \}$
- **对称差**: $A\oplus B=(A-B)\cup(B-A)$
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260806234009.png)

### 皮亚诺公理
> [!Note] 皮亚诺公理 (Peano axioms for natrual numbers)
> 1. **$0$ is a natural number.** There is a starting element, usually written $0$ (some texts use $1$).
> 2. **Every natural number has a successor.** (后继)For every $n \in N$, its successor $s(n)$ is also in $N$. So the operation $s$ is closed on $N$.
> 3. **$0$ is not the successor of any natural number.** Nothing comes "before" $0$: there is no $n$ with $s(n) = 0$.
> 4. **Different numbers have different successors (injectivity).** If $s(m) = s(n)$, then $m = n$. This prevents the sequence from folding back on itself.
> 5. **The induction axiom (principle of mathematical induction).** If a set $A \subseteq N$ contains $0$ and is closed under successor (i.e., $n \in A \implies s(n) \in A$), then $A = N$.
> 
> Written formally, the structure is a triple $(N, 0, s)$ satisfying:
> 戴德金-皮亚诺结构 
> - $0 \in N$
> - $\forall n \in N,\ s(n) \in N$
> - $\forall m, n \in N,\ s(m) = s(n) \to m = n$
> - $\forall n \in N,\ s(n) \ne 0$
> - $\forall A \subseteq N,\ (0 \in A) \wedge (\forall n \in A,\ s(n) \in A) \to A = N$ (归纳定理的表述)



- **集合**可以定义自然数
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260807221011.png)
- 我们设 $a$ 是集合，称 $a \cup \{ a \}$ 是 a 的 successor, 记为 $s(a)$，或 $a^{+}$
- 设 $A$ 是集合，若 $A$ 满足下列条件，称 $A$ 为归纳集
	- $\emptyset \in A$
	- $\forall a (a \in A \to s(a) \in A)$
	- $\mathbb{N}$ is the and-set of all induction sets
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260807221221.png)
- **自然数**上的小于关系可以用子集定义

### 自然数上的运算
- **加法**
	- $m+0=m$
	- $m+n^{+}=(m+n)^{+}$ 
- **乘法**
	- $m*0=0$
	- $m*n^{+}=m+m*n$ 
- **序关系**
	- $a\leq b \iff \exists c \in \mathbb{N}, a+c = b$


### 集合悖论
> [! Warning] 集合悖论阐述
> $A=\{ x|P(x) \}$，实际不能保证，对任意的性质 $P$，这样的定义都有意义
> - 平凡集：存在不以自己为元素的集合。
> - 包含所有平凡集的集合：$A=\{ x|x \not\in x \}$
- Russell Paradox


### 公理集合论（Axiomatic set theory）
- Zermelo-Fraenkel set theory with the axiom of Choice
	- ZFC 集合论


### 笛卡尔乘积
$$
A\times B =\{ (a,b)|a \in A \land b \in B \}
$$

## 关系及其运算

- 有序对 Ordered pair
	- $(a,b)$ is simple notation of $\{ \{ a \}, \{ a,b \} \}$
	- **一句话记忆**：${a}$ 负责"标记第一个位置"，${a,b}$ 负责"装下两个分量"，合起来让"谁在前"成为集合内在的、可判定的信息。有了它，笛卡尔积 $A\times B={(a,b)\mid a\in A,\ b\in B}$ 和二元关系（$A\times B$ 的子集）才在集合论里有严格的立足点。
- 笛卡尔积 (Cartesian Product)


> [!Note] （二元）关系
> Assume $A,B$ is set, then *relation* from $A$ to $B$ is a subset of $A\times B$
> - illustration
> 	- maybe *emptyset*
> 	- all elements are *Ordered Pair*


- **函数是一种特殊的关系**
	- $f:A\to B$
	- $R=\{ (x,f(x))\;|\; x \in A \}$
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260807223540.png)

#### 1. 定义与表示

- **关系**：$R$ 是 $A\times B$ 的子集，记 $R\subseteq A\times B$；若 $A=B$，叫"$A$ 上的关系"。
- **三种表示**：集合（列出有序对）、0-1 矩阵、有向图。
- **特殊关系**：空关系 $\emptyset$、全域关系 $E_A=A\times A$、恒等关系 $I_A={(a,a)\mid a\in A}$。

#### 2. 基本运算

|运算|定义|
|---|---|
|定义域/值域| $\operatorname{dom}R={x\mid\exists y,(x,y)\in R}$；$\operatorname{ran}R={y\mid\exists x,(x,y)\in R}$ |
|限制/像| $R\upharpoonright A={(x,y)\in R\mid x\in A}$；$R[A]=\operatorname{ran}(R\upharpoonright A)$ |
|逆| $R^{-1}={(y,x)\mid(x,y)\in R}$ |
|复合| $R_2\circ R_1={(a,c)\mid\exists b,((a,b)\in R_1\wedge(b,c)\in R_2)}$ |
|幂| $R^n=R\circ R\circ\cdots\circ R$（$n$ 次）|

**注意**：复合是先做 $R_1$ 再做 $R_2$，顺序不能反。例：$R_1={(a,a),(a,b),(b,d)}$，$R_2={(b,c),(b,d)}$，则 $R_2\circ R_1={(a,c),(a,d)}$，而 $R_1\circ R_2=\emptyset$。

常用性质：$(R_2\circ R_1)^{-1}=R_1^{-1}\circ R_2^{-1}$；复合满足结合律。

#### 3. 矩阵运算（0-1 矩阵）

- 并、交：对应位置 $\vee$、$\wedge$。
- 复合：布尔矩阵乘法 $\odot$，$(M_1\odot M_2)_{ij}=1 \iff \exists k,(M_1)_{ik}=1\wedge(M_2)_{kj}=1$。
- $M_{R_2\circ R_1}=M_{R_1}\odot M_{R_2}$；$M_{R^n}=M_R^{\odot n}$。

#### 4. 五种性质（看矩阵/图最容易）

| 性质             | 定义                                      | 矩阵特征             | 图特征       |
| -------------- | --------------------------------------- | ---------------- | --------- |
| 自反 reflexivity | $\forall a,,(a,a)\in R$                 | 主对角线全 1          | 每点有环      |
| 反自反            | $\forall a,,(a,a)\notin R$              | 主对角线全 0          | 每点无环      |
| 对称 Symmetry    | $(a,b)\in R\Rightarrow(b,a)\in R$       | 矩阵对称             | 有向边成对     |
| 反对称 anti-~     | $(a,b),(b,a)\in R\Rightarrow a=b$       | 对角线外无对称对         | 无成对反向边    |
| 传递 transitive  | $(a,b),(b,c)\in R\Rightarrow(a,c)\in R$ | $R^2\subseteq R$ | 两步可达必一步可达 |

易错点：空关系 $\emptyset$ 既对称又反对称，也传递；${(1,1),(2,2)}$ 同时对称且反对称。

#### 5. 闭包 Closure
> [!Note] 关系的闭包：一般概念
> 设 $R$ 是集合 $A$ 上的关系，$P$ 是给定的某种性质（如：自反、对称、传递），满足下列所有条件的关系 $R_{1}$ 称为 $R$ 的关于 $P$ 的闭包：
> - $R \subseteq R_{1}$
> - $R_{1}$ 满足性质 $P$
> - 如果存在集合 $A$ 上的关系 $R'$, $R'$ 满足性质 $P$ 并包含 $R$，则 $R_{1} \subseteq R'$





- 自反闭包：$r(R)=R\cup I_A$
- 对称闭包：$s(R)=R\cup R^{-1}$
- 传递闭包：$t(R)=R^*=R\cup R^2\cup R^3\cup\cdots$；有限集 $|A|=n$ 时只需到 $R^n$
- **Warshall 算法**（手算必会）：
$$
W_k[i, j]=W_{k-1}[i, j]\vee\bigl (W_{k-1}[i, k]\wedge W_{k-1}[k, j]\bigr) 
$$
	- 迭代 $k=1,2,\ldots,n$，$W_n$ 就是 $t(R)$ 的矩阵，复杂度 $O(n^3)$。
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260808214744.png)
#### 6. 等价关系（自反 + 对称 + 传递）
- 等价类 $x_R={y\mid xRy}$；商集 $A/R$。
	- **等于关系**的推广
- 所有等价类构成 $A$ 的一个**划分**；反过来，任意划分也定义一个等价关系（同一块 ⟺ 相关）。



## 函数及其运算
> [!Note] 函数 (function)的定义
> 设 $A$ 和 $B$ 是非空集合，从集合 $A$ 到 $B$ 的函数 $f$ 是对元素的一种指派，对 $A$ 的每个元素恰好指派 $B$ 的一个元素。
> 记作：$f:A\to B$
> - Well defined
> - $f:A\to B$
> - the domain of  $f$  is $A$, the codomain of $f$ is $B$
> - if $f$ designate $b$ in $B$ for $a$ in $A$, then written as $f(a)=b$. and now, we call $b$ is $a$ 's image.  and $a$ is one of $b$ 's original image
> - the image of elements in $A$ form a set called $f$ 's **range**
> - function is also called *mapping* or *transformation*

**集合定义**: 设 $F$ 为二元关系，$F$ 为函数指的是：
$$
(\forall x,y,z)(xFy \land xFz \to y=z)
$$
$$
F为函数 \leftrightarrow  (\forall x \in Dom(F)\to ( \exists!y)(xFy))
$$
- $\exists!$：exists and only one


- $B^{A}$：$A$ 到 $B$ 所有函数的集合，即 $\{ F \;|\;F:A\to B \}$

> [!Note] 子集在函数下的像
> Assume $f$ is a function from $A$ to $B$, $S$ is a subset of $A$, the image of $S$ under $f$, noted as $f(S)$, defined as below:
> - $f(S)=\{ t\;|\;\exists s \in S (t=f(s)) \}$

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260807234532.png)

### 函数性质
- **单射**(one-to-one, injection, injective function)
	- $\forall x_{1},x_{2} \in A$ if $x_{1}\neq x_{2}$, then $f(x_{1})\neq f(x_{2})$
- **满射**(surjection, surjective function, onto（映上）function)
	- $\forall y \in B, \exists x \in A$ let $f(x)=y$
	- $f(A)=B$（给映射满了）
- **双射**（one-to-one correspondence, bijection, bijective）
	- 满射+单射

### 反函数
> [!Note] if $f$ is one-to-one correspondence, then anti-function of $f$ is function from $B$ to $A$, noted as $f^{-1}$ 
> - not all function have anti-function
> - $f(a)=b$ iff $f^{-1}(b)=a$


## 集合的基数
- about how to compare the size of sets
	- can be counted
		- count then!
	- if not
		- ... cardinality
	- the cardinality of $S$ is noted as $|S|$

> [!Note] 等势关系
> - 如果存在从集合 $A$ 到集合 $B$ 的**双射**，则称集合 $A$ 与集合 $B$ 等势
> - 集合 $A$ 与 $B$ 等势：$A\approx B$
> - $A\approx B\implies$ A 和 B 中元素可以“**一一对应**”

- 找到任意一个**双射**
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260808000602.png)
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260808000820.png)
- **康托尔定理**
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260808001754.png)

# 数论初步
- 主要研究整数的性质
	- back to Euclid

### 整数集合
- $\mathbb{Z}$, Zahlen
- $\mathbb{Z}\approx \mathbb{N}$,
- 全序集
- 带余除法，同余

> [!Note] 同余 congruence modulo
$$
 a\equiv b(\mod m) \iff (\exists m \in \mathbb{Z}^{+})(m | (a-b)))
$$


> [!Note] 最大公约数 (Greatest common divisor, GCD)
$$
gcd(a,b)= \max \{ d \in \mathbb{Z}^{+} | (d|a) \land (d|b) \} 
$$
> Theorem:
> - linear composition
> 	- $(\exists s,t \in \mathbb{Z})(\gcd(a,b)=sa+tb)$
> - $\gcd(a,b)=gcd(a,b-a)$
> - $gcd(a,b)=gcd(b, a\mod b)$


### 质数
> [!Note] 算术基本定理
> 每个大于 $1$ 的整数皆可分解为有限个质数之积（这些质数称为**质因子**），若不考虑顺序则分解唯一

$$
n=p_{1}^{\alpha_{1}}p_{2}^{\alpha_{2}}\dots p_{k}^{\alpha_{k}}(p_{1}<p_{2}<\dots<p_{k}, \alpha_{i} \in \mathbb{Z}^{+})
$$


> [!Note] 定理：**质数定理**
>  $x \in \mathbb{R}^{+}$，$\pi(x)$ is prime number count function (i.e. the amount of prime number not-greater-than $x$), we hold
$$
 \lim_{ x \to \infty }  \frac{\pi(x)}{\frac{x}{\ln x}}=1
$$
- 质数的分布逐渐稀疏


- **中国剩余定理**
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260808011335.png)
- **线性同余方程组**的解存在定理

### 欧拉函数
> [!Note] **定义**
> for all $n \in \mathbb{Z}^{+}$
$$
\varphi(n)=|\{ m  \in \mathbb{Z}^{+} \; | \; m\leq n \land (m,n) =1 \}| 
$$
- 小于等于 $n$ 的**互质数的个数**


**容斥定理可证明**：
$$
\varphi(n)=n \prod_{p|n}\left( 1-\frac{1}{p} \right)
$$
- where $\{ p \}$ is all prime factor of $n$
- **一个计算式子**
- if $p$ is prime number -> $\varphi(p)=p-1$

> [!Note] Euler 定理
> 对 $a,n \in \mathbb{Z}^{+}$, if $(a,n)=1$, then:
$$
 a^{\varphi(n)}\equiv 1 (\mod n) 
$$

> [!Note] Fermat little theorem
> 设正整数 $a$ 不是质数 $p$ 的倍数，则
$$
a^{p-1} \equiv 1 \mod p 
$$


# 归纳与递归
略
> Look carefully ...
> and, find the pattern...
> and, prove it!


# 计数

## 集合计数
- 将属于某个集合的元素理解为“具有某种性质”的对象，则属于该集合的补集的元素则是“不具有某种性质”的对象
- **需要计数原理**

## 容斥原理
> [!Note] 容斥原理 Principle of Inclusion and Exclusion
> 假设 $A_{1},A_{2},\dots,A_{n}$ 是 $n$ 个有限集合，则它们的并集的元素个数是：
$$
\bigcup_{i=1}^{n} A_{i}=S_{1}-S_{2}+S_{3}-\dots+(-1)^{k-1}S_{k}+\dots+(-1)^{n-1}S_{n} 
$$
> where, $S_{k}=\sum_{1\leq i_{1} \leq i_{2}\leq\dots\leq i_{k}\leq n} |A_{i_{1}}\cap A_{i_{2}} \cap\dots\cap A_{i_{k}}|, k=1,2,\dots,n$

- can solve *derangement*

## 鸽笼原理
> if put $n$ goose into $m$ cages, and $m<n$, then you at least need one cage to hold no less than $2$ goose

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260808202238.png)
- 把划分看作笼子，选的数看作鸽子，则必然有一个鸽子落入和为 9 的划分之一中。

## 排列与组合
> [!Note] **排列**
> 考虑有 $n$ 个元素的集合，有序取出 $r$ 个元素，元素不重复，有多少种可能的取法？
$$
 P(n,r)=n(n-1)(n-2)\dots(n-r+1)= \frac{{n!}}{(n-r)!} 
$$
> n-排列是一个双射


> [!Note] 二项式定理推论
> Let $n$ be a non-negative integer. Then
> $$\sum_{k=0}^n\binom{n}{k}=2^n$$


> [!Note] Vandermonde’s Identity
> **代表元素的思想**：Let $m,n, r$ be nonnegative integers with $r$ not exceeding either $m$ or $n$, Then
> $$ { m+n \choose r} = \sum_{k=0}^{r} \binom{m}{r-k} \binom{n}{k} $$



# 离散概率
- **直觉的概率分析**
	1. 选定样本空间 (Find the sample space)
	2. Define events of interests
	3. Determine outcome probabilities
	4. Compute event probabilities

- **概率空间**：集合论给概率以数学定义

> [!Note] **概率空间**
> 可数**样本空间 $S$** 是一个可数集合

**贝叶斯定理**

$$
\mathrm{Pr}[F|E]=\frac{{\mathrm{Pr}[E|F]\mathrm{Pr}[F]}}{\mathrm{Pr}[E]}
=\frac{{\mathrm{Pr}[E|F] \mathrm{Pr}[F]}}{\mathrm{Pr}[E|F]\mathrm{Pr}[F]+\mathrm{Pr}[E|\bar{F}]\mathrm{Pr}[\bar{F}]}
$$

> [!Note] 随机变量
> Random variable is not actually a *variable*, but a function
> - **codomain** can be any non-empty *set*, but usually $\mathbb{R}$



> [!Note] Chebyshev's Inequality
> For r.v. $X$ on sample space $S$ and any positive real number $r$, we hold
> $$ p(|X(s)-E(X)|\geq r) \leq \frac{V(X)}{r^{2}} $$


# 偏序与偏序格
## 偏序关系 Partial Order
- **Definition**
	- 非空集合 $A$ 上的自反、反对称和传递的关系称为 $A$ 上的偏序关系
	- $\preceq$

> [!Note] 偏序集 (poset)
> **定义**：集合 $A$ 和 $A$ 上的偏序关系 $\preceq$ 一起叫做偏序集，记作 $(A, \preceq)$





> [!Note] 哈斯图 (Hasse Diagrams)
> 将偏序关系简化为哈斯图：
> - 省略所有顶点上的环
> - 省略所有因传递关系而引出的边
> - 根据箭头的方向自下而上重排列所有顶点，而后将所有的有向边替换为无向边

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260808220504.png) 


- **特殊元素**
	- 最小元，最大元，极小元，极大元
		- Least, greatest, maximal, minimal element
- 上界，下界
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260808220917.png)

## 格 lattice
作为一个代数系统。
- 用偏序集和偏序关系定义

> [!Note] Lattice
> **Definition**:
> 设 $(S, \preceq)$ 是偏序集，如果 $\forall x, y \preceq S$, $\{ x,y \}$ 都有最小上界和最大下界，则称 $S$ 关于偏序 $\preceq$ 作成一个格

- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260808221105.png)


# 代数系统
## 15 代数系统引论

- **运算与封闭性**：$f:A^n\to A$ 是 $n$ 元运算；若对 $A$ 中任意元素运算结果仍在 $A$，则运算在 $A$ 上**封闭**(*closeness*)，$(A,\circ)$ 构成代数系统。
- **运算性质**：
    - 结合律：$(a\circ b)\circ c=a\circ(b\circ c)$
    - 交换律：$a\circ b=b\circ a$
    - 分配律：$a\circ(b_c)=(a\circ b)_(a\circ c)$（涉及两个运算）
- **特殊元素**：
    - 单位元（幺元）$e$：$e\circ a=a\circ e=a$，若存在必唯一（左右幺元相等时唯一）
		- **左幺元**：$e_{L}$ is left-unit element. $\forall x \in S, e_{L} \circ x = x$
    - 零元 $\theta$：$\theta\circ a=a\circ\theta=\theta$
	
    - 逆元 $a^{-1}$：$a\circ a^{-1}=a^{-1}\circ a=e$；结合律下左逆 $=$ 右逆且唯一
- **例**：$x\circ y=x+y-xy$ ——交换、结合；幺元 $0$；零元 $1$；$x\ne1$ 时逆元 $\frac{x}{x-1}$。
- **同构/同态**：双射 $f$ 满足 $f(x\circ y)=f(x)*f(y)$ 为同构；不要求双射（满射则为满同态）是同态。
	- $\circ$ and $*$ note 2 different operation.

**同构关系**是等价关系

## 16 群论导引
- **结构层次**：半群（结合）$\subset$ 幺半群（$+$ 幺元）$\subset$ 群（$+$ 每元有逆）；**Abel 群**再 $+$ 交换。
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260808222147.png)

- **群的性质**：
	- 运算表中每行每列都是全体元素的排列（消去律）；
	- 群的阶 $|G|$；
	- 元素 $a$ 的阶 $\operatorname{ord}(a)$ 是最小正整数 $n$ 使 $a^n=e$（不存在则 $\infty$）。
- **结论**：四阶群都是 Abel 群；四阶群中元素阶只能取 $1,2,4$。

## 17 子群与拉格朗日定理

- **子群** $H\le G$：$H$ 对 $G$ 的运算自成群。平凡子群：$G$ 与 ${e}$。
- **判定定理**：
    1. $\forall a,b\in H$，$ab\in H$ 且 $a^{-1}\in H$；
    2. $\forall a,b\in H$，$ab^{-1}\in H$（最常用）；
    3. 有限非空 $H$ 只需封闭性。
- **生成子群** $\langle a\rangle={a^k\mid k\in\mathbb{Z}}$；$\operatorname{ord}(a)=|\langle a\rangle|$；$a^k=e\iff\operatorname{ord}(a)\mid k$。
- **中心** $C={a\mid ax=xa\ \forall x}$ 是子群。
- **左陪集** $aH={ah\mid h\in H}$；所有左陪集构成 $G$ 的划分；$a,b$ 同陪集 $\iff b^{-1}a\in H$（陪集关系是等价关系）。
- **拉格朗日定理**：

$$ |G|=|H|\cdot[G: H]\qquad\Rightarrow\qquad |H|\ \big|\ |G| $$


- **推论**：元素阶整除群阶；$|G|$ 为素数 $\Rightarrow G$ 循环且 Abel。应用：6 阶群必有 3 阶子群；阶 $<6$ 的群都交换。

## 18 循环群与群同构

- **循环群** $G=\langle a\rangle$：每元素都是 $a$ 的*幂*.
    - 无限循环群 $\cong(\mathbb{Z},+)$，生成元恰 $\pm a$（2 个）；
    - $n$ 阶循环群 $\cong(\mathbb{Z}_n,+_n)$，**生成元共 $\varphi(n)$ 个**：$a^k$ 是生成元 $\iff\gcd(k,n)=1$。
- **子群**：循环群的子群仍是循环群；$n$ 阶循环群对每个 $d\mid n$ 恰有一个 $d$ 阶子群。
- **同构**：同构是等价关系；两个 3 阶群必同构；四阶群恰有两个同构类：$\mathbb{Z}_4$ 与 Klein 四元群 $V_4$。
- **直积**：

$$ C_m\times C_n\cong C_{mn}\iff\gcd (m, n)=1 $$

- **数论联系**：$\varphi(m)\varphi(n)=\varphi(mn)$（$m,n$ 互素）；模 $n$ 与 $n$ 互素的剩余类构成乘法群；欧拉定理 $a^{\varphi(n)}\equiv1\pmod n$。

## 19 代数格

- **两种定义等价**：
    - 偏序格：$(S,\preceq)$ 中任两元素 $x,y$ 都有 $\operatorname{lub}{x,y}=x\vee y$ 与 $\operatorname{glb}{x,y}=x\wedge y$；
    - 代数格：$(L,\wedge,\vee)$ 满足**结合律 + 交换律 + 吸收律**：

$$ (x\wedge y)\wedge z=x\wedge (y\wedge z),\qquad (x\vee y)\vee z=x\vee (y\vee z) $$

$$ x\wedge y=y\wedge x,\qquad x\vee y=y\vee x $$
$$x\vee (x\wedge y)=x,\qquad x\wedge (x\vee y)=x $$
- 由代数格可定义 $x\preceq y\iff x\wedge y=x$（$\iff x\vee y=y$）。
- **对偶原理**：$\preceq$ 与 $\succeq$、$\wedge$ 与 $\vee$ 互换后命题仍真。
- **子格、格同态、格同构**（同态保 $\wedge,\vee$）。
- **分类**：分配格（判定：不含 $M_3$、$N_5$ 子格）；有界格（有最大元 $1$、最小元 $0$）；有补格（每元有补元）；**有补分配格 = 布尔格**（补元唯一，满足双重补律、德摩根律）。

## 20 布尔代数
- **布尔函数** $f:B^n\to B$，$B={0,1}$；$n$ 度布尔函数共有 $2^{2^n}$ 个。
- **布尔代数公理**（$+$、$\cdot$、$'$、$0$、$1$）：结合、交换、分配、同一、补五组；由此可推出吸收律、幂等律、支配律、双重补律、德摩根律、补元唯一。
- **标准例子**：$({0,1},+,\cdot,')$；幂集 $(P(A),\cup,\cap,\sim,\emptyset,A)$；$B^n$；$n$ 度布尔函数全体。
- **原子**：覆盖最小元 $0$ 的元素；不同原子之交为 $0$。
- **有限布尔代数表示定理**：

$$ B\cong (P (A),\cup,\cap,\sim,\emptyset, A)\qquad (A=\text{原子集}) $$

故 $|B|=2^n$；等势的有限布尔代数同构。

- **因子格** $D_n$ 是布尔代数 $\iff n$ 无平方因子（$n=p_1p_2\cdots p_k$，互异素数）。
- **应用**：逻辑电路设计、卡诺图化简；${+,\cdot,'}$ 函数完全。

**布尔代数**：有补的*分配格*
- 与含 $n$ 个元素的集合的*幂集代数*系统同构的布尔代数记为 $B_{n}$
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260809104201.png)




---

**考试易错点**：

1. 群验证四步：非空、结合、幺元、逆元，一个都不能省；
2. 判定子群优先用 $ab^{-1}\in H$；
3. 循环群生成元个数是 $\varphi(n)$，不是 $n$；
4. 格 ≠ 偏序集：偏序集还要"任两元素都有上下确界"才算格；
5. 布尔代数一定是有补分配格，且有限布尔代数基数必为 $2^n$。


# 图论初步
> [!Note] Definition of **Graph**
> Graph $G$ is a 3-element group: $G=(V,E,\varphi)$
> - $V$ is non-empty vertex *set*, $E$ is edge set, and $V \cap E=\emptyset$
> - $\varphi:E\to P(V)$, and $\forall e \in E. 1\leq |\varphi(e)|\leq {2}$
> 	- $\varphi$:*end* point set of edge $e$


> [!Note] **握手定理**
> - 无向图 $G$ 有 $m$ 条边，$n$ 个顶点 $v_{1},\dots,v_{n}$
> $$ \sum_{i=1}^{n} d (v_{i})=2m$$
> - We have: 无向图中_奇数度顶点必为偶数个_


> [!Note] 完全图和特殊的简单图
> - **完全图**
> - **简单图**：不同边有不同端点集-> $e_{1}\neq e_{2} \implies \varphi(e_{1})\neq \varphi(e_{2})$
> 	- 若简单图 $G$ 中任意两点均相邻，则称为完全图 $K_{n}$, where $n$ is the amount of vertex in the graph
> - Cycle
> - Wheel
> - n-cube

**正则图**：*顶点度相同的简单图*


> [!Note] sub graph
> Let $G= <V,E>, G' = <V',E'>$, if $V' \subseteq V, E' \subseteq E$, then $G'$ is the subgraph of $G$
> if $V'\subset V$ or $E' \subset E$, then called real-subgraph
> **诱导子图**

### 图的表示
- **关联矩阵**(incidence matrix)
- 无向图 $G=(V,E,\varphi)$, assume $V=\{ v_{1},\dots,v_{n} \},E=\{ e_{1},\dots,e_{m} \}$
- $M(G)=[m_{ij}]$：关联矩阵

$$
m_{ij}=\begin{cases}
1,\text{if $e_{j}$ link to $v_{i}$},v_{i}\in \varphi(e_{j})
\\
0, else
\end{cases}
$$


- **邻接矩阵**(adjacency matrix)
	- $A(G)=[a_{ij}]$
$$
a_{ij}=\begin{cases}
1,\text{if $v_{i}$ link to $v_{j}$}, \exists e \in E. \varphi(e)=(v_{i}, v_{j})\\
0, else
\end{cases}
$$

- **邻接表**
	- 若 $G=(V,E,\varphi)$ 没有**多重边**，即 $\varphi$ 是单射，列出这个图的所有边。对每个顶点列出与其邻接的顶点
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260809113811.png)

- **逆图**的 adjacency matrix 是原图的 adjacency matrix 的转置

- *邻接矩阵的有趣运算*
	- 矩阵乘法可以算出通路个数。
	- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260809122605.png)
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260809122634.png)


> [!Note] 图的同构
> - 设 $G_{1}=(V_{1},E_{1},\varphi_{1})$ 和 $G_{2}=(V_{2},E_{2},\varphi_{2})$ 是两个 *简单无向图*
> 	- 若存在双射 $f:V_{1}\to V_{2}$，$u$ 和 $v$ 在 $G_{1}$ 中相邻当且仅当 $f(u)$ 和 $f(v)$ 在 $G_{2}$ 中相邻。此时称 $f$ 是一个同构函数
> - **一般图（含多重边/有向图）的定义**：除了顶点双射 $f:V_1\to V_2$，还要求存在**边集双射** $g:E_1\to E_2$ 保持关联关系：
> $$ \varphi_1(e)=\{u,v\}\iff\varphi_2(g(e))=\{f(u),f(v)\} $$
> $\forall e \in E_{1},\varphi_{1}(e)=\{ u,v \}\iff g(e)\in E_{2},\varphi_{2}(g(e))=\{ f(u),f(v) \}$


> [!Question] How to check?
> - 若图 $G$ 和图 $H$ 同构，则对于任意自然数 $k$，
> 	- $G$ 的 $k$ 度顶点导出子图与 $H$ 的 $k$ 度顶点导出子图同构
> - 若对于任意自然数 $k$，$G$ 的 $k$ 度顶点导出子图与 $H$ 的 $k$ 度顶点导出子图同构，$G$ 与 $H$ 是否同构



## 图的连通性

### 通路
> [!Note] *Definition*
> 图 $G$ 从 $v_{0}$ 到 $v_{n}$ 的长度为 $n$ 的通路是 $G$ 的 $n$ 条边 $e_{1},\dots,e_{n}$ 的序列，满足以下性质
> - $\exists v_{i} \in V(0<i<n)$, let $v_{i-1}$ and $v_{i}$ is two end-point of $e_{i}$ ($1\leq i \leq n$)
- **回路**：起点和终点相同，长度大于 0
- 长度为 $0$ 的通路由单个顶点组成
- **简单通路**：边不重复，即, $\forall i,j, i\neq j\implies e_{i}\neq e_{j}$
- **初级通路**：点不重复，亦称为“*路径*”


### 无向图的连通性
> [!Note] *连通性的定义*
> 无向图 $G$ 称为是连通的，如果 $G$ 中任意两个不同顶点之间都有通路

> [!Note] *连通分支*
> - 极大连通子图
> - *每个无向图是若干个互不相交的连通分支的并*
> 	- 顶点存在通路也是一个等价关系
> 	- 连通分支的代表元素
> - 若图 $G$ 存在从 $u$ 到 $v$ 的通路，则一定有从 $u$ 到 $v$ 的简单通路


![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260809124530.png)
> [!Note] Cut vertex（割点）
> *Definition*: $G$ is graph, $v \in V_{G}$, if $p(G-v)>p(G)$, then $v$ is *cut vertex*
> Equivalent proposition:
> 1. $v$ is cut vertex
> 2. Exists a division of $V-\{ v \}$ as $\{ V_{1},V_{2} \}$, let $\forall u \in V_{1},w\in V_{2}$, $uw-$ passage all includes $v$
> 3. Exists vertex $u,w(u\neq v,w\neq v)$, let $\forall uw-$ passages includes $v$


*Cut Edge, bridge*的定义是类似的
- $e$ 是割边当且仅当 $e$ 不在 $G$ 的任一简单回路上。
- 等价命题
	1. $e$ 是割边
	2. $e$ 不在 $G$ 的任一简单回路上。
	3. 存在 $V$ 的分划 $\{ V_{1},V_{2} \}$，使得 $\forall u \in V_{1},w\in V_{2}$, uw 通路均包含 $e$
	4. 存在顶点 $u,w$，使得任意的 $uw-$ 通路均包含 $e$


> [!Note] 图的（点）连通度
> **定义**：使非平凡连通图 $G$ 成为*不连通图或者平凡图*需要删除的*最少*顶点数称为图 $G$ 的（点）连通度，记为 $\kappa(G)$
> **约定**：不连通图或者平凡图的连通度为 $0$，而 $\kappa(K_{n})=n-1$
> - 若图 $G$ 的连通度 *不小于*$k$，则称 $G$ 是 $k-$ 连通图

*边连通度*类似定义
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260809134632.png)
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260809134722.png)


## 欧拉图
> [!Note] **欧拉通路**
> 定义：包含图（无向图或有向图）中每条边的简单通路称为*欧拉通路*

> [!Note] **欧拉回路**
> **定义**：包含图中每条边的简单回路称为*欧拉回路*

- if $G$ has Euler's circuit, then $G$ is *Euler Graph*. 
- if $G$ has Euler's passage, but no *Euler Circuit*, then $G$ is semi-Euler Graph

> [!Tip] **欧拉图**中的顶点度数
> - 连通图 $G$ 是欧拉图，当且仅当 $G$ 中每个顶点的度数均为偶数

---
关于*欧拉图*的等价命题
- 设 $G$ 是非平凡连通图，以下三个命题等价：
	1. $G$ 是欧拉图
	2. $G$ 中每个顶点的度数均为偶数
	3. $G$ 中所有的边包含在相互没有公共边的简单回路中


> [!Note] How to build a Euler's circuit? 
> **idea**: we cannot use edges that has been used, so, when we build the *Euler's Circuit*, we always assume when we delete *edge* has been passed, the *edges* left must still be in one *connected component*
> ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260809222730.png)


### Hamilton 通路/回路
- $G$ 中 Hamilton 通路
	- 包含 $G$ 中所有顶点
	- 通路上各顶点不重复
- $G$ 中 Hamilton 回路
	- 包含 $G$ 中所有顶点
	- 除了起点和终点相同之外，通路上各顶点不重复
- Hamilton Passage problem can -> Hamilton circuit problem

- **必要的基本条件**
	- 如果图 $G=(V,E)$ 是 Hamilton 图，则对 $V$ 的任一非空子集 $S$，都有
$$
P(G-S)\leq |S|
$$


### Hamilton 图的充分条件
- Dirac 定理
	- 设 $G$ 是无向简单图，$|G|=n \geq 3$, if $\delta(G)\geq \dfrac{n}{2}$, then $G$ has Hamilton circuit
- Ore 定理
	- 设 $G$ 是无向简单图, 若 $G$ 中任意不相邻的顶点对 $u,v$ 均满足：$d(u)+d(v)\geq n$, then  $G$ has hamilton circuit

## 最短通路问题 Dijkstra

Dijkstra 算法
- **算法思想**
- if the *shortest path* is $s\dots uv$, then $s\dots u$ is the *shortest path* from $s$ to $u$

$$
d(s,u_{i+1})=\min \{ d(s,u_{j})+W(u_{j},u_{i+1})|j=1,,\dots,i \}
$$


- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260810224524.png)

- 在算法设计与分析中有详解
	- [[图 Graph]]

## 二部图
> [!note] 二部图（偶图）
> **顶点集**：划分 2 个类别（不相交），边的端点在不同类别中。
> **完全二部图**：来自不同类别的两个顶点均有边。

**二部图**（偶图）是一种特殊的图：顶点集能分成**两个互不相交的部分** $V_1,V_2$，并且**每条边的两个端点分别落在不同部分**——也就是说，边只"跨部"连接，同部分内部没有边。

$$ G=(V_1\cup V_2,\ E),\qquad E\subseteq V_1\times V_2 $$

- **匹配**（边独立集）：*互不相邻*的边的集合
- M-饱和点：M 中各边的端点

> [!note] *完备匹配*
> **定义**：设 $G$ 是二部图，二部划分为 $<V_{1},V_{2}>$，若 $G$ 中的匹配 $M$ 饱和 $V_{1}$ 中所有顶点，则称 $M$ 为 $V_{1}$ 到 $V_{2}$ 的**完备匹配**

> [!note] **交错路径**与**可增广交错路径**
> **定义**：
> - 设 $M$ 是 $G$ 中一个匹配。若 $G$ 中路径 $P$ 中 $M$ 与 $E_{G}-M$ 中的边交替出现，则称 $P$ 为 $M-交错路径$（也可以是回路）
> - 若 $P$ 的起点与终点都是 $M-$ -非饱和点（没有被匹配的顶点），则称 $P$ 是**可增广交错路径**

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260812183437.png)


> [!question] How to get a 最大匹配
> **Berge 定理**：
> - $M$ 是最大匹配 <=> 相对于 $M$ 没有增广路径

### Hall 定理 (1935, Marriage Theorem)
设二部图 $G\impliedby <V_{1},V_{2},E>$, 则 $G$ 有 $V_{1}$ 到 $V_{2}$ 的完备匹配 <=>对于任意的 $A \subseteq V_{1}$，有 $|N(A)|\geq|A|$


### 判定定理
一个图是二部图，当且仅当以下任一条件成立：

1. **可以用 2 种颜色给顶点着色**，使相邻顶点颜色不同；
2. **不含奇数长度的回路**（无奇圈）。

所以"圈 $C_6$ 是二部图（偶数圈），三角形 $C_3$ 不是（奇圈）"。判定时常用 BFS/DFS 交替染色，遇到矛盾（同色相邻）就不是二部图。


> [!question]
> 对于任意简单连通无向图G，下列说法不正确的有：
> 1. 若G有完美匹配，则G的顶点数一定是偶数
> 2. 若G有两个不同的完美匹配，则G中一定存在回路
> 3. 若G是树，且G有完美匹配，则G的最大匹配是唯一的
> 4. 若G是树，且G有完美匹配，则G存在哈密顿通路


答案是：**不正确的只有 ④**。逐个看：

**① 正确。** 完美匹配的每条边覆盖 2 个顶点，若 $|M|=k$，则 $|V|=2k$，必为偶数。

**② 正确。** 设 $M_1\ne M_2$ 是两个完美匹配，考虑对称差 $M_1\oplus M_2$：每个顶点恰好关联 $M_1$ 中的一条边和 $M_2$ 中的一条边，所以对称差中每个顶点度为 0 或 2。因为 $M_1\ne M_2$，至少有一个非空分量，它只能是（偶长度）**回路**。所以 $G$ 中一定有回路。（这也是"树至多有一个完美匹配"的另一种解释。）

**③ 正确。** 树若有完美匹配，则它**唯一**：任取一个树叶 $v$，它只能和唯一邻居 $u$ 匹配，删掉 $v,u$ 后剩下的仍是树，递归地强制出整个匹配。而完美匹配的大小是 $n/2$，已达到匹配上界，所以它也是唯一的最大匹配。

**④ 不正确。** 树有完美匹配并不保证有哈密顿通路。事实上：**树存在哈密顿通路 ⟺ 树本身就是一条通路**（哈密顿通路要用 $n-1$ 条边，而树总共只有 $n-1$ 条边，两者必须重合）。但完美匹配并不要求树是通路。

反例（6 个顶点）：

$$ V=\{a, b, c, c_1, d, d_1\},\quad E=\{ab,\ bc,\ cc_1,\ bd,\ dd_1\}$$ 
即 $b$ 同时连着 $a,c,d$，$c$ 连着 $c_1$，$d$ 连着 $d_1$：

- 完美匹配存在且唯一：${ab,\ cc_1,\ dd_1}$；
- 但 $b$ 的度为 3，这棵树不是通路，所以**没有哈密顿通路**。

---

**结论：不正确的只有 ④**（①、②、③ 都成立）。