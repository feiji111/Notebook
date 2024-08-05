# OpenCV



# 1. 一些常见问题



## 1.1 #include C++ header files in opencv

源自于stackoverflow上的提问[#include C++ header files in opencv](https://stackoverflow.com/questions/19177456/include-c-header-files-in-opencv)

如果想要引入OpenCV的头文件，有多种选择

```c++
#include <opencv2/opencv.hpp>
```

或者

```c++
#include <opencv2/imgproc/imgproc.hpp>
#include <opencv2/objdetect/objdetect.hpp>
#include <opencv2/highgui/highgui.hpp>
```

但是这两种方式还是有区别的，一种是**global includes**一种是**small includes**：

- 当采用global includes时，实际上的`opencv.hpp`还会include许许多多其其它的头文件，但是实际上并不是所有的头文件都会被用上，反而会导致编译时间的延长
- 而采用small includes则是更加精确按需include，更快的编译



## 1.2 OpenCV与Eigen



## 1.3 交叉编译OpenCV

更多关于交叉编辑的，看[这里](./cmake与makefile.md#1.4-交叉编译)

### 1.3.1 交叉编译OpenCV的过程总结

交叉编译OpenCV通过CMake + Unix Makefiles完成，可以通过CLI或者GUI两种方式来完成。

**CLI方式**：





### 1.3.2 

## 1.4 OpenCV Video I/O

```
The OpenCV Video I/O module is a set of classes and functions to read and write video or images sequence.
```

OpenCV Video I/O模块提供了[cv::VideoCapture](https://docs.opencv.org/4.9.0/d8/dfe/classcv_1_1VideoCapture.html)与[cv::VideoWriter](https://docs.opencv.org/4.9.0/dd/d9e/classcv_1_1VideoWriter.html)作为2-layer interface(二层接口，可以理解为前端)，而不同的video I/O APIs则充当后端。

<img src="assets/image-20240506162232574.png" alt="image-20240506162232574" style="zoom: 50%;" />

后端的video I/O APIs有许多，有可以分为两类：

- **interfaces to proprietary drivers or to external library**
  - **OpenNI2 for Kinect**
  - **Intel Perceptual Computing SDK**
  - **GStreamer**
  - **XIMEA Camera API**
  - **......**
- **interfaces to the video I/O library provided by the operating system**
  - **Direct Show (DSHOW)**
  - **Microsoft Media Foundation (MSMF)**
  - **Video 4 Linux (V4L)**
  - ......

如果同时有多个backend可供选择，OpenCV会选择第一个可用的后端，但也可以根据需要选择后端。

```c++
//declare a capture object
cv::VideoCapture cap(0, cv::CAP_MSMF);
//or specify the apiPreference with open
cap.open(0, cv::CAP_MSMF);
```

不同的backend后端采用不同的方式支持的property(即[cv::VideoCaptureProperties](https://docs.opencv.org/4.9.0/d4/d15/group__videoio__flags__base.html#gaeb8dd9c89c10a5c63c139bf7c4f5704d))，并且不同的后端支持的property也不尽相同，可能会**存在有的backend不支持某些属性的情况**。



对于不同backend的支持，则需要在编译OpenCV时通过CMake的选项指明需要启用哪一些backend。

videoio backend还可以分为built-in backends和plugins which will be loaded at runtime这两类。

```cmake
-DWITH_GSTREAMER=ON
```

To enable built-in videoio backends:

1. Enable corresponding CMake option, e.g. `-DWITH_GSTREAMER=ON`
2. Rebuild OpenCV

To enable dynamically-loaded videoio backend (currently supported: GStreamer and FFmpeg on Linux, MediaSDK on Linux and Windows):

1. Enable backend and add it to the list of plugins: `-DWITH_GSTREAMER=ON -DVIDEOIO_PLUGIN_LIST=gstreamer` CMake options
2. Rebuild OpenCV
3. Check that `libopencv_videoio_gstreamer.so` library exists in the `lib` directory



rtsp、rtmp、hls



# 2. Multimedia

## 2.1 一些基础知识

### 2.1.1 Video

#### 2.1.1.1 Video的一些基本属性与概念

Video的一些基本属性：

- **颜色深度**
- **分辨率**
- **DAR(Display Aspect Ratio)**
- **PAR(Pixel Aspect Ratio)**
- **SAR(Sample Aspect Ratio)**
- **FPS**
- **比特率(码率)** 播放一段视频每秒所需的数据量，比特率 = 宽 * 高 * 颜色深度 * 帧率。比特率又可以分为CBR(恒定比特率)与VBR(可变比特率)。这里又涉及到**逐行扫描**与**隔行扫描**技术。
- 饱和度



**隔行扫描**与**逐行扫描**





视频必须进行压缩，否则一个视频所需要的存储空间是非常大的。

```
比如对一个720p的一小时视频，其所需要的存储空间1280 x 720 x 24 x 30 x 3600 = 278GB
```

压缩视频，需要消除其中的冗余：**时间上的冗余与空间上的冗余**



#### 2.1.1.2 空间冗余

这一部分涉及到[色彩模型，色彩空间与色域](../图形学/图形学.md#色品图与色域)

**一个基本事实：人眼对于亮度信息比对于颜色信息要更为敏感**，因此可以通过压缩颜色信息，提高亮度信息实现压缩。

这一点图形学中人眼的构造中有讲到，看[这里](../图形学/GAMES101/GAMES101.md)



**YCbCr**(除了YCbCr之外，许多色彩模型也是同样将亮度与颜色分离)颜色模型将亮度与色度分离，**Y**表示亮度，**Cb**蓝色分量，**Cr**红色分量。





**色度抽样Chroma subsampling**技术，**色度抽样**是一种编码图像时，使**色度分辨率低于亮度**的技术。

<img src="assets/2560px-Common_chroma_subsampling_ratios_YCbCr_CORRECTED.svg-1715097658265-13.png" alt="undefined" style="zoom: 25%;" />

色度抽样通过三部分的比率表示**a​\:x\:y**：

- `a` 是水平采样参考 (通常是 4)，
- `x` 是第一行的色度样本数（相对于 a 的水平分辨率），
- `y` 是第二行的色度样本数。

**4:4:4**表明没有子采样

#### 2.1.1.3 时间冗余(帧间预测)

视频中的帧类型可以分为三类：

- **I帧** 关键帧`I‑frames are the least compressible but don't require other video frames to decode`
- **P帧** 预测帧`P‑frames can use data from previous frames to decompress and are more compressible than I‑frames`
- **B帧** 双向预测`B‑frames can use both previous and forward frames for data reference to get the highest amount of data compression`

不同类型的帧带来的代价与视频质量也不同：I帧>P帧>B帧。

![undefined](assets/2560px-I_P_and_B_frames.svg.png)

I-frame是一个完整的图像，比如JPG或者BMP图像。

P-frame只存储相对于前一帧的**改变量**。

B-frame只存储当前帧和前一帧和后一帧的差别。



在一个视频流中，I、B、P三种类型的帧如何组织起来，构成一个完整的视频，叫做[**Group of pictures(GOP)**](https://en.wikipedia.org/wiki/Group_of_pictures)



除了用残差，还可以通过运动补偿(**Motion compensation**)来进一步压缩。在讲运动补偿之前，首先需要明确视频中有两种顺序，一种是**播放顺序**，另一种是**编码解码顺序**。

<img src="assets/image-28.png" alt="I, P, and B-frames - Differences and Use Cases Made Easy - OTTVerse" style="zoom:50%;" />

因此两张相邻的帧，可以是编码顺序上相邻，也可以是播放顺序上相邻。









#### 2.1.1.4 视频编解码器发展与Container format

最早的数字视频编码标准是H.120

随后是H.261、H.263、H.264/AVC、H.265/HEVC

![Alt text](assets/codec_history_timeline.png)

**royalty有版税**，**royalty-free无版税**



编解码又分为**软解**和**硬解**：

- **软解** 使用运行在CPU之上的软件编解码
- **硬解** 使用特定的加速卡进行编解码，比如N卡的**NVDEC**与**NVENC**

软解和硬解各有优缺点：

- 软解的优势在于CPU由于是做通用计算，因此可以做到全解码，支持各种各样的格式，只需要安装对应格式的编解码器就可以。但是缺点在与占用CPU资源，效率较低。
- 硬解的优势在与专用芯片，效率高，速度快。但是缺点在与有时无法做到全解码，对于不同的格式，需要硬件上有相应支持才可以，兼容性比不过软解。



#### 2.1.1.5 编解码器的机制

图片分区

预测

转换



### 2.1.2 Camera

#### 2.1.2.2 Bayer Pattern与RAW格式

这一方面在[计算摄影学](../图形学/计算摄影学.md)中都包含。





#### 2.1.2.1 Webcam

当今的webcam基本上都是基于USB，在这种相机上，无法得到原始的RGB格式(RAW格式)，下面是来自Reddit这篇帖子[How should I get raw image data from a webcam in Linux without using a library?](https://www.reddit.com/r/C_Programming/comments/y7d8xe/how_should_i_get_raw_image_data_from_a_webcam_in/)的一个回答

```
You definitely should read this article in full:

https://en.wikipedia.org/wiki/USB_video_device_class

Any modern webcam you get is going to be USB based and that raw image data will be unavailable. The devices have hardware encoders inside them that will encode the data (either a single image, or streaming video) and send that compressed data over the USB. Even if you access that "raw data" you will need to decode it using something like FFmpeg.
```

webcam都内置一个硬件编码器，对于其捕获到的输入进行编码，然后将编码后的图片或者流通过USB传输到电脑。

因此，当采用OpenCV读取摄像头输入时，一般采用的`VideoCapture`类下的`read`方法，读取到的frame都是解码后的OpenCV BGR图像。



## 2.2 GStreamer

wiki官方对于GStreamer的介绍

```
GStreamer is a pipeline-based multimedia framework that links together a wide variety of media processing systems to complete complex workflows.
```



GStreamer整体的架构

<img src="assets/1280px-GStreamer_overview.svg.png" alt="undefined" style="zoom:50%;" />

GStreamer通过将许多processing elements连成一条pipeline，每一个processing element由一个plug-in提供。

<img src="assets/2560px-GStreamer_and_plug-in_types.svg.png" alt="undefined" style="zoom: 25%;" />



<img src="assets/2560px-GStreamer_Technical_Overview.svg.png" alt="undefined" style="zoom: 25%;" />



## 2.3 FFmpeg

FFmpeg作者，传奇程序员Fabrice Bellard，他的[主页](https://bellard.org/)。



### 2.3.1 色彩模式(Color Model)、图像像素格式(Pixel Format)与存储格式

在使用FFmpeg之前，了解一些常用的像素格式以及其如何存储时有必要的。

#### 2.3.1.1 YUV

之前在[空间冗余](#2.1.1.2-空间冗余)这一部分讲到了人眼对于亮度的信息更加敏感。因此可以采用**色度采样**的方式压缩图片。YUV格式正是利用了这一点。

YUV三个字母中，**Y**代表亮度(Luminance或Luma)，**U**和**V**则代表色度(Chrominance或Chroma)。

之前提到过YCbCr，其与YUV有着密切的关系

```
医学研究证明，人的肉眼对视频的Y分量更敏感，因此在通过对色度分量进行subsampling来减少色度分量后，肉眼将察觉不到图像质量的变化。如果只有Y信号分量，而没有U、V分量，那么这样表示的图像就是黑白灰度图像。彩色电视采用YUV空间正是为了用亮度信号Y解决彩色电视机与黑白电视机的兼容问题，使黑白电视机也能接收彩色电视信号。

YCbCr是在世界数字组织视频标准研制过程中，作为ITU-R BT.601建议的一部分，其实是YUV经过缩放和偏移的翻版。

1.YUV是模拟信号，其色彩模型源于RGB模型，即亮度与色度分离，适合图像算法的处理，常应用于在模拟广播电视中，其中 Y∈[0,1]，U,V∈[-0.5,0.5]。

2.YCbCr是数字信号，其色彩模型源于YUV模型，它是YUV压缩和偏移后的版本（所谓偏移就是从[-0.5,0.5]偏移到[0,1]，因此计算时候会加128），在数字视频领域应用广泛，是计算机中应用最多的格式，包括JPEG，MPEG，H.264/5，AVS等都采用YCbCr格式，我们通常广义上讲的YUV，严格来说，应该就是YCbCr。
```

因此后文YUV与YCbCr都代指一个概念。

而YCbCr可以进一步划分：

- TV range：Y、Cb、Cr的范围都是[16, 240]，广播电视采用的数字标准
- Full range：Y、Cb、Cr的范围都是[0, 255]，PC采用的标准

```
TV range要量化到16-235，主要是由于YUV最终在模拟域传输，因此为了防止数模转换时引起过冲现象，于是将数字域限定在16-235。至于为什么选择16和235，可自行了解Gibbs Phenomenon吉布斯现象。
```



YUV的**存储格式**分为两种：

- planar格式：先连续存储所有像素点的 Y 分量，然后存储 U 分量，最后是 V 分量。
- packed格式：每个像素点的 Y、U、V 分量是连续交替存储的。
- SemiPlanar格式：先连续存储所有像素点的Y，再连续交错U和V。



YUV的色度采样([Chroma Subsampling](https://cs.pynote.net/ag/image/202204032/#chroma-subsampling))也有多种格式：

- 4:4:4
- 4:2:2
- 4:1:1
- 4:2:0
- 4:4:0

<img src="assets/image-20240729160539467.png" alt="image-20240729160539467" style="zoom:80%;" />



存储格式和采样格式一组合，就有了多种YUV格式，以FFmpeg中的YUV格式为例，最后一个子母为**P**代表planar格式存储：

- planar格式
  - YUV 4:4:4
    - AV_PIX_FMT_YUV444P
    - ......
  - YUV 4:4:0
    - AV_PIX_FMT_YUV440P
    - AV_PIX_FMT_YUVJ440P
  - YUV 4:2:2
  - YUV 4:2:0
  - YUV 4:1:1

- packed格式
  - ......

FFmpeg中，YUV与YUVJ的格式上是完全相同的，只不过颜色空间不同，YUV采用的是[16, 255]的TV range；YUVJ采用的是[0, 255]的full range(JPEG转换公式，所以有一个**J**)。

```
libjpeg-turbo提供的转yuv的接口，得到的效果，与YUVJXXXP相同
```



按照三种存储格式：

- packed

  - UYVY422

    ![uyvy422.jpg](assets/uyvy422-1722242389259-3.jpg)

    YUYV422与YVYU422同理

  - YUV420

    ![yuv420.jpg](assets/yuv420-1722242431562-8.jpg)

- planar

  - YUV422P

    ![yuv422p.jpg](assets/yuv422p.jpg)

  - YUV420P(也叫I420)

    ![YUV420P码流的表现](assets/20200424135043.png)

    YVU420P，也叫YV12，12表示12 bit per pixel

- semiplanr

  - YUV422SP

    ![yuv422sp.jpg](assets/yuv422sp.jpg)

  - YUV420SP(也叫NV12)

    ![YUV420SP码流的表现](assets/20200424121342.png)

    YVU420SP，也叫NV21。



不同存储格式对于读取速度也有影响，planar格式需要3个for循环，而semiplanar格式只需要2个for循环，在后一个for循环中，可以同时读取Cb和Cr的数据。

#### 2.3.1.2 RGB

最为直观的color model就是RGB。

```
现在的计算机显示器，颜色的显示都是RGB模式，而视频编解码使用的大多都是YCbCr信号（YCbCr提供了最初的数据压缩，主流的420采样模式，直接就砍掉了一半的视频数据）。
```



### 2.3.2 FFmpeg工作机制

#### 2.3.2.1 转码

<img src="assets/image-20240730142926871.png" alt="image-20240730142926871" style="zoom:50%;" />



#### 2.3.2.2 Filtering



#### 2.3.2.3 Stream copy



#### 2.3.2.4 Loopback decoders



Stream selection







### 2.3.3 FFmpeg命令

#### 2.3.3.1 基本命令

基本命令结构

```
ffmpeg [global_options] {[input_file_options] -i input_url} ... {[output_file_options] output_url} ...
```

对于FFmpeg，输入和输出可以有任意多个，其输入可以是多种类型(regular files, pipes, network streams, grabbing devices, etc.)。

每个输入和输出中，都可以包含任意多的流stream，这些流可以是video/audio/subtitle/attachment/data。不同输入中的不同流在命令中通过`input_index:stream_index`来标识，比如`2:3`就是第三个输入的第四个流。



FFmpeg命令中的input_file_options/output_file_options都是作用在紧随其后的url的，因此顺序非常重要，必须先写好input然后再写output。



#### 2.3.3.2 Stream Specifiers

由于一些选项(比如bitrate和codec等)是作用在输入输出中的每一个stream的，因此存在一个Stream Specifier。

Stream Specifier附加在选项之后，用冒号分隔，比如对于codec选项，`-codec:a:1 ac3`，`-codec`是选项，而`:a:1`就是相应的Stream Specifier。

Stream Specifiers的格式：

- **stream_index**

- **stream_type[:additional_stream_specifier]**

  'v'和'V'代表video，'a'代表audio，'s'代表subtitle，'d'代表data，'t'代表attachments。**stream_index**可以作为**additional_stream_specifier**进一步

- ......

剩下的参考官方文档。



#### 2.3.3.3 Main options

ffmpeg的选项有很多，这里挑一些比较重要的。

`-codec/-c` 选项，使用格式如下：

**-c[:stream_specifier] codec (*input/output,per-stream*)**

**-codec[:stream_specifier] codec (*input/output,per-stream*)**

当它出现在output file之前，用以指定output file中的流的编码器；当它出现在input file之前，用以指定input file中的流的解码器。其中**codec**是encoder/decoder的名字，也可以是一个特殊的值**copy**，用以表明不对流做任何处理。

`-pix_fmt`选项，

# 3. OpenCV Mat详解

OpenCV的Mat的这几个属性：

- **data** 是一个uchar类型的指针
- **dim** 维度数(**注意不是通道数**)，3x3矩阵dim为2, 3x3x3矩阵dim为3
- **size** 各个维度的大小，注意该维度与channel无关，channel实际上是存储的元素属性，而不是容器的属性



# 4. OpenCV DNN

