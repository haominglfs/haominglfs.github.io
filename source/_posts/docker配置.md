---
title: docker配置
date: 2019-02-25 17:21:40
tags:
---

#### docker开机启动

```shell
systemctl enable docker.service
```

<!--more-->

#### docker-compose开机启动容器

```shell
vim /etc/rc.d/rc.local
/usr/local/bin/docker-compose -f /www/docker/trace_fecshop/docker-compose.yml up -d
#/www/docker/trace_fecshop 是你的docker-compose的目录
```

#### docker-compose-volumes说明

```shell
#docker-compose里两种设置方式都是可以持久化的
#第一种情况路径直接挂载到本地，比较直观，但需要管理本地的路径，而第二种使用卷标的方式，比较简洁，但你不知道#数据存在本地什么位置，下面说明如何查看docker的卷标
#1.
ghost:  
  image: ghost
  volumes:
    - ./ghost/config.js:/var/lib/ghost/config.js  #yml文件所在路径
    
#2.卷标
services:
 mysql:  
  image: mysql
  container_name: mysql
  volumes:
    - mysql:/var/lib/mysql
...
volumes:
 mysql:
 
#查看所有卷标
docker volume ls 
#查看具体的volume对应的真实地址
docker volume inspect vagrant_mysql

    
```

