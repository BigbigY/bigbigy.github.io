---
title: Go语言初识
tags: [Go]
categories: Go语言成长之路
type: "categories"
author: "luck"

---


# Go Install
Go下载地址：https://golang.org/dl/

#### 下载

```
wget https://dl.google.com/go/go1.10.darwin-amd64.pkg
```

#### 环境配置

```
➜ ~ vim ~/.zshrc
export GOPATH=/Users/wangyangyang/GoWorks
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOBIN
➜ ~ source ~/.zshrc
```

#### 查看版本

```
➜ ~ go version
go version go1.10 darwin/amd64
```

# VsCode Install

VsCode下载地址：https://code.visualstudio.com/Download

#### install Go扩展

打开vscode -> 查看 -> 扩展 -> 搜索Go并安装

![](http://ocppiicaw.bkt.clouddn.com/image/golang/vscode1.png)

#### 安装vscode中go的相关插件

- gocode (代码自动完成)
- gopkgs 
- go-outline (文件大纲)
- go-symbols
- guru
- gorename (重命名)
- gomodifytags
- goplay
- impl
- godef (快速提示信息,跳转到定义)
- goreturns (代码格式化)
- golint
- gotests
- dlv
- go-find-references (搜索参考引用)

#### 使用vscode 安装的插件会failed

```
Installing github.com/nsf/gocode SUCCEEDED
Installing github.com/uudashr/gopkgs/cmd/gopkgs SUCCEEDED
Installing github.com/ramya-rao-a/go-outline FAILED
Installing github.com/acroca/go-symbols FAILED
Installing golang.org/x/tools/cmd/guru FAILED
Installing golang.org/x/tools/cmd/gorename FAILED
Installing github.com/fatih/gomodifytags SUCCEEDED
Installing github.com/haya14busa/goplay/cmd/goplay SUCCEEDED
Installing github.com/josharian/impl FAILED
Installing github.com/rogpeppe/godef SUCCEEDED
Installing sourcegraph.com/sqs/goreturns FAILED
Installing github.com/golang/lint/golint FAILED
Installing github.com/cweill/gotests/... FAILED
Installing github.com/derekparker/delve/cmd/dlv SUCCEEDED
```

#### 解决方法

切换到以下目录：

```
➜ ~ cd $GOPATH/src/github.com/golang/
```

下载插件包：

```
git clone https://github.com/golang/tools.git tools
```

把```tools```目录下的所有文件拷贝到```$GOPATH/src/golang.org/x/tools```下，如果没有自行创建

#### 下面安装failed的插件(不需要FQ)

切换到GOPATH目录下，执行相关的```go install```命令

```
go install github.com/ramya-rao-a/go-outline
go install github.com/acroca/go-symbols
go install golang.org/x/tools/cmd/guru
go install golang.org/x/tools/cmd/gorename
go install github.com/josharian/impl
go install github.com/rogpeppe/godef
go install github.com/sqs/goreturns
go install github.com/golang/lint/golint
go install github.com/cweill/gotests/gotests
```

这样```vscode```下```go```开发需要安装的插件都已经安装成功

#### 创建相应目录
![](http://ocppiicaw.bkt.clouddn.com/image/golang/vscode2.png)
