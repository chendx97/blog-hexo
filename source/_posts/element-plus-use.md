---
title: element-plus自动按需引入
date: 2023-09-13 10:00:00
tags:
  - element-plus
  - 按需引入
  - 自动引入
categories:
  - 指南
cover: https://cdn.jsdelivr.net/gh/chendx97/CPics/img/cover%20(1).png
---

前言：使用`unplugin-vue-components`后，可以实现按需引入，而且也不用再使用`import`引入需要的组件，直接使用`<el-button>`等组件就可以。
## 1.安装npm包

```
npm i unplugin-vue-components unplugin-auto-import -D
```
## 2.配置
`vue.config.js`（这里是vuecli配置方法，其他脚手架需要参考文档）

```
const AutoImport = require('unplugin-auto-import/webpack')
const Components = require('unplugin-vue-components/webpack')
const { ElementPlusResolver } = require('unplugin-vue-components/resolvers')

module.exports = {
	configureWebpack: {
      plugins: [
        AutoImport({
          resolvers: [ElementPlusResolver()],
        }),
        Components({
          resolvers: [ElementPlusResolver()],
        }),
      ],
    }
}
```
## 3.具体使用
`app.vue`

```
<template>
	<div>
		<el-button>按钮</el-button>
	</div>
</template>
```
## 4.确认是否按需引入
首先，安装`webpack-bundle-analyzer`插件，
```
npm install --save-dev webpack-bundle-analyzer
```
然后执行以下命令：
```
npm run build --report
```
不用做任何配置，就可以看到终端打印出来打包后各文件大小。

如果想看详细的打包后文件的信息，可以在`vue.config.js`中做如下配置：
```
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
	configureWebpack: {
    	plugins: [
			new BundleAnalyzerPlugin()
		]
	}
}
```
然后打开地址`http://127.0.0.1:8888`就可以了。
据我个人测试，完整引入时，打包后`chunk.js`文件大小为`400k`左右。按需引入后，打包后`chunk.js`文件大小为`200k`左右。
## 4.非`<template>`中使用组件
如果在ts文件中使用`ELMessage`等组件，仍需要手动引入，代码如下。
```
import { ElMessage } from 'element-plus'
import 'element-plus/theme-chalk/el-message.css'
```
----------

参考文章：   
[element-plus](https://element-plus.gitee.io/zh-CN/guide/quickstart.html#%E6%8C%89%E9%9C%80%E5%AF%BC%E5%85%A5)   
[unplugin-vue-components](https://github.com/antfu/unplugin-vue-components)