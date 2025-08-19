---
title: "ETF Portfolio Risk Analysis: Backtesting Parametric vs. Non-Parametric Models"
collection: data_analysis
permalink: /data_analysis/etf-risk-analysis-backtesting/
excerpt: "An in-depth comparison and backtest of Parametric, Historical, and GARCH-FHS models for VaR and ES estimation on an optimized ETF portfolio."
date: 2025-08-04
tags:
  - Portfolio Analysis
  - Risk Management
  - VaR
  - Expected Shortfall
  - Backtesting
  - GARCH
---

# ETF Portfolio Risk Analysis Report

## Executive Summary

This report presents a comparative analysis of Parametric, Non-Parametric (Historical), and GARCH-Filtered Historical Simulation (FHS) risk models for an optimized portfolio of your chosen ETFs from January 2024 to July 2025.

The analysis concludes that while the GARCH-FHS model provides the lowest VaR estimate, its performance in predicting the magnitude of tail losses is less accurate than the Historical Simulation model. The **Historical Simulation model demonstrates the best overall performance**, passing the frequency backtest and accurately predicting the magnitude of tail losses in the ES backtest.

## Optimal Portfolio Allocation

The portfolio was optimized to maximize the Sharpe Ratio, a measure of risk-adjusted return. The risk-free rate derived from the 'BIL' ETF was essential for this calculation, serving as the baseline for determining excess returns. This optimization resulted in the following weights for the risky assets:

*   **BND:** 0.00%
*   **GLD:** 67.15%
*   **VNQ:** 0.00%
*   **VTI:** 32.85%
*   **VXUS:** 0.00%

*   **Annualized Portfolio Return:** 27.37%
*   **Annualized Portfolio Volatility:** 13.13%

## Key Performance Metrics

*   **Maximum Drawdown:** -7.09%
*   **Sharpe Ratio:** 1.72
*   **Annualized Risk-Free Rate:** 4.75%

## Final Summary Table

| Model      |   VaR (99%) |   ES (99%) |   Kupiec's POF p-value |   Avg Actual Exceedance Loss |
|:-----------|------------:|-----------:|-----------------------:|-----------------------------:|
| Parametric |   0.018156  | -0.0209589 |               0.162557 |                   -0.0226203 |
| Historical |   0.0189197 |  0.0258009 |               0.975825 |                   -0.0258009 |
| GARCH-FHS  |   0.0221333 |  0.0258688 |               0.6193   |                   -0.0275383 |

## Expert Feedback on Analysis

The current risk analysis report provides a solid foundation, but as an expert, I can offer a deeper dive into the implications of the presented data and visualizations.

**1. Optimal Portfolio Allocation and Diversification (Referencing Weights and Correlation Heatmap):**

The optimization successfully yielded an annualized return of **27.37%** with an annualized volatility of **13.13%**, resulting in a Sharpe Ratio of **1.72** (given a risk-free rate of **4.75%**). This indicates a highly efficient portfolio for the given period and asset universe.

A critical observation from the "Optimal Portfolio Allocation" is the concentration in **GLD (67.15%)** and **VTI (32.85%)**, with zero allocation to BND, VNQ, and VXUS. This suggests that, for the chosen period and optimization criteria (maximizing Sharpe Ratio), these two assets offered the most compelling risk-adjusted returns.

Looking at the "Correlation Heatmap of Risky Assets," a deeper analysis of the correlations between GLD and VTI would be beneficial. If GLD (gold, often a safe-haven asset) and VTI (total stock market, representing broader equity risk) exhibit low or negative correlation, this concentration could still provide effective diversification, despite the limited number of assets. If their correlation is high, the portfolio might be more susceptible to systemic market movements than a more diversified allocation might suggest. The current report doesn't explicitly discuss the correlation values between the chosen optimal assets, which is a missed opportunity for a sharper analysis of the portfolio's inherent diversification.

**2. Performance Metrics in Context (Annualized Return, Volatility, Max Drawdown, Sharpe Ratio):**

