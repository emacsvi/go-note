---
title: "备忘docker常用命令"
slug: "docer-cmds"
date: "2018-11-23 01:00:09"
categories:
    - docker
tags:
    - docker
---

docker commands.
<!--more-->

## centos6安装docker

Add the EPEL Repository
Docker is part of Extra Packages for Enterprise Linux (EPEL), which is a community repository of non-standard packages for the RHEL distribution. First, we’ll install the EPEL repository:
```bash
rpm -iUvh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
```

Then, as a matter of best practice, we’ll update our packages:
```bash
yum update -y
```
Installation
Now let’s install Docker by installing the docker-io package:
```bash
yum -y install docker-io
```
Once the installation completes, we’ll need to start the Docker daemon:
```bash
service docker start
```
And finally, and optionally, let’s configure Docker to start when the server boots:
```bash
chkconfig docker on
```
Download a Docker Container
Let’s begin using Docker! Download the centos Docker image:
```docker
docker pull centos
```
Run a Docker Container
Now, to setup a basic centos container with a bash shell, we just run one command. docker run will run a command in a new container, -i attaches stdin and stdout, -t allocates a tty, and we’re using the standard fedora container.
```docker
docker run -i -t centos /bin/bash
```
That’s it! You’re now using a bash shell inside of a centos docker container.

To disconnect, or detach, from the shell without exiting use the escape sequence Ctrl-p + Ctrl-q.

There are many community containers already available, which can be found through a search. In the command below I am searching for the keyword centos:
```docker
docker search centos
```
Be Sociable, Share!

修改`/etc/docker/daemon.json`源为163的：
```json
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```
再重启docker生效：`service docker restart`

## docker常用命令

查看版本
```docker
docker --version #查看版本
docker-compose --version #查看版本
docker-machine --version #查看版本
docker version #查看client和server端版本，并可以查看是否开启体验功能
```

检查
```docker
docker ps # 查看当前正在运行的image实例
docker ps -a #查看所有镜像实例
docker run hello-world #验证docker是否在运行中
docker inspect <task or container>   检查任务或容器
```

镜像操作
```docker
docker build -t <image-name> . #使用当前目录下的Dockerfile构建镜像
docker images #查看镜像
docker image ls -a  显示机器上所有的镜像
docker image rm <image id>      删除指定的镜像
docker image rm $(docker image ls -a -q)  删除所有的镜像
docker rmi [image-id/image-name] #删除指定的镜像，如docker rmi nginx
docker tag <image> <username>/<repository>:<tag> #为自定义的镜像打上tag。如：$docker tag hellopython followtry/demo:latest
docker push <username>/<repository>:<tag> #将自定义的镜像发布到仓库。如：docker push followtry/demo:latest
    上传后访问地址：https://cloud.docker.com/swarm/followtry/repository/docker/followtry/demo/general
docker pull <username>/<repository> #pull自定义的上传上去的镜像。如：$docker pull followtry/demo
docker run username/repository:tag #运行仓库的镜像
```

容器操作
```docker
docker container ls #列出所有运行中的容器
docker container ls -a #列出所有容器，包括未运行的
docker container ls -q     #只列出运行的容器的id集合
docker container stop <hash>  # 优雅停用指定的容器
docker container kill <hash>  #强制关闭指定的容器
docker container rm <hash>    #删除指定的容器
docker container rm $(docker container ls -a -q)  #删除所有的容器
docker run -d -p 8080:80 --name webserver nginx # 运行nginx镜像实例，-d：后台，-p:绑定端口8080到docker的80
docker stop <containerid/container-name> #停止容器webserver
docker start <containerid/container-name> #启动容器webserver
docker port <containerid/container-name> #查看指定容器的端口映射
docker logs -f <containerid/container-name> #查看指定容器的日志
docker top <containerid/container-name>  #查看容器的进程
docker inspect <containerid/container-name> #检查容器的底层信息
docker rm <containerid/container-name> #
```

## 参考文献

- [centos install docker](https://www.liquidweb.com/kb/how-to-install-docker-on-centos-6/)
