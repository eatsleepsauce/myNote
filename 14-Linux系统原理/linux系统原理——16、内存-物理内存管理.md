#### 物理内存管理

##### 一、物理内存模型

###### 1、平坦内存模型（Flat Memory Model）

我们可以从 0 开始对物理页编号，这样每个物理页都会有个页号。由于物理地址是连续的，页也是连续的，每个页大小也是一样的。因而对于任何一个地址，只要直接除一下每页的大小，很容易直接算出在哪一页。每个页有一个结构 struct page 表示，这个结构也是放在一个数组里面，这样根据页号，很容易通过下标找到相应的 struct page 结构。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/fa6c2b6166d02ac37637d7da4e4b579b.jpeg" alt="fa6c2b6166d02ac37637d7da4e4b579b" style="zoom:25%;" />

###### 2、SMP（Symmetric multiprocessing），即对称多处理器

在这种模式下，CPU 也会有多个，在总线的一侧。所有的内存条组成一大片内存，在总线的另一侧，所有的 CPU 访问内存都要过总线，而且距离都是一样的，这种模式称为 SMP（Symmetric multiprocessing），即对称多处理器。当然，它也有一个显著的缺点，就是总线会成为瓶颈，因为数据都要走它。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/8f158f58dda94ec04b26200073e15449.jpeg" alt="8f158f58dda94ec04b26200073e15449" style="zoom:25%;" />

###### 3、NUMA（Non-uniform memory access），非一致内存访问

在这种模式下，内存不是一整块。每个 CPU 都有自己的本地内存，CPU 访问本地内存不用过总线，因而速度要快很多，每个 CPU 和内存在一起，称为一个 NUMA 节点。但是，在本地内存不足的情况下，每个 CPU 都可以去另外的 NUMA 节点申请内存，这个时候访问延时就会比较长。

内存被分成了多个节点，每个节点再被分成一个一个的页面。由于页需要全局唯一定位，页还是需要有全局唯一的页号的。但是由于物理内存不是连起来的了，页号也就不再连续了。于是内存模型就变成了非连续内存模型，管理起来就复杂一些。



解析当前的主流场景，NUMA 方式：

（1） **NUMA 节点**，结构 **typedef struct pglist_data pg_data_t**，它里面有以下的成员变量：

- 每一个节点都有自己的 ID：node_id；
- node_mem_map 就是这个节点的 struct page 数组，用于描述这个节点里面的所有的页；
- node_start_pfn 是这个节点的起始页号；
- node_spanned_pages 是这个节点中包含不连续的物理内存地址的页面数；
- node_present_pages 是真正可用的物理页面的数目。



（2）**每一个节点分成一个个区域 zone**，表示区域的数据结构 zone ：

```c
struct zone {
......
  struct pglist_data  *zone_pgdat;
  struct per_cpu_pageset __percpu *pageset;

  unsigned long    zone_start_pfn;

  /*
   * spanned_pages is the total pages spanned by the zone, including
   * holes, which is calculated as:
   *   spanned_pages = zone_end_pfn - zone_start_pfn;
   *
   * present_pages is physical pages existing within the zone, which
   * is calculated as:
   *  present_pages = spanned_pages - absent_pages(pages in holes);
   *
   * managed_pages is present pages managed by the buddy system, which
   * is calculated as (reserved_pages includes pages allocated by the
   * bootmem allocator):
   *  managed_pages = present_pages - reserved_pages;
   *
   */
  unsigned long    managed_pages;
  unsigned long    spanned_pages;
  unsigned long    present_pages;

  const char    *name;
......
  /* free areas of different sizes */
  struct free_area  free_area[MAX_ORDER];

  /* zone flags, see below */
  unsigned long    flags;

  /* Primarily protects free_area */
  spinlock_t    lock;
......
} ____cacheline_internodealigned_in_
```

**zone_start_pfn** 表示属于这个 zone 的第一个页。

**spanned_pages = zone_end_pfn - zone_start_pfn**，也即 **spanned_pages** 指的是不管中间有没有物理内存空洞，反正就是最后的页号减去起始的页号。

**present_pages = spanned_pages - absent_pages(pages in holes)**，也即 **present_pages** 是这个 zone 在物理内存中真实存在的所有 page 数目。

**managed_pages = present_pages - reserved_pages**，也即 **managed_pages** 是这个 zone 被伙伴系统管理的所有的 page 数目。

**per_cpu_pageset** 用于区分冷热页。什么叫冷热页呢？如果一个页被加载到 CPU 高速缓存里面，这就是一个热页（Hot Page），CPU 读起来速度会快很多，如果没有就是冷页（Cold Page）。由于每个 CPU 都有自己的高速缓存，因而 per_cpu_pageset 也是每个 CPU 一个。



（3）**页**

组成物理内存的基本单位，页的数据结构 **struct page**。这是一个特别复杂的结构，里面有很多的 union，union 结构是在 C 语言中被用于同一块内存根据情况保存不同类型数据的一种方式。这里之所以用了 union，是因为一个物理页面使用模式有多种。

