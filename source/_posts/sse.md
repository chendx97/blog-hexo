---
title: 服务端主动推送消息Server-client events(SSE)
date: 2023-09-13 10:00:00
tags:
  - SSE
categories:
  - 请求
cover: 
---

# 前言
众所周知，用http可以实现从客户端请求服务端的消息，用websocket可以实现客户端和服务器的双向通信，但用SSE实现服务端主动推送的比较少。   
SSE相当于http的优点是可以由服务端主动推送消息。SSE相当于websocket的优点是断开连接后可以自动重连。

# 使用场景
* 消息统一推送
* 数据实时更新
* 代替轮询

# 使用方法
node的话，可以用阮老师的例子[node实现SSE](https://www.ruanyifeng.com/blog/2017/05/server-sent_events.html)

python可以参考下面的代码

```
from flask import Flask, Response
import time
import json

app = Flask(__name__)

def message_to_send():
    time.sleep(5)
    return json.dumps({"message": "hello"})


@app.route("/stream")
def stream():
    def event_stream():
        while True:
            print("send")
            yield "data:  {}\n\n".format(message_to_send())

    return Response(
        event_stream(),
        mimetype="text/event-stream",
        headers={"Access-Control-Allow-Origin": "*"},
    )


if __name__ == "__main__":
    app.run()

```
# 传递post参数
默认情况下，`sse`只能进行`get`请求，参数跟在url后面。借助`@microsoft/fetch-event-source`可以传递post参数。
```js
fetchEventSource('/api/sse', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
    },
    body: JSON.stringify({
        foo: 'bar'
    }),
});
```

# 注意事项
* 想要跨域需要设置`withCredentials: true`或服务端设置允许跨域。（`@microsoft/fetch-event-source`可以不用设置）
* 接口必须设置`mimetype`为`text/event-stream`
* 接口返回的每条消息必须以`\n`结尾，最后一条消息以`\n\n`结尾
* 有连接个数限制，每个域名下好像是限制6个
* 如果想测试自动重连，可以先关闭服务端的服务再打开