#### 文件系统简介

##### 一、文件系统的功能规划

我们最常用的外部存储就是硬盘，数据是以文件的形式保存在硬盘上的。为了管理这些文件，我们在规划文件系统的时候，需要考虑到以下几点。

- 第一点，文件系统要有 **严格的组织形式** ，使得文件能够以块为单位进行存储。这就像图书馆里，我们会设置一排排书架，然后再把书架分成一个个小格子，有的项目存放的资料非常多，一个格子放不下，就需要多个格子来存放。
- 第二点，文件系统中也要有 **索引区** ，用来方便查找一个文件分成的多个块都存放在了什么位置。

- 第三点，如果文件系统中有的文件是 **热点文件**，近期经常被读取和写入，文件系统应该有 **缓存层**。

- 第四点，文件应该用 **文件夹的形式组织 **起来，方便管理和查询。

- 第五点，Linux 内核要在自己的内存里面维护一套 **数据结构**，来保存哪些文件被 **哪些进程打开和使用**。

##### 二、文件系统相关命令行

###### 1、格式化

**格式化**，即将一块盘使用命令组织成一定格式的文件系统的过程。咱们买个硬盘或者 U 盘，经常说要先格式化，才能放文件，说的就是这个。

Windows 的常格式化的格式为 **NTFS（New Technology File System）**。

在 Linux 下面，常用的是 **ext3** 或者 **ext4**。

通过命令 **fdisk -l**，查看格式化和没有格式化的分区。通过命令 **mkfs.ext3** 或者 **mkfs.ext4** 进行格式化。

**格式化后的硬盘，需要挂在到某个目录下面**，才能作为普通的文件系统进行访问。

```
mount /dev/vdc1 /根目录/用户A目录/目录1
```

卸载使用 umount 命令。

```
umount /根目录/用户A目录/目录1
```

**ls -l** 的结果的第一位标识位可以看出来文件的类型：

- 表示普通文件；
- d 表示文件夹；
- c 表示字符设备文件；
- b 表示块设备文件；
- s 表示套接字 socket 文件；
- l 表示符号链接，也即软链接，就是通过名字指向另外一个文件。

##### 三、文件系统相关系统调用

示例：

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>

int main(int argc, char *argv[])
{
  int fd = -1;
  int ret = 1;
  int buffer = 1024;
  int num = 0;

  if((fd=open("./test", O_RDWR|O_CREAT|O_TRUNC))==-1)
  {
    printf("Open Error\n");
    exit(1);
  }

  ret = write(fd, &buffer, sizeof(int));
  if( ret < 0)
  {
    printf("write Error\n");
    exit(1);
  }
  printf("write %d byte(s)\n",ret);

  lseek(fd, 0L, SEEK_SET);
  ret= read(fd, &num, sizeof(int));
  if(ret==-1)
  {
    printf("read Error\n");
    exit(1);
  }
  printf("read %d byte(s)，the number is %d\n", ret, num);

  close(fd);

  return 0;
}
```

###### 1、open函数

open 打开一个文件时，操作系统会创建一些数据结构来表示这个被打开的文件。为了能够找到这些数据结构，在进程中，我们会为这个打开的文件分配一个 **文件描述符 fd（File Descriptor）**。

**文件描述符**，就是用来区分一个进程打开的多个文件的。它的 **作用域就是当前进程**，出了当前进程这个文件描述符就没有意义了。**对这个文件的所有操作都要靠 fd，包括最后关闭文件**。

open 函数中，有一些参数：

- O_CREAT 表示当文件不存在，创建一个新文件；
- O_RDWR 表示以读写方式打开；
- O_TRUNC 表示打开文件后，将文件的长度截断为 0。

###### 2、write函数

write 要用于写入数据。第一个参数就是文件描述符，第二个参数表示要写入的数据存放位置，第三个参数表示希望写入的字节数，返回值表示成功写入到文件的字节数。

###### 3、lseek函数

lseek 用于重新定位读写的位置，第一个参数是文件描述符，第二个参数是重新定位的位置，第三个参数是 SEEK_SET，表示起始位置为文件头，第二个参数和第三个参数合起来表示将读写位置设置为从文件头开始 0 的位置，也即从头开始读写。

###### 4、read函数

read 用于读取数据，第一个参数是文件描述符，第二个参数是读取来的数据存到指向的空间，第三个参数是希望读取的字节数，返回值表示成功读取的字节数。

###### 5、close函数

close 将关闭一个文件。

###### 6、获取文件状态

```c
int stat(const char *pathname, struct stat *statbuf);
int fstat(int fd, struct stat *statbuf);
int lstat(const char *pathname, struct stat *statbuf);

struct stat {
  dev_t     st_dev;         /* ID of device containing file */
  ino_t     st_ino;         /* Inode number */
  mode_t    st_mode;        /* File type and mode */
  nlink_t   st_nlink;       /* Number of hard links */
  uid_t     st_uid;         /* User ID of owner */
  gid_t     st_gid;         /* Group ID of owner */
  dev_t     st_rdev;        /* Device ID (if special file) */
  off_t     st_size;        /* Total size, in bytes */
  blksize_t st_blksize;     /* Block size for filesystem I/O */
  blkcnt_t  st_blocks;      /* Number of 512B blocks allocated */
  struct timespec st_atim;  /* Time of last access */
  struct timespec st_mtim;  /* Time of last modification */
  struct timespec st_ctim;  /* Time of last status change */
};
```

函数 stat 和 lstat 返回的是通过文件名查到的状态信息。这两个方法区别在于，stat 没有处理符号链接（软链接）的能力。如果一个文件是符号链接，stat 会直接返回它所指向的文件的属性，而 lstat 返回的就是这个符号链接的内容，fstat 则是通过文件描述符获取文件对应的属性。

###### 7、opendir函数

opendir 函数打开一个目录名所对应的 DIR 目录流。并返回指向 DIR 目录流的指针。流定位在 DIR 目录流的第一个条目。

###### 8、readdir函数

readdir 函数从 DIR 目录流中读取一个项目，返回的是一个指针，指向 dirent 结构体，且流的自动指向下一个目录条目。如果已经到流的最后一个条目，则返回 NULL。

###### 9、closedir函数

closedir() 关闭参数 dir 所指的目录流。

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <dirent.h>

int main(int argc, char *argv[])
{
  struct stat sb;
  DIR *dirp;
  struct dirent *direntp;
  char filename[128];
  if ((dirp = opendir("/root")) == NULL) {
    printf("Open Directory Error%s\n");
    exit(1);
  }
  while ((direntp = readdir(dirp)) != NULL){
    sprintf(filename, "/root/%s", direntp->d_name);
    if (lstat(filename, &sb) == -1)
    {
      printf("lstat Error%s\n");
      exit(1);
    }
    printf("name : %s, mode : %d, size : %d, user id : %d\n", direntp->d_name, sb.st_mode, sb.st_size, sb.st_uid);

  }
  closedir(dirp);
  return 0
}
```

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/2788a6267f8361c9b6c338b06a1afc50.png" alt="img" style="zoom:33%;" />