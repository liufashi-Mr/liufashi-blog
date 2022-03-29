---
title: oh-my-zsh安装与常用插件配置
categories:
  - 开发工具
tags:
  - oh-my-zsh
  - shell
---

## zsh 介绍

> 工欲善其事,必先利其器

zsh是一种 shell，兼容最常用的 bash 这种 shell 的命令和操作，bash 虽然很标准，但是自己日常使用方便更重要。oh-my-zsh 提供了丰富的插件和主题

## 安装

先使用命令查看系统支持的 shell `cat /etc/shells`
正常会是

```bash
/bin/sh
/bin/bash
/sbin/nologin
/bin/dash
/bin/tcsh
/bin/csh
```

如果没有就安装`sudo apt-get install -y zs`

[zsh 官网](http://www.zsh.org/)提供的下载方式

```bash
    sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

然后将 shell 切换为 zsh`chsh -s /bin/zsh`

## 插件

oh-my-zsh 中有内置有很多插件,使用命令`vim ~/.zshrc`打开,找到 plugins

```bash
plugins=(
   git
   extract
   z
   zsh-syntax-highlighting
   zsh-autosuggestions
 )
```

- extract 用于解压任何压缩文件，不必根据压缩文件的后缀名来记忆压缩软件
- z 可以直接跳转到曾经去过的文件夹,在项目多的时候非常实用,之前在 mac 上使用的 autojump,后面发现 zsh 内置的插件 z 也能达到同样的效果
下面两个则需要在自行配置

### zsh-autosuggestions

将项目克隆到` ~/.oh-my-zsh/plugins/`中,然后再`~/.zshrc`中的 plugins 中加上 zsh-autosuggestions 即可

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions ~/.oh-my-zsh/plugins/zsh-autosuggestions
```

### zsh-syntax-highlighting

方法同上

```bash
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/plugins/zsh-syntax-highlighting
```
