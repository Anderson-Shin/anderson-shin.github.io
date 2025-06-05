---
title: "Non-parametric methods for VaR, ES estimation"
collection: data_analysis
permalink: /data_analysis/portfolio_risk_estimation_practice2
excerpt: "Non-parametric approach utilizing different weighted bootstrap methodologies"
venue: "Data Analysis Post"
date: 2025-05-30
location: ""
---

# Weighted Bootstrap VaR & ES Analysis Using Portfolio with stocks in various sectors

This blog post explores advanced techniques for estimating **Value at Risk (VaR)** and **Expected Shortfall (ES)** using **weighted bootstrap** methods. We incorporate age-weighted, volatility-weighted, and correlation-weighted sampling strategies and compute both **Percentile intervals** and **BCa confidence intervals**, supported by diagnostic visualizations and financial intuition.

---

## 1. Portfolio Construction and Summary Statistics

### 1.1. Global Settings and Imports
```python
import yfinance as yf
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from scipy.stats import norm  # BCa CI calculation

# Global parameters
tickers = ['AAPL','JPM','XOM','UNH','HD','BA','DUK','AMT','NEM','DIS']
start_date = "2020-01-01"
end_date = "2025-05-31"
windows = [10, 21, 63]       # Lookback windows
alpha_var = 0.01            # VaR significance level
alpha_ci = 0.05             # CI significance level (95% CI)
n_bootstrap = 10000         # Number of bootstrap samples
```  

### 1.2. Ticker Sector Allocation and Data Download
```python
# Download adjusted closing prices
data = yf.download(tickers, start=start_date, end=end_date)["Close"]

# Calculate daily returns and equally weighted portfolio returns
returns = data.pct_change().dropna()
weights_eq = np.ones(len(tickers)) / len(tickers)
portfolio_returns = returns.dot(weights_eq)
sample = portfolio_returns.values  # 1D numpy array for bootstrap

# Compute performance metrics: daily mean, daily std, annualized return, volatility, max drawdown
trading_days = 252
stats = pd.DataFrame({
    'Daily Mean': portfolio_returns.mean(),
    'Daily Std': portfolio_returns.std(),
    'Annual Return': portfolio_returns.mean() * trading_days,
    'Annual Volatility': portfolio_returns.std() * np.sqrt(trading_days),
    'Max Drawdown': ((1 + portfolio_returns).cumprod() / (1 + portfolio_returns).cumprod().cummax() - 1).min()
}, index=[0]).T
display(stats)
```  

![port_performance](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/data_analysis_img/Weighted_Bootstrap/port_performance.png?raw=True)

**Remark & Insight:**  
- **Return‐to‐Volatility Profile:** An annualized return of roughly 12.2% against 22.1% risk suggests a Sharpe ratio near 0.55–0.60. While returns are solid, expect significant volatility year-over-year.  
- **Max Drawdown Context:** A ~38.18% drawdown likely corresponds to COVID‐19 sell‐off and subsequent market stress. Such a deep drawdown underlines why forward‐looking tail‐risk metrics (VaR, ES) are essential for risk management beyond historical drawdowns.  
- **Implications for Capital Planning:** Despite an appealing ~12% return, the 22% volatility and deep drawdown signal that any capital cushion (e.g., for a hedge fund or bank) cannot ignore potential for large, abrupt losses. Tail‐risk measures below quantify that need.

---

## 2. Constructing Sampling Weights

### 2.1. Age‐Based Weighting
```python
def get_age_weights(length, half_life):
    lam = 2 ** (-1 / half_life)
    idx = np.arange(1, length + 1)
    raw = (1 - lam) * np.power(lam, length - idx)
    return raw / raw.sum()
```
Recent observations carry exponentially more weight. The decay factor \(\lambda = 2^{-1/h}\), where \(h\) is the chosen half‐life (10, 21, or 63 days).  

**Commentary:**  
Age weighting allows the model to emphasize “recency.” A 10‐day half‐life captures only the past two weeks of market action, potentially missing older but still relevant tail events, while a 63‐day half‐life incorporates earlier crisis periods (e.g., March 2020). Practitioners must choose a half‐life informed by whether they believe short‐term or medium‐term shocks dominate.

### 2.2. Volatility‐Based Weighting
```python
def get_vol_weights(returns_series, window):
    rolling_vol = returns_series.rolling(window).std().fillna(method='bfill')
    raw = rolling_vol.values
    return raw / raw.sum()
```
Weights are proportional to rolling standard deviations over the window: \(w_t \propto \mathrm{RollingStdDev}(r_t)\).  

**Commentary:**  
Volatility weighting concentrates sampling on days with the highest realized volatility. A 10‐day window will overemphasize mega‐spikes like early 2020 and late 2022, producing extremely fat tail estimates. A longer window (63 days) smooths out overly concentrated clusters but still prioritizes major upheavals.

