#### 虚拟内存管理

对于 32 位系统，最大能够寻址 2^32=4G，其中用户态虚拟地址空间是 3G，内核态是 1G。

对于 64 位系统，虚拟地址只使用了 48 位。就像代码里面写的一样，1 左移了 47 位，就相当于 48 位地址空间一半的位置，0x0000800000000000，然后减去一个页，就是 0x00007FFFFFFFF000，共 128T。同样，内核空间也是 128T。内核空间和用户空间之间隔着很大的空隙，以此来进行隔离。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/89723dc967b59f6f49419082f6ab7659.jpg" alt="89723dc967b59f6f49419082f6ab7659" style="zoom: 25%;" />

##### 一、用户态虚拟内存

###### 1、struct mm_struct 

用户态虚拟空间里面有几类数据，例如代码、全局变量、堆、栈、内存映射区等。在 **struct mm_struct** 里面，有下面这些变量定义了这些区域的统计信息和位置。

```c
unsigned long mmap_base;  /* base of mmap area */
unsigned long total_vm;    /* Total pages mapped */
unsigned long locked_vm;  /* Pages that have PG_mlocked set */
unsigned long pinned_vm;  /* Refcount permanently increased */
unsigned long data_vm;    /* VM_WRITE & ~VM_SHARED & ~VM_STACK */
unsigned long exec_vm;    /* VM_EXEC & ~VM_WRITE & ~VM_STACK */
unsigned long stack_vm;    /* VM_STACK */
unsigned long start_code, end_code, start_data, end_data;
unsigned long start_brk, brk, start_stack;
unsigned long arg_start, arg_end, env_start, env_end;
```

**total_vm** 是总共映射的页的数目。我们知道，这么大的虚拟地址空间，不可能都有真实内存对应，所以这里是映射的数目。当内存吃紧的时候，有些页可以换出到硬盘上，有的页因为比较重要，不能换出。**locked_vm** 就是被锁定不能换出，**pinned_vm** 是不能换出，也不能移动。

**data_vm** 是存放数据的页的数目，**exec_vm** 是存放可执行文件的页的数目，**stack_vm** 是栈所占的页的数目。

**start_code** 和 **end_code** 表示可执行代码的开始和结束位置，**start_data** 和 **end_data** 表示已初始化数据的开始位置和结束位置。

**start_brk** 是堆的起始位置，**brk** 是堆当前的结束位置。**malloc 申请一小块内存的** 话，就是通过改变 brk 位置实现的。

**arg_start** 和 **arg_end** 是参数列表的位置， **env_start** 和 **env_end** 是环境变量的位置。它们都位于栈中最高地址的地方。

**mmap_base** 表示虚拟地址空间中用于内存映射的起始地址。一般情况下，这个空间是从高地址到低地址增长的。 **malloc 申请一大块内存的时候**，就是通过 mmap 在这里映射一块区域到物理内存。**加载动态链接库 so 文件，也是在这个区域里面**，映射一块区域到 so 文件。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/f83b8d49b4e74c0e255b5735044c1eb1.jpg" alt="f83b8d49b4e74c0e255b5735044c1eb1" style="zoom:25%;" />

###### 2、vm_area_struct

位置信息之外，**struct mm_struct** 里面还专门有一个结构 **vm_area_struct**，来描述这些区域的属性。

```c
struct vm_area_struct *mmap;    /* list of VMAs */
struct rb_root mm_rb;
```

这里面 **mmp**  一个是单链表，用于将这些区域串起来。另外还有一个红黑树。它的好处就是查找和修改都很快。这里用红黑树，就是为了快速查找一个内存区域，并在需要改变的时候，能够快速修改。

```c
struct vm_area_struct {
  /* The first cache line has the info for VMA tree walking. */
  unsigned long vm_start;    /* Our start address within vm_mm. */
  unsigned long vm_end;    /* The first byte after our end address within vm_mm. */
  /* linked list of VM areas per task, sorted by address */
  struct vm_area_struct *vm_next, *vm_prev;
  struct rb_node vm_rb;
  struct mm_struct *vm_mm;  /* The address space we belong to. */
  struct list_head anon_vma_chain; /* Serialized by mmap_sem &
            * page_table_lock */
  struct anon_vma *anon_vma;  /* Serialized by page_table_lock */
  /* Function pointers to deal with this struct. */
  const struct vm_operations_struct *vm_ops;
  struct file * vm_file;    /* File we map to (can be NULL). */
  void * vm_private_data;    /* was vm_pte (shared mem) */
} __randomize_layout;
```

**vm_start** 和 **vm_end** 指定了该区域在用户空间中的起始和结束地址。**vm_next** 和 **vm_prev** 将这个区域串在链表上。vm_rb 将这个区域放在红黑树上。vm_ops 里面是对这个内存区域可以做的操作的定义。

**虚拟内存区域可以映射到物理内存，也可以映射到文件，映射到物理内存的时候称为匿名映射，anon_vma 中，anoy 就是 anonymous，匿名的意思。映射到文件就需要有 vm_file 指定被映射的文件。**

###### 3、vm_area_struct 是如何和各个区域关联的

这个事情是在 load_elf_binary 里面实现的。加载内核的是它，启动第一个用户态进程 init 的是它，fork 完了以后，调用 exec 运行一个二进制程序的也是它。当 exec 运行一个二进制程序的时候，除了解析 ELF 的格式之外，另外一个重要的事情就是建立内存映射。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/7af58012466c7d006511a7e16143314c.jpeg" alt="7af58012466c7d006511a7e16143314c" style="zoom: 25%;" />

