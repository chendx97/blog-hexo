---
title: 💐Docker基础知识总结
date: 2024-09-02 10:00:00
tags:
  - docker
categories:
  - 指南
cover: 
---

# 1.docker的作用
打包、分发、部署  
打包：就是把你软件运行所需的依赖、第三方库、软件打包到一起，变成一个安装包；  
分发：你可以把你打包好的“安装包”上传到一个镜像仓库，其他人可以非常方便的获取和安装；  
部署：拿着“安装包”就可以一个命令运行起来你的应用，自动模拟出一模一样的运行环境，不管是在 Windows/Mac/Linux。   
# 2.docker命令
```docker
docker build -t test:v1 .     
```
-t 指定镜像名字和版本号；  
. 指在当前目录build，将当前目录作为构建上下文；   
参考链接：[docker build命令](https://docs.docker.com/engine/reference/commandline/build/)
```docker
docker run -dt -p 8089:8089 demo
```
-p port1:port2 指定端口号，port1要访问的端口号，port2docker内部使用的端口号；  
-d 后台运行，-i 以交互模式运行容器，-t为容器重新分配一个伪输入终端，-it在一个新终端里运行容器，-dt在终端的后台运行容器；  
参考链接：[docker run命令](https://docs.docker.com/engine/reference/commandline/run/)  

其他常见命令如下所示：  
docker ps 查看当前运行中的容器  
docker exec -it  containerid sh  // 进入容器，在容器内可以执行npm run start等命令  
docker images 查看镜像列表  
docker rm container-id 删除指定 id 的容器  
docker stop/start container-id 停止/启动指定 id 的容器  
docker rmi image-id 删除指定 id 的镜像

# 3.Dockerfile文件
文件中命令解释如下所示：  
```docker
FROM node:14-alpine # 指定基础镜像
WORKDIR /app # 指定shell语句运行的目录，如果目录不存在会自动创建
COPY a.txt /app/ # 将本地文件拷贝到容器的某目录下
# ADD a.txt /app/ # 拷贝本地或远程文件,可以自动解压缩，还会使构建缓存失效
RUN npm i # 运行shell语句，构建时运行
CMD  ["npm", "run", "start"]  # 指定镜像启动运行的脚本
ENTRYPOINT ["npm", "run", "start"] # 类似CMD  
MAINTAINER <name>  # 指定生成镜像的作者，LABEL更灵活。  
EXPOSE 8089 # 暴露镜像在容器内运行的端口号  
ENV PORT=8089 # 指定环境变量，运行时也生效  
ARG   # 构建参数，运行时无效，可以构建时候临时修改变量
```
教程1：https://docker.easydoc.net/doc/81170005/cCewZWoN/lTKfePfP  
教程2：https://www.bilibili.com/video/BV1k7411B7QL/?spm_id_from=333.337.search-card.all.click&vd_source=6452cc89d77d512e999c37668481d36b  
教程3：https://yeasy.gitbook.io/docker_practice/appendix/best_practices#dockerfile-zhi-ling  
# 4.dockerfile配置方案
方案1：
本地打包，然后复制到容器中。
```docker
# node 启动
FROM node:14-alpine

WORKDIR /app
COPY package.json /app/
COPY ./.output /app/.output

EXPOSE 8089
ENV PORT=8089
ENTRYPOINT ["npm", "run", "start"]
```

```docker
# nginx
FROM nginx:alpine

COPY /dist /usr/share/nginx/html/
COPY nginx.conf /etc/nginx/conf.d/nginx.conf
```
方案2：把本地整个文件夹复制到容器中，在容器中打包。需要添加.dockerignore文件，避免上传的文件夹比较大。
```docker
# node
FROM node:14-alpine

WORKDIR /app
COPY . /app/
RUN cd /app && yarn && yarn build
ENTRYPOINT ["npm", "run", "start"]
```
```docker
# nginx
FROM nginx:alpine

WORKDIR /app
COPY . /app/
RUN cd /app && yarn && yarn build

COPY /app/dist /usr/share/nginx/html/
COPY nginx.conf /etc/nginx/conf.d/nginx.conf
```

方案1需要在docker build前执行一遍yarn build，  
方案2打包出来的镜像比较大。
# 5.报错解决
This version of npm is compatible with lockfileVersion@1, but package-lock.json was generated for lockfileVersion@2  
docker版本和本地电脑中node版本不匹配，升级docker中的node，或使用yarn。
# 6.调试docker build
在dockerfile中只设置基础镜像，然后启动，在终端中运行需要的步骤，如果成功就把每个步骤的命令写入dockerfile文件中。
```docker
FROM node:14-alpine
ENTRYPOINT ["tail", "-f", "/dev/null"]
```
进入调试：
```bash
docker build -t demo .
docker run -dt -p 8089:8089 demo
docker exec -it  containerid sh
```
然后可以在终端中执行Dockerfile中想要添加的RUN命令。  

调试后关闭容器：
```bash
docker ps
docker stop id
```
