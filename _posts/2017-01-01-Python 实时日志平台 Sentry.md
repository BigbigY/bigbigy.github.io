---
title: 一步步构建 Python 实时日志平台 Sentry
tags: [Senrty]
categories: 日志
type: "categories"
author: "luck"
header-img: "img/home-bg-art.jpg"
---



# 简介 #
&#160;&#160;&#160;&#160;&#160;&#160;&#160;&#160;Sentry是一个实时的事件日志和聚合平台，基于 Django 构建。也是个很好用的错误日志服务器，可以将程序错误的详细情况集中捕获，并提供一个很漂亮的Web界面来浏览错误。 Sentry本身是用python写的，但它支持python、php、ruby、iOS等多种语言。


&#160;&#160;&#160;&#160;&#160;&#160;&#160;Sentry 可以帮助你将 Python 程序的所有 exception 自动记录下来，然后在一个好用的 UI 上呈现和搜索。处理 exception 是每个程序的必要部分，所以 Sentry 也几乎可以说是所有项目的必备组件。

<!-- more -->

# 一、Pip #
pip是python的一个很好的包管理软件，类似npm对于nodejs的关系。似乎pip一般不随python自动安装，但是一个叫easy_install的命令一般都是自带的，所以我们可以通过
sudo easy_install pip
来安装，至于为什么不直接用easy_install来安装所有依赖，通俗一点来讲，pip更流行

# 二、	Virtualenv环境 #
virtualenv就是用来为一个应用创建一套“隔离”的Python运行环境

    #pip install virtualenv
    
    #virtualenv sentry
    New python executable in /opt/sentry/bin/python
    Please make sure you remove any previous custom paths from your /root/.pydistutils.cfg file.
    Installing setuptools, pip, wheel...done.
    
    #source sentry/bin/activate
    就能激活出一个新的环境，在这个新环境下我们在进行后续操作
    # deactivate 
    退出环境

# 三、	安装sentry #
#pip install sentry==7.4.3

# 四、	安装mysql #
#service postgresql initdb
#/bin/systemctl start  postgresql.service
#su – postgres

略
# 五、	安装redis #
yum install redis –y
systemctl start  redis.service

#六、	配置 #
初始化创建配置文件

    # sentry init /root/sentry/sentry.conf.py
    
    Vim /root/sentry/sentry.conf.py


    #配置文件中你需要设置几处
    
     #数据库配置，推荐Postgresql，其次是Mysql #修改mysql配置
    
    DATABASES = {
    'default': {
    'ENGINE': 'django.db.backends.mysql',
    'NAME': 'sentry',
    #'NAME': os.path.join(CONF_ROOT, 'sentry.db'),
    'USER': 'root',
    'PASSWORD': '123456',
    'HOST': 'localhost',
    'PORT': '3306',
    }
    }
    SENTRY_ADMIN_EMAIL = 'wangyangyang@rqbao.com'
    
    	Redis配置
    SENTRY_REDIS_OPTIONS = {
    'hosts': {
    0: {
    'host': '127.0.0.1',
    'port': 6379,
    }
    }
    }
    web服务配置
    
    BROKER_URL = 'redis://localhost:6379'
    SENTRY_URL_PREFIX = 'http://sentry.example.com'
    SENTRY_WEB_HOST = '0.0.0.0'
    SENTRY_WEB_PORT = 9000
    SENTRY_WEB_OPTIONS = {
    # 'workers': 3,  # the number of gunicorn workers
    # 'secure_scheme_headers': {'X-FORWARDED-PROTO': 'https'},
    }
    邮件服务配置
    EMAIL_HOST = 'smtp.exmail.qq.com'
    EMAIL_HOST_PASSWORD = 'w15034619520'
    EMAIL_HOST_USER = 'wangyangyang@rqbao.com'
    EMAIL_PORT = 25
    SERVER_EMAIL = 'smtp.exmail.qq.com'
 

