# Docker
Learning docker base on https://yeasy.gitbooks.io/docker_practice/

## 一、概述 镜像（image），容器（container），仓库（repository）
### 1. 镜像（image）
Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。
Docker 设计时，就充分利用 Union FS 的技术，将其设计为分层存储的架构。镜像只是一个虚拟的概念，其实际体现并非由一个文件组成，而是由一组文件系统组成，或者说，由多层文件系统联合组成。
镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。比如，删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。因此，在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。
分层存储的特征还使得镜像的复用、定制变的更为容易。甚至可以用之前构建好的镜像作为基础层，然后进一步添加新的层，以定制自己所需的内容，构建新的镜像。

### 2. 容器（container）
镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的 类 和 实例 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
按照 Docker 最佳实践的要求，容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。所有的文件写入操作，都应该使用 数据卷（Volume）、或者绑定宿主目录，在这些位置的读写会跳过容器存储层，直接对宿主（或网络存储）发生读写，其性能和稳定性更高。

### 3. 仓库（repository）
镜像构建完成后，可以很容易的在当前宿主机上运行，但是，如果需要在其它服务器上使用这个镜像，我们就需要一个集中的存储、分发镜像的服务，Docker Registry 就是这样的服务。
一个 Docker Registry 中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。
通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 <仓库名>:<标签> 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest 作为默认标签。

## 二、详细介绍
### 1. 镜像
#### 1.1. 获取镜像
docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
Docker 镜像仓库地址：地址的格式一般是 <域名/IP>[:端口号]。默认地址是 Docker Hub。
仓库名：如之前所说，这里的仓库名是两段式名称，即 <用户名>/<软件名>。对于 Docker Hub，如果不给出用户名，则默认为 library，也就是官方镜像。
#### 1.2. 列出镜像
列出顶级层镜像：docker image ls 
显示包括中间层镜像： docker image ls -a
支持在命令行中加入过滤器： -f 
修改显示的格式： --format
#### 1.3. 删除镜像
docker image rm [选项] <镜像1> [<镜像2> ...]

### 2.容器
#### 2.1. 启动容器
启动容器有两种方式，一种是基于镜像新建一个容器并启动，另外一个是将在终止状态（stopped）的容器重新启动。
新建并启动： docker run
启动已终止容器： docker container start
#### 2.2. 后台运行
更多的时候，需要让 Docker 在后台运行而不是直接把执行命令的结果输出在当前宿主机下。此时，可以通过添加 -d 参数来实现。
查看容器的信息使用命令  docker container ls， 查看容器的输入使用命令 docker container logs [container ID or NAMES]
#### 2.3. 终止容器
使用命令 docker container stop
处于终止状态的容器，可以通过 docker container start 命令来重新启动； 此外，docker container restart 命令会将一个运行态的容器终止，然后再重新启动它。
#### 2.4. 进入容器
使用 -d 会使容器进入后台运行，如果要进入容器操作可以使用命令docker attach 或者 exec。 推荐用exec
因为使用 attach 时，用exit 退出会导致容器结束，而exec不会
#### 2.5. 导入导出容器
如果要导出本地某个容器使用命令： docker export
例如：docker export 7691a814370e > ubuntu.tar 这样就把快照导出到本地文件
导入容器快照使用命令： docker import 
例如：cat ubuntu.tar | docker import - test/ubuntu:v1.0 从容器快照文件中再导入为镜像
也可以通过指定 URL 或者某个目录来导入，例如 docker import http://example.com/exampleimage.tgz example/imagerepo

用户既可以使用 docker load 来导入镜像存储文件到本地镜像库，这样会保存历史记录和元数据信息（即仅保存容器当时的快照状态),所以文件比较大，也可以使用docker import导入容器快照文件，这样会丢弃历史纪录和元数据信息
#### 2.6. 删除容器
可以使用 docker container rm 来删除一个处于终止状态的容器。如果要删除正在运行的容器，可以添加 -f 参数。Docker 会发送 SIGKILL 信号给容器。
清理所有处于终止状态的容器， 使用命令 docker container prune

### 3. 仓库（Repository）
仓库是集中存放镜像的地方
注册服务器(Registry),实际上是管理仓库的具体服务器，每个服务器上可以有多个仓库，每个仓库可以有多个镜像。

#### 3.1. Docker Hub
官方维护一个公共仓库 Docker Hub, 其中包括了数量超过15，000的镜像。
注册：可以在 https://cloud.docker.com 免费注册一个账号
登陆：可通过 docker login 命令交互式的数据用户名和密码来完成命令行界面登录 Docker Hub
退出：可通过 docker logout 命令退出登录
拉去镜像：通过 docker search 查找官方仓库中镜像，并利用 docker pull 命令将它下载到本地
推送镜像：通过 docker push 命令将自己的镜像推送到docker hub中
自动创建（Automated Builds）：对于需要经常升级镜像内程序来说十分方便。自动创建允许用户通过Docker Hub指定跟踪一个目标网站（目前支持docker Hub和Bitbucket）上的项目，一旦项目发生新的提交或者创建新的tag，Docker Hub会自动构建镜像并推送到 Docker Hub 中
#### 3.2. 私有仓库
有时候使用 Docker Hub 这样的公共仓库不方便，用户可以创建一个本地仓库共私人使用。使用官方的工具 docker-registry 可以构建私有仓库

