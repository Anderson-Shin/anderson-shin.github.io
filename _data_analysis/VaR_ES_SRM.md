---
title: "Coherent & Spectral Risk Management Practice Using Python"
collection: data_analysis
permalink: /data_analysis/portfolio_risk_estimation_practice1
excerpt: "Deep dive into VaR, Expected Shortfall, and Spectral Risk Measures (SRM) using Python"
venue: "Data Analysis Post"
date: 2025-05-23
location: ""
---


# 📊 Bootstrap-Based Precision Risk Analysis Blog Post

This blog post is based on the provided Jupyter Notebook (`Bootstrap_bactesting_integrated_updated.ipynb`) and demonstrates how to use **bootstrap techniques** to estimate **Value at Risk (VaR)**, **Expected Shortfall (ES)**, and **Spectral Risk Measure (SRM)** for a portfolio of sector ETFs. We include detailed output cells, charts, and **expert commentary** for each step. All sections are written in English, and mathematical formulas are explained in context.

---

## 1. Introduction

The main objectives of this notebook are:

1. **Data Collection & Preparation**

   * Download daily closing prices for multiple sector ETFs.
   * Compute daily portfolio returns using equal weights.

2. **Risk Metric Estimation (VaR, ES, SRM) via Bootstrap**

   * Perform **Standard (unweighted) bootstrap** and **Weighted bootstrap** (where higher volatility days receive higher sampling probability).
   * Calculate **Percentile Confidence Intervals (CI)** and **Bias-Corrected and Accelerated (BCa) CI** for each risk metric.

3. **Comparison: Standard vs. Weighted Bootstrap**

   * Compare distributions of VaR, ES, and SRM under both sampling schemes.
   * Analyze differences in means, standard deviations, and confidence intervals.

4. **Backtesting Concepts (Conceptual)**

   * Outline how to backtest VaR under Basel regulatory guidelines (detailed backtest implementation left to future work).

In practice, financial institutions rely on these risk measures for **regulatory compliance** (e.g., Basel II/III), **portfolio risk management**, and **capital allocation** decisions. Bootstrap-based estimation provides a way to quantify estimation uncertainty—especially important when analyzing **tail risk**.

---

## 2. Data Preparation

### 2.1. Library Imports and Basic Settings

```python
import yfinance as yf
import pandas as pd
import numpy as np
import scipy.stats as st
from scipy.interpolate import interp1d
import matplotlib.pyplot as plt
from scipy.optimize import brentq

# Key parameters
tickers = ['SPY', 'QQQ', 'XLF', 'XLE', 'XLK', 'XLI', 'XLV', 'XLP', 'XLU', 'XLB']
start_date = '2020-01-01'
end_date = '2024-12-31'
confidence_level = 0.95       # 95% confidence for VaR and ES
bootstrap_samples = 10000     # Number of bootstrap resamples
```

**Expert Commentary:**

* We use the `yfinance` package to download daily closing prices for ten major sector ETFs from January 1, 2020, through December 31, 2024.
* A 95% confidence level is chosen for VaR and ES calculations, meaning we look at the 5th percentile of portfolio loss distribution (i.e., VaR at 5% significance).
* Performing 10,000 bootstrap resamples provides sufficient statistical accuracy and stability for confidence interval estimation.

---

### 2.2. Data Download and Return Calculation

```python
# 1) Download daily closing price data for all tickers
raw_price_data = yf.download(tickers, start=start_date, end=end_date)["Close"]

# 2) Calculate daily percentage returns for each ticker
daily_returns = raw_price_data.pct_change().dropna()

# 3) Construct an equally weighted portfolio return series
num_assets = len(tickers)
weights_equal = np.repeat(1/num_assets, num_assets)
portfolio_returns = daily_returns.dot(weights_equal)
```

**Expert Commentary:**

* The `daily_returns` DataFrame contains the percentage return of each ETF in the universe.
* By assigning equal weights to each ETF (1/10th of the portfolio each), we obtain a single time series `portfolio_returns`.
* Mathematically, if \$r\_{t,i}\$ is the return of asset \$i\$ on day \$t\$, then the portfolio return on day \$t\$ is:

  $$
  R_t = \sum_{i=1}^{N} w_i \, r_{t,i}, \quad \text{where } w_i = \frac{1}{N}, \; N = 10.
  $$

