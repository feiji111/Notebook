# Python相关问题



# 1、distro-info问题

问题描述

```

```



# 2、PIP和PIP3

pip和pip3是为了区别python和python3

如果电脑只安装了python3，pip和pip3是一样的。

但是如果电脑安装了python2，那么pip安装会把包安装在python2.7/site-packages下而pip3会安装在python3.7/site-packages下。



**为什么sudo -H pip install找不到但是sudo -H pip3 install找得到命令**

# 3、Python3.8-disutils



# 4、Python中的Wheel



# 5、[The following packages have unmet dependencies!](https://askubuntu.com/questions/563178/the-following-packages-have-unmet-dependencies)



# 6、Python虚拟环境

python虚拟环境管理有多种工具：

1. **pyenv** 用于管理不同的python版本
2. **virtualenv** 用于管理不同的python环境
3. **conda** 用于管理不同的python环境
4. **venv** 用于管理不同的python环境



# 7、apt下载的包问什么能用pip3运行



# 8、python与pip之间的依赖问题



# 9、Pip的安装过程

首先是**包的发行版Distribution**

Distribution有多种形式：

- **Source Distribution** 源码形式发行的包，需要经过编译后才能使用。通常用`python setup.py sdist`产生源发行版
- **Built Distribution** 已编译发行版。wheel就是已编译发行版。



wheel包本质上是一个zip文件，是已编译发行版的一种格式，包里一般不包含**.pyc**字节码。

wheel包的文件名组成`{dist}-{version}(-{build})?-{python}-{abi}-{platform}.whl`

以`tensorflow-2.3.1-cp36-cp36m-macosx_10_9_x86_64.whl`为例

其中：

- **dist**：包名
- **version**：包的版本号
- **cp36**：对python解释器和版本的要求，要求是CPython解释器，3.6版本。因此拿JPython就不能使用这个Wheel包
- **cp36m**：ABI
- **macosx_10_9_x86_64**：平台

**通用wheel包**：一般以**py2.py3-none-any**结尾，对python版本、ABI和平台都没有要求。

**manylinux平台标签**：

```
鉴于Linux系统有不同的发行版（Ubuntu，CentOS，Fedora等等），而你的包里有需要编译的C/C++代码，那有可能不同Linux发行版就不能运行你的包了，而为每个Linux发行版生成一个wheel又太麻烦，所以就诞生了manylinux系列标签：manylinux1（PEP513），manylinux2010（PEP571）和manylinux2014（PEP599）。manylinux标签的核心是一个CentOS的Docker镜像，打包了一些编译器套件、多版本Python和pip、动态库等来确保兼容性。这个在PEP513里面有提到。
```



使用Wheel包：

- python字节码.pyc由pip生成
- 不需要执行**setup.py**
- 体积小
- 不需要编译器

**PyPi**和**setup.py**



# 10、Python Binding

Python Binding可以通过Python调用C和C++的函数。



Python Binding的几个使用场景：

1. **You already have a large, tested, stable library written in C++** that you’d like to take advantage of in Python. This may be a communication library or a library to talk to a specific piece of hardware. What it does is unimportant.
2. **You want to speed up a particular section of your Python code** by converting a critical section to C. Not only does C have faster execution speed, but it also allows you to break free from the limitations of the **[GIL](https://realpython.com/python-gil/)**, provided you’re careful.
3. **You want to use Python test tools** to do large-scale testing of their systems.



## 10.1 Marshalling Data Types

 **marshalling**

```wiki
The process of transforming the memory representation of an object to a data format suitable for storage or transmission.
```

由于Python和C/C++是以不同方式存储data的，因此需要通过marshalling在二者之间做转换，传输数据。

```
以C为例，C中uint8_t就是8个bits，但是对于Python来说，everything is an object，因此需要更多的空间存储uint8_t。
```

## 10.2 Understanding Mutable and Immutable Values

Python objects可以是mutable和immutable的，这个概念类似于C中的**pass-by-value** 和 **pass-by-reference**。

## 10.3 Managing Memory

C/C++中需要用户自身管理好内存的分配与释放；而Python采用GC机制管理内存



# 11、OSError: /home/zhangkj/anaconda3/envs/nerf-torch/lib/python3.8/site-packages/nvidia/cublas/lib/libcublas.so.11: undefined symbol: cublasLtGetStatusString, version libcublasLt.so.11

**原因**：pytorch1.13会自动安装nvidia_cublas_cu11，nvidia_cuda_nvrtc_cu11，nvidia_cuda_runtime_cu11和nvidia_cudnn_cu11，但是如果系统预先装好了CUDA这几项是都有的。



**解决办法**：`pip uninstall nvidia_cublas_cu11`



# 12、Python中的pickle库

pickle用于序列化Python的对象，但是pickle对传入对象的要求是不能是内部类，也不能是lambda函数。因此这时需要用**dill**包代替。
