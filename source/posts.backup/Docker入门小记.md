---
title: Docker 入门小记
date: 2018-03-05 15:56:20
categories: 技术相关
tags: [docker]
toc: true
---
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker.png" width="45%" height="45%">

原本是准备寒假在家学习 Docker 的，但很无奈的是，回到家的我完全不知道如何写下新年的第一段代码。<br/>说来也巧，回到学校后，由于某些不可抗力，搁置已久的消防项目又得开始了，原来负责服务器端 Docker 部署的某大神学长外出实习，重新部署 Docker 服务的任务就落在了我的头上(无奈.jpg)。<br/>我原本是不准备记录下这篇文章的，但是当我写完了 Dockerfile，向服务器部署的时候发现，这次部署可能需要 —— 三个小时？一个人在图书馆坐着，像看剧一样地看着命令终端不断输出完成进度，实在无聊，就想着把最近两天学到的知识记录一下吧。
<!-- more -->

### Docker 是什么？

Docker 是 DotCloud 公司开发的基于 LXC(Linux Container，一种内核虚拟化技术)的高级容器引擎，go 语言编写，使用 Apache 2.0 开源协议，源代码托管在 Github 上。

### Docker 原理

我们可以发现近几年云计算非常流行，那么云计算是什么东西呢？
>云计算是指通过互联网提供动态易扩展且经常是虚拟化的资源，被划分为IaaS(Infrastructure as a Service)、PaaS(Platform as a Service)、SaaS(Software as a Service)。

但其中 PaaS 既不如 IaaS 那样灵活而自由，也不如 SaaS 那样可以直接推向消费者。有人说 PaaS 是未来的云计算，但是近几年 IaaS 和 SaaS 各自发展，反而是 PaaS 几乎裹足不前，虽然各种应用引擎层出不穷，但是没有什么人专门为 PaaS 开发应用。

这个时候 Docker 的出现完美地解决了这个问题，Docker 很好地实现了 PaaS，如果把云计算比作货轮的话，Docker 就是放在货轮上的集装箱了。

最开始我以为 Docker 和虚拟机差不多，因为都实现了虚拟化，而且都是可移植的。但是之后在各大技术交流平台上探索，我发现，这两个技术在本质上是有区别的，虚拟机直接面向了硬件层，而 Docker 则是面向操作系统层，它的出现和发展得益于 Linux 系统的 Namespace 机制和 CGroup 机制，这两个机制保证了 Docker 容器和外部环境的隔离，保障了 PaaS 的安全性。

- Namespace 机制

Namespace（命名空间）机制是 Linux 为我们提供的用于分离进程树、网络接口、挂载点以及进程间通信等资源的方法。在日常使用 Linux 或者 macOS 时，我们并没有运行多个完全分离的服务器的需要，但是如果我们在服务器上启动了多个服务，这些服务其实会相互影响的，每一个服务都能看到其他服务的进程，也可以访问宿主机器上的任意文件，这是很多时候我们都不愿意看到的，我们更希望运行在同一台机器上的不同服务能做到完全隔离，就像运行在多台不同的机器上一样。Docker 便是通过 Linux 的 Namespaces 对不同的容器实现了这样的隔离。

- CGroup 机制

我们通过 Linux 的命名空间为新创建的进程隔离了文件系统、网络并与宿主机器之间的进程相互隔离，但是命名空间并不能够为我们提供物理资源上的隔离，比如 CPU 或者内存，如果在同一台机器上运行了多个对彼此以及宿主机器一无所知的『容器』，这些容器却共同占用了宿主机器的物理资源。

如果其中的某一个容器正在执行 CPU 密集型的任务，那么就会影响其他容器中任务的性能与执行效率，导致多个容器相互影响并且抢占资源。如何对多个容器的资源使用进行限制就成了解决进程虚拟资源隔离之后的主要问题，而 Control Groups(简称 CGroups)就是能够隔离宿主机器上的物理资源，例如 CPU、内存、磁盘 I/O 和网络带宽。

> 在 CGroup 中，所有的任务就是一个系统的一个进程，而 CGroup 就是一组按照某种标准划分的进程，在 CGroup 这种机制中，所有的资源控制都是以 CGroup 作为单位实现的，每一个进程都可以随时加入一个 CGroup 也可以随时退出一个 CGroup。<p align="right">—— <a href="https://www.ibm.com/developerworks/cn/linux/1506_cgroup/index.html">CGroup 介绍、应用实例及原理描述</a></p>

### 安装 Docker

#### 设置存储库