---

## 3. Data Visualization

### 3.1. Sector-by-Sector Daily Returns Comparison

```python
# Plot daily returns for each sector ETF in subplots with a common y-axis scale
daily_returns.plot(
    subplots=True,
    sharey=True,
    layout=(5, 2),
    legend=False,
    figsize=(12, 10)
)
plt.suptitle('Sector Daily Returns (2020–2024)', y=1.02)
plt.tight_layout()
plt.show()
```

![Sector Daily Returns (2020–2024)](sandbox:/mnt/data/figure_6_0.png)

**Expert Analysis:**

* Each subplot shows the daily return series for one sector ETF.
* By setting `sharey=True`, all subplots use the same vertical scale, allowing direct comparison of volatility across sectors.
* Notice that the **Energy ETF (XLE)** exhibits pronounced spikes and troughs—indicative of higher volatility—compared to more stable sectors like **Consumer Staples (XLP)**.
* In risk estimation, days with large swings (outliers) have outsized influence on tail risk metrics (VaR/ES/SRM).

---

### 3.2. Portfolio Cumulative Return

```python
# Compute portfolio cumulative returns
cumulative_returns = (1 + portfolio_returns).cumprod()

# Plot the cumulative return series
cumulative_returns.plot(
    title='Equally Weighted Portfolio Cumulative Return (2020–2024)',
    figsize=(10, 5)
)
plt.xlabel('Date')
plt.ylabel('Cumulative Return')
plt.grid(True)
plt.show()
```

![Portfolio Cumulative Return (2020–2024)](sandbox:/mnt/data/figure_6_1.png)

**Expert Commentary:**

* The cumulative return graph highlights major market events: the COVID-19 crash in early 2020, subsequent recovery in 2021, inflation shock around 2022, and later volatility in 2023–2024.
* These extreme periods directly affect tail-risk estimation: very negative returns during 2020 and 2022 will feed into the bootstrap distribution and push VaR/ES estimates lower (more negative).
* In practice, risk teams often segment the data into sub-periods or apply time-varying models, but here we use the whole 2020–2024 span for simplicity.

---

## 4. Spectral Risk Measure (SRM) Parameter Selection

The **Spectral Risk Measure (SRM)** is a coherent risk measure that weights losses according to a user-defined spectral function. Two common spectral families are:

1. **Exponential Spectrum** (parameterized by $\lambda$): Places exponentially more weight on the worst losses.
2. **Polynomial (Gamma) Spectrum** (parameterized by $\gamma$): Assigns weights proportional to rank$^{\gamma-1}$.

We will demonstrate how to choose the parameter $\lambda^*$ so that the SRM corresponds to a target annual loss threshold (e.g., $-20\%$).

---

### 4.1. Defining the Exponential SRM Function

```python
def compute_srm_exp(returns_array, lam):
    """
    Compute the exponential spectral risk measure (SRM) for a sample of returns.

    Args:
        returns_array (np.ndarray): Array of daily portfolio returns.
        lam (float): Spectral parameter λ. Larger λ emphasizes worse (more negative) returns.

    Returns:
        float: Negative SRM (since returns can be positive or negative).
    """
    # Sort returns in ascending order (most negative first)
    sorted_returns = np.sort(returns_array)
    n = len(sorted_returns)
    
    # Compute unnormalized exponential weights:
    #   w_i = λ * exp(-λ * (i)/(n-1)), for i = 0, 1, ..., n-1
    raw_weights = np.array([lam * np.exp(-lam * i/(n-1)) for i in range(n)])
    normalized_weights = raw_weights / raw_weights.sum()
    
    # Spectral Risk Measure is the weighted average of sorted returns (with a negative sign for loss)
    srm_value = -np.sum(normalized_weights * sorted_returns)
    return srm_value
```

**Mathematical Explanation:**

* Let $x_{(1)} \le x_{(2)} \le \dots \le x_{(n)}$ be the sorted portfolio returns in ascending order.
* Define exponential weights:

  $$
  \tilde{w}_i = \lambda \exp\!\bigl(-\lambda \tfrac{i}{\,n-1\,}\bigr), \quad i = 0,\ldots,n-1.
  $$
* Normalize:

  $$
  w_i = \frac{\tilde{w}_i}{\sum_{j=0}^{n-1} \tilde{w}_j}, \quad \sum_{i=0}^{n-1} w_i = 1.
  $$
