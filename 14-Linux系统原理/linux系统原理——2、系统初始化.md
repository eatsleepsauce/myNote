#### 系统初始化

##### 一、x86架构

###### **1、计算机逻辑图**

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145719.jpg" alt="计算机逻辑图.jpeg" style="zoom: 50%;" />

CPU（Central Processing Unit，中央处理器）不是单纯的一块，它包括三个部分，运算单元、数据单元和控制单元。

**运算单元 **只管算，例如做加法、做位移等等。但是，它不知道应该算哪些数据，运算结果应该放在哪里。

**数据单元** 包括 CPU 内部的缓存和寄存器组，空间很小，但是速度飞快，可以暂时存放数据和运算结果（运算单元计算的数据如果每次都要经过总线，到内存里面现拿，这样就太慢了，所以就有了数据单元）。

**控制单元** 是一个统一的指挥中心，它可以获得下一条指令，然后执行这条指令。这个指令会指导运算单元取出数据单元中的某几个数据，计算出个结果，然后放在数据单元的某个地方（有了放数据的地方，也有了算的地方，还需要有个指挥到底做什么运算的地方，这就是控制单元。）。

###### **2、CPU和内存如何配合**

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145721.jpg" alt="CPU与内存.jpeg" style="zoom:67%;" />

**程序运行的过程中要操作的数据和产生的计算结果，都会放在数据段里面**。CPU怎么执行这些程序，操作这些数据，产生一些结果，并写回内存？

（1）CPU的控制单元里面由一个 **指令指针寄存器**，里面存放的是下一条指令在内存中的地址。

（2）控制单元会不停地将代码段的指令拿进来，存放到 **指令寄存器** 中。

（3）指令分为两部分，一部分是做什么操作，例如加法还是位移；一部分是操作哪些数据。指令执行的时候，就要把第一部分交给 **运算单元**，第二部分交给 **数据单元**。

（4）数据单元根据数据地址，将数据从数据段里读到 **数据寄存器** 里面，就可以参与运算了，运算单元做完运算，产生的结果会暂存在数据单元的数据寄存器里，最终会有指令将数据写回到内存中。

CPU和内存来来回回传数据，靠的是 **总线**。总线主要由两类，一个是 **地址总线**，另一个是 **数据总线**，地址总线的位数决定了能访问的地址范围（如：地址总线是两位，则CPU只能访问00 01 10 11四个位置）。数据总线的位数决定了一次能拿多少个数据进来。

###### **3、8086原理**

很多CPU都是从英特尔8086这款型号CPU开端的，所以统称为x86架构，现在CPU的数据总线和地址总线越来越宽，处理能力越来越强。8086 处理器虽然已经很老了，但是现在操作系统中的很多特性都和它有关，并且一直保持兼容。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145722.jpg" alt="型号和总线.jpg" style="zoom:67%;" />



<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145723.jpg" alt="8086CPU.jpeg" style="zoom:67%;" />

**数据单元：**

8086处理器内部有8个16位通用寄存器，也就是CPU的数据单元，分别是AX、BX、CX、DX、SP、BP、SI、DI。这些寄存器主要用于在计算过程中暂存数据。其中AX、BX、CX、DX可以分成两个8位的寄存器来使用，分别是AH、AL、BH、BL、CH、CL、DH、DL，H就是high（高位），L就是low（低位）。

**控制单元：**

**IP寄存器就是指令指针寄存器**（Instruction Pointer Register），指向代码段中下一条指令的位置。CPU会根据它来不断地将指令从内存的代码段中加载到CPU到指令队列中，然后交给运算单元去执行。

每个进程都分代码段和数据段，为了指向不同进程的地址空间，有四个16位的段寄存器，分别是CS、DS、SS、ES。

CS就是**代码段寄存器**（Code Segment Register），通过它可以找到代码在内存中的位置；DS是**数据段的寄存器**，通过它可以找到数据在内存中的位置。

在CS和DS中都存放着一个段的起始地址。代码段的偏移量在IP寄存器中，数据段的偏移量会放在通用寄存器中。

8086CPU中CS和DS都是16位的，IP寄存器和通用寄存器都是16位的，但是8086的地址总线是20位的。为了凑够20位，就是使用 **起始地址*16 + 偏移量**，也就是把CS和DS中的值左移4位变成20位的，加上16位的偏移量，这样最终得到20位的地址。