### 2.3. Correlation‐Based Weighting
```python
def get_corr_weights(returns_df, window):
    n = len(returns_df)
    corr_vals = np.zeros(n)
    for i in range(n):
        if i < window - 1:
            block = returns_df.iloc[0:window]
        else:
            block = returns_df.iloc[i - window + 1 : i + 1]
        corr_mat = block.corr().values
        off_diag = corr_mat[np.triu_indices_from(corr_mat, k=1)]
        corr_vals[i] = np.mean(np.abs(off_diag))
    return corr_vals / corr_vals.sum()
```
For each date, compute average off‐diagonal absolute correlations. High systemic correlation implies higher sampling weight: \(w_t \propto \frac{1}{n(n-1)} \sum_{i \neq j} |\rho_{ij}^{(t)}|\).  

**Commentary:**  
Correlation weighting captures periods when assets move in sync—often crisis or “risk‐off” regimes. A 10‐day correlation window may isolate a flash crisis, while 63 days captures broader contagion phases. This is particularly relevant for stress testing and macroprudential surveillance, where joint movements matter more than individual volatility spikes.

---

## 3. Bootstrap Risk Metric Estimation

### 3.1. Weighted Bootstrap Sampling and VaR/ES Calculations
```python
# 3.1.1 Weighted bootstrap sampling
def weighted_bootstrap(sample, weights, n_bootstrap):
    return np.random.choice(sample, size=n_bootstrap, replace=True, p=weights)

# 3.1.2 VaR and ES point estimates
def compute_var_es(bootstrap_returns, alpha):
    var = np.percentile(bootstrap_returns, 100 * alpha)  # negative loss value
    es = np.mean(bootstrap_returns[bootstrap_returns <= var]) if len(bootstrap_returns[bootstrap_returns <= var]) > 0 else np.nan
    return var, es

# 3.1.3 Generate ES bootstrap distribution for CI estimation
def generate_es_samples(sample, weights, alpha, n_bootstrap):
    es_samps = []
    for _ in range(n_bootstrap):
        bs_temp = weighted_bootstrap(sample, weights, len(sample))
        threshold_temp = np.percentile(bs_temp, 100 * alpha)
        es_temp = np.mean(bs_temp[bs_temp <= threshold_temp]) if len(bs_temp[bs_temp <= threshold_temp]) > 0 else np.nan
        if not np.isnan(es_temp):
            es_samps.append(es_temp)
    return np.array(es_samps)
```
For each method-window pair, draw \(n\_bootstrap=10000\) resamples. Each resample yields:  
- **VaR**(\(\alpha=1%\)): 1st percentile of the bootstrap sample (loss).  
- **ES**: average of all returns \(\le\) that 1% percentile (loss).  

**Remarks:**  
- This non‐parametric approach avoids heavy distributional assumptions.  
- Weighted resampling amplifies desired regimes—Age highlights recent days, Volatility accentuates turbulent clusters, Correlation underscores systemic episodes.  
- Point estimates capture central tail values (VaR, ES), but as the next section shows, confidence intervals vary greatly by weight and window.

---

## 4. Confidence Interval Estimation

### 4.1. Percentile Confidence Interval
```python
# 4.1.1 Percentile CI for 95% confidence

def percentile_ci(boot_dist, alpha=0.05):
    lower = np.percentile(boot_dist, 100 * (alpha/2))
    upper = np.percentile(boot_dist, 100 * (1 - alpha/2))
    return lower, upper
```  
Use \(\alpha\_ci=0.05\) for a 95% CI: lower at 2.5th percentile, upper at 97.5th percentile.  

**Commentary:**  
- Percentile CI directly reads off the bootstrap distribution.  
- If the bootstrap replications cluster (e.g., under a very small effective sample), the percentile CI can be deceptively narrow or even degenerate.  
- Regulators often require a 95% confidence band for VaR/ES models; percentile CI is simple but can understate tail uncertainty in heavily skewed samples.

### 4.2. BCa Confidence Interval
```python
# 4.2.1 BCa CI function definition

def bca_ci(sample, stat_func, boot_dist, alpha=0.05):
    # 1) Point estimate and bias correction (z0)
    theta_hat = stat_func(sample)
    prop_less = np.mean(boot_dist < theta_hat)
    z0 = norm.ppf(prop_less)

    # 2) Jackknife for acceleration parameter (a)
    n = len(sample)
    jack_vals = np.array([stat_func(np.delete(sample, i)) for i in range(n)])
    jack_mean = np.mean(jack_vals)
    a_num = np.sum((jack_mean - jack_vals) ** 3)
    a_den = 6 * (np.sum((jack_mean - jack_vals) ** 2) ** 1.5)
    a = a_num / a_den if a_den != 0 else 0

    # 3) zα/2 and z1−α/2
    z_alpha_lower = norm.ppf(alpha / 2)
    z_alpha_upper = norm.ppf(1 - alpha / 2)

    # 4) Compute adjusted z_lower, z_upper
    z_lower = z0 + (z0 + z_alpha_lower) / (1 - a * (z0 + z_alpha_lower))
    z_upper = z0 + (z0 + z_alpha_upper) / (1 - a * (z0 + z_alpha_upper))

    # 5) α1 = Φ(z_lower), α2 = Φ(z_upper)
    alpha1 = norm.cdf(z_lower)
    alpha2 = norm.cdf(z_upper)
    alpha1 = min(max(alpha1, 0), 1)
    alpha2 = min(max(alpha2, 0), 1)

    # 6) Handle NaN α's by fallback to standard percentile
    if np.isnan(alpha1): alpha1 = alpha / 2
    if np.isnan(alpha2): alpha2 = 1 - alpha / 2

    # 7) Percentile bounds for BCa
    lower = np.percentile(boot_dist, 100 * alpha1)
    upper = np.percentile(boot_dist, 100 * alpha2)
    if np.isnan(lower) or np.isnan(upper):
        print(f"[BCa CI Fail] jackknife std: {np.std(jack_vals):.6f}")
    return lower, upper
```
BCa adjusts for both bias (\(z_0\)) and acceleration (\(a\)) due to skew in the sampling distribution. When jackknife variability is low, it falls back gracefully to percentile.  

