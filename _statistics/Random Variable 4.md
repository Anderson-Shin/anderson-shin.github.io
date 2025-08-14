---
title: "A Deep Dive into Random Variables (Part 4): Expectation and Variance"
collection: statistics
permalink: /statistics/random-variables-part4/
excerpt: "The fourth post in our series on random variables. We explore two of the most fundamental concepts for summarizing distributions: Expectation (the average outcome) and Variance (the measure of spread or risk)."
date: 2025-08-15
tags:
  - Expectation
  - Variance
  - Mean
  - Standard Deviation
  - Random Variable
  - Statistics
---

# **Expectation and Variance**

So far in our series, we've learned how to define [random variables](https://anderson-shin.github.io/statistics/random-variables-part1/) and describe their behavior using [discrete](https://anderson-shin.github.io/statistics/random-variables-part2/) and [continuous](https://anderson-shin.github.io/statistics/random-variables-part3/) distributions. But often, we need to summarize a distribution with just a few key numbers. What is the "average" outcome we should expect? And how "spread out" are the possible values?

Today, we'll answer these questions by exploring two of the most fundamental concepts in statistics: **Expectation** and **Variance**.

## **The Expected Value: What's the Average Outcome?**

The **expected value** (or **expectation**, or **mean**) of a random variable **X**, denoted as **$E[X]$**, is the long-run average value we would expect if we repeated the experiment many times. It's the distribution's center of mass.

### **Calculating $E[X]$ for Discrete Variables**

For a discrete random variable, the expected value is a weighted average of all possible values, where each value is weighted by its probability.

**Definition (Discrete):** $E[X] = \sum_{x} x \cdot p(x)$

* Example 1: Fair Die Roll  
A fair die can land on {1, 2, 3, 4, 5, 6}, each with $p(x)=1/6$.  
$E[X] = 1(\frac{1}{6}) + 2(\frac{1}{6}) + 3(\frac{1}{6}) + 4(\frac{1}{6}) + 5(\frac{1}{6}) + 6(\frac{1}{6}) = \frac{21}{6} = 3.5$.  
* Example 2: Expectation of a Bernoulli Random Variable  
Recall the Bernoulli distribution, which models a single trial with success probability p. The random variable X is 1 if it's a success and 0 if it's a failure.  
$E[X] = (1 \cdot P(X=1)) + (0 \cdot P(X=0)) = (1 \cdot p) + (0 \cdot (1-p)) = p$.  
The expected value is simply the probability of success.

### **Calculating $E[X]$ for Continuous Variables**

For a continuous random variable, we replace the sum with an integral.

**Definition (Continuous):** $E[X] = \int_{-\infty}^{\infty} x \cdot f(x) \,dx$

* Example 1: Uniform Distribution  
For a uniform distribution on the interval $(\alpha,\beta)$, the PDF is $f(x) = \frac{1}{\beta-\alpha}$.  
$E[X] = \int_{\alpha}^{\beta} x \cdot \frac{1}{\beta-\alpha}dx = \frac{1}{\beta-\alpha} \left[ \frac{x^2}{2} \right]_{\alpha}^{\beta} = \frac{\alpha+\beta}{2}$.
  
* Example 2: Expectation of an Exponential Random Variable  
An exponential random variable has a PDF of $f(x) = \lambda e^{-\lambda x}$ for $x \ge 0$.

  $E[X] = \int_{0}^{\infty} x(\lambda e^{-\lambda x})dx$.  
  Using integration by parts: 
  
  ($\int udv = uv - \int vdu$) with $u=x$ and $dv = \lambda e^{-\lambda x} dx$, we get:

  $\begin{align}E[X] &= \left[ -x e^{-\lambda x} \right]_{0}^{\infty} - \int_{0}^{\infty} (-e^{-\lambda x})dx \\ 
  &= (0-0) + \int_{0}^{\infty} e^{-\lambda x}dx\\ &= \left[ -\frac{1}{\lambda} e^{-\lambda x} \right]_{0}^{\infty}\\ &= (0) - (-\frac{1}{\lambda}) = \frac{1}{\lambda}\end{align}$

  If events occur at a rate $\lambda$, the expected time until the first event is $1/\lambda$.

## **Expectation of a Function of a Random Variable**

What if we're interested in the expectation of a function of X, like $E[X^2]$? A wonderfully convenient property, sometimes called the **Law of the Unconscious Statistician**, lets us calculate this directly.

**Proposition:** For any real-valued function g:

