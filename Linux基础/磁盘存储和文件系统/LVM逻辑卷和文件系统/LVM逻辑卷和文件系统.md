## `LVM`逻辑卷和文件系统实现

逻辑卷的组成是，集成各种空间组合成物理卷，然后集合物理卷组合成一个卷组，然后在卷组上创建逻辑卷。所以要创建逻辑卷首先需要创建物理卷，然后组合物理卷创建卷组，最后创建可以使用的逻辑卷。

### 实验环境 

1. 一个`mbr`分区的5G空间(`sdb1`)    

2. 一个`gpt`分区的5G空间(`sdc1`)  

3. 一个未分区的磁盘(`sdd`)

#### 创建实验环境  
1. 建立一个`mbr`分区的`5G`空间(`sdb1`)

```bash
[root@centos7 ~]# fdisk /dev/sdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table
Building a new DOS disklabel with disk identifier 0x8898f839.
#选择n创建一个5G新分区
Command (m for help): n
Partition type:
   p   primary (0 primary, 0 extended, 4 free)
   e   extended
Select (default p): 
Using default response p
Partition number (1-4, default 1): 
First sector (2048-20971519, default 2048): 
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-20971519, default 20971519): +5G
Partition 1 of type Linux and of size 5 GiB is set
#将标签设置为LVM
Command (m for help): t
Selected partition 1
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'
#查看将要生成的设备
Command (m for help): p

Disk /dev/sdb: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x8898f839

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048    10487807     5242880   8e  Linux LVM
#确认无误写入磁盘
Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.
Syncing disks.
#查看是否同步到内存中
[root@centos7 ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0  200G  0 disk 
├─sda1   8:1    0    1G  0 part /boot
├─sda2   8:2    0  100G  0 part /
├─sda3   8:3    0   50G  0 part /data
├─sda4   8:4    0    1K  0 part 
└─sda5   8:5    0    2G  0 part [SWAP]
sdb      8:16   0   10G  0 disk 
└─sdb1   8:17   0    5G  0 part 
sdc      8:32   0   10G  0 disk 
sdd      8:48   0   10G  0 disk 
sr0     11:0    1   10G  0 rom  
#mbr分区的5G空间创建完毕
```

2. 建立一个`gpt`分区的`5G`空间

```bash
#使用gdisk创建gpt分区
[root@centos7 ~]# gdisk /dev/sdc
GPT fdisk (gdisk) version 0.8.10

Partition table scan:
  MBR: not present
  BSD: not present
  APM: not present
  GPT: not present

Creating new GPT entries.
#创建一个新分区
Command (? for help): n
Partition number (1-128, default 1): 
First sector (34-20971486, default = 2048) or {+-}size{KMGTP}: 
Last sector (2048-20971486, default = 20971486) or {+-}size{KMGTP}: +5G
Current type is 'Linux filesystem'
#调整分区标签为LVM
Hex code or GUID (L to show codes, Enter = 8300): 8e00 
Changed type of partition to 'Linux LVM'
#查看下将要创建的分区信息
Command (? for help): p
Disk /dev/sdc: 20971520 sectors, 10.0 GiB
Logical sector size: 512 bytes
Disk identifier (GUID): 56C9AA2B-2D21-45A5-A422-882476BA60A6
Partition table holds up to 128 entries
First usable sector is 34, last usable sector is 20971486
Partitions will be aligned on 2048-sector boundaries
Total free space is 10485693 sectors (5.0 GiB)

Number  Start (sector)    End (sector)  Size       Code  Name
   1            2048        10487807   5.0 GiB     8E00  Linux LVM
#将信息写入磁盘
Command (? for help): w

Final checks complete. About to write GPT data. THIS WILL OVERWRITE EXISTING
PARTITIONS!!

Do you want to proceed? (Y/N): y
OK; writing new GUID partition table (GPT) to /dev/sdc.
The operation has completed successfully.
#查看设置是否同步入内存
[root@centos7 ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0  200G  0 disk 
├─sda1   8:1    0    1G  0 part /boot
├─sda2   8:2    0  100G  0 part /
├─sda3   8:3    0   50G  0 part /data
├─sda4   8:4    0    1K  0 part 
└─sda5   8:5    0    2G  0 part [SWAP]
sdb      8:16   0   10G  0 disk 
└─sdb1   8:17   0    5G  0 part 
sdc      8:32   0   10G  0 disk 
└─sdc1   8:33   0    5G  0 part 
sdd      8:48   0   10G  0 disk 
sr0     11:0    1   10G  0 rom  
#gpt分区sd1的5G空间创建完毕
```
3. 一个未分区的磁盘

```bash
#未分区磁盘sdd已经存在无需创建
[root@centos7 ~]# lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0  200G  0 disk 
├─sda1   8:1    0    1G  0 part /boot
├─sda2   8:2    0  100G  0 part /
├─sda3   8:3    0   50G  0 part /data
├─sda4   8:4    0    1K  0 part 
└─sda5   8:5    0    2G  0 part [SWAP]
sdb      8:16   0   10G  0 disk 
└─sdb1   8:17   0    5G  0 part 
sdc      8:32   0   10G  0 disk 
└─sdc1   8:33   0    5G  0 part 
sdd      8:48   0   10G  0 disk 
sr0     11:0    1   10G  0 rom  
```

