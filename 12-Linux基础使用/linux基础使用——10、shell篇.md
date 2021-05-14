#### SHELL

Shell 是命令解释器，用于解释用户用对操作系统的操作。Unix哲学：一条命令只做一件事。

cat /etc/shells 查看系统中有多少种shell。

```shell
cat /etc/shells
```

##### 一、linux的启动过程

BIOS-> MBR(硬盘的第一个扇区，主引导记录) -> BOOTLOADER(grub) -> kernel - >systemd -> 系统初始化 -> shell

1. 启动的时候f2进入bios
2. 查看mbr引导， dd if=/dev/sda of=mbr.bin bs=446 (字节) count=1，没有文件系统，只能用 hexdump -C mbr.bin 查看（使用16字节查看）
3. grub

```
cd /boot/grub2  // grub所在目录
ls 
grupb2-editenv list //查看引导的内核版本
```

4. systemd

```
cd /etc/systemd/system
ls 
```

##### 二、**shell脚本**


为了组合命令和多次执行，使用脚本文件来保存需要执行的命令，赋予该文件执行权限 chmod u+rx filename，二进制文件只需要x权限，脚本需要 r 权限。

###### 1、普通命令组合模式

进入目录和查看的命令组合 （在终端命令行中直接输入）：

```
cd /var/ ;  ls
```

###### 2、shell脚本模式

bash脚本，使用.sh作为文件后缀   vim test.sh 保存命令组合，一行一个命令。

```
vim test.sh

vim编辑：
#!/bin/bash   #使用！表示不是注释，Sha-Bang，表示用bash解释
# demo #表示注释
cd /var/  #命令
ls
```

执行bash脚本，不同脚本执行方式的影响

- bash test.sh ，可以不用赋予执行权限，执行脚本的时候产生了新的进程，当前进程没影响
- ./test.sh，需要执行权限，执行脚本的时候产生了新的进程，当前进程没影响
- source ./test.sh   在当前进程里面执行
- .test.sh. 在当前进程里面执行

##### 三、内建命令和外部命令区别

内建命令不需要创建子进程，内建命令对当前shell生效。

##### 四、管道与重定向

###### 1、管道

管道和信号一样，也是进程通信的方式之一，匿名管道（管道符）是shell编程经常用到的通信工具。

管道符 | ，将一个命令的执行结果传递给后面的命令。

ps | cat

echo 123 | ps

```
cd /proc/pid/fd  可以看到进程的输出有管道信息
ls -l
```

使用管道符尽量规避使用 内建命令。（没有创建新的进程）

###### 2、重定向

重定向把标准输入、输出、错误输出和文件建立了链接。

一个进程默认会打开标准输入、输出、错误输出三个文件描述符。

- 输入重定向符号 “<”

  read var < /path/to/a/file

```
read var < test.txt
echo $var
```

- 输出重定向符号 “>” ">>" “2>” "&>"

  echo 123 >> /path/to/a/file

  - ">"先清空

  - “>>”  追加

  - "2>" 错误输出

  - "&>" 标准输出和错输出

```
echo $var > test2.txt
echo $var >> test3.txt 追加到文件中
```

- 输入和输出重定向组合使用，一般在shell脚本中产生新的文件时使用

```
vim  test.sh

#! /bin/bash
cat >/root/a.sh <<EOF
echo "hello word"
EOF
```

##### 五、变量

###### 1、变量定义

和其它编程语言的一样，字母、数字、下划线，不能以数字开头。

变量的赋值，shell的变量是弱类型，不区分类型，=号左右不能出现空格。

- 变量名=变量值 a=123
- 使用let 位变量赋值 let a= 0 +20
- 将命令赋值给变量 l=ls
- 将命令结果赋值给变量，使用 $() 或者 ``反引号  letc=$(ls -l/etc)
- 变量值有空格等特殊字符可以包含在"" 或 ''中

