---
title: Nginx实现目录浏览功能及权限校验
date: 2024-06-26 10:00:00
tags:
  - nginx
categories:
  - Nginx
cover: 
---
# 前言

需要实现的功能是，打开 http://a.b.c.d 可以访问服务器上的某个目录。而且，在第一次登录及登录过期时会进行登录，登录成功后才可以访问到目录。

# 开启目录浏览功能

使用 **autoindex** 指令可以开启目录浏览功能。  
不仅要设置autoindex，还必须要设置root或alias。


## 访问 / 时，根目录开启目录浏览
```
localtion / { 
    root /; 
    autoindex on; 
}
```
## 访问存在的path时，该子目录开启目录浏览
```
location /a/ { 
    root /; 
    autoindex on; 
}
```
## 访问不存在的path时，某个子目录开启目录浏览功能
```Nginx
location /x/ { 
    alias /b/; # 注意 最后一个/是必需的 
    autoindex on; 
}
```
## 其他autoindex指令

- *autoindex_exact_size* on | off;：控制是否显示文件的精确大小。默认为 on。

- *autoindex_format html* | xml | json | jsonp;：指定目录列表的格式。默认为 html。

- *autoindex_localtime* on | off;：控制是否显示文件的最后修改时间为本地时间。默认为 off。
# root和alias的区别

相同点：都用于指定请求的文件在文件系统中的位置。

不同点：

第一，root 可以在 http、server 或 location 块中使用，alias 只能在 **location** 块中使用。

第二，nginx将uri**拼接**在root指定的路径后面，但alias会**替代**指定的路径。

举例如下：
```
location /docs/ { 
    root /var/www/html; 
}
```
匹配到这条规则时，nginx查找的文件是 */var/www/html/docs/index.html*
```
location /docs/ { 
    alias /var/www/html/; 
}
```
匹配到这条规则时，nginx查找的文件是 */var/www/html/index.html*

# 权限校验

## 校验方法一：**auth_basic_user_file**

将用户名密码保存在一个文件中，将文件路径添加到配置中。

密码可以通过[tool-htpasswd](<https://tool.oschina.net/htpasswd>)生成，windows上加密算法选md5，linux上选crypt。
```
location / { 
    root /; 
    autoindex on; 
    auth_basic "Authorized users only"; 
    auth_basic_user_file /pwd/htpasswd; 
}
```
htpasswd生成的密码的前缀指示了加密方法。nginx会根据密码文件的内容自动识别加密方式。

## 校验方法二：请求自己的登录接口

具体思路：访问时判断是否有指定cookie，如果没有跳登录，如果有正常访问。登录接口需要能跳转回来。
```
location / { 
    if ($cookie_my_cookie = '') { 
        return http://x.x.x.x/login?redirect_url=http://a.b.c.d; 
    } 
    root /; 
    autoindex on; 
}
```
如果接口不能把token注入到cookie中，而是放在了跳转回来的query参数里，则需要自己注入cookie并且把url里的query清空，如下所示。
```
location / { 
    if ($cookie_my_cookie = '') { 
        return http://x.x.x.x/login?redirect_url=http://a.b.c.d/login; 
    } 
    root /; 
    autoindex on; 
} 
location /login/ { 
    add_header Set-Cookie "my_cookie=$args"; 
    rewrite ^/login(.*) /? permanent; # 注意 /后面的?是必需的 
}
```
# 涉及的Nginx知识点
## nginx变量

**$args**：query参数；

**$cookie_my_cookie**：cookie中name为my_cookie的值；

## nginx if语句

只有if，没有else。
```
if ($cookie_my_cookie = "") { 
    # 如果cookie为空，执行下面的操作 
} 

if ($cookie_my_cookie ~* "value") { 
    # 如果cookie包含value，执行下面的操作 
} 

if ($cookie_my_cookie !~* "value") { 
    # 如果cookie不包含value，执行下面的操作 
}
```
## 重定向

### return指令
```
location /x/ { 
    return 301 http://localhost:8083/abc; 
}
```
如果访问/x?123，会跳转到/abc。

return不仅可以返回一个url，也可以返回一句话或者一个json。

第一个参数是**状态码**，第二个参数是可选的**消息体**。
```
location / { 
    default_type text/html; 
    return 200 "Hello, World!"; 
} 

location / { 
    default_type application/json; 
    return 200 "{a:1}"; 
}
```
- text/plain：适用于返回的文本内容，默认值。

- text/html：适用于返回的HTML内容。

- application/json：适用于返回的JSON格式内容。

- application/xml：适用于返回的XML格式内容。

- application/octet-stream：适用于返回的二进制数据。

### rewrite指令
```
location /x/ { 
    rewrite ^/x(.*) /y? permanent; 
}
```
如果不加?会保留原来的query参数，加?会不保留。

### try_files指令
```
location /x/ { 
    try_files $uri$uri/ /index.html; 
}
```
如果找不到，就返回index.html。

*$uri*：尝试匹配确切的请求路径。

*$uri/*：尝试匹配请求路径的目录版本。

# 其他
## nginx命令
```bash
start nginx # 启动 
nginx -s reload # 刷新 
nginx -s stop # 关闭 
tasklist /fi "imagename eq nginx.exe" # 查看全部nginx进程 
taskkill /pid pid /f # 关闭某个nginx进程 
taskkill /f /im nginx.exe # 关闭所有nginx进程
```
## location /a和location /a/的区别

location /a匹配/a、/a/、/ab

location /a/匹配/a/、/a/b

## 刷新不生效
```
sendfile off;
```
还可以查看nginx进程是否有很多个，如果是全部关闭再启动。

## 配置本地域名映射

找到**hosts**文件，在C:\Windows\System32\drivers\etc下。
```
localhost my.com
```