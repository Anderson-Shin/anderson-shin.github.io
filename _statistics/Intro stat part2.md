---
title: "Introductory Statistics (Part 2): Putting Numbers on Possibilities"
collection: statistics
permalink: /statistics/probability-for-everyone-part2/
excerpt: "The second post in a beginner-friendly series on probability. In this follow-up, we dive into the mechanics of probability. Learn the addition rule for combining events, explore the powerful concept of conditional probability, and discover what it means for two events to be truly independent."
date: 2024-04-12
tags:
  - Independence
  - Conditional Probability
  - Combined Event
  - Statistics
---

# **Putting Numbers on Possibilities**

Welcome back. In our first post, we set the stage for our story. We learned how to define the world of an experiment with a **Sample Space** and how to identify the specific outcomes we care about with **Events**. It was all about organizing our thoughts.

Today, we're going to bring these concepts to life by assigning numbers to them. After all, probability is a measure. It tells us *how likely* an event is. But to do this, we need to agree on a few fundamental rules of the road.

### **The Rules of the Game**

Assigning a probability is like assigning a weight to an outcome. The heavier the weight, the more likely the outcome. This system of weights, however, must follow a few simple, common-sense rules.

First, the probability of any event must be a number between 0 and 1. A probability of 0 means the event is impossible, and a probability of 1 means it is certain. We can express this as:

$0 \le P(E) \le 1$  
Second, if we consider the entire sample space (all possible outcomes), its probability must be 1. This makes sense; when we run an experiment, something from the sample space is guaranteed to happen.

$P(S)=1$  
Now for the interesting part. How do we combine probabilities? Let's go back to our die roll. The sample space is {1, 2, 3, 4, 5, 6}, and for a fair die, each outcome has a probability of 1/6.

What is the probability of rolling a 1 OR a 3? These two events are mutually exclusive—they can't both happen on a single roll. In cases like this, we can simply add their probabilities. This is our third fundamental rule.

$P(1 \text{ or } 3) = P(1) + P(3) = \frac{1}{6} + \frac{1}{6} = \frac{2}{6}$  
But what if the events are *not* mutually exclusive? For example, what's the probability of rolling an even number **OR** a number less than 3?

* Event E (even): {2, 4, 6}  
* Event F (less than 3): {1, 2}

If we just add P(E)+P(F), we are counting the outcome '2' twice. We have to correct for this by subtracting the probability of the overlapping part, their intersection. This gives us the general addition rule:

$P(E \cup F) = P(E) + P(F) - P(E \cap F)$
$P(\text{even or } <3) = \frac{3}{6} + \frac{2}{6} - P(2) = \frac{5}{6} - \frac{1}{6} = \frac{4}{6}$

### **When Information Changes Everything: Conditional Probability**

Now we can explore one of the most powerful ideas in probability: how our calculations change when we get new information.

Let's say I draw a single card from a 52-card deck. The probability that it's a King is $P(\text{King}) = \frac{4}{52}$.

But what if I look at the card and tell you, "This card is a face card" (a Jack, Queen, or King)? The situation has changed. Our world, our sample space, has shrunk from 52 cards down to just the 12 face cards. Within this new, smaller world, there are still 4 Kings.

So, the new probability is $\frac{4}{12} = \frac{1}{3}$.

This is conditional probability: the probability of an event given that another event has occurred. We write it as $P(E\|F)$ and it reads "the probability of E given F." The formula for this perfectly captures our intuition:

$P(E\|F) = \frac{P(E \cap F)}{P(F)}$

Where:

It tells us to focus only on the outcomes where $F$ has happened $P(F)$ becomes our new denominator) and then find the proportion of those outcomes that also include $(P(E∩F))$.

### **When Information Changes Nothing: Independent Events**

Sometimes, new information is completely useless. It doesn't change the probability of our event of interest at all. When this happens, we say the two events are **independent**.

Imagine I flip a coin and you roll a die. The probability your die shows a 6 is $\frac{1}{6}$. If I tell you my coin landed on Heads, does that change the probability of you rolling a 6? Not at all. The two events are independent.

In formal terms, two events E and F are independent if knowing F has occurred does not change the probability of E.  
From $P(E|F) = P(E)$ we can derive the most common test for independence. If we substitute this into the conditional probability formula, we get the simple multiplication rule for independent events:  
$P(E \cap F) = P(E)P(F)$

If the probability of two events happening together is just the product of their individual probabilities, they have nothing to do with each other.

### **What's Next?**

Today we've made a big leap. We've gone from simply defining events to assigning them probabilities based on a few core rules. More importantly, we've learned how to update these probabilities with new information.

This sets us up for the final part of our series, where we'll look at one of the most elegant and powerful applications of these ideas: **Bayes' Formula**. It uses conditional probability to let us work backward—to figure out the probability of a cause, given that we've seen the effect. I hope you'll join me for it.
