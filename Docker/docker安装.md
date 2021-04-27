# Docker CE的安装
官方网址：https://www.docker.com/
系统版本选择：
Docker目前已经支持多种操作系统的安装运行，比如 Ubuntu、CentOS、Redhat、Debian、Fedora，甚至是还支持了Mac和Windows，在linux系统上需要内核版本在3.10或以上，docker版本号之前一直是0.x版本或1.X版本，但是从2017年3月1号开始改为每个季度发布一次稳版，其版本号规则也统一变更为YY.MM，例如17.09表示是 2017年9月份发布的，本次演示的操作系统使用 Centos 7.6为例。

Docker版本选择：
Docker之前没有区分版本，但是2017年推出(将docker更名为)新的项目Moby，github 地址：https://github.com/moby/moby，Moby项目属于Docker项目的全新上游，Docker 将是一个隶属于的 Moby 的子产品，而且之后的版本之后开始区分为CE版本（社区版本）和EE（企业收费版），CE社区版本和EE企业版本都是每个季度发布一个新版本，但是EE版本提供后期安全维护1年，而CE 版本是4个月，本次演示的Docker版本为18.03，以下为官方原文：
https://blog.docker.com/2017/03/docker-enterprise-edition/

***
## 使用官方安装脚本自动安装（仅适用于公网环境）
```bash
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
```
***
## 手动安装Docker(Ubuntu)
使用系统Ubuntu 18.04.2  
1.安装必要的一些系统工具
```bash
apt update 
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
```
2.安装gpg证书
```bash
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```
3.写入软件源信息
```bash
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
```
4.安装Docker-CE
```bash
sudo apt-get -y install docker-ce
```
5.配置阿里云的docker镜像加速
```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
 "registry-mirrors": ["https://gy97ij1m.mirror.aliyuncs.com"]
}
EOF
{
  "registry-mirrors": ["https://gy97ij1m.mirror.aliyuncs.com"]
}
sudo systemctl daemon-reload
sudo systemctl restart docker
```
6.查看docker的版本信息
```bash
root@mylinuxops:~# docker version
Client:
 Version:           18.09.7
 API version:       1.39
 Go version:        go1.10.8
 Git commit:        2d0083d
 Built:             Thu Jun 27 17:56:23 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          18.09.7
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.10.8
  Git commit:       2d0083d
  Built:            Thu Jun 27 17:23:02 2019
  OS/Arch:          linux/amd64
  Experimental:     false
```
7.验证docker信息
```bash
masuri@mylinuxops:~$ sudo docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 18.09.7
Storage Driver: overlay2
 Backing Filesystem: extfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
 Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary: docker-init
containerd version: 894b81a4b802e4eb2a91d1ce216b8817763c29fb
runc version: 425e105d5a03fabd737a126ad93d62a9eeede87f
init version: fec3683
Security Options:
 apparmor
 seccomp
  Profile: default
Kernel Version: 4.15.0-54-generic
Operating System: Ubuntu 18.04.2 LTS
OSType: linux
Architecture: x86_64
CPUs: 12
Total Memory: 3.83GiB
Name: mylinuxops
ID: 6DNE:LEH7:IBXO:PGXW:ZRAB:AQL4:TSBN:2QKT:ZBF3:HOPB:CZAF:AOWB
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Labels:
Experimental: false
Insecure Registries:
 127.0.0.0/8
Registry Mirrors:
 https://gy97ij1m.mirror.aliyuncs.com/
Live Restore Enabled: false
Product License: Community Engine

WARNING: No swap limit support
```
***
## 手动安装docker(CentOS7)
1.安装一些必要工具
```bash
yum install -y yum-utils device-mapper-persistent-data lvm2
```
2.添加软件源信息
```bash
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
3.更新并安装 Docker-CE
```bash
yum makecache fast
yum -y install docker-ce
```
4.启动镜像服务
```bash
systemctl start docker
```
5.配置阿里云的docker镜像加速
```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://gy97ij1m.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```
6.验证docker版本
```bash
[root@localhost ~]# docker version
Client:
 Version:           18.09.7
 API version:       1.39
 Go version:        go1.10.8
 Git commit:        2d0083d
 Built:             Thu Jun 27 17:56:06 2019
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          18.09.7
  API version:      1.39 (minimum version 1.12)
  Go version:       go1.10.8
  Git commit:       2d0083d
  Built:            Thu Jun 27 17:26:28 2019
  OS/Arch:          linux/amd64
  Experimental:     false
```
7.验证docker信息
```bash
[root@localhost ~]# docker info
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 18.09.7
Storage Driver: overlay2
 Backing Filesystem: xfs
 Supports d_type: true
 Native Overlay Diff: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
 Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary: docker-init
containerd version: 894b81a4b802e4eb2a91d1ce216b8817763c29fb
runc version: 425e105d5a03fabd737a126ad93d62a9eeede87f
init version: fec3683
Security Options:
 seccomp
  Profile: default
Kernel Version: 3.10.0-957.el7.x86_64
Operating System: CentOS Linux 7 (Core)
OSType: linux
Architecture: x86_64
CPUs: 1
Total Memory: 468.6MiB
Name: localhost.localdomain
ID: ONTA:5VC5:GYUU:CQ6P:SZFM:2R6K:4ZEJ:MUIK:R7NR:ROC5:NYZL:OLFS
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Labels:
Experimental: false
Insecure Registries:
 127.0.0.0/8
Registry Mirrors:
 https://gy97ij1m.mirror.aliyuncs.com/
Live Restore Enabled: false
Product License: Community Engine

WARNING: bridge-nf-call-iptables is disabled
WARNING: bridge-nf-call-ip6tables is disabled
```
## docker存储引擎
目前docker的默认存储引擎为overlay2，需要磁盘分区支持d-type文件分层功能，因此需要系统磁盘的额外支持。
官方文档关于存储引擎的选择文档：https://docs.docker.com/storage/storagedriver/select-storage-driver/
Docker官方推荐首选存储引擎为overlay2其次为devicemapper，但是devicemapper存在使用空间方面的一些限制存储空间的上限为100G，虽然可以通过后期配置解决，但官方依然推荐使用overlay2。
如果docker数据目录是一块单独的磁盘分区而且是xfs格式的，那么需要在格式化的时候加上参数-n ftype=1，否则后期在启动容器的时候会报错不支持dtype。
```bash
[root@localhost ~]# xfs_info /
meta-data=/dev/mapper/centos-root isize=512    agcount=4, agsize=13041408 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=52165632, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1      #ftype必须为1，否则将不支持dtype
log      =internal               bsize=4096   blocks=25471, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

