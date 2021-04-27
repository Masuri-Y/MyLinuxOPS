# docker简介
## 容器
统称来说，容器是一种工具，指的是可以装下其它物品的工具，以方便人类归纳放置物品、存储和异地运输，具体来说比如人类使用的衣柜、行李箱、背包等可以成为容器，但今天我们所说的容器是一种 IT 技术。
容器技术是虚拟化、云计算、大数据之后的一门新兴的并且是炙手可热的新技术，容器技术提高了硬件资源利用率、方便了企业的业务快速横向扩容、实现了业务宕机自愈功能，因此未来数年会是一个容器愈发流行的时代，这是一个对于IT行业来说非常有影响和价值的技术，而对于 IT 行业的从业者来说，熟练掌握容器技术无疑是一个很有前景的行业工作机会。
容器技术最早出现在freebsd叫做 jail。
## docker简介
首先Docker是一个在2013年开源的应用程序并且是一个基于go语言编写是一个开源的pass服务(Platform as a Service，平台即服务的缩写)，go语言是由google开发，docker公司最早叫dotCloud后由于Docker开源后大受欢迎就将公司改名为Docker Inc，总部位于美国加州的旧金山，Docker是基于linux内核实现，Docker最早采用LXC技术(LinuX Container 的简写，LXC是Linux原生支持的容器技术，可以提供轻量级的虚拟化，可以说docker就是基于LXC发展起来的，提供LXC的高级封装，发展标准的配置方法)，而虚拟化技术 KVM(Kernelbased Virtual Machine)基于模块实现，Docker后改为自己研发并开源的runc技术运行容器。
Docker相比虚拟机的交付速度更快，资源消耗更低，Docker采用客户端/服务端架构，使用远程API来管理和创建Docker容器，其可以轻松的创建一个轻量级的、可移植的、自给自足的容器，docker的三大理念是 build(构建)、ship(运输)、run(运行)，Docker遵从apache 2.0协议，并通过（namespace及cgroup等）来提供容器的资源隔离与安全保障等，所以Docker容器在运行时不需要类似虚拟机（空运行的虚拟机占用物理机6-8%性能）的额外资源开销，因此可以大幅提高资源利用率，总而言之Docker是一种用了新颖方式实现的轻量级虚拟机。类似于VM但是在原理和应用上和VM的差别还是很大的，并且docker的专业叫法是应用容器(Application Container)。
### Docker的组成
Docker 主机(HOST): 一个物理机或虚拟机，运行于Docker服务进程和容器
Docker 服务端(Server): Docker守护进程，运行docker容器。
Docker 客户端(Client): 客户端使用docker命令或其他功能调用Docker API
Docker 镜像(images): 镜像可以理解为创建实例使用的模板
Docker 容器(Container): 容器时从镜像生成对外提供服务的一个或一组服务。
### Docker对比虚拟机
资源利用率更高: 一台物理机可以运行数百个容器，但是一般只能运行数十个虚拟机
开销更小: 不需要启动单独的虚拟机占用硬件资源
启动速度更快: 可以在数秒内完成启动。
使用虚拟机是为了更好的实现服务运行环境隔离，每个虚拟机都有独立的内核，虚拟化可以实现不同操作系统的虚拟机，但是通常一个虚拟机只能运行一个服务，很明显资源利用率比较低且造成不必要的性能损耗，我们创建虚拟机的目的是为了运行应用程序，比如Nginx、PHP、Tomcat等Web程序，使用虚拟机无疑带俩了一些不必要的资源开销，但是容器技术则基本减少了中间环节带来较大的性能提升。
但是，一个宿主机运行了N个容器，多个容器带来的一以下问题怎么解决：
1.如何保证每个容器都有不同文件系统并且互不影响?
2.一个docker主进程内的各个容器都是其子进程，那么实现同一个主进程下不同类型的子进程，各个进程间通信能相互访问内存数据么？
3.每个容器怎么解决IP及端口的分配
4.多个容器的主机名能一样么？
5.每个容器都要不要有root用户？怎么解决账户重名问题？


