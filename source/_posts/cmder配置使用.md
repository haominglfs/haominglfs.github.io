---
title: cmder配置使用
date: 2018-07-14 17:37:56
tags: windows
---

##### 安装

[cmder官网](http://cmder.net/ )

下载的时候，有两个版本，分别是mini与full版；唯一的差别在于有没有内建msysgit工具，这是Git for Windows的标准配备；全安装版 cmder 自带了 msysgit, 压缩包 23M, 除了 git 本身这个命令之外, 里面可以使用大量的 linux 命令；比如 grep, curl(没有 wget)； 像vim, grep, tar, unzip, ssh, ls, bash, perl 对于爱折腾的Coder更是痛点需求。 

<!--more-->

##### 配置cmder

1. 把 cmder 加到环境变量：可以把`Cmder.exe`存放的目录添加到系统环境变量；加完之后，`Win+r`一下输入`cmder`，即可。 

2. 添加 cmder 到右键菜单：在某个文件夹中打开终端，在管理员权限的终端输入以下语句即可： 

   > Cmder.exe /REGISTER ALL

##### 常用快捷键

- 可以利用`Tab`，自动路径补全；
- 可以利用Ctrl+T建立新页签；
- 利用Ctrl+W关闭页签;
- 还可以透过Ctrl+Tab切换页签;
- Alt+F4：关闭所有页签
- Alt+Shift+1：开启cmd.exe
- Alt+Shift+2：开启powershell.exe
- Alt+Shift+3：开启powershell.exe (系统管理员权限)
- Ctrl+1：快速切换到第1个页签
- Ctrl+n：快速切换到第n个页签( n值无上限)
- Alt + enter： 切换到全屏状态；
- Ctr+r 历史命令搜索;
- End, Home, Ctrl : Traversing text with as usual on Windows

