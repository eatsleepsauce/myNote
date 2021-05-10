#### 软件安装与卸载

**软件包管理器** 是方便 **软件安装** 、**卸载** 和 **解决软件依赖关系**  的重要工具。

- CentOS、RedHat 使⽤ yum 包管理器，软件安装包格式为 rpm。
- Debian、Ubuntu 使⽤ apt 包管理器，软件安装包格式为 deb。

##### 一、rpm 安装 和 卸载

vim-common-7.4.10-5.el7.x86_64.rpm 

软件名称 软件版本 系统版本 平台

- rpm -ivh filename // 安装  --prefix path // 指定目录安装

```shell
rpm -ivh --prefix /usr/ceshi  xxxx.rpm
// -i 安装软件包
```

- rpm -e 包名，卸载软件包
- rpm -q 包名 ，查看包有没有安装
- rpm -qa 查看已经安装的包
- rpm -qi 包名 ，查询指定包的说明信息
- rpm -ql 包名， 查询指定包安装后生成的文件列表
- rpm -qc 包名，查询指定包安装的配置文档
- rpm -qd 包名，查询指定包安装的帮助文档
- rpm -q --scripts // 查询指定包中包含的脚本
- rpm -qa | grep java | xargs rpm -e --nodeps
- rpm -qf /path/to/somefile  // 查询指定文件由哪个安装包产生

```shell
rpm -qf /sbin/ifconfig
```

rpm 包的问题：需要自己解决依赖关系，软件包来源不可靠。

##### 二、yum安装与配置

**yum**( yellow dog updater,Modified) 是一个shell前端软件包管理器，基于rpm包管理，能够从**指定的服务器**自动下载rpm包并且安装，可以自动处理依赖关系，并且一次性安装所有依赖的软件包，无须繁琐地一次次下载、安装。

###### 1、配置yum源

CentOS yum 源  http://mirror.centos.org/centos/7/

国内镜像， https://opsx.alibaba.com/mirror

- 修改配置文件(因为国外的源很慢)
  /etc/yum.repos.d/ 目录下下载国内的yum源，比如，阿里或163的yum源
  
  ```shell
  wget -O /etc/yum.repos.d/CentOS-Base.repo  http://mirrors.aliyun.com/repo/Centos-7.repo
  ```
  
- yum clean all 清空本地的依赖缓存

- yum makecache 将依赖缓存下载到本地

###### 2、yum命令常用选项

- install 安装软件包 yum install xxx
- remove 卸载软件包 yum remove xxx
- list|grouplist 查看软件包 yum list
- update 升级软件包 yum update (可带软件包名，只升级具体的软件包，不带默认检查所有)

##### 三、其他方式安装

###### 1、二进制安装

###### 2、源代码编译安装

有时yum仓库里面没有最新的包，这时候就需要编译源码安装了。下载软件包，wget 资源地址。ps：

- wget https://openresty.org/download/openresty-1.15.8.1.tar.gz
- tar -zxf openresty-VERSION.tar.gz
- cd openresty-VERSION/ 
- ./configure --prefix=/usr/local/openresty
- make -j2 
- make install

##### 四、内核升级

###### 1、rpm格式内核

- 查看内核版本  uname -r
- 升级内核版本 yum install kernel-3.10.0
- 升级已安装的其他软件包和补丁 yum update

###### 2、源代码编译安装内核

- 安装依赖包
  - yum install gcc gcc-c++ make ncurses-devel openssl-devel elfutils-libelf-devel

- 下载并解压内核
  - https://www.kernel.org
  - tar xvf linux-5.1.10.tar.xz -C /usr/src/kernels

- 配置内核编译参数
  - cd /usr/src/kernels/linux-5.1.10/
  - make menuconfig | allyesconfig | allnoconfig

- 使用当前系统内核配置

  - cp /boot/config-kernelversion.platform /usr/src/kernels/

    linux-5.1.10/.config 

- 查看CPU
  - lscpu
- 编译
  - make -j2 all
- 安装内核
  - make modules_install
  - make install

##### 五、Grub配置文件

###### 1、grub是什么

GRUB是GRand Unified Bootloader的缩写，它是一个多重操作系统启动管理器。用来引导不同系统，如[windows](https://baike.baidu.com/item/windows/165458)，[linux](https://baike.baidu.com/item/linux/27050)。

GRUB是多启动规范的实现，它允许用户可以在计算机内同时拥有多个操作系统，并在计算机启动时选择希望运行的操作系统。

###### 2、grub配置文件

- /etc/default/grub 默认配置文件
- /etc/grub.d/ 更详细的配置目录，里面有很多配置文件
- /boot/grub2/grub.cfg
- Grub2-mkconfig -o /boot/grub.cfg

###### 3、使用单用户进入系统(忘记root密码)

