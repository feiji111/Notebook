# DeepSeek-V3/R1技术报告

<img src="https://scontent-hkg1-1.cdninstagram.com/v/t51.29350-15/471495843_893526176192557_6570394372080387060_n.jpg?stp=dst-jpg_e35_tt6&efg=eyJ2ZW5jb2RlX3RhZyI6InRocmVhZHMuRkVFRC5pbWFnZV91cmxnZW4uMTQ0MHgyMTI5LnNkci5mMjkzNTAuZGVmYXVsdF9pbWFnZSJ9&_nc_ht=scontent-hkg1-1.cdninstagram.com&_nc_cat=101&_nc_oc=Q6cZ2QFdDtuIvM6qwNQ_l-ztYjnvu3RWQcOVepEq7j8MftqOsZixdq_9HSRYhUq3M8rGqPo&_nc_ohc=-u8wplI6E5sQ7kNvgE4SrCP&_nc_gid=BTqyYiZLfRAHrpI6g0n0sQ&edm=APs17CUBAAAA&ccb=7-5&ig_cache_key=MzUzMTczMzg0Mzk5ODk1Mjc5Ng%3D%3D.3-ccb7-5&oh=00_AYG9GS_DehjYKO-zVbx1eaHynNnwoU3ls9zFtq1Dw8rB_w&oe=67EDF62B&_nc_sid=10d13b" alt="No photo description available." style="zoom:50%;" />

## Abstract

| 一些重要参数          | 解释                                                         | Value |
| --------------------- | ------------------------------------------------------------ | ----- |
| hidden_act            | 激活函数，V3采用的是SiLU                                     |       |
| hidden_size           |                                                              | 7168  |
| intermediate_size     | 由于V3的前3层是dense FFN结构，所以这里的intermediate_size是dense FFN的中间维度，大小是9xmoe_intermediate_size | 18432 |
| kv_lora_rank          | 原文中$c_t^{KV}$的维度                                       | 512   |
| q_lora_rank           | 原文中$c_t^{Q}$的维度                                        | 1536  |
| qk_nope_head_dim      | 每个head的query和key的非位置编码维度                         | 128   |
| qk_rope_head_dim      | 每个head的query和key的位置编码维度                           | 64    |
| v_head_dim            |                                                              | 128   |
| num_attention_heads   |                                                              | 128   |
| num_key_value_heads   |                                                              | 128   |
| moe_intermediate_size | 每个专家的FFN的中间维度                                      | 2048  |
| n_routed_experts      | 可被路由的专家个数                                           | 256   |
| n_shared_experts      | 共享专家个数                                                 | 1     |

DeepSeek-V3是一个**671B**参数量的**MoE**语言模型(单模态)，推理时每个token会激活**37B**(稀疏MoE模型)的参数，参数可以分解为几个部分：

1. Embedding Layer
2. MLA Layer(x61)，激活采用SiLU激活
3. MoE Layer(x61，前三层没有MoE，采用的稠密FFN)
4. RMSNorm Layer

| V3参数分布                    | Params | 占比 |
| ----------------------------- | ------ | ---- |
| Embedding Layer(输入输出共享) | \      | \    |
| MLA                           | \      | \    |
| MoE                           | \      | \    |
| RMSNorm                       | \      | \    |

DeepSeek-V3的主要贡献点：

算法方面：

1. 为了实现推理的高效的加速，继续沿用了**MLA**
2. 为了实现训练的加速，沿用了DeepSeek-V2的**DeepSeekMoE**结构
3. 同时为了提升模型的性能，采用了auxiliary-loss-free strategy，这个策略是为了实现MoE模型专家之间的负载均衡，同时避免显式引入一个load-balance loss影响模型表现
4. 以及MTP(multi-token prediction training objective，与投机采样类似，一次forward能够生成多个token)

在工程方面：

1. DeepSeek-V3采用FP8混合精度训练
2. DualPipe更高效的流水线并行，更少的bubbles以及通过计算和通信的重叠隐藏延迟
3. 专家并行
4. 更加精细的memory优化策略，使得不需要tensor parrallelism(tensor parrallelism通信代价非常高)

在训练与数据方面：

