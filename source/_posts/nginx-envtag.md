---
title: web项目的灰度发布/金丝雀发布
date: 2022-09-02 14:06:59
categories:
  - FE
tags:
  - nginx
  - 云服务器
  - CI/CD
---

## 前言

什么是灰度发布？

> 灰度发布，又被称之为金丝雀发布，是指某次新发布功能特性和旧功能特性之间能够以平滑过渡的方式呈现给用户，就像金丝雀的羽毛一样多种颜色平滑渐变。

对于前端而言灰度发布的作用

1. 一个项目多需求同时开发，需求测试阶段为了互相之间不影响
2. 重要更新时，测试新版本的性能和表现，以保障整体系统稳定的情况下，尽早发现、调整问题，降低风险

## 基本原理

使用`nginx` `$http_[自定义请求头]`读取自定义的请求头，根据请求头不同指向不同资源。

```conf
server {
	listen 443 ssl;#配置HTTPS的默认访问端口为443。
    #如果未在此处配置HTTPS的默认访问端口，可能会造成Nginx无法启动。
    #如果您使用Nginx 1.15.0及以上版本，请使用listen 443 ssl代替listen 443和ssl on。
    server_name admin.template.liufashi.top; #需要将yourdomain替换成证书绑定的域名。
    ssl_certificate ssl/admin/admin.template.liufashi.top.pem;  #需要将cert-file-name.pem替换成已上传的证书文件的名称。
    ssl_certificate_key  ssl/admin/admin.template.liufashi.top.key; #需要将cert-file-name.key替换成已上传的证书私钥文件的名称。
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;#表示使用的加密套件的类型。
    ssl_protocols TLSv1.1 TLSv1.2 TLSv1.3; #表示使用的TLS协议的类型。
    ssl_prefer_server_ciphers on;
	underscores_in_headers on; #开启header下划线支持
    location / {
		root /apps/react-admin/build;
        if ($http_envtag = "abcde"){
            root /apps/react-admin/gradation;
        }
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
    }
}
```

以上代码中是将带有`envtag:"abcde"`代理到不同的静态资源，实现灰度环境

## 使用

实现上面的根据请求头代理之后灰度环境是有了，但是是我们该怎么手动这两者之间切换呢？
这个时候我们就要借助浏览器插件[ModHeader](https://chrome.google.com/webstore/detail/modheader/idgpnmonknjnojddfkpgkljpfnnfcklj?hl=zh)
下载之后添加请求头名称和值
{% gallery %}![添加请求头](https://blog.liufashi.top/img/nginx-envtag/1.png){% endgallery %}
然后我们可以打开控制台，-> Network->刷新 然后你就可以看到请求头被加上去了
{% gallery %}![结果](https://blog.liufashi.top/img/nginx-envtag/2.png){% endgallery %}

然后就可以添加不同的请求头来达到切换灰度环境的效果，大家可以尝试下通过开关`envtag：abcde`来看看我这个项目的新老版本[www.liufashi.top](https://www.liufashi.top)

## 完整流程

正常在项目开发中的完整流程应该是类似这样的：

1. 需求来了
2. 前端写页面，后端写接口
3. 后端接口写完，确定灰度标签（请求头名称和值）发布到灰度环境
4. 前端根据该灰度标签确定此次发布指向的 pod（容器）。

## 结合 jenkins 实现

后面将会更新