关于不同操作系统 Docker 的安装方法，[Docker 的官方文档](https://docs.docker.com/)写得很详细，因为 Docker 官方推荐使用 Ubuntu 部署 Docker(Ubuntu 系统本身就实现了Docker容器运行需要的 AUFS 机制)，所以这里只记录一下 Ubuntu 16.04 系统下 Docker CE 的安装。

- 更新apt包索引
    ```bash
    $ sudo apt-get update
    ```
- 安装软件包，用于允许 apt 通过 HTTPS 访问 Docker 安装源
    ```bash
    $ sudo apt-get install \
        apt-transport-https \
        ca-certificates \
        curl \
        software-properties-common
    ```
- 添加 Docker 的官方 GPG 密钥：
    ```bash
    $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
    ```
    确认一下您已经获取到了指纹：
    ```bash
    $ sudo apt-key fingerprint 0EBFCD88

    pub   4096R/0EBFCD88 2017-02-22
          Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
    uid                  Docker Release (CE deb) <docker@docker.com>
    sub   4096R/F273FCD8 2017-02-22
    ```
- 添加储存库：
    ```bash
    $ sudo add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"
    ```

#### 安装 Docker CE

- 更新 apt 包索引
    ```bash
    $ sudo apt-get update
    ```
- 安装Docker CE
    ```bash
    $ sudo apt-get install docker-ce
    ```

### Docker 命令记录

因为 Docker 的运行绑定了`/var/run/docker.sock`文件，用于 docker 的守护进程(Docker daemon)和容器进程之间通信，而该文件的访问权限为 660，必须使用 sudo 命令才有该文件的执行权限。

#### 容器的生命周期管理

- run：Run a command in a new container
- start：Start one or more stopped containers
- stop：Stop one or more running containers
- restart：Restart one or more containers
- kill：Kill one or more running containers
- rm：Remove one or more containers
- pause：Pause all processes within one or more containers
- unpause：Unpause all processes within one or more containers
- create：Create a new container
- exec：Run a command in a running container

#### 容器操作

- ps：List containers
- inspect：Return low-level information on Docker objects
- top：Display the running processes of a container
- attach：Attach local standard input, output, and error streams to a running container
- events：Get real time events from the server
- logs：Fetch the logs of a container
- wait：Block until one or more containers stop, then print their exit codes
- export：Export a container's filesystem as a tar archive
- port：List port mappings or a specific mapping for the container

#### 容器 rootfs 命令

- commit：Create a new image from a container's changes
- cp：Copy files/folders between a container and the local filesystem
- diff：Inspect changes to files or directories on a container's filesystem

#### 镜像仓库操作

- login：Log in to a Docker registry
- logout：Log out from a Docker registry
- pull：Pull an image or a repository from a registry
- push：Push an image or a repository to a registry
- search：Search the Docker Hub for images

#### 本地镜像管理

- images：List images
- rmi：Remove one or more images
- tag：Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
- build：Build an image from a Dockerfile
- history：Show the history of an image
- save：Save one or more images to a tar archive (streamed to STDOUT by default)
- import：Import the contents from a tarball to create a filesystem image

#### Docker 信息

- info：Display system-wide information
- version：Show the Docker version information

### Dockerfile、Daemon 进程、镜像、容器之间的关系

前两天看了本书《Docker 全攻略》，里面提到 Docker 的出现统一了三界（开发、测试、生产）。Dockerfile 面向开发，Docker 镜像可以直接交付给我们的甲方，Docker容器面向部署和运维。Daemon 进程是 Docker 运行的主进程，负责和容器间的通信。

我们使用 Docker，最后会得到一个运行状态的容器，简单来说，使用Docker大概的步骤是：编写 Dockerfile 生成镜像，启动镜像，然后可以得到容器。

也就是说，Dockerfile 是生成镜像和容器的原材料，它会被 Docker 解释器解释，生成指定的镜像，镜像是静态的文件，启动镜像便能得到运行的容器，镜像和容器之间的关系可以理解为一个已经关机的电脑和一个开机状态的电脑的关系，顺便我们可以把 Dockerfile 理解为给电脑装系统时的用到的 ISO 镜像。

接下来，本文会重点介绍一下镜像和容器之间的关系，参考自我前两天在网上一篇文章，文章的链接已附在文末。

#### Docker 容器和镜像的关系

容器和镜像的大概关系可以参考下图：

<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/container.png" width="85%" height="85%">
如图，镜像是分层的文件，每一层都是只读的，其分层的方式类似于 git 的版本控制方式，除了最底层，每一层都会有它的父层，每一层都记录了和上一层的差异(diff)，而容器则是在顶层运行的一个可读写层。

现在，我们可以通过特定的命令来了解一下它们之间的关系。

- `sudo docker create <image-id>`
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/create.jpg" width="85%" height="85%">
`docker create`命令为指定的镜像(image)添加了一个可读写层，构成了一个新的容器，但这个容器并没有运行。
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/create1.png" width="85%" height="85%">

- `sudo docker start <container-id>`
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/start.jpg" width="85%" height="85%">
`Docker start`命令为容器文件系统创建了一个进程隔离空间。注意，每一个容器只能够有一个进程隔离空间。

- `sudo docker run <image-id>`
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/run.jpg" width="85%" height="85%">
我们可以看到`docker start`命令和`docker run`命令都生成了容器，它们有什么区别呢？
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/start1.png" width="85%" height="85%">
从图片可以看出，`docker run`命令先是利用镜像创建了一个容器，然后运行这个容器。

- `sudo docker ps`
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/ps.jpg" width="85%" height="85%">
`docker ps`命令会列出所有运行中的容器。这隐藏了非运行态容器的存在，如果想要找出这些容器，我们需要使用下面这个命令。

- `sudo docker ps –a`
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/psa.jpg" width="85%" height="85%">
`docker ps –a`命令会列出所有的容器，不管是运行的，还是停止的。

- `sudo docker images`
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/images.jpg" width="85%" height="85%">
`docker images`命令会列出了所有顶层(top-level)镜像。

- `sudo docker images –a`
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/imagesa.jpg" width="85%" height="85%">
`docker images –a`命令列出了所有的镜像，也可以说是列出了所有的可读层。如果你想要查看某一个 image-id 下的所有层，可以使用`docker history`来查看。

- `sudo docker stop <container-id>`
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/stop.jpg" width="85%" height="85%">
`docker stop`命令会向运行中的容器发送一个 SIGTERM 的信号，然后停止所有的进程。

- `sudo docker kill <container-id>`
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/kill.jpg" width="85%" height="85%">
`docker kill`命令向所有运行在容器中的进程发送了一个不友好的 SIGKILL 信号。

- `sudo docker pause <container-id>`
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/pause.jpg" width="85%" height="85%">
`docker stop`和`docker kill`命令会发送 UNIX 的信号给运行中的进程，`docker pause`命令则不一样，它利用了 cgroups 的特性将运行中的进程空间暂停。但是这种方式的不足之处在于发送一个 SIGTSTP 信号对于进程来说不够简单易懂，以至于不能够让所有进程暂停。

- `sudo docker rm <container-id>`
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/rm.jpg" width="85%" height="85%">
`docker rm`命令会移除构成容器的可读写层。注意，这个命令只能对非运行态容器执行。

- `sudo docker rmi <image-id>`
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/rmi.jpg" width="85%" height="85%">
`docker rmi`命令会移除构成镜像的一个只读层。你只能够使用`docker rmi`来移除最顶层(top level layer)(也可以说是镜像)，你也可以使用 -f 参数来强制删除中间的只读层。

- `sudo docker commit <container-id>`
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/commit.jpg" width="85%" height="85%">
`docker commit`命令将容器的可读写层转换为一个只读层，这样就把一个容器转换成了不可变的镜像。
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/commit1.png" width="85%" height="85%">

- `sudo docker build`
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/build.jpg" width="85%" height="85%">
`docker build`命令它会反复的执行多个命令，如下图
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/build1.png" width="85%" height="85%">
`docker build`命令根据 Dockerfile 文件中的 FROM 指令获取到镜像，然后重复地 1) `run(create和start)`、2) `修改`、3) `commit`。在循环中的每一步都会生成一个新的层，因此许多新的层会被创建。

