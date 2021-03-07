### motd和issue

motd和issue文件都是用来显示提示信息，区别在于motd文件的提示信息在登陆后显示，issue的提示信息在登陆前显示。

#### motd文件

`/etc/motd`文件内的内容可以用来显示登陆后提示。

##### 登陆后提示welcome信息

> 修改`/etc/motd`文件

```bash
[root@mylinuxops ~]# echo "welcome to mylinuxops" > /etc/motd
```

> 退出后再次登陆

```bash
[root@Computer01 ~]# ssh 172.16.11.61
welcome to mylinuxops		# 此为刚才输入的提示信息
Activate the web console with: systemctl enable --now cockpit.socket

Last login: Wed Mar  3 21:16:55 2021 from 172.16.11.51
[root@mylinuxops ~]# 
```

#### issue文件

`/etc/issue`文件内的内容用于显示登陆前提示

##### 登陆前提示welcome信息

> 修改/etc/issue文件

```bash
[root@mylinuxops ~]# vim /etc/issue
Welcome to MyLinuxOPS
\S
Kernel \r on an \m
```

> 退出后再次登陆

```bash
# 登陆前出现了welcome信息
Welcome to MyLinuxOPS
CentOS Linux 8
Kernel 4.18.0-240.el8.x86_64 on an x86_64

Activate the web console with: systemctl enable --now cockpit.socket

mylinuxops login: 
```

提示：issue文件中的一些支持变量可以使用`man agetty`进行查询