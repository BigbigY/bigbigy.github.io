---
title: 1-Elastic stack介绍
tags: [Elastic stack]
categories: 日志搜索
type: "categories"
author: "luck"
header-img: "img/post-bg-e2e-ux.jpg"
---

# 简介

```Elasticsearch```是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。

```Logstash```是一个开源的用于收集,分析和存储日志的工具。

```Kibana``` 也是一个开源和免费的工具，Kibana可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以汇总、分析和搜索重要数据日志。

```Beats```是elasticsearch公司开源的一款采集系统监控数据的代理agent，是在被监控服务器上以客户端形式运行的数据收集器的统称，可以直接把数据发送给Elasticsearch或者通过Logstash发送给Elasticsearch，然后进行后续的数据分析活动。Beats由如下组成:
- ```Packetbeat```：是一个网络数据包分析器，用于监控、收集网络流量信息，Packetbeat嗅探服务器之间的流量，解析应用层协议，并关联到消息的处理， 其支  持 ICMP (v4 and v6)、DNS、HTTP、Mysql、PostgreSQL、Redis、MongoDB、Memcache等协议；
- ```Filebeat```：用于监控、收集服务器日志文件，其已取代 logstash forwarder；
- ```Metricbeat```：可定期获取外部系统的监控指标信息，其可以监控、收集Apache、HAProxy、MongoDB、MySQL、Nginx、PostgreSQL、Redis、System、Zookeeper等服务；
- ```Winlogbeat```：用于监控、收集Windows系统的日志信息；
- ```Create your own Beat```：自定义beat ，如果上面的指标不能满足需求，elasticsarch鼓励开发者使用go语言，扩展实现自定义的beats，只需要按照模板，实现监控的输入，日志，输出等即可。
![](http://ocppiicaw.bkt.clouddn.com/elastic/elastic.png)
*Beats 将搜集到的数据发送到 Logstash，经 Logstash 解析、过滤后，将其发送到 Elasticsearch 存储，并由 Kibana 呈现给用户。*
*Beats 作为日志搜集器没有Logstash 作为日志搜集器消耗资源，解决了 Logstash 在各服务器节点上占用系统资源高的问题。*

# 相关链接：
>- 官网：https://www.elastic.co/start
- 过去版本选择下载：https://www.elastic.co/downloads/past-releases
- 5.x下载：https://www.elastic.co/v5
- kibana文档：https://www.elastic.co/guide/en/kibana/current/rpm.html
- elasticsearch文档：https://www.elastic.co/guide/en/elasticsearch/reference/current/install-elasticsearch.html
- logstack文档：https://www.elastic.co/guide/en/logstash/5.3/logstash-5-3-0.html
- beats文档：https://www.elastic.co/guide/en/beats/libbeat/5.3/release-notes-5.3.0.html


