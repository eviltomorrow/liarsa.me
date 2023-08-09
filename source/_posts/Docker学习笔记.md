---
title: Docker 学习笔记
catalog: true
date: 2023-08-08 23:00:00
subtitle: Docker 学习
tags:
    - 容器
    - Docker
categories:
    - 技术
index_img: /img/bg/wallhaven-vqdmxm.png
# banner_img: /img/bg/wallhaven-vqdmxm.png
---


# Docker 学习笔记

## 1、为什么要使用 Docker

- 更高效的利用系统资源
- 更快速的启动时间
- 一致的运行环境
- 持续交付和部署
- 更轻松的迁移
- 更轻松的维护和拓展

## 2、基本概念

### **镜像（Image）**

简介：操作系统分为 **内核** 和 **用户空间**。对于 `Linux` 而言，内核启动后，会挂载 `root` 文件系统为其提供用户空间支持。而 **Docker 镜像**（`Image`），就相当于是一个 `root` 文件系统。**Docker 镜像** 是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）。镜像 **不包含** 任何动态数据，其内容在构建之后也不会被改变。

在 Docker 设计时，就充分利用 [Union FS (opens new window)](https://en.wikipedia.org/wiki/Union_mount)的技术，将其设计为分层存储的架构。镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。

从 Docker 镜像仓库获取镜像的命令是 `docker pull`。其命令格式为：

```sh
$ docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]
```

运行 Docker 镜像。其命令格式为：

```sh
$ docker run -it --rm ubuntu:18.04 bash
```

列出已经下载下来的镜像，可以使用 docker image ls 命令

```sh
$ docker image ls
```

通过 `docker system df` 命令来便捷的查看镜像、容器、数据卷所占用的空间

```sh
$ docker system df
# 虚悬镜像问题，由于 docker pull 和 docker build 等操作导致 image 同版本号取代原来的镜像导致。
# 显示虚悬镜像
$ docker image ls -f dangling=true
# 删除虚悬镜像
$ docker image prune
```

列出特定的镜像

```sh
$ docker image ls ubuntu
$ docker image ls -f since=mongo:3.2
# -f 代表 -filter 的简称。
```

删除本地的镜像，可以使用 `docker image rm` 命令，其格式为：

```sh
$ docker image rm [选项] <镜像1> [<镜像2> ...]
# <镜像> 可以是 镜像短 ID、镜像长 ID、镜像名 或者 镜像摘要
```

Docker 提供了一个 `docker commit` 命令，可以将容器的存储层保存下来成为镜像。换句话说，就是在原有镜像的基础上，再叠加上容器的存储层，并构成新的镜像。

```sh
$ docker commit [选项] <容器ID或容器名> [<仓库名>[:<标签>]]
```

通过 Dockerfile 构建镜像，其内包含了一条条的 **指令(Instruction)**，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。`docker build` 命令进行镜像构建。

```sh
$ docker build [选项] <上下文路径/URL/->
# 在 Dockfile 目录下执行这个命令， 上下文路径中的文件都会上传到 Daemon 中
```

保存镜像

```sh
$ docker save alpine -o filename
# docker save alpine | gzip > alpine-latest.tar.gz
```

加载镜像

```sh
$ docker load -i alpine-latest.tar.gz
```

### 容器（Container）

容器的实质是进程，但与直接在宿主执行的进程不同，容器进程运行于属于自己的独立的 [命名空间 (opens new window)](https://en.wikipedia.org/wiki/Linux_namespaces)。因此容器可以拥有自己的 `root` 文件系统、自己的网络配置、自己的进程空间，甚至自己的用户 ID 空间。容器内的进程是运行在一个隔离的环境里，使用起来，就好像是在一个独立于宿主的系统下操作一样。

每一个容器运行时，是以镜像为基础层，在其上创建一个当前容器的存储层，我们可以称这个为容器运行时读写而准备的存储层为 容器存储层。容器存储层的生存周期和容器一样，容器消亡时，容器存储层也随之消亡。因此，任何保存于容器存储层的信息都会随容器删除而丢失。

- 启动容器

  启动容器有两种方式，一种是基于镜像新建一个容器并启动，另外一个是将在终止状态（`exited`）的容器重新启动。

```sh
# 所需要的命令主要为 docker run
$ docker run ubuntu:18.04 /bin/echo 'Hello world'
$ docker run -t -i ubuntu:18.04 /bin/bash
-t 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， -i 则让容器的标准输入保持打开。
```

当利用 `docker run` 来创建容器时，Docker 在后台运行的标准操作包括：

1. 检查本地是否存在指定的镜像，不存在就从 [registry](https://vuepress.mirror.docker-practice.com/repository/) 下载
2. 利用镜像创建并启动一个容器
3. 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
4. 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
5. 从地址池配置一个 ip 地址给容器
6. 执行用户指定的应用程序
7. 执行完毕后容器被终止

- 启动已终止容器

  可以利用 `docker container start` 命令，直接将一个已经终止（`exited`）的容器启动运行。

- 后台运行

- 需要让 Docker 在后台运行而不是直接把执行命令的结果输出在当前宿主机下。此时，可以通过添加 `-d` 参数来实现。

  ```sh
  $ docker run -d ubuntu:18.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
  77b2dc01fe0f3f1265df143181e7b9af5e05279a884f4776ee75350ea9d8017a
  ```

  **注：** 容器是否会长久运行，是和 `docker run` 指定的命令有关，和 `-d` 参数无关。要获取容器的输出信息，可以通过 `docker container logs` 命令。

- 终止容器

  使用 `docker container stop` 来终止一个运行中的容器。

  ```sh
  $ docker container ls -a
  ```

- 进入容器

  使用 `docker attach` 命令或 `docker exec` 命令，推荐大家使用 `docker exec` 命令。（attach 在 exit 后停止容器，exec 不会）

- 删除容器

  使用 `docker container rm` 来删除一个处于终止状态的容器。

### 仓库（Repository）

一个 Docker Registry 中可以包含多个 仓库（Repository）；每个仓库可以包含多个 标签（Tag）；每个标签对应一个镜像。通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 <仓库名>:<标签> 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest 作为默认标签。

## 3、Dockerfile 简介

### FROM 指定镜像基础

在`Dockerfile` 中 `FROM` 是必备的指令，并且必须是第一条指令。(特殊的镜像，名为 `scratch`。它表示一个空白的镜像。)

```sh
FROM <镜像名称>
# 样例
FROM scratch
```

### RUN 执行命令

`RUN` 指令是用来执行命令行命令的。(Shell 格式和 Exec 格式)

```sh
RUN <命令>
RUN ["可执行文件", "参数1", "参数2"]
# 样例
RUN set -x; buildDeps='gcc libc6-dev make wget' \
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



### COPY 复制文件

指令将从构建上下文目录中 <源路径> 的文件/目录复制到新的一层的镜像内的 <目标路径> 位置，使用 COPY 指令，源文件的各种元数据都会保留。比如读、写、执行权限、文件变更时间等。

```sh
COPY [--chown=<user>:<group>] <源路径>... <目标路径>
COPY [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]
# <目标路径> 可以是容器内的绝对路径，也可以是相对于工作目录的相对路径（工作目录可以用 WORKDIR 指令来指定）。目标路径不需要事先创建，如果目录不存在会在复制文件前先行创建缺失目录。
# 样例
COPY package.json /usr/src/app/
COPY hom* /mydir/
COPY --chown=55:mygroup files* /mydir/
```

### ADD 更高级的复制文件

`ADD` 指令和 `COPY` 的格式和性质基本一致。但是在 `COPY` 基础上增加了一些功能。

```sh
ADD [--chown=<user>:<group>] <源路径>... <目标路径>
ADD [--chown=<user>:<group>] ["<源路径1>",... "<目标路径>"]
# add 可以说是 copy 命令的增强版，源路径可以为 URL 或 TAR 压缩文件，ADD 则包含了更复杂的功能，其行为也不一定很清晰。最适合使用 ADD 的场合，就是所提及的需要自动解压缩的场合。同时，ADD 指令会令镜像构建缓存失效，从而可能会令镜像构建变得比较缓慢。
```

### CMD 容器启动命令

`CMD` 指令的格式和 `RUN` 相似，也是两种格式：

```sh
CMD <命令>
CMD ["可执行文件", "参数1", "参数2"...]
# 样例
CMD echo $HOME
docker run -it ubuntu cat /etc/os-release
```

### ENTRYPOINT 入口点

`ENTRYPOINT` 的格式和 `RUN` 指令格式一样，分为 `exec` 格式和 `shell` 格式。`ENTRYPOINT` 的目的和 `CMD` 一样，都是在指定容器启动程序及参数。

```sh
# 当指定了 ENTRYPOINT 后，CMD 的含义就发生了改变，不再是直接的运行其命令，而是将 CMD 的内容作为参数传给 ENTRYPOINT 指令
<ENTRYPOINT> "<CMD>"
# 场景 1 
FROM ubuntu:18.04
RUN apt-get update \
    && apt-get install -y curl \
    && rm -rf /var/lib/apt/lists/*
ENTRYPOINT [ "curl", "-s", "http://myip.ipip.net" ]
$ docker run myip curl -s http://myip.ipip.net -i

# 场景 2
FROM alpine:3.4
...
RUN addgroup -S redis && adduser -S -G redis redis
...
ENTRYPOINT ["docker-entrypoint.sh"]

EXPOSE 6379
CMD [ "redis-server" ]

#!/bin/sh
...
# allow the container to be started with `--user`
if [ "$1" = 'redis-server' -a "$(id -u)" = '0' ]; then
	find . \! -user redis -exec chown redis '{}' +
	exec gosu redis "$0" "$@"
fi
exec "$@"
```

### ENV 设置环境变量

下列指令可以支持环境变量展开： `ADD`、`COPY`、`ENV`、`EXPOSE`、`FROM`、`LABEL`、`USER`、`WORKDIR`、`VOLUME`、`STOPSIGNAL`、`ONBUILD`、`RUN`

```sh
ENV <key> <value>
ENV <key1>=<value1> <key2>=<value2>...
# 样例
ENV VERSION=1.0 DEBUG=on \
    NAME="Happy Feet"
```

### ARG 构建参数

构建参数和 `ENV` 的效果一样，都是设置环境变量。所不同的是，`ARG` 所设置的构建环境的环境变量，在将来容器运行时是不会存在这些环境变量的。但是不要因此就使用 `ARG` 保存密码之类的信息，因为 `docker history` 还是可以看到所有值的。ARG 指令有生效范围，如果在 `FROM` 指令之前指定，那么只能用于 `FROM` 指令中。

```sh
ARG <参数名>[=<默认值>]
# 样例
# 只在 FROM 中生效
ARG DOCKER_USERNAME=library
FROM ${DOCKER_USERNAME}/alpine
# 要想在 FROM 之后使用，必须再次指定
ARG DOCKER_USERNAME=library
RUN set -x ; echo ${DOCKER_USERNAME}
```

### VOLUME 定义匿名卷

事先指定某些目录挂载为匿名卷，这样在运行时如果用户不指定挂载，其应用也可以正常运行，不会向容器存储层写入大量数据。

```sh
VOLUME ["<路径1>", "<路径2>"...]
VOLUME <路径>
# 样例
VOLUME /data
```

### EXPOSE 声明端口

`EXPOSE` 指令是声明容器运行时提供服务的端口，这只是一个声明，在容器运行时并不会因为这个声明应用就会开启这个端口的服务。在 Dockerfile 中写入这样的声明有两个好处，一个是帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射；另一个用处则是在运行时使用随机端口映射时，也就是 `docker run -P` 时，会自动随机映射 `EXPOSE` 的端口。

```sh
EXPOSE <端口1> [<端口2>...]
# 样例
docker run -it -p <宿主端口>:<容器端口> --rm nginx
```

### WORKDIR 指定工作目录

使用 `WORKDIR` 指令可以来指定工作目录（或者称为当前目录），以后各层的当前目录就被改为指定的目录，如该目录不存在，`WORKDIR` 会帮你建立目录。

```sh
WORKDIR <工作目录路径>
# 样例
WORKDIR /app
RUN echo "hello" > world.txt
```

### USER 指定当前用户

`USER` 指令和 `WORKDIR` 相似，都是改变环境状态并影响以后的层。`WORKDIR` 是改变工作目录，`USER` 则是改变之后层的执行 `RUN`, `CMD` 以及 `ENTRYPOINT` 这类命令的身份。注意，`USER` 只是帮助你切换到指定用户而已，这个用户必须是事先建立好的，否则无法切换。

```sh
USER <用户名>[:<用户组>]
# 样例
RUN groupadd -r redis && useradd -r -g redis redis
USER redis
RUN [ "redis-server" ]
```

### HEALTHCHECK 健康检查

`HEALTHCHECK` 指令是告诉 Docker 应该如何进行判断容器的状态是否正常。

```sh
HEALTHCHECK [选项] CMD <命令>
HEALTHCHECK NONE
# 样例
FROM nginx
RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*
HEALTHCHECK --interval=5s --timeout=3s \
  CMD curl -fs http://localhost/ || exit 1
```

### LABEL 指令

`LABEL` 指令用来给镜像以键值对的形式添加一些元数据（metadata）

```sh
LABEL <key>=<value> <key>=<value> <key>=<value> ...
# 样例
LABEL org.opencontainers.image.documentation="https://yeasy.gitbooks.io"
```

### SHELL 指令

指令可以指定 `RUN` `ENTRYPOINT` `CMD` 指令的 shell，Linux 中默认为 `["/bin/sh", "-c"]`

```sh
SHELL ["/bin/sh", "-c"]
RUN lll ; ls
SHELL ["/bin/sh", "-cex"]
RUN lll ; ls
```

### ONBUILD 为他人做嫁衣裳

`ONBUILD` 是一个特殊的指令，它后面跟的是其它指令，比如 `RUN`, `COPY` 等，而这些指令，在当前镜像构建时并不会被执行。只有当以当前镜像为基础镜像，去构建下一级镜像的时候才会被执行。

```sh
ONBUILD <其它指令>
```

### 多阶段构建

为了解决源码丢失风险，镜像臃肿问题，和使用 shell 脚本构建的复杂性问题

```sh
FROM golang:alpine as builder
RUN apk --no-cache add git
WORKDIR /go/src/github.com/go/helloworld/
RUN go get -d -v github.com/go-sql-driver/mysql
COPY app.go .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .
FROM alpine:latest as prod
RUN apk --no-cache add ca-certificates
WORKDIR /root/
COPY --from=0 /go/src/github.com/go/helloworld/app .
CMD ["./app"]
# 样例
# 我们可以使用 as 来为某一阶段命名
FROM golang:alpine as builder
$ docker build --target builder -t username/imagename:tag .
```

## 4、数据管理

在容器中管理数据主要有两种方式：

- 数据卷（Volumes）
- 挂载主机目录 (Bind mounts)

### 数据卷

`数据卷` 是一个可供一个或多个容器使用的特殊目录，它绕过 UFS，可以提供很多有用的特性：

- `数据卷` 可以在容器之间共享和重用
- 对 `数据卷` 的修改会立马生效
- 对 `数据卷` 的更新，不会影响镜像
- `数据卷` 默认会一直存在，即使容器被删除

创建一个数据卷

```sh
$ docker volume create my-vol
```

查看所有的数据卷

```sh
$ docker volume ls
```

启动一个挂载数据卷的容器

```sh
$ docker run -d -P \
    --name web \
    # -v my-vol:/usr/share/nginx/html \
    --mount source=my-vol,target=/usr/share/nginx/html \
    nginx:alpine
```

删除数据卷

```sh
$ docker volume rm my-vol
```

### 挂载主机目录

使用 `--mount` 标记可以指定挂载一个本地主机的目录到容器中去。

```sh
$ docker run -d -P \
    --name web \
    # -v /src/webapp:/usr/share/nginx/html \
    --mount type=bind,source=/src/webapp,target=/usr/share/nginx/html \
    nginx:alpine
```

## 5、Docker 中的网络

Docker 允许通过外部访问容器或容器互联的方式来提供网络服务。

### 外部访问容器

容器中可以运行一些网络应用，要让外部也可以访问这些应用，可以通过 `-P` 或 `-p` 参数来指定端口映射。

`-p` 则可以指定要映射的端口，并且，在一个指定端口上只可以绑定一个容器。支持的格式有 `ip:hostPort:containerPort | ip::containerPort | hostPort:containerPort`。

```sh
$ docker run -d -p 127.0.0.1:80:80/udp nginx:alpine
```

`-p` 标记可以多次使用来绑定多个端口

```sh
$ docker run -d \
    -p 80:80 \
    -p 443:443 \
    nginx:alpine
```

### 容器互联

将容器加入自定义的 Docker 网络来连接多个容器

新建网络

```sh
 docker network create -d bridge my-net
```

连接容器

```sh
$ docker run -it --rm --name busybox1 --network my-net busybox sh
```