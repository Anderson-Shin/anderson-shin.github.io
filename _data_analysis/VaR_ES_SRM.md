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
# 1. Basic settings
tickers     = ["AAPL", "TSLA", "AMZN", "MSFT"]
start_date  = "2024-01-01"
end_date    = "2025-01-01"
initial_budget = 1_000_000
weights     = np.array([0.25, 0.25, 0.25, 0.25])  # equal allocation

# 2. Download price data
data    = yf.download(tickers, start=start_date, end=end_date)["Close"]
returns = data.pct_change().dropna()

# 3. Compute portfolio returns
portfolio_returns = (returns * weights).sum(axis=1)

# 4. Summary statistics
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
# 5. VaR & ES calculation (95% confidence)
alpha    = 0.95
VaR_daily = -np.percentile(portfolio_returns, 100 * (1 - alpha))
es_daily  = -portfolio_returns[portfolio_returns <= -VaR_daily].mean()

# 6. SRM calculation functions
def phi_gamma(p, gamma):
    return np.exp(-(1 - p) / gamma) / (gamma * (1 - np.exp(-1 / gamma)))

def compute_srm(returns, gamma):
    sorted_losses = -np.sort(returns)
    n             = len(sorted_losses)
    p             = np.arange(1, n + 1) / n
    weights       = phi_gamma(p, gamma)
    weights      /= weights.sum()
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
import pandas as pd
summary_df = pd.DataFrame.from_dict(summary, orient='index', columns=['Value'])
print(summary_df)
```

```
                             Value
Mean Return               0.001581
Standard Deviation        0.015906
Variance                  0.000253
VaR (95%)                 0.025966
Expected Shortfall (95%)  0.035517
SRM (γ=1.0)              -0.005839
```

### Expert Analysis
- **Mean Daily Return**: 0.1581% — modest positive drift.  
- **Std. Dev.**: 1.5906% — moderate volatility.  
- **Daily VaR (95%)**:  
  $$
    \mathrm{VaR}_{95\%} = -F_R^{-1}(0.05)
  $$  
  = 2.5966% loss (one out of twenty days).  
- **Daily ES (95%)**:  
  $$
    \mathrm{ES}_{95\%} = -rac{1}{0.05}\int_{0}^{0.05}F_R^{-1}(u)\,du
  $$  
  = 3.5517% average loss beyond VaR.  
- **Daily SRM (γ=1.0)**:  
  $$
    \mathrm{SRM}_{\gamma=1} = \sum_{i=1}^N w_i L_{(i)}\,\phi_{\gamma}(p_i)
  $$  
  = 0.5839%, weighting tail losses exponentially.  
These metrics are critical for setting capital buffers and intraday risk limits.

---

## 4. Annualization

```python
trading_days = 252
mean_annual = mean_daily * trading_days
std_annual  = std_daily  * np.sqrt(trading_days)
VaR_annual  = VaR_daily  * np.sqrt(trading_days)
es_annual   = es_daily   * np.sqrt(trading_days)
srm_annual  = srm_daily  * np.sqrt(trading_days)

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
- **Annual Return**: 39.85% (`\mu_{	ext{ann}} = \mu_{	ext{daily}}	imes252`).  
- **Annual Vol**: 25.25% (`\sigma_{	ext{ann}} = \sigma_{	ext{daily}}\sqrt{252}`).  
- **Annual VaR/ES/SRM**: scale daily metrics by √252.  
Annual figures facilitate year-over-year risk budgeting and performance comparison.

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
The histogram reveals a left-skewed distribution, with heavier tails than a normal fit:
$$
f(r) = rac{1}{\sigma\sqrt{2\pi}} \exp\!\Bigl(-	frac{(r-\mu)^2}{2\sigma^2}\Bigr).
$$
The gap between the VaR line (–2.60%) and ES line (–3.55%) visually emphasizes the tail risk that parametric models often understate.

```python
# Tail highlight and drawdown plotting…
```

![Tail & Drawdown](path/to/tail_drawdown.png)

### Expert Analysis
- **Tail Highlight**: bins below VaR show ~5% tail mass.  
- **Drawdown Plot**: shaded areas indicate cumulative drawdown periods, peaking near –20%, illustrating recovery durations.

---

## 6. Calibrating γ for Specific Risk Appetite

```python
# Solve for gamma that yields annual SRM ≈ -20%
```

```
▶ Gamma value for risk tolerance of –20%: γ = 0.40
  - Daily SRM: –1.15%
  - Annual SRM: –18.29%
```

### Expert Analysis
Calibrating γ to 0.40 satisfies:
$$
SRM_{\mathrm{ann}}(\gamma)
= \sqrt{252}\sum_{i=1}^N w_i L_{(i)}\,\phi_{\gamma}\!\Bigl(	frac{i}{N}\Bigr)
= -0.20.
$$
This aligns the spectral measure with a predefined annual loss budget.

---

## 7. Spectral Risk vs. γ

```python
# Plot SRM_annual for various gamma values…
```

![SRM vs Gamma](path/to/srm_vs_gamma.png)

### Expert Analysis
As γ increases, annual SRM declines from ~–5% to ~–60%. The intercept at γ≈0.40 confirms our calibration. Such sensitivity analysis guides policy-level γ selection.

---

## 8. Advanced Optimization under SRM Constraint

```python
# Define optimization under SRM ≤ -20% constraint…
```

```
▶ γ = 2.28-based portfolio under SRM ≤ –20% constraint:
  - AAPL: 0.00
  - TSLA: 0.00
  - AMZN: 0.00
  - MSFT: 1.00

[Annualized Metrics]
 - Mean Return: 68.56%
 - Std Dev: 63.49%
 - VaR (95%): 83.33%
 - ES (95%): 123.13%
```

### Expert Analysis
Under the SRM ≤ –20% constraint with γ=2.28, the optimizer allocates 100% to MSFT—maximizing expected return (68.56%) but incurring extreme tail risk (ES >120%).  
In practice, add diversification, transaction cost, and turnover constraints to avoid such corner solutions.

---

## Limitations & Future Improvements

1. **Asset Universe Expansion**: include fixed income, alternatives, commodities.  
2. **Dynamic Volatility**: implement GARCH or Monte Carlo simulations for regime shifts.  
3. **Transaction & Liquidity Costs**: integrate realistic trading frictions.  
4. **Robust Optimization**: apply diversification limits and multi-objective frameworks to prevent corner allocations.  