***至此实验环境搭建完毕***

### 创建`LVM`

1. 创建物理卷（`PV`）

```bash
# 将3块磁盘空间全部转换为物理卷（PV）
[root@centos7 data]# pvcreate /dev/sd{b1,c1,d}
  Physical volume "/dev/sdb1" successfully created.
  Physical volume "/dev/sdc1" successfully created.
  Physical volume "/dev/sdd" successfully created.
  [root@centos7 data]# pvs
  PV         VG  Fmt  Attr PSize  PFree 
  /dev/sdb1  vg0 lvm2 a--  <5.00g <5.00g
  /dev/sdc1  vg0 lvm2 a--  <5.00g <5.00g
  /dev/sdd       lvm2 ---  10.00g 10.00g

```
2. 创建卷组（`vg`）

```bash
# 出于后期增加卷组考虑此处先使用2块磁盘空间组成vg  
# 将mbr分区的sdb1和gpt分区的sdc1组成卷组
[root@centos7 data]# vgcreate vg0 /dev/sd{b1,c1}
  Volume group "vg0" successfully created
[root@centos7 data]# vgs
  VG  #PV #LV #SN Attr   VSize VFree
  vg0   2   0   0 wz--n- 9.99g 9.99g

```
3. 创建逻辑卷（`lv`）

```bash
# 创建一个6G的逻辑卷,一个3G的逻辑卷
[root@centos7 data]# lvcreate -n test1 -L 6G vg0
  Logical volume "test1" created.
[root@centos7 data]# lvcreate -n test2 -L 3G vg0
  Logical volume "test2" created.
[root@centos7 data]# lvs
  LV    VG  Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  test1 vg0 -wi-a----- 6.00g                                                    
  test2 vg0 -wi-a----- 3.00g            
```
***至此2个逻辑卷已经创建完成。接下来需要在逻辑卷上创建文件系统了***
### 创建文件系统
文件系统有很多种类，此处以`xfs`和`ext4`为例进行建立  
#### `ext4`文件系统
1. 在`/dev/vg0/test1`上建立`ext4`

```bash
[root@centos7 data]# mkfs.ext4 -L 'test1' /dev/vg0/test1
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=test1
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
393216 inodes, 1572864 blocks
78643 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=1610612736
48 block groups
32768 blocks per group, 32768 fragments per group
8192 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (32768 blocks): done
Writing superblocks and filesystem accounting information: done 

```
2. 将`ext4`文件系统挂载至`/data/test1`下

```bash
[root@centos7 data]# mkdir test1
[root@centos7 data]# mount /dev/vg0/test1 /data/test1/
[root@centos7 data]# cd /data/test1/
[root@centos7 test1]# ls
lost+found
[root@centos7 test1]# findmnt /data/test1
TARGET      SOURCE                FSTYPE OPTIONS
/data/test1 /dev/mapper/vg0-test1 ext4   rw,relatime,data=ordered
```

### `xfs`文件系统
1. 在`/dev/vg0/tset2`上建立`xfs`文件系统。

```bash
[root@centos7 test1]# findmnt /data/test1
TARGET      SOURCE                FSTYPE OPTIONS
/data/test1 /dev/mapper/vg0-test1 ext4   rw,relatime,data=ordered
[root@centos7 test1]# mkfs.xfs -L "test2" /dev/vg0/test2
meta-data=/dev/vg0/test2         isize=512    agcount=4, agsize=196608 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=786432, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```
2. 将文件系统挂载至`/data/test2`下

```bash
[root@centos7 test1]# mkdir /data/test2
[root@centos7 test1]# mount /dev/vg0/test2 /data/test2
[root@centos7 test1]# findmnt /data/test2
TARGET      SOURCE                FSTYPE OPTIONS
/data/test2 /dev/mapper/vg0-test2 xfs    rw,relatime,attr2,inode64,noquota
```
***2个文件系统已经创建完毕,由于后续实验需求现在两个磁盘上分别创建几个文件***

```bash
[root@centos7 test1]# touch file{1..10}
[root@centos7 test1]# ls
file1  file10  file2  file3  file4  file5  file6  file7  file8  file9  lost+found
[root@centos7 test1]# cd ../test2
[root@centos7 test2]# touch file{a..j}
[root@centos7 test2]# ls
filea  fileb  filec  filed  filee  filef  fileg  fileh  filei  filej
```
### 逻辑卷的容量增加和减少
##### 一、分别对逻辑卷test1及test2增加1G容量  
1. 查看卷组的相关信息

```bash
[root@centos7 test2]# vgdisplay
  --- Volume group ---
  VG Name               vg0
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               9.99 GiB
  PE Size               4.00 MiB
  Total PE              2558
  Alloc PE Size       2304 / 9.00 GiB
  Free  PE / Size       254 / 1016.00 MiB
  VG UUID               4CadZ4-W3dV-kjNa-leu2-u6jd-YMeq-o54JVU
```
***此时卷组中只剩下不足1G容量需要增加物理卷才能满足需求***  

