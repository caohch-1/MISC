---
title: OpenCV5-Image filtering
categories:
- OpenCV
mathjax: true

---

基于《OpenCV4快速入门》 @冯振、郭延宁、吕跃勇

# 图像卷积

卷积概念没啥好说的，注意如果卷积模板~~其实我喜欢叫卷积核~~不是中心对称的，在卷之前一定要转180度

还有一点，卷积核通常会进行归一化，即缩放使得其中所有元素合为1，因为不然可能导致卷完后输出矩阵中部分元素超出数据范围

OpenCV提供了**filter2D**函数实现卷积

函数原型：

- src：输入图像
- dst：输出图像
- ddepth：输出图像的数据类型，当为-1时，自动选择
- kernel：卷积核，CV_32FC1类型矩阵
- anchor：内核的基准点(锚点)，默认值(-1, -1)代表内核基准点位于kernel的中心位置
- delta：偏直，在计算结果中加上偏值
- borderType：像素外推选择标志

```c++
void cv::filter2D(InputArray src,
                  OutputArray dst,
                  int ddepth,
                  InputArray kernel,
                  Point anchor = Point(-1, -1),
                  double delta = 0,
                  int borderType = BORDER_DEFAULT
                 )
```

***Warning***：该函数不会翻转卷积核，需要手动翻转后再传入！！！

输出图像数据类型与输入图像数据类型联系

| 输入图像数据类型 |  输出图像可选数据类型   |
| :--------------: | :---------------------: |
|      CV_8U       | -1/CV_16S/CV_32F/CV_64F |
|  CV_16U/CV_16S   |    -1/CV_32F/CV_64F     |
|      CV_32F      |    -1/CV_32F/CV_64F     |
|      CV_64F      |        -1/CV_64F        |



# 噪音的种类与生成

## 椒盐噪音

椒盐噪音又称脉冲噪音，它会随即改变图像中的像素值

首先看随机函数**rand**

- high：输出随机数最大值
- low：最小值

```c++
int cvflann::rand()
    
double cvflann::rand_double(double high = 1.0,
                            double low = 0
                           )
int cvflann::rand_int(int high = RAND_MAX,
                      int low = 0
                     )
```

***Hint***：可以用取余的方式实现对随机数范围的设置，例如0~100是 $int\,a = rand()\%100$

在图像中添加椒盐噪音主要有如下四步

1. 确定噪点位置，可以通过上述随即函数实现
2. 确定噪点种类，因为椒盐噪音分白色和黑色
3. 修改像素灰度值. 需要判断通道数，通道数不同的图像中像素表示白色的方式不同
4. 得到有椒盐噪音的图像



i.e.

```c++
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;

void saltAndPepper(Mat image, int n) {
    for (int i = 0; i < n; ++i) {
        //random location
        int j, k;
        j = rand() % image.cols;
        k = rand() % image.rows;
        //random type
        int write_black = rand() % 2;
        
        if (write_black == 0) { //white
            if (image.type() == CV_8UC1) { //gray
                image.at<unsigned char>(k, j) = 255;
            } else if (image.type() == CV_8UC3) { //colorful
                image.at<Vec3b>(k, j)[0] = 255; //B
                image.at<Vec3b>(k, j)[1] = 255; //G
                image.at<Vec3b>(k, j)[2] = 255; //R
            }
        } else { //black
            if (image.type() == CV_8UC1) { //gray
                image.at<unsigned char>(k, j) = 255;
            } else if (image.type() == CV_8UC3) { //colorful
                image.at<Vec3b>(k, j)[0] = 0; //B
                image.at<Vec3b>(k, j)[1] = 0; //G
                image.at<Vec3b>(k, j)[2] = 0; //R
            }
        }
    }
}

int main() {
    Mat lena = imread("lena.png");
    Mat equalLena = imread("equalLena.png");

    imshow("lena", lena);
    imshow("equalLena", equalLena);
    saltAndPepper(lena, 10000);
    saltAndPepper(equalLena, 10000);
    imshow("processedLena", lena);
    imshow("processedEqualLena", equalLena);
    
    waitKey(0);
    return 0;
}
```

