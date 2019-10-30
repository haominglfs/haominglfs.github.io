---
title: svn安装和配置
date: 2018-07-16 13:11:14
tags:
---

1. 安装服务器端程序

   > yum install -y subversion

2. 创建并配置版本库

   * 创建版本库目录

     > mkdir -p /var/svn/repository

     版本库目录下创建具体的项目目录（可以多个）

   * 创建svn版本库

     > svnadmin  create /var/svn/repository/项目目录

3. 配置svn对应的服务

   * > svn://ip:3690/项目目录  （默认端口号3690）

   * 修改服务配置

     /etc/rc.d/init.d/svnserve (注意备份)

     原版：args="--daemon --pid-file=${pidfile} $OPTIONS"

     修改版：args="--daemon --root=/var/svn/repository(版本库根目录) --listen-port 2255(实际的端口号) --pid-file=${pidfile} $OPTIONS"

   * 启动svn服务

     > service  svnserve start 

4. 命令行客户端

   * 检出（完整下载版本库中的全部内容）

     > svn checkout svn://ip/项目目录  本地目录

   * 工作副本 

     * .svn所在目录为工作副本。
     * 版本控制相关操作都要在工作副本目录下执行。
     * 为了保证工作副本能够正常和服务器进行交互，一般不要删除.svn中的内容。

   * 添加

     *  svn 要求提交一个新建的文件前先把这个文件添加到版本控制体系中。

       > svn add 文件名 

     * svn提交

       > svn commit -m "提交信息" 要提交的文件

   * 查看服务器端文件内容

     > svn list svn:ip/项目目录 

   * 更新

     > svn update [文件名]

5. svn权限管理

   * 三个对应的配置文件

     * conf/svnserve.conf 

       > anon-access = read   匿名访问
       >
       > auth-access = write  授权访问 （注意空格）
       >
       > passwd-db= passwd 指定设置用户名密码的配置文件
       >
       > authz-db=authz 分配权限的配置文件 

     * conf/passwd

       用户名 = 密码

     * conf/authz

       > [groups]创建用户组  组名=组员，组员（使用，隔开）
       >
       > [/] （/表示版本库根目录）
       >
       > @组名=rw   配置组权限
       >
       > 用户名=r     配置用户名权限
       >
       > *=               其他人没有权限

       

   

   

   

   

   

   

   

   