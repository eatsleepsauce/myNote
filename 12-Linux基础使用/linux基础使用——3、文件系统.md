#### linux文件系统

##### 一、文件系统简介

linux没有windows中的磁盘符，只有根目录 /，linux根目录下面的目录基本都是约定的。

- /bin 可执行文件，用户命令
- /sbin 管理命令
- /usr/bin   /usr/sbin 系统预装的其他命令
- /boot 系统启动相关的文件，如内核、initrd以及grub(bootloader)就是前面虚拟机安装系统时设置的boot引导分区
- /dev 设备文件(鼠标键盘之类)
- **/etc 配置文件**
- /home 用户家目录
- /root 管理员家目录
- /lib 库文件(含第三方)
- /media 挂载点目录，移动设备
- /mnt 挂载点目录，额外的临时文件系统
- **/opt 可选目录，第三方程序的安装目录**
- /proc 伪文件系统，内核映射文件
- /sys 伪文件系统，跟硬件设备相关的属性映射文件
- /tmp 临时文件
- /var 可变化的文件

##### 二、文件系统相关命令

**（1）df 显示磁盘(分区)使用情况**，df -h 查看分区使用情况，-h单位gb

**（2）du 显示文件系统使用情况**，du -h 查看文件系统的使用具体情况，单位kb，一般先用df查看分区，再看具体文件的情况

**（3）ls 显示文件目录**，使用： ls [可选项] [文件名] ，ls /root  /home  （可以空格隔开多个目录，不带参数时候表示当前目录）
   常用可选 （可以使用man查看更多）

- -a：列出目录下所有文件和目录含. 及 ..
- -l ：列出目录文件和目录详细信息
- -r :  逆向排序展示，默认按名称，一般配合 -l 使用
-  -t :  按时间排序展示，和-r一样一般配合 -l 使用

- -R:  递归展示文件夹下的子文件

选项可以合并写，如ls -lrt  等同于 ls -l -r -t

```
ls -l
-rwxr-x---. 1 root root 1024 Sep 12 20:20 xxx
```

**文件类型：**

- \- 普通文件
- d 目录文件
- b 块设备文件
- c 字符设备文件
- l 符号链接文件
- p 命令管道文件
- s 套接字文件

**权限说明：**9位，每3位为一组（rwx 读写执行，-表示没有权限），第一组三个表示属主的权限，第二组三个表示属主的用户组权限，最后一组表示其它用户；. 是分隔符 ；1 表示硬链接的次数  root 第一个是属主，后一个root是属组文件大小，单位字节，可以添加 -h选项 按M来显示文件大小；时间戳表示最近的一次更新时间。

**（4） cd进入文件目录 、mkdir创建目录、cp拷贝、 mv移动或改名**

- . 表示当前目录
- .. 表示上一级目录
- cd // 回到家目录
- cd - //回到之前操作的目录
- cd /etc // 进入目录
- cd .. // 返回上一级
- cd ~ // 返回家目录
- pwd // 查看当前目录
- mkdir  aaa // 当前目录下创建目录 aaa
- **mkdir  -p aaa/bbb/ccc // 加-p 可以接连创建目录而不报错**
- mkdir aaa/ccc aaa/ddd // 同时创建多个目录
- mkdir aaa/{1,2,3}dir // aaa下创建1dir 2dir和3dir

- cp aaa bbb // 将aaa文件拷贝到当前目录的bbb目录下
- cp -v aaa bbb  // 展示复制过程
- cp -p aaa bbb  // 保留复制后文件的创建时间等
- cp -a aaa bbb  // 基本和复制前的文件一致包括创建时间和权限信息
- cp -r  bbb ccc  // 增加-r参数 拷贝目录，将bbb目录拷贝到ccc目录下

- mv xxx yyy  // 将xxx移动到当前目录的yyy目录下
- mv aaa  aaa.bak // 将aaa改名为aaa.bak，可以同时移动目录和重命名，mv /a/b /c/bb

**(5)、rm 删除、ln链接**

- rm  xxx  // 删除文件，需要确认
- rm -f xxx // 删除文件，无需确认
- rm -r  yyyy // 删除目录，增加-f参数，无需确认  rm -rf 
- rmdir xxx // 虽然可以删除目录，但是必须保证目录为空，所以不常用