* Discrete: $E[g(X)] = \sum_{x} g(x)p(x)$  
* Continuous: $E[g(X)] = \int_{-\infty}^{\infty} g(x)f(x) \,dx$  
* Demonstration (Discrete): Let X have the PMF: $p(0)=0.2,p(1)=0.5,p(2)=0.3$. Let's find $E[X^2]$.  
  Here, $g(x)=x^2$.  
  $E[X^2] = (0^2 \cdot p(0)) + (1^2 \cdot p(1)) + (2^2 \cdot p(2))$  
  $E[X^2] = (0 \cdot 0.2) + (1 \cdot 0.5) + (4 \cdot 0.3) = 0 + 0.5 + 1.2 = 1.7$.  
* Demonstration (Continuous): Let X be uniform on (0, 1). Let's find $E[X^3]$.  
  Here, $g(x)=x^3$ and $f(x)=1$ for $x \in (0,1)$.  
  $E[X^3] = \int_{0}^{1} x^3 \cdot 1 \,dx = \left[ \frac{x^4}{4} \right]_{0}^{1} = \frac{1}{4}$.

## **Variance: Measuring the Spread**

While expectation tells us about the center of a distribution, **variance** tells us how spread out the values are.

Definition: The variance of X, denoted Var(X), is the expected value of the squared deviation from the mean.  
$Var(X) = E[(X - E[X])^2]$

A more convenient computational formula is derived from this definition:

**Computational Formula:** $Var(X) = E[X^2] - (E[X])^2$

The **standard deviation**, denoted $\sigma$, is simply the square root of the variance: $\sigma = \sqrt{Var(X)}$.

* Example 1: Variance of a Bernoulli Random Variable  
Let X be a Bernoulli variable with probability of success p. We know $E[X]=p$.  
First, we find $E[X^2]$. Since X can only be 0 or 1, $X^2$ is the same as X.  
$E[X^2] = E[X] = p$.  
Now, we use the formula:  
$Var(X) = E[X^2] - (E[X])^2 = p - p^2 = p(1-p)$.  
* Example 2: Variance of a Uniform Random Variable  
Let X be uniformly distributed on (0, 1). We know $E[X]=1/2$.  
First, we find $E[X^2]$ using the expectation of a function rule:  
$E[X^2] = \int_{0}^{1} x^2 \cdot 1 \,dx = \left[ \frac{x^3}{3} \right]_{0}^{1} = \frac{1}{3}$.  
Now, we use the variance formula:  
$Var(X) = E[X^2] - (E[X])^2 = \frac{1}{3} - \left(\frac{1}{2}\right)^2 = \frac{1}{3} - \frac{1}{4} = \frac{4-3}{12} = \frac{1}{12}$.

## **Intuitive Real-Life Applications**

How are these concepts used in the real world? Let's consider a couple of scenarios.

### **Expectation: Making Business Decisions**

Imagine a company is launching a new app. They estimate the following outcomes for the first-year profit:

* **Best-case scenario:** $5 million profit (20% probability)  
* **Most likely scenario:** $1 million profit (50% probability)  
* **Worst-case scenario:** -$2 million loss (30% probability)

Let X be the random variable for the first-year profit. The expected profit is:  
$E[X] = (5M \cdot 0.2) + (1M \cdot 0.5) + (-2M \cdot 0.3)$  
$E[X] = 1M + 0.5M - 0.6M = 0.9M$ 

The expected profit is **$900,000**. This single number helps the company make a strategic decision. Even though there's a risk of loss, the "average" outcome is a significant profit, which might justify the investment.

### **Variance: Assessing Investment Risk**

Suppose you have $1,000 to invest and are considering two options:

1. **Investment A (Low Variance):** A government bond. Its annual return is a random variable with an expected value of **$E[A] = 2\%$** and a very low standard deviation of $\sigma_A=0.5$.  
2. **Investment B (High Variance):** A tech startup stock. Its annual return is a random variable with a higher expected value of **$E[B] = 8\%$**, but also a much higher standard deviation of $\sigma_B=30$.  
* **Expectation** tells you that, on average, Investment B will yield a higher return.  
* **Variance (and Standard Deviation)** tells you about the **risk**.  
  * With Investment A, your return will almost certainly be very close to 2%. The low variance means the outcome is predictable and safe.  
  * With Investment B, your return could be anywhere from a huge gain to a devastating loss. The high variance means the outcome is unpredictable and risky.

An investor who needs stable, predictable returns (e.g., for retirement) would prefer the low-variance option. A younger investor who can tolerate risk for a chance at higher rewards might choose the high-variance option. Variance is the mathematical measure of that risk and uncertainty.

## **What's Next?**

We've now equipped ourselves with the tools to describe a distribution's center and spread. But what happens when two or more random variables appear together? How do they influence each other?

In our next post, we'll explore **Joint Distributions** and a measure of how two variables move together, **Covariance**, to understand the expectation and variance of sums of random variables.
