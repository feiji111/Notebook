# PyTorch源码解析



# 1. Installation From Source

在分析源码之前，就必须能够成功从源码编译PyTorch。在编译PyTorch时，有非常多的选项可以选择，理清楚PyTorch的编译过程可以更好地了解PyTorch与其它组件之间的联系。







# 2. PyTorch的组成模块



# 3. PyTorch不同模块的机制



## 3.1 torch.autograd

**Autograd**是PyTorch中自动求导模块。自动求导是所有深度学习系统中最为重要的功能之一。

pytorch中的`Tensor`类有一个属性`requires_grad`，用以来声明需要对哪些Tensor自动求导。

