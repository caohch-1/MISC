---
title: OpenCV4-Image histogram and Template matching
categories:
- OpenCV
mathjax: true

---

基于《OpenCV4快速入门》 @冯振、郭延宁、吕跃勇

# 图像直方图的绘制

由于同一物体无论是旋转还是平移，在图像中都具有相同的灰度值，因此图像直方图具有平移不变性、放缩不变性等优点.

图像灰度值为横轴，灰度值个数或比例为纵轴.

通常情况下，灰度值代表亮暗程度，因此通过图像直方图可以分析图像亮暗对比度，并调整图像的亮暗程度

**calcHist**函数原型

- images：待统计直方图的图像数组，数据中所有图像应具有相同的尺寸和数据类型，并且数据类型只能是CV_8U, CV_16U和CV_32F中的一种，但是通道数可以不同
- nimages：输入图像的数量
- channels：需要统计的通道索引数组
- mask：操作掩码
- hist：输出的结果，是一个dims维度的数组
- dims：需要计算直方图的维度
- histSize：存放每个维度直方图的数组的尺寸
- ranges：每个图像通道中灰度值的取值范围
- uniform：直方图是否均匀的标志，默认是均匀
- accumulate：是否累积直方图的标志

```c++
void cv::calcHist(const Mat* images,
                  int nimages,
                  const int* channels,
                  InputArray mask,
                  OutputArray hist,
                  int dims,
                  const int* histSize,
                  const float** ranges,
                  bool uniform = true,
                  bool accumulate = false
                 )
```

***Hint***：由于该函数参数较多，建议使用时只统计单通道图像的灰度值分布，对于多通道图像，可以将图像每个通道分离后再统计

i.e.

```c++
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;

int main() {
    Mat img = imread("apple.jpg");
    Mat gray;
    cvtColor(img, gray, COLOR_BGR2GRAY);
    //set params
    int nimages = 1; //nimages
    const int channels[1] = {0}; //channels
    Mat mask = Mat(); //mask
    Mat hist; //hist
    int dims = 1; //dims
    const int histSize[1] = {256}; //histSize
    //ranges
    float inRanges[2] = {0, 255};
    const float *ranges[1] = {inRanges};
    //calculate
    calcHist(&gray, nimages, channels, mask, hist, dims, histSize, ranges);

    //draw hist
    int hist_width = 512;
    int hist_height = 400;
    int width = 2;
    Mat histImage = Mat::zeros(hist_height, hist_width, CV_8UC3);
    for (int i = 1; i <= hist.rows; ++i) {
        rectangle(histImage, Point(width * (i - 1), hist_height - 1),
                  Point(width * i - 1, hist_height - cvRound(hist.at<float>(i - 1) / 15)), Scalar(255, 255, 255), -1);
    }
    imshow("histImage", histImage);
    imshow("gray", gray);

    waitKey(0);
    return 0;
}
```



# 直方图操作

## 直方图归一化

**normalize**函数原型

- src：输入数组矩阵
- dst：输入与src相同大小的数组矩阵
- alpha：在范围归一化的情况下，归一化到下限边界的标准值
- beta：范围归一化的上限范围，不用于标准归一化
- norm_type：归一化过程中数据范数种类标志
- dtype：输出数据类型选择标志. 如果为负，与src一致，否则与src拥有相同通道数但是数据类型不同
- mask：掩码矩阵

```c++
void cv::normalize(InputArray src,
                   InputOutputArray dst,
                   double alpha = 1,
                   double beta = 0,
                   int norm_type = NORM_L2,
                   int dtype = -1,
                   InputArray mask = noArray()
                  )
```

norm_type参数常用标志，无论是否归一化或者采用哪种方式归一化，直方图的分布特性都不会改变

