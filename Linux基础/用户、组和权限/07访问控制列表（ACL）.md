### 访问控制列表（`ACL`）

`ACL`： Access Control List，实现灵活的权限管理

传统的文件权限控制权限只能实现简单的权限分配，`ACL`可以实现除了文件的所有者，所属组和其他人外，可以对更多的用户设置权限。

需要注意`ACL`的权限是依赖于文件系统的：

* `CentOS7`默认创建的`xfs`和`ext4`文件系统具有`ACL`功能

* `CentOS7`之前版本，默认手工创建的`ext4`文件系统无`ACL`功能，需要手动增加

  ```bash
  tune2fs –o acl /dev/sdb1
  mount –o acl /dev/sdb1	/mnt/test
  ```

#### `ACL`权限生效顺序

当去访问一个文件时，先判断一个文件的所有者是否为该用户，若是则权限生效`ACL`不再生效。若不是则判断该用户是否为通过`ACL`添加的用户，如果是则该`ACL`生效，之后内容不再查看。若不是则判断该用户是否为通过`ACL`添加的组内的用户，若是则`ACL`生效，之后内容不再查看。若以上都不是则其他用户的的权限生效。

所有者>自定义用户>自定义组>其他人

#### `ACL`中的`mask`

`mask`只影响除所有者和`other`的之外的人和组的最大权限（限高杆）

`mask`需要与用户的权限进行逻辑与运算后，才能变成有限的权限(Effective Permission)

用户或组的设置必须存在于mask权限设定范围内才会生效。

```bsah
setfacl	-m mask::rx	file
```

`--set`选项会把原有的`ACL`项都删除，用新的替代，需要注意的是一定要包含`UGO`的设置，不能象-m一样只是添加`ACL`就可以

```bash
setfacl --set u::rw,u:wang:rw,g::r,o::- file1
```

`ACL`文件上的group权限是mask 值（自定义用户，自定义组，拥有组的最大权限）,而非传统的组权限

#### `ACL`其他选项1

```bash
getfacl				# 可看到特殊权限：flags
# 通过ACL赋予目录默认x权限，目录内文件也不会继承x权限
base ACL 			# 不能删除
setfacl -k dir 		# 删除默认ACL权限，针对setfacl -m d:u:wang:rx directory来使用
setfacl –b file1 	# 清除所有ACL权限
getfacl file1 | setfacl --set-file=- file2	# 复制file1的acl权限给file2
```

#### `ACL`相关命令

```bash
#为多用户或者组的文件和目录赋予访问权限rwx 
mount -o acl /directory					# 要使用acl权限需要挂载时使用acl
getfacl file|directory					# 查看那文件或目录的ACL权限
setfacl -m u:wang:rwx file|directory 	# 设置wang用户对于文件或目录的ACL权限
setfacl -Rm g:sales:rwX directory 		# 递归设置sales组的对于目录的ACL权限
setfacl -M file.acl file|directory		# 将acl权限写入file.acl文件中通过调用文件设置acl
setfacl -m g:salesgroup:rw file|directory # 设置salesgroup组对于文件或目录的ACL权限
setfacl -m d:u:wang:rx directory		# d表示设置默认权限对目录使用，当在目录下创建文件时自动获取所设置的acl权限。
setfacl -x u:wang file|directory 		# 删除文件或目录中wang用户的ACL权限
setfacl -X file.acl directory			# 从fale.acl文件中读取acl权限，删除权限。
```

#### 备份和恢复`ACL`

文件操作命令`cp`和`mv`都支持`ACL`，只是`cp`命令需要加上`-p`参数。但是`tar`等常见的备份工具是不会保留目录和文件的`ACL`信息，这时候需要备份`ACL`权限方便恢复。

```bash
# 递归将dir1的acl权限写入acl.txt文件中
getfacl -R /tmp/dir1 > acl.txt
# 递归清除dir1目录内的ACL权限
setfacl -R -b /tmp/dir1
# 恢复ACL权限，以下两条命令2选1
setfacl -R --set-file=acl.txt /tmp/dir1 
setfacl --restore acl.txt
# 再次查看ACL权限。
getfacl -R /tmp/dir1
```

#### `ACL`的实现案例

##### 针对用户设置`ACL`权限

在不影响传统权限的情况下，让`wang`用户对某个文件不能读，`masuri`用户对这个文件能读的能写

```bash
# 当前目录下有个f2的文件
[root@mylinuxops data]# ll
total 4
-rw-r--r-- 1 root root 8 Mar 15 13:49 f2
# 对f2设置acl权限，wang用户什么权限都没有
[root@mylinuxops data]# setfacl -m u:wang:- f2
# 切换到wang用户对文件进行访问
[root@mylinuxops data]# su wang
[wang@mylinuxops data]$ cat f2 
cat: f2: Permission denied
# 对masuri用户设置读写权限
[root@mylinuxops data]# setfacl -m u:masuri:rw f2
# 切换到masuri用户对文件进行修改
[root@mylinuxops data]# su masuri
[masuri@mylinuxops data]$ ll f2
-rw-rw-r--+ 1 root root 8 Mar 15 13:49 f2		# 文件最后一位权限有+号表示该文件有ACL权限。
# 对文件进行修改
[masuri@mylinuxops data]$ echo "abc" >> f2
[masuri@mylinuxops data]$ cat f2 
123
123
abc
# 查看f2文件的facl权限
[masuri@mylinuxops data]$ getfacl f2
# file: f2
# owner: root
# group: root
user::rw-
user:wang:---		# wang用户没有权限
user:masuri:rw-		# masuri用户只有读写权限
group::r--
mask::rw-
other::r--
```

##### 针对组设置`ACL`权限

除了文件的属组和admin组的用户能对文件进行读写，其他用户都不行

```bash
# 创建一个admin组
[root@mylinuxops data]# groupadd admin
# 将masuri用户加入该组
[root@mylinuxops data]# usermod -aG admin masuri
# 创建一个file1文件
[root@mylinuxops data]# echo "test_group_facl" > file1
# 对其设置acl权限，admin组用户可以读写。
[root@mylinuxops data]# setfacl -m g:admin:rw file1 
# 切换到masuri用户
[root@mylinuxops data]# su masuri
# masuri用户在admin组内
[masuri@mylinuxops data]$ id masuri
uid=1001(masuri) gid=1001(masuri) groups=1001(masuri),1007(devops),1008(admin)
[masuri@mylinuxops data]$ ll
total 8
-rw-rw-r--+ 1 root root 12 Mar 15 14:57 f2
-rw-rw-r--+ 1 root root 16 Mar 15 15:26 file1
# 对file1文件可以进行修改
[masuri@mylinuxops data]$ echo "hello~" >> file1
# 可以查看file1文件内的内容
[masuri@mylinuxops data]$ cat file1 
test_group_facl
hello~
```

##### 删除`ACL`权限

将`f2`文件上的`admin`组的`ACL`权限删除

```bash
# 使用-x选项去除acl去权限
[root@mylinuxops data]# setfacl -x g:admin f2 
# 切换到masuri，对f2文件进行追加
[root@mylinuxops data]# su masuri
[masuri@mylinuxops data]$ echo "masuri_line" > f2
bash: f2: Permission denied
```