**Commentary:**  
- BCa attempts to account for skew and bias in bootstrap distributions.  
- A zero BCa CI (e.g., Age VaR) is not “perfect precision” but a sign BCa assumptions failed under extreme skew.  
- If BCa CI collapses, a fallback (percentile or bootstrap‐t) is recommended.

---

## 5. Summary of Bootstrap Results

Below, loop through each weighting method and lookback window to compute VaR₉₉, ES₉₉, and their 95% confidence intervals (Percentile and BCa):

```python
# 5.1. result calcuation & dataframe formatting
results = []
columns = [
    'Method','Window','VaR_99','ES_99',
    'VaR_LowerP','VaR_UpperP','VaR_LowerBCa','VaR_UpperBCa',
    'ES_LowerP','ES_UpperP','ES_LowerBCa','ES_UpperBCa'
]

for method in ['Age','Volatility','Correlation']:
    for w in windows:
        # 5.1.1 Generate weights based on method
        if method == 'Age':
            weights = get_age_weights(len(sample), half_life=w)
        elif method == 'Volatility':
            weights = get_vol_weights(portfolio_returns, window=w)
        else:
            weights = get_corr_weights(returns, window=w)

        # 5.1.2 Bootstrap sampling
        boot_samps = weighted_bootstrap(sample, weights, n_bootstrap)

        # 5.1.3 Compute VaR_99, ES_99 point estimates
        var_pt, es_pt = compute_var_es(boot_samps, alpha_var)

        # 5.1.4 Percentile CI for VaR
        var_low_p = np.percentile(boot_samps, 100 * (alpha_var / 2))
        var_up_p  = np.percentile(boot_samps, 100 * (1 - alpha_var / 2))

        # 5.1.5 Generate ES bootstrap distribution and its Percentile CI
        es_samps = generate_es_samples(sample, weights, alpha_var, n_bootstrap)
        es_low_p  = np.percentile(es_samps, 100 * (alpha_var / 2))
        es_up_p   = np.percentile(es_samps, 100 * (1 - alpha_var / 2))

        # 5.1.6 BCa CI for VaR and ES
        stat_var = lambda x: np.percentile(x, 100 * alpha_var)
        stat_es  = lambda x: np.mean(x[x <= np.percentile(x, 100 * alpha_var)]) if len(x[x <= np.percentile(x, 100 * alpha_var)]) > 0 else np.nan

        var_low_bca, var_up_bca = bca_ci(sample, stat_var, boot_samps, alpha_ci)
        es_low_bca, es_up_bca   = bca_ci(sample, stat_es, es_samps, alpha_ci)

        # 5.1.7 결과 저장
        results.append([
            method, w,
            var_pt, es_pt,
            var_low_p, var_up_p, var_low_bca, var_up_bca,
            es_low_p, es_up_p, es_low_bca, es_up_bca
        ])

results_df = pd.DataFrame(results, columns=columns)
print("\n=== Raw VaR/ES & CI Results ===")
display(results_df)
```
![summary](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/data_analysis_img/Weighted_Bootstrap/summary.png?raw=True)
  

**Expert Commentary on Tabular Results:**  

#### Age‐Weighted Method  
- **Window = 10 days**:  
  - _VaR₉₉_ ≈ 1.96%, _ES₉₉_ ≈ 4.40%.  
  - Percentile CI for VaR = [3.50%, 7.64%] (width ≈ 4.14%); BCa CI = [5.71%, 5.71%] (zero width—BCa degenerate due to skew).  
  - Percentile CI for ES = [2.41%, 5.09%] (width ≈ 2.68%); BCa CI = [2.61%, 5.09%] (width ≈ 2.48%).  
  - _Interpretation:_ Using only the past 10 days, Age weighting underestimates VaR but produces a wide percentile interval. A zero‐width BCa CI is a caution sign that BCa assumptions fail under extreme skew—do not trust that as “perfect precision.”  

- **Window = 21 days**:  
  - VaR₉₉ ≈ 3.50%, ES₉₉ ≈ 4.61%.  
  - VaR Percentile CI = [5.71%, 7.64%] (width ≈ 1.93%), BCa CI = [5.71%, 5.71%] (zero width).  
  - ES Percentile CI = [3.94%, 5.71%] (width ≈ 1.77%), BCa CI = [4.15%, 5.71%] (width ≈ 1.56%).  
  - _Insight:_ Mid‐term window picks up additional tail events (e.g., March 2020), pushing VaR from 1.96% to 3.50%. BCa for VaR still degenerates—use percentile or consider alternative methods.  

