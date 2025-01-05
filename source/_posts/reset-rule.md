---
title: 📃9条CSS重置样式规则
date: 2024-12-18 10:00:00
tags:
  - css reset
categories:
  - CSS
cover: 
---

# 前言
浏览器的一些默认样式，我们可能不需要。比如，`body {margin: 8px;}`，我们一般希望最外层没有外边距，特别是在有`<header />`或背景图时。再比如，`<img/>`默认展示为原图大小，原图的宽高一般都非常大，我们可能希望图片不要超过父元素。
# 重置的规则
接下来，详细讲一下部分规则，一些显而易见的规则就没必要一一说明了。
## line-height
浏览器的默认行高一般是1.2，但这个距离可能会有点紧凑，而且WCAG 标准规定行高应至少为 1.5。
行高可以继承，所以在`body`上设置就可以了。
```
body {
    line-height: 1.5;
}
```
这个方法有个缺点：如`<h1/>`的行间距太大了。

另一个方法是，以`font-size`为基础，每行添加固定高度。
```
* {
    line-height: calc(1em + 0.5rem);
}
```
这个方法的缺点是：不能设置在`body`上，因为不会为每个元素重新计算`1em`。
## 表单控件的字体继承
`font-size`、`font-family`、`font-weight`在除表单元素以外的控件中是可以继承的。`input`、`textarea`等元素会使用浏览器默认值。
这不合理，默认值太小，而且与父级不统一。所以，我们应该指定这些元素继承父级样式。
```
input, button, textarea, select {
  font: inherit;
}
```
## 改善换行
默认的换行是将超出的单词推送到下一行，有时会产生不好的效果，比如最后一行只有一个句号。
使用`text-wrap`属性可以优化换行的显示。对于`<p>`，使用`pretty`选项将确保最后一行有至少2个文本。对于`<h1>`，使用`balance`选项，将试图使每行的文本长度大致相同。
```
p {
  text-wrap: pretty;
}

h1, h2, h3, h4, h5, h6 {
  text-wrap: balance;
}
```
## 根堆叠上下文
`isolation`属性允许我们创建一个新的堆栈上下文，而无需设置`z-index`。
在使用`Vue`等框架时，如果对根元素设置`isolate`，它允许我们保证某些高优先级元素（模式、下拉菜单、工具提示）始终显示在应用程序中的其他元素之上。

# 完整规则
```
*, *::before, *::after {
  box-sizing: border-box;
}

* {
  margin: 0;
}

body {
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
}

img, picture, video, canvas, svg {
  display: block;
  max-width: 100%;
}

input, button, textarea, select {
  font: inherit;
}

p, h1, h2, h3, h4, h5, h6 {
  overflow-wrap: break-word;
}

p {
  text-wrap: pretty;
}
h1, h2, h3, h4, h5, h6 {
  text-wrap: balance;
}

#root, #__next {
  isolation: isolate;
}
```
不推荐在旧项目上设置这些规则，但推荐在新项目上都设置这些规则。
# 参考
[custom-css-reset](https://www.joshwcomeau.com/css/custom-css-reset/)
[MDN: text-wrap](https://developer.mozilla.org/en-US/docs/Web/CSS/text-wrap)
[理解CSS3 isolation: isolate的表现和作用](https://www.zhangxinxu.com/wordpress/2016/01/understand-css3-isolation-isolate/)