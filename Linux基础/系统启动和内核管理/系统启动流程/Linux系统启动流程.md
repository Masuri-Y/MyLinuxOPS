## Linux系统启动流程

`CentOS5/6/7`的启动流程大致相似，细节上略为不同。

### `Linux`组成

Linux: `kernel`+`rootfs`。目前的Linux是由内核和文件系统相关的工具组合而成。

* `kernel`: 进程管理、内存管理、网络管理、驱动程序、文件系统、安全功能

* `rootfs`: 程序和`glibc`

* 库: 函数集合, function, 调用接口（头文件负责描述） 
* 程序: 二进制执行文件

内核设计流派：

* 单内核(`monolithic kernel`): `Linux`把所有功能集成于同一个程序

* 微内核(`micro kernel`): `Windows`,`Solaris`每种功能使用一个单独子系统实现

### `CentOS6`启动流程

<img src="./image-20210416092855925.png" alt="image-20210416092855925"  />

`CentOS6`启动步骤大致分为一下10个步骤：

1. 加载`BIOS`的硬件信息，获取第一个启动设备

2. 读取第一个启动设备`MBR`的引导加载程序(`grub`)的启动信息

3. 加载核心操作系统的核心信息，核心开始解压缩，并尝试驱动所有的硬件设备

4. 核心执行`init`程序，并获取默认的运行信息
5. `init`程序执行`/etc/rc.d/rc.sysinit`文件

6. 启动核心的外挂模块

7. `init`执行运行的各个批处理文件(`scripts`) 
8. `init`执行`/etc/rc.d/rc.local`

9. 执行`/bin/login`程序，等待用户登录

10. 登录之后开始以`Shell`控制主机

#### `post`加电自检

`POST`: `Power-On-Self-Test`，加电自检，是BIOS功能的一个主要部分。负责完成对CPU、主板、内存、硬盘子系统、显示子系统、串并行接口、键盘等硬件情况的检测。

* `ROM`: `BIOS`，`Basic Input and Output System`，保存着有关计算机系统最重要的基本输入输出程序，系统信息设置、开机加电自检程序和系统启动自举程序等

* `RAM`：`CMOS`互补金属氧化物半导体，保存各项参数的设定

* 按次序查找引导设备，第一个有引导程序的设备为本次启动设备

#### `bootloader`引导加载器

一般启动是是用硬盘来进行引导，所以需要从硬盘上加载引导程序。这个引导程序就是`bootloader`

`bootloader`: 引导加载器，引导程序。不同的操作系统的引导程序不同。

* `windows`: `ntloader`，仅是启动OS

* `Linux`：功能丰富，提供菜单，允许用户选择要启动系统或不同的内核版本；把用户选定的内核装载到内存中的特定空间中，解压、展开，并把系统控制权移交给内核。
  * `LILO`：`LInux LOader`，早期Linux启动使用的`bootloader`。`CentOS5`版本后淘汰，改用`GRUB`。
  * `GRUB`: `GRand Unified Bootloader`
    * `GRUB 0.X`: `GRUB Legacy`， `GRUB2`(`CentOS7`后使用)

##### `GRUB`的存放位置

`MBR`分区：第一个扇区的前446个字节，存放的就是`bootloader`，中间64字节为分区表，最后为`55AA`标记位置

需要注意的是前446个字节只是存放了`GRUB`的其中一部分程序，另一部分存放在了`/boot`分区内。

GRUB分为两个阶段

* `primary boot loader` : 分区表内的为第一阶段(1st stage)，1.5 stage 
* `secondary boot loader` : `/boot`分区内的文件为第二阶段(2nd stage)，分区文件

`GRUB`启动完毕后会将程序的启动权限交给内核

#### `kernel`内核

`GRUB`启动完毕交给内核后内核会进行初始化工作：

1. 探测可识别到的所有硬件设备

2. 加载硬件驱动程序（借助于`ramdisk`加载驱动） 
3. 以只读方式挂载根文件系统
4. 运行用户空间的第一个应用程序: `/sbin/init`

##### Linux内核特点：

