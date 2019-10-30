---
title: windows7配置nfs
date: 2019-09-30 11:21:03
tags: nfs
---

1. 安装haneWIN NFS SERVER

   [下载地址](https://www.hanewin.net/index.html)

2. 打开nfs客户端，配置如下

   ![](https://cdn.jsdelivr.net/gh/haominglfs/images/nfs.PNG)

   编辑要共享的目录后，重启服务器，列表中就会显示共享的服务器。如果不生效，则打开win7的服务管理器重启nfsd服务。

   ![](https://cdn.jsdelivr.net/gh/haominglfs/images/services.PNG)

3. 问题

   * 保存配置文件时显示没有权限保存文件，需要以管理员身份运行nfs客户端。如何打开win7的管理员账户，可以参考[win7系统设置用户帐户为最高权限的操作方法](http://www.2016win10.com/w10/20399.html)

     