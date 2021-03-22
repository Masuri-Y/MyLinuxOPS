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

  ```bash
  # 以http为例单独安装将因为依赖关系报错
  [root@mylinuxops Packages]# rpm -ivh httpd-2.4.37-30.module_el8.3.0+561+97fdbbcc.x86_64.rpm 
  error: Failed dependencies:
          /etc/mime.types is needed by httpd-2.4.37-30.module_el8.3.0+561+97fdbbcc.x86_64
          httpd-filesystem is needed by httpd-2.4.37-30.module_el8.3.0+561+97fdbbcc.x86_64
          httpd-filesystem = 2.4.37-30.module_el8.3.0+561+97fdbbcc is needed by httpd-2.4.37-30.module_el8.3.0+561+97fdbbcc.x86_64
          httpd-tools = 2.4.37-30.module_el8.3.0+561+97fdbbcc is needed by httpd-2.4.37-30.module_el8.3.0+561+97fdbbcc.x86_64
          libapr-1.so.0()(64bit) is needed by httpd-2.4.37-30.module_el8.3.0+561+97fdbbcc.x86_64
          libaprutil-1.so.0()(64bit) is needed by httpd-2.4.37-30.module_el8.3.0+561+97fdbbcc.x86_64
          mod_http2 is needed by httpd-2.4.37-30.module_el8.3.0+561+97fdbbcc.x86_64
          system-logos-httpd is needed by httpd-2.4.37-30.module_el8.3.0+561+97fdbbcc.x86_64
          
  # 使用--nodeps忽略以来关系强行安装，不推荐。
  [root@mylinuxops Packages]# rpm -ivh --nodeps httpd-2.4.37-30.module_el8.3.0+561+97fdbbcc.x86_64.rpm 
  Verifying...                          ################################# [100%]
  Preparing...                          ################################# [100%]
  Updating / installing...
     1:httpd-2.4.37-30.module_el8.3.0+56################################# [100%]
  ```

* `--replacepkgs | replacefiles`: 和`-ivh`结合使用实现覆盖安装

  ```bash
  # 服务器上已经安装了tree
  [root@mylinuxops Packages]# rpm -ql tree
  /usr/bin/tree
  /usr/lib/.build-id
  /usr/lib/.build-id/d8
  /usr/lib/.build-id/d8/6d516d7cb07fb9334cb268af808119e33a5ac5
  /usr/share/doc/tree
  /usr/share/doc/tree/LICENSE
  /usr/share/doc/tree/README
  /usr/share/man/man1/tree.1.gz
  # 删除tree命令后重新安装
  [root@mylinuxops Packages]# rm -f /bin/tree 
  [root@mylinuxops Packages]# tree
  bash: tree: command not found
  [root@mylinuxops Packages]# rpm -ivh /misc/cd/BaseOS/Packages/tree-1.7.0-15.el8.x86_64.rpm 
  Verifying...                          ################################# [100%]
  Preparing...                          ################################# [100%]
          package tree-1.7.0-15.el8.x86_64 is already installed
  # 重新安装没有生效，这是因为在/var/lib/rpm中显示已经安装过了
  [root@mylinuxops Packages]# tree
  bash: tree: command not found
  # 所以此处需要使用--replacepkgs来覆盖安装
  [root@mylinuxops Packages]# rpm -ivh --replacepkgs /misc/cd/BaseOS/Packages/tree-1.7.0-15.el8.x86_64.rpm 
  Verifying...                          ################################# [100%]
  Preparing...                          ################################# [100%]
  Updating / installing...
     1:tree-1.7.0-15.el8                ################################# [100%]
  ```

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

1. 查询某个包是否安装时需要包名

```bash
[root@mylinuxops ~]# rpm -q vsftpd
vsftpd-3.0.3-32.el8.x86_64		# 如果已经安装则会显示
```

2. 查询某个包是否安装，使用`-qa`选项，配合`grep`模糊搜索

```bash
[root@mylinuxops ~]# rpm -qa | grep "^vs"
vsftpd-3.0.3-32.el8.x86_64
```

3. 使用通配符查询某个包是否安装的模糊搜索

```bash
[root@mylinuxops ~]# rpm -qa "vsf*"
vsftpd-3.0.3-32.el8.x86_64
```

#### 包卸载

```bash
rpm {-e|--erase} [--allmatches] [--nodeps] [--noscripts] [--notriggers] [--test] PACKAGE_NAME ...
```

当包卸载时，对应的配置文件不会删除， 以FILENAME.rpmsave形式保留

##### 包卸载示例

```bash
[root@mylinuxops ~]# rpm -e vsftpd
[root@mylinuxops ~]# rpm -q vsftpd
package vsftpd is not installed
```