- **Window = 63 days**:  
  - VaR₉₉ ≈ 3.50%, ES₉₉ ≈ 4.57%.  
  - VaR Percentile CI = [5.71%, 7.64%] (width ≈ 1.93%), BCa CI = [5.71%, 5.71%] (zero).  
  - ES Percentile CI = [3.36%, 5.71%] (width ≈ 2.35%), BCa CI = [3.89%, 5.71%] (width ≈ 1.82%).  
  - _Takeaway:_ Including older 63‐day events yields similar point estimates to 21 days for VaR, but ES tightens slightly. Age weighting remains the most conservative for ES uncertainty but unreliable for VaR BCa.

#### Volatility‐Weighted Method  
- **Window = 10 days**:  
  - VaR₉₉ ≈ 8.91%, ES₉₉ ≈   10.70%.  
  - VaR Percentile CI = [12.84%, 12.82%] (width ≈ 0.02%—extremely narrow because high‐vol days dominate); BCa CI = [12.84%, 8.998%] (width ≈ 3.84%).  
  - ES Percentile CI = [12.09%, 9.598%] (width ≈ 2.49%), BCa CI = [12.09%, 9.570%] (width ≈ 2.52%).  
  - _Analysis:_ A 10‐day window picks up extreme clusters (e.g., early 2020, late 2022), making the percentile CI nearly collapse—misleadingly tight. BCa restores realistic uncertainty (~3.8%), so use BCa over percentile in such volatile regimes.  

- **Window = 21 days**:  
  - VaR₉₉ ≈ 8.72%, ES₉₉ ≈ 10.60%.  
  - VaR Percentile CI = [10.24%, 7.79%] (width ≈ 2.45%), BCa CI = [12.84%, 7.462%] (width ≈ 5.38%).  
  - ES Percentile CI = [5.93%, 5.93%] (width ≈ 0%—degenerate), BCa CI = [5.93%, 0%] (width ≈ 5.93%).  
  - _Interpretation:_ Percentile CI collapse for ES is another sign of too few distinct extreme values. BCa CI (~5.93%) remains large—a more trustworthy reflection of tail uncertainty.  

- **Window = 63 days**:  
  - VaR₉₉ ≈   5.71%, ES₉₉ ≈   9.42%.  
  - VaR Percentile CI = [10.24%, 7.79%] (width ≈ 2.45%), BCa CI = [12.84%, 6.9488%] (width ≈ 5.89%).  
  - ES Percentile CI = [6.030%, 6.0197%] (width ≈ 0.01%—almost a point), BCa CI = [6.030%, 0.0001%] (width ≈ 6.03%).  
  - _Insight:_ At longer windows, extreme days (2020, 2022) remain in the 63‐day sample, so ES holds near 9.4%. Percentile CI artificially collapses again—BCa reflects true heavy‐tail uncertainty (~6%).  

#### Correlation‐Weighted Method  
- **Window = 10 days**:  
  - VaR₉₉ ≈ 5.45%, ES₉₉ ≈ 8.36%.  
  - VaR Percentile CI = [8.72%, 7.63%] (width ≈ 1.09%), BCa CI = [12.84%, 5.66%] (width ≈ 7.18%).  
  - ES Percentile CI = [6.80%, 4.54%] (width ≈ 2.26%), BCa CI = [6.80%, 4.54%] (width ≈ 2.26%).  
  - _Analysis:_ High 10‐day asset correlations produce a sharply fat left tail; VaR and ES are near −8% to −9%. The disparity between narrow percentile CI and wide BCa CI (~7%) shows a highly skewed bootstrap distribution—BCa is preferred for tail‐uncertainty.

- **Window = 21 days**:  
  - VaR₉₉ ≈ 5.53%, ES₉₉ ≈ 8.76%.  
  - VaR Percentile CI = [8.91%, 7.63%] (width ≈ 1.28%), BCa CI = [12.84%, 5.89%] (width ≈ 6.95%).  
  - ES Percentile CI = [6.53%, 5.89%] (width ≈ 0.64%), BCa CI = [6.53%, 4.82%] (width ≈ 1.71%).  
  - _Interpretation:_ A 21‐day window softens some extreme cluster effects, but still retains large systemic tail risk. BCa CI stays near 6% for VaR, ~1.7% for ES, consistent under correlation regimes.

- **Window = 63 days**:  
  - VaR₉₉ ≈ 5.45%, ES₉₉ ≈ 8.32%.  
  - VaR Percentile CI = [8.91%, 7.63%] (width ≈ 1.28%), BCa CI = [12.84%, 5.99%] (width ≈ 6.85%).  
  - ES Percentile CI = [6.47%, 6.00%] (width ≈ 0.47%), BCa CI = [6.47%, 5.24%] (width ≈ 1.23%).  
  - _Takeaway:_ Over 63 days, correlation weighting balances short‐lived spikes with broader contagion. The VaR BCa CI (~6.85%) shows persistent tail uncertainty; ES BCa CI (~1.23%) is narrower, reflecting moderately stable average tail losses.  

**Summary:**  
- **Volatility weighting** produces the most extreme point VaR/ES but also the largest statistical uncertainty (BCa widths up to ~6%).  
- **Correlation weighting** yields mid‐range tail estimates with moderate but stable BCa CIs (~6% for VaR).  
- **Age weighting** gives the smallest tail estimates but frequently leads to degenerate BCa intervals for VaR, signaling a lack of variability in heavily skewed bootstrap samples.  