```shell
cmd1=`ls /root`
cmd2=$(ls /root)
string1="hello bash"
string2='hello bash "test" '
```

###### 2、变量引用和作用范围

- ${变量名} 称作对变量的引用
- echo ${变量名} 查看变量的值
- ${变量名} 在部分情况下可以省略为 $变量名

变量的作用范围，当前的shell进程中

```shell
a=1
bash
echo $a
无结果
a=2
exit
echo $a
1
```

- 变量的导出，export 变量名，子进程就可以获取父进程的变量
- 变量的删除，unset 变量名，取消变量的赋值

###### 3、环境变量、预定义变量与位置变量

环境变量：每个shell打开都可以获得到的变量。

- set 和 env命令
- $? $$ $0 预定义变量，$? 上一条命令执行结果，$$ 当前进程的pid，$0 当前进程的名称。
- $path 环境变量PATH
- $PS1

```shell
env | more
echo $USER
echo $UID
echo $PATH
PATH=$PATH:/root  # 当前shell和子shell生效，对其它shell不生效
```

位置变量，$1 $2 ...  $n

```sh
vim 7.sh
#!/bin/bash
pos1=$1
pos2=$2
#${10} 第10个参数要有大括号echo $pos1 echo $pos2 echo ${3-_} 
#如果有第三个值就输出第三个值，如果没有就输出 _ ./7.sh -a -l-a-l
```

###### 4、环境变量的配置文件

配置文件，系统启动，终端启动

- /etc/profile   
- /etc/profile.d/
- ~/.bash_profile
- ~/.bashrc
- /etc/bashrc

不同登录方式，配置文件的启动顺序不一样，**修改完了不是立即生效，可以使用 exit 退出当前shell进程，或者  使用 source 配置文件名，让配置文件生效。**

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210116000218294.png" alt="image-20210116000218294" style="zoom: 50%;" />

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210116000549963.png" alt="image-20210116000549963" style="zoom: 50%;" />

##### 六、数组

- 定义数组，IPTS=( 111 333 222 )

- 显示数组的所有元素，echo ${IPTS[@]}

