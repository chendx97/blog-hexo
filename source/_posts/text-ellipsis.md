---
title: 🧐文本溢出+完整内容提示
date: 2024-09-25 10:00:00
tags:
  - tooltip
categories:
  - CSS
cover: 
---


# 前言

在开发中，经常会遇到超出省略并显示...的需求。有时，还需要做到在鼠标移入的时候显示具体内容。这些都很简单，但如果要实现“**文本如果超出就在鼠标移入时显示具体内容，文本不超出鼠标移入就不显示**”，就有一点点麻烦。而且，文本长度通常不固定，需要这种处理的文本有多个。所以，在此总结一下如何实现这个功能。

# 文本超出显示...

分为单行文本和多行文本2种情况，代码如下所示。

```css
// 单行文本
.text-single {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

// 多行文本
.text-multi {
  display: -webkit-box;
  overflow: hidden;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
}
```

# 方法一：根据scrollWidth和offsetWidth判断是否显示tooltip

```html
<div v-for="item in info">
    <el-tooltip :visible="nameVisible[item.key]" :content="item.content" placement="top">
      <div @mouseenter="(e: any) => handleMouseEnterName(e, item.key)" @mouseleave="handleMouseLeaveName(item.key)">
        {{ item.content }}
      </div>
    </el-tooltip>
  </div>
  
function handleMouseEnter(e: any, key: string) {
  if (e.target.scrollWidth > e.target.clientWidth) {
    nameVisible.value[key] = true;
  } else {
    nameVisible.value[key] = false;
  }
}

function handleMouseLeave(key: string) {
  nameVisible.value[key] = false;
}
```

# 方法二：借助title属性

```html
<div title="this is title">show title when hover</div>
```

仅设置title属性还达不到想要的效果，而且不能修改title的样式。\
如果要实现想要的效果，具体思路如下：用2份相同的文本A和B，B设置title属性，并且上移N行，父元素设置超出不显示。单行省略时N是2，超出2行省略时N是4。\
原因：假设单行省略，B上移2行后，如果文本只有一行，B移出了父元素范围不显示，鼠标移入A不显示title；如果文本有多行，B覆盖了A，鼠标移入显示title。

![633840727-9c53a45a583a5b50_fix732.webp](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/73dea461a6644cefa6eb83e02723d14c~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5rKz6LGa6byT6byT:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNjM4MTYxNDYwNDAzMjg3In0%3D&rk3s=e9ecf3d6&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1727360824&x-orig-sign=q3n2O7WUPkXjjIBDSx0zrSdXVXA%3D)
```html
<p class="wrap">
  <span class="txt">元素会被移出正常文档流，并不为元素预留</span>
  <span class="title" title="元素会被移出正常文档流，并不为元素预留">元素会被移出正常文档流，并不为元素预留</span>
</p>
.wrap{
    position: relative;
    line-height: 1.5;
    height: 1.5em;
    overflow: hidden;
}
.title{
    display: block;
    white-space: nowrap;
    text-overflow: ellipsis;
    background-color: #fff;
    overflow: hidden;
    position: relative;
    top: -3em;
}
.txt{
  display: block;
    max-height: 3em;
}
```
在此基础上，我们可以通过data-\*属性和伪元素来设置title样式。
```css
/* 隐藏title */
.title[title] {
  text-decoration: none;
}

/* 自定义title */
.title:hover::after {
  content: attr(data-title);
  position: absolute;
  padding: 5px;
  border: 1px solid #000;
  background: #fff;
  color: #000;
}
```
# 最后
参考链接：  
[文本溢出与 Tooltip，如何更好的处理二者](https://github.com/iplaces/blog/issues/3)   
[CSS 文本超出提示效果](https://segmentfault.com/a/1190000040057525#item-1)   
[设置title样式](https://segmentfault.com/q/1010000045092125)
