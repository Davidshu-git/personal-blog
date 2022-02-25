+++
title = "利用rsync同步博客静态资源"
description = ""
author = "David Shu"
date = "2021-10-30T10:37:13+08:00"
tags = ["rsync"]
categories = ["效率开发"]
removeBlur = false
[[images]]
  src = "https://static.code-david.cn/blog/DKhAcz.jpg"
  alt = ""
  stretch = ""

+++

之前碰到一个问题，对于部署在云服务器的静态博客资源，我是本地进行静态资源生成，然后进行静态资源上传，由于受限于云服务器的带宽，且每次都是全量上传，导致上传时间往往比较长，但其实静态资源本身只需要增量上传即可，那么就可以使用rsync工具进行增量上传

### Rsync简介

是Unix下一款同步软件，可以利用差分编码减少数据传输量，对于要同步两个计算机的文件需求是很棒的效率工具，我使用的本地端是mac，远端是linux，通常这两个OS都内置了这个软件

### 参数说明

1. -r参数，表示递归，可以使文件夹内的所有子文件夹都得到复制
2. -a参数能力大于-r参数，同时会复制所有的元信息
3. --delete参数，不使用该参数的时候，对于目标文件夹不执行删除，会导致某些文件存在于目标文件夹而不存在于源文件夹，要保持目标文件和源文件完全相同是需要加上这个参数的
4. -n，执行一个模拟传输操作，但实际上没执行，会输出一个模拟的结果
5. -v，输出冗余提示信息
6. -h，将参数比如大小等数据变为合理的适宜阅读的数值

### 远程传输

rsync默认支持ssh协议，可以使用-e参数来设置使用的协议和端口,举例如下：

```shell
rsync -av /source user@remote_machine:/destination
```

若是需要指定协议和端口，则使用如下方式：
```shell
rsync -av -e 'ssh -p 10022' /source user@remote_machine:/destination
```

### 小结

我每次在本地生成静态资源后，通过rsync即可快速将本地的静态资源快速部署到远端的服务器，附上我自己最常使用的指令：

```shell
rsync -avh --delete public sdw@shanghai:
```

实现功能为完全镜像同步到远端的public目录下，实测我自己的静态资源通过原来的全量上传大概需要30S，但是目前使用rsync可以立刻完成，时间缩短是很明显的，之后若是静态资源更大则体现的优势更加明显。