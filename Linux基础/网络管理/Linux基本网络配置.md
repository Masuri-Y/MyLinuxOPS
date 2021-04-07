## Linux基本网络配置

将Linux主机接入到网络，需要配置网络相关设置

一般包括如下内容：

* 主机名IP/netmask

* 路由：默认网关

* DNS服务器
  * 主DNS服务器
  * 次DNS服务器
  * 第三DNS服务器

### 网卡名称配置

在Linux中所有配置网络相关的内容都和网卡相关联，所以首先需要考虑清除所配置的相关内容需要对应到哪块网卡上，而CentOS6网卡名与CentOS7有所不同。

#### CentOS6网卡名称

`CentOS5`，`CentOS6`早期所使用的网卡名为：

* `eth[0,1,2...]`：以太网
* `ppp[0,1,2...]`：ppp拨号网络

以上网卡名并非不能修改，在某些特定情况下需要将其进行修改。如`CentOS6`虚拟机在`vmware`中被克隆后，克隆生成的主机其网卡为`eth1`而非`eth0`，这是因为`vmware`在复制时会将网卡的`MAC`地址进行更改，而原来的`eth0`和MAC的对应关系依然存在，所以其无法命名为`eth0`，需要在`/etc/udev/rules.d/70-persistent-net.rules`文件中将原先的对应关系进行删除，并修改新的MAC地址所对应的网卡名称为`eth0`。然后卸载网卡驱动模块，并重新加载。具体操作流程如下：

##### CentOS6网卡名称修改

1. 修改网络接口识别并命名相关的udev配置文件：

```bash
/etc/udev/rules.d/70-persistent-net.rules
```

2. 查看网卡驱动模块：

```bash
# 以下2条二选1
dmesg |grep –i eth 
ethtool -i eth0
```

3. 卸载网卡驱动：

```bash
# 以下2条二选1
modprobe -r e1000 
rmmod e1000
```

4. 装载网卡驱动：

```bash
modprobe e1000
```

#### CentOS7网络属性配置

CentOS 6之前，网络接口使用连续号码命名：eth0、eth1等,当增加或删除网卡时，名称可能会发生变化

CentOS 7使用基于硬件，设备拓扑和设置类型命名：

(1) 网卡命名机制

* systemd对网络设备的命名方式
  * 如果Firmware或BIOS为主板上集成的设备提供的索引信息可用，且可预测则根据此索引进行命名，例如eno1
  * 如果Firmware或BIOS为PCI-E扩展槽所提供的索引信息可用，且可预测， 则根据此索引进行命名，例如ens1
  * 如果硬件接口的物理位置信息可用，则根据此信息进行命名，例如enp2s0
  * 如果用户显式启动，也可根据MAC地址进行命名，enx2387a1dc56
  * 上述均不可用时，则使用传统命名机制

* 基于BIOS支持启用biosdevname软件内置网卡：`em1`,`em2`

* pci卡：pYpX	Y：slot ,X:port

(2) 名称组成格式

* en: Ethernet 有线局域网
* wl: wlan 无线局域网
* ww: wwan无线广域网

名称类型：

* `o<index>`: 集成设备的设备索引号

* `s<slot>`: 扩展槽的索引号

* `x<MAC>`: 基于MAC地址的命名

* `p<bus>s<slot>`: enp2s1

##### `CentOS7`网卡修改为传统命令方式

在生产环境中，CentOS7的网卡命名方式不利于统一管理，通常需要将其更改为传统的命名方式

1. 编辑`/etc/default/grub`配置文件或修改`/boot/grub2/grub.cfg`(不推荐修改此文件)

   ```bash
   # 在/etc/default/grub文件中GRUB_CMDLINE_LINUX行最后添加net.ifnames=0
   GRUB_CMDLINE_LINUX="rhgb quiet net.ifnames=0
   ```

2. 为grub2生成其配置文件

   ```bash
   grub2-mkconfig -o /etc/grub2.cfg
   ```

3. 重启系统

##### `CentOS7`修改主机名

配置文件:

 `/etc/hostname`，默认没有此文件，通过`DNS`反向解析获取主机名，主机名默认为：`localhost.localdomain`

设置主机名:

```bash
hostnamectl set-hostname centos7.magedu.com
```

删除文件`/etc/hostname`，恢复主机名`localhost.localdomain`

#### CentOS7网络配置工具

图形工具：`nm-connection-editor`

字符配置tui工具：`nmtui`

