#### GPU

##### 一、图形渲染的流程

现在我们电脑里面显示出来的 3D 的画面，其实是 **通过多边形组合出来的**。你可以看看下面这张图，你在玩的各种游戏，里面的人物的脸，并不是那个相机或者摄像头拍出来的，而是通过多边形建模（Polygon Modeling）创建出来的。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/0777aed6775051cfd83d0bb512de8722.png" alt="0777aed6775051cfd83d0bb512de8722" style="zoom: 33%;" />

而实际这些人物在画面里面的移动、动作，乃至根据光线发生的变化，都是通过计算机根据图形学的各种计算，实时渲染出来的。

这个对于图像进行实时渲染的过程，可以被分解成下面这样 5 个步骤：**顶点处理**（Vertex Processing）、**图元处理**（Primitive Processing）、**栅格化**（Rasterization）、**片段处理**（Fragment Processing）、**像素操作**（Pixel Operations）。

###### 1、顶点处理

图形渲染的第一步是顶点处理。构成多边形建模的每一个多边形呢，都有多个顶点（Vertex）。这些顶点都有一个在三维空间里的坐标。但是我们的屏幕是二维的，所以在确定当前视角的时候，我们需要 **把这些顶点在三维空间里面的位置，转化到屏幕这个二维空间里面**。这个转换的操作，就被叫作 **顶点处理**。

这样的转化都是通过线性代数的计算来进行的。我们的建模越精细，需要转换的顶点数量就越多，计算量就越大。而且，**这里面每一个顶点位置的转换，互相之间没有依赖，是可以并行独立计算的**。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/04c3da62c382e45b8f891cfa046169de.jpeg" alt="04c3da62c382e45b8f891cfa046169de" style="zoom:25%;" />

###### 2、图元处理

在顶点处理完成之后，需要开始进行第二步，**图元处理，其实就是要把顶点处理完成之后的各个顶点连起来，变成多边形**。其实转化后的顶点，仍然是在一个三维空间里，只是第三维的 Z 轴，是正对屏幕的“深度”。所以我们针对这些多边形，需要做一个操作，叫 **剔除和裁剪**（Cull and Clip），也就是把不在屏幕里面，或者一部分不在屏幕里面的内容给去掉，减少接下来流程的工作量。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/4a20559c43f93177d7a99081a0cd0e1d.jpeg" alt="4a20559c43f93177d7a99081a0cd0e1d" style="zoom:25%;" />

###### 3、栅格化

在图元处理完成之后呢，渲染还远远没有完成。需要进行第三步操作。这个操作就是**把它们（图元）转换成屏幕里面的一个个像素点。这个操作就叫 作栅格化**。这个栅格化操作，有一个特点 **和上面的顶点处理是一样的，就是每一个图元都可以并行独立地栅格化**。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/e60a58e632fc05dbf96eaa5cbb7fb2a6.jpeg" alt="e60a58e632fc05dbf96eaa5cbb7fb2a6" style="zoom:25%;" />

###### 4、片段处理

在栅格化变成了像素点之后，我们的图还是“黑白”的。我们还需要 **计算每一个像素的颜色、透明度等信息，给像素点上色**。这步操作就是 **片段处理**。这步操作，**同样每个片段可以并行独立进行和上面的顶点处理和栅格化一样**。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/490f298719e81beb1871c10566d56308.jpeg" alt="490f298719e81beb1871c10566d56308" style="zoom:25%;" />

###### 5、像素操作

最后一步，我们就要把不同的多边形的像素点“混合（Blending）”到一起。可能前面的多边形可能是半透明的，那么前后的颜色就要混合在一起变成一个新的颜色；或者前面的多边形遮挡住了后面的多边形，那么我们只要显示前面多边形的颜色就好了。最终，输出到显示设备。



