# Machine Learning Compilation



# 1. What is Machine Learning Compilation(MLC)

MLC其实就是模型部署的过程。



MLC不仅优化推理端，还需要优化训练端



MLC需要考虑的目标：

1. 



MLC会做Tensor Function(算子Operator)之间的变换:

- 算子融合Fison来减少IO消耗
- 算子变换(算子内优化，并行)



**abstraction**与**implementation**是任何系统方向非常重要的思想。



Primitive Tensor Function(也就是我们常说的算子)，对于一个算子，它是一个抽象，有不同的实现算子的方式(C，Python等等)。



Primitive Tensor Function transformation(算子变换):

1. 映射到函数库
2. Fine grained program transformation

不同的算子变换对于算子的抽象也有不同的要求，而我们希望找到一种抽象，其算子变换能够获得一些性能等方面的提升。



当我们涉及到对Primitive Tensor Function本身做算子内优化时，就涉及到另一种abstraction：Tensor Program Abstraction。













<img src="assets/1713773709318.jpeg" alt="No alternative text description for this image" style="zoom:50%;" />