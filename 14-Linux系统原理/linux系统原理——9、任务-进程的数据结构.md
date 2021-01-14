#### 进程的数据结构

在 Linux 里面，无论是进程，还是线程，到了内核里面，统一都叫 **任务（Task），由一个统一的结构 task_struct 进行管理**。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/75c4d28a9d2daa4acc1107832be84e2d.jpeg" alt="img" style="zoom: 25%;" />



<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/1c91956b52574b62a4418a7c6993d8bc.jpeg" alt="1c91956b52574b62a4418a7c6993d8bc" style="zoom: 33%;" />

task_struct 包含的内容：

##### 1、任务ID

每一个任务都应该有一个 ID，作为这个任务的唯一标识。到时候排期啊、下发任务啊等等，都按 ID 来，就不会产生歧义。task_struct 里面涉及任务 ID 的，有下面几个：

```c
pid_t pid;
pid_t tgid;
struct task_struct *group_leader; 
```

其中，pid 是 process id，tgid 是 thread group ID。任何一个进程，**如果只有主线程，那 pid 是自己，tgid 是自己，group_leader 指向的还是自己**。但是，**如果一个进程创建了其他线程，那就会有所变化了。线程有自己的 pid，tgid 就是进程的主线程的 pid，group_leader 指向的就是进程的主线程。**

##### 2、信号处理

```c
/* Signal handlers: */
struct signal_struct    *signal;
struct sighand_struct    *sighand;
sigset_t      blocked;
sigset_t      real_blocked;
sigset_t      saved_sigmask;
struct sigpending    pending;
unsigned long      sas_ss_sp;
size_t        sas_ss_size;
unsigned int      sas_ss_flags;
```

这里定义了哪些信号被阻塞暂不处理（blocked），哪些信号尚等待处理（pending），哪些信号正在通过信号处理函数进行处理（sighand）。处理的结果可以是忽略，可以是结束进程等等。

##### 3、任务状态

```c
 volatile long state;    /* -1 unrunnable, 0 runnable, >0 stopped */
 int exit_state;
 unsigned int flags;
```

state（状态）可以取的值定义在 include/linux/sched.h 头文件中。

```c
/* Used in tsk->state: */
#define TASK_RUNNING                    0
#define TASK_INTERRUPTIBLE              1
#define TASK_UNINTERRUPTIBLE            2
#define __TASK_STOPPED                  4
#define __TASK_TRACED                   8
/* Used in tsk->exit_state: */
#define EXIT_DEAD                       16
#define EXIT_ZOMBIE                     32
#define EXIT_TRACE                      (EXIT_ZOMBIE | EXIT_DEAD)
/* Used in tsk->state again: */
#define TASK_DEAD                       64
#define TASK_WAKEKILL                   128
#define TASK_WAKING                     256
#define TASK_PARKED                     512
#define TASK_NOLOAD                     1024
#define TASK_NEW                        2048
#define TASK_STATE_MAX                  4096
```

从定义的数值很容易看出来，state 是通过 bitset 的方式设置的，也就是说，当前是什么状态，哪一位就置一。**state如下：**

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/e2fa348c67ce41ef730048ff9ca4c988.jpeg" alt="e2fa348c67ce41ef730048ff9ca4c988" style="zoom:33%;" />

（1）**TASK_RUNNING**  并不是说进程正在运行，而是表示进程在时刻准备运行的状态。当处于这个状态的进程获得时间片的时候，就是在运行中；如果没有获得时间片，就说明它被其他进程抢占了，在等待再次分配时间片。

（2）**睡眠状态**，在运行中的进程，一旦要进行一些 I/O 操作，需要等待 I/O 完毕，这个时候会释放 CPU，进入睡眠状态。

- **TASK_INTERRUPTIBLE**，可中断的睡眠状态。是一种浅睡眠的状态，也就是说，虽然在睡眠，等待 I/O 完成，但是这个时候一个信号来的时候，进程还是要被唤醒。只不过唤醒后，不是继续刚才的操作，而是进行信号处理。
- **TASK_UNINTERRUPTIBLE**，不可中断的睡眠状态。这是一种深度睡眠状态，不可被信号唤醒，只能死等 I/O 操作完成。一旦 I/O 操作因为特殊原因不能完成，这个时候，谁也叫不醒这个进程了。你可能会说，我 kill 它呢？别忘了，kill 本身也是一个信号，既然这个状态不可被信号唤醒，kill 信号也被忽略了。除非重启电脑，没有其他办法。
- **TASK_KILLABLE**，可以终止的新睡眠状态。进程处于这种状态中，它的运行原理类似 TASK_UNINTERRUPTIBLE，只不过可以响应致命信号。

