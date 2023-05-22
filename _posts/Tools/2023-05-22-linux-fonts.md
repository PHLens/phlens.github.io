---
layout: single
title: linux 字体安装
toc: true
toc_label: "目录"
toc_icon: "bars"
category: 
tags: 
date: 2023-05-22, 8:39:37 晚上
---
# 安装
- 下载 `.tff` 字体（最好是带图标的字体，例如 [Nerd fonts](https://github.com/ryanoasis/nerd-fonts)）
- *直接双击 ttf 字体文件安装*

或者手动安装:
- 创建目录 `/usr/share/fonts/my_fonts`
- 将字体文件移动到上述目录中
- 执行以下命令安装：
```bash
sudo mkfontscale
sudo mkfontdir
sudo fc-cache
```
