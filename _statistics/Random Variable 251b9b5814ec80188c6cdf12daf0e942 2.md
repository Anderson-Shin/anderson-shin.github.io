# Random Variable

# Expectation of random variables

## Proposition 1.1

1. If $X$ is a discrete random variable with probability mass function $p(x)$, then for any real-valued function $g$:
    
    $$
    E[g(x)]=\sum_{x:p(x)>0}g(x)p(x)
    $$
    
2. If $X$ is a continuous random variable with probability density function $f(x)$, then for any real-valued function g,

$$
E[g(X)]=\int_{-\infin}^{\infin} g(x)f(x)dx
$$

## Corollary 1.1

If $a$ and $b$ are constants, then

**Proof** In the discrete case,

$$
\begin{aligned}E[aX+b] &= \sum_{x:p(x)>0} (aX+b)p(x)\\
&= a \sum_{x:p(x)>0}xp(x)+b \sum_{x:p(x)>0} p(x)\\
&= aE[X]+b
\end{aligned}
$$

In the continuous case,

$$
\begin{aligned}E[aX+b] &= \int_{-\infin}^{\infin}(aX+b)f(x)dx\\
&= a \int_{-\infin}^{\infin}xf(x)dx+b \int_{-\infin}^{\infin} f(x)dx\\
&= aE[X]+b
\end{aligned}
$$

# Jointly Distributed Random Variables

Okay, we have discussed about expectations and variances of discrete and continuous variables so far. Now, we are going to talk about something more frequently concerned in our real life. We are often interested in probability statements concerning two or more random variables.

To deal with such probabilities, we define, for any two random variables $X$ and $Y$, the *joint cumulative proabiltiy distribution function* of $X$ and $Y$  by

$$
F(a,b)=P\{X\leq a, Y\leq b\}, \quad -\infin<a, b<\infin
$$

The distribution of $X$ can be obtained from the joint distribution of $X$  and $Y$ as follows:

$$
\begin{aligned} F_X(a) &= P\{X\leq a\}\\
&= P\{X\leq a, Y < \infin \}\\
&=F(a,\infin)
\end{aligned}
$$

In the case where $X$ and $Y$ are both discrete random variables, it is convenient to define the *joint probability mass function* of $X,Y$ by

$$
p(x,y)=P\{X=x,Y=y\}
$$

The probability mass function of $X$ may be obtained from $p(x,y)$ by

$$
p_X(x)=\sum_{y:p(x,y)>0}p(x,y)
$$

Similarly,

$$
p_Y(y)=\sum_{x:p(x,y)>0}p(x,y)
$$

We say that $X$ and $Y$ are *jointly continuous* if there exists a function $f(x,y)$, defined for all real $x$ and $y$, having the property that for all sets $A$ and $B$ of real numbers

$$
P\{X \in A, Y \in B\} = \int_B\int_Af(x,y)dxdy
$$

The function $f(x,y)$ is called *joint probability density function* of $X$ and $Y$. The probability density of $X$ (marginal probability of $X$)can be obtained from a $f(x,y)$ by the following reasoning:

$$
\begin{aligned}
P\{X \in A \} &= P\{X \in A, \quad Y \in (-\infin,\infin)\}\\
&= \int_{-\infin}^{\infin}\int_Af(x,y)dxdy\\
&=\int_Af_X(x)dx
\end{aligned}
$$

where

$$
f_X(x)=\int_{-\infin}^{\infin}f(x,y)dy

$$

similarly,

$$
f_Y(y)=\int_{-\infin}^{\infin}f(x,y)dx

$$

If we take a look at Proposition 1, we can derive expectation of jointly discrete or continuous distribution.

$$
E[g(X,Y)] =
\begin{cases}
  \sum_y\sum_xg(x,y)p(x,y) & \text{in the discrete case} \\\\
  \int_{-\infty}^{\infty}\int_{-\infty}^{\infty} xf(x,y)dxdy, & \text{in the continuous case}
