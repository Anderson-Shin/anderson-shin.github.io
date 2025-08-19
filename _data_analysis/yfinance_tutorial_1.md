---
title: "Basic functions of yfinance library "
collection: data_analysis
permalink: /data_analysis/yfinance_tutorial_1
excerpt: 'Post about storng python API library to handle data of stocks in yahoo finance'
date: 2024-03-01
venue: 'data anlaysis post 1'
---
# Yfinance API tutorial 1

## What is yfinance api?

- The `yfinance` library is a popular open-source Python library that provides a simple and convenient way to download historical market data from Yahoo Finance. It allows you to retrieve historical stock price data, dividend data, and split data for stocks, ETFs (Exchange-Traded Funds), mutual funds, and indices.`pandas` and `matplotlib` are commonly used alongside `yfinance` to manipulate and visualize the downloaded historical market data.

```python
# import yfinance api
import yfinance as yf

#use Ticker fucntion to selcet the intersted stock
msft = yf.Ticker("MSFT")

# get all stock info
msft.info
```

![info.png](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/data_img/info.png?raw=true)

```python
# get historical market data in a given period e.g. 1mo, 1d,...etc
hist = msft.history(period="1mo")
hist.head()
```

![history.png](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/data_img/history.png?raw=true)

```python
# show meta information about the history (requires history() to be called first)
msft.history_metadata
```

![metadata.png](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/data_img/metadata.png?raw=true)

```python
# show actions (dividends, splits, capital gains)
msft.actions
msft.dividends
msft.splits
```

![dividends.png](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/data_img/dividends.png?raw=true)

![splits.png](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/data_img/splits.png?raw=true)

```python
# show share count
msft.get_shares_full(start="2022-01-01", end=None)
```

![share count.png](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/data_img/share%20count.png?raw=true)
