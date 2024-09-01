---
title: "jwt和token"
categories:
  - 权限
tags:
  - jwt
  - token
date: 2023-03-24
---

# 什么是token

Token是服务端生成的一串字符串，以作客户端进行请求的一个令牌，当第一次登录后，服务器生成一个Token便将此Token返回给客户端，以后客户端只需带上这个Token前来请求数据即可，无需再次带上用户名和密码。

## 特点

- token本身无需存储数据，直接去数据库验证用户对应的token对不对的上
- 需要维护token状态，所以可以实现指定令牌作废的功能

## 如何实现

利用uuid来生成大概率不重复的token，然后将其存储到数据库

# 什么jwt

Json web token (JWT), 是为了在网络应用环境间传递声明而执行的一种基于JSON的开放标准（(RFC 7519)
.该token被设计为紧凑且安全的，特别适用于分布式站点的单点登录（SSO）场景。JWT的声明一般被用来在身份提供者和服务提供者间传递被认证的用户身份信息，以便于从资源服务器获取资源，也可以增加一些额外的其它业务逻辑所必须的声明信息，该token也可直接被用于认证，也可被加密。

## 如何实现

jwt由三部分组成，第一部分为头部（header)，第二部分为载荷（payload)
，第三部分是签名（signature)，将三部分组合在一起就构成了jwt字符串

### header 头部

头部主要描述了jwt的加密方式，以及类型（统一写成JWT）

```
{
  "typ": "JWT",
  "alg": "HS256"
}
```

将上面的json对象进行base64编码就得到了在jwt中作为第一部分的header。


### payload 载荷

载荷主要存储一些有效信息（也是json），比如过期时间、用户id等。载荷有默认标准字段（选填）

官方规定了7个载荷字段有：

```
iss: jwt签发者
sub: jwt所面向的用户
aud: 接收jwt的一方
exp: jwt的过期时间，这个过期时间必须要大于签发时间
nbf: 定义在什么时间之前，该jwt都是不可用的.
iat: jwt的签发时间
jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击。
```

也可以添加自定义字段如userId

```
{
  "userID": "123456",
  "role": "user",
  "exp": 1679718315
}
```

将上面的json对象进行base64编码就得到了在jwt中作为第二部分的payload。

### 签名

jwt的第三部分是一个签名信息，签名信息由三部分组成：

1. header(base64编码后的)
2. payload(base64编码后的)
3. secret(密匙用于HS256加密)

这个部分需要base64加密后的header和base64加密后的payload使用.连接组成的字符串，然后通过header中声明的加密方式进行加盐secret组合加密，然后就构成了jwt的第三部分。

```
// php
$base64String = base64_encode($header).base64_encode($payload);
$signature = hash_hmac('sha256', $base64String, 'secret');
```

最后"{$header}.{$payload}.{$signature}"这样拼接在一起jwt token就生成完毕了

## 优点

- 因为有了payload部分，所以JWT可以在自身存储一些其他业务逻辑所必要的非敏感信息。
- 便于传输，jwt的构成非常简单，字节占用很小，所以它是非常便于传输的。
- 它不需要在服务端保存会话信息, 所以它易于应用的扩展
- 无状态，因为授权服务器不需要维护任何状态；令牌本身就是验证令牌持有者授权所需的全部内容。

## 缺点

无法作废已颁布的令牌（作废需要服务端存储维护token，这样的话jwt就失去了无状态的意义）

# 参考文章
https://www.jianshu.com/p/576dbf44b2ae

https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html

https://learnku.com/articles/22616