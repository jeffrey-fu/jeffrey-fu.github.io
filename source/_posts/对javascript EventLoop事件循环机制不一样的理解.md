---
title: 对javascript EventLoop事件循环机制不一样的理解
date: 2019-07-23 18:58:36
categories: font-end
tags: [JavaScript, 事件循环机制]
img: /2019/07/23/对javascript EventLoop事件循环机制不一样的理解/execute_stack.jpg
---

## 前置知识点：

- 浏览器原理，浏览器内核5种线程及协作，JS引擎单线程设计
  *推荐阅读：*
  - {% link 从浏览器多进程到JS单线程，JS运行机制最全面的一次梳理 https://segmentfault.com/a/1190000012925872 %}
  - {% link 【FE】浏览器渲染引擎「内核」 https://github.com/zwwill/blog/issues/2 %}
- js异步编程，Promise实现
  *推荐阅读：*
  - {% link Javascript异步编程的4种方法 http://www.ruanyifeng.com/blog/2012/12/asynchronous%EF%BC%BFjavascript.html %}
  - {% post_link 前端面试必考题Promise的源码解析 %}
- 堆、栈、队列、执行栈、任务、微任务、事件循环机制♻️
  *推荐阅读：*
  - {% link JavaScript异步编程-基础篇 https://juejin.im/post/5c6a033f5188254e5e61960a %}
  - {% link 彻底搞懂浏览器Event-loop https://juejin.im/post/5c947bca5188257de704121d %}
  - {% link 这一次，彻底弄懂 JavaScript 执行机制 https://juejin.im/post/59e85eebf265da430d571f89 %}
  - {% link 一次弄懂Event Loop（彻底解决此类面试问题） https://juejin.im/post/5c3d8956e51d4511dc72c200 %}

## 事件循环机制

> 相信读者读完以上推荐的文章后，已经知道`事件循环机制`是怎么一回事了吧，也能从容应对面试。接下来我要谈谈自己的理解：

### 为什么会有事件循环机制

- js设计之初就是单线程模式，代码也都是顺序执行，当遇到因为大量计算、http请求等需要额外的等待时间时，浏览器用户就会体验到卡顿了，所以所有的设计和改进初衷只有一个就是**要快**。

### 事件循环机制的产生

- 浏览器说我的内核是多线程，可以辅助JS引擎线程啊，[**Web Worker**](http://www.ruanyifeng.com/blog/2018/07/web-worker.html)线程提供大量计算辅助（不能操作DOM），**事件触发线程**，**定时触发器线程**， **异步http请求线程**。
{% asset_img brower.png brower %}
{% asset_img render.jpg render %}

- 执行栈（先进后出），由JS引擎线程控制，引用下面这个例子谈谈自己的理解：
{% asset_img execute_stack2.gif execute_stack2 %}
```js
console.log('script start');

setTimeout(function() {
  console.log('setTimeout');
}, 0);

Promise.resolve().then(function() {
  console.log('promise1');
}).then(function() {
  console.log('promise2');
});
console.log('script end');

// "script start"
// "script end"
// "promise1"
// "promise2"
// "setTimeout"
```

- 在`ES5`还没有`Promise`时代，异步回调很常见，上面例子中，通过解读Promise源码（{% post_link 前端面试必考题Promise的源码解析 %}），我们可以把`Promise`转换成如下图式回调（*个人理解，文章中的Promise源码也只是模拟，大部分浏览器已经原生支持*）。
  1. 打印完`script start`, `script end`主执行栈出栈，如果`Promise.resolve().then`换成`new Promise(executor)`，脑补`Promise`换成回调函数，那么这个函数一执行，`executor`函数也就执行了，然后遇到异步回调，回调函数被**其它对应的线程接手，启动观察者模式，完成后回调函数被推入事件任务队列，等待执行栈空了进入主线程执行**
{% asset_img callback.jpg callback %}

  2. 以上这种在异步函数中放同步函数的例子，为了合理解释输出顺序而推出了`microtasks`微任务的概念，请看下面的例子，脑补`Promise`换成回调函数，**`Promise.prototype.then`内部执行了`return new Promise()`，js引擎在捕捉到`Promise`时，放到了由js引擎自身控制的微任务队列等待执行，也就造成promise1、2、3、4错开打印**
```js
console.log('script start');

new Promise(function(resolve, reject) {
  console.log('promise1');
  resolve();
}).then(function() {
  console.log('promise2');
});

new Promise(function(resolve, reject) {
  console.log('promise3');
  resolve();
}).then(function() {
  console.log('promise4');
});

console.log('script end');

// "script start"
// "promise1"
// "promise3"
// "script end"
// "promise2"
// "promise4"

```

  3. `microtasks`微任务的概念完全为了解释异步函数中放同步函数的场景，**而且各类文章和面试都是这种题目和例子**，在实际开发过程中，**你会在`Promise`中这么写么？**，在我看来这种比较打印顺序太过于理论，而且可能会混乱你的思绪。就像下面的例子，`Promise`的`resolve`决定了Promise状态，就像在回调函数中满足了条件才会继续执行，例子中只是用`setTimeout`模拟异步请求，用之前的理论你可能觉得`setTimeout`被放入了事件任务队列，那没有`resolve`的`Promise`怎么解释呢？（*放到微任务里一直阻碍第一个`setTimeout`宏任务执行吗？显然是不可能的，这不是跟设计事件循环机制初衷冲突了么*）
```js
setTimeout(function() {
  console.log('setTimeout');
}, 0);

new Promise(function(resolve, reject) {
  setTimeout(function() { // 模拟异步请求
    console.log('promise1');
    resolve();
  }, 0);
}).then(function() {
  console.log('promise2');
});

new Promise(function(resolve, reject) {
  // resolve(); 注释掉resolve，使Promise一直处于‘pending’状态
}).then(function() {
  console.log('promise2');
});

// "setTimeout"
// "promise1"
// "promise2"

```

  4. 个人认为把`Promise`，`async/await`脑补成原始的回调函数（模拟源码中模拟异步是用的`setTimeout`函数），而js引擎捕捉到`setTimeout`, `setInterval`就转给**定时触发器线程**处理，捕捉到`XMLHttpReuqest`, `fetch`就转给**异步http请求线程**，跟**事件触发线程**一起管理着事件任务队列，**微任务的概念可以看作是当事件触发线程遇到几乎同时需要把回调函数放到事件任务队列时，`Promise`内部的异步标识函数优先级高于`setTimeout`函数吧**，以上例子中没有执行`resolve`的`Promise`状态一直处于'pending'，**事件触发线程**压根没有放入到事件任务队列，**总之浏览器会安排的妥妥的，不要打架，虽然js引擎线程只有一个（听我指挥排好队，咱们这都是同步代码执行ms级别，我开了很多其它线程处理需要等待的代码了）。**以下例子模拟所谓的几乎同时把回调函数放到事件任务队列，记得把`Promise`脑补成原始的回调函数。仿佛回到了没有微任务的时代。
  {% asset_img execute_stack.jpg execute_stack %}
```js
setTimeout(function() {
  console.log('setTimeout');
}, 0);

new Promise(function(resolve, reject) {
  setTimeout(function() { // 模拟异步请求
    console.log('promise1');
    resolve();
  }, 2000);
}).then(function() {
  console.log('promise2');
});

new Promise(function(resolve, reject) {
  setTimeout(function() { // 模拟异步请求
    console.log('promise3');
    resolve();
  }, 2000);
}).then(function() {
  console.log('promise4');
});

// "setTimeout"
// "promise1"
// "promise2"
// "promise3"
// "promise4"

```

*以上内容纯属未深入了解js情况下的个人理解，感觉是在努力摒弃微任务的概念，回归ES5回调函数时代，便于自身理解事件循环机制而做出的遐想。*

### 多环境下的事件循环机制

> 在node环境、浏览器环境以及各个不同版本下js引擎处理的方式还不太一样。node比浏览器还复杂些😢（以后学Node.js时再看吧）

*推荐阅读：*

- {% link NodeJS的Event Loop https://juejin.im/post/5c3d8956e51d4511dc72c200#heading-25 %}
- {% link 又被node的eventloop坑了，这次是node的锅 https://juejin.im/post/5c3e8d90f265da614274218a %}
- {% link 关于Chrome浏览器73以下版本和73版本下`async/await`区别 https://juejin.im/post/5c3d8956e51d4511dc72c200#heading-19 %}

##  参考文章：
- {% link 更快的异步函数和 Promise https://v8.js.cn/blog/fast-async/ %}
- {% link Tasks, microtasks, queues and schedules https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/ %}