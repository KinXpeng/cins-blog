---
title: Github Actions自动化部署前端项目（超详细）
date: 2022-09-02 17:01:58
id: 0901
tags: 'GitHub Actions'
categories: 'GitHub Actions'
---

> 前言

关于部署项目，一直以来我也没注重太多；作为前端开发，例如自己 [`github`](https://github.com/KinXpeng/) 代码仓库的项目,就直接使用 `git pages` 了，一是用起来方便，而是也不需要额外购置云服务器。

对于 `git pages` ，部署项目会先选择部署的分支，然后选择部署的目录就 OK 了，但是部署的目录只能是 docs 或者根目录，所以一直以来用的都是 docs 目录，在前端 vue 项目或者 react 项目打包输出成 docs 就行，所以对于 `Github Actions` 也没有太多的了解。

直到最近几天，准备搭建一个文档类型的网站，本来也是准备采用 docs 目录进行部署，但是搭建的项目里已经存在了 docs 目录，打包的时候就只能换个目录输出，这才有了以下的 `Github Actions`。

### 如何创建 wrokflow

- 关于 github 上如何创建代码仓库以及关联本地项目就不做介绍了，本文主要讲解下前端工程化项目的自动打包部署。
- 在代码仓库点击 `Actions` ，然后选择 `New workflow`。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c330c2c0cbfd40c9b9db82b080faac26~tplv-k3u1fbpfcp-watermark.image?)

- 这里的话新建自己的 `Workflow`。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d63017c64daa4fa0abb8a96c96dc3349~tplv-k3u1fbpfcp-watermark.image?)

- 默认为 `main.yml`，名称随意，点击提交即可。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9045166e41c143fe92984c3a311eb329~tplv-k3u1fbpfcp-watermark.image?)

- 之后自己的项目下就会生成一个 `.github` 的文件，拉取一下代码，在本地配置一下 actions。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ab44061e11f04ca3aa117b72c7de0ee2~tplv-k3u1fbpfcp-watermark.image?)

### 配置 wrokflow

- 打开拉取到本地的配置文件

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/645bef20ec7e4fd89b438a84d69d3071~tplv-k3u1fbpfcp-watermark.image?)

- 主要配置代码如下

  ```yml
  name: CI
   # 触发条件：在 push 到 main/master 分支后，新的 Github 项目 应该都是 main，而之前的项目一般都是 master
   on:
     push:
       branches:
         - main

   # 任务
   jobs:
     build-and-deploy:
       # 服务器环境：最新版 Ubuntu
       runs-on: ubuntu-latest
       steps:
         # 拉取代码
         - name: Checkout
           uses: actions/checkout@v2
           with:
             persist-credentials: false
         # 生成静态文件
         - name: Build
           run: |
             npm install  # 安装依赖
             npm run docs:build  # 执行打包

         # 部署到 GitHub Pages
         - name: Deploy
           uses: JamesIves/github-pages-deploy-action@v4.3.3
           with:
             ACCESS_TOKEN: ${{ secrets.ACCESS_TOKEN }} # 也就是我们刚才生成的 secret
             BRANCH: gh-pages # 部署到 gh-pages 分支，因为 main 分支存放的一般是源码，而 gh-pages 分支则用来存放生成的静态文件
             FOLDER: docs-dist # 生成的静态文件存放的地方
  ```

- 其中 `on` 部分为监听的分支，默认为 `main`。
- `jobs` 为执行的任务，安装依赖和执行打包的命令根据自己的需要修改，我这里的打包为文档，一般默认为 `npm run build` 就 OK。
- 最后的 `FOLDER` 为打包输出的目录，会将该目录的文件存放到 `gh-pages` 分支下。
- 注意：`ACCESS_TOKEN` 为 git 上生成的 secret，下面将会说如何生成 secret。

### 生成 secret

- 点击 git 中右上角的头像，选择设置。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/31cb4ccea692490490f59fcaa97aff0c~tplv-k3u1fbpfcp-watermark.image?)

- 选择 `Developer settings`

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/dad45cba40d34693b093c13973e24d0c~tplv-k3u1fbpfcp-watermark.image?)

- 选择最后一项，然后点击 `Generate new token`

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7759e52c6cfd4d72907658bb7ac961bc~tplv-k3u1fbpfcp-watermark.image?)

- 勾选上以下几项

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b87c0718976e43399bd74b322ca935fb~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f878d5afce14cb2a6a716bceab56ed6~tplv-k3u1fbpfcp-watermark.image?)

- 这时候就能看到创建的 token 了。
- 打开某个仓库的设置，点击 actions

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/663fb1127310464db1a5f4dbf7ec7b9f~tplv-k3u1fbpfcp-watermark.image?)

- 点击右上角的 New repository secret，将刚才生成的 token 存入其中。Name 和配置文件中的一致，我这里设置的是 ACCESS_TOKEN。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d2ea834e61a4a9293e903bd9c6c7839~tplv-k3u1fbpfcp-watermark.image?)

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b976829f85644d76866aff1e873d94ea~tplv-k3u1fbpfcp-watermark.image?)

- 完成后基本的配置就 OK 了。

### 配置 Github Pages

上述的步骤完成后，关于 actions 的配置也就结束了。本地代码 push 之后，就可以在 actions 里面看到正在执行的了。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/56d5e5e612094aab82ed3af5423981ba~tplv-k3u1fbpfcp-watermark.image?)
关于 git-pages 的配置，在设置里选择 pages，pages 的配置选择 `gh-pages` 分支和根目录即可。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b466bd9a70cb445c85057c8f5da4b1c8~tplv-k3u1fbpfcp-watermark.image?)
完成以上配置就可以看到能自动化部署成功了，这里是我部署后的页面。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2040e47c29fa448b97e39e64316c2aa5~tplv-k3u1fbpfcp-watermark.image?)
关于域名可以在 git pages 里绑定自己的域名，提前要在阿里云或者腾讯云做好域名解析即可；记录类型选择 `CNAME`，记录值为自己 github 的名称，如下所示。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c88a4eeb8bca49f19d1a5d1e387e32de~tplv-k3u1fbpfcp-watermark.image?)

> 喜欢的话点个赞吧！
