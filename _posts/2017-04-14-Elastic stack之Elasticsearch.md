---
title: 4-Elastic stack之Elasticsearch
tags: [Elastic stack,Elasticsearch]
categories: 自动化运维工具
type: "categories"
author: "luck"
header-img: "img/post-bg-e2e-ux.jpg"
---

# 简介
>Elasticsearch是一个基于Apache Lucene(TM)的开源搜索引擎。无论在开源还是专有领域，Lucene可以被认为是迄今为止最先进、性能最好的、功能最全的搜索引擎库。
但是，Lucene只是一个库。想要使用它，你必须使用Java来作为开发语言并将其直接集成到你的应用中，更糟糕的是，Lucene非常复杂，你需要深入了解检索的相关知识来理解它是如何工作的。
Elasticsearch也使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单。
不过，Elasticsearch不仅仅是Lucene和全文搜索，我们还能这样去描述它：
- 分布式的实时文件存储，每个字段都被索引并可被搜索
- 分布式的实时分析搜索引擎
- 可以扩展到上百台服务器，处理PB级结构化或非结构化数据
 而且，所有的这些功能被集成到一个服务里面，你的应用可以通过简单的RESTful API、各种语言的客户端甚至命令行与之交互。

# 一、安装
#### 1、安装依赖环境
>elastic需要Java 8

#### 2、卸载自带jdk，安装java环境：
```
$ rpm -qa | grep java
java-1.6.0-openjdk-1.6.0.41-1.13.13.1.el6_8.x86_64
tzdata-java-2017a-1.el6.noarch
java-1.7.0-openjdk-1.7.0.131-2.6.9.0.el6_8.x86_64
$ rpm -e --nodeps java-1.6.0-openjdk-1.6.0.41-1.13.13.1.el6_8.x86_64
$ rpm -e --nodeps tzdata-java-2017a-1.el6.noarch
$ rpm -e --nodeps java-1.7.0-openjdk-1.7.0.131-2.6.9.0.el6_8.x86_64
```

#### 3、 修改环境变量
```
$ tar xf jdk-8u45-linux-x64.tar.gz -C /usr/
$ vim /etc/profile
JAVA_HOME=/usr/jdk1.8.0_45
JRE_HOME=/usr/jdk1.8.0_45/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
CLASSPATH=:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export JAVA_HOME JRE_HOME PATH CLASSPATH
```

#### 4、测试 
```
$ java -version
java version "1.8.0_45"
Java(TM) SE Runtime Environment (build 1.8.0_45-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.45-b02, mixed mode)
```
#### 4、导入弹性搜索PGP密钥编辑
```
$ rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```
#### 6、下载并安装公共签名密钥：
```
$ vim /etc/yum.repos.d/elasticsearch.repo
[elasticsearch-5.x]
name=Elasticsearch repository for 5.x packages
baseurl=https://artifacts.elastic.co/packages/5.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```
#### 7、yum install
```
$ sudo yum install elasticsearch
$ /usr/share/elasticsearch/bin/elasticsearch --version
Version: 5.3.0, Build: 3adb13b/2017-03-23T03:31:50.652Z, JVM: 1.8.0_45

```

# 二、配置 Elasticsearch

#### 1、配置文件位置

Elasticsearch有两个配置文件：
- elasticsearch.yml 用于配置Elasticsearch
- log4j2.properties 用于配置Elasticsearch日志
这些文件位于配置目录中，其位置默认为 ```$ES_HOME/config/``` Debian和RPM软件包将config目录位置设置为```/etc/elasticsearch/```

配置目录的位置可以通过path.conf 设置更改，如下所示：

```
./bin/elasticsearch -Epath.conf=/path/to/my/config/
```

#### 2、配置文件格式编辑
配置格式为YAML，以下是更改数据路径和日志目录的示例：
```
$ mkdir -p /data/elasticsearch/{data,logs}
path:
    data: /data/elasticsearch/data
    logs: /data/elasticsearch/logs
或者
path.data: /data/elasticsearch/data
path.logs: /data/elasticsearch/logs
```
#### 3、环境变量
使用配置文件中的${...}符号引用的环境变量将替换为环境变量的值，例如：
```
node.name:    ${HOSTNAME}
network.host: ${ES_NETWORK_HOST}
```

