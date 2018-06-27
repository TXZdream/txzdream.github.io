---
title: travis与docker hub集成
date: 2018-06-07 18:30:05
tags:
- travis
- docker hub
- SAAD
---

## 前言

相信很多人都有在github上进行开发的经历，而很多人对docker和CI也都有一定的了解，那这篇文章主要介绍将github、travis和docker hub集成的方法，希望可以给大家带去一些参考

## 本文内容

### 你能看到什么

本文主要介绍github、travis及docker hub的集成，主要场景如下：当你在本地将写好的代码推送到github上时，希望系统可以自动将你的代码进行编译、测试、打包，甚至是直接推送到服务器上进行更新，这里就介绍几个免费的平台可以帮助你完成这些流程

### 你看不到什么

你只能从本文中获取这些平台的基本使用方法，而无法获取详细的配置过程，因为不同的代码对应着不同的配置，没有一个通吃的配置给你使用，所以在具体的实践中，你可能需要自己去阅读各个平台的文档，并根据你的项目，编写不同的配置文件

## 介绍(本篇废话较多，建议跳过)

在正文之前，先带着大家了解一下我们将要使用到的平台，这两个平台本身都可以独立地和github集成在一起，但是由于某些原因，我选择按照一定的次序来进行组合，而不是直接进行组合，在接下来的内容中将会有所提及

### Travis

在了解[travis](https://travis-ci.org/)之前，我们需要先了解一下什么是`Continuous Integration(CI)`，按照travis给出的文档中的定义如下：
```
Continuous Integration is the practice of merging in small code changes frequently - rather than merging in a large change at the end of a development cycle. The goal is to build healthier software by developing and testing in smaller increments. This is where Travis CI comes in.
```
也就是说，ci指的就是，当你写好一个功能之后，就将这个代码集成到主干上，而不是在一次开发周期结束后再这么做。那现在问题来了，为什么要这么做呢？其实很简单，人在写代码时候难免会犯错是吧，那如果我写完一个函数就集成一次，是不是就意味着我可以很快地发现错误呢？代码的错误少了，也就意味着代码的质量上来了，所以我们大概可以得到一个结论：CI可以帮助我们提高代码质量。

好了，既然了解了CI可以给我们带来的好处，那就做就是了，但是新的问题又来了：一般程序员都比较懒，希望一次性写好所有东西，之后只要输入一个命令就可以完成工作就行了，那我们这里的持续集成可不可以这样呢？那当然是可以的，travis就是这样一个平台，它可以帮助你在提交代码后，自动地完成后续工作，包括编译、测试甚至是部署，从而大大提高我们的效率

### Docker hub

一句话：docker hub就是一个存放容器镜像的地方。官方提供了一个免费的仓库使用：[docker hub](https://hub.docker.com/)

## 使用

好了，在完成了介绍之后，我们来看看具体怎么将这三者联合起来使用，具体的流程如下：

```
|--------------------------------------------------------------|
|                                                              |
|提交代码到github--->travis编译、测试、构建镜像----->推送到docker hub|
|                                                              |
|--------------------------------------------------------------|
```

大家应该都知道怎么使用github，所以我们先介绍一下travis的使用

### travis的使用

使用travis，你需要有一个配置文件，以及将你的github授权给travis，之后遵循以下步骤：

1. 编写`.travis.yml`文件

1. 在travis中打开该项目的开关

1. 推送到github

具体流程在官网文档](https://docs.travis-ci.com/user/getting-started/)中有，这里就不多说了，跟着以上流程来就可以了

### 编写Dockerfile

光有代码可不行，机器没办法根据我们的代码内容推测出怎么去构建镜像，所以我们需要有一个文件可以帮助机器去执行相应的流程，而在docker中就是使用Dockerfile文件来构建镜像的，具体的文件使用说明可以参考网上或者官方的一大堆文档，这里仅仅给出实例：

```shell
FROM ubuntu
ADD main /
ENTRYPOINT ["/main"]

RUN apt-get update && apt-get install -y ca-certificates
```

这个很简单，就是告诉docker，我们要使用ubuntu这个基础镜像，然后将当前文件下的编译好的main文件放到根目录下，启动镜像的命令就是`/main`，为了需要，我们需要运行一个命令，安装一个程序。是不是很简单？好像就是告诉一个流程一样，而只需要这个流程对了，就可以得到我们想要的东西

### 推送到docker hub

那我们希望在完成测试后直接构建镜像并推送到docker hub，而docker hub的账户密码又是私密的，那这里就可以使用到travis的环境变量啦，在项目的右上角有一个`more option`的按钮，点击`setting`后在环境变量一栏中，添加环境变量的内容，然后在travis中引用环境变量，就可以将自己的私密信息保留起来啦，这里给出一个travis的实例：

```shell
sudo: required
services:
  - docker

language: go
go:
  - "1.9"
go_import_path: github.com/sysu-saad-project/service-end

env:
  - DOCKER_COMPOSE_VERSION=1.21.0

before_install:
# Install docker-compose
  - sudo rm /usr/local/bin/docker-compose
  - curl -L https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` > docker-compose
  - chmod +x docker-compose
  - sudo mv docker-compose /usr/local/bin
# Create docker network
  - docker network create activity
  - docker network create dev

# Test step
script:
# Compile code
  - CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .
# Build image locally
  - docker build -t xiaxzh/script:dev .
  - go test ./...

after_success:
# Push to docker registry
  - echo "$DOCKER_PASSWORD" | docker login --username "$DOCKER_USERNAME" --password-stdin
  - docker push xiaxzh/script:dev
```

注意最后两行，这就是我们使用环境变量的方式，通过这样的方式，我们就可以将我们的用户名隐藏起来

## 总结

以上介绍了从推送代码到生成镜像的全过程，可能比较粗糙，但是整个过程还是比较明了的，就像遵循着一个流程过来一样，具体的部署这里没有涉及，感兴趣的可以自己上网查找相关资料。正如我前面所说的，这里只是给出了一个基本的流程，而其中相关的知识点需要大家去官方网站寻找文档解决。

最后，不懂的内容可以在下面的评论给出，我将会在我看到之后第一时间和您交流我的解决方案。