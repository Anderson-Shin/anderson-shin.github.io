---
layout: talk        
collection: data_analysis
permalink: /data_analysis/portfolio_risk_estimation_practice1
title: "Coherent & Spectral Risk Management Practice Using Python"
date: 2025-05-23
excerpt: '…'
venue: "Data Analysis Post"
location: ""
---
# Coherent & Spectral Risk Management Practice Using Python

## 1. Imports & Configuration

```python
import yfinance as yf
import numpy as np
import pandas as pd
from scipy.stats import norm
import matplotlib.pyplot as plt
```

```
# (No direct output)
```

### Expert Analysis
- **Mean Daily Return**: 0.1581% indicates a modest positive drift.
- **Std. Dev.**: 1.5906% shows moderate volatility.
- **Daily VaR (95%)**:
  $$
  \mathrm{VaR}_{95\%} = -F_R^{-1}(0.05)
  $$
  = 2.5966% loss, meaning one in twenty days we expect ≥2.60% drop.
- **Daily ES (95%)**:
  $$
  \mathrm{ES}_{95\%} = -\frac{1}{0.05}\int_{0}^{0.05} F_R^{-1}(u)\,du
  $$
  = 3.5517% average loss beyond VaR, capturing tail severity.
- **Daily SRM (γ=1.0)**:
  $$
  \mathrm{SRM}_{\gamma=1} = \sum_{i=1}^N w_i L_{(i)} \phi_{\gamma}(p_i)
  $$
  = 0.5839% emphasizing tail losses via exponential weighting.

A γ of 0.40 aligns annual SRM to –20%, matching a predefined risk budget. This corresponds to solving:
$$
SRM_{\mathrm{ann}}(\gamma) = \sqrt{252}\sum_{i=1}^N w_i L_{(i)} \phi_{\gamma}\Bigl(\frac{i}{N}\Bigr) = -0.20
$$

## 2. Data Download & Portfolio Setup

```python
# 1. Basic settings
tickers = ["AAPL", "TSLA", "AMZN", "MSFT"]
start_date = "2024-01-01"
end_date = "2025-01-01"
initial_budget = 1_000_000
weights = np.array([0.25, 0.25, 0.25, 0.25])  # equal allocation

# 2. Download price data
data = yf.download(tickers, start=start_date, end=end_date)["Close"]
returns = data.pct_change().dropna()

# 3. Compute portfolio returns
portfolio_returns = (returns * weights).sum(axis=1)

# 4. Summary statistics
mean_daily = portfolio_returns.mean()
std_daily = portfolio_returns.std()
var_daily = portfolio_returns.var()
```

```
[*********************100%***********************]  4 of 4 completed
```

### Expert Analysis
- **Mean Daily Return**: 0.1581% indicates a modest positive drift.
- **Std. Dev.**: 1.5906% shows moderate volatility.
- **Daily VaR (95%)**:
  $$
  \mathrm{VaR}_{95\%} = -F_R^{-1}(0.05)
  $$
  = 2.5966% loss, meaning one in twenty days we expect ≥2.60% drop.
- **Daily ES (95%)**:
  $$
  \mathrm{ES}_{95\%} = -\frac{1}{0.05}\int_{0}^{0.05} F_R^{-1}(u)\,du
  $$
  = 3.5517% average loss beyond VaR, capturing tail severity.
- **Daily SRM (γ=1.0)**:
  $$
  \mathrm{SRM}_{\gamma=1} = \sum_{i=1}^N w_i L_{(i)} \phi_{\gamma}(p_i)
  $$
  = 0.5839% emphasizing tail losses via exponential weighting.

A γ of 0.40 aligns annual SRM to –20%, matching a predefined risk budget. This corresponds to solving:
$$
SRM_{\mathrm{ann}}(\gamma) = \sqrt{252}\sum_{i=1}^N w_i L_{(i)} \phi_{\gamma}\Bigl(\frac{i}{N}\Bigr) = -0.20
$$

