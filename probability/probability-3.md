---
title: SI140 Lec3
categories:
- Probability
mathjax: true
---

# Random Variables

## Definition

A function from the **sample space** $S$ to the **real numbers** $R$.

## Probability Mass Function (PMF)

### Definition

The probability mass function (PMF) of a discrete r.v. $X$ is the function $p_X$ given by $p_X(x)=P(X=x)$.

Actually, 
$$
\begin{align}
P(X=x)&=P(s:X(s)=x,s\in S)\\&=\sum_{\quad s\in S\\S:X(s)=x}P(s)
\end{align}
$$

### Two criteria

- Nonnegative:$p_X(x)>0$ if $x=x_j$ for some $j$, and $p_X(x)=0$ otherwise;
- Sums to $1$: $\sum\limits_{j=1}^\infty p_X(x_j)=1$

# Bernoulli and Binomial

## Definition of Bernoulli Distribution

An r.v. $X$ is said to have the *Bernoulli distribution* with parameter $p$ if $P(X=1)=p$ and $p(X=0)=1-p$, where $0<p<1$. We write this as $X\sim Bern(p)$. 

## Definition of Indicator Random Varible

The *indicator random varible* of an event $A$ is th r.v. which equals $1$ if $A$ occurs and $0$ otherwise. We will denote the indicator r.v. of $A$ by $I_A$ or $I(A)$.

## Definition of Binomial Distribution

Suppose that $n$ independent Bernoulli trials are performed, each with the same success probability $p$. Let $X$ be the number od success. THe distributuon of $X$ is called the *BInomial distribution* with parameters $n$ and $p$. We write $X\sim Bin(n,p)$ to mean that $X$ has the Binomial distribution with parameters $n$ and $p$, where $n$ is a postive integer and $0<p<1$.

## Binomial PMF

### Theorem 1

If $X\sim Bin(n,p)$, then the PMF of $X$ is
$$
P(X=k)=\left(\begin{array}{cc} n\\k \end{array}\right)p^k(1-p)^{n-k}
$$
for $k=0,1,\cdots,n$ (and $P(X=k)=0$ otherwise).

### Theorem 2

Let $X\sim Bin(n,p)$, and $q=1-p$ (we often use $q$ to denote the failure of a *Bernouli trial*). Then $n-X\sim Bin(n,q)$.

## Example

### Statistical Multiplexing

There are $N$ senders using one multiplexor, the shared medium is $1$Mbps and each sender requests $100$kbps and their active probability is $0.1$.

What's $N_{max}$?

Res:$35$

### Multiple Access (Aloha Protocol)

- $N$ smart devices sharing a WiFi access point

- $\geq 2$ devices transmit simultaneously lead to collision

- Each device transmits with probability $p$ independently

- What's the transmission rate (the number of successful transmissions per unit time) ?

  Res:$e^{-1}=0.36$

# Hypergeometric

## Definition

Consider an urn with $w$ white balls and $b$ black balls. We draw $n$ balls out of the urn at random without replacement, such that all $\left(\begin{array}{c} w+b\\n \end{array}\right)$ samples are equally likely. Let $X$ be the number of white balls in the sample. Then $X$ is said to have the *Hypergeometric distribution* with parameters $w,b$, and $n$; we denote this by $X\sim HGeom(w,b,n)$.

## Hypergeometric PMF

If $X\sim HGeom(w,b,n)$, then the PMF of $X$ is
$$
P(X=k)=\frac{\left(\begin{array}{c} w\\k \end{array}\right)\left(\begin{array}{c} b\\n-k \end{array}\right)}{\left(\begin{array}{c} w+b\\n \end{array}\right)}
$$
for integers $k$ satisfying $0\leq k\leq w$ and $0\leq n-k\leq b$, and $P(X=k)=0$ otherwise.

## Identical Distribution

The $HGeom(w,b,n)$ and $HGeom(n,w+b-n,w)$ distributions are identical. That is, if $X\sim HGeom(w,b,n)$ and $Y\sim HGeom(n,w+b-n,w)$, then $X$ and $Y$ have the same distribution.

An simply explaination:

超几何分布的适用基础是总体根据两套标签进行分类。在球的例子里，对于 r.v. $X$ 而言，白黑是第一套标签，是否抓到作为样本是第二套标签；对于 $Y$ 则反过来。故而事实上 $X$ 和 $Y$ 都表示被抽出来的白球数量，故而分布相同。我们也可以用麻烦的代数方法验证。



# Discrete Uniform & Zipf Distribution

```python
pass
```



# Cumulative Distribution Functions (CDF)

## Definition

The *cumulative distribution function (CDF)* of an r.v. $X$ is the function $F_X$ given by $F_X(x)=P(X\leq x)$.

# Functions of Random Variables: Random Variables

## Definition

For an experiment with sample space $S$, an r.v. $X$, and a function $g:\mathbb{R}\rightarrow\mathbb{R}$, $g(X)$ is the r.v. that maps $s$ to $g(X(s))$ for all $s\in S$.

## Sympathetic Magic

GIven an r.v. $X$, trying to get the PMF of $2X$ by multiply the PMF of $X$ by 2:

- Wrong: $Y=2X\Rightarrow P(Y=y)=2P(X=y)$
- Right: $P(Y=y)=P(2X=y)=P(X=\frac{y}{2})$

# Independence of R.V.s

## Definition

Random variables $X$ and $Y$ are said to be *independent* if
$$
P(X\leq x, Y\leq y)=P(X\leq x)P(Y\leq y),
$$
for all $x,y \in \mathbb{R}$.

## I.I.D

We will often work with random variables that are independent and have the same distribution. We call such r.v.s independent and identically distributed, or i.i.d for short.

## Example

![bayesian](https://s1.ax1x.com/2020/10/19/0v5jlF.png)



# Binomial & Hypergeometric

## Connection 1

- Binomial $\Rightarrow$ Hypergeometric: **conditioning**
- Hypergeometric $\Rightarrow$ Binomial: **taking a limit**

## Connection 2

If $X\sim Bin(n,p)$, $Y\sim Bin(m,p)$, and $X$ is independent of $Y$, then the conditional distribution of $X$ given $X+Y=r$ is $HGeom(n,n,r)$.

## Connection 3

If $X\sim HGeom(w,b,n)$ and $N=w+b\rightarrow\infty$ such that $p=w/(w+b)$ remains fixed, then the PMF of $X$ converges to the $Bin(n,p)$ PMF.

Because when $N\rightarrow\infty$, whether with replacement become meaningless.



# Information Theory & Entropy

## Definition of Entropy

Given a random variable $X$ with a probability mass function $p(x)$ and a support $\mathcal{X}$. The entropy of $X$ is defined by
$$
H(X)=-\sum_{x\in\mathcal{X}}p(x)log_2p(x)
$$

## Example

One of 13 coins is fake. At least how many times to weigh can we know which of them is fake? (We can get a result $\in \{=,<,>\}$  every time we weigh, but we don't know whether the fake coin is lighter or heavier).

### Res

- 每次称重的有三个结果，故而每次提供$log_2 3$ bits 信息
- 假设要至少需要$n$次，则我们将获得$nlog_2 3$ bits 信息
- 总共13个硬币，故而需要$log_2 13$ bits 信息
- 假硬币可能重可能轻，故而需要$log_2 2$ bits 信息

故而列出下列不等式，求解
$$
\begin{align}
log_2 13+log_2 2&\leq nlog_2 3\\
log_2 26&\leq log_2 3^n\\
3^n&\geq 26\\
n&\geq 3
\end{align}
$$














