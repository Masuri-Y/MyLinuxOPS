# docker常用命令
docker命令是最常使用的docker客户端命令，其后面可以加不同的参数以实现响应的功能，常用的命令如下：
### 镜像的操作命令

#### 搜索镜像 docker search
在官方的docker仓库中搜索指定名称的docker镜像，也会有很多三方镜像。
```bash
masuri@mylinuxops:~$ sudo docker search centos:7.6  #带上版本号，指定版本号
masuri@mylinuxops:~$ sudo docker search centos  #不带版本号默认为latest
```
#### 下载镜像 docker pull
从docker仓库下载镜像到本地
```bash
docker pull 仓库服务器:端口/项目名称/镜像名称:tag号
docker pull registry.cn-beijing.aliyuncs.com/mylinuxops/neginx:latest
```
如果不指定镜像仓库默认为官方仓库
```bash
masuri@mylinuxops:~$ sudo docker pull centos
Using default tag: latest
latest: Pulling from library/centos
8ba884070f61: Pull complete 
Digest: sha256:a799dd8a2ded4a83484bbae769d97655392b3f86533ceb7dd96bbac929809f3c
Status: Downloaded newer image for centos:latest
```
#### 查看本地镜像 docker images
下载完成后使用docker images进行镜像的查看，下载完成后的镜像比下载的镜像要大，因为下载后会解压。
```bash
masuri@mylinuxops:~$ sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              f68d6e55e065        2 days ago          109MB
centos              latest              9f38484d220f        3 months ago        202MB

REPOSITORY #镜像所属的仓库名称
TAG #镜像版本号（标识符），默认为 latest
IMAGE ID #镜像唯一 ID 标示
CREATED #镜像创建时间
VIRTUAL SIZE #镜像的大小
```
#### 镜像导出 docker save
可以将镜像从本地导出为一个压缩文件，然后复制到其他服务器导入使用。镜像导出有2种方法。
```bash
#方法1：
masuri@mylinuxops:~$ sudo docker save nginx -o /data/nginx.tar.gz
masuri@mylinuxops:~$ ll /data/nginx.tar.gz 
-rw------- 1 root root 113087488 Jul  4 08:24 /data/nginx.tar.gz
#方法2：
masuri@mylinuxops:~$ sudo bash -c "docker save nginx > /data/nginx-1.tar.gz"
masuri@mylinuxops:~$ ll /data/
total 220888
drwxr-xr-x  2 root root      4096 Jul  4 08:32 ./
drwxr-xr-x 24 root root      4096 Jul  4 08:24 ../
-rw-r--r--  1 root root 113087488 Jul  4 08:32 nginx-1.tar.gz
-rw-------  1 root root 113087488 Jul  4 08:24 nginx.tar.gz
```
#### 镜像导入 docker load
和镜像导出一样，镜像的导入也有两种方式
```bash
#方法1
root@mylinuxops:~# docker images        #当前镜像
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
#导入镜像
root@mylinuxops:~# docker load < /data/nginx.tar.gz  
cf5b3c6798f7: Loading layer  58.45MB/58.45MB
197c666de9dd: Loading layer  54.62MB/54.62MB
d2f0b6dea592: Loading layer  3.584kB/3.584kB
Loaded image: nginx:latest
#查看本地镜像
root@mylinuxops:~# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              f68d6e55e065        2 days ago          109MB
#方法2
root@mylinuxops:~# docker loads -i /data/nginx.tar.gz 
```
#### 镜像删除 docker rmi
本地仓库中旧版本的镜像要记得及时删除，否则镜像迟早空间占满，一般生产中删除镜像时是基于镜像TAG号来删除的，镜像在制作时在tag号中会带有镜像制作时的时间，将所标注的时间取出，然后就能根据时间将某事件点之前的镜像全部删除了
```bash
#查看当前镜像
root@mylinuxops:~# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              f68d6e55e065        2 days ago          109MB
#删除nginx镜像
root@mylinuxops:~# docker rmi nginx     #参数可以是镜像名也可以是镜像的ID
Untagged: nginx:latest
Deleted: sha256:f68d6e55e06520f152403e6d96d0de5c9790a89b4cfc99f4626f68146fa1dbdc
Deleted: sha256:1b0c768769e2bb66e74a205317ba531473781a78b77feef8ea6fd7be7f4044e1
Deleted: sha256:34138fb60020a180e512485fb96fd42e286fb0d86cf1fa2506b11ff6b945b03f
Deleted: sha256:cf5b3c6798f77b1f78bf4e297b27cfa5b6caa982f04caeb5de7d13c255fd7a1e
#再次查看本地镜像
root@mylinuxops:~# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
```
### 容器操作的基础命令

