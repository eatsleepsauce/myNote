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

