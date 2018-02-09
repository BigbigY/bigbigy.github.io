---
title: Senrty错误日志收集系统（Docker篇）
tags: [Senrty]
categories: 日志
type: "categories"
author: "luck"
header-img: "img/home-bg-art.jpg"
---


# 简介
>```Sentry```是一个实时事件日志记录和汇集的日志平台，其专注于错误监控，以及提取一切事后处理所需的信息。他基于Django开发，目的在于帮助开发人员从散落在多个不同服务器上的日志文件里提取发掘异常，方便debug。Sentry由python编写，源码开放，性能卓越，易于扩展，目前著名的用户有Disqus, Path, mozilla, Pinterest等。它分为客户端和服务端，客户端就嵌入在你的应用程序中间，程序出现异常就向服务端发送消息，服务端将消息记录到数据库中并提供一个web节目方便查看。
 ```DSN（Data Source Name）```
当你完成sentry配置的时候，你会得到一个称为“DSN”的值，看起来像一个标准的URL。Sentry 服务支持多用户、多团队、多应用管理，每个应用都对应一个 ```PROJECT_ID```，以及用于身份认证的 ```PUBLIC_KEY 和 SECRET_KEY```。由此组成一个这样的 ```DSN：
'{PROTOCOL}://{PUBLIC_KEY}:{SECRET_KEY}@{HOST}/{PATH}{PROJECT_ID}'
PROTOCOL ```通常会是 http 或者 https，HOST 为 Sentry 服务的主机名和端口，PATH 通常为空。


