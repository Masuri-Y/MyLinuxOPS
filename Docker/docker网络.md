## `docker`网络

`docker`提供了4种网络:

1. 桥网络：docker中默认为`docker0 NAT`，可以进行改变，可以设置成隔离桥，仅主机桥，路由桥等等。
2. 共享桥（联盟式网络）：每个容器是靠内核中的名称空间来映射，`IPC`, `NAT`, `Mount`, `PID`, `User`, `UTS`。这些名称空间是可以被共享的。我们可以让每个容器内的`Mount`,`User`,`PID`进行独立，让`IPC`, `NAT`, `UTS`共享。这样可以实现让其进程，用户和文件系统独立，而网络空间是同一组。两个容器内通信可以直接通过`IPC`进行通信，对外可以使用一个主机名进行访问。这样类似于回到了早期的虚拟机。
3. 共享宿主机网络（host网络）：容器有自己的`Mount`, `User`, `Pid`, 但是其同时使用的宿主机的网络名称空间，也就是说其使用的是宿主机的网卡设备。当容器监听在某个套接字上，实际上则是监听在宿主机的套接字上。如容器监听在`*:80`，则意味着监听在宿主机的`*:80`上。
4. 空网络（none网路）：某些容器只是用来计算数据，数据运行完毕后就删除容器，此种容器无需使用到网络。所以在运行时，不分配网路。

查看当前主机上能被docker所使用的网络

```bash
[root@CentOS8 ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
a9dd9148cc49   bridge    bridge    local
5ee81e8f87a3   host      host      local
9bd2843207da   none      null      local
```

### 封闭式网络实现

创建一个封闭式网络，使用`--network`参数来指定

```bash
[root@CentOS8 ~]# docker run --name tinyweb2 --rm --network none -it  masuri/myimg /bin/sh
/ # ifconfig -a
lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

```

### `bridge`网络实现

`docker`容器创建时默认就是bridge网络

