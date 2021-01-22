#### 文件缓存

##### 一、系统调用层和虚拟文件系统层

文件系统的读写，其实就是调用系统函数 read 和 write。

read 和 write 的系统调用，在内核里面的定义:

```c
SYSCALL_DEFINE3(read, unsigned int, fd, char __user *, buf, size_t, count)
{
  struct fd f = fdget_pos(fd);
......
  loff_t pos = file_pos_read(f.file);
  ret = vfs_read(f.file, buf, count, &pos);
......
}

SYSCALL_DEFINE3(write, unsigned int, fd, const char __user *, buf,
    size_t, count)
{
  struct fd f = fdget_pos(fd);
......
  loff_t pos = file_pos_read(f.file);
    ret = vfs_write(f.file, buf, count, &pos);
......
}
```

```c
ssize_t __vfs_read(struct file *file, char __user *buf, size_t count,
       loff_t *pos)
{
  if (file->f_op->read)
    return file->f_op->read(file, buf, count, pos);
  else if (file->f_op->read_iter)
    return new_sync_read(file, buf, count, pos);
  else
    return -EINVAL;
}

ssize_t __vfs_write(struct file *file, const char __user *p, size_t count,
        loff_t *pos)
{
  if (file->f_op->write)
    return file->f_op->write(file, p, count, pos);
  else if (file->f_op->write_iter)
    return new_sync_write(file, p, count, pos);
  else
    return -EINVAL;
}
```

每一个打开的文件，都有一个 struct file 结构。这里面有一个 struct file_operations f_op，用于定义对这个文件做的操作。**_ _vfs_read 会调用相应文件系统的 file_operations 里面的 read 操作，__vfs_write 会调用相应文件系统 file_operations 里的 write 操作。**

##### 二、ext4文件系统层

对于 ext4 文件系统来讲，内核定义了一个 ext4_file_operations 结构，结构类型就是 **file_operations**  。

```c
const struct file_operations ext4_file_operations = {
......
  .read_iter  = ext4_file_read_iter,
  .write_iter  = ext4_file_write_iter,
......
}
```

由于 ext4 没有定义 read 和 write 函数，于是会调用 ext4_file_read_iter 和 ext4_file_write_iter。**ext4_file_read_iter 会调用 generic_file_read_iter**，**ext4_file_write_iter 会调用 __generic_file_write_iter**。

```c
ssize_t
generic_file_read_iter(struct kiocb *iocb, struct iov_iter *iter)
{
......
    if (iocb->ki_flags & IOCB_DIRECT) {
......
        struct address_space *mapping = file->f_mapping;
......
        retval = mapping->a_ops->direct_IO(iocb, iter);
    }
......
    retval = generic_file_buffered_read(iocb, iter, retval);
}

ssize_t __generic_file_write_iter(struct kiocb *iocb, struct iov_iter *from)
{
......
    if (iocb->ki_flags & IOCB_DIRECT) {
......
        written = generic_file_direct_write(iocb, from);
......
    } else {
......
    written = generic_perform_write(file, from, iocb->ki_pos);
......
    }
}
```

generic_file_read_iter 和 __generic_file_write_iter 有相似的逻辑，就是要 **区分是否用缓存**。

根据是否使用内存做缓存，把 **文件的 I/O 操作分为两种类型**:

-  **缓存 I/O**。大多数文件系统的默认 I/O 操作都是缓存 I/O。
  - 对于读操作来讲，操作系统会先检查，内核的缓冲区有没有需要的数据。如果已经缓存了，那就直接从缓存中返回；否则从磁盘中读取，然后缓存在操作系统的缓存中。
  - 对于写操作来讲，操作系统会先将数据从用户空间复制到内核空间的缓存中。这时对用户程序来说，写操作就已经完成。至于什么时候再写到磁盘中由操作系统决定，除非显式地调用了 sync 同步命令。

- **直接 IO**，就是应用程序直接访问磁盘数据，而不经过内核缓冲区，从而减少了在内核缓存和用户程序之间数据复制。

##### 三、带缓存的写入操作

带缓存写入的函数 **generic_perform_write**：