# 支持的语言与框架
![](http://ocppiicaw.bkt.clouddn.com/sentry/7.png)

# 优点
Sentry是一个集中式日志管理系统。它具备以下优点：
- 多项目，多用户
- 界面友好
- 可以配置异常出发规则，例如发送邮件
- 支持主流语言接口


# 依赖服务
- PostgreSQL
- Redis（最低版本要求是2.8.9，但是2.8.18,3.0或更新版本被推荐）

# 硬件
对于更典型的但仍然相当高的吞吐量设置，只要具有合理的IO（理想情况下为SSD）和大量内存，就可以运行单机。
您需要考虑的主要事项是：
- 事件TTL（您需要多长时间保留历史数据）
- 平均事件吞吐量
- 有多少事件组合在一起（这意味着它们被采样）
在某种程度上，哨兵每天正在处理大约400万个事件。这些数据的大部分存储了90天，占SSD的1.5TB左右。网络和工作者节点是商品（8GB-12GB RAM，便宜的SATA驱动器，8个内核），唯一的两个附加节点是专用的RabbitMQ和Postgres实例（SSD都是12GB-24GB内存）。在理论上，给定一个具有16+核心和SSD的单个高存储器，您可以处理给定数据集的整体。

# 一、安装前
安装Sentry服务器的两种方法。官方推荐的方法是使用Docker，还可以设置传统的 Python环境来安装。
- [通过Docker](https://docs.sentry.io/server/installation/docker/)
- [通过Python](https://docs.sentry.io/server/installation/python/)

# docker install
依赖
- [Docker 1.10+](https://www.docker.com/get-docker)

# docker-compose install
```
curl -L https://github.com/docker/compose/releases/download/1.1.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose 
或者
yum -y install epel-release  #epel扩展源
yum -y install python-setuptools python-pip
pip install -U docker-compose
```

# 二、Docker CE设置存储库
#### 1、安装```yum-utils```，它提供```yum-config-manager```实用程序：
```
$ sudo yum install -y yum-utils
```
#### 2、使用以下命令设置稳定版本库。您始终需要稳定的存储库，即使您也想安装边缘版本。
```
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
#### 3、可选：启用边缘存储库。该存储库包含在上述docker.repo文件中，但默认情况下禁用。您可以在稳定的存储库旁边启用它。
```
$ sudo yum-config-manager --enable docker-ce-edge
```
您可以通过运行 具有该标志的命令来禁用边缘存储库。要重新启用它，请使用该 标志。以下命令禁用边缘存储库。yum-config-manager--disable--enable
``` 
$ sudo yum-config-manager --disable docker-ce-edge
```
# 三、install docker
#### 1、更新yum包索引
```
sudo yum makecache fast
```

#### 2、安装最新版本的Docker
按版本号排序，从低到高
```
$ yum list docker-ce.x86_64  --showdupli
docker-ce.x86_64            17.03.1.ce-1.el7.centos      
docker-ce.x86_64            17.03.0.ce-1.el7.centos
```
#### 3、安装
```
Docker版       命令
Docker CE    sudo yum install docker-ce-<VERSION>  #免费
Docker EE    sudo yum install docker-ee-<VERSION> #收费
```
#### 4、启动
```
$ sudo systemctl start docker
$ sudo systemctl enable docker
```


# 四、配置

#### 1、克隆配置仓库
```
git clone https://github.com/BigbigY/onpremise.git
```
#### 2、本地数据库和哨兵配置目录。这个目录是用postgres绑定的，所以你不会失去状态！
```
mkdir -p data/{sentry,postgres}
```
#### 3、生成密钥、将它添加到```docker-compose.yml```的base作为```SENTRY_SECRET_KEY```
```
$ docker-compose run --rm web config generate-secret-key
Pulling postgres (postgres:9.5)...
9.5: Pulling from library/postgres
6d827a3ef358: Pull complete
...
Digest: sha256:ae6aed7509b24c99d22eea879a999cc5296bf8512cb0ca806710c21e28199258
Status: Downloaded newer image for postgres:9.5
Pulling redis (redis:3.2-alpine)...
3.2-alpine: Pulling from library/redis
627beaf3eaaf: Pull complete
...
Digest: sha256:9cd405cd1ec1410eaab064a1383d0d8854d1eef74a54e1e4a92fb4ec7bdc3ee7
Status: Downloaded newer image for redis:3.2-alpine
Pulling smtp (tianon/exim4:latest)...
latest: Pulling from tianon/exim4
6d827a3ef358: Already exists
62aaebfa0039: Pull complete
...
Digest: sha256:9d129c700b5e3672b62d8e257c30013029e119abc3c79200ee5f37de271f9ebc
Status: Downloaded newer image for tianon/exim4:latest
Pulling memcached (memcached:1.4)...
1.4: Pulling from library/memcached
e45e882ed798: Pull complete
...
Digest: sha256:69d0a53c6f8e9a94afbd0a4765bee47ee4eee4be1c5a91ebca311a2c3265bd01
Status: Downloaded newer image for memcached:1.4
Creating onpremise_redis_1
Creating onpremise_postgres_1
Creating onpremise_memcached_1
Creating onpremise_smtp_1
Building web
Step 1/1 : FROM sentry:8.15-onbuild
8.15-onbuild: Pulling from library/sentry
6d827a3ef358: Already exists
...
92845ff58dca: Pull complete
Digest: sha256:834252ebf977cbbcc6bcb9af4ee868eb3aee83c53c314fa344e9d03ce0894f23
Status: Downloaded newer image for sentry:8.15-onbuild
# Executing 4 build triggers...
Step 1/1 : COPY . /usr/src/sentry
Step 1/1 : RUN if [ -s requirements.txt ]; then pip install -r requirements.txt; fi
---> Running in d51b9144ace4
You must give at least one requirement to install (see "pip help install")
Step 1/1 : RUN if [ -s setup.py ]; then pip install -e .; fi
---> Running in 93778130aa3f
Step 1/1 : RUN if [ -s sentry.conf.py ]; then cp sentry.conf.py $SENTRY_CONF/; fi     && if [ -s config.yml ]; then cp config.yml $SENTRY_CONF/; fi
---> Running in bee839f0f38d
---> 2d9b74b5cc92
...
Removing intermediate container d51b9144ace4
Successfully built 2d9b74b5cc92
WARNING: Image for service web was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
k9#zup^a!px!(8b(&-z8w1-r8-hx_)vkd)9^tld-2q)4udrr8@
```
**查看已经下载到的docker images**
```
[root@bbtree onpremise]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
onpremise_web       latest              2d9b74b5cc92        19 minutes ago      546 MB
postgres            9.5                 f4b5a2e24152        13 days ago         265 MB
sentry              8.15-onbuild        88d88d343a8c        2 weeks ago         546 MB
tianon/exim4        latest              233bc593cf60        4 weeks ago         173 MB
memcached           1.4                 a5cffa71b5af        4 weeks ago         83.7 MB
redis               3.2-alpine          83638a6d3af2        6 weeks ago         19.8 MB
```
**查看运行容器 **
```
[root@bbtree onpremise]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
a37072928f11        tianon/exim4        "entrypoint.sh tin..."   5 minutes ago       Up 5 minutes        25/tcp              onpremise_smtp_1
651b2808a223        redis:3.2-alpine    "docker-entrypoint..."   5 minutes ago       Up 5 minutes        6379/tcp            onpremise_redis_1
6ddd6d01da98        postgres:9.5        "docker-entrypoint..."   5 minutes ago       Up 5 minutes        5432/tcp            onpremise_postgres_1
8a46622efc09        memcached:1.4       "docker-entrypoint..."   5 minutes ago       Up 5 minutes        11211/tcp           onpremise_memcached_1
```

#### 4、构建数据库。使用交互式提示创建用户帐户。
```
$ docker-compose run --rm web upgrade 
......
Would you like to create a user account now? [Y/n]: y
Email: wangyy02@bbtree.com
Password: 
Repeat for confirmation: 
Should this user be a superuser? [y/N]: y
User created: wangyy02@bbtree.com
Added to organization: sentry
- Loading initial data for sentry.
Installed 0 object(s) from 0 fixture(s)
Running migrations for nodestore:
- Migrating forwards to 0001_initial.
> nodestore:0001_initial
- Loading initial data for nodestore.
Installed 0 object(s) from 0 fixture(s)
......
```

#### 5、提升所有服务（分离/背景模式）。
```
$ docker-compose up -d  
[root@bbtree onpremise]# docker-compose up -d
Building base
Step 1/1 : FROM sentry:8.15-onbuild
# Executing 4 build triggers...
Step 1/1 : COPY . /usr/src/sentry
---> Using cache
Step 1/1 : RUN if [ -s requirements.txt ]; then pip install -r requirements.txt; fi
---> Using cache
Step 1/1 : RUN if [ -s setup.py ]; then pip install -e .; fi
---> Using cache
Step 1/1 : RUN if [ -s sentry.conf.py ]; then cp sentry.conf.py $SENTRY_CONF/; fi     && if [ -s config.yml ]; then cp config.yml $SENTRY_CONF/; fi
---> Using cache
---> 2d9b74b5cc92
Successfully built 2d9b74b5cc92
WARNING: Image for service base was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Building worker
Step 1/1 : FROM sentry:8.15-onbuild
# Executing 4 build triggers...
Step 1/1 : COPY . /usr/src/sentry
---> Using cache
Step 1/1 : RUN if [ -s requirements.txt ]; then pip install -r requirements.txt; fi
---> Using cache
Step 1/1 : RUN if [ -s setup.py ]; then pip install -e .; fi
---> Using cache
Step 1/1 : RUN if [ -s sentry.conf.py ]; then cp sentry.conf.py $SENTRY_CONF/; fi     && if [ -s config.yml ]; then cp config.yml $SENTRY_CONF/; fi
---> Using cache
---> 2d9b74b5cc92
Successfully built 2d9b74b5cc92
WARNING: Image for service worker was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
Building cron
Step 1/1 : FROM sentry:8.15-onbuild
# Executing 4 build triggers...
Step 1/1 : COPY . /usr/src/sentry
---> Using cache
Step 1/1 : RUN if [ -s requirements.txt ]; then pip install -r requirements.txt; fi
---> Using cache
Step 1/1 : RUN if [ -s setup.py ]; then pip install -e .; fi
---> Using cache
Step 1/1 : RUN if [ -s sentry.conf.py ]; then cp sentry.conf.py $SENTRY_CONF/; fi     && if [ -s config.yml ]; then cp config.yml $SENTRY_CONF/; fi
---> Using cache
---> 2d9b74b5cc92
Successfully built 2d9b74b5cc92
WARNING: Image for service cron was built because it did not already exist. To rebuild this image you must use `docker-compose build` or `docker-compose up --build`.
onpremise_postgres_1 is up-to-date
onpremise_memcached_1 is up-to-date
onpremise_smtp_1 is up-to-date
onpremise_redis_1 is up-to-date
Creating onpremise_base_1
Creating onpremise_web_1
Creating onpremise_cron_1
Creating onpremise_worker_1
[root@bbtree onpremise]# docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
44dc95ebcd28        onpremise_worker    "/entrypoint.sh ru..."   14 seconds ago      Up 6 seconds        9000/tcp                 onpremise_worker_1
05c768d0bf25        onpremise_cron      "/entrypoint.sh ru..."   14 seconds ago      Up 5 seconds        9000/tcp                 onpremise_cron_1
8d82862dac76        onpremise_web       "/entrypoint.sh ru..."   14 seconds ago      Up 6 seconds        0.0.0.0:9000->9000/tcp   onpremise_web_1
beba10feeaff        onpremise_base      "/entrypoint.sh ru..."   14 seconds ago      Up 10 seconds       9000/tcp                 onpremise_base_1
d6327bdf077d        postgres:9.5        "docker-entrypoint..."   About an hour ago   Up About an hour    5432/tcp                 onpremise_postgres_1
699afc45b48a        tianon/exim4        "entrypoint.sh tin..."   About an hour ago   Up About an hour    25/tcp                   onpremise_smtp_1
885dd16c8666        redis:3.2-alpine    "docker-entrypoint..."   About an hour ago   Up About an hour    6379/tcp                 onpremise_redis_1
77d159bd52a4        memcached:1.4       "docker-entrypoint..."   About an hour ago   Up About an hour    11211/tcp                onpremise_memcached_1
```
#### 6、访问您的实例
```
http://localhost:9000！
```

**登陆**
![](http://ocppiicaw.bkt.clouddn.com/sentry/1.jpg)
 
**设置访问地址及管理员邮箱**
![](http://ocppiicaw.bkt.clouddn.com/sentry/2.jpg)
**进入仪表盘** 
![](http://ocppiicaw.bkt.clouddn.com/sentry/3.png)
 
**新建团队**
![](http://ocppiicaw.bkt.clouddn.com/sentry/4.png)

**新建项目**
![](http://ocppiicaw.bkt.clouddn.com/sentry/5.png)

**待client接入** 
![](http://ocppiicaw.bkt.clouddn.com/sentry/6.png)
 
 

>- 官网：https://docs.sentry.io/
- https://www.postgresql.org/
- https://hub.docker.com/_/postgres/
- https://redis.io/
- https://launchpad.net/~chris-lea/+archive/ubuntu/redis-server
- https://hub.docker.com/_/redis/
- https://docs.docker.com/engine/installation/linux/centos/#install-from-a-package
- http://www.widuu.com/docker/compose/install.html

 