由于地址总线只有20位，所以只能访问1M的空间地址。而一个段的大小因为偏移量是16位的，所以一个段的最大的大小是2^16=64k。

###### **4、32位处理器**

在32位处理器中，有32根地址总线，可以访问4G的内存，使用原来的模式不行了，但是又不能完全抛弃原来的模式，因为这个架构是开放的，那么如何保持兼容呢？

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145724.jpg" alt="32位寄存器扩展.jpeg" style="zoom:67%;" />

（1）通用寄存器16位扩展到32位，依然保留16位和8位的使用方式。（高16位不能分成两个8位使用的原因是因为程序没有基于高位开发的，所以不兼容）

（2）指向下一条指令的指令指针寄存器IP，扩展成32位的，同样也兼容16位的。

（3）改动比较大的是段寄存器（Segment Register），因为原来8086CPU没有使用16位做一个段的起始地址，而是使用了一个不上不下的20位地址。这样每次都要左移四位，也就意味着段的起始地址不能是任何一个地方，只能是整除16的地方。**所以32位CPU重新定义了 CS、SS、DS、ES，它们任然是16位的，但是不再是段的起始地址。段的起始地址放在内存的某个地方。这个地方是一个表格，表格中的一项项段描述符（Segment Descriptor）才是真正的段起始地址。而段寄存器里面保存的是在这个表格中的哪一项，称为 选择子（Selector）**。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145725.jpg" alt="32位CPU变化.jpeg" style="zoom:67%;" />

这样将从段寄存器直接拿到段起始地址，就变成了先间接地从段寄存器找到表格中的一项，再从表格中的一项中拿到段起始地址。这种获取段起始地址段方式就会很灵活了，当然为了快速拿到段的起始地址，段描述符会从内存中拿到CPU的描述符高速缓存器中。

在32位的系统架构下，将之前的模式称为 **实模式（Real Pattern）**，后一种模式称为 **保护模式（Protected Pattern）**。系统刚启动的时候，CPU是处于实模式的，这个时候和原来是兼容的，当需要更多内存的时候，可以遵循一定的规则进行一系列的操作，然后切换到保护模式，这样就能够用到32位CPU更强大的能力。

##### 二、实模式到保护模式、从BIOS到bootloader

###### **1、BIOS时期**

计算机通电启动后CPU就可以执行指令了，这个时候没有操作系统，内存也是空的，CPU只能从 **ROM（Read Only Memory，只读存储器）**中读取早就固化了的一些初始化程序，也就是 **BIOS（Basic Input and Output System，基本输入输出系统）**。

刚启动的时候处于实模式，在x86系统中，将1M空间**最上面的 0xF0000 到 0xFFFFF 这 64k 映射给了ROM** （约定的），也就说访问这个部分地址的时候，会访问ROM。

（1）刚加电的时候会将CS设置为0xFFFF，将IP设置为0x0000，所以第一条指令指向了0xFFFF0，正是在ROM范围内。在这里会有一个JMP指令跳转到ROM做初始化，也就是BIOS开始进行初始化工作，BIOS第一步主要是检查系统硬件是不是都好着。

（2）为了基本输入输出设备能够正常使用，需要建立中断向量表和中断服务程序。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145726.jpg" alt="地址空间组成.jpeg" style="zoom: 50%;" />

###### **2、bootloader 时期**

BIOS做完自己的事情后，在BIOS的界面上，会看到一个启动盘的选项。它一般在第一个扇区，占512字节，而且以0xAA55结束（约定的），满足这个条件的时候就说明这是一个启动盘，在512字节内会启动相关代码。

启动盘第一个扇区的代码是谁放的？Linux中由个工具叫 **Grub2**，全称 **Grand Unifiled Bootloader Version2**，就是搞系统启动的。

（1）可以通过 grub2 -mkconfig -o /boot /grub2/grub .cfg 来配置系统启动选项，这里的选项会在系统启动的时候形成一个列表，供我们选择从哪个系统启动。