```c
ssize_t generic_perform_write(struct file *file,
        struct iov_iter *i, loff_t pos)
{
  struct address_space *mapping = file->f_mapping;
  const struct address_space_operations *a_ops = mapping->a_ops;
  do {
    struct page *page;
    unsigned long offset;  /* Offset into pagecache page */
    unsigned long bytes;  /* Bytes to write to page */
    status = a_ops->write_begin(file, mapping, pos, bytes, flags,
            &page, &fsdata);
    copied = iov_iter_copy_from_user_atomic(page, i, offset, bytes);
    flush_dcache_page(page);
    status = a_ops->write_end(file, mapping, pos, bytes, copied,
            page, fsdata);
    pos += copied;
    written += copied;

    balance_dirty_pages_ratelimited(mapping);
  } while (iov_iter_count(i));
}
```

函数主体是一个while循环，找出写入影响的所有页，然后依次写入，循环里面的逻辑：

- 对于每一页，先调用 address_space 的 write_begin 做一些准备（address_space，它主要用于在内存映射的时候将文件和内存页产生关联。同样，对于缓存来讲，也需要文件和内存页进行关联，这就要用到 address_space）。
- 调用 iov_iter_copy_from_user_atomic，将写入的内容从用户态拷贝到内核态的页中。
- 调用 address_space 的 write_end 完成写操作。
- 调用 balance_dirty_pages_ratelimited，看脏页是否太多，需要写回硬盘。所谓脏页，就是写入到缓存，但是还没有写入到硬盘的页面。

**每一步细节，暂不分析**～～～

脏页回写的触发场景：

- 系统调用write发现缓存数据太多的时候。
- 用户主动调用 sync，将缓存刷到硬盘上去，最终会调用 wakeup_flusher_threads，同步脏页。
- 当内存十分紧张，以至于无法分配页面的时候，会调用 free_more_memory，最终会调用 wakeup_flusher_threads，释放脏页。
- 脏页已经更新了较长时间，时间上超过了 timer，需要及时回写，保持内存和磁盘上数据一致性。

##### 四、带缓存的读操作

带缓存的读，对应的是函数 **generic_file_buffered_read**：

```c
static ssize_t generic_file_buffered_read(struct kiocb *iocb,
    struct iov_iter *iter, ssize_t written)
{
  struct file *filp = iocb->ki_filp;
  struct address_space *mapping = filp->f_mapping;
  struct inode *inode = mapping->host;
  for (;;) {
    struct page *page;
    pgoff_t end_index;
    loff_t isize;
    page = find_get_page(mapping, index);
    if (!page) {
      if (iocb->ki_flags & IOCB_NOWAIT)
        goto would_block;
      page_cache_sync_readahead(mapping,
          ra, filp,
          index, last_index - index);
      page = find_get_page(mapping, index);
      if (unlikely(page == NULL))
        goto no_cached_page;
    }
    if (PageReadahead(page)) {
      page_cache_async_readahead(mapping,
          ra, filp, page,
          index, last_index - index);
    }
    /*
     * Ok, we have the page, and it's up-to-date, so
     * now we can copy it to user space...
     */
    ret = copy_page_to_iter(page, offset, nr, iter);
    }
}
```

- 在 generic_file_buffered_read 函数中，我们需要先找到 page cache 里面是否有缓存页。如果没有找到，不但读取这一页，还要进行预读，这需要在 page_cache_sync_readahead 函数中实现。
- 预读完了以后，再试一把查找缓存页，应该能找到了。如果第一次找缓存页就找到了，我们还是要判断，是不是应该继续预读；如果需要，就调用 page_cache_async_readahead 发起一个异步预读。
- 最后，copy_page_to_iter 会将内容从内核缓存页拷贝到用户内存空间。

##### 五、总结

直接 I/O 读写的流程是一样的，调用 ext4_direct_IO，再往下就调用块设备层了。缓存 I/O 读写的流程不一样。对于读，从块设备读取到缓存中，然后从缓存中拷贝到用户态。对于写，从用户态拷贝到缓存，设置缓存页为脏，然后启动一个线程写入块设备。

<img src="https://liuyang-picbed.oss-cn-shanghai.aliyuncs.com/img/0c49a870b9e6441381fec8d9bf3dee65.png" alt="0c49a870b9e6441381fec8d9bf3dee65" style="zoom:33%;" />



