#### 调度策略

对于操作系统来讲，它面对的 CPU 的数量是有限的，干活儿都是它们，但是进程数目远远超过 CPU 的数目，因而就需要进行进程的调度，有效地分配 CPU 的时间，既要保证进程的最快响应，也要保证进程之间的公平。

linux里面进程分为两类，一种称为 **实时进程**，也就是需要尽快执行返回结果的那种。另一种是 **普通进程**，大部分的进程其实都是这种。对于这两种进程，我们的调度策略肯定是不同的。

**task_struct** 中，有一个成员变量，叫 **调度策略**。

```c
unsigned int policy;
```

定义如下：

```c
#define SCHED_NORMAL    0
#define SCHED_FIFO    1
#define SCHED_RR    2
#define SCHED_BATCH    3
#define SCHED_IDLE    5
#define SCHED_DEADLINE    6
```

**task_struct** 中，还有一个 **配合调度策略**的，就是 **优先级**。

```c
int prio, static_prio, normal_prio;
unsigned int rt_priority;
```

优先级其实就是一个数值，**对于实时进程，优先级的范围是 0～99**；**对于普通进程，优先级的范围是 100～139**。**数值越小，优先级越高**。

##### 一、实时调度策略

调度策略中 **SCHED_FIFO、SCHED_RR、SCHED_DEADLINE** 是 **实时进程的调度策略**。

（1）**SCHED_FIFO**，高优先级的进程可以抢占低优先级的进程，而相同优先级的进程，我们遵循先来先得。

（2）**SCHED_RR 轮流调度算法**，采用时间片，相同优先级的任务当用完时间片会被放到队列尾部，以保证公平性，而高优先级的任务也是可以抢占低优先级的任务。

（3）**SCHED_DEADLINE**，是按照任务的 deadline 进行调度的。当产生一个调度点的时候，DL 调度器总是选择其 deadline 距离当前时间点最近的那个任务，并调度它执行。

##### 二、普通调度策略

**普通进程的调度策略** 有，**SCHED_NORMAL、SCHED_BATCH、SCHED_IDLE**。

（1）**SCHED_NORMAL** 是普通的进程，保证公平就行。

（2）**SCHED_BATCH** 是后台进程，几乎不需要和前端进行交互。这类项目可以默默执行，不要影响需要交互的进程，可以降低它的优先级。

（3）**SCHED_IDLE** 是特别空闲的时候才跑的进程。

##### 三、调度实现

无论是 policy 还是 priority，都设置了一个变量，变量仅仅表示了应该这样这样干，但事情总要有人去干，谁呢？在 task_struct 里面，还有这样的成员变量：

```c
const struct sched_class *sched_class;
```

**sched_class**，是 **执行调度策略逻辑 **的。sched_class的实现：

（1）**stop_sched_class** 优先级最高的任务会使用这种策略，会中断所有其他线程，且不会被其他任务打断。

（2）**dl_sched_class** 就对应上面的 deadline 调度策略。

（3）**rt_sched_class** 就对应 RR 算法或者 FIFO 算法的调度策略，具体调度策略由进程的 task_struct->policy 指定。

（4）**fair_sched_class** 就是普通进程的调度策略。

（5）**idle_sched_class** 就是空闲进程的调度策略。

###### 1、调度算法——完全公平调度算法

实时进程的调度策略 RR 和 FIFO 相对简单一些，而且由于平时常遇到的都是普通进程。普通进程使用的调度策略是 **fair_sched_class**，顾名思义，对于普通进程来讲，公平是最重要的。

CFS 全称 Completely Fair Scheduling，叫完全公平调度。原理如下：

首先，需要记录下进程的运行时间。CPU 会提供一个时钟，过一段时间就触发一个时钟中断。就像咱们的表滴答一下，这个我们叫 Tick。CFS 会为每一个进程安排一个虚拟运行时间 vruntime。如果一个进程在运行，随着时间的增长，也就是一个个 tick 的到来，进程的 vruntime 将不断增大。没有得到执行的进程 vruntime 不变。

