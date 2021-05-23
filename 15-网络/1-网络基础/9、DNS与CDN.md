#### DNS与CDN

##### 一、DNS基本知识

###### 1、统一资源定位符(URL)

也被称作「网址」，用于定为互联网上的资源

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523214255724.png" alt="image-20210523214255724" style="zoom:50%;" />

###### 2、DNS(Domain Name System)域名解析系统

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523214517717.png" alt="image-20210523214517717" style="zoom:50%;" />

###### 3、DNS Query原理（过程）

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523214557049.png" alt="image-20210523214557049" style="zoom:50%;" />

- 先查询浏览器的本地缓存中查询（内存中）
- 本地没有缓存，查找操作系统的hosts文件，该文件在linux中的/etc/hosts里
- 上述步骤没有找到，DNS会查询本地服务提供商（ISP）
- ISP没找到，请求会执行root跟服务器，返回顶级域名服务器地址
- 浏览器发送请求给顶级域名服务器，返回权威域名服务地址
- 浏览器发送Loopup请求给权威域名服务系统，找到具体DNS纪录，返回给浏览器

###### 4、DNS 纪录

DNS的数据以记录形式存储，就叫DNS记录。DNS记录的种类非常多， 有30多种。每条DNS记录描述了网址(URL)的一种关系。

**（1）A记录**

功能：定义主机IP地址

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523220208778.png" alt="image-20210523220208778" style="zoom:50%;" />

**（2）AAAA记录**

功能：定义主机的IPv6地址

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523220251776.png" alt="image-20210523220251776" style="zoom:50%;" />

**（3）CNAME记录（Canonical Name Record）**

功能：定义域名的别名

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523220407590.png" alt="image-20210523220407590" style="zoom:50%;" />

**（4）MX记录（Mail exchanger record）**

功能：定义邮件服务器所在的位置

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523220537771.png" alt="image-20210523220537771" style="zoom:50%;" />

**（5）NS记录（Name Server Record）**

功能：定义DNS信息服务器所在的位置

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523220726972.png" alt="image-20210523220726972" style="zoom:50%;" />

**（6）SOA记录（Start of Authority Record）**

功能：定义在多个NS服务器中哪个是主服务器

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523220852763.png" alt="image-20210523220852763" style="zoom:50%;" />

**（7）TXT记录**

功能：提供一个文本信息

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523221011007.png" alt="image-20210523221011007" style="zoom:50%;" />

##### 二、DNS工具

-  dig（DNS lookup utility）：用来查询dns的工具
- nslookup：交互式查询域名服务工具
- host（DNS lookup utility）
- Switchhost工具

##### 三、CDN(内容分发网络)

###### 1、CDN是什么

基于地理位置的分布式代理服务器/数据中心。提供高可用，提升性能，提升体验。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523222152474.png" alt="image-20210523222152474" style="zoom:50%;" />

CDN上无法部署业务逻辑，更新慢，无法保证一致性，比较适合纯的静态资源，比如图片、视频、脚本文件、样式文件等。

有些CDN已经提供边缘计算的功能了。

###### 2、CDN实现原理

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210523222618149.png" alt="image-20210523222618149" style="zoom:50%;" />

需要重新弄个图！！！