```

menuentry 'CentOS Linux (3.10.0-862.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-862.el7.x86_64-advanced-b1aceb95-6b9e-464a-a589-bed66220ebee' {
  load_video
  set gfxpayload=keep
  insmod gzio
  insmod part_msdos
  insmod ext2
  set root='hd0,msdos1'
  if [ x$feature_platform_search_hint = xy ]; then
    search --no-floppy --fs-uuid --set=root --hint='hd0,msdos1'  b1aceb95-6b9e-464a-a589-bed66220ebee
  else
    search --no-floppy --fs-uuid --set=root b1aceb95-6b9e-464a-a589-bed66220ebee
  fi
  linux16 /boot/vmlinuz-3.10.0-862.el7.x86_64 root=UUID=b1aceb95-6b9e-464a-a589-bed66220ebee ro console=tty0 console=ttyS0,115200 crashkernel=auto net.ifnames=0 biosdevname=0 rhgb quiet 
  initrd16 /boot/initramfs-3.10.0-862.el7.x86_64.img
}
```

![启动项.png](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145725.png)



（2）使用grub2 -install /dev/sda，可以将启动程序安装到相应的位置，grub2第一个要安装是boot.img。它由boot.S编译而成，一共512字节，正式安装到启动盘的第一个扇区。这个扇区通常成为 **MBR（Master Boot Record，主引导记录/扇区）**。

（3）BIOS完成任务后，会将 **boot.img** 从硬盘第一个扇区加载到内存中的 0x7c00来运行，由于只有512字节，boot.img做不了太多事情，它做的最重要的一个事情就是加载grub2的另一个镜像 core.img。

（4）**core.img** 由**diskboot.img、lzma_decompress.img、kernel.img和一系列的模块**组成，功能比较丰富，能做很多事情。core.img的第一个扇区里面是diskboot.img（从硬盘启动），对应的代码就是 diskboot.S，boot.img将控制权交给diskboot.img后，diskboot.img的任务就是将core.img的其它部分加载进来。

（5）diskboot.img 先加载 **解压缩程序lzma_decompress.img**，再往下就是 **kernel.img (grub内核非linux内核)** ，最后是各个模块对应的镜像。（kernel.img是压缩过的，所以需要 lzma_decompress.img，lzma_decompress.img对应的代码时 startup_raw.S）

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145728.jpg" alt="加载顺序.jpeg" style="zoom: 67%;" />

**在真正的解压缩之前，lzma_decompress.img 调用 real_to_prot ，切换到保护模式**。

###### **3、实模式切换到保护模式**

切换到保护模式的大部分工作都与内存访问有关：

（1）第一项，**启动分段**，就是在内存里面建立段描述符表，将寄存器里面的段寄存器变成段选择子，指向某个段描述符，这样就能实现不同进程的切换了。

（2）第二项，**启动分页**，能够管理的内存变大了，就需要将内存分成相等大小的块。

（3）**打开 Gate 20**，也就是第21根地址线的控制线，切换保护模式的函数 DATA32 call real_to_prot 会打开Gate A20。

（4）有了空间后，对压缩的 **kernel.img 进行解压**，然后跳转到kernel.img运行，kernel.img对应的代码时startup.S以及一堆c文件，在startup.S中会 **调用grub_main**，这是grub kernel的主函数。

（5）grub_main 这个主函数会 **调用 grub_load_config() 解析 grub.cfg** 文件(上面的代码)的配置信息，如果正常grub_main 最后会调用 grub_command_execute("normal",0,0) 》grub_normal_execute()函数 》grub_show_menu()函数 会显示出让选择哪个操作系统的列表。

（6）**grub_menu_execute_entry() 开始解析并执行之前的选择**，linux16 命令表示装载指定的内核文件，并传递内核启动参数。于是 grub_cmd_linux() 函数会被调用——读取linux内核镜像头部的一些数据结构，放到内存中的数据结构，进行检查，如果通过则会读取整个linux内核镜像到内存。

（7）如果grub配置文件中还有initrd命令——用于为即将启动的内核传递 init ramdisk路径。grub_cmd_initrd()函数会被调用，将initramfs加载到内存中来。

（8）上面事情都做完后，**grub_command_execute(“boot”,0,0) 才开始真正地启动内核**。

##### 三、内核初始化

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

调用过程：用户态 》系统调用 》保存寄存器 》内核态执行系统调用 》恢复寄存器 》返回用户态继续执行

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145732.jpg" alt="用户态-内核态-用户态.jpg" style="zoom: 67%;" />

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145733.jpg" alt="CPU用户态-内核态.jpeg"  />

1号进程和普通用户进程不太一样，它是所有普通用户进程的祖宗，刚开始时并没有用户态，所以是从 **内核态到用户态**。它是怎么从内核态回到用户态的？

