---
title: Saltstack数据系统-Grains(2)
tags: Saltstack
categories: 自动化运维工具
type: "categories"
---

数据系统 

1、可以匹配minion
2、能搜集系统信息

Grains：saltstack的组件，在里面存放着minion启动的信息，存储的minion端，只有在minion启动的时候才会搜集，例如：搜集阿里云数据内存，升级后到16G时还是8G，c除非得重启minion，所以被称为静态数据系统

# 1、信息查询/搜集，例如查询机器 #

[root@node1-saltstack salt]# salt 'node2*' grains.ls

```
node2-saltstack:
    - SSDs
    ......
    - zmqversion
```

全部grains值

[root@node1-saltstack salt]# salt 'node2*' grains.items

```
node2-saltstack:
    ----------
    SSDs:
    biosreleasedate:
        05/20/2014
    快速服务代码：可以查机器什么时候过保
.......
```

查看单个信息(或者使用grains.get fqdn)：

[root@node1-saltstack salt]# salt 'node2*' grains.item fqdn

```
node2-saltstack:
    ----------
    fqdn:
        node2-saltstack
```

只显示值

[root@node1-saltstack salt]# salt 'node2*' grains.get fqdn

```
node2-saltstack:
    node2-saltstack
```

# 2、第二个应用场景，匹配minnin： #

搜集所有centos的信息：

[root@node1-saltstack salt]# salt -G os:CentOS cmd.run 'pwd'

```
node2-saltstack:
    /root
node1-saltstack:
    /root
```

-G ： 表示使用grains来匹配

# 3、支持自定义的grains #
   
可以定义指定的机器的grains类型，例如加监控的时候，把指定的机器设置成Nginx,然后匹配所有的nginx都安装nginx服务

vim /etc/salt/minion

```
#打开注释
grains:
  roles:
    - webserver
    - memcache
```

重启minion

    systemctl restart salt-minion

匹配所有roles为memcache的机器

[root@node1-saltstack salt]# salt -G 'roles:memcache' cmd.run 'echo hehe'

```
node1-saltstack:
    hehe
```

# 4、默认会在，/etc/salt/grains里面读 #

vim /etc/salt/grains

```
web: nginx
```

重启minion

    systemctl restart salt-minion

执行

[root@node1-saltstack salt]# salt -G 'web:nginx' cmd.run 'echo hehe'

```
node1-saltstack:
    hehe
```

第三种方法，在top.sls中执行grains过滤

vim top.sls

```
base:
  'web:nginx':
    - match: grain
    - apache
```

执行：

    [root@node1-saltstack salt]# salt '*' state.highstate
