---
title: macOS下搭建React Native的详细步骤
date: 2022-07-12 09:33:39
id: 0701
tags: React Native
categories: React Native
---

> 前言

最近一直在做 app 方面的开发，目前来说最常用的还是`uniapp`，心里多少有点不太想使用这个多端开发，编辑器也不顺手，想起了`react-native`和`flutter`，相比起来熟悉 react 的我选择了前者。

- 开发平台
  macOS Big Sur
- 系统版本
  11.2
- 芯片
  intel，2017 款 8+256 （配置着实有点低）

> 配置环境

说起配置环境，刚开始也不慌，毕竟作为开发配置过的环境已经很多了，但遇到了这个`react-native`不得不说一下。这是我配置 React-Native 环境的第三次吧，前两次不用说（失败了，每次都还耗费了两三个小时至少），这次也是耗费了三四个小时吧，勉强算是成功。心里也是松了口气，万事开头难，开头成功了相当于已经完成了 90%。

### 环境依赖安装

基本配置如下：

![Snipaste_2022-07-11_11-17-48.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1cabd042bdff4d0e8f828bfebd32b213~tplv-k3u1fbpfcp-watermark.image?)
附：我这里 pod -v 报错是因为使用的命令不对，改成 pod --version 就可以查看版本了。

![Snipaste_2022-07-11_11-19-11.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e388853170ca4a36867705ca27164653~tplv-k3u1fbpfcp-watermark.image?)
[官网](https://www.react-native.cn/)的说法：

![Snipaste_2022-07-11_13-52-04.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/151f2bd3fd264e63b02b4d3b88364ece~tplv-k3u1fbpfcp-watermark.image?)

- node 和 npm 的安装就不详说了，网上也有很多参考，重点说说下面的。
- 关于 yarn：
  ```js
  npm install -g yarn
  ```
- react-native：
  ```js
  yarn add react-native --exact
  ```
- brew：
  brew 的安装可以参考[这篇文章](https://www.jianshu.com/p/22122a1d4474)
- watchman：
  ```js
  brew install watchman
  ```
- pod（cocoapods）：
  `js brew install cocoapods `
  当以上这些安装好以及相应的版本号都能出现后，基本的环境搭建算是 OK 了，接下来就是创建项目和开发工具的搭建以及配置。（创建项目出错耗费了我好久）

### 项目创建

- 使用 npx react-native 创建项目
  `js npx react-native init testapp `
  看起来一切顺利：

![Snipaste_2022-07-11_14-06-26.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8d8d43db41c1455abf48ffcb5222d159~tplv-k3u1fbpfcp-watermark.image?)
这个过程可能有点长（等了将近七八分钟）：

![Snipaste_2022-07-11_14-10-31.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/88c3ede7fc7d4f47a7d2b002c7dc72c6~tplv-k3u1fbpfcp-watermark.image?)
估计就是连接不上外网，下载这个一般开个代理会比较快点，也可以运行下这个：

```js
1.  git config --global --unset http.proxy
2.  git config --global --unset https.proxy
```

设置完成后再重新创建项目试试，没啥报错的话就 OK 了，可以直接到安装软件那一步了。

**但是**我的本地还是报这个错无法解决，最后我查了好久，试试 init 创建项目（如下）：

```js
react-native init testapp
```

**结果**呢，又出现了另一个错误：

![Snipaste_2022-07-11_11-20-31.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eaad2c19a5f34ca5b23817deeb7a89ea~tplv-k3u1fbpfcp-watermark.image?)
此刻都有点想放弃了，配置个环境真麻烦。。。

此时的项目结构是这样：

![Snipaste_2022-07-11_13-16-08.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c69d8dd216ff4c1db2f7aaa8cc180f16~tplv-k3u1fbpfcp-watermark.image?)
然后各种百度、google 一大堆，有如下的方法：

- 将 react-native 更新到最新
  ```js
  yarn add react-native --exact
  ```
  结果还是一样。。。
- 指定项目的版本号：（试了好几次不行，最后花了好长时间，用了一个低的版本创建，结果就成功了！！！应该算是成功了吧），至少不报刚才的错误了。
  `js react-native init testapp --version 0.44.0 `
  ![Snipaste_2022-07-11_13-14-02.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9531d79c620c41e399a7358e29c635ff~tplv-k3u1fbpfcp-watermark.image?)

效果如下：（出来了运行命令，应该是没多大问题了）

![Snipaste_2022-07-11_13-14-34.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/da235b2d35c040b0a01f55ec34351320~tplv-k3u1fbpfcp-watermark.image?)

### 安装软件

- 上次安装了`Android Studio`，过程比较麻烦，后续环境没弄成功还被我卸载了。所以决定这次用 ios 的 Xcode 吧，反正也就是看看模拟器的效果。
- 在 App Store 搜索 xcode 安装，安装的时候提示了我这个，特意看了下系统的版本要求：

![Snipaste_2022-07-11_13-20-39.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3946428664ee45688684013d0fa726fe~tplv-k3u1fbpfcp-watermark.image?)
发现我自己的版本过低，万一安装出问题了重新下载比较麻烦（毕竟 10 多个 g）：

![Snipaste_2022-07-11_13-20-16.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26b0acded229401f9ce03fa4afd22c8b~tplv-k3u1fbpfcp-watermark.image?)

- 去[官网](https://xcodereleases.com/)看看，下载之前的版本：

  这里可以看到与自己电脑相匹配版本的 xcode：

![Snipaste_2022-07-11_13-22-31.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2e9382c19de645f99d3d18b66866a103~tplv-k3u1fbpfcp-watermark.image?)

- 找到自己电脑匹配的进行下载，下载的时候还要登录 AppleID：

![Snipaste_2022-07-11_13-24-38.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa6aa8d7a32a440b97bceeb481b8b538~tplv-k3u1fbpfcp-watermark.image?)

![Snipaste_2022-07-11_13-25-09.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a8bcb80278bc4665861cd5be251c5f60~tplv-k3u1fbpfcp-watermark.image?)
这个过程可能有点久，网速好点的话可能会快点。

![Snipaste_2022-07-11_13-26-21.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0f7340c241e94bac81cac5c4bd803039~tplv-k3u1fbpfcp-watermark.image?)

- 大概半来个小时吧，下载完成（主要看网速），完成后进行解压安装，解压也有点久，如果版本匹配的话更推荐在 App Store 下载，因为解压太久了，我差不多花了半小时才解压完。

![Snipaste_2022-07-11_14-44-45.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d0cabd3271d414bab0f018c5baf37b5~tplv-k3u1fbpfcp-watermark.image?)

![Snipaste_2022-07-11_14-48-16.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d71da549646040629146c7992c5c572c~tplv-k3u1fbpfcp-watermark.image?)

解压完成后：（可能是把大多数的 ios 和 tv 的 jdk 都下载了，有点大吧）

![Snipaste_2022-07-11_15-54-36.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ae12e4ac6004f569cbcebacf0ba9ded~tplv-k3u1fbpfcp-watermark.image?)

- 项目的目录结构是这样：（看到有个 index.ios.js 和 index.android.js 感觉不太妙，因为老的版本有个两个文件，新版本的话只有 index.js）

![Snipaste_2022-07-11_16-02-19.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3a7d3f0729f54ab5a8a01977a5d0e2c3~tplv-k3u1fbpfcp-watermark.image?)

- Xcode 安装好后运行项目中的 ios 目录，选择好模拟器的型号后，点击这个三角形运行。

![Snipaste_2022-07-11_16-05-15.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2edd082217804406adc739f10fe52a92~tplv-k3u1fbpfcp-watermark.image?)
如图所示：（等待开机）

![468401657525269_.pic.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/65db39ea25c94166b6a7f4414cd08724~tplv-k3u1fbpfcp-watermark.image?)
开机后，又等了一会，也没有出现想象中欢迎的界面：（白白的啥也没有，应该是出问题了）

![Snipaste_2022-07-11_15-46-44.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/813b861e366b45a2b5d35fcb1aacb270~tplv-k3u1fbpfcp-watermark.image?)

- 重新建一个高点的版本项目试试：（这次不指定版本号了，感觉可能又要出其它问题）
- 首先重新安装了下 cocoapods（一直是在安装 pods 依赖出的问题）
  `js sudo gem install cocoapods `
  关于 cocoapods 的问题可以参考这个[链接](https://stackoverflow.com/questions/58934022/react-native-error-failed-to-install-cocoapods-dependencies-for-ios-project-w/62192886)。
  ![Snipaste_2022-07-11_16-15-58.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/126cc989cf3643caa1d03d24f1beadf1~tplv-k3u1fbpfcp-watermark.image?)
- 重新建项目
  ```js
  npx react-native init AwesomeProject
  ```
- 又出现错误

![Snipaste_2022-07-11_16-14-12.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1731657f175f4cf58fadd2f247faa6f9~tplv-k3u1fbpfcp-watermark.image?)

- 这次按提示安装 pods（前几次也这样试了没用，这次在上面重新安装了执行）

```js
cd ./AwesmeProject/ios && pod install
```

![Snipaste_2022-07-11_16-37-57.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a92abbff90914b95a60ee2da39e417f9~tplv-k3u1fbpfcp-watermark.image?)
等了一会儿，好像也没有错误了。

![Snipaste_2022-07-11_16-38-23.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/19333635b61d42ccaa90cb90ebe1271e~tplv-k3u1fbpfcp-watermark.image?)

### 运行项目

- 这次的目录结构不一样

![Snipaste_2022-07-11_16-38-55.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/06108224e8bd4e718f11bb76891e7060~tplv-k3u1fbpfcp-watermark.image?)
按照 package.json 中的命令运行 ios 项目

- 用 Xcode 打开 ios 目录，运行后在建立索引，可能需要点时间，左侧是 ios 项目目录。

![Snipaste_2022-07-11_16-48-37.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b83f24cdc9e24ff1803ac31ddaad16e0~tplv-k3u1fbpfcp-watermark.image?)

- 模拟器效果（可以滑到第二页看到正在安装的 app）

![Snipaste_2022-07-11_17-03-54.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/020ea69b9a3a4d588b2c5ce091f50990~tplv-k3u1fbpfcp-watermark.image?)

- 等到安装完成，打开就能看到新建的项目了，这样就算是成功了。

![Snipaste_2022-07-11_17-03-32.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a7ecfac8a5a493bb4cb52a1e0ef28d9~tplv-k3u1fbpfcp-watermark.image?)

> 如果以上帮到你的话，点个赞吧！
