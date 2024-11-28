# PyTorch技巧



# 1. 提高显卡利用率技巧

既然需要提高显卡利用率，前提就是能够监测到显卡的利用率。对于N卡，检测显卡利用率有一下方法：

1. nvidia-smi dmon
2. Nvidia NSight
3. NVIDIA Data Center GPU Manager
4. NVIDIA Management Library



显卡利用率低的原因本质是CPU与GPU不协调。CPU负载太高而GPU的负载太小，导致GPU需要等待CPU处理完数据才能够进行计算。

当进行模型训练的时候，CPU有一下负载：

1. 