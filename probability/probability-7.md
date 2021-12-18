---
title: SI140 Lec7
categories:
- Probability
mathjax: true

---

# Transformations

## Change of Variables

### Theroem1: Change of Variables in One Dimension

Let $X$ be a continuous $r.v.$ with PDF $f_X$ , and let $Y = g(X)$, where $g$ is differentiable and strictly increasing (or strictly decreasing). Then the PDF of $Y$ is given by
$$
f_Y(y) = f_X(x)|\frac{dx}{dy}|,
$$
where  $x=g^{-1}(y).$ The  support of $Y $ is all $g(x)$ with $x$ in the support of $X$. 

### Theorem2: Change of Variables

Let $X=(X_1, \dots, X_n)$ be a continuous random vector with joint PDF $f_X(x)$, and let $Y = g(X)$ where $g$ is an invertible function from $\mathbb{R}^n$ to $\mathbb{R}^n$. Let $y = g(x)$ and suppose that all the partial derivatives $\frac{\partial x_i}{\partial y_i}$ exists and are continuous, so we can form the Jacobian matrix
$$
\frac{\partial \mathbf{x}}{\partial \mathbf{y}}=\left(\begin{array}{cccc}
\frac{\partial x_{1}}{\partial y_{1}} & \frac{\partial x_{1}}{\partial y_{2}} & \cdots & \frac{\partial x_{1}}{\partial y_{n}} \\
\vdots & \vdots & & \vdots \\
\frac{\partial x_{n}}{\partial y_{1}} & \frac{\partial x_{n}}{\partial y_{2}} & \cdots & \frac{\partial x_{n}}{\partial y_{n}}
\end{array}\right)
$$
Also assume that the determinant of the Jacobian matrix is never $0$. Then the joint PDF of $Y$ is
$$
f_{Y}(y)=f_{X}(x)\left|\frac{\partial x}{\partial y}\right|
$$

### Jacobian or not

- For discrete $r.v.s$: No Jacobian
- For continuous $r.v.s$: Jacobian Yes

### Box-Muller

Let $U \sim \operatorname{Unif}(0,2\pi),$ and let $T \sim \operatorname{Expo}(1)$ be independent of $U$ Define $X=\sqrt{2 T} \cos U$ and $Y=\sqrt{2 T} \sin U .$ Find the joint $\mathrm{PDF}$ of $(X, Y) .$ Are they independent? What are their marginal distributions?
$$
F_{X,Y}(x,y)=\frac{1}{\sqrt{2\pi}}e^{-\frac{x^2}{2}}\frac{1}{\sqrt{2\pi}}e^{-\frac{y^2}{2}}
$$

