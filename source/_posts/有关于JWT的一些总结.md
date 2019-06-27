---
title: 有关于JWT的一些总结
date: 2019-06-26 10:12:24
tags: [JWT, token, session]
---

在如今的前后端分离应用中，用户的认证和鉴权是相当重要的一环。实践中存在很多可用方案，各有千秋。
<!--more-->
常见的服务端认证方法有基于cookie的认证，如session;以及token认证，如JWT。前者依赖于cookie实现，在每次请求时都需要带上cookie，取出相应字段并与服务器端进行对比，以实现身份的认证。而后者仅仅需要在HTTP的头部附上token，由服务器check signature即可实现，无须担心cookie存在的CORS问题。

<br/>

#### 基于Session的会话管理

在web应用发展的初期，大部分采用基于Session的会话管理方式，逻辑如下：
1. 客户端使用用户名密码进行认证
2. 服务端生成并存储session，将sessionID通过cookie返回给客户端。
3. 客户端访问需要认证的接口时在cookie中携带sessionID
4. 服务端通过sessionID查找session并进行鉴权，返回客户端需要的数据。

![session会话管理逻辑逻辑](./session_check.jpg)

基于 Session 的方式存在多种问题：

1. 服务端需要存储session，并且由于session需要经常快速查找，通常存储在内存或内存数据库中，同时在线用户较多时需要占用大量的服务器资源
2. 当需要拓展时，创建session的服务器可能不是验证session的服务器，所以还需要将所有session单独存储并共享。
3. 由于客户端使用cookie存储sessionID，在跨域场景下需要进行兼容性处理，同时这种方式也难以防范CSRF攻击。

<br/>

#### 基于 Token 的会话管理

鉴于基于Session的会话管理方式存在上述多个缺点，无状态的基于Token的会话管理方式应运而生。
所谓无状态，就是服务端不再存储信息，甚至是不再存储session，逻辑如下：

1. 客户端使用用户密码进行认证
2. 服务端验证用户密码，通过购生成token返回给客户端
3. 客户端保存token，访问需要认证的接口时，在URL参数或是HTTP Header中加入token
4. 服务端通过解码token进行鉴权，返回给客户端需要的数据

![token会话管理验证逻辑](./token_check.jpg)

基于Token的会话管理方式有效解决了基于Session的会话管理方式带来的问题。

1. 服务端不需要存储和用户鉴权有关的信息，鉴权信息会被加密到Token中，服务端只需要读取token中包含的鉴权信息即可。
2. 避免了共享session导致的不易拓展问题。
3. 不需要依赖cookie，有效避免cookie带来的CSRF攻击问题
4. 使用CORS可以快速解决跨域问题

<br/>

#### JWT介绍

JWT是JSON Web Token的缩写，JWT本身没有定义任何技术实现，它只是定义了一种基于Token的会话管理的规则，涵盖Token需要包含的标准内容和Token的生成过程。

JWT由头部（Header）、负载（Payload）和签名（Signature）三部分组成，三部分的内容分别单独经过了Base64编码，以`.`拼接成一个JWT Token。

JWT的Header中存储了所使用的的加密算法和Token类型。
```
{
    "alg": "HS256",
    "typ": "JWT"
}
```

Payload是负载，JWT规范规定了一些字段，并推荐使用，开发者也可以自己指定字段和内容，例如：

```
{
  username: 'yage',
  email: 'sa@simpleapples.com',
  role: 'user',
  exp: 1544602234
}
```

需要注意的是，Payload的内容只经过了Base64编码，对客户端来说相当于明文存储，所以不要放置敏感信息。

Signature部分用来验证JWT Token是否被篡改，所以这部分会使用一个Secret将前两部分加密，逻辑如下：
`HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)`

<br/>

#### 它有什么优点？

* 1.充分依赖无状态 API ，契合 RESTful 设计原则（无状态的 HTTP）
```
状态：请求的状态是 client 与 server 交互过程中，保存下来的相关信息，客户端的保存在 page/request/session/application 或者全局作用域中，而 server 的一般存在 session 中。

有状态 API：server 保存了 client 的请求状态， server 通过 client 传递的 sessionID 在其 session 作用域内找到之前交互的信息并应答。

无状态 API：无状态是 RESTful 架构设计的一个非常重要的原则。无状态 API 的每一个请求都是独立的，它要求客户端保存所有需要的认证信息，每次发请求都要带上自己的状态，以 url 的形式提交包含 cookies 等状态的数据。
```
JWT 的设计契合无状态原则：用户登录之后，服务器会返回一串 token 并保存在本地，在这之后的服务器访问都要带上这串 token，来获得访问相关路由、服务及资源的权限。比如单点登录就比较多地使用了 JWT，因为它的体积小，并且经过简单处理（使用 HTTP 头带上 Bearer 属性 + token ）就可以支持跨域操作。

* 2.易于实现 CDN，将静态资源分布式管理
在传统的 session 验证中，服务端必须保存 session ID，用于与用户传过来的 cookie 验证。而一开始 sessionID 只会保存在一台服务器上，所以只能由一台 server 应答，就算其他服务器有空闲也无法应答，无法充分利用到分布式服务器的优点。 JWT 依赖的是在客户端本地保存验证信息，不需要利用服务器保存的信息来验证，所以任意一台服务器都可以应答，服务器的资源也能被较好地利用。

