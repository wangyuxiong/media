# Docker基本知识储备（一）

## 1. Docker是什么
 Docker基于Go语言。Docker的基础是Linux容器（LXC）等技术。目标是实现请谅解的操作系统虚拟化解决方案。
 
 Docker与传统的虚拟化方式的不同之处，如下图。
![](/media/content/BlogPost/images/14625185081048.jpg)￼

![](/media/content/BlogPost/images/14625184884123.jpg)￼


对比虚拟机总结如下： 

特性 | 容器  | 虚拟机
------- | -------   | ---- 
启动 | 秒级       | 分钟级  
硬盘使用 | 一般为 MB       | 一般为 GB
性能 | 接近原生        |弱于
系统支持量 | 单机支持上千个容器       | 一般几十个


### Docker带来的好处是：
- 更快速的交付和部署
- 更高效的虚拟化
- 更轻松的迁移和扩展
- 更简单的管理

## 基本概念
- 镜像（Image）
- 容器 （Container）
- 仓库 （Repository）


## 镜像

### 获取镜像
- 使用 docker pull获取所需的镜像

**example**: 从Docker hub 仓库下载一个Ubuntu 12.04操作系统的镜像

`sudo docker pull ubuntu:12.04` == `sudo docker pull registry.hub.docker.com/ubuntu:12.04`

也可指定其他仓库进行下载。

- 使用docker run 使用下载的镜像

**example：** 如创建一个容器，让其中运行bash应用。
`sudo docker run -t -i ubuntu:12.04 /bin/bash`

### 列出镜像
使用docker images显示本地已有的镜像

**列出的信息，可以见到如下几个字段：**

- REPOSITORY  ：来自于哪个仓库，比如 ubuntu
- TAG ：镜像的标记，比如 14.04
- IMAGE ID   ：它的  ID  号（唯一）   
- CREATED   ： 创建时间   
- VIRTUAL SIZE ： 镜像大小

其中镜像的ID唯一标识了镜像，类似于ubuntu:14.04和ubuntu:trusty，可以具有相同的镜像  ID ，说明它们实际上是同一镜像。

TAG信息用来标记来自同一个仓库的不同镜像。使用镜像的时候指定tag，如果不指定则使用默认标记latest。

用docker tag命令来修改镜像的标签: `sudo docker tag 5db5f8471261 ouruser/sinatra:devel`

### 创建镜像
#### 修改已有镜像
 1. 先使用系在的镜像启动容器： `sudo docker run -t -i training/sinatra /bin/bash`
 2. 在容器中添加json package： `root@0b2616b0e5a8:/# gem install json`
 3. 退出后，docker commit命令提交更新后的副本：`sudo docker commit -m "Added json gem" -a "Docker Newbee" 0b2616b0e5a8 ouruser/sinatra:v2`
 4. 查看本地镜像：`sudo docker images` 。 可以检查该镜像是否存在TAG=v2的标记 
 5. 使用新的镜像来启动容器： `sudo docker run -t -i ouruser/sinatra:v2 /bin/bash`

#### 使用dockerfile来创建镜像

 1. 新建一个目录和dockerfile文件：`mkdir sinatra&&cd sinatra&&touch Dockerfile`
 2. Dockerfile 中每一条指令都创建镜像的一层.
 3. 使用docker build来生成镜像：`sudo docker build -t="ouruser/sinatra:v2" .`
 
>     -t  标记来添加 tag
>     “.” 是 Dockerfile 所在的路径（当前目录），也可以替换为一个具体的 Dockerfile 的路径。
 
 
*docker file 文件实例：*
 
```# This is a comment
FROM ubuntu:14.04
MAINTAINER Docker Newbee <newbee@docker.com>
RUN apt-get -qq update
RUN apt-get -qqy install ruby ruby-dev
RUN gem install sinatra
```

> - 使用 # 来注释
> - FROM  指令告诉 Docker 使用哪个镜像作为基础
> - 接着是维护者的信息
> - RUN 开头的指令会在创建中运行，比如安装一个软件包，在这里使用 apt-get 来安装了一些软件

build 过程日志：


