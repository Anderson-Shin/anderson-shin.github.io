---
title: "Introductory Statistics (Part 1): Setting the Stage"
collection: statistics
permalink: /statistics/probability-for-everyone-part1/
excerpt: "The first post in a beginner-friendly series on probability. This introduction sets the stage by explaining the fundamental concepts of Sample Space and Events, the building blocks for understanding uncertainty."
date: 2025-08-13
tags:
  - Probability
  - Sample Space
  - Events
  - Statistics
---

# **Welcome to the World of Probability**

Does the word "probability" make you think of complex equations and confusing jargon? If so, I have some good news for you. At its heart, probability is simply my favorite way of understanding uncertainty, and it's something I believe we all use intuitively every day. When you check the weather forecast, decide whether to bring an umbrella, or guess who will win the big game, you're thinking in terms of probability.

This is the first post in a friendly, three-part series where I'll guide you through the fundamentals of probability theory. I promise, there's no scary math ahead. I'm going to start from the very beginning, using simple examples to build a solid foundation. So grab a cup of coffee, get comfortable, and let's begin our journey together.

## **The Stage for Our Story: Sample Space and Events**

Before I can talk about the chances of something happening, we first need to agree on what *can* happen. In probability, I like to call this "stage" for our experiment the **Sample Space**.

### **The Intuitive Idea**

Imagine a simple experiment: I roll a standard six-sided die. What are the possible outcomes?

I could get value of 1, 2, 3, 4, 5, or a 6\. That's the complete list. This set of all possible outcomes is what I call the **Sample Space**.

**Sample Space (S):** The set of all possible outcomes of an experiment.

Now, let's say I'm playing a game where I win if I roll an even number. I'm not interested in every single outcome, but a specific group of outcomes: $\{2, 4, 6\}$. This specific outcome or set of outcomes I care about is called an **Event**.

**Event (E):** A subset of the sample space. It's the specific outcome or group of outcomes I am interested in.

Here are a few more quick examples from my point of view:

* **My Experiment:** Flipping a coin.  
  * **Sample Space (S):** $\{\text{Heads}, \text{Tails}\}$  
  * **Event (E):** The coin landing on Heads.  
* **My Experiment:** Drawing a card from a standard deck.  
  * **Sample Space (S):** $\{\text{All 52 cards}\}$  
  * **Event (E):** Drawing a King.

### **A Touch of Terminology**

Sometimes I want to combine events. The language here comes from set theory, but the ideas are very simple.

![Image of a Venn diagram for three sets](/images/Image of a Venn diagram for three sets.jpeg)

* **Union $(E ∪ F)$:** This means event E **OR** event F (or both) happens. If $E$ is "rolling an even number" $\{2, 4, 6\}$ and $F$ is "rolling a number less than 3" $\{1, 2\}$, then $(E ∪ F)$ is $\{1, 2, 4, 6\}$.  
* **Intersection (E ∩ F or EF):** This means event E **AND** event F happen. Using the same E and F, the only outcome that is both even *and* less than 3 is 2\. So, $(E ∩ F)$ is $\{2\}$.  
* **Complement (Eᶜ):** This means event E **DOES NOT** happen. If E is "rolling an even number" $\{2, 4, 6\}$, then its complement Eᶜ is "not rolling an even number," which is $\{1, 3, 5\}$.

### **What's Next?

And that's it for today. We've done the essential first step: we've learned how to define the world of our experiment with a Sample Space and how to specify the outcomes we're interested in with Events. We haven't calculated any probabilities yet, but we've built the stage on which all the action will take place.

In the next post, we will start putting numbers to these ideas. We'll introduce the fundamental rules of probability and see how they allow us to calculate the chances of these events happening. I hope you'll join me for it.