$$
\text{Also, it's easy to oberserve that they are independent}
$$

## Convolutions

### Theorem1: Convolution Sums and Integrals

If $X$ and $Y$ are independent discrete $r . v . s,$ then the $P M F$ of their sum $T=X+Y$ is
$$
\begin{aligned}
P(T=t) &=\sum_{x} P(Y=t-x) P(X=x) \\
&=\sum_{y} P(X=t-y) P(Y=y)
\end{aligned}
$$
If $X$ and $Y$ are independent continuous $r . v . s,$ then the $P D F$ of their sum $T=X+Y$ is
$$
\begin{aligned}
f_{T}(t) &=\int_{-\infty}^{\infty} f_{Y}(t-x) f_{X}(x) d x \\
&=\int_{-\infty}^{\infty} f_{X}(t-y) f_{Y}(y) d y
\end{aligned}
$$

### Example: Exponential Convolution

$\text { Let } X, Y \stackrel{\text { i.i.d. }}{\sim} \operatorname{Expo}(\lambda) \text { . Find the distribution of } T=X+Y \text { . }$

**Res:** $f_T(t)=\lambda^2 t e^{-\lambda t}$



## Order Statistics

### Definition1:

For r.v.s $X_{1}, X_{2}, \ldots, X_{n}$, the order statistics are the random variables $X_{(1)}, \ldots, X_{(n)},$ where
$$
X_{(1)}=\min \left(X_{1}, \ldots, X_{n}\right)
$$
$$
X_{(2)} \text{ is the second-smallest of } X_{1}, \ldots, X_{n}
$$

$$
\vdots
$$

$$
X_{(n-1)} \text{ is the second-largest of } X_{1}, \ldots, X_{n}
$$

$$
X_{(n)}=\max \left(X_{1}, \ldots, X_{n}\right)
$$
Note that $X_{(1)} \leq X_{(2)} \leq \ldots \leq X_{(n)}$ by definition. We call $X_{(j)}$ the $j^{th}$  order statistic.



### CDF of Order Statistics

Let $X_{1}, \ldots, X_{n}$ be i.i.d. continuous $r . v . s$ with $C D F F .$ Then the CDF of the $j^{th}$ order statistic $X_{(j)}$ is
$$
P\left(X_{(j)} \leq x\right)=\sum_{k=j}^{n}\left(\begin{array}{c}
n \\
k
\end{array}\right) F(x)^{k}(1-F(x))^{n-k}
$$

### PDF of Order Statistic

Let $X_{1}, \ldots, X_{n}$ be $i.i.d.$ continuous $r . v . s$ with **CDF** $F$ and **PDF** $f$ . Then the marginal PDF of the $j^{th}$ order statistic $X_{(j)}$ is
$$
f_{X_{(j)}}(x)=n\left(\begin{array}{c}
n-1 \\
j-1
\end{array}\right) f(x) F(x)^{j-1}(1-F(x))^{n-j}
$$

### Joint PDF

Let $X_{1}, \ldots, X_{n}$ be $i.i.d.$ continuous r.v.s with **PDF** $f$. Then the joint **PDF** of all order statistics is
$$
f_{X_{(1)}, \ldots, X_{(n)}}\left(x_{1}, \ldots, x_{n}\right)=n ! \prod_{i=1}^{n} f\left(x_{i}\right), x_{1}<x_{2}<\cdots<x_{n}
$$

### Example: Order Statistics of Uniforms

$U_1,\dots,U_n\sim^{i.i.d} Unif(0,1)$, find order statistics

**Res:** $\int_0^x\frac{n!}{(j-1)!(n-j)!}t^{j-1}(1-t)^{n-j}\,dt$

### Related Identity

For $0<p<1,$ and nonnegative integer $k,$ we have
$$
\sum_{j=0}^{k}\left(\begin{array}{c}
n \\
j
\end{array}\right) p^{j}(1-p)^{n-j}=\frac{n !}{k !(n-k-1) !} \int_{p}^{1} x^{k}(1-x)^{n-k-1} d x
$$


## Beta-Binomial Conjugacy

### Definition1: 

An $r.v.$ $X$ is said to have the Beta distribution with parameters a and $b$, $a>0$ and $b>0,$ if its  **PDF** is
$$
f(x)=\frac{1}{\beta(a, b)} x^{a-1}(1-x)^{b-1}, 0<x<1
$$
where the constant $\beta(a, b)$ is chosen to make the PDF integrate to $1 .$ We write this as $X \sim \operatorname{Beta}(a, b)$. Beta distribution is a generalization of uniform distribution.



### Story: Bayes’ billiards

Show without using calculus that for any integers $k$ and $n$ with $0 \leq k \leq n$,
$$
\int_{0}^{1}\left(\begin{array}{l}
n \\
k
\end{array}\right) x^{k}(1-x)^{n-k} d x=\frac{1}{n+1}
$$
![DWDrX6.png](https://s3.ax1x.com/2020/12/01/DWDrX6.png)



![DWDD6x.png](https://s3.ax1x.com/2020/12/01/DWDD6x.png)



### Story: Beta Binomial Conjugacy

We have a coin that lands Heads with probability $p$, but we don't know what $p$ is. Our goal is to infer the value of $p$ after observing the outcomes of $n$ tosses of the coin. The larger that $n$ is, the more accurately we should be able to estimate $p$.



#### Bayesian Inference

- Treats all unknown quantities as random variables.
- In the Bayesian approach, we would treat the unknown probability $p$ as a random variable and give $p$ a distribution.
- This is called a **prior distribution**, and it reflects our uncertainty about the true value of $p$ before observing the coin tosses.
- After the experiment is performed and the data are gathered, the prior distribution is updated using Bayes’ rule; this yields the **posterior distribution**, which reflects our new beliefs about $p$.

#### Result

If win $k$ times of $n$ trails, we suppose $p\sim Beta(a, b)$ at first, then we will get the answer that $p\sim Beta(a+k, b+n-k)$

#### Furthermore

- Furthermore, notice the very simple formula for updating the distribution of $p$.

- We just add the number of observed successes, $k$, to the first parameter of the Beta distribution.

- We also add the number of observed failures, $n - k$, to the second parameter of the Beta distribution.

- So $a$ and $b$ have a concrete interpretation in this context:
  $$
  \left\{
  \begin{array}[l]
  &a\text{ as the number of prior successes in earlier experiments}\\
  b\text{ as the number of prior failures in earlier experiments}\\
  a,b:\text{ pseudo counts}
  \end{array}
  \right.
  $$

- 





#### Mean vs. Bayesian Average

- Infer the value of $p$ (probability of coin lands heads)
- Observed $k$ heads out of $n$ tosses of the coin
- Mean: $\frac{k}{n}$
- Bayesian Average: $E(p|X=k)=\frac{a+k}{a+b+n}$
- Suppose the prior distribution is Unif(0,1): $a=1$, $b=1$
- Bayesian Average: $\frac{k+1}{n+2}$
- When $k=n$, we have : 1(mean) vs. $\frac{n+1}{n+2}$(Bayesian Average)



#### Conclusion

If we have a Beta prior distribution on $p$ and data that are conditionally Binomial given $p$, then when going from prior to posterior, we don’t leave the family of Beta distributions. We say that **the Beta is the conjugate prior of the Binomial**.