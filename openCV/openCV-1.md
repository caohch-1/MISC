---
title: OpenCV1-What's OpenCV
categories:
- OpenCV
---

基于《OpenCV4快速入门》 @冯振、郭延宁、吕跃勇





# 什么是OpenCV

不说了

# 安装

没啥好说的

# 了解OpenCV的模块架构

Ubuntu20.04下是位于**/usr/local/include/opencv4/opencv2**，与书上的目录不太一样，可能是我安装路径或者系统不同的原因

一个个看看这些模块

- calib3d——这个名字其实是calibration(校准)和3D两个词的结合，这个模块主要包含相机标定与立体视觉等功能
- core——核心功能模块，主要包含基础模块与基本操作
- dnn——深度学习模块，该模块是OpenCV4的特色。其主要包括构建神经网络、加载序列化网络模型等
- features2d——由features(特征)和2D两个词结合，功能主要是处理图像特征点
- flann——Fast Library for Approximate Nearest Neighbors快速近似最近邻库，该模块是高维的近似近邻快速搜索算法库
- gapi——加速常规的图像处理，该模块主要充当框架
- highgui——高层GUI，包含创建和操作显示图像的窗口等
- imgcodes——图像文件读取与保存模块，功能顾名思义
- imgproc——由image和process缩写合成，是重要的图像处理模块
- ml——机器学习模块
- objdetect——目标检测模块，主要用于图像目标检测
- photo——计算摄影模块，主要包含图像修复和去噪
- stitching——图像拼接模块
- video——视频分析模块
- videoio——视频输入/输出模块，主要用于读取、写入视频或图像序列

# 源码示例程序展示

OpenCV4.1源码中包含了许多示例程序

Ubuntu20.04下是位于**~/opencv-4.1.2/samples**，与书上的目录不太一样，可能是我安装路径或者系统不同的原因

## 配置示例程序运行环境

由于我没有按照书上使用win+VS，而是Ubuntu+CLion，在运行示例文件时方法不太一样

首先CLion新建Project

然后从**~/opencv-4.1.2/samples**中把需要运行的示例代码移到Project目录下

然后修改CLion自动创建的CMakeLists.txt

在最后加入如下两行 OpenCVSamples是项目名称，按需修改

```cmake
find_package(OpenCV REQUIRED)

target_link_libraries(OpenCVSamples ${OpenCV_LIBS})
```

然后在CLion中run即可

接着会在 **cmake-build-debug**中生成对应可执行文件

依据cpp源文件中help( )的指示进行使用

## 边缘检测edge

首先介绍一个关于Canny边缘检测的示例程序

```bash
(base) caohch1@caohch1-Lenovo-XiaoXin-Air-14ARR:~/JetBrainProjects/CLionProjects/OpenCVSamples/cmake-build-debug$ ./OpenCVSamples lena.png

```

效果图：

![edge检测](https://s1.ax1x.com/2020/07/28/aAdd0g.png)



## K聚类kmeans

通过可视化界面显示K聚类算法

该算法将散点判定为不同群体，并确定每个群体的中心点位置

```bash
(base) caohch1@caohch1-Lenovo-XiaoXin-Air-14ARR:~/JetBrainProjects/CLionProjects/OpenCVSamples/cmake-build-debug$ ./OpenCVSamples
```

效果图：



![kmeans](https://s1.ax1x.com/2020/07/28/aAwDbD.png)



![kmeans2](https://s1.ax1x.com/2020/07/28/aAwBDO.png)



## 二维码识别qrcode

此处不加任何参数，默认开启摄像头进行检测

```bash
(base) caohch1@caohch1-Lenovo-XiaoXin-Air-14ARR:~/JetBrainProjects/CLionProjects/OpenCVSamples/cmake-build-debug$ ./OpenCVSamples
```

该示例程序没有help( )函数，但是在文件开始给出了使用方法

效果图：

二维码四周有紫色框框标定

终端中也会输出二维码中信息

![qrcode](https://s1.ax1x.com/2020/07/28/aADFu6.png)



## 相机使用video_capture_starter

该程序可以启动摄像头获取某一帧图像 也可以处理每一帧图像后恢复成视频

不想演示了，能不能赶紧来点干货，别搁这儿整花活



## 视频物体跟踪camshiftdemo

对鼠标选择区域物体跟踪

此处不加任何参数，默认开启0号摄像头

```bash
(base) caohch1@caohch1-Lenovo-XiaoXin-Air-14ARR:~/JetBrainProjects/CLionProjects/OpenCVSamples/cmake-build-debug$ ./OpenCVSamples
```

效果图：

这里我选取了一张书签

![camshiftdemo1](https://s1.ax1x.com/2020/07/28/aAr5Ss.png)

![camshiftdemo2](https://s1.ax1x.com/2020/07/28/aArhWj.png)

![camshiftdemo3](https://s1.ax1x.com/2020/07/28/aArfYQ.png)









