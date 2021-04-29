## `Dockerfile`指令详解

### 1.`FROM`指令

* `FROM`指令是最重要的一个且必须为`Dockerfile`文件开篇的第一个非注释行，用于为映像文件构建过程指定基准镜像，后续的指令运行于此基准镜像所提供的运行环境。

* 实践中，基准镜像可以是仍和可用镜像文件，默认情况下，`docker build`会在`docker`主机上查找指定的镜像文件，在其不存在时，则会从`Docker Hub Registry`上拉取所需的镜像文件。

  * 如果找不到指定的镜像文件，`docker build`会返回一个错误信息

格式：

  ```bash
  FROM <repository>[:<tag>] 
  # 或
  FROM <repository>@<digest>
  
  # <repository>: 指定作为baseimage的名称
  # <tag>: base image的标签，为可选项，省略时默认为lastest
  ```

### 2.`MAINTANIER`指令

* 用于让`Dockerfile`制作者提供本人的详细信息
* `Dockerfile`并不限制`MAINTAINER`指令可出现的位置，但推荐将其放置于`FROM`指令之后

格式：

```bash
MAINTANIER <authtor's detail>
# <authtor's detail>可以是任何文本信息，但约定俗成地使用作者名称及邮件地址

MAINTANIER "masuri<438214186@qq.com>"
```

### 3.`LABEL`指令

* `LABEL`指令用于替代`MAINTANIER`

* LABEL是key-value对，可以提供众多的信息。

格式：

```bash
LABEL <KEY>=<VALUE> <KEY>=<VALUE> <KEY>=<VALUE> ...
```

### 4.`COPY`指令

* 用于从Docker主机复制文件至创建的新镜像文件

格式：

```bash
COPY <SRC>...<DEST> 
# 或
COPY ["<SRC>",..."<DEST>"]
# <SRC>: 要复制的源文件或目录，支持使用通配符
# <DEST>: 目标路劲给，即正在创建的image的文件系统的路径；建议为<dest>使用绝对路径，否则，COPY指定则以WORKDIR指令所指的路径为起始路径;
# 在路径中有空白字符时，通常使用第二种格式
```

文件复制准则：

* `<SRC>`必须是build上下文中的路径，不能是其父目录中的文件
* 如果`<SRC>`是目录，则其内部文件或子目录会被递归复制，但`<SRC>`目录本身不会被复制
* 如果指定了多个`<SRC>`，或在`<SRC>`中使用了通配符，则`<DEST>`必须是以`/`结尾
* 如果`<DEST>`事先不存在，他将会被自动创建，这包括其父目录路径

#### 综合练习

建立一个`docker build`工作目录，在工作目录内创建一个`html`目录，在`html`目录中创建多个`.html`文件，要求`test1.html`文件不被打入镜像。

```bash
# 创建出工作目录，html目录及html文件
[root@CentOS8 ~]# mkdir build_workshop
[root@CentOS8 ~]# cd build_workshop/
[root@CentOS8 build_workshop]# mkdir html
[root@CentOS8 build_workshop]# echo "tiny web server based on busybox" >> html/index.html
[root@CentOS8 build_workshop]# echo "test1 file" >> html/test1.html
[root@CentOS8 build_workshop]# echo "test2 file" >> html/test2.html

# 编写dockerignore文件
[root@CentOS8 build_workshop]# echo "html/test1.html" >> .dockerignore

# 编写Dockerfile文件
[root@CentOS8 build_workshop]# vim Dockerfile
FROM busybox:latest
LABEL maintanier="Masuri<438214186@qq.com>"
COPY html /data/www/html/

# 在工作目录内build docker镜像
[root@CentOS8 build_workshop]# docker build . -t masuri/myweb:v0.01
Sending build context to Docker daemon  5.632kB
Step 1/3 : FROM busybox:latest
 ---> 388056c9a683
Step 2/3 : LABEL maintanier="Masuri<438214186@qq.com>"
 ---> Using cache
 ---> db9675c45b91
Step 3/3 : COPY html /data/www/html/
 ---> 81da6ac19386
Successfully built 81da6ac19386
Successfully tagged masuri/myweb:v0.01

# 启动镜像为容器，验证内容
[root@CentOS8 build_workshop]# docker run --name t1 -it --rm masuri/myweb:v0.01 /bin/sh
/ # ls /data/www/html/
index.html  test2.html                    # test1.html文件被忽略了
```

