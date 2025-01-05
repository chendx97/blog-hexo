---
title: git LFS解决大文件上传
date: 2024-02-04 10:00:00
tags:
  - git
categories:
  - 问题
cover: 
---
# 前言
git 限制上传文件大小在 100M 以内。在日常工作中，偶尔会遇到某次提交的文件总量过大或者某个文件过大，所以在此记录一下如何解决这个问题。
# 分批次提交
如果是提交很多文件，但单文件都不大的时候，可以选择分批次提交。如果已经 commit 才发现文件总量过大，则需要撤销 commit 。以下是会用到的 git 命令。
```bash
git reflog 查看所有操作的历史记录
git reset HEAD^2 上上一个版本
git reset @~ 撤销一次commit
```
如果仅撤销上一次，则直接用第3条命令。  
如果在尝试解决问题时进行了多次 commit，则可以通过第1条命令查询想要回退到的版本，然后通过第2条命令回退。
# LFS
首先，下载[git LFS](<https://git-lfs.com/>)。  
然后，关联需要用 LFS 上传的文件，比如下面的命令。
```bash
git lfs track 'xx.mp4'
```
最后，按照正常的提交流程进行提交。
   

以下是可能用到的命令
```bash
# 关联某类型文件
git lfs track '*.mp4' 

# 查找大于 nM 的文件
find ./ -size +100M
```