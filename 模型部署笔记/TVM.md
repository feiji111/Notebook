# TVM





# Halide



Halide中涉及到的一些概念：

- kernel
- stencil
- stream
- pipeline



一个image processing pipeline由两个部分组成：

- stencil computation
- stream programs



**stencil computation**，**实际上就是循环体内不包含条件判断**的就是模板计算，即一个array内的元素是以一个固定的pattern更新的。



一个image processing pipeline同时具备了wide与deep两个特点：

- wide体现在一幅图像往往需要处理非常多的像素
- deep体现在一个pipeline包含了非常多的阶段，大部分阶段是在做stencil computation，还有一些**complex reduction**阶段以及stages with global or data-dependent access patterns



在一个image processing pipeline中，stencil一般是以stencil pipeline形式出现的。

stencil pipeline：graphs of different stencil computations

