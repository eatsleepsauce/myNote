#### IP协议

##### 一、IP协议概述

###### 1、基本概念

IP协议(Internet Protocol)：网络层协议。

![image-20210523163919640](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523163919640.png)

###### 2、IP协议可能遇到的问题

-  封包损坏
- 丢包
- 重发
- 乱序

###### 3、网络层需要解决的3个问题

- 延迟
- 吞吐量
- 丢包率

###### 4、IP协议架构

IP协议目前主要有两种架构，IPv4 和 IPv6。

##### 二、IP协议的工作原理

![image-20210523164520994](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523164520994.png)

###### 1、分片

- 把数据切分成片（Fragmentation）
- 适配底层传输网络

![image-20210523164734909](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523164734909.png)

###### 2、增加协议头

![image-20210523164948789](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523164948789.png)

- Type Of Service：服务的类型，是为了响应不同的用户诉求，用来选择延迟、吞吐量和丢包率之间的关系。
- IHL（Internet Header Length)：IP协议头的大小。
- Total Length：报文(封包datagram)的长度
- Identification：报文的ID，发送方分配，代表顺序
- Fragment offset：描述是否要分包（拆分），和如何拆分
- Time To Live：封包存活的时间
- Protocol：描述上层的协议，比如TCP=6,UDP=17
- Options：可选项
- Checksum：检验封包的正确性

###### 3、延迟、吞吐量、丢包率

- 延迟——1bit的数据从网络的1个终端传送到另一个终端需要的时间
- 吞吐量——单位时间内可以传输的平均数据量
- 丢包率——发送出去的封包没有到达目的地的比例

注意：**这三个无法同时保证**

###### 4、Type of Service字段

![image-20210523165844714](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523165844714.png)

###### 5、寻址过程

- 寻址：给一个地址，然后找到这个东西
- IPv4地址(32位)：逐级寻址

![image-20210523170113293](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523170113293.png)

例如：103.16.3.1 

**（1）找到顶层网络**

103.16.3.1 最顶层的网络号和 255.0.0.0 （子网掩码）做位与运算得到：103.16.3.1 & 255.0.0.0 = 103.0.0.0（顶层网络）

**（2）找到下一层网络**

用IP地址和下一级的子网掩码做位与：103.16.3.1 & 255.255.0.0 = 103.16.0.0（下一级网络）

**（3）再到下一级网络**

使用 255.255.255.0 子网掩码找到下一级网络：103.16.3.1 & 255.255.255.0 = 103.16.3.0

**（4）定位设备**

设备就在子网 103.16.3.0 中；最终找到的设备号是 1

注意：这是示例，实际中子网掩码不一定都是255

###### 6、路由

- 若寻找的IP地址不在局域网中，需要路由找到去往对应网络的路径。
- IP地址和子网掩码位与的过程是由路由算法实现的。

![image-20210523170824138](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523170824138.png)



