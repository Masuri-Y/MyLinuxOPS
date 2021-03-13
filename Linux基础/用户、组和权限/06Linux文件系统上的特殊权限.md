### Linux文件系统上的特殊权限

除了传统的读、写、执行权限，Linux上还存在一些特殊权限：

`SUID`，`SGID`，`Sticky位`，这三个特殊权限能影响之前的实时权限的一些行为。

#### `SUID`权限



```bash
# /etc/shadow文件普通用户没有权限，读取和修改
[wang@mylinuxops ~]$ ll /etc/shadow
---------- 1 root root 1696 Mar 10 22:00 /etc/shadow
[wang@mylinuxops ~]$ cat /etc/shadow
cat: /etc/shadow: Permission denied
# 使用passwd工具，修改口令后，/etc/shadow文件被修改了
[wang@mylinuxops root]$ passwd
Changing password for user wang.
Current password: 
New password: 
Retype new password: 
passwd: all authentication tokens updated successfully.
# 修改时间变了
[wang@mylinuxops root]$ ll /etc/shadow
---------- 1 root root 1696 Mar 12 10:21 /etc/shadow
```

在之前所说的权限中，用户要能访问一个文件必须和他的身份有关和其所用的工具无关，但是在以上案例中发生了一些变化，这主要是`passwd`这个命令有特殊设置

```bash
[wang@mylinuxops root]$ ll /usr/bin/passwd 
-rwsr-xr-x. 1 root root 33600 Apr  7  2020 /usr/bin/passwd
```

