## `proc`和`sys`目录

### `/proc`目录

`/proc`目录内存放了许多和进程相关的内容

内核把自己内部状态信息及统计信息，以及可配置参数通过`proc`伪文件系统加以输出

帮助：`man proc`

参数：

* 只读：输出信息

* 可写：可接受用户指定“新值”来实现对内核某功能或特性的配置

#### `/proc/sys`目录

`sys`目录内定义了众多可以修改和配置的设置，如启用路由功能。可以使用`sysctl`命令或者重定向的方式来进行修改

(1) `sysctl`命令用于查看或设定此目录中诸多参数

```bash
sysctl -w path.to.parameter=VALUE
sysctl -w kernel.hostname=www.mylinuxops.com
```

(2) echo命令通过重定向方式也可以修改大多数参数的值

```bash
echo "VALUE" > /proc/sys/path/to/parameter
echo "websrv" > /proc/sys/kernel/hostname
```

注意：使用命令修改的参数无法保存，`/proc`是存在与内存中的系统重启后将丢失。所以需要将所命令所执行的配配置写入`sysctl`的配置文件中。

##### `sysctl`命令

默认配置文件：`/etc/sysctl.conf`

(1) 设置某参数

```bash
sysctl -w parameter=VALUE
```

(2) 通过读取配置文件设置参数 

```bash
sysctl -p [/path/to/conf_file]
```

(3) 查看所有生效参数

```bash
sysctl -a
```

常用的几个参数：

```bash
# ipv4的转发，1开启，0关闭
net.ipv4.ip_forward
# icmp报文忽略，1忽略，0不忽略
net.ipv4.icmp_echo_ignore_all
# 清空buffer和cache,
vm.drop_caches
```

### `/sys`目录

`sysfs`：为用户使用的伪文件系统，输出内核识别出的各硬件设备的相关属性信息，也有内核对硬件特性的设定信息；有些参数是可以修改的，用于调整硬件工作特性

`udev`通过此路径下输出的信息动态为各设备创建所需要设备文件，`udev`是运行用户空间程序

专用工具：`udevadmin`, `hotplug`

`udev`为设备创建设备文件时，会读取其事先定义好的规则文件，一般在`/etc/udev/rules.d`及`/usr/lib/udev/rules.d`目录下