---
title: 桌面端软件实现 GitHub OAuth 第三方登录
date: 2018-12-26 16:19:56
categories: 技术相关
tags: [github-oauth]
toc: true
reward: true
---
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/others/05.jpg" width="65%" height="65%">

OAuth (开放授权 Open Authorization) 是一个开放标准，允许用户授权第三方应用访问他们存储在另外的服务提供者上的信息，而不需要将用户名和密码提供给第三方应用或分享他们数据的所有内容。
但 GitHub OAuth 对桌面端的认证并没有直接的支持，本文提供了一种利用浏览器的缓存功能实现的针对桌面端 (移动端也适用) 的 GitHub OAuth 第三方登录方式，并提供了一个实例 (通过 Express.js 和 socket-io.js 搭建)。
<!-- more -->

## 思路和方法

### 网页应用的授权步骤

GitHub 提供了针对网站的第三方登录策略，主要步骤如下：
1. 注册一个 OAuth App，得到 App 的 **Client ID** 和 **Client Secret**，具体请查看 [Creating an OAuth App](https://developer.github.com/apps/building-oauth-apps/creating-an-oauth-app/)
1. 利用浏览器访问 `https://github.com/login/oauth/authorize?client_id={Client_ID}&scope=user:email` (将 **Client_ID** 替换为上一步得到的 **Client ID**，**scope** 参数是你希望得到的用户信息的内容，具体请查看 [understanding-scopes-for-oauth-apps](https://developer.github.com/apps/building-oauth-apps/understanding-scopes-for-oauth-apps/))
1. 当用户同意授权后，浏览器会重定向到我们到注册 App 时提供的回调地址，并在重定向链接中附带一个 **code** 参数。
1. 后台利用 **code** 参数和第一步得到的 **Client ID** 和 **Client Secret** 访问 `https://github.com/login/oauth/access_token?client_id=${Client_ID}&client_secret=${Client_Secret}&code=${code}` 便可得到一个 **access_token**
1. 利用得到的 **access_token** 结合 GitHub Api 便可访问该用户的信息

### 桌面端软件授权的问题

因为获取 **access_token** 需要 **code** 参数，而 **code** 参数是重定向链接的内容之一，只能在浏览器中获得。

在网页应用中，后台通过 **code** 参数后可以得到 **access_token**，然后将 **access_token** 放到渲染好的网页中，发送到浏览器，浏览器将 **access_token** 保存到本地缓存，之后每次都访问网页资源都把本地缓存的 **access_token** 发送到后台，后台便能获得需要的用户信息，通过网页发送到浏览器。

然而对于桌面端软件，软件可以自行打开浏览器访问认证链接，让用户确认是否授权，但得想个方法将后台获得的 **access_token** 发送到桌面端软件。其中最重要的是让后台知道不同的浏览器对应的桌面端软件是哪一个，解决这个问题之后，才能把用户在不同浏览器中确认授权后得到的 **access_token** 发送到对应的桌面端，桌面端把它存到软件的配置文件中。第二个问题是如何把 **access_token** 发送到桌面端。

### 实现方法

针对这两个问题，我分析了一下 [GitKraken](https://www.gitkraken.com/) 的登录策略，找到了他们利用 GitHub OAuth 登录的方法，这也是本篇文章介绍的重点内容。

对于后台如何主动向客户端发送 **access_token** 的问题，解决的办法比较简单，让后台和桌面端软件建立 Socket 连接实现双向通信，每个 Socket 连接都有一个唯一标识，即 Socket id，当后台得到 **access_token** 后，利用 Socket id 将 **access_token** 发送到指定的桌面端即可。

> 当然也可以采取轮询或 long pull 实现双向通信，但因为我们这里的后台相当于一个桥梁，用于连接客户端和 GitHub 的后台，所以对于每个客户端，都必须有一个唯一标识让后台知道每个 **access_token** 应该发给谁，Socket 连接本身提供了这个标识，无需我们额外产生，而这两种方法都需要自己产生一个唯一标识，相应的，后台也需要做一些多余的操作来储存这些标识，本文不采用这两种办法。

解决了第二个问题，第一个问题中区分每个客户端的问题也就解决了，接下来要做的是实现把浏览器和客户端一一对应起来，现在浏览器的缓存技术(缓存客户端和服务器连接的 Socket id)终于可以派上用场了，步骤如下：

![工作流](http://home.cs-tao.cc/github-content/contents/blog/image/others/github-oauth-flow.png)

1. 客户端主动建立 Socket 连接，得到 Socket id
1. 将 Socket id 和其他必要的参数 (比如区分是否是移动端的参数) 嵌入到一个让用户确认是否继续认证的链接中，打开浏览器访问该链接，这个网页需要做的工作是在用户点击确认后，把 Socket id 和其他参数存到本地缓存中，然后访问认证链接
(即 `https://github.com/login/oauth/authorize?client_id={Client_ID}&scope=user:email`)
1. 在认证页面，用户确认授权后，网页会被重定向到我们在注册 App 时提供的回调地址，并附带 **code** 参数，回调地址产生的连接进入阻塞状态，等待服务器返回 **access_token**
1. 回调地址对应的后台处理模块将 **code** 参数、**Client ID** 和 **Client Secret** 发送至 GitHub 后台
(链接是：`https://github.com/login/oauth/access_token?client_id=${Client_ID}&client_secret=${Client_Secret}&code=${code}`)
1. GitHub 后台验证 **code** 等参数的可用性，如果可用，GitHub 后台会将 **access_token** 返回给后台服务
1. 后台得到 **access_token** 后，将它发送给浏览器 (第 3 步的阻塞状态结束)
1. 浏览器得到 **access_token** 后，再将 **access_token**、缓存的 Socket id 和其他参数发送到后台，然后显示发送成功或失败的信息
1. 后台相应的处理模块通过这些参数，将 **access_token** 发送到指定 Socket id 的桌面端

## 实例

### 平台和框架

我用 Express.js 和 Socket-io.js 搭建了一个后台服务示例，GitHub 仓库地址：[whu-library-seat-ghauth](https://github.com/CS-Tao/whu-library-seat-ghauth)。

> 桌面端仓库：[whu-library-seat](https://github.com/CS-Tao/whu-library-seat)，移动端仓库：[whu-library-seat-mobile](https://github.com/CS-Tao/whu-library-seat-mobile)。

这里不再赘述代码上的细节，我将之前写的使用方法和截图记录在这里，供大家对比和参考。

### 使用方法

1. 点击软件下方的**钥匙**进入软件授权页面(第一次打开软件会默认进入本页面)

  ![图片加载失败](http://home.cs-tao.cc/github-content/contents/github/whu-library-seat/OAuth/1.png)

1. 点击`GitHub Star 永久授权`按钮，软件会打开系统浏览器访问认证页面

  ![图片加载失败](http://home.cs-tao.cc/github-content/contents/github/whu-library-seat/OAuth/3.png)

1. 点击`确定通过 GitHub 账号登录`，此时 GitHub 会让你确认是否授权(如果没有登录 GitHub，此时会进入登录页面)

  ![图片加载失败](http://home.cs-tao.cc/github-content/contents/github/whu-library-seat/OAuth/4.png)

1. 点击`Authorize CS-Tao`即可成功登录

  ![图片加载失败](http://home.cs-tao.cc/github-content/contents/github/whu-library-seat/OAuth/5.png)

1. 登录成功后返回软件，如果出现下面的弹窗，说明您还未对本仓库点星，请进行下一步

  ![图片加载失败](http://home.cs-tao.cc/github-content/contents/github/whu-library-seat/OAuth/5.1.png)

1. 如果您还未给本仓库点星，请到指定仓库点星以供管理员了解软件使用情况。桌面端进入：[whu-library-seat](https://github.com/CS-Tao/whu-library-seat)，移动端进入：[whu-library-seat-mobile](https://github.com/CS-Tao/whu-library-seat-mobile)

  - 桌面端点击右上角的`Star`按钮，按钮如下图所示：

    ![图片加载失败](http://home.cs-tao.cc/github-content/contents/github/whu-library-seat/OAuth/5.2.png)

  - 移动端需要登录才会显示`Star`按钮，登录状态下直接点击即可，按钮如下图所示：
  
    ![图片加载失败](http://home.cs-tao.cc/github-content/contents/github/whu-library-seat/OAuth/5.3.png)

1. 点星后回到本软件，点击确定即可

  ![图片加载失败](http://home.cs-tao.cc/github-content/contents/github/whu-library-seat/OAuth/6.png)