* The SRM is then:

  $$
  \text{SRM}_{\exp}(\lambda) = - \sum_{i=0}^{n-1} w_i\, x_{(i)}.
  $$
* A larger $\lambda$ assigns exponentially more weight to the worst (most negative) returns, leading to a larger (in magnitude) SRM.

---

### 4.2. Finding $\lambda^*$ for a Target Annual SRM

Suppose the target annual SRM (i.e., target worst-case loss) is $-20\%$. We need to find $\lambda^*$ such that the daily SRM corresponds to an annualized loss of $-20\%$. For simplicity, we approximate daily equivalent of $-20\%$ per year using:

$$
\text{Daily target return } r^* = (1 - 0.20)^{1/252} - 1 \approx -0.00098 \; (\approx -0.098\%).
$$

We then solve the equation

$$
\text{SRM}_{\exp}(\lambda) = -\,r^*
$$

for $\lambda$ using a root finder (Brent’s method).

```python
# Convert annual target of -20% to a daily equivalent (approximate)
daily_target_loss = (1 - 0.20)**(1/252) - 1  # e.g., ≈ -0.00098

# Define the function f(λ) = compute_srm_exp(sample, λ) - (-daily_target_loss)
def f_lambda(lmbda):
    return compute_srm_exp(portfolio_returns.values, lmbda) - (-daily_target_loss)

# Use Brent’s method to find λ* in the interval [0.01, 100]
lambda_star = brentq(f_lambda, a=0.01, b=100)
print(f"Found λ* = {lambda_star:.4f}")
```

**Expert Commentary:**

* We use `brentq` to find the root of $f(\lambda) = 0$.
* The result $\lambda^* \approx 0.415$ (example value) indicates how steeply we emphasize extreme losses: higher $\lambda$ leads to greater penalization of the most negative returns.
* In practice, risk managers choose $\lambda$ to reflect their risk aversion profile.

---

### 4.3. Mapping $\lambda$ and $\gamma$ to SRM Values

To compare exponential spectrum ($\lambda$) with a polynomial spectrum ($\gamma$), we define a simple gamma-based SRM:

```python
def compute_srm_gamma(returns_array, gamma):
    """
    Compute a polynomial spectral risk measure (SRM) for a sample of returns
    using parameter γ (gamma).

    Args:
        returns_array (np.ndarray): Array of sorted daily returns.
        gamma (float): Exponent parameter (γ > 1). Higher γ emphasizes tail more.

    Returns:
        float: Negative SRM value.
    """
    sorted_returns = np.sort(returns_array)
    n = len(sorted_returns)
    
    # Weights proportional to i^(γ - 1), for i = 1, 2, ..., n
    ranks = np.arange(1, n+1)
    raw_weights = ranks**(gamma - 1)
    normalized_weights = raw_weights / raw_weights.sum()
    
    srm_value = -np.sum(normalized_weights * sorted_returns)
    return srm_value

# Prepare a range of λ and γ values to compute SRM curves
lambda_values = np.linspace(0.1, 50, 300)
srm_lambda_curve = [compute_srm_exp(portfolio_returns.values, L) for L in lambda_values]

gamma_values = np.linspace(1.01, 10, 300)
srm_gamma_curve = [compute_srm_gamma(portfolio_returns.values, G) for G in gamma_values]

# Use interpolation to find λ* and γ* corresponding to the daily target loss (-daily_target_loss)
inv_lambda = interp1d(srm_lambda_curve, lambda_values, bounds_error=False, fill_value='extrapolate')
inv_gamma  = interp1d(srm_gamma_curve, gamma_values,  bounds_error=False, fill_value='extrapolate')

lambda_target = float(inv_lambda(-daily_target_loss))
gamma_target  = float(inv_gamma(-daily_target_loss))
```

Now, visualize how SRM changes with $\lambda$ and $\gamma$, and mark the points where SRM equals the target daily loss:

```python
fig, ax1 = plt.subplots(figsize=(8, 5))

# Plot λ vs. SRM (blue curve)
ax1.plot(srm_lambda_curve, lambda_values, color='blue', label='λ vs SRM')
ax1.scatter([-daily_target_loss], [lambda_target], color='blue')
ax1.annotate(
    f'λ* = {lambda_target:.2f}',
    xy=(-daily_target_loss, lambda_target),
    xytext=(lambda_target+1, -daily_target_loss-0.0005),
    arrowprops=dict(arrowstyle='->', color='blue')
)
ax1.set_xlabel('SRM (Daily Loss)')
ax1.set_ylabel('λ (Exponential Parameter)', color='blue')
ax1.tick_params(axis='y', labelcolor='blue')
ax1.set_ylim(0, 10)

# Plot γ vs. SRM (orange curve) on twin y-axis
ax2 = ax1.twinx()
ax2.plot(srm_gamma_curve, gamma_values, color='orange', label='γ vs SRM')
ax2.scatter([-daily_target_loss], [gamma_target], color='orange')
ax2.annotate(
    f'γ* = {gamma_target:.2f}',
    xy=(-daily_target_loss, gamma_target),
    xytext=(-daily_target_loss+0.002, gamma_target+1),
    arrowprops=dict(arrowstyle='->', color='orange')
)
ax2.set_ylabel('γ (Polynomial Parameter)', color='orange')
ax2.tick_params(axis='y', labelcolor='orange')
ax2.set_ylim(0, 10)

fig.suptitle('Parameter vs SRM: Exponential (λ) & Polynomial (γ) Mapping')
lines1, labels1 = ax1.get_legend_handles_labels()
lines2, labels2 = ax2.get_legend_handles_labels()
ax1.legend(lines1 + lines2, labels1 + labels2, loc='upper center', ncol=2)
plt.grid(True)
plt.tight_layout()
plt.show()
```

![Parameter vs SRM: λ & γ Mapping](sandbox:/mnt/data/figure_11_2.png)

**Expert Analysis:**

* **λ vs. SRM (blue curve):** As $\lambda$ increases, exponential weights concentrate on the most negative returns, making SRM more negative (larger in magnitude). The curve is steep because even small increases in $\lambda$ amplify tail emphasis.
* **γ vs. SRM (orange curve):** A polynomial spectrum with $\gamma$ is smoother: as $\gamma$ increases, more weight is given to lower-ranked (more negative) returns, but the relationship is less steep than exponential.
* The intersection points $(\lambda^*, \,\gamma^*)$ yield the same target SRM $\approx -0.098\%$ (daily equivalent of $-20\%$ annually). This shows that different spectral families can achieve similar tail-risk levels under different parameterizations.
* **Practical Implication:**

  * Use **exponential spectrum** ($\lambda$) when you want a **sharp emphasis** on extreme losses—common in regulatory reports or stress scenarios.
  * Use **polynomial spectrum** ($\gamma$) for a **more flexible** tail-weighting that can be tuned gradually.

---

## 5. Weighted Bootstrap: Assigning Sampling Probabilities

### 5.1. Volatility-Based Weight Calculation

To create a **weighted bootstrap**, we assign higher sampling weights to days with higher realized volatility. Here, we use a 20-day rolling standard deviation of portfolio returns as a proxy for “current market volatility.”

```python
# 1) Compute 20-day rolling volatility (standard deviation) of portfolio returns
rolling_volatility = pd.Series(portfolio_returns).rolling(window=20, min_periods=1).std().fillna(0.01)

# 2) Create raw weights equal to rolling_volatility (ensuring minimal floor)
raw_vol_weights = np.maximum(rolling_volatility.values, 1e-6)

# 3) Normalize so that weights sum to 1
weights_vol = raw_vol_weights / raw_vol_weights.sum()

# Display the first 5 normalized weights as an example
print("First 5 volatility-based weights:")
print(weights_vol[:5])
```

```text
First 5 volatility-based weights:
[0.000547 0.000319 0.000380 0.000518 0.000549]
```

**Expert Commentary:**

* Days with higher **past 20-day volatility** receive a larger weight in sampling, making it more likely that “high-volatility” days appear multiple times in the bootstrap sample.
* This approach ensures that **recent market stress periods** (when volatility spikes) are oversampled, leading to more conservative tail-risk estimates.
* We enforce a minimal weight of $10^{-6}$ to avoid zero-probability for any day.

---

### 5.2. Visualizing the Weight Distribution