## Linux Namespace技术：
namespace是Linux系统的的底层概念，在内核层实现，即有一些不同类型的命名空间被部署在内核内，各个docker容器运行在同一个dockers宿主机进程并且公用一个宿主机系统内核，各个dockers容器运行在宿主机的用户空间，每个容器都要有类似于虚拟机一样的相互隔离的运行空间，但是容器技术时在一个进程内实现循行指定服务的运行环境，并且还可以保护宿主机内核不受其他进程的干扰和影响，如文件系统空间、网络空间、进程空间等、主要通过以下技术实现容器运行空间的相互隔离：
|隔离类型|功能|系统调用参数|内核版本|
|:-|:-|:-|:-|
|MNT Namespace(mount)|提供磁盘挂载点和文件系统的隔离能力|CLONE_NEWS|Linux 2.4.19|
|IPC Namespace(Inter-Process Communication)|提供进程间通信的隔离能力|CLONE_NEWIPC|Linux 2.6.19|
|UTS Namespace(UNIX Timesharing System)|提供主机名隔离能力|CLONE_NEWUTS|Linux 2.6.19|
|PID Namespace(Process Identification)|提供进程隔离能力|CLONCE_NEWPID|Linux2.6.24|
|Net Namespace(network)|提供网络隔离能力|CLONE_NEWNET|Linux 2.6.29|
|User Namespace(user)|提供用户隔离能力|CLONE_NEWUSER|Linux 3.8|

### MNT Namespace
每个容器都要有独立的根文件系统有独立的用户空间，以实现在容器里启动服务并且使用容器的运行环境，即一个宿主机是Ubuntu的服务器，可以在里面启动一个centos运行环境的容器并且在容器里面启动一个Nginx服务，此Nginx运行时使用的运行换环境就是centos系统目录的运行环境，但是在容器里面是不能访问宿主机的资源，宿主机是使用了chroot技术把容器锁定到了一个指定的目录里面
例如：
```bash
/var/lib/containerd/io.containerd.runtime.v1.linux/moby/容器ID
```
示例：
启动3个容器，查看目录
```bash
root@mylinuxops:~# docker run -d --name nginx-1 -p 80:80 nginx
e23cb322256dfb8dc3033bc2eb7efbad7cb594fd5440968135e6ca879025f02f
root@mylinuxops:~# docker run -d --name nginx-2 -p 81:80 nginx
d4dd4830462c0ef61168728e5f6aedd2c86faf2374ae00f980b3a1f8fe061566
root@mylinuxops:~# docker run -d --name nginx-3 -p 82:80 nginx
b255239688bc201fae0a4fdfff91e76f6fbe983b87d1c643edb50c41c83560b7
```
进入容器，查看容器的根文件系统
```bash
root@mylinuxops:~# docker exec -it b255239688bc bash
root@b255239688bc:/# ls /
bin  boot  dev	etc  home  lib	lib64  media  mnt  opt	proc  root  run  sbin  srv  sys  tmp  usr  var
#查看容器的系统信息
root@b255239688bc:/# cat /etc/os-release 
PRETTY_NAME="Debian GNU/Linux 9 (stretch)"
NAME="Debian GNU/Linux"
VERSION_ID="9"
VERSION="9 (stretch)"
ID=debian
HOME_URL="https://www.debian.org/"
SUPPORT_URL="https://www.debian.org/support"
BUG_REPORT_URL="https://bugs.debian.org/"
```
### IPC Namespace
一个容器内的进程间通信，允许一个容器内不同进程的内存、缓存等数据访问，但是不能跨容器访问其他容器的数据。
### UTS Namespace
UTS namespace(Unix Timesharing System 包含了运行内核的名称、版本、底层体系结构类型等信息)用于系统标识，其中包含了hostname和域名domainname，它使得一个容器拥有属于自己的hostname标识，这个主机名标识独立于宿主机系统和其上的其他容器。
示例：
```bash
#查看宿主机内核
root@mylinuxops:~# uname -a
Linux mylinuxops 4.15.0-54-generic #58-Ubuntu SMP Mon Jun 24 10:55:24 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
#进入容器查看内核
root@mylinuxops:~# docker exec -it nginx-1 bash         #进入容器内部
root@e23cb322256d:/# uname -a               #查看内核版本
Linux e23cb322256d 4.15.0-54-generic #58-Ubuntu SMP Mon Jun 24 10:55:24 UTC 2019 x86_64 GNU/Linux               #容器内部的内核和宿主机的内核相同
root@e23cb322256d:/# hostname       #查看容器的主机名
e23cb322256d                        #容器有自己独立的主机名
```

### PID Namesapce
Linux系统中，有一个PID为1的进程（init/systemd）是其他所有进程的父进程，在每个容器内也有一个父进程来管理其下属的子进程，那么多个容器的进程通过PID namespace进程隔离（比如PID编号重复、容器内的主机成生成与回收子进程等）
示例：
进入容器内查看容器的进程
```bash
root@mylinuxops:~# docker pull centos


```