（3）**TASK_STOPPED** 是在进程接收到 SIGSTOP、SIGTTIN、SIGTSTP 或者 SIGTTOU 信号之后进入该状态。

（4）**TASK_TRACED** 表示进程被 debugger 等进程监视，进程执行被调试程序所停止。当一个进程被另外的进程所监视，每一个信号都会让进程进入该状态。

（5）**EXIT_ZOMBIE** 一旦一个进程要结束，先进入的是 EXIT_ZOMBIE 状态，但是这个时候它的父进程还没有使用 wait() 等系统调用来获知它的终止信息，此时进程就成了僵尸进程。

（6）**EXIT_DEAD** 是进程的最终状态

**EXIT_ZOMBIE** 和 **EXIT_DEAD** 也可以用于 **exit_state**。

**flags 标志**，例如 进程状态和进程的运行、调度有关系，还有其他的一些状态，flags的值：

```c
#define PF_EXITING    0x00000004
#define PF_VCPU      0x00000010
#define PF_FORKNOEXEC    0x00000040
```

（1）**PF_EXITING** 表示正在退出。当有这个 flag 的时候，在函数 find_alive_thread 中，找活着的线程，遇到有这个 flag 的，就直接跳过。

（2）**PF_VCPU** 表示进程运行在虚拟 CPU 上。在函数 account_system_time 中，统计进程的系统运行时间，如果有这个 flag，就调用 account_guest_time，按照客户机的时间进行统计。

（3）**PF_FORKNOEXEC** 表示 fork 完了，还没有 exec。在 _do_fork 函数里面调用 copy_process，这个时候把 flag 设置为 PF_FORKNOEXEC。当 exec 中调用了 load_elf_binary 的时候，又把这个 flag 去掉。

##### 4、调度相关

进程的状态切换往往涉及调度，下面这些字段都是用于调度的。

```c
//是否在运行队列上
int        on_rq;
//优先级
int        prio;
int        static_prio;
int        normal_prio;
unsigned int      rt_priority;
//调度器类
const struct sched_class  *sched_class;
//调度实体
struct sched_entity    se;
struct sched_rt_entity    rt;
struct sched_dl_entity    dl;
//调度策略
unsigned int      policy;
//可以使用哪些CPU
int        nr_cpus_allowed;
cpumask_t      cpus_allowed;
struct sched_info    sched_info;
```



##### 5、运行统计

在进程的运行过程中，会有一些统计量，具体你可以看下面的列表。这里面有进程在用户态和内核态消耗的时间、上下文切换的次数等等。

```c
u64        utime;//用户态消耗的CPU时间
u64        stime;//内核态消耗的CPU时间
unsigned long      nvcsw;//自愿(voluntary)上下文切换计数
unsigned long      nivcsw;//非自愿(involuntary)上下文切换计数
u64        start_time;//进程启动时间，不包含睡眠时间
u64        real_start_time;//进程启动时间，包含睡眠时间
```

##### 6、亲缘关系

任何一个进程都有父进程。所以，整个进程其实就是一棵进程树。而拥有同一父进程的所有进程都具有兄弟关系。

```c
struct task_struct *real_parent; /* real parent process */
struct task_struct *parent; /* recipient of SIGCHLD, wait4() reports */
struct list_head children;      /* list of my children */
struct list_head sibling;       /* linkage in my parent's children list */
```

parent 指向其父进程。当它终止时，必须向它的父进程发送信号。children 表示链表的头部。链表中的所有元素都是它的子进程。sibling 用于把当前进程插入到兄弟链表中。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/92711107d8dcdf2c19e8fe4ee3965304.jpeg" alt="92711107d8dcdf2c19e8fe4ee3965304" style="zoom:33%;" />

real_parent 和 parent 是一样的，但是也会有另外的情况存在。例如，bash 创建一个进程，那进程的 parent 和 real_parent 就都是 bash。如果在 bash 上使用 GDB 来 debug 一个进程，这个时候 GDB 是 parent，bash 是这个进程的 real_parent。

##### 7、权限

```c
/* Objective and real subjective task credentials (COW): */
const struct cred  *real_cred;
/* Effective (overridable) subjective task credentials (COW): */
const struct cred  *cred;
```

这个结构的注释里，Objective 和 Subjective。事实上，所谓的权限，就是我能操纵谁，谁能操纵我。

“操作”，就是一个对象对另一个对象进行某些动作。当动作要实施的时候，就要审核权限，当两边的权限匹配上了，就可以实施操作。其中，**real_cred 就是说明谁能操作我这个进程**，而 **cred 就是说明我这个进程能够操作谁**。

cred 的定义如下：

