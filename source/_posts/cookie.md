---
title: 开发中遇到的cookie问题
date: 2022-11-25 14:03:23
categories:
  - FE
  - 浏览器
tags:
  - FE
  - 开发工具
  - cookie
---

## 前言

什么是 cookie

> HTTP Cookie（也叫 Web Cookie 或浏览器 Cookie）是服务器发送到用户浏览器并保存在本地的一小块数据。浏览器会存储 cookie 并在下次向同一服务器再发起请求时携带并发送到服务器上。通常，它用于告知服务端两个请求是否来自同一浏览器——如保持用户的登录状态。Cookie 使基于无状态的 HTTP 协议记录稳定的状态信息成为了可能。

## cookie 的参数

原文[链接](https://juejin.cn/post/6916157109906341902/#heading-48)

- Name：cookie 的名称
- Value：cookie 的值，对于认证 cookie，value 值包括 web 服务器所提供的访问令牌；
- Size： cookie 的大小
- Path：可以访问此 cookie 的页面路径。 比如 domain 是 abc.com，path 是/test，那么只有/test 路径下的页面可以读取此 cookie。
- Secure： 指定是否使用 HTTPS 安全协议发送 Cookie。使用 HTTPS 安全协议，可以保护 Cookie 在浏览器和 Web 服务器间的传输过程中不被窃取和篡改。该方法也可用于 Web 站点的身份鉴别，即在 HTTPS 的连接建立阶段，浏览器会检查 Web 网站的 SSL 证书的有效性。但是基于兼容性的原因（比如有些网站使用自签署的证书）在检测到 SSL 证书无效时，浏览器并不会立即终止用户的连接请求，而是显示安全风险信息，用户仍可以选择继续访问该站点。
- Domain：可以访问该 cookie 的域名，Cookie 机制并未遵循严格的同源策略，允许一个子域可以设置或获取其父域的 Cookie。当需要实现单点登录方案时，- Cookie 的上述特性非常有用，然而也增加了 Cookie 受攻击的危险，比如攻击者可以借此发动会话定置攻击。因而，浏览器禁止在 Domain 属性中设置.org、.com 等通用顶级域名、以及在国家及地区顶级域下注册的二级域名，以减小攻击发生的范围。
- HTTP： 该字段包含 HTTPOnly 属性 ，该属性用来设置 cookie 能否通过脚本来访问，默认为空，即可以通过脚本访问。在客户端是不能通过 js 代码去设置一个 httpOnly 类型的 cookie 的，这种类型的 cookie 只能通过服务端来设置。该属性用于防止客户端脚本通过 document.cookie 属性访问 Cookie，有助于保护 Cookie 不被跨站脚本攻击窃取或篡改。但是，HTTPOnly 的应用仍存在局限性，一些浏览器可以阻止客户端脚本对 Cookie 的读操作，但允许写操作；此外大多数浏览器仍允许通过 XMLHTTP 对象读取 HTTP 响应中的 Set-Cookie 头。
- Expires/Max-size ： 此 cookie 的超时时间。若设置其值为一个时间，那么当到达此时间后，此 cookie 失效。不设置的话默认值是 Session，意思是 cookie 会和 session 一起失效。当浏览器关闭(不是浏览器标签页，而是整个浏览器) 后，此 cookie 失效。

![cookie](https://blog.liufashi.top/img/cookie/1.png)
![脚本拿不到设置httpOnly的cookie](https://blog.liufashi.top/img/cookie/2.png)

## cookie 在项目中的使用

通常在我们项目中调用登录接口成功后，服务端会在响应头里面添加 setCookie 头，里面设置用户信息在服务端 session 中对应的 sessionId，然后浏览器在收到这个响应头的时候会在浏览器的 cookie 中写入 cookie，setCookie 时不做特殊处理的话 cookie 的 Domain 为你发起请求的域，path 为`/`

## cookie 的携带规则

cookie 的携带如果是客户端和服务端在同一个域下面的时候不需要客户端做任何处理，如果是跨域的资源请求的话则需要客户端在请求的时候允许跨域携带 cookie，在 axios 中设置`withCredentials: true,` fetch 中设置`credentials: 'include'`,服务端也需要设置`Access-Control-Allow-Credentials:true`来允许，当然前提是设置了`Access-Control-Allow-Origin"`。若是使用 nginx 做转发需要添加以下配置

```conf
location / {
     add_header 'Access-Control-Allow-Origin' 'your Domain';
     add_header 'Access-Control-Allow-Credentials' 'true';
     add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS';
     add_header 'Access-Control-Allow-Headers' 'DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type';
     proxy_set_header Cookie $http_cookie;
}
```

我们可以尝试着起两个服务试试

- 服务 A：在 8000 端口，使用 express 托管静态资源的方式
- 服务 B：在 9000 端口
- index.html 中引入 axios 然后点击按钮向不同服务发起请求

服务 A 代码,一个登录接口添加 cookie，一个 user 接口返回信息，然后托管 public 下的静态资源`public/index.html`

```js
// src/app1.js
const express = require("express");
const app = express();
// `index.html` 加载时会请求login接口
// 设置`cookie`
app.get("/login", (req, res) => {
  res.cookie("user", "fashi", { maxAge: 2000000, httpOnly: true });
  res.json({ code: 0, message: "登录成功" });
});

// 此接口是检测`cookie`是否设置成功，如果设置成功的话，浏览器会自动携带上`cookie`
app.get("/user", (req, res) => {
  const user = req.headers.cookie.split("=")[1];
  res.json({ code: 0, user });
});

// 托管`index.html`页面
// 这样的话在`index.html`中发起的请求，默认的源就是`http://localhost:8000`
// 然后再去请求`http://localhost:9000`就会出现跨域了
app.use(express.static("public"));

app.listen("8000", () => {
  console.log("app1 running at port 8000");
});
```

服务 B 代码，有一个接口用来测试 cookie 是否携带上

```js
// src/app2.js
const express = require("express");
const app = express();
// 定义一个接口，index.html页面请求这个接口就是跨域（因为端口不同）
app.get("/anotherService", (req, res) => {
  res.json({ code: 0, msg: "这是9000端口返回的" });
});

app.listen("9000", () => {
  console.log("app2 running at port 9000");
});
```

index.html 代码

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <h2>this is index.html at port 8000</h2>
    <button id="button">发送同源请求</button>
    <button id="cross-button">跨域请求</button>
    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
    <script>
      const button = document.querySelector("#button");
      const crossButton = document.querySelector("#cross-button");

      axios.get("http://localhost:8000/login", {}).then((res) => {
        console.log(res);
      });
      // 发送同域请求
      button.onclick = function () {
        axios.get("http://localhost:8000/user", {}).then((res) => {
          console.log(res);
        });
      };
      // 发送跨域请求http
      crossButton.onclick = function () {
        axios({
          method: "get",
          url: "http://localhost:9000/anotherService",
          withCredentials: true,
        }).then((res) => {
          console.log(res);
        });
      };
    </script>
  </body>
</html>
```

页面如下
{% gallery %}![页面](https://blog.liufashi.top/img/cookie/3.png){% endgallery %}

以下为 login set-cookie 和发送同源请求的时候携带的 cookie
{% gallery %}![cookie](https://blog.liufashi.top/img/cookie/4.png){% endgallery %}

发送跨域请求的时候直接是被跨域拦截了，当我们解决掉跨域，然后在 axios 中设置 withCredentials 和服务端设置 Access-Control-Allow-Credentials:true 参数后发现 cookie 正常携带

index.html 改动

```js
crossButton.onclick = function () {
        axios({
          method: "get",
          url: "http://localhost:9000/anotherService",
          + withCredentials: true,
        }).then((res) => {
          console.log(res);
        });
      };
```

服务 B 改动

```js
// src/app2.js
const express = require("express");
const app = express();
app.all("*", (req, res, next) => {
  +res.header("Access-Control-Allow-Origin", "http://localhost:8000");
  +res.header("Access-Control-Allow-Credentials", "true");
  next();
});
// 定义一个接口，index.html页面请求这个接口就是跨域（因为端口不同）
app.get("/anotherService", (req, res) => {
  res.json({ code: 0, msg: "这是9000端口返回的" });
});

app.listen("9000", () => {
  console.log("app2 running at port 9000");
});
```

结果
{% gallery %}![cookie](https://blog.liufashi.top/img/cookie/5.png){% endgallery %}

以上就是添加 withCredentials 和 Access-Control-Allow-Credentials 的解决方案

## 问题

以上的方式只能解决 domain 一致，因其他原因（如跨站，http 请求）引起的 cookie 无法携带的问题，也就是 Domain 一样的非跨站请求。随着浏览器（主要指 google）的更新，对于 cookie 的一些策略也发生改变，我们在请求 8000 端口时，有个 login 接口 set-cookie user=fashi, 我们在另外一个 9000 端口的请求中也 set-cookie test=abc，如下

```js
app.get("/anotherService", (req, res) => {
  +res.cookie("test", "abc", { maxAge: 2000000, httpOnly: true });
  res.json({ code: 0, msg: "这是9000端口返回的" });
});
```

再将 index.html 的代码修改一下，让他发送一个跨站请求，将 localhost 改为 ip 地址模拟跨站

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <h2>this is index.html at port 8000</h2>
    <button id="button">发送同源请求</button>
    <button id="cross-button">跨域请求</button>
    + <button id="cross-site-button">跨站请求</button>

    <script src="https://cdn.jsdelivr.net/npm/axios/dist/axios.min.js"></script>
    <script>
      const button = document.querySelector("#button");
      const crossButton = document.querySelector("#cross-button");
      + const crossSiteButton = document.querySelector("#cross-site-button");

      axios.get("http://localhost:8000/login", {}).then((res) => {
        console.log(res);
      });
      // 发送同域请求
      button.onclick = function () {
        axios.get("http://localhost:8000/user", {}).then((res) => {
          console.log(res);
        });
      };
      // 发送跨域请求http
      crossButton.onclick = function () {
        axios({
          method: "get",
          url: "http://localhost:9000/anotherService",
          withCredentials: true,
        }).then((res) => {
          console.log(res);
        });
      };
      + crossSiteButton.onclick = function () {
      +   axios({
      +     method: "get",
      +     url: "http://172.160.96.172:9000/anotherService",
      +     withCredentials: true,
      +   }).then((res) => {
      +     console.log(res);
      +   });
      + };
    </script>
  </body>
</html>
```

{% gallery %}![cookie](https://blog.liufashi.top/img/cookie/6.png){% endgallery %}

然后我们发现这个响应头中的`set-cookie`写入失败，浏览器提示`sameSite`默认是`Lax`，因为是一个跨站请求而失败，需要我们设置`SameSite`属性为`None`解决。这就说明我当前的浏览器是不允许跨站写 cookie 的（和浏览器版本有关 后面会提到），同样跨站的 cookie 也是不允许携带的

## 尝试解决

- 低版本浏览器，以 chrome8.9 为例，浏览器地址栏输入：chrome://flags/，搜索找到：SameSite
  {% gallery %}![cookie](https://blog.liufashi.top/img/cookie/7.png){% endgallery %}
  有三个结果，将他们设置为 disable，此时相当于添加上了 SameSite=None；Secure
- 高版本浏览器只能是让服务端手动设置 SameSite=None；Secure 属性，我们按照这个操作一下，修改 B 服务代码

```js
app.get("/anotherService", (req, res) => {
  res.cookie("test", "abc", {
    maxAge: 2000000,
    httpOnly: true,
    + sameSite: "none",
    + secure: true,
  });
  res.json({ code: 0, msg: "这是9000端口返回的" });
});
```

再次发起请求，如下
{% gallery %}![cookie](https://blog.liufashi.top/img/cookie/8.png){% endgallery %}
然后 set-cookie 还是未成功，提示说设置 sameSite 为 none 之后连接必须是安全的，从上面 secure 字段的介绍，我们不难猜到需要 https。那就在我的云服务器上起一个服务，代码同服务 B，修改 IP 地址为服务器域名。结果如下
{% gallery %}![cookie](https://blog.liufashi.top/img/cookie/9.png){% endgallery %}
我们发现 set-cookie 成功并且 cookie 携带上了，这个时候如果时使用的 nginx 做的转发则还需要加上上述的 nginx 配置，因为现在 cookie 只是到了代理服务器，服务端还需要从代理服务器拿到 cookie。

## 解决方案

到这里相信 cookie 无法携带的问题的原因都已经了解了，上面没有理解也没关系，这里直接贴出解决方案

1. 场景一：服务端和客户端部署时候在同一个域同端口下
   我们不需要做特殊处理，但是我们需要在开发环境做一些优化，不然本地调试无法验证接口。低版本浏览器直接修改以上的 sameSite 设置即可，但是呢我们一般都会使用新版的，毕竟一个更新按钮挂在那里也难受，一些新的方便的功能享受不到。本地调试无法携带 cookie 的根本原因就是请求服务端的地址要么是测试环境地址或者后端的 ip 地址，这个时候无论是从`localhost`请求`https://[测试地址]`还是从`localhost`请求`局域网ip`都是一个跨站请求，需要后端设置`SameSite=None;Secure`，而且该设置在请求后端局域网 Ip 的时候还无效，因为 not Secure。那我们就可以从根本上解决这个问题，让他不跨站，这个时候需要我们 webpack 中的`devServer`出马了，只需要将请求通过 devServer 代理 从`localhost-> localhost` 这样开发环境就可以正常携带上 cookie 了。
   如下
{% gallery %}![cookie](https://blog.liufashi.top/img/cookie/10.png){% endgallery %}

2. 场景二：服务端和客户端部署时候在同一个域不同端口或不同三级域名下
   这个时候需要设置`withCredentials`和`Access-Control-Allow-Credentials`，其余操作一样

3. 场景三：不是同一个站点，类似于第三方cookie，这个时候就需要设置sameSite属性了
   另外若是有特殊的登录情况（也很常见），如第三方授权登录的时候，调用服务端接口，服务端调用第三方登录接口，这个时候cookie是保存在请求地址的那个域下面的，若是在开发环境使用devServer无法将该cookie携带到请求的域，因为你的localhost上没有这个cookie，这个时候需要手动将测试环境的cookie添加到浏览器中，这样devServer就会在请求的时候携带上。97版本的chrome（就是无法手动操作cookie的版本都可以这么设置）默认不允许手动操作cookie，需要修改设置，如下：浏览器输入 `chrome://flags/` 搜索 Partitioned Cookies ，修改为 enabled,点击 relaunch 重启浏览器即可。
