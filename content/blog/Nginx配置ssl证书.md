+++
title = "Nginx配置ssl证书"
description = ""
author = "David Shu"
date = "2021-11-11T13:33:57+08:00"
tags = ["nginx", "ssl"]
categories = ["个人建站"]
removeBlur = false
[[images]]
  src = "https://static.code-david.cn/blog/Tr46LW.jpg"
  alt = ""
  stretch = ""

+++

申请到的免费ssl证书该怎么部署到nginx服务器上呢，本文将展示所有步骤



### ssl证书的申请

一般的云服务提供商都有免费的ssl证书申请，对于个人开发者而言，免费的ssl证书已经够用了，相比于需要购买的的ssl证书可以省几百上千元每年，对于非商业使用的个人站点还是比较合适的，我个人在阿里云上申请了免费的ssl证书，阿里云每个自然年可以申请20个，如下所示：

![阿里云免费ssl证书](https://static.code-david.cn/blog/NHHjF0.png)

### Nginx的安装配置

我个人服务器用的是ubuntu18.04，所以直接使用apt包管理安装

```shell
sudo apt install nginx
sudo nginx -t
```

测试是否安装成功

```shell
sudo nginx -t
```

显示如下则表明成功了

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

直接输入地址进行访问可以看到nginx的默认页面，如下

![nginx默认页面](https://static.code-david.cn/blog/eMjnDj.png)

### 处理ssl证书

从证书申请方下载证书，修改文件夹名称为cert，文件夹内包含两个文件，分别是一个以.key结尾的文件和以.pem结尾的文件，并将这个文件夹放到nginx的安装目录，我这里是放到了/etc/nginx/目录下

### 配置nginx文件

编辑nginx的配置文件时，这样写入自己的配置，可以自己单独在如下位置创建一个文件

```shell
sudo vim /etc/nginx/sites-enabled/blog_site
```

在该文件下写如下内容：

```
server {
        listen 80;
        server_name code-david.cn; # 证书绑定的域名。
        rewrite ^(.*)$ https://$host$1 permanent; #将所有HTTP请求通过rewrite指令重定向到HTTPS。
        location / {
                index index.html index.htm;
        }
}
server {
        listen 443 ssl default_server;
        listen [::]:443 ssl default_server;
        ssl_certificate /etc/nginx/cert/6583390_code-david.cn.pem; # 对应证书文件
        ssl_certificate_key /etc/nginx/cert/6583390_code-david.cn.key; # 对应证书文件
        ssl_session_timeout 5m;
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_prefer_server_ciphers on;

        root /home/sdw/public;

        # Add index.php to the list if you are using PHP
        index index.html index.htm index.nginx-debian.html;

        server_name code-david.cn; # 静态资源的根目录

        location / {
                try_files $uri $uri/ =404;
        }
}
```

将nginx.conf文件中http模块内加上这样的一行

```
include /etc/nginx/sites-enabled/blog_site;
```

重新载入nginx配置文件即可

```shell
sudo nginx -s reload
```

### 验证

只需要通过https进入你的站点，看看是否有对应的证书即可
