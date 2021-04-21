## `Systemd`进程

`CentOS7`上系统中的第一个进程已经不是早期的`init`进程，他使用了`systemd`来代替了`init`。`CentOS7`上的`init`只是一个软连接指向了`systemd`

```bash
[root@mylinuxops ~]# ll /sbin/init
lrwxrwxrwx 1 root root 22 Apr  8 04:55 /sbin/init -> ../lib/systemd/systemd
```

`Systemd`: 系统启动和服务器守护进程管理器，负责在系统启动或运行时，激活系统资源，服务器进程和其它进程

`Systemd`新特性

* 系统引导时实现服务并行启动

* 按需启动守护进程

* 自动化的服务依赖关系管理

* 同时采用socket式与D-Bus总线式激活服务
* 系统状态快照

### `Unit`

`unit`表示不同类型的`systemd`对象，通过配置文件进行标识和配置；文件中主要包含了系统服务、监听`socke`t、保存的系统快照以及其它与`init`相关的信息

配置文件：

* `/usr/lib/systemd/system`: 每个服务最主要的启动脚本设置，类似于之前的`/etc/init.d/`

* `/run/systemd/system`: 系统执行过程中所产生的服务脚本，比上面目录优先运行。

* `/etc/systemd/system`: 管理员建立的执行脚本，类似于`/etc/rcN.d/Sxx`的功能，比上面目录优先运行。

以上三个目录一般只关注`/usr/lib/systemd/system`，其余两个目录都由系统自动进行维护。

#### `Unit`类型

系统中所有可用的unit类型可以使用`systemctl`命令进行查看。

```bash
# 查看系统中所有可用的Unit类型
[root@mylinuxops system]# systemctl -t help
Available unit types:
service		# 文件扩展名为.service, 用于定义系统服务
socket		# .socket, 用于标识进程间通信用的socket文件，也可在系统启动时， 延迟启动服务，实现按需启动
target		# 文件扩展名为.target，用于模拟实现运行级别，类似于早期的inittab中0-6
device		# .device, 用于定义内核识别的设备
mount		# .mount, 定义文件系统挂载点
automount	# .automount，文件系统的自动挂载点
swap		# .swap, 用于标识swap设备
timer
path		# .path，用于定义文件系统中的一个文件或目录使用,常用于当文件系统变化时，延迟激活服务，如：spool 目录
slice
scope
```

以上`unit`一般主要关注`service`, `target`, `socket`

#### `Systemd`的关键特性

1. 基于`socket`的激活机制: `socket`与服务程序分离。服务不启动，socket监听端口，当有用户访问时，再激活服务。

2. 基于`d-bus`的激活机制: 

3. 基于`device`的激活机制:  

4. 基于`path`的激活机制: 

5. 系统快照: 保存各`unit`的当前状态信息于持久存储设备中

6. 向后兼容`sysv init`脚本

### `systemctl`命令

`Systemd`所有的管理是由一个统一的二进制程序`systemctl`进行管理。由于其为二进制程序所以无法对其进行扩展和更改。

`Systemd`的命令能兼容早期的服务脚本

#### 服务管理类命令

命令格式：

```bash
systemctl COMMAND name.service
```

启动：

```bash
service name start 
# or
systemctl start name.service
```

停止：

```bash
service name stop
# or
systemctl stop name.service
```

重启：

```bash
service name restart
# or
systemctl restart name.service
```

状态：

```bash
service name status
# or
systemctl status name.service
```

条件式重启：已启动才重启，否则不做操作

```bash
service name condrestart
# or
systemctl try-restart name.service
```

重载或重启服务：先加载，再启动

```bash
systemctl reload-or-restart name.service
```

重载或条件式重启服务：

```bash
systemctl reload-or-try-restart name.service
```

禁止自动和手动启动：

```bash
systemctl mask name.service
```

取消禁止：

```bash
systemctl unmask name.service
```

#### 服务查看类命令

查看某服务当前激活与否的状态： 

```bash
systemctl is-active name.service
```

查看所有已经激活的服务：

```bash
systemctl list-units --type|-t service
```

查看所有服务：

```bash
systemctl list-units --type service --all|-a
```

`chkconfig`命令的对应关系：

* 设定某服务开机自启：

  ```bash
  chkconfig name on
  # systemd中命令
  systemctl enable name.service
  ```

* 设定某服务开机禁止启动：

  ```bash
  chkconfig name off
  # systemd中命令
  systemctl disable name.service
  ```

* 查看所有服务的开机自启状态：

  ```bash
  chkconfig --list
  # systemd中命令
  systemctl list-unit-files --type service
  ```

