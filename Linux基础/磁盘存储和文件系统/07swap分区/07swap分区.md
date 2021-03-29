## swap分区


swap交换分区是系统RAM的补充，Swap分区支持虚拟内存。当没有足够的RAM保存系统处理的数据时会将数据写入swap分区，当系统缺乏swap空间时，内核会因RAM内存耗尽而终止进程。配置过多swap空间会造成存储设备处于分配状态但闲置，造成浪费，过多swap空间还会掩盖内存泄露，所以swap分区可以根据物理内存的大小来分配，物理内存过小时可以设置为物理内存的2倍，随着物理内存的逐渐增大，swap的倍数可以逐渐递减。

在实际生产中不建议使用swap分区来当内存使用，毕竟磁盘的性能要弱于内存数倍

以下为演示swap分区的各种创建方法：

### 一、将分区创建为`swap`

##### 1.划分分区

新增一块硬盘`sdb`，对`sdb`进行分区
```bash
[root@centos7 ~]# fdisk /dev/sdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x9446b510.

Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): 
Using default response p
Partition number (1-4, default 1): 
First sector (2048-41943039, default 2048): 
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-41943039, default 41943039): +1G
Partition 1 of type Linux and of size 1 GiB is set

Command (m for help): t
Selected partition 1
Hex code (type L to list all codes): 82
Changed type of partition 'Linux' to 'Linux swap / Solaris'

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
```
##### 2.创建`swap`文件系统

```bash
[root@centos7 ~]# mkswap /dev/sdb1
Setting up swapspace version 1, size = 1048572 KiB
no label, UUID=7d411e84-9e3e-416a-99a1-b81973b0001b
```
##### 3.将swap文件系统写入`fstab`

```bash
[root@centos7 ~]# vim /etc/fstab
#
# /etc/fstab
# Created by anaconda on Tue Mar  5 21:07:19 2019
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=45490aa4-cf29-420d-a606-af32688b6707 /                       xfs     defaults        0 0
UUID=17dcd896-b7cf-48d0-b8bd-4c0b0f2c62b2 /boot                   xfs     defaults        0 0
UUID=4b6e1813-2c46-402a-869a-02cbbcb76ade /data                   xfs     defaults        0 0
UUID=0995b444-48c1-4423-92bc-2deda0d3c082 swap                    swap    defaults        0 0
UUID=7d411e84-9e3e-416a-99a1-b81973b0001b swap                  swap    defaults   0 0
```
注意：pri=5为swap分区的优先级，由于此处使用的为新硬盘，磁盘最外圈的读写速度为最快所以将优先级调为5，比系统安装时的swap等级高
##### 4.挂载swap

```bash
[root@centos7 ~]# swapon /dev/sdb1
[root@centos7 ~]# swapon -s
Filename				Type		Size	Used	Priority
/dev/sda5                              	partition	2097148	0	-2
/dev/sdb1                              	partition	1048572	0	-3
```
注意：由于此处使用的为新硬盘，磁盘最外圈的读写速度最快所以将优先级调至比sda5高，此处数字越大优先级越高
##### 5.调优先级

首先需要将挂载的swap卸载
```bash
[root@centos7 ~]# swapoff /dev/sdb1
```
修改/etc/fstab文件,将default修改为5，比sda5的-2高就行。
```bash
UUID=7d411e84-9e3e-416a-99a1-b81973b0001b swap                  swap    pri=5   0 0
```
重新挂载swap
```bash
[root@centos7 ~]# swapon -a
[root@centos7 ~]# swapon -s
Filename				Type		Size	Used	Priority
/dev/sda5                              	partition	2097148	0	-2
/dev/sdb1                              	partition	1048572	0	5
```
****
### 二、将文件创建为swap
由于某些场景下系统在创建时没有创建swap分区，并且系统上也没有多余的空间创建swap分区，此时可以考虑使用用文件来创建swap分区，具体操作方法如下：
##### 1.创建一个文件
使用`dd`创建一个文件
```bash
[root@centos7 ~]# dd if=/dev/zero of=swapfile bs=1M count=1024
1024+0 records in
1024+0 records out
1073741824 bytes (1.1 GB) copied, 5.02973 s, 213 MB/s
```
##### 2.对`swapfile`文件创建`swap`文件系统
```bash
[root@centos7 ~]# mkswap swapfile 
Setting up swapspace version 1, size = 1048572 KiB
no label, UUID=c7c8d93d-862b-48b1-aeb3-1390dbf2d6db
```
##### 3.挂载`swapfile`文件
```bash
[root@centos7 ~]# swapon -a
swapon: /root/swapfile: insecure permissions 0644, 0600 suggested.
[root@centos7 ~]# swapon -s
Filename				Type		Size	Used	Priority
/dev/sda5                              	partition	2097148	0	-2
/dev/sdb1                              	partition	1048572	0	5
/root/swapfile                         	file	1048572	0	-3
```
注意：此处的报警为权限问题，0644的权限会造成其他用户可以读其中的内容所以此处需要将其权限设置为600
```bash
[root@centos7 ~]# chmod 600 swapfile 
[root@centos7 ~]# ll swapfile 
-rw------- 1 root root 1073741824 Mar 26 05:48 swapfile
```

注意：由于使用的是文件来作为`swap`，在使用到`swap`时系统需要先去磁盘上找到此文件，所以速度较慢，另外文件在磁盘上所在的位置是未知的，所以所创建的`Swap`的性能未必有创建系统时所建的`swap`性能好。