```bash
[root@CentOS8 ~]# docker run --name tinyweb2 --rm --network bridge -it  masuri/myimg /bin/sh
/ # ifconfig -a
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:06
          inet addr:172.17.0.6  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:6 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:516 (516.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

可以使用`network inspect`命令来查看桥所关联的网络

```bash
[root@CentOS8 ~]# docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "a9dd9148cc499a95b78db267f1e68ec6df531a339c816812e28bbe8e8f9dcc63",
        "Created": "2021-04-27T14:11:26.720523679+08:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "367070d7060bc6335a62860e27e80eb165bbfd3076999cf40673acfbfa0dfcf2": {
                "Name": "c2",
                "EndpointID": "34c51821392b836a479ad7907c351ac8d7da20088dae9a965599ba11f443317e",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "76c823f4e87295de37dc738879d7eab73a4870af4c57382b5e0f03ed8346bb49": {
                "Name": "tinyweb2",
                "EndpointID": "2e7e2c047e7ff02450a12ab78c6b0825f241937921feab0be354c672709c389d",
                "MacAddress": "02:42:ac:11:00:06",
                "IPv4Address": "172.17.0.6/16",
                "IPv6Address": ""
            },
            "7e27895e233ee44db06d2e2514903a0e91881f12e99bce2bd5629ad507225f69": {
                "Name": "b1",
                "EndpointID": "f7f0e391fd29a1c7dda7d304859579f0651cfd0655293364bac9eee7d2dd8896",
                "MacAddress": "02:42:ac:11:00:04",
                "IPv4Address": "172.17.0.4/16",
                "IPv6Address": ""
            },
            "9876cf5e0a27ce6b85a12c83fb4cf51de5ddc74b5e6a9a7a45d873e34b45cad5": {
                "Name": "web",
                "EndpointID": "c5dab9abad14f83948ae6df2a8ec6624e5fbe718b873b5ef6cf27b0d0406641d",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            },
            "ce4c0ab76c8ef8e9523b1b918c635cdc286609b49199e6d96e6d9f291ec72646": {
                "Name": "tinyweb",
                "EndpointID": "602f973090d5f2dc83464ecda71d5b11431dc86f9643adf3a7adb1057cfe9b69",
                "MacAddress": "02:42:ac:11:00:05",
                "IPv4Address": "172.17.0.5/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

### 联盟式网络实现

1. 创建出第一个桥接式容器

```bash
[root@CentOS8 ~]# docker run --name tinyweb2 --rm --network bridge -it  masuri/myimg /bin/sh
/ # ifconfig -a
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:06
          inet addr:172.17.0.6  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:6 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:516 (516.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
          
/ # hostname
76c823f4e872
```

2. 创建第二个容器，加入第一个容器的网络。需要使用`--netwok container:`前缀来指定需要加入哪个容器的网络。

```bash
[root@CentOS8 ~]# docker run --name joinedtw1 -it --rm --network container:tinyweb2 masuri/myimg /bin/sh
/ # ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:06
          inet addr:172.17.0.6  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:16 errors:0 dropped:0 overruns:0 frame:0
          TX packets:4 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1216 (1.1 KiB)  TX bytes:280 (280.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

/ # hostname
76c823f4e872
```

### `host`网络实现

容器创建时使用`--network host`来指定使用`host`网络

```bash
[root@CentOS8 ~]# docker run --name tinyweb2 --network host --rm -it masuri/myimg /bin/sh
/ # ifconfig
docker0   Link encap:Ethernet  HWaddr 02:42:8D:88:5D:52
          inet addr:172.17.0.1  Bcast:172.17.255.255  Mask:255.255.0.0
          inet6 addr: fe80::42:8dff:fe88:5d52/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1679 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2999 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:80438 (78.5 KiB)  TX bytes:14751045 (14.0 MiB)

eth0      Link encap:Ethernet  HWaddr 52:54:00:73:18:F5
          inet addr:172.16.11.63  Bcast:172.16.11.255  Mask:255.255.255.0
          inet6 addr: fe80::a131:a51e:a1c9:b27e/64 Scope:Link
          inet6 addr: fe80::a164:dff:80b1:6133/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:150390 errors:0 dropped:434 overruns:0 frame:0
          TX packets:73539 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:340005632 (324.2 MiB)  TX bytes:7096199 (6.7 MiB)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

veth1d95d8e Link encap:Ethernet  HWaddr C6:E2:FE:A4:78:5F
          inet6 addr: fe80::c4e2:feff:fea4:785f/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:8 errors:0 dropped:0 overruns:0 frame:0
          TX packets:25 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:714 (714.0 B)  TX bytes:1820 (1.7 KiB)

veth70e93df Link encap:Ethernet  HWaddr 8A:94:F3:33:83:09
          inet6 addr: fe80::8894:f3ff:fe33:8309/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:18 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:1300 (1.2 KiB)

veth96b8ea7 Link encap:Ethernet  HWaddr CA:6F:EA:B6:B0:E5
          inet6 addr: fe80::c86f:eaff:feb6:b0e5/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:7 errors:0 dropped:0 overruns:0 frame:0
          TX packets:30 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:1273 (1.2 KiB)  TX bytes:2178 (2.1 KiB)

vetha75e35d Link encap:Ethernet  HWaddr D6:3B:12:F4:32:F5
          inet6 addr: fe80::d43b:12ff:fef4:32f5/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:19 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:1370 (1.3 KiB)
          
/ # hostname
CentOS8
```

### 其他网络相关参数

1. 在容器运行时使用`--hostname`为容器注入主机名

```bash
[root@CentOS8 ~]# docker run --name bbox2 -it --hostname mylinuxops --rm busybox:latest /bin/sh
/ # hostname
mylinuxops
```

2. 在容器运行时使用`--add-host`注入主机名的解析，`--add-host`选项可以重复使用多次，注入多条解析记录。

```bash
[root@CentOS8 ~]# docker run --name bbox2 -it --hostname bbox.mylinuxops.com --rm --add-host gw.mylinuxops.com:172.16.11.63 --add-host js.mylinuxops.com:172.16.11.61 busybox
/ # cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.16.11.63    gw.mylinuxops.com
172.16.11.61    js.mylinuxops.com
172.17.0.2      bbox.mylinuxops.com bbox
```

3. 容器运行时，使用`--dns`选项向容器中注入`DNS`服务器，使用`--dns-search`注入搜索域.

```bash
[root@CentOS8 ~]# docker run --name bbox2 -it --hostname bbox.mylinuxops.com --rm --add-host gw.mylinuxops.com:172.16.11.63 --dns 172.16.11.1 --dns-search mylinuxops.com busybox
/ # cat /etc/resolv.conf
search mylinuxops.com
nameserver 172.16.11.1
```

4. `-p`选项使用格式

   1. `-p <containerPort>`: 将指定的容器端口`<containerPort>`映射到主机所有地址的一个动态端口

   ```bash
   # 将容器内的80端口映射到宿主机的动态端口
   [root@CentOS8 ~]# docker run --name tinyweb -d -p 80 nginx:alpine
   8c21ba59714f85abb6f79712da783f36c05baf63befe1eb51f8d642530991f55
   
   # 将容器内的80端口映射到宿主及的动态端口49153
   [root@CentOS8 ~]# docker ps -a
   CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                                     NAMES
   8c21ba59714f   nginx:alpine   "/docker-entrypoint.…"   5 seconds ago   Up 3 seconds   0.0.0.0:49153->80/tcp, :::49153->80/tcp   tinyweb
   
   # 查看那宿主机上的端口
   [[root@CentOS8 ~]# ss -tnl | grep "49153"
   LISTEN    0         20480              0.0.0.0:49153            0.0.0.0:*
   LISTEN    0         20480                 [::]:49153               [::]:*
   ```

   2. `-p <hostPort>:<containerPort>`: 将容器的端口`<containerPort>`映射到指定的宿主机端口`<hostPort>`

   ```bash
   # 将容器内的80端口映射给宿主机的80端口
   [root@CentOS8 ~]# docker run --name tinyweb -d --rm -p 80:80 nginx:alpine
   969db1943b12c8d828c0121cfd83b3093f5a430ce4b7ace703673a7b27b92030
   
   # 查看容器的运行状态。
   [root@CentOS8 ~]# docker ps -a
   CONTAINER ID   IMAGE          COMMAND                  CREATED          STATUS          PORTS                               NAMES
   969db1943b12   nginx:alpine   "/docker-entrypoint.…"   19 seconds ago   Up 17 seconds   0.0.0.0:80->80/tcp, :::80->80/tcp   tinyweb
   
   # 宿主计算上80端口已经开启
   [root@CentOS8 ~]# ss -tnl | grep 80
   LISTEN    0         20480              0.0.0.0:80               0.0.0.0:*
   LISTEN    0         20480                 [::]:80                  [::]:*
   ```

   3. `-p <ip>::<containerPort>`: 将指定的容器端口`<containerPort>`映射至主机指定`<ip>`的动态端口

   ```bash
   [root@CentOS8 ~]# docker run --name tinyweb --rm -d -p 172.16.11.63::80 nginx:alpine
   3665182641d7b8ac622269bc37c3d7c7116c2e4137d684bfbcf4d399d973ea09
   [root@CentOS8 ~]# docker port tinyweb
   80/tcp -> 172.16.11.63:49153
   ```
   
   4. `-p <ip>:<hostPort>:<containerPort>`: 将指定的容器端口`<containerPort>`映射至主机指定`<ip>`的端口`<hostPort>`
   
   ```bash
   [root@CentOS8 ~]# docker run --name tinyweb -d --rm -p 172.16.11.63:80:80 nginx:alpine
   6daeebc3bb1fe22d1cc163786a306b0ca618643814800d01a14a17ec92f5af77
   [root@CentOS8 ~]# docker port tinyweb
   80/tcp -> 172.16.11.63:80
   ```
   

### `docker`自建网络

1. 使用`docker network create`创建出网络

```bash
[root@CentOS8 ~]# docker network create --subnet 10.10.0.0/24 --gateway 10.10.0.1 mybr0
52d5f694537615125d090970bc87e5fea017088e9b5fabce2c9d786946db1dfb
[root@CentOS8 ~]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
a9dd9148cc49   bridge    bridge    local
5ee81e8f87a3   host      host      local
52d5f6945376   mybr0     bridge    local
9bd2843207da   none      null      local

# br开头的网卡为新建的网络
[root@CentOS8 ~]# ifconfig
br-52d5f6945376: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 10.10.0.1  netmask 255.255.255.0  broadcast 10.10.0.255
        ether 02:42:b6:02:c0:e7  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

2. 更改网卡名

```bash
[root@CentOS8 ~]# docker run --name tinyweb --rm --network mybr0 -it nginx:alpine /bin/sh
557b49eef280d80ea4be651e5b2cfeeeff711ea60e39f21ebaeffc3c45c5d506
# 查看IP
/bin/sh: ifocnfig: not found
/ # ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
52: eth0@if52: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:0a:0a:00:02 brd ff:ff:ff:ff:ff:ff
    inet 10.10.0.2/24 brd 10.10.0.255 scope global eth0
       valid_lft forever preferred_lft forever
```

3. 将`tinyweb`这个容器连接到默认的桥接网络

```bash
[root@CentOS8 ~]# docker network connect bridge tinyweb
# 拆除网络需要使用 docker network disconnect命令
[root@CentOS8 ~]# docker exec tinyweb ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
52: eth0@if53: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:0a:0a:00:02 brd ff:ff:ff:ff:ff:ff
    inet 10.10.0.2/24 brd 10.10.0.255 scope global eth0
       valid_lft forever preferred_lft forever
# 默认桥接网络也加入容器内了
54: eth1@if55: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth1
       valid_lft forever preferred_lft forever
```

### 修改`docker`默认的`docker0`桥

修改`docker`默认的`docker0`桥需要修改其配置文件：`/etc/docker/daemon.json`

```bash
# 在配置文件中加入一行bip来指定docker0桥网关地址
[root@CentOS8 ~]# vim /etc/docker/daemon.json
{
  "registry-mirrors": ["https://gy97ij1m.mirror.aliyuncs.com","https://registry.docker-cn.com"],
  "bip": "172.31.0.1/16"   # 新的docker0桥地址
}
```

重启服务

```bash
[root@CentOS8 ~]# systemctl daemon-reload
[root@CentOS8 ~]# systemctl restart docker
```

重启后查看`docker0`地址

```bash
[root@CentOS8 ~]# ifconfig docker0
docker0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 172.31.0.1  netmask 255.255.0.0  broadcast 172.31.255.255
        inet6 fe80::42:8dff:fe88:5d52  prefixlen 64  scopeid 0x20<link>
        ether 02:42:8d:88:5d:52  txqueuelen 0  (Ethernet)
        RX packets 1686  bytes 81613 (79.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3026  bytes 14753301 (14.0 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

