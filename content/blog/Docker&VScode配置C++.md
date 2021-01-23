+++
title = "Docker&VScode配置C++编程环境"
description = "在macOS下如何进行Linux环境编程呢？本篇文章教你如何通过Docker和VScode实现环境部署"
author = "David Shu"
date = "2021-01-22T14:22:16+08:00"
tags = ["docker", "vscode"]
categories = ["c++"]
comments = true
removeBlur = false
[[images]]
  src = "https://static.code-david.cn/blog/docker_pre.jpeg"
  alt = ""
  stretch = "v"
+++

macOS虽然与Linux都属于类Unix系统，但是系统函数存在差异，目前生产环境的服务器大部分都是Linux，学习Linux系统编程仍有必要，这篇文章主要解决的问题是：在macOS下无Linux环境，如何快速进行Linux环境编程实验？
<!--more-->

## 前提知识
在C++开发中，若涉及到底层的系统函数调用时，由于系统之间的差异，对应的接口会有所差异，鉴于目前生产环境大部分为Linux，推荐主要学习Linux对应的系统接口函数，为了保持开发环境一致，对于使用Windows、macOS等系统的同学可以使用以下几种手段：
1. 直接使用Linux系统进行开发，硬件设备不足的情况下可能需要安装双系统，这种方式比较折腾
2. 使用远程开发的方式，远程建议使用云服务器进行学习，阿里云腾讯云等都有低价套餐，对于个人开发者，使用云主机是较为方便的方式，但这种方式有网络环境限制，对于网络连接存在困难的情况无法解决
3. 本地安装虚拟机方式，这也是比较推荐的方式之一，虚拟机安装的灵活性很大，可以实现本地的集群环境
4. 使用docker，类似虚拟机方式，但是docker更轻量化，部署更方便

## Docker
官方提供的getting-started教程非常不错，建议提前阅读。官方的docker hub已经提供了足够多的image选择，对于arm芯片的硬件也有对应的Linux版本可以安装，实现一个自定义的image有两种方式：

- 原始版：拉取原始image，如ubuntu:20.04等，在此基础直接构建container，进入container后当作刚刚安装好的虚拟机来使用
- 使用Dockerfile或docker compose文件进行image自定义，其中后者更适用于要开启多个container的情况，如分别构建web服务、数据库服务的container，Dockerfile主要用于image的自定义构建

## Dockerfile
使用Dockerfile，具体的写法如下：
```
FROM ubuntu
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends build-essential gdb cmake git ssh
ARG USERNAME=vscode
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Create the user
RUN groupadd --gid $USER_GID $USERNAME \
    && useradd --uid $USER_UID --gid $USER_GID -m $USERNAME \
    #
    # [Optional] Add sudo support. Omit if you don't need to install software after connecting.
    && apt-get update \
    && apt-get install -y sudo \
    && echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME


# ********************************************************
# * Anything else you want to do like clean up goes here *
# ********************************************************

# [Optional] Set the default user. Omit if you want to keep the default as root.
USER $USERNAME
```
以上主要基于官方的ubuntu image，安装了开发C++必须环境，并创建了一个具备sudo能力的名称为vscode的用户

Dockerfile参数解释:

- FROM 用于指明基于哪个image，可标注tag
- RUN 运行指令
- ARG 设定参数变量，可以被后续指令调用
- USER 设定当启动container时默认的登录用户

也可直接拉取我已经构建好的image，已经上传至docker hub，地址是807172378/cpp_dev_m1，这是基于arm架构的版本，已经在M1芯片的mabook上验证使用，构建历史如下：
![](https://static.code-david.cn/blog/GfsffN.png)

## VScode设置
VScode给docker提供了相当全面的支持，完全可以通过VScode对docker进行管理，需要提前安装remote container插件，使用vscode快速构建container的方式有以下两种：

### 直接打开本地文件夹
![](https://static.code-david.cn/blog/2OirIu.png)
- 优点：可以直接给予本地文件夹迅速构建
- 缺点：需要在container中重新配置git
- 本地文件夹是挂载在container上的，性能有折损
- 对本地文件夹存在污染可能

### 使用git repo进行构建
![](https://static.code-david.cn/blog/OibKOm.png)
优点：
- 方便git直接推送提交，创建了专用的volume，性能折损小
- 可快速基于git进行构建，通过保存在repo上的devcontainer配置文件直接构建
- 这种方式并不会污染本地的文件树，是完全隔离的

有一些问题需要注意：
  - 如果使用macOS采用了credential helper验证git，那么使用的是https的方式，这种方式在container会造成验证失败的情况
  - 如何解决：在使用VScode从repo创建container时使用ssh的url即可

## 一些问题

- VScode的cpp-tools在安装时会出现下载失败，可能是使用了代理的原因，临时解决方案是从vsix安装
- VScode配置container是通过.devcontainer文件夹中devcontainer.json文件进行，也可在同路径下包含一个Dockerfile进行image构建，修改文件后通过rebuild contaienr指令重新创建
- 每次关闭container保持提交代码的习惯，虽然container关闭重启后数据不会丢失，但是还是要注意避免删除container导致丢失代码的情况

