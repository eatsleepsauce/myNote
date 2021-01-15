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

##### 三、调度

无论是 policy 还是 priority，都设置了一个变量，变量仅仅表示了应该这样这样干，但事情总要有人去干，谁呢？在 task_struct 里面，还有这样的成员变量：

```c
const struct sched_class *sched_class;
```

