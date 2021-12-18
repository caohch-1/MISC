---
title: OpenCV8-Image Analysis and Restoration
categories:
- OpenCV
mathjax: true
---

基于《OpenCV4快速入门》 @冯振、郭延宁、吕跃勇

# 傅里叶变换

## 离散傅里叶变换

可以简单地把傅里叶变换理解成一个函数的分解工具，图像离散傅里叶变换后的结果会得到既有实数又有虚数的图像，在实际使用时常将结果分成实数图像和虚数图像，或者使用复数的幅值和相位来表示变换结果，可以分为幅值图像和相位图像

离散傅里叶变换能得到图像的频域信息，借此从另一个方面理解图像. 图像中像素波动较大的区域对应的频域是高频区域，因此高频区域体现图像的细节、纹理信息，而低频信息代表了图像的轮廓信息

OpenCV4提供了**dft**函数实现对图像的离散傅里叶变换，以及逆变换函数**idft**

一般来说，离散傅里叶变换算法倾向于对某些特定长度的输入矩阵进行处理，OpenCV4提供了用于计算最优输入矩阵尺寸的**getOptimalDFTSize**函数

确定最优尺寸后需要改变图像尺寸，为了不对图像进行缩放，OpenCV4提供了**copyMakeBorder**函数用于在图像周围形成外框

由于离散傅里叶变换完得到的数值可能为双通道的复数，在实际使用过程中更加关注复数的幅值，因此OpenCV4提供了**magnitude**函数用于计算由两个矩阵组成的二维向量矩阵的幅值矩阵

## 傅里叶变换进行卷积

傅里叶变换可以将两个矩阵的卷积转换成两个矩阵傅里叶变换结果的乘积，从而极大地提高卷积的计算速度

OpenCV4提供了**mulSpectrums**函数用于计算两个复数矩阵的乘积

## 离散余弦变换

类似于离散傅里叶变换，但是变换过程中只使用实数，主要用于对信号和图像的有损压缩

OpenCV4提供了**dct**函数实现离散余弦变换，以及逆变换函数**idct**



# 积分图像

积分图像主要用于快速计算图像某些区域像素的平均灰度

积分图像是比原图像尺寸大1的新图像，就是从$N\times N$变成$(N+1)\times(N+1)$

积分图像中每个像素的像素值为原图像中该像素点与坐标原点组成的矩形内所有像素点的和

![积分图像](https://img-blog.csdn.net/20150605235951859)



积分图像主要分为三种

标准求和积分图像$sum(x,y)=\sum_{y'<y}\sum_{x'<x}I(x',y')$

平方和积分图像$sum_{square}=\sum_{y'<y}\sum_{x'<x}I(x',y')^2$

倾斜求和积分图像$sum_{tilted}=\sum_{y'<y}\sum_{asb(x'-x)<y}I(x',y')$

OpenCV4提供了**integral**函数用于实现上述三种积分图像

# 图像分割

将图像中某一类的像素点与其他像素点分开，例如黑白图像中分割黑白

## 漫水填充法

该方法是根据像素灰度值之间的差值寻找相同区域以实现分割

我们可以将图像的像素值理解成像素点的高度，这样一幅图像可以看成崎岖不平的地面或者山地，向地面上某一个低洼的地方倾倒一定量的水，水将掩盖低于某个高度的区域

分为三步

1. 选择种子点(注水点)
2. 以种子点为中心，判断4-或8-邻域的像素值与种子点像素值的差值，将差值小于阈值的像素点添加进区域内
3. 将新加入的像素点作为i新的种子点，反复执行第二步，直到没有新的像素点被添加进该区域为止

OpenCV4提供了**floodFill**函数实现该方法分割图像



## 分水岭法

与漫水填充法类似，都是模拟水淹过地面山地，区别在与漫水填充法是从某个像素值进行分割，是一种局部分割算法，而分水岭法从全局出发，对全局进行分割

分水岭法在多个局部最低点开始注水，随着注水量增加，水位升高，两个相邻的凹陷区域的水汇集，形成分水岭

该方法的计算过程是一个迭代标注过程，经典的计算方式主要分为以下两步

1. 排序，对图像像素灰度级进行排序，确定注水点
2. 注水，不断淹没周围像素，形成分割线

OpenCV4提供了**watershed**函数实现分水岭法分割图像



## Grabcut法

使用高斯混合模型估计目标区域的背景和前景，通过迭代解决了能量函数最小化问题

OpenCV4提供了**grabCut**函数实现Grabcut法进行图像分割



## Mean-Shift法

又叫均值漂移法，基于颜色空间分布的图像分割算法

OpenCV4提供了**pyrMeanShiftFiltering**函数实现该方法

# 图像修复

OpenCV4提供了能够对含有较少污染或者水印的图像进行修复的**inpaint**函数




