### Net Namespace
每个容器都类似于虚拟机一样有自己的网卡、监听端口、TCP/IP协议栈等，Docker使用network namespace启动一个vethx接口，这样你的容器将有拥有它自己的桥接IP地址，通常是docker0，而docker0实质就是Linux的虚拟网桥，网桥是在OSI七层模型的数据链路层的网络设备，通过mac地址对网络进行划分，并且在不同网络直接传递数据。
示例：
```bash
#启动3个容器
root@mylinuxops:~# for i in {1..3};do docker run -d --name nginx-$i -p 8${i}:80  nginx; done
b13ed304f010ed4acd23e5a3c8f89c3aaf9aa8a4d24aba1453fc41696a558040
3c9dabd0b717f337dfa3c533845107cebc8ce29162b29146315d3139326c002b
c0fd384a3a51ea583f9c05b4e625e3a0f45a6d03d2c741e6f6c085b0b17110c9
#查看网卡的信息
root@mylinuxops:~# ifconfig
docker0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::42:73ff:fe76:6a7b  prefixlen 64  scopeid 0x20<link>
        ether 02:42:73:76:6a:7b  txqueuelen 0  (Ethernet)
        RX packets 108  bytes 3024 (3.0 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 27  bytes 2226 (2.2 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.27.10  netmask 255.255.255.0  broadcast 192.168.27.255
        inet6 fe80::20c:29ff:fe1b:c184  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:1b:c1:84  txqueuelen 1000  (Ethernet)
        RX packets 182858  bytes 250983216 (250.9 MB)
        RX errors 0  dropped 131  overruns 0  frame 0
        TX packets 28867  bytes 2095806 (2.0 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 372  bytes 34620 (34.6 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 372  bytes 34620 (34.6 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
#以下3张以veth开头的网卡为容器的桥接网卡
vethceb8fb1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::9069:e9ff:fe78:d8ea  prefixlen 64  scopeid 0x20<link>
        ether 92:69:e9:78:d8:ea  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 9  bytes 726 (726.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vethe036446: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::4:2ff:fe8f:a6d9  prefixlen 64  scopeid 0x20<link>
        ether 02:04:02:8f:a6:d9  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 9  bytes 726 (726.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

vethe370c87: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet6 fe80::9cb6:94ff:fefc:37aa  prefixlen 64  scopeid 0x20<link>
        ether 9e:b6:94:fc:37:aa  txqueuelen 0  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 11  bytes 906 (906.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
#查看宿主机桥接设备
root@mylinuxops:~# brctl show
bridge name	bridge id		STP enabled	interfaces
docker0		8000.024273766a7b	no		vethceb8fb1
						            	vethe036446
						            	vethe370c87
#3张veth网卡全都桥接在了docker0这张网卡上
```
容器向外网以及内网访问是通过IPtables规则来进行定义的
```bash
#查看NAT表上的规则
root@mylinuxops:~# iptables -t nat -nvL
Chain PREROUTING (policy ACCEPT 3 packets, 449 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    7  5655 DOCKER     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT 2 packets, 401 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 10 packets, 776 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER     all  --  *      *       0.0.0.0/0           !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT 10 packets, 776 bytes)         #源地址转换让容器通过宿主机地址向外网进行访问
 pkts bytes target     prot opt in     out     source               destination         
    0     0 MASQUERADE  all  --  *      !docker0  172.17.0.0/16        0.0.0.0/0           
    0     0 MASQUERADE  tcp  --  *      *       172.17.0.2           172.17.0.2           tcp dpt:80
    0     0 MASQUERADE  tcp  --  *      *       172.17.0.3           172.17.0.3           tcp dpt:80
    0     0 MASQUERADE  tcp  --  *      *       172.17.0.4           172.17.0.4           tcp dpt:80

Chain DOCKER (2 references)             #目标地址转换，以实现外网访问宿主机内部的容器
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  docker0 *       0.0.0.0/0            0.0.0.0/0           
    0     0 DNAT       tcp  --  !docker0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:81 to:172.17.0.2:80
    0     0 DNAT       tcp  --  !docker0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:82 to:172.17.0.3:80
    0     0 DNAT       tcp  --  !docker0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:83 to:172.17.0.4:80


#查看filter表的规则
root@mylinuxops:~# iptables -nvL
Chain INPUT (policy ACCEPT 750 packets, 80690 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy DROP 0 packets, 0 bytes)                  #此为转发规则
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER-USER  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 DOCKER-ISOLATION-STAGE-1  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
    0     0 DOCKER     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 ACCEPT     all  --  docker0 docker0  0.0.0.0/0            0.0.0.0/0           

Chain OUTPUT (policy ACCEPT 499 packets, 49638 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain DOCKER (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     tcp  --  !docker0 docker0  0.0.0.0/0            172.17.0.2           tcp dpt:80
    0     0 ACCEPT     tcp  --  !docker0 docker0  0.0.0.0/0            172.17.0.3           tcp dpt:80
    0     0 ACCEPT     tcp  --  !docker0 docker0  0.0.0.0/0            172.17.0.4           tcp dpt:80

Chain DOCKER-ISOLATION-STAGE-1 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER-ISOLATION-STAGE-2  all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-ISOLATION-STAGE-2 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DROP       all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           

Chain DOCKER-USER (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  *      *       0.0.0.0/0            0.0.0.0/0           
```

