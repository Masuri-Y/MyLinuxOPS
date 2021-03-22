### RPM包管理

CentOS系统上使用rpm命令管理程序包：安装、卸载、升级、查询、校验、数据库维护

#### 安装

```bash
rpm {-i|--install} [install-options] PACKAGE_FILE…
	-v: verbose
	-vv:
	-h: 以#显示程序包管理执行进度
	
rpm -ivh PACKAGE_FILE ...
```

[install-options]

* `--test`: 测试安装，但不真正执行安装，即dry run模式
* `--nodeps`：忽略依赖关系
* `--replacepkgs | replacefiles`
* `--nosignature`: 不检查来源合法性
* `--nodigest`：不检查包完整性
* `--noscripts`：不执行程序包脚本
  * `%pre`: 安装前脚本	--nopre
  * `%post`: 安装后脚本	--nopost
  * `%preun`: 卸载前脚本	--nopreun
  * `%postun`: 卸载后脚本	--nopostun

##### 安装示例

1. 在centos6上有个神奇目录可以自动挂载光盘，在centos7上需要安装autofs这个程序包后才有用

```bash
# 安装autofs
[root@mylinuxops Packages]# rpm -ivh autofs-5.1.4-43.el8.x86_64.rpm 
Verifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:autofs-1:5.1.4-43.el8            ################################# [100%]
# 启动autofs程序
[root@mylinuxops Packages]# systemctl start autofs
# 可以进入/misc/cd目录
[root@mylinuxops Packages]# cd /misc/cd
[root@mylinuxops cd]# ls
AppStream  BaseOS  EFI  images  isolinux  LICENSE  media.repo  TRANS.TBL
```

2. 安装vsftp软件

```bash
[root@mylinuxops ~]# rpm -ivh /misc/cd/AppStream/Packages/vsftpd-3.0.3-32.el8.x86_64.rpm 
Verifying...                          ################################# [100%]
Preparing...                          ################################# [100%]
Updating / installing...
   1:vsftpd-3.0.3-32.el8              ################################# [100%]
```

#### 包查询

```bash
rpm {-q|--query} [select-options] [query-options]
```

[select-options]

* `-a`: 所有包

* `-f`: 查看指定的文件由哪个程序包安装生成

* `-p rpmfile`: 针对尚未安装的程序包文件做查询操作

* `--whatprovides CAPABILITY`: 查询指定的CAPABILITY由哪个包所提供

* `--whatrequires CAPABILITY`: 查询指定的CAPABILITY被哪个包所依赖

查看rpm包内文件的方法

```bash
rpm2cpio 包文件 | cpio –itv	预览包内文件
rpm2cpio 包文件 | cpio –id	"*.conf" 释放包内文件
```

##### 查询示例

1.查询某个包是否安装时需要包名

```bash
[root@mylinuxops ~]# rpm -q vsftpd
vsftpd-3.0.3-32.el8.x86_64		# 如果已经安装则会显示
```

2.查询某个包是否安装，使用`-qa`选项，配合`grep`模糊搜索

```bash
[root@mylinuxops ~]# rpm -qa | grep "^vs"
vsftpd-3.0.3-32.el8.x86_64
```

