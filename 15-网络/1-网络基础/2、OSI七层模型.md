#### OSI七层模型

##### 一、OSI的产生背景

OSI模型指的是Open System Interconnection Reference Model，即开放式系统互联模型。它是世界上第一个试图在世界范围内规范网络标准的框架。

基础建设在学术界早已成型，如封包交换原理理论，数据传输能力等。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523152007519.png" alt="image-20210523152007519" style="zoom: 50%;" />

##### 二、七层模型详解

###### 1、应用层(Application Layer)

- 应用层位于OSI模型最上方
- 只关心业务逻辑，不关心数据的传输

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523152359830.png" alt="image-20210523152359830" style="zoom:50%;" />

###### 2、表示层(Presentation Layer)

负责协商用于传输的数据格式，并转换数据格式

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523152546382.png" alt="image-20210523152546382" style="zoom:50%;" />

###### 3、会话层(Session Layer)

- 负责管理两个连接网实体间的连接
- 功能及特点：建立连接，维持通信，释放连接

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523152738465.png" alt="image-20210523152738465" style="zoom:50%;" />

###### 4、传输层(Transport Layer)

负责将数据从一个实体(一个服务或应用)传输到另一个实体，但不负责数据传输到方式。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523152923446.png" alt="image-20210523152923446" style="zoom:50%;" />

传输层的能力：

- 数据分隔重组：将数据拆分后按顺序重组
- 纠错：在数据传输过程中出现问题后采取方式进行纠正
- 管理连接：处理数据的频繁交换
- 流量控制：控制传输数据的速率
- 端口寻址：标明参与传输的实体的端口号

###### 5、网络层(Network Layer)

负责把一个封包从一个IP地址传输到另一个IP地址

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523153344993.png" alt="image-20210523153344993" style="zoom:50%;" />

###### 6、数据链路层(Data Link Layer)

- 确保两个临近设备间数据的传输，并隐藏底层实现
- 帧同步：两个设备之间传输时的协商速率问题
- 数据纠错

###### 7、物理层(Physical Layer)

- 封装和隐藏具体的传输手段，并且提供稳定的传输接口
- 比如：电缆、光纤、蓝牙等

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523153616333.png" alt="image-20210523153616333" style="zoom:50%;" />

##### 三、OSI的问题

分层设计较为臃肿，并发每一层都是必要的。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523153735786.png" alt="image-20210523153735786" style="zoom:50%;" />



