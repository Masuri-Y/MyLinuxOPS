### 基于http的yum源文件搭建

#### 服务器端配置

1. 在服务器端安装httpd服务

```bash
[root@mylinuxops ~]# yum install httpd -y
```

2. 修改httpd配置文件

```bash
[root@mylinuxops conf.d]# vim /etc/httpd/conf/httpd.conf 
# 将以下几个配置修改，将路径指向到yum光盘所在的位置

#DocumentRoot "/var/www/html"
DocumentRoot "/misc/cd/"
...

#<Directory "/var/www/html">
<Directory "/misc/cd/">
```

3. 启动服务

```bash
[root@mylinuxops conf.d]# systemctl enable httpd
Created symlink /etc/systemd/system/multi-user.target.wants/httpd.service → /usr/lib/systemd/system/httpd.service.
[root@mylinuxops conf.d]# systemctl start httpd
```

#### 客户端验证

1. 配置yum客户端

```bash
[root@Computer01 ~]# vim /etc/yum.repos.d/base.repo 
[base1]
baseurl=http://172.16.11.61/BaseOS/
gpgcheck=0

[AppStream1]
baseurl=http://172.16.11.61/AppStream/
gpgcheck=0
```

2. 更新缓存

```bash
[root@Computer01 yum.repos.d]# yum makecache 
Repository 'base1' is missing name in configuration, using id.
Repository 'AppStream1' is missing name in configuration, using id.
base1                                                                                                                                 882 kB/s | 3.9 kB     00:00    
AppStream1                                                                                                                             54 MB/s | 6.2 MB     00:00    
Metadata cache created.

[root@Computer01 yum.repos.d]# yum repolist
Repository 'base1' is missing name in configuration, using id.
Repository 'AppStream1' is missing name in configuration, using id.
repo id                                                                            repo name
AppStream1                                                                         AppStream1
base1                                                                              base1
```

