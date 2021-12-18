---
title: OpenCV6-Image morphology Operation
categories:
- OpenCV
mathjax: true

---

基于《OpenCV4快速入门》 @冯振、郭延宁、吕跃勇

# 像素距离与连通域

## 图像像素距离变换

图像处理中常用的距离有欧氏距离、街区距离和棋盘距离

欧氏距离：$d=\sqrt{(x_1-x_2)^2+(y_1-y_2)^2}$

街区距离：$d=|x_1-x_2|+|y_1-y_2|$

棋盘距离：$d=max(|x_1-x_2|,|y_1-y_2|)$

OpenCV4提供了计算图像中不同距离的函数**distanceTransform**

函数原型：

- src：输入图像，数据图像类型为CV_8U
- dst：输出图像，数据类型为CV_8U或CV_32F的单通道
- labels：二维的标签数组(离散Voronoi图)，数据类型为CV_32S的单通道数据
- distanceType：选择计算两个像素之间距离方法的标志
- maskSize：距离变换掩码矩阵尺寸，可选尺寸为DIST_MASK_3($3\times3$)和DIST_MASK_5($5\times5$)
- labelType：要构建的标签数组的类型

```c++
void cv::distanceTransform(InputArray src,
                           OutputArray dst,
                           OutputArray labels,
                           int distanceType,
                           int maskSize,
                           int labelType = DIST_LABEL_CCOMP
                          )
```

距离计算方法选择标志表

| 标志参数  | 简记 |    作用    |
| :-------: | :--: | :--------: |
| DIST_USER |  -1  | 自定义距离 |
|  DIST_L1  |  1   |  街区距离  |
|  DIST_L2  |  2   |  欧氏距离  |
|  DIST_C   |  3   |  棋盘距离  |



标签数组类型选择标志表

|     标志参数     | 简记 |                      作用                       |
| :--------------: | :--: | :---------------------------------------------: |
| DIST_LABEL_CCOMP |  0   | 输入图像中每个连接的0像素都将被分配为相同的标签 |
| DIST_LABEL_PIXEL |  1   |        输入图像中每个0像素都有自己的标签        |



***Hint***：在使用欧氏距离时，推荐使用$5\times5$掩码；当选择棋盘距离时，掩码矩阵尺寸对计算结果没有影响



该函数还有一种重载，它取消了生成离散Voronoi图

函数原型：

- src：输入图像，数据图像类型为CV_8U
- dst：输出图像，数据类型为CV_8U或CV_32F的单通道
- distanceType：选择计算两个像素之间距离方法的标志
- maskSize：距离变换掩码矩阵尺寸，可选尺寸为DIST_MASK_3($3\times3$)和DIST_MASK_5($5\times5$)
- dstType：输出图像的数据类型，可以是CV_8U或者CV_32F

```c++
void cv::distanceTransform(InputArray src,
                           OutputArray dst,
                           int distanceType,
                           int maskSize,
                           int dstType = CV_32F
                          )
```

***Hint***：不建议使用尺寸过小或者黑色区域较多的图像



## 图像连通域分析

图像连通域是指图像中具有相同像素值并且位置相邻的像素组成的区域

连通域分析是指在图像中寻找彼此互相独立的连通域并将其标记出来

首先了解下图像邻域的概念，分为4-邻域和8-邻域

4-邻域定义下两个像素相邻必须在水平和垂直方向上相邻，相邻的两个像素坐标必须只有一位不同而且只能相差1个像素

8-邻域定义下两个像素相邻允许在对角线方向相邻，相邻的两个像素坐标在X和Y方向上的最大差值为1

常用的图像邻域分析法有两遍扫描法和种子填充法

首先是两遍扫描法

第一次遍历图像时会给每一个非零像素赋予一个数字标签，当某个像素的上方和左侧邻域内的像素已经有了数字标签时，取两者中较小值作为当前像素的标签，否则赋予一个新的数字标签，在第一次扫描过程中，同一个连通域可能会被赋予一个或多个不同的标签，故而在第二次扫描时需要将属于同一个连通域的不同标签合并. 下图gif是一个简单演示

