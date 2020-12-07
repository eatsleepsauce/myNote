####一、准备篇
**1、虚拟机及系统安装**
安装VMware Workstation，安装完成后在安装linux系统之前进入系统的bios设置保证虚拟化的能力是被打开的。
配置虚拟机的硬件能力：cpu、内存、硬盘、网络NAT模式等
安装操作系统，注意设置三个区：
（1）boot引导程序区 
（2）swap交换区——设置为虚拟机内存的两倍大小
（3）用户区——虚拟机的剩的最大空间

**2、网络配置**
找到虚拟机可以配置的网络ip信息(查看vmware的编辑-虚拟网络编辑器-更改设置，查看NAT模式类型下的子网ip信息，NAT设置中可以查看到网关ip信息）

配置网络：
```
cd /etc/sysconfig/network-scripts/
vi ifcfg-eth0

删除网卡物理地址以及uuid，方便后续虚拟机克隆
DEVICE=eth0
TYPE=Ethernet
ONBOOT=yes
NM_CONTROLLED=yes
IPADDR=xxx.xxx.xxx.xxx
GATEWAY=xxx.xxx.xxx.xxx
DNS1=114.114.114.114
DNS2=xxx.xxx.xxx.xxx
DNS3......

保存后 service network restart 重启网络服务
验证：ping  www.baidu.com...
```
**3、基于创建好的虚拟机克隆出多台虚拟机**
关闭已创建的虚拟机，右击进入快照管理进行拍摄快照生成快照，这个快照可以用于克隆也可用于当前虚拟机的恢复（右击虚拟机进入管理-克隆，选择之前生成的快照）。
克隆完后，需要进行几步操作：
（1）修改ip，参照上面网络配置
（2）修改hostname
```
cd /etc/sysconfig
vi network
```
（3）删除一个记录网卡物理地址和网卡名称关系的文件，删除后系统自动会重新生成一个
```
cat /etc/udev/rules.d/70-persistent-net.rules  可以查看到两者关系
rm -f /etc/udev/rules.d/70-persistent-net.rules 
reboot 重启
```
**4、Xshell、Xftp使用**
ssh root@xxx.xxx.xxx.xxx

####二、基础说明

**1、内外部命令**
shell自带的命令为内部命令；外部命令，不是shell自带的命令，由用户安装的。

**用户操作后，系统会**
(1)根据空格来切割字符串，把第一个位置认为是命令，其它位置的认为是命令的参数。
(2)判断命令是内部命令还是外部命令
(3)如果是内部命令直接交给linux内核执行，如果是外部命令先寻找这个命令的执行文件，再由内核执行。
```
查看内部命令
type cd
cd is a shell buitin

查看文件类型
file ifconfig
查看文件内容
cat 文件 

可执行文件的路径
whereis ifconfig
```
**外部命令执行会很慢么？** *不会，系统在PATH的目录范围内寻找找可执行文件。*
```
echo $PATH  查看path配置的哪些路径
```
**2、命令帮助文档**

(1) man 一般用于外部命令帮助文档，使用：man ifconfig，如果man命令不存在，安装man命令  yum install man。
man命令也带有参数，可以通过man man 查看，一般情况下 man 1 ls ，1代表命令可以省略，man -a xxx 会展示匹配的包含命令以及命令之外的(如 passwd就有命令和配置文件)

(2)help一般用于内部命令帮助文档，使用：help cd，help还有另外一种使用方法，cd --help。

(3)info比help更详细可以做为help的补充，使用：info ls 。

####三、linux文件系统

**1、文件系统简介**
linux没有windows中的磁盘符，只有根目录 /，linux根目录下面的目录基本都是约定的。

/bin 可执行文件，用户命令
/sbin 管理命令
/usr/bin   /usr/sbin 系统预装的其他命令
/boot 系统启动相关的文件，如内核、initrd以及grub(bootloader)就是前面虚拟机安装系统时设置的boot引导分区
/dev 设备文件(鼠标键盘之类)
**/etc 配置文件**
/home 用户家目录
/root 管理员家目录
/lib 库文件(含第三方)
/media 挂载点目录，移动设备
/mnt 挂载点目录，额外的临时文件系统
**/opt 可选目录，第三方程序的安装目录**
/proc 伪文件系统，内核映射文件
/sys 伪文件系统，跟硬件设备相关的属性映射文件
/tmp 临时文件
/var 可变化的文件

**2、文件系统相关命令**

(1)、df 显示磁盘(分区)使用情况
df -h 查看分区使用情况，-h单位gb

(2)、du 显示文件系统使用情况
du -h 查看文件系统的使用具体情况，单位kb，一般先用df查看分区，再看具体文件的情况

(3)、ls 显示文件目录，使用： ls [可选项] [文件名] ，ls /root  /home  （可以空格隔开多个目录，不带参数时候表示当前目录）
   常用可选 （可以使用man查看更多）
   -a：列出目录下所有文件和目录含. 及 ..
   -l ：列出目录文件和目录详细信息
   -r :  逆向排序展示，默认按名称，一般配合 -l 使用
   -t :  按时间排序展示，和-r一样一般配合 -l 使用
  -R:  递归展示文件夹下的子文件
  选项可以合并写，如ls -lrt  等同于 ls -l -r -t

```
ls -l
-rwxr-x---. 1 root root 1024 Sep 12 20:20 xxx
```
文件类型：
\- 普通文件
d 目录文件
b 块设备文件
c 字符设备文件
l 符号链接文件
p 命令管道文件
s 套接字文件

权限说明：
9位，每3位为一组（rwx 读写执行，-表示没有权限）
第一组三个表示属主的权限，第二组三个表示属主的用户组权限，最后一组表示其它用户
.是分隔符
1 表示硬链接的次数
root 第一个是属主，后一个root是属组
文件大小，单位字节，可以添加 -h选项 按M来显示文件大小
时间戳表示最近的一次更新时间

(4) cd进入文件目录 、mkdir创建目录、cp拷贝、 mv移动或改名

. 表示当前目录
.. 表示上一级目录
cd // 回到家目录
cd - //回到之前操作的目录
cd /etc // 进入目录
cd .. // 返回上一级
cd ~ // 返回家目录
pwd // 查看当前目录

mkdir  aaa // 当前目录下创建目录 aaa
**mkdir  -p aaa/bbb/ccc // 加-p 可以接连创建目录而不报错**
mkdir aaa/ccc aaa/ddd // 同时创建多个目录
mkdir aaa/{1,2,3}dir // aaa下创建1dir 2dir和3dir

cp aaa bbb // 将aaa文件拷贝到当前目录的bbb目录下
cp -v aaa bbb  // 展示复制过程
cp -p aaa bbb  // 保留复制后文件的创建时间等
cp -a aaa bbb  // 基本和复制前的文件一致包括创建时间和权限信息
cp -r  bbb ccc  // 增加-r参数 拷贝目录，将bbb目录拷贝到ccc目录下

mv xxx yyy  // 将xxx移动到当前目录的yyy目录下
mv aaa  aaa.bak // 将aaa改名为aaa.bak，可以同时移动目录和重命名，mv /a/b /c/bb

(5)、rm 删除、ln链接

rm  xxx  // 删除文件，需要确认
rm -f xxx // 删除文件，无需确认
rm -r  yyyy // 删除目录，增加-f参数，无需确认  rm -rf 
rmdir xxx // 虽然可以删除目录，但是必须保证目录为空，所以不常用

ln profile  aaa // 硬链接，aaa链接到了profile指向的文件，把profile删除了，aaa不影响，还能继续查看和编辑
ls -i // 查看具体的文件号，我们平时看到的aaa之类的只是个符号，这个符号链接到具体的文件
ln -s  profile bbb  // 软链接，bbb指向了profile，而profile链接到具体文件，删除profile，则bbb受影响，无法使用

(6)、stat 查看文件详细信息、touch 一致时间和创建文件
stat 查看更详细的文件信息，比ls -l 更详细
```
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
touch用于统一文件的 access、modify和change时间
```
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
**touch还可以用于创建文件  touch myfile**

**(7)、文件查看相关命令**

**cat  查看文件内容**，可以连接多个文件展示，cat 会将内容一下子全部展示出来（文件太大的话不好查看，一屏能展示的可以用cat）
cat myfile 
cat myfile1 myfile2 ...

**more 查看文件内容**，内容会分页展示，空格到下一页，回看比较麻烦
more myfile

**less  查看文件内容**，内容也是分页展示，空格到下一页，回看按b，less会将文件加载到内存中，读取太大文件的话无法加载到内容中，这时只能用more了。
less myfile

**head 查看文件前几行内容**
head - 5 myfile   查看文件前5行内容

**tail 查看文件**的后几行和监控文件同步更新
tail -5 myfile  查看文件最后5行内容
tail -f myfile  监控文件的同步更新，-f 同步更新

**wc统计文件长度**
wc -l myfile  查看文件有多少行 

**(8)、打包压缩和解压缩**
linux保留着传统的打包和压缩两个步骤，打包是将目录打包成一个文件。
**打包 tar**
tar cf /tmp/etc_bk.tar  /etc   // 将/etc 系统配置目录备份到tmp目录下的etc_bk.tar 文件中

**压缩形式有gzip 和bzip2，tar已经集成了者两个压缩方式**
tar zcf /tmp/etc_bk.tar.gz  /etc  // gzip 
tar jcf /tmp/etc_bk.tar.bz2 /etc  // bzip2 压缩得更小，所以速度慢

**解压(解包)**
tar xf /tmp/etc_bk.tar -C /root // 解包到 root目录下
tar zxf /tmp/etc_bk.tar.gz -C /root // 解压解包到 root目录下 gzip
tar jxf /tmp/etc_bk.tar.bz2 -C /root  //解压解包到 root目录下 bzip2
**注意，很多文件可能类似tgz、tbz2 这种单个扩展名的，其实就是双扩展名的简写。**

**(9)、管道的使用**
管道的作用就是将前面的输出做为后面的输入。
top -5  myfile  | tail -1  取第五行
有些命令不接收管道左边的输出做为输出，可以使用 **xargs** 接收前面的输出，并帮我们完善命令执行，如： 
echo  "/" | xargs ls -l 

####四、文本编辑器 vi
**vi 并不是真正的编辑，原理是创建一个隐藏文件，内容都在隐藏文件中，最后替换原文件**
**(1)、普通/编辑模式**
vi +10myfile  打开文件定位到第10行
vi +myfile  打开文件定位到最后一行

**字符位置移动光标：**  
h : 左   j : 下  k : 上  l(L小写): 下
**单词移动光标：**
w : 移至下一个单词的词首
e : 跳至当前或下一个单词的词尾
b : 跳至当前或前一个单词的词首
**行内移动光标：**
0 : 绝对行首
^ : 行首的第一个非空白字符
\$ : 绝对行尾
**行间移动光标：**
G : 文章末尾
3G : 第3行
gg : 文章开头
**翻屏：**
ctrl + f  : 往下翻页
ctrl + b : 往上翻页

**删除字符&替换单个字符：**
x : 删除光标位置的字符
3x : 删除光标位置往前的3个字符
r : 替换光标位置字符
**删除行和单词：**
dd :  删除一行
dw :  删除一个单词
**复制粘贴&剪切：**
yw : 复制一个单词
yy : 复制一行，光标所在行
3yy : 复制三行，从光标所在行算起
y\$ : 从光标复制到这一行行尾
dd : 剪切一行，光标所在行
d\$ : 光标处开始剪切到这一行行尾
p : 光标下行粘贴（如果不是整行比如y\$这种，就在光标后面粘贴）
P  : 光标上行粘贴 （如果不是整行比如y\$这种，就在光标前面粘贴）

**撤销&重做**
u : 撤销
ctrl + r : 恢复
.  : 重复上一步操作

**(2)、模式切换 （普通/编辑模式——插入/输入模式——末行/命令模式）**
i : 在光标所在字符的前面进入输入模式
I : 在光标所在行的行首进入输入模式
a : 在光标所在字符的后面进入输入模式
A : 在光标所在行的行尾进入输入模式
o : 在光标所在行的下一行进入输入模式
O : 在光标所在行的上一行进入输入模式

**esc : 从输入模式退出到编辑模式**
**:冒号 进入末行模式，从编辑模式才能进入**
:q 退出，没有修改文件
:wq  保存退出  shift + zz
:q! 不保存并退出
:w 保存， 可以:w /root/xx.txt 保存到某个文件
:w! 强行保存

**(3)、末行/命令模式**

**set设置** 
:set nu  设置行号
:set nonu 取消行号
:set readonly 设置文件只读

**如果需要每次打开vim时，set都是默认生效的，需要修改vim配置 /etc/vimrc**，如：在文件末尾写 set nu

**在末行模式下执行命令！！！ 可以在编辑时同时执行命令，非常有用**
:! ls -l  （:! 后面接命令） 

**查找功能**
/xxx  向下查询xxx
?xxx 向上查询xxx
n 找寻下一个xxx
N 找寻上一个xxx

**查找并替换**
:s/str1/str2/gi
**g 表示一行内全部替换，将 str1 替换 成 str2**
**i 表示忽略大小写**

/ 为间隔符，还可以为@ #这些
*还可以使用范围，在s前面添加范围：*
n 行号，如:2s/str1/str2/gi
. 当前光标行，:.s/str1/str2/gi
+n 偏移n行，:+10s/str1/str2/gi
\$ 末尾行，  \$-2 倒数第三行，:\$s/str1/str1/gi  
% 全文，:%s/str1/str2/gi
n,m n-m行，:0,\$s/str1/str2/gi

**删除**
:0,\$d  删除全部内容
:.,+2d 删除前三行(光标必须在第一行)
:1,3d 删除1-3行
:\$-1d 删除倒数第二行
:1,\$-1d 只保留最后一行

**复制**
:1,3y 复制1-3行 ， p粘贴

**(4)、可视模式**
大量重复操作时可一次行完成。
v 字符可视
V 行可视
ctrl + v 块可视，通常配合d 和 I（大写i）进行批量删除和修改操作

#####五、正则表达式
正则表达式是对字符串操作的一种逻辑公式，就是用事先定义好的一些特定字符及这些特定字符组合，组成一个规则字符串，这个“规则字符串”用来表达对字符串对一种过滤逻辑。
**(1)、匹配操作符**
\   转义字符
.   匹配任意单个字符
[12345a]   匹配其中一个,  [1-9] 匹配1-9中的一个 , [^12] 匹配非12中的
^   行首
$   行尾
\<,  \>  单词首尾边界，如\<abc\> 匹配单词abc
|   连接操作符 或
()  选择操作符 
\n  反向引用, 如  ([1-9]{2})(.*[a-z])\1\2 ，其中\1就表示([1-9]{2}) ,而\2表示(.*[a-z])


**(2)、重复操作符**
?  匹配0到1次
\*  匹配0到多次
\+  匹配1到多次
{n} 匹配n次
{n,} 匹配n到多次
{n,m} 匹配n到m次

**在文件中查询符合要求的行:**
```
grep "regex" file
特殊符号需要转义
```
#####六、文本分析
**(1)、cut**

**(2)、sort**

**(3)、wc**

**(4)、sed 行编辑器**

**(5)、awk文本分析**

#####七、用户与权限

**如何查看有哪些用户**
在/etc/passwd  和 /etc/shadow中可以查看到

**验证有没有用户**
id  yonghu 

**用户创建和删除**
useradd  yonghu  // 创建用户，也会默认创建一个名为yonghu的组
userdel yonghu  // 删除用户，还可以添加 -r 选项，能够删除用户数据
*// 删除用户时，附加信息也要删除，添加了-r 选项删除时可能不需要下面到删除*
rm -rf /home/yonghu
rm -rf /var/spool/mail/yonghu

passwd yonghu  // 给用户设置密码，直接passwd就是设置当前账号到密码
usermod  // 修改用户属性，如用户到家目录
chage  // 修改用户属性，主要修改用户到生命周期

**用户组创建和删除**
groupadd  // 创建组织
groupdel  // 删除组织
usermod -G groupname yonghu  // 把用户加到groupname组织中

**用户切换**
sudo  以自己的身份运行其他用户可以执行的命令，不需要其他用户的密码，避免了密码泄漏，例如，普通用户临时获取root权限执行命令，需要使用visudo配置使用sudo的用户(组)
```
## Read drop-in files from /etc/sudoers.d (the # here does not mean a comment)
#includedir /etc/sudoers.dllow root to run any commands anywhere
root    ALL=(ALL)       ALL

## Allows members of the 'sys' group to run networking, software,
## service management apps and more.
# %sys ALL = NETWORKING, SOFTWARE, SERVICES, STORAGE, DELEGATING, PROCESSES, LOCATE, DRIVERS

## Allows people in group wheel to run all commands
# %wheel        ALL=(ALL)       ALL

## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL

## Allows members of the users group to mount and unmount the
## cdrom as root
# %users  ALL=/sbin/mount /mnt/cdrom, /sbin/umount /mnt/cdrom

## Allows members of the users group to shutdown this system
# %users  localhost=/sbin/shutdown -h now

## Read drop-in files from /etc/sudoers.d (the # here does not mean a comment)
#includedir /etc/sudoers.d
```
su  su - username 使用login shell 方式切换用户(带 - 切换后进去的是自己的家目录)

**用户和用户组的配置文件**
/etc/passwd 保存用户信息
/etc/shadow  保存用户密码信息
/etc/group 保存组信息

**修改用权限 （注意：root用户不受这些权限控制)**
chown  可以修改属主和属组，如：chown yonghu /test，将test目录属主变成yonghu； chown :group /test 属组修改成功group，注意冒号不能少
chgrp  单独改组，不常用 chown代替了，例如 chgrp  group /test
chmod u=rwx xxx  属主可读可写可执行
chmod g+w xxxxx 属组增加写权限
chmod o-x xxxxx  其它删除执行权限
chmod u+x xxxx  // 属主增加执行权限
**数字方式设置**
r = 4 w = 2 x = 1
chmod 777 xxxx  // 属主、属组和其它可读可写可执行

