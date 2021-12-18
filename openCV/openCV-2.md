---
title: OpenCV2-Load,Show and Store Data
categories:
- OpenCV
mathjax: true
---

基于《OpenCV4快速入门》 @冯振、郭延宁、吕跃勇



# 图像储存容器

数字图像在计算机中以矩阵形式保存

OpenCV提供了一个Mat类用于储存矩阵数据

## Mat类介绍

Mat类分为**矩阵头**和**指向储存数据的矩阵指针**

**矩阵头**包含矩阵的尺寸、储存方法、地址和引用次数等，矩阵头的大小是个常数，不会随着矩阵尺寸的大小改变

在创建Mat类时，可以先创建矩阵头后赋值数据 

```c++
cv::Mat a; //创建矩阵头
a =cv::imread("test.jpeg"); //赋值图像数据, 矩阵指针指向像素数据
cv::Mat b=a; //复制矩阵头
```

值得注意的是，a,b虽然有各自的矩阵头，但是矩阵指针指向同一个矩阵数据，也就是说a,b对矩阵的改变会互相影响

然而删除a后b并不会指向空数据，只有两个都删除后才会释放矩阵数据

原理是矩阵头中引用次数只有为0时彩绘释放矩阵数据

还可以声明指定类型的Mat类，i.e.

```c++
cv::Mat A = cv::Mat_<double >(3,3);
```

下表是OpenCV中的数据类型与取值范围

| 数值类型 |    具体类型     |             取值范围              |
| :------: | :-------------: | :-------------------------------: |
|  CV_8U   | 8 位无符号整数  |            （0…..255）            |
|  CV_8S   |  8 位符号整数   |          （-128…..127）           |
|  CV_16U  | 16 位无符号整数 |           （0……65535）            |
|  CV_16S  |  16 位符号整数  |        （-32768…..32767）         |
|  CV_32S  |  32 位符号整数  |    （-2147483648……2147483647）    |
|  CV_32F  |   32 位浮点数   | （-FLT_MAX ………FLT_MAX，INF，NAN)  |
|  CV_64F  |   64 位浮点数   | （-DBL_MAX ……….DBL_MAX，INF，NAN) |



仅有数据类型还不够，还需要定义图像的通道

| 通道数标识 | 对应通道数 |
| :--------: | :--------: |
|     C1     |   单通道   |
|     C2     |   双通道   |
|     C3     |   3通道    |
|     C4     |   4通道    |



这样数据类型与通道结合就可以对图像数据类型完整定义，i.e.

```c++
cv::Mat a(640,480,CV_8UC3); //640*480的8位无符号整数三通道
cv::Mat b(3,3,CV_8UC1); //3*3的8位无符号整数单通道
cv::Mat c(3,3,CV_8U); //与b一样，C1标识可以忽略
```

***Warning***：CV_8U只能用在Mat类内部的方法，如果用 **cv::Mat_<CV_8U>(3,3)**或者**Mat a(3,3,unchar)**就会报错

## Mat类构造与赋值

### Mat类的构造

#### 默认构造函数使用方式

```c++
cv::Mat();
```

#### 根据输入矩阵的尺寸和类型构造

```c++
//行数，列数，数据类型
cv::Mat( int rows,
         int cols,
         int type);
//i.e.
cv::Mat a(640,480,CV_8UC3);
```

或者使用Size( )结构构造

```c++
//size：二维数组变量尺寸，type不说了
cv::Mat( Size size(),
         int type);
//i.e. 640*480的单通道矩阵
cv::Mat(cv::Size(480, 640), CV_8UC1);
```

***Warning***：Size是先列数再行数，和一般不一样

#### 利用已有构造

```c++
//这种构造方式只是复制了矩阵头
cv::Mat(const Mat& m);
//i.e.
cv::Mat a(cv::Size(480, 640), CV_8UC1);
cv::Mat b(a);
```

***Hint***：如果希望进行不互相影响的复制体，使用 **b=a.clone( )**实现

如果要构建子矩阵如下

