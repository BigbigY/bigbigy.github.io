---
title: 企业级Registry开源项目Harbor
tags: [Go]
categories: 基于k8s的私有容器云从0到1的建设之路
type: "categories"
author: "luck"

---


# Harbor简介
Harbor是一个用于存储和分发Docker镜像的企业级Registry服务器，通过添加一些企业必需的功能特性，例如安全、标识和管理等，扩展了开源Docker Distribution。作为一个企业级私有Registry服务器，Harbor提供了更好的性能和安全。提升用户使用Registry构建和运行环境传输镜像的效率。Harbor支持安装在多个Registry节点的镜像资源复制，镜像全部保存在私有Registry中， 确保数据和知识产权在公司内部网络中管控。另外，Harbor也提供了高级的安全特性，诸如用户管理，访问控制和活动审计等。
- 基于角色的访问控制 - 用户与Docker镜像仓库通过“项目”进行组织管理，一个用户可以对多个镜像仓库在同一命名空间（project）里有不同的权限。
- 镜像复制 - 镜像可以在多个Registry实例中复制（同步）。尤其适合于负载均衡，高可用，混合云和多云的场景。
- 图形化用户界面 - 用户可以通过浏览器来浏览，检索当前Docker镜像仓库，管理项目和命名空间。
- AD/LDAP 支持 - Harbor可以集成企业内部已有的AD/LDAP，用于鉴权认证管理。
- 审计管理 - 所有针对镜像仓库的操作都可以被记录追溯，用于审计管理。
- 国际化 - 已拥有英文、中文、德文、日文和俄文的本地化版本。更多的语言将会添加进来。
- RESTful API - RESTful API 提供给管理员对于Harbor更多的操控, 使得与其它管理软件集成变得更容易。
- 部署简单 - 提供在线和离线两种安装工具， 也可以安装到vSphere平台(OVA方式)虚拟设备。

## 依赖：
- docker 1.10.0+
- docker-compose 1.6.0+

# 安装[docker ce](https://docs.docker.com/engine/installation/linux/centos/)版
```
$ yum install -y yum-utils device-mapper-persistent-data lvm2
$ yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
$ yum -y install docker-ce-17.12.0.ce-1.el7.centos
$ systemctl restart docker
```

