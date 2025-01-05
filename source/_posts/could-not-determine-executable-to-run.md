---
title: npm error-could not determine executable to run
date: 2024-10-10 10:00:00
tags:
  - npm
categories:
  - 问题
cover: 
---

# 遇见问题
在用我自己的脚手架创建项目时，遇到了一个报错，但别人同样的操作没有报错。
![报错信息](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/image.png)   
然后打开.log文件，看到了更详细一点的报错信息。
![报错2](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/image%20(1).png)
# 分析问题
搜索`Error: could not determine executable to run`和`getBinFromManifest`发现，并没有明确的原因，很多人解决的方法[（解决方案集合）](https://stackoverflow.com/questions/67833794/npm-err-could-not-determine-executable-to-run)都完全不一样。我一一都试了也不行。    
然后再看报错信息的后面几行，发现了有什么地方不对劲，
![报错3](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/image%20(2).png)   
我要使用的`create-pro`的版本根本不是0.0.0，而打开第14行的链接，这个npm包也不是我要用的那个。我的库是放在某个scope下的，这个链接指向的包与我的包同名，但不在scope内。
但是我要用的库已经全局安装过了，不明白为什么npm拉错了包。
# 解决问题
```bash
# npm create pro // 原命令
npm create @chen/pro // √
```
# 总结
遇到这个报错的时候，如果也是使用脚手架，可以看看包是否在scope内。不然，可以先看看命令有没有语法错误，然后试试重启vscode、重新安装依赖、赋予当前用户对node完全控制的权限等方法。