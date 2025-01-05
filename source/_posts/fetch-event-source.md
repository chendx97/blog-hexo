---
title: 我的第一篇源码阅读之fetch-event-source
date: 2023-09-16 10:00:00
tags:
  - SSE
categories:
  - 请求
  - 源码学习
cover: 
---

# 前言

终于遇到一个简单的库来学习它的源码了。这个项目只有2个主要文件，代码加起来不到500行，是真的很mini了。
客户端向服务端发起请求用`xhr`或`fetch`，客户端与服务端双向通信用`websocket`，而服务端主动发起请求用`sse`。`chatGPT`就是用`sse`回复提问的。   
`window`中有一个叫`EventSource`的构造函数。一个`EventSource`实例会对服务器开启一个持久化的连接，以`text/event-stream`格式发送事件，此连接会一直保持开启直到通过调用`EventSource.close()`关闭。但使用`EventSource`时只能把参数加到`url`后面，而且也不能像`fetch`请求那样设置`header`等参数。借助`fetch-event-source`这个库就可以像发起`fetch`请求一样发起服务器单向通信请求。

# 目录结构


![1.webp](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/12dd0f1f322e466893e0def92d4d8f1e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

# 入口文件

`index.ts`是入口文件，里面只有2行代码，导出了`fetch.ts`和`parse.ts`中部分变量和方法。

```js
export { fetchEventSource, FetchEventSourceInit, EventStreamContentType } from './fetch';
export { EventSourceMessage } from './parse';
```

`export {...} from 'xx'`其实是`import` + `export`的缩写。、
上面的代码其实就是下面代码的缩写：

```js
import { fetchEventSource, FetchEventSourceInit, EventStreamContentType } from './fetch';
import { EventSourceMessage } from './parse';
export {
	fetchEventSource,
	FetchEventSourceInit,
	EventStreamContentType,
	EventSourceMessage,
}
```

# 发起请求


![1.5.webp](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/569c4d3e662449c594ca9b57531af51e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

1.  首先定义了变量`EventStreamContentType`，它的值是`sse`的`MIME Type`。它在2个地方使用。第一处是发起请求时设置`headers.accept`，告诉服务器只接受\*\*`text/event-stream`\*\*格式的数据。第2处是在连接建立时判断`response.headers.get('content-type')`是否等于`EventStreamContentType`的值，如果不是的话就抛出一个错误，声明期待的类型是`text/event-stream`。
2.  接下来定义了变量`DefaultRetryInterval`，`sse`有**自动重连**机制，这里定义了每次重连的默认间隔为1s。然后定义了变量`LastEventId`，表示上一次事件的id，添加在`headers`中发送到服务端。
3.  接下来定义了一个类型`FetchEventSourceInit`，它声明了`fetchEventSource`的第2个参数的类型。参数一共有7个。  
    `headers` 请求头。  
    `onopen` 连接建立时的回调函数，如果没有设置会调用默认的`defaultOnOpen`，这个默认回调里进行了返回值类型判断。  
    `onmessage`每次收到消息时的回调函数，参数是消息对象，它的类型就是`parse.ts`中定义的`EventSourceMessage`。  
    `onclose` 连接关闭时的回调函数。  
    `onerror`连接发送错误时的回调函数，如果没有指定这个回调或返回`undefined`就会发起重新连接请求。  
    `openWhenHidden` 默认为`false`，监听`visibilitychange`，当页面不可见时关闭连接，当页面重新可见时重新打开连接。  
    `fetch`发起请求的方法，默认为`window.fetch`。  

    `Record<string, string>`等价于`{[key: string]: string}`  
    `Promise<void>`定义了一个异步函数，返回值是`void`  
    `typeof fetch` 获取`fetch`的类型，`typeof`后面跟的是变量，表示类型定义  
4.  接下来就是最重要的`fetchEventSource`，它是一个异步函数，接受2个参数：`url`和`FetchEventSourceInit`类型的对象。  
    在这个方法中，首先定义了接受的媒体类型。然后添加监听`visibilitychange`事件，然后添加监听`abort`事件供使用者可以手动打断连接，然后发起连接，拿到返回值后将返回值传递给`onopen`，然后调用`getBytes`解析返回值，解析之后关闭连接。  
    用`try...catch`包裹发起连接和解析返回值以及关闭连接的过程，如果捕获到错误且不是主动打断的就发起重连。  
    再说一下主动打断连接这里，`fetchEventSource`的第2个参数可以传入一个信号`signal`，这个属性在`FetchEventSourceInit`中没有定义。借助`AbortController`中断连接，具体信息可以看[AbortController-MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/AbortController)。

`??`类似`||`，相同点在于根据前面的值判断返回前面的还是后面的，不同点在于`??`的第一个值为`null`或`undefined`时返回第二个值，`||`会将第一个值先转换为布尔值。比如

    ```
    0 ?? 2 // 0
    0 || 2 // 2
    ```

5\. `defaultOnOpen`定义默认`onopen`回调，主要是检查返回值类型。

# 解析消息


![2.PNG](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/f8f916378b15441497a3b83ca5ad0fd5~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)
首先使用`response.body`来获取响应的主体内容，并通过`getBytes`函数将其转换为字节数组。然后，使用`getLines`函数将字节数组拆分成行，并使用`getMessages`函数将每行解析为事件消息。

## 处理ReadableStream数据

```js
// 创建了一个数据读取器
const reader = response.getReader();
// 创建了一个文本解码器
const decoder = new TextDecoder();

reader.read().then(function processText({ done, value }) {
  // Result 对象包含了两个属性：
  // done  - 当 stream 传完所有数据时则变成 true
  // value - 数据片段。当 done 为 true 时始终为 undefined
  if (done) {
    return;
  }

  // 将字节流转换为字符
  const text = decoder.decode(value)

  // 内容
  console.log(text);

  // 再次调用这个函数以读取更多数据
  return reader.read().then(processText);
});

```

## 处理过程分析

```js
await getBytes(response.body!, getLines(getMessages(id => {
    if (id) {
        // store the id and send it back on the next retry:
        headers[LastEventId] = id;
    } else {
        // don't send the last-event-id header anymore:
        delete headers[LastEventId];
    }
  }, retry => {
    retryInterval = retry;
}, onmessage)));
```

首先执行的是`getBytes`方法，它创建一个读取器，用`while`循环读取流数据，每读取一段就执行`onChunk`解析流数据，`onChunk`就是在`fetch.ts`中`getLines`的返回值。  
`onChunk`将字节块按行分割，并将每行的字节子数组和字段长度传递给`onLine`回调函数。`onLine`则是`getMessages`的返回值。   
`getMessages`创建了一个解码器，返回一个名为onLine的函数，用于处理每个传入的行数据。它将行的字节子数组解码为字符串，并根据字段的类型进行相应的处理。比如，如果字段是`data`，它会将值追加到`message.data`中，如果`message.data`已经存在，则在原有值的基础上添加新值，并使用换行符分隔。   
将字节流先按行分割再解析是为了更好的处理数据，因为数据都是`field:value`格式的。

`TextDecoder`表示一个文本解码器，可以将字节流数据转换成指定码位流，默认是utf-8。

# 问题

在调试这个库的时候，在`html`中引入打包后的`esm`文件会报错文件找不到，因为文件名没有添加后缀。  
接口返回值的类型必须是`text/event-stream`类型的，就算是流数据也不行。

# 总结

`getBytes`的第2个参数是`getLines`的返回值，`getLines`的参数又是`getMessages`的返回值，嵌套的比较深。  
`onChunk`将字节块切割成一行一行的字节，涉及字节数据的知识。
