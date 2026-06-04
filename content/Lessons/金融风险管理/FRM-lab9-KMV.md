## 代码
按照实验手册参考实现。

```python
import pandas as pd
import numpy as np
from scipy.optimize import root, fsolve
from scipy.stats import norm
from arch import arch_model
import warnings
warnings.filterwarnings('ignore')


def compute_equity_value(market_price, shares_outstanding, net_asset_per_share, non_tradable_shares):
    """Step 1: Compute equity value E0"""
    tradable_value = shares_outstanding * market_price
    non_tradable_value = non_tradable_shares * net_asset_per_share
    return tradable_value + non_tradable_value


def compute_debt_value(short_term_debt, long_term_debt):
    """Step 1: Compute debt value D (default point DP)"""
    return short_term_debt + 0.5 * long_term_debt


def compute_equity_volatility(returns):
    """Step 1: Compute equity volatility σE using GARCH(1,1)"""
    returns = returns / 100
    returns = returns.dropna()

    am = arch_model(returns * 100, vol='Garch', p=1, q=1, dist='Normal', rescale=False)
    res = am.fit(disp='off')

    conditional_variance = (res.conditional_volatility / 100) ** 2

    avg_variance = conditional_variance.mean()
    annualized_vol = np.sqrt(avg_variance * 243)
    return annualized_vol


def kmv_system(vars, E, D, r, tau, sigma_E):
    """Step 2: KMV system of equations"""
    x, sigma_a = vars

    Va = x * E
    d1_num = np.log(Va / D) + (r + 0.5 * sigma_a**2) * tau
    d1_den = sigma_a * np.sqrt(tau)
    if d1_den == 0:
        return [1e10, 1e10]
    d1 = d1_num / d1_den
    d2 = d1 - sigma_a * np.sqrt(tau)

    eq1 = Va * norm.cdf(d1) - D * np.exp(-r * tau) * norm.cdf(d2) - E
    eq2 = (Va / E) * norm.cdf(d1) * sigma_a - sigma_E

    return [eq1, eq2]


def solve_kmv(E, D, r, tau, sigma_E):
    """Step 2: Solve KMV equations with better initial guesses and bounds"""
    initial_guesses = [
        [1.0, 0.1],
        [1.5, 0.2],
        [2.0, 0.3],
        [0.8, 0.05]
    ]

    best_sol = None
    min_error = 1e20

    for x0 in initial_guesses:
        try:
            sol = root(kmv_system, x0, args=(E, D, r, tau, sigma_E), method='lm')
            if sol.success:
				# 正则化error，和精度做对比
                error = np.linalg.norm(sol.fun)
                if error < min_error:
                    min_error = error
                    best_sol = sol.x
        except:
            continue

    if best_sol is None:
        best_sol = fsolve(kmv_system, [1.0, 0.1], args=(E, D, r, tau, sigma_E))

    x, sigma_a = best_sol
    Va = x * E
    return Va, sigma_a


def compute_default_probability(Va, sigma_a, D, mu=0, tau=1):
    """Step 3: Compute default distance DD and default probability PD"""
    ETVa = Va * np.exp(mu * tau)
    DD = (ETVa - D) / (ETVa * sigma_a)
    PD = norm.cdf(-DD)
    return DD, PD


def main():
	# this is my local path.
    value_data = pd.read_excel('/Users/kold/koldcode/nju-frm/lab9/价值数据.xlsx')
    return_data = pd.read_excel('/Users/kold/koldcode/nju-frm/lab9/行情数据.xlsx')

    results = []
    stock_names = ['华域汽车', '*ST斯太']

    for idx, row in value_data.iterrows():
        name = stock_names[idx]
        ticker = row['股票代码']
        is_st = row['是否ST']
        tau = row['评估时长']
        net_asset_per_share = row['每股净资产']
        non_tradable_shares = row['非流通股股本']
        monthly_avg_close = row['月均收盘价']
        tradable_shares = row['流通股股本']
        long_term_debt = row['长期债务价值']
        short_term_debt = row['短期债务价值']
        r = row['无风险利率']

        returns = return_data[name]

        E = compute_equity_value(monthly_avg_close, tradable_shares, net_asset_per_share, non_tradable_shares)
        D = compute_debt_value(short_term_debt, long_term_debt)
        sigma_E = compute_equity_volatility(returns)

        Va, sigma_a = solve_kmv(E, D, r, tau, sigma_E)

        DD, PD = compute_default_probability(Va, sigma_a, D, tau=tau)

        results.append({
            '股票名称': name,
            '股票代码': ticker,
            '是否ST': '是' if is_st else '否',
            '股权价值E': E,
            '债务价值D': D,
            '股权波动率σE': sigma_E,
            '资产价值Va': Va,
            '资产波动率σa': sigma_a,
            '违约距离DD': DD,
            '违约概率PD': PD
        })

    results_df = pd.DataFrame(results)
    print(results_df[['股票名称', '是否ST', '违约距离DD', '违约概率PD']])


if __name__ == "__main__":
    main()
```

## 结果

| 股票名称 | 是否 ST | 违约距离 DD | 违约概率 PD |
|----------|--------|------------|------------|
| 华域汽车 | 否     | 2.9545     | 0.001566   |
| *ST 斯太  | 是     | 1.4375     | 0.075287   |

我们发现 **\*ST 斯太** 的违约概率确实**大于 华域汽车**