#### 4、设置默认配置

可以使用default.前缀在命令行中指定新的默认设置 。这将指定默认使用的值，除非在配置文件中指定了另一个值。 
例如，如果Elasticsearch启动如下：
```
$ ./bin/elasticsearch -Edefault.node.name=My_Node
```
除非用命令行覆盖配置文件中的或与配置文件中的值，否则node.name将为该值。My_Nodees.node.namenode.name

#### 6、记录配置
Elasticsearch使用Log4j 2进行日志记录。可以使用log4j2.properties文件配置Log4j 2。Elasticsearch公开三个属性```${sys:es.logs.base_path}```， ```${sys:es.logs.cluster_name}```以及```${sys:es.logs.node_name}```（如果节点名称是通过明确设置node.name），可以在配置文件中引用，以确定日志文件的位置。该属性 ```${sys:es.logs.base_path}```将解析为日志目录，```${sys:es.logs.cluster_name}```将解析为群集名称（用作默认配置中日志文件名的前缀）， ```${sys:es.logs.node_name}```并将解析为节点名称（如果节点名称已显示设置）。
 
例如，如果您的日志目录（path.logs）```/var/log/elasticsearch```和您的群集被命名，production那么```${sys:es.logs.base_path}```将解析到```/var/log/elasticsearch```并``` ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}.log ```解决```/var/log/elasticsearch/production.log```。
```
appender.rolling.type = RollingFile #配置RollingFileappender
appender.rolling.name = rolling
appender.rolling.fileName = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}.log #登录 /var/log/elasticsearch/production.log
appender.rolling.layout.type = PatternLayout
appender.rolling.layout.pattern = [%d{ISO8601}][%-5p][%-25c] %.10000m%n
appender.rolling.filePattern = ${sys:es.logs.base_path}${sys:file.separator}${sys:es.logs.cluster_name}-%d{yyyy-MM-dd}.log #滚动日志到 /var/log/elasticsearch/production-yyyy-MM-dd.log
appender.rolling.policies.type = Policies
appender.rolling.policies.time.type = TimeBasedTriggeringPolicy #使用基于时间的滚动策略
appender.rolling.policies.time.interval = 1  #每天滚动日志 
appender.rolling.policies.time.modulate = true #在日边界对齐卷（而不是每二十四小时滚动一次）
```
如果要在特定时间段内保留日志文件，则可以使用带有删除操作的滚动策略。
```
appender.rolling.strategy.type = DefaultRolloverStrategy
appender.rolling.strategy.action.type = Delete
appender.rolling.strategy.action.basepath = ${sys:es.logs.base_path}
appender.rolling.strategy.action.condition.type = IfLastModified
appender.rolling.strategy.action.condition.age = 7D
appender.rolling.strategy.action.PathConditions.type = IfFileName
appender.rolling.strategy.action.PathConditions.glob = ${sys:es.logs.cluster_name}-*
```
解释：
- 配置 DefaultRolloverStrategy
- 配置Delete处理翻转的操作
- Elasticsearch logs基本路径
- 搬运时适用的条件
- 保留日志七天
- 只有删除超过7天的文件，如果它们与指定的glob匹配
- 从与glob匹配的基本路径中删除文件 ```${sys:es.logs.cluster_name}-*```; 这是日志文件滚动到的glob; 这只需要删除滚动的Elasticsearch日志，但也不会删除弃用和缓慢的日志

#### 7、禁用日志记录
```
$ logger.deprecation.level = warn
```
这将在您的日志目录中创建每日滚动弃用日志文件。定期检查这个文件，特别是当你打算升级到一个新的主要版本。
默认日志记录配置已将弃用日志的滚动策略设置为1 GB后滚动和压缩，并保留最多五个日志文件（四个卷轴日志和活动日志）。
您可以config/log4j2.properties通过将弃用日志级别设置为在文件中禁用它error。

