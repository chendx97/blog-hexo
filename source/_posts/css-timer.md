---
title: 【翻译】如何实现一个纯CSS计时器
date: 2024-06-17 10:00:00
tags:
  - timer
categories:
  - CSS
cover: 
---

# 前言

有时我们会在页面上使用计时器。通常我们会用js实现计时器功能，但某些时候更适合用css，比如某些邮件客户端不支持运行js。

# 4个属性

*   counter()：一个css函数，返回一个代表计数器的当前值的字符串。它通常和伪元素搭配使用，但是理论上可以在支持值的任何地方使用。

```css
    counter(计数器名称);
```
*   @property：自定义css属性，兼容性不是特别好，值的类型可以是*integer*、*number*、*color*、*url*、*percentage*、*transform-function*等。

```css

    @property --property-name { 
        syntax: "<color>"; // 值类型检测 
        inherits: false; // 是否允许继承 
        initial-value: #c0ffee; // 默认值 
    }
```
![caniuse](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/9dfff96036164eb2abe5299c708b6184~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

*   伪元素
*   @keyframes

# 3个要求

实现的效果如下图所示，鼠标移动到框内开始倒计时。
![效果](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/3521f2df17864273846772fb9422bc15~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

实现这个计时器需要做到以下3点：

*   可以从5递减到0的数字；
*   一种计时5秒并递减每个秒数的方法；
*   一种在页面上显示递减数字的方法；

第一点，我们可以用 **@property** 创建一个自定义属性。

    @property --n { 
       syntax: "<integer>"; 
       inherits: false; 
       initial-value: 0; 
    }

注意，是整数类型，包括0、正整数、负整数。如果希望是带小数点的数字可以用*number*。

第二点，我们可以用 **@keyframes** 实现。
```css
    @keyframes count { 
        from { --n: 5; } 
        to { --n: 0; } 
    } 
    .timer:hover::after { 
        animation: 5s linear count; 
    }
```
第三点，我们可以用 **counter-reset** 实现。
```css
    .timer:hover::after { 
        animation: 5s linear count; 
        animation-fill-mode: forwards; 
        counter-reset: n calc(0 + var(--n)); 
        content: counter(n); 
    }
```
content不接受数字类型的值，所以需要借助counter()。\
前面提过，counter()，接收计数器名字做参数，返回计数器当前值。

使用计数器前必须使用counter-reset初始化计数器。
counter-reset：第一个参数必选，计数器名；第二个参数，计数器初始值，默认是0。
# 源码
完整代码如下：
```css
@property --n {
  syntax: "<integer>";
  inherits: false;
  initial-value: 0;
}
@keyframes count {
  from { --n: 5; }
  to { --n: 0; }
}
.timer:hover::after {
  animation: 5s linear count;
  animation-fill-mode: forwards;
  counter-reset: n calc(0 + var(--n));
  content: counter(n);
}
.timer::after{
  display: block;
  content: '';
  line-height: 200px;
  height: 1lh;
  aspect-ratio: 1 / 1;
  font-size: 92pt;
  text-align: center;
  border: 1px dashed black;
}
```
# 进阶

借助这些知识，可以设计更复杂的计数器，比如向上计数，美化ui，与其他loading（比如圆形进度条等）结合。

[演示1](https://codepen.io/rpsthecoder/pen/XWwdLPK)
![1](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/2f2cdbb17e124c78bef09c91a18d5ba1~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

[演示2](https://codepen.io/rpsthecoder/pen/ExzxrWX)
![2](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/fc09a13466944f2b8b931ccf75585dcd~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)

另外，css自定义属性也可以在js中创建和修改，创建用 **registerProperty(**) ，修改用 **setProperty()** 。如果想知道动画是否完成，可以用 *animationend* 监听。

# 链接

[原文](https://frontendmasters.com/blog/how-to-make-a-css-timer)

[MDN：使用css计数器](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_counter_styles/Using_CSS_counters)
