---
title: OpenCV3-Basic Img Operation
categories:
- OpenCV
mathjax: true

---

基于《OpenCV4快速入门》 @冯振、郭延宁、吕跃勇



# 图像颜色空间

## 颜色模型与转换

### RGB颜色模型

该模型为立方体~~不难想象~~

<img src="https://ss.csdn.net/p?https://mmbiz.qpic.cn/mmbiz_png/KfLGF0ibu6cL0nqcZhH7pv1LT6icjwibN5icCtFIfQVjXU4U0fPwFXswhibgdG6wLTv08N4snCtlKWJ4tuV81MKyZ0w/640?wx_fmt=png" />

OpenCV中顺序相反

Channel1 is Blue

Channel2 is Green

Channel3 is Red

该模型基础上加入Channel4即为RGBA模型，Channel4 is Alpha透明度

### YUV颜色模型

电视信号系统采用的颜色编码模式

像素的亮度 Y

红色分量与亮度的信号差值 U

蓝色与亮度的差值 V

$$
\left\{
\begin{array}{rcl}
Y=&0.299R+0.587G+0.144B\\
U=&-0.147R-0.289G+0.436B\\
V=&0.615R-0.515G-0.100B\\
R=&Y+1.14V\\
G=&Y-0.39U-0.68V\\
B=&Y+2.03U
\end{array}\right.
$$


### HSV颜色模型

该模型为锥形~~不难想象 个屁~~

<img src="http://image.catbro.cn/dda3f983d4912.png" alt="HSV" style="zoom:150%;" />

H is Hue 色度

S is Saturation 饱和度

V is Value 亮度

由于三个参量的取值范围不同，故为空间模型为锥体，该模型更符合人类感知颜色的方式：颜色，深浅，暗亮

### Lab颜色模型

该模型是球型

![Lab](https://tse2-mm.cn.bing.net/th/id/OIP.BUPtYIN5BjySudu8Tpz2rQHaHc?pid=Api&rs=1)

弥补了RGB模型的不足

L is Luminosity

a and b are two color channels, from -128~127

### GRAY颜色模型

灰度图像模型，只有单通道

常用的RGB模型转成灰度图方式$GRAY=0.3R+0.59G+0.11B$

### 不同颜色模型之间的转换

**cvtColor**函数原型

- src：原始图像
- dst：目标图像
- code：颜色空间转换的标志
- dstCn：目标图像中的通道数，为0则自动从src和代码中导出

```c++
void cv::cvtColor(InputArray src,
                  OutputArray dst,
                  int code,
                  int dstCn = 0
                 )
```

***Warning***：如果转换过程中添加了alpha通道，则将其值设置为相应通道范围的最大值

cvtColor函数颜色空间转换常用标志表

![cvtColor](https://s1.ax1x.com/2020/07/28/aVQvh8.jpg)

i.e.

```c++
#include <iostream>
#include <vector>
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;

int main() {
    Mat img = imread("lena.png");
    if (img.empty()) {
        cout << "Failed to load img" << endl;
        return -1;
    }
    Mat gray, HSV, YUV, Lab, img32;
    img.convertTo(img32, CV_32F, 1.0 / 255); //turn CV_8U into CV_32F
    //img32.convertTo(img,CV_8U,255); //turn CV_32F into CV_8U
    cvtColor(img32, HSV, COLOR_BGR2HSV);
    cvtColor(img32, YUV, COLOR_BGR2YUV);
    cvtColor(img32, Lab, COLOR_BGR2Lab);
    cvtColor(img32, gray, COLOR_BGR2GRAY);

    imshow("original img", img32);
    imshow("HSV", HSV);
    imshow("YUV", YUV);
    imshow("Lab", Lab);
    imshow("GRAY", gray);
    waitKey(0);
    return 0;
}
```

效果图

![cvtColor图](https://s1.ax1x.com/2020/07/29/aVUhqS.png)



介绍下刚刚用到的数据类型转换函数 **convertTo**

- m：转换后输出的图像
- rtype：转换图像的数据类型
- alpha：转换过程中的缩放因子
- beta：转换过程中的偏置因子

```c++
void cv::Mat::convertTo(OutputArray m,
                        int rtype,
                        double alpha = 1,
                        double beta = 0
                       )
```

第三、四个参数用于声明两个数据类型之间的转换关系，的具体转换形式如下式
$$
m(x,y)=saturate\_cast<rtype>(\alpha (*this)(x,y)+\beta)
$$
通过转换公式可以知道，该转换方式就是将原有数据进行线性转换，并按照指定的数据类型输出。根据其转换规则也可以知道，该函数还能实现同种数据类型之间的线性变换。



## 多通道分离与合并

### 多通道分离函数 split

函数原型

- src：待分离图像
- mvbegin：分离后的单通道图像，为数组形式，数组大小需要与通道数相等
- m：待分离图像
- mv：分离后的单通道图像，为向量(vector)形式

```c++
void cv::split(const Mat& src,
               Mat* mvbegin
              )
    
void cv::split(InputArray m,
               OutputArrayOfArrays mv
              )
```

分离原理如下式
$$
mv[c](I)=src(I)_c
$$

### 多通道合并函数 merge

函数原型

- mv（第一种重载）：需要合并的图片数组，其中每个图像必须尺寸、数据类型相同
- count：输入的图像数组的长度，必须大于0
- mv（第二种重载）：需要合并的图像向量(vector)，其中每个图像必须尺寸、数据类型相同
- dst：合并后输出的图像，与mv[0]具有相同的尺寸和数据类型，通道数等于所有输入图像的通道数总和

```c++
void cv::merge(const Mat* mv,
               size_t count,
               OutputArray dst
              )
    
void cv::merge(InputArrayOfArrays mv,
               OutputArray dst
              )
```



### 图像多通道分离与合并例程

该程序分离了HSV和RGB图像

```c++
#include <iostream>
#include <vector>
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;

int main() {
    Mat img = imread("lena.png");
    if (img.empty()) {
        cout << "Failed to load img" << endl;
        return -1;
    }
    Mat HSV;
    cvtColor(img, HSV, COLOR_RGB2HSV);
    Mat imgs0, imgs1, imgs2; //to store results in array type
    Mat imgv0, imgv1, imgv2; //to store results in vector type
    Mat results0, results1, results2; //multi-channels merge results

    //input array type to do split and merge
    Mat imgs[3];
    split(img, imgs);
    imgs0 = imgs[0];
    imgs1 = imgs[1];
    imgs2 = imgs[2];
    imshow("RGB-B channel", imgs0);
    imshow("RGB-G channel", imgs1);
    imshow("RGB-R channel", imgs2);
    imgs[2] = img;

    merge(imgs, 3, results0); //make num of channels in array be different
//    imshow("result0",results0); //imshow cannot show img with more than 4 channels,so see result0 in ImageWatch

    // show G-channel situation
    Mat zero = cv::Mat::zeros(img.rows, img.cols, CV_8UC1);
    imgs[0] = zero;
    imgs[2] = zero;
    merge(imgs, 3, results1);
    imshow("result1", results1);

    //input vector to do split and merge
    vector<Mat> imgv;
    split(HSV, imgv);
    imgv0 = imgv.at(0);
    imgv1 = imgv.at(1);
    imgv2 = imgv.at(2);
    imshow("HSV-H channel", imgv0);
    imshow("HSV-S channel", imgv1);
    imshow("HSV-V channel", imgv2);

    imgv.push_back(HSV); //make num of channels in vector be different
    merge(imgv, results2);
//    imshow("result2",results2); //imshow cannot show img with more than 4 channels,so see result2 in ImageWatch

    waitKey(0);
    return 0;

}
```

效果图：

![mySplitAndMerge](https://s1.ax1x.com/2020/07/29/aVr3yn.png)

# 图像像素处理



## 图像像素统计

### 寻找图像像素最大值与最小值

**minMaxLoc**函数原型

- src：需要进行寻找的图像或矩阵，必须是单通道矩阵
- minVal：最小值
- maxVal：最大值
- minLoc：最小值坐标
- maxLoc：最大值坐标
- mask：掩模，用于设置在指定区域寻找最值

```c++
Mat cv::minMaxLoc(InputArray src,
                  double* minVal,
                  double* maxVal = 0,
                  Point* minLoc = 0,
                  Point* maxLoc = 0;
                  InputArray mask = noArray()
                 )
```

***Warning***：如果有多个最大最小值会返回从左向右方向第一个，同时输入参数时一定要添加取地址符



**Point**是一个新的数据类型，用于表示图像的像素坐标，由于图像左上角是坐标原点，故而 **Point(x,y)**表示第**x**列第**y**行

该函数只能接收单通道图像，如果是多通道必须先用**reshape**方法把多通道变成单通道，或者分别寻找每个通道的最值，然后进行比较，寻找全局最值

**reshape**函数原型

- cn：转换后矩阵的通道数
- rows：转换后矩阵行数，如果参数为0，则转换后行数与转换前相同

```c++
Mat cv::Mat::reshape(int cn,
                     int rows = 0
                    )
```



### 计算图像的平均值和标准差

**mean**函数原型

- src：待求平均值矩阵
- mask：掩模，用于设置在指定区域寻找平均值

```c++
cv::Scalar cv::mean(InputArray src,
                    InputArray mask = noArray()
                   )
```

计算原理如下式：
$$
N=\sum_{I,mask(I)\neq0}1 \\
M_c = (\sum_{I,mask(I)}src(I)_c)/N
$$
where $M_c$ is average value of $c^{th}$ channel，$src(I)_c$ is grayscale value of $c^{th}$ channel.



**meanStdDev**函数原型

- src：待求矩阵
- mean：图像每个通道的平均值，参数为Mat类型变量
- stddev：图像每个通道的标准差，参数为Mat类型变量
- mask：掩模，用于设置在指定区域寻找

```c++
void cv::meanStdDev(InputArray src,
                    OutputArray mean,
                    OutputArray stddev,
                    InputArray mask = noArray()
                   )
```

计算原理如下式
$$
N=\sum_{I,mask(I)\neq0}1 \\
M_c = (\sum_{I,mask(I)}src(I)_c)/N \\
stddev_c = \sqrt{\sum_{I,mask(I)}(src(I)_c-M_c)^2/N}
$$


## 两图像间的像素操作

### 两幅图像的比较运算

**max**和**min**函数原型

- src1：第一个图像矩阵
- src2：第二个图像矩阵，尺寸和通道数以及数据类型都需要与src1一致
- dst：保留对应位置较大(较小)灰度值后的图像矩阵，尺寸、通道数和数据类型与src1一致

```c++
void cv::max(InputArray src1,
             INputArray src2,
             OutputArray dst
            )
    
void cv::min(InputArray src1,
             INputArray src2,
             OutputArray dst
            )
```



### 两幅图像的逻辑运算

如果像素数据类型是二值，那不必多说

但是与上CV_8U这样的8位数据，会先转换到二进制再进行运算，特此说明

**逻辑运算**函数原型

- src1：第一个图像矩阵
- src2：第二个图像矩阵，尺寸等属性要求与src1一致
- dst：输出结果矩阵，尺寸等属性要求与src1一致
- mask：掩模，用于设置在指定区域进行运算

```c++
void cv::bitwise_and(InputArray src1,
                     InputArray src2,
                     OutputArray dst,
                     InputArray mask = noArray()
                    )
    
void cv::bitwise_or(InputArray src1,
                     InputArray src2,
                     OutputArray dst,
                     InputArray mask = noArray()
                    )
    
void cv::bitwise_xor(InputArray src1,
                     InputArray src2,
                     OutputArray dst,
                     InputArray mask = noArray()
                    )
    
void cv::bitwise_not(InputArray src1,
                     OutputArray dst,
                     InputArray mask = noArray()
                    )
```



## 图像二值化

### threshold函数

**threshold**函数原型

- src：待二值化图像，只能是CV_8U or CV_32F，通道数目要求与二值化方法有关
- dst：二值化后的图像
- thresh：二值化阈值
- maxval：二值化过程中最大值，它只在THRESH_BINARY和THRESH_BINARY_INV两种方法中才使用
- type：选择图像二值化方法的标志

```c++
double cv::threshold(InputArray src,
                     OutputArray dst,
                     double thresh,
                     double maxval,
                     int type
                    )
```

| 标志参数          | 简记 | 作用                                |
| ----------------- | ---- | ----------------------------------- |
| THRESH_BINARY     | 0    | 灰度值大于阈值的为最大值，其他值为0 |
| THRESH_BINARY_INV | 1    | 灰度值大于阈值的为0，其他值为最大值 |
| THRESH_TRUNC      | 2    | 灰度值大于阈值的为阈值，其他值不变  |
| THRESH_TOZERO     | 3    | 灰度值大于阈值的不变，其他值为0     |
| THRESH_TOZERO_INV | 4    | 灰度值大于阈值的为0，其他值不变     |
| THRESH_OTSU       | 8    | 大津法自动寻求全局阈值              |
| THRESH_TRIANGLE   | 16   | 三角形法自动寻求全局阈值            |

接下来详细介绍每种标志对应的二值化原理和需要的参数

### THRESH_BINARY and THRESH_BINARY_INV

$$
THRESH\_BINARY(x,y)=\left\{
\begin{array}{rcl}
&maxval&when\,src(x,y)>thresh \\
&0&other\,situation
\end{array}\right. \\
THRESH\_BINARY\_INV(x,y)=\left\{
\begin{array}{rcl}
&0&when\,src(x,y)>thresh \\
&maxval&other\,situation
\end{array}\right.
$$



### THRESH_TRUNC

该方法相当于重新给图像灰度值设定一个新的最大值，如下式
$$
THRESH\_TRUNC(x,y)=\left\{
\begin{array}{rcl}
&threshold&when\,src(x,y)>thresh \\
&src(x,y)&other\,situation
\end{array}\right.
$$


### THRESH_TOZERO and THRESH_TOZERO_INV

该方法不需要使用参数maxval，这点很容易从式子中理解
$$
THRESH\_TOZERO(x,y)=\left\{
\begin{array}{rcl}
&src(x,y)&when\,src(x,y)>thresh \\
&0&other\,situation
\end{array}\right. \\
THRESH\_TOZERO\_INV(x,y)=\left\{
\begin{array}{rcl}
&0&when\,src(x,y)>thresh \\
&src(x,y)&other\,situation
\end{array}\right.
$$

### THRESH_OTSU and THRESH_TRIANGLE

这两种标志是获取阈值的而非比较阈值的方法，这两个标志可以和之前的一起使用，用“ | ”分隔即可

如果函数最后一个参数设置了这个标志，那么第三个参数thresh的输入将会无效，该由系统给出

值得注意的是，目前只支持输入CV_8UC1类型图像



### adaptiveThreshold函数

**adaptiveThreshold**函数原型

- src：待二值化图像，只能是CV_8UC1
- dst：二值化后的图像
- maxval：二值化过程中最大值，它只在THRESH_BINARY和THRESH_BINARY_INV两种方法中才使用
- adaptiveMethod：自适应确定阈值的方法，分为均值法ADAPTIVE_THRESH_MEAN_C和高斯法ADAPTIVE_THRESH_GAUSSIAN_C两种
- thresholdType：选择图像二值化方法的标志，这里只能是THRESH_BINARY and THRESH_BINARY_INV
- blockSize：自适应确定阈值的像素领域大小，一般是3，5，7的奇数
- C：从平均值或者加权平均值中减去的常数，可以为正也可以为负

```c++
void cv::adaptiveThreshold(InputArray src,
                           OutputArray dst,
                           double maxValue,
                           int adaptiveMethod,
                           int thresholdType,
                           int blockSize,
                           double C
                          )
```



## LUT

前面介绍的阈值比较方法中都只有一个阈值，如果需要多个阈值进行比较，就需要用到显示查找表(Look-Up-Table, LUT)

LUT简单来说就是个像素灰度值的映射表

**LUT**函数原型

- src：输入图像矩阵，数据类型只能是CV_8U
- lut：256个像素灰度值的查找表，单通道或者与src通道数相同
- dst：输出图像矩阵，尺寸与src一致，数据类型与lut一致



# 图像变换



## 图像连接

### vconcat函数实现图像上下连接

**vconcat**函数原型1

- src：Mat矩阵类型数组，所有Mat列数必须相同
- nsrc：数组中Mat类型数据数目
- dst：连接后的Mat矩阵

```c++
void cv::vconcat(const Mat* src,
                 size_t nsrc,
                 OutputArray dst
                )
```

**vconcat**函数原型2

- src1：第一个需要连接的Mat矩阵
- src2：第二个需要连接的Mat矩阵
- dst：连接后的Mat矩阵

```c++
void cv::vconcat(InputArray src1,
                 InputArray src2,
                 OutputArray dst
                )
```



### hconcat函数实现图像左右连接

与**vconcat**类似，不赘述



## 图像尺寸变换

**resize**函数原型

- src：输入图像
- dst：输出图像
- dsize：输出图像的尺寸
- fx：水平轴的比例因子
- fy：垂直轴的比例因子
- interpolation：插值方法的标志

```c++
void cv::resize(InputArray src,
                OutputArray dst,
                double dsize,
                double fx = 0,
                double fy = 0,
                int interpolation = INTER_LINEAR
               )
```

***Hint***：dsize参数和fx,fy参数在实际使用时一般选一类填就可以了，当根据两类参数计算出来的输出图像尺寸不一致时，以dsize设置的为准，这两类参数的关系如式$dsize=Size(round(fx*src.cols),round(fy*src.rows))$

插值方法标志表

|      标志参数      | 简记 |                             作用                             |
| :----------------: | :--: | :----------------------------------------------------------: |
|   INTER_NEAREST    |  0   |                          最近邻插值                          |
|    INTER_LINEAR    |  1   |                          双线性插值                          |
|    INTER_CUBIC     |  2   |                          双三次插值                          |
|     INTER_AREA     |  3   | 使用像素区域关系重新采样，首选用于图像缩小，图像放大时效果与INTER——NEAREST相似 |
|   INTER_LANCZOS4   |  4   |                        Lanczos插值法                         |
| INTER_LINEAR_EXACT |  5   |                      位精确双线性插值法                      |
|     INTER_MAX      |  7   |                        用掩码进行插值                        |

***Hint***：一般来讲，缩小图片使用INTER_AREA；放大图片使用INTER_CUBIC和INTER_LINEAR，前者计算速度慢，但是效果较好~~其实后者效果相对也挺理想的~~



## 图像翻转变换

**flip**函数原型

- src：输入图像
- dst：输出图像
- flipCode：翻转方式标志 数值大于0表示绕y轴进行翻转；数值等于0表示绕x轴翻转；数值小于0表示绕两个轴都翻转

```c++
void cv::flip(InputArray src,
              OutputArray dst,
              int flipCode
             )
```



## 图像仿射变换

实现图像旋转首先需要确定旋转中心和角度，进而确定旋转矩阵，最终通过仿射变换实现图像旋转

针对该流程，OpenCV4提供了**getRotationMatrix2D**函数用于计算旋转矩阵， **warpAffine**函数用于实现图像仿射变换

### getRotationMatrix2D函数

函数原型

- center：旋转的中心位置
- angle：角度，单位为度，正值为逆时针旋转
- scale：两个轴的比例因子，实现旋转时缩放，不缩放则输入1

```c++
Mat cv::getRotationMatrix2D(Point2f center,
                            double angle,
                            double scale
                           )
```

该函数生成的旋转矩阵与旋转中心及角度的关系如下式
$$
Rotation=\left[\begin{array}{ccc}
\alpha & \beta &(1-\alpha)*center.x-\beta*center.y\\
-\beta & \alpha &\beta*center.x+(1+\alpha)*center.y \end{array} \right] \\
where:\\
\, \alpha=scale*cos(angle) \\
\beta=scale*sin(angle)
$$


### wrapAffine函数

函数原型

- src：输入图像
- dst：输出图像
- M：$2\times3$的变换矩阵，可以用之前求取的旋转矩阵
- dsize：输出图像的尺寸
- flags：插值方法标志，相比于图像尺寸变换多了两个
- borderMode：像素边界外推方法的标志
- borderValue：填充边界使用的数值，默认为0

```c++
void cv::wrapAffine(InputArray src,
                    OutputArray dst,
                    Size dsize,
                    int flags = INTER_LINEAR
                    int borderMode = BORDER_CONSTANT
                    const Scalar& borderValue = Scalar()
                   )
```

图像仿射变换插值方法的两个额外标志

| 方法标志           | 简记 | 作用                                                         |
| ------------------ | ---- | ------------------------------------------------------------ |
| WARP_FILL_OUTLIERS | 8    | 填充所有输出图像的像素，如果部分像素落在输入图像的边界外，则他们的值设定为fillval |
| WARP_INVERSE_MAP   | 16   | 设置为M输出图像到输入图像的反变换                            |

边界外推方法标志

![边界外推方法标志](https://s1.ax1x.com/2020/07/29/aeaBOP.jpg)



### 仿射变换的基本数学概念

仿射变换可以表示为**线性变换**和**平移变换**的叠加，其数学表示是先乘以一个**线性变换矩阵**再加上一个**平移向量**，其中**线性变换矩阵**为$2\times2$的矩阵，**平移向量**为$2\times1$的向量. 至此我们可以理解函数为什么需要输入$2\times3$的**变换矩阵M**. 假设现在有一个**线性变换矩阵A**和一个**平移向量B**，两者与**函数输入矩阵M**的关系如下式
$$
M=[A\quad B]=\left[\begin{array}{ccc}
a_{00} & a_{01} & b_{00}\\
a_{10} & a_{11} & b_{10} \end{array} \right] \\
$$
根据**线性变换矩阵A**和**平移向量B**，以及图像像素值$[x\quad y]^T$，仿射变换的数学原理可以用如下式表示
$$
T=A\left[ \begin{array}{c}
x\\
y \end{array}\right]+B
$$

### getAffineTransform函数

仿射变换又称三点变换，如果知道变换前后两幅图像中三个像素点坐标的对应关系，就可以求的仿射变换的变换矩阵M~~但凡线代没挂科都能理解~~

OpenCV提供了这一函数

函数原型

- src[ ]：源图像中的三个像素坐标
- dst[ ]：目标图像中的三个像素坐标

```c++
Mat cv::getAfflineTransform(const Point2f src[],
                            const Point3f dst[]
                           )
```



## 图像透视变换

透视变换中前后的变换关系可以用一个$3\times3$变换矩阵表示，该矩阵通过两幅图像中4个对应点的坐标求取，因此透视变换又叫四点变换

与仿射变换类似，OpenCV提供了相关的函数

透视变换原理示意图

<img src="https://s1.ax1x.com/2020/07/29/aerh6J.jpg" alt="透视变换原理示意图" style="zoom: 25%;" />



### getPerspectiveTransform函数

函数原型

- src[ ]：源图像中的三个像素坐标
- dst[ ]：目标图像中的三个像素坐标
- solveMethod：选择计算透视变换矩阵方法的标志

```c++
Mat cv::getPerspectiveTransform(const Point2f src[],
                                const Point3f dst[],
                                int solveMethod = DECOMP_LU
                               )
```

计算透视变换矩阵方法标志表

|    标志参数     | 简记 |                   作用                   |
| :-------------: | :--: | :--------------------------------------: |
|    DECOMP_LU    |  0   |         最佳主轴元素的高斯消元法         |
|   DECOMP_SVD    |  1   |              奇异值分解方法              |
|   DECOMP_EIG    |  2   |              特征值分解方法              |
| DECOMP_CHOLESKY |  3   |              Cholesky分解法              |
|    DECOMP_QR    |  4   |                 QR分解法                 |
|  DECOMP_NORMAL  |  16  | 使用正规方程公式，可以与其他标志一起使用 |



### warpPerspective函数

函数原型

- src：输入图像
- dst：输出图像
- M：$3\times3$的变换矩阵
- dsize：输出图像的尺寸
- flags：插值方法标志，相比于图像尺寸变换多了两个
- borderMode：像素边界外推方法的标志
- borderValue：填充边界使用的数值，默认为0

```c++
void cv::warpPerspective(InputArray src,
                         OutputArray dst,
                         InputArray M,
                         Size dsize,
                         int flags = INTER_LINEAR
                         int borderMode = BORDER_CONSTANT
                         const Scalar& borderValue = Scalar()
                        )
```



i.e.倾斜二维码图片复原

```c++
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;

int main() {
    Mat img0 = imread("noobcvqr.png");
    if (img0.empty()) {
        cout << "load img failed" << endl;
        return -1;
    }

    Mat rotation, img_warp; //rotation matrix and result matrix

    Point2f src_points[4], dst_points[4]; //four groups of points to calculate rotation matrix
    src_points[0] = Point2f(94.0, 374.0);
    src_points[1] = Point2f(507.0, 380.0);
    src_points[2] = Point2f(1.0, 623.0);
    src_points[3] = Point2f(627.0, 627.0);
    dst_points[0] = Point2f(0.0, 0.0);
    dst_points[1] = Point2f(627.0, 0.0);
    dst_points[2] = Point2f(0.0, 627.0);
    dst_points[3] = Point2f(627.0, 627.0);

    rotation = getPerspectiveTransform(src_points, dst_points);
    warpPerspective(img0, img_warp, rotation, img0.size());

    imshow("original", img0);
    imshow("res", img_warp);
    waitKey(0);
    return 0;
}
```



## 极坐标变化

将图像在直角坐标系与极坐标系中互相变换，它可以把圆形图像变换成矩形图像，常用于处理钟表、圆盘等图像

圆形图案边缘上的文字经过极坐标变换后可以垂直地排列在新图像的边缘，便于对文字进行识别

**warpPolar**函数原型

- src：源图像
- dst：输出图像
- dsize：目标图像大小
- center：极坐标原点坐标
- maxRadius：变换时边界圆的半径，它也决定了逆变换时的比例参数
- flags：插值方法与极坐标映射方法标志，两个方法之间用“ + ”或“ | ”连接

```c++
void cv::warpPolar(InputArray src,
                   OutputArray dst,
                   Size dsize,
                   Point2f center,
                   double maxRadius,
                   int flags
                  )
```

插值方法与之前尺寸变换的一致

坐标映射方法表如下

| 标志参数          | 作用             |
| ----------------- | ---------------- |
| WARP_POLAR_LINEAR | 极坐标变换       |
| WARP_POLAR_LOG    | 半对数极坐标变换 |
| WARP_INVERSE_MAP  | 逆变换           |



# 在图像上绘制几何图形



## 绘制圆形

**circle**函数原型

- img：需要绘制的图像
- center：圆心位置坐标
- color：半径，单位是像素
- thickness：轮廓宽度，如果数值为负，会绘制一个实心圆
- lineType：边界类型，可取值FILLED、LINE_4、LINE_8、LINE_AA
- shift：中心坐标和半径数值的小数位数

```c++
void cv::circle(InputOutputArray img,
                Point center,
                int radius,
                const Scalar& color,
                int thickness = 1,
                int lineType = LINE_8,
                int shift = 0
               )
```



## 绘制直线

**line**函数原型

- pt1：起点坐标
- pt2：终点坐标
- color：颜色，三通道表示

```c++
void cv::line(InputOutputArray img,
              Point pt1,
              Point pt2,
              const Scalar& color,
              int thickness = 1,
              int lineType = LINE_8,
              int shift = 0
             )
```

## 绘制椭圆

**ellipse**函数原型

- center：中心坐标
- axes：主轴大小一半
- angle：旋转角度，单位是度
- startAngle：弧起始角度
- endAngle：弧终止角度

```c++
void cv::ellipse(InputOutputArray img,
                 Point center,
                 Size axes,
                 double angle,
                 double startAngle,
                 double endAngle,
                 const Scalar& color,
                 int thickness = 1,
                 int lineType = LINE_8,
                 int shift = 0
                )
```



**ellipse2Poly**函数原型

- delta：后续折线顶点之间的角度，定义了近似精度
- pts：椭圆边缘像素坐标向量集合

该函数不将椭圆输出到图像中，而是通过vector将椭圆边缘的坐标点存储起来

```c++
void cv::ellipse2Poly(Point center,
                 	  Size axes,
                 	  int angle,
                 	  int arcStart,
                 	  int arcEnd,
                      int delta,
                      std::vector< Point >& pts
                	 )
```



## 绘制多边形

**rectangle**函数原型

- pt1：矩形的一个顶点
- pt2：与pt1相对的另一个顶点
- rec：矩形左上角顶点和长宽

```c++
void cv::rectangle(InputOutputArray img,
                   Point pt1,
                   Point pt2,
                   const Scalar& color,
                   int thickness = 1,
                   int lineType = LINE_8,
                   int shift = 0
                  )
    
void cv::rectangle(InputOutputArray img,
                   Rect rec,
                   const Scalar& color,
                   int thickness = 1,
                   int lineType = LINE_8,
                   int shift = 0
                  )   
```

**Rect**变量在OpenCV中用于表示矩形，该类型定义的格式为**Rect(像素的x坐标，像素的y坐标，矩形的宽，矩形的高)**



**fillPoly**函数原型

- pts：多边形顶点数组
- npts：每个多边形顶点数组中顶点的个数
- ncontours：绘制多边形的个数
- offset：所有顶点的可选偏移

```c++
void cv::fillPoly(InputOutputArray img,
                  const Point** pts,
                  const int* npts,
                  int ncontours,
                  const Scalar& color,
                  int lineType = LINE_8,
                  int shift = 0,
                  Point offset = Point()
                 )
```



## 文字生成

**putText**函数原型

- text：输出到图像中的文字
- org：图像中文字字符串的左下角像素坐标
- fontFace：字体类型的选择标志
- fontScale：字体大小
- bottomLeftOrigin：图像数据原点位置，默认为左上角，true则为左下角

```c++
void cv::putText(InputOutputArray img,
                 const String& text,
                 Point org,
                 int fontFace
                 double fontScale,
                 Scalar color,
                 int thickness = 1,
                 int lineType = LINE_8,
                 bool bottomLeftOrigin = false
                )
```

| 标志                        | 取值 | 含义                         |
| --------------------------- | ---- | ---------------------------- |
| FONT_HERSHEY_SIMPLEX        | 0    | 正常大小的无衬线字体         |
| FONT_HERSHEY_PLAIN          | 1    | 小尺寸的无衬线字体           |
| FONT_HERSHEY_DUPLEX         | 2    | 正常大小的较复杂的无衬线字体 |
| FONT_HERSHEY_COMPLEX        | 3    | 正常大小的衬线字体           |
| FONT_HERSHEY_TRIPLEX        | 4    | 正常大小的较复杂的衬线字体   |
| FONT_HERSHEY_COMPLEX_SAMLL  | 5    | 小尺寸的衬线字体             |
| FONT_HERSHEY_SCRIPT_SIMPLEX | 6    | 手写风格字体                 |
| FONT_HERSHEY_SCRIPT_COMPLEX | 7    | 复杂的手写风格字体           |
| FONT_ITALIC                 | 16   | 斜体字体                     |



# 感兴趣区域ROI

简单来说就是截图

```c++
img(Rect(p.x, p.y, width, height))
img(Range(rows_start, rows_end), Range(cols_start, cols_end))
```

以上两种截图方式都不难理解

这里主要说一个深拷贝函数

```c++
void cv::Mat::copyTo(OutputArray m) const
    
void cv::Mat::copyTo(OutputArray m,
                     InputArray mask,
                    ) const
void cv::copyTo(InputArray src,
                OutputArray dst,
                InputArray mask
               )
```





# 图像金字塔

通过多个分辨率表示图像的一种有效且简单的结构

一个图像金字塔是一系列以金字塔形状排列、分辨率逐步降低的图像集合，底部是待处理图像的高分辨率表示，顶部是低分辨率表示

## 高斯“金字塔”

解决尺度不准确性的一种常用办法

通过下采样不断缩小图像尺寸，一般情况下，每往上一层就会通过下采样缩小一次图像的尺寸，通常为一半

OpenCV4提供了**pyrDown**函数专门用于图像下采样计算

函数原型

- src：待下采样图像
- dst：输出图像
- dsize：输出尺寸，可以默认
- borderType：像素边界外推方法标志

```c++
void cv::pyrDown(InputArray src,
                 OutputArray dst,
                 const Size& dstsize = Size(),
                 int borderType = BORDER_DEFAULT
                )
```

无论输出尺寸是多少，都应该满足下式
$$
\left\{
\begin{array}{rcl}
&|dstsize.width2-src.cols|&\leqslant2\\
&|dstsize.height2-src.rows|&\leqslant2
\end{array}\right. \\
$$
该函数首先将原始图像与内核矩阵(如下)进行卷积，之后通过不使用偶数行和列的方式对图像进行下采样
$$
k=\frac{1}{256}
\left[\begin{array}{ccccc}
1&4&6&4&1\\
4&6&24&6&4\\
6&24&36&24&6\\
4&16&24&16&4\\
1&4&6&4&1
\end{array} \right]
$$
该函数功能与**resize**函数将图像尺寸缩小一样，但是使用算法不同

<img src="https://img-blog.csdnimg.cn/20200103112744969.png" alt="高斯金字塔" style="zoom:150%;" />

## 拉普拉斯“金字塔”

拉普拉斯金字塔与高斯金字塔相反，拉普拉斯通过上层小尺寸构建下层大尺寸，它具有预测残差的功能，需要与高斯金字塔一起使用

<img src="https://img-blog.csdnimg.cn/20200103113414447.png" alt="拉普拉斯金字塔" style="zoom:150%;" />

对于上采样操作，OpenCV4提供了**pyrUp**函数，它与下采样函数极度类似，此处不再赘述

函数原型

```c++
void cv::pyrUp(InputArray src,
                 OutputArray dst,
                 const Size& dstsize = Size(),
                 int borderType = BORDER_DEFAULT
                )
```





i.e.尝试构建一下

```c++
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;

int main() {
    Mat img = imread("lena.png");
    vector<Mat> Gauss, Laplace;
    int level = 3;
    //Gauss Pyramid
    Gauss.push_back(img);
    for (int i = 0; i < level; ++i) {
        Mat gauss;
        pyrDown(Gauss[i], gauss);
        Gauss.push_back(gauss);
    }
    //show
    for (int j = 0; j < Gauss.size(); ++j) {
        imshow("Gauss" + to_string(j), Gauss[j]);
    }
    
	//Laplace Pyramid
    for (int k = Gauss.size() - 1; k > 0; --k) {
        Mat laplace, upGauss;
        if (k == Gauss.size() - 1) {
            Mat down;
            pyrDown(Gauss[k], down);
            pyrUp(down, upGauss);
            laplace = Gauss[k] - upGauss;
            Laplace.push_back(laplace);
        }
        pyrUp(Gauss[k], upGauss);
        laplace = Gauss[k - 1] - upGauss;
        Laplace.push_back(laplace);
    }
    //show
    for (int l = 0; l < Laplace.size(); ++l) {
        imshow("Laplace" + to_string(l), Laplace[l]);
    }
    
    waitKey(0);
    return 0;
}
```

效果图

![金字塔](https://s1.ax1x.com/2020/07/29/am83Yn.png)

# 窗口交互操作



## 图像窗口滑条

**createTrackerbar**函数原型

- trackbarname：滑动条名称
- winname：创建滑动条窗口名称
- value：指向整数变量的指针，该指针指向的值反映滑块位置
- count：滑动条的最大取值
- onChange：每次滑块更改位置时要调用的函数的指针，其中函数原型是 void Foo(int, void*);，第一个参数是轨迹栏位置，第二个参数是用户数据. 如果回调是NULL指针，则不会调用任何回调，而只是更新数值
- userdata：传递给回调函数的可选参数

```c++
int cv::createTrackbar(const String& trackbarname,
                       const STring& winname,
                       int* value,
                       int count,
                       TracebarCallback onChange = 0,
                       void* userdata = 0
                      )
```



i.e. 滑动条改变亮度

```c++
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;

int value;

static void callBack(int, void *);

Mat img1, img2;

int main() {
    img1 = imread("lena.png");
    imshow("change brightness", img1);
    value = 100;

    createTrackbar("brightness percentage", "change brightness", &value, 600, callBack, 0);
    waitKey(0);
    return 0;
}

static void callBack(int, void *) {
    float a = value / 100.0;
    img2 = img1 * a;
    imshow("change brightness", img2);
}
```



## 鼠标响应

有时需要在图像中标记出重要区域，故而需要鼠标响应

**setMouseCallback**函数原型

- winname：添加鼠标响应的窗口的名字
- onMouse：鼠标响应的回调函数
- userdata：传递给回调函数的可选参数

```c++
void cv::setMouseCallback(const String& winname,
                          MouseCallback onMouse,
                          void* userdata = 0
                         )
```

**MouseCallback**类型原型

- event：鼠标响应事件，参数为EVENT_*形式
- x：鼠标指针在图像坐标系中的x坐标
- y：鼠标指针在图像坐标系中的y坐标
- flags：鼠标响应标志，参数为EVENT_FLAG_*形式
- userdata：传递给回调函数的可选参数

```c++
typedef void(* cv::MouseCallback)(int event,
                                  int x,
                                  int y,
                                  int flags,
                                  void* userdata
                                 )
```

event可选参数表

| 标志参数            | 简记 | 含义       |
| :-----------------: | :--: | :--------: |
| EVENT_MOUSEMOVE     | 0    | 移动       |
| EVENT_LBUTTONDOWN   | 1    | 左键点击 |
| EVENT_RBUTTONDOWN   | 2    | 右键点击 |
| EVENT_MBUTTONDOWN   | 3    | 中键点击 |
| EVENT_LBUTTONUP     | 4    | 释放左键 |
| EVENT_RBUTTONUP     | 5    | 释放右键 |
| EVENT_MBUTTONUP     | 6    | 释放中键 |
| EVENT_LBUTTONDBLCLK | 7    | 双击左键 |
| EVENT_RBUTTONDBLCLK | 8    | 双击右键 |
| EVENT_MBUTTONDBLCLK | 9    | 双击中键 |
| EVENT_MOUSEWHEEL    | 10   | 正值表示向前滚动，负值向后 |
| EVENT_MOUSEHWHEEL   | 11   | 正值表示向左滚动，负值向右 |

flags可选参数表

| 标志参数            | 简记 | 含义         |
| ------------------- | ---- | ------------ |
| EVENT_FLAG_LBUTTON  | 1    | 按住左键拖拽 |
| EVENT_FLAG_RBUTTON  | 2    | 按住右键拖拽 |
| EVENT_FLAG_MBUTTON  | 4    | 按住中键拖拽 |
| EVENT_FLAG_CTRLKEY  | 8    | Ctrl         |
| EVENT_FLAG_SHIFTKEY | 16   | Shift        |
| EVENT_FLAG_ALTKEY   | 32   | Alt          |



i.e. 在图像上绘制鼠标轨迹

```c++
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;

Mat img, imgPoint; //global img
Point prePoint; //pre-location of mouse, use it to draw line
void mouse(int event, int x, int y, int flags, void *);

int main() {

    img = imread("lena.png");
    img.copyTo(imgPoint);
    imshow("window1", img);
    imshow("window2", imgPoint);
    setMouseCallback("window1", mouse, 0);

    waitKey(0);
    return 0;
}

void mouse(int event, int x, int y, int flags, void *) {
    if (event == EVENT_RBUTTONDOWN) { //click right
        cout << "Please click left" << endl;
    }
    if (event == EVENT_LBUTTONDOWN) {
        prePoint = Point(x, y);
        cout << "start location" << prePoint << endl;
    }
    if (event == EVENT_MOUSEMOVE && (flags & EVENT_FLAG_LBUTTON)) { //left down and move
        //show trace by change pixel
        imgPoint.at<Vec3b>(y, x) = Vec3b(0, 0, 255);
        imgPoint.at<Vec3b>(y, x - 1) = Vec3b(0, 0, 255);
        imgPoint.at<Vec3b>(y, x + 1) = Vec3b(0, 0, 255);
        imgPoint.at<Vec3b>(y - 1, x) = Vec3b(0, 0, 255);
        imgPoint.at<Vec3b>(y + 1, x) = Vec3b(0, 0, 255);
        imshow("window2", imgPoint);

        //show trace by draw line
        Point pt(x, y);
        line(img, prePoint, pt, Scalar(0, 0, 255), 2, 5, 0);
        prePoint = pt;
        imshow("window1", img);
    }
}
```

效果图：

可以看出画直线的方法在鼠标快速移动下还有很好的表现，但是改变像素方式就不行会有断点

![鼠标响应](https://s1.ax1x.com/2020/07/29/amwZSH.png)