## 3. Value-at-Risk (VaR), Expected Shortfall (ES) & Spectral Risk Measure (SRM)

```python
# 5. VaR, ES calculation (95%)
alpha = 0.95
VaR_daily = -np.percentile(portfolio_returns, 100 * (1 - alpha))
es_daily = -portfolio_returns[portfolio_returns <= -VaR_daily].mean()

# 6. SRM calculation functions
def phi_gamma(p, gamma):
    return np.exp(-(1 - p) / gamma) / (gamma * (1 - np.exp(-1 / gamma)))

def compute_srm(returns, gamma):
    sorted_losses = -np.sort(returns)
    n = len(sorted_losses)
    p = np.arange(1, n + 1) / n
    weights = phi_gamma(p, gamma)
    weights /= weights.sum()
    return np.sum(weights * sorted_losses)

# Compute SRM with γ = 1.0
srm_daily = compute_srm(portfolio_returns.values, gamma=1.0)

# 7. Print summary
summary = {
    "Mean Return": mean_daily,
    "Standard Deviation": std_daily,
    "Variance": var_daily,
    "VaR (95%)": VaR_daily,
    "Expected Shortfall (95%)": es_daily,
    "SRM (γ=1.0)": srm_daily
}
summary_df = pd.DataFrame.from_dict(summary, orient='index', columns=['Value'])
print("=== Portfolio Risk Summary ===")
print(summary_df)
```

```
=== Portfolio Risk Summary ===
                             Value
Mean Return               0.001581
Standard Deviation        0.015906
Variance                  0.000253
VaR (95%)                 0.025966
Expected Shortfall (95%)  0.035517
SRM (γ=1.0)              -0.005839
```

### Expert Analysis
- **Mean Daily Return**: 0.1581% indicates a modest positive drift.
- **Std. Dev.**: 1.5906% shows moderate volatility.
- **Daily VaR (95%)**:
  $$
  \mathrm{VaR}_{95\%} = -F_R^{-1}(0.05)
  $$
  = 2.5966% loss, meaning one in twenty days we expect ≥2.60% drop.
- **Daily ES (95%)**:
  $$
  \mathrm{ES}_{95\%} = -\frac{1}{0.05}\int_{0}^{0.05} F_R^{-1}(u)\,du
  $$
  = 3.5517% average loss beyond VaR, capturing tail severity.
- **Daily SRM (γ=1.0)**:
  $$
  \mathrm{SRM}_{\gamma=1} = \sum_{i=1}^N w_i L_{(i)} \phi_{\gamma}(p_i)
  $$
  = 0.5839% emphasizing tail losses via exponential weighting.

A γ of 0.40 aligns annual SRM to –20%, matching a predefined risk budget. This corresponds to solving:
$$
SRM_{\mathrm{ann}}(\gamma) = \sqrt{252}\sum_{i=1}^N w_i L_{(i)} \phi_{\gamma}\Bigl(\frac{i}{N}\Bigr) = -0.20
$$

## 4. Annualization

```python
trading_days = 252
mean_annual = mean_daily * trading_days
std_annual = std_daily * np.sqrt(trading_days)
VaR_annual = VaR_daily * np.sqrt(trading_days)
es_annual = es_daily * np.sqrt(trading_days)
srm_annual = srm_daily * np.sqrt(trading_days)

annual_summary = pd.DataFrame({
    'Metric': [
        'Mean Annual Return', 'Std Dev Annual', 
        'VaR (95%) Annual', 'ES (95%) Annual', 'SRM (γ=1) Annual'
    ],
    'Value': [
        mean_annual, std_annual, VaR_annual, es_annual, srm_annual
    ]
})
print(annual_summary)
```

```
           Metric     Value
0  Mean Annual Return  0.398486
1    Std Dev Annual    0.252493
2  VaR (95%) Annual    0.412198
3  ES (95%) Annual     0.563808
4  SRM (γ=1) Annual   -0.092693
```

