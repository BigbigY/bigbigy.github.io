---
title: 3-Elastic stack之Logstach
tags: [Elastic stack,Logstach]
categories: 日志搜索
type: "categories"
author: "luck"
header-img: "img/post-bg-e2e-ux.jpg"
---

# Logstash简介

Logstash是具有实时流水线功能的开源数据收集引擎。可以接收，处理，转发日志的工具。支持系统日志，webserver日志，错误日志，应用日志，总之包括所有可以抛出来的日志类型。Logstash可以动态统一来自不同来源的数据，并将数据归一化到您选择的目的地。用于各种先进的下游分析和可视化用例。
![](http://ocppiicaw.bkt.clouddn.com/elastic/elk1.png)


# 一、安装依赖环境

>Logstash需要Java 8.不支持Java 9

#### 1、卸载自带jdk，安装java环境(我的系统版本是Centos6.5，7的话可以忽略此步骤)：

```
$ rpm -qa | grep java
java-1.6.0-openjdk-1.6.0.41-1.13.13.1.el6_8.x86_64
tzdata-java-2017a-1.el6.noarch
java-1.7.0-openjdk-1.7.0.131-2.6.9.0.el6_8.x86_64
$ rpm -e --nodeps java-1.6.0-openjdk-1.6.0.41-1.13.13.1.el6_8.x86_64
$ rpm -e --nodeps tzdata-java-2017a-1.el6.noarch
$ rpm -e --nodeps java-1.7.0-openjdk-1.7.0.131-2.6.9.0.el6_8.x86_64
```

#### 2、 修改环境变量

```
$ tar xf jdk-8u45-linux-x64.tar.gz -C /usr/
$ vim /etc/profile
JAVA_HOME=/usr/jdk1.8.0_45
JRE_HOME=/usr/jdk1.8.0_45/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
CLASSPATH=:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export JAVA_HOME JRE_HOME PATH CLASSPATH
```

#### 3、测试 

```
$ java -version
java version "1.8.0_45"
Java(TM) SE Runtime Environment (build 1.8.0_45-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.45-b02, mixed mode)
```

# 二、 安装Logstash（YUM）

#### 1、 下载并安装公共签名密钥：

```
$ rpm --import https://artifacts.elastic.co/GPG-KEY-elasticsearch
$ sudo yum install logstash
```

#### 2、/etc/yum.repos.d/，在具有.repo后缀的文件中将以下内容添加到您的目录中logstash.repo

```
[logstash-5.x]
name=Elastic repository for 5.x packages
baseurl=https://artifacts.elastic.co/packages/5.x/yum
gpgcheck=1
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch
enabled=1
autorefresh=1
type=rpm-md
```

#### 3. 安装

```
$ sudo yum install logstash
$ /usr/share/logstash/bin/logstash --version
logstash 5.3.0
```

#### 4、目录结构

- home ```/usr/share/logstash```
- bin ```/usr/share/logstash/bin```
- settings ```/usr/share/logstash/config```
- conf ```/usr/share/logstash/pipeline```
- plugins ```/usr/share/logstash/plugins```
 
# 三、测试

```
$ /usr/share/logstash/bin/logstash -e 'input { stdin { } } output { stdout {} }'
...
20:39:18.936 [Api Webserver] INFO  logstash.agent - Successfully started Logstash API endpoint {:port=>9600}
Hello,world!
2017-04-12T12:39:35.959Z 0.0.0.0 Hello,world!
```

# 四、使用Logstash解析日志

>在创建Logstash管道之前，您将配置Filebeat以将日志行发送到Logstash。该Filebeat客户端是一个轻量级的，资源友好的工具，从服务器上的文件收集日志，这些日志转发到处理您的Logstash实例。Filebeat设计用于可靠性和低延迟。Filebeat在主机上具有轻量的资源占用空间，该Beats input插件可最大限度地减少对Logstash实例的资源需求。
默认的Logstash安装包括Beats input插件。Beats输入插件使Logstash能够接收Elastic Beats框架中的事件，这意味着任何Beat编写以使用Beats框架（如Packetbeat和Metricbeat）也可以将事件数据发送到Logstash。

#### 1、安装Filebeat后，需要进行配置。打开filebeat.ymlFilebeat安装目录中的文件，并用以下行替换内容。确保paths指向logstash-tutorial.log您先前下载的示例Apache日志文件 ：

```
filebeat.prospectors:
- input_type: log
  paths:
    - /data/bbtree/www/luck-test/logs/localhost_access_log.*.txt
output.logstash:
  hosts: ["10.0.13.104:5043"]
```

#### 2、使用以下命令运行Filebeat：

```
sudo ./filebeat -e -c filebeat.yml -d "publish"
```

Filebeat将尝试在端口5043上连接。直到Logstash从活动的Beats插件开始，该端口上将不会有任何答案，因此您在该端口上无法连接的任何消息现在都是正常的。

#### 3、配置Logstash以进行文件输入编辑

接下来，您创建一个使用Beats输入插件从Beats接收事件的Logstash配置管道。
以下文本表示配置管道的骨架：
内容first-pipeline.conf应如下所示：

```
$ vim /etc/logstash/conf.d/first-pipeline.conf
# The # character at the beginning of a line indicates a comment. Use
# comments to describe your configuration.
input {
    beats {
        port => "5043"
    }
}
# The filter part of this file is commented out to indicate that it is
# optional.
# filter {
#
# }
output {
    stdout { codec => rubydebug }
}
```

#### 4、验证配置

```
$ /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/first-pipeline.conf --config.test_and_exit
WARNING: Could not find logstash.yml which is typically located in $LS_HOME/config or /etc/logstash. You can specify the path using --path.settings. Continuing using the defaults
Could not find log4j2 configuration at path /usr/share/logstash/config/log4j2.properties. Using default config which logs to console
Configuration OK
```

```--config.test_and_exit``` 选项解析您的配置文件并报告任何错误

#### 5、启动

```
$ /usr/share/logstash/bin/logstash -f /etc/logstash/conf.d/first-pipeline.conf --config.reload.automatic
...略...
{
    "@timestamp" => 2017-04-12T13:19:14.060Z,
        "offset" => 21679,
      "@version" => "1",
    "input_type" => "log",
          "beat" => {
        "hostname" => "filebate",
            "name" => "filebate",
         "version" => "5.3.0"
    },
          "host" => "filebate",
        "source" => "/data/bbtree/www/luck-test/logs/localhost_access_log.2017-04-12.txt",
       "message" => "10.0.13.97 - - [12/Apr/2017:21:10:22 +0800] \"GET /favicon.ico HTTP/1.1\" 200 21630",
          "type" => "log",
          "tags" => [
        [0] "beats_input_codec_plain_applied"
    ]
}
```

发现已经接受到了filebeat的数据
```--config.reload.automatic``` 选项启用自动配置重新加载，以便您每次修改配置文件时不必停止并重新启动Logstash。

#### 6、测试

**测试数据**

```
$ echo "从前有座山" >> /data/bbtree/www/luck-test/logs/localhost_access_log.2017-04-12.txt
```

**filebat获取到数据**

```
2017/04/12 13:57:44.277723 log.go:91: INFO Harvester started for file: /data/bbtree/www/luck-
test/logs/localhost_access_log.2017-04-12.txt
2017/04/12 13:57:45.144877 client.go:214: DBG  Publish: {
  "@timestamp": "2017-04-12T13:57:44.277Z",
  "beat": {
    "hostname": "filebate",
    "name": "filebate",
    "version": "5.3.0"
  },
  "input_type": "log",
  "message": "从前有座山",
  "offset": 21695,
  "source": "/data/bbtree/www/luck-test/logs/localhost_access_log.2017-04-12.txt",
  "type": "log"
}
```

**logstach接受到数据**

```
{
    "@timestamp" => 2017-04-12T13:57:44.277Z,
        "offset" => 21695,
      "@version" => "1",
          "beat" => {
        "hostname" => "filebate",
            "name" => "filebate",
         "version" => "5.3.0"
    },
    "input_type" => "log",
          "host" => "filebate",
        "source" => "/data/bbtree/www/luck-test/logs/localhost_access_log.2017-04-12.txt",
       "message" => "从前有座山",
          "type" => "log",
          "tags" => [
        [0] "beats_input_codec_plain_applied"
    ]
}
```

#### 7、使用Grok Filter Plugin 解析Web日志

现在你有一个工作流水线从Filebeat读取日志行。但是您会注意到日志消息的格式不理想。您要解析日志消息以从日志中创建特定的命名字段。为此，您将使用```grok```过滤器插件。
Web服务器日志示例中的代表行如下所示：
```
83.149.9.216  -   -  [04 / Jan / 2015：05：13：42 +0000]“GET /presentations/logstash-monitorama-2013/images/kibana-search.png。9.216 - - [ 04 / Jan / 2015 ：05 ：13 ：42 + 0000 ] “GET /presentations/logstash-monitorama-2013/images/kibana-search.png    
HTTP / 1.1“200 203023”http://semicomplete.com/presentations/logstash-monitorama-2013/“”Mozilla / 5.0（Macintosh; Intel 200 203023 “http://semicomplete.com/presentations/logstash-monitorama-2013/” “的Mozilla / 5.0（Macintosh上;英特尔  
Mac OS X 10_9_1）AppleWebKit / 537.36（KHTML，如Gecko）Chrome / 32.0.1700.77 Safari / 537.36“
```

行的开头的IP地址很容易识别，括号中的时间戳也是如此。要解析数据，您可以使用%{COMBINEDAPACHELOG}grok模式，哪些结构来自Apache日志的行使用以下模式：
信息     字段名称
IP Address   ```clientip ```
User ID  ```ident```
User Authentication ```auth```
timestamp  ```timestamp```
HTTP Verb ```verb```
Request body ```request```
HTTP Version ```httpversion```
HTTP Status Code  ```response```
Bytes served  ```bytes```
Referrer URL  ```referrer```
User agent  ```agent```

#### 8、编辑first-pipeline.conf文件并用以下filter文本替换整个部分：

```
$ cat /etc/logstash/conf.d/first-pipeline.conf
# comments to describe your configuration.
input {
    beats {
        port => "5043"
    }
}
 filter {
    grok {
        match => { "message" => "%{COMBINEDAPACHELOG}"}
    }
}

output {
    stdout { codec => rubydebug }
}
```

保存更改。由于您已启用自动配置重新加载，因此您不必重新启动Logstash来接收更改。但是，您确实需要强制Filebeat从头开始读取日志文件。要执行此操作，请转到Filebeat正在运行的终端窗口，然后按Ctrl + C关闭Filebeat。然后删除Filebeat注册表文件。例如，运行：
```
sudo rm /var/lib/filebeat/registry
```

接下来，使用以下命令重新启动Filebeat：

```
sudo ./filebeat -e -c filebeat.yml -d "publish"
```

在使用grok模式处理日志文件后，事件将具有以下JSON表示形式：

```
{
        "request" => "/presentations/logstash-monitorama-2013/images/kibana-search.png",
          "agent" => "\"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36\"",
         "offset" => 22833,
           "auth" => "-",
          "ident" => "-",
     "input_type" => "log",
           "verb" => "GET",
         "source" => "/data/bbtree/www/luck-test/logs/localhost_access_log.2017-04-12.txt",
        "message" => "83.149.9.216 - - [04/Jan/2015:05:13:42 +0000] \"GET /presentations/logstash-monitorama-2013/images/kibana-search.png HTTP/1.1\" 200 203023 \"http://semicomplete.com/presentations/logstash-monitorama-2013/\" \"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36\"",
           "type" => "log",
           "tags" => [
        [0] "beats_input_codec_plain_applied"
    ],
       "referrer" => "\"http://semicomplete.com/presentations/logstash-monitorama-2013/\"",
     "@timestamp" => 2017-04-12T14:41:29.775Z,
       "response" => "200",
          "bytes" => "203023",
       "clientip" => "83.149.9.216",
       "@version" => "1",
           "beat" => {
        "hostname" => "filebate",
            "name" => "filebate",
         "version" => "5.3.0"
    },
           "host" => "filebate",
    "httpversion" => "1.1",
      "timestamp" => "04/Jan/2015:05:13:42 +0000"
}
```

#### 9、使用Geoip Filter插件

将Logstash实例配置为使用geoip过滤器插件，将以下行添加到文件的filter部分first-pipeline.conf：
```
input {
    beats {
        port => "5043"
    }
}
filter {
    grok {
        match => { "message" => "%{COMBINEDAPACHELOG}"}
    }
    geoip {
        source => "clientip"
    }
}
output {
    stdout { codec => rubydebug }
}
```


#### 10、保存更改。要强制Filebeat从头开始读取日志文件，请先关闭Filebeat（按Ctrl + C），删除注册表文件，重新启动Filebeat：

**查看已经geoip部分，事件现在包含地理位置信息：**
```
{
        "request" => "/presentations/logstash-monitorama-2013/images/kibana-search.png",
          "agent" => "\"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36\"",
          "geoip" => {
             "city_name" => "Beijing",
              "timezone" => "Asia/Shanghai",
                    "ip" => "111.149.9.216",
              "latitude" => 39.9289,
         "country_code2" => "CN",
          "country_name" => "China",
        "continent_code" => "AS",
         "country_code3" => "CN",
           "region_name" => "Beijing",
              "location" => [
            [0] 116.3883,
            [1] 39.9289
        ],
             "longitude" => 116.3883,
           "region_code" => "11"
    },
         "offset" => 23312,
           "auth" => "-",
          "ident" => "-",
     "input_type" => "log",
           "verb" => "GET",
         "source" => "/data/bbtree/www/luck-test/logs/localhost_access_log.2017-04-12.txt",
        "message" => "111.149.9.216 - - [04/Jan/2015:05:13:42 +0000] \"GET /presentations/logstash-monitorama-2013/images/kibana-search.png HTTP/1.1\" 200 203023 \"http://semicomplete.com/presentations/logstash-monitorama-2013/\" \"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/32.0.1700.77 Safari/537.36\"",
           "type" => "log",
           "tags" => [
        [0] "beats_input_codec_plain_applied"
    ],
       "referrer" => "\"http://semicomplete.com/presentations/logstash-monitorama-2013/\"",
     "@timestamp" => 2017-04-12T15:03:30.739Z,
       "response" => "200",
          "bytes" => "203023",
       "clientip" => "111.149.9.216",
       "@version" => "1",
           "beat" => {
        "hostname" => "filebate",
            "name" => "filebate",
         "version" => "5.3.0"
    },
           "host" => "filebate",
    "httpversion" => "1.1",
      "timestamp" => "04/Jan/2015:05:13:42 +0000"
}
```

### 11、数据发送到到Elasticsearch

Web日志被分解成特定的字段，Logstash管道可以将数据索引到Elasticsearch集群中。编辑first-pipeline.conf文件并用以下output文本替换整个部分：
```
# cat /etc/logstash/conf.d/first-pipeline.conf
# comments to describe your configuration.
input {
    beats {
        port => "5043"
    }
}
filter {
    grok {
        match => { "message" => "%{COMBINEDAPACHELOG}"}
    }
    geoip {
        source => "clientip"
    }
}
 
#output {
#    stdout { codec => rubydebug }
#}
 
output {
    elasticsearch {
        hosts => [ "10.0.13.107:9200" ]
    }
}
```
#### 12、 保存更改。要强制Filebeat从头开始读取日志文件，请先关闭Filebeat（按Ctrl + C），删除注册表文件，然后使用以下命令重新启动Filebeat：

```
$ filebeat.sh -e -c filebeat.yml -d "publish"
```
#### 13、测试

```
curl -XGET 'localhost:9200/logstash-$DATE/_search?pretty&q=response=200'
```

# 五、拼接多个输入和输出插件编辑
https://www.elastic.co/guide/en/logstash/5.3/multiple-input-output-plugins.html