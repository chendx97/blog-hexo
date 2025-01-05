---
title: tailwind使用指南
date: 2024-06-18 10:00:00
tags:
  - tailwind
categories:
  - 指南
cover: 
---
# 前言

tailwind学习曲线比较陡峭，熟悉之后会方便开发，但熟悉之前需要频繁查文档。下面讲一下如何使用tailwind及常用样式，帮助快速上手。我使用的技术栈是**vite+vue**。

# 安装
### 安装
```bash
npm install -D tailwindcss postcss autoprefixer 
npx tailwindcss init -p
```
### 添加配置
```js
// tailwind.config.js 
export default { 
    content: [ "./index.html", "./src/**/*.{vue,js,ts,jsx,tsx}", ], 
    theme: { extend: {}, }, 
    plugins: [], 
}
```
### 引入tailwind样式
```css
// style.css 
@tailwind base; 
@tailwind components; 
@tailwind utilities;
```
在main.ts中导入该文件。

### 使用
```html
<h1 class="text-3xl font-bold underline"> Hello world! </h1>
```
# 常用样式

tailwind*预设*了一部分属性值，比如 w-1 表示 width: 0.25rem; /* 4px */。但新手可以不使用预设值，直接用 [] 设置自定义的值。

<div class="w-1"></div> <div class="w-[4px]"></div>

## 宽高

|               |                           |
| ------------- | ------------------------- |
| class         | 属性                        |
| w-[10px]     | width: 10px;              |
| h-[10px]     | height: 10px;             |
| min-w-[10px] | min-width: 10px;          |
| max-w-[10px] | max-width: 10px;          |
| size-[10px]  | width: 10px;height: 10px; |

## 布局

|                 |                         |
| --------------- | ----------------------- |
| class           | 属性                      |
| block           | display: block;         |
| flex            | display: flex;          |
| hidden          | display: none;          |
| box-border      | box-sizing: border-box; |
| float-left      | float: left;            |
| overflow-hidden | overflow: hidden;       |
| overflow-x-auto | overflow-x: auto;       |
| absolute        | position: absolute;     |
| top-[10px]     | top: 10px;              |

## flex

|                |                          |
| -------------- | ------------------------ |
| class          | 属性                       |
| flex-col       | flex-direction: column;  |
| flex-nowrap    | flex-wrap: nowrap;       |
| justify-center | justify-content: center; |
| items-center   | align-items: center;     |

## margin & padding

|            |                                       |
| ---------- | ------------------------------------- |
| class      | 属性                                    |
| m-[10px]  | margin: 10px;                         |
| mx-[10px] | margin-left: 10px;margin-right: 10px; |
| ml-[10px] | margin-left: 10px;                    |

padding就是把m换成p。**x**代表left和right。**y**代表top和bottom。

## font

|                 |                      |
| --------------- | -------------------- |
| class           | 属性                   |
| text-[16px]    | font-size: 16px;     |
| font-bold       | font-weight: 700;    |
| text-center     | text-align: center;  |
| tracking-[2px] | letter-spacing: 2px; |
| text-[#000]    | color: #000;         |
| line-clamp-2    | 超出2行省略               |
| leading-[30px] | line-height: 30px;   |

## 背景

|                               |                                            |
| ----------------------------- | ------------------------------------------ |
| class                         | 属性                                         |
| bg-[#000]                    | background-color: #000;                    |
| bg-[url(/src/assets/xx.png)] | background-image: url(/src/assets/xx.png); |
| bg-no-repeat                  | background-repeat: no-repeat;              |
| bg-cover                      | background-size: cover;                    |
| bg-center                     | background-position: center;               |

## border

|                |                      |
| -------------- | -------------------- |
| class          | 属性                   |
| border-[2px]  | border-width: 2px;   |
| border-[#000] | border-color: #000;  |
| border-solid   | border-style: solid; |
| rounded-[4px] | border-radius: 4px;  |
| divide-[#000] | 颜色为#000的分割线          |

## 动画

|                      |                                     |
| -------------------- | ----------------------------------- |
| class                | 属性                                  |
| translate-x-[10px]  | transform: translateX(10px);        |
| scale-[1.1]         | transform: scale(1.1);              |
| rotate-[60deg]      | transform: rotate(60deg);           |
| origin-center        | transform-origin: center;           |
| transition-[height] | transition-property: height;        |
| duration-[2000ms]   | transition-duration: 2000ms;        |
| ease-linear          | transition-timing-function: linear; |
| delay-[2000ms]      | transition-delay: 2000ms;           |

animation推荐theme.extend中设置。

## 交互

|                     |                       |
| ------------------- | --------------------- |
| class               | 属性                    |
| cursor-pointer      | cursor: pointer;      |
| pointer-events-none | pointer-events: none; |

# 自定义主题

在tailwind.config.js中可以自定义主题。可以设置的样式包括**screens**、**colors**、**borderRadius**等等。

在theme中设置表示覆盖某属性原来的所有选项。但，如果想保留某属性默认的选项，但又想添加新的选项，可以在theme.extend中配置。

比如，很多地方都需要设置border-radius: 6px，那可以进行如下配置：
```js
module.exports = { 
    theme: { 
        borderRadius: { 
            DEFAULT: '6px', 
            'lg': '12px', 
        }, 
    } 
}
```
使用如下：
```html
<div class="rounded"></div>
<div class="rounded-lg"></div>
```
第一个div的border-radius为6px，第二个div的border-radius为12px。
# 可能遇到的问题

## 动态设置class

利用 **data-\*** 属性
```html
<div data-x="a" class="bg-red data-[x=a]:bg-green"></div>
```
## 特殊属性不支持
如果某些属性，tailwind不支持，可以用 **@layer utilities {}** 设置
```css
@layer utilities { 
    .stroke-progress { 
        -webkit-text-stroke: 0.4rem #010507; 
    } 
}
```
## 使用tailwind同时使用UI组件库
如果舍不得UI组件的某些功能，想和组件库搭配使用

可以用 **@layer components{}** 修改UI库组件的样式
```css
@layer components { 
    .el-dialog {} 
}
```
## elementplus下拉框样式
tailwind+elementplus，给组件的下拉框设置样式时，需要设置 **:teleported="false"** ，@layer才会生效。
