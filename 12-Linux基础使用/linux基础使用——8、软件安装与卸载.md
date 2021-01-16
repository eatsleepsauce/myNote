#### 软件安装与卸载

##### 一、rpm 安装 和 卸载

rpm -ivh filename // 安装  --prefix path // 指定目录安装

```
rpm -ivh --prefix /usr/ceshi  xxxx.rpm
```

rpm -e 包名 // 卸载
rpm -qa 查看已经安装的包
rpm -q 包名 // 查看包有没有安装
rpm -qi 包名 // 查询指定包的说明信息
rpm -ql 包名 // 查询指定包安装后生成的文件列表
rpm -qc 包名 // 查询指定包安装的配置文档
rpm -qd 包名 // 查询指定包安装的帮助文档
rpm -q --scripts // 查询指定包中包含的脚本
rpm -qa | grep java | xargs rpm -e --nodeps
rpm -qf /path/to/somefile  // 查询指定文件由哪个安装包产生

```
rpm -qf /sbin/ifconfig
```

##### 二、yum安装与配置

yum( yellow dog updater,Modified) 是一个shell前端软件包管理器，基于rpm包管理，能够从**指定的服务器**自动下载rpm包并且安装，可以自动处理依赖关系，并且一次性安装所有依赖的软件包，无须繁琐地一次次下载、安装。
**配置yum源：**

- 修改配置文件(因为国外的源很慢)
  /etc/yum.repos.d/ 目录下下载国内的yum源，比如，阿里或163的yum源
- yum clean all 清空本地的依赖缓存
- yum makecache 将依赖缓存下载到本地

**yum命令常用选项：**

- install 安装软件包 yum install xxx
- remove 卸载软件包 yum remove xxx
- list|grouplist 查看软件包 yum list
- update 升级软件包 yum update (可带软件包名，只升级具体的软件包，不带默认检查所有)

##### 三、其它方式安装

- 二进制安装
- 源代码编译安装 ，有时yum仓库里面没有最新的包，这时候就需要编译源码安装了。
  下载软件包，wget 资源地址

##### 四、内核升级？？？

