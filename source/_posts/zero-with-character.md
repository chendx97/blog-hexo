---
title: 一个看起来只有2个字长度却有8的字符串引起的bug
date: 2023-12-14 10:00:00
tags:
  - 零宽字符
categories:
  - HTML
  - bug
cover: 
---

# 前言
我们有一个需求，用户的昵称如果长度超过6就截取前6个字符并显示...。今天，测试突然提了一个bug，某个用户的昵称只显示了...，鼠标hover的时候又显示2个字的昵称。刚看到这个问题的时候我也是一头雾水。
# 找出原因

![image.png](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/5c4b9b1adfc34b1987f357a1410f6ed6~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)  
在看到这个现象后，我发现其他昵称都显示正常，但实在摸不着头脑这到底是怎么回事。然后查看了一下其他2个字的昵称是没问题的，然后通过`console.log`发现这个昵称居然长度有8，走了截取的分支。然后通过google发现这里面应该包含了零宽字符。   
其实，第一时间就应该想到这个字符串不对劲的，但完全忘记了`零宽字符`的存在，走了不少弯路。    

在查找的过程中发现，`Array.from`可以查看字符串的真实长度，除了`emoji`。

![image.png](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/d2c30abafa5a48398261240a4424145e~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)   
不过`Array.from`并不能解决我的问题。
# 使用正则匹配unicode码点过滤零宽字符
在网上找了个方法来过滤掉这些看不见的字符，最常见的解决方案就是下面这行代码。

```js
str.replace(/[\u200b-\u200f\uFEFF\u202a-\u202e]/g, "");
```
然而并没有用，我开始怀疑是不是这个方法有问题，然后遍历了这个昵称，把它的每个字符都转换成码点，发现这个昵称里的零宽字符并不是常见的这几种。      

后来，又找到了一个比较完善的码点正则，但它太完善了，很长很长，也会过滤掉`emoji`，这可不行，用户昵称可能会包含`emoji`的。（这里就不贴出来代码了，太长了而且不适合我的情况。）
# 使用正则匹配unicode类别
一个字符有多种unicode属性，而正则支持按unicode属性匹配。

```js
function stripNonPrintableAndNormalize(text, stripSurrogatesAndFormats) {
    // strip control chars. optionally, keep surrogates and formats
    if(stripSurrogatesAndFormats) {
      text = text.replace(/\p{C}/gu, '');
    } else {
      text = text.replace(/\p{Cc}/gu, '');
      text = text.replace(/\p{Co}/gu, '');
      text = text.replace(/\p{Cn}/gu, '');
    }

    // other common tasks are to normalize newlines and other whitespace

    // normalize newline
    text = text.replace(/\n\r/g, '\n');
    text = text.replace(/\p{Zl}/gu, '\n');
    text = text.replace(/\p{Zp}/gu, '\n');

    // normalize space
    text = text.replace(/\p{Zs}/gu, ' ');

    return text;
}
console.log("⁮⁡⁪⁡⁠⁮河豚".length);
console.log(stripNonPrintableAndNormalize("⁮⁡⁪⁡⁠⁮河豚", true).length);
```

![image.png](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/333d342f3653422d8aafe3c1cf9b8116~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp)
# 总结
这个昵称其实就是包含了`&nobreak;`，通过unicode类别匹配可以过滤掉它。 

我之前有在原贴用户主页的控制台中看见了`&nobreak;`，但当时居然没当回事，以为是别人对昵称做的处理。如果直接搜它马上就能解决问题了，有不少人遇到`non-break-space`引发的bug。谨以此记，吸取教训。  

参考链接：[stackoverflow中的解决办法](https://stackoverflow.com/questions/11598786/how-to-replace-non-printable-unicode-characters-javascript)、[unicode属性](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Regular_expressions/Unicode_character_class_escape)