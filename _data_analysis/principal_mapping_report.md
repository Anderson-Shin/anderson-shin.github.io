---
title: "Bond ETF Portfolio Risk Analysis: Backtesting a Dynamic VaR Model with Principal Mapping"
collection: data_analysis
permalink: /data_analysis/bond-etf-var-principal-mapping/
excerpt: "A detailed implementation and backtest of a dynamic Value at Risk (VaR) model for a fixed-weight bond ETF portfolio, utilizing the Principal Mapping technique and validated with Kupiec's POF test."
date: 2025-08-13
tags:
  - Portfolio Analysis
  - Risk Management
  - VaR
  - Backtesting
  - Principal Mapping
  - Fixed Income
  - Bond ETF
---

# **1\. Introduction**

This report presents a comprehensive Value at Risk (VaR) analysis for a diversified bond ETF portfolio. The primary objective was to implement a **dynamic VaR model** using the **Principal Mapping** technique and to rigorously validate its predictive accuracy. VaR is a critical risk metric that quantifies potential portfolio losses, and this analysis demonstrates a robust framework for its application in managing fixed-income risk.

# **2\. Methodology**

## **2.1. Portfolio and Risk Factors**

The analysis was conducted on a hypothetical **$1,000,000 portfolio** with equal initial weights (33.3% each) allocated to three distinct bond ETFs: **SHY, LQD, and TLT**. The specific characteristics of each ETF used in the model are as follows:

| Ticker | Description | WAL (Years) | Effective Duration (Years) |
| :---- | :---- | :---- | :---- |
| **SHY** | 1-3 Year Treasury Bond | 1.94 | 1.85 |
| **LQD** | Inv. Grade Corporate Bond | 12.84 | 8.05 |
| **TLT** | 20+ Year Treasury Bond | 25.85 | 15.57 |


Daily changes in the U.S. Treasury zero-coupon yield curve at key vertices (1Y, 2Y, 5Y, 7Y, 10Y, 20Y, 30Y) were defined as the primary risk factors driving portfolio value changes.

## **2.2. VaR Calculation: Principal Mapping & Dynamic Estimation**

The portfolio's risk was modeled in three distinct steps: mapping market value exposures, calculating price sensitivity, and computing the final dynamic VaR.

### **Step 1: Exposure Mapping (Position Vector, x)**

The market value of each ETF was mapped to the two nearest standard yield curve vertices based on its **Weighted Average Life (WAL)** using linear interpolation. The aggregated exposures at each vertex form the position vector **x**.

### **Step 2: Price Sensitivity (Dollar Duration Vector, DD)**

For each vertex i, the dollar duration was computed as:

$DD_i​=x_i​×D_i​×0.01$  
Where:

* $DD_i$ is the Dollar Duration for vertex i.  
* $x_i$ is the mapped market value exposure at vertex i.  
* $D_i$ is the Effective Duration associated with vertex i.  
* The 0.01 factor scales the value to represent the dollar change for a 1 percentage point (100 basis points) change in yield.

### **Step 3: Dynamic VaR Calculation**

To capture time-varying market volatility, a **252-day rolling window** was used. For each day t in the backtesting period, the VaR was calculated as:

$\text{VaR}_t​ = z \times \sqrt{DD^TΣ_t​DD​}$  
Where:

* $z$ is the z-score for the 99% confidence level (≈2.33).  
* $DD$ is the static Dollar Duration vector.  
* $\Sigma_t$ is the covariance matrix of daily yield changes, re-estimated for each day t using data from the preceding 252 days.

## **2.3. Model Validation: Backtesting & Kupiec's POF Test**

The model's accuracy was validated by comparing the forecasted daily VaR against the portfolio's actual daily Profit & Loss (P&L). An "exception" is recorded if $\text{P&L}_t\leq\text{VaR}_t$

The statistical validity was assessed using **Kupiec's Proportion of Failures (POF) test**. The test uses a likelihood ratio statistic, LR\_POF, to determine if the observed failure rate is consistent with the expected failure rate. The formula is:

$\text{LRPOF}​=−2\ln(\frac{(1-p)^{N-x}p^x}{(1-\frac{x}{N})^{N-x}(\frac{x}{N})^x}$

Where:

* $N$ is the total number of observations.  
* $x$ is the number of exceptions.  
* $p$ is the expected exception rate (0.01 for 99% VaR).

The resulting statistic is compared against a Chi-squared distribution to obtain a p-value. A p-value greater than 0.05 indicates that the model is well-calibrated.

# **3\. Results and Analysis**
The analysis begins by examining the broader market context provided by the yield curve's behavior, followed by a detailed assessment of the portfolio's performance and the VaR model's accuracy.
## **3.1. Yield Curve Dynamics**

The chart below displays the historical movement of the U.S. Treasury zero-coupon yield curve from 2022 to 2025. A significant upward trend in interest rates across all maturities is visible throughout 2022 and early 2023, reflecting a period of monetary tightening. The subsequent period, which includes our backtesting timeframe, is characterized by heightened volatility and fluctuating yields. This dynamic environment provides a robust setting for testing the VaR model's ability to adapt to changing market conditions.

![Yield Curve dynamics](/images/yield_curve_dynamics.png)

## **3.2. Portfolio Performance Summary**

Over the backtesting period from January 3, 2024, to July 30, 2025, the portfolio exhibited the following performance characteristics:

| Metric | Value |
| :---- | :---- |
| **Total Cumulative Return** | 2.28% |
| **Annualized Volatility** | 7.68% |
| **Average 99% VaR** | **$11,532.74** |

The **Portfolio Cumulative Return** chart below illustrates the growth trajectory of the portfolio throughout the period. The Average 99% VaR indicates that, on an average day, the model predicted a maximum potential loss of approximately $11,533 at the 99% confidence level.

## **3.3. Risk Factor and Concentration Analysis**

![Yield Change Correlation Heatmap](/images/yield_change_correlation_heatmap.png)

The **Correlation Heatmap** reveals a high degree of positive correlation (often \>0.80) between adjacent yield curve vertices, particularly in the 5-to-30-year range. This suggests that the yield curve primarily moves in broad, coordinated shifts (i.e., parallel shifts, steepening, or flattening) driven by macroeconomic factors, rather than through isolated, random movements at individual points.

![Risk Concentration by Vertex](/images/risk_concentration.png)

The **Risk Concentration** chart visually represents the mapped exposures from the methodology section. It clearly shows that the portfolio's interest rate risk is dominated by its exposure to the 2-year, 10-year, 20-year, and 30-year vertices. This is a direct consequence of the portfolio's significant allocation to LQD (Duration: 8.05) and TLT (Duration: 15.57), indicating that the portfolio's overall volatility will be heavily influenced by changes in the mid-to-long end of the yield curve. The table below shows the resulting aggregated market value exposure at each vertex.

| Vertex | Market Value |
| :---- | :---- |
| 1 | $20,000.00 |
| 2 | $313,333.33 |
| 5 | $0.00 |
| 7 | $0.00 |
| 10 | $238,666.67 |
| 20 | $233,000.00 |
| 30 | $195,000.00 |

## **3.4. VaR Model Validation**

![Daily P&L Distribution](/images/daily_pnl_distribution.png)

The backtesting results provide strong evidence of the model's accuracy. The **Daily P\&L Distribution** histogram shows a near-normal distribution of returns, with the calculated VaR threshold effectively capturing the tail of negative outcomes.

![Daily P&L vs. 99% VaR](/images/pnl_vs_var.png)

The core backtesting results are summarized in the table below. The model experienced **3 exceptions**, a number that is not statistically different from the **1.41 exceptions** expected under a 99% confidence level.

| Metric | Value |
| :---- | :---- |
| **Total Observations** | 141 |
| **Number of Exceptions** | **3** |
| **Expected Exceptions (1%)** | 1.41 |
| **LR\_POF Statistic** | 1.368 |
| **P-value** | **0.242** |

This alignment is quantitatively confirmed by the **Kupiec's POF test**, which yielded a **p-value of 0.242**. As this p-value is substantially greater than the common 0.05 significance threshold, we fail to reject the null hypothesis, providing strong statistical evidence that the model is **well-calibrated and does not systematically under- or overestimate risk**.

The specific details of the three observed exceptions are provided in the table below, corresponding to the red markers in the subsequent chart.

| Date | Daily P\&L | 99% VaR Forecast |
| :---- | :---- | :---- |
| 2025-04-07 | \-$18,943.40 | $11,301.12 |
| 2025-04-08 | \-$11,949.71 | $11,439.08 |
| 2025-04-10 | \-$16,486.74 | $11,459.86 |

The **Daily P\&L vs. 99% VaR** chart visually confirms these findings, showing the three instances where the daily loss (gray line) breached the dynamic VaR threshold (red dashed line).

# **4\. Conclusion**

This analysis successfully implemented and validated a dynamic Value at Risk model for a diversified bond portfolio. The results demonstrate that the portfolio's risk is primarily driven by its exposure to long-duration assets, with risk factors exhibiting strong co-movement. The dynamic VaR model, which leverages Principal Mapping and a rolling-window estimation, proved highly effective.

The backtesting results, confirmed by a statistically insignificant Kupiec's POF test (p-value \= 0.242), show the model is **well-calibrated and provides a reliable measure of potential portfolio losses**. This framework therefore serves as a robust and practical tool for active risk management of fixed-income portfolios.

# **5\. Limitations and Further Research**

While the implemented model proved to be robust, it is important to acknowledge its limitations. The **Principal Mapping** technique, while efficient, does not fully capture non-linear risks such as **convexity**. For portfolios with significant optionality (e.g., mortgage-backed securities), this could lead to an underestimation of risk under large yield shifts.

For **further research**, the model's accuracy could be benchmarked against more sophisticated methodologies like **Historical Simulation** or **Monte Carlo Simulation**. Additionally, implementing **Stress Testing** and **Scenario Analysis** based on historical crises (e.g., the 2008 financial crisis) would provide deeper insights into the portfolio's tail risk.