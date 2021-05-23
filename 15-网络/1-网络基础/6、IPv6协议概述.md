#### IPv6协议

##### 一、IPv6出现的背景

IPv4只能支持43亿设备，不够用。目前IPv4是使用的拆分子网：

![image-20210523171452329](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523171452329.png)

##### 二、IPv6的工作原理

IPv6和IPv4工作原理相似，为切片、增加封包头、路由(寻址)几个阶段。

![image-20210523171629987](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523171629987.png)

##### 三、和IPv4的主要区别

###### 1、地址

- 地址数量：IPv4有4个8位，共16位，IPv6有8个16位，共128位
- 分割符号：
  - IPv4的地址用 . 分割，如 103.28.7.35 。每一个是8位，用0-255的数字表示。
  - IPv6的地址用 : 分割，如 0123:4567:89ab:cdef:0123:4567:89ab:cdef 。每个是一个16位的16进制数字，就是4个符。

![image-20210523171922833](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523171922833.png)

- 书写方式：IPv6地址可以简写

![image-20210523172042677](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523172042677.png)

![image-20210523172105858](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523172105858.png)

###### 2、寻址

**（1）全局单播**

- 站点前缀（Site Prefix)：48bit，一般是由ISP（Internet Service Providor，运营商）或者RIR(Regional Internet Registry， 地区性互联网注册机构)。RIR将IP地址分配给运营商。
- 子网号（Subnet ID)：16bit，用于站点内部区分子网。
- 接口号（Interface ID)：64bit，用于站点内部区分设备。

![image-20210523172559190](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523172559190.png)

全局单播地址例子：

![image-20210523172700514](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523172700514.png)

**（2）本地单播**

定义：给定地址，本地网定位设备

![image-20210523172752663](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523172752663.png)

例子：fe80::123e:456d

注意：Link-local必须以fe80开头

**（3）分组多播**

- 需要以8个1，也就是 ff00 开头，后面跟上一个分组的编号。 
- 所在的网络中已经定义了该分组编号，而且有设备可以识别这个编号。
- 拥有分组下设备的完整清单，并把数据发送给对应的设备们。
- IPv4也支持分组多播，但需要网络配置整体配合。

###### 3、新设备的接入

新设备接入IPv6后，会使用IPv6的邻居发现协议(Neighbour Discover Protocol)为自己申请一个IP地址。当新设备需要发送信息到目的地时，还可以通过ND协议广播查询目标设备。然后如果需要路由，还可以通过ND查找路由器。

![image-20210523173148545](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523173148545.png)

传统IPv4：使用ARP协议（Address Resolution Protocol，地址解析协议）。每个节点存储许多额外信息。

IPv6：更加无状态化，减少数据冗余带来的风险和负担，自动分配地址，做到了无状态接入设备。