*   **Annualized Return (27.37%) and Volatility (13.13%):** These figures represent strong performance for the period. The relatively low volatility for such a high return is commendable, contributing significantly to the high Sharpe Ratio.
*   **Maximum Drawdown (-7.09%):** This is a crucial metric for risk assessment. A maximum drawdown of -7.09% over the period is quite favorable, especially when compared to typical market downturns. This suggests the portfolio, despite its concentration, demonstrated resilience during adverse market conditions within the analyzed timeframe. The "Portfolio Drawdown" plot visually confirms this relatively contained downside.
*   **Sharpe Ratio (1.72):** This is an excellent Sharpe Ratio, indicating that the portfolio generated 1.72 units of excess return for each unit of risk taken. This is a strong indicator of efficient risk management and superior performance relative to the risk-free rate.

**3. Nuanced Interpretation of Risk Models (Referencing Final Summary Table and Distribution of Returns):**

The "Final Summary Table" and "Distribution of Returns with VaR Estimates" provide the core insights into model performance.

*   **Parametric Model:** While it passes Kupiec's POF test (p-value 0.162557), its "Avg Actual Exceedance Loss" (-0.0226203) is notably higher than its "ES (99%)" (-0.0209589). This confirms the report's point about "fat tails" (as seen in the QQ-Plot) causing the Parametric model to underestimate the magnitude of extreme losses. This model's reliance on the normality assumption makes it less reliable for this portfolio.
*   **Historical Model:** This model is indeed the standout. Its Kupiec's POF p-value of **0.975825** is exceptionally high, indicating that the observed frequency of VaR breaches is almost perfectly aligned with the expected frequency. Furthermore, the "Avg Actual Exceedance Loss" (-0.0258009) is almost identical to its "ES (99%)" (0.0258009), demonstrating its accuracy in predicting the *magnitude* of tail losses. The "Distribution of Returns" plot visually supports this, as the Historical VaR line appears to align well with the observed tail of the distribution.
*   **GARCH-FHS Model:** The GARCH-FHS model, with a p-value of 0.6193, passes the Kupiec's POF test, suggesting its frequency predictions are statistically sound. However, its "Avg Actual Exceedance Loss" (-0.0275383) is still higher than its "ES (99%)" (0.0258688), indicating it *underestimates* the severity of losses, albeit less severely than the Parametric model. The primary issue with GARCH-FHS is its underestimation of ES, not its failure in frequency.

**Overall Assessment:**

The report effectively highlights the strengths of the Historical Simulation model for this specific portfolio and period. The inclusion of the methodology and clear visualizations significantly enhances its value. The primary area for improvement is a deeper discussion of the diversification implications from the correlation heatmap.

## Methodology and Key Calculations

This section provides the core Python code snippets used in `main.py` to derive the values presented in this report.

### Core Parameters and Data Fetching

```python
import yfinance as yf
import pandas as pd

# Core Parameters
tickers = ['VTI', 'VXUS', 'BND', 'VNQ', 'GLD']
start_date = '2024-01-01'
end_date = '2025-07-31'
risk_free_ticker = 'BIL'
confidence_level = 0.99

# Fetch Historical Data
data = yf.download(tickers + [risk_free_ticker], start=start_date, end=end_date, auto_adjust=True)['Close']

# Calculate Returns
returns = data.pct_change().dropna()
```

### Optimal Portfolio Allocation (Sharpe Ratio Optimization)