- 第一种模式，要用就用一整页。这一整页的内存，或者直接和虚拟地址空间建立映射关系，我们把这种称为 **匿名页（Anonymous Page）**。或者 用于关联一个文件，然后再和虚拟地址空间建立映射关系，这样的文件，我们称为 **内存映射文件（Memory-mapped File）**。

  如果某一页是这种使用模式，则会使用 union 中的以下变量：

  - struct address_space *mapping 就是用于内存映射，如果是匿名页，最低位为 1；如果是映射文件，最低位为 0；
  - pgoff_t index 是在映射区的偏移量；
  - atomic_t _mapcount，每个进程都有自己的页表，这里指有多少个页表项指向了这个页；
  - struct list_head lru 表示这一页应该在一个链表上，例如这个页面被换出，就在换出页的链表中；
  - compound 相关的变量用于复合页（Compound Page），就是将物理上连续的两个或多个页看成一个独立的大页。

- 第二种模式，仅需分配小块内存。有时候，我们不需要一下子分配这么多的内存，例如分配一个 task_struct 结构，只需要分配小块的内存，去存储这个进程描述结构的对象。为了满足对这种小内存块的需要，Linux 系统采用了一种被称为 slab allocator 的技术，用于分配称为 slab 的一小块内存。它的基本原理是从内存管理模块申请一整块页，然后划分成多个小块的存储池，用复杂的队列来维护这些小块的状态（状态包括：被分配了 / 被放回池子 / 应该被回收）。

  如果某一页是用于分割成一小块一小块的内存进行分配的使用模式，则会使用 union 中的以下变量：

  - s_mem 是已经分配了正在使用的 slab 的第一个对象；
  - freelist 是池子中的空闲对象；
  - rcu_head 是需要释放的列表。

##### 二、页的分配

###### 1、大内存分配

对于要分配比较大的内存，例如到分配页级别的，可以使用 **伙伴系统（Buddy System）**。 

Linux 中的内存管理的“页”大小为 4KB。把所有的空闲页分组为 11 个页块链表，每个块链表分别包含很多个大小的页块，有 1、2、4、8、16、32、64、128、256、512 和 1024 个连续页的页块。最大可以申请 1024 个连续页，对应 4MB 大小的连续内存。每个页块的第一个页的物理地址是该页块大小的整数倍。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/2738c0c98d2ed31cbbe1fdcba01142cf.jpeg" alt="2738c0c98d2ed31cbbe1fdcba01142cf" style="zoom:25%;" />

在 struct zone 里面有以下的定义：

```c
struct free_area free_area[MAX_ORDER];
```

```
#define MAX_ORDER 11
```

当向内核请求分配 (2^(i-1)，2^i]数目的页块时，按照 2^i 页块请求处理。如果对应的页块链表中没有空闲页块，那我们就在更大的页块链表中去找。当分配的页块中有多余的页时，伙伴系统会根据多余的页块大小插入到对应的空闲页块链表中。

例如，要请求一个 128 个页的页块时，先检查 128 个页的页块链表是否有空闲块。如果没有，则查 256 个页的页块链表；如果有空闲块的话，则将 256 个页的页块分成两份，一份使用，一份插入 128 个页的页块链表中。如果还是没有，就查 512 个页的页块链表；如果有的话，就分裂为 128、128、256 三个页块，一个 128 的使用，剩余两个插入对应页块链表。

每一个 zone，都有伙伴系统维护的各种大小的队列。

为了方便分配，空闲页放在 **struct free_area** 里面，使用 **伙伴系统（Buddy System）** 进行管理和分配，每一页用 struct page 表示。

![3fa8123990e5ae2c86859f70a8351f4f](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/3fa8123990e5ae2c86859f70a8351f4f.jpeg)

###### 2、小内存的分配

如果遇到小的对象，会使用 slub 分配器进行分配。

<img src="https://static001.geekbang.org/resource/image/52/54/527e5c861fd06c6eb61a761e4214ba54.jpeg" alt="527e5c861fd06c6eb61a761e4214ba54" style="zoom:25%;" />



##### 三、页面换出

每个进程都有自己的虚拟地址空间，无论是 32 位还是 64 位，虚拟地址空间都非常大，物理内存不可能有这么多的空间放得下。所以，一般情况下，页面只有在被使用的时候，才会放在物理内存中。如果过了一段时间不被使用，即便用户进程并没有释放它，物理内存管理也有责任做一定的干预。例如，将这些物理内存中的页面换出到硬盘上去；将空出的物理内存，交给活跃的进程去使用。

触发页面换出的场景：

- 分配内存的时候，发现没有地方了，就试图回收一下。

- 内存管理系统应该主动去做的，这就是 **内核线程 kswapd**。在系统初始化的时候就被创建。这样它会进入一个无限循环，直到系统停止。在这个循环中，如果内存使用没有那么紧张，那它就什么都不做；如果内存紧张了，就需要去检查一下内存，看看是否需要换出一些内存页。