**（2）初始化2号进程——内核态总管**

2号进程通过 kernel_thread(kthreadadd,NULL,CLONE_FS|CLONE_FILES)创建，是所有内核态进程的祖宗。

##### 四、系统调用

**glibc对系统调用的封装**，linux提供了glibc这个中介，将系统调用封装成更加友好的接口，供我们调用。

glibc中有个文件syscalls.list 里面列着所有glibc的函数对应的系统调用。

glibc还有一个脚本 make-syscall.sh，根据syscall.list里面的配置对于每个封装好的系统调用生成一个文件，这个文件里面定义了一些宏，比如 #define SYSCALL_NAME open，这个宏被glibc的另一个文件syscall-template.S使用，而宏里面定义了系统调用的调用方式，显示任何一个系统调用最终都会调用DO_CALL这个宏，对于32位和64位 DO_CALL 的定义是不一样的。

###### **1、32位系统调用**

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145734.jpg" alt="32系统调用.jpg" style="zoom:67%;" />

```
/* Linux takes system call arguments in registers:
  syscall number  %eax       call-clobbered
  arg 1    %ebx       call-saved
  arg 2    %ecx       call-clobbered
  arg 3    %edx       call-clobbered
  arg 4    %esi       call-saved
  arg 5    %edi       call-saved
  arg 6    %ebp       call-saved
......
*/
#define DO_CALL(syscall_name, args)                           \
    PUSHARGS_##args                               \
    DOARGS_##args                                 \
    movl $SYS_ify (syscall_name), %eax;                          \
    ENTER_KERNEL                                  \
    POPARGS_##args
```

（1）将请求参数放到寄存器里面，根据系统调用的名称得到系统调用号，放在寄存器eax里面，然后执行ENTER_KERNEL。

（2）ENTER_KERNEL 是个宏，定义为 int  $0x80 ，int是interrupt中断的意思，int $0x80触发一个软中断，通过它可以陷入(trap)内核。

（3）内核启动的时候有个trap_init()，其中有这样的代码:

```
set_system_intr_gate(IA32_SYSCALL_VECTOR,entry_INT80_32);
```

这是一个软中断的陷入门，当接收到一个系统调用的时候，entry_INT80_32就被调用了，entry_INT80_32 ，通过push和SAVE_ALL将当前用户态的寄存器保存在pt_regs结构里面。保存所有寄存器后，然后调用 do_syscall_32_irqs_on。

```
ENTRY(entry_INT80_32)
        ASM_CLAC
        pushl   %eax                    /* pt_regs->orig_ax */
        SAVE_ALL pt_regs_ax=$-ENOSYS    /* save rest */
        movl    %esp, %eax
        call    do_syscall_32_irqs_on
.Lsyscall_32_done:
......
.Lirq_return:
  INTERRUPT_RETURN
```

（4）do_syscall_32_irqs_on 中将系统调用号从eax里面取出来，然后根据系统调用号，在系统调用表中找到相应的函数进行调用，并将寄存器中保存的参数取出来，作为函数参数。

```
static __always_inline void do_syscall_32_irqs_on(struct pt_regs *regs)
{
  struct thread_info *ti = current_thread_info();
  unsigned int nr = (unsigned int)regs->orig_ax;
......
  if (likely(nr < IA32_NR_syscalls)) {
    regs->ax = ia32_sys_call_table[nr](
      (unsigned int)regs->bx, (unsigned int)regs->cx,
      (unsigned int)regs->dx, (unsigned int)regs->si,
      (unsigned int)regs->di, (unsigned int)regs->bp);
  }
  syscall_return_slowpath(regs);
}
```

（5）系统调用结束后，回到 entry_INT80_32，在这后面紧接着调用 INTERRUPT_RETURN，也就是iret，iret指令将原来用户态保存的现场恢复回来，包括代码段、指令指针寄存器等。这时候用户态进程恢复执行。

```
#define INTERRUPT_RETURN                iret
```

###### **2、64位系统调用**

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145735.jpg" alt="64系统调用.jpg" style="zoom:67%;" />

```
/* The Linux/x86-64 kernel expects the system call parameters in
   registers according to the following table:
    syscall number  rax
    arg 1    rdi
    arg 2    rsi
    arg 3    rdx
    arg 4    r10
    arg 5    r8
    arg 6    r9
......
*/
#define DO_CALL(syscall_name, args)                \
  lea SYS_ify (syscall_name), %rax;                \
  syscall
```