![两遍扫描法](https://pic1.zhimg.com/v2-87301b46cf47a72590812c1e1658a37c_b.webp)

接着介绍种子填充法

源于计算机图像学，首先把所有非零像素放到一个集合中，之后在集合中随机选出一个像素作为种子像素，根据邻域关系不断扩充种子像素所在连通域，并在集合中删除扩充出的像素，直到种子像素所在连通域不能扩充，之后再从集合中随机选择一个像素作为新的种子像素，重复上述过程直至集合变成空集. 下图是一个简单gif演示

![种子填充](https://pic3.zhimg.com/v2-d386319f55ddcdab6b48a4911e2a3018_b.webp)





OpenCV4提供了用于提取不同连通域的**connectedComponents**函数，它有两个原型

函数原型一

- image：待标记的单通道图像，数据类型必须为CV_8U，最好经过二值化
- labels：标记不同连通域后的输出图像
- connectivity：邻域种类
- Itype：输出图像的数据类型，CV_32S或者CV_16U
- ccltype：标记算法类型的标志

```c++
int cv::connectedComponents(InputArray image,
                            OutputArray labels,
                            int connectivity,
                            int ltype,
                            int ccltype
                           )
```

返回连通域数目

标记算法标志表

|  标志参数   | 简记 |              作用              |
| :---------: | :--: | :----------------------------: |
|   CCL_WU    |  0   | 8-邻域使用SAUF，4-邻域使用SAUF |
| CCL_DEFAULT |  -1  | 8-邻域使用BBDT，4-邻域使用SAUF |
|  CCL_GRANA  |  1   | 8-邻域使用BBDT，4-邻域使用SAUF |



函数原型2(简化使用)

- image：待标记的单通道图像，数据类型必须为CV_8U，最好经过二值化
- labels：标记不同连通域后的输出图像
- connectivity：邻域种类
- Itype：输出图像的数据类型，CV_32S或者CV_16U

```c++
int cv::connectedComponents(InputArray image,
                            OutputArray labels,
                            int connectivity = 8,
                            int ltype = CV_32S
                           )
```



如果希望得到每个连通域中心位置或者在图像中标记出连通域所在的矩形区域，那就使用**connectedComponentsWithStats**函数

函数原型一

- image：待标记不同连通域的单通道图像，数据类型必须为CV_8U
- labels：标记不同连通域后的输出图像
- stats：含有不同连通域统计信息的矩阵，矩阵的数据类型为CV_32S，矩阵中第i行是标签为i的连通域的统计特性
- centroids：每个连通域的质心坐标，数据类型为CV_64F
- connectivity：标记连通域时使用的邻域种类，4表示4-邻域，8表示8-邻域
- ltype：输出图像的数据类型，目前支持CV_32S和CV_16U两种数据类型
- ccltype：标记连通域使用的算法类型标志

```c++
int cv::connectedComponentsWithStats(InputArray  image,
                                     OutputArray  labels,
                                     OutputArray  stats,
                                     OutputArray  centroids,
                                     int  connectivity,
                                     int  ltype,
                                     int  ccltype 
                                    )
```

如果图像中有N个连通域，那么该参数输出的矩阵尺寸为$N\times5$，矩阵中每一行分别保存每个连通域的统计特性，详细的如下表

|    标志参数    | 简记 |                             作用                             |
| :------------: | :--: | :----------------------------------------------------------: |
|  CC_STAT_LEFT  |  0   | 连通域内最左侧像素的x坐标，它是水平方向上的包含连通域边界框的开始 |
|  CC_STAT_TOP   |  1   | 连通域内最上方像素的y坐标，它是垂直方向上的包含连通域边界框的开始 |
| CC_STAT_WIDTH  |  2   |                    连通域边界框的水平长度                    |
| CC_STAT_HEIGHT |  3   |                    连通域边界框的垂直长度                    |
|  CC_STAT_AREA  |  4   |                  连通域的面积(以像素为单位)                  |
|  CC_STAT_MAX   |  5   |                 统计信息种类数目，无实际含义                 |

函数原型2(简化使用)

- image：待标记不同连通域的单通道图像，数据类型必须为CV_8U
- labels：标记不同连通域后的输出图像
- stats：不同连通域的统计信息矩阵，矩阵的数据类型为CV_32S
- centroids：每个连通域的质心坐标，数据类型为CV_64F
- connectivity：标记连通域时使用的邻域种类，4表示4-邻域，8表示8-邻域，默认参数值为8
- ltype：输出图像的数据类型，目前只支持CV_32S和CV_16U这两种数据类型，默认参数值为CV_32S

```c++
int cv::connectedComponentsWithStats(InputArray  image,
	                                 OutputArray  labels,
	                                 OutputArray  stats,
	                                 OutputArray  centroids,
                                     int  connectivity = 8,
	                                 int  ltype = CV_32S 
	                                )
```



# 腐蚀和膨胀

腐蚀和膨胀是形态学基本运算，可以用于去除图像中的噪声、分割出独立的区域或者将两个连通域连接在一起等

## 图像腐蚀

与卷积操作类似

结构元素内所有元素所覆盖的图像像素值均不为0，那么保留结构元素中心点对应的图像像素，否则将删除结构元素中心点对应的像素

<img src="http://5b0988e595225.cdn.sohucs.com/q_70,c_zoom,w_640/images/20181119/9cf88a46a7ab4189aa657a060e7a020a.webp" alt="腐蚀" style="zoom:150%;" />

结构元素可以根据需求自己生成，但是OpenCV4也提供了**getStructuringElement**用于生成常用的矩形、十字形和椭圆结构元素

函数原型

- shape：生成结构元素的种类
- ksize：结构元素的尺寸
- anchor：中心点的位置

```c++
Mat cv::getStructuringElement(int shape,
                              Size ksize,
                              Point anchor = Point(-1,-1)
                             )
```

结构元素种类表

|   标志参数    | 简记 |     作用     |
| :-----------: | :--: | :----------: |
|  MORPH_RECT   |  0   | 矩形元素矩阵 |
|  MORPH_CROSS  |  1   | 十字结构元素 |
| MORPH_ELLIPSE |  2   | 椭圆结构元素 |



OpenCV4也提供了用于图像腐蚀的**erode**函数

函数原型

- src：待腐蚀图像，通道数任意，数据类型必须是CV_8U、CV_16U、CV_16S、CV_32F或CV_64F
- dst：腐蚀后的输出图像
- kernel：结构元素
- anchor：中心点位置
- iterations：腐蚀次数
- borderType：像素外推法选择标志
- borderValue：使用边界不变外推法时的边界值

```c++
void cv::erode(InputArray src,
               OutputArray dst,
               InputArray kernel,
               Point anchor = Point(-1,-1),
               int iterations = 1,
               int borderType = BORDER_CONSTANT,
               const Scalar& borderValue = morphologyDefaultBorderValue()
              )
```

***HInt***：该函数的腐蚀过程只针对对非零像素

## 图像膨胀

与图像腐蚀相反

<img src="http://5b0988e595225.cdn.sohucs.com/q_70,c_zoom,w_640/images/20181119/c0910d8e3a80490b91898f5edda99e68.jpeg" alt="膨胀" style="zoom:150%;" />



OpenCV4也提供了用于图像膨胀的**dilate**函数

函数原型

- src：待膨胀图像，通道数任意，数据类型必须是CV_8U、CV_16U、CV_16S、CV_32F或CV_64F
- dst：膨胀后的输出图像
- kernel：结构元素
- anchor：中心点位置
- iterations：膨胀次数
- borderType：像素外推法选择标志
- borderValue：使用边界不变外推法时的边界值

```c++
void cv::dilate(InputArray src,
                OutputArray dst,
                InputArray kernel,
                Point anchor = Point(-1,-1),
                int iterations = 1,
                int borderType = BORDER_CONSTANT,
                const Scalar& borderValue = morphologyDefaultBorderValue()
               )
```



***jojo，人的记忆力力是有限的***

***我从短暂的大学生活中学到一件事......***

***越是记函数原型、参数列表的笔记，就越会发现人的记忆力是有限的......***

***除非超越函数***

***所以，我不记函数啦！jojo！***

***其实就是感觉记这些函数没啥用，又很浪费时间，后面只会记录一些原理性、概念性的东西了***

# 形态学应用

腐蚀可以将细小的噪声区域去除，但是会将图像主要区域的面积缩小

膨胀可以扩充每个区域的面积，填充较小的空洞，但是会增加噪声的面积

可以将这两者结合达到更好的处理效果

OpenCV4提供了**morphologyEx**函数实现以下运算

## 开运算

开运算可以去除图像中的噪声，消除较小的连通域，保留较大的连通域，同时能够在两个物体纤细的连接处将两个物体分离，并且在不明显改变较大连通域面积的同时平滑连通域的边界

步骤是先腐蚀，消除图像中的噪声和较小的连通域；后膨胀，弥补较大连通域因腐蚀而造成的面积减小

![开运算](https://s1.ax1x.com/2020/08/02/atHRjs.png)



## 闭运算

闭运算可以去除连通域内的小型空洞，平滑物体轮廓，连接两个临近的连通域

步骤是先膨胀，填充连通域内的小型空洞，扩大连通域的边界，将临近的连通域连接；后腐蚀，减少由膨胀引起的边界扩大和面积增大

![闭运算](https://s1.ax1x.com/2020/08/02/atHfun.png)



## 形态学梯度

形态学梯度能够描述目标边界，分为基本梯度、内部梯度和外部梯度

基本梯度是膨胀后和腐蚀后图像差值

内部梯度是原图像和腐蚀后图像差值

外部梯度是膨胀后图像和原图像差值

<img src="https://s1.ax1x.com/2020/08/02/at4oNQ.jpg" alt="形态学基本梯度" style="zoom: 67%;" />



## 顶帽运算

是原图像与开运算结果的差值，用于分离比邻近点亮一些的斑块

![顶帽元算](https://s1.ax1x.com/2020/08/02/at5KCd.jpg)



## 黑帽运算

黑帽运算是原图像与顶帽运算结果之间的差值

黑帽运算先进行闭运算，之后从闭运算结果中减去原图像

![黑帽运算](https://s1.ax1x.com/2020/08/02/at5DK0.jpg)





## 击中击不中变换

比图像腐蚀更严苛的一种操作，该变换要求原图像中需要存在与结构元素一模一样的结构

![击中击不中变换](https://s1.ax1x.com/2020/08/02/at5d8s.jpg)



## 图像细化

将图像的线条从多像素宽度减少到单位像素宽度，是模式识别领域的重要处理步骤之一，能够有效地减少数据量，减少图像的存储难度和识别难度

分为迭代和非迭代细化算法

非迭代不以像素为基础，该算法经过一次遍历，产生线条的某一中值或中心线，可以一次遍历产生“骨架”

迭代细化是指通过重复删除图像的边缘处满足一定条件的像素，最终得到“骨架”

迭代细化算法又分串行和并行，下面主要将并行细化算法中的**Zhang细化算法**

首先，循环所有前景像素点，对符合如下条件的像素点标记为待删除：

1. $2 \leqslant N(p_1) \leqslant6$，N(p)表示p像素8-邻域黑像素的数量

2. $A(P_1) = 1$，A(p)表示p像素8-邻域内按顺时针顺序前后两个像素分别为黑和白的对数

3. $P_2 \times P_4 \times P_6=0$

4. $P_4 \times P_6 \times P_8 = 0$

所有像素判断完后把所有标记的待删除点删除，接着进入第二轮循环，对符合如下条件的像素点标记为待删除：

1. $2 \leqslant N(p_1) \leqslant6$，N(p)表示p像素8-邻域黑像素的数量

2. $A(P_1) = 1$，A(p)表示p像素8-邻域内按顺时针顺序前后两个像素分别为黑和白的对数

3. $P_2 \times P_4 \times P_8=0$

4. $P_2 \times P_6 \times P_8 = 0$

所有像素判断完后把所有标记的待删除点删除，对同一副图像反复进行上述循环操作，直到没有可以删除的点

OpenCV4提供了用于将二值化图像细化的**thinning**函数

