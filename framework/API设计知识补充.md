# API设计知识补充


## [什么是cookie，什么是session?](https://juejin.cn/post/7133758754818326564)
  
   * Cookie: 采用的是在客户端保存状态和数据的方案 （客户浏览器上），单个Cookie保存的数据不能超过4K
       
        ps: 为了防止客服端伪造登录ID，或者token令牌，这里一般会用到AES等对称加密技术对真实数据进行加密，然后在服务端进行解密
           
   * Session： 采用在服务器端保存状态和数据的方案 （服务器上），当访问增多，会比较占用大量资源

   * Token:  这种方式跟session的方式流程差不多，不同的地方在于保存的是一个token值到redis，
       token一般是一串随机的字符(比如UUID)，value一般是用户ID，并且设置一个过期时间。每次请求服务的时候带上token在请求头，
      后端接收到token则根据token查一下redis是否存在，如果存在则表示用户已认证，如果token不存在则跳到登录界面让用户重新登录，
      登录成功后返回一个token值给客户端。   

## [什么是jwt](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)

  有header.payload.signature 三部分组成的字符串
   Header 部分是一个 JSON 对象，描述 JWT 的元数据,包含签名算法信息
   
   ```
   {
    "alg": "HS256",
      "typ": "JWT"
   }
   ```

   Payload 部分也是一个 JSON 对象，用来存放实际需要传递的数据。JWT 规定了7个官方字段，供选用。

   ```
     iss (issuer)：签发人
     exp (expiration time)：过期时间
     sub (subject)：主题
     aud (audience)：受众
     nbf (Not Before)：生效时间
     iat (Issued At)：签发时间
     jti (JWT ID)：编号

   ```

Signature 部分是对前两部分的签名，防止数据篡改。

首先，需要指定一个密钥（secret）。这个密钥只有服务器才知道，不能泄露给用户。然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照下面的公式产生签名。
```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```
算出签名以后，把 Header、Payload、Signature 三个部分拼成一个字符串，每个部分之间用"点"（.）分隔，就可以返回给用户。
