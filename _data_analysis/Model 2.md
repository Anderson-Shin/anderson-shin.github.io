---
title: "Term Structure Modeling Part 2: Model 2 (Drift Model)"
collection: data_analysis
permalink: /data_analysis/term-structure-modeling2/
excerpt: "Implementation and analysis of a Drift Term Structure model with parameter estimation, Monte Carlo simulations, and comparative analysis against the No-Drift Model."
date: 2025-08-21
toc: true
toc_sticky: true
tags:
  - Term Structure
  - Interest Rate Modeling
  - Drift Model
  - Monte Carlo Simulation
  - Risk Management
---

# Interest Rate Term Structure Analysis: Model 2 (Drift Model)

## Table of Contents
1. [Executive Summary](#-executive-summary)
2. [Methodology and Data Analysis](#-methodology-and-data-analysis)
   - [1. Data Preprocessing and Market Environment](#1-data-preprocessing-and-market-environment)
   - [2. Parameter Estimation](#2-parameter-estimation)
3. [Model 2 Implementation](#-model-2-implementation)
4. [Monte Carlo Simulation Results](#-monte-carlo-simulation-results)
5. [Model Validation and Performance Assessment](#-model-validation-and-performance-assessment)
6. [Comparative Analysis: Model 1 vs. Model 2](#-comparative-analysis-model-1-vs-model-2)
7. [Model Assessment: Strengths and Limitations](#-model-assessment-strengths-and-limitations)
8. [Economic Interpretation and Market Implications](#-economic-interpretation-and-market-implications)
9. [Conclusions and Key Findings](#-conclusions-and-key-findings)

## 📊 Executive Summary

This report presents a comprehensive analysis of interest rate dynamics using Model 2 (Drift Model), calibrated to U.S. Treasury yield data. Model 2 extends the basic No-Drift Model by incorporating a deterministic drift component, expressed as the stochastic differential equation (SDE) `dr = λdt + σdW`. Through rigorous parameter estimation and Monte Carlo simulation, we explore how this additional drift term affects the forecasting of future interest rate paths. The analysis reveals that while the drift parameter is not statistically significant at the 5% level (p-value = 0.071), it nonetheless has a substantial economic impact on long-term rate projections, resulting in a forecast mean of 8.21% after 5 years compared to the initial rate of 4.31%.

## 🔬 Methodology and Data Analysis

### 1. Data Preprocessing and Market Environment

The analysis utilizes U.S. Treasury yield data spanning from May 2020 to August 2025, providing a comprehensive view of interest rate behavior across various economic environments.

**Dataset Characteristics:**
- **Time Period**: 2020-05-21 to 2025-08-15
- **Maturities**: 1M, 3M, 6M, 2Y, 5Y, 10Y, 20Y, 30Y
- **Data Treatment**: Forward-fill methodology for missing values, percentage-to-decimal conversion

**Data Processing Pipeline:**
```python
# Load and preprocess Treasury yield data
import pandas as pd
import numpy as np

file_path = 'treasury yield.xlsx'
df = pd.read_excel(file_path)

# Format date column and set as index
df.rename(columns={df.columns[0]: 'Date'}, inplace=True)
df['Date'] = pd.to_datetime(df['Date'])
df.set_index('Date', inplace=True)

# Handle missing values and convert data types
df.replace('#N/A N/A', np.nan, inplace=True)
df = df.astype(float)
df.fillna(method='ffill', inplace=True)
df.dropna(inplace=True)

# Convert percentage rates to decimal format
df_decimal = df / 100
```

**Latest Interest Rate Levels (as of 2025-08-15):**

| Maturity | Rate  |
| -------- | ----- |
| 1M       | 4.31% |
| 3M       | 4.21% |
| 6M       | 4.07% |
| 2Y       | 3.75% |
| 5Y       | 3.84% |
| 10Y      | 4.32% |
| 20Y      | 4.89% |
| 30Y      | 4.92% |

### 2. Parameter Estimation

Model 2 requires the estimation of two key parameters: the drift coefficient (λ) and the volatility coefficient (σ). We employed three different estimation methods to ensure robustness.

**Parameter Estimation Methods:**

1. **Method of Moments**:
   ```python
   # Extract short-term rate (1M) and calculate rate changes
   short_rate = df_decimal['1M']
   rate_changes = short_rate.diff().dropna()
   
   # Calculate drift and volatility using method of moments
   dt = 1/252  # Daily time step
   lambda_hat = rate_changes.mean() / dt
   sigma_hat = rate_changes.std() / np.sqrt(dt)
   ```

2. **Maximum Likelihood Estimation**:
   ```python
   # Define negative log-likelihood function for Gaussian process
   def neg_log_likelihood(params, rate_changes, dt):
       lambda_param, sigma_param = params
       mu = lambda_param * dt
       var = sigma_param**2 * dt
       return -np.sum(norm.logpdf(rate_changes, loc=mu, scale=np.sqrt(var)))
   
   # Optimize to find parameters
   initial_guess = [lambda_hat, sigma_hat]
   result = minimize(neg_log_likelihood, initial_guess, 
                     args=(rate_changes.values, dt), 
                     method='BFGS')
   lambda_mle, sigma_mle = result.x
   ```

3. **Regression Analysis**:
   ```python
   # Create constant time step array for regression
   X = np.ones((len(rate_changes), 1)) * dt
   y = rate_changes.values
   
   # Fit linear regression model
   reg_model = LinearRegression(fit_intercept=False)
   reg_model.fit(X, y)
   lambda_reg = reg_model.coef_[0]
   
   # Calculate volatility from residuals
   residuals = y - reg_model.predict(X)
   sigma_reg = np.std(residuals) / np.sqrt(dt)
   ```

**Estimation Results:**

| Method | Drift Coefficient (λ) | Volatility (σ) |
| ------ | --------------------- | -------------- |
| Method of Moments | 0.007834 | 0.010081 |
| Maximum Likelihood | 0.007834 | 0.010077 |
| Regression Analysis | 0.007834 | 0.010077 |

**Final Model Parameters:**
- **Drift (λ)**: 0.007834 (0.78% annually)
- **Volatility (σ)**: 0.010077 (1.01% annually)
- **Initial Rate (r₀)**: 0.043136 (4.31%)

**Statistical Significance of Drift:**
A hypothesis test was conducted to assess whether the drift parameter is statistically different from zero.
- **t-statistic**: 1.8086
- **p-value**: 0.0705
- **Conclusion**: The drift parameter is not statistically significant at the 5% level, but shows potential significance at the 10% level.

## 🧮 Model 2 Implementation

Model 2 extends the basic interest rate model by incorporating a deterministic drift term, as defined by the stochastic differential equation:

`dr = λdt + σdW`

Where:
- `dr` is the infinitesimal change in the interest rate
- `λ` is the drift coefficient (0.78% annually)
- `dt` is the time increment
- `σ` is the volatility coefficient (1.01% annually)
- `dW` is a random increment from a standard Wiener process

In discrete time implementation, this becomes:

`r(t+Δt) = r(t) + λΔt + σ√Δt * Z`

Where Z ~ N(0,1) is a standard normal random variable.

The model assumes that interest rates have a constant directional trend (drift) plus random fluctuations. The positive drift parameter (λ = 0.007834) suggests an upward trend in rates over time.

## 📈 Monte Carlo Simulation Results

We simulated 10,000 future interest rate paths over a 5-year period using Model 2 parameters.

**Simulation Setup:**
- **Number of Paths**: 10,000
- **Simulation Period**: 5.0 years
- **Time Step**: Daily (Δt = 1/252)
- **Initial Rate (r₀)**: 4.31%
- **Drift (λ)**: 0.007834
- **Volatility (σ)**: 0.010077

**Statistical Analysis of Simulation Results:**

| Period | Average Rate | Std. Dev. | Min Rate | Max Rate | Theoretical Mean | Theoretical Std. Dev. |
|--------|--------------|-----------|----------|----------|------------------|------------------------|
| 1 Year | 5.10% | 1.01% | 2.05% | 10.16% | 5.09% | 1.01% |
| 3 Years | 6.66% | 1.74% | 1.33% | 13.52% | 6.66% | 1.75% |
| 5 Years | 8.21% | 2.24% | 0.58% | 16.68% | 8.23% | 2.25% |

**Key Observations:**
- The mean interest rate increased steadily over time, rising from 4.31% to 8.21% over 5 years
- Standard deviation grew with the square root of time, as expected
- The empirical drift effect after 5 years was approximately 3.91%
- The probability of negative interest rates was extremely low (0.1%), much lower than in Model 1 (5.25%)

![Model 2 Simulation Results](/images/model2_simulation.png)

The figure above illustrates the key simulation results from Model 2, including:

**Top Row:**
- **Left**: 100 sample interest rate paths showing the upward drift trend with volatility
- **Middle**: The average path with 95% confidence interval, clearly depicting the rising trend
- **Right**: The final interest rate distribution after 5 years, centered around 8.21%

**Middle Row:**
- **Left**: Mean interest rate evolution showing perfect alignment between empirical and theoretical projections
- **Middle**: Volatility evolution confirming the square-root-of-time rule
- **Right**: Negative interest rate probability over time, remaining near zero until year 3-4

**Bottom Row:**
- **Left**: Percentile evolution showing how all percentiles move upward over time due to drift
- **Middle**: Drift effect measurement showing the cumulative impact of the drift component over time
- **Right**: Model characteristics summary highlighting key parameters and simulation outcomes

## 🔍 Model Validation and Performance Assessment

The model's performance was rigorously evaluated by comparing simulated results with theoretical expectations across multiple dimensions.

**Validation Tests:**

- **Mean Path Analysis**: 
  - Theoretical mean at time t: r₀ + λt
  - Empirical results: The mean path followed the theoretical trajectory with remarkable precision
  - After 5 years: Theoretical mean = 8.23%, Empirical mean = 8.21% (0.24% relative error)

- **Variance Growth Analysis**: 
  - Theoretical variance at time t: σ²t
  - Empirical results: Variance increased linearly with time as predicted
  - Standard deviation at 5 years: Theoretical = 2.25%, Empirical = 2.24%

- **Distribution Tests**: 
  - **Shapiro-Wilk Test**: Applied to terminal distribution, p-value > 0.05, confirming normality
  - **Kolmogorov-Smirnov Test**: Comparing rate changes to normal distribution, p-value > 0.05, indicating excellent fit
  - **Visual Confirmation**: The histogram of final rates closely matched the theoretical normal distribution with the expected mean and variance

- **Percentile Evolution**: As shown in the "Percentile Evolution" plot, the empirical percentiles (5%, 25%, 50%, 75%, 95%) all demonstrated the expected upward trend due to the drift parameter, with the spread widening proportionally to the square root of time.

- **Drift Effect Isolation**: By comparing with Model 1, we quantified that the drift component alone contributed approximately 3.91% to the average rate after 5 years, exactly matching the theoretical calculation λ × T = 0.00783 × 5 = 0.0392 (3.92%).

## 🔄 Comparative Analysis: Model 1 vs. Model 2

A detailed comparison between the No-Drift Model (Model 1) and the Drift Model (Model 2) revealed significant differences in their forecasting capabilities. The visual comparison of simulation results clearly demonstrates the impact of adding a drift component to the interest rate model.

**Five-Year Forecast Comparison:**

| Metric | Model 1 (No Drift) | Model 2 (With Drift) |
|--------|-------------------|----------------------|
| Mean Rate | 4.30% | 8.21% |
| Standard Deviation | 2.23% | 2.24% |
| Minimum Rate | -3.33% | 0.58% |
| Maximum Rate | 12.76% | 16.68% |
| Negative Rate Probability | 5.25% | 0.10% |

**Key Differences Visualized in the Simulation Results:**

- **Path Trajectories**: Model 1 paths oscillate around the initial rate with equal probability of rising or falling, while Model 2 paths show a clear upward trend with a shifted distribution.

- **Mean Paths**: Model 1's mean path remains flat at the initial rate of 4.31%, while Model 2's mean path rises linearly to 8.21% after 5 years.

- **Confidence Intervals**: Both models show widening confidence intervals over time, but Model 2's intervals are shifted upward due to the drift component.

- **Terminal Distributions**: Model 1's distribution is centered at the initial rate, while Model 2's distribution is shifted right, centered around 8.21%.

- **Negative Rate Probability**: As visible in the "Negative Interest Rate Probability Over Time" plot, Model 2 maintains near-zero probability of negative rates until year 3, and even then reaches only 0.1%, whereas Model 1 shows an increasing probability reaching 5.25% by year 5.

- **Percentile Evolution**: Model 2's percentiles all trend upward over time, even the lower 5% percentile eventually rises above the initial rate, whereas Model 1's lower percentiles trend downward.

## ⚖️ Model Assessment: Strengths and Limitations

**Strengths of Model 2:**
- Incorporates directional market expectations through the drift parameter
- Significantly reduces the probability of negative interest rates
- Produces more economically intuitive forecasts for long-term horizons
- Maintains mathematical tractability and ease of implementation

**Limitations of Model 2:**
- The drift parameter is not statistically significant at the conventional 5% level
- Still lacks mean-reversion properties observed in real interest rates
- Assumes constant drift and volatility, regardless of rate level
- Cannot explain changes in yield curve shape (only parallel shifts)
- May overestimate long-term rates in a high-drift scenario

## 💹 Economic Interpretation and Market Implications

The positive drift parameter (λ = 0.78% annually) aligns with market expectations of rising interest rates following the current economic conditions. This upward bias reflects:

1. **Inflation Expectations**: Anticipated price increases driving nominal rates higher
2. **Central Bank Policy**: Projected tightening cycles over the medium term
3. **Term Premium**: Compensation demanded by investors for interest rate risk

**Market Implications:**
- Fixed income securities may face price pressure as rates rise
- Duration risk becomes more pronounced in a rising rate environment
- Financial institutions should prepare for higher funding costs
- Borrowers may benefit from locking in current rates for long-term financing

## 🏁 Conclusions and Key Findings

Model 2 demonstrates how incorporating a simple drift component significantly enhances the economic realism of interest rate forecasts. Although the drift parameter was not statistically significant at the 5% level (p-value = 0.071), its economic impact is substantial, projecting a 3.91% increase in rates over a 5-year horizon.

The comparative analysis reveals that Model 2 provides a more plausible representation of future interest rates by:
1. Dramatically reducing the probability of negative rates
2. Incorporating market directional expectations
3. Maintaining analytical tractability

However, for more sophisticated applications requiring mean-reversion or level-dependent volatility, more advanced models such as Vasicek or Cox-Ingersoll-Ross would be appropriate. Model 2 nonetheless represents an important step beyond the purely random walk approach of Model 1, providing valuable insights for basic interest rate risk management and scenario analysis.
