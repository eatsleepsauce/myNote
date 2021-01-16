#### shell

Shell 是命令解释器，用于解释用户用对操作系统的操作。

cat /etc/shells 查看系统中有多少种shell



linux的启动过程

BIOS- MBR - BOOTLOADER(grub) - kernel - systemd - 系统初始化 - shell

1、启动的时候f2进入bios

2、查看mbr引导， dd if=/dev/sda of=mbr.bin bs=446 (字节) count=1

没有文件系统，只能用 hexdump -C mbr.bin 查看（使用16字节查看）

3、grub相关   

```
cd /boot/grub2  // grub所在目录
ls 
grupb2-editenv list //查看引导的内核版本
```

4、systemd

```
cd /etc/systemd/system
ls 
```



5、shell脚本

一条命令只做一件事

为了组合命令和多次执行，使用脚本文件来保存需要执行的命令

赋予该文件执行权限 chmod u+rx filename    二进制文件只需要x权限，脚本需要r

进入目录和查看的组合 ：

```
cd /var/ ;  ls
```



bash脚本，使用.sh作为文件后缀   vim test.sh 保存命令组合，一行一个命令

```
vim test.sh

vim编辑：

#!/bin/bash   #使用！表示不是注释，Sha-Bang，表示用bash解释
# demo #表示注释
cd /var/  #命令
ls

```

执行bash脚本：

bash test.sh  或者 ./test.sh 或者 source ./test.sh. 或者 .filename.sh

6、不同脚本执行方式的影响

1、bash test.sh ，可以不用赋予执行权限，执行脚本的时候产生了新的进程，当前进程没影响

2、./test.sh，需要执行权限，执行脚本的时候产生了新的进程，当前进程没影响

3、source ./test.sh   在当前进程里面执行。

4、.test.sh. 在当前进程里面执行

**内建命令不需要创建子进程，内建命令对当前shell生效**



**7、管道与重定向**

**管道与管道符**

管道和信号一样，也是进程通信的方式之一

匿名管道（管道符）是shell编程经常用到的通信工具

管道符 | ，讲一个命令的执行结果传递给后面的命令

ps | cat

echo 123 | ps

```
cd /proc/pid/fd  可以看到进程的输出有管道信息
ls -l
```

使用管道符尽量规避使用 内建命令。（没有创建新的进程）

**重定向**

把标准输入、输出、错误输出和文件建立了链接

一个进程默认会打开标准输入、输出、错误输出三个文件描述符

（1）输入重定向符号 “<”

read var < /path/to/a/file

```
read var < test.txt
echo $var
```

（2）输出重定向符号 “>” ">>" “2>” "&>"

echo 123 >> /path/to/a/file

">"先清空

“>>”  追加

"2>" 错误输出

"&>" 标准输出和错输出

```
echo $var > test2.txt
echo $var >> test3.txt 追加到文件中
```

（3）输入和输出重定向组合使用，一般在shell脚本中产生新的文件时使用

```
vim  test.sh

#! /bin/bash
cat >/root/a.sh <<EOF
echo "hello word"
EOF

```

8、**变量**

（1）**变量定义**

和其它编程语言的一样，字母、数字、下划线，不能以数字开头

变量的赋值，shell的变量是弱类型，不区分类型

=号左右不能出现空格

变量名=变量值 a=123

使用let 位变量赋值 let a= 0 +20

将命令赋值给变量 l=ls

将命令结果赋值给变量，使用 $() 或者 ``反引号  letc=$(ls -l/etc)

变量值有空个等特殊字符可以包含在“ ” 或 ‘’中

```shell
cmd1=`ls /root`
cmd2=$(ls /root)
string1="hello bash"
string2='hello bash "test" '
```

（2）**变量引用和作用范围**

变量的引用：

${变量名} 称作对变量的引用

echo ${变量名} 查看变量的值

${变量名} 在部分情况下可以省略为 $变量名

变量的作用范围，当前的shell进程中

```
a=1
bash
echo $a
无结果
a=2
exit
echo $a
1
```

变量的导出，export 变量名，子进程就可以获取父进程的变量

变量的删除，unset 变量名，取消变量的赋值

（3）**环境变量、预定义变量与位置变量**

环境变量，每个shell打开都可以获得到的变量

- set 和 env命令
- $? $$ $0 预定义变量，$? 上一条命令是否执行成功，$$ 当前进程的pid，$0 当前进程的名称。
- $path 环境变量PATH
- $PS1

```
env | more
echo $USER
echo $UID
echo $PATH
PATH=$PATH:/root  # 当前shell和子shell生效，对其它shell不生效
```

位置变量，$1 $2 ...  $n

```sh
 vim 7.sh![image-20210116000519756](/Users/liuyang/Library/Application Support/typora-user-images/image-20210116000519756.png)#!/bin/bashpos1=$1pos2=$2#${10} 第10个参数要有大括号echo $pos1echo $pos2echo ${3-_} #如果有第三个值就输出第三个值，如果没有就输出 _ ./7.sh -a -l-a-l
```

（4）**环境变量的配置文件**

配置文件，系统启动，终端启动

- /etc/profile   
- /etc/profile.d/
- ~/.bash_profile
- ~/.bashrc
- /etc/bashrc

不同登录方式，配置文件的启动顺序不一样

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210116000218294.png" alt="image-20210116000218294" style="zoom: 50%;" />



<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210116000549963.png" alt="image-20210116000549963" style="zoom: 50%;" />

**修改完了不是立即生效，可以使用 exit 退出当前shell进程，或者  使用 source 配置文件名，让配置文件生效。**



