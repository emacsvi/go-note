---
title: "备忘docker-compose部署命令"
slug: "docker-compose-deploy"
date: "2018-11-23 01:00:25"
categories:
    - docker
tags:
    - docker
    - docker-compose
---

# 写在前面
在区块链开发中，有很多很麻烦的环境需要部署。这时候docker容器就派上了用场。因为有很多参数，有很多不同的版本，也有go,python,nodejs,leveldb等不同的语言环境，这时候用docker你会发现特别开心。

但是有个问题是很多应用有依赖关系，这时候你需要docker的编排工具。而且有时候需要跨多个主机来运行，需要docker的管理工具。唉，还有负载均衡，日志管理等，发现还不是我想的那么简单。想想还是算了，我暂时就用docker-compose来编排一下吧。其他的暂时不考虑。

考虑那么多一直在选择，还没有走出去看看遇到什么问题呢，这怎么行。

我需要的功能如下：
- 将所有我的每一个应用打包成**image**镜像放到公网云上，由**docker-compose**下载使用。这样更新也方便。
- 所有的image需要的配置文件，映射的目录与端口等都由**cfg.json**这样的配置文件来定。
- 在**docker-compose**中尽量的少做docker里面的配置工作。让编排工作与镜像尽量的保持独立性。

# 操作镜像

保存，加载镜像命令：

```bash
docker save imageID > filename
docker load < filename

# 或者这样
docker save -o ubuntu_14.04.tar ubuntu:14.04
docker load --input ubuntu_14.04.tar

# 最后修改标签
docker tag  d227774c296x docker_image:18.01.14.1702 
```

