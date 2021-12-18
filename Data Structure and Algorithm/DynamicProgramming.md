---
title: Dynamic Programming
categories:
- 算法
mathjax: true

---

***基于《算法导论》中的伪代码 以及部分网络资源***



# 钢条切割

| 长度 i  | 0    | 1    | 2    | 3    | 4    | 5    | 6    | 7    | 8    | 9    | 10   |
| ------- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| 价格 Pi | 0    | 1    | 5    | 8    | 9    | 10   | 17   | 17   | 20   | 24   | 30   |



## Problem: 

​	给定一段长度为**n**英寸的钢条和一个价格表Pi，求切割钢条方案，使得销售收益Rn最大。注意，如果长度为**n**英寸的钢条的价格Pn足够大，最优解可能就是完全不需要切割。



## Solution:

### Method1: Noramal

如果一个最优解将钢条切割成**k**段，那么最优切割方案
$$
n=i_1+i_2+...+i_k
$$
得到最大收益
$$
R_n=P_{i_1}+P_{i_2}+...+P_{i_k}
$$
**更一般的**，对于Rn，我们可以用更短的钢条的最优切割收益来描述
$$
R_n=max (p_n,r_1+r_{n-1},r_2+r_{n-2},...,r_{n-1}+r_1)
$$
第一个参数Pn对应不切割，后面的是所有切割方案对应的收益



### Method2: Recursion

与Method1相似但是更简单的递归求解方法

从左边割下长度**i**的一段，对右边剩下**n-i**继续切割，左边不操作

用一个式子更加清晰地说明
$$
R_n=\mathop{max}\limits_{1\leq i \leq n}(P_i+R_{n-i})
$$

```c++
int CUT_ROD(int *price, int length) {
    if (length == 0) {
        return 0;
    }
    int q = -1;

    for (int i = 1; i <= length; i++) {
        q = max(q, price[i] + CUT_ROD(price, length - i));
    }

    return q;
}
```



### Method3: Dynmaic Programming

#### Top-down with memoization

在过程中保存每个子问题的解 从而花费空间节省时间

**memory**就是已知解的数组

剩下与Method2中相似

```c++
int MEMOIZED_CUT_ROD(int *price, int length) {
    int memory[length + 1];
    for (int i = 0; i <= length; ++i) {
        memory[i] = -1;
    }
    return MEMOIZED_CUT_ROD_AUX(price, length, memory);
}

int MEMOIZED_CUT_ROD_AUX(int *price, int length, int *memory) {
    if (memory[length] >= 0) {
        return memory[length];
    }

    int q;

    if (length == 0) {
        q = 0;
    } else {
        q = -1;
        for (int i = 1; i <= length; ++i) {
            q = max(q, price[i] + MEMOIZED_CUT_ROD_AUX(price, length - i, memory));
        }
    }
    memory[length] = q;
    return q;
}
```



#### Bottom-up

由于没有递归 该方法的事件复杂性函数通常具有更小的系数

```c++
int BOTTOM_UP_CUT_ROD(int *price, int length) {
    int memory[length + 1];
    int q;
    memory[0] = 0;
    for (int j = 1; j <= length; ++j) {
        q = -1;
        for (int i = 1; i <= j; ++i) {
            //和Method2比较下 这里由于有了memory 所以不需要递归了
            q = max(q, price[i] + memory[j - i]);
        }
        memory[j] = q;
    }
    return memory[length];
}
```



## 重构解

如果还要出第一段长度 我们可以扩展以上算法

对于每个子问题不仅保存最优收益 还保存对应切割方案

```c++
//用来返回两个数组 一个收益 一个切割方案
struct Result {
    int *revenue;
    int *solution;

    Result(int *p, int *s) {
        revenue = p;
        solution = s;
    }
};

//主要处理过程
Result EXTENDED_BOTTOM_UP_ROD(int *price, int length) {
    //record revenue
    int memory_r[length + 1];
    memory_r[0] = 0;
    //record cut-solution
    int memory_s[length + 1];
    int q;

    for (int j = 1; j <= length; ++j) {
        q = -1;
        for (int i = 1; i <= j; ++i) {
            if (q < price[i] + memory_r[j - i]) {
                q = price[i] + memory_r[j - i];
                memory_s[j] = i;
            }
        }
        memory_r[j] = q;
    }

    return Result(memory_r, memory_s);
}

//打表
void PRINT_CUT_ROD_SOLUTION(int *price, int length) {
    Result res = EXTENDED_BOTTOM_UP_ROD(price, length);
    int cut1 = res.solution[length];
    int cut2 = length - cut1;
    int revenue = res.revenue[length];
    cout << "Cut solution: " << cut1 << "--" << cut2 << endl;
    cout << "Revenue: " << revenue << endl;
}
```