![norm_type](https://s1.ax1x.com/2020/07/30/auERGF.jpg)

## 直方图比较

**compareHist**函数原型

- H1：第一幅图像直方图
- H2：第二幅
- method：比较方法标志

```c++
double cv::compareHist(InputArray H1,
                       InputArray H2,
                       int method
                      )
```

以下是七种标志参数

| 标志参数              | 简记 | 名称或作用       | 备注                    |
| --------------------- | ---- | ---------------- | ----------------------- |
| HISTCMP_CORREL        | 0    | 相关法           | 1完全一致，0完全不相关  |
| HISTCMP_CHISQR        | 1    | 卡方法           | 0完全一致，越大越不相似 |
| HISTCMP_INTERSECT     | 2    | 直方图相交法     | 越大越相似              |
| HISTCMP_BHATTACHARYYA | 3    | 巴塔恰里雅距离法 | 0完全一致，越大越不相似 |
| HISTCMP_HELLINGER     | 3    | 与上相同         | 与上相同                |
| HISTCMP_CHISQR_ALT    | 4    | 替代卡方法       | 常用于纹理比较          |
| HISTCMP_KL_DIV        | 5    | 相对熵法         | 0完全一致，越大越不相似 |



# 直方图应用

## 直方图均衡化

如果一个图像的直方图都集中在一个区域，那么整体图像对比度比较小，不便于图像中纹理的识别

如果通过映射关系，将图像中的灰度值范围扩大，增加原来两个灰度值之间的插值，就可以提高图像对比度，进而将图像中的纹理显现出来

**equalizeHist**函数原型

- src：需要均衡化的CV_8UC1图像
- dst：输出图像

```c++
void cv::equalizeHist(InputArray src,
                      OutputArray dst
                     )
```





i.e.

```c++
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;

int main() {
    Mat img = imread("gearwheel.jpg");
    Mat gray, equalImg;
    cvtColor(img, gray, COLOR_BGR2GRAY);
    equalizeHist(gray, equalImg);
    imshow("original", gray);
    imshow("equalize", equalImg);
    waitKey(0);
    return 0;
}
```

效果图

![equalize](https://s1.ax1x.com/2020/07/30/aunfBj.png)



## 直方图匹配



将直方图映射成指定分布形式的算法称为直方图匹配

为了使两个图像直方图能够匹配，需要使用概率形式表示每个灰度值在图像像素中所占的比例，理想状况下，匹配操作后图像直方图分布形式应与目标分布一致，两者之间的累积概率分布也一致

寻找灰度值匹配的过程是直方图匹配算法的关键，可以通过构建原直方图累积概率与目标直方图累积概率之间的差值表，寻找原直方图中灰度值n的累积概率与目标直方图中所有灰度值累积概率差值的最小值

可以通过下面的例子轻松地理解这一过程

i.e. 

```c++
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;

void drawHist(Mat &hist, int type, string name) {
    int hist_w = 512;
    int hist_h = 400;
    int width = 2;
    Mat histImage = Mat::zeros(hist_h, hist_w, CV_8UC3);
    normalize(hist, hist, 1, 0, type, -1, Mat());
    for (int i = 1; i <= hist.rows; i++) {
        rectangle(histImage, Point(width * (i - 1), hist_h - 1),
                  Point(width * i - 1, hist_h - cvRound(20 * hist_h * hist.at<float>(i - 1)) - 1),
                  Scalar(255, 255, 255), -1);
    }
//    imshow(name, histImage);
}

int main() {
    Mat img1 = imread("histMatch.png");
    Mat img2 = imread("equalLena.png");

    Mat hist1, hist2;
    //calculate hist
    const int channels[1] = {0};
    float inRanges[2] = {0, 255};
    const float *ranges[1] = {inRanges};
    const int bins[1] = {256};
    calcHist(&img1, 1, channels, Mat(), hist1, 1, bins, ranges);
    calcHist(&img2, 1, channels, Mat(), hist2, 1, bins, ranges);
    //normalized hist
    drawHist(hist1, NORM_L1, "hist1");
    drawHist(hist2, NORM_L1, "hist2");

    //calculate accumulation probability
    float hist1_cdf[256] = {hist1.at<float>(0)};
    float hist2_cdf[256] = {hist2.at<float>(0)};
    for (int i = 1; i < 256; i++) {
        hist1_cdf[i] = hist1_cdf[i - 1] + hist1.at<float>(i);
        hist2_cdf[i] = hist2_cdf[i - 1] + hist2.at<float>(i);

    }

    //create accumulation probability error matrix
    float diff_cdf[256][256];
    for (int i = 0; i < 256; i++) {
        for (int j = 0; j < 256; j++) {
            diff_cdf[i][j] = fabs(hist1_cdf[i] - hist2_cdf[j]);
        }
    }

    //create LUT
    Mat lut(1, 256, CV_8U);
    for (int i = 0; i < 256; i++) {
        float min = diff_cdf[i][0];
        int index = 0;
        for (int j = 1; j < 256; j++) {
            if (min > diff_cdf[i][j]) {
                min = diff_cdf[i][j];
                index = j;
            }
        }
        lut.at<uchar>(i) = (uchar) index;
    }

    Mat result, hist3;
    LUT(img1, lut, result);
    imshow("original", img1);
    imshow("result", result);

    calcHist(&result, 1, channels, Mat(), hist3, 1, bins, ranges);
    drawHist(hist3, NORM_L1, "hist3");
    waitKey(0);
    return 0;
}
```

效果图

![match](https://s1.ax1x.com/2020/07/30/au1du8.png)



## 直方图反向投影

反向投影就是先计算某一特征的直方图模型，然后使用该模型去寻找图像中是否存在该特征的方法

- images：待统计直方图的图像数组，数据中所有图像应具有相同的尺寸和数据类型，并且数据类型只能是CV_8U, CV_16U和CV_32F中的一种，但是通道数可以不同
- nimages：输入图像的数量
- channels：需要统计的通道索引数组
- hist：输入直方图
- backProject：目标为反向投影图像
- ranges：每个图像通道中灰度值的取值范围
- scale：输出反向投影矩阵的比例因子
- uniform：直方图是否均匀的标志，默认是均匀

```c++
void cv::calcBackProject(const Mat* images,
                         int nimages,
                         const int* channels,
                         InputArray hist,
                         OutputArray backProject,
                         const float** ranges,
                         double scale = 1,
                         bool uniform = true
                        )
```

该函数使用时主要分以下四步

1. 加载模板图像和待反向投影图像
2. 转换图像颜色空间，常用的是灰度图像和HSV空间
3. 计算模板图像直方图，灰度图像为一维直方图，HSV图像为H-S通道的二维直方图
4. 调用该函数



i.e.

```c++
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;

void drawHist(Mat &hist, int type, string name) {
    int hist_w = 512;
    int hist_h = 400;
    int width = 2;
    Mat histImage = Mat::zeros(hist_h, hist_w, CV_8UC3);
    normalize(hist, hist, 255, 0, type, -1, Mat());
//    imshow(name,hist);
}

int main() {
    Mat img = imread("apple.jpg");
    Mat sub_img = imread("sub_apple.jpg");
    Mat img_HSV, sub_HSV, hist, hist2;

    imshow("img", img);
    imshow("sub_img", sub_img);

    //Turn into HSV model and get channel_S and channel_H
    cvtColor(img, img_HSV, COLOR_BGR2HSV);
    cvtColor(sub_img, sub_HSV, COLOR_BGR2HSV);
    int h_bins = 32;
    int s_bins = 32;
    int histSize[] = {h_bins, s_bins};
    //channel_H from 0-180
    float h_ranges[] = {0, 180};
    //channel_S from 0-256
    float s_ranges[] = {0, 256};
    //every channel's range
    const float *ranges[] = {h_ranges, s_ranges};
    //channel index
    int channels[] = {0, 1};
    //draw 2-dims H-S hist
    calcHist(&sub_HSV, 1, channels, Mat(), hist, 2, histSize, ranges, true, false);

    //normalize hist
    drawHist(hist, NORM_INF, "hist");

    Mat backproj;
    calcBackProject(&img_HSV, 1, channels, hist, backproj, ranges, 1.0);
    imshow("res", backproj);
    waitKey(0);
    return 0;
}
```

效果图

![back](https://s1.ax1x.com/2020/07/30/aucCJ1.png)



# 图像的模板匹配

比较像素灰度值来寻找相同内容称为图像的模板匹配

虽然很好理解，还是给个示意图叭

![模板匹配](https://tse4-mm.cn.bing.net/th/id/OIP.-MdyfuAuB4i_MNSoG7NmxgHaEM?pid=Api&rs=1)



**matchTemplate**函数原型

- image：待模板匹配的原始图像，数据类型为CV_8U和CV_32F
- templ：模板图像
- result：输出图像，数据类型是CV_32F，是单通道矩阵
- method：模板匹配方法标志
- mask：匹配模板掩码，默认不设置，目前仅支持在TM_SQDIFF和TM_CCORR_NORMED两种method中使用

```c++
void cv::matchTemplate(InputArray image,
                       InputArray templ,
                       OutputArray result,
                       int method,
                       InputArray mask = noArray()
                      )
```

如果原始图像的尺寸为$W\times H$，模板图像的尺寸为$w\times h$，那么输出图像的尺寸为$(W-w+1)\times (H-h+1)$，这很容易想象

method可选择标志表

| 标志参数         | 简记 | 方法名称             | 备注                      |
| ---------------- | ---- | -------------------- | ------------------------- |
| TM_SQDIFF        | 0    | 平方差匹配法         | 0完全匹配，越大越不匹配   |
| TM_SQDIFF_NORMED | 1    | 归一化平方差匹配法   | 0完全匹配，越大越不匹配   |
| TM_CCORR         | 2    | 相关匹配法           | 0最坏匹配结果，越大越匹配 |
| TM_CCORR_NORMED  | 3    | 归一化相关匹配法     | 1完全匹配，0完全不匹配    |
| TM_CCOEFF        | 4    | 系数匹配法           | 越大越匹配                |
| TM_CCOEFF_NORMED | 5    | 归一化相关系数匹配法 | 1完全匹配，-1完全不匹配   |



i.e.找脸

```c++
#include <iostream>
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;

int main() {
    Mat img = imread("lena.png");
    Mat temp = imread("lena_face.png");

    //match
    Mat result;
    matchTemplate(img, temp, result, TM_CCOEFF_NORMED);
    //locate
    double maxVal, minVal;
    Point maxLoc, minLoc;
    minMaxLoc(result, &minVal, &maxVal, &minLoc, &maxLoc);
    //draw
    rectangle(img, cv::Rect(maxLoc.x, maxLoc.y, temp.cols, temp.rows), Scalar(0, 0, 255), 2);
    //show
    imshow("original", img);
    imshow("temp", temp);
    imshow("result", result);

    waitKey(0);
    return 0;
}
```

效果图

![matchTemp](https://s1.ax1x.com/2020/07/30/auoIPS.png)









