---
title: memcached
date: 2018-08-05 15:43:38
tags:
---

### 安装

Libevent:

1. 官网下载，解压缩
2. ./configure  --prefix=/opt/install/libevent
3. make && make install

memecached:

1. 解压缩

2. ./configure --prefix=/opt/install/memcached  --with-libevent=/opt/install/libevent

3. make && make instll

   <!--more-->

### 启动

启动参数：

> -d   启动一个守护进程
>
> -m  分配给memcached 的内存数量（单位为M）
>
> -u   运行memcached 的用户
>
> -L  监听的服务器IP地址
>
> -p  监听的端口号
>
> -c  最大运行的并发连接数
>
> -P  设置Pid 文件

### 操作

![](http://ow83fnk93.bkt.clouddn.com/memcached.png)

### java客户端

![](http://ow83fnk93.bkt.clouddn.com/20180806231945.png)