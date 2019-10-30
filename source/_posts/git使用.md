---
title: git使用
date: 2017-04-30 11:35:33
tags:
    - git
categories:
    - git
---
## git使用
今天写了一个在在标签页显示数字的chrome扩展程序，打算提交到github,顺便学习了将一个已有的项目提交到github的方法。

<!--more-->

* 登录github，新建一个仓库
![](http://ww4.sinaimg.cn/large/006tKfTcly1ff4kewj8khj31dq0qe78o.jpg)
* 进入项目的本地目录，执行如下命令：

  ```git init``` 
  ```git remote add origin git@github.com:haominglfs/tab_number.git//与远程仓库建立关联``` 
  ```git add .``` 
  ```git commit -m 'tab_number extension of chrome v0.1'``` 
  ```git push -u origin master //push到远程仓库``` 
  

