---
title: 基于Jenkins+git+gradle发送到fir的android持续集成
tags: jenkins
categories: CICD
type: "categories"
---

# 简介 #

&#160;&#160;&#160;&#160;&#160;&#160;以前都是通过IDE(eclipse or Android Studio)手动生成apk通过QQ或者邮件发送给测试人员进行测试，现在要求对项目进行持续集成，也就是说通过某种方式定时(比如每晚凌晨三点)自动将git库中最新的代码pull下来编译打包，测试人员每天早上上班都能拿到最新的代码打包的Apk。
<!-- more -->
# 一、环境配置: #

## 1、包下载的网站： ##
http://tools.android-studio.org/

## 2、包版本： ##
android-sdk_r24.4.1-linux.tgz
apache-tomcat-8.0.30.tar.gz
gradle-2.13-all.zip
 jenkins.war

wget http://dl.google.com/android/android-sdk_r24.4.1-linux.tgz

wget https://services.gradle.org/distributions/gradle-2.13-all.zip

## 3、Centos 7  64bit; ##

## 4、 jdk1.8 ##

    JAVA_HOME=/usr/jdk1.8.0_65
    JRE_HOME=/usr/jdk1.8.0_65/jre
    PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
    CLASSPATH=:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
    export JAVA_HOME JRE_HOME PATH CLASSPATH
![](http://ocppiicaw.bkt.clouddn.com/jenkinsjdk.png)

## 5、 android sdk ##

    export SDK_HOME=/usr/android-sdk-linux
    export PATH=$SDK_HOME/tools:$SDK_HOME/platform-tools:$PATH
![](http://ocppiicaw.bkt.clouddn.com/android.png)
更新

    tools/android update sdk --no-ui
    ls /usr/android-sdk-linux/build-tools

24.0.1

查看全部的build-tools，根据自己项目所依赖的build-tools更新
`android list sdk --all` 
![](http://ocppiicaw.bkt.clouddn.com/6.png)  

    android update sdk --no-ui --all --filter 6

## 6、 gradle2.13 ##

    GRADLE_HOME=/usr/gradle-2.13
    export PATH=$PATH:$GRADLE_HOME/bin
 
![](http://ocppiicaw.bkt.clouddn.com/7.png)
# 二、安装jenkins。 #
链接:http://mirrors.jenkins-ci.org/war/latest/jenkins.war。

将下载的jenkins.war包直接放到tomcat下的webapps目录，启动tomcat,在浏览器输入:127.0.0.1:8080/Jenkins
![](http://ocppiicaw.bkt.clouddn.com/8.png) 

# 三，进入设置，安装插件 #

- Git plugin
- Gradle plugin
- GitLab Plugin
- Git client plugin
- Gitlab Hook Plugin


# 四、系统配置 #
## 1、android-sdk配置 ##
![](http://ocppiicaw.bkt.clouddn.com/9.png) 

## 2、JDK: ##
 ![](http://ocppiicaw.bkt.clouddn.com/10.png)
## 3、Git: ##
 ![](http://ocppiicaw.bkt.clouddn.com/11.png)
## 4、Gradle: ##
 ![](http://ocppiicaw.bkt.clouddn.com/12.png)


## 5、邮件配置  ##
 
![](http://ocppiicaw.bkt.clouddn.com/13.png)
![](http://ocppiicaw.bkt.clouddn.com/14.png)

# 五、项目配置： #
 
![](http://ocppiicaw.bkt.clouddn.com/16.png)
![](http://ocppiicaw.bkt.clouddn.com/17.png)
![](http://ocppiicaw.bkt.clouddn.com/18.png)

 
# 六、build #
 ![](http://ocppiicaw.bkt.clouddn.com/19.png)
# 七、检测结果 #
Build完以后检查一下
目录下生成了类似于如下的Apk，则表示这个系统是OK的

![](http://ocppiicaw.bkt.clouddn.com/20.png)
 

状态是乌云的时候是因为之前有构建失败的，删除再重新构建一次就变成太阳了
 
![](http://ocppiicaw.bkt.clouddn.com/21.png)
![](http://ocppiicaw.bkt.clouddn.com/22.png)
 

# 八、自动发布android应用到FIR.im #
## 1、介绍 ##
&#160;&#160;&#160;&#160;&#160;&#160;fir.im是国内首家为移动开发者提供 App 免费托管分发服务的平台，为移动开发者提供极速测试发布、崩溃收集分析、用户反馈收集等一系列开发测试效率工具服务，能够让开发者更专注于产品开发与优化。

## 2、fir.im Jenkins 插件使用方法 ##
http://blog.fir.im/jenkins/

## 3、下载安装插件 ##
http://7xju1s.com1.z0.glb.clouddn.com/fir-plugin-0629.hpi

 ![](http://ocppiicaw.bkt.clouddn.com/23.png)

## 4、进入在 配置 中找到 增加构建后操作步骤 ，并选择 Upload to fir.im 添加到 Jenkins 项目中 ##。
![](http://ocppiicaw.bkt.clouddn.com/24.png)
 
## 5、fir的APIToken需要登录fir获取 ##
 
![](http://ocppiicaw.bkt.clouddn.com/25.png)
## 6、构建后操作  ##

![](http://ocppiicaw.bkt.clouddn.com/26.png)
![](http://ocppiicaw.bkt.clouddn.com/27.png)

 
## 7、构建完成！查看fir上的Build是否为现在及成功！ ##
 ![](http://ocppiicaw.bkt.clouddn.com/28.png)
![](http://ocppiicaw.bkt.clouddn.com/29.png)
 
# 九、错误解决 #
遇到错误可以Clean build –stacktace –debug 查看问题详情

'command '/usr/android-sdk-linux/build-tools/23.0.3/aapt'' finished with non-zero exit value 127
如果缺少包ld-linux.so.2
搜索 
yum whatprovides ld-linux.so.2 

 ![](http://ocppiicaw.bkt.clouddn.com/30.png)

安装
yum install -y glibc-2.17-105.el7.i686

