---
title: 为什么只这一个接口502呢？
date: 2024-01-20 10:00:00
tags:
  - node
  - 502
categories:
  - bug
cover: 
---

# 前言
前几天遇到了一个没见过的bug，在此记录一下。前几天，新写了一个页面，里面有get请求、json参数的post请求、form参数的post请求，然后把这个项目打包出来的dist放入node中。更新到线上以后，发现仅有json参数的post请求出现nginx 502报错，其他接口都正常。
# 解决问题的思路
刚开始搜索了一下**nginx 502**，没发现什么有用信息。  

首先是发现其他get接口都正常，点击按钮时这个post接口报错，当时就有点懵。然后检查了同一个项目另一个post接口，发现另一个post接口也正常。所以，推理得**json类型参数的post请求**有问题。  

本地环境，这个json参数的请求没问题，测试环境是用dist+nginx部署的，这个请求也没问题。线上环境是**dist+node**部署，所以就定位到了http-proxy-middleware这个库。

在node+dist项目中，如果接口就是这个node的就不用代理了，但如果前端代码中请求了别的项目的接口，就需要用http-proxy-middleware添加代理配置。

搜索**http-proxy-middleware json请求502**，就发现了解决方案。
# 问题原因
在node项目中，会使用bodyParser这个库来解析post请求的参数，这个库会改写所有post请求的body，所以前端代码请求其他项目的post请求的body也会被修改。
# 解决办法
在代理别的项目的接口时修改body。

```js
app.use('/myapi', createProxyMiddleware({
  target: 'http://x.x.x.x:x',
  pathRewrite: (path) => path.replace("/myapi/", "/"),
  on: {
    proxyReq: (proxyReq, req: Request, res) => {
      if (!req.body || !Object.keys(req.body).length) {
        return;
      }

      const contentType = proxyReq.getHeader('Content-Type');
      const writeBody = (bodyData: string) => {
          proxyReq.setHeader('Content-Length', Buffer.byteLength(bodyData));
          proxyReq.write(bodyData);
      };

      if (contentType === 'application/json') {
        writeBody(JSON.stringify(req.body));
      }

      if (contentType === 'application/x-www-form-urlencoded') {
        writeBody(querystring.stringify(req.body));
      }
    },
  },
}));
```
在这个issue[参考issue](https://github.com/chimurai/http-proxy-middleware/issues/320)中发现了解决办法，然后又在[http-proxy-middleware的readme](https://github.com/chimurai/http-proxy-middleware?tab=readme-ov-file#intercept-and-manipulate-requests)中发现了相关记载。

![image.png](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/3aa13c5da7ea435998ff9c2ac28314c5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)