----------
   
    我的配置文件：
    from sentry.conf.server import *
    import os.path
    CONF_ROOT = os.path.dirname(__file__)
    
    DATABASES = {
    'default': {
    'ENGINE': 'django.db.backends.mysql',
    'NAME': 'sentry',
    'USER': 'sentryadmin',
    'HOST': '10.1.1.10',
    'PORT': '3306',
    }
    }
    
    SENTRY_USE_BIG_INTS = True
    
    SENTRY_ADMIN_EMAIL = 'wangyangyang@rqbao.com'
    
    SENTRY_REDIS_OPTIONS = {
    'hosts': {
    0: {
    'host': '127.0.0.1',
    'port': 6379,
    }
    }
    }
    
    SENTRY_CACHE = 'sentry.cache.redis.RedisCache'
    
    CELERY_ALWAYS_EAGER = False
    BROKER_URL = 'redis://localhost:6379'
    SENTRY_RATELIMITER = 'sentry.ratelimits.redis.RedisRateLimiter'
    SENTRY_BUFFER = 'sentry.buffer.redis.RedisBuffer'
    SENTRY_QUOTAS = 'sentry.quotas.redis.RedisQuota'
    SENTRY_TSDB = 'sentry.tsdb.redis.RedisTSDB'
    
    SENTRY_FILESTORE = 'django.core.files.storage.FileSystemStorage'
    SENTRY_FILESTORE_OPTIONS = {
    'location': '/tmp/sentry-files',
    }
    
    SENTRY_WEB_HOST = '0.0.0.0'
    SENTRY_WEB_PORT = 9000
    SENTRY_WEB_OPTIONS = {
    }
    EMAIL_BACKEND = 'django.core.mail.backends.smtp.EmailBackend'
    
    EMAIL_HOST = 'smtp.exmail.qq.com'
    EMAIL_HOST_PASSWORD = 'w15034619520'
    EMAIL_HOST_USER = 'wangyangyang@rqbao.com'
    EMAIL_PORT = 465 
    EMAIL_USE_TLS = False
    
    SERVER_EMAIL = 'wangyangyang@rqbao.com'
    
    MAILGUN_API_KEY = ''
    
    SECRET_KEY = 'jS2nEMPp2AAbbEWI92JO7ZbpZ+vZ90E9Cy7m5lVn94K4WQ6ibMeMnw=='
    
    TWITTER_CONSUMER_KEY = ''
    TWITTER_CONSUMER_SECRET = ''
    
    FACEBOOK_APP_ID = ''
    FACEBOOK_API_SECRET = ''
    
    GOOGLE_OAUTH2_CLIENT_ID = ''
    GOOGLE_OAUTH2_CLIENT_SECRET = ''
    
    GITHUB_APP_ID = ''
    GITHUB_API_SECRET = ''
    
    TRELLO_API_KEY = ''
    TRELLO_API_SECRET = ''
    
    BITBUCKET_CONSUMER_KEY = ''
    BITBUCKET_CONSUMER_SECRET = ''


# 七、 启动成功！ #
    为sentry项目初始化数据
    \# sentry --config=/sentry/sentry.conf.py upgrade
    创建新用户
    \# sentry --config=/sentry/sentry.conf.py createuser
    然后就可以启动服务了
    \# sentry --config=/sentry/sentry.conf.py start
    另外，还需要启动Worker
    \# sentry --config=/sentry/sentry.conf.py  celery worker -B

 

浏览器访问： http://IP:9000
![](http://ocppiicaw.bkt.clouddn.com/sentrylogin.png)

http://IP:9000/admin/

![](http://ocppiicaw.bkt.clouddn.com/sentryadmin.png) 

# 八、客户端： #
https://docs.getsentry.com/hosted/clients/java/modules/log4j/

    #pip install virtualenv
    #virtualenv sentry_client
    #source sentry_clinent/bin/activate
    #pip install raven

复制API key链接

 


配置log4j(我配置的是log4j.properties)

    #SentryAppender
    log4j.rootLogger=WARN, Console, RollingFile ,SentryAppender
    log4j.appender.SentryAppender=net.kencochrane.raven.log4j.SentryAppender
          log4j.appender.SentryAppender.dsn=http://9189bf0a8b064d6680eecec231538c0:634e759226b84ae5b745d5d9fefe55c9@IP:9000/3



测试：
    /root/sentry/bin/raven --help

     /sentry_client/bin/raven test http://933fd1686b514112b083a5fa13f08132:be807dd201574507bef0aa5d8e67181c@IP:9000/2
    Using DSN configuration:
      http://933fd1686b514112b083a5fa13f08132:be807dd201574507bef0aa5d8e67181c@IP:9000/2
    
    Client configuration:
      base_url   : http://10.0.0.1:9000
      project: 2
      public_key : 933fd1686b514112b083a5fa13f08132
      secret_key : be807dd201574507bef0aa5d8e67181c
    
    Sending a test message... Event ID was '33d7b85257ce4faaab2a8c968f607269'
    success!

展示

![](http://ocppiicaw.bkt.clouddn.com/sentry设置.png ) 

![](http://ocppiicaw.bkt.clouddn.com/sentry统计.png ) 

![](http://ocppiicaw.bkt.clouddn.com/sentry团队.png ) 
 
![](http://ocppiicaw.bkt.clouddn.com/sentry趋势图.png ) 

![](http://ocppiicaw.bkt.clouddn.com/sentry标签.png )

![](http://ocppiicaw.bkt.clouddn.com/sentry异常.png )
 
 
![](http://ocppiicaw.bkt.clouddn.com/sentry请求.png ) 
# 八、	错误解决 #
    错误
    error: Python.h: No such file or directory
    解决
    yum install python-devel –y
    
    错误
    error: libxml/xmlversion.h: No such file or directory
    error: libxml/xpath.h: No such file or directory
    解决
    yum install libxslt-devel –y
    
    错误
    error: openssl/opensslv.h: No such file or directory
    解决
    yum install openssl-devel –y
    
    错误
    Error loading MySQLdb module: No module named MySQLdb
    pip install MySQL-python
    
    错误
    EnvironmentError: mysql_config not found
    yum -y install mysql-devel 
    
    错误
    error: libpq-fe.h: No such file or directory
    yum install postgresql-devel -y