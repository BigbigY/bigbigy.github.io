---
title: Elastic stack介绍
tags: [日志]
categories: Elastic Stack 从入门到实践
type: "categories"
author: "luck"

---

# 简介

[Elastic Stack](https://www.elastic.co/start)是```ELK```日志系统的官方称呼，其实并不是一款软件，而是一整套解决方案，是三个软件产品的首字母缩写```Elasticsearch```、```Logstash```、```Kibana```。

![](http://ocppiicaw.bkt.clouddn.com/image/elk/elk_filebeat01.png)

# 工作流程
在需要收集日志的所有服务上部署filebeat，作为shipper用于收集日志，将过滤后的内容发送到Redis队列，然后读取redis、转换、解析日志并收集在一起交给全文搜索服务ElasticSearch，可以用ElasticSearch进行自定义搜索通过Kibana 来结合自定义搜索进行页面展示。


# ELK的用途与使用场景
传统意义上，ELK是作为替代Splunk的一个开源解决方案。Splunk 是日志分析领域的领导者。日志分析并不仅仅包括系统产生的错误日志，异常，也包括业务逻辑，或者任何文本类的分析。而基于日志的分析，能够在其上产生非常多的解决方案，譬如：

- 人员操作安全：开发人员不能登录线上服务器查看详细日志
- 海量数据实时查询：日志数据量大，查询速度慢，或者数据不够实时
- 日志集中查询：分布式系统，日志数据分散难以查找
- 问题排查：运维和开发能够快速的定位问题。
- 监控和预警： 日志，监控，预警是相辅相成的。基于日志的监控，预警使得运维有自己的机械战队，大大节省人力以及延长运维的寿命。
- 关联事件：多个数据源产生的日志进行联动分析，通过某种分析算法，就能够解决生活中各个问题。比如金融里的风险欺诈等。这个可以可以应用到无数领域了，取决于你的想象力。
- 数据分析： 这个对于数据分析师，还有算法工程师都是有所裨益的。


# 组件说明：
```Elasticsearch```是个开源分布式搜索引擎，它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。

```Logstash```是一个开源的用于收集,分析和存储日志的工具。

```Kibana``` 也是一个开源和免费的工具，Kibana可以为 Logstash 和 ElasticSearch 提供的日志分析友好的 Web 界面，可以汇总、分析和搜索重要数据日志。

```Beats```是elasticsearch公司开源的一款采集系统监控数据的代理agent，是在被监控服务器上以客户端形式运行的数据收集器的统称，可以直接把数据发送给Elasticsearch或者通过Logstash发送给Elasticsearch，然后进行后续的数据分析活动。Beats由如下组成:
-  ```Packetbeat```：是一个网络数据包分析器，用于监控、收集网络流量信息，Packetbeat嗅探服务器之间的流量，解析应用层协议，并关联到消息的处理， 其支  持 ICMP (v4 and v6)、DNS、HTTP、Mysql、PostgreSQL、Redis、MongoDB、Memcache等协议；
-  ```Filebeat```：用于监控、收集服务器日志文件，其已取代 logstash forwarder；
-  ```Metricbeat```：可定期获取外部系统的监控指标信息，其可以监控、收集Apache、HAProxy、MongoDB、MySQL、Nginx、PostgreSQL、Redis、System、Zookeeper等服务；
-  ```Winlogbeat```：用于监控、收集Windows系统的日志信息；
-  ```Create your own Beat```：自定义beat ，如果上面的指标不能满足需求，elasticsarch鼓励开发者使用go语言，扩展实现自定义的beats，只需要按照模板，实现监控的输入，日志，输出等即可。


# FQA
- [elastic 中文社区](https://elasticsearch.cn/)
- [github elasticsearch issues](https://github.com/elastic/elasticsearch/issues)
- [github logstash issues](https://github.com/elastic/logstash/issues)
- [github kibana issues](https://github.com/elastic/kibana/issues)