效果图

<img src="https://s1.ax1x.com/2020/07/30/aKsd8P.png" alt="椒盐噪音"  />



## 高斯噪音

高斯噪音是指噪声分布的概率密度函数服从高斯分布(正态分布)的一类噪音

高斯噪音的概率密度函数如下式
$$
p(z)=\frac{1}{\sqrt {2\pi\sigma}}e^{\frac{-(z-\mu)^2}{2\sigma ^2}}
$$
其中 $z表示图像像素的灰度值$，$\mu 表示像素值的平均值或期望值$，$\sigma 表示像素值的标准差$，$标准差的平方(\sigma ^2)称为方差$，区别于椒盐噪音随即出现在图像的任意位置，高斯噪音出现在图像的所有位置

![gauss分布](https://pic4.zhimg.com/v2-1834bfc381eed61bbc8415129d00560b_r.jpg)

先来看OpenCV提供的用于产生**均匀分布或者高斯分布的随机数**的**fill**函数

函数原型：

- mat：用于存放随机数的矩阵，目前只支持低于5通道的矩阵
- distType：随机数分布形式选择标志. (RNG::UNIFORM, 0)均匀分布，(RNG::NORMAL, 1)高斯分布
- a：确定分布规律的参数. 当选择均匀分布时，表示最小下限；当选择高斯分布时，表示均值
- b：确定分布规律的参数. 当选择均匀分布时，表示最大上限；当选择高斯分布时，表示标准差
- saturateRange：预饱和标志，仅用于均匀分布

```c++
void cv::RNG::fill(InputOutputArray mat,
                   int distType,
                   InputArray a,
                   InputArray b,
                   bool saturateRange = false
                  )
```

***HInt***：该函数属于RNG类，是一个非静态成员函数，需要先创建一个RNG类的变量再调用

在图像中添加高斯噪音主要有四步：

1. 创建一个与图像尺寸、数据类型和通道数相同的Mat类变量
2. 通过调用**fill**函数在Mat类变量中产生符合高斯分布的随机数
3. 将原始图像和含有高斯分布的随机数矩阵相加
4. 得到高斯噪音之后的图像

i.e.

```c++
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;

int main() {
    Mat lena = imread("lena.png");
    Mat equalLena = imread("equalLena.png");
    imshow("original lena", lena);
    imshow("original equalLena", equalLena);

    Mat lena_noise = Mat::zeros(lena.rows, lena.cols, lena.type());
    Mat equalLena_noise = Mat::zeros(equalLena.rows, equalLena.cols, equalLena.type());

    RNG rng;
    rng.fill(lena_noise, RNG::NORMAL, 10, 20);
    rng.fill(equalLena_noise, RNG::NORMAL, 15, 30);
    lena = lena + lena_noise;
    equalLena = equalLena + equalLena_noise;


    imshow("lena with noise", lena);
    imshow("equalLena with noise", equalLena);

    waitKey(0);
    return 0;
}
```

效果图

![高斯噪音](https://s1.ax1x.com/2020/07/30/aKRYVS.png)



# 线性滤波

线性滤波与图像的卷积操作过程相似，不同之处在于，图像滤波不需要将滤波器(卷积核)旋转180度

滤波器表示中心像素与滤波范围内其他像素之间的线性关系，通过滤波范围内所有像素值之间的线性组合，得到中心位置像素滤波后的像素值，因此这种方式称为线性滤波

## 均值滤波

均值滤波将滤波器内所有的像素值都看作中心像素值的测量，将滤波器内所有的像素值的平均值作为滤波器中心处图像像素值

其优点在于在像素值变换趋势一致的情况下，可以将受噪声影响而突然变化的像素值修正为周围邻近像素值的平均值，去除噪声影响

但是这种滤波方式会缩小像素值之间的差距，使得细节信息变模糊

**blur**函数原型

- src：待滤波图像，数据类型必须是CV_8U、CV_16U、CV_16S、CV_32F或CV_64F
- dst：输出图像
- ksize：滤波器尺寸
- anchor：滤波器锚点，默认是滤波器几何中心
- borderType：像素外推选择标志

```c++
void cv::blur(InputArray src,
              OutputArray dst,
              Size ksize,
              Point anchor = Point(-1, -1),
              int borderType = BORDER_DEFAULT
             )
```

输入滤波器尺寸后会自动生成，如下
$$
K=\frac{1}{ksize\cdot width \times ksize\cdot height}
\left[\begin{array}{cccccc}
1 & 1& 1& \cdots & 1& 1\\
1 & 1& 1& \cdots & 1& 1\\
\vdots & \vdots &\vdots& \ddots& \vdots& \vdots \\
1 & 1& 1& \cdots & 1& 1\\
\end{array}\right]
$$


## 方框滤波

方框滤波是均值滤波的一般形式

方框滤波也是求滤波器内所有的像素值的和，但是方框滤波可以选择不进行归一化，而是将所有像素值的和作为滤波结果

**boxFilter**函数原型

- src：待滤波图像
- dst：输出图像
- ddepth：输出图像的数据类型，-1时自动选择
- ksize：滤波器尺寸
- anchor：滤波器锚点，默认是滤波器几何中心
- normalize：是否归一化标志
- borderType：像素外推选择标志

```c++
void cv::boxFilter(InputArray src,
              	   OutputArray dst,
                   int ddepth,
              	   Size ksize,
              	   Point anchor = Point(-1, -1),
                   bool normalize = true,
              	   int borderType = BORDER_DEFAULT
             	  )
```

该函数和均值滤波函数**blur**类似，只是可以自己选择输出图像的数据类型

**sqrBoxFilter**函数原型，实现对滤波器内每个像素值的平方求和

```c++
void cv::sqrBoxFilter(InputArray src,
               	      OutputArray dst,
                      int ddepth,
              	      Size ksize,
              	      Point anchor = Point(-1, -1),
                      bool normalize = true,
              	      int borderType = BORDER_DEFAULT
             	     )
```

该函数在处理图像滤波任务时主要针对的是CV_32F数据类型的图像

## 高斯滤波

高斯滤波器考虑了像素离滤波器中心距离的影响，以滤波器中心位置为高斯分布的均值，更具高斯分布公式和每个像素离中心位置的距离计算出滤波器内每个位置的数值，从而形成一个如下图的高斯滤波器(也就是高斯函数的二维分布)

![gauss二维](https://tse2-mm.cn.bing.net/th/id/OIP.XIARg18Vb1_wDHlXv5eyFwAAAA?pid=Api&rs=1)







OpenCV4提供了对图像进行高斯滤波操作的**GaussianBlur**函数

函数原型

- src：待滤波图像，通道数任意，但是数据类型必须是CV_8U，CV_16U，CV_16S，CV_32F或CV_64F
- dst：输出图像
- ksize：高斯滤波器尺寸，可以不是正方形，但必须是正奇数。如果尺寸为0，那么由标准偏差计算尺寸
- sigmaX：X方向的标准偏差
- sigmaY：Y方向的标准偏差
- borderType：像素外推法选择标志

```c++
void GaussianBlur(InputArray src,　　　　
　　　　　　　　　　　OutputArray dst, 　　
　　　　　　　　　　　Size ksize, 　　　　　
　　　　　　　　　　　double sigmaX, 　　　　　
　　　　　　　　　　　double sigmaY=0, 　　　
　　　　　　　　　　　intborderType=BORDER_DEFAULT
                 )　　 
```

***Hint***：为了使计算结果符合预期，建议将该函数的第三、四和五参数都明确给出



高斯滤波器的尺寸和标准偏差存在着一定的互相转换关系，OpenCV4提供**getGaussianKernel**函数

- kisze：高斯滤波器的尺寸
- sigma：高斯滤波的标准差
- ktype：滤波器系数的数据类型，可以是CV_32F或者CV_64F

```c++
Mat cv::getGaussianKernel(int ksize,
                          double sigma,
                          int ktype = CV_64F
                         )
```

第二个参数如果是负数，就使用默认的公式计算高斯滤波器的尺寸与标准差 $sigma=0.3((ksize-1)0.5-1+0.8)$

生成二维高斯滤波器需要调用该函数两次，将X和Y方向的一维高斯滤波器相乘，得到最终的二维高斯滤波器



## 可分离滤波

图像滤波具有可分离性，这个性质在高斯滤波中我们不难看出。

可分离性指的是先对X(Y)方向滤波，再对另一个方向滤波的结果与将两个滤波器联合后整体滤波的结果相同

两个方向滤波器的联合就是将两个方向的滤波器相乘，得到一个矩形的滤波器

OpenCV4提供了可以输入两个方向的滤波器实现滤波的滤波函数**sepFilter2D**

函数原型

- src：输入图像
- dst：输出图像
- ddepth：输出图像的数据类型，当为-1时，自动选择
- kernelX：X方向滤波器
- kernelY：Y方向滤波器
- anchor：内核的基准点(锚点)，默认值(-1, -1)代表内核基准点位于kernel的中心位置
- delta：偏直，在计算结果中加上偏值
- borderType：像素外推选择标志

```c++
void cv::sepFilter2D(InputArray src,
               	     OutputArray dst,
                     int ddepth,
                     InputArray kernelX,
                     InputArray kernelY,
                     Point anchor = Point(-1, -1),
                     double delta = 0,
                     int borderType = BORDER_DEFAULT
                    )
```



# 非线性滤波

## 中值滤波

中值滤波就是用滤波器范围内的所有像素值的中值来替代滤波器中心位置像素值的滤波方法，是一种基于排序统计理论的能够有效抑制噪声的非线性信号处理办法

相比于均值滤波，中值滤波对于脉冲干扰信号和图像扫描噪声的处理效果更佳，中值滤波对图像的边缘信息保护效果更佳，可以避免图像细节模糊，但是处理时间远大于均值滤波

OpenCV提供了**medianBlur**函数

- src：输入图像，可以单通道、三通道或四通道. 数据类型与滤波器尺寸有关，当滤波器尺寸为3或5时，图像可以是CV_8U、CV_16U或CV_32F，对于尺寸较大的滤波器，只能是CV_8U
- dst：输出图像
- ksize：滤波器尺寸，必须是大于1的奇数

```c++
void cv::medianBlur(InputArray src,
                    OutputArray dst,
                    int ksize
                   )
```







## 双边滤波

双边滤波能够对图像边缘信息进行保留

是一种综合考虑滤波器内图像空域信息和滤波器内图像像素灰度值相似性的滤波算法

双边滤波对高频率的波动信号起到平滑作用，同时保留大幅值变化的信号波动

由于是空域滤波器和值域滤波器的结合，使得滤波器对边缘附近的像素进行滤波时，距离边缘较远的像素值不会对边缘上的像素值影响太多，进而保留边缘的清晰性

![双边滤波](https://pic1.zhimg.com/v2-0f37cf653d7b23347a47f930a2cd092c_r.jpg)



OpenCV提供了进行双边滤波操作的 **bilateralFilter**函数

函数原型

- src：待进行双边滤波的图像，图像数据类型必须是CV_8U、CV_32F或CV_64F，并且通道数必须为单通道或者三通道
- dst：输出图像
- d：滤波过程中每个像素邻域的直径, 如果该值非正，那么由第五个参数sigmaSpace计算得到
- sigmaColor：颜色空间滤波器的标准差值，这个参数越大，表明该像素领域内由越多的颜色被混合到一起，产生较大的半相等颜色区域
- sigmaSpace：空间坐标中滤波器的标准差值，这个参数越大，表明越远的像素会相互影响，从而使更大领域中由足够相似的颜色获取相同的颜色. 当第三个参数d大于0时，邻域范围d确定；当第三个参数小于或等于0时，邻域范围正比于这个参数数值
- borderType：像素外推选择标志

```c++
void cv::bilateralFilter(InputArray src,
                         OutputArray dst,
                         int d,
                         double sigmaColor,
                         double sigmaSpace,
                         int borderType = BORDER_DEFAULT
                        )
```

 ***Hint***：如果在实时系统中使用该函数，建议滤波器尺寸设置为5，对于离线处理含有大量噪音的滤波图像，可以设置为9. 该函数第四个第五个参数是两个滤波器的标准差值，为了简单起见，可以将两个参数设置成相同的数值，当他们小于10时，滤波作用较弱，大于150时，效果非常强烈

双边滤波具有美颜功能，如下图

![双边滤波美颜](https://s1.ax1x.com/2020/07/31/a1PABR.png)





# 图像的边缘检测

## 边缘检测原理

图像的边缘指的是图像中像素灰度直突然变化的区域，如果将图像的每一行像素和每一列像素都描述成一个关于灰度值的函数，那就可以通过导数值较大的区域寻找函数中突然变化的区域，进而确定图像中的边缘位置

由于图像是离散的信号，因此我们可以用临近的两个像素差值来表示像素灰度值函数的导数，如下
$$
\frac{df(x,y)}{dx} = f(x,y)-f(x-1,y)
$$
这种对x轴求导对应的滤波器为$[-1 \quad 1]$，同样对y轴方向求导的滤波器为$[-1\quad 1]^T$

但是这种求导方式的计算结果最接近于两个像素中间位置的梯度，而两个临近的像素中间不有任何像素

因此，如果要表示某个像素处的梯度，最接近的方式是求取前一个像素和后一个像素的差值，于是修改上式为
$$
\frac{df(x,y)}{dx} = \frac{f(x+1,y)-f(x-1,y)}{2}
$$
改进的求导方式对应的滤波器在X方向和Y方向分别为$[-0.5\quad 0\quad 0.5]和[-0.5\quad 0\quad 0.5]^T$

根据这种方式也可以用下式所示滤波器计算45度方向的梯度，寻找不同方向的边缘
$$
XY=\left[\begin{array}{cc}
1&0\\
0&-1\\
\end{array}\right] \quad
YX=\left[\begin{array}{cc}
0&1\\
-1&0\\
\end{array}\right] 
$$
由于图像边缘处像素值可能突然变高也可能突然变低，故而需要化为绝对值

OpenCV4提供了**convertScaleAbs**

函数原型

- src：输入矩阵
- dst：输出矩阵
- alpha：缩放因子
- beat：偏值

```c++
void cv::convertScaleAbs(InputArray src,
                         OutputArray dst,
                         double alpha = 1,
                         double beta = 0
                        )
```



## Sobel算子

边缘检测算子，求取边缘的思想与前文介绍的一致，但是Sobel算子结合了高斯平滑滤波的思想，将边缘检测滤波器尺寸由$ksize\times1$改进为$ksize\times ksize$，提高了对平滑边缘的响应

Sobel边缘检测算子提取图像边缘的过程大致分为以下三个步骤

第一步：

提取X方向的边缘，X方向一阶Sobel边缘检测算子如下所示
$$
\left[\begin{array}{ccc}
-1&0&1\\
-2&0&2\\
-1&0&1\\
\end{array}\right]
$$
第二步：

提取Y方向边缘，Y方向一阶Sobel边缘检测算子如下所示
$$
\left[\begin{array}{ccc}
-1&-2&-1\\
0&0&0\\
1&2&1\\
\end{array}\right]
$$


第三步：

综合两个方向的边缘信息得到整幅图像的边缘，有两种计算方式

第一种：求取两幅图对应像素的平方和的二次方根 $I(x,y)=\sqrt{I_x(x,y)^2+I_y(x,y)^2}$

第二种：求取两幅图对应像素的像素值的绝对值和 $I(x,y)=|I_x(x,y)^2|+|I_y(x,y)^2|$

OpenCV4提供了Sobel边缘提取函数

函数原型

- src：待边缘提取图像
- dst：输出图像
- ddepth：输出图像的数据类型
- dx：X方向差分阶数
- dy：Y方向差分阶数
- ksize：Sobel算子尺寸，必须是1、3、5或7
- scale：缩放因子
- delta：偏值
- borderType：像素外推方法标志

```c++
void cv::Sobel(InputArray src
               OutputArray dst,
               int ddepth,
               int dx,
               int dy,
               int ksize = 3,
               double scale = 1,
               double delta = 0,
               int borderType = BORDER_DAFAULT
              )
```

***Warning***：第三个参数ddepth不要使用CV_8U，因为提取边缘时可能出现负数；四、五和六参数是关键参数，任意一个方向的差分阶数都要小于算子的尺寸，特殊情况是当ksize=1时，任意一个方向的阶数要小于3(此时算子不再是正方形，而是$3\times1或者1\times3$). 一般情况下，差分阶数最大值为1时，算子尺寸选3；差分阶数的最大值为2时，算子尺寸选5；当差分阶数最大值为3时，算子尺寸选7.



## Scharr算子

Sobel算子可以有效提取图像边缘，但是对于图像中较弱的边缘提取效果较差，故而需要增大像素值间的差距

Scharr算子是对Sobel算子差异性的增强，故而原理和使用方式相同

该算子尺寸为$3\times3$，如下
$$
G_x=\left[\begin{array}{ccc}
-3&0&3\\
-10&0&10\\
-3&0&3\\
\end{array}\right]\quad
G_y=\left[\begin{array}{ccc}
-3&-10&-3\\
0&0&0\\
3&10&3\\
\end{array}\right]
$$




OpenCV4提供了Scharr边缘提取函数

- 函数原型

  - src：待边缘提取图像
  - dst：输出图像
  - ddepth：输出图像的数据类型
  - dx：X方向差分阶数
  - dy：Y方向差分阶数
  - scale：缩放因子
  - delta：偏值
  - borderType：像素外推方法标志

  ```c++
  void cv::Scharr(InputArray src
                  OutputArray dst,
                  int ddepth,
                  int dx,
                  int dy,
                  double scale = 1,
                  double delta = 0,
                  int borderType = BORDER_DAFAULT
                 )
  ```
  ***Warning***：四和五参数当中只有一个能为1，并且不能同时为0



## 生成边缘检测滤波器

OpenCV4提供了**getDerivKernels**函数，通过该函数可以得到不同尺寸、不同阶次的Sobel算子和Scharr算子的滤波器

- kx：行滤波器系数的输出矩阵
- ky：列滤波器系数的输出矩阵
- dx：X方向导数的阶次
- dy：Y方向导数的阶次
- ksize：滤波器大小，可选1、3、5、7或FILTER_SCHARR
- normalize：归一化标志
- ktype：滤波器系数类型，可选CV_32F或CV_64F

```c++
void cv::getDerivKernels(OutputArray kx,
                         OutputArray ky,
                         int dx,
                         int dy,
                         int ksize,
                         bool normalize = false,
                         int ktype = CV_32F
                        )
```

该函数可以用于生成前面说的两个算子

第五个参数取1、3、5或7就是Soble算子，取FILTER_SCHARR就是Scharr算子；且该参数需要大于第三和第四个参数；当该参数为1时，第三、四参数的最大值需要小于3；当该参数取FILTER_SCHARR时，三、四参数必须一个0一个1

## Laplacian算子

之前的算子都具有方向性，而该算子能够对任意方向的边缘进行提取

该算子是一种二阶导数算子，对噪音比较敏感，因此需要配合**高斯滤波**一起使用

该算子定义如下

![laplacian](https://s1.ax1x.com/2020/08/02/aYpAF1.png)


OpenCV4提供了Laplacian算子提取图像边缘的函数

函数原型

- src：待边缘提取图像
- dst：输出图像
- ddepth：输出图像的数据类型
- scale：缩放因子
- delta：偏值
- borderType：像素外推方法标志

```c++
void cv::Laplacian(InputArray src,
                   OutputArray dst,
                   int ddepth,
                   int ksize = 1,
                   double scale = 1,
                   double delta = 0,
                   int borderType = BORDER_DEFAULT
                  )
```

***Warning***：第三个参数不要使用CV_8U，会使得图像边缘提取不准确

***Hint***：第四个参数必须是正奇数. 它大于1，该函数通过Sobel算子计算图像X和Y方向的二阶导数并求和获得Laplacian算子；当它等于1，Laplacian算子如下
$$
\left[\begin{array}{ccc}
0&1&0\\
1&-4&1\\
0&1&0\\
\end{array}\right]
$$


## Canny算法

这是目前最优越的边缘检测算法之一，分五步

第一步：

使用高斯滤波平滑图片，减少噪点，一般使用如下$5\times5$滤波器：
$$
\left[\begin{array}{ccccc}
2&4&5&4&2\\
4&9&12&9&4\\
5&12&15&12&5\\
4&9&12&9&4\\
2&4&5&4&2\\
\end{array}\right]
$$
第二步：

计算图像中每个像素的梯度方向和幅值. 首先通过Sobel算子分别检测图像X和Y方向边缘，再利用下式计算梯度方向和幅值
$$
\theta=arctan(\frac{I_y}{I_x})\\
G=arctan\sqrt{I_x^2+I_y^2}
$$
为了简便，梯度方向常取值0、45、90、135这四个之一

第三步：

应用非极大值抑制算法消除边缘检测带来的杂散响应. 首先，将当前像素的梯度强度于沿正负梯度方向上的两个像素进行比较，如果当前像素的梯度强度与另外两个相比最大，那么该像素保留为边缘点，否则该像素点被抑制

第四步：

应用双阈值法划分强和弱边缘. 将边缘处的梯度值与两个阈值进行比较，如果某像素的梯度幅值小于较小的阈值，那么会被去除；如果某像素的梯度幅值大于较小阈值且小于较大阈值，标记为弱边缘；如果大于较大阈值，标记为强边缘

第五步：

消除孤立的弱边缘. 在弱边缘的8邻域范围寻找强边缘，如果存在，就保留该弱边缘，否则删除. 最终输出结果

OpenCV4提供了实现Canny算法的函数

函数原型

- image：输入图像，必须是CV_8U的单通道或三通道图像

- edges：输出图像，与输入图像具有相同尺寸的单通道图像，且数据类型为CV_8U

- threshold1：第一个滞后阈值

- threshold2：第二个滞后阈值

- apertureSize：Sobel算子直径

- L2gradient：计算梯度赋值方法的标志，共两种如下

$$
L_1=|\frac{dI}{dx}|+|\frac{dI}{dy}|\\
L_2=\sqrt{(\frac{dI}{dx})^2+(\frac{dI}{dy})^2}
$$

```c++
void cv::Canny(InputArray image,
               OutputArray edges,
               double threshold1,
               double threshold2,
               int apertureSize = 3,
               bool L2gradient = false
              )
```

***Hint***：较大较小阈值的比值一般在2：1到3：1















