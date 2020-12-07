#### 二、基础说明

##### 1、内外部命令

shell自带的命令为内部命令；外部命令，不是shell自带的命令，由用户安装的。

**用户操作后，系统会**
(1)根据空格来切割字符串，把第一个位置认为是命令，其它位置的认为是命令的参数。
(2)判断命令是内部命令还是外部命令
(3)如果是内部命令直接交给linux内核执行，如果是外部命令先寻找这个命令的执行文件，再由内核执行。

```
查看内部命令
type cd
cd is a shell buitin

查看文件类型
file ifconfig
查看文件内容
cat 文件 

可执行文件的路径
whereis ifconfig
```

**外部命令执行会很慢么？** *不会，系统在PATH的目录范围内寻找找可执行文件。*

```
echo $PATH  查看path配置的哪些路径
```

##### 2、命令帮助文档

(1) man 一般用于外部命令帮助文档，使用：man ifconfig，如果man命令不存在，安装man命令  yum install man。
man命令也带有参数，可以通过man man 查看，一般情况下 man 1 ls ，1代表命令可以省略，man -a xxx 会展示匹配的包含命令以及命令之外的(如 passwd就有命令和配置文件)

(2)help一般用于内部命令帮助文档，使用：help cd，help还有另外一种使用方法，cd --help。

(3)info比help更详细可以做为help的补充，使用：info ls 。
