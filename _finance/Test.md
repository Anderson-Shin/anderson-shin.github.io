---
title: "Coherent & Spectral Risk Management"
collection: finance
permalink: /finance/market_risk_estimation
excerpt: 'Deep dive into VaR, Expected Shortfall, and Spectral Risk Measures (SRM)'
venue: "Finance post"
date: 2025-05-19
location: ""
---

# Estimating Market Risk Measures

In today’s financial environment, managing market risk is not optional — it’s essential.  
This post introduces key tools to measure market risk, starting from basic return data to advanced concepts like Expected Shortfall (ES) and Spectral Risk Measure (SRM).  
We'll not only cover formulas, but also explain **why these tools matter** and how they're used in practice.

---

## Key Topics

- **Preliminary data issues**: How to deal with data in profit/loss form, rate-of-return form, and so on.
- **Basic methods of VaR estimation**: Estimating simple VaRs depending on distributional assumptions.
- **How to estimate coherent risk measures**: Such as Expected Shortfall and Spectral Risk Measures.
- **How to gauge the precision** of estimators using standard errors.
- **Overview**: How all methods fit together in practical risk management.

---

## 1.1 DATA

### Profit/Loss Form

$\frac{P}{L_t}=P_t+D_t-P_{t-1}$

Where:

- $P_t$: Asset price at time $t$  
- $D_t$: Cashflow (e.g., dividends)  
- $P_{t-1}$: Previous price  
→ Represents gain/loss from holding the asset over one period.

---

### Present Value of P/L

$$
PV\left(\frac{P}{L_t}\right)=\frac{P_t+D_t}{1+d}-P_{t-1}
$$

Where $d$ is the discount rate.

---

### Forward Value of P/L

$$
FV\left(\frac{P}{L_t}\right)=(P_t+D_t)-(1+d)P_{t-1}
$$

---

### Loss Perspective (for risk measurement)

$$
\frac{L}{P_t}=-\frac{P}{L_t}
$$

---

## Arithmetic vs Geometric Return

### Arithmetic Return

- Assumes $D_t$ is reinvested and doesn’t earn its own return.
- Not ideal for long-term horizons due to compounding bias.

### Geometric Return

- Assumes continuous reinvestment.
- More realistic in modeling as it reflects compounding and volatility.

**Key differences**:

1. Geometric return considers compounding.  
2. Reflects volatility more accurately.

---

## Estimating Parametric VaR

### Based on Profit/Loss Forms

$VaR(P/L)=-\mu_{P/L}+\sigma_{P/L}Z_\alpha$  
$VaR(L/P)=\mu_{L/P}+\sigma_{L/P}Z_\alpha$

---

### Arithmetic Return

$$
r^*=\mu_A-\sigma_AZ_\alpha
$$

$$
r_t=\frac{P_t-P_{t-1}}{P_{t-1}}=\frac{L}{P_t}
$$

$$
\alpha VaR=-(\mu_A-\sigma_AZ_\alpha)P_{t-1}
$$

---

### Geometric Return

$$
r_t = \ln\left(\frac{P_t}{P_{t-1}}\right)
$$

Under geometric return assumptions, returns are compounded continuously. The log return that corresponds to the quantile threshold is:

$$
R^* = \mu_R - \sigma_R Z_\alpha
$$

Using this, the projected asset price becomes:

$$
P^* = P_{t-1} \cdot e^{R^*} = P_{t-1} \cdot e^{\mu_R - \sigma_R Z_\alpha}
$$

And the Value-at-Risk is derived as:

$$
\alpha VaR = P_{t-1} \left(1 - e^{\mu_R - \sigma_R Z_\alpha}\right)
$$

---

## Coherent Risk Measures: Beyond VaR

While Value-at-Risk (VaR) is widely used, it has critical limitations — especially when it comes to extreme tail events.  
**Expected Shortfall (ES)** addresses some of these, and **Spectral Risk Measures (SRM)** go even further.  
Let's break down what these measures mean and why they're essential for a robust risk framework.

### Expected Shortfall (ES)

