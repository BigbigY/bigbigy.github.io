---
title: 基于etcd加saltstack的自动化扩容
tags: [etcd,saltstack,自动化]
categories: 自动化运维工具
type: "categories"
author: "luck"
header-img: "img/contact-bg.jpg"
---

自动扩容：
当某个指标到达一个零界点的时候触发

1、zabbix监控--当指标达到一个报警的阀值，会触发一个action--创建了一台虚拟机/docker--部署服务--部署代码--测试--加入集群--加入监控--通知

# 一、etcd #

etcd是一个高可用的键值存储系统，主要用于共享配置和服务发现。

- 简单: curl可访问的用户的API（HTTP+JSON）
- 安全: 可选的SSL客户端证书认证
- 快速: 单实例每秒 1000 次写操作
- 可靠: 使用Raft保证一致性

## 下载 ##

```
[root@node1-saltstack src]# tar xf etcd-v2.2.1-linux-amd64.tar.gz 
[root@node1-saltstack src]# cd etcd-v2.2.1-linux-amd64
[root@node1-saltstack etcd-v2.2.1-linux-amd64]# ll
drwxr-xr-x 6 luck luck     4096 10月 16 2015 Documentation
-rwxr-xr-x 1 luck luck 14359232 10月 16 2015 etcd
-rwxr-xr-x 1 luck luck 12719808 10月 16 2015 etcdctl
-rw-r--r-- 1 luck luck     5613 10月 16 2015 README-etcdctl.md
-rw-r--r-- 1 luck luck     4684 10月 16 2015 README.md
```

拷贝2个命令到系统默认执行目录：

    [root@node1-saltstack etcd-v2.2.1-linux-amd64]# cp etcd etcdctl /usr/local/bin/


## 查看版本： ##

```
[root@node1-saltstack etcd-v2.2.1-linux-amd64]# etcd --version
etcd Version: 2.2.1
Git SHA: 75f8282
Go Version: go1.5.1
Go OS/Arch: linux/amd64
```

### 创建目录： ###

```[root@node1-saltstack ~]# mkdir -p /data/etcd```

### 运行（或者使用screen）： ###

```
[root@node1-saltstack ~]# nohup etcd --name auto_scale --data-dir /data/etcd/ --listen-peer-urls 'http://192.168.1.6:2380,http://192.168.1.6:7001' --listen-client-urls 'http://192.168.1.6:2379,http://192.168.1.6:4001' --advertise-client-urls 'http://192.168.1.6:2379,http://192.168.1.6:4001' &
```

### 查看端口是否启动： ###

```
[root@node1-saltstack ~]# netstat -antup|grep etcd
tcp        0      0 192.168.1.6:2379        0.0.0.0:*               LISTEN      75965/etcd          
tcp        0      0 192.168.1.6:2380        0.0.0.0:*               LISTEN      75965/etcd          
tcp        0      0 192.168.1.6:7001        0.0.0.0:*               LISTEN      75965/etcd          
tcp        0      0 192.168.1.6:4001        0.0.0.0:*               LISTEN      75965/etcd 
```

### 设置 ###

```
[root@node1-saltstack ~]# curl -s http://192.168.1.6:2379/v2/keys/message -XPUT -d value="Hello world" | python -m json.tool
{
    "action": "set",
    "node": {
        "createdIndex": 5,
        "key": "/message",
        "modifiedIndex": 5,
        "value": "Hello world"
    }
}


获取kv
[root@node1-saltstack ~]# curl -s http://192.168.1.6:2379/v2/keys/message | python -m json.tool
{
    "action": "get",
    "node": {
        "createdIndex": 5,
        "key": "/message",
        "modifiedIndex": 5,
        "value": "Hello world"
    }
}


删除：
[root@node1-saltstack ~]# curl -s http://192.168.1.6:2379/v2/keys/message -XDELETE | python -m json.tool
{
    "action": "delete",
    "node": {
        "createdIndex": 5,
        "key": "/message",
        "modifiedIndex": 6
    },
    "prevNode": {
        "createdIndex": 5,
        "key": "/message",
        "modifiedIndex": 5,
        "value": "Hello world"
    }
}
[root@node1-saltstack ~]# curl -s http://192.168.1.6:2379/v2/keys/message | python -m json.tool
{
    "cause": "/message",
    "errorCode": 100,
    "index": 6,
    "message": "Key not found"
}


设置（超时时间5秒），可以设置自动下线：
[root@node1-saltstack ~]# curl -s http://192.168.1.6:2379/v2/keys/ttl_use -XPUT -d value="Hello world 1" -d ttl=5 | python -m json.tool
{
    "action": "set",
    "node": {
        "createdIndex": 11,
        "expiration": "2017-01-07T17:43:43.747818588Z",
        "key": "/ttl_use",
        "modifiedIndex": 11,
        "ttl": 5,
        "value": "Hello world 1"
    }
}

```


