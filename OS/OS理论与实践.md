# OS



# 1. 调度Schedler

CPU bound(也叫计算密集型)，这种程序CPU越快，程序也会越快，瓶颈在于CPU

I/O bound(也叫I/O密集型)，这种程序I/O越快，程序也会越快，瓶颈在与I/O系统

与之对应的还有

CPU Burst

I/O Burst



# 2. I/O

Linux下的I/O模型有五种



## 2.1 阻塞I/O



## 2.2 非阻塞I/O



## 2.3 I/O多路复用



## 2.4 信号驱动I/O



## 2.5 异步I/O



## 2.6 Zero-copy

当我们想读取磁盘上的文件，并且将其通过网络发送时，最直接的方式是先将文件读入内存中的buffer中，然后再将buffer写入到socket。但事实上这个过程效率非常低，需要进行4次copy(两次CPU copy，两次DMA copy)，并且需要需要4次内核态与用户态的切换。

![img](assets/1TzxTXsM7i1OL2WhH-ooaDQ.png)

但实际上中间两次内核态与用户态的切换以及中间两次copy是没有必要的，浪费了CPU cycles以及内存带宽。

因此Zero-copy就是为了节省CPU cycles以及Memory bandwidth的一种操作。

通过Zero-copy操作，内核能够直接将数据传输给socket，而不用经过用户态。

## 2.7 Copy-on-write

Copy-on-write(COW)，也叫做implicit sharing，shadowing。

COW在`fork()`系统调用，以及C++的`string`等地方使用到，非常经典的例子是`fork()`系统调用。



## 2.8 DMA





## 2.9 IOMMU

 **input–output memory management unit** (**IOMMU**)是一种MMU，用于将DMA-capable I/O bus连接到主存上。和传统的MMU不同，传统的MMU负责将CPU看到的虚拟地址转化为实际的物理地址。而IOMMU则是将设备看到的虚拟地址转化为实际的物理地址。

![undefined](assets/1920px-MMU_and_IOMMU.svg.png)





IOMMU的一个十分重要的应用场景是在虚拟化技术上。

AMD的[VIRTUALIZING IO THROUGH THE IO MEMORY MANAGEMENT UNIT (IOMMU)](https://pages.cs.wisc.edu/~basu/isca_iommu_tutorial/IOMMU_TUTORIAL_ASPLOS_2016.pdf)对IOMMU以及虚拟化有更加详细的介绍。