```c
struct cred {
......
        kuid_t          uid;            /* real UID of the task */
        kgid_t          gid;            /* real GID of the task */
        kuid_t          suid;           /* saved UID of the task */
        kgid_t          sgid;           /* saved GID of the task */
        kuid_t          euid;           /* effective UID of the task */
        kgid_t          egid;           /* effective GID of the task */
        kuid_t          fsuid;          /* UID for VFS ops */
        kgid_t          fsgid;          /* GID for VFS ops */
......
        kernel_cap_t    cap_inheritable; /* caps our children can inherit */
        kernel_cap_t    cap_permitted;  /* caps we're permitted */
        kernel_cap_t    cap_effective;  /* caps we can actually use */
        kernel_cap_t    cap_bset;       /* capability bounding set */
        kernel_cap_t    cap_ambient;    /* Ambient capability set */
......
} __randomize_layout;
```

大部分是关于用户和用户所属的用户组信息。

第一个是 uid 和 gid，注释是 real user/group id。一般情况下，谁启动的进程，就是谁的 ID。但是权限审核的时候，往往不比较这两个，也就是说不大起作用。

第二个是 euid 和 egid，注释是 effective user/group id。一看这个名字，就知道这个是起“作用”的。当这个进程要操作消息队列、共享内存、信号量等对象的时候，其实就是在比较这个用户和组是否有权限。

第三个是 fsuid 和 fsgid，也就是 filesystem user/group id。这个是对文件操作会审核的权限。一般说来，fsuid、euid，和 uid 是一样的，fsgid、egid，和 gid 也是一样的。因为谁启动的进程，就应该审核启动的用户到底有没有这个权限。但是也有特殊的情况。

**权限篇幅太大，这里不详述。很重要！！！要补**

##### 8、内存管理

每个进程都有自己独立的虚拟内存空间，这需要有一个数据结构来表示，就是 mm_struct。

```c
struct mm_struct                *mm;
struct mm_struct                *active_mm;
```

##### 9、文件与文件系统

每个进程有一个文件系统的数据结构，还有一个打开文件的数据结构。

```c
/* Filesystem information: */
struct fs_struct                *fs;
/* Open file information: */
struct files_struct             *files;
```

##### 10、内核栈

```c
struct thread_info    thread_info;
void  *stack;
```

在程序执行过程中，一旦调用到系统调用，就需要进入内核继续执行。那如何将用户态的执行和内核态的执行串起来呢？

**用户态函数栈**

32 位操作系统的情况。在 CPU 里，ESP（Extended Stack Pointer）是栈顶指针寄存器，入栈操作 Push 和出栈操作 Pop 指令，会自动调整 ESP 的值。另外有一个寄存器 EBP（Extended Base Pointer），是栈基地址指针寄存器，指向当前栈帧的最底部。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/aec865abccf0308155f4138cc905972e.jpg" alt="aec865abccf0308155f4138cc905972e" style="zoom:33%;" />

64 位操作系统，模式多少有些不一样。因为 64 位操作系统的寄存器数目比较多。rax 用于保存函数调用的返回结果。栈顶指针寄存器变成了 rsp，指向栈顶位置。堆栈的 Pop 和 Push 操作会自动调整 rsp，栈基指针寄存器变成了 rbp，指向当前栈帧的起始位置。  

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/770b0036a8b2695463cd95869f5adec0.jpg" alt="770b0036a8b2695463cd95869f5adec0" style="zoom:33%;" />

改变比较多的是参数传递。rdi、rsi、rdx、rcx、r8、r9 这 6 个寄存器，用于传递存储函数调用时的 6 个参数。如果超过 6 的时候，还是需要放到栈里面。



**内核态函数栈：**

通过系统调用，从进程的内存空间到内核中了。内核中也有各种各样的函数调用来调用去的，也需要这样一个机制，这该怎么办呢？

**上面的成员变量 stack，也就是内核栈，就派上了用场。**

内核栈是一个非常特殊的结构，如下图所示：

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/31d15bcd2a053235b5590977d12ffa2d.jpeg" alt="31d15bcd2a053235b5590977d12ffa2d" style="zoom:33%;" />

这段空间的最低位置，是一个 thread_info 结构。这个结构是对 task_struct 结构的补充。因为 task_struct 结构庞大但是通用，不同的体系结构就需要保存不同的东西，所以往往与体系结构有关的，都放在 thread_info 里面。

**通过 task_struct 找内核栈**

**通过内核栈找 task_struct**



在内核态，32 位和 64 位都使用内核栈，格式也稍有不同，主要集中在 pt_regs 结构上。

在内核态，32 位和 64 位的内核栈和 task_struct 的关联关系不同。32 位主要靠 thread_info，64 位主要靠 Per-CPU 变量。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/82ba663aad4f6bd946d48424196e515c.jpeg" alt="82ba663aad4f6bd946d48424196e515c" style="zoom:33%;" />

**内核栈比较重要，篇幅较大，后面补上**

