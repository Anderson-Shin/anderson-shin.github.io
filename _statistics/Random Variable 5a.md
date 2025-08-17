---
title: "A Deep Dive into Random Variables (Part 5a): Joint Distributions and Covariance"
collection: statistics
permalink: /statistics/random-variables-part5a/
excerpt: "The fifth post in our series on random variables. We move beyond single variables to explore how multiple variables interact, covering key concepts like Joint Distributions, Independence, and Covariance."
date: 2025-08-17
use_math: true
tags:
  - Expectation
  - Variance
  - Joint Distribution
  - Covariance
  - Independence
  - Random Variable
  - Statistics
---

In our random variable series, we discussed the expectation and variance of discrete and continuous variables in the last post. Today, we will go further with these concepts to discuss joint probability and covariance. From this post onwards, the topics become a bit more tricky, so please follow along closely! Let's begin by defining some propositions, continuing from our last post.

## Proposition 1.1

1. If $X$ is a discrete random variable with probability mass function $p(x)$, then for any real-valued function $g$:
    
    $$ E[g(x)]=\sum_{x:p(x)>0}g(x)p(x) $$
    
2. If $X$ is a continuous random variable with probability density function $f(x)$, then for any real-valued function g,

$$ E[g(X)]=\int_{-∞}^{∞} g(x)f(x)dx $$

## Corollary 1.1

If $a$ and $b$ are constants, then

**Proof** In the discrete case,

$$ \begin{aligned}E[aX+b] &= \sum_{x:p(x)>0} (aX+b)p(x)\\
&= a \sum_{x:p(x)>0}xp(x)+b \sum_{x:p(x)>0} p(x)\\
&= aE[X]+b
\end{aligned} $$

In the continuous case,

$$ \begin{aligned}E[aX+b] &= \int_{-∞}^{∞}(aX+b)f(x)dx\\
&= a \int_{-∞}^{∞}xf(x)dx+b \int_{-∞}^{∞} f(x)dx\\
&= aE[X]+b
\end{aligned} $$

***

**Intuitive Example: A Simple Game**

Imagine a game where you roll a standard six-sided die. Let the random variable `X` be the outcome of the roll. The possible values for `X` are {1, 2, 3, 4, 5, 6}, and each has a probability of 1/6.

The expected value of the roll is:
`E[X] = 1*(1/6) + 2*(1/6) + 3*(1/6) + 4*(1/6) + 5*(1/6) + 6*(1/6) = 3.5`

Now, let's say the game pays you based on the roll. The payout is calculated as `g(X) = 2X + 1`. This means you get $2 times the die roll plus an extra $1.

What is your expected payout, `E[2X + 1]`?

Using the corollary, we don't need to recalculate the whole sum. We can simply do:
`E[2X + 1] = 2 * E[X] + 1 = 2 * 3.5 + 1 = 7 + 1 = $8`.

This means if you played the game many times, you'd expect to win an average of $8 per game.

***

## Jointly Distributed Random Variables

Okay, we have discussed expectations and variances of discrete and continuous variables so far. Now, we are going to talk about something more frequently encountered in real life. We are often interested in probability statements concerning two or more random variables.

To deal with such probabilities, we define, for any two random variables $X$ and $Y$, the *joint cumulative probability distribution function* of $X$ and $Y$  by

$$ F(a,b)=P{X\leq a, Y\leq b}, \quad -∞<a, b<∞ $$

The distribution of $X$ can be obtained from the joint distribution of $X$  and $Y$ as follows:

$$ \begin{aligned} F_X(a) &= P{X\leq a}\
&= P{X\leq a, Y < ∞ }
\ &=F(a,∞)
\end{aligned} $$

In the case where $X$ and $Y$ are both discrete random variables, it is convenient to define the *joint probability mass function* of $X,Y$ by

$$ p(x,y)=P{X=x,Y=y} $$

The probability mass function of $X$ may be obtained from $p(x,y)$ by

$$ p_X(x)=\sum_{y:p(x,y)>0}p(x,y) $$

Similarly,

$$ p_Y(y)=\sum_{x:p(x,y)>0}p(x,y) $$

***

**Intuitive Example: Rolling Two Dice**

Let's say we roll two dice simultaneously: a red die (variable `X`) and a blue die (variable `Y`). Both are fair six-sided dice.

