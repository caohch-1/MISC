---
title: SI140 Lec6
categories:
- Probability
mathjax: true
---

# Joint Distribution

- Joint distribution provides complete information about how multiple r.v.s interact in high-dimensional space.
- Marginal distribution is the individual distribution of each r.v.
- Conditional distribution is the updated distribution for some r.v.s after observing other r.v.s

## Discrete Multivariate R.V.s

### Definition1:

The ***joint*** **CDF** of r.v.s $X$ and $Y$ is the function $F_{X,Y}$ given by 
$$
F_{X,Y}(x,y)=P(X\leq x, Y\leq y)
$$
The ***joint*** **CDF** of n r.v.s is defined analogously.

### Definition2:

The ***joint*** **PMF** of r.v.s $X$ and $Y$ is the function $f_{X,Y}$ given by 
$$
f_{X,Y}(x,y)=P(X= x, Y= y)
$$
The ***joint*** **PMF** of n discrete r.v.s is defined analogously.

### Definition3:

For discrete r.v.s $X$ and $Y$, the ***marginal*** **PMF** of $X$ is 
$$
P(X=x)=\sum_y P(X=x,Y=y),\,\,\text{can be explained by LOTP}
$$

### Definiton4:

For discrete r.v.s $X$ and $Y$, the conditional **PMF** of $X$ given $Y=y$ is
$$
P_{X|Y}(x|y)=P(X=x|Y=y)=\frac{P(X=x,Y=y)}{P(Y=y)}
$$

###  Independence of Discrete R.V.s

Random variables $X$ and $Y$ are independent if for all $x$ and $y$,
$$
F_{X,Y}(x,y)=F_X(x)F_Y(y)
$$
If $X$ and $Y$ are discrete, this is equivalent to the condition
$$
P(X=x,Y=y)=P(X=x)P(Y=y)
$$
for all $x$ and $y$, and it is also equivalent to the condition
$$
P(Y=y|X=x)=P(Y=y)
$$
for all $y$ and all $x$ such that $P(X=x)>0$.

### Example: Chicken-egg

Suppose a chicken lays a random number of eggs, $N$, where $N \sim Pois (\lambda )$. Each egg independently hatches with probability $p$ and fails to hatch with probability $q = 1-p$. Let $X$ be the number of eggs that hatch and Y the number that do not hatch, so $X + Y = N$. What is the joint PMF of $X$ and $Y$ ?

**Solution:**
$$
f_{X,Y}(i,j)=e^{-\lambda p}(\lambda p)^i\cdot e^{-\lambda p}(\lambda p)^j
$$

$$
\text{By the way, we can know that} P_X(i)\sim Pois
$$

### Related Theorem

If $X \sim Pois ( p)$, $Y \sim Pois ( q)$, and $X$ and $Y$ are independent, then $N = X + Y \sim Pois( )$ and $X|N = n \sim Bin(n, p)$.

If $N \sim Pois (\lambda)$ and $X |N = n \sim Bin(n, p)$, then $X \sim Pois (\lambda p)$, $Y = N- X \sim Pois (\lambda q)$, and $X$ and $Y$ are independent.





## Continuous Multivariate R.V.s

### Conditional PDF Given an Event