- `sudo docker exec <running-container-id>`
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/exec.jpg" width="85%" height="85%">
`docker exec`命令会在运行中的容器执行一个新进程。

- `sudo docker inspect <container-id> or <image-id>`
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/inspect.jpg" width="85%" height="85%">
`docker inspect`命令会提取出容器或者镜像最顶层的元数据。

- `sudo docker save <image-id>`
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/save.jpg" width="85%" height="85%">
`docker save`命令会创建一个镜像的压缩文件，这个文件能够在另外一个主机的 Docker 上使用。和 export 命令不同，这个命令为每一个层都保存了它们的元数据。这个命令只能对镜像生效。

- `sudo docker export <container-id>`
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/export.jpg" width="85%" height="85%">
`docker export`命令创建一个tar文件，并且移除了元数据和不必要的层，将多个层整合成了一个层，只保存了当前统一视角看到的内容

- `sudo docker history <image-id>`
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/docker/history.jpg" width="85%" height="85%">
`docker history`命令递归地输出指定镜像的历史镜像。

### 参考文章

- [官方文档](https://docs.docker.com/)
- [Docker 核心技术与实现原理](http://dockone.io/article/2941)
- [CGroup 介绍、应用实例及原理描述](https://www.ibm.com/developerworks/cn/linux/1506_cgroup/index.html)
- [10 张图带你深入理解 Docker 容器和镜像](http://dockone.io/article/783)
- [Dockerfile、Docker 镜像和 Docker 容器的关系](http://blog.csdn.net/zhousenshan/article/details/51501734)
- [菜鸟教程 - Docker 命令大全](http://www.runoob.com/docker/docker-command-manual.html)
- [为什么 Docker 如此受欢迎？](http://cloud.51cto.com/art/201406/442131.htm)
