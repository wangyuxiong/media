# Docker-Mac平台开发环境搭建

### 一.在OSX环境下安装Docker
Mac用户可以通过Docker Toolbox安装docker，它包含了下面一些组件和工具:
- Docker CLI client
- Docker Machine
- Docker Compose
- Docker GUI.Kitematic
- Docker QuickStart
- Oracle VM VirtualBox 

由于Docker Engine是使用linux特有的内核特性，Mac不能直接在OSX上直接运行Docker。需要使用docker-machine 来创建一台虚拟机，该虚拟机上安装linux系统来运行Docker。

#### step 1. Install Docker Toolbox
1. 官网下载 [Docker Toolbox](https://www.docker.com/products/docker-toolbox)
2. 下一步下一步搞定安装。

#### step 2. 验证安装
- 启动控制台，打开如下app。
![](/media/content/BlogPost/images/14637317100743.jpg)

- 查看输出，如下：
	
```
➜  ~ bash --login '/Applications/Docker/Docker Quickstart Terminal.app/Contents/Resources/Scripts/start.sh'
Creating CA: /Users/metaboy/.docker/machine/certs/ca.pem
Creating client certificate: /Users/metaboy/.docker/machine/certs/cert.pem
Running pre-create checks...
Creating machine...
(default) Copying /Users/metaboy/.docker/machine/cache/boot2docker.iso to /Users/metaboy/.docker/machine/machines/default/boot2docker.iso...
(default) Creating VirtualBox VM...
(default) Creating SSH key...
^@
(default) Starting the VM...
(default) Check network to re-create if needed...
(default) Found a new host-only adapter: "vboxnet0"
(default) Waiting for an IP...
Waiting for machine to be running, this may take a few minutes...
Detecting operating system of created instance...
Waiting for SSH to be available...
Detecting the provisioner...
Provisioning with boot2docker...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
Checking connection to Docker...
Docker is up and running!
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: /usr/local/bin/docker-machine env default


                        ##         .
                  ## ## ##        ==
               ## ## ## ## ##    ===
           /"""""""""""""""""\___/ ===
      ~~~ {~~ ~~~~ ~~~ ~~~~ ~~~ ~ /  ===- ~~~
           \______ o           __/
             \    \         __/
              \____\_______/


docker is configured to use the default machine with IP 192.168.99.100
For help getting started, check out the docs at https://docs.docker.com
```

- 使用docker run hello-world命令


### 二. 搞清楚images和containers
回到第一小节中的执行命令: `docker run hello-world` .简单分析命令的组成： 

- run ：告诉docker engine，运行一个container。
- hello-world： 一个镜像名称，如果本地没有，默认会从docker hub下载到本地。然后装载到container中。




### 三. 正式开始
以whalesay镜像为例子说明。

1. 去[docker hub](https://hub.docker.com/)网站上找到官方的whalesay镜像。
2. 查询到 [docker/whalesay](https://hub.docker.com/r/docker/whalesay/) 这个镜像，打开，如下图所示，会给出如何使用的说明：
   ![](/media/content/BlogPost/images/14637369611202.jpg)
3. 运行该镜像：`docker run docker/whalesay cowsay boo`  

```
➜  ~ docker run docker/whalesay cowsay boo
Unable to find image 'docker/whalesay:latest' locally
latest: Pulling from docker/whalesay

e190868d63f8: Pull complete
909cd34c6fd7: Pull complete
0b9bfabab7c1: Pull complete
a3ed95caeb02: Pull complete
00bf65475aba: Pull complete
c57b6bcc83e3: Pull complete
8978f6879e2f: Pull complete
8eed3712d2cf: Pull complete
Digest: sha256:178598e51a26abbc958b8a2e48825c90bc22e641de3d31e18aaf55f3258ba93b
Status: Downloaded newer image for docker/whalesay:latest
 _____
< boo >
 -----
    \
     \
      \
                    ##        .
              ## ## ##       ==
           ## ## ## ##      ===
       /""""""""""""""""___/ ===
  ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
       \______ o          __/
        \    \        __/
          \____\______/
➜  ~
```
4. 运行 `docker images `, 查看当前本机的镜像

```
➜  ~ docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              14.04               90d5884b1ee0        2 weeks ago         188 MB
hello-world         latest              94df4f0ce8a4        3 weeks ago         967 B
docker/whalesay     latest              6b362a9f73eb        12 months ago       247 MB
```
   
   
### 四. 构建自己的image
对于上面下载到本地的镜像whalesay，可能并不能满足我们的需求。我们如果想在这个基础上增加点内容，可以构建一个新的版本。

#### step 1. 准备一个Dockerfile文件

```
➜  mkdir mydockerbuild
➜  cd mydockerbuild
➜  mydockerbuild touch Dockerfile
➜  mydockerbuild vim Dockerfile
```

输入内容如下：

```
FROM docker/whalesay:latest

RUN apt-get -y update && apt-get install -y fortunes

CMD /usr/games/fortunes -a | cowsay
```

#### step 2. build image
- 运行命令： `docker build -t docker-whale .`


```
➜  mydockerbuild docker build -t docker-whale .
Sending build context to Docker daemon 2.048 kB
Step 1 : FROM docker/whalesay:latest
 ---> 6b362a9f73eb
Step 2 : RUN apt-get -y update && apt-get install -y fortunes
 ---> Running in f2875e6b6f60
Ign http://archive.ubuntu.com trusty InRelease
Get:1 http://archive.ubuntu.com trusty-updates InRelease [65.9 kB]
Get:2 http://archive.ubuntu.com trusty-security InRelease [65.9 kB]
Hit http://archive.ubuntu.com trusty Release.gpg
......
......
Setting up fortunes-min (1:1.99.1-7) ...
Setting up fortunes (1:1.99.1-7) ...
Processing triggers for libc-bin (2.19-0ubuntu6.6) ...
 ---> a36f33cf5d93
Removing intermediate container f2875e6b6f60
Step 3 : CMD /usr/games/fortunes -a | cowsay
 ---> Running in 83a641a19c96
 ---> aa3bd0e8cf17
Removing intermediate container 83a641a19c96
Successfully built aa3bd0e8cf17
```

- 运行命令：`docker images`
  
```
docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker-whale        latest              aa3bd0e8cf17        6 minutes ago       274.5 MB
ubuntu              14.04               90d5884b1ee0        2 weeks ago         188 MB
hello-world         latest              94df4f0ce8a4        3 weeks ago         967 B
docker/whalesay     latest              6b362a9f73eb        12 months ago       247 MB
```

- 运行这个images： `docker run docker-whale`

### 五. 在Docker hub上创建自己的仓库
在Docker Hub上注册一个账号，并创建一个仓库：
![](/media/content/BlogPost/images/14637689877412.jpg)


### 六. 上传自己的image
- 通过镜像ID打tag: `docker tag aa3bd0e8cf17 metaboy/docker-whale:latest`

```
➜  mydockerbuild docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker-whale        latest              aa3bd0e8cf17        6 minutes ago       274.5 MB
ubuntu              14.04               90d5884b1ee0        2 weeks ago         188 MB
hello-world         latest              94df4f0ce8a4        3 weeks ago         967 B
docker/whalesay     latest              6b362a9f73eb        12 months ago       247 MB
➜  mydockerbuild docker tag aa3bd0e8cf17 metaboy/docker-whale:latest
➜  mydockerbuild docker
 images
REPOSITORY             TAG                 IMAGE ID            CREATED             SIZE
docker-whale           latest              aa3bd0e8cf17        15 minutes ago      274.5 MB
metaboy/docker-whale   latest              aa3bd0e8cf17        15 minutes ago      274.5 MB
ubuntu                 14.04               90d5884b1ee0        2 weeks ago         188 MB
hello-world            latest              94df4f0ce8a4        3 weeks ago         967 B
docker/whalesay        latest              6b362a9f73eb        12 months ago       247 MB
➜  mydockerbuild
```

- 使用command line登录到Docker hub：

```
➜  mydockerbuild docker login --username=metaboy --email=yxiong.wang@gmail.com
Warning: '--email' is deprecated, it will be removed soon. See usage.
Password:
Login Succeeded
```

- 将镜像push到Docker hub仓库： docker push

国内非常慢，我等不了最终的结果出来。一般使用这个命令就可以：

```
➜  mydockerbuild docker push metaboy/docker-whale
The push refers to a repository [docker.io/metaboy/docker-whale]
a7bc4a1132b8: Pushing [=========>                                         ] 5.263 MB/27.5 MB
......
......
```

- 在用新镜像之前，我们先将whale相关的镜像删除：（名称和镜像id都可以满足）

```
➜  docker rmi -f aa3bd0e8cf17
➜  docker rmi -f docker-whale
```

- pull 你的新镜像：`docker run metaboy/docker-whale`



