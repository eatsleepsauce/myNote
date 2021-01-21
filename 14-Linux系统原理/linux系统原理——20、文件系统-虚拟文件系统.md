#### 虚拟文件系统

咱们的图书馆书架，也就是硬盘上的文件系统格式都搭建好了，现在我们还需要一个图书管理与借阅系统，也就是文件管理模块，不然我们怎么知道书都借给谁了呢？

进程要想往文件系统里面读写数据，需要很多层的组件一起合作：

- 在应用层，进程在进行文件读写操作时，可通过系统调用如 sys_open、sys_read、sys_write 等。
- 在内核，每个进程都需要为打开的文件，维护一定的数据结构。
- 在内核，整个系统打开的文件，也需要维护一定的数据结构。
- Linux 可以支持多达数十种不同的文件系统。它们的实现各不相同，因此 Linux 内核向用户空间提供了虚拟文件系统这个统一的接口，来对文件系统进行操作。它提供了常见的文件系统对象模型，例如 inode、directory entry、mount 等，以及操作这些对象的方法，例如 inode operations、directory operations、file operations 等。
- 然后就是对接的是真正的文件系统，例如我们上节讲的 ext4 文件系统。
- 为了读写 ext4 文件系统，要通过块设备 I/O 层，也即 BIO 层。这是文件系统层和块设备驱动的接口。
- 为了加快块设备的读写效率，我们还有一个缓存层。
- 最下层是块设备驱动程序。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/3c506edf93b15341da3db658e9970773.jpg" alt="3c506edf93b15341da3db658e9970773" style="zoom:33%;" />

解析系统调用是了解内核架构最有力的一把钥匙，这里我们只要重点关注这几个最重要的系统调用就可以了：

- mount 系统调用用于挂载文件系统；
- open 系统调用用于打开或者创建文件，创建要在 flags 中设置 O_CREAT，对于读写要设置 flags 为 O_RDWR；
- read 系统调用用于读取文件内容；
- write 系统调用用于写入文件内容。



文件的数据结构层次多，而且很复杂：

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/8070294bacd74e0ac5ccc5ac88be1bb9.png" alt="8070294bacd74e0ac5ccc5ac88be1bb9" style="zoom: 50%;" />

对于每一个进程，打开的文件**都有一个文件描述符，在 files_struct 里面会有文件描述符数组**。每个一个文件描述符是这个数组的下标，里面的内容指向一个 file 结构，表示打开的文件。这个结构里面有这个文件对应的 inode，最重要的是这个文件对应的操作 file_operation。如果操作这个文件，就看这个 file_operation 里面的定义了。

对于每一个打开的文件，都有一个 dentry 对应，虽然叫作 directory entry，但是不仅仅表示文件夹，也表示文件。它最重要的作用就是指向这个文件对应的 inode。

如果说 file 结构是一个文件打开以后才创建的，dentry 是放在一个 dentry cache 里面的，文件关闭了，他依然存在，因而他可以更长期地维护内存中的文件的表示和硬盘上文件的表示之间的关系。

inode 结构就表示硬盘上的 inode，包括块设备号等。

几乎每一种结构都有自己对应的 operation 结构，里面都是一些方法，因而当后面遇到对于某种结构进行处理的时候，如果不容易找到相应的处理函数，就先找这个 operation 结构，就清楚了。