#### 内存管理简介和分页机制

##### 一、内存管理

###### 1、一个内存管理系统至少应该做三件事情

- 第一，虚拟内存空间的管理，每个进程看到的是独立的、互不干扰的虚拟地址空间；

- 第二，物理内存的管理，物理内存地址只有内存管理模块能够使用；

- 第三，内存映射，需要将虚拟内存和物理内存映射、关联起来。课堂练习

虚拟内存一分为二，一部分用来放内核的东西，称为 **内核空间**，一部分用来放进程的东西，称为 **用户空间**。

普通进程的视角，觉着整个空间是它独占的，没有其他进程存在。当然另一个进程也这样认为，因为它们互相看不到对方。但是到了内核里面，无论是从哪个进程进来的，看到的都是同一个内核空间，看到的都是同一个进程列表。虽然内核栈是各用各的，但是如果想知道的话，还是能够知道每个进程的内核栈在哪里的。所以，如果要访问一些公共的数据结构，需要进行锁保护。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/4ed91c744220d8b4298237d2ab2eda9d.jpeg" alt="img" style="zoom:67%;" />



###### 2、内存使用的方式

```c
#include <stdio.h>
#include <stdlib.h>

int max_length = 128;

char * generate(int length){
  int i;
  char * buffer = (char*) malloc (length+1);
  if (buffer == NULL)
    return NULL;
  for (i=0; i<length; i++){
    buffer[i]=rand()%26+'a';
  }
  buffer[length]='\0';
  return buffer;
}

int main(int argc, char *argv[])
{
  int num;
  char * buffer;

  printf ("Input the string length : ");
  scanf ("%d", &num);

  if(num > max_length){
    num = max_length;
  }

  buffer = generate(num);

  printf ("Random string is: %s\n",buffer);
  free (buffer);

  return 0;
}
```

程序在使用内存时的几种方式：

- 代码需要放在内存里面；
- 全局变量，例如 max_length；
- 常量字符串"Input the string length : "；
- 函数栈，例如局部变量 num 是作为参数传给 generate 函数的，这里面涉及了函数调用，局部变量，函数参数等都是保存在函数栈上面的；
- 堆，malloc 分配的内存在堆里面；
- 这里面涉及对 glibc 的调用，所以 glibc 的代码是以 so 文件的形式存在的，也需要放在内存里面。

malloc 会调用系统调用，进入内核，所以这个程序一旦运行起来，内核部分还需要分配内存：

- 内核的代码要在内存里面；
- 内核中也有全局变量；
- 每个进程都要有一个 task_struct；
- 每个进程还有一个内核栈；
- 在内核里面也有动态分配的内存；
- 虚拟地址到物理地址的映射表放在哪里？

从最低位开始排起，先是  **Text Segment、Data Segment 和 BSS Segment**。**Text Segment** 是存放二进制可执行代码的位置，**Data Segment** 数据段存放的是初始化后的全局变量和静态变量，**BSS Segment** 通常是指用来存放程序中未初始化的全局变量和静态变量的一块内存区域（特点是:可读写的，在程序执行之前BSS段会自动清0。所以，未初始的全局变量在程序执行之前已经成0了）。在二进制执行文件里面，就有这三个部分。这里就是把二进制执行文件的三个部分加载到内存里面。

接下来是 **堆（Heap）段**。堆是往高地址增长的，是用来动态分配内存的区域，malloc 就是在这里面分配的。

接下来的区域是 **Memory Mapping Segment**。这块地址可以用来把文件映射进内存用的，如果二进制的执行文件依赖于某个动态链接库，就是在这个区域里面将 so 文件映射到了内存中。

再下面就是 **栈（Stack）**地址段。主线程的函数调用的函数栈就是用这里的。

##### 二、内存分页机制

对于物理内存，操作系统把它分成一块一块大小相同的页，这样更方便管理，例如有的内存页面长时间不用了，可以暂时写到硬盘上，称为 **换出**。一旦需要的时候，再加载进来，叫做 **换入**。这样可以扩大可用物理内存的大小，提高物理内存的利用率。

这个换入和换出都是以页为单位的。页面的大小一般为 4KB。为了能够定位和访问每个页，需要有个页表，保存每个页的起始地址，再加上在页内的偏移量，组成线性地址，就能对于内存中的每个位置进行访问了。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/abbcafe962d93fac976aa26b7fcb7440.jpg" alt="abbcafe962d93fac976aa26b7fcb7440" style="zoom: 33%;" />

虚拟地址分为两部分，**页号** 和 **页内偏移**。页号作为页表的索引，页表包含物理页每页所在物理内存的基地址。这个基地址与页内偏移的组合就形成了物理内存地址。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/8495dfcbaed235f7500c7e11149b2feb.jpg" alt="img" style="zoom: 33%;" />

（注意图中的物理页并不是按顺序排列的）

32 位环境下，虚拟地址空间共 4GB。如果分成 4KB 一个页，那就是 1M 个页。每个 **页表项** 需要 4 个字节来存储，那么整个 4GB 空间的映射就需要 4MB 的内存来存储映射表。如果每个进程都有自己的映射表，100 个进程就需要 400MB 的内存。对于内核来讲，有点大了 。

而且，页表中所有页表项必须提前建好，并且要求是连续的。如果不连续，就没有办法通过虚拟地址里面的页号找到对应的页表项了。 

（1）我们可以试着将页表再分页，4G 的空间需要 4M 的页表来存储映射。我们把这 4M 分成 1K（1024）个 4K，每个 4K 又能放在一页里面，这样 1K 个 4K 就是 1K 个页，这 1K 个页也需要一个表进行管理，我们称为 **页目录表**，这个页目录表里面有 1K 项，每项 4 个字节，页目录表大小也是 4K。

（2）页目录有 1K 项，用 10 位就可以表示访问页目录的哪一项。这一项其实对应的是一整页的页表项，也即 4K 的页表项。每个页表项也是 4 个字节，因而一整页的页表项是 1K 个。再用 10 位就可以表示访问页表项的哪一项，页表项中的一项对应的就是一个页，是存放数据的页，这个页的大小是 4K，用 12 位可以定位这个页内的任何一个位置。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/b6960eb0a7eea008d33f8e0c4facc8b8.jpg" alt="b6960eb0a7eea008d33f8e0c4facc8b8" style="zoom:33%;" />

假设只给这个进程分配了一个数据页。如果只使用页表，也需要完整的 1M 个页表项共 4M 的内存，但是如果使用了页目录，页目录需要 1K 个全部分配，占用内存 4K，但是里面只有一项使用了。到了页表项，只需要分配能够管理那个数据页的页表项页就可以了，也就是说，最多 4K，这样内存就节省多了。

对于 64 位的系统，两级肯定不够了，就变成了四级目录，分别是 **全局页目录项 PGD（Page Global Directory）**、**上层页目录项 PUD（Page Upper Directory）**、**中间页目录项 PMD（Page Middle Directory）**和 **页表项 PTE（Page Table Entry）**。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/42eff3e7574ac8ce2501210e25cd2c0b.jpg" alt="42eff3e7574ac8ce2501210e25cd2c0b" style="zoom:33%;" />