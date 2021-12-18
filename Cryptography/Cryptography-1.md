---
title: Cryptography-1:Symmetric Cryptography
categories:
- Cryptography 
mathjax: true
---

基于《图解密码技术》@[日]结城浩

# XOR异或

$$
0\oplus0=0\\
0\oplus1=1\\
1\oplus0=1\\
1\oplus1=1\\
$$

与加法不同，不需要进位

对同一个比特序列重复两次XOR运算就会复原

XOR运算会大量运用在后面的密码技术中

# 一次性密码本

一次性密码本是已经在理论上被证明不存在破译方法的加密方式，即使是对密钥空间进行暴力破解也无法破译

## 加密解密过程

$$
plaintext\oplus key=ciphertext\\
ciphertext\oplus key=plaintext
$$

这里的密钥$key$必须是随机比特序列

## 无法破译的原因

我们假设对一次性密码本的密文进行暴力破解，那么我们终有一天会尝试到加密时所用的$key$从而获得$plaintext$，这毋庸置疑，然而关键点在于我们**无法判断它是否是正确的明文**

i.e.

假设 $plaintext="midnight"$，它会被编码成64位的比特序列，被一串随机比特序列的$key$加密

Eve经过暴力破解后会获得诸如$"aaaaaaaa","abcdefgh","ZZZZZZZZ","onenight","mistress"$包括$"midnight"$的大量规则、不规则、有意义和无意义的字符串，而Eve无法判断他们是不是正确的明文

但是即使它如此优秀，无法被破译，可事实上无人会使用这种加密方式

- 密钥配送，这是最大的问题，Alice和Bob必须持有相同的$key$去加密解密，如果能安全传输$key$为什么不直接安全传输$plaintext$
- 密钥保存，和配送类似，$key$和$plaintext$长度相同，如果能安全保存$key$何不直接安全保存$plaintext$

# DES(Data Encryption Standard)

大人，时代变了，DES已经能被暴力破解了，不能再用了

DES是一种对称密码算法也是分组密码，将64比特明文加密成64比特密文，密钥长度56比特(事实上是64比特，但是每隔7比特会设置一个用于错误检查的比特)

值的关注的是DES的结构，也就是Feistel网络

Feistel网络中加密的各个步骤称为**轮(round)**，整个加密过程就是进行若干次轮的循环，DES是一种16轮循环的Feistel网络

<img src="https://s1.ax1x.com/2020/08/07/ahOWM6.png" alt="feistel" style="zoom: 67%;" />

上图是Feistel网络中一轮的示意图，可以看出输入比特序列的右侧并没有被加密，故而Feistel网络在两轮之间会把左右数据对调

<img src="https://s1.ax1x.com/2020/08/07/ahjnnf.png" alt="feistel" style="zoom:67%;" />

Feistel网络的特点在与加密的关键在与轮函数，而轮函数无论被设计的多么复杂，设计成没有反函数，最终也可以成功解密；而且Feistel网络的加密解密结构完全相同，这正是由于其在每一轮中对右侧序列不进行操作带来的



# 差分分析与线性分析

两者都是针对都是分组密码的分析方法

差分分析：改变一部分明文并分析密文如何随之改变，理论上明文即使只改变一个比特，密文也应该发生彻底的改变

线性分析：将明文和密文的一些对应比特进行XOR并计算结果为零的概率，结果应该为$\frac{1}{2}$，如果能找到大幅$\frac{1}{2}$的部分，就能借此推测密钥相关信息

# 三重DES

由于DES可以被暴力破解，故而开发了三重DES

<img src="https://s1.ax1x.com/2020/08/08/a4VFpV.png" alt="triple_des" style="zoom:67%;" />

若$key1=key2=key3$，则就是普通的DES，故而三重DES具备向下兼容性，得益于第二步是**解密**而非加密

若$key1=key3\neq key2$，则称为DES-EDE2

若$key1\neq key2 \neq key3$，则称为DES-EDE3

三重DES被银行等机构使用，但是其处理速度不高，除了特别重视向下兼容的场合，很少用于新用途



# AES(Advanced Encryption Standard)

取代DES成为新标准的一种对称密码算法，2000年从全世界范围内提交的众多算法中选出了名为Rijndael的对称密码算法，确定为AES

Rijndael没有采用Feistel网络，而是使用了SPN结构

Rijndael的输入分组为128比特(16字节)

## Step1：SubBytes

逐字节替换，可以想象成**简单替换密码**的256字母版本

![subBytes](https://s1.ax1x.com/2020/08/08/a4ZnKS.png)

## Step2：ShiftRows

以4字节为单位的行按照一定规则左平移，每一行平移字节数不同

![shiftRows](https://s1.ax1x.com/2020/08/08/a4ZKbQ.png)

## Step3：MixColumns

对一个4字节的值进行比特运算，将其变为另一个4字节值

![MixColumns](https://s1.ax1x.com/2020/08/08/a4ZuDg.png)

## Ste4：AddRoundKey

将Step3的输出结果与轮密钥进行XOR，至此一轮结束

![addRoundKey](https://s1.ax1x.com/2020/08/08/a4ZQEj.png)

















