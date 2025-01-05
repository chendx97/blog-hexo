---
title: ✨如何判断接口是否在设定时间内返回
date: 2024-07-02 10:00:00
tags:
  - axios
  - fetch
  - timeout
categories:
  - 请求
cover: 
---
# 前言

某些情况下，我们可能想要知道接口是否在设定时间内返回。比如，请求发出30s还没返回就可以提示用户当前堵塞是否继续排队。比如，某个请求正常情况下3s返回结果，如果等30s没结果就可以丢弃这次结果。比如，超过设定时间后自动重试。

# axios设置timeout
```js
import axios from 'axios'; 
// 设置超时时间为10秒 
const timeout = 10000; 
// 发送请求 
axios.get('https://api.example.com/data', { timeout: timeout }) 
.then(response => { 
    console.log('请求成功:', response.data); 
}) 
.catch(error => { 
    if (axios.isCancel(error)) { 
        console.log('请求取消:', error.message); 
    } else if (error.code === 'ECONNABORTED' && error.message.includes('timeout')) {
        console.log('请求超时'); 
    } else { 
        console.log('请求失败:', error); 
    } 
});
```
# vueuse的useFetch+timeout
```js
const { data } = useFetch(url, { timeout: 1000 })
```
如果只是在到达设定时间后进行一些处理，不丢弃超时结果，则如下所示。
```js
const { canAbort } = useFetch(url); 
setTimeout(() => { 
    if (canAbort.value) { 
        // ... 
    } 
}, 1000)
```
# fetch+AbortController
```js
// 创建一个AbortController实例 
const controller = new AbortController(); 
// 获取abort信号 
const signal = controller.signal; 
// 发送fetch请求，并将signal传递给fetch选项 
fetch(url, { signal }) 
.then(response => { 
    if (!response.ok) { 
        throw new Error('Network response was not ok'); 
    } 
    return response.json(); 
}) 
.then(data => { 
    console.log('请求成功:', data); 
}) 
.catch(error => { 
    if (error.name === 'AbortError') { 
        console.log('请求被取消'); 
    } else { 
        console.error('请求失败:', error); 
    } 
}); 
setTimeout(() => { 
    controller.abort(); 
}, 3000);
```
🍉 AbortController不支持IE。

# Promise.race + setTimeout

🍇 先复习一下Promise.race，接收一个包含多个promise对象的数组作为参数，*只要有一个*promise改变状态，Promise.race就立刻返回。
```js
const timerPromise = new Promise((resolve) => { 
    setTimeout(() => { 
        console.log("timer1"); 
        resolve("timer end"); // 这里会执行，但Promise.race().then接收不到 
    }, 10000); 
}); 
const fetchPromise = () => { 
    return fetch().then(response => response.json())
    .then(res => { 
        if (res.code !== 200) { 
            // 接口错误处理 
            return new Promise(() => { }); 
        } 
        return res; 
     }); 
 }) 
 Promise.race([timerPromise, fetchPromise]).then((res) => { 
     console.log(res); 
     if (res === 'timeout') { 
         // 处理超时后逻辑 
         return new Promise(() => { }); 
     } 
     return res; 
});
```
打印结果：  
start fetch  
get fetch res   
timer1

另外，需要注意，fetchPromise.then是先于Promise.race().then执行的。

这种方案可以实现在超时的时候进行一些操作，而不一定抛弃请求结果。比如下面的代码就可以在超时之后进行一些操作。
```js
const timerPromise = new Promise((resolve) => { 
    setTimeout(() => { 
        console.log("timer1"); 
        resolve("timer end"); 
    }, 10000); 
}); 
const fetchPromise = () => { 
    return fetch().then(response => response.json()).then(res => { 
        if (res.code !== 200) { 
            return new Promise(() => { }); 
        } 
        // !!! 在这里进行一些操作，就算先计时结束，这里也会执行。比如设置一些全局变量等。                     console.log('after timeout'); 
        return res; 
    }); 
}) 
Promise.race([timerPromise, fetchPromise]).then((res) => { 
    console.log(res); 
    if (res === 'timeout') { 
        // 处理超时后逻辑 
        return new Promise(() => { }); 
    } 
    return res; 
});
```