# docker-compose install
```
$ curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` &gt; /usr/local/bin/docker-compose
# chmod +x /usr/local/bin/docker-compose
$ docker-compose --version
docker-compose version 1.18.0, build 8dd22a9
```


# 安装步骤
安装步骤归结为以下几点
- 下载安装程序;
- 配置harbor.cfg ;
- 运行install.sh来安装并启动Harbor

1、下载
```
源码：
wget https://github.com/vmware/harbor/archive/v1.3.0.tar.gz
在线：
wget https://storage.googleapis.com/harbor-releases/harbor-online-installer-v1.3.0.tgz
离线：
wget https://storage.googleapis.com/harbor-releases/harbor-offline-installer-v1.3.0.tgz
```

2、配置
```
[root@hz-k8s-harbor-01 harbor]# cat harbor.cfg |grep -v '#'|grep -v '^$'
hostname = harbor.luck.com
ui_url_protocol = https
db_password = password
max_job_workers = 3
customize_crt = on
ssl_cert = /data/cert/luck.com_server.crt
ssl_cert_key = /data/cert/luck.com_server.key
secretkey_path = /data
admiral_url = NA
clair_db_password = password
log_rotate_count = 50
log_rotate_size = 200M
email_identity =
email_server = smtp.exmail.qq.com
email_server_port = 465
email_username = sysplat@luck.com
email_password = Sysplat@2017
email_from = admin <sysplat@luck.com>
email_ssl = true
email_insecure = false
harbor_admin_password = luck@2018
auth_mode = ldap_auth
ldap_url = ldap://ldap.luck.com:389
ldap_searchdn = uid=harbor,ou=connect,dc=luck,dc=com
ldap_search_pwd = harbor@2018
ldap_basedn = dc=luck,dc=com
ldap_uid = cn
ldap_scope = 3
ldap_timeout = 5
self_registration = on
token_expiration = 30
project_creation_restriction = everyone
db_host = 10.0.0.10
db_port = 3306
db_user = root
uaa_endpoint = uaa.mydomain.org
uaa_clientid= id
uaa_clientsecret= secret
uaa_ca_root= /path/to/uaa_ca.pem
```
hostname = 主机名：目标主机的主机名，用于访问UI和注册表服务。它应该是目标机器的IP地址或完全限定的域名（FQDN），例如198.13.48.154或 `hub.ymq.io`。不要使用localhost或127.0.0.1为主机名 - 注册表服务需要由外部客户端访问！
ui_url_protocol = （http或https，默认为http）用于访问UI和令牌/通知服务的协议。如果公证处于启用状态，则此参数必须为https。默认情况下，这是http。
customize_crt = （打开或关闭，默认打开）打开此属性时，准备脚本创建私钥和根证书，用于生成/验证注册表令牌。当由外部来源提供密钥和根证书时，将此属性设置为off
ssl_cert =SSL证书的路径，仅当协议设置为https时才应用
ssl_cert_key = SSL密钥的路径，仅当协议设置为https时才应用

3、修改完配置文件后，在的当前目录执行```./install.sh```，Harbor服务就会根据当期目录下的docker-compose.yml开始下载依赖的镜像，检测并按照顺序依次启动各个服务，Harbor依赖的镜像及启动服务如下
```
$  docker images
REPOSITORY TAG IMAGE ID CREATED SIZE
vmware/harbor-log v1.3.0 ed05c8b25de6 2 months ago 207MB
vmware/harbor-jobservice v1.3.0 7ae840da1da5 2 months ago 196MB
vmware/harbor-ui v1.3.0 eda2ad5c74ae 2 months ago 210MB
vmware/harbor-adminserver v1.3.0 8e14e93725ba 2 months ago 174MB
vmware/harbor-db v1.3.0 04f589bc4146 2 months ago 586MB
vmware/photon 1.0 521eac01cf8e 3 months ago 130MB
vmware/clair v2.0.1-photon 7a633033c5b1 4 months ago 365MB
vmware/postgresql 9.6.5-photon a5c79b0473d9 4 months ago 285MB
vmware/registry 2.6.2-photon c38af846a0da 4 months ago 240MB
vmware/mariadb-photon 10.2.10 eaaae71dea19 4 months ago 586MB
vmware/notary-photon signer-0.5.1 064b309ad822 4 months ago 246MB
vmware/notary-photon server-0.5.1 b8cc51024379 4 months ago 247MB
vmware/nginx-photon 1.11.13 2971c92cc1ae 4 months ago 200MB
vmware/harbor-db-migrator 1.3 6cac2b89f086 4 months ago 1.11GB
$ docker ps
CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES
845859fabed8 vmware/nginx-photon:1.11.13 "nginx -g 'daemon of…" 2 months ago Up 2 months 0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp, 0.0.0.0:4443->4443/tcp nginx
5ae7106fa0fb vmware/harbor-jobservice:v1.3.0 "/harbor/start.sh" 2 months ago Up 2 months (healthy) harbor-jobservice
078c43721426 vmware/harbor-ui:v1.3.0 "/harbor/start.sh" 2 months ago Up 2 months (healthy) harbor-ui
b860f4208f58 vmware/registry:2.6.2-photon "/entrypoint.sh serv…" 2 months ago Up 2 months (healthy) 5000/tcp registry
913116b25e90 vmware/harbor-adminserver:v1.3.0 "/harbor/start.sh" 2 months ago Up 2 months (healthy) harbor-adminserver
183a75f6ee53 vmware/harbor-log:v1.3.0 "/bin/sh -c /usr/loc…" 2 months ago Up 2 months (healthy) 127.0.0.1:1514->10514/tcp harbor-log
```

# 管理harbor
停止：
```
$ sudo docker-compose stop
Stopping nginx ... done
Stopping harbor-jobservice ... done
Stopping harbor-ui ... done
Stopping harbor-db ... done
Stopping registry ... done
Stopping harbor-log ... done
```

启动：
```
$ sudo docker-compose start
Starting log ... done
Starting ui ... done
Starting mysql ... done
Starting jobservice ... done
Starting registry ... done
Starting proxy ... done
```

# 更新配置
要更改Harbour的配置，请先停止现有的Harbour实例并更新harbor.cfg。然后运行prepare脚本来填充配置。最后重新创建并启动Harbour的实例：
```
$ sudo docker-compose down -v
$ vim harbor.cfg
$ sudo prepare
$ sudo docker-compose up -d
```

# 重新安装
删除Harbour的容器，同时保持图像数据和Harbour的数据库文件在文件系统上：
```
$ sudo docker-compose down -v
```
删除Harbour的数据库和图像数据（为了干净的重新安装）：
```
$ rm -r /data/database
$ rm -r /data/registry
```


# 迁移数据/使用外部数据库
如果此时镜像库中已经有了数据，我们需要做一些迁移工作。
attach到harbor db组件的container中，将registry这张表dump到registry.dump文件中：
```
#docker exec -i -t 38fdb9c305b0 bash
```
在db container中：
```
# mysqldump -u root -p --databases registry > registry.dump
```
回到node，将dump文件从container中copy出来：
```
#docker cp 6e1e4b576315:/root/registry.dump ./
```
再mysql login到external Database，将registry.dump文件导入：
```
# mysql -h external_db_ip -P 3306 -u harbor -p
# mysql> source ./registry.dump;
```

删除docker-compose.yml中以下
```
  mysql:
    image: vmware/harbor-db:v1.3.0
    container_name: harbor-db
    restart: always
    volumes:
      - /data/database:/var/lib/mysql:z
    networks:
      - harbor
    env_file:
      - ./common/config/db/env
    depends_on:
      - log
    logging:
      driver: "syslog"
      options:
        syslog-address: "tcp://127.0.0.1:1514"
        tag: "mysql"
