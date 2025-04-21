---
title: 🎀用流式写入处理大文件下载
date: 2025-04-21 21:30:00
tags:
  - http
categories:
  - 问题
cover: https://cdn.jsdelivr.net/gh/chendx97/CPics/img/cover%20(3).png
---

# 介绍
流式写入直接将数据流写入用户磁盘，避免传统方法(如`Blob`或`URL.createObjectURL`)因内存限制导致的大文件下载问题。
`StreamSaver.js`库使用浏览器原生的 [Streams API](https://developer.mozilla.org/zh-CN/docs/Web/API/Streams_API)，逐块写入数据到磁盘。通过 `Service Worker` 和中间人(`MITM`)技术，模拟服务器响应，绕过浏览器对下载文件大小的限制。而且，不需要服务端做任何修改。

# 使用
[StreamSaver文档](https://github.com/jimmywarting/StreamSaver.js)  

安装：
```bash
npm i streamsaver
```
使用：
```js
import { createWriteStream } from 'streamsaver';

const fileStream = createWriteStream('download.json');
const writer = fileStream.getWriter();

function download () {
    const response = await fetch('');
    const contentLength = response.headers.get('content-length');
    let receivedBytes = 0;
    const reader = response.body.getReader();
    // 分块处理
    while (true) {
        const { done, value } = await reader.read();
         if (done) break;
          
        // 写入文件流
        await writer.write(value);
          
        // 更新进度（如果有内容长度）
        if (contentLength) {
            receivedBytes += value.length;
          }
    }
    await writer.close();
}
```
# 原理
1. 用户点击下载按钮；
2. 动态创建一个隐藏的iframe，加载MITM脚本；
3. MITM脚本在iframe中注册Service Worker，并声明其作用域；
4. 主页面和iframe通过postMessage通信，传递数据快；
5. Service Worker接收数据，通过流式API写入本地文件；

浏览器要求文件下载必须由用户主动触发，如点击事件。iframe的创建和MITM脚本的加载会在用户点击事件的同步上下文中完成。这样，后续通过iframe触发的下载操作仍然被视为用户手势的延续，避免被浏览器阻止。   
浏览器默认禁止脚本直接操作本地文件系统，且下载操作通常需要与当前页面同源。该iframe的源被设置为一个独立的、与主页面不同的虚拟URL，从而创建一个“隔离的上下文”。这个隔离的上下文可以绕过主页面的一些安全策略，允许直接与Service Worker通信并触发下载。
Service Worker需要注册在特定的作用域下，且通常需要与页面同源。iframe中加载的MIMT脚本会动态注册一个Service Worker，并控制其作用域。通过将Service Worker隔离在iframe中，可以避免与主应用的Service Worker冲突，同时确保下载逻辑的独立性。
主页面通过postMessage向iframe发送数据库，iframe中的MIMT脚本将数据转发给Service Worker，Service Worker将数据流式写入磁盘。

# 缺点
因为通过iframe处理下载逻辑，生产环境需要使用`https`，否则会有不安全混合内容限制。  
如果不是`https`，则 `StreamSaver` 会改用`popup`，下载时页面左上角会有小弹窗闪现。

# 更好的方法
如果不考虑浏览器兼容性，可以用这个方法。
```js
try {
    const response = await fetch(...);
    
    const reader = response.body?.getReader();
    const chunks = [];

    while (true) {
      const { done, value } = await reader!.read();
      if (done) break;
      chunks.push(value);
    }

    const blob = new Blob(chunks);
    const link = document.createElement('a');
    link.href = URL.createObjectURL(blob);
    link.download = 'download.zip';
    link.click();
    URL.revokeObjectURL(link.href);
  } catch (error) {
    ElMessage.error('导出失败');
    console.error(error);
  }
```
# 总结
`streamsaver`最好在https环境中使用，`getReader`可能不兼容旧浏览器。
`streamsaver`先选择保存位置再下载，`getReader`先下载再选择保存位置。