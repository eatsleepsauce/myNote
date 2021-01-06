#### 如何提升CPU吞吐率

程序的 CPU 执行时间 = 指令数 × CPI × Clock Cycle Time（CPI 的倒数，又叫作 IPC（Instruction Per Clock），也就是一个时钟周期里面能够执行的指令数，代表了 CPU 的吞吐率。）

反复优化流水线架构的 CPU 里，最佳情况下，IPC 也只能到 1。因为无论做了哪些流水线层面的优化，即使做到了指令执行层面的乱序执行，CPU 仍然只能在一个时钟周期里面，取一条指令。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/dd88d0dbf3a88b09d5e8fb6d9e3aea13.jpeg" alt="dd88d0dbf3a88b09d5e8fb6d9e3aea13" style="zoom:25%;" />

现在用的 Intel CPU 或者 ARM 的 CPU，一般的 CPI 都能做到 2 以上，这是怎么做到的呢？

##### 一、多发射与超标量

在指令乱序执行的过程中，取指令（IF）和指令译码（ID）部分并不是并行进行的。其实只要把取指令和指令译码，也一样通过增加硬件的方式，并行进行就好了。一次性从内存里面取出多条指令，然后分发给多个并行的指令译码器，进行译码，然后对应交给不同的功能单元去处理。这样，在一个时钟周期里，能够完成的指令就不只一条了。IPC 也就能做到大于 1 了。这种 CPU 设计，叫作 **多发射（Mulitple Issue）**和 **超标量（Superscalar）**。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/85f15ec667d09fd2d368822904029b32.jpeg" alt="85f15ec667d09fd2d368822904029b32" style="zoom:25%;" />

在超标量的 CPU 里面，有很多条并行的流水线，而不是只有一条流水线。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/2e96fe0985a4ae3bd7a58c345def29d3.jpeg" alt="2e96fe0985a4ae3bd7a58c345def29d3" style="zoom:25%;" />

##### 二、超长指令字设计

无论是之前的乱序执行，还是更进一步的超标量技术，在实际的硬件层面，其实实施起来都挺麻烦的。这是因为，在乱序执行和超标量的体系里面，CPU 要解决依赖冲突的问题。CPU 需要在指令执行之前，去判断指令之间是否有依赖关系。如果有对应的依赖关系，指令就不能分发到执行阶段。这些对于依赖关系的检测，都会使得我们的 CPU 电路变得更加复杂。于是，计算机科学家和工程师们就又有了一个大胆的想法。我们  **能不能不把分析和解决依赖关系的事情，放在硬件里面，而是放到软件里面来干呢**？

**超长指令字设计（Very Long Instruction Word，VLIW）**。这个设计，不仅想让编译器来优化指令数，还想直接通过编译器，来优化 CPI。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/22b3f723ceee5950ac20a7b874dabbde.jpeg" alt="22b3f723ceee5950ac20a7b874dabbde" style="zoom: 25%;" />

编译器在编译过程中，其实也能够知道前后数据的依赖。于是，我们可以让编译器把没有依赖关系的代码位置进行交换。然后，再把多条连续的指令打包成一个指令包。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/f16a1ae443418caca0dc2fc3cec200f6.jpeg" alt="f16a1ae443418caca0dc2fc3cec200f6" style="zoom: 25%;" />

CPU 在运行的时候，不再是取一条指令，而是取出一个指令包。然后，译码解析整个指令包，解析出 3 条指令直接并行运行。使用超长指令字架构的 CPU，同样是采用流水线架构的。也就是说，一组（Group）指令，仍然要经历多个时钟周期。同样的，下一组指令并不是等上一组指令执行完成之后再执行，而是在上一组指令的指令译码阶段，就开始取指令了。**流水线停顿这件事情在超长指令字里面，很多时候也是由编译器来做的。（这是如何实现的？）** 

##### 三、超线程

超线程的 CPU，其实是把一个物理层面 CPU 核心，“伪装”成两个逻辑层面的 CPU 核心。这个 CPU，会在硬件层面增加很多电路，比如，在一个物理 CPU 核心内部，会有双份的 PC 寄存器、指令寄存器乃至条件码寄存器。使得我们可以在一个 CPU 核心内部，维护两个不同线程的指令的状态信息。在外面看起来，似乎有两个逻辑层面的 CPU 在同时运行。所以，超线程技术一般也被叫作 **同时多线程**（Simultaneous Multi-Threading，简称 SMT）技术。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/96aa1220ff27776f55091c55c2eddbc8.jpeg" alt="96aa1220ff27776f55091c55c2eddbc8" style="zoom: 25%;" />

在 CPU 的其他功能组件上，Intel 可不会提供双份。无论是指令译码器还是 ALU，一个 CPU 核心仍然只有一份。因为超线程并不是真的去同时运行两个指令，否则就变成物理多核了。超线程的目的，是在一个线程 A 的指令，在流水线里停顿的时候，让另外一个线程去执行指令。因为这个时候，CPU 的译码器和 ALU 就空出来了，那么另外一个线程 B，就可以拿来干自己需要的事情。这个线程 B 可没有对于线程 A 里面指令的关联和依赖。

##### 四、单指令多数据流

**单指令多数据流 SIMD**（Single Instruction Multiple Data）。在获取数据和执行指令的时候，都做到了并行。一方面，在从内存里面读取数据的时候，SIMD 是一次性读取多个数据。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/48ddcd5ac345091c1be5963d5ef7d7a6.jpeg" alt="48ddcd5ac345091c1be5963d5ef7d7a6" style="zoom: 25%;" />

对于那些在计算层面存在大量“数据并行”（Data Parallelism）的计算中，使用 SIMD 是一个很划算的办法。在这个大量的“数据并行”，其实通常就是实践当中的向量运算或者矩阵运算。在实际的程序开发过程中，过去通常是在进行图片、视频、音频的处理。最近几年则通常是在进行各种机器学习算法的计算。（没有依赖和冒险）



关于超标量和多发射的相关知识，看《计算机组成与设计：硬件 / 软件接口》的 4.10 部分。

Intel CPU 里面的 SIMD 指令具体长什么样，可以去读一读《计算机组成与设计：硬件 / 软件接口》的 3.7 章节。