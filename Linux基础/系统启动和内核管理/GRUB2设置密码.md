## `GRUB2`密码设置和删除

`GRUB2`和`GRUB0.97`一样可以设置密码，与`gurb 0.97`不同的是密码是通过命令设置的

#### `grub2`设置密码

1. 使用`grub2-setpassword`设置密码

```bash
[root@mylinuxops ~]# grub2-setpassword
Enter password:
Confirm password:
```

2. 密码创建后，在`/boot/grub2/`目录下生成密码的加密文件`user.cfg`

```bash
[root@mylinuxops ~]# ll /boot/grub2/user.cfg
-rw------- 1 root root 298 Apr 21 13:49 /boot/grub2/user.cfg
```

#### `grub2`密码删除

如果不需要使用密码只需要删除密码的加密文件修行

```bash
[root@mylinuxops ~]# rm -rf /boot/grub2/user.cfg
```

