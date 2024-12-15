# PyTorch技巧



# 1. 提高显卡利用率技巧

既然需要提高显卡利用率，前提就是能够监测到显卡的利用率。对于N卡，检测显卡利用率有一下方法：

1. nvidia-smi dmon
2. Nvidia NSight
3. NVIDIA Data Center GPU Manager
4. NVIDIA Management Library



显卡利用率低的原因本质是CPU与GPU不协调。CPU负载太高而GPU的负载太小，导致GPU需要等待CPU处理完数据才能够进行计算。

当进行模型训练的时候，CPU有以下负载，分为几大部份：

1. CPU 参与的数据运算和逻辑控制
2. CPU 为调用 CUDA API 所做的准备和清理工作
3. CPU与GPU之间的数据传输



下面分别来看各个CPU负载以及解决办法。



## 1.1 CPU 参与的数据运算和逻辑控制

1. 计算图的构建和维护
2. 数据加载到内存，解码以及数据增强需要CPU进行







## 1.2 CPU 为调用 CUDA API 所做的准备和清理工作

> 比如一个 nn.Linear，它最终调用了 cuBLAS 中的 GEMM，但 PyTorch 需要从 Python 前端执行到 C++ 后端，需要为输出 tensor 和必要的 workspace 分配内存，需要根据 device、合适的精度、正向传播/反向传播等一些列信息决定执行哪个算子，cuBLAS 需要根据输入参数的信息查询 heuristics 决定调用哪一个 CUDA kernel，PyTorch 还需要在调用 kernel 后维护计算图信息、执行必要的清理工作。调用一个 CUDA kernel 需要准备一系列复杂的运行时环境，这些都由 CPU 负责完成。当模型中存在大量的小 CUDA kernel 时，CPU 准备运行时环境的时间就会超过 CUDA kernel 在 GPU 上的执行时间，造成 GPU 需要等待 CPU 完成运行时环境的准备，从而降低了GPU 利用率。

​	•	**CUDA 上下文管理：**

​		•	每个线程需要与 CUDA 驱动程序通信，确保 CUDA 上下文的正确管理。

​		•	包括内存分配、核函数调用等，CPU 负责将这些指令发送给 GPU。

​	•	**内存分配和释放：**

​		•	GPU 的显存管理由 CUDA 驱动程序完成，但 CPU 需要管理主机（主机内存）和 GPU（显存）之间的内存分配和协调。

​	•	**CUDA 流管理：**

​		•	在 PyTorch 中，CUDA 流用于控制核函数的执行顺序和并行性。

​		•	CPU 需要控制核函数的分配和执行，确保计算按指定流完成。

​	•	**异步操作管理：**

​		•	CUDA 操作（如矩阵乘法或卷积）通常是异步的。CPU 需要检查 GPU 的任务完成状态，确保结果可以被安全使用。

​		•	如果某些操作需要同步（如 torch.cuda.synchronize()），CPU 还会等待 GPU 计算完成。



## 1.3 CPU与GPU之间的数据传输

​	•	**前向传播的数据传输：**

​	•	训练时，数据从主机内存（CPU 加载的）传输到 GPU 显存中，用于模型计算。

​	•	如果 DataLoader 没有启用 pin_memory，这一步会导致非页锁定内存与 GPU 通信速度变慢。

​	•	**后向传播的梯度传输：**

​	•	在多卡训练（如 DDP）中，各 GPU 计算的梯度需要通过 CPU 汇总或广播。

​	•	CPU 管理 reduce 操作，将各卡的梯度合并。

​	•	**权重更新的数据传输：**

​	•	训练时，更新后的权重需要从 CPU 或 GPU 广播到所有使用的设备。

​	•	如果模型较大，权重传输会成为性能瓶颈。



## 1.4 解决方案

目前有的一些技巧：

1. 开个Redis把数据全部存进去
2. Linux shm
3. Nvidia-DALI
4. 提高worker个数。实际上worker的数量是需要权衡的。内存的占用是与dataloader worker的数量成倍数增长的。

