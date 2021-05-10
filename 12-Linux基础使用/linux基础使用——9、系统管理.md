#### 系统管理

##### 一、网络管理

###### **1、网络状态查看**  

**网络状态查看工具：**

- net-tools

  - ifconfig，查网卡信息。

    ifconfig eth0 查看具体网卡信息， eth0 第一块网卡名称（网络接口）。

    第一个网络接口（网卡）还有可能是以下名字：  

    - eno1 板载网卡 
    -  ens33 PCI-E网卡  
    - enp0s3 无法获取物理信息的PCI-E 网卡  
    - CentOS 7 使用了一致网络设备命名，以上都不匹配则使用 eth0。网络接口名称修改，网卡命名规则受biosdevname和net.ifnames两个参数影响，如果想网卡命名为eth0，可以编辑 /etc/default/grub文件，增加biosdevname=0 net.ifnames=0，更新grub，命令 grub2-mkconfig -o /boot/grub2/grub.cfg，最后重启，命令 reboot。

  - mii-tool eth0(网卡名称)，查看网卡物理连接情况

  - route，查看网关信息，route -n，使用 -n参数不解析主机名，因为ip解析成主机名会浪费时间，正常可以不解析主机名。

  - netstat 查看服务的监听地址和端口

- iproute2
  -  -ip  
  - -ss

###### 2、网络配置

- 设置网卡信息，ifconfig <接口> <IP地址>[netmask 子网掩码] 

  使用：ifconfig eth0 192.168.152.11 netmask 255.255.255.0

- 启用网卡，ifup <接口> 启用网卡

- 停用网卡，ifdown <接口> 停用网卡

网关相关配置，添加网关，使用route命令

- route add default gw <网关ip>，设置默认网关  
- route add -host <指定ip> gw <网关ip>，设置访问具体主机时走指定网关 
-  route add -net <指定网段> netmask <子网掩码> gw <网关ip> 设置特定网段的访问走指定的网关  

网关删除

- route del default gw <网关ip> 删除默认网关

###### 3、网络故障排除

- ping 检查当前主机和目标主机是否畅通

- traceroute 追踪当前主机和目标主机之间到每一跳

- mtr 检查当前主机到目标主机访问时是否有包丢失

- nslookup 查看域名对应到ip
- telnet 检查目标主机的端口是否畅通 telnet [www.baidu.com](www.baidu.com) 80
- tcpdump 网络抓包工具，分析数据包
- netstat 查看服务的监听地址和端口，netstat -ntpl (n不要解析成主机名，t 只找tcp协议的，p进程，l listen监听)
- ss

###### 4、网络服务管理和配置文件

**我们正常设置的配置，linux重启完后配置就又还原了，要想每次启动都使用新的配置，只有修改linux配置文件**

**网络服务管理程序分为两种，分别为 SysV 和 systemd (CentOS 7以上版本) ，一般情况下只使用一种网络服务，使用一个的同时禁止另一个，正常使用network服务，不启用NetworkManager服务。** 

SysV基于network 服务，有如下：

- service network start|stop|restart 网络服务的启动、终止和重启  service network status 查看当前网络服务状态

- chkconfig --list network 查看network服务的设置

systemd基于NetworkManager服务，有如下：

- systemctl list-unit-files NetworkManager.service 查看NetworkManager 服务

- systemctl start|stop|restart NetworkManager

- systemctl enable|disable NetworkManager 开启或禁用NetworkManager服务

###### 5、配置文件

- ifcfg-eth0 网卡配置文件，eth0为网卡名称，文件路径 /etc/sysconfig/network-scripts/

- /etc/hosts 主机名相关配置。
  - hostname 可以查看当前主机名，hostname xxx 设置主机名，系统重启后会还原。  
  - hostnamectl set-hostname xxx  修改了主机名后，将主机名和ip关系写到/etc/hosts文件中，127.0.0.1 xxx，这样主机名就可以永久生效。

##### 二、进程管理

###### 1、进程的概念与进程查看