2. 对`vg0`增加物理卷，将刚才未加入卷组的`sdd`加入卷组

```bash
[root@centos7 test2]# vgextend vg0 /dev/sdd
  Volume group "vg0" successfully extended
#查看卷组情况，此时已经有10.99G空闲
[root@centos7 test2]# vgs
  VG  #PV #LV #SN Attr   VSize   VFree  
  vg0   3   2   0 wz--n- <19.99g <10.99g
```
3. 对ext文件系统的逻辑卷`test1`进行扩容

```bash
[root@centos7 test2]# lvextend -r -L +1G /dev/vg0/test1
  Size of logical volume vg0/test1 changed from 6.00 GiB (1536 extents) to 7.00 GiB (1792 extents).
  Logical volume vg0/test1 successfully resized.
resize2fs 1.42.9 (28-Dec-2013)
Filesystem at /dev/mapper/vg0-test1 is mounted on /data/test1; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/mapper/vg0-test1 is now 1835008 blocks long.
#查看是否成功
[root@centos7 test2]# lvs
  LV    VG  Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  test1 vg0 -wi-ao---- 7.00g                                                    
  test2 vg0 -wi-ao---- 3.00g      
```
4.对`xfs`文件系统的逻辑卷t`est2`进行扩容

```bash
[root@centos7 test2]# lvextend -r -L +1G /dev/vg0/test2
  Size of logical volume vg0/test2 changed from 3.00 GiB (768 extents) to 4.00 GiB (1024 extents).
  Logical volume vg0/test2 successfully resized.
meta-data=/dev/mapper/vg0-test2  isize=512    agcount=4, agsize=196608 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=786432, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 786432 to 1048576
#查看是否扩容成空
[root@centos7 test2]# lvs
  LV    VG  Attr       LSize Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  test1 vg0 -wi-ao---- 7.00g                                                    
  test2 vg0 -wi-ao---- 4.00g    
```
5. 查看文件系统内的文件是否丢失

```bash
[root@centos7 test2]# cd /data
[root@centos7 data]# ls test1 test2
test1:
file1  file10  file2  file3  file4  file5  file6  file7  file8  file9  lost+found

test2:
filea  fileb  filec  filed  filee  filef  fileg  fileh  filei  filej
```
***对逻辑卷做扩容可以在线操作并且不会造成数据丢失的***
##### 二、对逻辑卷做缩减
逻辑卷是否能缩减取决于文件系统是否支持，`XFS`文件系统不支持缩减，所以此处以`ext4`为例，
将逻辑卷`test1`缩减至`4G`，***对文件系统做缩减需要先对其内的数据做备份防止数据丢失***  

1. 逻辑卷缩减首先要将设备离线

```bash
[root@centos7 data]# umount /data/test1
```
2. 对逻辑卷缩减前首先先要将文件系统的空间缩减

```bash
#缩减文件系统至4G
[root@centos7 data]# resize2fs /dev/vg0/test1 4G
resize2fs 1.42.9 (28-Dec-2013)
Please run 'e2fsck -f /dev/vg0/test1' first.
#缩减文件前需要先对文件系统做检查，所以此处提示先对文件系统做检查
[root@centos7 data]# e2fsck -f /dev/vg0/test1
e2fsck 1.42.9 (28-Dec-2013)
Pass 1: Checking inodes, blocks, and sizes
Pass 2: Checking directory structure
Pass 3: Checking directory connectivity
Pass 4: Checking reference counts
Pass 5: Checking group summary information
test1: 21/458752 files (0.0% non-contiguous), 68479/1835008 blocks
[root@centos7 data]# resize2fs /dev/vg0/test1 4G
resize2fs 1.42.9 (28-Dec-2013)
Resizing the filesystem on /dev/vg0/test1 to 1048576 (4k) blocks.
The filesystem on /dev/vg0/test1 is now 1048576 blocks long.
#缩减成功
```
3. 对逻辑卷进行缩减

```bash
[root@centos7 data]# lvreduce -L 4G /dev/vg0/test1 
  WARNING: Reducing active logical volume to 4.00 GiB.
  THIS MAY DESTROY YOUR DATA (filesystem etc.)
Do you really want to reduce vg0/test1? [y/n]: y
  Size of logical volume vg0/test1 changed from 7.00 GiB (1792 extents) to 4.00 GiB (1024 extents).
  Logical volume vg0/test1 successfully resized.
#逻辑卷缩减成功
```
4. 挂载文件系统，查看内部数据

```bash
[root@centos7 data]# mount /dev/vg0/test1 /data/test1
[root@centos7 data]# findmnt /data/test1
TARGET      SOURCE                FSTYPE OPTIONS
/data/test1 /dev/mapper/vg0-test1 ext4   rw,relatime,data=ordered
[root@centos7 data]# cd test1
[root@centos7 test1]# ls
file1  file10  file2  file3  file4  file5  file6  file7  file8  file9  lost+found
```