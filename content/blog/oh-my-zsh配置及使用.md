+++
title = "Oh My Zsh配置及使用"
description = ""
author = "David Shu"
date = "2021-10-28T15:25:45+08:00"
tags = ["terminal"]
categories = ["效率开发"]
removeBlur = false
[[images]]
  src = "https://static.code-david.cn/blog/ICKPk9.png"
  alt = ""
  stretch = ""

+++
oh-my-zsh可以将terminal变得更好用，我个人最常用的其实就两个，语法高亮和自动补全提示，下面我将简单介绍如何配置

### 安装zsh

安装oh-my-zsh的前提是已经装好了zsh，不同OS的方式不一样，[参考这里](https://gist.github.com/derhuerst/12a1558a4b408b3b2b6e#file-linux-md)进行配置，举例如下：

```shell
sudo apt install zsh
```

### 安装oh-my-zsh

安装指令如下所示，两条选一条即可

```shell
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
sh -c "$(wget https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh -O -)"
```

按照提示讲zsh设置为默认的sh

### 基本插件

[autosuggestion](https://github.com/zsh-users/zsh-autosuggestions)

首先clone

```shell
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
```

然后修改~/.zshrc

```
plugins=( 
    # other plugins...
    zsh-autosuggestions
)
```

source 一下进行生效

```shell
source ~/.zshrc
```

[syntax highlight](https://github.com/zsh-users/zsh-syntax-highlighting)

一样首先 clone

```shell
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

修改~/.zshrc，需要注意的是，这个字段必须在插件的尾部

```
plugins=( [plugins...] zsh-syntax-highlighting)
```

source生效

```shell
source ~/.zshrc
```

