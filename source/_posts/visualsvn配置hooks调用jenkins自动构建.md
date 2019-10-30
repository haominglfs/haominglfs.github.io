---
title: visualsvn配置hooks调用jenkins自动构建
date: 2018-10-23 16:48:38
tags: svn,jenkins
---

### jenkins配置

1. 为了让Jenkins中的job可以被触发，job需要被显式地配置为启用SCM轮询才行，未启用SCM轮询选项的job将不会被post-commit hook所触发。下图为在job中启用SCM轮询的示例：

   ![](http://ow83fnk93.bkt.clouddn.com/20181023165142.png)

2. 如果Jenkins启用了跨站点请求伪造防护(默认启用)选项，那么上面的请求会返回一个403错误("No valid crumb was included")。在"系统管理"→"全局安全配置"中，可以看到跨站点请求伪造防护是否有启用：

   ![](http://ow83fnk93.bkt.clouddn.com/20181023165419.png)

### VisualSVN配置

1. 提交到 VisualSVN Server 时 hook 的 post-commit.bat（post-commit.cmd） 不执行的解决方法：

   这是因为 bat 文件执行需要权限，而 VisualSVN Server 默认用的是 NETWORK 用户组，该组没有执行 bat 的权限，导致了 post-commit.bat 文件不能执行，解决方法如下：

   ![](http://ow83fnk93.bkt.clouddn.com/20181023170119.png)

2. 配置hooks

   ![](http://ow83fnk93.bkt.clouddn.com/20181023170416.png)

   curl -X POST  http://localhost:6080/job/cecaudit/build?delay=0sec --user admin:123456789 --data-urlencode json=

   链接地址为jenkins中的job地址（点击立即构建时的链接），—user后为 用户名:密码

   Ps:需要先安装curl或wget

