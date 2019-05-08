---
title: JS 中 Function 和 Object 的关系
date: 2019-04-04 21:55:45
categories: 技术相关
tags: [JacaScript]
reward: true
---
<img src="https://raw.githubusercontent.com/CS-Tao/github-content/master/contents/blog/image/others/javascript.png" width="65%" height="65%">

> js 的世界很大，并不是只有对象

这段时间在复习前端基础知识，遇到了下面这道面试题：
```js
var F = function(){}
var f = new F()
Object.prototype.a = function(){}
Function.prototype.b = function(){}

console.log(f.a)
console.log(f.b)
```
请问这段代码会输出什么？
<!-- more -->
打开浏览器控制台，将上面这段代码复制进去，回车，我们可以得到它的答案：
```js
ƒ (){}
undefined
```
细看我们可以发现这道题并不难，对于`f.a`，JavaScript 解释器会按照原型链的规则去查找 f 的 a 属性：
1. 在 f 本身查找是否有 a 属性，显然没有，接下来将会进入 f 的原型链查找；
1. 查找`f.__proto__`是否有 a 属性，而`f.__proto__ === F.prototype`，显然也没有；
1. 继续查找`f.__proto__.__proto__`，即`F.prototype.__proto__`，而`F.prototype.__proto__ === Object.prototype`，可以发现`Object.prototype` 中有 a 属性。

对于`f.b`，解释器会进行和上面一样的操作，但并不会在原型链中找到 b 属性， 因为`f.__proto__.__proto__.__proto__ === Object.prototype.__proto__ === null`，结束查找

那么代码中的`Function.prototype.b = function(){}`能有什么作用呢？不难发现：
```js
console.log(F.b) // ƒ (){}
// 甚至还有
console.log(Object.b) // ƒ (){}
console.log(Object.b === F.b) // true
```
看起来 F 和 Object 都是 Function 的实例？的确是这样的，所有的函数都是由 Function 的原型构造出来的，这么说 Function 是 JavaScript 的一个很基础的工具咯？但 JavaScript 不是万物皆对象吗，那 Object 和 Function 到底是什么关系呢？

我们可以做以下测试：
```js
F instanceof Object // true
F instanceof Function // true
Object instanceof Function // true
Function instanceof Object // true
Object == Function // false
```
完全懵了...

为什么会出现这么怪异的结果，Object 和 Function 谁更底层一些？它们为什么会"互为实例"？

通过对 JavaScript 各个概念出现先后顺序的分析，我得到了如下的一张图：

![Js_Function_Object.png](https://raw.githubusercontent.com/CS-Tao/github-content/master/contents/blog/image/others/Js_Function_Object.png)

毫无疑问 JavaScript 是我学过的最糟糕的语言...

首先解释一下这张图：
1. 在 JavaScript 中，对象原型是最最基础的概念，通过对象原型得到了函数原型；
1. 利用函数原型，可以构造出 Object()、Function() 函数，这个构造过程和其它函数的构造过程基本一致，但应该没有使用 new 运算符；
1. 得到 Object 和 Function 后，便可以将对象原型和函数原型分别挂载到 Object 和 Function 的 prototype 上了；
1. 利用装备上 prototype 的 Object 和 Function，便可以实现 new 运算。道生一，一生二，二生三，三生万物...

结合该图和 instanceof 的判断规则，我们可以解释上面的结果：

先附上 instanceof 判断规则的伪代码：
```js
function instance_of(L, R) { // L 表示左表达式，R 表示右表达式
  if (isBasicType(L)) return false; // 如果是基础类型直接返回 false
  var O = R.prototype; // 取 R 的显示原型
  L = L.__proto__; // 取 L 的隐式原型
  while (true) { 
    if (L === null) 
      return false; 
    if (O === L)
      return true; 
    L = L.__proto__; 
  } 
}
```
结果解释：
```js
F instanceof Object // true, F.__proto__.__proto__ === '函数原型'.__proto__ === '对象原型' === Object.prototype
F instanceof Function // true, F.__proto__ === '函数原型' === Function.prototype
Object instanceof Function // true, Object.__proto__ === '函数原型' === Function.prototype
Function instanceof Object // true, Function.__proto__.__proto__ === '函数原型'.__proto__ === '对象原型' === Object.prototype
```
我们还可以从上图发现其它一些有趣的信息：
```js
Function.prototype === Function.__proto__ // true
Object.prototype === Object.__proto__.__proto__ // true
```
好了到此为止，我不该入门前端的...

---
2019.05.08 更新

引用自 [高能！typeof Function.prototype 引发的先有 Function 还是先有 Object 的探讨](https://segmentfault.com/a/1190000005754797)，看来函数原型并不是由对象原型构造的，下面是浏览器或 Node 在初始化 JavaScript 环境的详细过程：

> 1. 用 C/C++ 构造内部数据结构创建一个 OP 即(Object.prototype)以及初始化其内部属性但不包括行为。
> 1. 用 C/C++ 构造内部数据结构创建一个 FP 即(Function.prototype)以及初始化其内部属性但不包括行为。
> 1. 将 FP 的[[Prototype]]指向 OP。
> 1. 用 C/C++ 构造内部数据结构创建各种内置引用类型。
> 1. 将各内置引用类型的[[Prototype]]指向 FP。
> 1. 将 Function 的 prototype 指向 FP。
> 1. 将 Object 的 prototype 指向 OP。
> 1. 用 Function 实例化出 OP，FP，以及 Object 的行为并挂载。
> 1. 用 Object 实例化出除 Object 以及 Function 的其他内置引用类型的 prototype 属性对象。
> 1. 用 Function 实例化出除Object 以及 Function 的其他内置引用类型的 prototype 属性对象的行为并挂载。
> 1. 实例化内置对象 Math 以及 Grobal。至此，所有内置类型构建完成。
