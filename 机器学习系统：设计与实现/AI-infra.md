# AI infra/MLsys

最近在知乎以及微信公众号上，看到了许多关于AI infra、MLsys以及大规模分布式训练的文章，里面涵盖的内容从存储、网络、体系结构、编译、系统以及机器学习等计算机几乎所有重要的方向，同时与HPC联系非常密切，个人认为非常值得学习。同时也看到了许多先前未曾听到过的非常牛的技术，并且工业界也在广泛使用。但是由于这一方面的知识过于零散，因此在这里做一个总结，对于AI infra/MLsys个方向的技术做一个概述，后续有时间再深入。

- AI框架
- 并行框架
- 通信库
- 算子库
- AI compiler
- 编程语言(CUDA等)
- 调度系统
- 存储系统
- 容错系统
- 内存管理系统

# 1. AI infra/MLsys

采用IBM官方对于AI infra的解释[What is AI infrastructure?](https://www.ibm.com/topics/ai-infrastructure)

```
AI (artificial intelligence) infrastructure, also known as an AI stack, is a term that refers to the hardware and software needed to create and deploy AI-powered applications and solutions. 
```



## 1.1 History

AI infra的发展与大数据是分不开的。

当我们谈到大数据，绕不开的就是Google分别于03，04以及06年发表的三篇论文：GFS，MapReduce与Bigtable。这三者是驱动Google的三架马车，同时，也是大数据时代的开端。

那么问题来了，在Google提出这三架马车之前，难道就没有类似的大数据处理系统？而又是为什么是GFS，MapReduce与Bigtable能够作为大数据的开端？

