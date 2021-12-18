---
title: SI140 Lec4
categories:
- Probability
mathjax: true
---

# Expectation & Variance

## Definition

The expected value (also called the expectation or mean) of a discrete $r.v.$ $X$ whose distinct possible values are $x_1,x_2,\dots$ is defined by
$$
E(X)=\sum\limits_{j=1}^\infty x_j\cdot P(X=x_j)
$$

## Distribution

If $X$ and $Y$ are discrete $r.v.$s with the same distribution, then $E (X ) = E(Y)$  (if either side exists).

## Linearity

For any $r.v.$s $X$, $Y$ and any constant $c$,
$$
E(X+Y)=E(X)+E(Y)
$$

$$
E(cX)=cE(X)
$$

## Monotonicity of Expectation

Let $X$ and $Y$ be $r.v.$s such that $X\geq Y$ with probability 1. Then $E (X )\geq E (Y )$, with equality holding if and only if $X = Y$ with probability 1.

## Survival Function

有时候通过定义很难计算期望，可以考虑使用存活函数

Let $X$ be a nonnegative integer-valued $r.v.$. Let $F$ be the **CDF** of $X$ , and $G (x) = 1-F (x) = P(X > x)$. The function $G$ is called the
survival function of $X$. Then
$$
E(X)=\sum\limits_{n=0}^\infty G(n)
$$
That is, we can obtain the expectation of $X$ by summing up the survival function (or, stated otherwise, summing up tail probabilities of the distribution).

### Proof

$$
E(X)=\sum\limits_{n=0}^\infty n\cdot P(X=n)=0\cdot P(X=0) +1\cdot P(X=1)+2\cdot P(X=2)+\cdots
$$

$$
\left[\begin{array}{ccccc}
&P(X\geq 1): & P(X=1) &P(X=2) &P(X=3) &\cdots\\
&P(X\geq 2): &  &P(X=2) &P(X=3) &\cdots\\
&P(X\geq 3): &  & &P(X=3) &\cdots\\
&\vdots &\vdots &\vdots &\vdots &\vdots\\
&Sum: &1\cdot P(X=1) & 2\cdot P(X=2) &3\cdot P(X=3) &\cdots
\end{array}\right]
$$

## Law Of The Unconscious Statistician (LOTUS)

可以看成$r.v.$函数的期望吧，反正直观上很好理解

If $X$ is a discrete $r.v.$ and $g$ is a function from $\mathbb{R}$ to $\mathbb{R}$, then
$$
E(g(x))=\sum_x g(x)\cdot P(X=x)
$$
where the sum is taken over all possible values of $X$.

## Variance and Standard Deviation

The variance of an $r.v.$ $X$ is
$$
Var(X)=E(X-EX)^2=E(X)^2-(EX)^2
$$
The square root of the variance is called the standard deviation (**SD**):
$$
SD(X)=\sqrt{Var(X)}
$$

## Properties of Variance

- For any $r.v.$ $X$ , $Var (X ) = E (X^2 )-(EX )^2$ 
- $Var (X + c) =Var (X )$ for any constant $c$
- $Var (cX ) = c^2 Var (X )$ for any constant $c$
- If $X$ and $Y$ are independent, then $Var (X + Y ) = Var (X ) + Var (Y )$
- $Var (X ) \geq0$ with equality if and only if $P(X = a) = 1$ for some constant $a$.

# Geometric and Negative Binomial

## Geometric Distribution

Consider a sequence of independent Bernoulli trials, each with the same success probability $p\in(0, 1)$, with trials performed until a success occurs. Let $X$ be the number of **failures** before the first successful trial. Then $X$ has the Geometric distribution with parameter $p$; we denote this by $X \sim Geom(p)$.

## Geometric PMF

If $X \sim Geom(p)$, then the **PMF** of $X$ is
$$
P(X=k)=q^k p
$$
for $k = 0, 1, 2,\dots$, where $q=1-p$

## Memoryless Property

If $X\sim Geom(p)$, then for any positive integer $n$,
$$
P(X\geq n+k|X\geq k)=P(X\geq n)
$$
for $k=1,2,3,\dots$

**Also,**

Suppose for any positive integer $n$, discrete random variable $X$ satisfies
$$
P(X\geq n+k|X\geq k)=P(X\geq n)
$$
for $k = 0, 1, 2,\dots$,  then $X \sim Geom(p)$.

```bash
Geometric distribution is the one and the only one discrete distribution that is memoryless.
```

## First Success Distribution

In a sequence of independent Bernoulli trials with success probability $p$, let $Y$ be the number of trials until the first successful trial, including
the success. Then $Y$ has the First Success distribution with parameter $p$; we denote this by $Y \sim FS(p)$.

### Example: Geometric & First Success Expectation

Let $X \sim Geom(p)$ and $Y \sim FS(p)$, find $E (X )$ and $E (Y )$.

**Solution:**

Calculate $E(X)$ By survival function
$$
P(X\geq k)=q^k=(1-p)^k
$$

