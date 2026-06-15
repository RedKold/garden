- 课本
	- 郑振龙金融工程学 (第 6 版)
	- 宋逢明
	- Options, Futures and Other Derivatives
- Score
	- 20% daily
		- 10% personal
		- 10% team
		- [[Financial Engineering-test]]
	- 80% final

Finance is Optimally allocate financial resources across time and uncertainty

经济学是研究个人、企业、政府等如何在稀缺资源条件下进行选择和决策的科学。

均值-方差理论
- 风险受益理论


有效市场假说（Efficient Market Hypothesis, EMH）认为，在一个信息充分、竞争激烈的市场中，所有已知信息都会迅速反映在证券价格中，投资者无法通过分析历史价格获得超额收益。

- 价格充分反映可获得的信息

- CRR 模型
- CIR 模型

为各种金融问题提供创造性的解决方案：**金融工程的根本目的**
- 满足市场丰富多样的金融需求 2



## 金融衍生品的假设

- 市场是完全竞争的
- 市场参与者厌恶风险，且希望财富越多越好
	- risk and return is matched
- 市场不存在套利机会
	- in economics, we have a one-price theorem 
	- 无套利原则是经济学一价定律的应用

### 连续复利 Continuous Compounding
amount $A$, return rate $R$, invested  for $n$ years,  if the rate is *compounded* once per annum, the terminal value of the investment is:
$$
A(1+R)^{n}
$$
if compounded $m$ times return per year, then final value is 
$$
A\left( 1+\frac{R}{m} \right)^{mn}
$$
let $m\to \infty$, then we form *Continuous compounding*, final value is:
$$
\lim_{ n \to \infty } A\left( 1+\frac{R}{m} \right)^{mn}=Ae^{Rn}
$$

#必考 

$$
PV=\sum{\frac{FV_{i}}{(1+r_{i})^{i}}}
$$

债息 coupon
利息 interest
股利 dividend
- **本质**：使用资金的成本


**绝对定价法**
- *一般均衡定价法*

**严格套利**的三大特征
- 无风险/复制/零投资

## Rates
### Risk-free rate
在考虑衍生品定价的时候，我们经常需要引入一个 riskless 的 portfolio，假设这个投资组合的 return 是 risk-free-rate
- 起到了关键作用

### Zero rates
The $n$ -year zero-coupon is
- the rate of interest earned on an investment that starts *today* and lasts for $n$ years
- All the interest and principal is realized at the end of $n$ years. **no intermediate payments**
Suppose a 5-year zero rate with continuous compounding is quoted as 5% per annum. This means that INR100, if invested for 5 years, grows to 
$$
100\times e^{0.05\times {5}} = 128.40
$$

as more general:
$$
\lim_{ n \to \infty } A\left( 1+\frac{R}{m} \right)^{mn}=Ae^{Rn}
$$

## Bond Pricing （债券定价）
Most bonds pay coupons to the holder periodically

The theoretical price of a *bond*:
- as the present *value* of all cash flows that will be received by the owner of the bond

# Determination of Forward and Futures Prices

## Generalization for Forward Price

We consider a forward contract on an investment asset with price $S_{0}$ that provides no income.
- $T$ is the time to maturity
- $r$ is the risk-free rate
- $F_{0}$ is the forward price
$$
F_{0}=S_{0}e^{rT}
$$


## **复利定价法** #必考 
假设一种不支付红利股票目前的市价为 10 元，我们知道在 3 个月后，该股票价格要么是 11 元，要么是 9 元。假设现在的无风险年利率等于 10%，现在我们要找出一份 3 个月期协议价格为 10.5 元的该股票欧式看涨期权的价值

**类似的这种题**。

- 构造无风险资产
	- 一单位看涨期权空头，$\Delta$ 单位标的股票多头的组合
	- 满足无风险，无风险的贴现，计算出 $\Delta$
	- 计算出 PV
	- $\Delta S-f=\Delta S_{d}-f$



## 复制定价法


## 风险中性定价
股票上升概率为 $P$, 下降概率 $(1-P)$，未来期望值按无风险利率贴现的现值必须等于该股票目前的价值，该概率可以求：

$$
S=e^{-r(T-t)}\left[ SuP+Sd(1-P) \right] 
$$
- $u$ 是上涨因子，$d$ 是下跌因子.

where $e^{-r(T-t)}$ 是一个无风险连续复利的**贴现因子**

then we have
$$
P=\frac{{e^{r(T-t)-d}}}{u-d}
$$
$$
f=e^{-r(T-t)}\left[ Pf_{u}+(1-P)f_{d} \right] 
$$
- $f_{u}$：股价涨了，期权到期赚多少
- $f_{d}$：股价跌了，期权到期赚多少
- 用同一个风险中性概率 $P$ 计算期权未来期望收益
- 无风险利率 $e^{-r(T-t)}$ 折现


	- option(期权)是类似的
