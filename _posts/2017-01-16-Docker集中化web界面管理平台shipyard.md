---
title: Docker集中化web界面管理平台shipyard
tags: [docker,shipyar]
categories: 微服务与容器
type: "categories"
author: "luck"
header-img: "img/contact-bg.jpg"
---

# shipyard Dashboard #

官方文档：https://shipyard-project.com/docs/deploy/manual/

Shipyard（github）是建立在docker集群管理工具Citadel之上的可以管理容器、主机等资源的web图形化工具。包括core和extension两个版本，core即shipyard主要是把多个 Docker host上的 containers 统一管理（支持跨越多个host），extension即shipyard-extensions添加了应用路由和负载均衡、集中化日志、部署等。

# 一、几个概念 #

## engine ##
一个shipyard管理的docker集群可以包含一个或多个engine（引擎），一个engine就是监听tcp端口的docker daemon。shipyard管理docker daemon、images、containers完全基于Docker API，不需要做其他的修改。另外，shipyard可以对每个engine做资源限制，包括CPU和内存；因为TCP监听相比Unix socket方式会有一定的安全隐患，所以shipyard还支持通过SSL证书与docker后台进程安全通信。
 
## rethinkdb ##
RethinkDB是一个shipyard项目的一个docker镜像，用来存放账号（account）、引擎（engine）、服务密钥（service key）、扩展元数据（extension metadata）等信息，但不会存储任何有关容器或镜像的内容。一般会启动一个shipyard/rethinkdb容器shipyard-rethinkdb-data来使用它的/data作为数据卷供另外rethinkdb一个挂载，专门用于数据存储。

# 二、搭建过程 #

修改tcp监听

Shipyard 要管理和控制 Docker host 的话需要先修改 Docker host 上的默认配置使其监听tcp端口(可以继续保持Unix socket）。有以下2种方式

1、docker -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock -d 启动docker daemon。如果为了避免每次启动都写这么长的命令，可以直接在/etc/init/docker.conf中修改。

2、修改/etc/default/docker的DOCKER_OPTS，在里面加入以下：
-H tcp://0.0.0.0:2375 -H unix://var/run/docker.sock

3、重启

    systemctl restart docker

4、Datastore

Shipyard使用RethinkDB的数据存储

    $docker run \
    -ti \
    -d \
    --restart=always \
    --name shipyard-rethinkdb \
    rethinkdb

5、Discovery
使用etcd做 key/value为后端做支持

    docker run \
    -ti \
    -d \
    -p 4001:4001 \
    -p 7001:7001 \
    --restart=always \
    --name shipyard-discovery \
    microbox/etcd -name discovery

6、Proxy

因为前边已经手动在配置文件添加的-H tcp://0.0.0.0:235 -H unix://var/run/docker.sock，所以此处我并没有执行以下命令

    $ docker run \
    -ti \
    -d \
    -p 2375:2375 \
    --hostname=$HOSTNAME \
    --restart=always \
    --name shipyard-proxy \
    -v /var/run/docker.sock:/var/run/docker.sock \
    -e PORT=2375 \
    shipyard/docker-proxy:latest

7、Swarm Manager

运行配置管理容器

    $docker run \
    -ti \
    -d \
    --restart=always \
    --name shipyard-swarm-manager \
    swarm:latest \
    manage --host tcp://0.0.0.0:3375 etcd://<IP-OF-HOST>:4001

8、Swarm Agent

    $ docker run \
    -ti \
    -d \
    --restart=always \
    --name shipyard-swarm-agent \
    swarm:latest \
    join --addr <ip-of-host>:2375 etcd://<ip-of-host>:4001

9、Controller

运行Shipyard Controller

    $ docker run \
    -ti \
    -d \
    --restart=always \
    --name shipyard-controller \
    --link shipyard-rethinkdb:rethinkdb \
    --link shipyard-swarm-manager:swarm \
    -p 8080:8080 \
    shipyard/shipyard:latest \
    server \
    -d tcp://swarm:3375

10、通过浏览器访问http://host:8080来访问shipyard UI界面

第一次run后，关闭再次启动时直接使用：

    sudo docker stop shipyard shipyard-rethinkdb shipyard-rethinkdb-data
    sudo docker start shipyard-rethinkdb-data shipyard-rethinkdb shipyard

访问http://ip:8080   默认用户名/密码为 admin/shipyard

![](http://ocppiicaw.bkt.clouddn.com/docker/docker-shipyard1.png)

容器管理
![](http://ocppiicaw.bkt.clouddn.com/docker/docker-shipyard2.png)

测试跑tomcat:

1、下载镜像

![](http://ocppiicaw.bkt.clouddn.com/docker/docker-shipyard3.png)

![](http://ocppiicaw.bkt.clouddn.com/docker/docker-shipyard4.png)

2、部署
![](http://ocppiicaw.bkt.clouddn.com/docker/docker-shipyard9.png)

3、运行
![](http://ocppiicaw.bkt.clouddn.com/docker/docker-shipyard6.png)

![](http://ocppiicaw.bkt.clouddn.com/docker/docker-shipyard7.png)

4、访问tomcat
![](http://ocppiicaw.bkt.clouddn.com/docker/docker-shipyard8.png)
