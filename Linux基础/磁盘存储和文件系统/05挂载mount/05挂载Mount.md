## 挂载`Mount`

挂载：将额外文件系统与根文件系统某现存的目录建立起关联关系，进而使得此目录做为其它文件访问入口的行为

卸载：为解除此关联关系的过程

把设备关联挂载点：mount Point 

```bash
mount 挂载设备 挂载点
```

卸载时：可使用设备，也可以使用挂载点

```bash
umount	设备名|挂载点
```

挂载点下原有文件在挂载完成后会被临时隐藏

挂载点目录一般为空

### `mount`命令挂载文件系统

挂载方法：`mount DEVICE MOUNT_POINT`

mount：通过查看`/etc/mtab`文件显示当前已挂载的所有设备

```bash
mount [-fnrsvw] [-t vfstype] [-o options] device dir
```

`device`：指明要挂载的设备

1. 设备文件：例如`/dev/sda5`

2. 卷标：`-L 'LABEL'`, 例如 `-L 'MYDATA'`

3. `UUID`, `-U 'UUID'`, 例如 `-U '0c50523c-43f1-45e7- 85c0-a126711d406e'`

4. 伪文件系统名称：`proc`, `sysfs`, `devtmpfs`, `configfs`

`dir`：挂载点

1. 事先存在，建议使用空目录

2. 进程正在使用中的设备无法被卸载

#### `mount`常用选项

* `-t vsftype`: 指定要挂载的设备上的文件系统类型

* `-r` `readonly`，只读挂载

* `-w` read and write, 读写挂载

* `-n` 不更新`/etc/mtab`，`mount`不可见

* `-a` 自动挂载所有支持自动挂载的设备(定义在了`/etc/fstab`文件中，且挂载选项中有auto功能)

* `-L 'LABEL'` 以卷标指定挂载设备

* `-U 'UUID'` 以`UUID`指定要挂载的设备

* `-B, --bind` 绑定目录到另一个目录上

查看内核追踪到的已挂载的所有设备

```bash
cat /proc/mounts
```

* -o options：(挂载文件系统的选项)，多个选项使用逗号分隔
  * `async` : 异步模式	
  * `sync`: 同步模式,内存更改时，同时写磁盘
  * `atime/noatime`: 包含目录和文件
  * `diratime/nodiratime` 目录的访问时间戳
  * `auto/noauto`: 是否支持自动挂载,是否支持-a选项
  * `exec/noexec`: 是否支持将文件系统上运行应用程序
  * `dev/nodev`: 是否支持在此文件系统上使用设备文件
  * `suid/nosuid`:是否支持`suid`和`sgid`权限
  * `remount`: 重新挂载
  * `ro`: 只读	
  * `rw`: 读写
  * `user/nouser`: 是否允许普通用户挂载此设备，`/etc/fstab`使用
  * `acl`: 启用此文件系统上的`acl`功能
  * `loop`: 使用loop设备
  * `defaults`: 相当于`rw`, `suid`, `dev`, `exec`, `auto`, `nouser`, `async`

### 取消挂载

取消挂载步骤如下

1. 查看挂载情况，如果有挂载则显示挂载信息，没有则不显示。

   ```bash
   findmnt MOUNT_POINT|device
   ```

2. 查看正在访问指定文件系统的进程，如果设备正在被访问则执行3否则执行4

   ```bash
   # 以下两条命令为查询挂的设备是否有用户正在访问
   lsof MOUNT_POINT
   fuser -v MOUNT_POINT
   ```

3. 终止所有在正访问指定的文件系统的进程

   ```bash
   fuser -km MOUNT_POINT
   ```

4. 卸载

   ```bash
   # 卸载挂载设备，以下二选1
   umount DEVICE 
   umount MOUNT_POINT
   ```

   

