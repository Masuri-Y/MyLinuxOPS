### YUM

`YUM`: Yellowdog Update Modifier，依赖于C/S架构来实现的。rpm的前端程序，可解决软件包相关依赖性，可在多个库之间定位软件包，up2date的替代工具。

#### `YUM`运行流程

`YUM`在使用时，需要预先配置服务器端和客户端：

* 服务器端: 用来存放yum的仓库(`yum repository`)，仓库内包含了众多的安装包(rpm)以及安装包的元数据(metadata)，并将这些资源发布到网络上。可以利用一些现有服务来实现：`http`、`https`、`ftp`、`file`（本地）

* 客户端：客户端和服务器端可以不再同一台电脑。需要在客户端的配置文件中配置服务器端的仓库路径。

客户端配置完毕后执行`yum install`命令后会先读取配置文件，根据配置文件中的服务器路径连接服务器，将服务器上的元数据信息下载到本地的`yum cache`中，元数据中查询是否有所要安装的包，如果有则查询该包是否依赖于其他包，若依赖于其他第三方包，则一起安装。查询到存在要安装的包后`yum`会再次连接到服务器将需要的包下载到本地进行安装。安装完毕后`yum`会将下载到本地的包进行删除。元数据不删除，当下次安装时`yum`会直接查询本地的元数据。

#### YUM客户端配置文件

yum客户端配置文件：

```bash
/etc/yum.conf：为所有仓库提供公共配置
/etc/yum.repos.d/*.repo：为仓库的指向提供配置
```

仓库指向的定义：

```bash
[repositoryID]
name=Some name for this repository 
baseurl=url://path/to/repository/ 
enabled={1|0}						# 启用和禁用仓库，默认为1启用仓库，0为禁用
gpgcheck={1|0} 						# 作校验，0为不做校验，1为做校验
gpgkey=URL 							# 若gpgcheck=1或没有则必须指定gpgkey的位置。
enablegroups={1|0}
failovermethod={roundrobin|priority} 
# roundrobin：意为随机挑选，默认值
# priority:按顺序访问
cost=	# 默认为1000
```

yum的repo配置文件中可用的变量：

* `$releasever`: 当前OS的发行版的主版本号

* `$arch`: 平台，i386,i486,i586,x86_64等

* `$basearch`: 基础平台；i386, x86_64

* `$YUM0`-`$YUM9`: 自定义变量

##### yum客户端配置示例

修改配置文件

```bash
[root@mylinuxops ~]# vim /etc/yum.repos.d/base.repo
# 以下为配置文件中的必要内容
[BaseOS]												# 仓库名称
baseurl=file:///misc/cd/BaseOS/							# 仓库url
gpgkey=/etc/pki/rpm-gpg/RPM-GPG-KEY-centosofficial		# gpg检查，验证的公钥位置

[AppStream]
baseurl=file:///misc/cd/AppStream
gpgcheck=0												# 若gpgcheck=0，则无需公钥
```

使用yum进行安装

```bash
[root@mylinuxops ~]# yum install httpd
Repository 'BaseOS' is missing name in configuration, using id.
Repository 'AppStream' is missing name in configuration, using id.
Last metadata expiration check: 0:00:26 ago on Tue 23 Mar 2021 01:30:19 PM CST.
Dependencies resolved.
======================================================================================================================================================================
 Package                                 Architecture                Version                                                     Repository                      Size
======================================================================================================================================================================
Installing:
 httpd                                   x86_64                      2.4.37-30.module_el8.3.0+561+97fdbbcc                       AppStream                      1.7 M
Installing dependencies:
 apr                                     x86_64                      1.6.3-11.el8                                                AppStream                      125 k
 apr-util                                x86_64                      1.6.1-6.el8                                                 AppStream                      105 k
 centos-logos-httpd                      noarch                      80.5-2.el8                                                  BaseOS                          24 k
 httpd-filesystem                        noarch                      2.4.37-30.module_el8.3.0+561+97fdbbcc                       AppStream                       37 k
 httpd-tools                             x86_64                      2.4.37-30.module_el8.3.0+561+97fdbbcc                       AppStream                      104 k
 mailcap                                 noarch                      2.1.48-3.el8                                                BaseOS                          39 k
 mod_http2                               x86_64                      1.15.7-2.module_el8.3.0+477+498bb568                        AppStream                      154 k
Installing weak dependencies:
 apr-util-bdb                            x86_64                      1.6.1-6.el8                                                 AppStream                       25 k
 apr-util-openssl                        x86_64                      1.6.1-6.el8                                                 AppStream                       27 k

Transaction Summary
======================================================================================================================================================================
Install  10 Packages

Total size: 2.3 M
Installed size: 6.0 M
```

