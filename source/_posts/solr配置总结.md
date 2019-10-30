---
title: solr配置总结
date: 2019-03-24 21:30:39
tags:
---

#### 环境

solr版本：4.10.4

tomcat7

jdk8

<!--more-->

#### solr文件内容介绍

![](https://raw.githubusercontent.com/haominglfs/images/master/20190324213537.png)

bin：solr的运行脚本

contrib：solr的一些扩展jar包，用于增强solr的功能。

dist：该目录包含build过程中产生的war和jar文件，以及相关的依赖文件。

docs：solr的API文档

example：solr工程的例子目录：

l  example/solr：

​           该目录是一个标准的SolrHome，它包含一个默认的SolrCore

l  example/multicore：

​           该目录包含了在Solr的multicore中设置的多个Core目录。

l  example/webapps：

​    该目录中包括一个solr.war，该war可作为solr的运行实例工程。

licenses：solr相关的一些许可信息

#### SolrCore配置

* SolrHome是Solr服务运行的主目录，该目录中包括了多个SolrCore目录。SolrCore目录中包含了运行Solr实例所有的配置文件和数据文件，Solr实例就是SolrCore。

  每个SolrCore提供单独的搜索和索引服务。

![](https://raw.githubusercontent.com/haominglfs/images/master/20190324214216.png)

![image-20190324214249343](/Users/haominglfs/Library/Application Support/typora-user-images/image-20190324214249343.png)

* 创建SolrCore