进程：运行中的程序，从程序开始运行到终止的整个生命周期是可管理的。

C程序的启动是从main函数开始的

```c
int main(int arg, char *argv[])
```

进程的终止方式分正常终止和异常终止：

- 正常终止也分为从main返回、调用exit等方式

- 异常终止分为调用abort、接收信号等 

**进程查看命令**    

- ps  查看当前终端运行的进程，查看所有的进程 ps -e，ps -ef 能展示命令所在目录，ps -elf 能查看线程信息，使用man查看更多参数

- pstree 将进程以树状的结构展示，进程也有子父级关系

- top 查看进程动态信息，默认按3秒刷新，可以按s修改刷新时间，top -p xxx 查看进程号为xxx的进程

###### 2、进程的控制命令

**（1）进程的优先级调整**    

- nice 范围从-20到19，值越小优先级越高，抢占资源就越多，nice -n 10 ./xxx.sh 启动的时候设置优先级

- renice 重新设置优先级，renice -n 15 xxx，对运行状态的，进程号为xxx的进程进行优先级设置 

**（2）进程的作业控制**

- jobs 将进程调回到终端前台运行， jobs 后使用 fg 1 进行前台运行 

- &符号，将进程在终端后台运行，./xxx.sh & ，control + z  进程进行终端后台挂起，再次运行可以使用jobs ，后续可以使用bg 1 或 fg 1 选择后台运行还是前台运行

###### 3、进程的通信方式

信号是进程间通信方式之一，典型的用法是：终端输入终端命令，如control + c ，通过信号机制终止一个程序的运行。

使用信号的常用快捷键和命令：

- kill -l 查看所有信号

- control + c 通知终端前台进程终止，相当于kill -2 pid，对应 2号信号 SIGINT

- kill -9 pid 立即结束程序，不能被阻塞和处理，对应9号信号 SIGKILL

###### 4、守护进程

守护进程不需要在终端启动，输出会输出到特定的地方，占用的目录为根目录。

- **使用nohup与 & 符号配合运行一个命令**

  如：nohup tail -f /var/log/messages &
  - nohup命令使进程忽略hangup(挂起)信号，正常关闭终端后进程也会关闭，nohup可以保证终端关闭后进程依然运行，输出会转到别等地方（因为终端已经关闭了，输出不到终端了）。
  - nohup后进程输出路径可以通过 cd /proc/pid 查看fd下面到标准输入和标准输出所链接到的具体目录。

- **使用screen命令**，screen提供了一个运行环境，主要避免远程登录终端时因网络中断导致终端关闭进而导致运行的进程关闭。    
  - 使用screen命令直接进入screen环境，如果没有安装可以使用yum来安装。
  - ctrl + a 后 再按d(detached) 退出screen环境
  - screen -ls 查看screen的会话
  - screen -r sessionid 恢复会话

##### 三、系统日志

/var/log 下面有大量日志

- messages 系统常规日志

- dmesg  内核启动日志

- secure 系统安全日志

- cron 周期性的任务日志

##### 四、服务管理工具systemctl 

###### 1、服务（提供常⻅功能的守护进程）集中管理⼯具

- service
- systemctl

###### 2、systemctl 常⻅操作

- systemctl start | stop | restart | reload | enable | disable 服务名称
- 软件包安装的服务单元 /usr/lib/systemd/system/

###### 3、systemctl 的服务配置

- [Unit]
  - Requires = 新的依赖服务
  - After = 新的依赖服务
- [Service]
- [Install]
  - 安装到哪个默认启动级别 /lib/systemd/system
  - systemctl get-default | set-default

##### 五、SELinux 简介

安全增强型 Linux（Security-Enhanced Linux）简称 SELinux，它是一个 Linux 内核模块，也是 Linux 的一个安全子系统。SELinux 主要作用就是最大限度地减小系统中服务进程可访问的资源（最小权限原则）。

