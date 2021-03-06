---
title: Cryptography-2:Block Cipher
categories:
- Cryptography 
mathjax: true
---

基于《图解密码技术》@[日]结城浩

# 分组密码和流密码

**分组密码**是每次只能处理特定长度的一块数据的一类密码算法

**流密码**是对数据流进行连续处理的一类密码算法

在分组密码中，明文长度超过密码长度就需要对加密算法进行迭代，迭代的方法就称为分组密码的模式

# ECB模式

Electronic CodeBook电子密码本模式

这种模式非常简单，但是存在弱点故而一般不使用

![ECB](https://s1.ax1x.com/2020/08/08/aI2AJO.png)

可以看出ECB模式中明文密文分组一一对应，只要观察密文，就可以看出存在怎样的重复组合，从而作为破译线索

针对ECB模式的攻击主要是**改变密文分组顺序**

假设Mallory可以改变密文分组的顺序，那就可能会有如下的攻击(只是一个例子，事实上可以造成许多意想不到的攻击)

![ECB1](https://s1.ax1x.com/2020/08/08/aIRkBn.png)

可能的答案是把**口令**全部替换成对应的**名称**，在攻击一个计算机系统时名称往往比密码更容易获取到，通过这种方式Mallory就绕开了原本的密码

# CBC模式

Cipher Block Chaining密文分组链接

CBC模式中，首先将明文分组与前一个密文分组进行XOR运算，然后再进行加密

![CBC](https://s1.ax1x.com/2020/08/08/aIRRgg.png)



CBC模式的特点在于密文分组中如果有一个分组**损坏**，解密时最多有两个分组受到影响；但是如果有一个分组中有比特**缺失**，解密时后面的所有分组都会被影响

![CBC2](https://s1.ax1x.com/2020/08/08/aIW8MQ.png)



![CBC3](https://s1.ax1x.com/2020/08/08/aIW1xg.png)

针对CBC模式的攻击一般有两种，第一种是通过**翻转初始化向量中的某一位比特**，那么相应明文分组中的对应一位比特也会翻转，从而获取可能的破译信息；还有一种是**填充提示攻击**，该攻击利用分组密码的填充部分，攻击者反复修改填充部分的比特获取对应的服务器返回信息，从而获取破译线索，这种攻击方式适用于几乎所有分组密码，这种攻击听上去不太实际，然而2014年对SSL3.0造成重大影响的POODLE攻击就是该种攻击

# CFB模式

Cipher Feedback密文反馈

该模式下前一个密文分组会被送到密码算法的输入端

![CFB](https://s1.ax1x.com/2020/08/08/aIfMwR.png)

CFB模式和一次性密码本很相似，CFB中密码算法的输出相当于一次性密码本中的随机比特序列，当然密码算法的输出毕竟还是计算得到的，故而CFB模式不会像一次性密码本那样具备理论上不可破译的性质；CFB模式中密码算法生成的比特序列称为**密钥流**；CFB模式中明文数据可以被逐比特加密，因此CFB模式可以看作是一种用**分组密码来实现流密码**的方式

对CFB模式可以实施**重放攻击**

Alice向Bob发送有四个密文分组的消息，被Mallory窃听并且保留下了后三个分组，后来Alice又向Bob发送了一条有四个密文分组的消息，Mallory替换了其中的后三个分组，那么只有明文分组2会解密出错，明文分组3、4都会被替换成之前的消息，而Bob无法分辨分组2到底是被人恶意替换还是通信错误导致的

![CFB3](https://s1.ax1x.com/2020/08/08/aI4cZj.png)

# OFB模式

Output-Feedback输出反馈模式

OFB模式中，密码算法的输出会反馈到密码算法的输入中

![OFB](https://s1.ax1x.com/2020/08/08/aI4oyF.png)

在OFB模式中密钥流可以提前通过密码算法生成，与明文分组无关，而众所周知XOR运算很快，这意味着只要提前准备好密钥流就可以快速加密，换而言之，生成密钥流和进行XOR运算可以并行

# CTR模式

CounTeR计数器模式

通过将逐次累加的计数器进行加密来生成密钥流的流密码

![CTR](https://s1.ax1x.com/2020/08/08/aII65n.png)

关于计数器的生成，每次加密时都会生成一个不同的值(nonce)来作为计数器的初始值，当分组长度为128比特时，计数器的初始值可能向下面这样

![CTR1](https://s1.ax1x.com/2020/08/08/aIIOxK.png)

CTR模式中可以以任意顺序对分组进行加密解密，这意味着可以实现并行计算

再来看攻击，通过翻转密文分组中的某些比特，那么相应明文分组中的对应比特也会翻转，从而获取可能的破译信息，这是坏处；换个角度看，这一特点也保证如果通信过程中只有一个分组的密文受到损坏，那么不会影响解密后其他的明文分组，错误不会被放大，这也是好处



















