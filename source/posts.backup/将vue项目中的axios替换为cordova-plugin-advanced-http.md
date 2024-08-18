---
title: 将 Vue 项目中的 Axios 替换为 cordova-plugin-advanced-http
date: 2018-11-03 11:03:44
categories: 技术相关
tags: [Vue, cordova]
reward: true
---
<img src="http://home.cs-tao.cc/github-content/contents/blog/image/others/cordova-vue.png" width="20%" height="20%">

对于 Vue 项目，官方建议的网络请求工具是 Axios，但在 Vue 项目和 cordova 集成的时候，我发现想要更改网络请求的 header 并不是一件很简单的事(比如修改 User-Agent)，cordova 对 Axios 发出的请求似乎还添加了一层封装。为了修改 User-Agent，不得已最后还是使用了 cordova 自己的 http 插件：[cordova-plugin-advanced-http](https://github.com/sfelipegp/cordova-plugin-advanced-http).<br/>不难发现 Axios 是使用的 Promise 实现异步网络请求，而 cordova-plugin-advanced-http 使用的却是函数回调。所以实现这个转换最关键问题其实是如何将函数回调替换为 Promise.
<!-- more -->

> 当然，能实现这个转换的前提是您在项目中是通过 `axios.create()` 创建了全局能访问到的实例，并且所有的网络请求都是调用这个实例实现的。也就是说本文记录的是如何将原来使用 `axios.create()` 创建的实例转换为 cordova-plugin-advanced-http 的网络请求函数。

具体步骤如下：

### 1. 添加 cordova-plugin-advanced-http 插件

```bash
cordova plugin add cordova-plugin-advanced-http
```

### 2. 添加 cordova 全局变量到 Vue 实例

在 main.js 中添加下面的代码：
```js
Vue.cordova = Vue.prototype.$cordova = window.cordova
```

### 3. 修改核心文件

原文件内容：

```js
import axios from 'axios'

// 创建 Axios 实例
const request = axios.create({
  baseURL: 'http://127.0.0.1:8888',
  timeout: 5000,
  withCredentials: true
})

// 发出之前的拦截器
request.interceptors.request.use(config => {
  // 修改请求头
  config.headers['Content-Type'] = 'application/x-www-form-urlencoded; charset=UTF-8'
  config.headers['User-Agent'] = null
  return config
}, error => {
  Promise.reject(error)
})

// 接收到请求后的拦截器
request.interceptors.response.use(response => response, error => {
  // 打印错误
  console.error(error.message)
  return Promise.reject(error)
})

export default request
```

修改之后的文件内容：

```js
import Vue from 'vue'

// 修改请求头
Vue.cordova.plugin.http.setHeader('*', 'Content-Type', 'application/x-www-form-urlencoded; charset=UTF-8')
Vue.cordova.plugin.http.setHeader('*', 'User-Agent', '')

const baseURL = 'http://127.0.0.1:8888'

const request = (options) => {
  // 将原 Axios 的配置项改为 cordova-plugin-advanced-http 的配置项
  var sendOptions = {
    method: options.method,
    timeout: 5,
    headers: options.headers ? options.headers : null,
    params: options.params ? options.params : null,
    data: options.data ? options.data : null
  }
  // cordova-plugin-advanced-http 的 headers 只支持字符串
  for (var key in sendOptions.params) {
    sendOptions.params[key] = options.params[key].toString()
  }
  var url = baseURL + options.url
  return new Promise((resolve, reject) => {
    Vue.cordova.plugin.http.sendRequest(url, sendOptions, (response) => {
      response.data = JSON.parse(response.data)
      resolve(response)
    }, (response) => {
      var error = {
        message: response.error
      }
      // 打印错误
      console.error(error.message)
      reject(error)
    })
  })
}

export default request
```

这样的修改可以使 request 的调用方法不变，即：

```js
// import request from 'file/path'

request({
  url: '/get/demo',
  method: 'get',
  headers: {
    token: 'SJCU5VFS8C5SC2F'
  },
  params: {
    username: account,
    password: passwd
  }
}).then((response) => {
  // 处理数据
  console.log(JSON.stringify(response))
}).catch((error) => {
  // 处理错误
  console.error(error.message)
})
```

### 注意

修改之后的文件只能在 cordava 的 `deviceready` 事件触发之后调用(否则 `Vue.$cordova 为 undefined`)，所以最好像按下面这样加载 Vue 实例(如果提前调用 `import App from './App'`，上面的代码会被提前加载)

```js
// 加载 Vue 实例
document.addEventListener('deviceready', function () {
  new Vue({
    components: { App: require('./App').default },
    router: require('./router').default,
    store: require('./store').default,
    template: '<App/>'
  }).$mount('#app')
}, false)
```