- MAC（强制访问控制）与 DAC（⾃主访问控制）
- 查看 SELinux 的命令
  - getenforce
  - /usr/sbin/sestatus
  - ps -Z and ls -Z and id -Z
- 关闭 SELinux
  - setenforce 0
  - /etc/selinux/sysconfig    

##### 六、内存与磁盘管理

###### 1、内存和磁盘使用率查看    

**内存使用率常用命令**：

- free 查看内存和交换分区情况， -m参数后以M为单位展示大小，(小知识：正常情况下最好是不要让交换区被使用，交换区使用意味着内存空间不够了，程序运行会变慢)

- top 展示动态数据

**磁盘使用率的查看**：

- fdisk 既能查看磁盘又能分区，查看磁盘分区 fdisk -l，还有另外一个命令 parted -l查看的内容差不多。

- df 常用df -h，用做fdisk的补充，能看到分区挂载的目录以及磁盘分区更详细的空间使用信息。

- du，du -h 查看文件的大小，统计文件真正的长度。

- du 与 ls的区别，ls -lh 查看的是文件开始到结尾的长度，中间存在空洞的长度也算。    

###### 2、常见文件系统

（1）ext4

- 超级快

- 超级块副本

- i 节点(inode) ，记录文件的权限信息等，文件名记录在父目录的inode中，使用ls -i 查看文件的inode，我们看到的文件编号就是i节点。

- 数据块(datablock)，记录文件的数据，du 就是统计等数据块大小 。

（2）xfs

（3）NTFS（需要额外安装软件）

**ext4和xfs提供了 facl 更好的对文件权限控制**

###### 3、磁盘分区与挂载

**常见命令：**

- fdisk 创建磁盘分区

- mkfs 格式化分区 创建文件系统

- parted 磁盘大于2T时使用parted进行分区

- mount 挂载

**常见配置文件：**

- /etc/fstab 
- 正常mount都是保存内存中的，没有持久化，一旦重启就失效了，通过fstab可以保证开机时就挂载

###### 4、用户的磁盘配额

- xfs文件系统的用户磁盘配额quota
- mkfs.xfs /dev/sdb1
- mkdir /mnt/disk1
- mount -o uquota,gquota /dev/sdb1 /mnt/disk1
- chmod 1777 /mnt/disk1
- xfs_quota -x -c ‘report -ugibh’ /mnt/disk1 
- xfs_quota -x -c ‘limit -u isoft=5 ihard=10 user1’ /mnt/disk1 

###### 5、交换分区查看与创建

扩展swap分区大小，和正常分区一样先创建分区，后执行：

- mkswap 将分区变成swap分区

- swapon 开启swap，关闭分区做为swap，可以使用swapoff

还可以使用文件制作交换分区：

- dd if=/dev/zero bs=4M count=1024 of=/swapfile （bs块大小4m，个数1024个）

- mkswap /swapfile

- swapon /swapfile

每次开机都要生效，需要修改/etc/fstab，参见 3磁盘分区与挂载。

###### 6、软件RAID的使用

RAID的常见级别及含义：

- RAID0 striping条带式，提高单盘吞吐率，至少两块磁盘，每块均摊

- RAID1 mirroring镜像方式，提高可靠性，至少两个块磁盘，存储相同数据增加可靠性

- RAID5 有奇偶校验，至少三块磁盘，例如，前两块依次写入数据，第三块存第一和第二块的校验数据，只要不挂两个以上磁盘都能恢另外一个

- RAID10 是RAID1与RAID0的结合，至少4块磁盘

RAID卡帮我们实现了磁盘的访问，我们工作中使用的也是RAID卡(RAID控制器)

软件RAID的使用(工作中不使用)：mdadm

###### 7、逻辑卷管理LVM

- 逻辑卷和文件系统的关系
- 为linux创建逻辑卷
- 动态扩容逻辑卷

##### 七、系统综合状态查看

- 使⽤ sar 命令查看系统综合状态
- 使⽤第三⽅命令查看⽹络流量
  - yum install epel-release
  - yum install iftop
  - iftop -P