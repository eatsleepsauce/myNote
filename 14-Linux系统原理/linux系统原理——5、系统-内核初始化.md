#### 内核初始化

内核的启动从入口函数 start_kernel() 开始。在init/main.c文件中，start_kernel相当于内核的main函数。这个函数里面是各种各样的初始化函数 xxx_init。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145730.jpg" alt="内核初始化.jpeg" style="zoom:67%;" />

###### **1、INIT_TASK(init_task)**

在操作系统里面，先要有个创始进程，有一行指令 set_task_stack_end_magic(&init_task)。参数init_task，它的定义是struct task_struct init_task = INIT_TASK(init_task)。它是系统创建的第一个进程，称为 **0号进程**，这是**唯一一个没有通过fork或则kernel_thread产生的进程**，是进程列表的第一个。

###### **2、trap_init()**

trap_init() 里面设置了很多中断门(Interrupt Gate)，用于处理各种中断，其中有一个 set_system_intr_gate(IA32_SYSCALL_VECTOR, entry_INT80_32) 这是系统调用的中断门。系统调用也是通过中断的方式进行的（64位的有另外的系统调用方法）。

###### **3、mm_init()** 

mm_init()就是用来初始化内存管理模块的。

###### **4、sched_init()**

sched_init()就是用于初始化调度模块的。

###### **5、vfs_caches_init()**

vfs_caches_init()会用来初始化基于内存的文件系统rootfs。在这个函数里面会调用mn_init() 》init_rootfs()。这里面有一行代码，register_filesystem(&rootfs_fs_type)。在VFS虚拟文件系统里面注册了一种类型，定义为 struct file_system_type roots_fs_type。

为了兼容各种各样的文件系统，我们需要将文件的相关数据结构和操作抽象出来，形成一个抽象层对上提供统一接口，这个抽象层就是VFS（Virtual File System）虚拟文件系统。

###### **6、reset_init()**

（**1）初始化1号进程——用户态总管**

1号进程用 kernel_thread(kernel_init, NULL,CLONE_FS)创建，1号进程对操作系统来讲，具有划时代的意义，一旦有了用户进程就可以做一定的区分，哪些是核心资源，哪些是非核心资源。x86提供了分层的权限机制，把区域分成了4个Ring，越往外权限越低。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145731.jpg" alt="Ring0-4.jpg" style="zoom:67%;" />

操作系统很好地利用了这个机制，将能够访问关键资源的代码放在Ring0，我们称为 **内核态(Kernel Model)** ；将普通的程序代码放在Ring3，我们称为 **用户态(User Model)**。

当一个用户态的程序运行到一半，要访问一个核心资源，例如访问网卡发送一个网络包，就需要暂停当前的运行，调用系统调用，接下来就轮到内核中的代码运行了。系统调用结束后，返回用户态，让暂停运行的程序接着运行。暂停其实就是把当时CPU的寄存器的值都暂存到一个地方，恢复就是恢复CPU寄存器的值，这样就能接着运行了。

**调用过程：用户态 》系统调用 》保存寄存器 》内核态执行系统调用 》恢复寄存器 》返回用户态继续执行**

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145732.jpg" alt="用户态-内核态-用户态.jpg" style="zoom: 67%;" />

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145733.jpg" alt="CPU用户态-内核态.jpeg"  />

1号进程和普通用户进程不太一样，它是所有普通用户进程的祖宗，刚开始时并没有用户态，所以是从 **内核态到用户态**。它是怎么从内核态回到用户态的？

**（2）初始化2号进程——内核态总管**

2号进程通过 kernel_thread(kthreadadd,NULL,CLONE_FS|CLONE_FILES)创建，是所有内核态进程的祖宗。