1. 预训练数据采用14.8T的高质量语料库
2. 预训练中采用了两阶段的context length训练，第一个阶段的context length为32K，第二阶段的context length为128K
3. post-training阶段采用了DeepSeek-R1，通过蒸馏从DeepSeek-R1中蒸馏出long CoT模型的reasoning能力到标准的LLM中



## 模型结构

一个整体的结构

<img src="../../Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/a984feac72ae20d8c3b05bdfa4621663/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/assets/image-20250101141734207.png" alt="image-20250101141734207" style="zoom:50%;" />

### MoE结构

最早的采用MoE结构的开源模型是Mixtral 8x7B(实际是46B的大小)。

<img src="../../Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/a984feac72ae20d8c3b05bdfa4621663/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/assets/image-20250330002835246.png" alt="image-20250330002835246" style="zoom:50%;" />

MoE训练过程中会出现负载不均衡的现象，有的专家接收到的Token比较多，有的专家接收到的Token比较少，就会导致有的FFN层充分训练，但是有的FFN层欠训练，所以需要有负载均衡的机制来保证每一个专家都能够得到充分的训练。

整体MoE结构的核心：

1. Gating/Router的设置
2. 负载均衡的auxiliary loss的设置
3. 专家的设置



### DeepSeekMoE

相比于其它的MoE结构，DeepSeekMoE有以下特点：

1. Fine-grianed experts，DeepSeekMoE有256 + 1个的众多专家数量，每一次激活1 + 8个专家
2. Auxiliary-loss free load balancing机制
3. Complementary Sequence-Wise Auxiliary Loss
4. Node-Limited Routing
5. No Token-Dropping
6. sigmoid函数

1个共享专家(稳定被激活)和256个专家(每个Token激活其中8个专家)。

整体的结构和DeepSeek-V2一样。但是通过引入auxiliary-loss-free load balancing strategy，可以避免因负载均衡带来的性能下降。

> 对于MoE模型，不同专家之间不均衡的负载会带来性能下降以及推理成本的上升。所以一般会通过一个辅助的损失来实现负载均衡，但是引入辅助的损失会给原始的梯度带来噪声，对MoE模型最终的性能带来损失。

<img src="../../Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/a984feac72ae20d8c3b05bdfa4621663/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/assets/image-20250101144713331.png" alt="image-20250101144713331" style="zoom:50%;" />

**auxiliary-loss-free load balancing strategy**通过引入一个偏置项$b$(每一个专家都有一个偏置项)，实现loss-free。

$N_r$是routed experts的数量(256)，$N_s$是shared epxert的数量(1)，$K_r$是每一个Token激活的专家数量(8)。

<img src="../../Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/a984feac72ae20d8c3b05bdfa4621663/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/assets/image-20250101150045850.png" alt="image-20250101150045850" style="zoom:50%;" />

这个偏置项只用来做routing，而不会用来做加权。偏置项大小的调整通过引入一个参数$\gamma$。

在实际的训练/推理过程中，会检测每一个batch对每一个专家的负载，对于负载低的专家，会对其的$b + \gamma$；对于负载过高的专家，会对其的$b - \gamma$。

