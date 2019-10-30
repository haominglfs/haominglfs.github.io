---
title: nginx笔记
date: 2019-03-06 20:16:36
tags:
---

#### 安装

```shell
brew install nginx
```

![](https://raw.githubusercontent.com/haominglfs/images/master/20190306202143.png)

> /usr/local/var/log/nginx   日志地址

<!--more-->

#### 配置

1. location  root指令：

   ```shell
   location /img/ {
       alias /var/www/image/;
   }
   #若按照上述配置的话，则访问/img/目录里面的文件时，ningx会自动去/var/www/image/目录找文件
   location /img/ {
       root /var/www/image;
   }
   #若按照这种配置的话，则访问/img/目录下的文件时，nginx会去/var/www/image/img/目录下找文件。
   alias是一个目录别名的定义，root则是最上层目录的定义。
   一直以为root是指的/var/www/image目录下，应该 是 /var/www/image/img/ 
   还有一个重要的区别是alias后面必须要用“/”结束，否则会找不到文件的。。。而root则可有可无
   可以是绝对目录，也可以是相对目录(相对于nginx的安装目录)
   
   uri解析
   =  精确匹配
   ```

2. 虚拟主机

   * 基于域名
   * 基于端口
   * 基于ip

