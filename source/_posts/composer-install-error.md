---
title: "homestead composer install显示git@github.com's password"
categories:
  - php
tags:
  - php
  - composer
  - homestead
date: 2022-06-01
---
# 起因
项目执行composer install时显示git@github.com's password。

# 解决过程
用composer install -vvv命令显示

    Failed to download 包名 from dist: curl error 60 while downloading 包github地址: no alternative certificate subject name matches target host name 'api.github.com' Now trying to download from source
    
谷歌该问题发现说的都是证书问题https://github.com/symfony/flex/issues/484， 但是禁用证书验证之后还是不能解决问题。

最后猜测是墙的问题，开启vpn，重启homestead，执行：
    
    composer install

就解决了