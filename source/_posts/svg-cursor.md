---
title: 给svg设置cursor不生效
date: 2024-02-26 10:00:00
tags:
  - svg
  - cursor
categories:
  - bug
cover: 
---

# 前言
项目中一般用 svg 做小图标或背景图，背景图不需要设置 cursor， 小图标会写成单个的 IconXx.vue 文件。前不久遇到一个需求，需要展示 svg，并且可点击，这就需要设置 cursor，但直接设置不会生效。
# svg 插入页面的方法
SVG 文件可通过以下标签嵌入 HTML 文档：`<embed>`、`<object>`、`<iframe>`。  
SVG的代码可以直接嵌入到HTML页面中，或您可以直接链接到SVG文件。

```html
<embed src="circle1.svg" type="image/svg+xml" />

<object data="circle1.svg" type="image/svg+xml"></object>

<iframe src="circle1.svg"></iframe>

<svg xmlns="http://www.w3.org/2000/svg" version="1.1">
   <circle cx="100" cy="50" r="40" stroke="black" stroke-width="2" fill="red" />
</svg>

<a href="circle1.svg">查看 SVG 文件</a>
```
# 在 svg 代码中设置 cursor
这样不但可以设置整个 svg 的光标，而且可以设置里面单个元素的光标。   
缺点是需要能修改 svg 代码。
```html
<svg ...>
  <style>
    svg { cursor: pointer; }

    /* OR */

    .element-name { cursor: pointer; }
  </style>
  ...
</svg>
```
# 包裹一下 svg
在 svg 的外层添加一层 div，给这个 div 设置光标。

```html
<div class="svg-wrapper">
    <object>
        <embed src="img.svg"></embed>
    </object>
</div>
```
```css
.svg-wrapper {
    cursor: pointer;
}
```