*   **Joint Probability:** What is the probability that the red die shows a 2 (`X=2`) AND the blue die shows a 5 (`Y=5`)?
    Since there are 36 possible outcomes (6 faces on the red die x 6 faces on the blue die), and each is equally likely, the probability of this specific combination is `p(2, 5) = 1/36`. This is the **joint probability**.

*   **Marginal Probability:** What is the probability that the red die shows a 2 (`X=2`), regardless of what the blue die shows?
    The blue die could be a 1, 2, 3, 4, 5, or 6. So we need to sum up all the possibilities where `X=2`:
    `p_X(2) = p(2,1) + p(2,2) + p(2,3) + p(2,4) + p(2,5) + p(2,6)`
    `p_X(2) = 1/36 + 1/36 + 1/36 + 1/36 + 1/36 + 1/36 = 6/36 = 1/6`.
    This is the **marginal probability** of `X=2`. It's just the simple probability of a single die roll, which makes perfect sense.

***

We say that $X$ and $Y$ are *jointly continuous* if there exists a function $f(x,y)$, defined for all real $x$ and $y$, having the property that for all sets $A$ and $B$ of real numbers

$$ P{X \in A, Y \in B} = \int_B\int_Af(x,y)dxdy $$

The function $f(x,y)$ is called *joint probability density function* of $X$ and $Y$. The probability density of $X$ (marginal probability of $X$) can be obtained from a $f(x,y)$ by the following reasoning:

$$ \begin{aligned}
 P{X \in A } &= P{X \in A, \quad Y \in (-∞,∞)}\\
&= \int_{-∞}^{∞}\int_Af(x,y)dxdy\\
&=\int_Af_X(x)dx
\end{aligned} $$

where

$$ f_X(x)=\int_{-∞}^{∞}f(x,y)dy $$

Similarly,

$$ f_Y(y)=\int_{-∞}^{∞}f(x,y)dx $$

If we take a look at Proposition 1, we can derive the expectation of a function of jointly distributed random variables.

$$ E[g(X,Y)] =
\begin{cases}
  \sum_y\sum_xg(x,y)p(x,y) & \text{in the discrete case}\\
  \int_{-∞}^{∞}\int_{-∞}^{∞} g(x,y)f(x,y)dxdy & \text{in the continuous case}
\end{cases}
$$

For example, if $g(X,Y)= X+Y$, then in the continuous case,

$$ \begin{aligned}
  E[X + Y] &= \int_{-∞}^{∞} \int_{-∞}^{∞} (x + y) f(x, y) \; dx \; dy \\
  &= \int_{-∞}^{∞} \int_{-∞}^{∞} xf(x, y) \; dx \; dy + \int_{-∞}^{∞} \int_{-∞}^{∞} yf(x, y) \; dx \; dy \\
  &= E[X] + E[Y]
\end{aligned} $$

## Independent Random Variable

The random variables $X$  and $Y$ are said to be *independent* if, for all $a,b$

$$ P{X\le a, Y\leq b}=P{X\leq a}P{Y\leq b} $$

In other words, $X$  and $Y$ are independent if, for all $a$ and $b$, the events $E_a={X\leq a}$ and $F_b={Y\leq b}$ are independent.

In terms of the joint distribution function $F$ of $X$ and $Y$, we have that $X$ and $Y$ are independent if

$$ F(a,b)=F_X(a)F_Y(b) \quad \text{for all a, b} $$

when $X$ and $Y$ are discrete, the condition of independence reduces to 

$$ p(x,y)=P_X(x)P_Y(y) \tag{a} $$

while if $X$  and $Y$ are jointly continuous, independence reduces to 

$$ f(x,y)=f_X(x)f_Y(y) \tag{b} $$

To prove this statement, consider first the discrete version, and suppose that the joint probability mass function $p(x,y)$ satisfies Equation (a).

$$ \begin{aligned}
 P{X\leq a, Y\leq b} &= \sum_{y\leq b}\sum_{x\leq a}p(x,y)\\
&= \sum_{y\leq b}\sum_{x\leq a}p_X(x)p_Y(y)\\
&=\sum_{y\leq b}p_Y(y)\sum_{x\leq a}p_X(x)\\
&= P\set{Y\leq b}P\set{X\leq a}
\end{aligned} $$

