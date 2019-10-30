---
title: mac安装autojump
date: 2018-08-01 22:11:25
tags:
---

1. brew install autojump

2. 编辑 vim ~/.zshrc

   * 找到 plugins=，在后面添加autojump：plugins=(git autojump)
   * 新开一行，添加：[[ -s $(brew --prefix)/etc/profile.d/autojump.sh ]] && . $(brew --prefix)/etc/profile.d/autojump.sh

3. j  + 跳转的目录

   

   

   