#### 如何映射

原理性的东西了太多了，先记个大概后期能力够了补细节

##### 一、用户态映射

用户态的内存映射机制包含以下几个部分：

- 用户态内存映射函数 mmap，包括用它来做匿名映射和文件映射。
- 用户态的页表结构，存储位置在 mm_struct 中。
- 在用户态访问没有映射的内存会引发缺页异常，分配物理页表、补齐页表。如果是匿名映射则分配物理内存；如果是 swap，则将 swap 文件读入；如果是文件映射，则将文件读入。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/78d351d0105c8e5bf0e49c685a2c1a44.jpg" alt="78d351d0105c8e5bf0e49c685a2c1a44" style="zoom:33%;" />

##### 二、内核态映射

和用户态页表不同，在系统初始化的时候，我们就要创建内核页表了。

在用户态可以通过 malloc 函数分配内存，当然 malloc 在分配比较大的内存的时候，底层调用的是 mmap，当然也可以直接通过 mmap 做内存映射，在内核里面也有相应的函数。



内核态的后面再补～～～



##### 三、总结

对于内核态，kmalloc 在分配大内存的时候，以及 vmalloc 分配不连续物理页的时候，直接使用伙伴系统，分配后转换为虚拟地址，访问的时候需要通过内核页表进行映射。

对于 kmem_cache 以及 kmalloc 分配小内存，则使用 slub 分配器，将伙伴系统分配出来的大块内存切成一小块一小块进行分配。

kmem_cache 和 kmalloc 的部分不会被换出，因为用这两个函数分配的内存多用于保持内核关键的数据结构。内核态中 vmalloc 分配的部分会被换出，因而当访问的时候，发现不在，就会调用 do_page_fault。

对于用户态的内存分配，或者直接调用 mmap 系统调用分配，或者调用 malloc。调用 malloc 的时候，如果分配小的内存，就用 sys_brk 系统调用；如果分配大的内存，还是用 sys_mmap 系统调用。正常情况下，用户态的内存都是可以换出的，因而一旦发现内存中不存在，就会调用 do_page_fault。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/274e22b3f5196a4c68bb6813fb643f9a.png" alt="274e22b3f5196a4c68bb6813fb643f9a" style="zoom:33%;" />

