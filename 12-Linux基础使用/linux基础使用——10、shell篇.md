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



