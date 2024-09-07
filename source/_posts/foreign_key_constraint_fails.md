---
title: "laravel 创建外键失败"
categories:
- php
tags:
- php
- laravel
date: 2021-09-24
---

在设计表的时候为了保证表于表之间的数据完整性使用了外键。但是在写完迁移后运行却报错了报错信息为
>Integrity constraint violation: 1452 Cannot add or update a child row: a foreign key constraint fails

谷歌查询了一番发现出现此报错的原因有如下三种：
>1、 需要设置的外键字段user_id(假设)和users的id类型不同。例如，当user_id为 INT 且用户id为 BIGINT 时。

>2、列user_id不为空，您需要将其设为可为空

>3、user_id列中存在用户id中不存在的值

最后将需要设置外键的字段设为nullable(可为空)就行了