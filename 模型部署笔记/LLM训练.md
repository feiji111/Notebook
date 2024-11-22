# LLM训练



# 1. Scaling Law

与Scaling Law有关的几篇文章：

[Scaling Laws for Neural Language Models](https://arxiv.org/pdf/2001.08361)

[Training Compute-Optimal Large Language Models](https://arxiv.org/pdf/2203.15556)

[Scaling Language Models: Methods, Analysis & Insights from Training Gopher](https://arxiv.org/pdf/2112.11446)





模型参数量大小N，数据量大小Token数D，以及计算量大小C。



# 2. 预训练数据



## 2.1 预训练数据来源

英文有许多语料库，Wikipedia + ArXiv + C4 + Github + Common Crawler基本能满足要求。但是中文高质量语料库欠缺。



## 2.2 预训练数据准备



### 2.2.1 concat-and-chunk



## 2.3 预训练策略

在传统的深度学习模型训练中，epoch的设置可以类似于传统机器学习模型训练的迭代次数，一般认为越多的epoch会让模型拟合得越好。然而，在LLM时代，很多模型的epoch只有1次或者几次。



# 3. DPO/PPO