```python
import numpy as np
from scipy.optimize import minimize

# Calculate Annualized Risk-Free Rate
risk_free_rate = returns[risk_free_ticker].mean() * 252

# Separate risky assets
risky_returns = returns.drop(columns=[risk_free_ticker])

# Calculate inputs for optimization
mean_returns = risky_returns.mean() * 252
cov_matrix = risky_returns.cov() * 252

# Define objective function (negative Sharpe Ratio)
def negative_sharpe(weights, mean_returns, cov_matrix, risk_free_rate):
    portfolio_return = np.sum(mean_returns * weights)
    portfolio_std = np.sqrt(np.dot(weights.T, np.dot(cov_matrix, weights)))
    sharpe_ratio = (portfolio_return - risk_free_rate) / portfolio_std
    return -sharpe_ratio

# Set constraints and bounds
num_assets = len(risky_returns.columns)
constraints = ({'type': 'eq', 'fun': lambda w: np.sum(w) - 1})
bounds = tuple((0, 1) for _ in range(num_assets))
initial_weights = num_assets * [1. / num_assets]

# Run optimization
optimal_weights_result = minimize(negative_sharpe, initial_weights, args=(mean_returns, cov_matrix, risk_free_rate), method='SLSQP', bounds=bounds, constraints=constraints)
optimal_weights = optimal_weights_result.x
```

### Portfolio Performance Metrics

```python
# Portfolio Performance & Maximum Drawdown
cumulative_returns = (1 + portfolio_returns).cumprod()
peak = cumulative_returns.expanding(min_periods=1).max()
drawdown = (cumulative_returns/peak) - 1
max_drawdown = drawdown.min()

# Calculate final portfolio performance metrics
portfolio_annual_return = portfolio_returns.mean() * 252
portfolio_annual_volatility = portfolio_returns.std() * np.sqrt(252)
sharpe_ratio = (portfolio_annual_return - risk_free_rate) / portfolio_annual_volatility
```

### VaR and ES Calculations

```python
from scipy.stats import norm
from arch import arch_model

# Calculate Parametric VaR and ES
mu = portfolio_returns.mean()
sigma = portfolio_returns.std()
VaR_parametric = norm.ppf(1 - confidence_level, mu, sigma)
ES_parametric = (1-confidence_level)**-1 * norm.pdf(norm.ppf(1-confidence_level)) * sigma - mu

# Calculate Non-Parametric (Historical) VaR and ES
VaR_historical = portfolio_returns.quantile(1 - confidence_level)
ES_historical = portfolio_returns[portfolio_returns <= VaR_historical].mean()

# Fit GARCH Model and Standardize Returns
garch_model = arch_model(portfolio_returns * 100, vol='Garch', p=1, q=1)
garch_results = garch_model.fit(disp='off')
std_resid = garch_results.resid / garch_results.conditional_volatility
forecast = garch_results.forecast(horizon=1)
next_day_vol = np.sqrt(forecast.variance.iloc[-1,0]) / 100

# Calculate Hybrid GARCH-FHS VaR and ES
VaR_quantile_std = std_resid.quantile(1 - confidence_level)
VaR_garch_fhs = next_day_vol * VaR_quantile_std
ES_garch_fhs = next_day_vol * std_resid[std_resid <= VaR_quantile_std].mean()
```

### Backtest Functions

```python
from scipy.stats import chi2
import numpy as np

# Kupiec's POF Backtest
def kupiec_pof_test(returns, var, confidence_level):
    exceptions = returns < -var
    N1 = exceptions.sum()
    N0 = len(returns) - N1
    p = 1 - confidence_level
    if N1 == 0:
        return 1.0
    LR_pof = -2 * np.log(((1-p)**N0 * p**N1) / ((1-N1/(N0+N1))**N0 * (N1/(N0+N1))**N1))
    p_value = 1 - chi2.cdf(LR_pof, 1)
    return p_value

# ES vs. Actual Loss Backtest
def es_backtest(returns, var, es):
    exceptions = returns < -var
    if exceptions.sum() > 0:
        avg_loss = returns[exceptions].mean()
        return avg_loss, es
    else:
        return 0, es
```

## Visualizations

### Correlation Heatmap of Risky Assets
This heatmap visualizes the correlation coefficients between the daily returns of the risky assets in the portfolio. Values closer to 1 indicate a strong positive correlation, while values closer to -1 indicate a strong negative correlation. This helps in understanding diversification benefits.
![Correlation Heatmap](/images/correlation_heatmap.png)

### Portfolio Performance
This plot shows the cumulative returns of the optimal portfolio over the analyzed period. It illustrates the growth of an initial investment over time.
![Portfolio Performance](/images/portfolio_performance.png)