### Expert Analysis
- **Mean Daily Return**: 0.1581% indicates a modest positive drift.
- **Std. Dev.**: 1.5906% shows moderate volatility.
- **Daily VaR (95%)**:
  $$
  \mathrm{VaR}_{95\%} = -F_R^{-1}(0.05)
  $$
  = 2.5966% loss, meaning one in twenty days we expect ≥2.60% drop.
- **Daily ES (95%)**:
  $$
  \mathrm{ES}_{95\%} = -\frac{1}{0.05}\int_{0}^{0.05} F_R^{-1}(u)\,du
  $$
  = 3.5517% average loss beyond VaR, capturing tail severity.
- **Daily SRM (γ=1.0)**:
  $$
  \mathrm{SRM}_{\gamma=1} = \sum_{i=1}^N w_i L_{(i)} \phi_{\gamma}(p_i)
  $$
  = 0.5839% emphasizing tail losses via exponential weighting.

A γ of 0.40 aligns annual SRM to –20%, matching a predefined risk budget. This corresponds to solving:
$$
SRM_{\mathrm{ann}}(\gamma) = \sqrt{252}\sum_{i=1}^N w_i L_{(i)} \phi_{\gamma}\Bigl(\frac{i}{N}\Bigr) = -0.20
$$

## 5. Return Distribution & Tail-Risk Visualization

```python
plt.figure(figsize=(10, 6))
plt.hist(portfolio_returns, bins=50, density=True, alpha=0.6)
plt.axvline(-VaR_daily, linestyle='--', label='VaR (95%)')
plt.axvline(-es_daily, linestyle='--', label='ES (95%)')
plt.show()
```

![Hist with VaR/ES](path/to/hist_var_es.png)

### Expert Analysis
- **Mean Daily Return**: 0.1581% indicates a modest positive drift.
- **Std. Dev.**: 1.5906% shows moderate volatility.
- **Daily VaR (95%)**:
  $$
  \mathrm{VaR}_{95\%} = -F_R^{-1}(0.05)
  $$
  = 2.5966% loss, meaning one in twenty days we expect ≥2.60% drop.
- **Daily ES (95%)**:
  $$
  \mathrm{ES}_{95\%} = -\frac{1}{0.05}\int_{0}^{0.05} F_R^{-1}(u)\,du
  $$
  = 3.5517% average loss beyond VaR, capturing tail severity.
- **Daily SRM (γ=1.0)**:
  $$
  \mathrm{SRM}_{\gamma=1} = \sum_{i=1}^N w_i L_{(i)} \phi_{\gamma}(p_i)
  $$
  = 0.5839% emphasizing tail losses via exponential weighting.

A γ of 0.40 aligns annual SRM to –20%, matching a predefined risk budget. This corresponds to solving:
$$
SRM_{\mathrm{ann}}(\gamma) = \sqrt{252}\sum_{i=1}^N w_i L_{(i)} \phi_{\gamma}\Bigl(\frac{i}{N}\Bigr) = -0.20
$$

## 6. Calibrating γ for Specific Risk Appetite

```python
# ...
```

```
▶ Gamma value for risk tolerance of –20%: γ = 0.40
  - Daily SRM: –1.15%
  - Annual SRM: –18.29%
```

### Expert Analysis
- **Mean Daily Return**: 0.1581% indicates a modest positive drift.
- **Std. Dev.**: 1.5906% shows moderate volatility.
- **Daily VaR (95%)**:
  $$
  \mathrm{VaR}_{95\%} = -F_R^{-1}(0.05)
  $$
  = 2.5966% loss, meaning one in twenty days we expect ≥2.60% drop.
- **Daily ES (95%)**:
  $$
  \mathrm{ES}_{95\%} = -\frac{1}{0.05}\int_{0}^{0.05} F_R^{-1}(u)\,du
  $$
  = 3.5517% average loss beyond VaR, capturing tail severity.
- **Daily SRM (γ=1.0)**:
  $$
  \mathrm{SRM}_{\gamma=1} = \sum_{i=1}^N w_i L_{(i)} \phi_{\gamma}(p_i)
  $$
  = 0.5839% emphasizing tail losses via exponential weighting.

