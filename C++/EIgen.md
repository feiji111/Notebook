# Eigen库的使用



# 1、Eigen库的结构

**Eigen库只有头文件，是一个纯用头文件搭建起来的库，没有.so或者.a二进制文件，所以在使用时，只需要引入Eigen头文件即可。**







# 2、编译Eigen库时遇到的一些问题

## 2.1 'add_const_t' is not a member of std, did you mean 'add_const'

这个错误通常发生在使用C++标准库时，某些名称没有被正确识别或不存在。问题 `'add_const_t' is not a member of std, did you mean 'add_const'` 指出标准模板库中没有发现 `add_const_t`。这可能是因为你的代码或库使用了不支持的C++标准版本。

`add_const_t` 是C++14中引入的类型特性模板别名(Type Trait Alias)，它是 `std::add_const<Type>::type` 的简化版。如果你在使用C++11，就会出现你遇到的这个问题，因为C++11没有定义 `add_const_t` 。

因此解决办法就是确保使用的C++标准版本至少是C++14或更高。

关于如何利用CMake与Makefile设置C++标准版本，参考[这里](./cmake与makefile.md#1.6-在CMake与Makefile中指定C和C++标准的版本)

