采取一般法计算 VaR，采取样本外检验回测方法。
编程语言: `Python`

# Code
```python

import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
from datetime import datetime

# 读取数据
df = pd.read_excel('前复权价格.xlsx')
df['日期'] = pd.to_datetime(df['日期'])
df = df.sort_values('日期').reset_index(drop=True)

# 股票代码和名称映射
stocks = {
    '000333': 'close_000333',
    '300059': 'close_300059',
    '600036': 'close_600036'
}
stock_names = ['000333', '300059', '600036']

# 设置参数
confidence_level = 0.95
alpha = 1 - confidence_level
shares_per_stock = 100  # 每只股票持有100股

# 划分估计期和回测期
estimation_end = pd.to_datetime('2019-12-30')
backtest_start = pd.to_datetime('2019-12-31')
backtest_end = pd.to_datetime('2020-03-31')

estimation_data = df[df['日期'] <= estimation_end].copy()
backtest_data = df[(df['日期'] >= backtest_start) & (df['日期'] <= backtest_end)].copy()

print("=" * 60)
print("实验6：基于历史模拟法的VaR计算")
print("=" * 60)
print(f"估计期: 2016-01-01 至 {estimation_end.date()}")
print(f"回测期: {backtest_start.date()} 至 {backtest_end.date()}")
print(f"置信水平: {confidence_level*100}%")
print("=" * 60)

# 计算日收益率
def calculate_returns(prices):
    returns = np.diff(prices) / prices[:-1]
    return returns

# 历史模拟法（一般法）计算VaR
def historical_var(returns, confidence_level, initial_value):
    sorted_returns = np.sort(returns)
    n = len(sorted_returns)
    var_index = int(np.floor(alpha * n)) - 1  # 第alpha*(n)个（从0开始）
    if var_index < 0:
        var_index = 0
    var_return = sorted_returns[var_index]
    var_value = -var_return * initial_value
    return var_value, var_return, sorted_returns

# 计算每只股票的VaR
results = {}
for stock_code in stock_names:
    prices = estimation_data[stocks[stock_code]].values
    returns = calculate_returns(prices)
    current_price = prices[-1]
    initial_value = current_price * shares_per_stock

    var_value, var_return, sorted_returns = historical_var(returns, confidence_level, initial_value)

    results[stock_code] = {
        'returns': returns,
        'current_price': current_price,
        'initial_value': initial_value,
        'var_value': var_value,
        'var_return': var_return,
        'sorted_returns': sorted_returns
    }

    print(f"\n股票 {stock_code}:")
    print(f"  当前价格(2019-12-30): {current_price:.4f}")
    print(f"  持仓市值: {initial_value:.2f}")
    print(f"  收益率样本数: {len(returns)}")
    print(f"  95%置信水平下VaR: {var_value:.2f}")
    print(f"  对应收益率分位数: {var_return:.4%}")

# 计算组合VaR
print("\n" + "=" * 60)
print("组合VaR计算")
print("=" * 60)

# 组合收益率（考虑市值权重）
prices_000333 = estimation_data[stocks['000333']].values
prices_300059 = estimation_data[stocks['300059']].values
prices_600036 = estimation_data[stocks['600036']].values

returns_000333 = calculate_returns(prices_000333)
returns_300059 = calculate_returns(prices_300059)
returns_600036 = calculate_returns(prices_600036)

# 计算各股票市值
value_000333 = prices_000333[-1] * shares_per_stock
value_300059 = prices_300059[-1] * shares_per_stock
value_600036 = prices_600036[-1] * shares_per_stock
total_value = value_000333 + value_300059 + value_600036

# 计算组合收益率（使用市值加权）
portfolio_returns = (returns_000333 * value_000333 +
                     returns_300059 * value_300059 +
                     returns_600036 * value_600036) / total_value

portfolio_var, portfolio_var_return, _ = historical_var(portfolio_returns, confidence_level, total_value)

print(f"组合总市值: {total_value:.2f}")
print(f"  000333: {value_000333:.2f} ({value_000333/total_value*100:.2f}%)")
print(f"  300059: {value_300059:.2f} ({value_300059/total_value*100:.2f}%)")
print(f"  600036: {value_600036:.2f} ({value_600036/total_value*100:.2f}%)")
print(f"\n组合95%置信水平VaR: {portfolio_var:.2f}")
print(f"对应收益率分位数: {portfolio_var_return:.4%}")

# 回测
print("\n" + "=" * 60)
print("回测检验（样本外检验）")
print("=" * 60)

# 使用估计期计算的VaR，在回测期中检验
backtest_prices = backtest_data[['日期'] + [stocks[sc] for sc in stock_names]].copy()

# 计算回测期日收益率和实际损失
backtest_results = []
violations = {sc: 0 for sc in stock_names}
violations['portfolio'] = 0

for i in range(1, len(backtest_prices)):
    date = backtest_prices.iloc[i]['日期']

    # 单只股票
    row_data = {}
    row_data['日期'] = date

    total_pnl = 0
    total_value_t0 = 0

    for sc in stock_names:
        p0 = backtest_prices.iloc[i-1][stocks[sc]]
        p1 = backtest_prices.iloc[i][stocks[sc]]
        ret = (p1 - p0) / p0
        pnl = (p0 - p1) * shares_per_stock  # 损失=（昨日价格-今日价格）*股数

        row_data[f'{sc}_收益率'] = ret
        row_data[f'{sc}_损失'] = pnl

        # 判断是否突破VaR
        if pnl > results[sc]['var_value']:
            violations[sc] += 1
            row_data[f'{sc}_突破'] = '是'
        else:
            row_data[f'{sc}_突破'] = '否'

        total_pnl += pnl
        total_value_t0 += p0 * shares_per_stock

    # 组合
    portfolio_ret = total_pnl / total_value_t0 if total_value_t0 != 0 else 0
    row_data['组合损失'] = total_pnl
    row_data['组合收益率'] = portfolio_ret

    if total_pnl > portfolio_var:
        violations['portfolio'] += 1
        row_data['组合突破'] = '是'
    else:
        row_data['组合突破'] = '否'

    backtest_results.append(row_data)

backtest_df = pd.DataFrame(backtest_results)

print(f"\n回测期交易日数: {len(backtest_df)}")
print(f"期望突破次数: {len(backtest_df) * alpha:.1f}")
print("\n实际突破次数:")
for sc in stock_names:
    print(f"  股票{sc}: {violations[sc]} 次")
print(f"  组合: {violations['portfolio']} 次")

print("\n" + "=" * 60)
print("回测详细结果（前10天）:")
print("=" * 60)
print(backtest_df[['日期', '组合损失', '组合突破'] + [f'{sc}_损失' for sc in stock_names] + [f'{sc}_突破' for sc in stock_names]].head(10))

# 保存结果到文件
with open('实验6结果报告.txt', 'w', encoding='utf-8') as f:
    f.write("=" * 60 + "\n")
    f.write("实验6：基于历史模拟法的VaR计算结果报告\n")
    f.write("=" * 60 + "\n\n")

    f.write(f"估计期: 2016-01-01 至 {estimation_end.date()}\n")
    f.write(f"回测期: {backtest_start.date()} 至 {backtest_end.date()}\n")
    f.write(f"置信水平: {confidence_level*100}%\n\n")

    for stock_code in stock_names:
        res = results[stock_code]
        f.write(f"股票 {stock_code}:\n")
        f.write(f"  当前价格: {res['current_price']:.4f}\n")
        f.write(f"  持仓市值: {res['initial_value']:.2f}\n")
        f.write(f"  95% VaR: {res['var_value']:.2f}\n")
        f.write(f"  对应收益率: {res['var_return']:.4%}\n\n")

    f.write("组合:\n")
    f.write(f"  总市值: {total_value:.2f}\n")
    f.write(f"  95% VaR: {portfolio_var:.2f}\n\n")

    f.write("回测结果:\n")
    f.write(f"  回测天数: {len(backtest_df)}\n")
    f.write(f"  期望突破次数: {len(backtest_df)*alpha:.1f}\n")
    for sc in stock_names:
        f.write(f"  股票{sc}突破次数: {violations[sc]}\n")
    f.write(f"  组合突破次数: {violations['portfolio']}\n")

print("\n" + "=" * 60)
print("结果已保存至: 实验6结果报告.txt")
print("=" * 60)

```

# Result
估计期: 2016-01-01 至 2019-12-30
回测期: 2019-12-31 至 2020-03-31
置信水平: 95.0%

股票 000333:
  当前价格: 58.2300
  持仓市值: 5823.00
  95% VaR: 173.60
  对应收益率: -2.9813%

股票 300059:
  当前价格: 15.7900
  持仓市值: 1579.00
  95% VaR: 65.72
  对应收益率: -4.1618%

股票 600036:
  当前价格: 37.8300
  持仓市值: 3783.00
  95% VaR: 89.96
  对应收益率: -2.3781%

组合:
  总市值: 11185.00
  95% VaR: 262.30

回测结果:
  回测天数: 58
  期望突破次数: 2.9
  股票 000333 突破次数: 7
  股票 300059 突破次数: 9
  股票 600036 突破次数: 7
  组合突破次数: 7
