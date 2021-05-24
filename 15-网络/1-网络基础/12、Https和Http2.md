#### HTTPS和HTTP2.0

##### 一、为什么需要HTTPS

HTTPS（HTTP Over SecureSocket Layer）可以保证网络传输环境的安全。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210524205211591.png" alt="image-20210524205211591" style="zoom: 25%;" />

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210524205457786.png" alt="image-20210524205457786" style="zoom: 25%;" />

SSL逐渐被TSL替代。

##### 二、HTTPS工作原理

HTTPS采用对称加密的方式加密传输的数据，而对称加密的密钥是采用非对称加密的方式协商的。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210524210319496.png" alt="image-20210524210319496" style="zoom: 25%;" />

- TCP协议三次握手，建立TCP连接
- 服务器利用TCP将证书发送给浏览器
- 浏览器通过本地Root CA验证网站证书
- 浏览器用证书的公钥加密：协商对称加密的算法和密钥
- 服务器响应，确定对称加密算法和密钥
- 建立会话（往来数据使用对称加密）

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210524212303694.png" alt="image-20210524212303694" style="zoom:50%;" />

##### 三、HTTP2.0

HTTPS主要解决HTTP的安全问题，而HTTP2.0主要解决的是性能问题。

HTTP1.1 keep-alive的问题：

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202021-05-24%20%E4%B8%8B%E5%8D%889.45.41.png" alt="屏幕快照 2021-05-24 下午9.45.41" style="zoom:50%;" />

HTTP2.0不阻塞。

HTTP2.0进行了头部压缩，使用数字代表特定的头。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210524215230052.png" alt="image-20210524215230052" style="zoom:50%;" />