```c++
//主要用于在原图中截图 它的复制体与本体同样是会互相影响的
cv::Mat(const Mat& m,
        const Range& rowRange,
        const Range& colRange); //第三个参数缺省值为Range::all()
//i.e.
cv::Mat a(cv::Size(480, 640), CV_8UC1);
cv::Mat b(a, cv::Range(2, 5), cv::Range(2, 5));
```



### Mat类的赋值

#### 构造时赋值

```c++
cv::Mat( int rows,
         int cols,
         int type
         const Scalar& s); //s是给矩阵中每个像素赋值的参数变量
//i.e.
cv::Mat a(2, 2, CV_8UC3, cv::Scalar(0, 0, 255));
```

#### 枚举赋值

```c++
cv::Mat a = (cv::Mat_<int>(3, 3) << 1, 2, 3, 4, 5, 6, 7, 8, 9);
```

***Warning***：该方法要求输入数据个数与矩阵元素个数完全相同，否则报错

#### 循环赋值

```c++
cv::Mat a = cv::Mat_<int>(3, 3);
for (int i = 0; i < a.rows; ++i) {
    for (int j = 0; j < a.cols; ++j) {
        a.at<int>(i, j) = i + j;
    }
}
```

***Warning***： **at**赋值时类型必须与声明时一样

#### 类方法赋值

```c++
cv::Mat a = cv::Mat::eye(3, 3, CV_8UC3); //单位矩阵，如果不是方阵则在主对角线位置处放1
cv::Mat b = (cv::Mat_<int>(1, 3) << 1, 2, 3);
cv::Mat c = cv::Mat::diag(b); //构建对角矩阵，参数必须是Mat类型一维变量。用来存放对角元素的数值
cv::Mat d = cv::Mat::ones(3, 3, CV_8UC1); //构建元素全为1的矩阵
cv::Mat e = cv::Mat::zeros(4, 2, CV_8UC3); //构建元素全为0的矩阵
```

#### 利用数组进行赋值

当矩阵中的元素数目>数组中的元素数目时使用 -1.0737418e+08填充

小于时则把矩阵填满，数组中多余的不管了

关于赋值过程是首先将矩阵中第一个元素的所有通道依次赋值，之后再下一个

```c++
float a[8] = {5, 6, 7, 8, 1, 2, 3, 4};
cv::Mat b = cv::Mat(2, 2, CV_32FC2, a);
cv::Mat c = cv::Mat(2, 4, CV_32FC1, a);
```