## 二、saltstack ##

依赖：python-etcd这个包

装包：

```
[root@node1-saltstack ~]# yum install python-pip -y
[root@node1-saltstack ~]# pip install python-etcd
```

设置master连etcd的配置：

```
[root@node1-saltstack ~]# vim /etc/salt/master
etcd_pillar_config:
  etcd.host: 192.168.1.6
  etcd.port: 4001
ext_pillar:
  - etcd: etcd_pillar_config root=/salt/haproxy/
```

重启

```systemctl restart master```


设置：

```
[root@node1-saltstack ~]# curl -s http://192.168.1.6:2379/v2/keys/salt/haproxy/backend_www_bighug_com/web-node1 -XPUT -d value="192.168.1.6:8080" | python -m json.tool
{
    "action": "set",
    "node": {
        "createdIndex": 13,
        "key": "/salt/haproxy/backend_www_bighug_com/web-node1",
        "modifiedIndex": 13,
        "value": "192.168.1.6:8080"
    }
}
```

可以发现已经有了

```
[root@node1-saltstack ~]# salt '*' pillar.items
node2-saltstack:
    ----------
    apache:
        httpd
    backend_www_bighug_com:
        ----------
        web-node1:
            192.168.1.6:8080
node1-saltstack:
    ----------
    apache:
        httpd
    backend_www_bighug_com:
        ----------
        web-node1:
            192.168.1.6:8080

```

保证haproxy正常
![](http://ocppiicaw.bkt.clouddn.com/haproxy/haproxy2.png)

改配置：

[root@node1-saltstack ~]# vim /srv/salt/prod/cluster/files/haproxy-outside.cfg
![](http://ocppiicaw.bkt.clouddn.com/haproxy/haproxy5.png)

因为已经在master里面指定了root=/salt/haproxy/
所以不用指定目录：pillar.backend_www_bighug_com.iteritems()

[root@node1-saltstack ~]# vim /srv/salt/prod/cluster/haproxy-outside.sls

```
include:
  - haproxy.install
haproxy-service:
  file.managed:
    - name: /etc/haproxy/haproxy.cfg
    - source: salt://cluster/files/haproxy-outside.cfg
    - user: root
    - group: root
    - mode: 644
    - template: jinja
  service.running:
    - name: haproxy
    - enable: True
    - reload: True
    - require:
      - cmd: haproxy-init
    - watch:
      - file: haproxy-service
```

执行
[root@node1-saltstack ~]# salt '*' state.highstate

![](http://ocppiicaw.bkt.clouddn.com/haproxy/haproxy3.png)

加节点：

```
curl -s http://192.168.1.6:2379/v2/keys/salt/haproxy/backend_www_bighug_com/web-node2 -XPUT -d value="192.168.1.6:8080" | python -m json.tool
curl -s http://192.168.1.6:2379/v2/keys/salt/haproxy/backend_www_bighug_com/web-node3 -XPUT -d value="192.168.1.6:8080" | python -m json.tool
curl -s http://192.168.1.6:2379/v2/keys/salt/haproxy/backend_www_bighug_com/web-node4 -XPUT -d value="192.168.1.6:8080" | python -m json.tool
```

![](http://ocppiicaw.bkt.clouddn.com/haproxy/haproxy4.png)
