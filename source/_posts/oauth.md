---
title: "oauth"
categories:
  - 权限
tags:
  - oauth
date: 2024-06-17
---
# 什么是oauth
oauth 是一种授权机制，主旨是给予访问者特定的权限访问对应的资源（例如获取头像名字等信息）

# oauth 实现方式
oauth 有四种方式去实现分别是授权码式、隐藏式、密码式、凭证式，其中授权码式是最常用的授权方式

1. 授权码式
发放一个授权码，通过授权码获取token，然后通过token来获取对应的资源信息
2. 隐藏式
授权完之后直接发放token给客户端（跳过授权码步骤）
3. 密码式
客户端传用户的账户密码来获取token
4. 凭证式
直接发放token

# 具体实现
这里只实现最常用的授权码式授权，以github授权为例子

## 获取github client_id和client_secret
第三方登录一般都需要网站登记，所以需要先去github网站填写网站授权信息，这是网址，填写对应信息即可获取client_id和client_secret（授权回调地址指的是授权完之后github要跳转的地址）
![alt 填写注册新的OAuth应用程序](/images/github_oauth.png)

## 跳转按钮
设置一个a标签跳转授权
```
<a href="https://github.com/login/oauth/authorize?client_id=xxxxxx&redirect_uri=http://study.test/oauth/redirect">github</a>
```
client_id填获取到的client_id，redirect_uri填设置的回调地址

## 处理回调
授权成功之后github会跳转到设置的回调地址并带code参数返回，拿到code就能通过code来获取token然后再通过token来获取用户信息，以下是用laravel处理的回调代码。
```php
// 接收数据
$data = $request->all();

// 判断是否授权失败
if (isset($data['error'])) {
    return "取消授权";
}

$code = $data['code'];
if (!$code) {
    return "授权失败";
}

// 获取token
$resultToken = \Illuminate\Support\Facades\Http::withHeaders([
    'Accept' => 'application/json'
])->post('https://github.com/login/oauth/access_token', [
    'client_id'     => 'xxxxx',
    'client_secret' => 'xxxxx',
    'code'          => $code
])->json();

$token = $resultToken['access_token'];
if (!$token) {
    return '未获取token';
}

// 通过token获取用户信息
$user = Http::withHeaders([
    'Accept' => 'application/json',
    'Authorization' => 'bearer '. $token
])->get('https://api.github.com/user')->json();

if (!$user) {
    return '信息获取失败';
}

$name = $user['login'];
$avatarUrl = $user['avatar_url'];
```
处理完回调之后就能拿到用户的信息了
