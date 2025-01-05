---
title: 文本片段：比Ctrl+F更智能
date: 2024-12-20 10:00:00
tags:
  - 文本片段
categories:
  - HTML
cover: 
---

# 概念
以前，你可以通过`url`跳转到其他页面顶部，也可以通过`id`跳转到某个DOM节点。**文本片段**允许你直接链接到网页中的特定文本部分，而且可以高亮链接文本部分。它可以用来生成更有效的内容共享链接，让用户互相传递。
# 语法
```
https://example.com#:~:text=[prefix-,]textStart[,textEnd][,-suffix]
```
接下来是每部分的详细说明：
**#**
`#`是必须的，它是url片段标识符的一部分，用于指示浏览器跳转到页面中的特定位置或文本。
**:~:**
片段指令，告诉浏览器接下来是一个或多个指令，跳转后这些指令会从url中剥离。
**text=**
代表接下来是一段文本指令。
**textStart**
一个文本字符串，指定链接文本的开始。
**textEnd**
一个文本字符串，指定链接文本的结束。
**prefix-**
一个文本字符串+一个连字符，指定链接文本前面应该有什么文本，有助于更精确的选择链接文本。
**-suffix**
一个连字符+一个文本字符串，指定链接文本后面应该有什么文本。

# 使用说明
- 只高亮第一个匹配到的链接文本。
- `textStart`、`textEnd`、`prefix`、`suffix`需要做百分比编码(encodeURIComponent)。
- 匹配大小写不敏感。
- 单独的`textStart`、`textEnd`、`prefix`、`suffix`字符串需要完全位于同一个块级元素中，完整的匹配可以跨越多个边界。
- 出于安全考虑，如果是`<a>`需要添加`rel="noopener"`，如果是`window.open()`需要添加`noopener`。
- 文本片段只在非同一页面、用户发起的导航中被调用。
- 文本片段只适用于主框架，不支持`iframe`。
- 如果不想支持文本片段，可以设置`Document-Policy: force-load-at-top`。
- 默认跳转到第一个匹配到的链接文本，如果没匹配到或不支持，跳转到文档顶部。
- 如果匹配到`<details>`内的内容，它会自动打开。

# 示例
```html
<!--第一次出现“for”的地方-->
https://example.com#:~:text=for

<!--第一次以“人类”开头而且以“URL”结尾的地方-->
https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/a#:~:text=人类,URL 

<!--第一次出现“for”而且前面是“asking”的地方-->
https://example.com#:~:text=asking-,for

<!--
    第一个高亮是第一次出现“导致”的地方，
    第二个高亮是第一次出现“链接的”的地方，
    并且跳转到第一个高亮的地方。
-->
https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/a#:~:text=导致&text=链接的

<!--
    第一个高亮是第一次出现“链接的 URL”而且后面紧跟“，”的地方，
    第二个高亮是以“页面”开头而且以“浏览环境”结尾，而且前面是“当前”的地方，
    并且跳转到第一个高亮的地方。
-->
https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/a#:~:text=链接的%20URL,-，&text=当前-,页面,浏览环境
```

# 给文本片段添加样式
```css
::target-text {
  background-color: rebeccapurple;
  color: white;
}
```
支持的属性：
- `color`
- `background-color`
- `text-decoration`、`text-underline-position`、`text-underline-offset`
- `text-shadow`
- `stroke-color`、`fill-color`、`stroke-width`
- 自定义属性

# 检测浏览器是否支持文本片段
```js
document.fragmentDirective;
```
如果是空对象代表支持，如果是undefined代表不支持。
目前仍是空对象，未来可能包含片段信息。

# 缺点
只能跳转并高亮第一个匹配到的地方，不能跳转并高亮以后匹配到的地方。如果尝试按`Tab`键，焦点会移至下一个可聚焦元素(如链接)。

# 参考链接
https://alfy.blog/2024/10/19/linking-directly-to-web-page-content.html
https://developer.mozilla.org/zh-CN/docs/Web/URI/Fragment/Text_fragments