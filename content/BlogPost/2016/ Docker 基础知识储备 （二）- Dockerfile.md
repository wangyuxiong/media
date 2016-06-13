# Docker 基础知识储备 （二）- Dockerfile

使用Dockerfile创建自定义的镜像。

## 基本结构
Dockerfile由一行行命令语句组成，支持# 开头的注释行

四部分： 基础镜像信息，维护者信息，镜像操作指令和容器启动时执行的指令。

example:

```
# This dockerfile uses the ubuntu image
# VERSION 2 - EDITION 1
# Author: docker_user
# Command format: Instruction [arguments / command] ..

# Base image to use, this must be set as the first line
FROM ubuntu

# Maintainer: docker_user <docker_user at email.com> (@docker_user)
MAINTAINER docker_user docker_user@email.com

# Commands to update the image
RUN echo "deb http://archive.ubuntu.com/ubuntu/ raring main universe" >> /etc/apt/sources.list
RUN apt-get update && apt-get install -y nginx
RUN echo "\ndaemon off;" >> /etc/nginx/nginx.conf

# Commands when creating a new container
CMD /usr/sbin/nginx
```

## 指令
### FROM 
**格式：**`FROM image or  FROM image:tag`
**注意：**第一条指令必须是 FROM 指令。一个Dockerfile中创建多个镜像，可以使用多个FROM指令。

### MAINTAINER
**格式：** `MAINTAINER name`

### RUN
**格式：** `RUN command or RUN ["executable","param1","param2"]`
**注意：** RUN command 是在shell终端运行命令，即 /bin/sh -c;

### CMD
**格式：** 
 - `CMD ["executable","param1","param2"]`  使用  exec  执行，推荐方式
 - `CMD command param1 param2`  在  /bin/sh  中执行，提供给需要交互的应用
 - `CMD ["param1","param2"]`  提供给  ENTRYPOINT  的默认参数

** 注意：** 指定启动容器时执行的命令，每个 Dockerfile 只能有一条  CMD  命令。如果指定了多条命令，只有最后一条会被执行。

### EXPOSE
**格式：** `EXPOSE <port> [<port>...] `
**注意：**告诉 Docker 服务端容器暴露的端口号，供互联系统使用。在启动容器时需要通过 -P，Docker 主机会自动分配一个端口转发到指定的端口。

### ENV
**格式：** `ENV <key> <value>`
**注意：**指定一个环境变量，会被后续  RUN  指令使用，并在容器运行时保持

### ADD
**格式：** `ADD <src> <dest>`
**注意：**该命令将复制指定的  <src>  到容器中的  <dest>.其中  <src>  可以是Dockerfile所在目录的一个相对路径；也可以是一个 URL；还可以是一个 tar 文件（自动解压为目录）。

### COPY
**格式：** `COPY <src> <dest> `
**注意：** 复制本地主机的  <src> （为 Dockerfile 所在目录的相对路径）到容器中的  <dest> .当使用本地目录为源目录时，推荐使用  COPY 

### ENTRYPOINT
**格式：**
- `ENTRYPOINT ["executable", "param1", "param2"] `
- `ENTRYPOINT command param1 param2` （shell中执行）

注：配置容器启动后执行的命令，并且不可被  docker run  提供的参数覆盖。每个 Dockerfile 中只能有一个  ENTRYPOINT ，当指定多个时，只有最后一个起效。

### VOLUME
**格式：** `VOLUME ["/data"]`
**注意：**创建一个可以从本地主机或其他容器挂载的挂载点，一般用来存放数据库和需要保持的数据等

### USER
**格式：** `USER daemon`
**注意：**指定运行容器时的用户名或 UID，后续的  RUN  也会使用指定用户。
当服务不需要管理员权限时，可以通过该命令指定运行用户。并且可以在之前创建所需要的用户，例如： `RUN groupadd -r postgres && useradd -r -g postgres postgres` 。要临时获取管理员权限可以使用  gosu ，而不推荐  sudo 

### WORKDIR
**格式：** `WORKDIR /path/to/workdir `
**注意：**为后续的  RUN 、 CMD 、 ENTRYPOINT  指令配置工作目录。如：

```
WORKDIR /a
WORKDIR b
WORKDIR c
RUN pwd
```

则最终路径为 `/a/b/c` 。

### ONBUILD
**格式：** `ONBUILD [INSTRUCTION] `
**注意：** 配置当所创建的镜像作为其它新创建镜像的基础镜像时，所执行的操作指令。
Dockerfile 使用如下的内容创建了镜像  image-A 。

```
[...]
ONBUILD ADD . /app/src
ONBUILD RUN /usr/local/bin/python-build --dir /app/src
[...]
```

基于 image-A 创建新的镜像时，新的Dockerfile中使用  FROM image-A 指定基础镜像时，会自动执行  ONBUILD  指令内容，等价于在后面添加了两条指令。

```
FROM image-A

#Automatically run the following
ADD . /app/src
RUN /usr/local/bin/python-build --dir /app/src

```