---

## 6. Formatted Table (Percentages and CI Widths)

```python
# 6.1. Increasing readability by making formatted version (formatted_df)
formatted_df = results_df.copy()

# 6.1.1 Point estimates & CI bounds → absolute percentage strings
percent_cols = [
    'VaR_99', 'ES_99',
    'VaR_LowerP', 'VaR_UpperP', 'VaR_LowerBCa', 'VaR_UpperBCa',
    'ES_LowerP', 'ES_UpperP', 'ES_LowerBCa', 'ES_UpperBCa'
]
for col in percent_cols:
    formatted_df[col] = formatted_df[col].apply(lambda x: f"{abs(x):.2%}")

# 6.1.2 Compute CI width columns in results_df, then format
results_df['VaR_Percentile_CI_Width'] = results_df['VaR_UpperP'] - results_df['VaR_LowerP']
results_df['VaR_BCa_CI_Width']        = results_df['VaR_UpperBCa'] - results_df['VaR_LowerBCa']
results_df['ES_Percentile_CI_Width']  = results_df['ES_UpperP'] - results_df['ES_LowerP']
results_df['ES_BCa_CI_Width']         = results_df['ES_UpperBCa'] - results_df['ES_LowerBCa']

ci_width_cols = [
    'VaR_Percentile_CI_Width', 'VaR_BCa_CI_Width',
    'ES_Percentile_CI_Width', 'ES_BCa_CI_Width'
]
for col in ci_width_cols:
    formatted_df[col] = results_df[col].apply(lambda x: f"{x:.4f}")

# 6.1.3 Reorder columns
display_cols = [
    'Method', 'Window',
    'VaR_99', 'ES_99',
    'VaR_LowerP', 'VaR_UpperP', 'VaR_LowerBCa', 'VaR_UpperBCa',
    'VaR_Percentile_CI_Width', 'VaR_BCa_CI_Width',
    'ES_LowerP', 'ES_UpperP', 'ES_LowerBCa', 'ES_UpperBCa',
    'ES_Percentile_CI_Width', 'ES_BCa_CI_Width'
]
formatted_df = formatted_df[display_cols]

print("\n=== Formatted VaR/ES & CI Results ===")
display(formatted_df)
```  
![summary_format](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/data_analysis_img/Weighted_Bootstrap/summary_format.png?raw=True)

> **Table Explanation:**  
> - Values are shown as **absolute percentages** for clarity (e.g., “5.45%” rather than “–0.0545”).  
> - **VaR_99**, **ES_99**: point estimates.  
> - **VaR_LowerP/UpperP**, **ES_LowerP/UpperP**: 95% Percentile CI bounds.  
> - **VaR_LowerBCa/UpperBCa**, **ES_LowerBCa/UpperBCa**: 95% BCa CI bounds.  
> - **VaR_Percentile_CI_Width**, **VaR_BCa_CI_Width**, **ES_Percentile_CI_Width**, **ES_BCa_CI_Width**: widths of each interval.

---

## 7. Diagnostic Visualizations

### 7.1. VaR & ES Distribution Histograms (3×3 Grid)
```python
lookup = {(row.Method, row.Window): (row.VaR_99, row.ES_99) for row in results_df.itertuples()}

methods = ['Age', 'Volatility', 'Correlation']
windows = [10, 21, 63]

fig, axes = plt.subplots(nrows=3, ncols=3, figsize=(15, 12), sharex=True, sharey=True)
for i, method in enumerate(methods):
    for j, w in enumerate(windows):
        ax = axes[i, j]
        # Compute weights and color per method
        if method == 'Age':
            weights = get_age_weights(len(sample), half_life=w)
            hist_color = 'C0'
        elif method == 'Volatility':
            weights = get_vol_weights(portfolio_returns, window=w)
            hist_color = 'C3'
        else:
            weights = get_corr_weights(returns, window=w)
            hist_color = 'C1'

        # Bootstrap sampling
        boot_returns = weighted_bootstrap(sample, weights, n_bootstrap)

        # Plot histogram (fixed binwidth)
        sns.histplot(
            boot_returns,
            binwidth=0.002,
            kde=False,
            color=hist_color,
            alpha=0.6,
            ax=ax
        )

        # Retrieve VaR₉₉ and ES₉₉ from results_df to match formatted_df
        var_pt, es_pt = lookup[(method, w)]

        # Plot colored vertical lines for VaR and ES, and include values in legend labels
        ax.axvline(var_pt, color='orange', linestyle='-', linewidth=2,
                   label=f"VaR₉₉: {var_pt:.2%}")
        ax.axvline(es_pt, color='green', linestyle='-', linewidth=2,
                   label=f"ES₉₉: {es_pt:.2%}")

        # Simplified titles and labels
        ax.set_title(f"{method[:3]} | {w}", fontsize=10)
        if i == 2:
            ax.set_xlabel("Return")
        if j == 0:
            ax.set_ylabel("Frequency")

        # Only show legend once per subplot with numeric values
        ax.legend(loc='upper right', fontsize=7)

# Super title and layout adjustments
plt.suptitle("Bootstrap Return Distributions with VaR₉₉ & ES₉₉", fontsize=16, y=0.95)
plt.tight_layout(rect=[0, 0, 1, 0.94])

# Fix common x/y limits for comparison
for ax in axes.flat:
    ax.set_xlim(-0.15, 0.15)
    ax.set_ylim(0, 1200)

plt.show()
```  
![distribution](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/data_analysis_img/Weighted_Bootstrap/distribution.png?raw=True)