#### `yum`命令

yum命令的用法：

```bash
yum [options] [command] [package ...]
```

显示仓库列表：

```bash
yum repolist [all|enabled|disabled]
```

显示程序包：

```bash
yum list
yum list [all | glob_exp1] [glob_exp2] [...]
yum list {available|installed|updates} [glob_exp1] [...]
```

安装程序包：

```bash
yum install package1 [package2] [...]
yum reinstall package1 [package2] [...]	(重新安装)
```

升级程序包：

```bash
yum update [package1] [package2] [...]
yum downgrade package1 [package2] [...] (降级)
```

检查可用升级：

```bash
yum check-update
```

卸载程序包：

```bash
yum remove | erase package1 [package2] [...]
```

查看程序包information： 

```bash
yum info [...]
```

查看指定的特性(可以是某文件)是由哪个程序包所提供： 

```bash
yum provides | whatprovides feature1 [feature2] [...]
```

清理本地缓存：

```bash
# 清除/var/cache/yum/$basearch/$releasever缓存
yum clean [ packages | metadata | expire-cache | rpmdb | plugins | all ]
```

构建缓存：

```bash
yum makecache
```

搜索：

```bash
# 以指定的关键字搜索程序包名及summary信息
yum search string1 [string2] [...]
```

查看指定包所依赖的capabilities：

```bash
yum deplist package1 [package2] [...]
```

查看yum事务历史：

```bash
# 查看程序安装历史
yum history [info|list|packages-list|packages-info| summary|addon-info|redo|undo| rollback|new|sync|stats]

# 查看安装历史记录
yum history

# 从历史中查看某个以安装的包的信息
yum history info NUM

# 通过历史卸载程序，好处是能将其相关的依赖的包一起删除
yum history undo NUM

# yum历史是通过yum日志：/var/log/yum.log 来获得的
```

安装及升级本地程序包：

```bash
yum localinstall rpmfile1 [rpmfile2] [...] (用install替代)
yum localupdate rpmfile1 [rpmfile2] [...] (用update替代)
```

包组管理的相关命令：

```bash
yum groupinstall group1 [group2] [...] 
yum groupupdate group1 [group2] [...]
yum grouplist [hidden] [groupwildcard] [...] 
yum groupremove group1 [group2] [...] 
yum groupinfo group1 [...]
```

yum的命令行选项：

* --nogpgcheck：禁止进行gpg check

* -y: 自动回答为“yes”

* -q：静默模式

* --disablerepo=repoidglob：临时禁用此处指定的repo

* --enablerepo=repoidglob：临时启用此处指定的repo

* --noplugins：禁用所有插件

#### 系统安装光盘作为本地yum仓库：

* 挂载光盘至某目录，例如/mnt/cdrom

  ```bash
  mount /dev/cdrom /mnt/cdrom
  ```

* 创建配置文件

  ```bash
  [CentOS7] 
  name= CentOS7
  baseurl= file:///mnt/cdrom/
  gpgcheck= 0
  enabled= 1
  ```

#### 创建yum仓库：

```bash
# 当手上只有rpm包时，使用下面的命令可以创建出rpm包的元数据从而建立仓库。
createrepo [options] <directory>
```

#### 命令行创建yum配置文件

`yum-config-manager`命令可以通过命令行创建`yum`配置文件

```bash
# 生成172.16.0.1_cobbler_ks_mirror_CentOS-X-x86_64_.repo
yum-config-manager	--add-repo= http://172.16.0.1/cobbler/ks_mirror/7/
yum-config-manager --disable "仓库名"		# 禁用仓库
yum-config-manager --enable	"仓库名" 		# 启用仓库
```

#### 