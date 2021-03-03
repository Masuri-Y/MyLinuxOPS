### motd文件

`/etc/motd`文件内的内容可以用来显示登陆提示。

#### 登陆时提示welcome信息

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

