[docker菜鸟教程](https://www.runoob.com/docker/docker-tutorial.html)

Docker 架构
Docker 包括三个基本概念:

- 镜像（Image）：Docker 镜像（Image），就相当于是一个 root 文件系统。比如官方镜像 ubuntu:16.04 就包含了完整的一套 Ubuntu16.04 最小系统的 root 文件系统。Docker 镜像是用于创建 Docker 容器的模板，比如 Ubuntu 系统。

- 容器（Container）：镜像（Image）和容器（Container）的关系，容器是独立运行的一个或一组应用，是镜像运行时的实体, 就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。

- 仓库（Repository）：仓库可看着一个代码控制中心，用来保存镜像。
Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。
Docker Hub(https://hub.docker.com) 提供了庞大的镜像集合供使用。
一个 Docker Registry 中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。
通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 <仓库名>:<标签> 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 latest 作为默认标签。

[官方docker hub](https://hub.docker.com/)

![docker文件系统](https://wstimg-dev.oss-cn-hangzhou.aliyuncs.com/upload/2e1cc6a2.png)


## docker安装 （以CentOs为例）
卸载旧版本
```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```
设置镜像
```
$ sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```
```
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
安装
```
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

## 修改docker镜像
### Ubuntu14.04、Debian7Wheezy
对于使用 upstart 的系统而言，编辑 /etc/default/docker 文件，在其中的 DOCKER_OPTS 中配置加速器地址：
DOCKER_OPTS="--registry-mirror=https://registry.docker-cn.com"
重新启动服务:
```
$ sudo service docker restart
```

### Ubuntu16.04+、Debian8+、CentOS7
对于使用 systemd 的系统，请在 /etc/docker/daemon.json 中写入如下内容（如果文件不存在请新建该文件）：
{"registry-mirrors":["https://registry.docker-cn.com"]}
之后重新启动服务：
```
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

### Windows 10
对于使用 Windows 10 的系统，在系统右下角托盘 Docker 图标内右键菜单选择 Settings，打开配置窗口后左侧导航菜单选择 Daemon。在 Registrymirrors 一栏中填写加速器地址 https://registry.docker-cn.com ，之后点击 Apply 保存后 Docker 就会重启并应用配置的镜像地址了。

### Mac OS X
对于使用 Mac OS X 的用户，在任务栏点击 Docker for mac 应用图标-> Perferences...-> Daemon-> Registrymirrors。在列表中填写加速器地址 https://registry.docker-cn.com 。修改完成之后，点击 Apply&Restart 按钮，Docker 就会重启并应用配置的镜像地址了。

### docker国内镜像：
```
mirrors = {
    "azure": "http://dockerhub.azk8s.cn",
    "tencent": "https://mirror.ccs.tencentyun.com",
    "netease": "https://hub-mirror.c.163.com",
    "ustc": "https://docker.mirrors.ustc.edu.cn",
    "qiniu": "https://reg-mirror.qiniu.com",
    "docker": "https://registry.docker-cn.com",
    "aliyun": "https://78brq6cn.mirror.aliyuncs.com" 
}
```

> 阿里云的Docker加速器
- 阿里云 - [开发者平台](https://dev.aliyun.com/)
- 阿里云 - [容器Hub服务控制台](https://cr.console.aliyun.com/)


## Docker 容器使用

### 获取容器镜像
```
$ docker pull ubuntu
```

### 运行交互式的容器
```
docker run -i -t ubuntu:15.10 /bin/bash

root@0123ce188bd8:/#
```
参数说明:
- -t: 在新容器内指定一个伪终端或终端。
- -i: 允许你对容器内的标准输入 (STDIN) 进行交互。
- -d:让容器在后台运行。
- -P:将容器内部使用的网络端口映射到我们使用的主机上。

```
cat /proc/version
exit
```

### 启动容器（后台模式）
```
docker run -d ubuntu:15.10 /bin/sh -c "while true; do echo hello world; sleep 1; done"
2b1b7a428627c51ab8810d541d759f072b4fc75487eed05812646b8534a2fe63
docker ps
docker logs 2b1b7a428627
docker logs amazing_cori
```

### 停止容器
```
docker stop amazing_cori
```

### 进入容器
```
docker exec -it 243c32535da7 /bin/bash
```

### 导出和导入容器
```
docker export 1e560fca3906 > ubuntu.tar
cat docker/ubuntu.tar | docker import - test/ubuntu:v1
```

### 删除容器
```
docker rm -f 1e560fca3906
```



## Docker 镜像使用

> image其实就是一个文件系统，它与宿主机的内核一起为程序提供一个虚拟的linux环境。在启动docker container时，依据image，docker会为container构建出一个虚拟的linux环境。
![image文件系统](https://www.aboutyun.com/data/attachment/album/201512/29/171913e46fjq641zrlxq4q.png)

### 创建镜像
```
docker ps -l 
docker commit -m="has update" -a="peter" e218edb10161 linkenpeng/ubuntu:v2
```

### 删除镜像
```
docker rmi golang:1.11-alpine
```

### 构建镜像
Dockerfile
```
FROM    centos:6.7
MAINTAINER      Fisher "collin_linken@qq.com"

RUN     /bin/echo 'root:123456' |chpasswd
RUN     useradd peter
RUN     /bin/echo 'peter:123456' |chpasswd
RUN     /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
EXPOSE  22
EXPOSE  80
CMD     /usr/sbin/sshd -D
```

构建
```
docker build -t centos:6.7 .
```

参数说明：
- -t ：指定要创建的目标镜像名
- . ：Dockerfile 文件所在目录，可以指定Dockerfile 的绝对路径


## 容器使用
### 网络端口映射
```
docker run -d -P training/webapp python app.py
```
- -P : 是容器内部端口随机映射到主机的高端口。
- -p : 是容器内部端口绑定到指定的主机端口。
```
docker run -d -p 5000:5000 training/webapp python app.py
docker run -d -p 127.0.0.1:5001:5000 training/webapp python app.py
docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py
docker run -d -P --name runoob training/webapp python app.py
```

### 新建网络
```
$ docker network create -d bridge test-net
```

### 连接容器
```
$ docker run -itd --name test1 --network test-net ubuntu /bin/bash
$ docker run -itd --name test2 --network test-net ubuntu /bin/bash
进入test1 可以ping通test2
进入test2 可以ping通test1
```

### 配置 DNS
在宿主机的 /etc/docker/daemon.json 文件中增加以下内容来设置全部容器的 DNS：
```
{
  "dns" : [
    "114.114.114.114",
    "8.8.8.8"
  ]
}
```

> 设置后，启动容器的 DNS 会自动配置为 114.114.114.114 和 8.8.8.8。
配置完，需要重启 docker 才能生效。
查看容器的 DNS 是否生效可以使用以下命令，它会输出容器的 DNS 信息：

```
$ docker run -it --rm ubuntu  cat etc/resolv.conf
手动指定
$ docker run -it --rm host_ubuntu  --dns=114.114.114.114 --dns-search=test.com ubuntu
```



## Docker 仓库管理
```
$ docker login
$ docker logout
$ docker search ubuntu
$ docker pull ubuntu
$ docker image ls
$ docker tag ubuntu:18.04 username/ubuntu:18.04
$ docker push username/ubuntu:18.04
```



## Docker Dockerfile
### FROM 和 RUN 指令的作用
- FROM：定制的镜像都是基于 FROM 的镜像，这里的 nginx 就是定制需要的基础镜像。后续的操作都是基于 nginx。
- RUN：用于执行后面跟着的命令行命令。有以下俩种格式：
 

shell 格式：
```
RUN <命令行命令>
# <命令行命令> 等同于，在终端操作的 shell 命令。
```

exec 格式：
```
RUN ["可执行文件", "参数1", "参数2"]
# 例如：
# RUN ["./test.php", "dev", "offline"] 等价于 RUN ./test.php dev offline
```

> 注意：Dockerfile 的指令每执行一次都会在 docker 上新建一层。所以过多无意义的层，会造成镜像膨胀过大。例如：

```
ROM centos
RUN yum install wget
RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz"
RUN tar -xvf redis.tar.gz
以上执行会创建 3 层镜像。可简化为以下格式：
FROM centos
RUN yum install wget \
    && wget -O redis.tar.gz "http://download.redis.io/releases/redis-5.0.3.tar.gz" \
    && tar -xvf redis.tar.gz

```

### 构建镜像
```
$docker build -t nginx:test .
```

> 最后一个 . 是上下文路径，那么什么是上下文路径呢？

上下文路径，是指 docker 在构建镜像，有时候想要使用到本机的文件（比如复制），docker build 命令得知这个路径后，会将路径下的所有内容打包。

### 指令详解
```
COPY 复制指令，从上下文目录中复制文件或者目录到容器里指定路径。

ADD 指令和 COPY 的使用格式一致（同样需求下，官方推荐使用 COPY）。

CMD 类似于 RUN 指令，用于运行程序，但二者运行的时间点不同:
CMD 在docker run 时运行。
RUN 是在 docker build。

ENTRYPOINT
类似于 CMD 指令，但其不会被 docker run 的命令行参数指定的指令所覆盖，而且这些命令行参数会被当作参数送给 ENTRYPOINT 指令指定的程序。

ENV 设置环境变量，定义了环境变量，那么在后续的指令中，就可以使用这个环境变量。

ARG
构建参数，与 ENV 作用一至。不过作用域不一样。ARG 设置的环境变量仅对 Dockerfile 内有效，也就是说只有 docker build 的过程中有效，构建好的镜像内不存在此环境变量。

VOLUME
定义匿名数据卷。在启动容器时忘记挂载数据卷，会自动挂载到匿名卷。
作用：
避免重要的数据，因容器重启而丢失，这是非常致命的。
避免容器不断变大。

EXPOSE
仅仅只是声明端口。
作用：
帮助镜像使用者理解这个镜像服务的守护端口，以方便配置映射。
在运行时使用随机端口映射时，也就是 docker run -P 时，会自动随机映射 EXPOSE 的端口。

WORKDIR
指定工作目录。用 WORKDIR 指定的工作目录，会在构建镜像的每一层中都存在。（WORKDIR 指定的工作目录，必须是提前创建好的）。

USER
用于指定执行后续命令的用户和用户组，这边只是切换后续命令执行的用户（用户和用户组必须提前已经存在）。

HEALTHCHECK
用于指定某个程序或者指令来监控 docker 容器服务的运行状态。

ONBUILD
用于延迟构建命令的执行。简单的说，就是 Dockerfile 里用 ONBUILD 指定的命令，在本次构建镜像的过程中不会执行（假设镜像为 test-build）。
```


## 构建实例
### 构建jdk8
1. 下载jdk-8u201-linux-x64.tar.gz
2. 制作Dockerfile

jdk官方
```
FROM centos:centos7
MAINTAINER peter
RUN mkdir /usr/local/jdk
WORKDIR /usr/local/jdk
ADD jdk-8u201-linux-x64.tar.gz /usr/local/jdk

ENV JAVA_HOME /usr/local/jdk/jdk1.8.0_201
ENV JRE_HOME /usr/local/jdk/jdk1.8.0_201/jre
ENV PATH $JAVA_HOME/bin:$PATH
```

openjdk [openjdk下载](https://adoptopenjdk.net/releases.html)
```
FROM centos:centos7.7.1908
MAINTAINER linkenpeng
RUN mkdir /usr/local/jdk && \
    ln -s /usr/local/jdk/jdk8u242-b08-jre /usr/local/jdk/jre
WORKDIR /usr/local/jdk
ADD OpenJDK8U-jre_x64_linux_hotspot_8u242b08.tar.gz /usr/local/jdk

ENV JAVA_HOME /usr/local/jdk/jre
ENV PATH ${PATH}:${JAVA_HOME}/bin
```

3. 启动
```
sudo docker run -di --name=jdk1.8 jdk1.8
```
4. 进入容器
```
$ sudo docker exec -it jdk1.8 /bin/bash
```

5. 创建镜像并发布：
```
:~$ docker commit -m="centos7.7.1908 openjdk jdk8u242-b08-jre" -a="peter" 4a5ac8fbeba0 linkenpeng/centos7.7-openjdk8-jre:1.2

:~$ docker push linkenpeng/centos7.7-openjdk8-jre:1.2
```

### 登录私服：
```
[admin@k8s-master ~]$ sudo docker login reg.was.ink
sudo docker tag linkenpeng/centos7.7-openjdk8-jre:1.2 reg.was.ink/mcom/javabase-centos-openjdk:1.0 
[admin@k8s-master ~]$ sudo docker push reg.was.ink/mcom/javabase-centos-openjdk:1.0
```

## Docker Compose
> Compose 是用于定义和运行多容器 Docker 应用程序的工具。通过 Compose，您可以使用 YML 文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从 YML 文件配置中创建并启动所有服务。

Compose 使用的三个步骤：

- 使用 Dockerfile 定义应用程序的环境。
- 使用 docker-compose.yml 定义构成应用程序的服务，这样它们可以在隔离环境中一起运行。
- 最后，执行 docker-compose up 命令来启动并运行整个应用程序。

```
# yaml 配置实例
version: '3'
services:
  web:
    build: .
    ports:
   - "5000:5000"
    volumes:
   - .:/code
    - logvolume01:/var/log
    links:
   - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```



## Docker Machine
> Docker Machine 是一种可以让您在虚拟主机上安装 Docker 的工具，并可以使用 docker-machine 命令来管理主机。
Docker Machine 也可以集中管理所有的 docker 主机，比如快速的给 100 台服务器安装上 docker。

[Docker常用命令](https://docs.docker.com/engine/reference/commandline/build/)