命令行工具：`nmcli`

##### nmcli命令

```bash
nmcli [ OPTIONS ] OBJECT { COMMAND | help } 

device - show and manage network interfaces 
nmcli device help

connection - start, stop, and manage network connections
nmcli connection help
```

修改IP地址等属性：

```bash
nmcli connection modify IFACE [+|-]setting.property value

setting.property:
ipv4.addresses 
ipv4.gateway 
ipv4.dns1 
ipv4.method 
manual | auto
```

修改配置文件执行生效：

```bash
systemctl restart network
nmcli con reload
```

nmcli命令生效： 

```bash
nmcli con down eth0 ;nmcli con up eth0
```

##### nmcli子命令

| Command                    | Use                                                          |
| -------------------------- | ------------------------------------------------------------ |
| nmcli dev status           | List all devices                                             |
| nmcli con show             | List all connections                                         |
| nmcli con up "\<ID\>"      | Activate a connection                                        |
| nmcli con down "\<ID\>"    | Deactive a connection. The connection will restart if autoconnect is yes. |
| nmcli dev dis \<DEV\>      | Bring down an interface and temporarily  disable autoconnect. |
| nmcli net off              | Disable all managed interfaces                               |
| nmcli con add  ..          | Add a new connection.                                        |
| nmcli con mod "\<ID\>" ... | Modify a connection                                          |
| nmcli con del "\<ID\>"     | Delete a connection                                          |

##### nmcli命令与配置文件参数

| nmcli con mod                                 | ifcfg-* 文件                                             |
| --------------------------------------------- | -------------------------------------------------------- |
| ipv4.method manual                            | BOOTPROTO=none                                           |
| ipv4.method auto                              | BOOTPROTO=dhcp                                           |
| ipv4.addresses "192.168.2.1/24 192.168.2.254" | IPADDR=192.168.2.1<br>PREFIX=24<BR>GATEWAY=192.168.2.254 |
| ipv4.dns 8.8.8.8                              | DNS=8.8.8.8                                              |
| ipv4.dns-search example.com                   | DOMAIN=example.com                                       |
| ipv4.ignore-auto-dns true                     | PEERDNS=no                                               |
| connection.autoconnect yes                    | ONBOOT=yes                                               |
| connection.id eth0                            | NAME=eth0                                                |
| connection.interface-name eth0                | DEVICE=eth0                                              |
| 802-3-ethernet.mac-address ...                | HWADDR=...                                               |



NeworkManager是管理和监控网络设置的守护进程

设备即网络接口，连接是对网络接口的配置，一个网络接口可有多个连接配置，但同时只有一个连接配置生效

显示所有包括不活动连接

```bash
nmcli con show
```

显示所有活动连接

```bash
nmcli con show --active
```

显示网络连接配置

```bash
nmcli con show "System eth0"
```

显示设备状态

```bash
nmcli dev status
```

显示网络接口属性

```bash
nmcli dev show eth0
```

创建新连接default，IP自动通过dhcp获取

```bash
nmcli con add con-name default type Ethernet ifname eth0
```

删除连接

```bash
nmcli con del default
```

创建新连接static ，指定静态IP，不自动连接

```bash
nmcti con add con-name static ifname eth0 autoconnect no type Ethernet ipv4.addresses 172.25.X.10/24 ipv4.gateway 172.25.X.254
```

启用static连接配置

```bash
nmcli con up static
```

启用default连接配置

```bash
nmcli con up default
```

查看帮助

```bash
nmcli con add help
```

修改连接设置

```bash
nmcli con mod "static" connection.autoconnect no 
nmcli con mod "static" ipv4.dns 172.25.X.254 
nmcli con mod "static" +ipv4.dns 8.8.8.8
nmcli con mod "static" -ipv4.dns 8.8.8.8
nmcli con mod "static" ipv4.addresses "172.16.X.10/24 172.16.X.254" 
nmcli con mod "static" +ipv4.addresses 10.10.10.10/16
```

DNS设置，存放在/etc/resolv.conf文件中

`PEERDNS=no`表示当IP通过dhcp自动获取时，dns仍是手动设置，不自动获取等价于下面命令：

```bash
nmcli con mod "system eth0" ipv4.ignore-auto-dns yes
```

修改连接配置后，需要重新加载配置

```bash
nmcli con reload
nmcli con down "system eth0" # 可被自动激活
nmcli con up "system eth0"
nmcli dev dis eth0 # 禁用网卡，访止被自动激活
```



