---
title: Docker Practice
date: 19-03-03 20:19:47
tags:
categories: 其他
---
## Docker准备知识

最常用操作从网上找dockerfile或者docker-compose

```bash
# ****************************** 容器 ****************************** #
# 查看正在运行的容器
$ docker ps
# 查看所有容器
$ docker ps -a
# 启动/停止某个容器
$ docker start/stop id/name
# 以交互方式启动一个容器
$ docker start -i id/name
# 进入某个容器(使用exit退出后容器也跟着停止运行)
$ docker attach id/name
# 启动一个伪终端以交互式的方式进入某个运行的容器（使用exit退出后容器不停止运行）
$ docker exec -it id/name
# 删除某个容器
$ docker rm id/name
# 复制ubuntu容器并且重命名为test且运行，然后以伪终端交互式方式进入容器，运行bash
$ docker run --name test -ti ubuntu /bin/bash

# ****************************** 镜像 ****************************** #
# 查看本地镜像
$ docker images
# 删除某个镜像
$ docker rmi id/name
# 基于当前目录下的Dockerfile，创建一个名为name:flag的镜像
$ docker build -t name:flag .

# 你可以通过 docker search 命令来查找官方仓库中的镜像，并利用 docker pull 命令来将它下载到本地。从dockerhub
$ docker pull java

```

你可能还需要了解一点Docker Compose的知识：

> Docker Compose 是 Docker 容器进行编排的工具，定义和运行多容器的应用，可以一条命令启动多个容器。
>
> 使用Compose 基本上分为三步：
>
> 1. Dockerfile 定义应用的运行环境
> 2. docker-compose.yml 定义组成应用的各服务
> 3. docker-compose up -d 启动整个应用
> 4. docker-compose down 停止整个应用

### Docker 镜像

我们都知道，操作系统分为内核和用户空间。对于 Linux 而言，内核启动后，会挂载 `root` 文件系统为其提供用户空间支持。而 Docker 镜像（Image），就相当于是一个 `root` 文件系统。比如官方镜像 `ubuntu:18.04` 就包含了完整的一套 Ubuntu 18.04 最小系统的 `root` 文件系统。

Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像不包含任何动态数据，其内容在构建之后也不会被改变。

**分层存储**

