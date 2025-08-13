---
title: "A Deep Dive into Random Variables (Part 2): Discrete distributions"
collection: statistics
permalink: /statistics/random-variables-part2/
excerpt: "The first post in a series on random variables. We lay the groundwork by defining what a random variable is, distinguishing between discrete and continuous types, and introducing the all-powerful Cumulative Distribution Function (CDF)."
date: 2025-08-13
tags:
  - Random Variable
  - Discrete Variable
  - Continuous Variable
  - CDF
  - Statistics
---

# **Understanding Discrete Random Variables**

Welcome back\! In our [A Deep Dive into Random Variables (Part 1): The Foundation](https://anderson-shin.github.io/statistics/random-variables-part1/), we established that a random variable is a rule that assigns a number to an outcome. We also introduced the two main types: discrete and continuous. Today, we're zeroing in on **discrete random variables**—the ones that involve counting.

How many times will a coin land on heads? How many defective items are in a batch? How many customers will arrive in the next hour? These are all questions answered by discrete random variables. Let's explore the tools we use to describe them and meet the "celebrities" of the discrete distribution world.

## **The Probability Mass Function (PMF)**

While the Cumulative Distribution Function (CDF) we met last time works for all random variables, discrete variables have a more direct tool called the **Probability Mass Function (PMF)**.

The PMF, denoted as p(a), gives us the probability that the random variable **X** is *exactly* equal to some value *a*.

**Definition:** $p(a)=P(X=a)$

Think of "mass" as probability. The PMF tells you exactly how much probability mass is located at each specific point.

The PMF has two simple but crucial properties:

1. The probability at any specific value xi​ must be positive: $p(x_i​)>0$.  
2. The sum of the probabilities over all possible values must equal 1\. If $X$ can take values $x1​,x2​,...,$ then:  
$∑_{i=1}^∞​p(xi​)=1$

Example: Rolling a Fair Die  
If $X$ is the outcome of a fair die roll, its PMF is straightforward:  
$p(1)=\frac{1}{6}, p(2)=\frac{1}{6}, ..., p(6)\frac{1}{6}$.  
And $p(x)=0$ for any other value of $x$. Simple\!

## **The "Big Four" of Discrete Distributions**

While there are many discrete distributions, four of them appear so frequently that they deserve a special introduction.

### **1\. The Bernoulli Distribution: The Building Block**

The Bernoulli distribution is the simplest of all. It models a single trial with only two possible outcomes: success or failure.

* **The Idea:** One trial, two outcomes. Think of it as a single coin flip.  
* **The Parameters:** A single probability, **p**, which represents the probability of success.  
* **The PMF:** Let's say success is "1" and failure is "0".  
  * $p(1)=p$  
  * $p(0)=1−p$  
* **Example:** A single free-throw attempt. If a basketball player has a 75% chance of making a shot, that's a Bernoulli trial with p=0.75.

### **2\. The Binomial Distribution: Counting Successes**

What if you have more than one Bernoulli trial? The Binomial distribution describes the number of successes in a *fixed number* of independent trials.

* **The Idea:** Performing *n* independent Bernoulli trials and counting the number of successes.  
* **The Parameters:**  
  * **n:** The total number of trials.  
  * **p:** The probability of success on any given trial.  
* The PMF: The probability of getting exactly k successes in n trials is:  
  $P(X=k)={n \choose k}p^k(1−p)^{n−k}$  
  * The $n \choose k$​ part (read "n choose k") counts how many different ways you can arrange the *k* successes among the *n* trials.  
  * The $p^k(1−p)^{n−k}$ part is the probability of any *one* of those specific arrangements occurring.  
* **Example:** You flip a fair coin 10 times (n=10,p=0.5). What's the probability of getting exactly 7 heads $(k=7)$? The Binomial distribution can tell you\!

### **3\. The Geometric Distribution: Waiting for Success**

Instead of counting successes in a fixed number of trials, what if you keep going until you get your *first* success? That's the Geometric distribution.

* **The Idea:** How many trials does it take to get the first success?  
* **The Parameters:** A single probability, **p**, the probability of success on any trial.  
* The PMF: The probability that the first success occurs on the n-th trial is:  
  $P(X=n)=p(1−p)^{n−1}$  
  * This is intuitive: It's the probability of having n−1 failures in a row, followed by one success.  
* **Example:** You're rolling a die until you get a 6\. The probability of success is $p=1/6$. The probability that it takes you exactly 3 rolls is $P(X=3)=(\frac{5}{6})^2 \times (\frac{1}{6})$.

### **4\. The Poisson Distribution: Events in an Interval**

The Poisson distribution is a bit different. It describes the number of times an event occurs over a fixed interval of time or space, given you know the average rate at which it occurs.

* **The Idea:** Counting events over an interval when the average rate is constant.  
* **The Parameters:** **λ (lambda)**, the average number of events in that interval.  
* The PMF: The probability of observing exactly k events is:  
  $P(X=k)=\frac{e^{-\lambda} \cdot \lambda^k}{k!}$  
* **Example:** A call center receives an average of 10 calls per hour (λ=10). The Poisson distribution can tell you the probability of receiving exactly 15 calls in a given hour. It's also famously used to approximate the Binomial distribution when *n* is very large and *p* is very small.

## Distribution visualzation
I have coded using html to render the simple distribution visualzation and their skewness changes according to probability and number of trials

<iframe src="/assets/descrete_distribution_visualization.html" 
        width="100%" 
        height="750" 
        style="border:1px solid #ccc; border-radius: 8px;" 
        title="Interactive Discrete Distribution Visualizer">
</iframe>

## **What's Next?**

We've now met the key players in the world of discrete random variables. We understand how the PMF gives us the probability of specific outcomes for these "counting" variables.

In our next post, we'll shift our focus to the other side of the coin: **continuous random variables**. We'll introduce their version of the PMF—the **Probability Density Function (PDF)**—and explore essential distributions like the Uniform, Exponential, and the king of them all, the Normal distribution. See you there\!