* 3 验证解耦，随处生成
无需使用特定的身份验证方案，只要拥有生成 token 所需的验证信息，在何处都可以调用相应接口生成 token，无需繁琐的耦合的验证操作，可谓是一次生成，永久使用。

* 4.比 cookie 更支持原生移动端应用
原生的移动应用对 cookie 与 session 的支持不够好，而对 token 的方式支持较好。



<br/>

#### JWT 优势 & 问题

JWT 拥有基于 Token 的会话管理方式所拥有的一切优势，不依赖 Cookie，使得其可以防止 CSRF 攻击，也能在禁用 Cookie 的浏览器环境中正常运行。

而 JWT 的最大优势是服务端不再需要存储 Session，使得服务端认证鉴权业务可以方便扩展，避免存储 Session 所需要引入的 Redis 等组件，降低了系统架构复杂度。但这也是 JWT 最大的劣势，由于有效期存储在 Token 中，JWT Token 一旦签发，就会在有效期内一直可用，无法在服务端废止，当用户进行登出操作，只能依赖客户端删除掉本地存储的 JWT Token，如果需要禁用用户，单纯使用 JWT 就无法做到了。

<br/>

#### 基于 JWT 的实践


既然 JWT 依然存在诸多问题，甚至无法满足一些业务上的需求，但是我们依然可以基于 JWT 在实践中进行一些改进，来形成一个折中的方案，毕竟，在用户会话管理场景下，没有银弹。

前面讲的 Token，都是 Access Token，也就是访问资源接口时所需要的 Token，还有另外一种 Token，Refresh Token，通常情况下，Refresh Token 的有效期会比较长，而 Access Token 的有效期比较短，当 Access Token 由于过期而失效时，使用 Refresh Token 就可以获取到新的 Access Token，如果 Refresh Token 也失效了，用户就只能重新登录了。

在 JWT 的实践中，引入 Refresh Token，将会话管理流程改进如下。

1. 客户端使用用户名密码进行认证
2. 服务端生成有效时间较短的 Access Token（例如 10 分钟），和有效时间较长的 Refresh Token（例如 7 天）
3. 客户端访问需要认证的接口时，携带 Access Token
4. 如果 Access Token 没有过期，服务端鉴权后返回给客户端需要的数据
5. 如果携带 Access Token 访问需要认证的接口时鉴权失败（例如返回 401 错误），则客户端使用 Refresh Token 向刷新接口申请新的 Access Token
6. 如果 Refresh Token 没有过期，服务端向客户端下发新的 Access Token
7. 客户端使用新的 Access Token 访问需要认证的接口

![JWT实践](./jwt.jpg)

将生成的 Refresh Token 以及过期时间存储在服务端的数据库中，由于 Refresh Token 不会在客户端请求业务接口时验证，只有在申请新的 Access Token 时才会验证，所以将 Refresh Token 存储在数据库中，不会对业务接口的响应时间造成影响，也不需要像 Session 一样一直保持在内存中以应对大量的请求。

上述的架构，提供了服务端禁用用户 Token 的方式，当用户需要登出或禁用用户时，只需要将服务端的 Refresh Token 禁用或删除，用户就会在 Access Token 过期后，由于无法获取到新的 Access Token 而再也无法访问需要认证的接口。这样的方式虽然会有一定的窗口期（取决于 Access Token 的失效时间），但是结合用户登出时客户端删除 Access Token 的操作，基本上可以适应常规情况下对用户认证鉴权的精度要求。

<br/>

#### 思考：
token存储到localstorage或cookie哪一个会更安全？

存储在localstorage中
优点：
方便存取，符合Restful 最佳实践。
缺点：
localstorage可以被 javascript 访问，所以容易受到XSS攻击。尤其是项目中用到很多第三方的Javascript类库。另外，需要应用程序来保证Token只在HTTPS下传输。

存储在Cookie中
优点：
可以指定 httponly，来防止被Javascript读取，也可以指定secure，来保证token只在HTTPS下传输。
缺点：
不符合Restful 最佳实践。
容易遭受CSRF攻击 （可以在服务器端检查 Refer 和 Origin）

总的来说，将令牌存储在设置为httpOnly的cookie中通常被认为是最安全的。不过在大多数情况下，将令牌存储在localstorage中也是可以接受的。更重要的是根据项目需求进行选择，假如你需要将令牌发送到多个后端（在不同的起源上），你就不能将它放在cookie中，因为那只会被发送到它自己的源头。

<br/>

参考资料: &ensp;Server端的认证神器——JWT(一) <https://zhuanlan.zhihu.com/p/27370773>
         &emsp;&emsp;&emsp;&emsp;&emsp;基于 JWT + Refresh Token 的用户认证实践 <https://zhuanlan.zhihu.com/p/52300092> 
         &emsp;&emsp;&emsp;&emsp;&emsp;前端应该知道的web登录 <https://zhuanlan.zhihu.com/p/62336927>


