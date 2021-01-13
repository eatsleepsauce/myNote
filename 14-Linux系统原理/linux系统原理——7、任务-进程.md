#### 进程

##### 一、用系统调用创建进程——写程序

（1）安装开发工具，以centOS7操作系统为例:

```
yum -y groupinstall "Development Tools"
```

（2）可以使用Vim来创建并编辑一个文件，比如创建一个文件process.c，里面用一个函数封装通用的创建进程逻辑。

```
    #include <stdio.h>
    #include <stdlib.h>
    #include <sys/types.h>
    #include <unistd.h>
        
    extern int create_process (char* program, char** arg_list);
        
    int create_process (char* program, char** arg_list)
    {
        pid_t child_pid;
        child_pid = fork ();
        if (child_pid != 0)
            return child_pid;
        else {
            execvp (program, arg_list);
            abort ();
        }
   }
```

fork系统调用创建进程，里面的if-else逻辑可以看出根据fork返回值不通，父子进程就此分开了，在子进程中调用execvp运行一个新的程序。

（3）编写第二个文件createprocess.c，调用上面的函数，创建的子程序运行了一个简单的命令ls。

```
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>

extern int create_process (char* program, char** arg_list);

int main ()
{
    char* arg_list[] = {
        "ls",
        "-l",
        "/etc/yum.repos.d/",
        NULL
    };
    create_process ("ls", arg_list);
    return 0;
}
```

##### 二、程序的二进制格式——进行编译

在linux下面，二进制的程序有严格的格式，这个格式为ELF（Executeable and Linkable Format，可执行与可链接格式）。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145704.jpg" alt="编译过程.jpeg" style="zoom:50%;" />

编译两个.c结尾的源文件，在编译的时候，先做预处理工作，例如将头文件嵌入到正文中，主要是将定义的宏展开，然后才是真正的编译过程，最终编译成 **.o的目标文件**，这是 **ELF的第一种类型**，**可重定位文件（Relocatable File）**。

```
gcc -c -fPIC process.c
gcc -c -fPIC createprocess.c
```

文件格式：

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145705.jpg" alt="目标文件.oELF第一种类型.jpg" style="zoom:50%;" />

（1）ELF文件的头是用于描述整个文件的，格式在内核中有定义，分别为struct elf32_hdr 和 struct elf64_hdr。

（2）后面是一个一个的section，编译好的二进制文件里面应该是代码、全局变量还有静态变量等。

.text ：放编译好的二进制可执行代码

.data：已初始化好的全局变量

.rodata：只读数据，例如字符串常量、const的变量

.bss：未初始化全局变量，运行时会置0

.symtab：符号表，记录的则是函数和变量

.strtab：字符串表、字符串常量和变量名

这里没有局部变量的原因是，局部变量都放在栈里面，是程序运行过程中随时分配空间，随时释放的。

（3）section的元数据信息也需要一个地方保存，就是最后的section header table。在这个表里面，每个section都有一项，在代码里面也有定义 struct elf32_shdr 和 struct elf64_shdr。在ELF的头里面有描述这个文件的section header table的位置，有多少个表项等信息。

可重定位？编译好的代码和变量将来都是加载到内存中的，在内存中都有一定位置，例如create_process函数在process.o的目标文件中，将来被谁调用，在哪儿调用是不清楚的，所以.o里面的位置是不确定的，但是必须是可重定位的。有的section，例如.rel.text，.rel.data就与重定位有关，例如 create process.o里面调用了create_process函数，而这个函数在process.o这个文件里面，因而createprocess.o里面根本不可能知道被调用函数的位置，所以只好在rel.text里面标注，这个函数是需要重定位的。

想让create_process这个函数作为库文件被重用，不能以.o的形式存在，而是要形成库文件，最简单的类型是**静态链接库**.a 文件（Archives），仅仅将一系列对象文件(.o)归档为一个文件，使用命令 ar 创建。

```
ar cr libstaticprocess.a process.o
```

这里的libstaticprocess.a 里面只有一个.o，但是实际情况可以有多个.o，当有程序要使用这个静态链接库的时候，会讲.o文件提取出来，链接到程序中。

```
gcc -o staticcreateprocess createprocess.o -L. -lstaticprocess
```

这个命令，-L表示在当前目录下寻找.a文件，-lstaticprocess会自动补全文件名，比如加前缀lib，后缀.a，变成libstaticprocess.a，找个.a文件后，将里面的process.o 取出来与createprocess.o做一个链接，形成二进制可执行文件 staticcreateprocess。在这个链接的过程，重定位就起作用了，原来createprocess.o里面调用了create_process函数，但是不能确定位置，现在将process.o合并了起来就知道位置了。

形成的二进制文件叫**可执行文件**，是 **ELF的第二种格式**，格式如下

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/2020-12-08-145706.jpg" alt="可执行文件ELF的第二种类型.jpg" style="zoom:50%;" />

这个格式和.o文件大致相似，静态链接库一旦链接进去，代码和变量的section都合并了，因而程序运行的时候，就不依赖这个库是否存在。但是这样有一个缺点，就是相同的代码段，如果被多个程序使用的话，在内存里面就有多份，一旦静态链接库更新了，如果二进制可执行文件不重新编译，也就不会随着更新了。

与静态链接库相对的，有个叫 **动态链接库（Shared Libraries）**，不仅仅是一组对象文件的简单归档，而是多个对象文件的重新组合，可被多个程序共享。

```
gcc -shared -fPIC -o libdynamicprocess.so process.o
```

当一个动态链接库被链接到一个程序文件中的时候，最后程序文件并不包括动态链接库中的代码，而仅仅包括对动态链接库的引用，并且不保存动态链接库的全路径，仅仅保存动态链接库的名称。

