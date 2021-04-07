# 多网卡bonding后再进行桥接

1. 创建bond0配置文件

```bash
[root@localhost ~]# vim /etc/sysconfig/network-scripts/ifcfg-bond0
BOOTPROTO=static
NAME=bond0
DEVICE=bond0
ONBOOT=yes
BONDING_MASTER=yes
BONDING_OPTS="mode=1 miimon=100"
BRIDGE=br0
```

2. 配置第一块网卡

```bash
[root@localhost ~]# vim /etc/sysconfig/network-scripts/ifcfg-ens33 
TYPE=Ethernet
BOOTPROTO=static
NAME=ens33
DEVICE=ens33
ONBOOT=yes
MASTER=bond0
SLAVE=yes
```

3. 配置第二块网卡

```bash
[root@localhost ~]# vim /etc/sysconfig/network-scripts/ifcfg-ens37
TYPE=Ethernet
BOOTPROTO=static
NAME=ens37
DEVICE=ens37
ONBOOT=yes
MASTER=bond0
SLAVE=yes
```

4. 创建桥接网卡

```bash
[root@localhost ~]# vim /etc/sysconfig/network-scripts/ifcfg-br0
TYPE=Bridge
BOOTPROTO=static
NAME=br0
DEVICE=br0
ONBOOT=yes
IPADDR=172.20.27.20
PREFIX=16
GATEWAY=172.20.0.1
DNS=114.114.114.114
```

5. 重启服务前安装bridge-utils工具，否则桥接会失败

```bash
[root@localhost ~]# yum install bridge-utils -y
```

6. 重启网络服务

```bash
[root@localhost ~]# systemctl restart network
```

7. 查看网卡信息

```bash
[root@localhost ~]# ifconfig
bond0: flags=5187<UP,BROADCAST,RUNNING,MASTER,MULTICAST>  mtu 1500
        ether 00:0c:29:ec:ff:de  txqueuelen 1000  (Ethernet)
        RX packets 82828  bytes 7152167 (6.8 MiB)
        RX errors 0  dropped 9  overruns 0  frame 0
        TX packets 85  bytes 11029 (10.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

br0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.20.27.20  netmask 255.255.0.0  broadcast 172.20.255.255
        inet6 fe80::a071:bfff:fe15:1a3e  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:ec:ff:de  txqueuelen 1000  (Ethernet)
        RX packets 1751  bytes 146176 (142.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 89  bytes 9719 (9.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens33: flags=6211<UP,BROADCAST,RUNNING,SLAVE,MULTICAST>  mtu 1500
        ether 00:0c:29:ec:ff:de  txqueuelen 1000  (Ethernet)
        RX packets 96937  bytes 8252388 (7.8 MiB)
        RX errors 0  dropped 53  overruns 0  frame 0
        TX packets 1321  bytes 177696 (173.5 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens37: flags=6211<UP,BROADCAST,RUNNING,SLAVE,MULTICAST>  mtu 1500
        ether 00:0c:29:ec:ff:de  txqueuelen 1000  (Ethernet)
        RX packets 6294  bytes 491553 (480.0 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 16  bytes 1823 (1.7 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 22  bytes 1848 (1.8 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 22  bytes 1848 (1.8 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

服务重启以后`ens33`，`ens37`，`bond0`，`br0`的`mac`地址均相同，`IP`地址在`br0`上。