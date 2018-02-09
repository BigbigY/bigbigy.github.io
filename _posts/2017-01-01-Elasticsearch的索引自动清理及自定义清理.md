---
title: Elasticsearch的索引清理及自定义清理
tags: [ELK,slasticsearch]
categories: 日志
type: "categories"
---
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;近发现elasticsearch近期索引文件大的吓人，清理了下之前的索引文件，发现服务器性能大大的减轻了一半，想一直保留近一个月的索引文件，但是又不想每个月手动清楚，在此写了一个小脚本
<!-- more -->

# 手动删除 #

    rm -rf *2016-07-*

# api删除 #

    curl -XDELETE 'http://127.0.0.1:9200/logstash-2016-07-*'

清理掉了所有 7月份的索引文件，我发现curl 删除比rm删除要快出很多


    脚本加api删除（推荐）

    cat es-index-clear.sh
    #/bin/bash
    #es-index-clear
    #获取上个月份日期
    LAST_DATA=`date -d "last month"+%Y-%m`
    #删除上个月份所有的索引
    curl -XDELETE'http://127.0.0.1:9200/*-'${LAST_DATA}'-*'


# 添加到任务计划 #

    crontab -e
    0 1 5 * * /script/es-index-clear.sh