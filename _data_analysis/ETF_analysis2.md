---
title: "ETF Portfolio Risk Analysis 2: Backtesting Parametric vs. Non-Parametric Models"
collection: data_analysis
permalink: /data_analysis/etf-risk-analysis-backtesting2/
excerpt: "An in-depth comparison and backtest of three risk models for VaR & ES estimation on an optimized ETF portfolio, analyzing the performance of 'Buy-and-Hold' vs. 'Dynamic Rebalancing' strategies."
date: 2025-08-11
tags:
  - Portfolio Analysis
  - Risk Management
  - VaR
  - Expected Shortfall
  - Backtesting
  - GARCH
  - Asset Allocation
  - Rebalancing
---

# **ETF Portfolio Risk Analysis Report**

This blog post is practical application for my previous post in the data analysis blog [“ETF Portfolio Risk Analysis: Backtesting Parametric vs. Non-Parametric Models”](https://anderson-shin.github.io/data_analysis/etf-risk-analysis-backtesting)  and demonstrates how rebalancing technique could be considered. I have added individual components performance data to give clearer view for diversification effect.

## **1. Executive Summary**

This report provides an in-depth risk analysis of an optimized portfolio composed of five ETFs (VTI, VXUS, BND, VNQ, GLD) for the period **January 1, 2024, to July 31, 2025**. The analysis evaluates three distinct risk models (Parametric, Historical, and GARCH-FHS) and compares the performance of a "Buy-and-Hold" strategy against a "Quarterly Dynamic Rebalancing" strategy.

**Key Findings:**

1. **Optimal Risk Model:** The **Historical Simulation model** was identified as the most reliable risk assessment tool. It successfully passed the Kupiec's POF backtest with a high p-value (0.98) and its Expected Shortfall (ES) estimate of \-2.58% almost perfectly matched the average actual loss during tail events.  
2. **Optimal Investment Strategy:** The **Buy-and-Hold strategy** delivered superior performance compared to dynamic rebalancing, achieving a higher Sharpe Ratio (1.72 vs. 1.14). This indicates that for the given period, maintaining the initial optimal allocation was more efficient than incurring the costs and potential whipsaws of frequent trading.  
3. **Optimal Portfolio Composition:** The portfolio was allocated to **US Equities (VTI) at 32.85%** and **Gold (GLD) at 67.15%**. This structure leveraged the exceptionally low correlation (0.11) between the two assets to achieve effective diversification and strong risk-adjusted returns.

## **2. Methodology**

This analysis was conducted using a modular Python framework. The process is broken down into the following key steps:

1. **Data Fetching (data_loader.py):** Historical daily 'Adjusted Close' prices for the specified tickers were fetched from Yahoo Finance using the yfinance library.  
   ```python
   # Fetches data for all tickers including the risk-free asset  
   data = fetch_data(tickers + [risk_free_ticker], start_date, end_date)
   ```

2. **Portfolio Optimization (portfolio_optimizer.py):** The portfolio was optimized to maximize the Sharpe Ratio. This was achieved using the scipy.optimize.minimize function with the 'SLSQP' method to find the optimal asset weights.  
   ```python
   # Calculates optimal weights to maximize the Sharpe Ratio  
   optimal_weights = optimize_portfolio(risky_returns, risk_free_rate)
   ```

3. **Risk Modeling (risk_analyzer.py):** Three different models were used to calculate Value at Risk (VaR) and Expected Shortfall (ES) at a 99% confidence level.  
   * **Parametric:** Assumes a normal distribution of returns.  
   * **Historical:** Uses the empirical distribution of past returns.  
   * **GARCH-FHS:** A hybrid model using a GARCH(1,1) model (from the arch library) to forecast volatility combined with historical simulation on standardized residuals.  
4. **Backtesting (risk_analyzer.py):** The VaR models were backtested using Kupiec's Proportion of Failures (POF) test to check the frequency of exceptions. The ES models were validated by comparing the predicted ES to the average of actual losses that exceeded the VaR threshold.  
5.  **Dynamic Rebalancing (`portfolio_optimizer.py`):** To contrast the static "Buy-and-Hold" approach, a dynamic rebalancing strategy was simulated. This strategy was implemented using the `run_dynamic_rebalancing` function with a frequency set to **Quarter End ('QE')**. The process works as follows:
    * At the end of each quarter, the strategy uses all available historical return data *up to that point* to re-run the Sharpe Ratio optimization, calculating a new set of optimal weights.
    * These new weights are then applied to the portfolio for the *entire next quarter* (the holding period).
    * The daily returns for each holding period are calculated and then concatenated to create a single, continuous performance series for the dynamically rebalanced portfolio. This method allows the strategy to adapt to changing market conditions based on the most recent historical performance.

## **3. Optimal Portfolio Composition & Strategy Analysis**

### **3.1. Portfolio Asset Allocation & Diversification**

The portfolio was optimized to maximize the Sharpe Ratio, resulting in the following asset allocation:

* **BND (U.S. Total Bond Market):** 0.00%  
* **GLD (Gold):** 67.15%  
* **VNQ (U.S. Real Estate):** 0.00%  
* **VTI (U.S. Total Stock Market):** 32.85%  
* **VXUS (Total International Stock):** 0.00%

The heatmap below visualizes the correlation between the assets. The key takeaway is the **very low correlation of 0.11 between VTI and GLD**. This lack of correlation is the primary driver of the portfolio's efficiency, as the two assets' price movements do not strongly influence each other, providing a powerful diversification benefit.

![Correlation Heatmap](/images/20250811_200754_correlation_heatmap.png)

### **3.2. Rationale for Asset Selection: Individual Risk-Return Profiles**

The chart below illustrates the annualized return and volatility for each asset.

![Individual Asset Risk-Return Profiles](/images/20250811_200754_individual_asset_performance.png)

During the analysis period, **GLD** delivered the highest annualized return (30.6%), while **VTI** also showed strong performance (20.8%). The optimization algorithm selected these two top-performing assets and capitalized on their low correlation to minimize volatility and maximize risk-adjusted returns.

| Ticker | Annualized Return | Annualized Volatility |
| :---- | :---- | :---- |
| BND | 3.65% | 5.31% |
| GLD | 30.58% | 16.66% |
| VNQ | 6.22% | 17.52% |
| VTI | 20.81% | 17.53% |
| VXUS | 15.80% | 15.02% |

### **3.3. Strategy Comparison: Dynamic Rebalancing vs. Buy-and-Hold**

To determine the most effective investment approach for the period, the performance of the static **"Buy-and-Hold"** strategy was compared against an active **"Dynamic Rebalancing"** strategy.

* The **Buy-and-Hold** approach involves applying the initial optimal weights and holding them for the entire duration, representing a passive investment style.
* The **Dynamic Rebalancing** approach, in contrast, actively adapts to the market. At the end of each quarter, it re-evaluates past performance to calculate new optimal weights, which are then held until the next rebalancing date. This strategy aims to systematically sell high and buy low, potentially reducing risk.

The following table and chart compare the results of these two distinct approaches.

![Rebalancing vs. Buy-and-Hold](/images/20250811_200754_rebalancing_comparison.png)



| Strategy | Annualized Return | Annualized Volatility | Sharpe Ratio | Max Drawdown (MDD) |
| :---- | :---- | :---- | :---- | :---- |
| **Buy and Hold** | **27.37%** | **13.13%** | **1.72** | \-7.09% |
| Dynamic Rebalancing | 29.83% | 21.93% | 1.14 | **\-5.86%** |

**Analysis:** The **Buy-and-Hold** strategy generated a higher return and a superior Sharpe Ratio. This outperformance is attributed to the strong upward trend of both VTI and GLD during the period, where allowing the winners to run was more profitable than rebalancing. However, it is noteworthy that the Dynamic Rebalancing strategy provided better downside protection, as evidenced by its lower Maximum Drawdown.

### **3.4. Buy-and-Hold Performance Deep-Dive**

The **Buy-and-Hold** strategy, being the recommended approach, demonstrated strong growth over the period. The chart below shows the cumulative performance, illustrating the growth of a hypothetical $1 investment.

![Portfolio Performance](/images/20250811_200754_portfolio_performance.png)

The portfolio's resilience is highlighted by its drawdown profile. The **Maximum Drawdown was \-7.09%**, meaning the largest peak-to-trough decline experienced was relatively contained. This specific event is visible as the lowest point in the drawdown chart below.

![Portfolio Drawdown](/images/20250811_200754_drawdown.png)

## **4. Portfolio Risk Model Evaluation**

### **4.1. Return Distribution and Risk Characteristics**

The portfolio's daily return distribution exhibits "fat tails," meaning the probability of extreme losses is higher than a normal distribution would suggest. This is visually confirmed by the histogram and the QQ-Plot.

| Distribution of Returns with VaR Estimates | QQ-Plot of Portfolio Returns |
| :---- | :---- |
| ![Distribution of Returns with VaR Estimates](/images/20250811_200754_returns_distribution.png) | ![QQ-Plot of Portfolio Returns](/images/20250811_200754_qq_plot.png) |

**Analysis:** The histogram on the left shows the frequency of daily returns, with the vertical lines representing the VaR estimates from each model. The QQ-plot on the right compares the portfolio's returns (blue dots) to a theoretical normal distribution (red line). The dots deviating from the line at the lower and upper ends are clear visual evidence of "fat tails," indicating that extreme events are more common than a normal distribution would predict.

### **4.2. VaR and ES Model Backtesting Results**

Three risk models were backtested to assess their accuracy in predicting Value at Risk (VaR) and Expected Shortfall (ES) at a 99% confidence level.

| Model | VaR (99%) | ES (99%) | Kupiec's POF p-value | Avg. Actual Exceedance Loss | Accuracy |
| :---- | :---- | :---- | :---- | :---- | :---- |
| Parametric | \-1.82% | \-2.10% | 0.16 | \-2.26% | Inaccurate |
| **Historical** | **\-1.89%** | **\-2.58%** | **0.98** | **\-2.58%** | **Highly Accurate** |
| GARCH-FHS | \-2.21% | \-2.59% | 0.62 | \-2.75% | Moderately Accurate |

**Analysis:** The Parametric model, which assumes normality, significantly underestimated the magnitude of actual losses due to the fat-tailed nature of the returns. In contrast, the **Historical model** proved to be the most reliable, demonstrating exceptional accuracy in predicting both the frequency of losses (p-value of 0.98) and their average magnitude (-2.58% predicted vs. \-2.58% actual).

## **5. Conclusion and Recommendations**

This analysis yields three primary conclusions:

1. **Portfolio Strength:** The allocation strategy, leveraging the low correlation between U.S. equities (VTI) and gold (GLD), was highly effective at generating superior risk-adjusted returns.  
2. **Optimal Strategy:** For the analyzed period, which was characterized by strong market trends, a passive **Buy-and-Hold** approach was more profitable than an active rebalancing strategy.  
3. **Best Risk Tool:** The **Historical Simulation model**, which relies on actual past data rather than theoretical assumptions, provided the most accurate measure of the portfolio's tail risk.

**Recommendations:**

* **Risk Management:** It is recommended to use **Historical VaR and ES** as the primary metrics for setting risk limits and monitoring the portfolio's downside exposure.  
* **Strategy Monitoring:** In a different market regime, such as a volatile, sideways market, the Dynamic Rebalancing strategy might prove superior due to its better downside protection. Therefore, ongoing performance monitoring is crucial.  
* **Further Analysis:** As a next step, we recommend conducting **Stress Tests** based on specific macroeconomic scenarios (e.g., an inflation shock or interest rate event) to further probe the portfolio's resilience and identify potential vulnerabilities.
