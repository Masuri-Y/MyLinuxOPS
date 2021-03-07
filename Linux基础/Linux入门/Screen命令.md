### Screen命令

screen命令可以实现桌面共享命令行界面和后台执行任务的效果，screen在CentOS8上在epel源中，需要先安装配置epel源，然后进行安装。

```bash
[root@mylinuxops ~]# dnf install epel-release-8-8.el8.noarch -y
[root@mylinuxops ~]# dnf install screen -y
```

#### `screen`使用

日常工作中可能需要执行一个需要耗时长，且终端不能中断的任务，如备份。此时就可以使用screen命令将正在执行的任务扔

> 使用`screen -S`创建一个窗口

```bash
[root@mylinuxops ~]# screen -S ping
```

> 在新的窗口中执行`ping`命令

```bash
[root@mylinuxops ~]# ping 172.16.11.1
PING 172.16.11.1 (172.16.11.1) 56(84) bytes of data.
64 bytes from 172.16.11.1: icmp_seq=1 ttl=254 time=1.59 ms
64 bytes from 172.16.11.1: icmp_seq=2 ttl=254 time=1.48 ms
64 bytes from 172.16.11.1: icmp_seq=3 ttl=254 time=1.71 ms
```

使用`ctrl + a,d`快捷键退出screen，此时执行ping命令的screen依旧存在。

> 使用`screen -ls`进行查看，当前ping的窗口被剥离

```bash
[root@mylinuxops ~]# screen -ls
There is a screen on:
        34533.ping      (Detached)
1 Socket in /run/screen/S-root.
```

> 使用`screen -r` 将窗口关联回去 

```bash
64 bytes from 172.16.11.1: icmp_seq=289 ttl=254 time=1.57 ms
64 bytes from 172.16.11.1: icmp_seq=290 ttl=254 time=1.50 ms
64 bytes from 172.16.11.1: icmp_seq=291 ttl=254 time=1.54 ms
64 bytes from 172.16.11.1: icmp_seq=292 ttl=254 time=1.61 ms
64 bytes from 172.16.11.1: icmp_seq=293 ttl=254 time=1.65 ms
# 窗口内依旧在执行ping命令
```