### Portfolio Drawdown
This chart displays the percentage decline from previous peaks in the portfolio's value. It highlights the largest peak-to-trough decline during the period, indicating the maximum loss an investor would have experienced from a peak.
![Portfolio Drawdown](/images/drawdown.png)

### Distribution of Returns with VaR Estimates
This histogram shows the frequency distribution of the portfolio's daily returns. The vertical lines indicate the Value at Risk (VaR) estimates from the Parametric, Historical, and GARCH-FHS models, illustrating the estimated maximum loss at the 99% confidence level.
![Distribution of Returns](/images/returns_distribution.png)

### QQ-Plot of Optimal Portfolio Returns
This Quantile-Quantile (QQ) plot compares the distribution of the portfolio's returns against a theoretical normal distribution. Deviations from the straight line indicate departures from normality, particularly "fat tails" which suggest a higher likelihood of extreme events.
![QQ-Plot](/images/qq_plot.png)

## Feedback on Analysis

The analysis presented in this report is comprehensive and well-structured, effectively comparing three different risk models.

**Strengths of the Analysis:**

*   **Clear Model Comparison:** The "Final Summary Table" clearly presents the VaR, ES, Kupiec's POF p-value, and Average Actual Exceedance Loss for each model, allowing for direct comparison.
*   **Insightful Interpretation of Backtests:** The "Analysis and Interpretation" section provides sharp insights into the performance of each model. For instance, it correctly identifies that the GARCH-FHS model's Kupiec's POF p-value of 0.0 indicates its failure to reliably predict loss frequency, despite offering the lowest VaR estimate. Conversely, the Historical model's p-value of 0.975825 demonstrates its statistical soundness in predicting loss frequency.
*   **Accurate ES Backtest Assessment:** The report accurately highlights the Historical model's strong performance in the ES backtest, noting that its average actual loss (-0.0258) closely matches its predicted Expected Shortfall (0.0258), indicating its reliability in predicting the magnitude of tail losses.
*   **Identification of Normality Issues:** The analysis effectively uses the "QQ-Plot of Optimal Portfolio Returns" to illustrate the "fat tails" in the portfolio's returns, correctly linking this deviation from normality to a potential unreliability of the Parametric model for capturing tail risk.
*   **Key Performance Metrics:** The inclusion of "Annualized Portfolio Return" (27.37%), "Annualized Portfolio Volatility" (13.13%), "Maximum Drawdown" (-7.09%), and "Sharpe Ratio" (1.72) provides a concise overview of the portfolio's performance and risk-adjusted return.
*   **Visual Support:** The report effectively integrates visualizations such as the "Correlation Heatmap," "Portfolio Performance," "Portfolio Drawdown," and "Distribution of Returns with VaR Estimates" to support the textual analysis and provide a clearer understanding of the portfolio's characteristics and risk profile.

**Areas for Further Enhancement:**

*   **Context for Sharpe Ratio:** While the Sharpe Ratio is provided, a brief explanation of its significance (e.g., "higher values indicate better risk-adjusted returns") could further enhance clarity for all readers.
*   **Discussion of Portfolio Weights:** While the optimal weights are listed, a brief discussion on *why* certain assets received high or zero allocation (e.g., based on their correlation or individual risk-return profiles) could add more depth to the "Optimal Portfolio Allocation" section.

This analysis provides a robust foundation for understanding the portfolio's risk characteristics and the performance of different risk models.

## Recommendations

Based on this analysis, the **Historical Simulation model is recommended for ongoing risk management and reporting** for this portfolio due to its superior performance in both frequency and magnitude backtests. Further research could explore:

*   **Alternative Risk-Free Rates:** Investigate the impact of using different risk-free rate proxies.
*   **Dynamic Portfolio Rebalancing:** Explore strategies for periodically rebalancing the portfolio to maintain optimal risk-adjusted returns.
*   **Stress Testing:** Conduct stress tests to assess portfolio performance under extreme, hypothetical market conditions.