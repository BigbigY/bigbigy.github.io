---
title: 2-Elastic stack之Filebeat
tags: [Elastic stack,Filebeat]
categories: 日志搜索
type: "categories"
author: "luck"
header-img: "img/post-bg-e2e-ux.jpg"
---

# Filebeat介绍

filebeat最初是基于logstash-forwarder源码的日志数据shipper。Filebeat安装在服务器上作为代理来监视日志目录或特定的日志文件，要么将日志转发到Logstash进行解析，要么直接发送到Elasticsearch进行索引。

# 一、安装Filebeat

1. 下载并安装公共签名密钥：
```
sudo rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
```

2. 在目录中创建一个扩展.repo名为（例如elastic.repo）的/etc/yum.repos.d/文件，并添加以下行：
```
[elastic-5.x]
name=Elastic repository for 5.x packages
baseurl=https://artifacts.elastic.co/packages/5.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

3. 安装
```
$ sudo yum install filebeat
$ filebeat.sh –version 
filebeat version 5.3.0 (amd64), libbeat 5.3.0
```

4. 要在启动期间将Beat配置为自动启动，请运行：
```
sudo chkconfig --add filebeat
```

5. 其他安装方法请参考
```
https://www.elastic.co/guide/en/beats/filebeat/5.3/filebeat-installation.html
```

# 二、配置文件概述

1. 路径：
```
/etc/filebeat/filebeat.yml
```

2. 定义日志文件路径:

 ```
filebeat.prospectors:
- input_type: log
  paths:
    - /var/log/*.log
```

>此示例中的/var/log/*.log探测器收集路径中的所有文件，这意味着Filebeat将收集以目录/var/log/结尾的所有文件.log
要从预定义级别的子目录中获取所有文件，可以使用以下模式： /var/log/*/*.log。这.log将从子文件夹中获取所有文件/var/log。它不会从/var/log文件夹本身获取日志文件。目前，不可能递归地获取目录的所有子目录中的所有文件。

3.  如果要将输出发送到Elasticsearch，请设置Filebeat可以找到Elasticsearch安装的IP地址和端口：
```
output.elasticsearch:
  hosts: ["192.168.1.42:9200"]
```

# 三、配置Filebeat发送到Logstash

提示：要使用Logstash作为输出，还必须 设置Logstash以接收Beats的事件。

1. 禁用Elasticsearch为输出并且取消注释logstash部分来启用Logstash输出：
```
#output.elasticsearch:
#hosts: ["192.168.1.42:9200"]
#----------------------------- Logstash output --------------------------------
output.logstash:
  hosts: ["192.168.1.14:5044"]
```

  该hosts选项指定Logstash服务器和Logstash配置为侦听传入的Beats连接端口(5044)

2. 测试配置文件
```
/usr/bin/filebeat.sh -configtest -e
2017/04/11 14:18:09.178552 beat.go:285: INFO Home path: [/usr/share/filebeat] Config path: [/etc/filebeat] Data path: [/var/lib/filebeat] Logs path: [/var/log/filebeat]
2017/04/11 14:18:09.178607 beat.go:186: INFO Setup Beat: filebeat; Version: 5.3.0
2017/04/11 14:18:09.178714 logstash.go:90: INFO Max Retries set to: 3
2017/04/11 14:18:09.178812 outputs.go:108: INFO Activated logstash as output plugin.
2017/04/11 14:18:09.178944 publish.go:295: INFO Publisher name: localhost
2017/04/11 14:18:09.179534 async.go:63: INFO Flush Interval set to: 1s
2017/04/11 14:18:09.179556 async.go:64: INFO Max Bulk Size set to: 2048
```Config OK```
```

# 四、在弹性搜索中加载索引模板
>在Elasticsearch中，索引模板用于定义设置和映射，以确定如何分析字段。
Filebeat的建议索引模板文件由Filebeat软件包安装。如果您在filebeat.yml配置文件中接受模板加载的默认配置，Filebeat会在成功连接到Elasticsearch后自动加载该模板。如果模板已经存在，它不会被覆盖，除非您配置Filebeat来执行此操作。
如果要禁用自动模板加载，或者要加载自己的模板，可以在Filebeat配置文件中更改模板加载的设置。如果选择禁用自动模板加载，则需要手动加载模板。
 - 配置模板加载 - 仅支持Elasticsearch输出
 - 手动加载模板 - Logstash输出所需


1. 配置模板加载
  默认情况下，Filebeat会自动加载推荐的模板文件```filebeat.template.json```，如果启用了Elasticsearch输出。您可以通过调整文件中的选项```template.name```和```template.path```选项来配置filebeat以加载其他模板```filebeat.yml```：
默认情况下，如果索引中已存在模板，则不会覆盖该模板。要覆盖现有模板，请template.overwrite: true在配置文件中进行设置
要禁用自动模板加载，请注释Elasticsearch输出下的模板部分。
如果使用Logstash输出，则不支持自动加载模板的选项
```
output.elasticsearch:
hosts: ["localhost:9200"]
template.name: "filebeat"
template.path: "filebeat.template.json"
template.overwrite: false
```

2. 加载模板手动
  如果禁用自动模板加载，则需要运行以下命令来加载模板：
```
curl -XPUT 'http://localhost:9200/_template/filebeat' -d@/etc/filebeat/filebeat.template.json
```

3. 其他
如果您已经使用Filebeat将数据索引到Elasticsearch，索引可能包含旧文档。加载索引模板后，可以从filebeat- *中删除旧文档，强制Kibana查看最新的文档。使用此命令：
```
curl -XDELETE 'http://localhost:9200/filebeat-*'
```

# 五、启动Filebeat
```
sudo /etc/init.d/filebeat startFilebeat
```

 现在可以将日志文件发送到定义的输出。

# 六、加载 Kibana Index Pattern

我们不提供用于可视化Filebeat数据的预构建仪表板。但是，为了使您更容易地在Kibana中探索Filebeat数据，我们创建了一个Filebeat索引模式：```filebeat-*```。要加载此模式，您可以使用为导入仪表板提供的脚本：
```
./scripts/import_dashboards -only-index
```

---
# 相关链接：

>- 官网：https://www.elastic.co/start
- 过去版本选择下载：https://www.elastic.co/downloads/past-releases
- 5.x下载：https://www.elastic.co/v5
- kibana文档：https://www.elastic.co/guide/en/kibana/current/rpm.html
- elasticsearch文档：https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html
- logstack文档：https://www.elastic.co/guide/en/logstash/5.3/logstash-5-3-0.html
- beats文档：https://www.elastic.co/guide/en/beats/libbeat/5.3/release-notes-5.3.0.html
- Filebeat：https://www.elastic.co/guide/en/beats/filebeat/5.3/filebeat-getting-started.html

 