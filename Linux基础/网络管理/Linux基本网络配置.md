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

##### `CentOS7`修改为传统命令方式

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

