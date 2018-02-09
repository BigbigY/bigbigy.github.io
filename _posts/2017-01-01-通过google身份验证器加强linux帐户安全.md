---
title: 通过google身份验证器加强linux帐户安全
tags: google
categories: 安全
type: "categories"
---

![](http://ocppiicaw.bkt.clouddn.com/google/google1.jpg)
<!--more-->

谷歌身份验证器生成的是动态验证码，默认30秒更新。修改配置，SSH登录必须在输入密码之前输入动态验证码。即使账号和密码泄露，验证码输入错误，仍然无法登录。苹果或者安卓手机端可以安装身份验证器App读取验证码。

# 1.安装前准备 #

- Git 地址：https://github.com/google/google-authenticator.git
- 关闭Selinux ：setenforce 0
- 安装依赖：yum -y install gcc make pam-devel libpng-devel libtool wget git
- 添加阿里云epel 源：
  RHEL 6/Centos 6
  wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-6.repo
  RHEL 7/Centos 7
  wget -O /etc/yum.repos.d/epel.repo http://mirrors.aliyun.com/repo/epel-7.repo
- 安装Qrencode，谷歌身份验证器需要调用该程序生成二维码并显示
  yum install -y qrencode

## 2.安装谷歌身份验证器 ##

    git clone https://github.com/google/google-authenticator.git
    cd /root/google-authenticator/libpam

# 编译并安装 #

    [root@node1 libpam]# ./bootstrap.sh && ./configure && make && make install

复制google 身份验证器pam模块到系统下

    cp /usr/local/lib/security/pam_google_authenticator.so /lib64/security/

# 3.配置/etc/pam.d/sshd #

auth       required     pam_google_authenticator.so
添加图中红色框内的一行内容 

![](http://ocppiicaw.bkt.clouddn.com/google/google2.png)

# 4.修改SSH服务配置/etc/ssh/sshd_config #

ChallengeResponseAuthentication no->yes
sed -i 's#^ChallengeResponseAuthentication no#ChallengeResponseAuthentication yes#' /etc/ssh/sshd_config

# 5.重启SSH服务 #

a)RHEL 6 / Centos6
service sshd restart
b)RHEL7 /Centos 7
systemctl resart sshd

# 6.安装完成，开始验证。 #

a)切换到需要验证的系统账户，本案例使用root用户。
b)运行程序
    [root@DevOps24h ~]# google-authenticator
    Do you want authentication tokens to be time-based (y/n) y
    https://www.google.com/chartchs=200x200&chld=M|0&cht=qr&chl=otpauth://totp/root@DevOps24h%3Fsecret%3DJS57SLVUDEEA7SQ7LD6BEBWGAA%26issuer%3DDevOps24h
    
![](http://ocppiicaw.bkt.clouddn.com/google/google3.png)
     
    使用谷歌验证器扫描条码
    
    Your new secret key is: UCHS76IOUFPE24AH2ZNXZZB7CM
    Your verification code is 153002
    Your emergency scratch codes are: **<<<<后备验证码保存好**
      14452348
      62320562
      50956254
      81894167
      45465522
    Do you want me to update your "/root/.google_authenticator" file (y/n) y
    Do you want to disallow multiple uses of the same authentication
    token? This restricts you to one login about every 30s, but it increases
    your chances to notice or even prevent man-in-the-middle attacks (y/n)
    Do you want to disallow multiple uses of the same authentication
    token? This restricts you to one login about every 30s, but it increases
    your chances to notice or even prevent man-in-the-middle attacks (y/n) y
    By default, tokens are good for 30 seconds. In order to compensate for
    possible time-skew between the client and the server, we allow an extra
    token before and after the current time. If you experience problems with
    poor time synchronization, you can increase the window from its default
    size of +-1min (window size of 3) to about +-4min (window size of
    17 acceptable tokens).
    Do you want to do so? (y/n) y
    If the computer that you are logging into isn't hardened against brute-force
    login attempts, you can enable rate-limiting for the authentication module.
    By default, this limits attackers to no more than 3 login attempts every 30s.
    Do you want to enable rate-limiting (y/n) y 

# 7.终端设置google 二次身份验证登陆 #

打开xshell(其他终端类似)，选择登陆主机的属性。设置登陆方法为Keyboard Interactive
![](http://ocppiicaw.bkt.clouddn.com/google/google4.png)

 
用户名：root（根据实际情况填写）
密码为空就好（因为需要先二次验证，再填入密码）
![](http://ocppiicaw.bkt.clouddn.com/google/google5.png) 

二次验证码输入

![](http://ocppiicaw.bkt.clouddn.com/google/google6.png)

打开手机谷歌验证器，将验证码输入

 ![](http://ocppiicaw.bkt.clouddn.com/google/google7.png)
 
系统用户密码输入

![](http://ocppiicaw.bkt.clouddn.com/google/google8.png)
 
# 8.完美登陆 #
 
![](http://ocppiicaw.bkt.clouddn.com/google/google9.png)

# 9.取消google验证 #

1) vim /etc/pam.d/sshd
注释红色框内
 
![](http://ocppiicaw.bkt.clouddn.com/google/google10.png)

2) vim /etc/ssh/sshd_config
```
把原来的yes改为no
ChallengeResponseAuthentication no
```