# 矩阵链乘法

## Problem:

给定n个矩阵的链<$A_1, A_2, ..., A_n$>

矩阵$A_i$的规模为$p_{i-1}\times p_i (1\leq i \leq n)$

求完全括号化方案，使得计算乘积$A_1 A_2...A_n$所需标量乘法次数最少.

i.e.

<$A_1, A_2, A_3$> 其中规模分别为$10\times100，100\times5，5\times50$



如果按照 $((A_1 A_2)A_3)$ 

 $10\times100\times5=5000$ -->

 $10\times5\times50=2500$   -->

 $5000+2500=7500$



如果按照 $(A_1(A_2 A_3))$ 

$100\times5\times50=25000$    -->

 $10\times100\times50=50000$ -->

 $50000+25000=75000$

 

## Solution:

### 否定穷举:

看看穷举的递归方程
$$
P(n)=\left\{
\begin{array}{rcl}
1\qquad\quad&如果n=1\\
\sum\limits_{k=1}^{n-1} P(k)P(n-k)&如果n\leq2\\
\end{array}\right.
$$


他的结果是$\Omega(2^n)$，好哥哥你应该也知道这很糟糕



### 应用动态规划方案:

#### Step1: 最优括号化方案的结构特征

很容易发现可以把问题划分为两个子问题

$A_i A_{i+1}...A_k$和$A_{k+1} A_{k+2}...A_j$的最优化括号问题 分别求出子问题的最优解 再结合起来



#### Step2: 一个递归求解方案

我们用**m[i, j]**表示计算矩阵$A_{i..j}$所需标量乘法次数的最小值

那么原问题最优解可以表示为**m[1, n]**

容易得到下面的式子  ( $A_i对应矩阵大小是 p_{i-1}\times p_i$ )
$$
m[i,j]=\left\{
\begin{array}{rcl}
0\qquad\qquad\qquad\qquad&如果i=j\\
\mathop{min}\limits_{i\leq k \leq j} \{m[i,k]+m[k+1,j]+p_{i-1}p_k p_j\}&如果i<j\\
\end{array}\right.
$$

**m[i, j]**的值给出了子问题最优解的代价，但还需要更多信息来构造最优解

为此用**s[i, j]**保存$A_i A_{i+1}...A_j$最优括号化方案的分割点位置k，即使得$m[i,j]=m[i,k]+m[k+1,j]+p_{i-1} p_k p_j$成立的k值



#### Step3: 计算最优代价

先来看看代码

```c++
/*
p是矩阵链 其长度是矩阵个数+1;
length是矩阵个数;
m用于存放每个已解子问题的最优值;
s用于存放m的对应切割点
Note:与算法导论以及许多网络资源上的代码不同 我在此处全部采用从0开始的下标 而非从1
*/
void MATRIX_CHAIN_ORDER(int *p, int length, std::vector<std::vector<int>> &m, std::vector<std::vector<int>> &s) {
    //链的长度为1时代价是0 故而对角线元素变成0
    for (int i = 0; i < length; ++i) {
        m[i][i] = 0;
    }

    int j;
    int q;
    for (int l = 2; l <= length; ++l) { //l是指链的长度
        for (int i = 0; i <= length - l; ++i) {
            j = i + l - 1;
            m[i][j] = 100000000;
            for (int k = i; k <= j - 1; ++k) {
                //计算代价 如果小 就替换
                q = m[i][k] + m[k + 1][j] + p[i] * p[k + 1] * p[j + 1];
                if (q < m[i][j]) {
                    m[i][j] = q;
                    s[i][j-1] = k;
                }

            }

        }

    }
}
```



举个例子说明这段代码

**Note:与算法导论以及许多网络资源上的代码不同 我在此处全部采用从0开始的下标 而非从1**

| Matrix | A0           | A1           | A2          | A3          | A4           | A5           |
| ------ | ------------ | ------------ | ----------- | ----------- | ------------ | ------------ |
| Size   | $30\times35$ | $35\times15$ | $15\times5$ | $5\times10$ | $10\times20$ | $20\times25$ |

**故 p={30, 35, 15, 5, 10, 20, 25}; length=6**

经过上述算法，我们获得

**表格m:** 

| $ j\downarrow \qquad;\qquad i\to$ |   0   |   1   |  2   |  3   |  4   |  5   |
| :-------------------------------: | :---: | :---: | :--: | :--: | :--: | :--: |
|                 0                 |   0   | None  | None | None | None | None |
|                 1                 | 15750 |   0   | None | None | None | None |
|                 2                 | 7875  | 2625  |  0   | None | None | None |
|                 3                 | 9375  | 4375  | 750  |  0   | None | None |
|                 4                 | 11875 | 7125  | 2500 | 1000 |  0   | None |
|                 5                 | 15125 | 10500 | 5375 | 3500 | 5000 |  0   |

 

**表格s:**

| $j\downarrow \qquad;\qquad i\to$ |  0   |  1   |  2   |  3   |  4   |
| :------------------------------: | :--: | :--: | :--: | :--: | :--: |
|                1                 |  0   | None | None | None | None |
|                2                 |  0   |  1   | None | None | None |
|                3                 |  2   |  2   |  2   | None | None |
|                4                 |  2   |  2   |  2   |  3   | None |
|                5                 |  2   |  2   |  2   |  4   |  4   |



可以发现运行时间实际上是$\Omega(n^3)$

~~由于强迫症所以下标用了从0开始，然后出现了一堆问题，虽然最后解决了，但是网络上许多从1开始的代码可能更容易理解~~



#### Step4: 构造最优解

我们已经离答案很近了，接下来只需要直接地指出如何加括号即可

```c++
/*
s是之前获得的分割点表格
Note:与算法导论以及许多网络资源上的代码不同 我在此处全部采用从0开始的下标 而非从1
*/
void PRINT_OPTIMAL_PARENS(std::vector<std::vector<int>> &s, int start, int end) {
    if (start == end) {
        cout << "A" << start;
    } else {
        cout << "(";
        PRINT_OPTIMAL_PARENS(s, start, s[start][end - 1]);
        PRINT_OPTIMAL_PARENS(s, s[start][end - 1] + 1, end);
        cout << ")";
    }
}
```

这并不难理解



#### Final: 跑一跑

正式运行一下我们之前列出的例子

```c++
int main() {
    int test[] = {30, 35, 15, 5, 10, 20, 25}; //矩阵链长度
    int length = 6; //矩阵个数
    std::vector<std::vector<int>> m(length, std::vector<int>(length)); //创建m
    std::vector<std::vector<int>> s(length - 1, std::vector<int>(length - 1)); //创建s
    MATRIX_CHAIN_ORDER(test, length, m, s); //芜湖 起飞
    PRINT_OPTIMAL_PARENS(s, 0, length - 1); //得到输出 ((A0(A1A2))((A3A4)A5))
    return 0;
}
```

### 

### 重构最优解

==永远不要对一个算法感到满足== 

从实际考虑，我们通常会把子问题的解存在一个表里

加入备忘机制

不做太多解释，上面的看懂了这个也很容易理解

```c++
int LOOKUP_CHAIN(std::vector<std::vector<int>> &m, int *p, int start, int end, std::vector<std::vector<int>> &s) {
    if (m[start][end] < 999999) {
        return m[start][end];
    }

    int q;

    if (start == end) {
        m[start][end] = 0;
    } else {
        for (int i = start; i <= end - 1; ++i) {
            q = LOOKUP_CHAIN(m, p, start, i, s) + LOOKUP_CHAIN(m, p, i + 1, end, s) + p[start] * p[i + 1] * p[end + 1];
            if (q < m[start][end]) {
                m[start][end] = q;
                s[start][end - 1] = i;
            }

        }
    }

    return m[start][end];
}

int MEMORIZED_MATRIX_CHAIN(int *p, int length, std::vector<std::vector<int>> &m, std::vector<std::vector<int>> &s) {
    for (int i = 0; i < length; ++i) {
        for (int j = i; j < length; ++j) {
            m[i][j] = 999999;
        }
    }

    return LOOKUP_CHAIN(m, p, 0, length - 1, s);
}
```