那些 vruntime 少的，原来受到了不公平的对待，需要给它补上，所以会优先运行这样的进程。

如何给优先级高的进程多分时间呢？优先级高的袋子大，优先级低的袋子小。这样球就不能按照个数分配了，要按照比例来，大口袋的放了一半和小口袋放了一半，里面的球数目虽然差很多，也认为是公平的。

**虚拟运行时间 vruntime += 实际运行时间 delta_exec * NICE_0_LOAD/ 权重**

同样的实际运行时间，给高权重的算少了，低权重的算多了，但是当选取下一个运行进程的时候，还是按照最小的 vruntime 来的，这样高权重的获得的实际运行时间自然就多了。

###### 2、调度队列与调度实体

 CFS 需要一个数据结构来对 vruntime 进行排序，找出最小的那个。

（1）这个能够排序的数据结构不但需要查询的时候，能够快速找到最小的。

（2）更新的时候也需要能够快速地调整排序，要知道 vruntime 可是经常在变的，变了再插入这个数据结构，就需要重新排序。

**能够平衡查询和更新速度的是树，在这里使用的是红黑树。**

红黑树的的节点是应该包括 vruntime 的，称为 **调度实体**。

在 task_struct 中有这样的成员变量：

```c
struct sched_entity se;
struct sched_rt_entity rt;
struct sched_dl_entity dl;
```

实时调度实体 sched_rt_entity，Deadline 调度实体 sched_dl_entity，以及完全公平算法调度实体 sched_entity。不光 CFS 调度策略需要有这样一个数据结构进行排序，其他的调度策略也同样有自己的数据结构进行排序，因为 **任何一个策略做调度的时候，都是要区分谁先运行谁后运行**。

进程根据自己是实时的，还是普通的类型，通过这个成员变量，将自己挂在某一个数据结构里面，和其他的进程排序，等待被调度。如果这个进程是个普通进程，则通过 sched_entity，将自己挂在这棵红黑树上。

普通进程的调度实体定义如下，这里面包含了 vruntime 和权重 load_weight，以及对于运行时间的统计。

```c
struct sched_entity {
  struct load_weight    load;
  struct rb_node      run_node;
  struct list_head    group_node;
  unsigned int      on_rq;
  u64        exec_start;
  u64        sum_exec_runtime;
  u64        vruntime;
  u64        prev_sum_exec_runtime;
  u64        nr_migrations;
  struct sched_statistics    statistics;
......
};
```

所有可运行的进程通过不断地插入操作最终都存储在以时间为顺序的红黑树中，vruntime 最小的在树的左侧，vruntime 最多的在树的右侧。 CFS 调度策略会选择红黑树最左边的叶子节点作为下一个将获得 CPU 的任务。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/c2b86e79f19d811ce10774688fc0c093.jpeg" alt="img" style="zoom: 33%;" />

这棵红黑树放在哪里呢？就像每个软件工程师写代码的时候，会将任务排成队列，做完一个做下一个。CPU 也是这样的，每个 CPU 都有自己的 struct rq 结构，其用于描述在此 CPU 上所运行的所有进程，其包括一个实时进程队列 rt_rq 和一个 CFS 运行队列 cfs_rq，在调度时，调度器首先会先去 **实时进程队列** 找是否有实时进程需要运行，如果没有才会去 **CFS 运行队列** 找是否有进程需要运行。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/ac043a08627b40b85e624477d937f3fd.jpeg" alt="img" style="zoom: 33%;" />

###### 3、调度工作

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/10381dbafe0f78d80beb87560a9506af.jpeg" alt="img" style="zoom:33%;" />

一个 CPU 上有一个队列，CFS 的队列是一棵红黑树，树的每一个节点都是一个 sched_entity，每个 sched_entity 都属于一个 task_struct，task_struct 里面有指针指向这个进程属于哪个调度类。

