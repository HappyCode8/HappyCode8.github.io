# JJWT

一个JWT**(java web token)**由三部分组成

- Header(头部)——base64编码当然Json字符串，里边说明类型和使用的算法
- Payload(载荷)——base64编码的Json字符串，里边存放有效信息
- Signature(签名)——使用指定算法，通过Header和Payload加盐计算的字符串

各部分以”.“分隔，例如

`eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ3eWoiLCJleHAiOjE2MDI0MTU0NTUsImlhdCI6MTYwMjQxMzY1NX0.gNZZdUztdC9qwzInC_wyYA5dGbAHps6i2yy_5_7IVE4`

直接通过base64解码即可获得[base64在线加解密](https://tool.oschina.net/encrypt?type=3)，上述JWT的Header和Payload分别为

` {"alg":"HS256"}{"sub":"wyj","exp":1602415455,"iat":1602413655}`

请求的时候在HTTP的headers参数里面的authorization里边，值的前面加`Bearer`关键字和空格，除此之外也可以在url和request body中传递

## Header

如上所示，Header里主要说明类型和使用算法

## Payload

### 标准中注册的声明

- iss： jwt签发者
- sub: jwt所面向的用户
- aud: 接收jwt的一方
- exp: jwt的过期时间，这个过期时间必须要大于签发时间
- nbf: 定义在什么时间之前，该jwt都是不可用的.
- iat: jwt的签发时间
- jti: jwt的唯一身份标识，主要用来作为一次性token,从而回避重放攻击

### 公共的声明

可以添加任何信息，但不建议添加敏感信息

### 私有的声明

私有声明是提供者和消费者所共同定义的声明

## Signature

```java
encodedString = base64UrlEncode(header) + '.' + base64UrlEncode(payload);
signature = HMACSHA256(encodedString, 'secret');
```

其中的**’secret‘**是服务端的私钥，需要严格保密，如果客户端获得了这个秘钥，那就可以自己签发token了。**服务端主要通过第三部分来确认这个token确实是服务端签发的，因为只有服务端能通过秘钥根据Header与Payload验证重新加密对比签名**。

## 签名算法

HS256:对称签名

RS256:非对称签名

