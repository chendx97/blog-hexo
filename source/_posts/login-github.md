---
title: 🔔 如何接入github登录
date: 2025-05-27 22:30:00
tags:
  - github
  - 登录
categories:
  - 指南
cover: 
---

github登录通过**OAuth**协议接入第三方平台，用户无需注册新账号。下面详细介绍一下接入流程。

# 创建 GitHub OAuth 应用
访问 [GitHub Developer Settings](https://github.com/settings/developers) 创建新应用。填写 `Application name`、`Homepage URL`、`Authorization callback URL`。保存生成的`Client ID` 和 `Client Secret`。

==！Client Secret 需要另外记录下来，它只在生成的时候可见。==

# 跳转到授权页
```js
const handleLogin = () => {
  window.location.href = 'https://github.com/login/oauth/authorize?client_id=Ov23li276DdOmpC29BIR&scope=read:user';
}
```
判断是否登录，如果没有登录则跳转到 github 的登录授权页。

# 授权后返回code
第一步填写了`Authorization callback URL`，所以用户授权后会返回此页面。在此页面中，可以从url的query参数中拿到`code`。
```
const urlParams = new URLSearchParams(window.location.search);
  const code = urlParams.get('code');
```

# 后端用code换取Access Token
前端将上一步的`code`传递给后端。
后端用此`code`获取`Access Token`。
```js
const { code } = req.query;

const params = new URLSearchParams();
params.append('client_id', process.env.GITHUB_CLIENT_ID);
params.append('client_secret', process.env.GITHUB_CLIENT_SECRET);
params.append('code', code);
const tokenRes = await fetch('https://github.com/login/oauth/access_token', {
  method: 'POST',
  body: params.toString(),
  headers: {
    'Content-Type': 'application/x-www-form-urlencoded',
    'Accept': 'application/json'
  },
  agent,
}).then(res => res.json());
if (tokenRes.error === 'bad_verification_code') {
  return res.jsonFail('code无效');
}
const accessToken = tokenRes.access_token;
```

# 用Access Token获取用户信息
```
const userInfo = await fetch('https://api.github.com/user', {
  headers: {
    Authorization: `Bearer ${accessToken}`,
    'User-Agent': process.env.APP_NAME,
  },
}).then(res => res.json());
```

# 存储用户信息并返回
```
let user = await User.findOne({ name: userInfo.login });
if (!user) {
  user = await User.create({
    name: userInfo.login,
    avatar: userInfo.avatar_url,
    githubUrl: userInfo.html_url,
    token: accessToken,
    createdAt: Date.now(),
    lastLoginAt: Date.now(),
  });
} else {
  user.lastLoginAt = Date.now();
  await user.save();
}
res.jsonSuccess(user);
```

# 总结
接入github登录的流程总结如下：
1.创建GitHub OAuth应用
2.点击按钮，跳转至GitHub授权页
3.用户点击授权后，返回code
4.携带code请求后端，后端用code换取Access Token，并用Access Token获取用户信息
5.后端返回用户信息