Now it is time to prove for the continuous version. Suppose that the joint probability distribution function $f(x,y)$ satisfies Equation (b).

$$ \begin{aligned}
F(a,b) &= P(X \le a, Y \le b)\\
&= \int_{-∞}^{b} \int_{-∞}^{a} f(x,y)\; dx\; dy \\
&= \int_{-∞}^{b} f_Y(y) \left( \int_{-∞}^{a} f_X(x) \, dx \right) dy \\
&= \left( \int_{-∞}^{a} f_X(x) \, dx \right) \left( \int_{-∞}^{b} f_Y(y) \, dy \right) \\
& = F_X(a)F_Y(b)
\end{aligned} $$

***

**Intuitive Example: Independence**

*   **Independent Events:** The two dice rolls (`X` and `Y` from the previous example) are **independent**. Knowing the outcome of the red die gives you no information about the outcome of the blue die. The probability of the blue die being a 5 remains 1/6, even if you know the red die was a 2.
    `P(Y=5 | X=2) = P(Y=5) = 1/6`.

*   **Dependent Events:** Now consider two different variables. Let `X` be the amount of rainfall today (in mm) and `Y` be the number of umbrellas sold. These variables are **not independent**. If you know there was heavy rainfall (`X > 10mm`), it becomes much more likely that a large number of umbrellas were sold (`Y` is high).
    `P(Y is high | X is high) > P(Y is high)`.

***

## Proposition 1.2

If $X$ and $Y$ are independent, then for any function $h$ and $g$

$$ E[g(X)h(Y)] = E[g(X)]E[h(Y)] $$

## Covariance and Variance of Sums of Random Variables

The covariance of any two random variables $X$ and $Y$, denoted by $\text{Cov}(X,Y)$, is defined by:

$$ \begin{aligned}
\text{Cov}(X,Y) &= E[(X-E[X])(Y-E[Y])]\\
 &=E[XY - YE[X] - XE[Y] + E[X]E[Y]] \\
 &=E[XY] - E[Y]E[X] - E[X]E[Y] + E[X]E[Y] \\
 &= E[XY]-E[X]E[Y]
\end{aligned} $$

Note that if $X$ and $Y$ are independent, then by Proposition 1.2, it follows that $\text{Cov}(X,Y)=0$. 

***

**Intuitive Example: What Covariance Tells Us**

Covariance is a measure of how two variables move in relation to each other.

*   **Positive Covariance:** Consider `X` = daily temperature and `Y` = ice cream sales. As the temperature increases, ice cream sales tend to increase as well. These variables move in the same direction, so they would have a **positive covariance**.

*   **Negative Covariance:** Consider `X` = hours spent studying for an exam and `Y` = number of mistakes made on the exam. Generally, as study hours increase, the number of mistakes tends to decrease. These variables move in opposite directions, so they would have a **negative covariance**.

*   **Zero Covariance:** Consider `X` = a person's shoe size and `Y` = their score on a history test. There is no logical reason to believe these two variables are related. They are independent, and their covariance would be **zero**.

***

Let’s consider a special case where $X$ and $Y$ are indicator variables for whether or not events $A$ and $B$ occur. That is, for events $A$ and $B$, define 

$$ X=\begin{cases} 1, & \text{if A occurs}\\
 0, & \text{otherwise}
\end{cases} \quad Y=\begin{cases} 1, & \text{if B occurs}\\
 0, & \text{otherwise}
\end{cases} $$

Then,

$$ \text{Cov}(X,Y) = E[XY] - E[X]E[Y] $$

and because $XY$ will equal 1 or 0 depending on whether or not both $X$ and $Y$ equal 1, we see that

$$ \text{Cov}(X,Y) = P\set{X=1, Y=1}-P\set{X=1}P\set{Y=1} $$

From this we can observe that

$$ \begin{aligned}
  \text{Cov}(X, Y) > 0 &\Leftrightarrow P{X = 1, Y = 1} > P{X = 1}P{Y = 1} \\
  &\Leftrightarrow \frac{P{X = 1, Y = 1}}{P{X = 1}} > P{Y = 1} \\
  &\Leftrightarrow P{Y = 1 \mid X = 1} > P{Y = 1}
