---
title: "Term Structure Modeling Part 1: Model 1 (No-Drift Model)"
collection: data_analysis
permalink: /data_analysis/term-structure-modeling1/
excerpt: "Implementation and analysis of a No-Drift Term Structure model with Monte Carlo simulations for risk assessment and performance validation."
date: 2025-08-21
toc: true
toc_sticky: true
tags:
  - Term Structure
  - Interest Rate Modeling
  - No-Drift Model
  - Monte Carlo Simulation
---

# Interest Rate Term Structure Analysis: Model 1 (No-Drift Model)

## Table of Contents
1. [Executive Summary](#-executive-summary)
2. [Methodology and Data Analysis](#-methodology-and-data-analysis)
   - [1. Data Preprocessing and Market Environment](#1-data-preprocessing-and-market-environment)
   - [2. Volatility Parameter Estimation](#2-volatility-parameter-estimation)
3. [Model 1 Implementation](#-model-1-implementation)
4. [Monte Carlo Simulation Results](#-monte-carlo-simulation-results)
5. [Model Validation and Performance Assessment](#-model-validation-and-performance-assessment)
6. [Risk Analysis: VaR and ES](#-risk-analysis-var-and-es)
7. [Model Assessment: Strengths and Limitations](#-model-assessment-strengths-and-limitations)
8. [Conclusions and Key Findings](#-conclusions-and-key-findings)

## 📊 Executive Summary

This report presents a comprehensive analysis of short-rate dynamics using the No-Drift Model (Model 1), calibrated to U.S. Treasury yield data. This model, represented by the stochastic differential equation (SDE) `dr = σ dW`, is the most fundamental interest rate model, considering only the random volatility of interest rates without a drift component. Through Monte Carlo simulation, we forecast future interest rate paths, assess the model's statistical properties, and evaluate its inherent risks. The report also discusses the model's advantages and significant limitations to explore its applicability in real-world financial markets.

## 🔬 Methodology and Data Analysis

### 1. Data Preprocessing and Market Environment

To ensure robust model calibration, we utilized U.S. Treasury yield data spanning multiple market cycles. The dataset covers the period from May 2020 to August 2025, reflecting interest rate dynamics in various economic environments.

**Dataset Characteristics:**
- **Time Period**: 2020-05-21 to 2025-08-15
- **Maturities**: 1M, 3M, 6M, 2Y, 5Y, 10Y, 20Y, 30Y
- **Data Treatment**: Missing values were handled using a forward-fill method, and percentage units were converted to decimals for analysis.

**Data Processing Pipeline:**
```python
# Step 1: Load and preprocess Treasury yield data
import pandas as pd
import numpy as np

# Load the Excel file containing Treasury yields
file_path = 'treasury yield.xlsx'
df = pd.read_excel(file_path)

# Rename the first column to 'Date' and set it as a datetime index
df.rename(columns={df.columns[0]: 'Date'}, inplace=True)
df['Date'] = pd.to_datetime(df['Date'])
df.set_index('Date', inplace=True)

# Handle missing values and convert data types
df.replace('#N/A N/A', np.nan, inplace=True)
df = df.astype(float)
df.fillna(method='ffill', inplace=True)
df.dropna(inplace=True)

# Simplify column names
df.columns = ['1M', '3M', '6M', '2Y', '5Y', '10Y', '20Y', '30Y']
```

**Latest Interest Rate Levels (as of 2025-08-15):**
| Maturity | Rate  |
|----------|-------|
| 1M       | 4.31% |
| 3M       | 4.21% |
| 6M       | 4.07% |
| 2Y       | 3.75% |
| 5Y       | 3.84% |
| 10Y      | 4.32% |
| 20Y      | 4.89% |
| 30Y      | 4.92% |

### 2. Volatility Parameter Estimation

The instantaneous volatility (σ), a key parameter of Model 1, was estimated based on the daily changes of the 1-month Treasury bill yield.

**Volatility Calculation Methodology:**
```python
# Extract and preprocess the short-term rate (1M)
short_rate = df['1M'] / 100
# Calculate daily rate changes
rate_changes = short_rate.diff().dropna()
# Calculate annualized volatility
dt = 1 / 252  # Assuming 252 business days in a year
volatility_annualized = rate_changes.std() / np.sqrt(dt)
```

**Daily Interest Rate Change Statistics:**
- **Mean**: 0.000031
- **Standard Deviation**: 0.000635
- **Minimum**: -0.004511
- **Maximum**: 0.009141

**Volatility Estimation Results:**
| Parameter                | Value    | Interpretation |
|--------------------------|----------|----------------|
| Daily Volatility         | 0.000635 | Daily std. dev. of rate changes |
| Annualized Volatility (σ)| 0.010073 | Annual std. dev. (1.01%) |

The volatility parameter provides insight into expected interest rate variability:
- After 1 year: Standard deviation of ±1.01% 
- After 2 years: Standard deviation of ±1.42%

![Daily Rate Changes and Distribution](images/model1_volatility_distribution.png)

## 🧮 Model 1 Implementation

Model 1 is the most basic form of an interest rate model with no drift, defined by the following stochastic differential equation (SDE):

`dr = σ dW`

Where:
- `dr` is the infinitesimal change in the interest rate
- `σ` is the instantaneous volatility of the interest rate (0.010073)
- `dW` is a random increment from a standard Wiener process

In discrete time implementation, this becomes:

`r(t+Δt) = r(t) + σ√Δt * Z`

Where Z ~ N(0,1) is a standard normal random variable.

This model assumes that the future path of the interest rate is determined solely by random shocks, with no drift (directional trend), mean-reversion, or market expectation components. As a result, the expected value of future rates equals the current rate, and the variance grows linearly with time.

## 📈 Monte Carlo Simulation Results

We simulated 10,000 future interest rate paths over a 5-year period using Model 1. The initial rate `r₀` was set to 4.31%, reflecting the latest 1-month Treasury yield.

**Simulation Setup:**
- **Number of Simulation Paths**: 10,000
- **Simulation Period**: 5.0 years
- **Time Steps**: 1,260 (daily)
- **Time Interval (Δt)**: 0.003968 (1/252)
- **Initial Rate (r₀)**: 4.31%
- **Volatility (σ)**: 0.010073

**Statistical Analysis of Simulation Results:**

| Time Horizon | Average Rate | Std. Dev. | Min Rate | Max Rate | Theoretical Std. Dev. |
|--------------|--------------|-----------|----------|----------|-----------------------|
| 0.25 Year    | 4.31%        | 0.50%     | 2.37%    | 6.42%    | 0.50%                 |
| 0.5 Year     | 4.32%        | 0.72%     | 1.85%    | 7.03%    | 0.71%                 |
| 1.0 Year     | 4.31%        | 1.01%     | 0.76%    | 9.01%    | 1.01%                 |
| 2.0 Years    | 4.30%        | 1.41%     | -1.23%   | 9.53%    | 1.42%                 |
| 3.0 Years    | 4.30%        | 1.74%     | -1.71%   | 11.02%   | 1.74%                 |
| 5.0 Years    | 4.30%        | 2.23%     | -3.33%   | 12.76%   | 2.25%                 |

**Key Observations:**
- The mean interest rate remained constant at approximately 4.31%, consistent with the no-drift assumption
- Standard deviation increased proportionally to the square root of time, matching theoretical expectations
- After 5 years, the simulated rates showed a wide dispersion from -3.33% to 12.76%
- 525 out of 10,000 simulated paths (5.25%) resulted in negative interest rates
- The distribution of rates after 5 years was approximately normal, with a median of 4.29%

**Interest Rate Distribution After 5 Years:**
- **Mean**: 4.30%
- **Median**: 4.29% 
- **Standard Deviation**: 2.23%
- **5th Percentile**: 0.60%
- **25th Percentile**: 2.78%
- **75th Percentile**: 5.81%
- **95th Percentile**: 8.04%

![Model 1 Simulation Results](images/model1_simulation.png)

## 🔍 Model Validation and Performance Assessment

The validity of the model was rigorously tested by comparing the statistical properties of the simulated interest rate paths with their theoretical distributions.

**Statistical Tests and Validation:**

- **Mean Path Analysis**: The empirical mean of simulated rates remained at 4.30-4.31% throughout the simulation period, confirming the model's no-drift property.
  
- **Variance Growth**: The empirical variance increased linearly with time (σ²t), as evidenced by the close match between theoretical and simulated standard deviations at each time point.
  
- **Normality Tests**:
  - **Shapiro-Wilk Test**: Applied to the 1-year interest rate distribution, yielded a p-value > 0.05, confirming normality.
  - **Kolmogorov-Smirnov Test**: Comparing simulated increments to a normal distribution produced a high p-value (statistic = 0.008, p = 0.995), strongly supporting distributional alignment.
  
- **Autocorrelation Analysis**: Tests on rate changes showed negligible autocorrelation (mean = 0.006), validating the Markov property of the model.

- **Percentile Evolution**: The empirical percentiles (5%, 25%, 50%, 75%, 95%) evolved over time as predicted by the theoretical distribution, diverging symmetrically from the initial rate.

## ⚠️ Risk Analysis: VaR and ES

Based on the simulation results, we performed a comprehensive risk assessment of future interest rate distributions.

**One-Year Horizon Risk Metrics:**
- **Value at Risk (VaR)**: At a 95% confidence level, the 1-year VaR was 2.65%. This indicates a 5% probability that the interest rate will fall below 2.65% after one year.
  
- **Expected Shortfall (ES)**: The 1-year ES at 95% confidence was 1.98%, representing the average rate in the worst 5% of scenarios.
  
- **Negative Rate Probability**: After 1 year, the probability of negative rates was approximately 0%, but this increases to 5.25% by year 5.

**Risk Evolution Over Time:**
The probability of extreme rate movements increases with the time horizon, as evidenced by the widening confidence intervals. This highlights the growing uncertainty in long-term forecasts under the no-drift assumption.

## ⚖️ Model Assessment: Strengths and Limitations

**Strengths:**
- **Simplicity**: The model's parsimonious structure with just one parameter makes it easy to understand, implement, and calibrate.
- **Analytical Tractability**: Closed-form solutions exist for bond pricing and many interest rate derivatives.
- **Brownian Motion Properties**: Inherits well-understood mathematical properties including the Markov property and independent increments.
- **Ease of Interpretation**: Results are straightforward to interpret without complex interactions between parameters.

**Limitations:**
- **Negative Interest Rates**: The model allows rates to become negative (5.25% probability after 5 years), which may be unrealistic in many economic environments.
- **No Mean Reversion**: Real interest rates typically exhibit mean-reverting behavior, which this model fails to capture.
- **Constant Volatility**: The model assumes the same volatility regardless of the interest rate level, contrary to empirical evidence.
- **No Term Structure Modeling**: Cannot generate realistic yield curves or explain their dynamics, as it only models the short rate.
- **No Market Expectations**: Fails to incorporate forward-looking market information or risk premia.

## 🏁 Conclusions and Key Findings

Model 1 (No-Drift Model) serves as a foundational framework for understanding interest rate randomness through the lens of Brownian motion. Our extensive Monte Carlo simulation with 10,000 paths demonstrated that:

1. The model accurately captures the increasing uncertainty of future interest rates, with standard deviations growing proportionally to the square root of time.

2. The simulation confirmed the theoretical properties of the model, including constant mean, linearly increasing variance, and normally distributed rate changes.

3. The significant probability of negative rates (5.25% after 5 years) represents a fundamental limitation for long-term forecasting and highlights the need for more sophisticated models.

4. While unsuitable for pricing complex interest rate derivatives or long-term risk management, Model 1 provides valuable insights into the stochastic nature of interest rates and serves as an important building block for more advanced models.

For applications requiring greater realism, particularly regarding mean reversion, positive rate constraints, or incorporation of term structure, models such as Vasicek, Cox-Ingersoll-Ross (CIR), or Hull-White should be considered. Nevertheless, Model 1's elegant simplicity makes it an excellent starting point for understanding the mathematical foundations of interest rate modeling.