**特殊权限**
了解即可，SUID SGID SBIT


#####八、软件安装与卸载
**(1)、rpm 安装 和 卸载**
rpm -ivh filename // 安装  --prefix path // 指定目录安装
```
rpm -ivh --prefix /usr/ceshi  xxxx.rpm
```

rpm -e 包名 // 卸载
rpm -qa 查看已经安装的包
rpm -q 包名 // 查看包有没有安装
rpm -qi 包名 // 查询指定包的说明信息
rpm -ql 包名 // 查询指定包安装后生成的文件列表
rpm -qc 包名 // 查询指定包安装的配置文档
rpm -qd 包名 // 查询指定包安装的帮助文档
rpm -q --scripts // 查询指定包中包含的脚本
rpm -qa | grep java | xargs rpm -e --nodeps
rpm -qf /path/to/somefile  // 查询指定文件由哪个安装包产生
```
rpm -qf /sbin/ifconfig
```

**(2)、yum安装与配置**
yum( yellow dog updater,Modified) 是一个shell前端软件包管理器，基于rpm包管理，能够从**指定的服务器**自动下载rpm包并且安装，可以自动处理依赖关系，并且一次性安装所有依赖的软件包，无须繁琐地一次次下载、安装。
**配置yum源：**
- 修改配置文件(因为国外的源很慢)
/etc/yum.repos.d/ 目录下下载国内的yum源，比如，阿里或163的yum源
- yum clean all 清空本地的依赖缓存
- yum makecache 将依赖缓存下载到本地

