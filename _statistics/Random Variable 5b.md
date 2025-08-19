---
title: "A Deep Dive into Random Variables (Part 5b): Joint Distributions [Change of variables]"
collection: statistics
permalink: /statistics/random-variables-part5b/
excerpt: "The sixth post in our series on random variables. Further dive into changes of variables in joint distribution function."
date: 2024-06-06
use_math: true
tags:
  - Joint Distribution
  - Covariance
  - Jacobian determinant
  - Random Variable
  - Statistics
---

In our last post, we explored the fascinating world of jointly distributed random variables, learning how to describe the relationship between two variables using concepts like joint PDFs and covariance. But what if we want to find the distribution of a *function* of these random variables? For example, if we know the joint distribution of a component's length ($X_1$) and width ($X_2$), how can we find the distribution of its area ($Y_1 = X_1 \cdot X_2$)?

This is where the **Change of Variables** technique comes in, and it's a powerful tool for transforming distributions.

## The Change of Variables Formula

When you have two random variables, $X_1$ and $X_2$, with a known joint probability density function (PDF) $f_{X_1, X_2}(x_1, x_2)$, you often need to find the joint PDF of new random variables, $Y_1$ and $Y_2$, which are functions of the original variables. That is, $Y_1 = g_1(X_1, X_2)$ and $Y_2 = g_2(X_1, X_2)$.

To find the new joint PDF, $f_{Y_1, Y_2}(y_1, y_2)$, you can use the following formula:

$$
f_{Y_1,Y_2}(y_1, y_2) = \frac{f_{X_1,X_2}(x_1, x_2)}{\lvert J(x_1, x_2)\rvert}
$$

This formula is valid if two conditions are met:

1. You can uniquely solve the original functions for $x_1$ and $x_2$. In other words, you can find the inverse functions $x_1 = h_1(y_1, y_2)$ and $x_2 = h_2(y_1, y_2)$.

2. The functions have continuous partial derivatives, and a special term called the **Jacobian determinant**, $J(x_1, x_2)$, is not zero.

The **Jacobian determinant** is defined as:

$$
\begin{aligned}
J(x_1, x_2) &= 
\begin{vmatrix}
\frac{\partial g_1}{\partial x_1} & \frac{\partial g_1}{\partial x_2} \\
\frac{\partial g_2}{\partial x_1} & \frac{\partial g_2}{\partial x_2}
\end{vmatrix} \\
&= \frac{\partial g_1}{\partial x_1}\frac{\partial g_2}{\partial x_2} - \frac{\partial g_1}{\partial x_2}\frac{\partial g_2}{\partial x_1}
\end{aligned}
$$

## Intuitive Explanation 

Think of this process like stretching or shrinking a map.

Imagine the original joint PDF, $f_{X_1, X_2}(x_1, x_2)$, is a lumpy surface over a flat map (the $x_1, x_2$ plane). The height of the surface at any point represents the probability density. The total volume under this surface is 1.

When you transform the variables from $(X_1, X_2)$ to $(Y_1, Y_2)$, you are essentially stretching, shrinking, or rotating the original map. A small square on the original map might become a stretched-out parallelogram on the new map.

The **Jacobian determinant**, $\lvert J(x_1, x_2)\rvert$, is the **scaling factor** that tells you how much the area of that small square has changed during the transformation.

* If $\lvert J\rvert > 1$, the area has expanded.

* If $\lvert J\rvert < 1$, the area has shrunk.

The formula $f_{Y_1,Y_2}(y_1, y_2) = f_{X_1,X_2}(x_1, x_2) \div \lvert J(x_1, x_2)\rvert$ ensures that the total probability remains 1. If the area expands ($\lvert J\rvert > 1$), the probability density must decrease (you divide by a larger number) to keep the total volume constant. Conversely, if the area shrinks ($\lvert J\rvert < 1$), the density must increase. It's like spreading the same amount of butter over a larger or smaller piece of toast; the thickness of the butter (the density) changes to compensate.

## A Simpler Example: Sum and Difference of Uniform Variables

Let's walk through a more straightforward example. Suppose $X$ and $Y$ are independent random variables, both uniformly distributed on the interval $(0, 1)$. Their joint PDF is:

$$
f_{X,Y}(x,y) = 1, \quad \text{for } 0 < x < 1, 0 < y < 1
$$

and 0 otherwise.

We want to find the joint distribution of their **sum** and **difference**.

The transformations are:
$U = g_1(X,Y) = X + Y$
$V = g_2(X,Y) = X - Y$

**Step 1: Find the inverse functions.**
We solve for $x$ and $y$ in terms of $u$ and $v$. This is a simple system of linear equations:
Adding the two equations: $u + v = 2x \implies x = \frac{u+v}{2}$
Subtracting the second from the first: $u - v = 2y \implies y = \frac{u-v}{2}$
So, our inverse functions are $x = h_1(u,v) = \frac{u+v}{2}$ and $y = h_2(u,v) = \frac{u-v}{2}$.

**Step 2: Calculate the Jacobian.**
We find the partial derivatives of the original functions, $g_1$ and $g_2$:

$$
\frac{\partial g_1}{\partial x} = 1, \quad \frac{\partial g_1}{\partial y} = 1
$$

$$
\frac{\partial g_2}{\partial x} = 1, \quad \frac{\partial g_2}{\partial y} = -1
$$

Now, we compute the determinant:

$$
\begin{aligned}
J(x,y) &= 
\begin{vmatrix}
1 & 1 \\
1 & -1
\end{vmatrix} \\
&= (1)(-1) - (1)(1) = -2
\end{aligned}
$$

The scaling factor we need is the inverse of the absolute value: $\frac{1}{\lvert J(x,y)\rvert} = \frac{1}{\lvert -2\rvert} = \frac{1}{2}$.

**Step 3: Apply the formula.**
Now we plug everything into the main formula:

$$
f_{U,V}(u,v) = f_{X,Y}\left(\frac{u+v}{2}, \frac{u-v}{2}\right) \cdot \frac{1}{\lvert J\rvert}
$$

Since the original PDF $f_{X,Y}$ is just 1 (within its domain), this becomes:

$$
f_{U,V}(u,v) = 1 \cdot \frac{1}{2} = \frac{1}{2}
$$

This new density is valid over a new domain defined by the original constraints $0 < x < 1$ and $0 < y < 1$. This new domain in the u-v plane is a square rotated by 45 degrees.

You can take a look at the visualization I have prepared for the example we mentioned above.

<iframe src="/assets/change_of_variables_rv5b.html" 
        width="100%" 
        height="750" 
        style="border:1px solid #ccc; border-radius: 8px;" 
        title="Interactive Continous Distribution Visualizer">
</iframe>

This simpler example clearly shows how the Jacobian acts as a constant scaling factor for the density when the transformation is linear. For the next post, we are going to take a look at the "Moment Generating Function". Keep it tuned!