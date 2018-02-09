---
title: Ansible playbook
tags: playbook
categories: 自动化运维工具
type: "categories"
---
# Playbooks 简介 #

Playbooks 与 adhoc 相比,是一种完全不同的运用 ansible 的方式,是非常之强大的.
简单来说,playbooks 是一种简单的配置管理系统与多机器部署系统的基础.与现有的其他系统有不同之处,且非常适合于复杂应用的部署.
<!-- more -->
# Playbook 语言的示例 #
Playbooks 的格式是YAML（详见:YAML 语法）,语法做到最小化,意在避免 playbooks 成为一种编程语言或是脚本,但它也并不是一个配置模型或过程的模型.

    ---
    - hosts: webserver
      vars:    //定义变量
        http_port: 80
        max_clients: 200
      remote_user: root
      tasks:
      - name: ensure apache is at the latest version
        yum: pkg=httpd state=latest
      - name: write the apache config file
        template: src=/srv/httpd.j2 dest=/etc/httpd.conf
       notify:
       - restart apache
      - name: ensure apache is running
        service: name=httpd state=started
      handlers:
        - name: restart apache
          service: name=httpd state=restarted

# playbook基础 #

##主机与用户##

用法1：

    ---
    - hosts: webservers
      remote_user: root
      sudo: yes
用法2：

    tasks:
      - name: test connection
        ping:
        remote_user: yourname
        sudo: yes

## sudo 执行命令 ##
用法1：

    ---
      sudo: yes
用法2：

    tasks:
        sudo: yes
## Tasks 列表 ##
每一个 play 包含了一个 task 列表（任务列表）.一个 task 在其所对应的所有主机上（通过 host pattern 匹配的所有主机）执行完毕之后,下一个 task 才会执行.有一点需要明白的是（很重要）,在一个 play 之中,所有 hosts 会获取相同的任务指令,这是 play 的一个目的所在,也就是将一组选出的 hosts 映射到 task.

用法1：

    tasks:
      - name: make sure apache is running
        service: name=httpd state=running
用法2：

    tasks:
      - name: disable selinux
        command: /sbin/setenforce 0

### 命令返回值设置为0 ###

    tasks:
      - name: run this command and ignore the result
        shell: /usr/bin/somecommand || /bin/true
或者是这样:

    tasks:
      - name: run this command and ignore the result
        shell: /usr/bin/somecommand
        ignore_errors: True

# Handlers:在发生改变时执行的操作 #
当一个文件的内容被改动时,重启两个 services:

    - name: template configuration file
      template: src=template.j2 dest=/etc/foo.conf
      notify:
       - restart memcached
       - restart apache

