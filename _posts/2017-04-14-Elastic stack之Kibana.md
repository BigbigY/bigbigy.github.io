---
title: 5-Elastic stack之Kibana
tags: [Elastic stack,Kibana]
categories: 自动化运维工具
type: "categories"
author: "luck"
header-img: "img/post-bg-e2e-ux.jpg"
---

# 简介
>Kibana 是一个为 Logstash 和 ElasticSearch 提供的日志分析的 Web 接口。可使用它对日志进行高效的搜索、可视化、分析等各种操作。


# 一、 安装Kibana（YUM）

#### 1、 下载并安装公共签名密钥：

```
rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
```

#### 2、/etc/yum.repos.d/，在具有.repo后缀的文件中将以下内容添加到您的目录中logstash.repo

```
[kibana-5.x]
name=Kibana repository for 5.x packages
baseurl=https://artifacts.elastic.co/packages/5.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

#### 3. 安装
```
sudo yum install kibana
```

# 二、配置
```
$ sed '/#/d' /etc/kibana/kibana.yml |grep -v '^$'
server.port: 5601
server.host: "10.0.13.125"
server.name: "bbtree-kibana"
elasticsearch.url: "http://10.0.13.126:9200"
```

# 三、相关文章
- [Kibana官方文档](https://www.elastic.co/guide/en/kibana/current/rpm.html )
- [X-Pack](https://www.elastic.co/guide/en/x-pack/current/xpack-introduction.html)
 

# 四、展示

##  进入Kibana
![](http://ocppiicaw.bkt.clouddn.com/elastic/kibana1.png)
 ## 查看接受到的数据
![](http://ocppiicaw.bkt.clouddn.com/elastic/kibana2.png)
 

## 时间点查找
![](http://ocppiicaw.bkt.clouddn.com/elastic/kibana3.png)
 

 
## 搜索
温馨提示：中文关键字需要加引号
![](http://ocppiicaw.bkt.clouddn.com/elastic/kibana4.png)


## 图表
![](http://ocppiicaw.bkt.clouddn.com/elastic/kibana5.png)
![](http://ocppiicaw.bkt.clouddn.com/elastic/kibana6.png)
![](http://ocppiicaw.bkt.clouddn.com/elastic/kibana7.png)
![](http://ocppiicaw.bkt.clouddn.com/elastic/kibana8.png)
![](http://ocppiicaw.bkt.clouddn.com/elastic/kibana9.png)
![](http://ocppiicaw.bkt.clouddn.com/elastic/kibana10.png)

 
# 比之前使用的4.0漂亮了好多
![](http://ocppiicaw.bkt.clouddn.com/elastic/kibana11.png)
![](http://ocppiicaw.bkt.clouddn.com/elastic/kibana12.png) 



# 其他展示
![](http://ocppiicaw.bkt.clouddn.com/elastic/kibana13.png)