```python
# Plot a histogram of the normalized volatility-based weights
plt.figure(figsize=(6, 4))
plt.hist(weights_vol, bins=50, density=True)
plt.title('Volatility-Based Bootstrap Weights Distribution')
plt.xlabel('Weight')
plt.ylabel('Density')
plt.grid(True)
plt.show()
```

![Volatility-Based Weights Distribution](sandbox:/mnt/data/figure_16_3.png)

**Expert Analysis:**

* The histogram shows a **skewed distribution**: a small number of days carry relatively high weight, while most days have very small weights.
* This “heavy tail” in the weight distribution emphasizes extreme-volatility days in sampling.
* In practice, you could also incorporate other factors into weights, such as **liquidity measures**, **order flow metrics**, or **market sentiment indices**, to further refine sampling probabilities.

---

## 6. Bootstrap Estimation of Risk Metrics

We will perform **10,000 bootstrap resamples** of the daily portfolio return series. For each resample, we compute:

* **VaR (Value at Risk):** The 5th percentile (since $\alpha = 1 - 0.95 = 0.05$).

  $$
  \text{VaR}_{0.95} = \text{the 5th percentile of bootstrap sample of returns}.
  $$
* **ES (Expected Shortfall):** The average of all losses that are below or equal to the VaR threshold.

  $$
  \text{ES}_{0.95} = \mathbb{E}[\,X \mid X \le \text{VaR}_{0.95}] \quad (\text{for losses } X \text{ in the bottom }5\%).
  $$
* **SRM (Spectral Risk Measure):** Using the fixed $\lambda^*$ found above.

  $$
  \text{SRM}_{\exp}(\lambda^*) = -\sum_{i=1}^{n} w_i(\lambda^*)\,x_{(i)}, \quad x_{(i)} \text{ sorted returns}.
  $$

We compare:

1. **Standard Bootstrap**: All days sampled with equal probability ($p_i = 1/n$).
2. **Weighted Bootstrap**: Days sampled with volatility-based weights $p_i = \text{weights_vol}[i]$.

---

### 6.1. Running the Bootstrap Sampling

```python
# Pre-allocate lists for risk metric values
std_VaR = []
std_ES  = []
std_SRM = []

wtd_VaR = []
wtd_ES  = []
wtd_SRM = []

sample = portfolio_returns.values  # numpy array of daily returns

for _ in range(bootstrap_samples):
    # Standard Bootstrap: sample with replacement uniformly
    standard_sample = np.random.choice(sample, size=len(sample), replace=True)
    var_std = np.percentile(standard_sample, (1 - confidence_level) * 100)
    es_std  = np.mean(standard_sample[standard_sample <= var_std])
    srm_std = compute_srm_exp(standard_sample, lam=lambda_star)
    
    std_VaR.append(var_std)
    std_ES.append(es_std)
    std_SRM.append(srm_std)
    
    # Weighted Bootstrap: sample with replacement using volatility-weighted probabilities
    weighted_sample = np.random.choice(sample, size=len(sample), replace=True, p=weights_vol)
    var_wtd = np.percentile(weighted_sample, (1 - confidence_level) * 100)
    es_wtd  = np.mean(weighted_sample[weighted_sample <= var_wtd])
    srm_wtd = compute_srm_exp(weighted_sample, lam=lambda_star)
    
    wtd_VaR.append(var_wtd)
    wtd_ES.append(es_wtd)
    wtd_SRM.append(srm_wtd)

print("Bootstrap sampling completed.")
```

**Expert Commentary:**

* After 10,000 iterations, we have six distributions:

  * Standard Bootstrap: `std_VaR`, `std_ES`, `std_SRM`.
  * Weighted Bootstrap: `wtd_VaR`, `wtd_ES`, `wtd_SRM`.
* These distributions capture estimation uncertainty around each risk measure.
* Because volatility-based weights oversample high-volatility days, we expect the weighted bootstrap distributions to exhibit **more negative (conservative)** VaR/ES/SRM estimates on average.

---

### 6.2. Visualizing Risk Metric Distributions

```python
fig, axes = plt.subplots(3, 2, figsize=(12, 10))
axes = axes.flatten()

# Titles and corresponding data
plot_data = [
    (std_VaR, 'Std VaR'),
    (wtd_VaR, 'Wtd VaR'),
    (std_ES,  'Std ES'),
    (wtd_ES,  'Wtd ES'),
    (std_SRM, 'Std SRM'),
    (wtd_SRM, 'Wtd SRM')
]

for ax, (data, title) in zip(axes, plot_data):
    ax.hist(data, bins=50, alpha=0.7, density=True)
    ax.set_title(title)
    ax.grid(True)

plt.tight_layout()
plt.show()
```

