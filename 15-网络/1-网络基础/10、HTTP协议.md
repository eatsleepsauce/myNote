#### HTTP协议

##### 一、HTTP概述

- HTTP协议(Hyper Text Transfer Protocol): 应用层协议
- 目标: 是处理客户端和服务端之间的通信

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523225711713.png" alt="image-20210523225711713" style="zoom: 33%;" />

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523225841396.png" alt="image-20210523225841396" style="zoom: 33%;" />



###### 1、请求

一次请求，分成头(Header)和体(Body)。 下面是一个请求头+消息体的示例：

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523225922443.png" alt="image-20210523225922443" style="zoom: 33%;" />

###### 2、返回

一次返回，也同样分(Header)和体(Body)。 下面是一个返回头+消息体的示例：

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523230001672.png" alt="image-20210523230001672" style="zoom: 33%;" />

##### 二、请求头和返回头

HTTP协议通过请求头和返回头控制协议工作。无论请求头还是返回头都是Key/Value的形式。

常见头：

###### 1、Content-Length 发送/接收Body内容的字节数

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523230445810.png" alt="image-20210523230445810" style="zoom:50%;" />

###### 2、User-Agent

这个字段可以帮助统计客户端用了什么浏览器、操作系统等 

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523230541075.png" alt="image-20210523230541075" style="zoom:50%;" />

###### 3、Content-Type

请求的时候，告知服务端数据的媒体类（MediaType/MIME Type)。返回的时候告知客户端，数据的媒体类型。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523230633991.png" alt="image-20210523230633991" style="zoom:50%;" />

###### 4、Origin

描述请求来源地址

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523230728430.png" alt="image-20210523230728430" style="zoom:50%;" />

###### 5、Accept

是HTTP协议协商能力的体现，用于建议服务端返回何种媒体类型(MIME Type)

- */*代表所有类型(默认)
- 多个类型用逗号隔开例如：text/html, application/json
- Accept-Encoding：建议服务端发送哪种编码（压缩算法）
- deflate, gzip;q=1.0, *;q=0.5
- Accept-Language：建议服务端传递哪种语言
- Accept-Language：fr-CH, fr;q=0.9, en;q=0.8, de;q=0.7, *;q=0.5

###### 6、Refer

- 告诉服务端打开当前页面的上一张页面的URL
- 非浏览器环境有时候不发送Referer（或者虚拟Referer,通常是爬虫)
- 常用于用户行为分析

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523231049614.png" alt="image-20210523231049614" style="zoom:50%;" />

###### 7、Connection

决定HTTP连接（不是TCP连接）是否在当前事务完成后关闭。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523231214545.png" alt="image-20210523231214545" style="zoom:50%;" />

解释：

##### 三、HTTP的方法

- GET：从服务器获取资源
- POST：在服务器创建资源
- PUT：在服务器修改资源
- DELETE：在服务器删除资源
- OPTION ：CORS跨域重点说明
- TRACE ：用于显示调试信息 多数网站不支持
- CONNECT： 代理部分讲解
- PATCH ：对资源进行部分更新(极少用)

##### 四、状态码

- 1xx：提供信息 
- 100 continue 101 切换协议(switch protocol) 
- 2xx：成功 
- 3xx：重定向 
- 4xx：客户端错误 
- 5xx：服务端错误

###### 1、2xx状态码  （成功）

- 200 – OK 
- 201 – Created 已创建 
- 202 – Accepted 已接收 
- 203 – Non-Authoritative Information 非权威内容 
- 204 – No Content 没有内容 
- 205 – Reset Content 重置内容 
- 206 – Partial Content 服务器下发了部分内容(range header) 

###### 2、3xx状态码（重定向）

- 300 – Multiple Choices 用户请求了多个选项的资源（返回选项列表）
- 301 – Moved Permanently 永久转移
  - 资源被永久移动到新的地址
  - 客户端收到301请求后，通常用户会向新地址发起GET请求
- 302 – Found 资源被找到（以前是临时转移）
  - 资源临时放到新地址
  - 302是http1.0提出的，最早叫做Moved Temporarily； 很多浏览器实现的时候没有遵照标准，把所有请求都重定向为GET
  - 1999年标准委员会增加了303和307，并将302重新定义为Found。
- 303 – See Other 可以使用GET方法在另一个URL找到资源
  - 资源临时放到新地址
  - 303告诉客户端使用GET方法重定向资源
- 304 – Not Modified 没有修改（缓存部分特别说明）
- 305 – Use Proxy 需要代理
- 307 – Temporary Redirect 临时重定向
  - 资源临时放到新地址
  - 307告诉客户端使用原请求的method重定向资源
- 308 – Permanent Redirect 永久重定向 
  - 资源被永久移动到新的地址
  - 客户端收到308请求后，延用旧的method(POST/GET/PUT)到新地址

###### 3、4xx状态码（客户端问题）

- 400 – Bad Request 请求格式错误
- 401 – Unauthorized 没有授权
- 402 – Payment Required 请先付费
- 403 – Forbidden 禁止访问
- 404 – Not Found 没有找到
- 405 – Method Not Allowed 方法不被允许
- 406 – Not Acceptable 服务端可以提供的内容和客户端期待的不一样

###### 4、5xx状态码

- 500 – Internal Server Error(内部服务器错误)
- 501 – Not Implemented（没有实现)
- 502 – Bad Gateway(网关错误，主要是指网关无法到达服务器）
- 503 – Service Unavailable(服务不可用，比如dos、ddos，服务器无法提供服务，资源被耗尽了。)
- 504 – Gateway Timeout(网关超时)
- 505 – HTTP Version Not Supported（版本不支持)



