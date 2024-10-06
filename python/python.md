# 1. Python Binding

在介绍Python Binding之前，需要先了解什么是[Language binding](../linux折腾日记/类unix.md#Language-binding)

Python Binding可以通过Python调用C和C++的函数。

相应的工具有：

- pybind11(TensorFlow采用这种方案)
- cpython
- numba.jit
- Triton
- ctypes(PyTorch采用这种方案，ctypes是python的标准库)
- swig(TensorFlow之前采用这种方案)

Python Binding的几个使用场景：

1. **You already have a large, tested, stable library written in C++** that you’d like to take advantage of in Python. This may be a communication library or a library to talk to a specific piece of hardware. What it does is unimportant.
2. **You want to speed up a particular section of your Python code** by converting a critical section to C. Not only does C have faster execution speed, but it also allows you to break free from the limitations of the **[GIL](https://realpython.com/python-gil/)**, provided you’re careful.
3. **You want to use Python test tools** to do large-scale testing of their systems.



## 1.1 Marshalling Data Types

 **marshalling(也就是序列化)**

```wiki
The process of transforming the memory representation of an object to a data format suitable for storage or transmission.
```

由于Python和C/C++是以不同方式存储data的，因此需要通过marshalling在二者之间做转换，传输数据。

```
以C为例，C中uint8_t就是8个bits，但是对于Python来说，everything is an object，因此需要更多的空间存储uint8_t。
```

所以Python bindings需要在Python的数据类型与C/C++的数据类型之间做转换，转换过程中还需要考虑溢出等问题。



## 1.2 Understanding Mutable and Immutable Values

Python objects可以是mutable和immutable(不可变)的，这个概念类似于C中的**pass-by-value** 和 **pass-by-reference**。

```
Mutable objects are those that allow you to change their value or data in place without affecting the object’s identity. In contrast, immutable objects don’t allow this kind of operation. 
```

Python中的immutable type有：

1. numbers: `int()`, `float()`, `complex()`
2. immutable sequences: `str()`, `tuple()`, `frozenset()`, `bytes()`

mutable type有：

1. mutable sequences: `list()`, `bytearray()`
2. set type: `set()`
3. mapping type: `dict()`
4. classes, class instances
5. etc.



在进一步深入mutable与immutable之前，需要先简单介绍一下Python中的**variables**与**objects**的区别。

在Python中，一切都是object。而变量只是一个标签，是指向object的一个引用，并没有类型与大小的概念。

- **Variables** hold references to objects.
- **Objects** live in concrete memory positions.



而immutable与mutable的概念只对object起作用，对于variables并不适用。不论variable指向的对象是mutable还是immutable，都可以通过这个variable进行修改，只不过对于mutable和immutable object的行为是不同的。

immutable object中的值是无法被更改的，举一个非常经典的例子

```
a = 1
print(a, id(a))  # 1 1737864256
a += 1
print(a, id(a))  # 2 1737864288
```

`a`是一个variable，其指向一个int类型的immutable object。因此当对`a`进行自增操作时，由于int类型是immutable的，因此会创建一个新的int类型对象，这个int类型在之前的基础上加一，然后将`a`改为指向这个新的int类型对象。从而完成这个自增操作。

而对于mutable object，修改一个mutable object既可以通过上面的方式，通过创建新的对象然后修改变量的引用实现，也可以通过直接对对象进行修改实现(所谓的in place方式)。x

由于Python中存在GC机制，因此当一个对象没有任何变量引用时，Python会自动回收这个对象。



但是还是有[方法](https://zhuanlan.zhihu.com/p/89897898)能够绕过immutable的限制的



## 1.3 Managing Memory

C/C++中需要用户自身管理好内存的分配与释放；而Python采用GC机制管理内存

因此在创建Python bindings时，需要考虑到python对象的生命周期。



## 1.4 Python bindings的一些工具

`invoke`类似于C/C++中的`make`，只不过是python的版本。



# 2. Pip的安装过程

首先是**包的发行版Distribution**

Distribution有多种形式：

- **Source Distribution** 源码形式发行的包，需要经过编译后才能使用。通常用`python setup.py sdist`产生源发行版
- **Built Distribution** 已编译发行版。wheel就是已编译发行版。



wheel包本质上是一个zip文件，是已编译发行版的一种格式，包里一般包含**.pyc**字节码。

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



# 3. Python中的pickle库

pickle用于序列化Python的对象，但是pickle对传入对象的要求是不能是内部类，也不能是lambda函数。因此这时需要用**dill**包代替。



# 4. Python中的各种文件

python中，除了.py文件，还有许多其它格式的文件。

`.py`这个不用多解释，python源代码文件。

`.ipynb`是Jupyter Notebook文件的扩展名，它代表"IPython Notebook"。IPython全称Interactive Python，是一种Python shell。相比于传统的Python shell，IPython有其独特的优势。当我们提到IPython时，就一定会提到[jupyter](#Jupyter)

`.pyi`是Python中的类型提示文件，用于提供代码的静态类型信息，一般与对应的`.py`文件同名，可以自动被关联。

`.pyc`是Python字节码文件，被编译过的`.py`文件，存储二进制信息。

`.pyd`是Python扩展模块的扩展名，用于表示使用C或C++编写的二进制Python扩展模块文件。`.pyd`也是编译后的二进制文件。

`.pyw`是Python窗口化脚本文件的扩展名。

`.pyx`是Cython源代码文件的扩展名。

# 5. Python Interpreter

有多种Python解释器：

- CPython
- PyPy
- JPython(Jython)
- Cython
- Numba
- IronPython
- IPython 实际上IPython还是基于CPython的，只不过IPython是基于CPython的一个交互式解释器(REPL)，在交互方面有增强。

# 6. Jupyter



# 7. Python中的数据类型

