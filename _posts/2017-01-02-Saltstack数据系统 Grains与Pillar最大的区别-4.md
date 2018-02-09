---
title: Saltstack数据系统 Grains与Pillar最大的区别(4)
tags: Saltstack
categories: 自动化运维工具
type: "categories"
---


名称	存储位置	数据类型	数据采集更新方式	应用
Grains	Minion端	静态数据	Minion启动时收集，也可以使用saltutil.sync_grains进行刷新。	存储Minion基本数据。比如用于匹配Minion,自身数据可以用来做资产管理等。
Pillar	Master端	动态数据	在Master端定义，指定给对应的Minion。可以使用saltuil.refresh_pillar刷新	存储Master指定的数据，只有指定的Minion可以看到。用于敏感数据保存。

 