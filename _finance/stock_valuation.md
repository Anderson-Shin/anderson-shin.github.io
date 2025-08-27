---
title: "Stock valuation"
collection: finance
permalink: /finance/stock_valuation
excerpt: 'Basic concept of stock valuation'
date: 2023-11-08
venue: 'Finance post 2'
---
# Gordon Growth Model

- Stock Valuationation model is a mechanism that converts a set of forecasts of a series of company and economic variables into a forecast of market value for the company’s stock.
- Dividend Discount Model (DDM) or Gordon Model
    - Intrinsic Value $V_0$ : Present Value (PV) of all cash payments from the stock, including dividends and the proceeds from the ultimate sale of the stock, discounted at an appropriate risk-adjusted interest rate  $k$, called Market Capitalization Rate.
    
    $$
    V_0 = \frac{D_1}{1+k} + \frac{P_1}{1+k}
    $$
    
    where $D_1$ is the expected dividend at time 1, and $P_1$ is the expected stock price at time 1.
    
    - Market Price($P_0$): It can be different from the intrinsic value of a stock.
    - Suppose the Stock will always be sold for its intrinsic value; then the stock price should equal the present value of all expected future dividends into perpetuity.
        
        $$
        P_0 = \frac{D_1}{1+k} +\frac{D_2}{(1+k)^2}+\frac{D_3}{(1+k)^3}+....
        $$
        
    1. Constant Growth Model 
        
        To make DDM practical, we introduce a simplifying assumption that dividends are trending upward at a stable growth rate of $g$.
        
        $$
        D_0=\frac{D_1}{1+g}, D_1 = \frac{D_0(1+g)}{(k+g)}
        $$
        
        $$
        P_0 = \frac{D_1}{1+k}+\frac{D_1(1+g)}{(1+k)^2}+\frac{D_1(1+g)^2}{(1+k)^3}+... = \frac{D_1}{k-g}
        $$
        
        aforementioned is derived from $P_0= \frac{D_1}{1+k}\sum_{i=0}^n (\frac{1+g}{1+k})^i$
        
        ![Q1_stock.png](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/Q1_stock.png?raw=true))

        
        given that g = 0.02, k = 0.04, D1 = $5, P_3 = $120
        
        $$
        V_0=\frac{5}{1+0.04}+\frac{5(1+0.02)}{(1+0.04)^2}+\frac{5(1+0.02)^2+120}{(1+0.04)^3} =120.83
        $$
        
    2. Two-stage model
        
        Assume that a period of extraordinary growth rate of G (good or bad) will continue for a certain amount of time $(n)$, after which the growth rate will change to a level of $g$ at which it is expected to continue indefinitely.
        
        for time with growth rate G + time with growth rate g 
        
        $P_0 = [\frac{D_1}{1+k}+\frac{D_1(1+G)}{(1+k)^2}+\frac{D_1(1+G)^2}{(1+k)^3}+... = \frac{D_1(1+G)^{n-1}}{k-g}] + \frac{P_n}{(1+k)^n}$
        
        where $P_n =\frac{D_{n+1}}{1+k}+\frac{D_{n+1}(1+g)}{(1+k)^2}+\frac{D_{n+1}(1+g)^2}{(1+k)^3} + ...$
        
        여기서 다시 sum of finite series of geometric sequence를 사용하면;
        
        let $P_G$ is the market price for growth rate $G$ for time $n$, 
        let $P_g$ is the market price for growth rate $g$ for a time after $n$;
        
        $P_G = \frac{}{}$$\frac{\frac{D_1}{1+k}(1-(\frac{1+G}{1+k})^n)}{1-\frac{1+G}{1+k}}$ → $\frac{D_1}{k-G}[1-(\frac{1+G}{1+k})^n]$
        
        Since $D_{n+1}=D_1(1+G)^{n-1}(1+g)$;
        
        $P_g=[\frac{D_{n+1}}{k-g}(\frac{1}{1+k})^n]$→$[\frac{D_1(1+G)^{n-1}(1+g)}{k-g}(\frac{1}{1+k})^n]$
        
        Therefore, $P_0 = \frac{D_1}{k-G}[1-(\frac{1+G}{1+k})^n] + \frac{D_1(1+G)^{n-1}(1+g)}{k-g}(\frac{1}{1+k})^n$
        

![Q2_stock.png](https://github.com/Anderson-Shin/anderson-shin.github.io/blob/master/images/Q2_stock.png?raw=true)

a)
 stage1: Calculating the present value of dividends with an extraordinary growth rate of 15% (denoted as $P_1$)

$$
P_1=\frac{2(1.15)}{1.10}+\frac{2(1.15)^2}{1.10^2}+\frac{2(1.15)^3}{1.10^3}
$$

$$
=6.5622
$$

$$
P_2=\frac{1}{1.10^3}\cdot \lbrack \frac{2(1.15)^3(1.05)}{1.10}+\frac{2(1.15)^3(1.05)^2}{1.10^2}+...\rbrack
$$

$$
=\frac{1}{1.10^3}\cdot \lbrack 2(1.15)^3\times \frac{1.05}{1.10-1.05}\rbrack
$$

$$
=47.9915
$$

There for,

Value of the stock = $P_1+P_2$ =$54.5537

b) stage 1: $P_1=6.5622$

stage 2: Calculating the present value of dividends with growth rates decreasing linearly from 15% to 5% for 2 years, (denoted as $P'_2$)

$$
P'_2=\frac{1}{1.10^3}\times\lbrack \frac{2(1.15)^3(1.15-1\times\frac{0.15-0.05}{2})}{1.10})+\frac{2(1.15)^3(1.15-1\times\frac{0.15-0.05}{2})(1.15-2\times\frac{0.15-0.05}{2})}{1.10^2}\rbrack
$$

$$
=\frac{1}{1.10^3}\times(3.0418+2.9035)
$$

$$
=4.4667
$$

Stage 3: Calculating the present value of dividends with a stable growth rate of 5% (denoted as $P'_3$)

$$
P'_3=\frac{1}{1.10^5}(\frac{2(1.15)^3(1.10)(1.05)^2}{1.10}+\frac{2(1.15)^3(1.10)(1.05)^3}{1.10^2}+...)
$$

$$
=45.8101
$$

Therefore,

value of stock = $P_1+P'_2+P'_3$ = $56.8390