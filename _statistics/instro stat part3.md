---
title: "Introductory Statistics (Part 3): Reversing Time with Bayes' Formula"
collection: statistics
permalink: /statistics/probability-for-everyone-part3/
excerpt: "The final post in our beginner-friendly introductory probability series. Discover the power of Bayes' Formula to work backward from an effect to its cause. We explore this profound concept with a clear, real-world example of medical testing."
date: 2025-08-13
tags:
  - Bayes' Formula
  - Conditional Probability
  - Prior Probability
  - Posterior Probability
  - Statistics
---

# **Reversing Time with Bayes' Formula**

Hello and welcome to the final installment of our instroductory probability series. In the first part, we set our stage with sample spaces and events. In the second, we learned the rules for assigning probabilities and saw how they change with new information through conditional probability.

Today, we assemble those pieces to explore one of the most profound ideas in this field: **Bayes' Formula**. It’s a beautifully simple equation that allows us to do something that feels almost like reversing time: to work backward from a known effect to find the probability of its original cause. It’s a tool for updating our beliefs in a logical way, and it quietly powers much of our modern world.

## **A Common, Confusing Problem**

Let's begin with a classic scenario that shows why we need a tool like Bayes' Formula.

Imagine a lab develops a test for a certain rare disease. This disease affects just 1 in 1,000 people. The test itself is quite good:

* It's **99% accurate** if you have the disease (it correctly identifies 99% of sick people).  
* It has a **2% "false positive" rate** (it incorrectly flags 2% of healthy people as having the disease).

Now, suppose you take the test, and your result comes back **positive**. What is the probability that you actually have the disease?

Most people's intuition jumps to a high number, like 99% or perhaps a bit less. The reality, however, is quite different, and this is where our intuition can often lead us astray.

### **Let's Think It Through Intuitively**

Instead of starting with a formula, let's imagine a representative group of 10,000 people.

1. How many people have the disease?  
   The disease affects 1 in 1,000 people, so in our group of 10,000, about 10 people are sick. This leaves 9,990 people who are healthy.  
2. How many of the sick people will test positive?  
   The test is 99% accurate for sick people. Of the 10 sick people, about $0.99×10≈10$ will get a true positive result.  
3. How many of the healthy people will test positive?  
   The false positive rate is 2%. Of the 9,990 healthy people, $0.02×9,990≈200 $ will still get a false positive result.  
4. So, what's the answer?  
   In total, about $10+200=210$ people in our group received a positive test result. Of those 210 people, how many actually have the disease? Only 10\.

The probability that you have the disease, given your positive test, is the ratio of true positives to all positives:

Total number of people who test positiveNumber of sick people who test positive​=21010​≈0.047  
That's just **4.7%**. This is a surprisingly low number. It's because the disease is so rare that the small percentage of false positives from the very large healthy population ends up being a much larger group than the true positives from the small sick population.

## **The Elegance of Bayes' Formula**

What we just did with a concrete example is exactly what Bayes' Formula does in a more general way. It provides a clean, formal structure for this "reversal" of logic.

Bayes' Formula:

$P(A∣B)=P(B)P(B∣A)P(A)​$

Let's translate this into the terms of our problem:

* $A$ \= The event that you actually have the disease.  
* $B$ \= The event that you test positive.

We want to find P(A∣B): the probability of having the disease, given a positive test.

Let's identify each piece of the formula from the problem description:

* $P(A)$**: The Prior.** This is our belief before seeing evidence. It's the general probability of having the disease, which is 10001​=0.001.  
* $P(B∣A)$**: The Likelihood.** This is the probability of testing positive *if* you have the disease. We were told this is 99%, or 0.99.  
* $P(B)$: The Evidence. This is the overall probability of anyone testing positive. As we saw, this includes both true positives and false positives. We calculate it using the rules we learned in the last post:  
  $P(B)=P(\text{true positive})+P(\text{false positive})$  
  $P(B)=P(B∣A)P(A)+P(B∣A^C)P(A^C)$  
  $P(B)=(0.99×0.001)+(0.02×0.999)≈0.02097$

Now, we just assemble the formula:

$P(A∣B)=0.020970.99×0.001​≈0.020970.00099​≈0.0472$

We arrive at the same result, about 4.7%. The formula is a compact and reliable way to structure this kind of reasoning.

## **A Way of Thinking**

Bayes' Formula is more than just an equation; it's a model for learning. It shows us how to logically update our initial beliefs $(P(A))$ in the face of new evidence (B) to arrive at a more refined, posterior belief ($P(A∣B)$). This principle is the engine behind much of modern data science and artificial intelligence.

## **Series Conclusion**

And with that, our journey comes to a close. We started by simply setting the stage with sample spaces and events. We established the rules of the game with the axioms of probability. We then learned how to adjust our views with conditional probability, and finally, we've seen how to reverse our logic to uncover the probability of a cause from its effect.

I hope this series has helped demystify the world of probability for you. It's a field that combines rigorous logic with a deep understanding of uncertainty, and it's a tool that can help us all think more clearly about the world around us.