$$
\begin{align}
E(X)&=\sum\limits_{k=0}^\infty P(X>k)\\
&=\sum_{k=1}^\infty P(x\geq k)\\
&=\sum_{k=1}^\infty q^k\\
&=\frac{q}{1-q}\\
&=\frac{1-p}{p}
\end{align}
$$

Then calculate $E(Y)$ 
$$
E(Y)=E(1+X)=1+E(X)=\frac{1}{p}
$$

## Negative Binomial Distribution

In a sequence of independent Bernoulli trials with success probability $p$, if $X$ is the number of failures before the $r^{th}$ success, then $X$ is said to have the Negative Binomial distribution with parameters $r$ and $p$, denoted $X \sim NBin(r , p)$.

## Negative Binomial PMF

If $X \sim NBin(r , p)$, then the **PMF** of $X$ is
$$
P(X=n)=\left(\begin{array}{c}n+r-1\\r-1\end{array}\right)p^rq^n
$$
for $n = 0, 1, 2,\dots$ , where $q=1-p$.

## Geometric & Negative Binomial

Let $X \sim NBin(r , p)$, viewed as the number of failures before the $r^{th}$ success in a sequence of independent Bernoulli trials with success probability $p$. Then we can write $X = X_1 + \cdots + X_r$ where the $X_i$ are *i.i.d.* $Geom(p)$.

### Example: Expectation

Let $X \sim NBin(r , p)$, find $E (X)$

**Solution:**
$$
X_i \sim Geon(p) \text{ and } E(X_i)=\frac{1-p}{p}
$$

$$
X=X_1+X_2+\cdots+X_r
$$

$$
E(X)=E(X_1)+E(X_2)+\cdots+E(X_r)=r\cdot \frac{1-p}{p}
$$

## Example: Coupon Collector

Suppose there are $n$ types of toys, which you are collecting one by one , with the goal of getting a complete set. When collecting toys, the toy types are random (as is sometimes the case, for example, with toys included in cereal boxes or included with kids’ meals from a fast food restaurant). Assume that each time you collect a toy, it is equally likely to be any of the $n$ types. Let $N$ denote the number of toys needed until you have a complete set. Find $E (N)$.

Res$\approx n[ln(n)+0.57]$

# Indicator R.V.s and The Fundamental Bridge

## Properties of Indicator R.V.

Let $A$ and $B$ be events. Then the following properties hold:

- $(I_A )^k = I_A$ for any positive integer $k$
- $I_{A^\complement} = 1- I_A$
- $I_{A\cap B}=I_A I_B$
- $I_{A\cup B}=I_A + I_B- I_A I_B$

## Fundamental Bridge Between Probability and Expectation

There is a one-to-one correspondence between events and indicator $r.v.$s, and the probability of an event $A$ is the expected value of its indicator $r.v.$ $I_ A$:
$$
E(I_A)=P(A)
$$

### Example: Booler’s Inequality

