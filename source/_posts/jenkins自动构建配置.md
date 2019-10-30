---
title: jenkins自动构建配置
date: 2018-09-04 18:17:57
tags:
---

#### 安装jenkins

1. 官网下载jenkins [jenkins](https://jenkins.io/)
2. 终端运行：java -jar jenkins.war --httpPort=8080  8080为端口号，可以自行设置
3. 在浏览器中输入：http://localhost:8080  访问jenkins服务
4. 首次启动jenkins，出于安全考虑，jenkins会生成一个随机的口令到 /root/.jenkins/secrets/initialAdminPassword 文件中，复制文件中的口令到jenkins即可通过访问。

#### 安装ant

1. 官网下载Ant  [Ant](http://ant.apache.org/)

2. 配置环境变量

   > ANT_HOME    ant的根路径
   >
   > path             $ANT_HOME/bin
   >
   > classpath      $ANT_HOME/lib

3. 验证   终端输入  ant     有正常的输出，则表示安装成功

#### 配置tomcat

1. 配置tomcat-users.xml 添加角色和用户

   ```xml
   <role rolename="manager-gui"/> 
   <role rolename="manager-script"/>
   <user username="tomcat" password="123456"  roles="manager-gui, manager-script"/>
   ```

2. 配置TOMCAT_HOME/conf/context.xml，在<Context>元素中增加一个属性antiResourceLocking="true" antiJARLocking="true"，默认是"false"。

   ```xml
   <Context antiResourceLocking="true" antiJARLocking="true">
   ```

   以上为了解决Jenkins部署异常：The Tomcat Manager responded FAIL - Deployed application at context path]。

   异常原因：

   * Tomcat应用更新时，把新的WAR包放到webapps目录下，Tomcat就会自动把原来的同名webapp删除，并把WAR包解压，运行新的 webapp
   * 但是，有时候Tomcat并不能把旧的webapp完全删除，通常会留下WEB-INF/lib下的某个jar包，必须关闭Tomcat才能删除，这就导致自动部署失败
   * 解决方法是在<Context>元素中增加一个属性antiResourceLocking="true" antiJARLocking="true"，默认是"false"。这样就可以热部署了
   * 实际上，这两个参数就是配置Tomcat的资源锁定和Jar包锁定策略

#### 安装jenkins插件

![](http://ow83fnk93.bkt.clouddn.com/20180904185758.png)

安装svn插件 Subversion Plug-in

安装 Deploy to container Plugin 插件

#### 创建jenkins任务(svn+ant+tomcat)

1. 新建任务，构建一个自由风格的软件项目![](http://ow83fnk93.bkt.clouddn.com/20180904190948.png)

2. svn 配置![](http://ow83fnk93.bkt.clouddn.com/20180904191202.png)

3. 构建环境和构建![](http://ow83fnk93.bkt.clouddn.com/20180904191312.png)

4. 构建后操作![](http://ow83fnk93.bkt.clouddn.com/20180904191529.png)

   