```

去掉或者注释到Proxy部分对mysql的依赖：
```
    depends_on:
      #- mysql
      - registry
      - ui
      - log
```

重加载配置启动:
```
docker-compose down -v
./prepare
sudo docker-compose up -d
```

# 修改存储与阿里云oss
```
vim common/templates/registry/config.yml
    #filesystem:
    #    rootdirectory: /storage
    oss:
      accesskeyid: LTAIMzrQ6n
      accesskeysecret: jwGZvryt0AoiUxk0FY0u
      region: oss-cn-hangzhou
      endpoint: oss-cn-hangzhou-internal.aliyuncs.com
      #internal: optional internal endpoint
      bucket: harbor-luck-com
      #encrypt: optional data encryption setting
      #secure: optional ssl setting
      #chunksize: optional size valye
      #rootdirectory: optional root directory
```


# docker 常用命令

进入容器：
```
docker exec -i 69d1 bash
```

删除所有镜像：
```
docker rmi $(docker images -q)
```

# 基本操作
```
$ docker login hub.luck.com
$ docker tag nginx hub.luck.com/library/k8s-test-nginx
$ docker push hub.luck.com/library/k8s-test-nginx
```


# 功能说明
- 项目：新增/删除项目，查看镜像仓库，给项目添加成员、查看操作日志、复制项目等
- 日志：仓库各个镜像create、push、pull等操作日志
- 系统管理 
用户管理：新增/删除用户、设置管理员等
复制管理：新增/删除从库目标、新建/删除/启停复制规则等
配置管理：认证模式、复制、邮箱设置、系统设置等
- 其他设置 
用户设置：修改用户名、邮箱、名称信息
修改密码：修改用户密码
注意：非系统管理员用户登录，只能看到有权限的项目和日志，其他模块不可见。
