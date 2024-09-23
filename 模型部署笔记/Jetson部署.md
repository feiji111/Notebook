# Jetson TX2部署模型

# 1、权重文件

## 1.1

权重文件有多种格式



### 1.1.1 Pytorch

对于pytorch，保存模型有三种存储方式



### 1.1.2 TF



## 1.2

网络结构与权重的获取。

网络参数的获取：

网络结构的获取：对于目标检测（YOLO），网络结构分为**Backbone**，**Neck**，**Head**。

1. **TF-TRT**
2. **ONNX2TensorRT**
3. **手动构造模型结构，手动转移权重信息（tensorrtx）**



172.30.192.175(为什么校园网wifi不显示子网掩码？？) 网关172.30.128.1

172.30.255.231/17

# 2、JetPack



# 3、OpenCV

首先是**OpenCV**，**libopencv**以及**opencv-python**之间的关系。

- **libopencv**



JetSon预装的OpenCV并没有GPU的组件，即不支持CUDA。需要从源码编译安装，因此涉及到删除旧版本的OpenCV并且编译新版本。

Nvidia官方解释是OpenCV的GPU部分可能存在Bug不能保证稳定性，另一种选择是Nvidia自身的**VisionWorks**



## 3.1 包管理器安装（apt等）

`sudo apt install libopencv-dev python3-opencv`

安装后会有以下几个包：

- **libopencv-dev**：这只是一个metapackage，提供了OpenCV依赖的一些包，这些包中包含了头文件和库文件

- **opencv-python**：opencv的python**绑定库（binding）**在python上使用opencv有两种方式：

  1. `apt installing python3-opencv`
  2. `pip installing opencv-python`

  `opencv-python`是由opencv官方维护，保持最新；而`python3-opencv`不是官方维护。

- **libopencv**：包含相关的头文件与库文件

## 3.2 源码编译OpenCV

[jetson-container](https://github.com/dusty-nv/jetson-containers)

## 3.3 OpenCV-Python binding

关于**Python Bindings**见[Python](../python/python的一些问题.md)



## 3.3 Nvidia-VPI（Vision Programming Interface）

