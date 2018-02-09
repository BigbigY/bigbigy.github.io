---
title: 企业级wiki之confluence完美破解与汉化
tags: confluence
categories: wiki
type: "categories"
---



# 简介 #
Confluence是一个专业的企业知识管理与协同软件，也可以用于构建企业wiki。通过它可以实现团队成员之间的协作和知识共享。Confluence为团队提供一个协作环境。在这里，团队成员齐心协力，各擅其能，协同地编写文档和管理项目。从此打破不同团队、不同部门以及个人之间信息孤岛的僵局，Confluence真正实现了组织资源共享。Confluence使用简单，但它强大的编辑和站点管理特征能够帮助团队成员之间共享信息、文档协作、集体讨论，信息推送。
<!-- more -->

# 前提条件 #
- jdk必须1.7以上  
- MySQL5.5以上    
- 8090端口没有被占用
- 创建数据库：confluence  账号及密码

# 需要的文件 #
- confluence5.10安装包：atlassian-confluence-5.10.0-x64.bin  。
- 破解包：confluence5.x-crack.zip 
- mysql-connector-java-5.1.32-bin.jar
- 中文语言包：Confluence-5.10.0-rc1-language-pack-zh_CN.jar


#  运行安装包 #
     chmod +x atlassian-confluence-5.10.0-x64.bin 
     . /atlassian-confluence-5.10.0-x64.bin
    [root@iZ25ffu62qkZ confluence5.10]# ./atlassian-confluence-5.10.0-x64.bin 
    Unpacking JRE ...
    Starting Installer ...
    Sep 27, 2016 1:49:05 PM java.util.prefs.FileSystemPreferences$1 run
    INFO: Created user preferences directory.
    Sep 27, 2016 1:49:05 PM java.util.prefs.FileSystemPreferences$2 run
    INFO: Created system preferences directory in java.home.
    
    This will install Confluence 5.10.0 on your computer.
    OK [o, Enter], Cancel [c]
    o <--
    Choose the appropriate installation or upgrade option.
    Please choose one of the following:
    Express Install (uses default settings) [1], 
    Custom Install (recommended for advanced users) [2, Enter], 
    Upgrade an existing Confluence installation [3]
    1 <--
    See where Confluence will be installed and the settings that will be used.
    Installation Directory: /opt/atlassian/confluence 
    Home Directory: /var/atlassian/application-data/confluence 
    HTTP Port: 8090 
    RMI Port: 8000 
    Install as service: Yes 
    Install [i, Enter], Exit [e]
    i <--
    
    Extracting files ...
       
    
    Please wait a few moments while Confluence starts up.
    Launching Confluence ...
    Installation of Confluence 5.10.0 is complete
    Your installation of Confluence 5.10.0 is now ready and can be accessed via
    your browser.
    Confluence 5.10.0 can be accessed at http://localhost:8090
    Finishing installation ...

接下来在浏览器中打开http://localhost:8090,选择production installation，点next，下一步直接next，最后记下Server ID，留着破解用。
![](http://ocppiicaw.bkt.clouddn.com/cf1.png) 
![](http://ocppiicaw.bkt.clouddn.com/cf2.png) 

记住Server ID,破解时会用到
![](http://ocppiicaw.bkt.clouddn.com/cf3.png)
 

 
停掉Confluence 服务 

    service confluence stop  

# 在自己虚拟机或者其他有图形界面系统操作 #

    将confluence5.1-crack.zip 解压 
    将/opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-decoder-v2-3.2.jar 复制出来。替换confluence5.1-crack 中的atlassian-extras-2.4.jar
    chmod +x keygen.sh
    ./keygen.sh   #执行破解文件
![](http://ocppiicaw.bkt.clouddn.com/cf4.png)

##  注： ##
必须是在图形界面下，因为这个运行需要图形。如果没有图形，那么就会报错。(可以在有图形的其他机器上执行，然后将atlassian-extras-2.4.jar复制回来)

  输入name（随便写）和Server ID，点.patch，选择当前目录下的atlassian-extras-2.4.jar，点.gen!  得到key.
![](http://ocppiicaw.bkt.clouddn.com/cf5.png)

保存key


- 将破解好的包atlassian-extras-2.4.jar    复制到 /opt/atlassian/confluence/confluence/WEB-INF/lib/atlassian-extras-decoder-v2-3.2.jar
- 复制mysql-connector-java-5.1.32-bin.jar 到 /opt/atlassian/confluence/confluence/WEB-INF/lib/
- 复制中文包Confluence-5.10.0-rc1-language-pack-zh_CN.jar 到/opt/atlassian/confluence/confluence/WEB-INF/lib/
- 启动confluence:  service confluence start
 
# 安装confluence #

![](http://ocppiicaw.bkt.clouddn.com/cf6.png)

选择自己数据库版本
![](http://ocppiicaw.bkt.clouddn.com/cf7.png)
![](http://ocppiicaw.bkt.clouddn.com/cf8.png)

配置数据库连接
![](http://ocppiicaw.bkt.clouddn.com/cf9.png)
![](http://ocppiicaw.bkt.clouddn.com/cf10.png)
![](http://ocppiicaw.bkt.clouddn.com/cf11.png)
![](http://ocppiicaw.bkt.clouddn.com/cf12.png)
![](http://ocppiicaw.bkt.clouddn.com/cf13.png)
![](http://ocppiicaw.bkt.clouddn.com/cf14.png)
 

# CentOS 7 安装图形化桌面 #
## 安装包 ##
    yum groupinstall "GNOME Desktop" "Graphical Administration Tools" -y
## 切换图形界面 ##
    init5 or startx

