---
title: shadowsocks配置
date: 2017-04-19 22:30:33
tags:
    - shadowsocks
categories:
    - vpn
---
#shadowsocks配置
--------
* 第一步 用远程工具登录aws主机
* 第二步：安装shadowsocks依赖
	
	1. `sudo -s ` //获取超级管理员权限
	
	2. `apt-get update`//更新apt-get
	
	3. `apt-get install python-pip`//安装pyton包管理工具

	4. `pip install shadowsocks`//安装shadowsocks
	
	5. `ssserver -c /etc/shadowsocks.json -d start`//启动shadowsocks
	
	   <!--more-->
	
* 第三步:配置shadowsocks

	1. `vi /etc/shadowsocks.json`//编辑配置文件
	2. 单一端口配置
	
		```
		{
		    "server":"0.0.0.0",
		    "server_port":端口,
		    "local_address":"127.0.0.1",
		    "local_port":1080,
		    "password":"连接密码",
		    "timeout":300,
		    "method":"aes-256-cfb",
		    "fast_open":false
		    }
		```
	3. 多端口配置
	
	  ```
		  {
		    "server":"0.0.0.0",
		    "port_password": {
		        "端口1": "连接密码1",
		        "端口2" : "连接密码2"
		    },
		    "timeout":300,
		    "method":"aes-256-cfb",
		    "fast_open": false
	    }
	  ```
* 开启aws 入站端口

 配置好shaodowsocks后，还需要将配置中的端口打开,这样客户端的服务才能链接得上EC2中的shadowsocks服务
首先打开正在运行的实例，向右滚动表格，看到最后一项，安全组，点击进入
![](https://segmentfault.com/image?src=http://7fvd05.com1.z0.glb.clouddn.com/QQ20150815-0.png&objectId=1190000003101075&token=cdc26e8e1b55555923c613f33248fa51)

![](https://segmentfault.com/image?src=http://7fvd05.com1.z0.glb.clouddn.com/QQ20150815-4.png&objectId=1190000003101075&token=a6157d5b581643cfb706a380e15816f7)

默认是开启了一个22端口（这是给ssh访问的），再建一个如下图红框标示的端口，我的shadowsocks配置的端口是8388，所以这里就开启8388，
![](https://segmentfault.com/image?src=http://7fvd05.com1.z0.glb.clouddn.com/QQ20150815-5.png&objectId=1190000003101075&token=9b7e5a2ca61848931c4b48136d7985da)

* 配置文件编辑完成后，接下来就可以部署运行了：

```
ssserver -c /etc/shadowsocks.json -d start
```
当然，我们可不希望每次重启服务器都手动启动 SS, 因此我们要把这条命令放到这个文件下：/etc/rc.d/rc.local，这样以后就能开机自动运行了。


