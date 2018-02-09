---
title: linux中shell基础
tags: shell基础
categories: shell
type: "categories"
---
linux中shell变量$#,$@,$0,$1,$2的含义解释: 
# 变量 #
<!-- more -->
    $$ 
    Shell本身的PID（ProcessID） 
    $! 
    Shell最后运行的后台Process的PID 
    $? 
    最后运行的命令的结束代码（返回值） 
    $- 
    使用Set命令设定的Flag一览 
    $* 
    所有参数列表。如"$*"用「"」括起来的情况、以"$1 $2 … $n"的形式输出所有参数。 
    $@ 
    所有参数列表。如"$@"用「"」括起来的情况、以"$1" "$2" … "$n" 的形式输出所有参数。 
    $# 
    添加到Shell的参数个数 
    $0 
    Shell本身的文件名 
    $1～$n 
    添加到Shell的各参数值。$1是第1参数、$2是第2参数…
 
## 示例： ##

    #!/bin/bash
    printf "The complete list is %s\n" "$$"
    printf "The complete list is %s\n" "$!"
    printf "The complete list is %s\n" "$?"
    printf "The complete list is %s\n" "$*"
    printf "The complete list is %s\n" "$@"
    printf "The complete list is %s\n" "$#"
    printf "The complete list is %s\n" "$0"
    printf "The complete list is %s\n" "$1"
    printf "The complete list is %s\n" "$2

## 结果： ##

    [luck@localhost ~]$ sh test.sh 123456 QQ
    The complete list is 24249
    The complete list is
    The complete list is 0
    The complete list is 123456 QQ
    The complete list is 123456
    The complete list is QQ
    The complete list is 2
    The complete list is test.sh
    The complete list is 123456
    The complete list is QQ

# 比较 #
## 字符比较 ##
    -eq   等于
    -ne   不等于
    -gt   大于
    -lt   小于
    -le   小于等于
    -ge   大于等于
    -z    空串
     =    两个字符相等
    !=    两个字符不等
    -n    非空串

## 算术比较运算符 ## 
 
    num1-eq num2  等于 [ 3 -eq $mynum ] 
    num1-ne num2  不等于 [ 3 -ne $mynum ] 
    num1-lt num2  小于 [ 3 -lt $mynum ] 
    num1-le num2  小于或等于 [ 3 -le $mynum ] 
    num1-gt num2  大于 [ 3 -gt $mynum ] 
    num1-ge num2  大于或等于 [ 3 -ge $mynum ]

## 字符串比较运算符 ## 
（请注意引号的使用，这是防止空格扰乱代码的好方法）
  
    -z string               假如 string长度为零，则为真  [ -z "$myvar" ] 
    -n string               假如 string长度非零，则为真  [ -n "$myvar" ] 
    string1= string2        假如 string1和 string2相同，则为真  [ "$myvar" = "one two three" ] 
    string1!= string2       假如 string1和 string2不同，则为真  [ "$myvar" != "one two three" ] 


# 文档判断 #
    -e filename 		    假如 filename存在，则为真  [ -e /var/log/syslog ] 
    -d filename  		    假如 filename为目录，则为真  [ -d /tmp/mydir ] 
    -f filename  		    假如 filename为常规文档，则为真  [ -f /usr/bin/grep ] 
    -L filename             假如 filename为符号链接，则为真  [ -L /usr/bin/grep ] 
    -r filename             假如 filename可读，则为真  [ -r /var/log/syslog ] 
    -w filename             假如 filename可写，则为真  [ -w /var/mytmp.txt ] 
    -x filename             假如 filename可执行，则为真  [ -L /usr/bin/grep ] 
    filename1-nt filename2  假如 filename1比 filename2新，则为真  [ /tmp/install/etc/services -nt /etc/services ] 
    filename1-ot filename2  假如 filename1比 filename2旧，则为真  [ /boot/bzImage -ot arch/i386/boot/bzImage ] 


# bash中的引用 #

    ''	：	强引用
    ""	：	弱引用
    ``	：	命令引用
    
    # echo '$PATH'
    $PATH

    # echo "$PATH"
    /usr/lib64/qt-3.3/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin

    # echo `pwd`
    /home/testuser

# 通配 #
- *				：匹配任意长度的任意字符；
- ?				：匹配任意单个字符；
- [ ]			：匹配指定集合内的任意单个字符；
- [a-z], [A-Z]  ：不区分字符大小写；
- [0-9]			：所有数字
- [a-z0-9]		：所有字母以及数字
- [[:upper:]]	：所有大写字母；
- [[:lower:]]	：所有小写字母；
- [[:digit:]]	：所有的数字；
- [[:alpha:]]	：所有字母；
- [[:alnum:]]	：所有字母和数字；
- [[:space:]]	：空白字符；
- [[:punct:]]	：标点符号；
- [^ ]	   	：匹配指定集合外的任意单个字符；
- [^[:alpha:]]

## 例子： ##

    1、显示/etc目录下，以非字母开头，后面跟了一个字母及其它任意长度任意字符的文件或目录； 
     ls  -d  /etc/[^[:alpha:]][a-z]*
    
    2、复制/etc目录下，所以n开头，以非数字结尾的文件或目录至/tmp/etc目录下；
     mkdir /tmp/etc
     cp  -r  /etc/n*[^0-9]  /tmp/etc/

    3、显示/usr/share/man目录下，所有以man开头，后跟一个数字结尾的文件或目录；
    ls  -d  /ur/share/man/man[0-9]
    
    4、复制/etc目录下，所以p,m,r开头的，且以.conf结尾的文件或目录至/tmp/conf.d目录下；
    mkdir  /tmp/conf.d/
    cp  -r  /etc/[pmr]*.conf   /tmp/conf.d/