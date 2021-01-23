+++
title = "GCP小技巧 远程开发"
description = "远程开发时经常会遇到一些问题，本篇文章带你一一解决"
author = "David Shu"
date = "2021-01-23T19:28:30+08:00"
tags = ["remote ssh", "vscode", "gcp"]
categories = ["GCP"]
removeBlur = true
[[images]]
  src = "https://static.code-david.cn/blog/remote_vscode.png"
  alt = "remote vscode"
  stretch = ""
+++
由于谷歌云在国内并无服务器，使用GCP开发中首先要面对的就是网络连接问题，同时还存在或多或少的其他问题，这篇文章将说明如何通过vscode remote、ssh等工具配置来解决这些问题。

## VScode remote
VScode中自带的远程ssh插件是一个非常好用的工具，首先要做的就是安装这个插件

![remote ssh](https://static.code-david.cn/blog/2cQkzP.png)

既可以直接通过F1快捷键使用`Remote-SSH: Open Configuration File`，也可手动创建`.ssh/config `配置文件，配置文件中配置好对应的远程主机，一般写法如下：
```
Host <设置检索搜索匹配的名称>
  HostName <主机的地址>
  User <用户名>
  Post <使用的端口，不设置的话默认是22>
```

假设配置的检索匹配名称是beijing，若配置了密钥那么ssh登录就可以非常简单的使用如下指令完成：
```shell
ssh beijing
```
补充一点，设置使用密钥而不是密码的方式有一个方法，利用ssh-copy-id程序，不过前提是使用ssh-keygen生成了密钥，设置指令如下：
```shell
ssh-keygen -t rsa
ssh-copy-id beijing
```

## 让SSH走代理连接
本篇文章的重点来了，既然GCP服务器在国外，那么如何实现稳定的连接呢，答案是可以走代理，让SSH走稳定的代理服务器，以该服务器作为跳板连接到GCP服务器，如何实现呢，所需要做的就是在`.ssh/config`文件中对应的主机增加如下一行：
```
Host <设置检索搜索匹配的名称>
  HostName <主机的地址>
  User <用户名>
  Post <使用的端口，不设置的话默认是22>
  ProxyCommand nc -X 5 -x 127.0.0.1:7890 %h %p
```
下面来解释该指令，该指令是利用nc去访问对应的代理，-X 5指的是使用socks5，-x指的是指定代理服务器和端口，%h是主机，%p是端口，这俩是固定要跟在后方的，127.0.0.1:7890就是你要用作为跳板的代理服务器以及端口了，可以是本地的科学上网代理服务器，也可以是其他的跳板机器。经过上述配置之后，ssh的连接就很稳定了，且延迟情况也将得到改观，需要注意一点：

- 以上配置是在macOS下进行的，对应于windows可能会有不同的配置，主要是windows没有nc程序，但可以使用其他的程序替代，如connect.exe等，可以如下配置：
```
  ProxyCommand C:\Windows\System32\OpenSSH\ssh.exe -W %h:%p
```

## ssh的妙用

ssh主要的功能是远程开发，但其还有一些妙用，比如以下情况可以派上用场：

- 某台服务器算力很强，但是其不具备科学上网能力，如何让服务器具备科学上网能力呢，可使用ssh做端口转发，让远程主机的端口收到请求后转发到本地端口，借用本地的科学上网代理来实现科学上网，本地运行指令如下：
```
ssh -R localhost:7890:localhost:7890 <检索名称>
```
第一个localhost:7890是远程端，第二个是本地端，让远程的7890端口接收请求，然后转发到本地7890端口，远程命令行代理指令如下：
```
export https_proxy=http://127.0.0.1:7890 http_proxy=http://127.0.0.1:7890 all_proxy=socks5://127.0.0.1:7890
```

这样两行指令即可实现无法科学上网的服务器利用本地机作为代理进行科学上网