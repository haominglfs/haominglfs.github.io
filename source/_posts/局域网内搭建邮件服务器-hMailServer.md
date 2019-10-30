---
title: 局域网内搭建邮件服务器-hMailServer
date: 2018-11-01 13:44:59
tags: jenkins
---

### hMailServer安装

1. [hMailServer](https://www.hmailserver.com/),到hMailServer网站下载最新版本hMailServer安装包。

2. 双击安装文件

   ![](http://ow83fnk93.bkt.clouddn.com/20181101135057.png)

   ![](http://ow83fnk93.bkt.clouddn.com/20181101135255.png)

   ![](http://ow83fnk93.bkt.clouddn.com/20181101135402.png)

   ![](http://ow83fnk93.bkt.clouddn.com/20181101135510.png)

   ![](http://ow83fnk93.bkt.clouddn.com/20181101135557.png)

   ![](http://ow83fnk93.bkt.clouddn.com/20181101135913.png)

   ![](http://ow83fnk93.bkt.clouddn.com/20181101135944.png)

   设置密码，该密码在配置服务器时使用。

   ![](http://ow83fnk93.bkt.clouddn.com/20181101140101.png)

   ![](http://ow83fnk93.bkt.clouddn.com/20181101140136.png)

3. 配置hMailServer

   * 打开配置程序

     ![](http://ow83fnk93.bkt.clouddn.com/20181101140316.png)

   * 因为内网中没有DNS服务器，所以域名随便取一个。

     ![](http://ow83fnk93.bkt.clouddn.com/20181101140918.png)

   * 添加邮件地址

     ![](http://ow83fnk93.bkt.clouddn.com/20181101141154.png)

   * 配置SMTP

     ![](http://ow83fnk93.bkt.clouddn.com/20181101141649.png)

   * 打开logging

     ![](http://ow83fnk93.bkt.clouddn.com/20181101141752.png)

   * 配置ip Ranges

     ![](http://ow83fnk93.bkt.clouddn.com/20181101141855.png)

     ![](http://ow83fnk93.bkt.clouddn.com/20181101141949.png)

   * 修改hosts文件，添加SMTP的域名解析为本机地址。

     ![](http://ow83fnk93.bkt.clouddn.com/20181101142158.png)

   * 到此，hmailServer服务端配置完成。

4. 配置foxmail客户端

   ![](http://ow83fnk93.bkt.clouddn.com/20181101142726.png)