* 用来列出该服务在哪些运行级别下启用和禁用

  ```bash
  chkconfig sshd –list ==>
  # systemd中命令
  ls /etc/systemd/system/*.wants/sshd.service
  ```

* 查看服务是否开机自启：

```bash
systemctl is-enabled name.service
```

#### 其它命令

查看服务的依赖关系：

```bash
systemctl list-dependencies name.service
```

杀掉进程：

```bash
systemctl kill unitname
```

#### 服务的状态

使用`systemctl`命令可以显示所有服务的运行状态

```bash
systemctl list-unit-files --type service --all
```

常见状态：

* `loaded`: Unit配置文件已处理

* `active(running)`: 一次或多次持续处理的运行

* `active(exited)`: 成功完成一次性的配置

* `active(waiting)`: 运行中，等待一个事件

* `inactive`: 不运行

* `enabled`: 开机启动

* `disabled`: 开机不启动

* `static`: 开机不启动，但可被另一个启用的服务激活

### `Service Unit`文件格式

`/etc/systemd/system`: 系统管理员和用户使用

`/usr/lib/systemd/system`: 发行版打包者使用

以 “#” 开头的行后面的内容会被认为是注释

相关布尔值，1、yes、on、true 都是开启，0、no、off、false 都是关闭

时间单位默认是秒，所以要用毫秒（ms）分钟（m）等须显式说明



#### `service unit file`文件通常由三部分组成：

* `[Unit]`: 定义与Unit类型无关的通用选项；用于提供unit的描述信息、unit行为及依赖关系等

* `[Service]`: 与特定类型相关的专用选项；此处为Service类型

* `[Install]`: 定义由`systemctl enable`以及`systemctl	disable`命令在实现服务启用或禁用时用到的一些选项

#### Unit段的常用选项

* `Description`: 描述信息

* `After`: 定义`unit`的启动次序，表示当前`unit`应该晚于哪些`unit`启动，其功能与`Before`相反

* `Requires`: 依赖到的其它`units`，强依赖，被依赖的`units`无法激活时，当前`unit`也无法激活

* `Wants`: 依赖到的其它`units`，弱依赖

* `Conflicts`: 定义`units`间的冲突关系

#### `Service`段的常用选项

* `Type`：定义影响`ExecStart及`相关参数的功能的`unit`进程启动类型

* `simple`：默认值，这个`daemon`主要由`ExecStart`接的指令串来启动，启动后常驻于内存中

* `forking`：由`ExecStart`启动的程序透过`spawns`延伸出其他子程序来作为此`daemon`的主要服务。原生父程序在启动结束后就会终止

* `oneshot`：与`simple`类似，不过这个程序在工作完毕后就结束了，不会常驻在内存中。

* `dbus`：与simple类似，但这个daemon必须要在取得一个`D-Bus`的名称后，才会继续运作.因此通常也要同时设定`BusNname= `才行

* `notify`: 在启动完成后会发送一个通知消息。还需要配合`NotifyAccess`来让`Systemd`接收消息

* `idle`: 与`simple`类似，要执行这个`daemon`必须要所有的工作都顺利执行完毕后才会执行。这类的`daemon`通常是开机到最后才执行即可的服务

* `EnvironmentFile`: 环境配置文件

* `ExecStart`: 指明启动`unit`要运行命令或脚本的绝对路径

* `ExecStartPre`: `ExecStart`前运行

* `ExecStartPost`: `ExecStart`后运行

* `ExecStop`：指明停止`unit`要运行的命令或脚本

* `Restart`：当设定`Restart=1`时，则当次`daemon`服务意外终止后，会再次自动启动此服务

#### Install段的常用选项

* `Alias`: 别名，可使用`systemctl command Alias.service`

* `RequiredBy`: 被哪些units所依赖，强依赖

* `WantedBy`: 被哪些units所依赖，弱依赖

* `Also`: 安装本服务的时候还要安装别的相关服务

注意：对于新创建的`unit`文件，或者修改了的`unit`文件，要通知`systemd`重载此配置文件,而后可以选择重启

```bash
systemctl daemon-reload name.service
```

#### `Service Unit`使用示例

1. 编写service unit文件

```
[root@mylinuxops testdir]# vim /usr/lib/systemd/system/backup.service
[Unit]
Description=backup /etc
Requires=atd.service
[Service]
Type=simple
ExecStart=/bin/bash -c "echo /testdir/bak.sh | at now"
[Install]
WantedBy=multi-user.target
```

2. 编写备份脚本

