---
title: "记laravel框架在正式环境更新代码之后请求接口返回Call to a member function getModel() on null的问题"
categories:
  - php
tags:
  - php
  - laravel
date: 2022-05-30 
---

# 具体经过
在正式服拉取最新代码之后，新增的接口前端反映无法请求成功，调查日志发现接口返回的信息是Call to a member function getModel() on null。

# 解决过程
首先Call to a member function getModel() on null这条信息的意思是接口地址访问错误，或者接口地址不存在，
查看routes/api.php文件发现有定义路由，但是请求接口却返回地址不存在。

初步判断可能是缓存了路由，导致请求访问不到最新的路由，使用此命令清除路由缓存
```
php artisan route:clear
```

然后就成功解决了😂