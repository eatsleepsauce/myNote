#### 用户与权限

- **如何查看有哪些用户**
  在/etc/passwd  和 /etc/shadow中可以查看到

- **验证有没有用户**
  id  yonghu 

##### 一、用户创建和删除

- useradd  yonghu  // 创建用户，也会默认创建一个名为yonghu的组
- userdel yonghu  // 删除用户，还可以添加 -r 选项，能够删除用户的附加信息

删除用户时，附加信息也要删除，如果没有添加还需要删除下面的信息

- rm -rf /home/yonghu
- rm -rf /var/spool/mail/yonghu

- passwd yonghu  // 给用户设置密码，直接passwd就是设置当前账号到密码
- usermod  // 修改用户属性，如用户到家目录
- chage  // 修改用户属性，主要修改用户到生命周期

##### 二、用户组创建和删除

- groupadd  // 创建组织
- groupdel  // 删除组织
- usermod -G groupname yonghu  // 把用户加到groupname组织中

##### 三、用户切换

- **sudo  以自己的身份运行其他用户可以执行的命令，不需要其他用户的密码，避免了密码泄漏**，例如，普通用户临时获取root权限执行命令，需要使用visudo配置使用sudo的用户(组)

```shell
## Read drop-in files from /etc/sudoers.d (the # here does not mean a comment)
#includedir /etc/sudoers.dllow root to run any commands anywhere
root    ALL=(ALL)       ALL

## Allows members of the 'sys' group to run networking, software,
## service management apps and more.
# %sys ALL = NETWORKING, SOFTWARE, SERVICES, STORAGE, DELEGATING, PROCESSES, LOCATE, DRIVERS

## Allows people in group wheel to run all commands
# %wheel        ALL=(ALL)       ALL

## Same thing without a password
# %wheel        ALL=(ALL)       NOPASSWD: ALL

## Allows members of the users group to mount and unmount the
## cdrom as root
# %users  ALL=/sbin/mount /mnt/cdrom, /sbin/umount /mnt/cdrom

## Allows members of the users group to shutdown this system
# %users  localhost=/sbin/shutdown -h now

## Read drop-in files from /etc/sudoers.d (the # here does not mean a comment)
#includedir /etc/sudoers.d
```

- **su，su - username 使用login shell 方式切换用户(带 - 切换后进去的是自己的家目录)**

##### 四、用户和用户组的配置文件

- /etc/passwd 保存用户信息
- /etc/shadow  保存用户密码信息
- /etc/group 保存组信息

##### 五、修改用权限 （注意：root用户不受这些权限控制)

- chown  可以修改属主和属组，

  如：chown yonghu /test，将test目录属主变成yonghu； chown :group /test 属组修改成功group，注意冒号不能少

- chgrp  单独改组，不常用，被chown代替了，例如 chgrp  group /test

- chmod u=rwx xxx  属主可读可写可执行
  chmod g+w xxxxx 属组增加写权限
  chmod o-x xxxxx  其它删除执行权限
  chmod u+x xxxx  // 属主增加执行权限
  **数字方式设置**
  r = 4 w = 2 x = 1
  chmod 777 xxxx  // 属主、属组和其它可读可写可执行

**特殊权限**
了解即可，SUID SGID SBIT