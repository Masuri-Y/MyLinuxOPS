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
  
  ```bash
  # --scripts用来查看应用程序内是否带有脚本
  [root@mylinuxops ~]# rpm -q --scripts postfix
  preinstall scriptlet (using /bin/sh):		# 安装前脚本
  # Add user and groups if necessary
  /usr/sbin/groupadd -g 90 -r postdrop 2>/dev/null
  /usr/sbin/groupadd -g 89 -r postfix 2>/dev/null
  
  # 如果不想安装安装前脚本，只需要在安装时使用--nopre选项
  rpm -ivh --nopre postfix
  # 所有脚本都不使用则使用--noscripts选项。
  ```

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

[query-options]

* `--changelog`: 查询rpm包的changelog

* `-c`: 查询程序的配置文件

* `-d`: 查询程序的文档

* `-i`: information

* `-l`: 查看指定的程序包安装后生成的所有文件

* `--scripts`: 程序包自带的脚本

* `--provides`: 列出指定程序包所提供的CAPABILITY

* `-R`: 查询指定的程序包所依赖的CAPABILITY

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

4. 查询已经安装包的信息

```bash
[root@mylinuxops ~]# rpm -qi postfix 
Name        : postfix
Epoch       : 2
Version     : 3.3.1
Release     : 12.el8
Architecture: x86_64
Install Date: Tue 23 Mar 2021 09:46:26 AM CST
Group       : System Environment/Daemons
Size        : 4348909
License     : (IBM and GPLv2+) or (EPL-2.0 and GPLv2+)
Signature   : RSA/SHA256, Thu 09 Apr 2020 12:00:24 PM CST, Key ID 05b555b38483c65d
Source RPM  : postfix-3.3.1-12.el8.src.rpm
Build Date  : Tue 07 Apr 2020 11:11:59 AM CST
Build Host  : x86-01.mbox.centos.org
Relocations : (not relocatable)
Packager    : CentOS Buildsys <bugs@centos.org>
Vendor      : CentOS
URL         : http://www.postfix.org
Summary     : Postfix Mail Transport Agent
Description :
Postfix is a Mail Transport Agent (MTA).
```

5. 查询未安装的包的信息，需要使用-p选项，并且补全路径。

```bash
[root@mylinuxops ~]# rpm -qpi /misc/cd/AppStream/Packages/vsftpd-3.0.3-32.el8.x86_64.rpm 
Name        : vsftpd
Version     : 3.0.3
Release     : 32.el8
Architecture: x86_64
Install Date: (not installed)
Group       : System Environment/Daemons
Size        : 351530
License     : GPLv2 with exceptions
Signature   : RSA/SHA256, Wed 29 Apr 2020 12:08:42 AM CST, Key ID 05b555b38483c65d
Source RPM  : vsftpd-3.0.3-32.el8.src.rpm
Build Date  : Mon 27 Apr 2020 10:04:03 AM CST
Build Host  : x86-01.mbox.centos.org
Relocations : (not relocatable)
Packager    : CentOS Buildsys <bugs@centos.org>
Vendor      : CentOS
URL         : https://security.appspot.com/vsftpd.html
Summary     : Very Secure Ftp Daemon
Description :
vsftpd is a Very Secure FTP daemon. It was written completely from
scratch.
```

6. 包未安装，要查询安装后会生成的文件列表

```bash
[root@mylinuxops ~]# rpm -qpl /misc/cd/AppStream/Packages/vsftpd-3.0.3-32.el8.x86_64.rpm 
/etc/logrotate.d/vsftpd
/etc/pam.d/vsftpd
/etc/vsftpd
/etc/vsftpd/ftpusers
/etc/vsftpd/user_list
/etc/vsftpd/vsftpd.conf
/etc/vsftpd/vsftpd_conf_migrate.sh
...

# 如果包已经安装，则使用-ql选项来查询。
```

7. 查询磁盘上的文件来自于哪个包

```bash
[root@mylinuxops ~]# rpm -qf /bin/cat
coreutils-8.30-8.el8.x86_64
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

#### 包校验

```bash
rpm {-V|--verify} [select-options] [verify-options] 
```

* S file Size differs
* M Mode differs (includes permissions and file type) 
* 5 digest (formerly MD5 sum) differs
* D Device major/minor number mismatch
* L readLink(2) path mismatch U User ownership differs
* G Group ownership differs
* T mTime differs
* P capabilities differ

```bash
# 验证某个文件距离安装后是否发生变化
[root@mylinuxops ~]# rpm -V tree

# 以上没有变化所以么有输出，对其进行echo之后
[root@mylinuxops ~]# echo >> /usr/bin/tree
[root@mylinuxops ~]# rpm -V tree
S.5....T.    /usr/bin/tree	# 发生了变化
```

包来源的合法性验证及完整性验证

* 完整性验证：SHA256

* 来源合法性验证：RSA

公钥加密

* 对称加密：加密、解密使用同一密钥
* 非对称加密：密钥是成对儿的
  * public key: 公钥，公开所有人
  * secret key: 私钥, 不能公开

##### 导入所需要公钥

```bash
# 导入公钥，CentOS 7发行版光盘提供：RPM-GPG-KEY-CentOS-7
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7 

# 查看共要内容
rpm -qa "gpg-pubkey*"

# 检查包的完整性和签名
rpm	-K|checksig	rpmfile
```

#### 包升级

```bash
rpm {-U|--upgrade} [install-options] PACKAGE_FILE...
rpm {-F|--freshen} [install-options] PACKAGE_FILE... 
```

* `upgrade`: 安装有旧版程序包，则"升级"，如果不存在旧版程序包，则"安装"。
* `freshen`: 安装有旧版程序包，则"省级"，如果不存在旧版程序包，则不执行升级操作。

常用命令

```bash
rpm -Uvh PACKAGE_FILE ...
rpm -Fvh PACKAGE_FILE ...
```

选项：

* `--oldpackage`: 降级
* `--force`: 强制安装

##### 注意：

1. 不要对内核做升级操作；Linux支持多内核版本并存，因此直接安装新版本内核
2. 如果原程序包的配置文件安装后曾被修改，升级时，新版本提供的同一个配置文件不会直接覆盖老版本的配置文件，而把新版本文件重命名(FILENAME.rpmnew)后保留

#### rpm数据库

rpm安装的程序都被被记录在数据库中`/var/lib/rpm`

```bash
rpm {--initdb|--rebuilddb} 
initdb: 初始化,如果事先不存在数据库，则新建之。否则，不执行任何操作
rebuilddb：重建已安装的包头的数据库索引目录
```