- 显示数组元素个数，echo ${#IPTS[@]}

- 显示数组的第一个元素，echo ${IPTS[0]}

##### 七、特殊字符、转义符和引用符

特殊字符：一个字符不仅有字面意义，还有元意（meta-meaning）

###### 1、特殊字符

- \# 注释符
- ; 命令分隔符
- \ 转义字符
- :  空指令
- .  和 source命令相同
- ~ 家目录
- , 分隔目录

###### 2、单个字符前的转义符号

- \n \r \t 单个字母的转义
- \\$  \\"  \\\ 单个非字母的转义

###### 3、引号

- '  单引号，完全引用
- " 双引号，不完全引用
- ` 执行命令，反引号，主要将命令括起来

单引号和双引号对变量的作用不同。双引号会解释变量。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210116134108047.png" alt="image-20210116134108047" style="zoom: 50%;" />

###### 4、括号

- 圆括号 ()  (())  $()   单独使用圆括号会产生一个子shell ，(xyz=123)；数组初始化 IPS=(ip1 ip2 ip3)
- 方括号[]  [[]]  单独使用方括号是测试 或 数组元素功能，两个方括号表示测试表达式。
- 尖括号 <>  重定向符号
- 花括号{}， 输出范围 echo {0..9} ，文件复制 cp /etc/passwd{,.bak}

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210116141318154.png" alt="image-20210116141318154" style="zoom:50%;" />

###### 5、运算和逻辑符号

- \+  -  * / % 算数运算符
- \> < = 比较运算符
- && || !  逻辑运算符

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210116141630700.png" alt="image-20210116141630700" style="zoom:50%;" />



##### 八、运算符

###### 1、赋值运算符

- =复制运算符，用于算数赋值和字符串赋值
- 使用unset 取消变量的赋值
- =除了作为赋值运算符还可以作为测试操作符

###### 2、算数运算符

- 基本运算符

- \+ \- \* /  \**(乘方) %

- 使用 expr进行运算（只支持整数，不支持浮点数）

- expr 4 + 5

###### 3、数字常量

- let “变量名=变量值”
- 变量值使用0开头为八进制
- 变量值使用0x开头为十六进制

###### 4、双圆括号是let命令的简化

- ((a=10))

- ((a++))

- Echo $((10+20))

  

  <img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210116135720479.png" alt="image-20210116135720479" style="zoom:50%;" />

##### 九、退出和测试（条件）

###### 1、退出与退出状态

- exit
- exit 10 返回10给shell，返回非0为不正常退出
- $? 判断当前shell前一个进程是否正常退出

```shell
vim test.sh
#!/bin/bash
ppwd
exit

bash  test.sh
```

###### 2、测试命令test

- test命令用于检查文件或者比较值
- test可以做以下测试：
  - 文件测试
  - 整数比较测试
  - 字符串测试 

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210116214810834.png" alt="image-20210116214810834" style="zoom: 50%;" />

- test测试语句可以简化为 [] 符号

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210116214938789.png" alt="image-20210116214938789" style="zoom:50%;" />

- [] 符号还有扩展写法 [[]] 支持&&  ||  <  >，[] 不支持直接使用<>，只能用lt gt。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210116215122232.png" alt="image-20210116215122232" style="zoom:50%;" />

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210116215202334.png" alt="image-20210116215202334" style="zoom:50%;" />

**test中使用 = 等号判断时， 等号左右要有空格。**可使用man test 查看test的用法。

##### 十、分支与循环

###### 1、使用if-then 语句

```
if [测试条件成立] 或者 命令执行返回值是否为0
then 执行相应命令
fi 结束
```

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210116220125642.png" alt="image-20210116220125642" style="zoom:50%;" />

if 后面跟着命令：

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210116220320065.png" alt="image-20210116220320065" style="zoom:50%;" />

###### 2、使用if-then-else语句

if-then-else 语句可以在条件不成立时也运行相应的命令

```
if [测试条件成立] 或者 命令执行返回值为 0
then 执行相应命令
elif [测试条件成立]
then 执行相应命令
else 测试条件不成立，执行相应命令
fi 结束
```

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210116221728247.png" alt="image-20210116221728247" style="zoom:50%;" />

###### 3、嵌套if的使用

```
if [测试条件成立]
then 执行相应命令
	if [测试条件成立]
	then 执行相应命令
	fi
fi 结束
```

###### 4、分支

case语句和select语句可以构成分支

```
case "$变量" in
"情况1"）
	命令...;;
"情况2")
	命令...;;
* )
	命令...;;
esac
```

###### 5、for 循环

```
for 参数 in 列表
do 执行的命令
done封闭一个循环
```

可以使用反引号 或 $() 方式执行命令，命令的结果当作列表进行处理。

列表中包含多个变量，变量使用空格分隔。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210117104804925.png" alt="image-20210117104804925" style="zoom:50%;" />

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210117105025249.png" alt="image-20210117105025249" style="zoom:50%;" />

###### 6、C风格的for

```
for((变量初始化;循环判断条件;变量变化))
do
	循环执行的命令
done
```

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210117105343741.png" alt="image-20210117105343741" style="zoom:50%;" />

###### 7、while循环和until循环

while 循环

```
while test测试是否成立
do
	命令