# 三、elasticserch集群

#### 1、添加hosts
```
vim /etc/hosts
10.0.13.107 elastic-1
10.0.13.125 elastic-2
```

#### ２、配置修改
```
[root@elastic-1 ~]# sed /#/d /etc/elasticsearch/elasticsearch.yml
cluster.name: es-bbtree-cluster
node.name: elastic-1
path.data: /data/elasticsearch/data
path.logs: /data/elasticsearch/logs
network.host: 10.0.13.126
http.port: 9200
bootstrap.system_call_filter: false
discovery.zen.ping.unicast.hosts: ["10.0.13.126", "10.0.13.125"]
discovery.zen.minimum_master_nodes: 2
－－－－－－－－－－－－－－－－－－－－－－－－
[root@elastic-2 ~]# sed /#/d /etc/elasticsearch/elasticsearch.yml
cluster.name: es-bbtree-cluster
node.name: elastic-2
path.data:  /data/elasticsearch/data
path.logs:  /data/elasticsearch/logs
network.host: 10.0.13.125
http.port: 9200
bootstrap.system_call_filter: false 
discovery.zen.ping.unicast.hosts: ["10.0.13.126", "10.0.13.125"]
discovery.zen.minimum_master_nodes: 2
```

# 四、安装head插件
插件地址：https://github.com/mobz/elasticsearch-head
#### 1、安装nodejs
nodejs官网下载地址：https://nodejs.org/dist/
```
wget https://nodejs.org/dist/v7.9.0/node-v7.9.0-linux-x64.tar.gz
```

#### 2、配置环境变量
```
tar -xf node-v7.9.0-linux-x64.tar.gz -C /usr/
vim /etc/profile
export JAVA_HOME JRE_HOME PATH CLASSPATH
export NODE_HOME=/usr/node-v7.9.0-linux-x64
export PATH=$PATH:$NODE_HOME/bin
source /etc/profile
```

#### 3、安装head插件
```
git clone git://github.com/mobz/elasticsearch-head.git
npm install -g grunt --registry=https://registry.npm.taobao.org
```

#### 4、修改head目录下的Gruntfile.js配置，head默认监听127.0.0.1
```
options: {
        hostname: '0.0.0.0'，
        port: 9100,
        base: '.',
        keepalive: true
```
#### 5、elasticsearch配置允许跨域访问
修改elasticsearch配置文件elasticsearch.yml
```
http.cors.enabled: true
http.cors.allow-origin: "*"
```
#### 6、启动查看
```
[root@elastic-1 elasticsearch-head]# vim Gruntfile.js
[root@elastic-1 elasticsearch-head]# node_modules/grunt/bin/grunt server
>> Local Npm module "grunt-contrib-jasmine" not found. Is it installed?
 
Running "connect:server" (connect) task
Waiting forever...
Started connect web server on http://localhost:9100
```
url:http://10.0.13.126:9100/

![](http://ocppiicaw.bkt.clouddn.com/elastic/elastic_head.png)

# 错误解决
>ERROR: bootstrap checks failed
max file descriptors [65535] for elasticsearch process likely too low, increase to at least [65536]
memory locking requested for elasticsearch process but memory is not locked
max number of threads [1024] for user [jason] likely too low, increase to at least [2048]
max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]
system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk

```
＄vim /etc/security/limits.conf
elasticsearch hard nofile 65536     # 针对 max file descriptors
elasticsearch soft nproc 2048       # 针对 max number of threads
 
$ vim /etc/sysctl.conf
vm.max_map_count=262144          # 针对 max virtual memory areas
 
$ vim /etc/elasticsearch/elasticsearch.yml
bootstrap.system_call_filter: false      # 针对 system call filters failed to install, 参见 https://www.elastic.co/guide/en/elasticsearch/reference/current/system-call-filter-check.html

value for setting "discovery.zen.minimum_master_nodes" is too low. This can result in data loss! Please set it to at least a quorum of master-eligible nodes (current value: [1], total number of master-eligible nodes used for publishing in this round: [2])
```