\end{aligned} $$

That is, the covariance of $X$ and $Y$ is positive if the outcome $X=1$ makes it more likely that $Y=1$ (which, as is easily seen by symmetry, also implies the reverse). Intuitively, if we know that $X$ and $Y$ have a positive covariance, then it is more likely for $X=1$ to happen if we already know that $Y=1$.

## properties of Covariance

For any random variables, $X,Y,Z$ and constant $c$,

1. $\text{Cov}(X,X) = \text{Var}(X)$
2. $\text{Cov}(X,Y) = \text{Cov}(Y,X)$
3. $\text{Cov}(cX,Y) = c\text{Cov}(X,Y)$
4. $\text{Cov}(X,Y+Z) = \text{Cov}(X,Y) + \text{Cov}(X,Z)$

The first three properties are immediate. I will prove the final property below:

$$ \begin{aligned}
\text{Cov}(X,Y+Z) &= E[X(Y+Z)]-E[X]E[Y+Z]\\
&=E[XY]-E[X]E[Y]+E[XZ]-E[X]E[Z]
\\ &=\text{Cov}(X,Y)+\text{Cov}(X,Z)
\end{aligned} $$

If $X_1,...,X_n$ are independent and identically distributed, then the random variable $\bar{X} = \sum_{i=1}^n \frac{X_i}{n}$ is called the *sample mean*.

## proposition 1.3

Suppose that $X_1,...,X_n$ are independent and identically distributed with expected value $\mu$ and variance $\sigma^2$. Then:

1. $E[\bar{X}]=\mu$
2. $\text{Var}(\bar{X})=\frac{\sigma^2}{n}$
3. $\text{Cov}(\bar{X},X_i-\bar{X})=0,\;i=1,...,n$

Since (a) and (b) are quite intuitive, I will prove (c) below:

$$ \begin{aligned}
\text{Cov}(\bar{X},X_i-\bar{X})&= \text{Cov}(\bar{X},X_i)-\text{Cov}(\bar{X},\bar{X}) \\ 
&=\frac{1}{n}\text{Cov}(X_i+ \sum_{j\neq i}X_j,X_i) - \text{Var}(\bar{X}) \\
& = \frac{1}{n}\text{Cov}(X_i,X_i) +\frac{1}{n}\text{Cov}(\sum_{j\neq i}X_j,X_i)-\frac{\sigma^2}{n} \\
&= \frac{\sigma^2}{n} -\frac{\sigma^2}{n} = 0  
\end{aligned} $$

***

**Concrete Example: Student Heights**

Imagine we measure the heights of a small sample of 5 students (in cm):
`{165, 170, 175, 180, 185}`

1.  **The Sample Mean (`X̄`)** is the average height:
    `X̄ = (165 + 170 + 175 + 180 + 185) / 5 = 175 cm`.
    This value (175 cm) represents the central point of our data.

2.  **The Deviations (`Xᵢ - X̄`)** are how far each student's height is from that average:
    *   For `X₁=165`, the deviation is `165 - 175 = -10 cm`.
    *   For `X₂=170`, the deviation is `170 - 175 = -5 cm`.
    *   For `X₃=175`, the deviation is `175 - 175 = 0 cm`.
    *   And so on... the deviations are `{-10, -5, 0, 5, 10}`.

Proposition 1.3c tells us that `Cov(X̄, Xᵢ - X̄) = 0`. In this context, it means that knowing the average height of the group (175 cm) gives you no information about one student's specific deviation from that average. The average could be 175 cm for a group with very little variation (e.g., {174, 175, 175, 175, 176}) or a group with a lot of variation, like our example. The center of the data and the spread around the center are independent pieces of information.

***

The **Sample Mean ($\bar{X}$)**: This is a representative value that tells us where the centre of our entire dataset is located.

A **Deviation (or Residual), ($X_i-\bar{X}$)**: This shows how far an individual data point ($X_i$) is from that center.

The fact that the covariance between these two is zero signifies a powerful concept: **information about the overall central location of the data is independent of information about how individual data points deviate from that centre.**

Today's post was a bit mathematical, but these concepts are crucial. In the next post, we'll discuss more on the joint distribution functinon, especailly on multivate change of variables.