* 支持模块化：`.ko`（内核对象）如：文件系统，硬件驱动，网络协议等
* 支持内核模块的动态装载和卸载

##### 内核组成部分：

* 核心文件：`/boot/vmlinuz-VERSION-release`
  * `ramdisk`: 辅助的伪根系统。对于必要的驱动文件会存放在此伪根系统中。
    * `CentOS 5`: `/boot/initrd-VERSION-release.img`
    * `CentOS 6,7`: `/boot/initramfs-VERSION-release.img`

模块文件：`/lib/modules/VERSION-release`

##### `ramdisk`：

内核中的特性之一：使用缓冲和缓存来加速对磁盘上的文件访问，并加载相应的硬件驱动

在`CentOS5`上系统启动时，内核需要先加载`ramdisk`，通过`ramdisk`进加载文件系统，速度偏慢。

在`CentOS6`后改为了`ramfs`，系统启动后内核直接加载文件系统，从而提高了速度。

`ramdisk` --> `ramfs`提高速度

`CentOS 5`: `initrd.img`

* 工 具 程 序: `mkinitrd` 

`CentOS 6，7`: `initramfs.img`

* 工具程序: `mkinitrd`, `dracut`

**注意**：一但此文件遭到破坏，系统将无法启动。此时需要进入救援模式重新制作`ramdisk`文件

```bash
# ramdisk文件的制作：
# (1)mkinitrd命令
# 为当前正在使用的内核重新制作ramdisk文件
mkinitrd /boot/initramfs-$(uname -r).img $(uname -r)
# (2)dracut命令
# 为当前正在使用的内核重新制作ramdisk文件
dracut /boot/initramfs-$(uname -r).img $(uname -r)
```

##### 系统初始化流程：

`POST` --> `BootSequence(BIOS)` --> `Bootloader(MBR)` -->`kernel(ramdisk)` --> `rootfs(readonly)` --> `init(systemd)`

`init`程序的类型：

* `SysV`: `init`, `CentOS 5`之前
  * 配置文件：`/etc/inittab`

* Upstart: `init`, `CentOS 6`
  * 配置文件: `/etc/inittab`, `/etc/init/*.conf`

* `Systemd`: `systemd`, `CentOS 7`
  * 配置文件：`/usr/lib/systemd/system`, `/etc/systemd/system`

#### `init`初始化

`init`进程再运行时有一些初始化的配置文件`centos6`及`centos6`之前的系统会读取一个专门的配置文件`/etc/inittab`，来决定运行再哪种模式下。

```bash
]# vim /etc/inittab
# inittab is only used by upstart for the default runlevel.
#
# ADDING OTHER CONFIGURATION HERE WILL HAVE NO EFFECT ON YOUR SYSTEM.
#
# System initialization is started by /etc/init/rcS.conf
#
# Individual runlevels are started by /etc/init/rc.conf
#
# Ctrl-Alt-Delete is handled by /etc/init/control-alt-delete.conf
#
# Terminal gettys are handled by /etc/init/tty.conf and /etc/init/serial.conf,
# with configuration in /etc/sysconfig/init.
#
# For information on how to write upstart event handlers, or how
# upstart works, see init(5), init(8), and initctl(8).
#
# Default runlevel. The runlevels used are:
#   0 - halt (Do NOT set initdefault to this)
#   1 - Single user mode
#   2 - Multiuser, without NFS (The same as 3, if you do not have networking)
#   3 - Full multiuser mode
#   4 - unused
#   5 - X11
#   6 - reboot (Do NOT set initdefault to this)
# 
id:3:initdefault:
# id: 表示行标识
# 3: 表示3模式
# initdefault: 表示开机进入此模式
```

运行级别：为系统运行或维护等目的而设定；`0-6`: 7个级别

* 0：关机

* 1：单用户模式(root自动登录), single, 维护模式

* 2: 多用户模式，启动网络功能，但不会启动NFS；维护模式

* 3：多用户模式，正常模式；文本界面

* 4：预留级别；可同3级别

* 5：多用户模式，正常模式；图形界面

* 6：重启

默认级别：3, 5

切换级别：`init #`

查看级别：`runlevel`; `who` 