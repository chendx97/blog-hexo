---
title: 从0到1实现一个npm库
date: 2023-10-11 10:00:00
tags:
  - npm
categories:
  - 指南
cover: 
---

# 项目的目录结构
首先创建一个空项目，
```bash
npm create vue@lastest
npm i
```
然后把`components`从`src`中挪到根目录下，把`src`改名为`examples`，修改`index.html`中`main.ts`的路径。  
具体的项目结构如下所示：

- components
- examples
- node_modules
- public
- index.html
- package.json
- vite.config.ts
# 导出组件
在`components/index.ts`中定义导出的组件。    
以下代码是导出单个组件，

```js
// components/index.ts
import { App } from 'vue';
import MyButton from './Button';

const MyButtonPlugin = {
  install: (app: App) => {
    app.component('MyButton', MyButton);
  }
}


export default MyButtonPlugin;
```
如果有多个组件需要导出，

```js
// components/Button/index.ts
import { App } from 'vue';
import MyButton from './Button';

const MyButtonPlugin = {
  install: (app: App) => {
    app.component('MyButton', MyButton);
  }
}


export default MyButtonPlugin;
```

```js
// components/index.ts
import MyButton from "./Button";
import MyTable from "./Table";

const components: any = {
  MyButton,
  MyTable,
};

const MyUI = {
  install: (Vue: App) => {
    Object.keys(components).forEach((key) => {
      Vue.use(components[key]);
    });
  },
};

// 导出多个组件，方便按需引入
export { MyButton, MyTable };

// 全部导出
export default MyUI;
```
*`app.use`用来安装插件，插件可以是具有`install`方法的对象，也可以是当作`install`方法的函数。*
# 引入组件
在`examples/main.ts`中使用`app.use`来安装插件，然后就可以全局使用了。
```js
// 整个引入
import MyUI from '../components';

const app = createApp(App);
app.use(MyUI);
```

```js
// 按需引入
import { MyButton } from '../components';

const app = createApp(App);
app.use(MyButton);
```
```js
// App.vue
<my-button />
```
# 打包组件库
首先修改`package.json`文件，去掉`private`字段，添加`main`字段，`files`字段，
```js
"main": "dist/index.umd.js",
"files": [
    "dist",
    "components"
  ],
```
然后修改`vite.config.ts`配置，
```js
build: {
    lib: {
      entry: path.resolve(__dirname, "components"),
      name: "MyUI",
      fileName: (format, fileName) => `${fileName}.${format}.js`,
    },
    rollupOptions: {
      external: ['vue'],
      output: {
        globals: {
          vue: 'Vue',
        },
      },
    },
  },
```
测试打包后的文件:
```js
// main.ts
import test from '../dist/index.es.js';
import '../dist/style.css';

app.use(test);
```
# 发布库
如果没有`npm`账号需要先注册一下。  
如果`npm`源不是默认源，需要修改为默认源。
```bash
npm login
npm publish
```
注：每次`push`的时候都要修改`package.json`中的`version`。