```
$ sudo docker build -t="ouruser/sinatra:v2" .
Uploading context  2.56 kB
Uploading context
Step 0 : FROM ubuntu:14.04
 ---> 99ec81b80c55
Step 1 : MAINTAINER Newbee <newbee@docker.com>
 ---> Running in 7c5664a8a0c1
 ---> 2fa8ca4e2a13
Removing intermediate container 7c5664a8a0c1
Step 2 : RUN apt-get -qq update
 ---> Running in b07cc3fb4256
 ---> 50d21070ec0c
Removing intermediate container b07cc3fb4256
Step 3 : RUN apt-get -qqy install ruby ruby-dev
 ---> Running in a5b038dd127e
Selecting previously unselected package libasan0:amd64.
(Reading database ... 11518 files and directories currently installed.)
Preparing to unpack .../libasan0_4.8.2-19ubuntu1_amd64.deb ...
Setting up ruby (1:1.9.3.4) ...
Setting up ruby1.9.1 (1.9.3.484-2ubuntu1) ...
Processing triggers for libc-bin (2.19-0ubuntu6) ...
 ---> 2acb20f17878
Removing intermediate container a5b038dd127e
Step 4 : RUN gem install sinatra
 ---> Running in 5e9d0065c1f7
. . .
Successfully installed rack-protection-1.5.3
Successfully installed sinatra-1.4.5
4 gems installed
 ---> 324104cde6ad
Removing intermediate container 5e9d0065c1f7
Successfully built 324104cde6ad
```

*通过日志可以得出build过程如下：*

> 可以看到 build 进程在执行操作。它要做的第一件事情就是上传这个 Dockerfile 内容，因为所有的操作都要依据 Dockerfile 来进行。 然后，Dockfile 中的指令被一条一条的执行。每一步都创建了一个新的容器，在容器中执行指令并提交修改（就跟之前介绍过的  docker commit  一样）。当所有的指令都执行完毕之后，返回了最终的镜像 id。所有的中间步骤所产生的容器都被删除和清理了。
 
*常用命令*：
ADD命令负责本地文件到镜像： `ADD myApp /var/www`
EXPOSE命令向外开放端口： `EXPOSE 80`
CMD描述容器启动后运行的程序：`CMD ["/usr/sbin/apachectl", "-D", "FOREGROUND"]`

#### 从本地文件系统导入