### 5.`ADD`指令

* `ADD`指令类似于`COPY`指令，`ADD`支持使用`tar`文件和`url`路径

格式：

```bash
ADD <SRC>...<DEST>
# 或
ADD ["<SRC>",..."<DEST>"]
```

操作准则

* 同`COPY`指令
* 如果`<SRC>`为`URL`且`<DEST>`不以`/`结尾，则`<src>`指定的文件被下载并直接被创建为`<dest>`;如果`<dest>`以`/`结尾，则文件名`URL`指定的文件将被直接下载并保存为`<dest>/<filename>`。
* 如果`<SRC>`是一个本地系统上的压缩格式的tar文件，它将被展开为一个目录，其行为类似于`tar -x`命令；然而通过`URL`获取到的`tar`文件将不会自动展开。
* 如果`<src>`有多个，或其间接或直接使用了通配符，则`<dest>`必须是一个以/结尾的目录路径；如果`<dest>`不以`/`结尾，则其被视作一个普通文件，`<src>`的内容将被直接写入到`<dest>`

#### `ADD`指令示例

```bash
# 创建工作目录
[root@CentOS8 ~]# mkdir build_workshop
[root@CentOS8 ~]# cd build_workshop/

# 下载一个nginx打包压缩文件
[root@CentOS8 build_workshop]# wget http://nginx.org/download/nginx-1.20.0.tar.gz
[root@CentOS8 build_workshop]# ls
nginx-1.20.0.tar.gz

# 编写dockerfile，分别使用URL和本地文件两种方式添加文件
[root@CentOS8 build_workshop]# vim Dockerfile
FROM busybox:latest
LABEL maintanier="Masuri<438214186@qq.com>"
ADD http://nginx.org/download/nginx-1.18.0.tar.gz /tmp/
ADD nginx-1.20.0.tar.gz /usr/src/

# 制作镜像
[root@CentOS8 build_workshop]# docker build . -t masuri/myweb:v0.02
Sending build context to Docker daemon  1.064MB
Step 1/4 : FROM busybox:latest
 ---> 388056c9a683
Step 2/4 : LABEL maintanier="Masuri<438214186@qq.com>"
 ---> Using cache
 ---> db9675c45b91
Step 3/4 : ADD http://nginx.org/download/nginx-1.18.0.tar.gz /tmp/
Downloading [==================================================>]   1.04MB/1.04MB
 ---> b76454839061
Step 4/4 : ADD nginx-1.20.0.tar.gz /usr/src/
 ---> 5a2400adf1b0
Successfully built 5a2400adf1b0
Successfully tagged masuri/myweb:v0.02

# 启动容器验证
[root@CentOS8 build_workshop]# docker run --name t2 --rm -it masuri/myweb:v0.02 /bin/sh
/ # ls /usr/src
nginx-1.20.0			# 本地的tar文件放到src目录下被解压
/ # ls /tmp
nginx-1.18.0.tar.gz		# 使用url的tar文件没有被解压
```

### 6.`WORKDIR`指令

* 用于为`Dockerfile`中所有的`RUN`、`CMD`、`ENTRYPOINT`、`COPY`和`ADD`指定设定工作目录

格式：

```bash
WORKDIR <DIRPATH>
# 在Dockerfile文件中，WORKDIR指令可以出现多次，其路径也可以为相对路径，不过，其是相对此前一个WORKDIR指令指定的路径
# 另外WORKDIR也可以调用由ENV指定定义的变量
# 如：
WORKDIR /var/log
WORKDIR $STATEPATH
```

### 7.`VOLUME`指令

* 用于在`image`中创建一个挂载点目录，以挂载`Docker host`上的卷或其他容器上的卷

格式

```bash
VOLUME <MOUNTPOINT>
# 或
VOLUME ["<MOUNTPOINT>"]
```

* 如果挂载点目录下此前存在文件，`docker run`命令会在卷挂载完成后将此前目录中所存在的所有文件复制到新挂载的卷中

