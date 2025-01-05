---
title: canvas之measureText测量文本
date: 2023-10-17 10:00:00
tags:
  - canvas
categories:
  - HTML
  - CSS
cover: 
---

# 前言
这篇文章讲的是关于`canvas`的一个方法：`measureText`，这个方法接受一个字符串参数，返回关于这个字符串的宽度和水平线到顶部或底部的距离等等。主要涉及的知识点有：获取文本宽度、文本在垂直方向的对齐方式、基线、行高。
# 获取字符串宽度
```js
<canvas id="canvas" width="200" height="200"></canvas>
<script>
    const canvas = document.getElementById('canvas');
    const context = canvas.getContext('2d');
    context.font = '16px Microsoft YaHei';
    console.log(context.measureText('我你他'));
</script>
```
打印出来是一个对象，其中的`width`属性就是传入的字符串的宽度。有2点需要注意一下，第一点是需要设置`canvas`中的文字`font`属性与需要测量的字符串的值一致。第2点是`width`值有一点点误差，我只测试了在`Chrome`中的情况，误差如下。
- 中文，`width`是整数，无误差。
- 英文，测量值略小于实际值0.01左右，实际值=测量值的四舍五入保留2位小数再向上取整保留2位小数。
- 数字，测量值略小于实际值0.01左右，实际值=测量值的四舍五入保留2位小数再向上取整保留2位小数。
比如测量值是85.7109，实际值是85.72。测量值是62.1796875，实际值是62.19。

# 基线相关的基本概念
`measureText`的返回值是`TextMetrics`对象，它的属性除了`width`，还有一些是与基线、顶线相关的值，下面先了解一下相关概念。借用一下别人的图片。

![line.jfif](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/71bf2577b6bd43ad84d11966bd6ac5a1~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)
- 基线，x的下沿
- 内容区，从顶线到底线的区域
- 行高，内容区+上下空白区域的高度，等于相邻行的基线之间的距离
- 行距，从上一行基线到下一行顶线的距离
- 行内框，内容区+上下空白区域，它的高度就是`line-height`指定的高度
- 行框，一行内多个字符串的行内框的最大值  

接下来了解一下`TextMetrics`对象其他属性的含义。
`ctx`的`textBaseline`指定文字在垂直方向的对齐方式，文本基线的位置，之前那个基线(x的下沿)是标准的字母基线。
- fontBoundingBoxAscent，从文本基线到行框顶部的距离
- fontBoundingBoxDescent，从文本基线到行框底部的距离
- actualBoundingBoxAscent，从文本基线到顶线的距离
- actualBoundingBoxDescent，从文本基线到底线的距离
- actualBoundingBoxLeft，从水平对齐方式的对齐点到行框最左边的距离
- actualBoundingBoxRight，从水平对齐方式的对齐点到行框最右边的距离

总结，前4个属性暂不清楚有什么使用场景，后面2个属性可以用来计算文本宽度，`MDN`推荐用这2个值相加来获取倾斜字符串的绝对宽度。如果不是斜体，我推荐用`width`。
# 了解测量文本可以解决什么问题
### 垂直居中
最常见的就是一行中图片和文字如何垂直居中。
默认是基于基线对齐的，设置为基于中线就可以做到垂直居中了。

```js
.img {
    width: 30px;
    vertical-align: center;
}
.name {
    vertical-align: center;
}
```
### 图片下方有空白区域
这段空白就是底线到行框的距离，行框的大小又取决于行高，所以设置`line-height`或`font-size`为0即可。
# measureText的应用场景
### 动态调整文字大小

```js
const getFontSize = (str, parentWidth) => {
  const canvas = document.createElement('canvas');
  const context = canvas.getContext('2d');
  context.font = '16px Microsoft YaHei';
  const { width } = context.measureText(str);
  if (parentWidth - width < 0) {
    return 14;
  }
  return 16;
}
```
比如用户拖拽某个卡片大小时，或某个容器的字符串不固定长度时就可以这样来调整`font-size`。
### 绝对定位的不定长文本水平居中

```js
const getTextPos = (str, parentWidth) => {
  const canvas = document.createElement('canvas');
  const context = canvas.getContext('2d');
  context.font = '16px Microsoft YaHei';
  const { width } = context.measureText(str);
  return (parentWidth - width) / 2;
}
textDom.style.left = getTextPos('abcdefg中文', parentDom.width) + 'px';
```
正常情况下，就算是不定长文本也可以设置`text-align: center`来水平居中。但，如果这段不定长文本是绝对定位的就需要计算一下应该设置的位置。
### 判断字符串会不会容器范围

```js
const isBeyond = (str, parentWidth) => {
  const canvas = document.createElement('canvas');
  const context = canvas.getContext('2d');
  context.font = '16px Microsoft YaHei';
  const { width } = context.measureText(str);
  return width - parentWidth;
}
```
# 总结
需要获取文本宽度的情况可以考虑这个方法。