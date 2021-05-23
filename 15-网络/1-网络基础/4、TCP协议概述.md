#### TCP协议

##### 一、概述

TCP全名是(Transport Control Protocol)，是一个可以提供可靠的、支持全双工、连接导向的协议，因此在客户端和服务端之间传输数据的时候，是必须先建立一个连接的。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523155811815.png" alt="image-20210523155811815" style="zoom:50%;" />

###### 1、什么连接

- 是虚拟、抽象的概念
- 能让两个通信的程序间确保彼此都在线
- 加快响应请求速度
- 连接也被称为会话(Session）
- 使通信更稳定、安全
- 消耗更多资源

###### 2、什么是全双工

- 单工：任何时刻数据只能单向发送
- 半双工：允许数据在两个方向上传输，在某一时刻，只允许数据在一个方向上传输
- 全双工：任何时刻都能双向发送数据

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523160137432.png" alt="image-20210523160137432" style="zoom:50%;" />

###### 3、如何保证可靠

可靠性指数据保证无损传输。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523160301025.png" alt="image-20210523160301025" style="zoom:50%;" />

##### 二、TCP协议的工作过程

###### 1、建立连接的工程（三次握手）

- 客户端发送SYN
  - 服务端准备好进行连接
- 服务端针对客户端的SYN给ACK，服务端发送SYN。
- 客服端转变就绪，客户端发送ACK

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523160420800.png" alt="image-20210523160420800" style="zoom:50%;" />

###### 2、断开连接的过程（四次挥手）

- 客户端发送断开请求 FIN
- 服务端收到请求，发送ACK
- 服务端经过一个等待（自己的事情处理完），确定可以关闭连接，发送FIN
- 客户端收到FIN，处理完自己的事情后发送ACK

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523161617152.png" alt="image-20210523161617152" style="zoom:50%;" />

###### 3、数据传输

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523161757661.png" alt="image-20210523161757661" style="zoom:50%;" />

**（1）报文拆分**

- 应用层数据很大时无法一次性传输完
- 拆分后可实现并行传输

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523161921992.png" alt="image-20210523161921992" style="zoom:50%;" />

**（2）顺序保证**

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523162032205.png" alt="image-20210523162032205" style="zoom:50%;" />

- TCP序号：发送序号(Seq)、接收序号(Ack)
- 一个端的发送序号是另一个端的接受序号

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523162153415.png" alt="image-20210523162153415" style="zoom:50%;" />

+1 是约定，正常seq加的是上次传输的大小，ack 是加收到的大小

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202021-05-23%20%E4%B8%8B%E5%8D%8810.44.58.png" alt="屏幕快照 2021-05-23 下午10.44.58" style="zoom:50%;" />

**（3）TCP头**

- 源端口：描述发送方机器上的应用
- 目标端口：描述接收方服务器上的应用
- 发送序号(Seq)/接收序号(Ack)

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523162354936.png" alt="image-20210523162354936" style="zoom:50%;" />

TCP头标志位：

- NS、CWR、ECE：TCP扩展协议
- ECN：显示拥塞控制协议，有助于帮助解决延迟和丢包问题
- URG：紧急标志位
- SYN（Synchronize Sequence Numbers)：同步序号，也就是在建立连接
- FIN： 终止连接
- ACK（Achnowledgment)：响应
- PSH（push）：传送数据
- RST（Reset Connection)： 重置连接

##### 三、TCP协议周边配置

###### 1、纠错能力

保证数据可靠性

- TCP拥有一个16bit的Checksum字段
- Checksum是一个函数，把原文映射到一个不可逆的16bit编码中，这样就可以知道原文在传输过程中有没有发生变化

###### 2、流控能力

协同两边速率，保证可靠性

- 主要目标：让发送方和接收方协商一个合理的收发速率，让两边都可以稳定的工作。
- 利用滑动窗口

###### 3、拥塞控制能力

确定网络的拥堵情况决定传输速率