经过这完整的 5 个步骤之后，我们就完成了从三维空间里的数据的渲染，变成屏幕上你可以看到的 3D 动画了。这样 5 个步骤的渲染流程一般也被称之为 **图形流水线（Graphic Pipeline）**。

##### 二、为什么需要GPU

如果用 CPU 来进行这个图片渲染过程，需要花上多少资源呢？

>在上世纪 90 年代的时候，屏幕的分辨率还没有现在那么高。一般的 CRT 显示器也就是 640×480 的分辨率。这意味着屏幕上有 30 万个像素需要渲染。为了让我们的眼睛看到画面不晕眩，我们希望画面能有 60 帧。于是，每秒我们就要重新渲染 60 次这个画面。也就是说，每秒我们需要完成 1800 万次单个像素的渲染。从栅格化开始，每个像素有 3 个流水线步骤，即使每次步骤只有 1 个指令，那我们也需要 5400 万条指令，也就是 54M 条指令。

因为图形渲染的流程是固定的，那我们直接用硬件来处理这部分过程，不用 CPU 来计算是不是就好了？很显然，这样的硬件会比制造有同样计算性能的 CPU 要便宜得多。**因为整个计算流程是完全固定的，不需要流水线停顿、乱序执行等等的各类导致 CPU 计算变得复杂的问题**。

所以 **刚开始的GPU，除顶点处理的过程还是都由 CPU 进行的，后续所有到图元和像素级别的处理都是GPU处理**。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/852288ae6b69b7e649c81f90c9fd7cdb.jpeg" alt="852288ae6b69b7e649c81f90c9fd7cdb" style="zoom:25%;" />

##### 三、现代GPU

早期GPU没有“顶点处理“这个步骤。在当时，把多边形的顶点进行线性变化，转化到我们的屏幕的坐标系的工作还是由 CPU 完成的。所以，CPU 的性能越好，能够支持的多边形也就越多，对应的多边形建模的效果自然也就越像真人。而 3D 游戏的多边形性能也受限于我们 CPU 的性能。无论你的显卡有多快，如果 CPU 不行，3D 画面一样还是不行。

###### 1、早期可编程管线的GPU

除 “顶点处理” 挪到了GPU中，一开始的 **可编程管线** （Programable Function Pipeline） GPU，可以在顶点处理（Vertex Processing）和片段处理（Fragment Processing）部分进行编程处理。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/2724f76ffa4222eae01521cd2dffd16d.jpeg" alt="2724f76ffa4222eae01521cd2dffd16d" style="zoom:25%;" />

这些可以编程的接口，我们称之为 **Shader**，中文名称就是 **着色器**。之所以叫“着色器”，是因为一开始这些“可编程”的接口，**只能修改顶点处理和片段处理部分的程序逻辑**——主要是光照、亮度、颜色等等的处理，所以叫着色器。 

这个时候的 GPU，有两类 Shader，也就是 **顶点着色器（Vertex Shader）** 和 **片段着色器（Fragment Shader）**。



虽然在顶点处理和片段处理上的具体逻辑不太一样，但是 **里面用到的指令集可以用同一套**。然而在整个渲染管线里，Vertext Shader 运行的时候，Fragment Shader 停在那里什么也没干。Fragment Shader 在运行的时候，Vertext Shader 也停在那里发呆。于是，**统一着色器架构（Unified Shader Architecture）**就应运而生了。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/dab4ed01f50995d82e6e5d970b54c693.jpeg" alt="dab4ed01f50995d82e6e5d970b54c693" style="zoom:25%;" />

正是因为 **Shader 变成一个“通用”的模块，才有了把 GPU 拿来做各种通用计算的用法**，也就是 GPGPU（General-Purpose Computing on Graphics Processing Units，通用图形处理器）。而正是因为 GPU 可以拿来做各种通用的计算，才有了过去 10 年深度学习的火热。

###### 2、现代GPU

现代GPU的三个核心创意：

**（1）芯片瘦身**

**（2）多核并行和SIMT**

**（3）超线程**