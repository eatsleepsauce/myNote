#### 进程创建和线程创建

##### 一、进程创建

fork 创建进程，它包含两个重要的事件：

一个是将 task_struct 结构复制一份并且初始化。

另一个是试图唤醒新创建的子进程。

![img](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/9d9c5779436da40cabf8e8599eb85558.jpeg)

##### 二、线程创建

创建一个线程调用的是 pthread_create，线程不是一个完全由内核实现的机制，它是由内核态和用户态合作完成的。pthread_create 不是一个系统调用，是 Glibc 库的一个函数。

创建进程的话，调用的系统调用是 fork，在 copy_process 函数里面，会将五大结构 files_struct、fs_struct、sighand_struct、signal_struct、mm_struct 都复制一遍，从此父进程和子进程各用各的数据结构。而创建线程的话，调用的是系统调用 clone，在 copy_process 函数里面， 五大结构仅仅是引用计数加一，也即线程共享进程的数据结构。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/14635b1613d04df9f217c3508ae8524b.jpeg" alt="img" style="zoom:67%;" />