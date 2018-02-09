---
title: Hadoop之完全分布式HDFS集群环境部署
tags: hadoop
categories: 大数据
type: "categories"
---

![](http://ocppiicaw.bkt.clouddn.com/hadoop1.png) 
<!--more--> 
# 初识： #
Hadoop实现了一个分布式文件系统（Hadoop Distributed File System），简称HDFS。HDFS有高容错性的特点，并且设计用来部署在低廉的硬件上；而且它提供高吞吐量来访问应用程序的数据，适合那些有着超大数据集的应用程序。HDFS放宽了POSIX的要求，可以以流的形式访问文件系统中的数据。


# 网络配置 #
master: 10.1.1.10
slave1: 10.1.1.11
slave2: 10.1.1.12

一台机器作为master另外两台是slave，接着在/etc/hosts中把所有集群的主机信息都写进去。

Vim	/etc/hosts     #修改主机与IP的映射关系
添加到/etc/hosts的最后面
10.1.1.10  master
10.1.1.11  slave1
10.1.1.12  slave2

配置好后可以在各个主机上进行ping master和ping slave1/slave2测试一下，是否相互ping得通。

# SSH 无密登录 #
下面是我写的一个自动化SSH 无密登录脚本，运行脚本前需要安装expect包，ubuntu 系统下直接执行：sudo apt-get install expect就可以了。该脚本运行在namenode上，运行时只需要将IP_1改成对应的datanode地址，PWD_1是对应datanode密码。
    #!/bin/sh 
    IP_1=10.1.1.10, 10.1.1.11, 10.1.1.12
    PWD_1=123456
    
    key_generate() {
    expect -c "set timeout -1;
    spawn ssh-keygen -t dsa;
    expect {
    {Enter file in which to save the key*} {send -- \r;exp_continue}
    {Enter passphrase*} {send -- \r;exp_continue}
    {Enter same passphrase again:} {send -- \r;exp_continue}
    {Overwrite (y/n)*} {send -- n\r;exp_continue}
    eof {exit 0;}
    };"
    }
    
    auto_ssh_copy_id () {
    expect -c "set timeout -1;
    spawn ssh-copy-id -i $HOME/.ssh/id_dsa.pub root@$1;
    expect {
    {Are you sure you want to continue connecting *} {send -- yes\r;exp_continue;}
    {*password:} {send -- $2\r;exp_continue;}
    eof {exit 0;}
    };"
    }
    
    rm -rf ~/.ssh
    
    key_generate
    
    ips_1=$(echo $IP_1 | tr ',' ' ')
    for ip in $ips_1
    do
    auto_ssh_copy_id $ip  $PWD_1
    done
    
    eval &(ssh-agent)
    ssh-add

# 安装JDK #
http://bighug.top/2016/10/18/jdk%E5%AE%89%E8%A3%85/

# 安装HADOOP #

解压hadoop包到

## 配置 ##
 配置/data/service/hadoop-2.7.0/etc/hadoop目录下的core-site.xml 

    <configuration>
    <property>
     <name>fs.defaultFS</name>
     <value>hdfs://master:9000</value>
    </property>
    </configuration>

配置/data/service/hadoop-2.7.0/etc/hadoop目录下的hdfs-site.xml

    <configuration>
    <property>
     <name>dfs.replication</name>
     <value>1</value>
     </property>
     <property>
     <name>dfs.namenode.secondary.http-address</name>
     <value>master:9001</value>
     </property>
    </configuration>

配置/data/service/hadoop-2.7.0/etc/hadoop目录下的mapred-site.xml

    <configuration>
    <property>
     <name>mapreduce.framework.name</name>
     <value>yarn</value>
     </property>
    <property>
     <name>mapreduce.jobhistory.address</name>
     <value>master:10020</value>
    </property>
    <property>
     <name>mapreduce.jobhistory.webapp.address</name>
     <value>master:19888</value>
    </property>
    </configuration>

配置/data/service/hadoop-2.7.0/etc/hadoop目录下的yarn-site.xml 

    <configuration>
    
    <!-- Site specific YARN configuration properties -->
    <property>
    <name>yarn.nodemanager.aux-services</name>
     <value>mapreduce_shuffle</value>
    </property>
    <property>
     <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
     </property>
     <property>
     <name>yarn.resourcemanager.address</name>
     <value>master:8032</value>
     </property>
    <property>
    <name>yarn.resourcemanager.scheduler.address</name>
     <value>master:8030</value>
     </property>
     <property>
     <name>yarn.resourcemanager.resource-tracker.address</name>
     <value>master:8031</value>
     </property>
     <property>
    <name>yarn.resourcemanager.admin.address</name>
     <value>master:8033</value>
     </property>
    <property>
    <name>yarn.resourcemanager.webapp.address</name>
     <value>master:8088</value>
     </property>
    </configuration>

## 修改环境变量 ##

配置/data/service/hadoop-2.7.0/etc/hadoop目录下hadoop-env.sh、yarn-env.sh的JAVA_HOME，不设置的话 

    export JAVA_HOME=/usr/jdk1.8.0_65

## 修改slave文件 ##

配置/data/service/hadoop-2.7.0/etc/hadoop目录下的slaves，删除默认的localhost，增加2个从节点，

    h-slave1
    h-slave2

## 复制节点 ##

将配置好的Hadoop复制到各个节点对应位置上，通过scp传送

    scp -r hadoop-2.7.0 slave1:/data/service/
    scp -r hadoop-2.7.0 slave2:/data/service/

## 启动服务 ##

在Master服务器启动hadoop，从节点会自动启动，进入/home/hadoop/hadoop-2.7.0目录

### (1)初始化，输入命令 ###

    bin/hdfs namenode –format
    ……..
    16/10/18 00:07:34 INFO common.Storage: Storage directory /data/hdfs/dfs/name has been successfully formatted.
    16/10/18 00:07:35 INFO namenode.NNStorageRetentionManager: Going to retain 1 images with txid >= 0
    16/10/18 00:07:35 INFO util.ExitUtil: Exiting with status 0
    16/10/18 00:07:35 INFO namenode.NameNode: SHUTDOWN_MSG: 
    /************************************************************
    SHUTDOWN_MSG: Shutting down NameNode at localhost/127.0.0.1
    ************************************************************/
只要出现“successfully formatted”就表示成功了。

### (2)执行启动 ###

全部启动sbin/start-all.sh，也可以分开sbin/start-dfs.sh、sbin/start-yarn.sh

### (3)停止的话 ###

输入命令，sbin/stop-all.sh

    [root@localhost hadoop-2.7.0]# sbin/start-all.sh 
    This script is Deprecated. Instead use start-dfs.sh and start-yarn.sh
    Incorrect configuration: namenode address dfs.namenode.servicerpc-address or dfs.namenode.rpc-address is not configured.
    Starting namenodes on []
    slave-02: starting namenode, logging to /data/hadoop-2.7.0/logs/hadoop-root-namenode-localhost.out
    slave-03: starting namenode, logging to /data/hadoop-2.7.0/logs/hadoop-root-namenode-localhost.out
    localhost: starting namenode, logging to /data/hadoop-2.7.0/logs/hadoop-root-namenode-localhost.out
    slave-02: starting datanode, logging to /data/hadoop-2.7.0/logs/hadoop-root-datanode-localhost.out
    slave-03: starting datanode, logging to /data/hadoop-2.7.0/logs/hadoop-root-datanode-localhost.out
    localhost: starting datanode, logging to /data/hadoop-2.7.0/logs/hadoop-root-datanode-localhost.out
    Starting secondary namenodes [0.0.0.0]
    0.0.0.0: starting secondarynamenode, logging to /data/hadoop-2.7.0/logs/hadoop-root-secondarynamenode-localhost.out
    starting yarn daemons
    starting resourcemanager, logging to /data/hadoop-2.7.0/logs/yarn-root-resourcemanager-localhost.out
    slave-02: starting nodemanager, logging to /data/hadoop-2.7.0/logs/yarn-root-nodemanager-localhost.out
    slave-03: starting nodemanager, logging to /data/hadoop-2.7.0/logs/yarn-root-nodemanager-localhost.out
    localhost: starting nodemanager, logging to /data/hadoop-2.7.0/logs/yarn-root-nodemanager-localhost.out

### (4)验证 ###

输入命令，jps，可以看到相关信息

    master
    [root@localhost hadoop-2.7.0]# jps
    15376 Jps
    15074 NodeManager
    14963 ResourceManager
    12766 QuorumPeerMain
    Slave1
    [root@localhost ~]# jps
    13184 NodeManager
    12420 QuorumPeerMain
    13303 Jps
    Slave2
    [root@localhost ~]# jps
    13376 Jps
    13256 NodeManager
    12458 QuorumPeerMain

# Web访问 #
要先开放端口或者直接关闭防火墙

(1)浏览器打开http://ip:8088/  能显示你的集群状态
![](http://ocppiicaw.bkt.clouddn.com/hadoop2.png) 
 
(2)浏览器打开http://ip:50070/  能进行一些节点的管理
![](http://ocppiicaw.bkt.clouddn.com/hadoop3.png) 
 
除此之外，还有很多有用的端口，当然这也是和你的配置文件相关的

# 安装完成。 #
到这里为止，你已经完成了第一个任务的分布式计算, 这只是大数据应用的开始，之后的工作就是，结合自己的情况，编写程序调用Hadoop的接口，发挥hdfs、mapreduce的作用。
注意：在你重新格式化分布式文件系统之前，需要将文件系统中的数据先清除，否则，datanode将创建不成功，这一点很重要

# 文件操作 #
Hadoop使用的是HDFS，能够实现的功能和我们使用的磁盘系统类似。并且支持通配符，如*。

## 查看文件列表 ##
    ./bin/hadoop fs -ls /test
## 创建文件目录 ##
    ./bin/hadoop fs -mkdir /test
## 删除文件 ##
    ./bin/hadoop fs -rm /test/index.jsp
## 上传文件 ##
    ./bin/hadoop fs -put /opt/index.jsp /test
## 下载文件 ##
    ./bin/hadoop fs -get /test/index.jsp /opt/index.jsp
##  查看文件内容 ##
    ./bin/hadoop fs -cat /test/index.jsp
