---
title: jwt
date: 2020-05-30 21:35:42
tags: login
categories:
---

jwt 全称 JSON Web Token，用来解决请求的认证问题。
jwt 使用算法加密来认证的，所以服务器端无需储存任何信息。

# JWT 的三个组成部分

## 1. Header

头部由算法和令牌类型构成

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

将 Json 进行 Base64URL 编码得到第一部分

<!--- more --->

## 2. Payload

用来存放实际需要传递的数据，官方默认了 7 个数据

```
iss (issuer)：签发人
exp (expiration time)：过期时间
sub (subject)：主题
aud (audience)：受众
nbf (Not Before)：生效时间
iat (Issued At)：签发时间
jti (JWT ID)：编号
```

除此之外可添加私有字段  
将 Json 进行 Base64URL 编码得到第二部分

## 3. Signature

Signature 部分是对前两部分的签名，防止数据篡改。
将第一部分和第二部分的 base64 再加上密钥通过第一部分的算法加密，得到第三部分
密钥是自行设定，是加密的核心，不可以泄露。

**参考**  
http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html  
https://jwt.io/
