---
title: 第一次开发node接口-干货总结
date: 2023-10-17 10:00:00
tags:
  - node
categories:
  - 指南
cover: 
---

技术栈：`node` + `express` + `mongoose`  
[git地址](https://github.com/chendx97/node-template)，可直接使用。
# nodemon 热更新
使用`nodemon`启动项目，可以在修改代码时不用重新启动。
```bash
// 安装
npm i nodemon
```
安装之后，修改启动命令，
```js
// package.json
{
    "scripts": {
        "start": "nodemon index.js"
    }
}
```
# 中间件
## body-parser
`body-parser`解析请求参数，使我们可以通过`req.body.xx`访问参数。
```js
// index.js
var bodyParser = require('body-parser');
app.use(bodyParser.json());
```
## cookie-parser
`cookie-parser`解析请求携带的`cookie`，使我们可以通过`req.cookies`访问`cookie`。
```js
const email = JSON.parse(req.cookies[config.cookieName]).email;
```
```js
// index.js
var cookieParser = require('cookie-parser');
app.use(cookieParser());
```
# 路由中间件
借助路由中间件，我们可以格式化接口返回值。
```js
// routers/index.js
var express = require('express');
var router = express.Router();

router.use((req, res, next) => {
  res.jsonSuccess = (data) => {
    res.json({
      code: 200,
      message: 'success',
      result: data,
    });
  };
  res.jsonFail = (message) => {
    res.json({
      code: 500,
      message: message.message || '服务器出了点问题',
    });
  };
  next();
});

module.exports = router;
```
# mongoose指令
## 查
`find`查找所有符合条件的数据；  
`findOne`查找第一个符合条件的数据；   
`findById`查找符合条件的数据，查询条件为`id`；  

`Model.find(conditions, [projection], [options], [callback])`
```js
model.find(
    {age:{$gte:18,$lte:80}}, // age字段大于等于18且小于等于80 的数据
    {_id:0,__v:0}, // 数据文档不返回_id和__v字段
    {sort:{age:-1}} // 根据age字段以大到小顺序返回数据文档
)
```
`$gte`是操作符，类似的操作符还有不少：   
- `$eq` 相等  
- `$ne` 不相等   
- `$gt` 大于
- `$gte` 大于等于
- `$lt` 小于
- `$lte` 小于等于
- `$in` 与数组中任意一个匹配
- `$nin` 与数组中每一个都不匹配
- `$and` 满足数组中所有条件
- `$nor` 所有条件都不满足
- `$or` 满足数组的其中一个条件
- `$not` 不满足条件
- `$exists` 存在指定字段
- `$type` 属于指定类型
- `$all` 匹配包含查询数组所有条件的数组字段
- `$elemMatch` 匹配数组字段中的某个值满足指定的所有条件
- `$size` 匹配数组字段的`length` 与查询条件相同   Same as querySame as query conditions

`projection`指定查询结果包含或不包含哪些字段，也可以写成字符串：`"-_id -__v"`，不能包含不包含同时使用。  
`findOne`和`findById`的区别在于`id`为`undefined`的情况，`findOne({ _id: undefined })`返回任意一条数据，`findById(undefined)`返回`null`。
## 增
`save([options], [options.safe], [options.validateBeforeSave], [fn])`增加一条，需要写实例化；   
`Model.create(doc(s), [callback])`增加一条或多条；   
`Model.insertMany(doc(s), [options], [callback])`增加多条；   

`save()`的示例如下：
```js
// models/user.js
var mongoose = require('./index');

var UserSchema = mongoose.Schema({
  email: {
    type: String,
    unique: true,
  },
});

var UserModel = mongoose.model('User', UserSchema);

module.exports = UserModel;
```
```js
// routers/user.js
var router = require('./index.js');
var User = require('../models/user');

// 新增项目
router.post('/user/add', async function (req, res) {
  try {
    const user = new User(req.body);
    await user.save(); // save
    res.jsonSuccess();
  } catch (error) {
    res.jsonFail(error);
  }
});

module.exports = router;
```
## 删
`findOneAndDelete`找到第一个符合条件的并删除；   
`findByIdAndDelete`查询条件为`id`；   
`deleteMany`删除所有符合条件的；   

`Model.findOneAndDelete(filter, options, callback)`

## 改
`findOneAndUpdate`和`findByIdAndUpdate`都是先查找然后再修改；  
`updateOne`也是查找符合条件的第一条数据并修改；   
`updateMany`是批量修改数据；  

`Model.findOneAndUpdate(filter, update, [options], [callback])`

举个例子：
```js
const info = await User.findByIdAndUpdate(
    {
        _id: req.body.id
    },
    {
        $set: {
            pass: true
        }
    },
    {}
)
```
# 解析excel文件
使用`multer`解析form-data数据，[npm地址](https://www.npmjs.com/package/multer)。  
使用`node-xlsx`解析excel，[npm地址](https://www.npmjs.com/package/node-xlsx)。
```js
const multer = require('multer');
const upload = multer();
const xlsx = require('node-xlsx');

router.post('/upload', upload.single('file'), async function (req, res) {
    const sheets = xlsx.parse(path.resolve(__dirname, '../test.xlsx'));
    const data = sheets[0].data; // 二维数组
});
```
使用`req.file.mimetype`可以获取文件的`MIME type`，可以以此来判断文件类型，`excel`文件的`MIME type`是`application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`或`application/vnd.ms-excel`。
# 下载文件接口
使用`sendFile`可以直接返回文件类型数据。
```js
router.get('/download', async function (req, res) {
  try {
    const filePath = path.resolve(__dirname, '../test.xlsx');
    res.sendFile(filePath);
  } catch (error) {
    res.jsonFail(error);
  }
});
```
```js
const handleDownload = async () => {
  const response = await fetch('/api/download');
  const blob = await response.blob();
  saveAs(blob, 'test.xlsx');
};
```
# docker部署配置
```bash
# Dockefile文件

# 使用Node.js的官方镜像作为基础 
FROM node:14 

# 在容器中创建一个工作目录 
WORKDIR /app 

# 复制应用程序文件到容器中 
COPY package*.json ./ 

# 安装依赖项 
RUN npm install 

# 复制应用程序代码到容器中 
COPY . . 

# 暴露容器的端口 
EXPOSE 3000 

# 定义容器启动时运行的命令 
CMD ["npm", "start"]
```