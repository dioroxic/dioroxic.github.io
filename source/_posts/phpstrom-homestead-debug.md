---
title: "phpstrom+homestead 配置xdebug"
categories:
  - php
tags:
  - php
  - xdebug
  - phpstrom
date: 2021-11-01
---

关于phpstrom+homestead配置xdebug的文章有很多，但是大多数文章讲解的配置都比较繁琐，
一些没有必要配置的东西也去配置了太麻烦了，所以自己写一篇来记录我是如何简单配置phpstrom+homestead+debug的。

# 首先查看homestead xdebug的端口
查看phpinfo，确认xdebug端口
![alt 查看xdebug端口](/images/phpstrom_confirm_xdebug_version.png)

如上图所示我的xdebug的端口为9003，所以需要去phpstrom设置xdebug的端口为9003

# 设置phpstrom
这里需要设置两个地方一个是xdebug的端口，一个是server文件映射到远程文件

# 首先去设置xdebug的端口
点击设置然后选择：语言和框架 > php > debug
![alt 设置xdebug端口](/images/phpstrom_setting_xdebug_port.png)
然后保存设置

# 设置server
点击设置然后选择：语言和框架 > php > servers
![alt 设置servers](/images/phpstrom_setting_servers_debug.png)
然后保存设置

# 开始调试
最后开启debug和安装谷歌xdebug插件就可以开始愉快的调试啦
![alt 开启debug](/images/phpstrom_open_debug.png)