### User Namespace
各个容器可能会出现重名的用户和用户组名称，或重复的用户UID或者GID，那么怎么隔离各个容器内的用户空间呢？
User Namespace允许在各个宿主机的各个容器空间内创建相同的用户名以及相同的用户UID和GID，只是会把用户的作用范围限制在每个容器内，即A容器和B容器可以有相同的用户名称和ID账户，但是此用户的有效范围仅为当前的容器内，不能访问另外一个容器内的文件系统，即相互隔离、互不影响、永不相见。
示例：
```bash
#进入容器查看容器内部的用户
root@mylinuxops:~# docker exec -it nginx-1 bash
root@b13ed304f010:/# id
uid=0(root) gid=0(root) groups=0(root)
root@b13ed304f010:/# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/bin/false
nginx:x:101:101:nginx user,,,:/nonexistent:/bin/false
```

## Linux Control Groups
在一个容器，如果不对其做任何资源限制，则宿主机会允许其占用无限大的内存空间，有时候会因为代码bug程序会一直盛情内存，直到把宿主机内存占完，为了避免此类为题的出现，宿主机有必要对容器进行资源分配限制，比如CPU、内存、磁盘、网络带宽等等。此外，还能够对进程进行优先级设置，以及将进程挂起和恢复等操作。
### 验证系统cgroups
Cgroups在内核层默认已经开启
查看Ubuntu的cgroup
```bash
root@mylinuxops:~# cat /boot/config-4.15.0-54-generic | grep CGROUP
CONFIG_CGROUPS=y
CONFIG_BLK_CGROUP=y
# CONFIG_DEBUG_BLK_CGROUP is not set
CONFIG_CGROUP_WRITEBACK=y
CONFIG_CGROUP_SCHED=y
CONFIG_CGROUP_PIDS=y
CONFIG_CGROUP_RDMA=y
CONFIG_CGROUP_FREEZER=y
CONFIG_CGROUP_HUGETLB=y
CONFIG_CGROUP_DEVICE=y
CONFIG_CGROUP_CPUACCT=y
CONFIG_CGROUP_PERF=y
CONFIG_CGROUP_BPF=y
# CONFIG_CGROUP_DEBUG is not set
CONFIG_SOCK_CGROUP_DATA=y
CONFIG_NETFILTER_XT_MATCH_CGROUP=m
CONFIG_NET_CLS_CGROUP=m
CONFIG_CGROUP_NET_PRIO=y
CONFIG_CGROUP_NET_CLASSID=y
```
查看CentOS的Cgroup
```bash
[root@localhost ~]# cat /boot/config-3.10.0-957.el7.x86_64 | grep CGROUP
CONFIG_CGROUPS=y
# CONFIG_CGROUP_DEBUG is not set
CONFIG_CGROUP_FREEZER=y
CONFIG_CGROUP_PIDS=y
CONFIG_CGROUP_DEVICE=y
CONFIG_CGROUP_CPUACCT=y
CONFIG_CGROUP_HUGETLB=y
CONFIG_CGROUP_PERF=y
CONFIG_CGROUP_SCHED=y
CONFIG_BLK_CGROUP=y
# CONFIG_DEBUG_BLK_CGROUP is not set
CONFIG_NETFILTER_XT_MATCH_CGROUP=m
CONFIG_NET_CLS_CGROUP=y
CONFIG_NETPRIO_CGROUP=y
```
对比centos和ubuntu，内核较新的ubuntu支持功能的比centos更加多

