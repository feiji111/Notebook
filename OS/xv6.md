# xv6

在讲xv6这个操作系统之前，首先需要讲一下MIT的操作系统课程

2019 年之前叫 MIT6.828，之后改名为 6.s081，2021 又改了一波名字，最新的名字是 6.1810



最新的xv6采用RISCV架构。构建环境需要用到交叉编译工具链。

交叉编译工具链有一套统一的命名规则，采用Target Triplet来描述一个平台

```
<arch>- <arch>-<vendor>-<os>-<libc/abi>
```

<img src="assets/image-20240929224534981.png" alt="image-20240929224534981" style="zoom:67%;" />