## General case

## 状态价格定价技术 (Arrow-Debru 基本证券)
- State price:
	- some state return is 1, other is 0
- **两个基本证券**
- 相当于正交基
$$
\begin{align}
a=[1,0] \\
b=[0,1]
\end{align}
$$
- 用正交组合定价

#  [[远期与期货定价]]

# 期权与期权市场

## 期权的定义和种类
> [!Note] 什么是期权？
> 期权 (Option)，是指赋予其购买者在规定期限内按双方约定的价格（简称**执行价格，Exercise Price or Striking Price）** 购买或出售一定数量某种资产（称为**标的资产**，**Underlying Assets**）的权利的合约

### 看涨期权和看跌期权
- **看涨期权**(Call Option)
	- 赋予期权买者未来按约定价格**购买**标的资产的权利
- **看跌期权**(Put Option) -btw, put 有丢掉的意思
	- 赋予期权买者未来按约定价格**出售**标的资产的权利


### 理解期权的多头/空头
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260525152006.png)
**二者的权利义务是对等的**
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260525152618.png)

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260525153056.png)
- **横轴**：标的资产的市场价格
- **纵轴**：期权的价值

### **欧式期权**与**美式期权**
> [!Note] **期权种类**：欧式期权，美式期权
> - 欧式期权
> 	- 欧式期权的多方只有在期权到期日才能执行期权
> - 美式期权
> 	- 美式期权允许多方在期权到期前的任何时间执行期权
> - 百慕大期权
> 	- 介于欧式和美式之间，期权的执行期一般为到期日的某一段时间

****

## 期权合约的标的资产
> [!Note] 按照标的物来划分：
> 金融期权合约可分为**股票期权、股价指数期权、金融期货期权、利率期货、信用期权、货币期权**（or**外汇期权**）及**互换期权**
> - 标的资产不同，期权的特性、定价和风险管理也呈现出不同的特点



## 权证

权证（Warrants）是发行人与持有者之间的一种契约，其发行人可以是上市公司，也可以是上市公司股东或投资银行等第三者。权证允许持有人在约定的时间（行权时间），可以用约定的价格（行权价格）向发行人购买或卖出一定数量的标的资产。

如果权证由上市公司自己发行，就叫做股本权证。

如果权证由独立的第三方 (通常是投资银行) 发行，则称为备兑权证。



---
> [!Tip] **权证**和期权的区别：
> **权证**有发行环节
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260601162432.png)


## 内嵌期权与实物期权
> [!Note] **内嵌期权**
> 所谓**内嵌期权**是指在普通的金融产品中，加上一个具有期权性质的条款，使得该产品成为普通金融工具和期权的一个组合。
> e.g.:**可转债、可赎回债券、可回售债券**
> ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260601162533.png)



> [!Note] 实物期权
> 以**实物资产**为标的物的未来选择权


# 期权的回报与价格分析

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260601164510.png)
- **欧式期权**
- **执行价格的确定**
	- 40
	- 因为这个时候我们开始有回报。
	- 不过我们仍然亏了一个**期权费**
- 盈亏是 0 的点
	- **把期权费涨回来的点**。

---
**由于期权合约**是零和游戏 (Zero-Sum Games)
- 多头和空头的愿望是对立的

所以**看涨期权**多头和空头的曲线关于 $x$ 轴对称

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260601164850.png)

- **看跌**也是同理
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260601165425.png)
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260601165457.png)


## 欧式期权回报公式

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260601165544.png)
### 欧式期权回报公式（到期日 $T$）

不看期权费，只看**回报 (Payoff)**：

| 类型 | 条件 | 回报公式 |
|------|------|---------|
| **看涨 Call** | $S_T > K$（实值） | $S_T - K$ |
| | $S_T \leq K$（平值/虚值） | $0$ |
| **看跌 Put** | $S_T < K$（实值） | $K - S_T$ |
| | $S_T \geq K$（平值/虚值） | $0$ |

统一写成：

$$
\text{Call Payoff} = \max(S_T - K, 0)
$$
$$
\text{Put Payoff} = \max(K - S_T, 0)
$$

**考虑期权费后的净损益**：

$$
\text{Call Profit} = \max(S_T - K, 0) - c
$$
$$
\text{Put Profit} = \max(K - S_T, 0) - p
$$

其中 $c$ 是看涨期权费，$p$ 是看跌期权费。盈亏平衡点：
- **Call**: $S_T = K + c$
- **Put**: $S_T = K - p$

## 期权价值的特性

