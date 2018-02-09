---
title: cloudera CM,CDH介绍
tags: [cloudera,CM,CDH]
categories: 大数据
type: "categories"
author: "luck"
header-img: "img/post-bg-unix-linux.jpg"
---

> 总结地址:

https://www.cloudera.com/products/product-components/cloudera-manager.html

# 相关文档地址

> 官网地址：http://www.cloudera.com/content/cloudera/en/home.html
项目演示视屏地址：https://www.cloudera.com/products/product-components/cloudera-manager/cloudera-manager-demos.html

# 安装包下载：

>同时附上各个版本包的地址：
Cloudera文档汇总
http://www.cloudera.com/content/support/en/documentation.html
CDH4、CDH5包汇总
http://archive.cloudera.com/cdh4/
http://archive.cloudera.com/cdh5/
CM4、CM5包汇总
http://archive.cloudera.com/cm4/
http://archive.cloudera.com/cm5/
以前版本地址：
CDH1~CDH3
http://archive-primary.cloudera.com/cdh/


# What is the CDH？

hadoop是一个开源项目，所以很多公司在这个基础进行商业化，不收费的Hadoop版本主要有三个（均是国外厂商），分别是：Apache（最原始的版本，所有发行版均基于这个版本进行改进）、Cloudera版本（Cloudera’s Distribution Including Apache Hadoop，简称CDH）、Hortonworks版本(Hortonworks Data Platform，简称“HDP”）。对于国内而言，绝大多数选择CDH版本，Cloudera对hadoop做了相应的改变。Cloudera公司的发行版，我们将该版本称为CDH（Cloudera Distribution Hadoop）。
 
Cloudera Express版本是免费的
Cloudera Enterprise是需要购买注册码的


# What is the Cloudera Manager？

Cloudera Manager（简称CM），Cloudera开发公司的产品，用于管理CDH集群，其主要功能是对CDH集群进行监控，大大改善原生ApacheHadoop的安装、配置复杂和需要使用第三方开源的监控工具所带来的诸多问题，可进行节点安装、配置、服务配置等，提供web窗口界面提高了Hadoop配置可见度，而且降低了集群参数设置的复杂度。其中，CDH是Cloudera公司的开源产品，可以不依靠CM独立安装。CM有free版本，提供60天在收费版中才能使用的高级功能的免费使用期限。
Cloudera Manager是CDH市场领先的管理平台。通过Cloudera Manger，运维人员得以提高集群的性能，提升服务质量，提高合规性并降低管理成本。

> 最新版CM下载地址：https://www.cloudera.com/downloads/manager/5-10-0.html
安装要求：https://www.cloudera.com/documentation/enterprise/latest/topics/installation_reqts.html
安装概述：https://www.cloudera.com/documentation/enterprise/latest/topics/installation_installation.html
故障排除：https://www.cloudera.com/documentation/enterprise/latest/topics/cm_ig_troubleshooting.html

# Cloudera Manager轻松实现Hadoop的什么？

- 自动部署和配置
- 可定制监控和报告
- 轻松，强大的故障排除
- 零宕机维护

# Cloudera主要发布了3个类型的产品

![](http://ocppiicaw.bkt.clouddn.com/CDH/CM.jpg)