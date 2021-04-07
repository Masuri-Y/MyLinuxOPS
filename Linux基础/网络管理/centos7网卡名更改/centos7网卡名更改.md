## CentOS 7网卡名修改

CentOS7网卡名与CentOS6不同，然而7的早期版本与最新的版本的网卡名又有所不同，在生产环境中，为了实现自动化管理，首先要实现全部标准化，所以要将网卡名同一设置为和CentOS6相同的ethN的格式，具体的修改方法如下：  

1. 首先修改/etc/default/grub文件,在GRUB_CMDLINE_LINUX行最后添加net.ifnames=0

```bash
[root@mylinuxops ~]# vim /etc/default/grub 
GRUB_TIMEOUT=5
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="crashkernel=auto rhgb quiet net.ifnames=0"
GRUB_DISABLE_RECOVERY="true"
```
2. 使用grub2-mkconfig命令重新生成/boot/grub2/grub.cfg文件

```bash
[root@mylinuxops ~]# grub2-mkconfig -o /boot/grub2/grub.cfg 
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-957.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-957.el7.x86_64.img
Found linux image: /boot/vmlinuz-0-rescue-30905c0f8bf344f4af5b53a826370629
Found initrd image: /boot/initramfs-0-rescue-30905c0f8bf344f4af5b53a826370629.img
done
```
3. 重启后主机网卡已改为传统的ethN模式

```
[root@mylinuxops ~]# ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.172.130  netmask 255.255.255.0  broadcast 192.168.172.255
        inet6 fe80::68e7:2a55:44f4:593  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:63:21:a6  txqueuelen 1000  (Ethernet)
        RX packets 4  bytes 806 (806.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 37  bytes 5389 (5.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

