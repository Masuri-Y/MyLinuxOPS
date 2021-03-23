### `DNF`(`DaNdiFied`)

`DNF` 介绍：新一代的RPM软件包管理器。`DNF` 发行日期是2015年5月11日，`DNF`包管理器采用`Python`编写，发行许可为`GPL v2`，首先出现在`Fedora 18`发行版中。在`RHEL 8.0`版本正式取代了 `YUM`，`DNF`包管理器克服了YUM包管理器的一些瓶颈，提升了包括用户体验，内存占用，依赖分析，运行速度。

#### `DNF`在`CentOS7`上安装

安装所需软件包，依赖`epel`源

```bash
wget http://springdale.math.ias.edu/data/puias/unsupported/7/x86_64/dnf-conf-0.6.4-1. sdl7.noarch.rpm
wget http://springdale.math.ias.edu/data/puias/unsupported/7/x86_64//dnf-0.6.4-.sdl7.noarch.rpm
wget http://springdale.math.ias.edu/data/puias/unsupported/7/x86_64/python-dnf-0.6.4-2.sdl7.noarch.rpm

yum install python-dnf-0.6.4-2.sdl7.noarch.rpm dnf-0.6.4-2.sdl7.noarch.rpm dnf-conf-0.6.4-2.sdl7.noarch.rpm
```

#### `DNF`相关文件

配置文件: `/etc/dnf/dnf.conf`

仓库文件: `/etc/yum.repos.d/ *.repo`

日志: `/var/log/dnf.rpm.log`

帮助: `man dnf`

#### `DNF`使用

`dnf` 用法与yum一致

```bash
dnf	[options] <command> [<arguments>...] 
dnf --version
dnf	repolist 
dnf	clean all 
dnf	makecache
dnf list installed 
dnf list available 
dnf search nano 
dnf history
dnf history undo 1
```