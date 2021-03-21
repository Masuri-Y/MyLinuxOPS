### `Xargs`命令

由于很多命令不支持管道`|`来传递参数，`xargs`用于产生某个命令的参数，`xargs`可以读入`stdin`的数据，并且以空格符或回车符将`stdin`的数据分隔成为参数

许多命令不能接受过多参数，命令执行可能会失败，`xargs`可以解决。

```bash
# 将echo的输出传给xargs，xargs将echo的输出逐个传给touch，作为touch的参数进行执行
echo f{1..50000} | xargs touch 
```

注意：文件名或者是其他意义的名词内含有空格符的情况，需要使用`xargs -d`指定参数间的分隔符进行处理。

```bash
# 创建出a.txt,b.txt,"x y.txt"
[root@mylinuxops data]# touch a.txt b.txt "x y".txt
[root@mylinuxops data]# ll
total 0
-rw-r--r-- 1 root root 0 Mar 19 22:59  a.txt
-rw-r--r-- 1 root root 0 Mar 19 22:59  b.txt
-rw-r--r-- 1 root root 0 Mar 19 22:59 'x y.txt'
# 删除"x y.txt"时出错，xargs将其分为了2个文件 
[root@mylinuxops data]# find -name "*.txt" | xargs rm
rm: cannot remove './x': No such file or directory
rm: cannot remove 'y.txt': No such file or directory
# 使用xarg -d '\n'来指定分隔符
[root@mylinuxops data]# find -name "*.txt" | xargs -d '\n' rm
[root@mylinuxops data]# ll
total 0
```

#### `xargs`使用

```bash
# 创建user1-user10，10个用户，useradd一次不能接受多个参数，所以xargs需要使用-n指定每次传送的参数个数
[root@mylinuxops ~]# echo user{1..10} | xargs -n 1 useradd
# 查看用户是否创建
[root@mylinuxops ~]# echo user{1..10} | xargs -n 1 id
uid=1013(user1) gid=1013(user1) groups=1013(user1)
uid=1014(user2) gid=1016(user2) groups=1016(user2)
uid=1015(user3) gid=1017(user3) groups=1017(user3)
uid=1016(user4) gid=1018(user4) groups=1018(user4)
uid=1017(user5) gid=1019(user5) groups=1019(user5)
uid=1018(user6) gid=1020(user6) groups=1020(user6)
uid=1019(user7) gid=1021(user7) groups=1021(user7)
uid=1020(user8) gid=1022(user8) groups=1022(user8)
uid=1021(user9) gid=1023(user9) groups=1023(user9)
uid=1022(user10) gid=1024(user10) groups=1024(user10)
```

其他示例：

```bash
ls	| xargs	rm										# 删除当前目录下的大量文件
find /sbin/ -perm +700 | ls -l						# 这个命令是错误的，ls没有标准输入。
find /bin/ -perm /7000 | xargs ls -Sl				# 查找有特殊权限的文件（只要有一个特殊权限，即满足）
find /bin/ -perm -7000 | xargs ls -Sl				# 此命令和上面有何区别？（3个特殊权限都要有）
find -type f -name “*.txt” -print0 | xargs -0 rm 	# 以字符nul分隔
```