**yum命令常用选项：**
- install 安装软件包 yum install xxx
- remove 卸载软件包 yum remove xxx
- list|grouplist 查看软件包 yum list
- update 升级软件包 yum update (可带软件包名，只升级具体的软件包，不带默认检查所有)

**(3)、其它方式安装**
- 二进制安装
- 源代码编译安装 ，有时yum仓库里面没有最新的包，这时候就需要编译源码安装了。
下载软件包，wget 资源地址

**(4)、升级内核**


配置本地yum源：


#####九、系统管理
**1、网络管理**
**(1)、网络状态查看**
**网络状态查看工具**，常用两种net-tools和iproute
**net-tools**
- 查网卡信息，命令 ifconfig  (普通用户执行命令 /sbin/ifconfig)，ifconfig eth0 查看具体网卡信息。
   eth0 第一块网卡名称（网络接口），第一个网络接口还有可能是以下名字：
    eno1 板载网卡
    ens33 PCI-E网卡
    enp0s3 无法获取物理信息的PCI-E 网卡
    *CentOS 7 使用了一致网络设备命名，以上都不匹配则使用 eth0。*
网络接口名称修改，网卡命名规则受biosdevname和net.ifnames两个参数影响，如果想网卡命名为eth0，可以编辑 /etc/default/grub文件，增加biosdevname=0 net.ifnames=0，更新grub，命令 grub2-mkconfig -o /boot/grub2/grub.cfg，最后重启，命令 reboot。