**如何检测专家的负载以及如何调整偏置项的更多细节在[AUXILIARY-LOSS-FREE LOAD BALANCING STRATEGY FOR MIXTURE-OF-EXPERTS](https://arxiv.org/pdf/2408.15664)中。**

<img src="../../Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/a984feac72ae20d8c3b05bdfa4621663/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/assets/image-20250401005612611.png" alt="image-20250401005612611" style="zoom:50%;" />

<img src="../../Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/a984feac72ae20d8c3b05bdfa4621663/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/assets/image-20250401004742175.png" alt="image-20250401004742175" style="zoom:50%;" />

每一个专家会记录一个历史的batch的负载信息(应该是上一个batch的负载信息$c_i$)。这里的$u$大小(也就是先前提到的$\gamma$)设置为0.001。



**Complementary Sequence-Wise Auxiliary Loss**。为了避免同一个sequence内部的imbalance，采用complementary sequence-wise balance loss。

$T$是序列中的Token数量。

<img src="../../Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/a984feac72ae20d8c3b05bdfa4621663/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/assets/image-20250101154325953.png" alt="image-20250101154325953" style="zoom:50%;" />

一步步分解上面的loss：

1. $\mathbb{1}\left( s_{i,t} \in \text{Topk}\left( \left\{ s_{j,t} \mid 1 \leq j \leq N_r \right\}, K_r \right) \right)$表示如果第$t$个token被第$i$个专家选中，就为1，否则为0。所以$f_i$就代表一个batch中每个专家被token选中的频率。
2. $s'_{i,t} = \frac{s_{i,t}}{\sum_{j=1}^{N_r} s_{j,t}}$相当于一个softmax归一化操作，每一个token $t$路由对第$i$个专家的"亲和度"。
3. $P_i$相当于序列对于专家$i$的亲和度。

可以把$P_i$看作每个专家的期望负载，而$f_i$是每一个专家的实际负载

所以整个$L_{Bal}$的含义就是：尽量让一个batch对于每一个专家的$f_i$和$P_i$相近，这样的loss是最小的。

**Node-Limited Routing**

Node-Limited Routing机制限制了每一个token至多会被发送到$M$个节点的专家进行处理，选择依据是affinity scores的和。在V3的场景下$M=4$，$K_r = 8$，意味着每次选中4个节点(一共8个节点)，每个节点选中2个专家。这一部份与接下来的专家并行有关。

首先会先选择$M$个节点，这M个节点中的专家有最高的affinity scores。然后再在这$M$个专家中通过Top-K选择$K_r$个专家。

**Experts Parallelism**

由于每个专家的推理是独立进行的，因此天然适合于将不同专家分布到不同的GPU/Node上并行推理。

V3采用的64路EP并行，而一共有256个routed专家，因此每张卡上是有4个专家的。而一个节点一般都是有8卡，因此每个节点是32个专家。



### MLA

MLA中涉及到以下几个参数：

1. $W^{DKV}$ 
2. $W^{UV}$
3. $W^{UK}$
4. $W^{KR}$
5. $W^{DQ}$
6. $W^{UQ}$
7. $W^{QR}$

之前讲过了pass

### MTP

<img src="../../Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/a984feac72ae20d8c3b05bdfa4621663/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/assets/image-20250101150903580.png" alt="image-20250101150903580" style="zoom:50%;" />

MTP背后的核心思想是希望一次推理能够生成多个Token。

一些符号声明：

1. $T$，序列的长度
2. $D$，MTP模块的个数
3. $i$，序列中第$i$个token
4. $k$，序列中第$k$个token


$h_i^k$是第$i$个token在第$k$预测深度上输出的表征，是要预测序列中第$i + k$位置的token，对应的是label中第$i + k + 1$位置的token。



与先前的一些利用投机采样一次推理生成多个Token的工作(EAGLE)不同，MTP是在训练阶段就采用的。

每个MTP模块的loss计算如下

<img src="../../Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/a984feac72ae20d8c3b05bdfa4621663/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/assets/image-20250402125828194.png" alt="image-20250402125828194" style="zoom:50%;" />

对所有的MTP loss平均得到MTP loss

<img src="../../Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/a984feac72ae20d8c3b05bdfa4621663/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/assets/image-20250402125956110.png" alt="image-20250402125956110" style="zoom:50%;" />

实际DeepSeek-V3采用的MTP的个数为1，即每一次前向产生2个token。

## Infrastructures

一些背景知识：

1. 几种并行策略：TP，DP，PP，SP，EP

   ![训练大模型并行和内存优化技术](https://pic1.zhimg.com/v2-8140e778fa7e41e95b113e36ec79de95_720w.jpg?source=7e7ef6e2&needBackground=1)

2. gradient checkpoint(recomputation)

3. computation-communication overlap(计算与通信重叠)

4. All-to-all通信操作



DeepSeek-V3的训练集群是在2048xH800上训练。每个节点8xH800通过NVLink以互联，节点之间通过NVSwitch互联。不同节点之间的通信通过InfiniBand(IB)网络。

并行策略如下

| TP   | PP   | EP   | SP   | DP          | CP(Context Parallism) |
| ---- | ---- | ---- | ---- | ----------- | --------------------- |
| 1    | 16   | 64   | 10   | 128(ZeRO-1) | 1                     |

<img src="https://pic4.zhimg.com/v2-2fe85bf77cd1ea64f93790561a47ca97_1440w.jpg" alt="img" style="zoom:50%;" />

训练采用的框架是HAI-LLM框架(自己搭建的)。并行策略采用16路流水线并行，64路专家并行，ZeRO-1的数据并行。训练并没有采用张量并行。



### DualPipe

流水线并行(PP)的演进：

1. GPipe朴素流水并行(F-then-B)

   <img src="../../Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/a984feac72ae20d8c3b05bdfa4621663/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/assets/image-20250401223703205.png" alt="image-20250401223703205" style="zoom:50%;" />

2. PipeDream(1F1B)，一个mini-batch前向完成之后立刻进入反向传播阶段，及时释放不必要的显存。同时模型参数的更新涉及到异步的操作。

   <img src="../../Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/a984feac72ae20d8c3b05bdfa4621663/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/assets/image-20250401223938591.png" alt="image-20250401223938591" style="zoom:50%;" />

3. Zero Bubble Pipeline(ZB1P)

   

优化PP的核心就是**减少bubble率**以及**减少显存占用**。

<img src="../../Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/a984feac72ae20d8c3b05bdfa4621663/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/assets/image-20250330100458066.png" alt="image-20250330100458066" style="zoom:50%;" />

<img src="../../Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/a984feac72ae20d8c3b05bdfa4621663/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/assets/image-20250401010707537.png" alt="image-20250401010707537" style="zoom:50%;" />

### Efficient Communication Kernels



### FP8 Training

混合精度训练的通用范式(Apex/Torch amp)

<img src="../../Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/a984feac72ae20d8c3b05bdfa4621663/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/assets/image-20250401010917236.png" alt="image-20250401010917236" style="zoom:50%;" />

## 预训练

预训练使用的语料一共有14.8T tokens。

为了拓展V3的上下文长度，整个预训练也分为几个阶段：

1. 初始的context window的大小是4k
2. 4k拓展到32k的context window大小训练1000steps
3. 32k拓展到128k，训练1000steps

<img src="../../Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/a984feac72ae20d8c3b05bdfa4621663/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/assets/image-20250402132358909.png" alt="image-20250402132358909" style="zoom:50%;" />





## Post Training与数据飞轮

### SFT

SFT阶段数据一共有1.5M。SFT数据分为两大类，Reasoning data和Non-reasoning data，分别对应日常生活中的两类任务。

整体的数据Pipeline都是通过内部更强的模型来合成数据，原文表述：

> For reasoning-related datasets, including those focused on mathematics, code competition problems, and logic puzzles, we generate the data by leveraging an internal DeepSeek-R1 model.

Reasoning data主要涉及到数学、代码等逻辑强需要推理的任务；Non-reasoning data主要涉及到一些日常任务。

OpenAI针对这两类的任务分别由两个模型处理：

1. 擅长处理日常任务的GPT-4o
2. 擅长推理的o1，严格的推理模型

而Anthropic的Claude-3.5-Sonnet则是将两个模型和为一个混合模型处理，兼顾日常任务(fast thinking)与推理(slow thinking)，V3类似。R1是一个严格的推理模型。

**Reasoning SFT data**主要通过另外的专家模型合成：

1. 专家模型聚焦于一个特定的领域，比如数学、代码等，每个专家模型是经过SFT以及RL获得的
2. 对于每一个样本，会生成两条SFT数据，一条是由原始专家模型生产的$<problem, original\ response>$；另一条是由**R1生成**的$<system\ prompt, problem, R1\ response>$。
3. 在对专家模型微调的过程中，利用原始数据和R1生成的数据进行RL，从而使得专家模型能够生成兼顾原始风格以及R1风格模式的数据
4. 专家模型经过RL之后，即可作为一个数据生成器，对生成的数据结果拒绝采样，作为新的SFT数据用于SFT V3-Base

> **对于推理类的query保留专家模型答案的准确性和简洁性 以及 R1的推理模式，而后在RL阶段通过高温采样来融合不同风格，即让奖励信号去决定什么时候该采用什么风格**

**Non-Reasoning data**则是利用DeepSeek-V2.5合成，并且人工标注确认正确性。



但一个明显的问题是，R1是基于V3-Base训练得到的，但是V3却反过来用R1产生的数据做SFT？

V3/R1中涉及到的几个模型：

1. DeepSeek-V3-Base，结果pretrain得到的模型
2. DeepSeek-V3，在DeepSeek-V3的基础上经过SFT和RL得到的模型
3. DeepSeek-R1-Zero，在DeepSeek-V3的基础上经过RL得到的模型
4. DeepSeek-R1，DeepSeek-V3的基础上经过SFT和RL得到的模型

V3对标的是GPT-4o，更擅长处理日常任务；而R1对标的是o1，擅长推理任务，二者都是从V3-Base结果后训练得到的，**然后利用各自后训练的版本生成对应能力的数据，比如推理数据和通用数据，互相提供给对方进行后训练加强**，相当于一个左脚踩右脚上天(bootstrapping)的过程。

**V2.5 → R1-lite-preview → V3 → R1-zero → R1**

<img src="https://pic2.zhimg.com/80/v2-1d45e39aef005336a8d5b3ac296e1ec5_1440w.webp" alt="img" style="zoom: 50%;" />



### RL

V3的RL过程结合了Rule-based Reward Model和Model-based Reward Model两种奖励模型的方式。

剩下的就是GRPO那一套流程(之前讲过)



### 合成数据与数据飞轮

两部分数据需要合成：

1. 合成Prompt
2. 合成pair对(RL需要的偏好数据)

后训练阶段需要用到非常多的高质量标注数据。

LLama3的飞轮，一共迭代6轮

<img src="../../Library/Containers/com.tencent.xinWeChat/Data/Library/Application Support/com.tencent.xinWeChat/2.0b4.0.9/a984feac72ae20d8c3b05bdfa4621663/Message/MessageTemp/9e20f478899dc29eb19741386f9343c8/File/assets/image-20250402134025487.png" alt="image-20250402134025487" style="zoom: 33%;" />



## R1-Zero/R1的训练流程

R1-Zero是在V3-Base的基础之上直接过RL得到的。R1-Zero采用的Reward-Model是一个纯Rule-Base RM同时也是一个ORM，两条规则：

1. Accuracy rewards，对就是对错就是错，只看结果不看过程
2. Format rewards，检测输出是否带有CoT，通过正则匹配<think>和</think>来确定输出中是否带有CoT



**Reject Sampling**

这里的Reject Sampling并非采样定理中的reject sampling，而是指对一个prompt生成N条回答，计算对应的reward，从中挑选出最好的作为SFT数据。



R1的训练分为四个阶段，两轮RL两轮SFT：

1. Cold Start(SFT)

   Cold Start的数据是通过DeepSeek-R1-Zero产生的，然后结果人工清洗得到的，量级大约是数千条，数据特点是长CoT数据，带有reflection与vertification。

2. Reasoning-oriented Reinforcement Learning(RL)

   与R1-Zero的RL过程相同，纯Rule-Based RM。主要是为了增强模型的在数学和coding等任务上的推理能力。

3. Rejection Sampling and Supervised Fine-Tuning(SFT)

   这一部分利用到了阶段**[2]**得到的RL模型，用于生产高质量的SFT数据，包含Reasoning data和Non-Reasoning data两部分。600k(Reasoning)+200k(Non-Reasoning)一共800k的SFT数据。

   600k的Reasoning data是利用**[2]**中的RL模型，结合拒绝采样得到的；200k的Non-Reasoning data是利用DeepSeek V3的部分SFT数据 + DeepSeek V3生成

4. Reinforcement Learning for all Scenarios(RL)

   Rule-based RW + Model-Based RW

<img src="https://substackcdn.com/image/fetch/w_1456,c_limit,f_auto,q_auto:good,fl_progressive:steep/https%3A%2F%2Fsubstack-post-media.s3.amazonaws.com%2Fpublic%2Fimages%2Fdf7f99f0-d154-49e5-b60a-4d148e0a61be_1548x1154.png" alt="img" style="zoom:50%;" />

<img src="https://pbs.twimg.com/media/GhxkCRMWoAAtXos?format=jpg&name=medium" alt="Image" style="zoom:50%;" />



## More

1. LLM as Judge与self-play，利用更强大的模型进行打分，模型T生成答案->给自己打分->做成DPO数据训练模型T+1形成一个self-play闭环
2. 软硬件协同设计







