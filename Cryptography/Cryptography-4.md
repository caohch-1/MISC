---
title: Cryptography-4:One-way Hash Function
categories:
- Cryptography 
mathjax: true
---

基于《图解密码技术》@[日]结城浩

# 单向散列函数性质

## 散列值长度固定

单向散列函数的输入必须能够是任意长度的消息，且无论输入的消息多长，必须能够生成长度短且固定的散列值

## 抗碰撞性

两个不同的消息产生同一个散列值称为**碰撞**，如果要将单向散列函数用于完整性检查，必须保证在事实上不可能被人为地发现碰撞

弱抗碰撞性：要找到和某条消息具有相同散列值的另一条消息是非常困难的

强抗碰撞性：要找到散列值相同的两条不同的消息是非常困难的

## 单向性

无法通过散列值反推出消息，基于口令的加密(PBE)和伪随机数生成器等技术中就用到了单向性



# SHA-3

单向散列函数的具体例子有很多，这里主要讲述SHA-3

SHA-3是一种作为新标准发布的单向散列函数算法，用来替代理论上已被找出攻击方法的SHA-1算法

Keccak算法于2012年击败别的候选算法被确定为SHA-3标准

## 海绵结构

Keccak算法采用了与SHA-1、SHA-2完全不同的**海绵结构**，该结构中输入的数据在进行填充后经过**吸收阶段**和**挤出阶段**，最终生成散列值

![SHA-3](https://s1.ax1x.com/2020/08/11/aXUqud.png)

吸收阶段流程：

- 将经过填充的输入消息按照每r比特为一组分割成若干个输入分组
- 首先，将“内部状态的$r$比特”与“输入分组1”进行XOR，将其结果作为“函数$f$的输入值”
- 然后，将“函数f的输出值$r$个比特”与“输入分组2”进行XOR，将其结果再次作为“函数$f$的输入值”
- 反复执行上述步骤，直到达到最后一个输入分组
- 等所有的输入分组处理完成后，结束吸收阶段，进入挤出阶段

函数$f$的作用是将输入的数据进行复杂的搅拌操作并输出结果（输入和输出的长度均为$b=r+c$个比特），其操作对象是长度为$b=r+c$个比特的内部状态，内部状态的初始值为0. 每次被吸收的输入分组长度为$r$个比特，因此$r$被称为比特率。

通过上图可以看出，函数$f$的输入长度不是$r$个比特，而是$r+c$个比特，这意味着内部状态中有$c$个比特时不受输入分组内容直接影响的（但会通过函数$f$受到间接影响）。这里的$c$被称为容量。

挤出阶段流程

- 首先，将“函数$f$的输出值中的$r$个比特”保存为“输出分组1”，并将整个输出值（$r+c$个比特）再次输入到函数$f$中
- 然后，将“函数$f$的树池值中的$r$个比特”保存为“输出分组2”，并将整个输出值（$r+c$个比特）再次输入到函数$f$中
- 反复执行上面的步骤，直到获得所需长度的输出数据

## 双工结构

海绵结构的变形

在海绵结构中，只有将输入的消息全部吸收完毕之后才能开始输出，但在双工结构中，输入和输出是以相同的速率进行的

![双工](https://s1.ax1x.com/2020/08/11/aXaGUx.png)

## Keccak内部结构

每个小方块代表一个比特

![内部结构](https://s1.ax1x.com/2020/08/11/aXaIZn.png)

再来看看内部的搅拌函数$f$，它包含五个步骤：$\theta、\rho、\pi、\chi、\iota$

$\theta$：

将位置不同的两个column中各自5个比特通过XOR运算加起来(图中$\sum$标志)，然后再与置换目标比特求XOR并覆盖目标比特

![theta](https://s1.ax1x.com/2020/08/11/aXdKFP.png)

$\rho$：

沿$z$轴(lane方向)进行比特平移

![rho](https://s1.ax1x.com/2020/08/11/aXdsOJ.png)

$\pi$：

如图是对一片slice应用该步骤时的样子，事实上整条lane上的slice都会执行这一操作

![pi](https://s1.ax1x.com/2020/08/11/aXdffK.png)

$\chi$：

如图是对一个row应用该步骤的情景，这里使用了逻辑电路中的符号，其中**小三角后加点**代表输入比特取反(NOT)，**矩形后加半圆**表示两个输入均为1时输出1(AND)

![chi](https://s1.ax1x.com/2020/08/11/aXdH0A.png)

$\iota$：

用一个固定的轮常数对整个state的所有比特进行XOR运算，目的是让内部状态具备非对称性



# 对单向散列函数的攻击

## 暴力破解

每次稍微改变一下消息的值，然后对消息求散列值，其实就是寻找具有特定散列值的消息，是针对**弱抗碰撞性**的攻击

找出具有相同散列值的消息的攻击一般分为两种

- 原像攻击：给定散列值，找出具有该值的任意消息
- 第二原像攻击：给定一条消息1，找出另外一条消息2，他们散列值相同

## 生日攻击

找到散列相同的两条消息，而散列值可以是任意的，这称为生日攻击或者冲突攻击，是针对**强抗碰撞性**的攻击

这种攻击原理来源于**生日悖论**，具体不展开，很好理解

这种攻击比起暴力破解尝试次数要少的多



# 单向散列函数的问题

单向散列函数能够辨别消息是否被**篡改**，但是无法辨别**伪装**

我们通常不仅仅需要检查文件的完整性，还需要确认这个文件是否真的属于某个人，这时就需要**认证**，可以通过**消息验证码**和**数字签名**技术实现，以后会具体讲述














