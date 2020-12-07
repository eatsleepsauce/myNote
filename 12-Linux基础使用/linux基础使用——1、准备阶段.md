#### 一、准备篇

##### 1、虚拟机及系统安装

安装VMware Workstation，安装完成后在安装linux系统之前进入系统的bios设置保证虚拟化的能力是被打开的。
配置虚拟机的硬件能力：cpu、内存、硬盘、网络NAT模式等
安装操作系统，注意设置三个区：
（1）boot引导程序区 
（2）swap交换区——设置为虚拟机内存的两倍大小
（3）用户区——虚拟机的剩的最大空间

##### 2、网络配置

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
##### 3、基于创建好的虚拟机克隆出多台虚拟机

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
##### 4、Xshell、Xftp使用

ssh root@xxx.xxx.xxx.xxx
