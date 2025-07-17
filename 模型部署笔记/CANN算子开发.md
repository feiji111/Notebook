# CANN架构与算子开发





## CANN架构

![image-20250705102647217](./assets/image-20250705102647217.png)

## CANN算子开发方式



### 基于TBE的算子开发

TBE DSL/TBE TIK

算子的执行分为计算过程与调度过程



## Ascend平台算子融合记录

针对Qwen3 Dense模型。

Qwen3 Dense模型结构如下

<img src="./assets/image-20250716091600940.png" alt="image-20250716091600940" style="zoom:50%;" />

主要分为以下几类算子：

1. Attention类
2. Linear类
3. Rope旋转位置编码
4. RMSNorm算子