- ln profile  aaa // 硬链接，aaa链接到了profile指向的文件，把profile删除了，aaa不影响，还能继续查看和编辑
  ls -i // 查看具体的文件号，我们平时看到的aaa之类的只是个符号，这个符号链接到具体的文件
  ln -s  profile bbb  // 软链接，bbb指向了profile，而profile链接到具体文件，删除profile，则bbb受影响，无法使用

**(6)、stat 查看文件详细信息、touch 一致时间和创建文件**

- stat 查看更详细的文件信息，比ls -l 更详细

```sh
[root@node01 ~]# stat myfile
  File: `myfile'
  Size: 70        	Blocks: 8          IO Block: 4096   regular file
Device: 803h/2051d	Inode: 4194318     Links: 1
Access: (0646/-rw-r--rw-)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2020-09-20 01:32:28.745483645 +0800
Modify: 2020-09-20 01:33:06.843484058 +0800
Change: 2020-09-20 01:33:06.844484058 +0800
// access 表示文件的访问时间
// modify 文件内容修改的时间
// change 文件元数据修改的时间，比如 文件权限修改，修改文件内容也会修改change
```

- touch用于统一文件的 access、modify和change时间

```sh
[root@node01 ~]# touch myfile
[root@node01 ~]# stat myfile
  File: `myfile'
  Size: 70        	Blocks: 8          IO Block: 4096   regular file
Device: 803h/2051d	Inode: 4194318     Links: 1
Access: (0646/-rw-r--rw-)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2020-09-20 01:46:58.376483329 +0800
Modify: 2020-09-20 01:46:58.376483329 +0800
Change: 2020-09-20 01:46:58.376483329 +0800
```

- **touch还可以用于创建文件  touch myfile**

**(7)、文件查看相关命令**

- **cat  查看文件内容**，可以连接多个文件展示，cat 会将内容一下子全部展示出来（文件太大的话不好查看，一屏能展示的可以用cat）
  cat myfile 
  cat myfile1 myfile2 ...

- **more 查看文件内容**，内容会分页展示，空格到下一页，回看比较麻烦
  more myfile

- **less  查看文件内容**，内容也是分页展示，空格到下一页，回看按b，less会将文件加载到内存中，读取太大文件的话无法加载到内容中，这时只能用more了。
  less myfile

- **head 查看文件前几行内容**
  head - 5 myfile   查看文件前5行内容

- **tail 查看文件**的后几行和监控文件同步更新
  tail -5 myfile  查看文件最后5行内容
  tail -f myfile  监控文件的同步更新，-f 同步更新

- **wc统计文件长度**
  wc -l myfile  查看文件有多少行 

**(8)、打包压缩和解压缩**
linux保留着传统的打包和压缩两个步骤，打包是将目录打包成一个文件。

- **打包 tar**
  tar cf /tmp/etc_bk.tar  /etc   // 将/etc 系统配置目录备份到tmp目录下的etc_bk.tar 文件中

- **压缩形式有gzip 和bzip2，tar已经集成了者两个压缩方式**
  tar zcf /tmp/etc_bk.tar.gz  /etc  // gzip 
  tar jcf /tmp/etc_bk.tar.bz2 /etc  // bzip2 压缩得更小，所以速度慢

- **解压(解包)**
  tar xf /tmp/etc_bk.tar -C /root // 解包到 root目录下
  tar zxf /tmp/etc_bk.tar.gz -C /root // 解压解包到 root目录下 gzip
  tar jxf /tmp/etc_bk.tar.bz2 -C /root  //解压解包到 root目录下 bzip2
  **注意，很多文件可能类似tgz、tbz2 这种单个扩展名的，其实就是双扩展名的简写。**

**(9)、管道的使用**
管道的作用就是将前面的输出做为后面的输入。
top -5  myfile  | tail -1  取第五行
有些命令不接收管道左边的输出做为输出，可以使用 **xargs** 接收前面的输出，并帮我们完善命令执行，如： 
echo  "/" | xargs ls -l 