$$
ES_\alpha=\mathbb{E}[L\mid L\geq VaR_\alpha]=\frac{1}{1-\alpha}\int_\alpha^1VaR_p\,dp
$$

→ Measures the average of losses worse than the VaR threshold.

---

### Spectral Risk Measure (SRM)

#### Why Spectral Risk Measure?

Traditional risk measures like VaR or ES treat all extreme losses **beyond a threshold** as equally important.  
But in practice:

- Not all tail losses are equally severe.
- Institutions and investors have different **risk attitudes**.

> **Spectral Risk Measure (SRM)** incorporates this idea by applying **weights** to tail losses that reflect the user’s risk aversion.

It allows more **flexible**, **customizable**, and **realistic** risk measurement.

#### Visualizing SRM Weight Function

Below is how SRM assigns weights ($\phi_{\gamma(p)}$) to losses based on their severity, controlled by the **risk aversion parameter** $\gamma$:

![Spectral Risk Weights](https://raw.githubusercontent.com/Anderson-Shin/anderson-shin.github.io/master/images/spectral_risk_weights.png)

This diagram illustrates how the weight function $\phi_\gamma(p)$ changes with different levels of risk aversion:

- **Small $\gamma$ (e.g., 0.5)**: The curve rises sharply near $p=1$, putting **more emphasis on extreme tail losses**.
- **Large $\gamma$ (e.g., 2)**: The curve increases more gently, assigning **more balanced weights** across the distribution.

This visual helps compare how **sensitive** SRM becomes to extreme quantiles based on investor risk preferences.

#### Spectral Risk Measure Formula

$$
M_\phi=\int_0^1\phi(p)q_p\,dp
$$

Where:

- $q_p$: loss at quantile $p$  
- $\phi(p)$: weight function over quantiles (increasing for more risk-averse agents)

> **Interpretation**:  
> “Take each possible loss, multiply it by how much you care about it, and average.”

If $\phi(p)$ is **constant** → becomes Expected Shortfall  
If $\phi(p)$ is **increasing** → emphasizes more extreme losses

#### Weight Function Example

$$
\phi_\gamma(p)=\frac{e^{-(1-p)/\gamma}}{\gamma(1-e^{-1/\gamma})}
$$

---

### Summary Table

| Category         | Expected Shortfall (ES) | Spectral Risk Measure (SRM)          |
| ---------------- | ----------------------- | ------------------------------------ |
| Weight Function  | Fixed $1/(1-\alpha)$    | Custom-defined $\phi(p)$             |
| Tail Sensitivity | Uniform                 | Increases as $p \to 1$               |
| Computation      | Simple average          | Weighted average                     |
| Flexibility      | Limited                 | High — customizable by risk attitude |
| Example          | Mean of 5% worst losses | Weighted mean with risk focus        |

> ✅ **Use Case Example**:  
> Banks: use ES to comply with Basel capital requirements  
> Insurers: prefer SRM to better capture sensitivity to tail risks

---

## Halving Error (Estimation Accuracy)

| Tail Slices | Estimated SRM | Halving Error |
| ----------- | ------------- | ------------- |
| 100         | 1.5853        | 0.2114        |
| 200         | 1.7074        | 0.1221        |
| 400         | 1.7751        | 0.0678        |
| 1600        | 1.8317        | 0.0197        |
| 51200       | 1.8529        | 0.0008        |

→ More tail slices improve accuracy by reducing numerical integration error.

---

## Standard Error Estimation

To ensure the reliability of any risk estimate — whether it's VaR, ES, or SRM — it's important to consider how precise that estimate is.  
This is where **standard errors** and **sampling variability** come into play.

$$
Var(q)\approx\frac{p(1-p)}{n[f(q)]^2}
$$

Where:

- $n$: sample size  
- $f(q)$: density function at quantile $q=VaR_\alpha$

Standard error tends to:

- **Decrease** as sample size $n$ increases  
- **Increase** as we move toward tail quantiles (extreme $q$ values)

---

📌 **Final Takeaway**:  
Great risk measurement is not just about math — it’s about mindset.  
By using tools like ES and SRM, you're not just quantifying losses — you're aligning them with how your organization actually thinks about risk.