### cgroups具体实现
```bash
blkio：块设备IO限制。
cpu：使用调度程序为cgroup任务提供cpu的访问。
cpuacct：产生cgroup任务的cpu资源报告。
cpuset：如果是多核心的cpu，这个子系统会为cgroup任务分配单独的cpu和内存。
devices：允许或拒绝cgroup任务对设备的访问。
freezer：暂停和恢复cgroup任务。
memory：设置每个cgroup的内存限制以及产生内存资源报告。
net_cls：标记每个网络包以供cgroup方便使用。
ns：命名空间子系统。
perf_event：增加了对每group的监测跟踪的能力，可以监测属于某个特定的group的所有线程以及运行在特定CPU上的线程。
```
### 查看系统cgroups
```bash
[root@localhost ~]# ll /sys/fs/cgroup/
total 0
drwxr-xr-x 2 root root  0 Jul  4 21:49 blkio
lrwxrwxrwx 1 root root 11 Jul  4 21:49 cpu -> cpu,cpuacct
lrwxrwxrwx 1 root root 11 Jul  4 21:49 cpuacct -> cpu,cpuacct
drwxr-xr-x 2 root root  0 Jul  4 21:49 cpu,cpuacct
drwxr-xr-x 2 root root  0 Jul  4 21:49 cpuset
drwxr-xr-x 2 root root  0 Jul  4 21:49 devices
drwxr-xr-x 2 root root  0 Jul  4 21:49 freezer
drwxr-xr-x 2 root root  0 Jul  4 21:49 hugetlb
drwxr-xr-x 2 root root  0 Jul  4 21:49 memory
lrwxrwxrwx 1 root root 16 Jul  4 21:49 net_cls -> net_cls,net_prio
drwxr-xr-x 2 root root  0 Jul  4 21:49 net_cls,net_prio
lrwxrwxrwx 1 root root 16 Jul  4 21:49 net_prio -> net_cls,net_prio
drwxr-xr-x 2 root root  0 Jul  4 21:49 perf_event
drwxr-xr-x 2 root root  0 Jul  4 21:49 pids
drwxr-xr-x 4 root root  0 Jul  4 21:49 systemd
```
有了以上的chroot、namespace、cgroups就具备了基础的容器运行环境，但是还需要有相应的容器创建与删除的管理工具、以及怎么样把容器运行起来、容器数据怎么处理、怎么进行启动与关闭等问题需要解决，于是容器管理技术出现了。

## 容器管理工具
目前主要是使用docker，早期有使用lxc
### lxc
LXC: lxc为linux container的缩写。可以提供轻量级的虚拟化，以便隔离进程和资源，官方网站：：https://linuxcontainers.org/
lxc 启动容器依赖于模板，清华模板源：
https://mirrors.tuna.tsinghua.edu.cn/help/lxc-images/，但是做模板相对较难，需要手动一步步创构建文件系统、准备基础目录及可执行程序等，而且在大规模使用容器的场景很难横向扩展，另外后期代码升级也需要重新从头构建模板，
基于以上种种原因便有了docker。

### docker
Docker启动一个容器也需要一个外部模板但是叫做镜像，docker的镜像可以保存在一个公共的地方共享使用，只要把镜像下载下来就可以使用，最主要的是可以在镜像基础之上做自定义配置并且可以再把其提交为一个镜像，一个镜像可以被启动为多个容器。
Docker的镜像是分层的，镜像底层为库文件且只读层即不能写入也不能删除数据，从镜像加载启动为一个容器后会生成一个可写层，其写入的数据会复制到容器目录，但是容器内的数据在删除容器后也会被随之删除。
#### docker的优势和缺点
优势：
快速部署：短时间内可以部署成百上千个应用，更快速交付到线上。
高效虚拟化：不需要额外的hypervisor支持，直接基于linux实现应用虚拟化，相比虚拟机大幅提高性能和效率。
节省开支：提高服务器利用率，降低IT支出。
简化配置：将运行环境打包保存至容器，使用时直接启动即可。
快速迁移和扩展：可夸平台运行在物理机、虚拟机、公有云等环境，良好的兼
容性可以方便将应用从A宿主机迁移到B宿主机，甚至是A平台迁移到B平台。
缺点：
隔离性：各应用之间的隔离不如虚拟机彻底。
