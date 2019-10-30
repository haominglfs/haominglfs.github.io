---
title: tmux配置使用
date: 2018-07-23 21:57:48
tags: shell
---

### 什么是tmux

tmux是一个工具，用于在终端窗口中运行多个终端会话，可以使终端会话进入后台运行。

<!--more-->

### 安装

> $ brew install tmux

### 快捷键前缀

为了使自身的快捷键不和其他软件的快捷键产生冲突，tmux提供了一个快捷键前缀。当使用快捷键时要先按下快捷键前缀，然后再按下快捷键，默认的前缀是Ctrl-b

### 创建会话

> tmux new -s <name-of-my-session>

假如还需要开发另一个项目，可以再创建一个新会话，但原来的会话不会消失，若要创建一个新会话，只需要按下

<prefix> :，然后输入

> new -s <name-of-my-new-session>

除非显式的关闭会话，否则tmux的会话在重启计算机之前都不会消失。

### 切换会话

1. 获取会话列表

   > <prefix> s

   列表中的每个会话都有一个 ID，该 ID 是从 0 开始的。按下对应的 ID 就可以进入会话。如果你已经创建了一个或多个会话，但是还没有运行 tmux，那么可以输入如下命令以接入已开启的会话:

   > tmux attach

2. 会话外获取会话列表：

   > tmux  ls
   >
   > tmux attach/a -t   <name-of-session>    在会话外进入session
   >
   > tmux attach/a   进入列表第一个会话
   >
   > <prefix> d    临时退出但不删除会话
   >
   > <prefix> :kill-session 在会话内退出并删除session
   >
   > <prefix> :kill-server 删除所有session
   >
   > tmux kill-session -t <name-of-session> 在会话外删除指定session  

### 窗口

一个tmux中可以包含多个窗口。

> <prefix> c   创建窗口
>
> <prefix> w  查看窗口列表
>
> <prefix> 0 切换到指定窗口，窗口对应的数字
>
> <prefix> n 切换到下一个窗口
>
> <prefix> p 切换到上一个窗口
>
> <prefix> l 在相邻的两个窗口切换
>
> <prefix> , 重命名窗口
>
> <prefix> f 在多个窗口里搜索关键字
>
> <prefix> & 删除窗口

### 窗格

一个tmux窗口可以分割成多个窗格，并且窗格可以在不同的窗口中移动、合并、拆分。

> <prefix> "  水平分割
>
> <prefix> % 垂直分割
>
> <prefix> o 按顺序在Pane之间移动
>
> <prefix> 方向键   上下左右选择pane
>
> <prefix> :resize-pane -U   #向上调整大小
>
> <prefix> :resize-pane -D #向下
>
> <prefix> :resize-pane -L #向左
>
> <prefix> :resize-pane -R #向右
>
> <prefix> :resize-pane -D 5 #向下移动5行
>
> <prefix>  { （往左边，往上面）
>
> <prefix>  } （往右边，往下面）
>
> <prefix> x 删除pane
>
> <prefix> 空格  更换pane排版
>
> <prefix> ！ 移动pane至新的window
>
> <prefix> :join-pane -t $window_name   移动pane合并至某个window
>
> <prefix>  Ctrl+o   按顺序移动pane位置
>
> <prefix>  q 显示pane编号

### 滚动屏幕

> <prefix>  [  进入copy-mode 模式，就可以进行屏幕滚动，q键退出。