done
```

until循环与while循环相反，循环测试为假时，执行循环，为真时循环停止。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210117110226414.png" alt="image-20210117110226414" style="zoom:50%;" />

###### 8、循环的嵌套和break、continue语句

- 循环和循环可以嵌套
- 循环中可以嵌套判断，反过来也可以嵌套
- 循环可以使用break 和 continue 语句在循环中退出

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210117111049424.png" alt="image-20210117111049424" style="zoom:50%;" />

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210117111424276.png" alt="image-20210117111424276" style="zoom:50%;" />

###### 9、使用循环处理位置参数

- 命令行参数可以使用 $1 $2 $3 ……$n 进行读取
- $0 代表脚本名称
- $* 和 $@ 代表所有位置参数
- $# 代表位置参数的数量

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210117112438561.png" alt="image-20210117112438561" style="zoom:50%;" />

##### 十一、自定义函数

- 函数的定义

```shell
function fname(){
命令
}
```

- 函数的执行

```
fname
```

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202021-01-25%20%E4%B8%8B%E5%8D%889.41.29.png" alt="屏幕快照 2021-01-25 下午9.41.29" style="zoom: 50%;" />

- 函数的清除

```
unset fname
```

- 使用函数参数

```shell
function cdls(){
> cd $1
> ls
}


cdls /tmp
```

函数的参数 , $1 $2 $3...$n 

- 函数内部变量，只在函数内部生效  

  local 变量名

```shell
#！/bin/bash
# functions
checkpid(){
	local i
	for i in $*; do
			[-d "/proce/$i"] && return 0
	done
	
	return 1
	
}
```

![image-20210510212139769](/Users/liuyang/Library/Application Support/typora-user-images/image-20210510212139769.png)

脚本中的函数，可以通过source 引入，然后执行。

##### 十二、系统自建函数库和自建脚本

###### 1、系统自建函数

- 在 /etc/init.d/functions 中

- 使用 source 函数脚本文件，导入函数

  ```shell
  source /etc/init.d/functions
  ```

###### 2、系统自建脚本

- /etc/profile 环境变量
- ~/.bashrc
- ~/.bash_profile

##### 十三、脚本控制

###### 1、脚本优先级控制

- 可以使用nice 和 renice 调整脚本优先级（及时发现脚本的不可控行为后使用）

- 避免出现 “不可控的” 死循环
  - 死循环导致cpu占用过高
  - 死循环导致死机

举例：fork炸弹

###### 2、捕获信号

###### 1、捕获信号脚本的编写

- kill默认会发送15号信号给应用程序
- ctrl + c 发送2号信号给应用程序
- 9号信号不可阻塞

```shell
#!/bin/bash

#signal demo
#捕获15信号
trap "echo signal 15" 15
#捕获2信号 -可以加入到一些备份脚本中，防止被误操作
trap “echo signal 2” 2
echo $$
```

###### 十四、计划任务

###### 1、一次性计划任务at

```shell
at 18:31
at> echo hello >/tmp/hello.txt   
at> <EOT>  ctrl + d 提交任务

atq 查看一次执行计划
```

- 执行一次性计划时，如果不是系统内建命令，需要使用命令的全路径。
- 执行一次性计划时，如果是脚本则需要使用source 引入环境变量。

- 执行计划时，没有终端，所以需要重定向输出。

###### 2、周期性计划任务

cron

- 配置方式
  - crontab -e
- 查看现有的计划任务
  - crontab -l
- 配置格式：
  - 分钟 小时 日期 月份 星期 执行的命令
  - 注意命令的路径和输出
  - 实现秒级别的执行，需要借助第三方软件

```
crontab -e

#每分钟执行一次
* * * * * /usr/bin/date >> /tmp/date.txt
* * * * 1-5 周一到周五每分钟。。。
```

- 可以在 /var/log/cron中看到任务执行

###### 3、延时计划任务

anacrontab 延时计划任务，vim /etc/anacrontab，里面有 日 周 月的延迟执行，

小时的延迟执行，vim /etc/cron.d/0hourly

###### 4、计划任务加锁flock

示例中的a.sh是一个执行很久的脚本，只有ctrl + c 后 第二个终端的a.sh才能正常执行。

![image-20210513231518709](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/image-20210513231518709.png)

