---
title: "Optimization of portfolio 2"
collection: data_analysis
permalink: /data_analysis/yfinance_tutorial_4
excerpt: 'Optimization of portfolio when short sale is allowed using python. And aslo allocation of riksy assets and risk-free assets in the targeting portfolio'
date: 2024-03-23
venue: 'data anlaysis post 4'
---

# Optimization of the portfolio when the short sale is allowed

```python
# import libraries
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
import yfinance as yf
from scipy.optimize import minimize

# get input data

stocks = []
numbers = int(input("How many stocks to include in portfolio?"))
for i in range(numbers):
    i = input("name of the stock?")
    stocks.append(i)
start = input("start date : ")
end = input("end date : ")
num_portfolios = int(input("set the range : "))
risk_free_rate = float(input("risk-free rate: "))

# download stock data
data = yf.download(stocks, start=start, end=end)['Adj Close']

# daily return data
returns = data.pct_change()
```

```python
#function for calculating portfolio volatility, return

def portfolio_annualised_performance(weights, mean_returns, cov_matrix):
    returns = np.sum(mean_returns*weights ) *252
    std = np.sqrt(np.dot(weights.T, np.dot(cov_matrix, weights))) * np.sqrt(252)
    return std, returns

# function for generating random portfolio

def random_portfolios(num_portfolios, mean_returns, cov_matrix, risk_free_rate):
    results = np.zeros((3,num_portfolios))
    weights_record = []
    for i in range(num_portfolios):
        weights = np.random.uniform(-1,1,len(stocks))  # generating numbers between -1 and 1
        weights /= np.sum(weights)  # stnadardize
        weights_record.append(weights)
        portfolio_std_dev, portfolio_return = portfolio_annualised_performance(weights, mean_returns, cov_matrix)
        results[0,i] = portfolio_std_dev
        results[1,i] = portfolio_return
        results[2,i] = (portfolio_return - risk_free_rate) / portfolio_std_dev
    return results, weights_record

mean_returns = returns.mean()
cov_matrix = returns.cov()
```

```python
def display_simulated_ef_with_random(mean_returns, cov_matrix, num_portfolios, risk_free_rate):
    results, weights = random_portfolios(num_portfolios, mean_returns, cov_matrix, risk_free_rate)
    
    max_sharpe_idx = np.argmax(results[2])
    sdp, rp = results[0, max_sharpe_idx], results[1, max_sharpe_idx]
    max_sharpe_allocation = pd.DataFrame(weights[max_sharpe_idx], index=returns.columns, columns=['allocation'])
    max_sharpe_allocation.allocation = [round(i*100,2) for i in max_sharpe_allocation.allocation]
    max_sharpe_allocation = max_sharpe_allocation.T

    min_vol_idx = np.argmin(results[0])
    sdp_min, rp_min = results[0, min_vol_idx], results[1, min_vol_idx]
    min_vol_allocation = pd.DataFrame(weights[min_vol_idx], index=returns.columns, columns=['allocation'])
    min_vol_allocation.allocation = [round(i*100,2) for i in min_vol_allocation.allocation]
    min_vol_allocation = min_vol_allocation.T
    
    print("-"*80)
    print("Maximum Sharpe Ratio Portfolio Allocation\n")
    print("Annualised Return:", round(rp, 2))
    print("Annualised Volatility:", round(sdp, 2))
    print("Max sharpe ratio: ", round((rp-risk_free_rate)/sdp,2))

    print("\n")
    print(max_sharpe_allocation)
    print("-"*80)
    print("Minimum Volatility Portfolio Allocation\n")
    print("Annualised Return:", round(rp_min, 2))
    print("Annualised Volatility:", round(sdp_min, 2))
    print("\n")
    print(min_vol_allocation)
    return results, weights, sdp, rp, sdp_min, rp_min

results, weights, sdp, rp, sdp_min, rp_min = display_simulated_ef_with_random(mean_returns, cov_matrix, num_portfolios, risk_free_rate)
```

```python
# target portfolio calculation and plotting
print("-"*80)
target_return = float(input("Target return: "))
target_risk = float(input("Target risk: "))

weights_tangent = weights[np.argmax(results[2])]

weights_rf = 1 - weights_tangent.sum()

weights_target_return = (target_return - risk_free_rate) / (rp - risk_free_rate) * weights_tangent
weights_target_return_rf = 1 - weights_target_return.sum()

weights_target_risk = target_risk / sdp * weights_tangent
weights_target_risk_rf = 1 - weights_target_risk.sum()

print("\nOptimized portfolio for target return")
print("Weights for stocks: ", weights_target_return)
print("Weights for risk-free asset: ", weights_target_return_rf)

print("-"*80)
print("\nOptimized portfolio for target risk")
print("Weights for stocks: ", weights_target_risk)
print("Weights for risk-free asset: ", weights_target_risk_rf)
```

![Short_input.png](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/short_input.png?raw=True)

Following from the previous post, we shall continue with optimization of portfolio when short sale is allowed.  Above inputs are exactly same as previous post.

![short_max_sharpe.png](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/short_max_sharpe.png?raw=True)

As we can see, GOOG is negatively valued for allocation, indicating we should short GOOG and buy extra stocks. Sharpe ratio is also higher than the previous value which was below 3.00.

![short_min.png](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/short_min.png?raw=True)

Minimum volatility portfolio is about same as previous post.

![short_target.png](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/short_target.png?raw=True)

Targeted portfolio shows the difference where it shows more allocation of weights on the risk-free asset. This is logical since short sale is risky and need deep understanding on the market tendency and sensitivity.
