# 1. Transformer

## 1.1 基本知识

不再赘述

## 1.2 Transformer时间复杂度与空间复杂度





# 2. MoEs

参考Hugging face对MoEs的介绍[Mixture of Experts Explained](https://huggingface.co/blog/moe)

MoEs是Transformer的一种变体。简单的来说，就是将Transformer的FFN层(成为dense FFN)替换为MoE layer。这个MoE layer由多个expert组成，每一个expert是一个较为轻量的FFN，但是也可以是一个更加复杂的网络(比如expert也可以是一个MoE，这种结构就是hierarchical MoEs)。

一般来说MoE layers包含两个主要的部分：

- **Sparse MoE layers**
- **A gate network or router**



MoEs的思想类似于集成学习(**Ensemble learning**)。此后MoEs又分为两个领域：**Experts as components**与**Conditional Computation**。



