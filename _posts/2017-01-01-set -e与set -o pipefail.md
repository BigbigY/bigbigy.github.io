---
title: set -e与set -o pipefail
tags: set命令
categories: shell
type: "categories"
---
# set -e #
set命令的-e参数，linux自带的说明如下：

"Exit immediately if a simple command exits with a non-zero status."

也就是说，在"set -e"之后出现的代码，一旦出现了返回值非零，整个脚本就会立即退出。有的人喜欢使用这个参数，是出于保证代码安全性的考虑。但有的时候，这种美好的初衷，也会导致严重的问题。
<!-- more -->
举个栗子：

    cat test_set.sh
    #/bin/bash
    set -e 
    ls /etc/index.html
    echo "测试！"
执行输出：

    [root@ansible1 ~]# sh test.sh 
    ls: 无法访问/etc/index.html: 没有那个文件或目录
    

# set -o pipefail #
对于set命令-o参数的pipefail选项，linux是这样解释的：

"If set, the return value of a pipeline is the value of the last (rightmost) command to exit with a  non-zero  status,or zero if all commands in the pipeline exit successfully.  This option is disabled by default."

设置了这个选项以后，包含管道命令的语句的返回值，会变成最后一个返回非零的管道命令的返回值。听起来比较绕，其实也很简单：

    cat test_set.sh
    set -o pipefail
    ls /etc/index.html
    echo "test"
    echo $?

设置了set -o pipefail，返回从右往左第一个非零返回值，即ls的返回值1

    [root@ansible1 ~]# sh test.sh 
    ls: 无法访问/etc/index.html: 没有那个文件或目录
    test
    0 
没有set -o pipefail，默认返回最后一个管道命令的返回值

注释掉set -o pipefail 这一行，再次运行，输出：

    [root@ansible1 ~]# sh test.sh 
    ls: 无法访问/etc/index.html: 没有那个文件或目录
    0