## docker仓库之分布式Harbor
Harbor 是一个用于存储和分发 Docker 镜像的企业级 Registry 服务器，由vmware 开源，其通过添加一些企业必需的功能特性，例如安全、标识和管理等，扩展了开源 Docker Distribution。

作为一个企业级私有 Registry 服务器，Harbor 提供了更好的性能和安全。提升用户使用 Registry 构建和运行环境传输镜像的效率。Harbor 支持安装在多个 Registry 节点的镜像资源复制，镜像全部保存在私有 Registry 中， 确保数据和知识产权在公司内部网络中管控，另外，Harbor 也提供了高级的安全特性，诸如用户管理，访问控制和活动审计等。

官网地址：https://vmware.github.io/harbor/cn/

官方 github 地址：https://github.com/vmware/harbor

### Harbor 功能官方介绍：
基于角色的访问控制：用户与 Docker 镜像仓库通过“项目”进行组织管理，一个用户可以对多个镜像仓库在同一命名空间（project）里有不同的权限。

镜像复制：镜像可以在多个 Registry 实例中复制（同步）。尤其适合于负载均衡，高可用，混合云和多云的场景。

图形化用户界面：用户可以通过浏览器来浏览，检索当前 Docker 镜像仓库，管理项目和命名空间。

AD/LDAP 支：Harbor 可以集成企业内部已有的 AD/LDAP，用于鉴权认证管理。

审计管理：所有针对镜像仓库的操作都可以被记录追溯，用于审计管理。

国际化：已拥有英文、中文、德文、日文和俄文的本地化版本。更多的语言将会添加进来。

RESTful API - RESTful API ：提供给管理员对于 Harbor 更多的操控, 使得与其它管理软件集成变得更容易。

部署简单：提供在线和离线两种安装工具， 也可以安装到 vSphere 平台(OVA 方式)虚拟设备。

## 安装Harbor
下载地址：https://github.com/vmware/harbor/releases

安装文档：https://github.com/vmware/harbor/blob/master/docs/installation_guide.md

本次安装示范使用harbor1.7.5版本

***
1.添加一块磁盘用来当harbor的存储使用，将磁盘格式化挂载到docker目录

```bash
#将磁盘格式化为xfs，需要注意格式化完毕后查看ftype是否为1
[root@localhost ~]# mkfs.xfs /dev/sdb
#创建出docker的数据目录，并把磁盘挂载上去
[root@localhost ~]# mkdir /var/lib/docker
[root@localhost ~]# mount /dev/sdb /var/lib/docker/
```
2.下载并安装docker
```bash
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```
3.下载harbor
```bash
[root@localhost ~]# wget https://storage.googleapis.com/harbor-releases/release-1.7.0/harbor-offline-installer-v1.7.5.tgz
#解压
[root@localhost ~]# tar xf harbor-offline-installer-v1.7.5.tgz -C /usr/local/src
```
4.编辑配置文件
```bash
[root@localhost ~]# vim /usr/local/harbor/harbor.cfg 
#hostname改为本机地址
hostname = 192.168.27.10
#修改密码
harbor_admin_password = 123456
```
5.安装docker组件
```bash
#需要启用epel源
[root@localhost harbor]# yum install docker-compose -y
```
6.执行安装脚本
```bash
[root@localhost ~]# cd /usr/local/src/harbor/
[root@localhost harbor]# ./install.sh 
```