### 期权的时间价值
> [!Note] What is 期权的的时间价值
> **期权的时间价值**是在期权尚未到期时，标的资产价格的 boding

### 期权的内在价值
Intrinsic Value: 就是合同的价格。
- 不考虑标的资产的价格波动的情况下，期权条款赋予期权多头的最高价值。

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260615141322.png)

#必考 

### 期权价格的确定：两分法

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260615141408.png)

> [!Note] **时间价值的深入理解**
> 期权时间价值的来源是什么呢？答案是，标的资产价格变化导致期权价格变化的不对称性使得期权总价值超过其内在价值，这就是期权时间价值的来源。换句话说，无论将来价格怎么波动，期权多头的亏损永远是有限的，而增加的盈利却可能是无限的，因此标的资产的波动对于期权所有者来说是利大于弊的，这种不对称导致多头愿意为了一段时间内的波动多付期权费，从而产生了时间价值。


![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260615142316.png)

### 期权价格的上限
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260615142424.png)
### 期权价格的下限
由于时间价值一定大于等于 0，所以这就是一个自然的下限——**内在价值**


![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260615142947.png)

## 看跌期权与看涨期权之间的平价关系（Put-Call Parity）

### 基本公式（欧式期权）

$$
C + PV(K) = P + S
$$

其中：
- $C$ = 看涨期权价格
- $P$ = 看跌期权价格
- $S$ = 标的资产当前价格
- $PV(K) = K e^{-rT}$ = 执行价的现值（连续复利）

### 核心思想

两个组合在到期日的 payoff 完全相同，根据**无套利原则**，它们在今天的价格必须相等：

| 组合 A | 组合 B |
|--------|--------|
| 买入一份看涨期权 $C$ | 买入一份看跌期权 $P$ |
| 持有现金 $PV(K)$ | 买入一股标的资产 $S$ |

到期时两个组合的价值都是 $\max(S_T, K)$，因此期初价格相等。

### 两个关键推论

1. **用期权复制远期合约（合成远期）**：
   $$
   C - P = S - PV(K)
   $$
   - 买入看涨 + 卖出看跌（同一执行价）= 远期多头
   - 即用一堆期权可以复制出远期合约

2. **用期权和远期复制另一个期权**：
   $$
   C = P + S - PV(K) \quad \text{—— 看跌 + 股票 + 借贷 = 看涨}
   $$
   $$
   P = C - S + PV(K) \quad \text{—— 看涨 + 卖空股票 + 现金 = 看跌}
   $$

### 直观理解

Put-Call Parity 说明看涨和看跌期权不是独立定价的——给定其中一个的价格，另一个就被无套利约束锁定。如果违背这个关系，就存在套利机会（买入被低估的一方，卖出被高估的一方）。


## 布朗运动 (Brownian Motion) / 维纳过程 (Wiener Process)

### 定义

布朗运动 $B_t$（或 $W_t$）是一个连续时间随机过程，满足：

1. **$B_0 = 0$**
2. **独立增量**：对 $0 \le s < t$，增量 $B_t - B_s$ 与过去 $\{B_u: u \le s\}$ 独立
3. **正态增量**：$B_t - B_s \sim N(0, t-s)$，即均值为 0，方差为 $t-s$
4. **连续路径**：$B_t$ 关于 $t$ 几乎处处连续

### 性质

- **鞅 (Martingale)**：$E[B_t | \mathcal{F}_s] = B_s$，即未来期望等于当前值
- **二次变分 (Quadratic Variation)**：$[B, B]_t = t$，这是布朗运动区别于可微函数的关键性质
  - 直观上：$E[(dB)^2] = dt$，即 $(dB)^2 \to dt$（在均方意义下）
- **非可微**：路径几乎处处连续但不可微
- **方差随时间线性增长**：$\text{Var}(B_t) = t$

### 广义布朗运动 (Generalized Brownian Motion)

$$
dx = a \, dt + b \, dz
$$

- $a \, dt$：**漂移项 (drift)** — 确定性趋势
- $b \, dz$：**扩散项 (diffusion)** — 随机波动，$dz$ 是标准布朗运动
- $x$ 的增量服从：$\Delta x \sim N(a\Delta t, b^2\Delta t)$

## 伊藤过程 (Itô Process)

### 定义

伊藤过程是更一般的随机过程形式：

$$
dx = a(x, t) \, dt + b(x, t) \, dz
$$

- $a(x, t)$：**瞬时漂移率**（drift rate）
- $b(x, t)^2$：**瞬时方差率**（variance rate）
- $dz$：标准布朗运动

即漂移和扩散系数可以依赖于当前状态 $x$ 和时间 $t$。
- 引入了时间 $t$ 这个变量。

### 几何布朗运动 (Geometric Brownian Motion, GBM)