```
gcc -o dynamiccreateprocess createprocess.o -L. -ldynamicprocess
```

当运行这个程序的时候，首先寻找动态链接库，然后加载它，默认情况下，系统在/lib和/usr/lib文件夹下寻找动态链接库。如果找不到就会报错，我们可以设定LD_LIBRARY_PATH环境变量，程序运行的时候会在此环境变量指定的文件夹下寻找动态链接库。

**动态链接库，就是 ELF的第三种类型，共享对象文件（Shared Object）.so文件**。

基于动态链接库创建出来的可执行二进制文件还是ELF，但是稍有不同。

（1）首先，多了一个.interp的segment，这里面是ld-linux.so，这是动态链接器，也就是说，运行时的链接都是它做的。

（2）另外，.so文件中还多了两个section，一个是.plt，过程链接表（Procedure Linkage Table , PLT），一个是.got，全局偏移量表（Global Offset Table, GOT）。

dynamiccreateprocess这个程序要调用libdynamicprocess.so里面的create_process函数，由于是运行时才去找，编译的时候，根本不知道这个函数在哪里，所以就在PLT过程链接表这个section里面建立一项PLT[x]，这一项也是一些代码，这些代码会在运行的时候找真正的create_process函数。

PLT[x]怎么找呢，就使用到了GOT，这里面也会为create_process函数创建一项GOT[y]。这一项是运行时create_process函数在内存中真正的地址。

GOT怎么知道地址呢，对于create_process函数，GOT一开始就会创建一项GOT[y]，但是这里面没有真正的地址，因为它也不知道，PLT[x]调它的时候，它又回调PLT，告诉PLT我也不知道地址，这时候PLT会转而调用PLT[0]，PLT[0]转而调用GOT[2]，这里面是ld_linux.so的入口函数，也就是动态链接器的入口，这个函数会找到加载到内存中libdynamicprocess.so里面的create_process函数的地址，然后把这个地址放在GOT[y]里面，这样下次PLT[x]就可以直接使用GOT[y]里面的函数地址了，而不需要每次都链接。

##### 三、运行程序为进程

![程序到进程.jpeg](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/2020-12-08-145707.jpg)

图右边的文件是编译过程，生产so文件和可执行文件，放在硬盘上，图的左边用户态的进程A 执行fork，创建进程B，在进程B的处理逻辑中，执行了 exec 系列系统调用，这个系统调用会通过 load_elf_binary 方法，将刚才生成的可执行文件，加载到进程B的内存中执行。

exec这个系列比较特殊，它是一组函数：

（1）包含p的函数（execvp，execlp）会在PATH路径下面寻找程序

（2）不包含p的函数需要输入程序的全路径

（3）包含v的函数（execv，execvp，execve）以数组的形式接收参数

（4）包含l的函数（execl，execlp，execle）以列表的形式接收参数

（5）包含e的函数（exceve，execle）以数组的形式接收环境变量

##### 四、进程树

所有进程都是从父进程fork过来的，总归有一个祖宗进程，就是我们系统启动的init进程。

![进程树.jpeg](https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/2020-12-08-145713.jpg)

1号进程时/sbin/init，在cenOS7中，这个进程是被软链接到systemd的。ls -l  /sbin/init

```
/sbin/init -> ../lib/systemd/systemd
```

系统启动之后，init进程会启动很多deamon进程，为系统运行提供服务，然后就是启动getty，让用户登录，登录后运行shell，用户启动的进程都是通过shell运行的，从而形成了一棵进程树。

通过 ps -ef 查看当前系统启动的进程

```
[root@deployer ~]# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0  2018 ?        00:00:29 /usr/lib/systemd/systemd --system --deserialize 21
root         2     0  0  2018 ?        00:00:00 [kthreadd]
root         3     2  0  2018 ?        00:00:00 [ksoftirqd/0]
root         5     2  0  2018 ?        00:00:00 [kworker/0:0H]
root         9     2  0  2018 ?        00:00:40 [rcu_sched]
......
root       337     2  0  2018 ?        00:00:01 [kworker/3:1H]
root       380     1  0  2018 ?        00:00:00 /usr/lib/systemd/systemd-udevd
root       415     1  0  2018 ?        00:00:01 /sbin/auditd
root       498     1  0  2018 ?        00:00:03 /usr/lib/systemd/systemd-logind
......
root       852     1  0  2018 ?        00:06:25 /usr/sbin/rsyslogd -n
root      2580     1  0  2018 ?        00:00:00 /usr/sbin/sshd -D
root     29058     2  0 Jan03 ?        00:00:01 [kworker/1:2]
root     29672     2  0 Jan04 ?        00:00:09 [kworker/2:1]
root     30467     1  0 Jan06 ?        00:00:00 /usr/sbin/crond -n
root     31574     2  0 Jan08 ?        00:00:01 [kworker/u128:2]
......
root     32792  2580  0 Jan10 ?        00:00:00 sshd: root@pts/0
root     32794 32792  0 Jan10 pts/0    00:00:00 -bash
root     32901 32794  0 00:01 pts/0    00:00:00 ps -ef
```

PID 1的进程就是我们的init进程systemd，PID 2的进程是内核线程kthreadd，CMD里面用户态的不带中括号，内核态的带中括号。用户态进程的祖先都是1号进程，内核态进程的祖先都是2号进程。tty那一列，是问号的说明不是前台启动的，一般都是后台服务。

pts的父进程是sshd，bash的父进程是pts，ps -ef这个命令的父进程是bash。

##### 5、分析工具

**对应ELF，有几个工具可以查看文件：**

（1）readelf工具分析ELF的信息

（2）objdump工具用来显示二进制文件的信息

（3）hexdump工具用来查看文件的十六进程编码

（4）nm工具用来显示关于指定文件中符号的信息