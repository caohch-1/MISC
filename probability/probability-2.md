---
title: SI140 Lec2
categories:
- Probability
mathjax: true
---

# Definition & Intuition

New data & information may affect our uncertainties

Keep in mind: Thinking conditionally! All probabilities are conditional! Conditioning is the soul of statistics!

Definition:

If $A$ and $B$ are events with $P(B)>0$, then the conditional probability of $A$ given $B$, denoted by $P(A|B)$, is defined as
$$
P(A|B)=\frac{P(A\cap B)}{P(B)}
$$
$P(A)$: prior probability of $A$

$P(A|B)$: posterior probability of $A$

$B$: new data/ information

# Bayes' Rule & LOTP

## Chain Rule

For any events $A_1,\dots,A_n$ with postive probabilities,
$$
P(A_1,\dots,A_n)=P(A_1)P(A_2|A_1)P(A_3|A_1,A_2)\cdots P(A_n|A_1,\dots,A_{n-1})
$$
An interesting question raised by Prof.Shao in class:

Suppose you have $n$ machines, how to reduce the time to calculate $P(A_1,\dots,A_n)$ ?

Use $log$ turn the equation into 
$$
log(P(A_1,\dots,A_n))=log(P(A_1))+log(P(A_2|A_1))+log(P(A_3|A_1,A_2))+\cdots +log(P(A_n|A_1,\dots,A_{n-1}))
$$
Let every machine calculate one part in the right hand of the equation.

 Using $log$ when meeting continuous multiplication is a common operation in actual engineering application because addition is much faster than multiplication.



## Bayes' Rule

For any events $A$ and $B$ with postive probabilities,
$$
P(A|B)=\frac{P(B|A)P(A)}{P(B)}
$$

## The Law of Total Probability(LOTP)

Let $A_1,\dots,A_n$ be a partition of the sample space $S$ (i.e., the $A_i$ are disjoint events and their union is $S$), with $P(A_i)>0$ for all $i$. Then
$$
P(B)=\sum_{i=1}^n P(B|A_i)P(A_i)
$$
![lotp](https://s1.ax1x.com/2020/09/26/0izCTO.png)



## Inference & Bayes' Rule

Let $A_1,\dots,A_n$ be a partition of the sample space $S$ (i.e., the $A_i$ are disjoint events and their union is $S$), with $P(A_i)>0$ for all $i$. Then for any event $B$ such that $P(B)>0$, we have
$$
P(A_i|B)=\frac{P(A_i)P(B|A_i)}{P(A_1)P(B|A_1)+\cdots +P(A_n)P(B|A_n)}
$$

## Example

### Bayes Spam Filter

A spam filter is designed by looking at commonly occurring phrases in spam. Suppose that 80% of email is spam. In 10% of the spam emails, the phrase “free money” is used, whereas this phrase is only used in 1% of non-spam emails. A new email has just arrived, which does mention “free money”. What is the probability that it is spam?

Ans: $\frac{80}{82}$

Reference: https://en.wikipedia.org/wiki/Naive_Bayes_spam_filtering

### Random Coins

You have one fair coin, and one biased coin which lands Heads with probability 3/4. You pick one of the coins at random and flip it three times. It lands Heads all three times. Given this information, what is the probability that the coin you picked is the fair one?

Ans: $\frac{8}{35}$

### Communication Channel

Suppose that Alice sends only one bit (a 0 or 1) to Bob, with equal probabilities. If she sends a 0, there is a 5% chance of an error occurring, resulting in Bob receiving a 1; if she sends a 1, there is a 5% chance of an error occurring, resulting in Bob receiving a 0.
Given that Bob receives a 1, what is the probability that Alice actually sent a 1?

Ans: $0.95$

# Conditional Probability

Def function $\hat{P}(\cdot)=P(\cdot|E)$ 

$\hat{P}$ is also a probability function with sample space $S$ 

## Bayes' Rule with Extra Conditioning

Provided that $P(A\cap E)>0$ and $P(B\cap E)>0$, we have
$$
P(A|B,E)=\frac{P(B|A,E)P)(A|E)}{P(B|E)}
$$


## LOTP with Extra Conditioning

Let $A_1,\dots,A_n$ be a partition of the sample space $S$ (i.e., the $A_i$ are disjoint events and their union is $S$), with $P(A_i)>0$ for all $i$. Then
$$
P(B|E)=\sum_{i=1}^n P(B|A_i,E)P(A_i|E)
$$

## Example

### Random Coin

You have one fair coin, and one biased coin which lands Heads with probability 3/4. You pick one of the coins at random and flip it three times. It lands Heads all three times. If we toss the coin a fourth time, what is the probability that it will land Heads once more?

Ans: $\frac{97}{140}$



## Approaches for Finding $P(A|B,C)$

- Think of $B,C$ as the single event $B\cap C$.
- Use Bayes' rule with extra conditioning on $C$.
- Use Bayes' rule with extra conditioning on $B$.

# Independence of Events

## Independence of Two Events

Events $A$ and $B$ are independence if
$$
P(A\cap B)=P(A)P(B)
$$
If $P(A)>0$ and $P(B)>0$, then this is equivalent to
$$
P(A|B)=P(A)
$$

$$
P(B|A)=P(B)
$$

## Independence vs. Disjointness

$A,B$ disjoint ====> $P(A\cap B)=0$

$A,B$ independent ====> $P(A\cap B)=P(A)P(B)$



## Independence of Three Events

Events $A,B$ and $C$ are independent if all of the following equations hold:
$$
P(A\cap B)=P(A)P(B)
$$

$$
P(A\cap C)=P(A)P(C)
$$

$$
P(B\cap C)=P(B)P(C)
$$

$$
P(A\cap B\cap C)=P(A)P(B)P(C) \quad//can't \,be \,omitted
$$



## Conditional Independence

Events $A$ and $B$ are said to be conditionally independent given $E$ if:
$$
P(A\cap B|E)=P(A|E)P(B|E)
$$


# Conditioning As A Problem-Solving Tool

## Laplace's Rule of Succession

There are $k + 1$ coins in a box. When flipped, the $i$ coin will turn up heads with probability $i/k$, $i = 0, 1,\dots , k.$ A coin is randomly selected from the box and is then repeatedly flipped. If the first $n$ flips all result in heads, what is the conditional probability that the $(n + 1)^{th}$ flip will do likewise?

Res=$\frac{\sum_{i=0}^k (\frac{i}{k})^{n+1}}{\sum_{i=0}^k (\frac{i}{k})^n}=\frac{n+1}{n+2}$

## Branching Process

A single amoeba, Bobo, lives in a pond. After one minute Bobo will either die, split into two amoebas, or stay the same, with equal probability, and in subsequent minutes all living amoebas will behave the same way, independently. What is the probability that the amoeba population will eventually die out?

Res=$1$





