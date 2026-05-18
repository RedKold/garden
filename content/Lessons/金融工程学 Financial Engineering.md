- 课本
	- 郑振龙金融工程学 (第 6 版)
	- 宋逢明
	- Options, Futures and Other Derivatives
- Score
	- 20% daily
		- 10% personal
		- 10% team
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

# 远期和期货定价
- Bid：银行买入远期利率协议的利率报价
- Ask：银行卖出远期利率协议的利率报价


## 远期价格和期货价格
- **远期价值**（forward value）是指远期合约本身的价值

- **远期价格**(forward price) 是指使远期合约签订时价值为零的交割价格。远期价格是理论上的交割价格

---
类似的，定义
- **期货价格**(Futures Prices) 为使得期货合约价值为 0 的理论交割价格


## 远期利率协议
### 概述
远期利率协议（FRA）是买卖双方同意从未来某一商定的时刻开始的一定时期内，按协议利率借贷一笔数额确定、以具体货币表示的名义本金的协议



## 利率期货
> [!Note] 什么是利率期货？
> **利率期货**是指以利率敏感证券(国债？)作为标的资产的期货合约。
> - **短期利率期货**：是以（期货合约到期时）期限不超过 1 年的货币市场利率工具为交易标的的利率期货，其典型代表为在 CME 交易的 3 个月欧洲美元期货
> - **长期利率期货**：以（期货合约到期时）长期期限的资本市场利率工具为交易标的的利率期货。典型代表：长期美国国债期货 (30Year U.S. Treasury Bonds Futures)
> 
>  **规避风险**
> -  远期利率协议中的多头是规避利率上升风险的一方，而利率期货的多头则是规避期货价格上升风险，即规避利率下跌风险的一方。

- 多头总是规避价格上升风险的交易者，而利率期货的 头则是规避期货价格的上升风险，即规避利率下跌风险的一方。
- **利率期货**本质是国债一类的产品，参考式子 $P=\frac{C(债息)}{R(贴现率)}$，利率是贴现率，现在你容易思考了。


- T-Bill : $\leq 1 year$
- T-Note $(1, 10]$
- T-Bond $>10$


### 欧洲美元期货结算
- 每个基点（0.01%）变动的价值：
$$
1000000\times 0.0001 \times \frac{1}{4} =25 美元
$$
- **到期现货价**：
$$
100\times(1-实际3个月期\text{LIBOR})
$$

- 到期多头盈亏
$$
\begin{align}
到期多头盈亏 & = \\
 & =[100\times(1-\text{LIBOR})-Q]\times 100 \times 25\\
  & =(期货利率-\text{LIBOR}\times 10000 \times 25)
\end{align}
$$


- **欧洲美元期货**的报价：以“ Q (Quote，报价) = 1**00-期货利率\*100** ”给出
$$
Q=100-期货利率\times100
$$
	- 称之为 IMM 指数 (International M Market)
	- **贴现率**（价格）discount

- eg #必考 
- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260518150416.png)

## 中金所的 5 年期国债期货
### 债券的报价 #必考
- 报价时通常报出面值每 100 元的价格
	- 不同市场的最小报价单位往往不同
- 债券报价时使用的是**净价而非全价**
	- 全价 (full price)
		- 附息票债券现货或期货交割时多方实际支付和空方实际收到的价格是债券的真实价格
	- 净价 (clean price)
		- 等于全价减去应计利息 (accrued interest)
		
- **应计利息**：上一个付息日以来的利息（按比例计算）
$$
\text{全价}=\text{净价（报价）}+\text{应计利息}
$$
	- 这个很重要可能会考！ #必考 


## 可交割券与标准券
- 合约到期月首日剩余期限为 4-5.25 年的任何记账式附息国债**均为该期货合约的可交割券**
- **标准券**：
	- 面值为 100 万
	- 息票率为 3%
	- 在交割月首日的剩余到期期限为 5 年整的虚拟债券，是其他实际可交割债券价值的衡量标准
- 标准券可以做一个基准

### 转换因子（conversion factor）
- 每 1 元面值的可交割债券的**未来现金流**按 3%的年到期收益率贴现到**交割月首日的价值**，再扣掉该债券每 1 元面值**应计利息后的余额**

> [!Note] **转换因子的计算**
> 在交割过程中，各可交割国债的票面利率、到期时间等各不相同，为具可比性，所有可交割券与虚拟券都以同样的到期收益率 YTM (3%) 贴现至最后交割日，与虚拟券的价格比值即为转换因子

- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260518153738.png)


**计算公式**

$$
CF=\frac{{1}}{{\left( 1+\frac{r}{f} \right)^{xf/12}}}\left[ \frac{c}{f}+\frac{c}{r}\left( 1-\frac{1}{\left( 1+\frac{r}{f} \right)^{n-1}} \right) +\frac{1}{\left( 1+\frac{r}{f} \right)^{n-1}} \right] - {\frac{c}{f}} \times \frac{{12-xf}}{12}
$$

- $r$：表示国债期货标准合约利率，定为 3%
- $x$：表示交割月距离下一个付息月的月份数（当交割月是付息月时，x=6/12，取决于交割券是一年付 2 次还是 1 次）
- $n$：表示剩余付息次数
- $c$：表示可交割券的票面利率
- $f$：表示可交割券每年的付息次数


$$CF = \text{折现因子} \times \Big[ \text{交割后第一笔利息} + \text{后续年金(剩余利息)} + \text{面值本金} \Big] - \text{应计利息扣减}$$
- $\frac{c}{f}$ 对应第一笔利息
- $\frac{c}{r}\left( 1-\frac{1}{\left( 1+\frac{r}{f} \right)^{n-1}} \right)$ 是永续年金的阶段，代表后续年金
- $\frac{1}{\left( 1+\frac{r}{f} \right)^{n-1}}$ 表示面值本金
	- 我们是研究 1 元的


### 最便宜可交割券 (CTD)
> [!Note] 最便宜可交割券 (the cheapest to deliver, CTD)
> 购买交割券所付的价格与交割期货时空方收到的现金之差最小的债券
- 交割日：交割成本最小

### 确定准 CTD 券
- **常见规则**：交割日之前，IRR 最高
- 隐含回购利率 (Implied Repo Rate, IRR)
- 无付息情形
$$
\text{IRR}=\frac{{约定的期货全价-今天现货全价}}{今天现货全价}\times \frac{{365}}{T-t}
$$

- ![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260518163814.png)
- 可以简单理解为一个*收益率*



## 国债期货价格的确定

- 假定 CTD 券和交割日期已知，不考虑期权：
1. 根据 CTD 券现货报价，算出现货全价。
2. 根据支付已知现金收益的现货定价公式，用 CTD 券现货全价算出 CTD 券期货全价
$$
F=(S-{I})e^{r(T-t)}
$$
1. 减去配对缴款日应计利息，算出 CTD 券期货净价
2. 除以转换因子，即为标准券期货净价（期货报价）


### 题目案例
#必考 

![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260518165507.png)
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260518165517.png)
