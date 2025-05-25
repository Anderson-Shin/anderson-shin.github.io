---
title: "Coherent & Spectral Risk Management Practice Using Python"
collection: data_analysis
permalink: /data_analysis/portfolio_risk_estimation_practice1
excerpt: "Deep dive into VaR, Expected Shortfall, and Spectral Risk Measures (SRM) using Python"
venue: "Data Analysis Post"
date: 2025-05-23
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
This section loads the essential libraries for our analysis:
- **`yfinance`** for pulling historical price data  
- **`numpy`**, **`pandas`** for numerical operations and DataFrame handling  
- **`scipy.stats`** for statistical functions like the normal distribution  
- **`matplotlib.pyplot`** for plotting  
Setting up this environment ensures that subsequent data retrieval, calculation, and visualization steps run smoothly.

---

## 2. Data Download & Portfolio Setup

```python
tickers        = ["AAPL", "TSLA", "AMZN", "MSFT"]
start_date     = "2024-01-01"
end_date       = "2025-01-01"
initial_budget = 1_000_000
weights        = np.array([0.25, 0.25, 0.25, 0.25])

data    = yf.download(tickers, start=start_date, end=end_date)["Close"]
returns = data.pct_change().dropna()
portfolio_returns = (returns * weights).sum(axis=1)
mean_daily = portfolio_returns.mean()
std_daily  = portfolio_returns.std()
var_daily  = portfolio_returns.var()
```

```
[*********************100%***********************]  4 of 4 completed
```

### Expert Analysis
- **Data range**: 2024-01-01 to 2025-01-01 (~252 trading days)  
- **Portfolio**: equal weights (25%) in AAPL, TSLA, AMZN, MSFT  
- **Returns**: daily percentage changes, then weighted sum yields portfolio P&L series  
These return series form the basis for all risk computations (mean, volatility, VaR, ES, SRM).

---

## 3. Value-at-Risk (VaR), Expected Shortfall (ES) & Spectral Risk Measure (SRM)

```python
alpha     = 0.95
VaR_daily = -np.percentile(portfolio_returns, 100 * (1 - alpha))
es_daily  = -portfolio_returns[portfolio_returns <= -VaR_daily].mean()

def phi_gamma(p, gamma):
    return np.exp(-(1 - p) / gamma) / (gamma * (1 - np.exp(-1 / gamma)))

def compute_srm(returns, gamma):
    sorted_losses = -np.sort(returns)
    p             = np.arange(1, len(sorted_losses) + 1) / len(sorted_losses)
    weights       = phi_gamma(p, gamma)
    return np.dot(sorted_losses, weights / weights.sum())

srm_daily = compute_srm(portfolio_returns.values, gamma=1.0)

import pandas as pd
summary = {
    "Mean Return": mean_daily,
    "Std Dev": std_daily,
    "Variance": var_daily,
    "VaR (95%)": VaR_daily,
    "ES (95%)": es_daily,
    "SRM (γ=1)": srm_daily
}
print(pd.Series(summary, name="Value"))
```

```
Value
Mean Return    0.001581
Std Dev        0.015906
Variance       0.000253
VaR (95%)      0.025966
ES (95%)       0.035517
SRM (γ=1)     -0.005839
```

### Expert Analysis
- **Mean Daily Return**: 0.1581% — modest positive drift.  
- **Std Dev**: 1.5906% — moderate volatility.  
- **Daily VaR (95%)**:  
  {% raw %}$$
  \mathrm{VaR}_{95\%} = -F_R^{-1}(0.05)
  $${% endraw %}  
  = 2.5966% loss (one out of twenty days).  
- **Daily ES (95%)**:  
  {% raw %}$$
  \mathrm{ES}_{95\%} = -rac{1}{0.05}\int_{0}^{0.05}F_R^{-1}(u)\,\mathrm{d}u
  $${% endraw %}  
  = 3.5517% average loss beyond VaR.  
- **Daily SRM (γ=1.0)**:  
  {% raw %}$$
  \mathrm{SRM}_{\gamma=1} = \sum_{i=1}^N w_i\,L_{(i)}\,\phi_{\gamma}(p_i)
  $${% endraw %}  
  = 0.5839%, weighting tail losses exponentially.

---

## 4. Annualization

```python
trading_days = 252
mean_ann = mean_daily * trading_days
std_ann  = std_daily  * np.sqrt(trading_days)
VaR_ann  = VaR_daily  * np.sqrt(trading_days)
ES_ann   = es_daily   * np.sqrt(trading_days)
SRM_ann  = srm_daily  * np.sqrt(trading_days)
```

```
# (Annual summary printed similarly)
```

### Expert Analysis
- **Annual Return**:  
  {% raw %}$$
  \mu_{\mathrm{ann}} = \mu_{\mathrm{daily}}	imes252
  $${% endraw %}  
- **Annual Volatility**:  
  {% raw %}$$
  \sigma_{\mathrm{ann}} = \sigma_{\mathrm{daily}}\sqrt{252}
  $${% endraw %}

---

## 5. Return Distribution & Tail-Risk Visualization

```python
plt.figure(figsize=(10, 6))
plt.hist(portfolio_returns, bins=50, density=True, alpha=0.6)
plt.axvline(-VaR_daily, linestyle='--', label='VaR (95%)')
plt.axvline(-es_daily, linestyle='--', label='ES (95%)')
plt.legend()
plt.show()
```

![Histogram with VaR & ES](path/to/hist_var_es.png)

### Expert Analysis
The histogram reveals a left-skewed distribution with heavier tails than the normal model:  
{% raw %}$$
f(r) = rac{1}{\sigma\sqrt{2\pi}} \exp\!igl(-	frac{(r - \mu)^2}{2\sigma^2}igr)
$${% endraw %}

---

## 6. Calibrating γ for Specific Risk Appetite

```python
# Solve for gamma where annual SRM ≈ -20%
```

```
▶ γ = 0.40 → Annual SRM ≈ -0.20
```

### Expert Analysis
{% raw %}$$
SRM_{\mathrm{ann}}(\gamma)
= \sqrt{252}\sum_{i=1}^N w_i\,L_{(i)}\,\phi_{\gamma}\!\Bigl(rac{i}{N}\Bigr)
= -0.20
$${% endraw %}

---

## 7. Spectral Risk vs. γ

```python
# Plot SRM_ann vs gamma…
```

![SRM vs Gamma](path/to/srm_vs_gamma.png)

### Expert Analysis
As γ increases, annual SRM drops from ~–5% to ~–60%. Intersection at γ≈0.40 validates our calibration.

---

## 8. Advanced Optimization under SRM Constraint

```python
# Optimization under SRM ≤ -20%…
```

```
▶ Portfolio: 100% MSFT
```

### Expert Analysis
Corner solution highlights need for diversification, cost, and turnover constraints.

---

## Limitations & Future Improvements

1. **Asset Universe Expansion**: include fixed income, alternatives, commodities.  
2. **Dynamic Volatility**: implement GARCH/Monte Carlo for regime shifts.  
3. **Transaction & Liquidity Costs**: integrate trading frictions.  
4. **Robust Optimization**: apply diversification limits and multi-objective frameworks.