（1）和32位系统调用一样，还是将系统调用名称转换成系统调用号，放到寄存器rax中，不过后面进行了真正调用，该用syscall指令了，而不是使用中断了。传递参数的寄存器也变了。

（2）syscall指令还是使用了一种特殊的寄存器，**特殊模块寄存器(Model specific Register，简称 MSR)**，这种寄存器是CPU为 了完成某些特殊控制功能为目的的寄存器，其中就有系统调用。

（3）系统初始化的时候，trap_init() 除了中断模式，还有会调用 cpu_init->sys call_init。这里面有代码：

```
wrmsrl(MSR_LSTAR, (unsigned long)entry_SYSCALL_64);
```

rdmsr 和 wrmsr 是用来读写特殊模块寄存器的。MSR_LSTAR这个特殊的寄存器，当syscall指令调用的时候，会从这个寄存器里面拿出函数地址来调用，也就是调用 entry_SYSCALL_64。

entry_SYSCALL_64 先保存了很多寄存器到 pt_regs 结构里面，例如用户态的代码段、数据段、保存参数的寄存器，然后调用 entry_SYSCALL64_slow_path->do_syscall_64。

```

ENTRY(entry_SYSCALL_64)
        /* Construct struct pt_regs on stack */
        pushq   $__USER_DS                      /* pt_regs->ss */
        pushq   PER_CPU_VAR(rsp_scratch)        /* pt_regs->sp */
        pushq   %r11                            /* pt_regs->flags */
        pushq   $__USER_CS                      /* pt_regs->cs */
        pushq   %rcx                            /* pt_regs->ip */
        pushq   %rax                            /* pt_regs->orig_ax */
        pushq   %rdi                            /* pt_regs->di */
        pushq   %rsi                            /* pt_regs->si */
        pushq   %rdx                            /* pt_regs->dx */
        pushq   %rcx                            /* pt_regs->cx */
        pushq   $-ENOSYS                        /* pt_regs->ax */
        pushq   %r8                             /* pt_regs->r8 */
        pushq   %r9                             /* pt_regs->r9 */
        pushq   %r10                            /* pt_regs->r10 */
        pushq   %r11                            /* pt_regs->r11 */
        sub     $(6*8), %rsp                    /* pt_regs->bp, bx, r12-15 not saved */
        movq    PER_CPU_VAR(current_task), %r11
        testl   $_TIF_WORK_SYSCALL_ENTRY|_TIF_ALLWORK_MASK, TASK_TI_flags(%r11)
        jnz     entry_SYSCALL64_slow_path
......
entry_SYSCALL64_slow_path:
        /* IRQs are off. */
        SAVE_EXTRA_REGS
        movq    %rsp, %rdi
        call    do_syscall_64           /* returns with IRQs disabled */
return_from_SYSCALL_64:
  RESTORE_EXTRA_REGS
  TRACE_IRQS_IRETQ
  movq  RCX(%rsp), %rcx
  movq  RIP(%rsp), %r11
    movq  R11(%rsp), %r11
......
syscall_return_via_sysret:
  /* rcx and r11 are already restored (see code above) */
  RESTORE_C_REGS_EXCEPT_RCX_R11
  movq  RSP(%rsp), %rsp
  USERGS_SYSRET64
```

（4）在do_syscall_64里面，从rax里面拿出系统调用号，然后根据系统调用号，在系统调用表sys_call_table中找到相对应的函数进程调用，并将寄存器中保存的参数取出来，作为函数参数。

```
__visible void do_syscall_64(struct pt_regs *regs)
{
        struct thread_info *ti = current_thread_info();
        unsigned long nr = regs->orig_ax;
......
        if (likely((nr & __SYSCALL_MASK) < NR_syscalls)) {
                regs->ax = sys_call_table[nr & __SYSCALL_MASK](
                        regs->di, regs->si, regs->dx,
                        regs->r10, regs->r8, regs->r9);
        }
        syscall_return_slowpath(regs);
}
```

（5）系统调用返回后，执行的是USERGS_SYSRET64，返回用户态的指令变成了sysretq。

```
#define USERGS_SYSRET64        \
  swapgs;          \
  sysretq;
```

###### **3、系统调用表**

###### **4、完整的64位系统调用**

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145736.jpg" alt="64位系统调用全过程.jpg" style="zoom:67%;" />