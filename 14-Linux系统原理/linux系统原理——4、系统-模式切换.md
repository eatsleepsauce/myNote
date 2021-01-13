##### 模式切换

##### 一、BIOS时期

计算机通电启动后CPU就可以执行指令了，这个时候没有操作系统，内存也是空的，CPU只能从 **ROM（Read Only Memory，只读存储器）**中读取早就固化了的一些初始化程序，也就是 **BIOS（Basic Input and Output System，基本输入输出系统）**。

刚启动的时候处于实模式，在x86系统中，将1M空间**最上面的 0xF0000 到 0xFFFFF 这 64k 映射给了ROM** （约定的），也就说访问这个部分地址的时候，会访问ROM。

（1）刚加电的时候会将CS设置为0xFFFF，将IP设置为0x0000，所以第一条指令指向了0xFFFF0，正是在ROM范围内。在这里会有一个JMP指令跳转到ROM做初始化，也就是BIOS开始进行初始化工作，BIOS第一步主要是检查系统硬件是不是都好着。

（2）为了基本输入输出设备能够正常使用，需要建立中断向量表和中断服务程序。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145726.jpg" alt="地址空间组成.jpeg" style="zoom: 50%;" />

##### **二、bootloader 时期**

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

![启动项.png](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/2020-12-08-145725.png)



（2）使用grub2 -install /dev/sda，可以将启动程序安装到相应的位置，grub2第一个要安装是boot.img。它由boot.S编译而成，一共512字节，正式安装到启动盘的第一个扇区。这个扇区通常成为 **MBR（Master Boot Record，主引导记录/扇区）**。

（3）BIOS完成任务后，会将 **boot.img** 从硬盘第一个扇区加载到内存中的 0x7c00来运行，由于只有512字节，boot.img做不了太多事情，它做的最重要的一个事情就是加载grub2的另一个镜像 core.img。

（4）**core.img** 由**diskboot.img、lzma_decompress.img、kernel.img和一系列的模块**组成，功能比较丰富，能做很多事情。core.img的第一个扇区里面是diskboot.img（从硬盘启动），对应的代码就是 diskboot.S，boot.img将控制权交给diskboot.img后，diskboot.img的任务就是将core.img的其它部分加载进来。

（5）diskboot.img 先加载 **解压缩程序lzma_decompress.img**，再往下就是 **kernel.img (grub内核非linux内核)** ，最后是各个模块对应的镜像。（kernel.img是压缩过的，所以需要 lzma_decompress.img，lzma_decompress.img对应的代码时 startup_raw.S）

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145728.jpg" alt="加载顺序.jpeg" style="zoom: 67%;" />

**在真正的解压缩之前，lzma_decompress.img 调用 real_to_prot ，切换到保护模式**。

##### 三、实模式切换到保护模式

切换到保护模式的大部分工作都与内存访问有关：

（1）第一项，**启动分段**，就是在内存里面建立段描述符表，将寄存器里面的段寄存器变成段选择子，指向某个段描述符，这样就能实现不同进程的切换了。

（2）第二项，**启动分页**，能够管理的内存变大了，就需要将内存分成相等大小的块。

（3）**打开 Gate 20**，也就是第21根地址线的控制线，切换保护模式的函数 DATA32 call real_to_prot 会打开Gate A20。

（4）有了空间后，对压缩的 **kernel.img 进行解压**，然后跳转到kernel.img运行，kernel.img对应的代码时startup.S以及一堆c文件，在startup.S中会 **调用grub_main**，这是grub kernel的主函数。

（5）grub_main 这个主函数会 **调用 grub_load_config() 解析 grub.cfg** 文件(上面的代码)的配置信息，如果正常grub_main 最后会调用 grub_command_execute("normal",0,0) 》grub_normal_execute()函数 》grub_show_menu()函数 会显示出让选择哪个操作系统的列表。

（6）**grub_menu_execute_entry() 开始解析并执行之前的选择**，linux16 命令表示装载指定的内核文件，并传递内核启动参数。于是 grub_cmd_linux() 函数会被调用——读取linux内核镜像头部的一些数据结构，放到内存中的数据结构，进行检查，如果通过则会读取整个linux内核镜像到内存。

（7）如果grub配置文件中还有initrd命令——用于为即将启动的内核传递 init ramdisk路径。grub_cmd_initrd()函数会被调用，将initramfs加载到内存中来。

（8）上面事情都做完后，**grub_command_execute(“boot”,0,0) 才开始真正地启动内核**。