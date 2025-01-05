---
title: 实现自定义脚手架
date: 2024-04-23 10:00:00
tags:
  - vue
  - cli
categories:
  - 指南
cover: 
---
# 为什么要自定义脚手架？

首先，每次创建项目后都需要很多重复性操作，比如安装UI库、css预处理器。在业务层面，每次也要添加一些必备的功能，比如登录、404页面等。其次，可以锻炼自己的技术。

# 脚手架的功能

脚手架创建项目的流程：

1. **让用户选择模板；**

1. **根据用户输入填充模板；**

1. **下载模板文件到创建的新文件夹中；**

从框架来看，创建项目的时候需要选择是vue框架还是react框架，以及是否需要添加 typescript。

从插件来看，需要安装UI库、CSS预处理器、路由、全局状态管理器。

从项目功能来看，还需要封装fetch请求、登录、未登录页面、404页面、部署配置文件。

再参考一下vite，所以我们可以先设置几个模板，然后把小功能填充到模板里面。

# 具体的逻辑

**1.判断远程是否有更新版本，如果有询问用户是否更新。**

当前版本从package.json中获取。

远程版本用`exec()`获取：

```js
exec('npm show xx version', (error, stdout, stderr) => {})
```
**2.询问用户想要的功能**

用`inquirer`库。

询问项目名、框架、是否添加 typescript、是否添加 electron 和 是否添加登录

**3.拉取模板，拷贝模板到当前项目。**

模板就是完整的最基础的项目，如果是必需的功能就直接加进去，如果需要用户选择是否添加则把该功能所在文件改为**ejs**文件。

在拷贝中判断文件是否是ejs文件，如果是则把用户的选择都传进去。

如果已经有同名项目，则在项目名后加（n）。

*注：拷贝的时候模板需要用相对路径获取。*

*可以用ora库添加loading。*

# 可能遇到的问题

**1.npm create命令**

npm create等价于npm init

npm init 是新建package.json文件，但 npm create xx 会先安装一个create-xx的包，然后执行这个包中的create-xx命令

通过npm exec create-xx来执行，npm exec可以执行本地或远程包中的命令。

所以，我们可以**设置package.json中项目名为xx**，发布脚手架后，用户就可以通过**npm create xx**来通过脚手架创建项目。

**2.npm link**

可以通过以下命令在本地测试脚手架。
```js

npm link 将当前目录链接到全局

npm list -g --depth=0 查看全局list

npm unlink -g xx 删除，如果是全局一定要加-g

```
**3.命令无法执行**

bin/xx.js文件开头要加
```js
#!/usr/bin/env node

```
script声明命令时，文件需要添加.js后缀

**4.ejs条件渲染时留下空行**

<%_ ... _%>是不可行的。需要将开始标签紧贴着前一行，结束标签紧贴着可能显示的行。
```js

{
    a: 1,<% if (flag) { %> 
    b: 2,<% } %> 
    c: 3, 
}
```
**5.生成的项目中不包含.ignore文件**

将.ignore重命名为ignore，复制完所有文件后，单独复制ignore文件为.ignore，并删除复制后的ignore
```js
fs.copyFileSync(`${sourceDir}/gitignore`, `${targetDir}/.gitignore`); fs.unlinkSync(`${targetDir}/gitignore`);
```

**6.引用requirer时报错Error [ERR_REQUIRE_ESM]: require() of ES Module ... from ... is not supported**

降版本，可以降到v8.2.0
