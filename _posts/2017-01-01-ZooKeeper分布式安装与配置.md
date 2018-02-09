---
title: ZooKeeper分布式安装与配置
tags: zookeeper
categories: 分布式
type: "categories"
---

![](http://ocppiicaw.bkt.clouddn.com/zookeeper/logo.jpg)

<!--more-->

Zookeeper 安装和配置
Zookeeper的安装和配置十分简单, 既可以配置成单机模式, 也可以配置成集群模式. 下面将分别进行介绍.

环境：
10.1.250.10
10.1.250.11
10.1.250.12

ZooKeeper集群中具有两个关键的角色：Leader和Follower。集群中所有的结点作为一个整体对分布式应用提供服务，集群中每个结点之间都互相连接，所以，在配置的ZooKeeper集群的时候，每一个结点的host到IP地址的映射都要配置上集群中其它结点的映射信息。
原理讲解：

http://cailin.iteye.com/blog/2014486/

    Vim /etc/hosts
    10.1.250.10 zoo1
    10.1.250.11 zoo2
    10.1.250.12 zoo3

# 单机模式 #
下载zookeeper的安装包之后, 解压到合适目录. 进入zookeeper目录下的conf子目录, 创建zoo.cfg:

    tickTime=2000
    dataDir=/data/service/zookeeper/data
    dataLogDir=/data/service/zookeeper/logs
    clientPort=2181   

参数说明:

- tickTime: zookeeper中使用的基本时间单位, 毫秒值.
- dataDir: 数据目录. 可以是任意目录.
- dataLogDir: log目录, 同样可以是任意目录. 如果没有设置该参数, 将使用和dataDir相同的设置.
- clientPort: 监听client连接的端口号.

至此, zookeeper的单机模式已经配置好了. 启动server只需运行脚本:

    bin/zkServer.sh start  

Server启动之后, 就可以启动client连接server了, 执行脚本:


    bin/zkCli.sh -server localhost:2181  
 
# 伪集群模式 #
所谓伪集群, 是指在单台机器中启动多个zookeeper进程, 并组成一个集群. 以启动3个zookeeper进程为例.
将zookeeper的目录拷贝2份:
   
|--zookeeper0  
|--zookeeper1  
|--zookeeper2  
 更改zookeeper0/conf/zoo.cfg文件为:

    tickTime=2000
    initLimit=5
    syncLimit=2
    dataDir=/data/service/zookeeper0/data
    dataLogDir=/data/service/zookeeper0/logs
    clientPort=2180  
    server.0=127.0.0.1:8880:7770
    server.1=127.0.0.1:8881:7771
    server.2=127.0.0.1:8882:7772  

新增了几个参数, 其含义如下:
initLimit: zookeeper集群中的包含多台server, 其中一台为leader, 集群中其余的server为follower. initLimit参数配置初始化连接时, follower和leader之间的最长心跳时间. 此时该参数设置为5, 说明时间限制为5倍tickTime, 即5*2000=10000ms=10s.
syncLimit: 该参数配置leader和follower之间发送消息, 请求和应答的最大时间长度. 此时该参数设置为2, 说明时间限制为2倍tickTime, 即4000ms.
server.X=A:B:C 其中X是一个数字, 表示这是第几号server. A是该server所在的IP地址. B配置该server和集群中的leader交换消息所使用的端口. C配置选举leader时所使用的端口. 由于配置的是伪集群模式, 所以各个server的B, C参数必须不同.
参照zookeeper0/conf/zoo.cfg, 配置zookeeper1/conf/zoo.cfg, 和zookeeper2/conf/zoo.cfg文件. 只需更改dataDir, dataLogDir, clientPort参数即可.

例如：

    [root@zk ~]# cat /usr/local/zookeeper0/conf/zoo.cfg 
    tickTime=2000
    initLimit=5
    syncLimit=2
    dataDir=/usr/local/zookeeper0/data
    dataLogDir=/usr/local/zookeeper0/logs
    clientPort=8581
    server.0=namenode:8880:7770
    server.1=namenode:8881:7771
    server.2=namenode:8882:7772 
    [root@zk ~]# cat /usr/local/zookeeper1/conf/zoo.cfg 
    tickTime=2000
    initLimit=5
    syncLimit=2
    dataDir=/usr/local/zookeeper1/data
    dataLogDir=/usr/local/zookeeper1/logs
    clientPort=8582
    server.0=namenode:8880:7770
    server.1=namenode:8881:7771
    server.2=namenode:8882:7772
    [root@zk ~]# cat /usr/local/zookeeper2/conf/zoo.cfg 
    tickTime=2000
    initLimit=5
    syncLimit=2
    dataDir=/usr/local/zookeeper2/data
    dataLogDir=/usr/local/zookeeper2/logs
    clientPort=8583
    server.0=namenode:8880:7770
    server.1=namenode:8881:7771
    server.2=namenode:8882:7772
    
在之前设置的dataDir中新建myid文件, 写入一个数字, 该数字表示这是第几号server. 该数字必须和zoo.cfg文件中的server.X中的X一一对应.
/data/service/zookeeper0/data/myid文件中写入0, /data/service/zookeeper1/data/myid文件中写入1, /data/service/zookeeper2/data/myid文件中写入2.
分别进入zookeeper0/bin,/zookeeper1/bin, zookeeper2/bin三个目录, 启动server.
任意选择一个server目录, 启动客户端:

    bin/zkCli.sh -server localhost:2180 

启动验证

    ./bin/zkServer.sh start
    [root@zk ~]# /usr/local/zookeeper0/bin/zkServer.sh status
    JMX enabled by default
    Using config: /usr/local/zookeeper0/bin/../conf/zoo.cfg
    Mode: follower
    [root@zk ~]# /usr/local/zookeeper1/bin/zkServer.sh status
    JMX enabled by default
    Using config: /usr/local/zookeeper1/bin/../conf/zoo.cfg
    Mode: leader
    [root@zk ~]# /usr/local/zookeeper2/bin/zkServer.sh status
    JMX enabled by default
    Using config: /usr/local/zookeeper2/bin/../conf/zoo.cfg
    Mode: follower

# 完全分布式集群模式 #
集群模式的配置和伪集群基本一致.
由于集群模式下, 各server部署在不同的机器上, 因此各server的conf/zoo.cfg文件可以完全一样.
下面是一个示例:
   
    cat /data/zookeeper-3.4.9/bin/zoo.cfg
    tickTime=2000
    initLimit=5
    syncLimit=2
    dataDir=/data/zookeeper/data
    dataLogDir=/data/zookeeper/logs
    clientPort=2181  
    server.10=10.1.250.10:2888:3888  
    server.11=10.1.250.11:2888:3888
    server.12=10.1.250.12:2888:3888  

同步到其他节点：

    scp –r zookeeper-3.4.9 10.1.250.11:/data/
    scp –r zookeeper-3.4.9 10.1.250.12:/data/


需要注意的是, 各server的dataDir目录下的myid文件中的数字必须不同.
10.1.250.10 server的myid为10, 10.1.250.11 server的myid为11, 10.1.250.12 server的myid为12.

启动

    ./bin/zkServer.sh start

注意：在启动过程中会抛异常，这属于正常现象，待全部集群启动就ok
 安装验证
可以通过ZooKeeper的脚本来查看启动状态，包括集群中各个结点的角色（或是Leader，或是Follower），如下所示，是在ZooKeeper集群中的每个结点上查询的结果：

     [root@node1 bin]$ zkServer.sh status
    JMX enabled by default
    Using config:/data /zookeeper-3.4.9/bin/../conf/zoo.cfg
    Mode: follower
     
    [root@node2 bin]$ zkServer.sh status
    JMX enabled by default
    Using config: /data /zookeeper-3.4.9/bin/../conf/zoo.cfg
    Mode: leader
     
    [root@node3 bin]$ zkServer.sh status
    JMX enabled by default
    Using config: /data /zookeeper-3.4.9/bin/../conf/zoo.cfg
    Mode: follower