映射完毕后，什么情况下会修改呢？

第一种情况是函数的调用，涉及函数栈的改变，主要是改变栈顶指针。

第二种情况是通过 malloc 申请一个堆内的空间，当然底层要么执行 brk，要么执行 mmap。



##### 二、内核态虚拟内存

内核态的虚拟空间和某一个进程没有关系，所有进程通过系统调用进入到内核之后，看到的虚拟地址空间都是一样的。

###### 1、32位内核态

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/83a6511faf802014fbc2c02afc397a04.jpg" alt="83a6511faf802014fbc2c02afc397a04" style="zoom:25%;" />

32 位的内核态虚拟地址空间一共就 1G，占绝大部分的前 896M，我们称为 **直接映射区**。所谓的直接映射区，就是这一块空间是连续的，和物理内存是非常简单的映射关系，其实就是虚拟内存地址减去 3G，就得到物理内存的位置。

（1）**896M 分解**

在系统启动的时候，物理内存的前 1M 已经被占用了，从 1M 开始加载内核代码段，然后就是内核的全局变量、BSS 等，也是 ELF 里面涵盖的。这样内核的代码段，全局变量，BSS 也就会被映射到 3G 后的虚拟地址空间里面。具体的物理内存布局可以查看 /proc/iomem。

在内核运行的过程中，如果碰到系统调用创建进程，会创建 task_struct 这样的实例，内核的进程管理代码会将实例创建在 3G 至 3G+896M 的虚拟空间中，当然也会被放在物理内存里面的前 896M 里面，相应的页表也会被创建。

在内核运行的过程中，会涉及内核栈的分配，内核的进程管理的代码会将内核栈创建在 3G 至 3G+896M 的虚拟空间中，当然也就会被放在物理内存里面的前 896M 里面，相应的页表也会被创建。

**在内核中，除了内存管理模块直接操作物理地址之外，内核的其他模块，仍然要操作虚拟地址，而虚拟地址是需要内存管理模块分配和映射好的。**

（2）**剩余虚拟内存地址**

- 在 896M 到 VMALLOC_START 之间有 8M 的空间。
- VMALLOC_START 到 VMALLOC_END 之间称为内核动态映射空间，也即内核想像用户态进程一样 malloc 申请内存，在内核里面可以使用 vmalloc。假设物理内存里面，896M 到 1.5G 之间已经被用户态进程占用了，并且映射关系放在了进程的页表中，内核 vmalloc 的时候，只能从分配物理内存 1.5G 开始，就需要使用这一段的虚拟地址进行映射，映射关系放在专门给内核自己用的页表里面。
- PKMAP_BASE 到 FIXADDR_START 的空间称为持久内核映射。使用 alloc_pages() 函数的时候，在物理内存的高端内存得到 struct page 结构，可以调用 kmap 将其映射到这个区域。
- FIXADDR_START 到 FIXADDR_TOP(0xFFFF F000) 的空间，称为固定映射区域，主要用于满足特殊需求。
- 在最后一个区域可以通过 kmap_atomic 实现临时内核映射。假设用户态的进程要映射一个文件到内存中，先要映射用户态进程空间的一段虚拟地址到物理内存，然后将文件内容写入这个物理内存供用户态进程访问。给用户态进程分配物理内存页可以通过 alloc_pages()，分配完毕后，按说将用户态进程虚拟地址和物理内存的映射关系放在用户态进程的页表中，就完事大吉了。这个时候，用户态进程可以通过用户态的虚拟地址，也即 0 至 3G 的部分，经过页表映射后访问物理内存，并不需要内核态的虚拟地址里面也划出一块来，映射到这个物理内存页。但是如果要把文件内容写入物理内存，这件事情要内核来干了，这就只好通过 kmap_atomic 做一个临时映射，写入物理内存完毕后，再 kunmap_atomic 来解映射即可。

###### 2、64位内核态

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/7eaf620768c62ff53e5ea2b11b4940f6.jpg" alt="7eaf620768c62ff53e5ea2b11b4940f6" style="zoom:25%;" />

- 从 0xffff800000000000 开始就是内核的部分，只不过一开始有 8T 的空档区域。
- 从 PAGE_OFFSET_BASE(0xffff880000000000) 开始的 64T 的虚拟地址空间是直接映射区域，也就是减去 PAGE_OFFSET 就是物理地址。虚拟地址和物理地址之间的映射在大部分情况下还是会通过建立页表的方式进行映射。
- 从 VMALLOC_START（0xffffc90000000000）开始到 VMALLOC_END（0xffffe90000000000）的 32T 的空间是给 vmalloc 的。
- 从 VMEMMAP_START（0xffffea0000000000）开始的 1T 空间用于存放物理页面的描述结构 struct page 的。
- 从 START_KERNEL_map（0xffffffff80000000）开始的 512M 用于存放内核代码段、全局变量、BSS 等。这里对应到物理内存开始的位置，减去 __START_KERNEL_map 就能得到物理内存的地址。



32位对应关系图：

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/2861968d1907bc314b82c34c221aace8.jpeg" alt="2861968d1907bc314b82c34c221aace8" style="zoom:25%;" />

64位对应关系图：

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/2ad275ff8fdf6aafced4a7aeea4ca0ce.jpeg" alt="2ad275ff8fdf6aafced4a7aeea4ca0ce" style="zoom:25%;" />