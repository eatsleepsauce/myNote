#### 硬盘文件系统

文件系统就是安装在硬盘之上。目前 Linux 下最主流的文件系统格式——ext 系列的文件系统的格式。

##### 一、inode与块的存储

###### 1、块

硬盘分成相同大小的单元，我们称为 **块（Block）**。一块的大小是扇区大小的整数倍（扇区512字节），默认是 4K。在格式化的时候，这个值是可以设定的。

一大块硬盘被分成了一个个小的块，用来存放文件的数据部分。存放一个文件，就不用给他分配一块连续的空间了。我们可以分散成一个个小块进行存放。这样就灵活得多，也比较容易添加、删除和插入数据。

###### 2、inode

**inode** 是一个结构，用来存放 **文件分成几个块**，**每一块在哪儿**，以及 **文件的元数据** 信息（名字、权限等）等等。

inode 的“i”是 index 的意思，其实就是“索引”，类似图书馆的索引区域。我们每个文件都会对应一个 inode；一个文件夹就是一个文件，也对应一个 inode。

**inode数据结构如下**：

```c
struct ext4_inode {
  __le16  i_mode;    /* File mode */
  __le16  i_uid;    /* Low 16 bits of Owner Uid */
  __le32  i_size_lo;  /* Size in bytes */
  __le32  i_atime;  /* Access time */
  __le32  i_ctime;  /* Inode Change time */
  __le32  i_mtime;  /* Modification time */
  __le32  i_dtime;  /* Deletion Time */
  __le16  i_gid;    /* Low 16 bits of Group Id */
  __le16  i_links_count;  /* Links count */
  __le32  i_blocks_lo;  /* Blocks count */
  __le32  i_flags;  /* File flags */
......
  __le32  i_block[EXT4_N_BLOCKS];/* Pointers to blocks */
  __le32  i_generation;  /* File version (for NFS) */
  __le32  i_file_acl_lo;  /* File ACL */
  __le32  i_size_high;
......
};
```

inode 里面有文件的读写权限 i_mode，属于哪个用户 i_uid，哪个组 i_gid，大小是多少 i_size_io，占用多少个块 i_blocks_io。ls 命令行的时候，列出来的权限、用户、大小这些信息，就是从这里面取出来的。

还有几个与文件相关的时间。i_atime 是 access time，是最近一次访问文件的时间；i_ctime 是 change time，是最近一次更改 inode 的时间；i_mtime 是 modify time，是最近一次更改文件的时间。

“**某个文件分成几块、每一块在哪里**”，这些在 inode 里面，保存在 **i_block** 里面。

（1）**在 ext2 和 ext3 中**

其中前 12 项直接保存了块的位置，可以通过 i_block[0-11]，直接得到保存文件内容的块。如果一个文件比较大，12 块放不下。i_block[12] 就不能直接放数据块的位置了，这个块里面不放数据块，而是放数据块的位置，这个块称为 **间接块**。再大一些，i_block[13]会指向一个块，可以用二次间接块。二次间接块里面存放了间接块的位置，间接块里面存放了数据块的位置，数据块里面存放的是真正的数据。再大一些，i_block[14]会指向三次间接块。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/73349c0fab1a92d4e1ae0c684cfe06e2.jpeg" alt="73349c0fab1a92d4e1ae0c684cfe06e2" style="zoom:33%;" />

对于大文件来讲，要多次读取硬盘才能找到相应的块，这样访问速度就会比较慢。

（2）**在ext4中**

为了解决ext2和ext3中的问题，ext4 做了一定的改变。它引入了一个新的概念，叫做 **Extents**。

一个文件大小为 128M，如果使用 4k 大小的块进行存储，需要 32k 个块。如果按照 ext2 或者 ext3 那样散着放，数量太大了。但是 Extents 可以用于存放连续的块，我们可以把 128M 放在一个 Extents 里面。这样的话，对大文件的读写性能提高了，文件碎片也减少了。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/b8f184696be8d37ad6f2e2a4f12d002a.jpeg" alt="b8f184696be8d37ad6f2e2a4f12d002a" style="zoom:33%;" />

树有一个个的节点，有叶子节点，也有分支节点。每个节点都有一个头，**ext4_extent_header** 可以用来描述某个节点。

```c
struct ext4_extent_header {
  __le16  eh_magic;  /* probably will support different formats */
  __le16  eh_entries;  /* number of valid entries */
  __le16  eh_max;    /* capacity of store in entries */
  __le16  eh_depth;  /* has tree real underlying blocks? */
  __le32  eh_generation;  /* generation of the tree */
};
```

eh_entries 表示这个节点里面有多少项。项分两种：

- 叶子节点，这一项会直接指向硬盘上的连续块的地址，称为 **数据节点 ext4_extent**；
- 分支节点，这一项会指向下一层的分支节点或者叶子节点，称为 **索引节点 ext4_extent_idx**。

这两种类型的项的大小都是 12 个 byte。

