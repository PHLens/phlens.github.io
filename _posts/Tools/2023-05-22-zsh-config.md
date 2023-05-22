---
layout: single
title: ZSH配置
toc: true
toc_label: "目录"
toc_icon: "bars"
category: Tools
tags: #essay/2023
date: 2023-05-22-2005
---
首先安装 zsh ：
`brew/apt install zsh`
# oh-my-zsh
安装:
`sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`

## 主题 
- [powerlevel10k](https://github.com/romkatv/powerlevel10k#oh-my-zsh)

## 插件 
- [autosuggestions](https://github.com/zsh-users/zsh-autosuggestions/blob/master/INSTALL.md)
- [syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)

远程连接保持终端配置：[xxh](https://github.com/xxh/xxh#seamless-oh-my-zsh-demo)
- xxh报错没有权限解决办法：`chmod -R 755 ~/.xxh`
- 安装ohmyzsh插件：`source xxh.ash +I xxh-plugin-zsh-ohmyzsh`
- 插件手动拷贝到`.xxh/.xxh/plugins/xxh-plugin-zsh-ohmyzsh/build/ohmyzsh/plugins`
 
## 字体
参考[Linux 字体安装](../linux-fonts/)

终端设置，选择字体即可正确显示 powerline 图标

# Tmux
[MyTmux](https://github.com/PHLens/MyTmux)
安装：
```bash
cd /home/XXX
ln -s -f MyTmux/.tmux.conf
cp MyTmux/.tmux.conf.local .
```
快捷键
- 删除panel：`<prefix> + x`
- 删除 window：`<prefix> + &` 

Reference:  [oh-my-tmux](https://github.com/gpakosz/.tmux)
