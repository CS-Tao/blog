---
title: Vue 中手动触发 window.onresize() 函数
date: 2018-05-19 22:47:25
categories: 技术相关
tags: [Vue]
---

``` JavaScript
// 添加 Vue 静态方法, 在全局通过调用 Vue.triggerResize() 触发
Vue.triggerResize = () => {
  let e = document.createEvent('Event')
  e.initEvent('resize', true, true)
  window.dispatchEvent(e)
}

// 添加 Vue 实例方法, 在 Vue 实例中通过调用 this.$triggerResize() 触发
Vue.prototype.$triggerResize = Vue.triggerResize
```