$$
dS = \mu S \, dt + \sigma S \, dz
$$

- 用于股票价格建模（如 Black-Scholes 模型）
- $\mu$：预期收益率，$\sigma$：波动率
- 股价对数服从正态分布：$\ln S_T \sim N(\ln S_0 + (\mu - \frac{\sigma^2}{2})T, \sigma^2 T)$

### 伊藤引理 (Itô's Lemma) — 核心工具

若 $x$ 服从伊藤过程 $dx = a\,dt + b\,dz$，则函数 $G(x, t)$ 满足：

$$
dG = \left( \frac{\partial G}{\partial x}a + \frac{\partial G}{\partial t} + \frac{1}{2}\frac{\partial^2 G}{\partial x^2}b^2 \right) dt + \frac{\partial G}{\partial x}b \, dz
$$

与普通微积分的关键区别在于多了 $\frac{1}{2}\frac{\partial^2 G}{\partial x^2}b^2 \, dt$ 这一项，它来源于 $(dz)^2 \to dt$。

普通微积分中是二阶小量所以扔掉；伊藤微积分中由于，不是二阶小量而是**一阶小量**，所以保留了下来。

#### 应用：对数变换

对 GBM $dS = \mu S dt + \sigma S dz$，取 $G = \ln S$：

$$
d(\ln S) = \left( \mu - \frac{\sigma^2}{2} \right) dt + \sigma dz
$$

即对数价格服从广义布朗运动。

### 伊藤过程 vs 普通微积分

| 概念 | 普通微积分 | 伊藤随机微积分 |
|------|-----------|--------------|
| $(dt)^2$ | 忽略 | 忽略 |
| $dt \cdot dz$ | — | 忽略 |
| $(dz)^2$ | — | $\to dt$（关键！） |

这一差异使得随机微积分中的链式法则多出一个修正项 $-\frac{1}{2}\sigma^2 dt$，这正是 Black-Scholes 方程推导的关键。

## Black-Scholes 方程

### 基本假设

1. 股价服从几何布朗运动：$dS = \mu S dt + \sigma S dz$
2. 允许卖空标的资产
3. 无交易费用和税收，证券连续交易
4. 在衍生品有效期内无红利
5. 无风险利率 $r$ 为常数
6. 不存在无风险套利机会

### 方程推导思路

构造**无风险投资组合**：买入一份看涨期权 + 卖空 $\Delta$ 股股票（$\Delta$ 动态调整）。

组合价值：$\Pi = V - \Delta S$

对 $\Pi$ 运用伊藤引理，并选择 $\Delta = \frac{\partial V}{\partial S}$（Delta 对冲）消去随机项 $dz$，得到：

$$
\frac{\partial V}{\partial t} + rS\frac{\partial V}{\partial S} + \frac{1}{2}\sigma^2 S^2 \frac{\partial^2 V}{\partial S^2} = rV
$$

这就是 **Black-Scholes 偏微分方程（PDE）**。

### 方程含义

| 项                                                           | 含义                        |
| ----------------------------------------------------------- | ------------------------- |
| $\frac{\partial V}{\partial t}$                             | 期权价值随时间衰减（Theta）          |
| $rS\frac{\partial V}{\partial S}$                           | 融资成本（持有 Delta 头寸的资金成本）    |
| $\frac{1}{2}\sigma^2 S^2 \frac{\partial^2 V}{\partial S^2}$ | 凸性价值（Gamma）— 来源于伊藤引理的二阶修正 |
| $rV$                                                        | 无风险组合应获得的无风险收益            |

### Black-Scholes 公式（欧式看涨期权）

解上述 PDE（加上边界条件）得到**显式解**：

$$
C = S_0 N(d_1) - K e^{-rT} N(d_2)
$$

其中：

$$
d_1 = \frac{\ln(S_0/K) + (r + \sigma^2/2)T}{\sigma\sqrt{T}}, \quad
d_2 = d_1 - \sigma\sqrt{T}
$$

- $N(\cdot)$：标准正态分布的累积分布函数
- $S_0$：当前股价
- $K$：执行价
- $T$：到期时间
- $r$：无风险利率
- $\sigma$：波动率

欧式看跌期权（由 Put-Call Parity 得到）：

$$
P = K e^{-rT} N(-d_2) - S_0 N(-d_1)
$$

### 核心洞见

> B-S 公式说明期权价格不依赖于股票的预期收益率 $\mu$，只依赖于无风险利率 $r$ 和波动率 $\sigma$。

这是因为 $\mu$ 已经在 Delta 对冲过程中被消除——"风险中性定价"的思想：在无套利市场中，可以假设所有投资者都是风险中性的，所有资产收益率都等于无风险利率 $r$。