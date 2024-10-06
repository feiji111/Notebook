# DeepSpeed

# 1. Introduction

DeepSpeed可以做到：

1.  partitioning the optimizer state, gradients, parameters，节省内存
2. **offloading optimizer memory and computation to a CPU or NVMe**



ZeRO是DeepSpeed的核心，ZeRO分为几个级别，和编译器的优化级别一样：

1. ZeRO-1
2. ZeRO-2
3. ZeRO + offload
4. ZeRO-3
5. ZeRO-3 + offload

这几种优化级别从上到下，memory-efficient提高，但是速度变慢。



在深入这几个优化级别之前，首先需要了解GPU训练/推理时的显存分配细节。

我们以LLM为例。目前的LLM大小一般都设置为7B，13B，52B，130B等几个档次，这么设计一部分原因是需要匹配部署时的显存，为了能够充分利用某种规格的算力。



首先定义一些符号：
$$
\textbf{h} \ | \ number \ of \ attention \ heads \\
\textbf{b} \ | \ batch \ size \\
\textbf{s} \ | \ sequence \ length \\
\textbf{d}_{embed} | \ dimension \ of \ embedding \\
\textbf{d}_{model} \ | \ transformer \ input \ dimension \\
\textbf{d}_{ff} \ | \ FFN \ layer \ hiden \ dimension \\
\textbf{L} \ | \ Transformer \ layers \\
\textbf{d}_k \ | \ dimension \ of \ keys \ and \ queries \\
\textbf{d}_v \ | \ dimension \ values \\
\textbf{v} \ | \ vocabulary \ size 
$$


这里涉及到一个vocabulary size与sequence length的区别，二者是不同的。vocabulary size是采用分词器tokenizer处理corpus时产生的总的不同的token的个数，相当于LLM的字典。而sequence length相当于LLM一次能够处理的token的个数，相当于一个window size。



**训练时**

训练阶段显存开销大致分为4个部分：模型参数，优化器，梯度，中间结果:

1. 模型参数：模型参数占用显存与采用的数据类型有关，FP32，FP16，BF16等，分别对应x4，x2，x2的显存开销。
2. 优化器：优化器占用显存与优化器以及采用的数据类型有关。Adam优化器带一阶动量需要的参数量是1x模型参数量，而带二阶动量需要2x模型参数量(**一阶 + 二阶，所以是两倍**)。在此基础上，采用FP32，FP32，FP16，BF16等，分别对应x4，x2，x2的显存开销。[PyTorch中有许多优化器](PyTorch源码解析.md#3.7-Optimizer)，不同优化器带来的参数量是不同的。
3. 梯度：一个参数对应一个梯度值，1x模型参数量。同时也有数据类型的划分，FP32，FP32，FP16，BF16等，分别对应x4，x2，x2的显存开销。
4. 中间计算结果：反向传播时需要用到中间计算结果。这个与batch大小以及序列长度有关。这个也是可以计算的。

上面的分析只是一个理想情况，实际情况还需要考虑许多其它因素。



需要考虑训练框架：DeepSpeed，Megatron等不同框架的显存占用是不同的



**对于模型参数**：

1. 对于embedding层，embedding层实际上实在做一个查表的工作。对于输入$[b, s]$，需要在embedding table(维度为$[v,d_{model}]$)上做embedding lookup的操作，实际上就是token的ID到embedding的映射操作。得到LLM的输入$[b,s,d_{model}]$。这一部分的参数量$v \times d_{model} + b \times s \times d_{model}$
2. 对于QKV projection层，Q和K的projection的维度是$[d_{model},d_k]$，V的projection的维度是$[d_{model}, d_v]$。h个header，那么总的参数量就是$h \times (d_{model} \times d_k + d_{model} \times d_v)$，L层就是$L \times h \times (2 \times d_{model} \times d_k + d_{model} \times d_v)$
3. 对于Self-Attention层，并没有引入参数
4. 对于Attention Project层，维度是$[hd_{v},d_{model}]$，L层就是$L \times h \times d_v \times d_{model}$
5. 对于FFN层，先升维到$4d_{model}$然后再降维到$d_{model}$，L层就是$2 \times L \times d_{model} \times d_{ff}$
6. 最后还有de_embedding层，完成从embedding到vocabulary的映射，维度为$[d_{model},v]$，这个参数可以与第一层embedding层共享，从而减少模型参数量，降低过拟合的风险。



因此通过上面的分析，模型参数量为
$$
v \times d_{model} + L \times h \times (2 \times d_{model} \times d_k + d_{model} \times d_v) + L \times h \times d_v \times d_{model} + 2 \times L \times d_{model} \times d_{ff}
$$
一般来说，$d_k = d_v = d_{model} / h$，并且$d_{ff} = 4 \times d_{v}$

所以上面的式子简化
$$
v \times d_{model} + 12 \times L \times d_{model}^{2}
$$



**对于优化器**



**对于梯度**



**对于中间结果(activation激活)**

每一层的LayerNorm的mean和variance参数量是$[b,s]$，同时每一层在训练过程中都有一个dropout mask，这个mask每个元素只占一个字节，参数量为$[b,s,d_{model}]$。

Self-Attention部分：

1. 每一层的输入参数量是$[b,s,d_{model}]$
2. 对于Q和K，合起来参数量为$[2,b,s,d_{model}]$
3. $QK^T$得到的attention mask参数量为$[b,h,s,s]$
4. softmax dropout的mask参数量$[b,h,s^2]$
5. Attention over V参数量$[b,s,d_{model}]$，同时还需要存储dropout的输出，参数量$[b,h,s^2]$

FFN部分，对于两层MLP的情况：

1. 第一层MLP，输入参数量$[b,s,d_{model}]$
2. 第二层MLP，输入参数量$[b,s,4d_{model}]$
3. GeLU激活需要额外保存$[b,s,4d_{model}]$的参数量，以在反向传播中使用
4. dropout mask还需要$[b,s,d_{model}]$

LayerNorm部分：

1. 每一层LayerNorm也会存储其输入，参数量$[b,s,d_{model}]$，由于每一层有两个LayerNorm，所以$[2,b,s,d_{model}]$

因此，对于每一层transformer，需要的activation参数量为,
$$
34bsd_{model} + 5bhs^2
$$
而L层就需要
$$
L(34bsd_{model} + 5bhs^2)
$$


按照上面的计算，对于



但是这样中间结果会占用非常多的显存。



上面的只是最vanilla的例子，现实当中会采用许多trick来减小显存占用：

1. **激活重计算(Recomputation)**，recomputation是一个非常常用的优化手段，通常通过时间换空间。FlashAttention中也用到了类似的思想。**PyTorch中的checkpoint机制也是如此**。
2. **梯度累计(Gradient Accumulation)**
3. 







**推理时**







**实例分析**

我们以LLAMA2为例，需要注意**LLAMA2的所有线性层都是不带bias偏置的**

| 参数规模 | 词向量维度$d_{model}$ | FFN hidden dimension $d_{ff}$ | 层数 | Attention | 头数 | 头维度 | 学习率LR | Content Length | Batch | Vocabulary Size | 数据量(Token) | 具体参数量  |
| -------- | --------------------- | ----------------------------- | ---- | --------- | ---- | ------ | -------- | -------------- | ----- | --------------- | ------------- | ----------- |
| 7B       | 4096                  | 11008                         | 32   | MHA       | 32   | 128    | 0.0003   | 4k(4096)       | 4M    | 32000           | 2T            | 6738415616  |
| 13B      | 5120                  | 13824                         | 40   | MHA       | 40   | 128    | 0.0003   | 4k(4096)       | 4M    | 32000           | 2T            | 13015864320 |
| 34B      | 6656                  | 17920                         | 60   | GQA       |      |        | 0.00015  | 4k(4096)       | 4M    | 32000           | 2T            | 32528943616 |
| 70B      | 8192                  |                               | 80   | GQA       |      |        | 0.00015  | 4k(4096)       | 4M    | 32000           | 2T            |             |

将LLAM2 7B带入上式中，结果大概是6.57B。

(**这里可能位置编码并没有引入参数，还需要查看资料确认，暂时先不算**)加上位置编码参数，位置编码参数是每一层共享的，并且batch内也是共享的，大小为$[s, d_{model}]$



但是距离LLAMA2 7B的参数量还是有一点距离，问题出在LLAMA的FFN实际上是投射了3次的，而之前我们采用的是两次投射(GPT-3)。

LLAMA的FFN层采用的是***SwiGLU***激活函数，
$$
FFN_{SwiGLU}(x,W,V,W_{2}) = (Swish_{1}(xW) \odot xV)W_{2}
$$
LLAMA2 7B的FFN层的hidden dimension是11008，这个隐藏层维度是通过$\frac{2}{3} 4d$然后取最近的256的倍数得到的，也就是前面的11008。

LLAMA采用的RMSNorm也有参数每一层要做两次RMSNorm，每一个RMSNorm的参数量为$d_{model}$

LLAMA2 7B的输出**de_embedding并不与embedding层共享参数**，因此还需要加上这一部分。

LLAMA2 7B最后的输出还需要做一次RMSNorm，$[1,d_{model}]$

加上上面之后，LLAMA2 7B的参数量具体是**6738415616**。



因此如果采用BF16存储，那么模型参数就需要占用13.5GB的显存，与huggingface上LLAMA2 7B的模型大小正好相同。







# 2. DeepSpeed的使用

使用DeepSpeed需要写一个`config`文件(格式可以是json，yaml，xml等类似的)。