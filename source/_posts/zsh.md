---
title: "zsh 配置"
categories:
  - linux
tags:
  - linux
  - zsh
date: 2023-01-19
---

# 下载安装oh my zsh
## 1、下载oh my zsh
```
git clone https://github.com/ohmyzsh/ohmyzsh.git ~/.oh-my-zsh
```

## 2、复制配置文件
```
cp ~/.oh-my-zsh/templates/zshrc.zsh-template ~/.zshrc
```

## 3、使配置文件生效
```
source ~/.zshrc
```

# 安装powerlevel10k主题
## 1、下载powerlevel10k主题
```
git clone https://github.com/romkatv/powerlevel10k.git $ZSH_CUSTOM/themes/powerlevel10k
```
## 2、设置主题
修改.zshrc文件
```
vim .zshrc 
```
修改ZSH_THEME
```
ZSH_THEME="powerlevel10k/powerlevel10k"
```

## 3、使配置文件生效，配置powerlevel10k
```
source ~/.zshrc
```