要从本地文件系统导入一个镜像，可以使用 openvz（容器虚拟化的先锋技术）的模板来创建： openvz 的模板下载地址为 [templates](http://openvz.org/Download/templates/precreated) 。

比如，先下载了一个 ubuntu-14.04 的镜像，之后使用以下命令导入：
`sudo cat ubuntu-14.04-x86_64-minimal.tar.gz  |docker import - ubuntu:14.04`

然后查看新导入的镜像。

#### 上传镜像
用户在docker hub或私有的hub完成注册后，可以使用 docker push命令推送到仓库来共享。
`sudo docker push ouruser/sinatra`


### 导出和载入镜像
- 导出镜像：使用docker save，导出镜像到本地文件。`sudo docker save -o ubuntu_14.04.tar ubuntu:14.04`
- 载入镜像：使用docker load，从导出的本地文件中到再导入到本地镜像库。
`sudo docker load --input ubuntu_14.04.tar` or `sudo docker load < ubuntu_14.04.tar`

### 移除本地镜像
使用  `docker rmi`  命令。注意 `docker rm`  命令是移除容器。

**注意：在删除镜像之前要先用  docker rm  删掉依赖于这个镜像的所有容器。**

- 清理所有未打过标签的本地镜像：
`sudo docker rmi $(docker images -q -f "dangling=true")` or `sudo docker rmi $(docker images --quiet --filter "dangling=true")`


### 镜像的实现原理

Docker 镜像是怎么实现增量的修改和维护的？ 每个镜像都由很多层次构成，Docker 使用 [Union FS](https://en.wikipedia.org/wiki/UnionFS) 将这些不同的层结合到一个镜像中去。
通常 Union FS 有两个用途：
- 一方面可以实现不借助 LVM、RAID 将多个 disk 挂到同一个目录下。
- 另一个更常用的就是将一个只读的分支和一个可写的分支联合在一起。

Live CD 正是基于此方法可以允许在镜像不变的基础上允许用户在其上进行一些写操作。 Docker 在 AUFS 上构建的容器也是利用了类似的原理。


## 容器
简单的说，容器是独立运行的一个或一组应用，以及它们的运行态环境。

### 启动容器
#### 新建并启动
需要的命令是：docker run。例如：
`sudo docker run -t -i ubuntu:14.04 /bin/bash`

- -t  选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上。 
- -i  则让容器的标准输入保持打开

利用  docker run  来创建容器时，Docker 在后台运行的标准操作包括：
 
- 检查本地是否存在指定的镜像，不存在就从公有仓库下载
- 利用镜像创建并启动一个容器
- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
- 从地址池配置一个 ip 地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被终止

#### 启动已终止容器
利用  docker start  命令，直接将一个已经终止的容器启动运行

### 守护态运行
更多的时候，需要让 Docker在后台运行而不是直接把执行命令的结果输出在当前宿主机下。此时，可以通过添加  -d  参数来实现。
`sudo docker run -d ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"`

如果想获取容器的输出信息，可以通过 `docker logs [container ID or NAMES]`

### 终止容器
- 使用  docker stop  来终止一个运行中的容器
- docker start  命令来重新启动
- docker restart  命令会将一个运行态的容器终止，然后再重新启动它

### 进入容器
某些时候需要进入容器进行操作，有很多种方法，包括使用  docker attach  命令或  nsenter  工具等。

#### attach命令
命令：`sudo docker run -idt ubuntu`

*存在问题：* 当多个窗口同时 attach 到同一个容器的时候，所有窗口都会同步显示。当某个窗口因命令阻塞时,其他窗口也无法执行操作了。

#### nsernter命令
- 安装：nsenter  工具在 util-linux 包2.23版本后包含。 如果系统中没有该包，需要安装一个。
  
```
$ cd /tmp
$ curl https://www.kernel.org/pub/linux/utils/util-linux/v2.24/util-linux-2.24.tar.gz | tar -zxf- 
$ cd util-linux-2.24
$ ./configure --without-ncurses
$ make nsenter && sudo cp nsenter /usr/local/bin
```

- 使用：
  1. 需要获取到容器的第一个进程PID。`PID=$(docker inspect --format "{{ .State.Pid }}" <container>)`
  2. 连接到容器：`nsenter --target $PID --mount --uts --ipc --net --pid`

  
```
$ sudo docker run -idt ubuntu
243c32535da7d142fb0e6df616a3c3ada0b8ab417937c853a9e1c251f499f550
$ sudo docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
243c32535da7        ubuntu:latest       "/bin/bash"         18 seconds ago      Up 17 seconds                           nostalgic_hypatia
$ PID=$(docker-pid 243c32535da7)
10981
$ sudo nsenter --target 10981 --mount --uts --ipc --net --pid
root@243c32535da7:/#
```

当然还有一些更简单的手段可以优化这个过程。如下载 [.bashrc_docker](https://github.com/yeasy/docker_practice/raw/master/_local/.bashrc_docker)

### 导出和导入容器
#### 导出容器：
导出容器快照到本地文件：
 `sudo docker export 7691a814370e > ubuntu.tar`
 
#### 导入容器快照： 
  - 指定url或某个目录导入：`$sudo docker import http://example.com/exampleimage.tgz example/imagerepo`
  
  - 从容器快照文件中再导入为镜像：`cat ubuntu.tar | sudo docker import - test/ubuntu:v1.0`

 *注：用户既可以使用  docker load  来导入镜像存储文件到本地镜像库，也可以使用  docker import  来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。*
 
 
### 删除容器
可以使用  docker rm  来删除一个处于终止状态的容器。`$sudo docker rm  trusting_newton `

如果是运行中的容器，需要添加 -f 参数

#### 清理所有处于终止状态的容器
使用命令： `docker rm $(docker ps -a -q)`  可以全部清理掉。

## 仓库
仓库（Repository）是集中存放镜像的地方。注册服务器（Registry）实际上是管理仓库的具体服务器，每个服务器上可以有多个仓库，而每个仓库下面有多个镜像
如：对于仓库地址  dl.dockerpool.com/ubuntu  来说， dl.dockerpool.com  是注册服务器地址， ubuntu  是仓库名。

### Docker Hub
Docker 官方维护了一个公共仓库 Docker Hub，其中已经包括了超过 15,000 的镜像。大部分需求，都可以通过在 Docker Hub 中直接下载镜像来实现。

#### 登录
通过执行  docker login  命令来输入用户名、密码和邮箱来完成注册和登录。 注册成功后，本地用户目录的  .dockercfg  中将保存用户的认证信息。

#### 基本操作
- sudo docker search centos
- sudo docker pull centos

### 私有仓库
docker-registry  是官方提供的工具，可以用于构建私有的镜像仓库。

#### 安装运行docker-registry
##### 容器运行

- 安装docker后，使用官方registry来运行：`$ sudo docker run -d -p 5000:5000 registry`
 
- 用户可以通过指定参数来配置私有仓库位置，例如配置镜像存储到 Amazon S3 服务。

```
$ sudo docker run \
         -e SETTINGS_FLAVOR=s3 \
         -e AWS_BUCKET=acme-docker \
         -e STORAGE_PATH=/registry \
         -e AWS_KEY=AKIAHSHB43HS3J92MXZ \
         -e AWS_SECRET=xdDowwlK7TJajV1Y7EoOZrmuPEJlHYcNP2k4j49T \
         -e SEARCH_BACKEND=sqlalchemy \
         -p 5000:5000 \
         registry
```

- 还可以指定本地路径（如  /home/user/registry-conf  ）下的配置文件。
`$ sudo docker run -d -p 5000:5000 -v /home/user/registry-conf:/registry-conf -e DOCKER_REGISTRY_CONFIG=/registry-conf/config.yml registry`

默认情况下，仓库会被创建在容器的 /tmp/registry 下。可以通过 -v 参数来将镜像文件存放在本地的指定路径。 例如下面的例子将上传的镜像放到/opt/data/registry目录。


##### 本地安装

- 源安装：

```
$ sudo apt-get install -y build-essential python-dev libevent-dev python-pip liblzma-dev
$ sudo pip install docker-registry
```

- 从 docker-registry 项目下载源码进行安装。

```
$ sudo apt-get install build-essential python-dev libevent-dev python-pip libssl-dev liblzma-dev libffi-dev
$ git clone https://github.com/docker/docker-registry.git
$ cd docker-registry
$ sudo python setup.py install
```

- 修改配置文件，主要修改 dev 模板段的  storage_path  到本地的存储仓库的路径。
`$ cp config/config_sample.yml config/config.yml`

- 启动 Web 服务。
`$ sudo gunicorn -c contrib/gunicorn.py docker_registry.wsgi:application`
或者
`$ sudo gunicorn --access-logfile - --error-logfile - -k gevent -b 0.0.0.0:5000 -w 4 --max-requests 100 docker_registry.wsgi:application`

- 使用 curl 访问本地的 5000 端口，看到输出 docker-registry 的版本信息说明运行成功。

#### 在私有仓库上传、下载、搜索镜像
创建好私有仓库之后，就可以使用  docker tag  来标记一个镜像，然后推送它到仓库，别的机器上就可以下载下来了。

- 使用 docker tag  将  ba58  这个镜像标记为  192.168.7.26:5000/test

```
$ sudo docker tag ba58 192.168.7.26:5000/test
root ~ # docker images
REPOSITORY                        TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
ubuntu                            14.04               ba5877dc9bec        6 weeks ago         192.7 MB
ubuntu                            latest              ba5877dc9bec        6 weeks ago         192.7 MB
192.168.7.26:5000/test            latest              ba5877dc9bec        6 weeks ago         192.7 MB
```

- 使用  docker push  上传标记的镜像。

```
$ sudo docker push 192.168.7.26:5000/test
The push refers to a repository [192.168.7.26:5000/test] (len: 1)
Sending image list
Pushing repository 192.168.7.26:5000/test (1 tags)
Image 511136ea3c5a already pushed, skipping
Image 9bad880da3d2 already pushed, skipping
Image 25f11f5fb0cb already pushed, skipping
Image ebc34468f71d already pushed, skipping
Image 2318d26665ef already pushed, skipping
Image ba5877dc9bec already pushed, skipping
Pushing tag for rev [ba5877dc9bec] on {http://192.168.7.26:5000/v1/repositories/test/tags/latest}
```

- 用 curl 查看仓库中的镜像
`$ curl http://192.168.7.26:5000/v1/search`

- 可以到另外一台机器去下载这个镜像
`sudo docker pull 192.168.7.26:5000/test`


### 仓库配置文件
Docker 的 Registry 利用配置文件提供了一些仓库的模板（flavor），用户可以直接使用它们来进行开发或生产部署。

在  config_sample.yml  文件中，可以看到一些现成的模板段：
 
 - common ：基础配置
 - local ：存储数据到本地文件系统
 - s3 ：存储数据到 AWS S3 中
 - dev ：使用  local  模板的基本配置
 - test ：单元测试使用
 - prod ：生产环境配置（基本上跟s3配置类似）
 - gcs ：存储数据到 Google 的云存储
 - swift ：存储数据到 OpenStack Swift 服务
 - glance ：存储数据到 OpenStack Glance 服务，本地文件系统为后备
 - glance-swift ：存储数据到 OpenStack Glance 服务，Swift 为后备
 - elliptics ：存储数据到 Elliptics key/value 存储

用户也可以添加自定义的模版段。默认情况下使用的模板是dev，要使用某个模板作为默认值，可以添加 SETTINGS_FLAVOR 到环境变量中，例如`export SETTINGS_FLAVOR=dev`
另外，配置文件中支持从环境变量中加载值，语法格式为  `_env:VARIABLENAME[:DEFAULT]`