-  查看网卡物流链接情况，命令 mii-tool  eth0(网卡名称)  
- 查看网关信息，命令 route ，route -n，使用 -n参数不解析主机名，因为ip解析成主机名会浪费时间，正常可以不解析主机名。
- netstat

**iproute2**
-ip
-ss

**(2)、网络配置**
- 设置网卡信息，ifconfig <接口> <IP地址>[netmask 子网掩码]  使用：ifconfig eth0 192.168.152.11 netmask 255.255.255.0 
- 启用网卡，ifup <接口> 启用网卡
- 停用网卡，ifdown <接口> 停用网卡
- 添加网关，使用route命令
route add default gw <网关ip>，设置默认网关
route add -host <指定ip> gw <网关ip>，设置访问具体主机时走指定网关
route add -net <指定网段> netmask <子网掩码> gw <网关ip> 设置特定网段的访问走指定的网关
**网关删除，route del default gw <网关ip> 删除默认网关**

**(3)、网络故障排除**
- ping 检查当前主机和目标主机是否畅通
- traceroute 追踪当前主机和目标主机之间到每一跳
- mtr 检查当前主机到目标主机访问时是否有包丢失
- nslookup 查看域名对应到ip
- telnet 检查目标主机的端口是否畅通  telnet www.baidu.com 80
- tcpdump  网络抓包工具，分析数据包
- netstat  查看服务的监听地址和端口，netstat -ntpl (n不要解析成主机名，t 只找tcp协议的，p进程，l listen监听)
- ss 

