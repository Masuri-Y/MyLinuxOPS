### Linux文件系统上的特殊权限

除了传统的读、写、执行权限，Linux上还存在一些特殊权限：

`SUID`，`SGID`，`Sticky位`，这三个特殊权限能影响之前的实时权限的一些行为。

#### `SUID`权限

什么是`SUID`？

`SUID`表示，当一个用户去运行一个可执行程序时，他将打破以前默认的执行方式（用户能否访问和工具无关，只和用户的身份有关），但是有了`SUID`权限后就变为，当用户在使用工具访问文件时，若工具拥有`SUID`权限，那么这个用户的身份就变为这个工具的所有者身份，若所使用工具的所有者为root那么此时用户的身份将变为root。

##### `SUID`在Linux中的使用

Linux系统中`passwd`命令就存在`SUID`权限

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

在之前所说的权限中，用户要能访问一个文件必须和他的身份有关和其所用的工具无关，但是在以上案例中发生了一些变化，这主要是`passwd`这个命令有特殊设置。

```bash
[wang@mylinuxops root]$ ll /usr/bin/passwd 
-rwsr-xr-x. 1 root root 33600 Apr  7  2020 /usr/bin/passwd
# 属主上的执行权限为s，这个就是SUID权限
```

`SUID`作用在所有者的执行位的位置，表现为`s`。

##### `SUID`设定方法

`SUID`在设置时需要注意以下两点：

* `SUID`只对二进制可执行程序有效
* `SUID`设置在目录上没有意义

`SUID`权限设定：

```bash
# 加SUID权限
chmod u+s FILE...
# 取消suid权限
chmod u-s FILE...

# SUID数字法是独立计算的。数字法表示为4，其后3位数字为该文件正常的权限
# 数字法添加SUID权限
chmod 4xxx FILE...
# 数字法取消SUID权限
chmod 0xxx|xxx FILE
```

##### `SUID`使用示例

> 添加`SUID`权限

`/usr/bin/cat`原本没有`SUID`权限。所以普通用户在访问`/etc/shadow`文件时会显示权限不够

```bash
[wang@mylinuxops ~]$ ll /usr/bin/cat
-rwxr-xr-x. 1 root root 38504 Apr 27  2020 /usr/bin/cat
[wang@mylinuxops ~]$ cat /etc/shadow
cat: /etc/shadow: Permission denied
```

为`/usr/bin/cat`添加`SUID`权限。

```bash
[root@mylinuxops ~]# chmod u+s /usr/bin/cat
[root@mylinuxops ~]# ll /usr/bin/cat
-rwsr-xr-x. 1 root root 38504 Apr 27  2020 /usr/bin/cat
```

再次切换到wang用户，使用`cat`来查看`/etc/shadow`文件

```bash
[root@mylinuxops ~]# su - wang
Last login: Sat Mar 13 20:02:24 CST 2021 on pts/0
[wang@mylinuxops ~]$ cat /etc/shadow
root:$6$S2FdLAWA3tyLcwtn$FED0DTbFi4UZLyjGRRIbcQq6p4OI19BxevmzlFmgIK7cbX1lXWaFS3Q47RBIicIAAjv81B.IjrkWqsiGwFiNr0::0:99999:7:::
bin:*:18397:0:99999:7:::
daemon:*:18397:0:99999:7:::
....
```

需要注意某些可执行文件拥有了`SUID`之后可能会导致权限过大造成危险操作，如`vim`、`nano`、`cat`等等，所以慎用`SUID`权限

> 取消`SUID`权限

取消`/usr/bin/cat`的`SUID`权限

```bash
[root@mylinuxops ~]# chmod u-s /usr/bin/cat
[root@mylinuxops ~]# ll /usr/bin/cat
-rwxr-xr-x. 1 root root 38504 Apr 27  2020 /usr/bin/cat
```

#### `SGID`权限

`SUID`是作用在所有者的执行位上，若`s`权限作用在所属组的执行位上又有什么效果？

其含义与`SUID`类似，`SUID`表示用户继承了该二进制可执行文件的属主权限。作用在所属组的执行位上则表示该用户继承了该二进制可执行文件的所属主的权限，也就是`SUID`权限。

与`SUID`不同`SGID`还可以作用在目录上。

