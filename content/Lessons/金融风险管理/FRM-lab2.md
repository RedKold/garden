
```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
08康美债 债券定价与关键利率久期计算
Bond Pricing and Key Rate Duration Analysis
"""

import numpy as np
from scipy.optimize import brentq
from scipy.interpolate import interp1d
from datetime import datetime, timedelta


# ============================================================================
# 1. 债券基本信息 / Bond Information
# ============================================================================

# 日期信息
settlement_date = datetime(2008, 7, 17)  # 结算日
issue_date = datetime(2008, 5, 26)       # 发行日

# 现金流信息
cash_flow_dates = [
    datetime(2009, 5, 26),
    datetime(2010, 5, 26),
    datetime(2011, 5, 26),
    datetime(2012, 5, 26),
    datetime(2013, 5, 26),
    datetime(2014, 5, 26),
]

cash_flows = [0.8, 0.8, 0.8, 0.8, 0.8, 100.8]  # 现金流金额

dirty_price = 70.11  # 全价
coupon_rate = 0.8    # 年票面利率（元）

# 利率曲线信息（关键年限和对应的即期利率）
key_tenors = np.array([0, 1, 2, 3, 5, 7])  # 关键年限
spot_rates = np.array([0.0315, 0.0523, 0.0557, 0.0576, 0.0590, 0.0600])  # 即期利率


# ============================================================================
# 2. 计算应付利息和净价 / Calculate Accrued Interest and Clean Price
# ============================================================================

def calculate_accrued_interest(settlement_date, last_coupon_date, next_coupon_date, coupon):
    """计算应付利息（实际天数/365）"""
    days_accrued = (settlement_date - last_coupon_date).days
    days_in_period = (next_coupon_date - last_coupon_date).days
    accrued = coupon * days_accrued / 365
    return accrued

# 上一付息日是发行日
last_coupon = issue_date
next_coupon = cash_flow_dates[0]
accrued_interest = calculate_accrued_interest(settlement_date, last_coupon, next_coupon, coupon_rate)
clean_price = dirty_price - accrued_interest

print("=" * 60)
print("=== 债券基本信息 ===")
print("=" * 60)
print(f"结算日期: {settlement_date.strftime('%Y-%m-%d')}")
print(f"全价: {dirty_price:.2f}")
print(f"应付利息: {accrued_interest:.4f}")
print(f"净价: {clean_price:.4f}")
print()


# ============================================================================
# 3. 计算到期收益率 (YTM) / Calculate Yield to Maturity
# ============================================================================

def calculate_years_to_cashflow(settlement_date, cashflow_date):
    """计算从结算日到现金流日期的年限（实际天数/365）"""
    days = (cashflow_date - settlement_date).days
    return days / 365.0

def bond_price_from_ytm(ytm, settlement_date, cf_dates, cf_amounts):
    """根据YTM计算债券价格"""
    pv = 0
    for cf_date, cf_amount in zip(cf_dates, cf_amounts):
        t = calculate_years_to_cashflow(settlement_date, cf_date)
        pv += cf_amount / ((1 + ytm) ** t)
    return pv

# 求解YTM：使得现值等于净价
def ytm_objective(ytm):
    return bond_price_from_ytm(ytm, settlement_date, cash_flow_dates, cash_flows) - clean_price

ytm = brentq(ytm_objective, 0.01, 0.20)  # 在1%到20%之间搜索

print("=" * 60)
print("=== 到期收益率 ===")
print("=" * 60)
print(f"YTM: {ytm * 100:.4f}%")
print()


# ============================================================================
# 4. 计算理论价格 / Calculate Theoretical Price
# ============================================================================

# 构建利率曲线插值函数
rate_curve = interp1d(key_tenors, spot_rates, kind='linear', fill_value='extrapolate')

def calculate_theoretical_price(settlement_date, cf_dates, cf_amounts, rate_curve):
    """使用利率曲线计算理论价格"""
    pv = 0
    for cf_date, cf_amount in zip(cf_dates, cf_amounts):
        t = calculate_years_to_cashflow(settlement_date, cf_date)
        spot_rate = rate_curve(t)
        discount_factor = 1 / ((1 + spot_rate) ** t)
        pv += cf_amount * discount_factor
    return pv

theoretical_price = calculate_theoretical_price(settlement_date, cash_flow_dates, cash_flows, rate_curve)

print("=" * 60)
print("=== 理论价格 ===")
print("=" * 60)
print(f"理论价格: {theoretical_price:.4f}")
print()


# ============================================================================
# 5. 计算久期、修正久期和凸度 / Calculate Duration and Convexity
# ============================================================================

def calculate_duration_convexity(ytm, settlement_date, cf_dates, cf_amounts):
    """使用数值方法计算久期和凸度"""
    dy = 0.0001  # 1bp扰动

    # 计算三个点的价格
    P0 = bond_price_from_ytm(ytm, settlement_date, cf_dates, cf_amounts)
    P_up = bond_price_from_ytm(ytm + dy, settlement_date, cf_dates, cf_amounts)
    P_down = bond_price_from_ytm(ytm - dy, settlement_date, cf_dates, cf_amounts)

    # 修正久期（Modified Duration）= -1/P * dP/dy
    modified_duration = -(P_up - P_down) / (2 * dy * P0)

    # 久期（Macaulay Duration）= 修正久期 * (1 + ytm)
    duration = modified_duration * (1 + ytm)

    # 凸度（Convexity）
    convexity = (P_up + P_down - 2 * P0) / ((dy ** 2) * P0)

    return duration, modified_duration, convexity

duration, modified_duration, convexity = calculate_duration_convexity(
    ytm, settlement_date, cash_flow_dates, cash_flows
)

print("=" * 60)
print("=== 久期和凸度（假设利率平行移动）===")
print("=" * 60)
print(f"久期: {duration:.4f}")
print(f"修正久期: {modified_duration:.4f}")
print(f"凸度: {convexity:.4f}")
print()


# ============================================================================
# 6. 计算关键利率久期 / Calculate Key Rate Duration
# ============================================================================

def calculate_key_rate_duration(key_tenors, spot_rates, settlement_date, cf_dates, cf_amounts):
    """计算关键利率久期"""
    # 基准理论价格
    base_curve = interp1d(key_tenors, spot_rates, kind='linear', fill_value='extrapolate')
    P0 = calculate_theoretical_price(settlement_date, cf_dates, cf_amounts, base_curve)

    key_rate_durations = []
    dy = 0.0001  # 1bp扰动

    for i, tenor in enumerate(key_tenors):
        # 上移该关键点利率1bp
        rates_up = spot_rates.copy()
        rates_up[i] += dy
        curve_up = interp1d(key_tenors, rates_up, kind='linear', fill_value='extrapolate')
        P_up = calculate_theoretical_price(settlement_date, cf_dates, cf_amounts, curve_up)

        # 下移该关键点利率1bp
        rates_down = spot_rates.copy()
        rates_down[i] -= dy
        curve_down = interp1d(key_tenors, rates_down, kind='linear', fill_value='extrapolate')
        P_down = calculate_theoretical_price(settlement_date, cf_dates, cf_amounts, curve_down)

        # 关键利率久期
        krd = -(P_up - P_down) / (2 * dy * P0)
        key_rate_durations.append(krd)

    return key_rate_durations

key_rate_durations = calculate_key_rate_duration(
    key_tenors, spot_rates, settlement_date, cash_flow_dates, cash_flows
)

print("=" * 60)
print("=== 关键利率久期 ===")
print("=" * 60)
for tenor, krd in zip(key_tenors, key_rate_durations):
    print(f"{int(tenor)}年关键利率久期: {krd:.4f}")
print(f"\n关键利率久期之和: {sum(key_rate_durations):.4f}")
print()


# ============================================================================
# 7. 验证 / Verification
# ============================================================================

print("=" * 60)
print("=== 验证 ===")
print("=" * 60)
print(f"修正久期: {modified_duration:.4f}")
print(f"关键利率久期之和: {sum(key_rate_durations):.4f}")
print(f"差异: {abs(modified_duration - sum(key_rate_durations)):.4f}")
print("=" * 60)
```


# Result
![image.png|400](https://kold.oss-cn-shanghai.aliyuncs.com/20260402092438.png)