**(4)、网络服务管理和配置文件**

**我们正常设置的配置，linux重启完后配置就又还原了，要想每次启动都使用新的配置，只有修改linux配置文件**

**网络服务管理程序分为两种，分别为SysV和systemd(CentOS 7以上版本)**
SysV基于network 服务，有如下：
- service network start|stop|restart 网络服务的启动、终止和重启
service network status 查看当前网络服务状态
- chkconfig --list network 查看network服务的设置

**一般情况下只使用一种网络服务，使用一个的同时禁止另一个，正常使用network服务，不启用NetworkManager服务**
systemd基于NetworkManager服务，有如下：
- systemctl list-unit-files NetworkManager.service 查看NetworkManager 服务
- systemctl start|stop|restart NetworkManager 
- systemctl enable|disable NetworkManager 开启或禁用NetworkManager服务

**配置文件**
- ifcfg-eth0 网卡配置文件，eth0为网卡名称，文件路径 /etc/sysconfig/network-scripts/
- /etc/hosts 主机名相关配置，hostname 可以查看当前主机名，hostname  xxx 设置主机名重启后还原。
永久生效，可以使用hostnamectl set-hostname xxx
修改了主机名后，将主机名和ip关系写到/etc/hosts文件中，127.0.0.1 xxx





















**3、bash shell中变量定义及进程简单管理**
bash是个软件，能够帮我们进行linux内核调用。
```
//普通变量
a=1
b=xxx
echo $a // 检查

//集合
arr=(1 2 3)  // 括号不能有空格，里面元素空格隔开
echo $arr // 输出第一个元素
echo ${arr[1]} // 输出第二个元素
echo ${arr[2]} // 输出第三个元素

//追加信息
echo ~$b~
~xxx~ 输出

//打印当前bash的端口号
echo $$

//查看进程和进程号
ps -ef

//杀死进程
kill -9 进程号

```
**4、hash优化**
hash 优化命令的查询时间，原本每次执行命令时都去PATH配置的目录寻找执行文件，如果命令执行完成后，执行hash命令，则会将命令执行文件所在位置缓存起来，下次执行时不用再到PATH配置的目录中寻找。
```
ps -ef
...
hash
// 显示
hits   command
1      /bin/ps

//删除hash缓存
hash -r

```

#####十、shell篇

#####十一、服务器管理