除了根节点，其他的节点都保存在一个块 4k 里面，4k 扣除 ext4_extent_header 的 12 个 byte，剩下的能够放 340 项，每个 extent 最大能表示 128MB 的数据，340 个 extent 会使你表示的文件达到 42.5GB。这已经非常大了，如果再大，我们可以增加树的深度。

##### 二、inode位图和块位图

在文件系统里面，专门弄了 **一个块来保存 inode 的位图**。在这 4k 里面，每一位对应一个 inode。如果是 1，表示这个 inode 已经被用了；如果是 0，则表示没被用。同样，也弄了一个块保存 block 的位图。

##### 三、文件系统的格式

数据块的位图是放在一个块里面的，共 4k。每位表示一个数据块，如果每个数据块也是按默认的 4K，最大可以表示空间也就是 128M。

如果采用“一个块的位图 + 一系列的块”，外加“一个块的 inode 的位图 + 一系列的 inode 的结构”，文件系统最多能够表示 128M。现在很多文件都比这个大。

通过  **块组** 这样的结构（有 N 多的块组），就能够表示很大的文件。块组使用数据结构 **ext4_group_desc**，这个结构又包含了成员变量，**inode 位图 bg_inode_bitmap_lo**、**块位图 bg_block_bitmap_lo**、**inode 列表 bg_inode_table_lo**。

因为块组有多个，块组描述符也同样组成一个列表，我们把这些称为 **块组描述符表**。

块组里面还有 有一个数据结构，对整个文件系统的情况进行描述，这个就是 **超级块ext4_super_block**。包括了，整个文件系统一共有多少 inode，s_inodes_count；一共有多少块，s_blocks_count_lo，每个块组有多少 inode，s_inodes_per_group，每个块组有多少块，s_blocks_per_group 等全局信息。

**文件系统格式：**

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/e3718f0af6a2523a43606a0c4003631b.jpeg" alt="e3718f0af6a2523a43606a0c4003631b" style="zoom:33%;" />

**超级块和块组描述符表都是全局信息，而且这些数据很重要。如果这些数据丢失了，整个文件系统都打不开了。**

**Meta Block Groups 特性 ** ，暂不描述。

##### 四、目录的存储格式

目录本身也是个文件，也有 inode。inode 里面也是指向一些块。和普通文件不同的是，普通文件的块里面保存的是文件数据，而目录文件的块里面保存的是目录里面一项一项的文件信息。这些信息我们称为 ext4_dir_entry。(第二个版本 ext4_dir_entry_2 是将一个 16 位的 name_len，变成了一个 8 位的 name_len 和 8 位的 file_type。)

```c
struct ext4_dir_entry {
  __le32  inode;      /* Inode number */
  __le16  rec_len;    /* Directory entry length */
  __le16  name_len;    /* Name length */
  char  name[EXT4_NAME_LEN];  /* File name */
};
struct ext4_dir_entry_2 {
  __le32  inode;      /* Inode number */
  __le16  rec_len;    /* Directory entry length */
  __u8  name_len;    /* Name length */
  __u8  file_type;
  char  name[EXT4_NAME_LEN];  /* File name */
};
```

在目录文件的块中，最简单的保存格式是列表，就是一项一项地将 ext4_dir_entry_2 列在哪里。每一项都会保存这个目录的下一级的文件的文件名和对应的 inode，通过这个 inode，就能找到真正的文件。第一项是“.”，表示当前目录，第二项是“…”，表示上一级目录，接下来就是一项一项的文件名和 inode。有时候，如果一个目录下面的文件太多的时候，按照列表一个个去找，太慢了，于是我们就添加了索引的模式。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/3ea2ad5704f20538d9c911b02f42086d.jpeg" alt="3ea2ad5704f20538d9c911b02f42086d" style="zoom:33%;" />

##### 五、软链接和硬链接的存储格式

**硬链接（Hard Link）**和 **软链接（Symbolic Link）**。所谓的链接（Link），我们可以认为是文件的别名，而链接又可分为两种，硬链接与软链接。通过下面的命令可以创建。

```
ln -s 创建的是软链接，不带 -s 创建的是硬链接。
ln [参数][源文件或目录][目标文件或目录]
```

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/45a6cfdd9d45e30dc2f38f0d2572be7b.jpeg" alt="45a6cfdd9d45e30dc2f38f0d2572be7b" style="zoom:33%;" />

**硬链接与原始文件共用一个 inode 的**，但是 **inode 是不跨文件系统的**，每个文件系统都有自己的 inode 列表，因而硬链接是没有办法跨文件系统的。(各文件系统有自己的inode信息)

**软链接相当于重新创建了一个文件，这个文件也有独立的 inode**，只不过打开这个文件看里面内容的时候，内容指向另外的一个文件。这就很灵活了。我们可以跨文件系统，甚至目标文件被删除了，链接文件还是在的，只不过指向的文件找不到了而已。



<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/f81bf3e5a6cd060c3225a8ae1803a138.png" alt="f81bf3e5a6cd060c3225a8ae1803a138" style="zoom:33%;" />

