---
title: "A Deep Dive into Random Variables (Part 3): Continuous Distributions"
collection: statistics
permalink: /statistics/random-variables-part3/
excerpt: "In the third post of our series, we explore the world of continuous random variables. We'll introduce the Probability Density Function (PDF) and dive into key distributions like the Uniform, Exponential, Gamma, and the famous Normal distribution."
date: 2025-08-14
tags:
  - Random Variable
  - Continuous Variable
  - PDF
  - Uniform Distribution
  - Exponential Distribution
  - Gamma Distribution
  - Normal Distribution
  - Statistics
---

# **Understanding Continuous Random Variables**


## **A Deep Dive into Random Variables, Post 3: Exploring Continuous Random Variables**

In our first two posts, we built a [solid foundation](https://www.google.com/search?q=https://anderson-shin.github.io/statistics/random-variables-part1/) and explored the world of [discrete random variables](https://anderson-shin.github.io/statistics/random-variables-part2/). Now, we pivot from counting to measuring. Welcome to the domain of **continuous random variables**\!

What is the exact temperature tomorrow? How long will a battery last? What is a person's precise height? These values can fall anywhere within a continuous range.

## **The Probability Density Function (PDF)**

For a continuous random variable **X**, we cannot speak of the probability of X taking on a specific value, because that probability is zero. Instead, we use a **Probability Density Function (PDF)**, denoted as $f(x)$, to define the probability that X falls within any given set (or interval) of real numbers.

Formal Definition: A random variable X is continuous if there exists a non-negative function f(x), such that for any set of real numbers B:  
$P{X∈B}=∫_B​f(x)dx$

This means the probability is found by integrating the density function over the set B. For an interval $[a,b]$, this becomes the familiar $P(a≤X≤b)=∫_a^b​f(x)dx$. Since X must take on some value, the total area under the PDF must be 1: $∫_{−∞}^∞​f(x)dx=1$.

### **Connecting the PDF and CDF**

The PDF and the Cumulative Distribution Function (CDF) are directly linked. The CDF, F(a), is the integral of the PDF from negative infinity up to *a*.

$F(a)=P(X≤a)=∫_{−∞}^a​f(x)dx$

Conversely, the PDF is the derivative of the CDF:  
$f(a)=\frac{d}{da}​F(a)$

### **An Intuitive View of Density**

So, what does the value $f(a)$ at a specific point *a* actually mean if it's not a probability? It represents the *likelihood* of the random variable being near *a*. For a very small interval of length ϵ around *a*, the probability is approximately the area of a rectangle with height $f(a)$ and width $ϵ$.

$P(a−\frac{ϵ}{2}​≤X≤a+\frac{ϵ}{2}​)≈ϵf(a)$

A higher f(a) means the random variable is more likely to be found in a small interval around *a*.

*The probability of X falling in a tiny interval around 'a' is approximately the area of the rectangle:* $ϵ⋅f(a)$

## **The "Big Four" of Continuous Distributions**

Let's meet four of the most important continuous distributions, this time with a bit more detail.

### **1\. The Uniform Distribution**

This distribution models a situation where all outcomes in a fixed range are equally likely.

* **The Idea:** Any value in the range from **α** to **β** has an equal chance of occurring.  
* **The PDF:** $f(x) = \begin{cases} 
1 & \text{if } 0 < x < 1 \\
0, & \text{otherwise}
\end{cases}$
* **The CDF:** $F(a) = \begin{cases} 
0 & \text{if } a \le \alpha \\
\frac{a-\alpha}{\beta-\alpha} & \text{if } \alpha < a < \beta \\
1 & \text{if } a \ge \beta
\end{cases}$ 
* Example: If X is uniformly distributed over (0, 10), what is $P(1<X<6)$?  
  $P(1<X<6)=∫_1^6​\frac{1}{10}​dx​=\frac{1}{2}​$.

### **2\. The Exponential Distribution**

This distribution models the time until a single event occurs in a Poisson process.

* **The Idea:** Waiting time for an event happening at a constant average rate.  
* **The Parameter:** **λ (lambda)**, the rate parameter (e.g., failures per hour).  
* **The PDF:** $f(x) = \begin{cases} 
\lambda e^{-\lambda x} & \text{if } \quad x \ge 0 \\
0, & \text{if} \quad x\le0
\end{cases}$​  
* **The CDF:** $F(a)=∫_0^a​λe^{−λx}dx=1−e^{−λa}$ for $a≥0$.  
* Example: A component's lifetime X (in hours) is exponential with λ=0.001. The probability it lasts over 1200 hours is:  
  $P(X>1200)=1−F(1200)=1−(1−e−^{0.001⋅1200})=e^{−1.2}≈0.3012$.

### **3\. The Gamma Distribution**

A generalization of the Exponential, this models the waiting time for the **α-th** event.

* **The Idea:** Waiting time for a specified number of events.  
* **The Parameters:**  
  * **α (alpha):** The shape parameter (number of events).  
  * **λ (lambda):** The rate parameter.  
* **The PDF:** $f(x) = \begin{cases} 
\frac{\lambda e^{-\lambda x} (\lambda x)^{\alpha-1}}{\Gamma(\alpha)}, & \text{if } x \ge 0 \\
0, & \text{otherwise}
\end{cases}$ 
  * Where $Γ(α)=∫_0^∞​e^{−y}y^{α−1}dy$ is the Gamma function. For integer $α$, $Γ(α)=(α−1)!$.  
* **Relationship to Exponential:** The Exponential is a Gamma distribution with α=1.

### **4\. The Normal Distribution**

The "bell curve," central to statistics due to its prevalence in nature and its role in the Central Limit Theorem.

* **The Idea:** Data clustering symmetrically around a mean.  
* **The Parameters:**  
  * **μ (mu):** The mean.  
  * **σ² (sigma-squared):** The variance.  
* **The PDF:** $f(x)=\frac{1}{\sqrt{2π}\sigma}\cdot e^{\frac{(x-\mu)^2}{2\sigma^2}}$
* **The Standard Normal:** A crucial result is that if X is normal with parameters μ and $σ²$, then the variable $Z=\frac{X−μ}{\sigma}$​ is a **standard normal** random variable with mean 0 and variance 1\. This transformation allows us to use a single standard normal table for probability calculations for *any* normal distribution.  
* **Example:** Test scores (X) are normal with $μ=100,σ=15$. Find $P(X>130)$.  
  * **Step 1: Standardize.** $P(X>130)=P(\frac{X−100}{15}​>\frac{130-100}{15}​)=P(Z>2)$.  
  * **Step 2: Use Z-table or software.** The probability P(Z\>2) is approximately **0.0228**.

## Distribution visualzation
To truly understand how these distributions behave, it's best to see them in action. The interactive tool below allows you to adjust the parameters for each distribution and watch how the Probability Density Function (PDF) and Cumulative Distribution Function (CDF) change in real-time.

<iframe src="/assets/continuous_distribution_visualization.html" 
        width="100%" 
        height="750" 
        style="border:1px solid #ccc; border-radius: 8px;" 
        title="Interactive Continous Distribution Visualizer">
</iframe>

## **What's Next?**

Now that we have a more formal understanding of continuous distributions, we are ready to explore their properties. In the next post, we'll dive into two of the most important concepts in probability: **Expectation and Variance**. See you there\!