`SGID`在数字法表现为2。若`SUID`和`SGID`同时出现，则数字法表示为6.

##### `SGID`作用在可执行文件上

表示继承该二进制文件的属组的权限。

设定方法：

```bash
# mode法+SGID
chmod g+s FILE
# mode法-SGID
chmod g-s FILE

# SGID在数字法表现为2
# 数字法+SGID
chmod 2xxx FILE
# 数字法-SGID
chmod 0xxx|xxx FILE
```

##### `SGID`作用在目录上

默认情况下，用户创建文件时，其属组为此用户所属的主组，一旦某目录被设定了`SGID`，则对此目录有写权限的用户在此目录中创建的文件所属的组为此目录的属组。通常用于创建一个协作目录。

设定方法：

```bash
# mode法+SGID
chmod g+s DIR
# mode法-SGID
chmod g-s DIR

# SGID在数字法表现为2
# 数字法+SGID
chmod 2xxx DIR
# 数字法-SGID
chmod 0xxx|xxx DIR
```

##### `SGID`使用示例



#### `Sticky`位

具有写权限的目录通常用户可以删除该目录中的任何文件，无视该文件的权限或拥有权。这样的做法是不合理的，所以出现`Sticky`位这个权限。

Sticky位作用在other的执行位上，表现为一个t字母。

在目录上设置`Sticky` 位后，只有文件的所有者或root可以删除该文件。

`Sticky`设置在文件上没有意义

在Linux系统中`/tmp`目录就是设置的`Sticky`位。(`tmp`目录用来存放临时文件，所有用户的文件都会将文件创建在此目录内，防止用户误删其他用户文件，所以设置`Sticky`)

```bash
[masuri@mylinuxops data]$ ll -d /tmp
drwxrwxrwt. 5 root root 4096 Mar 13 22:32 /tmp
```

##### `Sticky`位设定方法

`Sticky`需要设定在目录上， 对文件没有作用。

`Sticky`用数字法设置时使用1表示。

```bash
# mode法+Sticky
chmod o+t DIR...
# mode法-Sticky
chmod o-t DIR...
# 数字法+Sticky
chmod 1xxx DIR...
# 数字法-Sticky
chmod 0xxx|xxx DIR...
```

##### `Sticky`位的使用

目录下存在一个`test_dir`目录其other的权限为`rwx`

```bash
[root@mylinuxops data]# ll
total 4
drwxr-xrwx 2 root root 4096 Mar 13 22:29 test_dir
```

wang用户和masuri用户分别在test_dir目录下创建一个自己的文件

```bash
[root@mylinuxops data]# su masuri
[masuri@mylinuxops data]$ touch test_dir/masuri_file
[masuri@mylinuxops data]$ exit
exit
[root@mylinuxops data]# su wang
[wang@mylinuxops data]$ touch test_dir/wang_file
[wang@mylinuxops data]$ ll test_dir/
total 0
-rw-rw-r-- 1 masuri masuri 0 Mar 13 22:30 masuri_file
-rw-rw-r-- 1 wang   wang   0 Mar 13 22:30 wang_file
```

由于所创建的文件属主和数组非wang用户，所有wang用户无法对masuri_file进行修改，但是却能删除masuri的文件

```bash
# 修改masuri_file，权限不够
[wang@mylinuxops data]$ echo "123" >> test_dir/masuri_file 
bash: test_dir/masuri_file: Permission denied
# 但是能删除
[wang@mylinuxops data]$ rm -rf test_dir/masuri_file 
[wang@mylinuxops data]$ ll test_dir/
total 4
-rw-rw-r-- 1 wang wang 4 Mar 13 22:31 wang_file
```

以上做法是不合理的，所以给目录添加上Sticky位后，非自己的文件就无法再删除了。

```bash
# 给test_dir目录加上sticky位
[root@mylinuxops data]# chmod o+t test_dir
[masuri@mylinuxops data]$ ll -d test_dir
drwxr-xrwt 2 root root 4096 Mar 13 22:31 test_dir
# 切换到masuri用户，将wang_file文件删除
[root@mylinuxops data]# su masuri
[masuri@mylinuxops data]$ rm -rf test_dir/wang_file 
# 删除失败
rm: cannot remove 'test_dir/wang_file': Operation not permitted
```



