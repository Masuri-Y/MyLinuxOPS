## 网络组team

网络组是`centos7`上新出的一个技术，它的作用和bonding类似，是将多个网卡聚合在一起方法，从而实现冗错和提高吞吐量，不同于旧版中bonding技术，网路组提供更好的性能和扩展性，它是由内核驱动和`teamd`守护进程实现。  

网路组可以工作在多种方式(runner)    

```bash
 broadcast   
 roundrobin     
 activebackup   
 loadbalance   
 lacp (implements the 802.3ad Link Aggregation Control Protocol)   
```

* 启动网络组接口不会自动启动网络组中的`port`接口
* 启动网络组接口中的`port`接口总会自动启动网络组接口
* 禁用网络组接口会自动禁用网络组中的`port`接口
* 没有`port`接口的网络组接口可以启动静态`IP`连接
* 启用`DHCP`连接时，没有`port`接口的网络组会等待`port`接口的加入

#### 创建网络组相关命令

1. 创建网络组接口

   ```bash
   nmcli con add type team con-name CNAME ifname INAME [config JSON] 
   
   # CNAME 连接名，INAME 接口名
   # JSON 指定runner方式，格式：'{"runner": {"name": "METHOD"}}'
   # METHOD 可以是broadcast, roundrobin,activebackup, loadbalance, lacp
   ```

2. 创建port接口

   ```bash
   nmcli con add type team-slave con-name CNAME ifname INAME master
   TEAM
   # CNAME 连接名
   # INAME 网络接口名
   # TEAM 网络组接口名
   # 连接名若不指定，默认为team-slave-IFACE
   ```

3. 将原先设备禁用，启用team

   ```bash
   nmcli dev dis INAME
   nmcli con up CNAME
   # INAME 设备名 CNAME 网络组接口名或port接口
   ```

   

### team实现

准备一台`CentOS7`主机，网卡2块。

#### 创建网络组   
1. 创建网路网

```bash
[root@centos7 ~]# nmcli connection add con-name team0 ifname team0 type team ipv4.method manual ipv4.addresses 192.168.172.100 config '{"runner":{"name":"loadbalance"}}'
Connection 'team0' (24db0099-b9fa-4aae-ace0-9421e3c69278) successfully added.
```
2. 添加物理网卡

分别将`ens33`和`ens37`添加至网路组内

```bash
[root@centos7 ~]# nmcli connection add con-name team0-ens33 ifname ens33 type team-slave master team0
Connection 'team0-ens33' (0d00650a-e379-4c70-9f62-ba268af1a208) successfully added.
[root@centos7 ~]# nmcli connection add con-name team0-ens37 ifname ens37 type team-slave master team0
Connection 'team0-ens37' (2916ab1f-2e3c-477b-aaaf-52dfaecaaeb7) successfully added.
```
3. 将物理网卡和网络组关联起来

```bash
#由于刚才只是将物理网卡添加至网络组内，所以此时team0-ens33和team0-ens37并未启用
[root@centos7 ~]# nmcli connection
NAME                UUID                                  TYPE      DEVICE 
ens33               fca2f13f-7310-4595-bbb1-e6d0e3662aff  ethernet  ens33  
team0               24db0099-b9fa-4aae-ace0-9421e3c69278  team      team0  
virbr0              803d85ba-4e80-470f-bcf5-1b22b5653026  bridge    virbr0 
Wired connection 1  3f019cd5-7685-3368-960c-101e35cd6ce7  ethernet  ens37  
team0-ens33         0d00650a-e379-4c70-9f62-ba268af1a208  ethernet  --     
team0-ens37         2916ab1f-2e3c-477b-aaaf-52dfaecaaeb7  ethernet  --   
#将网络组内的物理网卡关联起来
[root@centos7 ~]# nmcli connection up team0-ens33
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/7)
[root@centos7 ~]# nmcli connection up team0-ens37
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/8)
#此时team-ens33和team-ens37都已经启用，网络组创建成功
[root@centos7 ~]# nmcli connection 
NAME                UUID                                  TYPE      DEVICE 
team0               24db0099-b9fa-4aae-ace0-9421e3c69278  team      team0  
team0-ens33         0d00650a-e379-4c70-9f62-ba268af1a208  ethernet  ens33  
team0-ens37         2916ab1f-2e3c-477b-aaaf-52dfaecaaeb7  ethernet  ens37  
virbr0              803d85ba-4e80-470f-bcf5-1b22b5653026  bridge    virbr0 
ens33               fca2f13f-7310-4595-bbb1-e6d0e3662aff  ethernet  --     
Wired connection 1  3f019cd5-7685-3368-960c-101e35cd6ce7  ethernet  --   
```
4. 查看网络组状态

```bash
[root@centos7 ~]# teamdctl team0 state
setup:
  runner: loadbalance
ports:
  ens33
    link watches:
      link summary: up
      instance[link_watch_0]:
        name: ethtool
        link: up
        down count: 0
  ens37
    link watches:
      link summary: up
      instance[link_watch_0]:
        name: ethtool
        link: up
        down count: 0
```
***
#### 网络组的删除
1. 删除相关配置文件  

由于`nmcli`命令在执行时会自动生成网卡的配置文件，所以删除网路组时需要将相应的配置文件进行删除

```bash
[root@centos7 ~]# rm -vf /etc/sysconfig/network-scripts/ifcfg-team0*
removed ‘/etc/sysconfig/network-scripts/ifcfg-team0’
removed ‘/etc/sysconfig/network-scripts/ifcfg-team0-ens33’
removed ‘/etc/sysconfig/network-scripts/ifcfg-team0-ens37’
```
2. 取消相关网卡的关联

```bash
#将网络组中的ens33及ens37取消关联
[root@centos7 ~]# nmcli connection down team0-ens33
Connection 'team0-ens33' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/7)
[root@centos7 ~]# nmcli connection down team0-ens37
Connection 'team0-ens37' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/8)
#删除链接ens33和ens37
[root@centos7 ~]# nmcli connection delete team0-ens33
Connection 'team0-ens33' (0d00650a-e379-4c70-9f62-ba268af1a208) successfully deleted.
[root@centos7 ~]# nmcli connection delete team0-ens37
Connection 'team0-ens37' (2916ab1f-2e3c-477b-aaaf-52dfaecaaeb7) successfully deleted.
```
3. 将网路组删除

```bash
#先禁用网路组
[root@centos7 ~]# nmcli connection down team0 
Connection 'team0' successfully deactivated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/6)
#将网路组删除
[root@centos7 ~]# nmcli connection delete team0 
Connection 'team0' (24db0099-b9fa-4aae-ace0-9421e3c69278) successfully deleted.
```
此时网路组已经从主机上删除
```bash
[root@centos7 ~]# nmcli connection 
NAME                UUID                                  TYPE      DEVICE 
ens33               fca2f13f-7310-4595-bbb1-e6d0e3662aff  ethernet  ens33  
virbr0              803d85ba-4e80-470f-bcf5-1b22b5653026  bridge    virbr0 
Wired connection 1  3f019cd5-7685-3368-960c-101e35cd6ce7  ethernet  ens37  

```