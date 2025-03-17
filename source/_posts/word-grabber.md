---
title: 双端拾词助手
date: 2025-03-08 23:00:00
tags:
  - 浏览器插件
  - 微信小程序
  - node
categories:
  - 指南
cover: https://cdn.jsdelivr.net/gh/chendx97/CPics/img/20250308231025.png
---
# 介绍
这是一个帮助用户在日常中学习单词的软件，包含浏览器插件和微信小程序。

使用流程：
1.安装浏览器插件；
2.微信扫码登录小程序；
3.在网页中选中单词，即可获取翻译结果，可以收藏或取消收藏该单词；
4.在微信小程序中复习单词；

浏览器插件：

![收藏](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/%E6%8D%95%E8%8E%B7.PNG)    

微信小程序：

![小程序](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/20250308231025.png)

# 开发浏览器插件注意事项
## content.js与background.js的区别
content.js：
- 直接与网页内容交互，可以修改网页结构、监听用户点击事件等；
- 无法访问页面的js变量或函数，请求受同源策略限制；
- 仅能调用部分Chrome API，如chrome.runtime.sendMessage、chrome.storage；
- 无法直接访问chrome.tabs或chrome.windows等涉及浏览器全局操作的API


background.js：
- 作为全局后台服务，负责处理跨域请求、调用高权限API（通知推送、浏览器标签管理等）；
- 生命周期与浏览器同步，支持跨域访问；
- 可调用所有Chrome API；
## 消息传递
content -> background
```js
// content script发送消息
chrome.runtime.sendMessage({ action: "fetchData" }, response => {
  console.log("收到后台响应:", response);
});

// background script监听
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  if (request.action === "fetchData") {
    sendResponse({ data: "来自后台的数据" });
  }
});
```
background -> content
需通过chrome.tabs.sendMessage指定目标标签页
```
chrome.tabs.query({ active: true }, tabs => {
  chrome.tabs.sendMessage(tabs[0].id, { action: "updateUI" });
});
```
## storage存储数据
```js
// 存储
chrome.storage.local.set({ info: 'abc' });

// 获取
chrome.storage.local.get(['info'], (result) => {
  console.log(result);
});
```
## 生命周期钩子
```js
// 监听扩展安装和更新事件
chrome.runtime.onInstalled.addListener(() => {
  // ...
});

// 监听扩展启动事件
chrome.runtime.onStartup.addListener(() => {
  // ...
});
```
# 开发微信小程序注意事项
## 扫码登录
微信扫码登录小程序的流程如下：
1. 服务端根据scene从小程序API获取二维码；
2. 浏览器插件从服务端获取并展示二维码，轮询是否登录；
3. 手机扫描二维码，微信小程序可以从query中拿到scene。登录后拿到token，请求登录接口，返回scene和token；
4. 服务端以scene为key记录是否登录。缓存scene和token，查询是否登录根据sessionStorage中的数据判断；
5. 浏览器插件查询到已登录，结束轮询；

主要代码如下：
```js
// 获取accessToken
fetch(`https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid=${appid}&secret=${secret}`)

// 根据accessToken获取二维码
const scene = randomBytes(16).toString('hex');
const response = await fetch(`https://api.weixin.qq.com/wxa/getwxacodeunlimit?access_token=${accessToken}`, {
    method: 'POST',
    body: JSON.stringify({
      scene,
      width: 200,
    }),
  });
const buffer = await response.arrayBuffer();
const base64Image = Buffer.from(buffer).toString('base64');
  res.jsonSuccess({
    qrcode: `data:image/png;base64,${base64Image}`,
    scene,
});
```
## 获取用户信息
`wx.getUserInfo`和`wx.getUserProfile`已经被废弃，无法拿到用户头像和昵称。

需要让用户自己选择头像和昵称。
```
<!-- 用户信息弹窗 -->
<view class="user-info-modal" wx:if="{{isLoggedIn && (!userName || !userAvatar)}}">
  <view class="user-info-content">
    <button class="avatar-wrapper" open-type="chooseAvatar" bind:chooseavatar="onChooseAvatar">
      <image class="avatar" src="{{userAvatar}}"></image>
    </button>
    <input type="nickname" class="weui-input" placeholder="请输入昵称" bindinput="onInputName" />
  </view>
</view>
```
## 上传微信头像
```
const { userName, userAvatar, scene } = this.data
wx.uploadFile({
  url: 'xx',
  filePath: userAvatar,
  name: 'file',
  formData: {
    openid: wx.getStorageSync('token'),
    scene,
    nickname: userName
  },
})
```

# 总结
这是一个日常用得上的项目，又锻炼了开发浏览器插件、微信小程序、node。最复杂的是扫码登录那块。

ps: 因为发布浏览器插件需要付费，so，可以通过 [下载插件](https://github.com/chendx97/word-grabber-extension) 来手动安装浏览器插件。   

小程序二维码：
![小程序二维码](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/%E5%B0%8F%E7%A8%8B%E5%BA%8F.png) 