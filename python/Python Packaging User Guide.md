# Python Packaging User Guide



# 1、 Packaging Python libraries and tools

tools Python’s ecosystem provides for distributing Python code to developers:

1. PyPi
2. setup.py
3. wheel
4. ......

都是Python生态中的一些工具。

## 1.1 Python Modules

一个单独的python文件，只依赖于standard library，可以被redistributed和reused。

这种方式适合于分发简单的脚本，但要求双方的python版本是兼容的。

## 1.2 Python source distributions

多个纯python文件组成的module更难被分发，因为**绝大多数传输协议只支持一次传一个文件**。因此这种情况下，可以采用Python原生packaging tool创建一个**sdist（source Distribution Package）**



Python的sdists是一种compressed archives（.tar.gz文件）。

### 1.2.1 Source distribution format

## 1.3 Python binary distributions

```
If you rely on any non-Python code, or non-Python packages (such as libxml2 in the case of lxml, or BLAS libraries in the case of numpy), you will need to use the format detailed in the next section, which also has many advantages for pure-Python libraries.
```

Python能够与C, C++, Fortran, Rust等其它语言编写的library结合。

![A summary of Python's packaging capabilities for tools and libraries.](assets/py_pkg_tools_and_libs.png)

## 1.4 Packaging Python Projects

本节有关如何创建，build以及上传package。

### 1.4.1 Creating the package files

一个package的结构大概如下

```
packaging_tutorial/
├── LICENSE
├── pyproject.toml
├── README.md
├── src/
│   └── example_package_YOUR_USERNAME_HERE/
│       ├── __init__.py
│       └── example.py
└── tests/
```

**\_\_init\_\_.py**使得相应的目录能够作为一个package被import，即使这个文件里面的内容是空的。

**tests/**用于存放test files。

### 1.4.2 Choosing a build backend





有许多backend可供选择

# 2、Packaging Python applications