A γ of 0.40 aligns annual SRM to –20%, matching a predefined risk budget. This corresponds to solving:
$$
SRM_{\mathrm{ann}}(\gamma) = \sqrt{252}\sum_{i=1}^N w_i L_{(i)} \phi_{\gamma}\Bigl(\frac{i}{N}\Bigr) = -0.20
$$

## 7. Spectral Risk vs. γ

```python
# ...
```

![SRM vs Gamma](path/to/srm_vs_gamma.png)

### Expert Analysis
- **Mean Daily Return**: 0.1581% indicates a modest positive drift.
- **Std. Dev.**: 1.5906% shows moderate volatility.
- **Daily VaR (95%)**:
  $$
  \mathrm{VaR}_{95\%} = -F_R^{-1}(0.05)
  $$
  = 2.5966% loss, meaning one in twenty days we expect ≥2.60% drop.
- **Daily ES (95%)**:
  $$
  \mathrm{ES}_{95\%} = -\frac{1}{0.05}\int_{0}^{0.05} F_R^{-1}(u)\,du
  $$
  = 3.5517% average loss beyond VaR, capturing tail severity.
- **Daily SRM (γ=1.0)**:
  $$
  \mathrm{SRM}_{\gamma=1} = \sum_{i=1}^N w_i L_{(i)} \phi_{\gamma}(p_i)
  $$
  = 0.5839% emphasizing tail losses via exponential weighting.

A γ of 0.40 aligns annual SRM to –20%, matching a predefined risk budget. This corresponds to solving:
$$
SRM_{\mathrm{ann}}(\gamma) = \sqrt{252}\sum_{i=1}^N w_i L_{(i)} \phi_{\gamma}\Bigl(\frac{i}{N}\Bigr) = -0.20
$$

## 8. Advanced Optimization under SRM Constraint

```python
# Optimization code
```

```
▶ γ = 2.28-based portfolio under SRM ≤ –20% constraint:
  - AAPL: 0.0000
  - TSLA: 0.0000
  - AMZN: 0.0000
  - MSFT: 1.0000

[Annualized Metrics]
 - Mean Return: 68.56%
 - Std Dev: 63.49%
 - VaR (95%): 83.33%
 - ES (95%): 123.13%
```

### Expert Analysis
- **Mean Daily Return**: 0.1581% indicates a modest positive drift.
- **Std. Dev.**: 1.5906% shows moderate volatility.
- **Daily VaR (95%)**:
  $$
  \mathrm{VaR}_{95\%} = -F_R^{-1}(0.05)
  $$
  = 2.5966% loss, meaning one in twenty days we expect ≥2.60% drop.
- **Daily ES (95%)**:
  $$
  \mathrm{ES}_{95\%} = -\frac{1}{0.05}\int_{0}^{0.05} F_R^{-1}(u)\,du
  $$
  = 3.5517% average loss beyond VaR, capturing tail severity.
- **Daily SRM (γ=1.0)**:
  $$
  \mathrm{SRM}_{\gamma=1} = \sum_{i=1}^N w_i L_{(i)} \phi_{\gamma}(p_i)
  $$
  = 0.5839% emphasizing tail losses via exponential weighting.

A γ of 0.40 aligns annual SRM to –20%, matching a predefined risk budget. This corresponds to solving:
$$
SRM_{\mathrm{ann}}(\gamma) = \sqrt{252}\sum_{i=1}^N w_i L_{(i)} \phi_{\gamma}\Bigl(\frac{i}{N}\Bigr) = -0.20
$$

## Limitations & Future Improvements

1. **Asset Universe Expansion**: Include fixed income, alternatives, commodities.  
2. **Dynamic Volatility Modeling**: Use GARCH or Monte Carlo for regime shifts.  
3. **Transaction & Liquidity Costs**: Model real-world trading frictions.  
4. **Robust Optimization**: Incorporate diversification limits and multi-objective frameworks to avoid corner solutions.  
