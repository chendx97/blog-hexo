---
title: 🛠如何使用CICD实现自动化部署
date: 2024-10-08 10:00:00
tags:
  - CICD
  - Git
categories:
  - 指南
cover: 
---

# CICD

CICD 是“持续集成”（Continuous Integration）和“持续交付/部署”（Continuous Delivery/Deployment）的缩写。它是一种软件开发实践，旨在通过自动化工具和流程提高软件开发的质量和效率。

持续集成要求开发人员频繁地将代码提交到共享的版本控制系统中。

持续集成的流程：

1.  开发人员提交代码到版本控制系统（如 Git）。
2.  自动化构建系统自动拉取最新的代码。&#x20;
3.  执行自动化构建和测试。&#x20;
4.  发现并报告错误或失败的测试。

持续交付是在持续集成的基础上进一步扩展，确保每次提交的代码都可以随时部署到生产环境中。

持续交付的流程：

1.  自动化构建和测试。
2.  自动化部署到测试环境。
3.  手动或自动化部署到生产环境。

持续部署是在持续交付的基础上进一步扩展，自动将通过测试的代码部署到生产环境中。

持续部署的流程：

1.  自动化构建和测试。
2.  自动化部署到测试环境。&#x20;
3.  自动化部署到生产环境。

# Github Actions

GitHub Actions 是 GitHub 提供的一种强大的自动化工具，用于自动化构建、测试和部署代码。它允许你在 GitHub 仓库中直接编写和运行工作流（workflows），从而实现持续集成（CI）和持续部署（CD）。

Workflow：一个工作流（workflow）是由一系列步骤（steps）组成的 YAML 文件，用于定义自动化任务。

Job：一个工作流可以包含一个或多个作业（job），每个作业在不同的虚拟环境中运行。

Step：每个作业由一系列步骤（step）组成，每个步骤可以执行 shell 命令、运行脚本或使用预定义的动作（action）。

Action：动作（action）是 GitHub Actions 中的基本构建块，可以是自定义的脚本、容器或预定义的社区动作。

工作流的触发方式有很多，一般使用**push**。

常用动作如下所示：
```
    - name: Checkout Repository
      uses: actions/checkout@v3 # 检出仓库中的代码

    - name: Setup Node.js
      uses: actions/setup-node@v3 # 设置node环境
      with:
        node-version: 16

    - name: Install Dependencies
      run: npm install # 运行shell脚本或命令

    env: # 定义环境变量
      NODE_ENV: production
      DATABASE_URL: ${{ secrets.DATABASE_URL }}
```
# 创建工作流

在项目根目录下，创建文件.github/workflows/deploy.yml，内容如下所示，
```
    name: Deploy to GitHub Pages # 定义工作流的名称

    on: # 定义触发工作流的事件，如push、pull_request等
      push: # 当有代码推送到特定分支时触发工作流
        branches: # 限定触发工作的分支，数组
          - main
    	if: contains(github.event.head_commit.message, 'deploy') # 当提交一个包含 deploy 关键字的 commit 时触发
    # 以上，总的来说，在main分支提交包含 deploy 的commit 时，会触发工作流

    jobs: # 定义工作流中的作业（job）
      build-and-deploy: # 定义一个名为 build-and-deploy 的作业，每个作业有一个唯一的名称
        runs-on: ubuntu-latest # 指定作业运行的操作系统环境

        steps: # 定义作业中的步骤（step）
        - name: Checkout Repository # name 定义步骤的名称，便于识别和调试
          uses: actions/checkout@v3 # 使用预定义的 actions/checkout 动作来检出代码。
          # uses 后面指定动作的来源和版本，actions/checkout@v3 表示使用 actions/checkout 的第 3 版本。

        - name: Setup Node.js
          uses: actions/setup-node@v3 # 使用预定义的 actions/setup-node 动作来设置 Node.js 环境。
          with: # 定义动作的输入参数
            node-version: 20
        
        - name: Clear Cache
          run: | # 执行 shell 命令或脚本。run 后面指定要执行的命令或脚本，| 表示多行命令。
            rm -rf package-lock.json node_modules # 删除 package-lock.json 和 node_modules 目录。

        - name: Install Dependencies
          run: npm install # 安装项目依赖

        - name: Build Project
          run: npm run build # 构建项目

        - name: Deploy to GitHub Pages
          uses: peaceiris/actions-gh-pages@v3 # 使用预定义的 peaceiris/actions-gh-pages 动作来部署项目到 GitHub Pages。
          with:
            github_token: ${{ secrets.GITHUB_TOKEN }}
            publish_dir: ./dist
            user_name: 'your name'
            user_email: 'your email'
```
另外，项目需要添加base路径配置，

```javascript
// vite.config.js

base: process.env.NODE_ENV === 'production' ? '/your repository/' : '/',
```

然后确保`Settings/Pages/Build and deployment中`，branch为**gh-pages**，目录为/(root)。

最后，通过`Settings/Pages/Github Pages`下显示的地址就可以访问项目了。

![image](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/20241007235921.png)

# 可能遇到的问题

问题1：推送无权限

![image](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/20241007225937.png)

解决办法：

检查权限设置，在`Settings/Actions/General/Workflow permissions`配置中，选择`Read and write permissions`。

问题2：Cannot find module @rollup/rollup-linux-x64-gnu.

![image](https://cdn.jsdelivr.net/gh/chendx97/CPics/img/20241007230511.png)

解决办法：

清除缓存，在**npm install**前添加如下配置，

       - name: Clear Cache
          run: |
            rm -rf package-lock.json node_modules

