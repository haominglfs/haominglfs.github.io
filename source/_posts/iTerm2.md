---
title: iTerm2
date: 2017-04-19 22:34:20
tags:
    - iTerm2配置
categories:
    - iTerm2
---
#iTerm2配置
* 配色  
1.`git clone git@github.com:altercation/solarized.git`
2.这里我们要使用的是iterm2-colors-solarized目录下的，包括Solarized         Dark.itermcolors和Solarized Light.itermcolors两个配置文件。  
3.打开Preferences->Profiles->Color面板，在Color Presets中将以上  两个配置方案导入，然后选择Solarized Dark或者Solarized Light即可。一般推荐使用Solarized Dark，Solarized Light有种亮瞎的感觉。
* oh-my-zsh  
1.接下来，用oh-my-zsh来武装zsh，一行命令搞定：
  `sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"`
2.oh-my-zsh中提供了多套主题可供选择，有不同的输出样式及配色。默认应该是robbyrussell，感觉中规中矩没啥亮点。翻看了一下，发现了agnoster主题，感觉非常入眼。
接下来，编辑~/.zshrc，找到变量ZSH_THEME将其赋值改为agnoster即可。  
3.为了显示正常，需要安装powerline字体，方法如下：  
`git clone git@github.com:powerline/fonts.git`  
`cd fonts`  
`./install`  
然后，在iTerm2->Preferences->Profiles->Text面板中将Non-ASCII Font改成Roboto Mono Powerline，显示就正常了！  



