---
title: CSS变量和CSS函数
date: 2024-09-23 10:00:00
tags:
  - css
categories:
  - CSS
cover: https://cdn.jsdelivr.net/gh/chendx97/CPics/img/cover%20(2).png
---

# 前言
CSS变量和CSS函数可以帮助开发者方便地实现功能。
# CSS变量
```css
:root {
  --main-bg-color: brown;
}

element {
  background-color: var(--main-bg-color);
}
```
CSS变量名以--开头；  
变量可以继承父元素的值；   
var第2个参数为备用值，变量不生效时被使用；
# CSS函数
常用的css函数包括：calc()、linear-gradient()、translate(), rotate(), scale()、rgba()、hsla()、attr()、clamp()、min()、max()。  
## calc()执行计算
```css
.container {
  width: calc(100% - 30px);
}
```
## linear-gradient()创建一个线性渐变
```css
.background {
  background-image: linear-gradient(to right, red, yellow);
}
```
## transform函数改变元素的形状、位置和尺寸
```css
.moved {
  transform: translate(50px, 100px);
}
```
## rgba()、hsla()定义颜色
rgba(红，绿，蓝，透明度)  
hsla(色调，饱和度，亮度，透明度)  
```css
.text {
  color: rgba(255, 99, 71, 0.5);
}
```
## attr()引用元素的属性值
```css
.box::before {
  content: attr(data-tooltip);
}
```
## clamp()限制一个值在一个特定范围内
它接受三个参数：最小值、优选值和最大值。
```css
h1 {
  font-size: clamp(2rem, 4vw + 1rem, 8rem);
}
 ```
这个函数的工作方式是：如果优选值小于最小值，那么返回最小值；如果优选值大于最大值，那么返回最大值；如果优选值在最小值和最大值之间，那么返回优选值。   
例如，font-size: clamp(12px, 5vw, 24px); 这个 CSS 规则会设置字体大小为视口宽度的 5%，但是不会小于 12px，也不会大于 24px。这在响应式设计中非常有用，可以确保元素的大小在不同的设备和屏幕尺寸上都能保持合适的比例，同时也不会过小或过大。   
## min()、max()返回一组数值中最小或最大一个值
```css
.container {
  width: min(100%, 600px);
}
```
参数的类型：   
\<length>：长度值，如 px, em, rem, vw, vh 等。   
\<number>：数字值。   
\<percentage>：百分比。   
\<calc()>：另一个 calc() 函数的表达式。