![对比](https://s1.ax1x.com/2020/07/28/aAjks1.jpg)



## Mat类支持的运算

### 加减乘除

演示就不写了，是个人都会，有几点需要注意

- 加减运算时两个Mat中的数据类型必须相同
- 乘除运算中常数与Mat变量进行运算后保留Mat类变量的数据类型
- 乘法有三种，接下来一个个讲



1." * "乘法

```c++
    cv::Mat a= cv::Mat::ones(3,3,CV_32FC1);
    cv::Mat b= cv::Mat::ones(3,3,CV_32FC1);
    cv::Mat c=a*b;
```

返回的是个矩阵，就是数学中最常见的矩阵乘法



2.dot( )内积

```c++
    cv::Mat a= cv::Mat::ones(3,3,CV_32FC1);
    cv::Mat b= cv::Mat::ones(3,3,CV_32FC1);
	double k = a.dot(b);
```

返回的是一个double

例如存在向量$D=[d_1,d_2,d_3]$和$E=[e_1,e_2,e_3]^{T}$，经过dot( )方法运算结果为：$f=d_1 e_1+d_2 e_2 +d_3 e_3$



3.mul( )对应积

```c++
    cv::Mat a = cv::Mat::ones(3, 3, CV_32FC1);
    cv::Mat b = cv::Mat::ones(3, 3, CV_32FC1);
    cv::Mat c = a.mul(b);
```

返回的是个矩阵，不同于前两种的是参与mul( )的两矩阵在保证数据类型相同情况下，可以是任何一种类型，并且默认的输出数据类型与两个Mat一致

***Warning***： 在图像处理领域，常用的数据类型是CV_8U，其范围是0~255，因此在使用该方法时需要防止出现数据溢出



## Mat类元素的读取

Mat类矩阵常用的属性

|    属性     |             作用             |
| :---------: | :--------------------------: |
|    cols     |           矩阵列数           |
|    rows     |           矩阵行数           |
|    step     | 以字节为单位的矩阵的有效宽度 |
| elemSize( ) |       每个元素的字节数       |
|  total( )   |       矩阵中元素的个数       |
| channels( ) |         矩阵的通道数         |

### at方法

读取单通道矩阵元素

```c++
    cv::Mat a = (cv::Mat_<unsigned char>(3, 3) << 1, 2, 3, 4, 5, 6, 7, 8, 9);
    int value = (int) a.at<unsigned char>(0, 0);
```

读取多通道矩阵元素

多通道矩阵每一个元素坐标处都是多个数据，故而在OpenCV中针对三通道矩阵定义了cv::Vec3b,cv::Vec3s,cv::Vec3w,cv::Vec3d,cv::Vec3f,cv::Vec3i共六种类型用于表示同一个元素的三个通道数据，名称中的数字表示通道数，最后的字母是数据类型的缩写，二、四通道也遵循这一命名方式

```c++
    cv::Mat b(3, 4, CV_8UC3, cv::Scalar(0, 0, 1));
    cv::Vec3b vc3 = b.at<cv::Vec3b>(0, 0);
    int first = (int) vc3.val[0];
    int second = (int) vc3.val[1];
    int third = (int) vc3.val[2];
```

### 通过指针ptr读取Mat类矩阵中的元素

```c++
    cv::Mat b(3, 4, CV_8UC3, cv::Scalar(0, 0, 1));
    for (int i = 0; i < b.rows; ++i) {
        unsigned char *ptr = b.ptr<unsigned char>(i);
        for (int j = 0; j < b.cols * b.channels(); ++j) {
            std::cout << (int) ptr[j] << std::endl;
        }
    }
```

### 通过迭代器访问Mat类矩阵中的元素

```c++
    cv::Mat a = (cv::Mat_<unsigned char>(3, 3) << 1, 2, 3, 4, 5, 6, 7, 8, 9);
    cv::MatIterator_<unsigned char> it = a.begin<unsigned char>();
    cv::MatIterator_<unsigned char> it_end = a.end<unsigned char>();
    for (int i = 0; it != it_end; ++it) {
        std::cout << (int) (*it) << " ";
        if ((++i % a.cols) == 0) {
            std::cout << std::endl;
        }
    }
```

### 通过矩阵元素的地址定位方式访问元素

row是元素所在行号

col是元素所在列号

channel是元素所在通道号

```c++
int value = (int) (*(b.data + b.step[0] * row + b.step[1] * col + channel));
```

## 

# 图像读取与显示

## 图像读取函数 imread

函数原型

- filename：图像文件名
- flags：读取图像形式的标志，默认参数是按照彩色图像格式

```c++
cv::Mat cv::imread(const String& filename,
                   int flags=IMREAD_COLOR
                  )
```

无法读取图像则会返回一个空矩阵

flags参数表：

|          标志参数          | 简记 |                             作用                             |
| :------------------------: | :--: | :----------------------------------------------------------: |
|      IMREAD_UNCHANGED      |  -1  |          按照图像原样读取，保留Alpha通道（第4通道）          |
|      IMREAD_GRAYSCALE      |  0   |                将图像转成单通道灰度图像后读取                |
|        IMREAD_COLOR        |  1   |                 将图像转换成3通道BGR彩色图像                 |
|      IMREAD_ANYDEPTH       |  2   |    保留原图像的16位、32位深度，不声明该参数则转成8位读取     |
|      IMREAD_ANYCOLOR       |  4   |                   以任何可能的颜色读取图像                   |
|      IMREAD_LOAD_GDAL      |  8   |                   使用gdal驱动程序加载图像                   |
| IMREAD_REDUCED_GRAYSCALE_2 |  16  | 将图像转成单通道灰度图像，尺寸缩小1/2，可以更改最后一位数字实现缩小1/4（最后一位改为4）和1/8（最后一位改为8） |
|   IMREAD_REDUCED_COLOR_2   |  17  | 将图像转成3通道彩色图像，尺寸缩小1/2，可以更改最后一位数字实现缩小1/4（最后一位改为4）和1/8（最后一位改为8） |
| IMREAD_IGNORE_ORIENTATION  | 128  |                    不以EXIF的方向旋转图像                    |



## 图像窗口函数 namedWindow

函数原型

- winname：窗口名称，必须是唯一的
- flags：窗口属性设置

```c++
void cv::namedWindow(const String& winname,
                     int flags = WINDOW_AUTOSIZE
                    )
```

窗口占用内存，可以通过 **cv::destroyWindow( )**和**cv::destroyAllWIndows( )**关闭

![namedWindow](https://s1.ax1x.com/2020/07/28/aE2LIP.png)

## 图像显示函数 imshow

函数原型

- winname：窗口名字
- mat：要显示的图像矩阵

```c++
void cv::imshow(const String& winname,
                InputArray mat
               )
```

***Hint***：此函数运行后会继续执行后面的程序，为了防止程序快速结束导致图片转瞬即逝，一般会在后面跟上 **cv::waitKey**函数



# 视频加载与摄像头调用

## 视频数据的读取

VideoCapture类构造函数

- filename：视频或图像序列名称 注意如果要读取图片序列需要将多个图像的名称统一为“前缀+数字”形式，通过“前缀+%02d”形式调用
- apiPreference：编码格式、是否调用OpenNI等设置

```c++
cv::VideoCapture(const String& filename,
                 int apiPreference = CAP_ANY
                )
```

读取成功与否可以用 **isOpened**函数判断

构造函数只是将视频文件加载到了 **VideoCapture**类变量中，还要导出到 **Mat**类变量里，该操作可以通过“<<”运算符将图像按顺序赋值，全部赋值完成后 **Mat**类变量会为空矩阵

该类同时提供了可以查看视频属性的 **get**方法，下表给出 **get**方法中的常用标志和含义

|       标志参数        | 简记 |                  作用                  |
| :-------------------: | :--: | :------------------------------------: |
|   CAP_PROP_POS_MSEC   |  0   | 视频文件的当前位置（播放）以毫秒为单位 |
| CAP_PROP_FRAME_WIDTH  |  3   |           视频流中图像的宽度           |
| CAP_PROP_FRAME_HEIGHT |  4   |           视频流中图像的高度           |
|     CAP_PROP_FPS      |  5   |     视频流中图像的帧率（每秒帧数）     |
|    CAP_PROP_FOURCC    |  6   |           编解码的4字符代码            |
| CAP_PROP_FRAME_COUNT  |  7   |           视频流中图像的帧数           |
|    CAP_PROP_FORMAT    |  8   |          返回的Mat对象的格式           |
|  CAP_PROP_BRIGHTNESS  |  10  |       图像的亮度(仅适用于照相机)       |
|   CAP_PROP_CONTRAST   |  11  |      图像的对比度(仅适用于照相机)      |
|  CAP_PROP_SATURATION  |  12  |      图像的饱和度(仅适用于照相机)      |
|     CAP_PROP_HUE      |  13  |       图像的色调(仅适用于照相机)       |
|     CAP_PROP_GAIN     |  14  |       图像的增益(仅适用于照相机)       |



i.e. 读取视频并且按照原帧率播放

```c++
#include <iostream>
#include "opencv2/opencv.hpp"

using namespace std;
using namespace cv;

int main() {
    system("color F0");
    VideoCapture video("cup.mp4"); //读取视频文件
    if (video.isOpened()) {
        //打印视频基本信息
        cout << "CV_CAP_PROP_FRAME_WIDTH: " << video.get(CAP_PROP_FRAME_WIDTH) << endl;
        cout << "CAP_PROP_FRAME_HEIGHT: " << video.get(CAP_PROP_FRAME_HEIGHT) << endl;
        cout << "CAP_PROP_FPS: " << video.get(CAP_PROP_FPS) << endl;
        cout << "CAP_PROP_FRAME_COUNT" << video.get(CAP_PROP_FRAME_COUNT) << endl;
    } else {
        cout << "Please check the filename" << endl;
        return -1;
    }
    while (true) {
        //将视频从VideoCapture导出到Mat类
        Mat frame;
        video >> frame;
        //frame为空矩阵则表示结束
        if (frame.empty()) {
            break;
        }
        imshow("video", frame);
        waitKey(1000 / video.get(CAP_PROP_FPS));
    }
    waitKey();
    return 0;
}
```



## 摄像头调用

与读取视频文件没什么不同，只有第一个参数变成了要打开的摄像头的ID

```c++
cv::VideoCapture(int index,
                 int apiPreference = CAP_ANY
                )
```



# 数据保存

## 图像的保存

函数原型

- filename：保存图像的文件名
- img：将要保存的Mat类矩阵变量
- params：保存图片属性格式设置标志

```c++
bool cv::imwrite(const String& filename,
                 InputArray img,
                 const std::verctor<int>& params = std::vector<int>()
                )
```

第三个参数一般不需要设置，可选标志及作用表

![imwrite](https://s1.ax1x.com/2020/07/28/aETfi9.png)

i.e.生成带有Alpha通道的矩阵，并保存成PNG格式图像的程序

```c++
#include <iostream>
#include "opencv2/opencv.hpp"

using namespace std;
using namespace cv;

void AlphaMat(Mat &mat) {
    CV_Assert(mat.channels() == 4);
    for (int i = 0; i < mat.rows; ++i) {
        for (int j = 0; j < mat.cols; ++j) {
            Vec4b &bgra = mat.at<Vec4b>(i, j);
            bgra[0] = UCHAR_MAX; //blue channel
            bgra[1] = saturate_cast<unsigned char>(
                    (float(mat.cols - j)) / ((float) mat.cols) * UCHAR_MAX); //green channel
            bgra[2] = saturate_cast<unsigned char>(
                    (float(mat.rows - i)) / ((float) mat.cols) * UCHAR_MAX); //red channel
            bgra[3] = saturate_cast<unsigned char>(0.5 * (bgra[1] + bgra[2])); //Alpha channel
        }
    }
}

int main() {
    //Create Mat with Alpha channel
    Mat mat(480, 640, CV_8UC4);
    AlphaMat(mat);
    vector<int> compression_params;
    compression_params.push_back(IMWRITE_PNG_COMPRESSION); //PNG格式图像压缩标志
    compression_params.push_back(9); //设置最高压缩质量
    bool result = imwrite("alpha.jpg", mat, compression_params);
    if (!result) {
        cout << "Failed" << endl;
        return -1;
    } else {
        cout << "Succeed" << endl;
        return 0;
    }
}
```

效果图：

![alpha](https://s1.ax1x.com/2020/07/28/aEbBRA.jpg)

## 视频的保存

**VideoWriter**类构造函数

- filename：视频文件名
- fourcc：压缩帧的4字符编解码器代码，如果赋值“-1”则会自动搜索合适的
- fps： 帧率
- frameSize：视频帧尺寸
- isColor：是否为彩色视频

```c++
cv::VideoWriter(const String& filename,
                int fourcc,
                double fps,
                Size frameSize,
                bool isColor = true)
```

也可以通过 **open( )**函数指定这些参数



i.e.打开摄像头并保存录下的视频

```c++
#include <iostream>
#include "opencv2/opencv.hpp"

using namespace std;
using namespace cv;


int main() {
    Mat img;
    VideoCapture video(0);  //use certain camera

    //read video
    //VideoCapture video;
    //video.open("cup.mp4");

    if (!video.isOpened()) {
        cout << "Camera open failed";
        return -1;
    }

    video >> img;  //get image
    if (img.empty()) {
        cout << "Get image failed" << endl;
        return -1;
    }
    bool isColor = (img.type() == CV_8UC3);  //judge whether video is colorful

    VideoWriter writer;
    int codec = VideoWriter::fourcc('M', 'J', 'P', 'G');  //choose encoding format

    double fps = 25.0;  //set video fps
    string filename = "live.avi";  //set video filename
    writer.open(filename, codec, fps, img.size(), isColor);  //create video stream to store file

    if (!writer.isOpened()) {
        cout << "Open video failed" << endl;
        return -1;
    }

    while (1) {
        if (!video.read(img)) {
            cout << "lose camera connection or finish video reading" << endl;
            break;
        }
        writer.write(img);  //write img into video stream
        //writer << img;
        imshow("Live", img);  //show img
        char c = waitKey(50);
        if (c == 27)  //press ESC to quit
        {
            break;
        }
    }
    // automatically close video stream when exit
    //video.release();
    //writer.release();
    return 0;
}
```



## 保存和读取XML和YMAL文件

**FileStorage**类

- filename：文件名
- flags：操作类型标志
- encoding：编码格式

```c++
cv::FileStorage(const String& filename,
                int flags,
                const String& encoding = String()
               )
```

| 标志参数 | 简记 |                含义                |
| :------: | :--: | :--------------------------------: |
|   READ   |  0   |           读取文件中数据           |
|  WRITE   |  1   |     向文件中重新写入数据，覆盖     |
|  APPEND  |  2   | 向文件中继续写入数据，在原数据之后 |
|  MEMORY  |  4   |    将数据写入或读取到内部缓冲区    |

打开文件后类似C++中创建的数据流，使用“<<”操作符将数据写入文件，通过“>>”操作符从文件中读取数据

**write**函数原型

- name：写入文件的变量名称
- val：变量值

i.e.

```c++
#include <iostream>
#include "opencv2/opencv.hpp"
#include <string>

using namespace std;
using namespace cv;

int main(int argc, char **argv) {
    string fileName = "datas.yaml";
    //open file in write mode
    cv::FileStorage fwrite(fileName, cv::FileStorage::WRITE);

    //store Mat type data
    Mat mat = Mat::eye(3, 3, CV_8U);
    fwrite.write("mat", mat);
    //store float type data
    float x = 100;
    fwrite << "x" << x;
    //store string type data
    String str = "Learn OpenCV 4";
    fwrite << "str" << str;
    //store array type data
    fwrite << "number_array" << "[" << 4 << 5 << 6 << "]";
    //store multi-nodes type data
    fwrite << "multi_nodes" << "{" << "month" << 8 << "day" << 28 << "year"
           << 2019 << "time" << "[" << 0 << 1 << 2 << 3 << "]" << "}";

    //close file
    fwrite.release();

    //open file in read mode
    cv::FileStorage fread(fileName, cv::FileStorage::READ);
    if (!fread.isOpened()) {
        cout << "Open file failed" << endl;
        return -1;
    }

    //read data in file
    float xRead;

    //float
    fread["x"] >> xRead;
    cout << "x=" << xRead << endl;

    //string
    string strRead;
    fread["str"] >> strRead;
    cout << "str=" << strRead << endl;

    //array
    FileNode fileNode = fread["number_array"];
    cout << "number_array=[";
    for (FileNodeIterator i = fileNode.begin(); i != fileNode.end(); i++) {
        float a;
        *i >> a;
        cout << a << " ";
    }
    cout << "]" << endl;

    //Mat
    Mat matRead;
    fread["mat="] >> matRead;
    cout << "mat=" << mat << endl;

    //multi-nodes
    FileNode fileNode1 = fread["multi_nodes"];
    int month = (int) fileNode1["month"];
    int day = (int) fileNode1["day"];
    int year = (int) fileNode1["year"];
    cout << "multi_nodes:" << endl
         << "  month=" << month << "  day=" << day << "  year=" << year;
    cout << "  time=[";
    for (int i = 0; i < 4; i++) {
        int a = (int) fileNode1["time"][i];
        cout << a << " ";
    }
    cout << "]" << endl;

    //close file
    fread.release();
    return 0;
}
```



























































