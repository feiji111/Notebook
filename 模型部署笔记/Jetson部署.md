# Jetson TX2部署模型

# 1、权重文件

## 1.1

权重文件有多种格式

### 1.1.1 序列化(serialization)与反序列化(deserialization)

*"pickling"* 是将 Python 对象及其所拥有的层次结构转化为一个字节流(**字符串或者文件**，**序列化也可以序列化成字符串**)的过程，而 *"unpickling"* 是相反的操作，会将（来自一个 [binary file](https://docs.python.org/zh-cn/3/glossary.html#term-binary-file) 或者 [bytes-like object](https://docs.python.org/zh-cn/3/glossary.html#term-bytes-like-object) 的）字节流转化回一个对象层次结构。

序列化也叫做：**“serialization”, “marshalling,” [[1\]](https://docs.python.org/3/library/pickle.html#id7) or “flattening”**

这一部分还与**[json]()**以及**[yaml]()**有联系，本质上都是如何通存储结构化的对象(object)的信息。

不同的序列化的方式也相应的有不同的物理格式，目前的比较常用的格式有：

1. **JSON**
2. **YAML**
3. **CSV**
4. **XML**
5. **PICKLE**

而pytorch保存模型实际上就使用到了python的序列化/反序列化库**pickle**。



### 1.1.2 Pytorch

对于pytorch，保存模型有三种存储方式

**保存模型结构和参数**

```python
torch.save(model_object, './model.pth')
```

加载时

```
loaded_model = torch.load(save_dir)
```

**只保存参数**，网络结构由对应网络的class来定义

```python
torch.save(model_object.state_dict(), './params.pth')
```

加载时需要先获得模型结构

```python
loaded_model = models.resnet152()   #注意这里需要对模型结构有定义
loaded_model.load_state_dict(torch.load(save_dir))
```

**保存训练检查点(模型结构，模型权重state_dict，优化器状态以及训练进度等)**

```
loaded_model = models.resnet152()
checkpoint = torch.load('path_to_checkpoint')
loaded_model.load_state_dict(checkpoint[''])
```

torch的模型文件后缀有.pkl，.pt，.pth，.pth.tar多种格式，对于不同的存储方式采用何种后缀并没有统一的规定，一般来说

| 存储方式                 | 拓展名   |
| ------------------------ | -------- |
| 只存储模型权重           | .pth     |
| 模型权重与模型结构       | .pt      |
| 保存训练检查点checkpoint | .pth.tar |
| 以上三种                 | .pkl     |



**Pytorch单卡和多卡模型存储也有不同**

**多卡并行的模型每层的名称前多了一个“module”**，因此在单卡加载多卡模型时需要注意。

### 1.1.3 TF



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

