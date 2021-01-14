#### 进程的数据结构

在 Linux 里面，无论是进程，还是线程，到了内核里面，统一都叫 **任务（Task），由一个统一的结构 task_struct 进行管理**。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/75c4d28a9d2daa4acc1107832be84e2d.jpeg" alt="img" style="zoom: 25%;" />