#### 启动容器 docker run
命令格式
```bash
docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
```
1.从镜像启动一个容器
```bash
# 启动一个容器并直接进入容器
root@mylinuxops:~# docker run -it centos bash
[root@36d4d4f48ad1 /]# 
# 退出容器
[root@36d4d4f48ad1 /]# exit
exit
```
2.一般将容器启动后都是将其运行在后台
```bash
# 使用-d选项将容器运行在后台
root@mylinuxops:~# docker run -d -it centos 
e9440765b19c81bfada4d8f4b26dfc2ec61c40ef79e0573a9267ccbf3ee060ad
root@mylinuxops:~# 
```
3.自定义容器名称
```bash
# 运行容器时使用--name指定容器的名称
root@mylinuxops:~# docker run -it --name nginx-test nginx
```
4.创建并进入容器
```bash
#在创建容器时需要使用 -i指定进入容器 -t指定tty，在最后还要加上具体的命令
root@mylinuxops:~# docker run -it --name nginx-test-1 nginx /bin/bash
root@26dccaaabe88:/# exit
exit
#退出后容器将退出
```
5.容器单次运行
```bash
#制作镜像和测试的环境下使用，进入容器执行操作后退出即将容器删除
root@mylinuxops:~# docker run -it --name nginx-rm nginx /bin/bash
root@7cc02218c515:/# exit
exit
#验证
root@mylinuxops:~# docker ps -f name=nginx-rm
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```
#### 查看所有容器的进程  docker ps
1.查看启动的容器的进程
```bash
# 查看启动的容器的进程
root@mylinuxops:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
e9440765b19c        centos              "/bin/bash"         11 minutes ago      Up 11 minutes                           nervous_fermi
```
2.查看所有容器的进程，包括未启动的
```bash
root@mylinuxops:~# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                         PORTS               NAMES
e9440765b19c        centos              "/bin/bash"         20 minutes ago      Up 20 minutes                                      nervous_fermi
36d4d4f48ad1        centos              "bash"              About an hour ago   Exited (0) About an hour ago                       infallible_carson
#36d4d4f48ad1 此为已经关闭的容器进程
```
#### 删除运行中的容器  docker rm
即使容器正在运行也会被删除
```bash 
#显示当前所有容器
root@mylinuxops:~# docker ps -a     
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                         PORTS               NAMES
e9440765b19c        centos              "/bin/bash"         20 minutes ago      Up 20 minutes                                      nervous_fermi
36d4d4f48ad1        centos              "bash"              About an hour ago   Exited (0) About an hour ago                       infallible_carson
#删除容器id为36d4d4f48ad1的容器
root@mylinuxops:~# docker rm -f 36d4d4f48ad1  
36d4d4f48ad1
#再次查看所有容器
root@mylinuxops:~# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
e9440765b19c        centos              "/bin/bash"         34 minutes ago      Up 34 minutes                           nervous_fermi
#36d4d4f48ad1 容器已经被关闭
```
2.批量删除已经退出的容器
```bash
#查看当前所有的容器进程，可以看到当前有许多已经退出的容器，逐个删除太麻烦可以一次性将多个退出的容器退出
root@mylinuxops:~# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                     PORTS               NAMES
4af198cb73a9        centos              "/bin/bash"         4 minutes ago       Exited (0) 4 minutes ago                       fervent_lumiere
1fd78f369bdc        centos              "/bin/bash"         4 minutes ago       Exited (0) 4 minutes ago                       relaxed_perlman
a0bf4772d01b        centos              "/bin/bash"         5 minutes ago       Exited (0) 5 minutes ago                       boring_yonath
e9440765b19c        centos              "/bin/bash"         42 minutes ago      Up 42 minutes                                  nervous_fermi
#使用ps查找出所有状态为exited的容器然后将其关闭，使用-v表示将其容器内的数据一起删除
root@mylinuxops:~# docker rm -fv `docker ps -qf status=exited`
4af198cb73a9
1fd78f369bdc
a0bf4772d01b
```
3.删除所有容器
```bash
#产看当前的所有容器
root@mylinuxops:~# docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
03087c03827a        centos              "/bin/bash"         10 seconds ago      Exited (0) 9 seconds ago                        keen_meitner
6522e6efb71e        centos              "/bin/bash"         12 seconds ago      Exited (0) 10 seconds ago                       ecstatic_johnson
1c36d3d550f1        centos              "/bin/bash"         13 seconds ago      Exited (0) 12 seconds ago                       keen_montalcini
e9440765b19c        centos              "/bin/bash"         About an hour ago   Up About an hour                                nervous_fermi
#查找出所有的容器的id然后将其删除
root@mylinuxops:~# docker rm -fv `docker ps -aq`
03087c03827a
6522e6efb71e
1c36d3d550f1
e9440765b19c
```
#### 指定端口映射
1.从本地端口映射到容器端口
```bash
#启动一个nginx容器，将本机的81号端口映射到容器的80端口
root@mylinuxops:~# docker run -d --name nginx-1 -p 81:80 nginx
4b396fbe1379073a2c197263384400fd6a1f952f44ffc0135d47aa945bc8c659
root@mylinuxops:~# 
#验证
root@mylinuxops:~# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                NAMES
4b396fbe1379        nginx               "nginx -g 'daemon of…"   About a minute ago   Up About a minute   0.0.0.0:81->80/tcp   nginx-1
```
2.本地的IP+端口映射到容器的端口
```bash
#这种方法一般使用于宿主机有多个地址的情况下
root@mylinuxops:~# docker run -d -p 192.168.27.10:82:80 --name nginx-2 nginx
3e87d3dc7722bcfc3def3f7e519c8f9fb2673c70ff906d5246c5e5962528b4d8
#查看nginx-2的端口映射
root@mylinuxops:~# docker ps -f "name=nginx-2"
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
3e87d3dc7722        nginx               "nginx -g 'daemon of…"   3 minutes ago       Up 3 minutes        192.168.27.10:82->80/tcp   nginx-2
```
3.本地的IP的随机端口映射到容器的端口
```bash
root@mylinuxops:~# docker run -d -p 192.168.27.10::80 --name nginx-3 nginx
7066dcf8a065
#验证
root@mylinuxops:~# docker ps -f "name=nginx-3"
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                         NAMES
7066dcf8a065        nginx               "nginx -g 'daemon of…"   About a minute ago   Up About a minute   192.168.27.10:32768->80/tcp   nginx-3
```
4.使用本机的ip+port/协议，默认为tcp协议
```bash
root@mylinuxops:~# docker run -d -p192.168.27.10:53:53/udp --name nginx-4 nginx
e2b5651195f42afeb3b315afd692853fcbe72c93c65b783e638f7d72167ab299
#验证
root@mylinuxops:~# docker ps -f name=nginx-4
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                              NAMES
e2b5651195f4        nginx               "nginx -g 'daemon of…"   49 seconds ago      Up 48 seconds       192.168.27.10:53->53/udp, 80/tcp   nginx-4
```
5.一次性映射多个端口和协议
```bash
root@mylinuxops:~# docker run -d-p 86:80/tcp -p 443:443/tcp -p 54:54/udp --name nginx-6 nginx
1d96f3555909
#验证
root@mylinuxops:~# docker ps -f name=nginx-6
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                                                          NAMES
1d96f3555909        nginx               "nginx -g 'daemon of…"   6 minutes ago       Up 6 minutes        0.0.0.0:54->54/udp, 0.0.0.0:443->443/tcp, 0.0.0.0:86->80/tcp   nginx-6
```
#### 随机映射端口
使用-P分配随机端口，随机端口映射默认从32768开始向后分配端口
```bash
root@mylinuxops:~# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED              STATUS              PORTS                              NAMES
e1b1e33b68f1        nginx               "nginx -g 'daemon of…"   About a minute ago   Up About a minute   0.0.0.0:32768->80/tcp              infallible_colden
```
#### 查看容器已经映射的端口 docker port
使用docker port [容器名称] 查看映射的段口
```bash
root@mylinuxops:~# docker port nginx-1
80/tcp -> 0.0.0.0:81
```
#### 容器的启动和关闭
```bash
docker start|stop  容器ID
```
### 进入到正在运行的容器
#### 1.使用attach命令
语法格式
```bash 
docker attach 容器名
```
attach类似于vnc，操作会在各个容器界面显示，所有使用此方式进入容器的操作都是同步显示的且exit后容器将被关闭，且使用exit退出后容器关闭，不推荐使用，需要进入到有 shell环境的容器，
#### 2.使用exec命令
执行单次命令与进入容器，不是很推荐此方式，虽然exit退出容器还在运行
```bash
# 查看当前正在运行的容器
root@mylinuxops:~# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
7cc02218c515        nginx               "/bin/bash"         6 minutes ago       Up 4 seconds        80/tcp              nginx-rm
# 使用docker exec进入容器，需要带上-it选项和执行的命令
root@mylinuxops:~# docker exec -it 7cc02218c515 /bin/bash
root@7cc02218c515:/# 
```
#### 3.使用nsenter命令
推荐使用此方式，nsenter命令需要通过PID进入到容器内部，不过可以使用docker inspect获取到容器的PID:
1.首先需要安装nsenter命令
```bash
root@mylinuxops:~# apt install util-linux -y
```
2.使用docker inspect + 容器ID 可以获取到容器的各种信息
```bash
#不过滤参数为列出容器所有的信息
root@mylinuxops:~# docker inspect 7cc02218c515
#查询容器相关的某个参数可以使用-f选项加上参数名，比如获取容器的ip地址
root@mylinuxops:~# docker inspect -f "{{.NetworkSettings.IPAddress}}" 7cc02218c515
172.17.0.2          #获得了容器的IP地址
#或取容器的PID号，最后所带的参数可以时容器名称也可以是容器ID
root@mylinuxops:~# docker inspect -f "{{.State.Pid}}" 7cc02218c515  
10874       #获得了容器的PID号
```
3.使用PID号进入容器
```bash
#执行nsenter命令进入容器
root@mylinuxops:~# nsenter -t 10874 -m -u -i -n -p
root@7cc02218c515:/# 
```
#### 4.使用脚本进入容器

在使用nsenter命令进入容器时，一般会将命令写入脚本然后进行调用，一般建议在每台运行k8s的主机上都存放一个脚本方便进入容器
```bash
root@mylinuxops:~# vim docker-in.sh
#!/bin/bash
docker_in(){
        NAME_ID=$1
        PID=$(docker inspect -f "{{.State.Pid}}" ${NAME_ID})
        nsenter -t ${PID} -m -u -i -n -p
}
docker_in $1
```
测试脚本
```bash
#启动一个容器
root@mylinuxops:~# docker run -d --name nginx-test nginx 
81c2c728e5cc22f94a551778df0d61260faa1f269e86d0e36002b7ba4869f819
#执行脚本进入容器
root@mylinuxops:~# bash docker-in.sh nginx-test
root@81c2c728e5cc:/#              
```
#### 指定容器DNS
容器的DNS服务，默认采用宿主机的dns地址
一是将dns地址配置在宿主机
二是将参数配置在docker启动脚本里，使用--dns IP 来指定dns
```bash
root@mylinuxops:~# docker run -it --dns 223.6.6.6 nginx bash
root@813870a2f3a8:/# cat /etc/resolv.conf 
nameserver 223.6.6.6
```

