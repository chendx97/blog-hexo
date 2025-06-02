---
title: 知识点总结（一）
date: 2025-06-02 14:00:00
tags:
categories:
  - 总结
cover: https://cdn.jsdelivr.net/gh/chendx97/CPics/img/202506021436473.png
---


好记性不如烂笔头。把遇到的不常见的知识点都记录下来，避免下次再遇到时还不理解。总结一下也能加深自己的印象。

## clearTimeout
**执行clearTimeout(timer)后，定时器回调不会执行，但timer的值不变。**

**防抖**：事件触发后等待指定时间，若期间没有再次触发，则执行函数；若期间重复触发，则重新计时。
```js
const debounce = (func, wait) => {
  let timeout;
  return (...args) => {
    clearTimeout(timeout);
    timeout = setTimeout(() => {
      func.apply(null, args);
    }, wait);
  };
};
```
**节流**：在指定时间间隔内，无论触发多少次，函数只执行一次。
```js
const throttle = (func, wait) => {
  let lastTime = 0;
  let timeout;
  return (...args) => {
    const now = Date.now();
    const remaining = wait - (now - lastTime);
    if (remaining <= 0) {
      func.apply(null, args);
      lastTime = now;
    } else if (!timeout) {
      timeout = setTimeout(() => {
        func.apply(null, args);
        lastTime = Date.now();
        timeout = null;
      }, remaining);
    }
  };
};
```
## line-break
`line-break`属性控制标点符号、单词、连续字符在行尾的换行方式，尤其影响：
- 标点（如句号、逗号）能否出现在行首/行尾
- 连续字母或数字（如长单词、URL）是否允许断开
- 中文/日文文本的换行规则

```css
.element {
  line-break: auto | loose | normal | strict | anywhere;
}
```

**1. `auto` (默认值)**
行为：浏览器根据文本语言自动选择换行规则
示例：中文文本默认允许标点出现在行首，英文则更倾向于避免断开单词

**2. `loose`**
行为：宽松换行，允许更多换行机会
适用场景：短行文本（如报纸排版）
特点：
允许标点出现在行首
更频繁断开长单词或连续字符

**3. `normal`**
行为：使用标准换行规则（介于 loose 和 strict 之间）
特点：
标点通常不出现在行首
仅在允许断点处换行

**4. `strict`**
行为：严格换行，遵循排版规范（如 CSS Text Module Level 3）
特点：
禁止标点出现在行首
仅在明确允许的断点换行

**5. `anywhere` (谨慎使用)**
行为：任意位置均可换行（类似 word-break: break-all）
特点：
无视语言规则，强制在字符间换行
可能导致单词中间断开

## JSON.stringify
```js
JSON.stringify(value[, replacer[, space]])
```
接收3个参数。
第一个参数要处理的对象；

第二个参数处理对象的属性，可以是函数或数组。函数参数为key和value，返回undefined代表过滤掉该属性，返回其他值代表替换原属性值。数组为字符串数组，表示仅返回数组中的属性，顶层属性。

第三个参数控制缩进和格式化，可以是数字或字符串。数字为每级缩进空格数，大于10截断。字符串表示用该字符串作为缩进符。

## 切换node版本
方法1：nvm
修改启动命令
```js
"dev": "nvm use v18.20.4 && vite",
```
方法2：nvm-desktop
[nvm-desktop github](https://github.com/1111mp/nvm-desktop)

## linux命令
```bash
# 删除非空目录
rm -r /path/to/directory

# 移动文件夹
mv /path/from/ ./

# 重命名文件夹
mv xx yy
```
## nginx 504
原因：请求超时

解决方案：
```nginx
location /api {
    proxy_pass http://backend_server;  # 上游服务器地址
    proxy_read_timeout 300s;     # 默认60秒，调整为300秒（按需）
    proxy_connect_timeout 75s;   # 连接上游服务器的超时时间
    proxy_send_timeout 60s;      # 发送请求到上游服务器的超时
}
```
## FontFace文件路径问题
`FontFace`用于动态加载字体。
```js
const font = new FontFace('iconfont', 'url(./src/assets/fonts/iconfont.woff)');
await font.load();
document.fonts.add(font);
```

虽然以上代码在与assets同级的App.vue中，但url前面是 ==./src/assets==，其他都不行。原因未知。
font文件放public下用绝对路径也访问不到，只能这样。
如果不知道怎么能访问到字体文件，可以用@font-face加载一个字体文件，在network中看看这个文件的url是什么样。

## vuex的mutation调用
mutation只能接收2个固定参数，第一个是state，第二个是payload。如果尝试传递多个参数，只有第一个会传递成功，其他参数被忽略。
```js
mutations: {
    setUserInfo(state, { isSuper, adminList, userList }) {
      state.isSuper = isSuper;
      state.adminList = adminList;
      state.userList = userList;
    },
},
  
methods: {
    ...mapMutations(['setIsLoginVisible', 'setUserInfo']),
    test() {
        this.setUserInfo({
            isSuper: true,
            adminList: [],
            userList: []                
        })
    }
}
```
## 下载Blob格式压缩包时报错
即使页面是 HTTP，某些浏览器也会将 Blob URL 识别为潜在风险
```js
const blob = new Blob([data], {type: 'application/zip'});
const url = URL.createObjectURL(blob); // 可能触发警告
```

解决方案：改用 Base64 下载
```
const reader = new FileReader();
reader.onload = () => {
  const link = document.createElement('a');
  link.href = reader.result;
  link.download = 'file.zip';
  link.click();
};
reader.readAsDataURL(blob);
```
## npm login时报错Public registration is not allowed
由于使用了不支持公开注册的npm镜像源导致。例如淘宝镜像（registry.npmmirror.com）默认禁止公共账号注册和登录操作。

解决办法：
```bash
# 临时方案（仅当前命令生效）
npm login --registry=https://registry.npmjs.org/

# 永久方案（修改全局配置）
npm config set registry https://registry.npmjs.org/
```
## <el-scrollbar>与<el-backtop>搭配
`<el-backtop>`需要放`在<el-scrollbar>`里面，而且target为 **.el-scrollbar__wrap**。
```html
<el-scrollbar height="calc(100vh - 210px)">
    <SimilarItem v-for="item in searchResult" :key="item.id" :info="item" />
    <el-backtop target=".el-scrollbar__wrap" :right="100" :bottom="100"/>
</el-scrollbar>
```

## disabled-devtool使用
```js
DisableDevtool.md5('xxxx') // 无用
DisableDevtool({ 
  md5: 'xx', // 生效，32位小写
});
```