```bash
[root@mylinuxops testdir]# vim /testdir/bak.sh
#!/bin/bash
tar cvf /data/etc-`date +%F`.tar /etc

# 添加执行权限
[root@mylinuxops testdir]# chmod +x bak.sh
```

3. 启动服务

```bash
[root@mylinuxops testdir]# systemctl start backup.service
```

4. 验证

```bash
[root@mylinuxops testdir]# ls /data/
etc.tar
```

#### 运行级别

target units：

unit配置文件：.target

```bash
ls /usr/lib/systemd/system/*.target
systemctl list-unit-files --type target	--all
```

运行级别：

```bash
0 ==> runlevel0.target, poweroff.target
1 ==> runlevel1.target, rescue.target
2 ==> runlevel2.target, multi-user.target
3 ==> runlevel3.target, multi-user.target
4 ==> runlevel4.target, multi-user.target
5 ==> runlevel5.target, graphical.target
6 ==> runlevel6.target, reboot.target
```

查看依赖性：

```bash
systemctl list-dependencies graphical.target
```

级别切换：

```bash
# CentOS6
init N 
# CentOS7
systemctl isolate name.target

# 示例：
systemctl isolate multi-user.target
```

注：只有`/lib/systemd/system/*.target`文件中`AllowIsolate=yes`才能切换(修改文件需执行`systemctl daemon-reload`才能生效)

查看target：

```bash
runlevel;who -r
# CentOS7 命令
systemctl list-units --type target
```

获取默认运行级别：

```bash
# centos6 通过文件查看
/etc/inittab
# centos7 命令查看
systemctl get-default
```

修改默认级别：

```bash
# CentOS6为修改文件
/etc/inittab
# CentOS7命令修改
systemctl set-default name.target
```

示例：

```bash
[root@mylinuxops data]# systemctl set-default multi-user.target
[root@mylinuxops data]# ls -l /etc/systemd/system/default.target
lrwxrwxrwx. 1 root root 41 Mar  1 13:15 /etc/systemd/system/default.target -> /usr/lib/systemd/system/multi-user.target
```

切换至紧急救援模式： 

```bash
systemctl rescue
```

切换至emergency模式： 

```bash
systemctl emergency
```

#### 其他命令

传统命令`init`，`poweroff`，`halt`，`reboot`都成为`systemctl`的软链接

关机：`systemctl halt`、`systemctl poweroff` 

重启：`systemctl reboot`

挂起：`systemctl suspend` 

休眠：`systemctl hibernate`

休眠并挂起：`systemctl hybrid-sleep`

#### `CentOS7` 启动引导顺序

* `UEFi`或`BIOS`初始化，运行`POST`开机自检

* 选择启动设备

* 引导装载程序, `centos7`是`grub2`

* 加载装载程序的配置文件：
  * `/etc/grub.d/`
  * `/etc/default/grub`
  * `/boot/grub2/grub.cfg`

* 加载`initramfs`驱动模块

* 加载内核选项

* 内核初始化，`centos7`使用`systemd`代替`init`

* 执行`initrd.target`所有单元，包括挂载`/etc/fstab`

* 从`initramfs`根文件系统切换到磁盘根目录

* `systemd`执行默认`target`配置，配置文件`/etc/systemd/system/default.target`

*  `systemd`执行`sysinit.target`初始化系统及`basic.target`准备操作系统

* `systemd`启动`multi-user.target`下的本机与服务器服务

* `systemd`执行`multi-user.target`下的`/etc/rc.d/rc.local`

* `Systemd`执行`multi-user.target`下的`getty.target`及登录服务

* `systemd`执行`graphical需要的服务

#### `CentOS7`救援模式进入

设置内核参数，只影响当次启动

* 启动时，在`linux16`行后添加
  * `systemd.unit=desired.target`
  * `systemd.unit=emergency.target`
  * `systemd.unit=rescue.target`

* `rescue.target`: 比`emergency`支持更多的功能，例如日志等

* `systemctl default`进入默认`target`

####  `CentOS7`口令破解

* 启动时任意键暂停启动

* 按e键进入编辑模式

* 将光标移动`linux16`开始的行，添加内核参数`rd.break`

* 按`ctrl-x`启动

* ```bash
  # 默认为只读方式挂在根，需要改为读写挂在
  mount –o remount,rw /sysroot
  # 切根到系统的根目录
  chroot /sysroot
  # 修改密码
  passwd root
  # 如果默认开启了SeLinux则需要执行此步骤
  touch /.autorelabel
  # 退出切根
  exit
  # 重启服务器
  reboot
  ```