**Histogram Commentary:**

#### Age‐Weighted (Blue Rows)
1. **Window = 10 days:**
   - **VaR ≈ –1.96%**, **ES ≈ –4.40%**. The left green line (ES) lies well left of the orange line (VaR), showing that the average of the worst 1% of losses (~4.4%) is more than twice as large as the 1st percentile VaR (~1.96%). This disparity highlights how recent two‐week data alone can underestimate extreme tail risk.
2. **Window = 21 days:**
   - **VaR ≈ –3.50%**, **ES ≈ –4.61%**. Expanding to three weeks captures additional drawdowns (notably early 2020), deepening VaR and slightly increasing ES. The green ES line moves further left, tightening the gap to VaR, indicating more consistent sampling of severe losses.
3. **Window = 63 days:**
   - **VaR ≈ –3.50%**, **ES ≈ –4.57%**. Over a three‐month window, VaR stays the same as 21 days, but ES marginally lowers. This suggests extreme events from the early pandemic remain within the 63‐day lookback, while a broader sample tempers ES slightly due to inclusion of some less extreme days.

#### Volatility‐Weighted (Red Rows)
1. **Window = 10 days:**
   - **VaR ≈ –8.91%**, **ES ≈ –10.70%**. High daily volatility periods (e.g., March 2020, late 2022) dominate the sample. The ES line at –10.7% shows that the severest 1% of outcomes are nearly 10.7% losses, far left of the VaR line at –8.91%.
2. **Window = 21 days:**
   - **VaR ≈ –8.72%**, **ES ≈ –10.60%**. A three‐week rolling volatility window sustains fat tails. Although VaR slightly improves (from –8.91% to –8.72%), ES remains around –10.6%, showing persistent extreme risk over a longer lookback.
3. **Window = 63 days:**
   - **VaR ≈ –5.71%**, **ES ≈ –9.42%**. A 63‐day volatility window softens VaR by pulling in calmer periods alongside spikes, but ES stays deep (–9.42%), reflecting that the worst 1% of returns across three months still average about 9.4% losses.

#### Correlation‐Weighted (Orange Rows)
1. **Window = 10 days:**
   - **VaR ≈ –5.45%**, **ES ≈ –8.36%**. Short‐term asset comovements spike during crisis windows; the averaged off‐diagonal correlation weights emphasize clusters of joint losses. VaR sits at –5.45%, while ES (–8.36%) is significantly deeper, indicating strongly correlated downside outcomes.
2. **Window = 21 days:**
   - **VaR ≈ –5.53%**, **ES ≈ –8.76%**. A three‐week window continues to highlight systemic risk. VaR deepens marginally to –5.53%, and ES steepens to –8.76%, reinforcing that correlated sell‐offs extend deeper than single‐asset volatility suggests.
3. **Window = 63 days:**
   - **VaR ≈ –5.45%**, **ES ≈ –8.32%**. Over a quarter, correlation weighting includes several contagion episodes (e.g., SVB panic, inflation shocks), but the VaR remains near –5.45%. ES slightly reduces to –8.32%, suggesting a slight “pull to normal” as diverse rolling periods are combined.

**Overall Insight:**
- **Volatility weighting** yields the most extreme VaR/ES losses across all windows, consistent with sampling heavy‐volatility clusters.  
- **Correlation weighting** also produces deep tails but is slightly less extreme than volatility weighting, reflecting that systemic comovements—while severe—do not always coincide with peak volatility spikes.  
- **Age weighting** is mildest for VaR (below 4% in all windows) but shows a steep ES‐to‐VaR ratio (>2x at 10 days), signaling that a handful of severe days drive most tail risk in a small lookback.  

These histograms illustrate the shape and extremity of each weighted bootstrap distribution. Analysts should note not only point estimates but also how far ES shifts left of VaR, emphasizing average worst‐case outcomes beyond the 1% threshold.

---

### 7.2. Confidence Interval Widths (Bar Charts)
```python
# 7.2.1 VaR CI width bar chart
eprint("# VaR CI Width Comparison by Method and Window")
var_ci_long = results_df[["Method", "Window", "VaR_Percentile_CI_Width", "VaR_BCa_CI_Width"]].melt(
    id_vars=["Method", "Window"], var_name="CI_Type", value_name="Width"
)
plt.figure(figsize=(10, 6))
sns.barplot(data=var_ci_long, x="Window", y="Width", hue="CI_Type")
plt.title("VaR 99% Confidence Interval Widths by Method")
plt.ylabel("CI Width (loss %)")
plt.xlabel("Lookback Window (Days)")
plt.grid(True)
plt.show()

# 7.2.2 ES CI width bar chart
eprint("# ES CI Width Comparison by Method and Window")
es_ci_long = results_df[["Method", "Window", "ES_Percentile_CI_Width", "ES_BCa_CI_Width"]].melt(
    id_vars=["Method", "Window"], var_name="CI_Type", value_name="Width"
)
plt.figure(figsize=(10, 6))
sns.barplot(data=es_ci_long, x="Window", y="Width", hue="CI_Type")
plt.title("ES 99% Confidence Interval Widths by Method")
plt.ylabel("CI Width (loss %)")
plt.xlabel("Lookback Window (Days)")
plt.grid(True)
plt.show()
```  

