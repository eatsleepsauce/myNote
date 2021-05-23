#### UDP协议

##### 一、UDP协议背景及概念

###### 1、背景

- 1980年由科学家David P. Reed提出
- 协议简单，搭建在IP协议之上
- 尽可能的减少通信机制，速度非常快
- 该协议的RFC只有两页

###### 2、UDP协议介绍

- 全称:  User Datagram Protocol，用户数据报协议
- 定义：在传输层提供直接发送报文(Datagram)的能力。Datagram是数据传输的最小单位。
- 目标：发送报文，无法拆分数据

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523203137391.png" alt="image-20210523203137391" style="zoom:50%;" />

##### 二、UDP的封包格式

设计目标：允许用户直接发送报文的情况下最大限度的简化应用的设计

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523203307324.png" alt="image-20210523203307324" style="zoom:50%;" />

##### 三、UDP和TCP的区别

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523203357313.png" alt="image-20210523203357313" style="zoom:50%;" />

##### 四、场景分析

###### 1、聊天室适合UDP么

不合适，聊天室并发高，单流量不大，依赖可靠性。

###### 2、HTTP 协议适不适合UDP

合适，HTTP 3.0 就是建立在UDP上的，应用层控制