* 如果`docker run`指令不指定`-v`选项来绑定宿主机上的卷路径，`volume`指令使用的将是`docker`所管理的卷

#### `VOLUME`示例

```bash
# 当前目录下文件
[root@CentOS8 build_workshop]# tree
.
├── Dockerfile
├── html
│   ├── index.html
│   ├── test1.html
│   └── test2.html
└── nginx-1.20.0.tar.gz

# 创建Dockerfile文件，指定WORKDIR，以及挂载的卷
[root@CentOS8 build_workshop]# vim Dockerfile
FROM busybox:latest
LABEL maintanier="Masuri<438214186@qq.com>"
COPY html /data/www/html/
ADD http://nginx.org/download/nginx-1.18.0.tar.gz /tmp/
ADD nginx-1.20.0.tar.gz /usr/src/
WORKDIR /data/www/
VOLUME /data/www/html/

# build镜像
[root@CentOS8 build_workshop]# docker build . -t masuri/myweb:v0.03

# 启动容器验证，原先html目录下的内容依旧存在
[root@CentOS8 build_workshop]# docker run --name t1 -it masuri/myweb:v0.03 /bin/sh
/data/www # ls
html
/data/www # cd html/
/data/www/html # ls
index.html  test1.html  test2.html

# 在宿主机上查看挂载卷位置
[root@CentOS8 build_workshop]# docker inspect -f {{.Mounts}} t1
[{volume 8a8764f450c705c226918c80a9e408f56620244be86969e668ff455ca96fbbe8 /var/lib/docker/volumes/8a8764f450c705c226918c80a9e408f56620244be86969e668ff455ca96fbbe8/_data /data/www/html local  true }]

# 查看卷内数据
[root@CentOS8 build_workshop]# ls /var/lib/docker/volumes/8a8764f450c705c226918c80a9e408f56620244be86969e668ff455ca96fbbe8/_data
index.html  test1.html  test2.html
```

### 8.`EXPOSE`指令

* 用于为容器打开指定要监听的端口以实现与外部通信

格式

```bash
EXPOSE <port>[/<protocol>] [<port>[/<protocol>]...]
# <protocol> 用于指定传输层协议，可为tcp或udp二者之一，默认为TCP协议
```

* `EXPOSE`指令可一次指定多个端口，例如

  ```bash
  EXPOSE 11211/udp 11211/tcp
  ```

#### `EXPOSE`示例

```bash
# 当前目录下文件
[root@CentOS8 build_workshop]# tree
.
├── Dockerfile
├── html
│   ├── index.html
│   ├── test1.html
│   └── test2.html
└── nginx-1.20.0.tar.gz

# 创建Dockerfile文件，暴露出80端口
[root@CentOS8 build_workshop]# vim Dockerfile
FROM busybox:latest
LABEL maintanier="Masuri<438214186@qq.com>"
COPY html /data/www/html
ADD http://nginx.org/download/nginx-1.18.0.tar.gz /tmp/
ADD nginx-1.20.0.tar.gz /usr/src/
WORKDIR /data/www/
VOLUME /data/www/html
EXPOSE 80/tcp

# 构建镜像文件
[root@CentOS8 build_workshop]# docker build . -t masuri/myweb:v0.04

# 运行容器
[root@CentOS8 build_workshop]# docker run --name t1 -it -P  masuri/myweb:v0.04 /bin/sh
# 需要注意虽然Dockerfile中带了暴露80端口，若是容器运行时不使用-P是不会将端口暴露出去的

# 查看端口映射
[root@CentOS8 build_workshop]# docker port t1
80/tcp -> 0.0.0.0:49153
80/tcp -> :::49153
```

### 9.`ENV`指令

* 用于为镜像定义所需的环境变量，并可被`Dockerfile`文件中位于其后的其他指令(如`ENV`、`ADD`、`COPY`等)所调用
* 调用格式为`$variable_name`或`${variable_name}`

格式

```bash
ENV <KEY> <VALUE> 
# 或
ENV <KEY>=<VALUE> ...
```

