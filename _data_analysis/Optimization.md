---
title: "Optimization of portfolio 1"
collection: data_analysis
permalink: /data_analysis/yfinance_tutorial_3
excerpt: 'Optimization of portfolio when short sale is not allowed using python. And aslo allocation of riksy assets and risk-free assets in the targeting portfolio'
date: 2024-03-16
venue: 'data anlaysis post 3'
---

# Optimization of the portfolio when a short sale is not allowed.

```python
# import libraries 
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
import yfinance as yf
from scipy.optimize import minimize

# input functions for analysis
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

# daily return calculation
returns = data.pct_change()

# Annualised performance function
def portfolio_annualised_performance(weights, mean_returns, cov_matrix):
    returns = np.sum(mean_returns*weights ) *252
    std = np.sqrt(np.dot(weights.T, np.dot(cov_matrix, weights))) * np.sqrt(252)
    return std, returns

# Generating random portfolios for plotting
def random_portfolios(num_portfolios, mean_returns, cov_matrix, risk_free_rate):
    results = np.zeros((3,num_portfolios))
    weights_record = []
    for i in range(num_portfolios):
        weights = np.random.random(len(stocks))
        weights /= np.sum(weights)
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
#plotting function

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
    
    plt.figure(figsize=(10, 7))
    plt.scatter(results[0, :], results[1, :], c=results[2, :], cmap='YlGnBu', marker='o', s=10, alpha=0.3)
    plt.colorbar(label='Sharpe ratio')
    plt.scatter(sdp, rp, marker='*', color='r', s=500, label='Maximum Sharpe ratio')
    plt.scatter(sdp_min, rp_min, marker='*', color='g', s=500, label='Minimum volatility')
		
		# Add tangential line from risk-free rate to the efficient frontier
    m = (rp - risk_free_rate) / sdp  # slope of the line
    x = np.linspace(0, 0.3, 200)  # range of x values
    plt.plot(x, risk_free_rate + m * x, linestyle='-.', color='black', label='Capital Market Line')

    plt.title('Simulated Portfolio Optimization based on Efficient Frontier')
    plt.xlabel('annualised volatility')
    plt.ylabel('annualised returns')
    plt.legend(labelspacing=0.8)
    return results, weights, sdp, rp, sdp_min, rp_min
```

```python
# Target portfolio
results, weights, sdp, rp, sdp_min, rp_min = display_simulated_ef_with_random(mean_returns, cov_matrix, num_portfolios, risk_free_rate)

target_return = float(input("Target return: "))
target_risk = float(input("Target risk: "))

weights_tangent = weights[np.argmax(results[2])]

weights_rf = 1 - weights_tangent.sum()

weights_target_return = (target_return - risk_free_rate) / (rp - risk_free_rate) * weights_tangent
weights_target_return_rf = 1 - weights_target_return.sum()

weights_target_risk = target_risk / sdp * weights_tangent
weights_target_risk_rf = 1 - weights_target_risk.sum()
```

```python
#results
print("\nOptimized portfolio for target return")
print("Weights for stocks: ", weights_target_return)
print("Weights for risk-free asset: ", weights_target_return_rf)

print("\nOptimized portfolio for target risk")
print("Weights for stocks: ", weights_target_risk)
print("Weights for risk-free asset: ", weights_target_risk_rf)
```

```python
# indicating minimum volatility and maximum sharpe ratio portfolios on the plot

plt.figure(figsize=(10, 7))
plt.scatter(results[0, :], results[1, :], c=results[2, :], cmap='YlGnBu', marker='o', s=10, alpha=0.3)
plt.colorbar(label='Sharpe ratio')
plt.scatter(sdp, rp, marker='*', color='r', s=500, label='Maximum Sharpe ratio')
plt.scatter(sdp_min, rp_min, marker='*', color='g', s=500, label='Minimum volatility')

# capital market line plotting
m = (rp - risk_free_rate) / sdp
x = np.linspace(0, 0.3, 200)
plt.plot(x, risk_free_rate + m * x, linestyle='-.', color='black', label='Capital Market Line')

# Target portfolio pltting
plt.scatter(np.sqrt(np.dot(weights_target_return.T, np.dot(cov_matrix, weights_target_return))) * np.sqrt(252),
            np.sum(mean_returns*weights_target_return) *252,
            marker='X', color='b', s=200, label='Target Return')
plt.scatter(target_risk,
            np.sum(mean_returns*weights_target_risk) *252,
            marker='X', color='y', s=200, label='Target Risk')

plt.title('Simulated Portfolio Optimization based on Efficient Frontier')
plt.xlabel('annualised volatility')
plt.ylabel('annualised returns')
plt.legend(labelspacing=0.8)
```

![input_values.png](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/input_values.png?raw=True)

As we run the code, we can set how many stocks to be analyzed in the portfolio. For this case I chose 5 of the largest market capacity size stocks. The data collected is between 2023-01-01 to 2023-12-01. Range above indicates the number of random portfolios generated and risk-free rate is 2023 average 10-years treasury bond rate.

![max_sharpe_ratio_.png](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/max_sharpe_ratio_.png?raw=True)

Sharpe ratio indicates the how portfolio worked compare to its risk. As we can observe from the above, sharpe ratio is 2.98 ****which indicates that portfolio made up of AAPL,AMZN,GOOG,MSFT,NVDA was worth its risk in 2023.

![min_variance_.png](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/min_variance_.png?raw=True)

minimum volatility portfolio shows the smallest variance portfolio available. As you can see from above, compare to max sharpe ratio portfolio, its risk is 7% lesser but the return is 34% lesser. According to magnitude of risk aversion, minimum volatility can be considered as most risk aversing portfolio. 

![min_variancemax_sharpe.png](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/min_variancemax_sharpe.png?raw=True)

Plot above shows the CML tangential to efficient frontier with maximum sharpe ratio portfolio and minimum volatility portfolio.

![target_portfolio.png](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/target_portfolio.png?raw=True)

To fulfill one’s desire and risk tolerance, we can consider targeting portfolio with fixed expected return or risk. I have set target return as 0.6 and target risk as 0.2.  Then diversified the following portfolios with risk-free asset. 

![min_variance_max_sharpe_target.png](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/min_variance_max_sharpe_target.png?raw=True)

And finally plot above contains all the values we obtained throughout this analysis.