\end{cases}
$$

For example, if $g(X,Y)= X+Y$, then in this case,

$$
\begin{aligned}
  E[X + Y] &= \int_{-\infty}^{\infty} \int_{-\infty}^{\infty} (x + y) f(x, y) \, dx \, dy \\
  &= \int_{-\infty}^{\infty} \int_{-\infty}^{\infty} xf(x, y) \, dx \, dy + \int_{-\infty}^{\infty} \int_{-\infty}^{\infty} yf(x, y) \, dx \, dy \\
  &= E[X] + E[Y]
\end{aligned}
$$

# Independent Random Variable

The random variables $X$  and $Y$ are said to be *independent* if, for all $a,b$

$$
P\set{X\le a, Y\leq b}=P\set{X\leq a}P\set{Y\leq b}
$$

In other words, $X$  and $Y$ are independent if, for all $a$ and $b$, the eventes $E_a=\set{X\leq a}$ and $F_b=\set{Y\leq b}$ are independent.

In terms of the joint distribution function $F$ of $X$ and $Y$, we have that $X$ and $Y$ are independent if

$$
F(a,b)=F_X(a)F_Y(b) \quad \text{for all a, b}
$$

when $X$ and $Y$ are discrete, the condition of independence reduces to 

$$
p(x,y)=P_X(x)P_Y(y) \tag{1}
$$

while if $X$  and $Y$ are jointly continuous, independence reduces to 

$$
f(x,y)=f_X(x)f_Y(y) \tag{2}
$$

To prove this statement, consider first the discrete version, and supose that the joint probability mass function $p(x,y)$ satisfies Equaiton (1).

$$
\begin{aligned}
P\set{X\leq a, Y\leq b} &= \sum_{y\leq b}\sum_{x\leq a}p(x,y)\\
&= \sum_{y\leq b}\sum_{x\leq a}p_X(x)p_Y(y)\\
&=\sum_{y\leq b}p_Y(y)\sum_{x\leq a}p_X(x)\\
&= P\set{Y\leq b}P\set{X\leq a}

\end{aligned}
$$

Now it is time to prove for the continuous version, and suppose that the joint probability distribution function $f(x,y)$ satisfy Equation (2).

$$
\begin{aligned}
F(a,b) &= P(X \le a, Y \le b)\\
&= \int_{-\infty}^{b} \int_{-\infty}^{a} f(x,y)\; dx\; dy \\
&= \int_{-\infty}^{b} f_Y(y) \left( \int_{-\infty}^{a} f_X(x) \, dx \right) dy \\
&= \left( \int_{-\infty}^{a} f_X(x) \, dx \right) \left( \int_{-\infty}^{b} f_Y(y) \, dy \right) \\
& = F_X(a)F_Y(b)

\end{aligned}
$$

## Proposition 1.2

If $X$ and $Y$ are independent, then for any function $h$ and $g$

$$
E[g(X)h(Y)] = E[g(X)]E[h(Y)]
$$

# Covariance and Variance of Sums of Random Variables

The covariance of any two random variables $X$ and $Y$, denoted by $\text{Cov}(X,Y)$, is defined by:

$$
\begin{aligned}
\text{Cov}(X,Y) &= E[(X-E[X])E[(Y-E(Y)]\\
&=E[XY - YE[X] - XE[Y] + E[X]E[Y]] \\
&=E[XY] - E[Y]E[X] - E[X]E[Y] + E[X]E[Y] \\
&= E[XY]-E[X]E[Y]
\end{aligned}
$$

Note that if $X$ and $Y$ are independent, then by Proposition 1.2, it follows that $\text{Cov}(X,Y)=0$. 

Let’s consider special case where $X$ and $Y$ are indicator variables for whether or not the events $A$ and $B$ occur. That is, for events $A$ and $B$, define 

$$
X=\begin{cases} 1, \quad \text{if A occurs}\\ 
0, \quad \text{otherwise, }
\end{cases}

Y=\begin{cases} 1, \quad \text{if B occurs}\\ 
0, \quad \text{otherwise, }
\end{cases}
$$

Then,

$$
\text{Cov}(X,Y) = E[XY] - E[X]E[Y]
$$

and because $XY$ will eqaul 1 or 0 depending on whether or not both $X$ and $Y$ equal 1, we see that

$$
\text{Cov}(X,Y) = P\set{X=1, Y=1}-P\set{X=1}P\set{Y=1}
$$

From this we can observe that

$$
\begin{aligned}
  \text{Cov}(X, Y) > 0 &\Leftrightarrow P\{X = 1, Y = 1\} > P\{X = 1\}P\{Y = 1\} \\
  &\Leftrightarrow \frac{P\{X = 1, Y = 1\}}{P\{X = 1\}} > P\{Y = 1\} \\
  &\Leftrightarrow P\{Y = 1 \mid X = 1\} > P\{Y = 1\}
\end{aligned}
$$

That is, the covariance of $X$ and $Y$ is positive if the outcome $X=1$ makes it more likely that $Y=1$ (which, as is easily seen by symmetry, also implies the reverse).  Intuitively speaking, it is likely that  $X=1$ to happen if we know that $Y=1$ already happened, given that they both have positive covariance.

## properties of Covariance

For any random variables, $X,Y,Z$ and constant $c$,

1. $\text{Cov}(X,X) = \text{Var}(X)$
2. $\text{Cov}(X,Y) = \text{Cov}(Y,X)$
3. $\text{Cov}(cX,Y) = c\text{Cov}(X,Y)$
4. $\text{Cov}(X,Y+Z) = \text{Cov}(X,Y) + \text{Cov}(X,Z)$

I think first three properties are immediate, so I will proof the final property as below:

$$
\begin{aligned}
\text{Cov}(X,Y+Z) &= E[X(Y+Z)]-E[X]E[Y+Z]\\
&=E[XY]-E[X]E[Y]+E[XZ]-E[X]E[Z]\\
&=\text{Cov}(X,Y)+\text{Cov}(X,Z)

\end{aligned}
$$

If $X_1,...,X_n$ are independent and identically distributed, then the random variable $\bar{X} = \sum_{i=1}^n \frac{X_i}{n}$ is called the *sample mean*.

## proposition 1.3

Suppose that $X_1,...,X_n$ are independent and identically distributed with expected value $\mu$ and variance $\sigma^2$. Then:

1. $E[\bar{X}]=\mu$
2. $\text{Var}(\bar{X})=\frac{\sigma^2}{n}$
3. $\text{Cov}(\bar{X},X-\bar{X})=0,\;i=1,...,n$

Since a),b) is quite intuitive, I will prove c) as below:

$$
\begin{aligned}
\text{Cov}(\bar{X},X-\bar{X})&= \text{Cov}(\bar{X},X_i)-\text{Cov}(\bar{X},\bar{X})\\
&=\frac{1}{n}\text{Cov}(X_i+ \sum_{j\neq i}X_j,X_i) - \text{Var}(\bar{X}) \\
& = \frac{1}{n}\text{Cov}(X_i,X_i) +\frac{1}{n}\text{Cov}(\sum_{j\neq i}X_j,X_i)-\frac{\sigma^2}{n} \\
&= \frac{\sigma^2}{n} -\frac{\sigma^2}{n} = 0  
\end{aligned}
$$

The **Sample Mean ($\bar{X}$)**: This is a representative value that tells us where the centre of our entire dataset is located.

A **Deviation (or Residual), ($X_i-\bar{X}$)**: This shows how far an individual data point ($X_i$) is from that center.

The fact that the covariance between these two is zero signifies a powerful concept: **information about the overall central location of the data is independent of information about how individual data points deviate from that centre.**