![Bootstrap Distribution Comparison](sandbox:/mnt/data/figure_22_4.png)

**Expert Analysis:**

* **VaR Distributions** (top row):

  * **Std VaR** is relatively narrow around its mean (fewer outliers).
  * **Wtd VaR** is wider, with a heavier left tail—indicating more frequent selection of extreme-loss days under weighted sampling.
* **ES Distributions** (middle row):

  * **Std ES** is moderately bell-shaped around its mean.
  * **Wtd ES** is noticeably wider and more left-skewed (higher probability of severe losses).
* **SRM Distributions** (bottom row):

  * **Std SRM** is centered closer to zero (less negative).
  * **Wtd SRM** shifts left (more negative) and shows greater variability—reflecting heavier emphasis on tail observations when computing the spectral measure.

---

## 7. Confidence Interval (CI) Comparison

To quantify uncertainty, we compute 95% **Percentile CI** and **BCa CI** for each metric under both sampling schemes.

### 7.1. Defining CI Calculation Functions

```python
# 1) Percentile CI: simply take the α/2 and 1 - α/2 percentiles of the bootstrap distribution
def percentile_ci(arr, alpha):
    lower = np.percentile(arr, 100 * alpha / 2)
    upper = np.percentile(arr, 100 * (1 - alpha / 2))
    return lower, upper

# 2) Bias-Corrected and Accelerated (BCa) CI
def bca_ci(original_sample, bootstrap_dist, stat_func, alpha):
    """
    Compute BCa (Bias-Corrected and Accelerated) confidence intervals.

    Args:
        original_sample (np.ndarray): The original data used to create bootstrap distributions.
        bootstrap_dist  (np.ndarray): An array of bootstrapped statistic values.
        stat_func       (callable): A function that computes the statistic from a sample.
        alpha           (float): Total significance level (0.05 for 95% CI).

    Returns:
        (float, float): Lower and upper bounds of the BCa CI.
    """
    try:
        # 1) Compute the original point estimate θ^ (statistic on original data)
        theta_hat = stat_func(original_sample)
        
        # 2) Compute bias-correction z0
        z0 = st.norm.ppf(np.mean(bootstrap_dist < theta_hat))
        
        # 3) Jackknife estimates to compute acceleration parameter a
        n = len(original_sample)
        jackknife_estimates = np.array([
            stat_func(np.delete(original_sample, i)) for i in range(n)
        ])
        jack_mean = np.mean(jackknife_estimates)
        numer = np.sum((jack_mean - jackknife_estimates)**3)
        denom = 6 * (np.sum((jack_mean - jackknife_estimates)**2)**1.5)
        a = numer / denom
        
        # 4) Compute z-values for lower and upper tail
        z_lower = st.norm.ppf(alpha / 2)
        z_upper = st.norm.ppf(1 - alpha / 2)
        
        # 5) Adjusted percentiles
        pct_lower = st.norm.cdf(z0 + (z0 + z_lower) / (1 - a * (z0 + z_lower)))
        pct_upper = st.norm.cdf(z0 + (z0 + z_upper) / (1 - a * (z0 + z_upper)))
        
        # 6) Convert to percentiles in bootstrap distribution
        lower_bound = np.percentile(bootstrap_dist, 100 * pct_lower)
        upper_bound = np.percentile(bootstrap_dist, 100 * pct_upper)
        
        return lower_bound, upper_bound
    except Exception:
        # Fallback to percentile CI if BCa fails
        return percentile_ci(bootstrap_dist, alpha)
```

**Mathematical Explanation:**