[![Dd1SBt.png](https://s3.ax1x.com/2020/11/25/Dd1SBt.png)](https://imgchr.com/i/Dd1SBt)

### Definition1:

If $X$ and $Y$ are continuous with ***joint*** **CDF** $F_{X ,Y}$ , their joint PDF is the derivative of the ***joint*** **CDF** with respect to $x$ and $y$ :
$$
f_{X,Y}=\frac{\partial^2}{\partial x\partial y}F_{X,Y}(x,y)
$$

### Definition1:

For continuous r.v.s $X$ and $Y$ with joint PDF $f_{X ,Y}$, the ***marginal*** **PDF** of $X$ is
$$
f_X(x)=\int_{-\infty}^{+\infty}f_{X,Y}(x,y)\,dy
$$
This is the PDF of $X$ , viewing $X$ individually rather than jointly with $Y$ .

### DEfinition3:

For continuous r.v.s $X$ and $Y$ with joint PDF $f_{X ,Y}$ , the ***conditional*** **PDF** of $Y$ given $X = x$ is
$$
f_{Y|X}(y|x)=\frac{f_{X,Y}(x,y)}{f_X(x)}
$$

### Technique Issue

What is the meaning of conditioning on zero-probability event $X = x$ for a continuous r.v. $X$.

We are actually conditioning on the event that $X$ falls within a small interval of $x: X \in(x -\epsilon, x + \epsilon)$ and then taking a limit as $\epsilon\rightarrow 0$.



### Continuous form of Bayes’ Rule and LOTP

For continuous r.v.s $X$ and $Y$,
$$
f_{Y|X}(y|x)=\frac{f_{X|Y}(x|y)f_Y(y)}{f_X(x)}
$$

$$
f_X(x)=\int_{-\infty}^{+\infty}f_{X|Y}(x|y)f_{Y}(y)\,dy
$$

### Bayes’ Rule: Inference Perspective

[![Dd8T7q.png](https://s3.ax1x.com/2020/11/25/Dd8T7q.png)](https://imgchr.com/i/Dd8T7q)

### Example: 

A light bulb produced by the GE company is known to have an exponential distributed lifetime $Y$. However, the company has been experiencing quality control problems. On any given day, the parameter $\lambda$ of the PDF of $Y$ is actually a random variable. uniformly distributed in the interval $[1, 3/2]$. We test a light bulb and record its lifetime. What we can say about the underlying parameter $\lambda$?

**Solution:**  $f_{\Lambda|Y}(\lambda|y)=\frac{2\cdot \lambda e^{-\lambda y}}{\int_{1}^{\frac{3}{2}}2te^{-ty}\,dt}$

### General Bayes' Rule

![D6rzwj.png](https://s3.ax1x.com/2020/11/28/D6rzwj.png)

### General LOTP

![D6rxmQ.png](https://s3.ax1x.com/2020/11/28/D6rxmQ.png)

### Example:

A binary signal $S$ is transmitted, and we are given that $P(S = 1) = p$ and $P(S = 1) = 1-p$. The received signal is $Y = N + S$, where $N$ is normal noise, with zero mean and unit variance, independent of $S$. What is the probability that $S = 1$, as a function of the observed value $y$ of $Y$ ?

**Solution:** $P(S=1|Y=y)=\frac{pe^y}{pe^y+(1-p)e^{-y}}$

### Example: Comparing Exponentials of Different Rates

Let $T_1 \sim Expo( \lambda_1 )$, $T_2 \sim Expo( \lambda_2 )$, $T_1$ and $T_2$ are independent. Find $P(T_1 < T_2 )$.

**Solution:**  $\frac{\lambda_1}{\lambda_1+\lambda_2}$

### Example: Which Company Made the Lightbulb?

[![DdNKsg.png](https://s3.ax1x.com/2020/11/25/DdNKsg.png)](https://imgchr.com/i/DdNKsg)

**Solution:** $a):F_T(t)=1-p_0e^{-\lambda_0t}-p_1e^{-\lambda_1 t}$, $f_T(t)=p_0\lambda_0e^{-\lambda_0t}+p_1\lambda_1 e^{-\lambda_1t}$  $c):P(I=1|T=t)=\frac{p_1\lambda_1}{p_0\lambda_0e^{(\lambda_1-\lambda_0)t}+p_1\lambda_1}.\,\,\text{When }t\rightarrow\infty,\,t\rightarrow0.$



### Independence of Continuous R.V.s

Random variables $X$ and $Y$ are independent if for all $x$ and $y$,
$$
F_{X,Y}(x,y)=F_X(x)F_Y(y)
$$
If $X$ and $Y$ are continuous with joint PDF $f_{X ,Y}$ , this is equivalent to the condition
$$
f_{X,Y}(x,y)=f_X(x)f_Y(y)
$$
for all $x$ and $y$, and it is also equivalent to the condition
$$
f_{Y|X}(y|x)=f_T(y)
$$
for all $y$ and all $x$ such that $f_X (x) > 0$.



### Proposition

Suppose that the ***joint*** **PDF** $f_{X ,Y}$ of $X$ and $Y$ factors as 
$$
f_{X ,Y} (x, y ) = g (x) h (y )
$$
for all $x$ and $y$ , where $g$ and $h$ are nonnegative functions. Then $X$ and $Y$ are independent. Also, if either $g$ or $h$ is a valid **PDF**, then the other one is a valid **PDF** too and $g$ and $h$ are the marginal **PDFs** of $X$ and $Y$ , respectively. (The analogous result in the discrete case also holds.)

### 2D LOTUS

Let $g$ be a function from $\mathbb{R}^2$ to $\mathbb{R}$. If $X$ and $Y$ are discrete, then 
$$
E(g(X,Y))=\sum_x\sum_yg(x,y)P(X=x,Y=y)
$$
If $X$ and $Y$ are continuous with joint **PDF** $f_{X,Y}$, then
$$
E(g(X,Y))=\int_{-\infty}^{+\infty}\int_{-\infty}^{+\infty}g(x,y)f_{X,Y}(x,y)\,dxdy
$$

### Example: Expected Distance between Two Uniforms

For $X,Y\sim^{iid}Unif(0,1)$, find $E(|X-Y|)$, $E(max(X,Y))$, and $E(min(X,Y))$.

Res: $\frac{1}{3},\,\frac{2}{3},\,\frac{1}{3}$

### Example: Expected Distance between Two Normals

For $X,Y\sim^{iid} \mathcal{N}(0,1)$, find $E(|X-Y|)$.

Res: $\frac{2}{\sqrt{\pi}}$

## Covariance and Correlation

### Definition1: 

The ***covariance*** between $r.v.s$ $X$ and $Y$ is
$$
Cov(X,Y)=E((X-EX)(Y-EY)).
$$
Multiplying this out and using linearity, we have an equivalent expression:
$$
Cov(X,Y)=E(XY)-E(X)E(Y).
$$

### Key Properties of Covariance

[![D6URbt.png](https://s3.ax1x.com/2020/11/28/D6URbt.png)](https://imgchr.com/i/D6URbt)



### Definition2:

The correlation between $r.v.s$ $X$ and $Y$ is 
$$
Corr(X,Y)=\frac{Cov(X,Y)}{\sqrt{Var(X)Var(Y)}}
$$
(This is undefined in the degenerate cases $Var(X)=0$ or $Var(Y)= 0$.)

### Definition3:

Given $r.v.s$ $X$ and $Y$, if $Cov(X,Y)=0$ or $Corr(X,Y)=0$, $X$ and $Y$ are uncorrelated.

### Uncorrelated Theorem

If $X$ and $Y$ are independent, then they are uncorrelated.

### Covariance & Correlation

- Measure a tendency of two $r.v.s$ $X$ & $Y$ to go up or down together
- Positive covariance (Correlation): when $X$ goes up, $Y$ also tends to go up
- Negative covariance (Correlation): when $X$ goes up, $Y$ tends to go down
  

### Correlation Bounds Theorem

For any $r.v.s$ $X$ and $Y$,
$$
-1\leq Corr(X,Y)\leq 1
$$

### Example: Exponential Max and Min

Let $X$ and $Y$ be $i.i.d.$ $Expo(1)$ $r.v.s$. Find the correlation between $\max(X , Y )$ and $\min(X , Y )$.

Res: $\frac{1}{\sqrt{5}}$

## Multinomial Distribution

### Story

Each of $n$ objects is independently placed into one of $k$ categories. An object is placed into category $j$ with probability $p_j$ , where the $p_j$ are nonnegative and $\sum_{j=1}^k p_j = 1$. Let $X_1$ be the number of objects in category 1, $X_2$ the number of objects in category 2, etc., so that $X_1 + \dots + X_k = n$. Then $X = (X_1 , \dots , X_k )$ is said to have the **Multinomial distribution** with parameters $n$ and $p = (p_1 , \dots , p_k )$. We write this as $X \sim Mult_k (n, p)$.

### Multinomial Joint PMF

If $X\sim Mult_k(n,p)$, then the ***joint*** **PMF** of $X$ is 
$$
P(X_1=n_1,\dots, X_k=n_k)=\frac{n!}{n_1!n_2!\dots n_k!}\cdots p_1^{n_1}p_2^{n_2}\cdots p_k^{n_k}
$$
for $n_1,\dots,n_k$ satisfying $n_1+\cdots+n_k = n$.

### Multinomial Marginals Theorem

If $X \sim Mult_k (n, p)$, then $X_j \sim Bin (n, p_j )$.

### Multinomial Lumping Theorem

If $X \sim Mult_k (n, p)$, then for any distinct $i$ and $j$, $X_i + X_j \sim Bin(n, p_i + p_j )$. The random vector of counts obtained from merging categories $i$ and $j$ is still ***Multinomial***. For example, merging categories $1$ and $2$ gives
$$
(X_1+X_2,X_3,\dots,X_k)\sim Mult_{k-1}(n,(p_1+p_2,p_3,\dots,p_n))
$$

### Multinomial Conditioning Theorem

If $X\sim Mult_k(n,p)$, then 
$$
(X_2,\dots,X_k)|X_1=n_1\sim Mult_{k-1}(n-n_1,(p_2',\dots,p_k')),
$$
where $p_j'=p_j/(p_2+\cdots+p_k)$.

### Covariance in A Multinomial Theorem

Let $(X_1,\dots,X_k)\sim Mult_k(n,p)$, where $p=(p_1,\dots,p_k)$. For $i\neq j$, $Cov(X_i, X_j)= -np_i p_j$.

## Multivariate Normal

### Definition1:

A random vector $X = (X_1 , \dots, X_k )$ is said to have a ***Multivariate Normal*** (MVN) distribution if every linear combination of the $X_j$ has
a Normal distribution. That is, we require
$$
t_1X_!+\dots+t_kX_k
$$
to have a Normal distribution for any choice of constants $t_1 , \dots, t_k$. If $t_1 X_1 + \dots + t_k X_k$ is a constant (such as when all $t_i = 0$), we consider it to have a Normal distribution, albeit a degenerate Normal with variance 0. An important special case is $k = 2$; this distribution is called the Bivariate Normal (BVN).

### Theorem1:

If $(X_1 , X_2 , X_3 )$ is Multivariate Normal, then so is the subvector $(X_1 , X_2 )$.

### Theorem2:

If $X = (X_1 , \dots, X_n )$ and $Y = (Y_1 , \dots, Y_m )$ are MVN vectors with $X$ independent of $Y$, then the concatenated random vector $W = (X_1 , \dots, X_n , Y_1 , \dots, Y_m )$ is Multivariate Normal.

### Parameters of MVN

Parameters of an MVN random vector $(X_1 , \dots , X_k )$ are:

- the mean vector $(μ_1 , \dots , μ_k )$, where $E (X_j ) = μ_j$ .
- the covariance matrix, which is the $k \times k$ matrix of covariance between components, arranged so that the row $i$, column $j$ entry is $Cov (X_i , X_j )$.

### Joint MGF

The joint MGF of a random vector $X = (X_1 , \dots, X_k )$ is the function which takes a vector of constants $t = (t_1 , \dots, t_k )$ and returns
$$
M(t)=E(e^{t'X})=E(e^{t_1X_1+\dots+t_kX_k})
$$
We require this expectation to be finite in a box around the origin in $\mathbb{R}^k$; otherwise we say the joint MGF does not exist.

### Theorem3:

Within an MVN random vector, uncorrelated implies independent. That is, if $X \sim MVN$ can be written as $X = (X_1 , X_2 )$, where $X_1$ and $X_2$ are subvectors, and every component of $X_1$ is uncorrelated with every component of $X_2$ , then $X_1$ and $X_2$ are independent. In particular, if $(X , Y )$ is ***Bivariate Normal*** and $Corr(X , Y ) = 0$, then $X$ and $Y$ are independent.

### Example: Independence of Sum and Difference

Let $X , Y \sim N (0, 1)$. Find the joint distribution of $(X + Y , X-Y)$.

**Res:** $f_{\mathcal{N}(0,2)}\cdot f_{\mathcal{N}(0,2)}$

### Example: Independence of Sample Mean and Sample Variance

Let $X_1,\dots, X_n$ be i.i.d. $\mathcal{N}(\mu, \sigma^2)$, with $n\geq 2$. Define
$$
\bar{X_n}=\frac{1}{n}(X_1+\cdots+X_n),
$$

$$
S_n^2=\frac{1}{n-1}\sum_{j=1}^n (X_j-\bar{X_n})^2.
$$

The sample mean $\bar{X_n}$ has expectation $\mu$ (the true mean) and the sample variance $S_n^2$ has expectation $\sigma^2$ (the true variance). Show that $\bar{X_n}$  and $S_n^2$ are independent by applying MVN ideas and results to the vector $\bar{X}_n , X_1- \bar{X}_n , \dots, X_n- \bar{X}_n .$

### Summary

![DgO5He.png](https://s3.ax1x.com/2020/11/30/DgO5He.png)