因为镜像包含操作系统完整的 `root` 文件系统，其体积往往是庞大的，因此在 Docker 设计时，就充分利用 [Union FS](https://en.wikipedia.org/wiki/Union_mount) 的技术，将其设计为分层存储的架构。所以严格来说，镜像并非是像一个 ISO 那样的打包文件，镜像只是一个虚拟的概念，其实际体现并非由一个文件组成，而是由一组文件系统组成，或者说，由多层文件系统联合组成。

镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。比如，删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。因此，在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。

分层存储的特征还使得镜像的复用、定制变的更为容易。甚至可以用之前构建好的镜像作为基础层，然后进一步添加新的层，以定制自己所需的内容，构建新的镜像。

关于镜像构建，将会在后续相关章节中做进一步的讲解。

### Docker 容器

镜像（`Image`）和容器（`Container`）的关系，就像是面向对象程序设计中的 `类` 和 `实例` 一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 [命名空间](https://en.wikipedia.org/wiki/Linux_namespaces)。因此容器可以拥有自己的 `root` 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。这种特性使得容器封装的应用比直接在宿主运行更加安全。也因为这种隔离的特性，很多人初学 Docker 时常常会混淆容器和虚拟机。

前面讲过镜像使用的是分层存储，容器也是如此。每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层，我们可以称这个为容器运行时读写而准备的存储层为 **容器存储层**。

容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，任何保存于容器存储层的信息都会随容器删除而丢失。

按照 Docker 最佳实践的要求，容器不应该向其存储层内写入任何数据，容器存储层要保持无状态化。所有的文件写入操作，都应该使用 [数据卷（Volume）](https://yeasy.gitbooks.io/docker_practice/data_management/volume.html)、或者绑定宿主目录，在这些位置的读写会跳过容器存储层，直接对宿主（或网络存储）发生读写，其性能和稳定性更高。

数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。因此，使用数据卷后，容器删除或者重新运行之后，数据却不会丢失。

### Docker仓库

镜像构建完成后，可以很容易的在当前宿主机上运行，但是，如果需要在其它服务器上使用这个镜像，我们就需要一个集中的存储、分发镜像的服务，[Docker Registry](https://yeasy.gitbooks.io/docker_practice/repository/registry.html) 就是这样的服务。

一个 **Docker Registry** 中可以包含多个 **仓库**（`Repository`）；每个仓库可以包含多个 **标签**（`Tag`）；每个标签对应一个镜像。

通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 `<仓库名>:<标签>` 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 `latest` 作为默认标签。

以 [Ubuntu 镜像](https://hub.docker.com/_/ubuntu) 为例，`ubuntu` 是仓库的名字，其内包含有不同的版本标签，如，`16.04`, `18.04`。我们可以通过 `ubuntu:16.04`，或者 `ubuntu:18.04` 来具体指定所需哪个版本的镜像。如果忽略了标签，比如 `ubuntu`，那将视为 `ubuntu:latest`。

仓库名经常以 *两段式路径* 形式出现，比如 `jwilder/nginx-proxy`，前者往往意味着 Docker Registry 多用户环境下的用户名，后者则往往是对应的软件名。但这并非绝对，取决于所使用的具体 Docker Registry 的软件或服务。

### 使用 Dockerfile 定制镜像

从刚才的 `docker commit` 的学习中，我们可以了解到，镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是 Dockerfile。

Dockerfile 是一个文本文件，其内包含了一条条的 **指令(Instruction)**，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。

还以之前定制 `nginx` 镜像为例，这次我们使用 Dockerfile 来定制。

在一个空白目录中，建立一个文本文件，并命名为 `Dockerfile`：

```bash
$ mkdir mynginx
$ cd mynginx
$ touch Dockerfile
```

其内容为：

```docker
FROM nginx
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

这个 Dockerfile 很简单，一共就两行。涉及到了两条指令，`FROM` 和 `RUN`。

#### FROM 指定基础镜像

所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。就像我们之前运行了一个 `nginx` 镜像的容器，再进行修改一样，基础镜像是必须指定的。而 `FROM` 就是指定 **基础镜像**，因此一个 `Dockerfile` 中 `FROM` 是必备的指令，并且必须是第一条指令。

在 [Docker Hub](https://hub.docker.com/search?q=&type=image&image_filter=official) 上有非常多的高质量的官方镜像，有可以直接拿来使用的服务类的镜像，如 [`nginx`](https://hub.docker.com/_/nginx/)、[`redis`](https://hub.docker.com/_/redis/)、[`mongo`](https://hub.docker.com/_/mongo/)、[`mysql`](https://hub.docker.com/_/mysql/)、[`httpd`](https://hub.docker.com/_/httpd/)、[`php`](https://hub.docker.com/_/php/)、[`tomcat`](https://hub.docker.com/_/tomcat/) 等；也有一些方便开发、构建、运行各种语言应用的镜像，如 [`node`](https://hub.docker.com/_/node)、[`openjdk`](https://hub.docker.com/_/openjdk/)、[`python`](https://hub.docker.com/_/python/)、[`ruby`](https://hub.docker.com/_/ruby/)、[`golang`](https://hub.docker.com/_/golang/) 等。可以在其中寻找一个最符合我们最终目标的镜像为基础镜像进行定制。

如果没有找到对应服务的镜像，官方镜像中还提供了一些更为基础的操作系统镜像，如 [`ubuntu`](https://hub.docker.com/_/ubuntu/)、[`debian`](https://hub.docker.com/_/debian/)、[`centos`](https://hub.docker.com/_/centos/)、[`fedora`](https://hub.docker.com/_/fedora/)、[`alpine`](https://hub.docker.com/_/alpine/) 等，这些操作系统的软件库为我们提供了更广阔的扩展空间。

除了选择现有镜像为基础镜像外，Docker 还存在一个特殊的镜像，名为 `scratch`。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。

```docker
FROM scratch
...
```

如果你以 `scratch` 为基础镜像的话，意味着你不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。

不以任何系统为基础，直接将可执行文件复制进镜像的做法并不罕见，比如 [`swarm`](https://hub.docker.com/_/swarm/)、[`etcd`](https://quay.io/repository/coreos/etcd)。对于 Linux 下静态编译的程序来说，并不需要有操作系统提供运行时支持，所需的一切库都已经在可执行文件里了，因此直接 `FROM scratch` 会让镜像体积更加小巧。使用 [Go 语言](https://golang.org/) 开发的应用很多会使用这种方式来制作镜像，这也是为什么有人认为 Go 是特别适合容器微服务架构的语言的原因之一。

#### RUN 执行命令

`RUN` 指令是用来执行命令行命令的。由于命令行的强大能力，`RUN` 指令在定制镜像时是最常用的指令之一。其格式有两种：

- *shell* 格式：`RUN <命令>`，就像直接在命令行中输入的命令一样。刚才写的 Dockerfile 中的 `RUN` 指令就是这种格式。

```docker
RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
```

- *exec* 格式：`RUN ["可执行文件", "参数1", "参数2"]`，这更像是函数调用中的格式。

既然 `RUN` 就像 Shell 脚本一样可以执行命令，那么我们是否就可以像 Shell 脚本一样把每个命令对应一个 RUN 呢？比如这样：

```docker
FROM debian:stretch

RUN apt-get update
RUN apt-get install -y gcc libc6-dev make wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN mkdir -p /usr/src/redis
RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1
RUN make -C /usr/src/redis
RUN make -C /usr/src/redis install
```

之前说过，Dockerfile 中每一个指令都会建立一层，`RUN` 也不例外。每一个 `RUN` 的行为，就和刚才我们手工建立镜像的过程一样：新建立一层，在其上执行这些命令，执行结束后，`commit` 这一层的修改，构成新的镜像。

而上面的这种写法，创建了 7 层镜像。这是完全没有意义的，而且很多运行时不需要的东西，都被装进了镜像里，比如编译环境、更新的软件包等等。结果就是产生非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。 这是很多初学 Docker 的人常犯的一个错误。

*Union FS 是有最大层数限制的，比如 AUFS，曾经是最大不得超过 42 层，现在是不得超过 127 层。*

上面的 `Dockerfile` 正确的写法应该是这样：

```docker
FROM debian:stretch

RUN buildDeps='gcc libc6-dev make wget' \
    && apt-get update \
    && apt-get install -y $buildDeps \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && mkdir -p /usr/src/redis \
    && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
    && make -C /usr/src/redis \
    && make -C /usr/src/redis install \
    && rm -rf /var/lib/apt/lists/* \
    && rm redis.tar.gz \
    && rm -r /usr/src/redis \
    && apt-get purge -y --auto-remove $buildDeps
```

首先，之前所有的命令只有一个目的，就是编译、安装 redis 可执行文件。因此没有必要建立很多层，这只是一层的事情。因此，这里没有使用很多个 `RUN` 对一一对应不同的命令，而是仅仅使用一个 `RUN` 指令，并使用 `&&` 将各个所需命令串联起来。将之前的 7 层，简化为了 1 层。在撰写 Dockerfile 的时候，要经常提醒自己，这并不是在写 Shell 脚本，而是在定义每一层该如何构建。

并且，这里为了格式化还进行了换行。Dockerfile 支持 Shell 类的行尾添加 `\` 的命令换行方式，以及行首 `#` 进行注释的格式。良好的格式，比如换行、缩进、注释等，会让维护、排障更为容易，这是一个比较好的习惯。

此外，还可以看到这一组命令的最后添加了清理工作的命令，删除了为了编译构建所需要的软件，清理了所有下载、展开的文件，并且还清理了 `apt` 缓存文件。这是很重要的一步，我们之前说过，镜像是多层存储，每一层的东西并不会在下一层被删除，会一直跟随着镜像。因此镜像构建时，一定要确保每一层只添加真正需要添加的东西，任何无关的东西都应该清理掉。

很多人初学 Docker 制作出了很臃肿的镜像的原因之一，就是忘记了每一层构建的最后一定要清理掉无关文件。

#### 构建镜像

好了，让我们再回到之前定制的 nginx 镜像的 Dockerfile 来。现在我们明白了这个 Dockerfile 的内容，那么让我们来构建这个镜像吧。

在 `Dockerfile` 文件所在目录执行：

```bash
$ docker build -t nginx:v3 .
Sending build context to Docker daemon 2.048 kB
Step 1 : FROM nginx
 ---> e43d811ce2f4
Step 2 : RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html
 ---> Running in 9cdc27646c7b
 ---> 44aa4490ce2c
Removing intermediate container 9cdc27646c7b
Successfully built 44aa4490ce2c
```

从命令的输出结果中，我们可以清晰的看到镜像的构建过程。在 `Step 2` 中，如同我们之前所说的那样，`RUN` 指令启动了一个容器 `9cdc27646c7b`，执行了所要求的命令，并最后提交了这一层 `44aa4490ce2c`，随后删除了所用到的这个容器 `9cdc27646c7b`。

这里我们使用了 `docker build` 命令进行镜像构建。其格式为：

```bash
docker build [选项] <上下文路径/URL/->
```

在这里我们指定了最终镜像的名称 `-t nginx:v3`，构建成功后，我们可以像之前运行 `nginx:v2` 那样来运行这个镜像，其结果会和 `nginx:v2` 一样。

#### 镜像构建上下文（Context）

如果注意，会看到 `docker build` 命令最后有一个 `.`。`.` 表示当前目录，而 `Dockerfile` 就在当前目录，因此不少初学者以为这个路径是在指定 `Dockerfile` 所在路径，这么理解其实是不准确的。如果对应上面的命令格式，你可能会发现，这是在指定 **上下文路径**。那么什么是上下文呢？

首先我们要理解 `docker build` 的工作原理。Docker 在运行时分为 Docker 引擎（也就是服务端守护进程）和客户端工具。Docker 的引擎提供了一组 REST API，被称为 [Docker Remote API](https://docs.docker.com/develop/sdk/)，而如 `docker` 命令这样的客户端工具，则是通过这组 API 与 Docker 引擎交互，从而完成各种功能。因此，虽然表面上我们好像是在本机执行各种 `docker` 功能，但实际上，一切都是使用的远程调用形式在服务端（Docker 引擎）完成。也因为这种 C/S 设计，让我们操作远程服务器的 Docker 引擎变得轻而易举。

当我们进行镜像构建的时候，并非所有定制都会通过 `RUN` 指令完成，经常会需要将一些本地文件复制进镜像，比如通过 `COPY` 指令、`ADD` 指令等。而 `docker build` 命令构建镜像，其实并非在本地构建，而是在服务端，也就是 Docker 引擎中构建的。那么在这种客户端/服务端的架构中，如何才能让服务端获得本地文件呢？

这就引入了上下文的概念。当构建的时候，用户会指定构建镜像上下文的路径，`docker build` 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。

如果在 `Dockerfile` 中这么写：

```docker
COPY ./package.json /app/
```

这并不是要复制执行 `docker build` 命令所在的目录下的 `package.json`，也不是复制 `Dockerfile` 所在目录下的 `package.json`，而是复制 **上下文（context）** 目录下的 `package.json`。

因此，`COPY` 这类指令中的源文件的路径都是*相对路径*。这也是初学者经常会问的为什么 `COPY ../package.json /app` 或者 `COPY /opt/xxxx /app` 无法工作的原因，因为这些路径已经超出了上下文的范围，Docker 引擎无法获得这些位置的文件。如果真的需要那些文件，应该将它们复制到上下文目录中去。

现在就可以理解刚才的命令 `docker build -t nginx:v3 .` 中的这个 `.`，实际上是在指定上下文的目录，`docker build` 命令会将该目录下的内容打包交给 Docker 引擎以帮助构建镜像。

如果观察 `docker build` 输出，我们其实已经看到了这个发送上下文的过程：

```bash
$ docker build -t nginx:v3 .
Sending build context to Docker daemon 2.048 kB
...
```

理解构建上下文对于镜像构建是很重要的，避免犯一些不应该的错误。比如有些初学者在发现 `COPY /opt/xxxx /app` 不工作后，于是干脆将 `Dockerfile` 放到了硬盘根目录去构建，结果发现 `docker build` 执行后，在发送一个几十 GB 的东西，极为缓慢而且很容易构建失败。那是因为这种做法是在让 `docker build` 打包整个硬盘，这显然是使用错误。

一般来说，应该会将 `Dockerfile` 置于一个空目录下，或者项目根目录下。如果该目录下没有所需文件，那么应该把所需文件复制一份过来。如果目录下有些东西确实不希望构建时传给 Docker 引擎，那么可以用 `.gitignore` 一样的语法写一个 `.dockerignore`，该文件是用于剔除不需要作为上下文传递给 Docker 引擎的。

那么为什么会有人误以为 `.` 是指定 `Dockerfile` 所在目录呢？这是因为在默认情况下，如果不额外指定 `Dockerfile` 的话，会将上下文目录下的名为 `Dockerfile` 的文件作为 Dockerfile。

这只是默认行为，实际上 `Dockerfile` 的文件名并不要求必须为 `Dockerfile`，而且并不要求必须位于上下文目录中，比如可以用 `-f ../Dockerfile.php` 参数指定某个文件作为 `Dockerfile`。

当然，一般大家习惯性的会使用默认的文件名 `Dockerfile`，以及会将其置于镜像构建上下文目录中。

#### 其它 `docker build` 的用法

##### 直接用 Git repo 进行构建

或许你已经注意到了，`docker build` 还支持从 URL 构建，比如可以直接从 Git repo 中构建：

```bash
$ docker build https://github.com/twang2218/gitlab-ce-zh.git#:11.1

Sending build context to Docker daemon 2.048 kB
Step 1 : FROM gitlab/gitlab-ce:11.1.0-ce.0
11.1.0-ce.0: Pulling from gitlab/gitlab-ce
aed15891ba52: Already exists
773ae8583d14: Already exists
...
```

这行命令指定了构建所需的 Git repo，并且指定默认的 `master` 分支，构建目录为 `/11.1/`，然后 Docker 就会自己去 `git clone` 这个项目、切换到指定分支、并进入到指定目录后开始构建。

##### 用给定的 tar 压缩包构建

```bash
$ docker build http://server/context.tar.gz
```

如果所给出的 URL 不是个 Git repo，而是个 `tar` 压缩包，那么 Docker 引擎会下载这个包，并自动解压缩，以其作为上下文，开始构建。

从标准输入中读取 Dockerfile 进行构建

```bash
docker build - < Dockerfile
```

或

```bash
cat Dockerfile | docker build -
```

如果标准输入传入的是文本文件，则将其视为 `Dockerfile`，并开始构建。这种形式由于直接从标准输入中读取 Dockerfile 的内容，它没有上下文，因此不可以像其他方法那样可以将本地文件 `COPY` 进镜像之类的事情。

##### 从标准输入中读取上下文压缩包进行构建

```bash
$ docker build - < context.tar.gz
```

如果发现标准输入的文件格式是 `gzip`、`bzip2` 以及 `xz` 的话，将会使其为上下文压缩包，直接将其展开，将里面视为上下文，并开始构建。

## Docker归纳

## Dockerfile归纳

```dockerfile
#基于centos镜像有点类似pull
FROM centos

#维护人的信息
MAINTAINER The CentOS Project <303323496@qq.com>

#安装httpd软件包
RUN yum -y update
RUN yum -y install httpd

#开启80端口
EXPOSE 80

#复制网站首页文件至镜像中web站点下
ADD index.html /var/www/html/index.html

#复制该脚本至镜像中，并修改其权限
ADD run.sh /run.sh
RUN chmod 775 /run.sh

#当启动容器时执行的脚本文件
CMD ["/run.sh"]
```

　　由上可知，Dockerfile结构大致分为四个部分：

　　（1）基础镜像信息

　　（2）维护者信息

　　（3）镜像操作指令

　　（4）容器启动时执行指令。

　　Dockerfile每行支持一条指令，每条指令可带多个参数，支持使用以#号开头的注释。下面会对上面使用到的一些常用指令做一些介绍。

```dockerfile
# Use an official Python runtime as an image
FROM python:3.6

# The EXPOSE instruction indicates the ports on which a container 
# will listen for connections
# Since Flask apps listen to port 5000  by default, we expose it
EXPOSE 5000

# Sets the working directory for following COPY and CMD instructions
# Notice we haven’t created a directory by this name - this instruction 
# creates a directory with this name if it doesn’t exist
WORKDIR /app

# Install any needed packages specified in requirements.txt
COPY requirements.txt /app
RUN pip install -r requirements.txt

# Run app.py when the container launches
COPY app.py /app
CMD python app.py
```

![img](https://img2018.cnblogs.com/blog/450977/201905/450977-20190512115951746-136143052.png)

## Docker-compose归纳

使用dockerfile定制镜像/比如前后端

```yaml
version: '3' # 表示该 Docker-Compose 文件使用的是 Version 3 file
services:
  nginx:
    restart: always
    build: ./nginx
    ports:
      - "80:80"
    volumes:
      - .:/www/static
      - web-data:/usr/src/app/static
    links:
      - web:web
      
     
```

```
访问 http://localhost:9000/hello 即可访问微服务接口
```
使用默认dockhub镜像，不需要dockerfile/

比如数据库，数据库建议不使用docker

```yaml

  db:
    image: mysql:5.7
    ports:
      - "32000:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - ./db:/docker-entrypoint-initdb.d/:ro
```



## Flask部署到docker

```python
from flask import Flask
 
app = Flask(__name__)
 
 __
@app.route('/')
def hello_whale():
    return ‘Whale, Hello there!’
 
if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0')
```

用虚拟环境生成requirement文件

然后编写dockerfile

```dockerfile
FROM ubuntu:latest
RUN apt-get update -y
RUN apt-get install -y python-pip python-dev build-essential
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
ENTRYPOINT ["python"]
CMD ["app.py"]
```

## 版本2 (简单flask)

```dockerfile
FROM ubuntu:16.04

MAINTANER Your Name "youremail@domain.tld"

RUN apt-get update -y && \
    apt-get install -y python-pip python-dev

# We copy just the requirements.txt first to leverage Docker cache
COPY ./requirements.txt /app/requirements.txt

WORKDIR /app

RUN pip install -r requirements.txt

COPY . /app

ENTRYPOINT [ "python" ]
CMD [ "app.py" ]
```

To build this image, I’ll run:

```
$ docker build -t flask-sample:latest .
```

And to run the container, I’ll run:

```
$ docker run -d -p 5000:5000 flask-sample
```

Now, if I do a docker ps -a…

![img](https://codefresh.io/wp-content/uploads/2017/06/Screen-Shot-2017-06-07-at-4.52.38-PM.png)

现在可以进入浏览器 查看5000端口了



### Docker-compose

Cool- so, now I Dockerized my Flask app. Let’s run it using Docker Compose.

First we’ll need to remove our container running at port 5000. You can do that by typing **docker ps -a**, into your terminal, and copying the container ID to write:

Now, in our flask_demo directory, I need to make a docker-compose.yml with the following in it:

```yaml
web:
  build: ./web
  ports:
   - "5000:5000"
  volumes:
   - .:/code
```

That’s it! From our chloe-flask-demo directory, run…

```cmd
docker-compose up
```

现在可以进入浏览器 查看5000端口测试了

Now, go to port 5000, and…

## 版本三源码(简单Flask)

In this post, I'll show how to serve a simple Flask application with Gunicorn, running inside a Docker container.

Let's begin from creating a minimal Flask application:

```python
from flask import Flask

app = Flask(__name__)


@app.route('/')
@app.route('/index')
def index():
    return 'Hello world!'
```

Next, let's write the command that will run the Gunicorn server:

```cmd
#!/bin/sh

gunicorn --chdir app main:app -w 2 --threads 2 -b 0.0.0.0:8003
```

The parameters are pretty much self-explanatory: We are telling Gunicorn to spawn 2 worker processes, running 2 threads each. We are also accepting connections from outside, and overriding Gunicorn's default port (8000).

Our basic Dockerfile:

```python
FROM python:3.7.3-slim

COPY requirements.txt /
RUN pip3 install -r /requirements.txt

COPY . /app
WORKDIR /app

ENTRYPOINT ["./gunicorn_starter.sh"]
```

Let's build our image:

```cmd
docker build -t flask/hello-world .
```

and run:

```cmd
docker run -p 8003:8003 flask/hello-world
```

Now we should be able to access our endpoint:

```cmd
$ curl localhost:8003
Hello world!
```

### Bonus - Makefile

Let's create a simple *Makefile* that allows us to build, run and kill our image/container:

```
app_name = gunicorn_flask

build:
    @docker build -t $(app_name) .

run:
    docker run --detach -p 8003:8003 $(app_name)

kill:
    @echo 'Killing container...'
    @docker ps | grep $(app_name) | awk '{print $$1}' | xargs docker
```

Now we should be able to run:

```
# build Docker image
make build
# run the container
make run
# destroy it
make kill
```

## 版本四(Flask/mysql  )

### Let’s start

All code used in this tutorial is available here:

[ stavshamir / docker-tutorial](https://github.com/stavshamir/docker-tutorial)

If you don’t have them installed yet, install [Docker](https://docs.docker.com/install/linux/docker-ce/ubuntu/) and [docker-compose](https://docs.docker.com/compose/install/). We begin with the following project layout:

```
dockerize/
├── app
│   └── app.py
└── db
    └── init.sql
```

- *app.py* — contains the Flask app which connects to the database and exposes one REST API endpoint
- *init.sql* — an SQL script to initialize the database before the first time the app runs

### Creating a Docker image for our app[Permalink](https://stavshamir.github.io/python/dockerizing-a-flask-mysql-app-with-docker-compose/#creating-a-docker-image-for-our-app)

We want to create a Docker image for our app, so we need to create a Dockerfile in the app directory. A Dockerfile contains a set of instructions describing our desired image and allow its automatic build.

```dockerfile
# Use an official Python runtime as an image
FROM python:3.6

# The EXPOSE instruction indicates the ports on which a container 
# will listen for connections
# Since Flask apps listen to port 5000  by default, we expose it
EXPOSE 5000

# Sets the working directory for following COPY and CMD instructions
# Notice we haven’t created a directory by this name - this instruction 
# creates a directory with this name if it doesn’t exist
WORKDIR /app

# Install any needed packages specified in requirements.txt
COPY requirements.txt /app
RUN pip install -r requirements.txt

# Run app.py when the container launches
COPY app.py /app
CMD python app.py
```

What this does is simply as described in the file — base the image on a Python 3.6 image, expose port 5000 (for Flask), create a working directory to which requirements.txt and app.py will be copied, install the needed packages and run the app.

We need our dependencies (Flask and mysql-connector) to be installed and delivered with the image, so we need to create the aforementioned requirements.txt file:

```
Flask
mysql-connector
```

Now we can create a docker image for our app, but we still can’t use it, since it depends on MySQL, which, as good practice commens, will reside in a different container. We will use docker-compose to facilitate the orchestration of the two independant containers into one working app.

### Creating a docker-compose.yml[Permalink](https://stavshamir.github.io/python/dockerizing-a-flask-mysql-app-with-docker-compose/#creating-a-docker-composeyml)

So let’s create a new file, docker-compose.yml, in our project’s root directory:

```
version: "2"
services:
  app:
    build: ./app
    links:
      - db
    ports:
      - "5000:5000"
```

We are using two services, one is a container which exposes the REST API (app), and one contains the database (db).

- *build:* specifies the directory which contains the Dockerfile containing the instructions for building this service
- *links:* links this service to another container. This will also allow us to use the name of the service instead of having to find the ip of the database container, and express a dependency which will determine the order of start up of the container
- *ports:* mapping of <Host>:<Container> ports.

```
  db:
    image: mysql:5.7
    ports:
      - "32000:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root
    volumes:
      - ./db:/docker-entrypoint-initdb.d/:ro
```

- *image:* Like the FROM instruction from the Dockerfile. Instead of writing a new Dockerfile, we are using an existing image from a repository. It’s important to specify the version — if your installed mysql client is not of the same version problems may occur.
- *environment:* add environment variables. The specified variable is required for this image, and as its name suggests, configures the password for the root user of MySQL in this container. More variables are specified here.
- *ports:* Since I already have a running mysql instance on my host using this port, I am mapping it to a different one. Notice that the mapping is only from host to container, so our app service container will still use port 3306 to connect to the database.
- *volumes:* since we want the container to be initialized with our schema, we wire the directory containing our init.sql script to the entry point for this container, which by the image’s specification runs all .sql scripts in the given directory.

We are now ready to start the dockerized app! But before we do that, let’s peek at the code connecting to the database (app.py):

```
config = {
        'user': 'root',
        'password': 'root',
        'host': 'db',
        'port': '3306',
        'database': 'knights'
    }
connection = mysql.connector.connect(**config)
```

We are connecting as root with the password configured in the docker-compose file. Notice that we explicitly define the host (which is localhost by default) since the SQL service is actually in a different container than the one running this code. We can (and should) use the name ‘db’ since this is the name of the service we defined and linked to earlier, and the port is 3306 and not 32000 since this code is not running on the host.

### Running the app[Permalink](https://stavshamir.github.io/python/dockerizing-a-flask-mysql-app-with-docker-compose/#running-the-app)

In order to run the our dockerized app, we will execute the following command from the terminal:

```
$ docker-compose up
```

You can see the image being built, the packages installed according to the requirements.txt, etc. If everything went right, you will see the following line:

```
app_1  |  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
```

We can find out that everything is running as expected by typing this url in a browser or using curl, and receiving the following response:

```
{"favorite_colors": [{"Lancelot": "blue"}, {"Galahad": "yellow"}]}
```

You can access the database directly using the mysql client and following command (make sure your client is the same version of MySQL specified in the docker-compose.yml):

```
$ mysql --host=127.0.0.1 --port=32000 -u root -p
```

It is a must to specify the host IP, since [when the default ‘localhost’ is used MySQL ignores the port parameter](https://dev.mysql.com/doc/refman/5.7/en/connecting.html).

### Conclusion[Permalink](https://stavshamir.github.io/python/dockerizing-a-flask-mysql-app-with-docker-compose/#conclusion)

We have learned how to dockerize a simple Flask-MySQL using docker-compose. Now this app can be used without tiresome preconfiguration on every host with Docker and docker-compose, and is encapsulated from the host environment!

## *版本五(nginx flask  postgres)

### Local Setup

Along with Docker (v18.09.2) we’ll be using -

- *Docker Compose* (v1.23.2) - previously known as fig - for orchestrating a multi-container application into a single app, and
- *Docker Machine* (v0.16.1) for creating Docker hosts both locally and in the cloud.

Follow the directions [here](http://docs.docker.com/compose/install/) and [here](https://docs.docker.com/machine/install-machine/) to install Docker Compose and Machine, respectively.

Running either older Mac OS X or Windows versions, then your best bet is to install [Docker Toolbox](https://www.docker.com/docker-toolbox).

Test out the installs:

```
$ docker-machine --version
docker-machine version 0.16.1, build cce350d7
$ docker-compose --version
docker-compose version 1.23.2, build 1110ad01
```

Next clone the project from the [repository](https://github.com/realpython/orchestrating-docker) or create your own project based on the project structure found on the repo:

```
├── docker-compose.yml
├── nginx
│   ├── Dockerfile
│   └── sites-enabled
│       └── flask_project
└── web
    ├── Dockerfile
    ├── app.py
    ├── config.py
    ├── create_db.py
    ├── models.py
    ├── requirements.txt
    ├── static
    │   ├── css
    │   │   ├── bootstrap.min.css
    │   │   └── main.css
    │   ├── img
    │   └── js
    │       ├── bootstrap.min.js
    │       └── main.js
    └── templates
        ├── _base.html
        └── index.html
```

We’re now ready to get the containers up and running. Enter Docker Machine.

### Docker Machine

To start Docker Machine, first make sure you’re in the project root and then simply run:

```
$ docker-machine create -d virtualbox dev;
Creating CA: /Users/realpython/.docker/machine/certs/ca.pem
Creating client certificate: /Users/realpython/.docker/machine/certs/cert.pem
Running pre-create checks...
(dev) Image cache directory does not exist, creating it at /Users/realpython/.docker/machine/cache...
(dev) No default Boot2Docker ISO found locally, downloading the latest release...
(dev) Latest release for github.com/boot2docker/boot2docker is v18.09.3
(dev) Downloading /Users/realpython/.docker/machine/cache/boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v18.09.3/boot2docker.iso...
(dev) 0%....10%....20%....30%....40%....50%....60%....70%....80%....90%....100%
Creating machine...
(dev) Copying /Users/realpython/.docker/machine/cache/boot2docker.iso to /Users/realpython/.docker/machine/machines/dev/boot2docker.iso...
(dev) Creating VirtualBox VM...
(dev) Creating SSH key...
(dev) Starting the VM...
(dev) Check network to re-create if needed...
(dev) Found a new host-only adapter: "vboxnet0"
(dev) Waiting for an IP...
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
To see how to connect your Docker Client to the Docker Engine running on this virtual machine, run: docker-machine env dev
```

The `create` command setup a “machine” (called *dev*) for Docker development. In essence, it downloaded boot2docker and started a VM with Docker running. Now just point the Docker client at the *dev* machine via:

```
$ eval "$(docker-machine env dev)"
```

Run the following command to view the currently running Machines:

```
$ docker-machine ls
NAME   ACTIVE   DRIVER       STATE     URL                         SWARM   DOCKER     ERRORS
dev    *        virtualbox   Running   tcp://192.168.99.100:2376           v18.09.3
```

Next, let’s fire up the containers with Docker Compose and get the Flask app and Postgres database up and running.

### Docker Compose

Take a look at the *docker-compose.yml* file:

```
version: '3'

services:
  web:
    restart: always
    build: ./web
    expose:
      - "8000"
    links:
      - postgres:postgres
    volumes:
      - web-data:/usr/src/app/static
    env_file: 
      - .env
    command: /usr/local/bin/gunicorn -w 2 -b :8000 app:app

  nginx:
    restart: always
    build: ./nginx
    ports:
      - "80:80"
    volumes:
      - .:/www/static
      - web-data:/usr/src/app/static
    links:
      - web:web

  data:
    image: postgres:latest
    volumes:
      - db-data:/var/lib/postgresql/data
    command: "true"

  postgres:
    restart: always
    image: postgres:latest
    volumes:
      - db-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  db-data:
  web-data:
```

Here, we’re defining four services - *web*, *nginx*, *postgres*, and *data*.

1. First, the *web* service is built via the instructions in the *Dockerfile* within the “web” directory - where the Python environment is setup, requirements are installed, and the Flask app is fired up on port 8000. That port is then forwarded to port 80 on the host environment - e.g., the Docker Machine. This service also adds environment variables to the container that are defined in the *.env* file.
2. The *nginx* service is used for reverse proxy to forward requests either to the Flask app or the static files.
3. Next, the *postgres* service is built from the the official [PostgreSQL image](https://registry.hub.docker.com/_/postgres/) from [Docker Hub](https://hub.docker.com/), which install Postgres and runs the server on the default port 5432.
4. Finally, notice how there is a separate [volume](https://docs.docker.com/userguide/dockervolumes/) container that’s used to store the database data, *db-data*. This helps ensure that the data persists even if the Postgres container is completely destroyed.

Now, to get the containers running, build the images and then start the services:

```
$ docker-compose build
$ docker-compose up -d
```

**Tip:** You can even run the above commands combined in a single one:

```
$ docker-compose up --build -d
```

Grab a cup of coffee. Or two. Check out the [Real Python courses](https://realpython.com/courses/). This will take a while the first time you run it.

We also need to create the database table:

```
$ docker-compose run web /usr/local/bin/python create_db.py
```

Open your browser and navigate to the IP address associated with Docker Machine (`docker-machine ip`):

[![Flask app form](https://files.realpython.com/media/flask_app_docker.77aba550c513.png)](https://files.realpython.com/media/flask_app_docker.77aba550c513.png)

Nice!

To see which environment variables are available to the *web* service, run:

```
$ docker-compose run web env
```

To view the logs:

```
$ docker-compose logs
```

You can also enter the Postgres Shell - since we forward the port to the host environment in the *docker-compose.yml* file - to add users/roles as well as databases via:

```
$ docker-compose run postgres psql -h 192.168.99.100 -p 5432 -U postgres --password
```

Once done, stop the processes via `docker-compose down`.

<iframe frameborder="0" src="https://e32e9320cef45e229efaa6fef53e5527.safeframe.googlesyndication.com/safeframe/1-0-37/html/container.html" id="google_ads_iframe_/124067137/realpython728x90FS_2_0" title="3rd party ad content" name="" scrolling="no" marginwidth="0" marginheight="0" width="300" height="250" data-is-safeframe="true" sandbox="allow-forms allow-pointer-lock allow-popups allow-popups-to-escape-sandbox allow-same-origin allow-scripts allow-top-navigation-by-user-activation" data-google-container-id="e32e9320cef45e229efaa6fef53e5527" data-load-complete="true" style="box-sizing: border-box; border: 0px; vertical-align: bottom;"></iframe>
[ Remove ads](https://realpython.com/account/join/)

### Deployment

So, with our app running locally, we can now push this exact same environment to a cloud hosting provider with Docker Machine. Let’s deploy to a [Digital Ocean](https://www.digitalocean.com/?refcode=d8f211a4b4c2) droplet.

After you [sign up](https://www.digitalocean.com/?refcode=d8f211a4b4c2) for Digital Ocean, generate a [Personal Access Token](https://www.digitalocean.com/community/tutorials/how-to-use-the-digitalocean-api-v2), and then run the following command:

```
$ docker-machine create \
-d digitalocean \
--digitalocean-access-token ADD_YOUR_TOKEN_HERE \
production
```

This will take a few minutes to provision the droplet and setup a new Docker Machine called *production*:

```
Running pre-create checks...
Creating machine...
Waiting for machine to be running, this may take a few minutes...
Machine is running, waiting for SSH to be available...
Detecting operating system of created instance...
Provisioning created instance...
Copying certs to the local machine directory...
Copying certs to the remote machine...
Setting Docker configuration on the remote daemon...
To see how to connect Docker to this machine, run: docker-machine env production
```

Now we have two Machines running, one locally and one on Digital Ocean:

```
$ docker-machine ls
NAME         ACTIVE   DRIVER         STATE     URL                         SWARM   DOCKER     ERRORS
dev          *        virtualbox     Running   tcp://192.168.99.100:2376           v18.09.3
production   -        digitalocean   Running   tcp://104.131.93.156:2376           v18.09.3
```

Then set *production* as the active machine and load the Docker environment into the shell:

```
$ eval "$(docker-machine env production)"
```

Finally, let’s build the Flask app again in the cloud:

```
$ docker-compose build
$ docker-compose up -d
$ docker-compose run web /usr/local/bin/python create_db.py
```

Grab the IP address associated with that Digital Ocean account from the [control panel](https://cloud.digitalocean.com/droplets) and view it in the browser. If all went well, you should see your app running.

### Conclusion

Cheers!

- Grab the code from the [Github repo](https://github.com/realpython/orchestrating-docker) (star it too!).
- Comment below with questions.
- Next time we’ll extend this workflow to include two more Docker containers running the Flask app and incorporate load balancing into the mix. Stay tuned!



## 版本二源码

### Project Setup

Create a new project directory and install Flask:

```cmd
$ mkdir flask-on-docker && cd flask-on-docker
$ mkdir services && cd services
$ mkdir web && cd web
$ mkdir project
$ python3.8 -m venv env
$ source env/bin/activate
(env)$ pip install flask==1.1.1
```

> Feel free to swap out virtualenv and Pip for [Poetry](https://python-poetry.org/) or [Pipenv](https://pipenv.kennethreitz.org/).

Next, let's create a new Flask app.

Add an *__init__.py* file to the "project" directory and configure the first route:

```python
from flask import Flask, jsonify


app = Flask(__name__)


@app.route("/")
def hello_world():
    return jsonify(hello="world")
```

Then, to configure the Flask CLI tool to run and manage the app from the command line, add a *manage.py* file to the "web" directory:

```python
from flask.cli import FlaskGroup

from project import app


cli = FlaskGroup(app)


if __name__ == "__main__":
    cli()
```

Here, we created a new `FlaskGroup` instance to extend the normal CLI with commands related to the Flask app.

Run the server from the "web" directory:

```c&amp;#39;m&amp;#39;d
(env)$ export FLASK_APP=project/__init__.py
(env)$ python manage.py run
```

Navigate to http://localhost:5000/. You should see:

```
{
  "hello": "world"
}
```

Kill the server once done. Exit then remove the virtual environment as well.

Create a *requirements.txt* file in the "web" directory and add Flask as a dependency:

```
Flask==1.1.1
```

Your project structure should look like:

```
└── services
    └── web
        ├── manage.py
        ├── project
        │   └── __init__.py
        └── requirements.txt
```

#### dockerfile

```dockerfile
# pull official base image
FROM python:3.8.1-slim-buster

# set work directory
WORKDIR /usr/src/app

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# install dependencies
RUN pip install --upgrade pip
COPY ./requirements.txt /usr/src/app/requirements.txt
RUN pip install -r requirements.txt

# copy project
COPY . /usr/src/app/
```

So, we started with a `slim-buster`-based [Docker image](https://hub.docker.com/_/python/) for Python 3.8.1. We then set a [working directory](https://docs.docker.com/engine/reference/builder/#workdir) along with two environment variables:

1. `PYTHONDONTWRITEBYTECODE`: Prevents Python from writing pyc files to disc (equivalent to `python -B` [option](https://docs.python.org/3/using/cmdline.html#id1))
2. `PYTHONUNBUFFERED`: Prevents Python from buffering stdout and stderr (equivalent to `python -u` [option](https://docs.python.org/3/using/cmdline.html#cmdoption-u))

Finally, we updated Pip, copied over the *requirements.txt* file, installed the dependencies, and copied over the Flask app itself.

> Review [Docker for Python Developers](https://mherman.org/presentations/dockercon-2018) for more on structuring Dockerfiles as well as some best practices for configuring Docker for Python-based development.

Next, add a *docker-compose.yml* file to the project root:

#### Docker-compose

```yaml
version: '3.7'

services:
  web:
    build: ./services/web
    command: python manage.py run -h 0.0.0.0
    volumes:
      - ./services/web/:/usr/src/app/
    ports:
      - 5000:5000
    env_file:
      - ./.env.dev
```

Review the [Compose file reference](https://docs.docker.com/compose/compose-file/) for info on how this file works.

Then, create a *.env.dev* file in the project root to store environment variables for development:

```
FLASK_APP=project/__init__.py
FLASK_ENV=development
```

Build the image:

```
$ docker-compose build
```

Once the image is built, run the container:

```
$ docker-compose up -d
```

Navigate to http://localhost:5000/ to again view the hello world sanity check.

> Check for errors in the logs if this doesn't work via `docker-compose logs -f`.

### Postgres

To configure Postgres, we need to add a new service to the *docker-compose.yml* file, set up [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/), and install [Psycopg2](http://initd.org/psycopg/).

First, add a new service called `db` to *docker-compose.yml*:

```yaml
version: '3.7'

services:
  web:
    build: ./services/web
    command: python manage.py run -h 0.0.0.0
    volumes:
      - ./services/web/:/usr/src/app/
    ports:
      - 5000:5000
    env_file:
      - ./.env.dev
    depends_on:
      - db
  db:
    image: postgres:12-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    environment:
      - POSTGRES_USER=hello_flask
      - POSTGRES_PASSWORD=hello_flask
      - POSTGRES_DB=hello_flask_dev

volumes:
  postgres_data:
```

To persist the data beyond the life of the container we configured a volume. This config will bind `postgres_data` to the `/var/lib/postgresql/data/` directory in the container.

We also added an environment key to define a name for the default database and set a username and password.

We also added an environment key to define a name for the default database and set a username and password.

> Review the "Environment Variables" section of the [Postgres Docker Hub page](https://hub.docker.com/_/postgres) for more info.

Add a `DATABASE_URL` environment variable to *.env.dev* as well:

```
FLASK_APP=project/__init__.py
FLASK_ENV=development
DATABASE_URL=postgresql://hello_flask:hello_flask@db:5432/hello_flask_dev
```

Then, add a new file called *config.py* to the "project" directory, where we'll define environment-specific [configuration](https://flask.palletsprojects.com/config/) variables:

```
import os


basedir = os.path.abspath(os.path.dirname(__file__))


class Config(object):
    SQLALCHEMY_DATABASE_URI = os.getenv("DATABASE_URL", "sqlite://")
    SQLALCHEMY_TRACK_MODIFICATIONS = False
```

Here, the database is configured based on the `DATABASE_URL` environment variable that we just defined. Take note of the default value.

Update *__init__.py* to pull in the config on init:

```
from flask import Flask, jsonify


app = Flask(__name__)
app.config.from_object("project.config.Config")


@app.route("/")
def hello_world():
    return jsonify(hello="world")
```

Add [Flask-SQLAlchemy](https://flask-sqlalchemy.palletsprojects.com/) and [Psycopg2](http://initd.org/psycopg/) to *requirements.txt*:

```
Flask==1.1.1
Flask-SQLAlchemy==2.4.1
psycopg2-binary==2.8.4
```

Update *__init__.py* again to create a new `SQLAlchemy` instance and define a database model:

```
from flask import Flask, jsonify
from flask_sqlalchemy import SQLAlchemy


app = Flask(__name__)
app.config.from_object("project.config.Config")
db = SQLAlchemy(app)


class User(db.Model):
    __tablename__ = "users"

    id = db.Column(db.Integer, primary_key=True)
    email = db.Column(db.String(128), unique=True, nullable=False)
    active = db.Column(db.Boolean(), default=True, nullable=False)

    def __init__(self, email):
        self.email = email


@app.route("/")
def hello_world():
    return jsonify(hello="world")
```

Finally, update *manage.py*:

```
from flask.cli import FlaskGroup

from project import app, db


cli = FlaskGroup(app)


@cli.command("create_db")
def create_db():
    db.drop_all()
    db.create_all()
    db.session.commit()


if __name__ == "__main__":
    cli()
```

This registers a new command, `create_db`, to the CLI so that we can run it from the command line, which we'll use shortly to apply the model to the database.

Build the new image and spin up the two containers:

```
$ docker-compose up -d --build
```

Create the table:

```
$ docker-compose exec web python manage.py create_db
```

> Get the following error?
>
> ```
> sqlalchemy.exc.OperationalError: (psycopg2.OperationalError)
> FATAL:  database "hello_flask_dev" does not exist
> ```
>
> Run `docker-compose down -v` to remove the volumes along with the containers. Then, re-build the images, run the containers, and apply the migrations.

Ensure the `users` table was created:

```
$ docker-compose exec db psql --username=hello_flask --dbname=hello_flask_dev

psql (12.2)
Type "help" for help.

hello_flask_dev=# \l
                                        List of databases
      Name       |    Owner    | Encoding |  Collate   |   Ctype    |      Access privileges
-----------------+-------------+----------+------------+------------+-----------------------------
 hello_flask_dev | hello_flask | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres        | hello_flask | UTF8     | en_US.utf8 | en_US.utf8 |
 template0       | hello_flask | UTF8     | en_US.utf8 | en_US.utf8 | =c/hello_flask             +
                 |             |          |            |            | hello_flask=CTc/hello_flask
 template1       | hello_flask | UTF8     | en_US.utf8 | en_US.utf8 | =c/hello_flask             +
                 |             |          |            |            | hello_flask=CTc/hello_flask
(4 rows)

hello_flask_dev=# \c hello_flask_dev
You are now connected to database "hello_flask_dev" as user "hello_flask".

hello_flask_dev=# \dt
          List of relations
 Schema | Name  | Type  |    Owner
--------+-------+-------+-------------
 public | users | table | hello_flask
(1 row)

hello_flask_dev=# \q
```

You can check that the volume was created as well by running:

```
$ docker volume inspect flask-on-docker_postgres_data
```

You should see something similar to:

```
[
    {
        "CreatedAt": "2020-02-24T13:39:47Z",
        "Driver": "local",
        "Labels": {
            "com.docker.compose.project": "flask-on-docker",
            "com.docker.compose.version": "1.24.1",
            "com.docker.compose.volume": "postgres_data"
        },
        "Mountpoint": "/var/lib/docker/volumes/flask-on-docker_postgres_data/_data",
        "Name": "flask-on-docker_postgres_data",
        "Options": null,
        "Scope": "local"
    }
]
```

Next, add an *entrypoint.sh* file to the "web" directory to verify that Postgres is up *and* healthy *before* creating the database table and running the Flask development server:

```
#!/bin/sh

if [ "$DATABASE" = "postgres" ]
then
    echo "Waiting for postgres..."

    while ! nc -z $SQL_HOST $SQL_PORT; do
      sleep 0.1
    done

    echo "PostgreSQL started"
fi

python manage.py create_db

exec "$@"
```

Take note of the environment variables.

Update the file permissions locally:

```
$ chmod +x services/web/entrypoint.sh
```

Then, update the Dockerfile to install [Netcat](http://netcat.sourceforge.net/), copy over the *entrypoint.sh* file, and run the file as the Docker [entrypoint](https://docs.docker.com/engine/reference/builder/#entrypoint) command:

```
# pull official base image
FROM python:3.8.1-slim-buster

# set work directory
WORKDIR /usr/src/app

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# install system dependencies
RUN apt-get update && apt-get install -y netcat

# install dependencies
RUN pip install --upgrade pip
COPY ./requirements.txt /usr/src/app/requirements.txt
RUN pip install -r requirements.txt

# copy project
COPY . /usr/src/app/

# run entrypoint.sh
ENTRYPOINT ["/usr/src/app/entrypoint.sh"]
```

Add the `SQL_HOST`, `SQL_PORT`, and `DATABASE` environment variables, for the *entrypoint.sh* script, to *.env.dev*:

```
FLASK_APP=project/__init__.py
FLASK_ENV=development
DATABASE_URL=postgresql://hello_flask:hello_flask@db:5432/hello_flask_dev
SQL_HOST=db
SQL_PORT=5432
DATABASE=postgres
```

Test it out again:

1. Re-build the images
2. Run the containers
3. Try http://localhost:5000/

Let's also add a CLI seed command for adding sample users to the `users` table in *manage.py*:

```
from flask.cli import FlaskGroup

from project import app, db, User


cli = FlaskGroup(app)


@cli.command("create_db")
def create_db():
    db.drop_all()
    db.create_all()
    db.session.commit()


@cli.command("seed_db")
def seed_db():
    db.session.add(User(email="michael@mherman.org"))
    db.session.commit()


if __name__ == "__main__":
    cli()
```

Try it out:

```
$ docker-compose exec web python manage.py seed_db

$ docker-compose exec db psql --username=hello_flask --dbname=hello_flask_dev

psql (12.0)
Type "help" for help.

hello_flask_dev=# \c hello_flask_dev
You are now connected to database "hello_flask_dev" as user "hello_flask".

hello_flask_dev=# select * from users;
 id |        email        | active
----+---------------------+--------
  1 | michael@mherman.org | t
(1 row)

hello_flask_dev=# \q
```

> Despite adding Postgres, we can still create an independent Docker image for Flask by not setting the `DATABASE_URL` environment variable. To test, build a new image and then run a new container:
>
> ```
> $ docker build -f ./services/web/Dockerfile -t hello_flask:latest ./services/web
> $ docker run -p 5001:5000 \
>     -e "FLASK_APP=project/__init__.py" -e "FLASK_ENV=development" \
>     hello_flask python /usr/src/app/manage.py run -h 0.0.0.0
> ```
>
> You should be able to view the hello world sanity check at [http://localhost:5001](http://localhost:5001/).

### Gunicorn

Moving along, for production environments, let's add [Gunicorn](https://gunicorn.org/), a production-grade WSGI server, to the requirements file:

```
Flask==1.1.1
Flask-SQLAlchemy==2.4.1
gunicorn==20.0.4
psycopg2-binary==2.8.4
```

Since we still want to use Flask's built-in server in development, create a new compose file called *docker-compose.prod.yml* for production:

```
version: '3.7'

services:
  web:
    build: ./services/web
    command: gunicorn --bind 0.0.0.0:5000 manage:app
    ports:
      - 5000:5000
    env_file:
      - ./.env.prod
    depends_on:
      - db
  db:
    image: postgres:12-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    env_file:
      - ./.env.prod.db

volumes:
  postgres_data:
```

> If you have multiple environments, you may want to look at using a [docker-compose.override.yml](https://docs.docker.com/compose/extends/) configuration file. With this approach, you'd add your base config to a *docker-compose.yml* file and then use a *docker-compose.override.yml* file to override those config settings based on the environment.

Take note of the default `command`. We're running Gunicorn rather than the Flask development server. We also removed the volume from the `web` service since we don't need it in production. Finally, we're using [separate environment variable files](https://docs.docker.com/compose/env-file/) to define environment variables for both services that will be passed to the container at runtime.

*.env.prod*:

```
FLASK_APP=project/__init__.py
FLASK_ENV=production
DATABASE_URL=postgresql://hello_flask:hello_flask@db:5432/hello_flask_prod
SQL_HOST=db
SQL_PORT=5432
DATABASE=postgres
```

*.env.prod.db*:

```
POSTGRES_USER=hello_flask
POSTGRES_PASSWORD=hello_flask
POSTGRES_DB=hello_flask_prod
```

Add the two files to the project root. You'll probably want to keep them out of version control, so add them to a *.gitignore* file.

Bring [down](https://docs.docker.com/compose/reference/down/) the development containers (and the associated volumes with the `-v` flag):

```
$ docker-compose down -v
```

Then, build the production images and spin up the containers:

```
$ docker-compose -f docker-compose.prod.yml up -d --build
```

Verify that the `hello_flask_prod` database was created along with the `users` table. Test out http://localhost:5000/.

> Again, if the container fails to start, check for errors in the logs via `docker-compose -f docker-compose.prod.yml logs -f`.

### Production Dockerfile

Did you notice that we're still running the `create_db` command, which drops all existing tables and then creates the tables from the models, every time the container is run? This is fine in development, but let's create a new entrypoint file for production.

*entrypoint.prod.sh*:

```
#!/bin/sh

if [ "$DATABASE" = "postgres" ]
then
    echo "Waiting for postgres..."

    while ! nc -z $SQL_HOST $SQL_PORT; do
      sleep 0.1
    done

    echo "PostgreSQL started"
fi

exec "$@"
```

> Alternatively, instead of creating a new entrypoint file, you could alter the existing one like so:
>
> ```
> #!/bin/sh
> 
> if [ "$DATABASE" = "postgres" ]
> then
>     echo "Waiting for postgres..."
> 
>     while ! nc -z $SQL_HOST $SQL_PORT; do
>       sleep 0.1
>     done
> 
>     echo "PostgreSQL started"
> fi
> 
> if [ "$FLASK_ENV" = "development" ]
> then
>     echo "Creating the database tables..."
>     python manage.py create_db
>     echo "Tables created"
> fi
> 
> exec "$@"
> ```

Update the file permissions locally:

```
$ chmod +x services/web/entrypoint.prod.sh
```

To use this file, create a new Dockerfile called *Dockerfile.prod* for use with production builds:

```
###########
# BUILDER #
###########

# pull official base image
FROM python:3.8.1-slim-buster as builder

# set work directory
WORKDIR /usr/src/app

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# install system dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc

# lint
RUN pip install --upgrade pip
RUN pip install flake8
COPY . /usr/src/app/
RUN flake8 --ignore=E501,F401 .

# install python dependencies
COPY ./requirements.txt .
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /usr/src/app/wheels -r requirements.txt


#########
# FINAL #
#########

# pull official base image
FROM python:3.8.1-slim-buster

# create directory for the app user
RUN mkdir -p /home/app

# create the app user
RUN addgroup -S app && adduser -S app -G app

# create the appropriate directories
ENV HOME=/home/app
ENV APP_HOME=/home/app/web
RUN mkdir $APP_HOME
WORKDIR $APP_HOME

# install dependencies
RUN apt-get update && apt-get install -y --no-install-recommends netcat
COPY --from=builder /usr/src/app/wheels /wheels
COPY --from=builder /usr/src/app/requirements.txt .
RUN pip install --upgrade pip
RUN pip install --no-cache /wheels/*

# copy entrypoint-prod.sh
COPY ./entrypoint.prod.sh $APP_HOME

# copy project
COPY . $APP_HOME

# chown all the files to the app user
RUN chown -R app:app $APP_HOME

# change to the app user
USER app

# run entrypoint.prod.sh
ENTRYPOINT ["/home/app/web/entrypoint.prod.sh"]
```

Here, we used a Docker [multi-stage build](https://docs.docker.com/develop/develop-images/multistage-build/) to reduce the final image size. Essentially, `builder` is a temporary image that's used for building the Python wheels. The wheels are then copied over to the final production image and the `builder` image is discarded.

> You could take the [multi-stage build approach](https://stackoverflow.com/a/53101932/1799408) a step further and use a single *Dockerfile* instead of creating two Dockerfiles. Think of the pros and cons of using this approach over two different files.

Did you notice that we created a non-root user? By default, Docker runs container processes as root inside of a container. This is a bad practice since attackers can gain root access to the Docker host if they manage to break out of the container. If you're root in the container, you'll be root on the host.

Update the `web` service within the *docker-compose.prod.yml* file to build with *Dockerfile.prod*:

```
web:
  build:
    context: ./services/web
    dockerfile: Dockerfile.prod
  command: gunicorn --bind 0.0.0.0:5000 manage:app
  ports:
    - 5000:5000
  env_file:
    - ./.env.prod
  depends_on:
    - db
```

Try it out:

```
$ docker-compose -f docker-compose.prod.yml down -v
$ docker-compose -f docker-compose.prod.yml up -d --build
$ docker-compose -f docker-compose.prod.yml exec web python manage.py create_db
```

### Nginx

Next, let's add Nginx into the mix to act as a [reverse proxy](https://www.nginx.com/resources/glossary/reverse-proxy-server/) for Gunicorn to handle client requests as well as serve up static files.

Add the service to *docker-compose.prod.yml*:

```
nginx:
  build: ./services/nginx
  ports:
    - 1337:80
  depends_on:
    - web
```

Then, in the "services" directory, create the following files and folders:

```
└── nginx
    ├── Dockerfile
    └── nginx.conf
```

*Dockerfile*:

```
FROM nginx:1.17-alpine

RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d
```

*nginx.conf*:

```
upstream hello_flask {
    server web:5000;
}

server {

    listen 80;

    location / {
        proxy_pass http://hello_flask;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }

}
```

> Review [How to Configure NGINX for a Flask Web Application](https://www.patricksoftwareblog.com/how-to-configure-nginx-for-a-flask-web-application/) for more info on configuring Nginx to work with Flask.

Then, update the `web` service, in *docker-compose.prod.yml*, replacing `ports` with `expose`:

```
web:
  build:
    context: ./services/web
    dockerfile: Dockerfile.prod
  command: gunicorn --bind 0.0.0.0:5000 manage:app
  expose:
    - 5000
  env_file:
    - ./.env.prod
  depends_on:
    - db
```

Now, port 5000 is only exposed internally, to other Docker services. The port will no longer be published to the host machine.

> For more on ports vs expose, review [this](https://stackoverflow.com/questions/40801772/what-is-the-difference-between-docker-compose-ports-vs-expose) Stack Overflow question.

Test it out again:

```
$ docker-compose -f docker-compose.prod.yml down -v
$ docker-compose -f docker-compose.prod.yml up -d --build
$ docker-compose -f docker-compose.prod.yml exec web python manage.py create_db
```

Ensure the app is up and running at [http://localhost:1337](http://localhost:1337/).

Your project structure should now look like:

```
├── .env.dev
├── .env.prod
├── .env.prod.db
├── .gitignore
├── docker-compose.prod.yml
├── docker-compose.yml
└── services
    ├── nginx
    │   ├── Dockerfile
    │   └── nginx.conf
    └── web
        ├── Dockerfile
        ├── Dockerfile.prod
        ├── entrypoint.prod.sh
        ├── entrypoint.sh
        ├── manage.py
        ├── project
        │   ├── __init__.py
        │   └── config.py
        └── requirements.txt
```

Bring the containers down once done:

```
$ docker-compose -f docker-compose.prod.yml down -v
```

Since Gunicorn is an application server, it will not serve up static files. So, how should both static and media files be handled in this particular configuration?

### Static Files

Start by creating the following files and folders in the "services/web/project" folder:

```
└── static
    └── hello.txt
```

Add some text to *hello.txt*:

```
hi!
```

Add a new route handler to *__init__.py*:

```
@app.route("/static/<path:filename>")
def staticfiles(filename):
    return send_from_directory(app.config["STATIC_FOLDER"], filename)
```

Don't forget to import [send_from_directory](https://flask.palletsprojects.com/api/#flask.send_from_directory):

```
from flask import Flask, jsonify, send_from_directory
```

Finally, add the `STATIC_FOLDER` config to *services/web/project/config.py*

```
import os


basedir = os.path.abspath(os.path.dirname(__file__))


class Config(object):
    SQLALCHEMY_DATABASE_URI = os.getenv("DATABASE_URL", "sqlite://")
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    STATIC_FOLDER = f"{os.getenv('APP_FOLDER')}/project/static"
```

#### Development

Add the `APP_FOLDER` environment variable to *.env.dev*:

```
FLASK_APP=project/__init__.py
FLASK_ENV=development
DATABASE_URL=postgresql://hello_flask:hello_flask@db:5432/hello_flask_dev
SQL_HOST=db
SQL_PORT=5432
DATABASE=postgres
APP_FOLDER=/usr/src/app
```

To test, first re-build the images and spin up the new containers per usual. Once done, ensure http://localhost:5000/static/hello.txt serves up the file correctly.

#### Production

For production, add a volume to the `web` and `nginx` services in *docker-compose.prod.yml* so that each container will share a directory named "static":

```
version: '3.7'

services:
  web:
    build:
      context: ./services/web
      dockerfile: Dockerfile.prod
    command: gunicorn --bind 0.0.0.0:5000 manage:app
    volumes:
      - static_volume:/home/app/web/project/static
    expose:
      - 5000
    env_file:
      - ./.env.prod
    depends_on:
      - db
  db:
    image: postgres:12-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    env_file:
      - ./.env.prod.db
  nginx:
    build: ./services/nginx
    volumes:
      - static_volume:/home/app/web/project/static
    ports:
      - 1337:80
    depends_on:
      - web

volumes:
  postgres_data:
  static_volume:
```

Next, update the Nginx configuration to route static file requests to the "static" folder:

```
upstream hello_flask {
    server web:5000;
}

server {

    listen 80;

    location / {
        proxy_pass http://hello_flask;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }

    location /static/ {
        alias /home/app/web/project/static/;
    }

}
```

Add the `APP_FOLDER` environment variable to *.env.prod*:

```
FLASK_APP=project/__init__.py
FLASK_ENV=production
DATABASE_URL=postgresql://hello_flask:hello_flask@db:5432/hello_flask_prod
SQL_HOST=db
SQL_PORT=5432
DATABASE=postgres
APP_FOLDER=/home/app/web
```

> Where does this directory path come from? Compare this path to the path added to *.env.dev*. Why do they do they differ?

Spin down the development containers:

```
$ docker-compose down -v
```

Test:

```
$ docker-compose -f docker-compose.prod.yml up -d --build
```

Again, requests to `http://localhost:1337/static/*` will be served from the "static" directory.

Navigate to http://localhost:1337/static/hello.txt and ensure the static asset is loaded correctly.

You can also verify in the logs -- via `docker-compose -f docker-compose.prod.yml logs -f` -- that requests to the static files are served up successfully via Nginx:

```
nginx_1  | 172.23.0.1 - - [24/Feb/2020:17:01:24 +0000] "GET /static/hello.txt HTTP/1.1" 200 4 "-"
           "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:73.0) Gecko/20100101 Firefox/73.0" "-"
```

Bring the containers once done:

```
$ docker-compose -f docker-compose.prod.yml down -v
```

### Media Files

To test out the handling of user-uploaded media files, add two new route handlers to *__init__.py*:

```
@app.route("/media/<path:filename>")
def mediafiles(filename):
    return send_from_directory(app.config["MEDIA_FOLDER"], filename)


@app.route("/upload", methods=["GET", "POST"])
def upload_file():
    if request.method == "POST":
        file = request.files["file"]
        filename = secure_filename(file.filename)
        file.save(os.path.join(app.config["MEDIA_FOLDER"], filename))
    return f"""
    <!doctype html>
    <title>upload new File</title>
    <form action="" method=post enctype=multipart/form-data>
      <p><input type=file name=file><input type=submit value=Upload>
    </form>
    """
```

Update the imports as well:

```
import os

from werkzeug.utils import secure_filename
from flask import (
    Flask,
    jsonify,
    send_from_directory,
    request,
    redirect,
    url_for
)
from flask_sqlalchemy import SQLAlchemy
```

Add the `MEDIA_FOLDER` config to *services/web/project/config.py*:

```
import os


basedir = os.path.abspath(os.path.dirname(__file__))


class Config(object):
    SQLALCHEMY_DATABASE_URI = os.getenv("DATABASE_URL", "sqlite://")
    SQLALCHEMY_TRACK_MODIFICATIONS = False
    STATIC_FOLDER = f"{os.getenv('APP_FOLDER')}/project/static"
    MEDIA_FOLDER = f"{os.getenv('APP_FOLDER')}/project/media"
```

Finally, create a new folder called "media" in the "project" folder.

#### Development

Test:

```
$ docker-compose up -d --build
```

You should be able to upload an image at http://localhost:5000/upload, and then view the image at http://localhost:5000/media/IMAGE_FILE_NAME.

#### Production

For production, add another volume to the `web` and `nginx` services:

```
version: '3.7'

services:
  web:
    build:
      context: ./services/web
      dockerfile: Dockerfile.prod
    command: gunicorn --bind 0.0.0.0:5000 manage:app
    volumes:
      - static_volume:/home/app/web/project/static
      - media_volume:/home/app/web/project/media
    expose:
      - 5000
    env_file:
      - ./.env.prod
    depends_on:
      - db
  db:
    image: postgres:12-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    env_file:
      - ./.env.prod.db
  nginx:
    build: ./services/nginx
    volumes:
      - static_volume:/home/app/web/project/static
      - media_volume:/home/app/web/project/media
    ports:
      - 1337:80
    depends_on:
      - web

volumes:
  postgres_data:
  static_volume:
  media_volume:
```

Next, update the Nginx configuration to route media file requests to the "media" folder:

```
upstream hello_flask {
    server web:5000;
}

server {

    listen 80;

    location / {
        proxy_pass http://hello_flask;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }

    location /static/ {
        alias /home/app/web/project/static/;
    }

    location /media/ {
        alias /home/app/web/project/media/;
    }

}
```

Re-build:

```
$ docker-compose down -v

$ docker-compose -f docker-compose.prod.yml up -d --build
$ docker-compose -f docker-compose.prod.yml exec web python manage.py create_db
```

Test it out one final time:

1. Upload an image at http://localhost:1337/upload.
2. Then, view the image at http://localhost:1337/media/IMAGE_FILE_NAME.

### Conclusion

In this tutorial, we walked through how to containerize a Flask application with Postgres for development. We also created a production-ready Docker Compose file that adds Gunicorn and Nginx into the mix to handle static and media files. You can now test out a production setup locally.

In terms of actual deployment to a production environment, you'll probably want to use a:

1. Fully managed database service -- like [RDS](https://aws.amazon.com/rds/) or [Cloud SQL](https://cloud.google.com/sql/) -- rather than managing your own Postgres instance within a container.
2. Non-root user for the `db` and `nginx` services

You can find the code in the [flask-on-docker](https://github.com/testdrivenio/flask-on-docker) repo.