* 第一种格式中，`<key>`之后的所有内容均会被视作其`<value>`的组成部分，因此一次只能设置一个变量；
* 第二种格式可以一次设置多个变量，每个变量为一个"`<key>=<value>`"的键值对，如果`<value>`中包含空格，可以以反斜线`\`进行转义，也可通过对`<value>`加引号进行标识；另外，反斜线也可用于续行；
* 定义多个变量时，建议使用第二种方式，以便在同一层中完成所有功能。

#### `ENV`示例

```bash
# 当前目录下文件
[root@CentOS8 build_workshop]# tree
.
├── Dockerfile
├── html
│   ├── index.html
│   ├── test1.html
│   └── test2.html
└── nginx-1.20.0.tar.gz

# 将/data/www/html/目录定义成变量，之后使用时进行调用
[root@CentOS8 build_workshop]# vim Dockerfile
FROM busybox:latest
LABEL maintanier="Masuri<438214186@qq.com>"
ENV webhome="/data/www/html/"
COPY html $webhome
ADD http://nginx.org/download/nginx-1.18.0.tar.gz /tmp/
ADD nginx-1.20.0.tar.gz /usr/src/
WORKDIR /data/www/
VOLUME $webhome
EXPOSE 80/tcp

# 构建镜像文件
[root@CentOS8 build_workshop]# docker build . -t masuri/myweb:v0.05

# 运行容器验证ENV是否生效
[root@CentOS8 build_workshop]# docker run --name t1 -P -it masuri/myweb:v0.05 /bin/sh
/data/www # ls html/
index.html  test1.html  test2.html  # 文件已存在
```

### 10.`ARG`指令

* `ENV`指令在容器构建时直接被定死无法改变，如果需要改变则需要对`Dockerfile`文件进行修改。`ARG`指令可以定义一个变量并且此变量可以没有值，让用户在构建镜像时使用`--build-arg <varname>=<value>`，来临时进行更改。

* 

格式

```bash
ARG <NAME>[=default value]
```

#### `ARG`指令示例

```bash
# 当前目录下文件
[root@CentOS8 build_workshop]# tree
.
├── Dockerfile
├── html
│   ├── index.html
│   ├── test1.html
│   └── test2.html
└── nginx-1.20.0.tar.gz

# 将/data/www/html/目录定义成ARG指令变量，之后使用时进行调用
[root@CentOS8 build_workshop]# vim Dockerfile
FROM busybox:latest
LABEL maintanier="Masuri<438214186@qq.com>"
ARG webhome="/data/www/html/"
COPY html $webhome
ADD http://nginx.org/download/nginx-1.18.0.tar.gz $webhome
ADD nginx-1.20.0.tar.gz /usr/src/
WORKDIR $webhome
VOLUME $webhome
EXPOSE 80/tcp

# 构建镜像文件，将webhome变量更改
[root@CentOS8 build_workshop]# docker build --build-arg webhome="/data/htdocs/" . -t masuri/myweb:v0.06

# 进入容器验证，构建时的webhome生效
[root@CentOS8 build_workshop]# docker run --name t1 -it masuri/myweb:v0.06 /bin/sh
/data/htdocs # ls
index.html           nginx-1.18.0.tar.gz  test1.html           test2.html
```

### 11.`RUN`指令

* 用于指定`docker build`过程中运行的程序，其可以是任何命令

格式

```bash
RUN <COMMAND>
# 或
RUN ["<executable>","<param1>","<param2>"]
```

* 第一种格式中，`<command>`通常是一个shell命令，且以`/bin/sh -c`来运行它，这意味着此进程在容器中的`PID`不为1，不能接收UNIX信号，因此，当使用`docker stop <container>`命令停止容器时，此进程接收不到`SIGTERM`信号

* 第二种语法格式中的参数是一个`JSON`格式的数组，其中`<executable>`为要运行的命令，后面的`<parmN>`为传递给命令的选项或参数；然而，此种格式指定的命令不会以`/bin/sh -c`来发起，因此常见的shell操作如变量替换以及通配符(?*)替换将不会进行；不过，如果要运行的命令依赖于shell特性的话，可以将其替换为类似下面的格式。

  ```bash
  RUN ["/bin/sh","-c","<executable>","<param1>"]
  ```

#### RUN指令示例

```bash
```

