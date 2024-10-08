# Numbers everyone (programmer) should know (2023)

```
L1 cache reference                           0.5 ns
Branch mispredict                            5   ns
L2 cache reference                           7   ns                      14x L1 cache
Mutex lock/unlock                           25   ns
Main memory reference                      100   ns                      20x L2 cache, 200x L1 cache
Compress 1K bytes with Snappy            3,000   ns        3 µs
Read 1 MB sequentially from memory      20,000   ns       20 µs          ~50GB/sec DDR5
Read 1 MB sequentially from NVMe       100,000   ns      100 µs          ~10GB/sec NVMe, 5x memory
Round trip within same datacenter      500,000   ns      500 µs
Read 1 MB sequentially from SSD      2,000,000   ns    2,000 µs    2 ms  ~0.5GB/sec SSD, 100x memory, 20x NVMe
Read 1 MB sequentially from HDD      6,000,000   ns    6,000 µs    6 ms  ~150MB/sec 300x memory, 60x NVMe, 3x SSD
Send 1 MB over 1 Gbps network       10,000,000   ns   10,000 µs   10 ms
Disk seek                           10,000,000   ns   10,000 µs   10 ms  20x datacenter roundtrip
Send packet CA->Netherlands->CA    150,000,000   ns  150,000 µs  150 ms
```



# Infrastructure



# 1. 硬件部分

## 1.1 PCIe Switch

在PCIe Swtich方面，博通是处于垄断地位。

## 1.2 NVLink

## 1.3 NVSwitch

## 1.4 HBM

## 1.5 NVIDIA HGX与DGX

参考：

1. [STH NVIDIA DGX versus NVIDIA HGX What is the Difference](https://www.servethehome.com/nvidia-dgx-versus-nvidia-hgx-what-is-the-difference/)

HGX与DGX是NVIDIA不同的产品线，是NVIDIA售卖其8x GPU system with NVLink的不同形式。

首先需要明确的是，HGX与DGX都是NVIDIA搭载NVLink互联的8x GPU平台，采用的是SXM接口。



早先(P100时代)NVIDIA的商业模式是，不同的OEM厂商会构建自己的GPU主板，然后从NVIDIA购买SXM规格的GPU，最后再由OEM厂商(或者个人或组织)组装这些GPU到主板上。

但是不论是OEM厂商还是个人安装这些GPU到主板上都会有不小的难度。



而随着V100时代的到来，NVIDIA标准化了整个8x SXM GPU平台，更多的NVLinks，并且添加了NVSwitch，整个baseboard主板的拓扑结构被NVIDIA标准化了，这就是HGX。

除了baseboard的拓扑结构被标准化，GPU散热器也是由NVIDIA规定，并且由NVIDIA将GPUs和散热器安装在baseboard值上。



因此，OEM厂商可以直接从NVIDIA购买组装好的8x GPU baseboard(**HGX**)，然后自行在这个baseboard的基础上添加CPU，RAM，Storage等。但是这个**HGX baseboard**的拓扑是无法被改变的。



而到了A100和H100时代，

![Inspur NF5488A5 NVIDIA HGX A100 8 GPU Assembly 8x A100 And NVSwitch Heatsinks Side 2](assets/Inspur-NF5488A5-NVIDIA-HGX-A100-8-GPU-Assembly-8x-A100-and-NVSwitch-Heatsinks-Side-2.jpg)

A100 HGX platform代号"**Delta**"，H100 HGX platform代号"**Delta  Next**"



所以总的来说，NVIDIA HGX



## 1.6 SXM







