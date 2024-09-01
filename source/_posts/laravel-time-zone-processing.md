---
title: "laravel管理时区"
categories:
  - php
tags:
  - php
  - laravel
  - 时区
date: 2024-07-22
---

## 起因
很多情况下不同的用户有着不同的时区，那么在laravel中该怎么去管理不同用户之间的时区切换呢。

## 用UTC来时区管理
可以统一使用UTC时间来管理时区。因为通过UTC（世界协调时间）可以轻易的将时间转换成任意时区的时间，所以laravel中默认设置的时区就是UTC。

那么该如何正确使用UTC时间呢？首先数据库里的时间必须要统一存储成UTC时间，而不能是其他时区的时间。其次在读取数据的时候需要将时间转换成对应用户时区的时间，下面这段代码就是将用户的create_at（UTC）字段转换为用户所在时区（Asia/Shanghai）的时间。
```php
// $user->create_at（2024-07-22 02:31:35），auth()->user()->timezone（Asia/Shanghai）
echo  $user->create_at->tz(auth()->user()->timezone)->toDateTimeString(); // 2024-07-22 10:31:35
```
那么在解决完时间显示问题之后，该怎么在用户指定了时间的情况下取出对应的时区数据呢？答案是将用户提交的时间的时间转换成UTC时间就行了，然后再通过UTC时间去查询对应的数据。因为数据库中存储的是UTC时间所以需要将提交的时间转换成UTC之后再去查询。下面这段代码会将用户提交的时间转换成UTC时间然后去数据库中查询数据
```php
$requestTime = '2024-07-22 10:31:35';// 请求的时间（Asia/Shanghai）
// 将提交的时间转成UTC
$conversionTimeToUTC = \Carbon\Carbon::parse($requestTime, 'Asia/Shanghai')
    ->timezone('UTC')
    ->toDateTimeString(); // 2024-07-22 02:31:35
TestModel::whereData('create_at', $conversionTimeToUTC)->first();
```
这样数据库查询对应时区时间的问题就解决啦。虽然通过时区来转换时间这个问题解决了，但是要怎么获取用户的时区呢？

## 获取用户时区
可以通过jamesmills/laravel-timezone这个包来设置用户的时区，也可以通过js来获取时区。

## 参考文章
https://laravel-news.com/laravel-timezones