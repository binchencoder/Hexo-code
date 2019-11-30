---
title: Mac平台搭建Golang环境
date: 2019-08-08 12:10:18
tags:
	- Golang
categories:
	- 后端
---


![Golang](./Mac平台搭建Golang环境/golang.png)

# Overview

我大概是在两年前开始接触Golang语言，当时我们公司在北美成立研发中心，核心成员都是来自Google、微软等世界一流互联网公司。那时起我们才真正有了CTO这个职位。他来自Google，所以把Google的核心开发语言Golang带到我们公司并大力推广。要求全员学习Golang。

当时业务部门的同事都比较抵触学习Golang，包括我在内。大家都把Java语言当作了吃饭的饭碗，认为Golang是一门比较难的语言，不情愿在这个上面花时间。后来事实证明当初的想法是错误的，其实Golang入门还是很容易的，包括搭建环境，相比较Java而言，要容易的多。

听我啰嗦了这么多，大家可不要认为Golang就是万能的。任何一门语言都有其优点和缺点，Golang也不例外。

> **优点**

- Golang语法简单，可读性非常高，入门容易
- 基于 goroutines 和 channels 的简单并发编程
- 丰富的标准库
- Golang性能优越
- 语言层面定义源代码的格式化
- 标准化的测试框架
- Golang程序编译快，方便操作
- Defer声明，避免忘记清理
- 方法多返回值：定义function可以返回多个值

> **缺点**

- Golang忽略了现代语言的进步
- 接口是结构类型
- 没有枚举
- := / var 两难选择

说了这么多，让我们开始动手吧，本篇内容只会介绍如何搭建Golang开发环境

# Mac搭建Golang开发环境

介绍两种方式安装Golang环境

## 通过HomeBrew安装

Homebrew有点类似于Linux操作系统中的apt-get（Ubuntu）、yum（yum），Mac的操作系统中使用它解决包依赖问题，套用官方的话来说：使用 Homebrew 安装 Apple 没有预装但 你需要的东西。

### 安装homebrew，已有则跳过

```
fabric:~ fabric$ ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

安装成功则提示

```
==> Installation successful!

==> Homebrew has enabled anonymous aggregate user behaviour analytics.
Read the analytics documentation (and how to opt-out) here:
  https://docs.brew.sh/Analytics.html

==> Next steps:
- Run `brew help` to get started
- Further documentation: 
    https://docs.brew.sh
```

#### 安装Golang

- 首先看看有哪些Golang版本可用

```
chenbindeMacBook-Pro:BazelWorkspace chenbin$ brew search go
==> Formulae
algol68g                      gnu-go                        gocr                          google-authenticator-libpam   gosu                          lgogdownloader                percona-server-mongodb
anycable-go                   go                            gocryptfs                     google-benchmark              gotags                        libgosu                       protoc-gen-go
arangodb                      go-bindata                    godep                         google-java-format            goto                          mongo-c-driver                pygobject
argon2                        go-jira                       goenv                         google-sparsehash             gource                        mongo-cxx-driver              pygobject3
aws-google-auth               go-statik                     gofabric8                     google-sql-tool               govendor                      mongo-orchestration           ringojs
bogofilter                    go@1.10                       goffice                       googler                       gowsdl                        mongodb                       spaceinvaders-go
cargo-completion              go@1.11                       golang-migrate                goolabs                       gox                           mongodb@3.0                   spigot
certigo                       go@1.9                        gollum                        goose                         gst-plugins-good              mongodb@3.2                   svgo
cgoban                        goaccess                      golo                          gopass                        gx-go                         mongodb@3.4                   wego
clingo                        goad                          gom                           gor                           hugo                          mongodb@3.6                   wireguard-go
django-completion             gobby                         gomplate                      goreleaser                    jfrog-cli-go                  mongoose                      write-good
forego                        gobject-introspection         goocanvas                     goreman                       jpegoptim                     pango
fuego                         gobuster                      goofys                        gost                          lego                          pangomm
```
我们发现最新的有1.11可以使用

- 安装brew下最新版本的Golang

```
fabric:~ fabric$ brew install go@1.11
```

- 配置Golang的环境变量

```
chenbindeMacBook-Pro:BazelWorkspace chenbin$ vi /Users/chenbin/.bash_profile
```
> **NOTE:** 在Linux环境下是 **vim ~/.bashrc**

```
export GOROOT=/usr/local/go
export GOPATH=/Volumes/BazelWorkspace/bazel-go

export PATH=$GOROOT/bin:$GOPATH/bin:$PATH
```
> **NOTE:** GOPATH可以根据个人习惯设置为其他目录

让改动生效

```
chenbindeMacBook-Pro:BazelWorkspace chenbin$ source ~/.bash_profile
```

- 试一试Golang是否安装成功

出现以下内容，则安装成功

```
chenbindeMacBook-Pro:BazelWorkspace chenbin$ go env
GOARCH="amd64"
GOBIN=""
GOCACHE="/Users/chenbin/Library/Caches/go-build"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOOS="darwin"
GOPATH="/Volumes/BazelWorkspace/bazel-go"
GOPROXY=""
GORACE=""
GOROOT="/usr/local/go"
GOTMPDIR=""
GOTOOLDIR="/usr/local/go/pkg/tool/darwin_amd64"
GCCGO="gccgo"
CC="clang"
CXX="clang++"
CGO_ENABLED="1"
GOMOD=""
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/var/folders/hw/12mwhf310xd8m8k3bhtjxqp00000gn/T/go-build698784611=/tmp/go-build -gno-record-gcc-switches -fno-common"
```

## 直接下载Golang包

- 直接在github下载源程序包

**根据自己的需求下载适当的版本，推荐选择当下最新版本**

下载地址： [https://github.com/golang/go/releases](https://github.com/golang/go/releases)

- 解压到你想配置的GOROOT目录

- 配置Golang环境变量（与通过Homebrew安装Golang的配置方式一样)


## END

我个人推荐通过直接下载源程序包的方式安装，不仅省去了安装Homebrew的步骤，而且耗时更短。

我们开发golang的code一般放在 **$GOPATH/src** 目录下，下一章我会教大家如何利用工具简单而且方便的开发Golang

> **NOTE:** 提前剧透一下，大家可以先看看 [https://github.com/linuxerwang/gobazel](https://github.com/linuxerwang/gobazel)