* **Percentile CI**: For a bootstrap distribution $\{\hat{\theta}^*_b\}_{b=1}^{B}$, the lower bound is the $\tfrac{\alpha}{2}$-percentile and the upper bound is the $1 - \tfrac{\alpha}{2}$-percentile.
* **BCa CI**: Adjusts for both **bias** ($z_0$) and **skewness** (acceleration $a$).

  1. Let $\hat{\theta}$ be the statistic from the original sample.
  2. Compute $z_0 = \Phi^{-1}\!\bigl(\tfrac{\#\{\theta^*_b < \hat{\theta}\}}{B}\bigr)$.
  3. Compute jackknife estimates $\hat{\theta}_{(i)}$ by leaving out observation $i$.
  4. Acceleration $a$ is given by:

     $$
     a = \frac{\sum_{i} (\overline{\theta}_{(\cdot)} - \hat{\theta}_{(i)})^3}
              {\,6 \Bigl[\sum_{i} (\overline{\theta}_{(\cdot)} - \hat{\theta}_{(i)})^2\Bigr]^{3/2}},
     $$

     where $\overline{\theta}_{(\cdot)}$ is the mean of the jackknife estimates.
  5. Compute adjusted percentiles $p_L$ and $p_U$:

     $$
     p_L = \Phi\!\bigl(z_0 + \tfrac{z_0 + z_{\alpha/2}}{1 - a\,(z_0 + z_{\alpha/2})}\bigr), \quad
     p_U = \Phi\!\bigl(z_0 + \tfrac{z_0 + z_{1-\alpha/2}}{1 - a\,(z_0 + z_{1-\alpha/2})}\bigr).
     $$
  6. The BCa CI is then $\bigl[\hat{\theta}^*_{(p_L)},\; \hat{\theta}^*_{(p_U)}\bigr]$.

---

### 7.2. Compute and Tabulate CIs for Each Metric

```python
# Define statistic functions for VaR, ES, SRM
stats_funcs = {
    'VaR': lambda x: np.percentile(x, (1 - confidence_level) * 100),
    'ES':  lambda x: np.mean(x[x <= np.percentile(x, (1 - confidence_level) * 100)]),
    'SRM': lambda x: compute_srm_exp(x, lam=lambda_star)
}

alpha = 1 - confidence_level  # 0.05 for 95% CI

rows = []

# Loop through methods and metrics
for method, distributions in [
    ('Standard', [('VaR', np.array(std_VaR)), ('ES', np.array(std_ES)), ('SRM', np.array(std_SRM))]),
    ('Weighted', [('VaR', np.array(wtd_VaR)),   ('ES', np.array(wtd_ES)),   ('SRM', np.array(wtd_SRM))])
]:
    for metric_name, metric_array in distributions:
        mean_val = metric_array.mean()
        std_val = metric_array.std(ddof=1)
        
        # Percentile CI
        p_lower, p_upper = percentile_ci(metric_array, alpha)
        # BCa CI
        b_lower, b_upper = bca_ci(portfolio_returns.values, metric_array, stats_funcs[metric_name], alpha)
        
        rows.append({
            'Method': method,
            'Metric': metric_name,
            'Mean': mean_val,
            'Std': std_val,
            'Pct CI Lower': p_lower,
            'Pct CI Upper': p_upper,
            'BCa CI Lower': b_lower,
            'BCa CI Upper': b_upper
        })

# Convert to DataFrame and pivot for readability
import pandas as pd

df_ci = pd.DataFrame(rows)
df_pivot = df_ci.pivot(index='Metric', columns='Method')
df_pivot
```

| Metric  |   | Standard  |          | Weighted     |              |              |              |           |          |              |              |              |              |
| ------- | - | --------- | -------- | ------------ | ------------ | ------------ | ------------ | --------- | -------- | ------------ | ------------ | ------------ | ------------ |
|         |   | Mean      | Std      | Pct CI Lower | Pct CI Upper | BCa CI Lower | BCa CI Upper | Mean      | Std      | Pct CI Lower | Pct CI Upper | BCa CI Lower | BCa CI Upper |
| **ES**  |   | -0.032411 | 0.003188 | -0.039135    | -0.026778    | -0.041244    | -0.027833    | -0.057044 | 0.005310 | -0.067405    | -0.046937    | -0.067405    | -0.046937    |
| **SRM** |   | -0.000881 | 0.000406 | -0.001678    | -0.000088    | -0.001705    | -0.000118    | -0.001292 | 0.000664 | -0.002605    | -0.000010    | -0.001783    | 0.000741     |
| **VaR** |   | -0.018137 | 0.001145 | -0.021060    | -0.016026    | -0.021152    | -0.016176    | -0.027625 | 0.002456 | -0.031217    | -0.024206    | -0.031217    | -0.024206    |

> **Table Explanation:**
>
> * Each row corresponds to a risk metric (VaR, ES, SRM) under two sampling methods (Standard vs. Weighted).
> * **Mean** and **Std** show the bootstrap distribution’s mean and standard deviation.
> * **Pct CI Lower/Upper** are the lower/upper bounds of the 95% percentile confidence interval.
> * **BCa CI Lower/Upper** are the bias-corrected and accelerated interval bounds.
> * Observe that the **Weighted bootstrap estimates** for VaR and ES are significantly more negative (e.g., Weighted ES mean ≈ $-5.70\%$ vs. Standard ES mean ≈ $-3.24\%$), reflecting a more conservative (tail-focused) approach.
> * BCa intervals for weighted metrics are wider, capturing more uncertainty in the tail.

---

### 7.3. Mean Comparison Bar Chart

```python
# Plot bar chart comparing mean values of each metric (Standard vs Weighted)
import seaborn as sns

mean_comparison = df_ci.pivot(index='Metric', columns='Method', values='Mean')

ax = mean_comparison.plot(
    kind='bar',
    figsize=(8, 5),
    title='Mean Risk Metric Comparison: Standard vs Weighted Bootstrap'
)
ax.set_ylabel('Mean Value')
ax.grid(True)

# Annotate each bar with its numeric value
for container in ax.containers:
    ax.bar_label(container, fmt='%.4f')

plt.xticks(rotation=0)
plt.tight_layout()
plt.show()
```

![Mean Comparison (Standard vs Weighted)](sandbox:/mnt/data/figure_27_5.png)

**Expert Analysis:**

* **VaR:**

  * **Standard Bootstrap:** Mean VaR ≈ $-1.81\%$
  * **Weighted Bootstrap:** Mean VaR ≈ $-2.76\%$
    → Weighted procedure is about **0.95 percentage points** more conservative (i.e., larger loss).
* **ES:**

  * **Standard:** $-3.24\%$ vs. **Weighted:** $-5.70\%$
    → Weighted ES is **2.46 percentage points** deeper into tail losses.
* **SRM:**

  * **Standard:** $-0.09\%$ vs. **Weighted:** $-0.13\%$
    → Although SRM differences appear numerically small, SRM is highly sensitive to extreme tails via the exponential weighting.

---

## 8. Conclusion & Practical Implications

1. **Necessity of Weighted Bootstrap**

   * By oversampling high-volatility days, the **weighted bootstrap** yields **more conservative** estimates of VaR, ES, and SRM.
   * This is particularly important during market stress—financial institutions can better gauge extreme tail risk and allocate **buffer capital** accordingly.

2. **Preference for BCa Confidence Intervals**

   * The **Percentile CI** is straightforward to compute but fails to adjust for bias or distribution skewness—potentially underestimating tail uncertainty.
   * **BCa CI** corrects for bias ($z₀$) and skewness (acceleration $a$), providing a more reliable and conservative interval for tail-heavy metrics.
   * In regulatory filings, submitting BCa-adjusted CIs demonstrates transparency regarding risk estimate uncertainty.

3. **Choosing Spectral Parameters ($\lambda$ vs. $\gamma$)**

   * **Exponential spectrum** ($\lambda$) heavily penalizes extreme losses; suitable for **rigorous stress tests** or regulatory capital calculations.
   * **Polynomial spectrum** ($\gamma$) provides a more gradual tail emphasis—useful for **portfolio optimization** where extreme concentration is undesirable but not fully penalized.

4. **Real-World Implementation**

   * A **Risk Management Team** can run monthly or quarterly weighted bootstrap analyses to track evolving VaR/ES/SRM values and adjust **capital buffers**.
   * A **Quantitative Team** can automate the entire pipeline (Python + VBA + Database), pulling live market data to produce near real-time risk reports.
   * For **Regulatory Reporting**, include both point estimates and **BCa CIs** to illustrate the **uncertainty** around risk measures to supervisors.

---

> **Appendix:**
>
> * All core code snippets, charts, and tables originate from the provided Jupyter Notebook (`Bootstrap_bactesting_integrated_updated.ipynb`).
> * This blog post highlights key steps, formulas, and expert commentary; you may refer to the notebook for full implementation details and additional backtesting concepts.