## 创建镜像
**格式：**`docker build [选项] 路径`
**注意：**
- 该命令将读取指定路径下的dockerfile，并将内容发送给docker服务端，由服务端来创建镜像。
- 可以通过  .dockerignore  文件（每一行添加一条匹配模式）来让 Docker 忽略路径下的目录和文件。
- 要指定镜像的标签信息，可以通过  -t  选项。如 `sudo docker build -t myrepo/myapp /tmp/test1/`

## 最佳实践
### 1、使用缓存
Dockerfile的每条指令都会将结果提交为新的镜像，下一个指令将会基于上一步指令的镜像的基础上构建，如果一个镜像存在相同的父镜像和指令（除了ADD），Docker将会使用镜像而不是执行该指令，即缓存。
为了有效地利用缓存，你需要保持你的Dockerfile一致，并且尽量在末尾修改。所以，我们应该使用常用且不变的Dockerfile开始指令来利用缓存。

**example：**

```
FROM ubuntu
MAINTAINER Michael Crosby <michael@crosbymichael.com>
RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
RUN apt-get update
......
```

### 2、使用标签
始终通过-t标记来构建镜像,一个简单的可读标签将帮助你管理每个创建的镜像。

`docker build -t="metaboy/dockertest" .`

### 3、公开端口
在Dockerfile中你有能力映射私有和公有端口，但是你永远不要通过Dockerfile映射公有端口。通过映射公有端口到主机上，你将只能运行一个容器化应用程序实例。

### 4、CMD与ENTRYPOINT的语法
使用CMD和ENTRYPOINT时，请务必使用数组语法.它执行的命令完全像你期望的那样.

### 5、CMD和ENTRYPOINT 结合使用更好
docker run命令中的参数都会传递给ENTRYPOINT指令，而不用担心它被覆盖（跟CMD不同）。当与CMD一起使用时ENTRYPOINT的表现会更好。让我们来研究一下我的Rethinkdb Dockerfile，看看如何使用它。


```
＃Dockerfile for Rethinkdb 
＃http://www.rethinkdb.com/

FROM ubuntu

MAINTAINER Michael Crosby <michael@crosbymichael.com>

RUN echo "deb http://archive.ubuntu.com/ubuntu precise main universe" > /etc/apt/sources.list
RUN apt-get update
RUN apt-get upgrade -y

RUN apt-get install -y python-software-properties
RUN add-apt-repository ppa:rethinkdb/ppa
RUN apt-get update
RUN apt-get install -y rethinkdb

＃Rethinkdb process
EXPOSE 28015
＃Rethinkdb admin console
EXPOSE 8080

＃Create the /rethinkdb_data dir structure
RUN /usr/bin/rethinkdb create

ENTRYPOINT ["/usr/bin/rethinkdb"]

CMD ["--help"]
```

当ENTRYPOINT指令出现时，我们知道每次运行该镜像，在docker run过程中传递的所有参数将成为ENTRYPOINT（/usr/bin/rethinkdb）的参数。

在Dockerfile中我还设置了一个默认CMD参数--help。这样做是为了docker run期间如果没有参数的传递，rethinkdb将会给用户显示默认的帮助文档。这是你所期望的与rethinkdb交互相同的功能。


### 6、不要开机初始化
容器模型是进程而不是机器.不需要开机初始化。

### 7、可信任构建

### 8、不在构建中升级版本
更新将发生在基础镜像里，你不需要在你的容器内来apt-get upgrade更新。因为在隔离情况下，如果更新时试图修改 init 或改变容器内的设备，更新可能会经常失败。它还可能会产生不一致的镜像。

### 9、使用小型基础镜像

### 10、使用特定的标签
Dockerfile 中FROM应始终包含依赖的基础镜像的完整仓库名和标签。比如说FROM debian:jessie而不仅仅是FROM debian。

### 11、常见指令组合
您的apt-get update应该与apt-get install组合。此外，你应该采取\的优势使用多行来进行安装。谨记层和缓存都是不错的。不要害怕过多的层，因为缓存是大救星。当然，你应当尽量使用上游的包。

### 12、使用自己的基础镜像


##  dockerfile语法检测和优化工具

之前有一个在线Dockerfile语法检测的网站，给出的建议都还不错。

> 
> - 要使用 tag，但不要 latest。
> - Debian 挺好的，不要总 Ubuntu。
> - apt-get update 在前，rm -rf /var/lib/apt/lists/* 在后。
> - yum install，不忘 yum clean。
> - 多 RUN 要合并，来减少层数。
> - 无用的软件，不要乱安装。
> - COPY 放最后，缓存很开心。
> - 善用 dockerignore，不浪费传输。
> - 不忘 MAINTAINER，这都是我的


## 备注
- Dockerfile最佳实践: [http://dockone.io/article/131](http://dockone.io/article/131)
- 推荐一个在线 Dockerfile 语法检查优化工具: [http://dockone.io/article/854](http://dockone.io/article/854)
- Docker —— 从入门到实践: [https://yeasy.gitbooks.io/docker_practice/content/](https://yeasy.gitbooks.io/docker_practice/content/)