[http://blog.51cto.com/wutengfei/2060800](http://blog.51cto.com/wutengfei/2060800)

通过image保存的镜像会保存操作历史，可以回滚到历史版本。

# 所有的镜像

### centosgo
创建一个用来在mac上进行编译go语言的镜像。这样就不用开虚拟机来编译和运行。都可以启动一个很小的docker容器则可以：

```docker
# version: 0.0.1
# author: liwei
FROM hub.c.163.com/public/centos:7.2.1511
MAINTAINER liwei liweilijie@gmail.com
ENV REFRESHED_AT 2018-06-29

# 数据目录
COPY go.tar.gz /root/
COPY profile /root/
RUN tar -C /usr/local -xzf /root/go.tar.gz
# 设置环境变量
# go setting
ENV GOROOT /usr/local/go
ENV GOBIN $GOROOT/bin
ENV GOPKG $GOROOT/pkg/tool/linux_amd64
ENV GOARCH amd64
ENV GOOS linux
ENV GOPATH /data
ENV PATH $PATH:$GOBIN:$GOPKG:$GOPATH/bin
VOLUME ["/data"]
# 容器启动时要运行的命令
WORKDIR $GOPATH
```

### clsyncdata
这个程序是用来在运维客户端上同步数据库数据的程序。帮php来实现的功能。这是一个很简单的功能。
- 此程序会打印日志出来
- 此程序由一个daemon程序来进行守护。并且日志也由daemon程序来管理。daemon程序的名称叫supervisord由我修改过源码的。

步骤：
- 将二进制文件放入/bin/目录下，打包到程序里面。
- 放入配置文件

```docker
# version: 0.0.1
# author: liwei
FROM hub.c.163.com/public/centos:7.2.1511
MAINTAINER liwei liweilijie@gmail.com
ENV REFRESHED_AT 2018-06-29

# 增加二进制文件到系统
COPY supervisord /bin/
COPY clsyncdata /bin/
# 映射数据目录
VOLUME ["/data"]
# 容器启动时要运行的命令
WORKDIR /data
ENTRYPOINT ["/bin/supervisord", "-d"]
```

接下来生成镜像并且生成容器：

```bash
docker build -t="emacsvicom/centosgo:1.10.3" .
docker run -it --name go -v /Users/liwei/golang:/data emacsvicom/centosgo:1.10.3 /bin/bash
```

这里遇到一个坑：
- 我的supervisord是用来重启clsyncdata与管理其日志文件的daemon程序。这个程序如果在后台启动的话，docker运行不成功。只能将其启动到前台。
- 所以我增加了一个`-d`的传入参数以便启动到前台。

```bash
# 创建镜像
docker build -t="emacsvicom/clsyncdata:v1" .
# 运行程序
docker run -d --name c1 -v $PWD:/data emacsvicom/clsyncdata:v1
```

### srsyncdata

```docker
# version: 0.0.1
# author: liwei
FROM hub.c.163.com/public/centos:7.2.1511
MAINTAINER liwei liweilijie@gmail.com
ENV REFRESHED_AT 2018-06-30

# 增加二进制文件到系统
COPY supervisord /bin/
COPY srsyncdata /bin/
# 映射数据目录
VOLUME ["/data"]
# 容器启动时要运行的命令
WORKDIR /data
# -p 12099:12099
EXPOSE 12099
ENTRYPOINT ["/bin/supervisord", "-d"]
```

### sync
生成docker-compose文件，将clsyncdata 与 srsyncdata部署在一个机器上暂时测试使用：

```yaml
version: '3'
services:
  srsyncdata:
    image: emacsvicom/srsyncdata:v1
    container_name: srsyncdata
    volumes:
      - ./srsyncdata:/data
      - /etc/localtime:/etc/localtime
    ports:
      - '12099:12099'
    restart: always
    extra_hosts:
        - "server.yidai.cc:192.168.253.242"
  clsyncdata:
    image: emacsvicom/clsyncdata:v1
    container_name: clsyncdata
    volumes:
      - ./clsyncdata:/data
      - /etc/localtime:/etc/localtime
    restart: always
    extra_hosts:
        - "server.yidai.cc:192.168.253.242"
```

# 注意事项

## 容器如何访问宿主机的mysql
docker搭建了lnmp环境后，如果需要访问安装在宿主机上的数据库或中间件，是不能直接使用127.0.0.1这个ip的，这个ip在容器中指向容器自己，那么应该怎么去访问宿主机呢：

例如你的docker环境的虚拟IP是192.168.99.100，那么宿主机同样会托管一个和192.168.99.100同网段的虚拟IP，并且会是主IP：192.168.99.1，那么就简单了，在容器中访问192.168.99.1这个地址就等于访问宿主机，问题解决

注意，通过192.168.99.1访问宿主机，等于换了一个ip，如果数据库或中间件限制了本机访问或者做了ip段限制，要记得添加192.168.99.1到白名单


## 容器时间与宿主机时间不对

- 在Dockerfile中加入 `RUN echo "Asia/Shanghai" > /etc/timezone` 设置时区
- 启动的时候挂载宿主机的时间 `-v /etc/localtime:/etc/localtime`，如果是**docker-compose**启动的话，就在里面加**volume**

## docker-compose设置网络

[设置网络](https://blog.alejandrocelaya.com/2017/04/21/set-specific-ip-addresses-to-docker-containers-created-with-docker-compose/)

```yaml
version: '3'

services:
    test_1:
        container_name: test_1
        image: some:image
        networks:
            testing_net:
                ipv4_address: 172.28.1.1

    test_2:
        container_name: test_2
        image: some:image
        networks:
            testing_net:
                ipv4_address: 172.28.1.2

    test_3:
        container_name: test_3
        image: some:image
        networks:
            testing_net:
                ipv4_address: 172.28.1.3

networks:
    testing_net:
        ipam:
            driver: default
            config:
                - subnet: 172.28.0.0/16
```

或者这样：

```yaml
version: '3'
services:
  eth-stats:
    image: quay.io/amis/ethstats:latest
    ports:
      - '3000:3000'
    environment:
      - WS_SECRET=bb98a0b6442386d0cdf8a31b267892c1
    restart: always
    networks:
      app_net:
        ipv4_address: 172.16.239.9
  validator-0:
    hostname: validator-0
    image: quay.io/amis/geth:latest
    ports:
      - '30303:30303'
      - '8545:8545'
    networks:
      app_net:
        ipv4_address: 172.16.239.10
    restart: always
  validator-1:
    hostname: validator-1
    image: quay.io/amis/geth:latest
    ports:
      - '30304:30303'
      - '8546:8545'
    networks:
      app_net:
        ipv4_address: 172.16.239.11
    restart: always
  validator-2:
    hostname: validator-2
    image: quay.io/amis/geth:latest
    ports:
      - '30305:30303'
      - '8547:8545'
    networks:
      app_net:
        ipv4_address: 172.16.239.12
    restart: always
  validator-3:
    hostname: validator-3
    image: quay.io/amis/geth:latest
    ports:
      - '30306:30303'
      - '8548:8545'

    networks:
      app_net:
        ipv4_address: 172.16.239.13
    restart: always
networks:
  app_net:
    driver: bridge
    ipam:
      driver: default
      config:
      - subnet: 172.16.239.0/24
```



### nodejs问题

在部署nodejs的环境的时候，在我们平常的操作中有如下几个步骤：

1. 下载nodejs的源代码。
2. 下载完成之后`npm install`安装package.json里面项目依赖的包，该依赖包全安装在本地的`node_modules`目录下面。
3. 最后再修改config.json的配置文件，比如端口号,`ip`地址等参数信息。
4. 再`npm start` 启动该服务。

这时候其实有个问题，如果我们在开发node项目的时候，需要不停的修改源文件，那每次都开启一个docker container而且每次都要npm install岂不是很老火。而这里需要明白一个挂载目录的问题。

* 与 volumes 相比， bind mounts 有一些功能限制。挂载的时候，bind mounts需要指定`宿主机`上的`源路径`必须是**绝对路径**。
* 在使用 volume 时，如果镜像中的挂载点目录原本就有文件，那么这些文件会复制到 volume 中。但是bind mount不会。**这一点很重要**。

```docker
FROM hub.c.163.com/public/ubuntu:16.04-tools

MAINTAINER liwei liweilijie@gmail.com

# 安装mongodb
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 7F0CEB10 && \
    echo 'deb http://downloads-distro.mongodb.org/repo/ubuntu-upstart dist 10gen' \
      | tee /etc/apt/sources.list.d/10gen.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends mongodb-org && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# 安装nodejs npm
RUN apt-get update
RUN apt-get -qq update
RUN apt-get install -y nodejs npm
RUN update-alternatives --install /usr/bin/node node /usr/bin/nodejs 10

# 安装nodejs的依赖包node_modules
RUN mkdir /ndata
COPY ./data/package.json /ndata/
RUN npm config set registry https://registry.npm.taobao.org
RUN cd /ndata && npm install

# 安装supervisord
COPY ./data/supervisord /bin/
#RUN mv /ndata/supervisord /bin/

RUN mkdir /rdata
VOLUME ["/rdata", "/data/db"]
WORKDIR /rdata
EXPOSE 27017 28017 3000

# 将supervisord放在rdata里面，修改supervisord.conf文件来运行mongodb和npm start两个程序
ENTRYPOINT ["/bin/supervisord", "-d"]
#CMD ["sh", "/ndata/run.sh"]

```

然后再创建docker-compose.yml来进行对各个目录的映射关系。

* mgo\_data是在上面用到的mongodb数据目录/data/db
* rdata是runData的数据目录，里面是对supervisord.conf以及各个log文件查看的配置。
* ndata/node\_modules则是一个volumes的目录，在上面build容器的时候会npm install创建node\_modules数据目录。这时候挂载数据卷的时候就会将以前的文件内容复制到里面，以达到数据共享的目的。
* [nodejs的node\_modues问题](https://stackoverflow.com/questions/30043872/docker-compose-node-modules-not-present-in-a-volume-after-npm-install-succeeds)

```yaml
version: '3'
services:
  explorer_nodejs:
    build: .
    volumes:
      - ./mgo_data:/data/db
      - ./rdata:/rdata
      - ./data:/ndata
      - /ndata/node_modules
    ports:
      - '13000:3000'
    restart: always

```


## 环境变量

```yaml
environment:
  - PGDATA=/var/lib/postgresql/data/pgdata
  - POSTGRES_PASSWORD=development
```

## 参考文献

- [存储数据卷](https://docs-cn.docker.octowhale.com/engine/admin/volumes/)