![booler](https://s1.ax1x.com/2020/10/26/BnJP2T.png)

### Example: Inclusion-Exclusion Formula

![IEF](https://s1.ax1x.com/2020/10/26/BnJixU.png)

# Moments and Indicators

## Moments of Indicator Methods

Given $n$ events $A_1 ,\cdots, A_n$ and indicators $I_j , j = 1, . . . , n$.

$X=\sum_{j=1}^n I_j$: the number of events that occur

$\left(\begin{array}{c}X\\2\end{array}\right)=\sum\limits_{i<j}I_i I_j$: the number of pairs of distinct events that occur

$E(\left(\begin{array}{c}X\\2\end{array}\right))=\sum\limits_{i<j}P(A_i\cap A_j)$

$E(X^2)=2 \sum\limits_{i<j}P(A_i\cap A_j)+ E(X)$

$Var(X)=2\sum\limits_{i<j}P(A_i\cap A_j)+E(X)-(EX)^2$

## Moments of Binomial Random Variables

![moments](https://s1.ax1x.com/2020/10/26/BnYa79.png)

# Poisson

## Poisson Distribution

An $r.v.$ $X$ has the Poisson distribution with parameter $\lambda$ if the **PMF** of $X$ is
$$
P(X=k)=\frac{e^{-\lambda}\lambda^k}{k!},\qquad k=0,1,2,\dots
$$
We write this as $X \sim Pois(\lambda )$.

### Example: E and Var

Prove that $E(X)=Var(X)=\lambda$ if $X\sim Pois(\lambda)$

## Poisson Approximation

Let $A_1 , A_2 , \cdots , A_n$ be events with $p_j = P(A_j )$, where $n$ is large, the $p_j$ are small, and the $A_j$ are independent or weakly dependent. Let
$$
X=\sum_{j=1}^n I(A_j)
$$
count how many of the $A_j$ occur. Then $X$ is approximately $Pois(\lambda)$, with $\lambda=\sum\limits_{j=1}^n p_j$.

### Example: Birthday Probelm revisited

再次考虑生日问题，利用泊松近似，则至少一对匹配成功的概率为
$$
P(X\geq 1) = 1-P(X=0)\approx1-e^{-\lambda}
$$

$$
When\,\, m=23, \lambda = \frac{253}{365},\,e^{-\lambda}\approx0.500002 
$$

## Poisson & Binomial

- Poisson $\Rightarrow$ Binomial: conditioning
- Binomial $\Rightarrow$ Poisson taking a limit

## Sum of Independent Poissons

If $X \sim Pois( \lambda_1 )$, $Y \sim Pois( \lambda_2 )$, and $X$ is independent of $Y$ , then $X + Y \sim Pois( \lambda_1 + \lambda_2 )$.

## Poisson Given A Sum of Poissons

If $X \sim Pois( \lambda_1 )$, $Y \sim Pois( \lambda_2 )$, and $X$ is independent of $Y$ , then the ***conditional distribution*** of $X$ given $X + Y = n$ is $Bin(n,\frac{\lambda_1}{\lambda_1+\lambda_2})$.

## Poisson Approximation to Binomial

If $X \sim Bin(n, p)$ and we let $n \rightarrow \infty$ and $p \rightarrow 0$ such that $\lambda= np$ remains fixed, then the **PMF** of $X$ converges to the $Pois(\lambda )$ **PMF**. More generally, the same conclusion holds if $n \rightarrow \infty$ and $p \rightarrow 0$ in such a way that np converges to a constant $\lambda$.

## Example: Visitors to A Website

The owner of a certain website is studying the distribution of the number of visitors to the site. Every day, a million people independently decide whether to visit the site, with probability $p = 2 \times 10^ 6$ of visiting. Give a good approximation for the probability of getting at least three visitors on a particular day.

Res=$1-5e^{-2}\approx0.3233$, it's easy to solve by poisson approximation.

# Distance between Two Probability Distributions

## Total Variation Distance

- Distance measure between two probability distributions

The total variation distance between two distributions $\mu$ and $\mathcal{v}$ on a countable set $\Omega$ is
$$
d_{TV}(\mu,\mathcal{v})=||\mu-\mathcal{v}||_TV=\frac{1}{2}\cdot \sum_{x\in\Omega}|\mu(x)-\mathcal{v}(x)|
$$

### Example: 

Let $\mu$ be the distribution with $\mu(1) = p$ and $\mu(0) = 1 -p$. Let $\mathcal{v}$ be a Poisson distribution with mean $p$. Then we have $d_{TV} (μ,\mathcal{v})\leq p^2$.

![distance](https://s1.ax1x.com/2020/10/26/BuE80S.png)

## The Law of Small Numbers

Given independent random variables $Y_1 , \dots , Y _n$ such that for any $1 \leq m \leq n$, $\mathbb{P}(Y_m=1)=p_m$ and $\mathbb{P}(Y_m=0)=1-p_m$. Let $S_n = Y_1 + \dots + Y_n $. Suppose
$$
\sum_{m=1}^n p_m\rightarrow \lambda\in(0,\infty) \text{ as } n\rightarrow\infty,
$$
and,
$$
\max\limits_{1\leq m\leq n} p_m\rightarrow 0 \text{ as } n\rightarrow \infty,
$$
then,
$$
d_{TV}(S_n,Poi(\lambda))\rightarrow0 \text{ as } n\rightarrow\infty.
$$

## Gap of Poisson Approximation

- A bound on the gap due to Hodges and Le Cam (1960): $d_{TV}(S_n,Poi(\lambda))\leq\sum\limits_{m=1}^n p_m^2$
- by Stein-Chen method (C.Stein 1987) we can have a tighter bound on the gap: $d_{TV}(S_n,Poi(\lambda))\leq\min(1,\frac{1}{\lambda})\sum\limits_{m=1}^np_m^2$.

# Probability Generating Functions

## Definition

The ***probability generating function*** (**PGF**) of a nonnegative integer-valued $r.v.$ $X$ with **PMF** $p_k = P(X = k)$ is the generating function
of the **PMF**. By **LOTUS**, this is
$$
E(t^X)=\sum_{k=0}^\infty p_k t^k.
$$
The **PGF** converges to a value in $[-1, 1]$ for all $t$ in $[-1, 1]$ since $\sum\limits_{k=1}^\infty p_k=1$ and $|p_kt^k|\leq p_k$ for $|t|\leq 1$.



### Example: Generating Dice Probabilities

Let $X$ be the total from rolling $6$ fair dice, and let $X_1 , \dots , X_6$ be the individual rolls. What is $P(X = 18)$?

Res=$\frac{3431}{6^6}$, it's easy to solve by probability generating function.

## PGF and Moments

Let $X$ be a nonnegative integer-valued $r.v.$ with **PMF** $p_k = P(X = k)$, and the **PGF** of $X$ is $g (t) =\sum\limits_{k=0}^\infty p_kt^k $, we have

- $E(X)=g'(t)|_{t=1}$
- $E(X(X-1))=g''(t)|_{t=1}$