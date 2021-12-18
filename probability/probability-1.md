---
title: SI140 Lec1
categories:
- Probability
mathjax: true
---

# Counting

## Generalized Birthday Problem

If the probability that at least two people have the same number is 50%, then $k\approx 1.18\sqrt{n}$.

Proof:
$$
\begin{align*}
P(A^C)&=\frac{n(n-1)\cdots(n-k+1)}{n^k}\\
	  &=1\cdot(1-\frac{1}{n})\cdot(1-\frac{2}{n})\cdots(1-\frac{k-1}{n})\\
	  &=e^{-\frac{1+2+\cdots+(k-1)}{n}}\\
	  &=e^{-\frac{k\cdot(k-1)}{2n}}\\
	  &\approx e^{-\frac{k^2}{2n}}\\
if\,P(A^C)&=0.5,\,k\approx 1.18\sqrt{n}
\end{align*}
$$
Application: Hash Collision, Cryptographic Hash Function



## Story Proof

**Story Proof的好处在与构建一个例子，从而得以简化复杂的数学推导进而快速证明**

### Team Capatin

For any postive integers $n$ and $k$ with $k\leq n$,
$$
n\cdot C_{n-1}^{k-1}=k\cdot C_{n}^{k}
$$
Consider a group of $n$ people.

Choose a team of $k$ people, one of the team member is captain.

LHS: First choose captain, then team member

RHS: First choose team member, then captain



### Vandermonde's identity

A famous relationship between binomial coefficients, called *Vandermonde's identity*, says that 
$$
C_{m+n}^{k}=\sum_{j=0}^{k}C_m^j\cdot C_n^{k-j}
$$
Consider a group of $m$ women and $n$ men.

Choose a team of $k$ people.

LHS: Directly choose $k$ people

RHS: First $j$ men, then $k-j$ women 



## Bose-Einstein

How many ways are there to choose $k$ times from a set of $n$ objects with replacement, if order doesn't matter (we only care about how many times each object was choosen, not the order in which they were chosen)?

A equivalent problem is $x_1+x_2\cdots+x_n=k,\,x_i\geq 0,\,i=1,2,\dots,n.$ (Diophantine Equation

Answer is $C_{n+k-1}^{n-1}$(can be solved easily using knowledge in discrete mathematics



## Summary of counting

|                     | Order Matters         | Order Not Matter |
| ------------------- | --------------------- | ---------------- |
| with replacement    | $n^k$                 | $C_{n+k-1}^{k}$  |
| without replacement | $n(n-1)\cdots(n-k+1)$ | $C_n^k$          |



# Definition of Probability

## Two axioms

1.$P(\emptyset)=0,\,P(S)=1$

2.If $A_1,A_2,\dots$ are disjoint events, then $P(\bigcup_{j=1}^\infty A_j)=\sum_{j=1}^\infty P(A_j)$

## Two views

The *frequentist view:* probability represents a long-run frequency over a large number of repetitions of an experiment

The *Bayesian view*: probability represents a degree of blief about the event in question

## Properties of Probability

1. $P(A^C)=1-P(A)$

   Proof:

   $P(A\cup A^C)=P(S)=1$

   $P(A)+P(A^C)=1$

   

2. If $A\subseteq B,$ then $P(A)\leq P(B)$

	Proof:

	$B=A\cup (B\cap A^C)$

	$P(B)=P(A)+P(B\cap A^C)$

	Because $P(B\cap A^C)\geq 0$

   So $P(B)\geq P(A)$
   
   
   
3.  $P(A\cup B)=P(A)+P(B)-P(A\cap B)$

   Proof: 

   $P(A\cup B)=P(A\cup (B\cap A^C))=P(A)+P(B\cap A……C)$

   $P(B)=P((A\cup B)\cup(B\cap A^C))=P(A\cup B)+P(B\cap A^C)$

   $P(B\cap A^C)=P(B)-P(A\cap B)$

   $P(A\cup B)-P(A)=P(B)-P(A\cap B)$

## Bonferroni's Inequality

For any $n$ events $A_1,\dots,A_n$, we have
$$
P(A_1\cap A_2\cap\cdots\cap A_n)\geq P(A_1)+P(A_2)+\cdots+P(A_n)-(n-1)
$$
Proof: 
$$
\begin{align}
1^\circ P(A_1\cap A_2\cap\cdots\cap A_n)&=1-P(\overline{A_1\cap A_2\cap\cdots\cap A_n})\\
&=1-P(\bar{A_1}\cup\cdots\cup\bar{A_n})\\
2^\circ P(\bar{A_1}\cup\cdots\cup\bar{A_n})&\leq P(\bar{A_1})+P(\bar{A_2})+\cdots+P(\bar{A_n})\\
&=(1-P(A_1))+(1-P(A_2))+\cdots+(1-P(A_n))\\
&=n-(P(A_1)+P(A_2)+\cdots+P(A_n))\\
3^\circ P(A_1\cap A_2\cap\cdots\cap A_n)&\geq 1-n+(P(A_1)+P(A_2)+\cdots+P(A_n)
\end{align}
$$


## Boole's Inequality

For any events $A_1,A_2,\dots$, we have
$$
P(\bigcup_{i=1}^\infty A_i)\leq\sum_{i=1}^\infty P(A_i)
$$
Proof:
$$
\begin{align}
1^\circ \text{We need to find }&B\text{ satisfy }\bigcup_{i=1}^\infty A_i=\bigcup_{i=1}^\infty B_i,\,B_i\cap B_j=\emptyset\\
2^\circ A_1\cup\cdots\cup& A_n=A_1\cup(A_2\cup\bar{A_1})\cup(A_3\cup\overline{A_1\cup A_2})\cdots\cup(A_n\cup\overline{A_1\cup\cdots\cup A_{n-1}})\\
3^\circ B_i&=A_i\cap\overline{\bigcup_{j=1}^{i-1}A_j},\,i=1,\dots\infty(B_i\subseteq A_i)\\
4^\circ P(\bigcup_{i=1}^\infty A_i)&=P(\bigcup_{i=1}^\infty B_i)\\
&=\sum_{i=1}^\infty P(B_i)\\
&=\sum_{i=1}^\infty P(A_i\cap\overline{\bigcup_{j=1}^{i-1}A_j})\\
&\leq\sum_{i=1}^\infty P(A_i)
\end{align}
$$








