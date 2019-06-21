---
title: 前端面试必考题Promise的源码解析
date: 2019-06-18 17:55:45
categories: font-end
tags: [JavaScript, Promise]
img: /2019/06/18/前端面试必考题Promise的源码解析/promise_min.jpg
---

{% asset_img promise.jpg promise %}

## 考察知识点：
- js迭代器
- js构造函数、原型、原型链
- js执行栈、事件循环机制

## 伪源码实现：

### Promise构造函数
- 创建一个`Promise`实例，通过`new`一个`Promise`（接受一个函数作为参数），该函数接受`resolve`, `reject`两个方法作为参数。
- 每个`Promise`只有三种状态：`pending`，`fulfilled` 和 `rejected`，状态只能从 `pending` 转移到 `fulfilled` 或者 `rejected`，一旦状态变成`fulfilled` 或者 `rejected`，就不能再更改其状态。

```js
function Promise3(executor) {
  const self = this;
  self.status = 'pending';
  self.value = undefined;
  self.reason = undefined;
  self.onResolvedCallbacks = [];
  self.onRejectedCallbacks = [];

  function resolve(value) {
    if (self.status === 'pending') {
      self.status = 'resolved';
      self.value = value;
      self.onResolvedCallbacks.forEach(fn => fn());
    }
  }

  function reject(reason) {
    if (self.status === 'pending') {
      self.status = 'rejected';
      self.reason = reason;
      self.onRejectedCallbacks.forEach(fn => fn());
    }
  }

  try {
    executor(resolve, reject);
  } catch (e) {
    reject(e);
  }
}
```

### 定义在Promise原型上方法`then`，可被实例调用
- 通过判断上一个Promise的状态执行不同的代码
- 正常情况下第一个Promise执行器中是异步代码，会先执行then()函数，then函数两个参数`onFulfilled`，`onRejected`会被暂存到`onResolvedCallbacks`，`onRejectedCallbacks`中，在第一步的异步执行器`executor(resolve, reject)`中完成执行
- 为了实现Promise的链式调用，在`then`方法中`return promise`，所以在`new Promise`的执行器中手动加了`setTimeout`实现异步执行

```js
Promise3.prototype.then = function (onFulfilled, onRejected) {
  onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
  onRejected = typeof onRejected === 'function' ? onRejected : err => { throw err };
  const self = this;
  let promise2;
  if (self.status === 'resolved') {
    promise2 = new Promise3(function(resolve, reject) {
      setTimeout(function () {
        try {
          let x = onFulfilled(self.value);
          resolvePromise(promise2, x, resolve, reject);
        } catch (e) {
          reject(e);
        }
      });
    });
  }
  if (self.status === 'rejected') {
    promise2 = new Promise3(function(resolve, reject) {
      setTimeout(function () {
        try {
          let x = onRejected(self.reason);
          resolvePromise(promise2, x, resolve, reject);
        } catch (e) {
          reject(e);
        }
      });
    });
  }
  if (self.status === 'pending') {
    promise2 = new Promise3(function(resolve, reject) {
      self.onResolvedCallbacks.push(function () {
        setTimeout(function() {
          try {
            let x = onFulfilled(self.value);
            resolvePromise(promise2, x, resolve, reject);
          } catch (e) {
            onRejected(e);
          }
        });
      });
      self.onRejectedCallbacks.push(function () {
        setTimeout(function() {
          try {
            let x = onRejected(self.reason);
            resolvePromise(promise2, x, resolve, reject);
          } catch (e) {
            onRejected(e);
          }
        });
      });
    });
  }
  return promise2;
}
```

### 定义`resolvePromise`方法
- 上一个Promise返回值有3中情况（也是该函数取名的缘由）
  1. 普通值
  2. promise对象
  3. thenable对象/函数
- 通过自身迭代和在`then`方法中的引用，直至返回`resolve`普通值或者报错

```js
function resolvePromise(promise2, x, resolve, reject) {
  if (promise2 === x) {
    return reject(new TypeError('Chaining cycle')); // 循环引用报错
  }

  let called;
  if (x && (typeof x === 'object' || typeof x === 'function')) {
    try {
      let then = x.then;
      if (typeof then === 'function') {
        then.call(x, function(y) {
          if (called) return;
          called = true;
          resolvePromise(promise2, y, resolve, reject);
        }, function(err) {
          if (called) return;
          called = true;
          reject(err);
        });
      } else {
        resolve(x);
      }
    } catch (e) {
      if (called) return;
      called = true;
      reject(e);
    }
  } else {
    resolve(x);
  }
}
```
## Promise的静态方法及其它方法：
### Promise.resolve

```js
Promise.resolve = function (param) {
  if (param instanceof Promise) {
    return param;
  }
  return new Promise((resolve, reject) => {
    if (param && param.then && typeof param.then === 'function') {
      setTimeout(() => {
        param.then(resolve, reject);
      });
    } else {
      resolve(param);
    }
  });
}
```
### Promise.reject

```js
Promise.reject = function (reason) {
  return new Promise((resolve, reject) => {
    reject(reason);
  });
}
```

### Promise.prototype.catch

```js
Promise.prototype.catch = function (onRejected) {
  return this.then(null, onRejected);
}
```

### Promise.prototype.finally

```js
Promise.prototype.catch = function (callback) {
  return this.then((value) => {
    return Promise.resolve(callback()).then(() => {
      return value;
    });
  }, (err) => {
    return Promise.resolve(callback()).then(() => {
      throw err;
    });
  });
}
```

### Promise.all

```js
Promise.all = function (promises) {
  return new Promise((resolve, reject) => {
    let index = 0;
    let result = [];
    if (promises.length === 0) {
      resolve(result);
    } else {
      function processValue(i, data) {
        result[i] = data;
        if (++index === promises.length) {
          resolve(result);
        }
      }
      promises.forEach((promise, i) => {
        Promise.resolve(promise).then((data) => {
          processValue(i, data);
        }, (err) => {
          reject(err);
          return;
        });
      });
    }
  });
}
```

### Promise.race

```js
Promise.race = function (promises) {
  return new Promise((resolve, reject) => {
    if (promises.length === 0) {
      return;
    } else {
      promises.forEach((promise, i) => {
        Promise.resolve(promise).then((data) => {
          resolve(data);
          return;
        }, (err) => {
          reject(err);
          return;
        });
      });
    }
  });
}
```

## [promise测试库](https://github.com/promises-aplus/promises-tests)

```bash
npm install promises-aplus-tests -D
npx promises-aplus-tests promise.js
```

##  参考文章：
- {% link Promise/A+ 规范文档 https://promisesaplus.com/ %}
- {% link Promise详解与实现（Promise/A+规范） https://www.jianshu.com/p/459a856c476f %}
- {% link 手写实现满足 Promise/A+ 规范的 Promise https://www.jianshu.com/p/8d5c3a9e6181 %}
- {% link Promise的源码实现（完美符合Promise/A+规范） https://www.cnblogs.com/zhouyangla/p/10781697.html %}
- {% link 手动实现一个满足promises-aplus-tests的Promise https://juejin.im/post/5aab286a6fb9a028d3752621 %}