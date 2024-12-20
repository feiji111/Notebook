# 1. CPU



## 1.1 Architecture Microarchitecture And Implementation

这三者是不同的概念，但是互有联系，右图出自[arm](https://www.arm.com/architecture/cpu)<img src="assets/architecture-layer-diagram-600.jpeg" alt="Architecture Layer Diagram " style="zoom:50%;" />

```
The Arm CPU architecture is implemented by a wide range of microarchitectures to deliver software compatibility across a broad range of power, performance, and area points.

- The CPU architecture defines the basic instruction set, and the exception and memory models that are relied on by the operating system and hypervisor.
- The CPU microarchitecture determines how an implementation meets the architectural contract by defining the design of the processor and covering such things as: power, performance, area, pipeline length, and levels of cache.
```

Microarchitecture是对于Architecture的实现，对于同一个Architecture可以有多个Microarchitecture实现。这里的Architecture一般是指ISA(Instruction Set Architecture)。



更加广义的Computer Architecture包含一下几个方面：

- **[Instruction Set Architecture](https://en.wikipedia.org/wiki/Instruction_set_architecture)**
- **[Microarchitecture](https://en.wikipedia.org/wiki/Microarchitecture)**
- **[Systems design](https://en.wikipedia.org/wiki/Systems_design)**



wiki官方对于Computer architecture的一些理解：

- It can sometimes be a high-level description that ignores details of the implementation.[[2\]](https://en.wikipedia.org/wiki/Computer_architecture#cite_note-2) At a more detailed level, the description may include the [instruction set architecture](https://en.wikipedia.org/wiki/Instruction_set_architecture) design, [microarchitecture](https://en.wikipedia.org/wiki/Microarchitecture) design, [logic design](https://en.wikipedia.org/wiki/Logic_design), and [implementation](https://en.wikipedia.org/wiki/Implementation).[[3\]](https://en.wikipedia.org/wiki/Computer_architecture#cite_note-3)



## 1.2 Instruction Set Architecture

### 1.2.1 Classification of ISAs

按照架构的复杂性分类：

- **CISC** 更多的[specialized instructions]()，只不过这些指令很少被使用，一般一些专业软件会使用到这些指令
- **RISC** 对于一些经常被使用的指令高效实现，对于复杂但是不常用的指令，通过[subroutine]()实现(加速经常性事件[**Amdahl's law**](https://en.wikipedia.org/wiki/Amdahl%27s_law))

还有一些其它的ISA，比如VLIW([very long instruction word](https://en.wikipedia.org/wiki/Very_long_instruction_word))，EPIC(*[explicitly parallel instruction computing](https://en.wikipedia.org/wiki/Explicitly_parallel_instruction_computing)*)。

- These architectures seek to exploit [**instruction-level parallelism**](https://en.wikipedia.org/wiki/Instruction-level_parallelism) with less hardware than RISC and CISC by making the [compiler](https://en.wikipedia.org/wiki/Compiler) responsible for instruction issue and scheduling.[[4\]](https://en.wikipedia.org/wiki/Instruction_set_architecture#cite_note-4)



但是也有一些比RISC更加精简的ISA，但是目前只存在于理论，MISC([minimal instruction set computer](https://en.wikipedia.org/wiki/Minimal_instruction_set_computer))与OISC([one-instruction set computer](https://en.wikipedia.org/wiki/One-instruction_set_computer))。



对于一些ISAs，在其基础上还有一些指令集的拓展[Instruction set](https://en.wikipedia.org/wiki/Instruction_set) [extensions](https://en.wikipedia.org/wiki/Processor_supplementary_capability)

<img src="assets/image-20240611163712183.png" alt="image-20240611163712183" style="zoom:50%;" />





CISC：x86 familiy

RISC：arm，RISC-V，mips，PowerPC

VLIW：IA-64

### 1.2.2 Instruction set implementation

这一部分就是[Microarchitecture](#1.3-Microarchitecture)所涉及到的内容



### 1.2.3 machine code

assembly language与numerical machine code都是machine code，只不过表现形式不同。assembly language采用助记号的形式，方便阅读；而numerical machine code就是纯0/1序列。二者之间通过**汇编器**转换。

ISA与machine code并不是一个概念，machine code是0/1的序列，而ISA是诸如`add`等的汇编指令。

早期的CPU，不同的CPU都有专门的machine code，因此新的无法做到向后兼容。而通过引入ISA，作为一个abstraction layer，规定了一套指令集的行为以及编码规则，**不关心具体的实现**。从而使得同一系列的CPUs能够相互兼容。



比如对于RISC-V指令集，其每一条指令长度固定为32bit，不同指令的编码也不同，其具体的行为也不同。这些就是ISA规定的内容。

<img src="assets/image-20240612142544799.png" alt="image-20240612142544799" style="zoom:67%;" />

**只要CPU是同一个ISA，那么对于同一条指令，不同CPU上最后生成的machine code(0/1序列)是相同的。**但是ISA不规定ISA的实现方式，即不规定指令的执行方式。具体表现为，对于一条加法指令，其生成的machine code的0/1序列如何对应到一组控制信号控制control unit控制CPU的data flow是不同的，但最终都要实现一个加法操作。这一部分就涉及到microarchitecture。



**这里就涉及到一个问题，对于同样是x86架构的intel和AMD的CPU，对于同样的x86_64指令，最后汇编器生成的机器码是否会相同？**



### 1.2.4 一些拓展指令

当我们使用CPU-Z查看CPU信息时(CPU-Z查看CPU信息也涉及到一种拓展指令，**[CPUID](#1.2.4.2-CPUID)**)，

#### 1.2.4.1 Advanced Vector Extensions(AVX)

AVX是一种SIMD类型的指令，拓展了原有的X86架构。最早由Intel提出，并且应用在Sandy Bridge微架构中。

AVX2在AVX的基础上，增加了256位的整数向量操作，并且提出了一些新的指令。最早应用在Haswell微架构中。

AVX-512则由256位拓展到了512位，最早应用在Intel [Xeon Phi x200](https://en.wikipedia.org/wiki/Xeon_Phi#Knights_Landing)中。



#### 1.2.4.2 MMX



#### 1.2.4.3 SSE



#### 1.2.4.4 加密解密相关拓展指令



#### 1.2.4.5 虚拟化相关指令



#### 1.2.4.6 CPUID





那么对于上述的这些拓展指令，实际上是如何使用的，参考[算法工程化]()

## 1.3 Microarchitecture

对于同一个ISA，不同的microarchitecture对于最终的`cost, performance, power consumption, size`这几个方面有不同的影响，需要在这些指标当中作出权衡。



首先，CPU主要包含ALUs，Registers以及Controllers(control unit)......(当然还有一些其它的components)。

CPU主要分为两部分：

- **datapath** datapath部分包括ALU，Register等，这些又可以细分为
  - 组合逻辑单元combinational elements 输出只与输入有关。ALU属于此类。
  - 状态单元state element 输出不仅与输入有关，还与当前的状态有关。寄存器，寄存器组(**register file**)，memory属于此类。
- **control** control部分包括control unit，control又可以包含两部分，**a combinational part that lacks state and a sequential**
  **control unit that handles sequencing and the main control in a multicycle design**



因此设计一个CPU，就涉及到**datapath**与**control**的设计，大概的流程：

1. 分析每条指令的功能，并用RTL(Register Transfer Language) 来表示。
2. 根据指令的功能给出所需的元件，并考虑如何将他们互连(datapath的设计)。
3. 确定每个元件所需控制信号的取值。
4. 汇总所有指令所涉及到的控制信号，生成一张反映指令与控制信号之间关系的真值表。
5. 根据表得到每个控制信号的逻辑表达式，据此设计控制器电路。



设计microarchitecture时，会使用一些已经设计好的模块(IP核？)，比如adders，registers，ALUs等，只需要考虑如何将其连接起来即可。并且会用到**RTL([register transfer language](https://en.wikipedia.org/wiki/Register_transfer_language))**。使用RTL可以能够描述每一条指令的decoding以及sequencing过程，而Controller就是用以实现RTL描述的decoding以及sequencing过程。

Controller的实现方式有两种：

- hardwire
- [microcode](https://en.wikipedia.org/wiki/Microcode)



除此之外，CPU的设计还可以分为：

- 单周期实现
- 多周期实现



### 1.3.1 Control实现

前面提到，control又可以分为组合逻辑部分与时序逻辑部分。组合逻辑部分一般是decoder，ALU controller等，而单周期的control可以只用组合逻辑就可以实现。下面都是以RISC-V ISA为例，研究RISC-V的不同实现方式。

这些实现都是依赖于RISC-V的指令格式，如下

<img src="assets/image-20240615161109134.png" alt="image-20240615161109134" style="zoom:67%;" />

#### 1.3.1.1 hardwire方式(Hardwired Control Unit)

hardwire的方式实现是通过组合逻辑电路实现。对于每一条指令的machine code(0/1序列)，control unit会产生相应的control signals，从而控制datapath，实现对应的操作。

大致结构如下

<img src="assets/1920px-Animation_of_an_LDA_instruction_performed_by_the_control_matrix_of_a_simple_hardwired_control_unit.gif" alt="undefined" style="zoom: 33%;" />

hardwire方式的实现又可以分为Single Cycle和Multi Cycle实现。



##### 1.3.1.1.1 Single Cycle实现

![image-20240615155329036](assets/image-20240615155329036.png)

单周期的实现的hardwired control unit就只需要组合逻辑电路，对于每条指令产生对应的控制信号。

这里以RISC-V的部分指令为例，按照[1.3](#1.3-Microarchitecture)中的设计方法，得到一个真值表。RISC-V中指令类型除了与op字段有关，还和func字段有关。因此需要两个控制器，op(主控，用以产生ALUOp bits)，func(ALU局控，产生真正的用于控制ALU的信号)，所以还需要一个func字段相关的真值表。

**这种多层的decoding是一种常用的技术。**

ALU control function有4个bit的输出(**Operation3, Operation2, Operation1, and Operation0**)，真值表如下

<img src="assets/image-20240615155037248.png" alt="image-20240615155037248" style="zoom: 67%;" />

实际上ALU control function的真值表是从下面这张表简化而来的(**参考数电中的真值表化简**)。

<img src="assets/image-20240615161802613.png" alt="image-20240615161802613" style="zoom:67%;" />

通过真值表，可以设计出ALU controller

<img src="assets/image-20240615155809263.png" alt="image-20240615155809263" style="zoom:67%;" />

每条指令与真值表是一一对应的关系，通过真值表，设计组合逻辑电路，就可以实现decoder。

而main control function真值表如下

<img src="assets/image-20240615160205875.png" alt="image-20240615160205875" style="zoom:67%;" />

得到真值表后，就可以利用数字逻辑的方法设计组合逻辑电路，设计出的main controller

<img src="assets/image-20240615160302097.png" alt="image-20240615160302097" style="zoom:67%;" />





采用单周期，弊端在于由于每一条指令执行的各个阶段都在一个时钟周期内完成，因此data path上的对于一条指令的执行任何一个functional unit不能被使用到两次，如果有的单元需要被用到两次(比如Memory既要读取一次指令又要读取一次数据)，就需要多个同样的functional unit。

##### 1.3.1.1.2 Multi Cycle实现

而对于Multi Cycle的hardwired control的实现，则需要组合逻辑与时序逻辑的结合。

sequential control function以opcode字段与当前状态作为输入，输出是control signal以及下一个状态。



整体的结构如下

<img src="assets/Hardwire.png" alt="img" style="zoom:50%;" />

整个fixed logic circuit由encoder与decoder构成。

输入为IR(Instruction Register)中的指令，以及External Inputs(比如intertupt signals，以及Conditional Codes(比如一些标记位)，输出为control signals。

decoder用于解码指令，解码后的输出与external input和conditional code一起作为encoder的输入，最后输出control signal。









一条指令有时并不是在一个时钟周期内执行完毕的，会被拆解成几个部分，在时钟的协调下执行。



采用hardwire方式实现的control units更快但是不灵活。







比如对于MIPS中的



### 1.3.2 Microcode方式(Microcode方式只适用于多周期，Microprogrammed Control Unit)

microcode也叫做microprogram，由Maurice Wilkes发明，他也因此获得了图灵奖。

采用microcode方式实现的control units，microcode也相当于ISA与microarchitecture之间的一层abstraction layer。

通过microcode来实现ISA的指令(machine code)，microcode是circuit-level的操作。每一条指令通过微程序(microprogram)实现，微程序由一系列微指令(microinstruction)构成，而一条微指令又是由多个微命令(微操作，micro-operations，也叫做μOps)。微命令/微操作可以理解为微架构内部的操作。**一般来说一个微操作对应着CPU微架构中的datapath一个组件的控制信号。**

这些μOps会被按照一定的编码方式编码，然后存放在微指令中。

<img src="assets/image-20241016191439004.png" alt="image-20241016191439004" style="zoom:50%;" />

微指令通过microcode execution engine解码，产生对应的控制信号。所有指令的微程序存放在Control Storage(控存)中，这个控存可以是一个ROM也可以是PLA。

![undefined](assets/2560px-Micro-operations.svg.png)

一些复杂的指令会被解码成多条μOps，



微指令格式的设计非常重要：

1. 水平型微指令(**Horizontal microcode**)：
   - 直接控制(不编码)
   - 字段直接编码(译)法
   - 字段间接编码(译)法
2. 垂直型微指令(**Vertical microcode**)





一些硬件厂商有时也会用microcode代指firmware。







## 1.4 System Design

**von Neumann architecture(Princeton architecture)**

<img src="assets/2560px-Von_Neumann_Architecture.svg.png" alt="undefined" style="zoom: 25%;" />

**Harvard architecture**

<img src="assets/2560px-Harvard_architecture.svg.png" alt="undefined" style="zoom:25%;" />



## 1.5 Modern Processor

参考：

- [Modern Microprocessors A 90-Minute Guide!](https://www.lighterra.com/papers/modernmicroprocessors/)

现代处理器的几个主要特点：

- pipelining (superscalar, OOO, VLIW, branch prediction, predication)
- multi-core and simultaneous multi-threading (SMT, hyper-threading)
- SIMD vector instructions (MMX/SSE/AVX, AltiVec, NEON)
- caches and the memory hierarchy



### 1.5.1 Pipeling & Instruction-Level Parallelism



取指(Fetch)->解码(Decode)->执行(Execute)->写回(Write Back)

现代处理器的深度大约是15~20。

Fetch与Decode阶段构成前端(Front End)， 6~10个流水线阶段。

Execute与Write Back构成后端(Back End)，6~10个流水线阶段。





Branch-prediction



### 1.5.2 



## 1.6 实例分析Intel





# 2. 制程

制程**technology node** (also **process node**, **process technology** or simply **node**)，最早是用以表示栅极的长度(**gate length**)和金属半间距(M1 half-pitch size，这里的M1指的是lowest metal layer，metal 1)。

每一次制程的提升，一部分是缩短晶体管之间的距离(缩小half-pitch size)，另一个就是缩小晶体管的尺寸(gate length)。

但是后来随着制程的不断进步(FinFET等技术，Intel在22nm节点首先引入)，"***nm**"这个数字失去了其原本的意义，最近的一些制程纯粹是指采用特定技术制造的特定一代芯片，不对应于任何栅极长度或半间距，只是代表一种技术的迭代，就像车型号一样具备不同的意义

```
“It used to be the technology node, the node number, means something, some features on the wafer. Today, these numbers are just numbers. They’re like models in a car – it’s like BMW 5-series or Mazda 6.  It doesn’t matter what the number is, it’s just a destination of the next technology, the name for it. So, let’s not confuse ourselves with the name of the node with what the technology actually offers.”
```

![tech node naming.svg](assets/881px-tech_node_naming.svg.png)

线性的0.7倍缩放，在面积上就约为0.5倍，遵循着摩尔定律。

- **0.7x feature size叫做一个“大换代”Full-node**
- **0.9x feature size叫做一个“小换代”Half-node**

# 3. Intel

![image-20240717185904956](assets/image-20240717185904956.png)

## 3.1 Intel制程





## 3.2 List of Intel CPU microarchitectures(Intel CPU微架构的演变)



### 3.2.1 8086



### 3.2.2 80186



### 3.2.3 80286



### 3.2.4 80386



### 3.2.5 80486



### 3.2.6 P5

初代Pentium(也叫586)采用P5架构，dieshot如下

![Intel Pentium A80501 66 MHz SX950 die image](assets/1920px-Intel_Pentium_A80501_66_SX950.JPG)

![undefined](assets/1920px-Intel_Pentium_arch.svg.png)

P5架构的特性：

- superscalar架构
- 64-bits external databus
- code caches与data caches分离
- FPU(float-point-unit)
- Four-input address adders
- hardware-based multiplier
- Vitualized interrupt
- Branch prediction



**[Superscalar]()**



**Branch Prediction**



### 3.2.7 P6





### 3.2.8 NetBurst



在NetBurst架构下，首次出现了Intel的多核CPU，**Pentium D**。但是Pentium D的双核并不是现在常见的双核集成在一块Die上，而是在两个Die上，将2个奔腾4Prescott的核心封装在一起，通过前端总线（FSB）分别连接北桥，通过北桥来连接两个核心。因此这种双核其实是一种“**胶水多核**”。

### 3.2.9 Core



### 3.2.10 Nehalem



### 3.2.11 Sandy Bridge



### 3.2.12 Haswell



### 3.2.13 Skylake



### 3.2.14 Cypress Cove



### 3.2.15 Palm Cove



### 3.2.16 Sunny Cove



### 3.2.17 Willow Cove



### 3.2.18 Golden Cove



### 3.2.19 Raptor Cove



### 3.2.20 Redwood Cove



### 3.2.21 Lion Cove





### Tick–tock model

Intel于2007年采用Tick-tock模式。

```
Under this model, every microarchitecture change (tock) was followed by a die shrink of the process technology (tick).
```

Tock改进微架构，Tick改进制程工艺。



# 4. AMD



## 4.1 AMD采用制程

AMD采用TSMC制程。



## 4.2 List of AMD CPU microarchitectures(AMD CPU微架构的演变)



### 4.2.? ZEN2





## 4.? AMD Secure Processor 

AMD Secure Processor(**AMD Platform Security Processor** (**PSP**), officially known as **AMD Secure Technology**)是一种[trusted execution environment](https://en.wikipedia.org/wiki/Trusted_execution_environment)子系统。

AMD的CPU是x86架构，但是其内部有一个ARM Cortex-A5的ARM核心。这个ARM核心带有[TrustZone](https://en.wikipedia.org/wiki/TrustZone)拓展。这个ARM核心被集成到了AMD的CPU die中，作为一个协处理器。这个处理器上运行着一个on-chip firmware，用于验证SPI ROM以及从这个ROM中加载off-chip firmware。



AMD的EYPC系列CPU会锁主板就是因为这个Secure Processor的存在。

DELL的官方对其有相应的说明[Defense in-depth: Comprehensive Security on PowerEdge AMD EPYC Generation 2 (Rome) Servers](https://dl.dell.com/manuals/common/security_poweredge_amd_epyc_gen2.pdf)

```
On system power-on or reset, the AMD Secure Processor executes its firmware while the main CPU cores are held in reset. One of the AMD Secure Processor’s tasks is to provide a secure hardware root-of-trust by authenticating the initial PowerEdge BIOS firmware. If the initial PowerEdge BIOS is corrupted or compromised, the AMD Secure Processor will halt the system and prevent OS boot. If no corruption, the AMD Secure Processor starts the main CPU cores, and initial BIOS execution begins. 

The very first time a CPU is powered on (typically in the Dell EMC factory) the AMD Secure Processor permanently stores a unique Dell EMC ID inside the CPU. This is also the case when a new off-the-shelf CPU is installed in a Dell EMC server. The unique Dell EMC ID inside the CPU binds the CPU to the Dell EMC server. Consequently, the AMD Secure Processor may not allow a PowerEdge server to boot if a CPU is transferred from a non-Dell EMC server (and CPU transferred from a Dell EMC server to a non-Dell EMC server may not boot). 
```







# 5. 关于X86授权

x86最早由Intel提出，最早用于Intel 8086 CPU上。当时IBM指定了Intel的8086处理器作为IBM个人电脑的cpu芯片。但是IBM要求同一个芯片至少要拥有两家供应商。当时Intel还是小公司，于是机缘巧合找到AMD作为第二供应商。后来还有其它公司得到x86授权。比如[Cyrix](https://link.zhihu.com/?target=https%3A//zh.wikipedia.org/wiki/Cyrix)（现为[威盛电子](https://link.zhihu.com/?target=https%3A//zh.wikipedia.org/wiki/%E5%A8%81%E7%9B%9B%E9%9B%BB%E5%AD%90)所收购）。

```
1982年2月，AMD与Intel签约，成为得到许可的8086与8088制造业者和第二货源生产商。IBM要用Intel 8088在他们的IBM PC，但是IBM当时的政策要求他们所使用的芯片至少要有两个货源。在同样的安排下，AMD之后生产80286。
但是在1986年，Intel撤回了这个协定，拒绝传达i386的技术详情。由于PC clone市场的流行与增长，Intel可以依照自己的规格来制造CPU，而不是依照IBM规格。
AMD告Intel毁约，仲裁判AMD胜诉。但是Intel对此提出上诉。接下来，长期的法庭战争在1994年结束。加州最高法院判AMD胜诉，要Intel赔超过10亿美元的赔偿金。后来的法庭战争聚集在AMD是否有权利使用Intel派生的微代码(Microcode)。裁决没有明显的偏向任何一方。在面对这个不确定的情况，AMD被迫开发“防尘室版”的Intel微代码。他们的方式是：一组工程师描述微代码的功能，另一组在没有参考原微代码的情况下，自行开发拥有同样功能的微代码。
1991年，AMD发布Am386，Intel 80386的复制版。AMD在一年以内就销售了100万只芯片。AMD接下来在1993年发布Am486。两者都以比Intel版本更低的价格销售。很多OEM，包括Compaq都使用Am486。由于电脑工业生产周期的缩短，逆向工程Intel产品的策略令AMD越来越难继续生存下去。因为这意味着他们的技术将一直落在Intel的后头。因此，他们开始开发他们自己的微处理器。[2]
```

因此目前拥有X86授权的Intel，AMD，VIA(威盛)，兆芯等为数不多的企业。



# 6. CPU片内总线

参考：

- https://pcper.com/2017/06/intel-skylake-x-and-skylake-sp-utilize-mesh-architecture-for-intra-chip-communication/
- https://www.servethehome.com/the-new-intel-mesh-interconnect-architecture-and-platform-implications/
- https://www.servethehome.com/things-are-getting-meshy-next-generation-intel-skylake-sp-cpus-mesh-architecture/
- https://www.intel.cn/content/www/cn/zh/developer/articles/technical/xeon-processor-scalable-family-technical-overview.html
- [破茧化蝶，从Ring Bus到Mesh网络，CPU片内总线的进化之路](https://zhuanlan.zhihu.com/p/32216294)

![img](assets/220820031101.jpg)



片内总线负责CPU内部各个模块之间的连接。

最早的单核CPU采取的就是Star结构的片内总线，最中间的是CPU核心。但随着CPU由单核向多核转变，Star结构无法满足多核处理器的要求，因此片内总线由Star转为了Ring结构。

<img src="assets/8dbc-xeon-processor-5.jpg" alt="xeon-processor-5.jpg" style="zoom:67%;" />

上面是Intel Nehalem架构的片内总线。



Ring总线由两个环组成，一个顺时针环和一个逆时针环。不同模块通过**Ring Stop**挂载到总线上。Ring Stop中的control logic会决定是否数据会交给对应的模块。Ring总线的设计以下好处

```
1.双环设计可以保证任何两个ring stop之间距离不超过Ring Stop总数的一半，延迟较低。
2.各个模块之间交互方便，不需要Core中转。这样一些高级加速技术，如DCA（Direct Cache Access), Crystal Beach等等应运而生。
3.高速的ring bus保证了性能的极大提高。Core to Core latency 只有60ns左右，而带宽则高达数百G(记得Nehalem是192GB/s).
4.方便灵活。增加一个Core，只要在Ring上面加个新的ring stop就好，不用像以前一样考虑复杂的互联问题。
```

但是CPU核心数量比较少的时候，Ring总线能够性能非常好，但是随着核心数量的增加，Ring总线的长度也会变长，Ring总线的性能会随之下降，跨核之间的通信延迟会很高。

一般来说，单Ring总线的极限是12核心，多于12C之后会带来严重的延迟。因此就有了多Ring总线。



下面是Intel 单Ring，1.5 Ring与双Ring总线结构。

Intel针对不同的规格的Die(核心数量)也有不同的定义： **Low Core Count (LCC)**，**Medium Core Count (MCC)**，**High Core Count (HCC)**。

![img](assets/syvQyzBnPZ52rRWuJfRyZV.png)

![img](assets/R469BCeP6uv4kzr7efAPxM.png)

在多Ring结构中，两个Ring通过一个双向的Pipeline连接。由于Ring间的通信延迟要远高于Ring内的通信延迟，因此两个Ring属于不同的[NUMA]() node。



但是Ring总线的自身局限，使得其无法用于超多核心的CPU中，因此Mesh总线出现。Intel于Skylake与Knight Landing中引入了新的Mesh结构的片内总线。

<img src="assets/d7f1-intelmesh.png" alt="Intel Skylake-X and Skylake-SP Utilize Mesh Architecture for Intra-Chip Communication" style="zoom:67%;" />



Mesh总线结构相比于Ring总线有以下优势

```
1.首先当然是灵活性。新的模块或者节点在Mesh中增加十分方便，它带来的延迟不是像ring bus一样线性增加，而是非线性的。从而可以容纳更多的内核。
2.设计弹性很好，不需要1.5 ring和2ring的委曲求全。
3.双向mesh网络减小了两个node之间的延迟。过去两个node之间通讯，最坏要绕过半个ring。而mesh整体node之间距离大大缩减。
4.外部延迟大大缩短
```

访存延迟

![img](assets/Broadwell-Ring-v-Skylake-Mesh-DRAM-Example.jpg)

IO延迟

![Broadwell Ring V Skylake Mesh PCIe Example](assets/Broadwell-Ring-v-Skylake-Mesh-PCIe-Example.jpg)



但是Mesh带来的弊端是Die的大小会变大。 



# 7. NoC(Network on Chip) for CPU

现如今的处理器都是多核处理器，随着CPU核心的增加，核间通信就显得至关重要。



# 8. 众核 

Xeon Phi

# 9. NUMA与UMA



## 9.1 UMA(Uniform memory access)

UMA是一种多核处理器的内存架构：

1. UMA与SMP是配套的，UMA采用bus-based SMP架构，所有核心连接在内存总线上访问内存，不同核心访问内存是相同的
2. UMA采用了crossbar switches
3. UMA采用[multistage interconnection networks](https://en.wikipedia.org/wiki/Multistage_interconnection_networks)



早期的计算机，内存控制器还没有整合进 CPU，所有的内存访问都需要经过北桥芯片来完成。CPU 通过前端总线（FSB，Front Side Bus）连接到北桥芯片，然后北桥芯片连接到内存，内存控制器集成在北桥芯片里面。

UMA的拓展能力有限，随着处理器核心数量不断增加，更多的核心竞争内存总线，访问冲突会迅速增加。FSB成为性能瓶颈。



## 9.2 NUMA(Non-uniform memory access)

NUMA也是一种为多核处理器设计的内存架构。在NUMA架构下，所有的CPU核心被划分为多个NUMA结点，所有NUMA结点共享系统内存。每个NUMA结点有自己的local memory，同时也可以访问其它NUMA结点的non-local memory，只不过访问会更慢。

NUMA是为了解决UMA的缺点。

1. CPU 厂商把内存控制器集成到 CPU 内部，一般一个 CPU socket 会有一个独立的内存控制器。
2. 每个 CPU scoket 独立连接到一部分内存，这部分 CPU 直连的内存称为“本地内存”。
3. CPU 之间通过 QPI（Quick Path Interconnect） 总线进行连接。CPU 可以通过 QPI 总线访问不和自己直连的“远程内存”。

![img](assets/20220602110808.png)

但是像这种多CPU只有在多路服务器上才能够见到，因此一般只有在多路服务器上才能够对NUMA进行配置，而在一般的消费级电脑上无法配置。

![03-01-System_socket_die_core_HT](assets/03-01-System_socket_die_core_HT.svg)

# 10. CPU视角下的整数、定点与浮点

定点与浮点都是计算机中用于表示小数的方式。



当谈论到浮点数时，就必须提到FPU以及IEEE 754 Floating Point Standard。

早期的CPU(8088/8086，80286，80386)是无法进行浮点数运算的，只能够进行定点运算。



而为了让CPU能够执行浮点数运算，通常有以下三种方式：

1. A floating-point unit emulator (a floating-point library in software，即软件模拟浮点运算，称之为软浮点)
2. Add-on FPU hardware
3. Integrated FPU (in hardware)



接下来分别简单讲一下三种方式的具体实现。

## 10.1 Floating-point library(软件模拟FPU)



## 10.2 Integrated FPUs



## 10.3 Add-on FPUs





现代CPU中，ALU负责整数运算，而FPU负责浮点数运算。

因此CPU的性能也分为整数性能与浮点数性能，并且各自有其各自的应用场景。





但是对于一些嵌入式处理器来说，由于资源首先，并不是所有的CPU都有FPU。



# 11. 查看CPU信息

Linux下查看CPU信息有很多方式，`lspci`是比较常用的



步进stepping

修订Revision

对于一个CPU，第一个版本是A-0(修订A，步进0)，后续每当对CPU进行修改时，步进都会增加。一般来说，小改动步进增加；大改动修订增加。

但是lscpu并不能获得修订号，修订号与步进号的获得要更加复杂。

Model CPU型号



CPU Family：

- 0=8086/8088 processor
- 2=Intel 286 processor