**Remakr & Insight:**  

#### VaR 99% CI Widths

![var_barchart](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/data_analysis_img/Weighted_Bootstrap/var_barchart.png?raw=True)

- **Age‐Weighted:** Percentile CI increases from ~11.14% (window 10) to ~13.34% (windows 21,63). BCa CI is zero for all windows—signaling BCa’s failure under skew; one should not interpret that as no uncertainty, but as a need to rely on percentile or an alternative approach.  
- **Volatility‐Weighted:** At **window 10**, percentile CI is extremely wide (~25.65%), reflecting volatile sampling. BCa CI is narrower (~10.44%), but still substantial. As window increases to 21 and 63, percentile CI shrinks to ~18.03% and ~16.70%, while BCa narrows to ~9.96% and ~9.34%.  
- **Correlation‐Weighted:** Percentile CI remains fairly stable (~16.35%–16.54%), while BCa CI also holds near ~8.6%–8.7%, decreasing slightly to ~7.97% at window 63.  

> **Takeaway:** Volatility weighting yields extreme VaR uncertainty if you rely on percentiles. BCa corrects somewhat but still leaves ~9–10% uncertainty. Correlation weighting offers middle ground: moderate point VaR (~5.45%–5.53%) with ~8% BCa uncertainty.

#### ES 99% CI Widths 

![es_barchart](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/data_analysis_img/Weighted_Bootstrap/es_barchart.png?raw=True)

- **Age‐Weighted:** Percentile CI shrinks from ~3.30% (window 10) to ~1.77% (window 21) then rises to ~2.34% (window 63). BCa CI is ~2.48%, ~1.56%, and ~1.82%, indicating moderate uncertainty.  
- **Volatility‐Weighted:** Percentile CI grows from ~3.84% (10 days) to ~4.63% (21 days) then declines slightly to ~4.58% (63 days). BCa CI is ~2.52% at 10 days, but collapses to ~0% at 21 days and ~0.01% at 63 days—another sign BCa fails under an extremely concentrated ES bootstrap.  
- **Correlation‐Weighted:** Percentile CI stays ~5.05% (10 days), ~4.97% (21 days), ~4.93% (63 days). BCa CI narrows from ~2.26% to ~1.71% to ~1.23%, reflecting a relatively stable but meaningful tail uncertainty for ES.  

> **Implications:** If your institution requires an 8% expected shortfall, a ~2%‐wide BCa band (for correlation weighting) suggests holding ~10% capital, whereas under volatility weighting you might erroneously see a near‐zero BCa and underbook losses.

---

### 7.3. CI Width Heatmap Analysis (2×2 Grid)
```python
# 7.3.1 Pivot tables for heatmaps
var_percentile_width = results_df.pivot(index="Method", columns="Window", values="VaR_Percentile_CI_Width")
var_bca_width       = results_df.pivot(index="Method", columns="Window", values="VaR_BCa_CI_Width")
es_percentile_width = results_df.pivot(index="Method", columns="Window", values="ES_Percentile_CI_Width")
es_bca_width        = results_df.pivot(index="Method", columns="Window", values="ES_BCa_CI_Width")

# 7.3.2 Plotting 2x2 heatmaps
fig, axes = plt.subplots(nrows=2, ncols=2, figsize=(12, 10))

sns.heatmap(var_percentile_width, annot=True, fmt=".4f", cmap="Blues", ax=axes[0,0])
axes[0,0].set_title("VaR 99% Percentile CI Width")
axes[0,0].set_ylabel("Method")
axes[0,0].set_xlabel("Window (Days)")

sns.heatmap(var_bca_width, annot=True, fmt=".4f", cmap="Greens", ax=axes[0,1])
axes[0,1].set_title("VaR 99% BCa CI Width")
axes[0,1].set_ylabel("Method")
axes[0,1].set_xlabel("Window (Days)")

sns.heatmap(es_percentile_width, annot=True, fmt=".4f", cmap="Oranges", ax=axes[1,0])
axes[1,0].set_title("ES 99% Percentile CI Width")
axes[1,0].set_ylabel("Method")
axes[1,0].set_xlabel("Window (Days)")

sns.heatmap(es_bca_width, annot=True, fmt=".4f", cmap="Purples", ax=axes[1,1])
axes[1,1].set_title("ES 99% BCa CI Width")
axes[1,1].set_ylabel("Method")
axes[1,1].set_xlabel("Window (Days)")

plt.suptitle("CI Width Comparison Heatmaps", fontsize=16, y=1.02)
plt.tight_layout()
plt.show()
```  
![heatmap](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/data_analysis_img/Weighted_Bootstrap/heatmap.png?raw=True)

