---
title: "Types of return and its normality"
collection: finance
permalink: /finance/return and its normality
excerpt: 'return and its normality'
date: 2024-01-29
venue: 'Finance post 5'
---

## simple interest

$FV_t$ = Future value at t, $R_t$ = simple interest at time t, $A$ = principal invested for n years.

$FV_1=A(1+R_1)$ → at t =1

$FV_2=A(1+R_2)$→ at t =2

$FV_n=A(1+R_1)$ → at t =n

If $R_1 = R_2 = ...R_n$, then, $FV_n=A(1+nR)$

## compound interest

$FV_t$ = Future value at t, $R_t$ = simple interest at time t, $A$ = principal invested for n years.

$FV_1=A(1+R_1)$ → at t =1

$FV_2=A(1+R_1)(1+R_2)$→ at t =2

$FV_n=A(1+R_1)(1+R_2)...(1+R_n)$ → at t =n

If $R_1 = R_2 = ...R_n=R$, then, $FV_n=A(1+R)^n$

The rule of 72; time taken to double the principal invested = 72/interest rate

effective interest rate ,$R_A$= rate that would produce the same result after 1 year without compounding; $(1+\frac{R}{m})^m=1+R_A$ → $R_A = (1+\frac{R}{m})^m -1$

return on asset 

$R(t_0,t_1)\frac {P_{t1}-P_{t_0}}{P_{t_0}}=...R_t=\frac{P_t-P_{t-1}}{P_{t-1}}$

Since $\frac{P_t-P_{t-1}}{P_{t-1}}=\frac{P_t}{P_{t-1}}-1; R_t+1 = \frac{P_t}{P_{t-1}}$

 $1+R_t(k)=\frac{P_t}{P_{t-1}}\times \frac{P_{t-1}}{P_{t-2}}\times \frac{P_{t-2}}{P_{t-3}}...\times \frac{P_{t-k}}{P_{t-k-1}}$

$R_t(k)= \frac{P_t}{P_{t-1}}\times \frac{P_{t-1}}{P_{t-2}}\times \frac{P_{t-2}}{P_{t-3}}...\times \frac{P_{t-k}}{P_{t-k-1}}-1$

## Continuously Compounded Interest

- Also called *force of interest*, *logarithmic* return or *log return*
- $r_t=ln(p_t)-ln(p_{t-1})=ln(1+R_t)$
    - divide the time period $(t-1,t)$ in to $m$ sub-periods
    - Continuous compounding $(m\to \infin)$
    - Initial value $A = P_{t-1}$
    - Interest rate $R = r_t \space\space for \space period(t-1,t)$
    - Future value $P_t = \lim_{m\to \infin}\space P_{t-1}(1+\frac{r_t}{m})^m\space=P_{t-1}\cdot e^{r_t}$
    - therefore; interest rate ; $r_t=ln(P_t)-ln(P_{t-1})$

- Remark
    - Multi-period simple returns are not additive
        - $R_t(2) = (1+R_t)(1+R_{t-1})-1 = R_t+R_{t-1}+R_tR{t-1}\approx R_t+R_{t-1}$
            
            since $R_tR_{t-1}$  is multiplication of very small number.
            
    - Multi-period log returns are additive
        - $r_t(2) = ln[1+R_t(2)]=ln[(1+R_t)(1+R_{t-1})]=ln(1+R_t)+ln(1+R_{t-1})=r_t+r_{t-1}$

## Adjustment for dividends

- Where Dt = dividends payment between t-1 and t

$$
R_t=\frac{P_t-P_{t-1}+D_t}{P_{t+1}}
$$

- Log return at time t;

$$
r_t=ln(P_t+D_t)-ln(P_{t-1})
$$

## Annualized Return

- $R_A=(1+R)^n-1$

## Log returns

- $r_A=r_1+r_2+...r_n\space for\space same\space r; r_A=12r$

A higher variance means higher in total risk exposure and standard deviation is also a **measure of total risk exposure of an asset.**

## Some Probability models of return

Model 1 : Normal Distribution

- **limitation**
1. It does not guarantee that $R_t\ge-1$
2. Sum of $i.d.d$ normal distributions follows Normal distribution but the product of $i.d.d$ Normal distributions does not follow Normal distribution.

**Model 2: Log-Normal Distribution**

$r_t=ln(1+R_t)\sim N(\mu,\sigma^2)$

- Simple return $R_t\ge-1$ is guaranteed
- The multi-period log return is available; $r_t(k)=r_t+r_{t-1}+....+r_{t-k+1}$
- tail of distribution of normal random variable is too thin
- Tail fatness is also an important issue in risk management

## Test for normality

- Skewness measures the symmetry of distribution
- $Skew(X) = \gamma=E(Z^3), where \space Z=\frac{X-\mu}{\sigma}$
- $Skew(X)> 0\leftrightarrow$ **long right tail**
- $Skew(X)< 0\leftrightarrow$ **long left tail**
- Kurtosis measure tail thickness or peakedness
- $Kurt(X)=\delta=E(Z^4), where \space Z=\frac{X-\mu}{\sigma}$
- **Large $Kurt(X)\rightarrow$ Fat tails**
- Small $Kurt(X)\rightarrow$ **Thin tails**

## Jarque-Bera Normality Test

- Suppose $r_1, r_2,…,r_T$ are log returns on an asset
- Hypothesis;
$H_0 :r_t$ follows i.i.d. Normal
$H_1: H_0$ is not True
- Test statistic

$JB=T[\frac{\hat\gamma^2}{6}+\frac{(\hat\delta^-3)^2}{24}]$,

where

$\hat\gamma = \frac{1}{T-1}\sum_{t=1}^T(\frac{r_t-\bar{r}}{s})^3$

$\hat{\sigma}=\frac{1}{T-1}\sum_{t=1}^T(\frac{r_t-\bar{r}}{s})^4$

$s=\sqrt{\frac{1}{T-1}\sum_{t=1}^T(r_t-\bar{r})^2}$

$\bar{r}=\frac{1}{T}\sum_{t=1}^Tr_t$
- Asymptotic; $T\to \infty;\space JB\sim\chi^2_{2}$

- Under $H_0$ if $T$ is large, $JB$ follows chi-squared distribution with 2 degrees of freedom.
- Decision Rule
    - At the 5% significance level, reject $H_0\space if \space JP>\chi^2_{0.05,df=2}=5.991$
