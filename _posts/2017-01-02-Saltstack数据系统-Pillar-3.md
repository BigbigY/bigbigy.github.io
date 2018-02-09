---
title: Saltstack数据系统-Pillar(3)
tags: Saltstack
categories: 自动化运维工具
type: "categories"
---

Pillar也是saltstack的一个组件，给minion指定想要的数据，安全性好，主要做配置管理

查看系统有哪些pillar（master端设置的，默认是False的）

## 开启pillar ##

vim /etc/salt/master 

```
#开启
pillar_opts: True
```

## 重启 ##

```
systemctl restart salt-master
```


## 查看所有pillar（字典返回，）： ##

```salt '*' pillar.items```

## 定义pillar根目录，改回False ##

vim /etc/salt/master 

```
pillar_opts: False
.........
pillar_roots:
  base:
    - /srv/pillar
```

## 创建目录： ##
mkdir /srv/pillar

## 重启salt-master: ##
systemctl restart salt-master

## 自定义pillar： ##

[root@node1-saltstack salt]# vim /srv/pillar/apache.sls
![](http://ocppiicaw.bkt.clouddn.com/elif.png)

## 使用top文件： ##

[root@node1-saltstack salt]# cat /srv/pillar/top.sls

```
base:
  '*':
    - apache
```

执行：

[root@node1-saltstack salt]# salt '*' pillar.items

```
node2-saltstack:
    ----------
    apache:
        httpd
node1-saltstack:
    ----------
    apache:
        httpd
```

pillar也以用来定位主机：

如果发现不可用，需要对minion进行刷新：

[root@node1-saltstack salt]# salt -I 'apache:httpd' test.ping

```
node1-saltstack:
    Minion did not return. [No response]
node2-saltstack:
    Minion did not return. [No response]
```

刷新命令：

[root@node1-saltstack salt]# salt '*' saltutil.refresh_pillar

```
node2-saltstack:
    True
node1-saltstack:
    True
```

再次执行就会成功：

[root@node1-saltstack salt]# salt -I 'apache:httpd' test.ping

```
node1-saltstack:
    True
node2-saltstack:
    True
```