#### Heatmap Commentary:  
- **VaR 99% Percentile CI (Blues):**  
  - Volatility weighting is darkest (~0.2565 at 10 days, then ~0.18 at 21 days, ~0.167 at 63 days), showing largest percentile uncertainty.  
  - Correlation is mid‐blue (~0.1635–0.1654), fairly stable across windows.  
  - Age is lightest (~0.1114 at 10 days, ~0.1334 at 21, ~0.1334 at 63), but note Age BCa is zero so Age percentile CI underrepresents true uncertainty if BCa fails.  

- **VaR 99% BCa CI (Greens):**  
  - Age is white (0.0000) at all windows—indicating BCa failure.  
  - Volatility (~0.1044 → 0.0996 → 0.0934) shows moderate but meaningful shrinkage.  
  - Correlation (~0.0860 → 0.0872 → 0.0797) is slightly narrower at 63 days.  

- **ES 99% Percentile CI (Oranges):**  
  - Correlation is darkest (~0.0505 → 0.0497 → 0.0493), signifying highest ES percentile uncertainty.  
  - Volatility (~0.0384 → 0.0463 → 0.0458) peaks at 21 days, reflecting overlapping volatile regimes.  
  - Age (~0.0330 → 0.0177 → 0.0234) is smallest overall but nonmonotonic—Age window adjustments shift which tail events are included.  

- **ES 99% BCa CI (Purples):**  
  - Age (~0.0248 → 0.0155 → 0.0182) moderately wide but stable.  
  - Volatility (~0.0252 → 0.0000 → 0.0001) collapses to near zero at 21 and 63 days—another BCa failure symptom.  
  - Correlation (~0.0226 → 0.0171 → 0.0123) narrows as window lengthens, indicating reduced systemic ES uncertainty over longer horizons.  

**Overall:**  
- Percentile CI is widest under **Volatility** (deep Blues/Oranges), moderate under **Correlation**, smallest under **Age**.  
- BCa CI is zero for Age‐VaR, nearly zero for Volatility‐ES at longer windows, and moderate for **Correlation** in all cases.  
- Decision‐makers should note that when BCa collapses, percentile CI—even if wide—is a more reliable gauge of tail risk.

---

## 8. Integrated Assessment

1. **Relative Tail Severity:**  
   - **Volatility weighting** yields the most extreme tail losses (VaR ~ 5.7–8.9%, ES ~ 9.4–10.7%) for short to mid‐term windows. Capital allocators should set aside high buffers if relying on a 10–21 day volatility window.  
   - **Correlation weighting** produces midrange tail losses (VaR ~ 5.45–5.53%, ES ~ 8.32–8.76%). These estimates capture joint asset comovements and are suited for systemic risk monitoring.  
   - **Age weighting** offers the mildest losses (VaR ~ 1.96–3.50%, ES ~ 4.40–4.61%), but these may grossly understate risk if major tail events lie outside a short lookback.  

2. **Uncertainty Around Tail Estimates:**  
   - **Volatility Percentile CI** can be misleadingly narrow when extreme days dominate (e.g., VaR ~12.82–12.84% wide ~0.02%). The corresponding **BCa CI (~10.44%)** is more realistic.  
   - **Correlation BCa CI** remains stable (~8% for VaR, ~2% for ES) across windows, highlighting moderate but meaningful uncertainty.  
   - **Age BCa CI** collapsing to zero is a statistical red flag—clearly, BCa failed under severe skew; always cross‐check with percentile CI.  

3. **Model‐Choice Recommendations:**  
   - For **tactical rebalancing** (short‐term), an **Age = 10-day** VaR of ~1.96% with wide percentile CI (~4%) could suffice, but managers should overlay stress tests.  
   - For **regulatory capital**, consider **Volatility = 21-day** or **Correlation = 63-day** weighted VaR/ES. A 21-day volatility VaR of ~8.72% with BCa CI ~5.38% is defensible under FRTB’s tail regulations, as is correlation VaR ~5.53% with BCa CI ~6.95%.  
   - For **systemic risk surveillance**, **Correlation = 63-day** ES of ~7.63% with BCa CI ~1.23% offers a stable, interpretable tail measure.  

4. **Statistical Warnings & Best Practices:**  
   - If BCa yields zero width, revert to percentile or alternative acceleration methods.  
   - Present multiple lookback windows side-by-side to illustrate sensitivity.  
   - Complement non‐parametric bootstrap with parametric EVT or GARCH‐t fits to triangulate tail risk.  
   - Always communicate loss measures as absolute percentages for clarity (e.g., “5.45% loss” rather than “–0.0545”).

**Conclusion:**  
In addition to its forward‐looking application, this weighted bootstrap framework can also be adapted for backtesting. By recording the daily 99% VaR and ES estimates generated at each lookback date and comparing them to the actual next‐day (or next‐period) returns, risk managers can assess how often realized losses exceed the predicted VaR, and whether the ES forecasts align with the worst observed tail losses. This two‐step approach (predictive estimation via weighted bootstrap, followed by realized‐loss comparison) ensures the model’s accuracy over time, allowing institutions to validate and recalibrate their tail‐risk measures as part of a robust backtesting regime. After covering parametric methods as well as EVT application, I will try to post about backtesting analysis upon real time data :)
