---
title: Vue响应式原理及手动实现一个简易的MVVM
date: 2019-08-04 15:36:39
categories: font-end
tags: [vue, mvvm]
img: /2019/08/04/Vue响应式原理及手动实现一个简易的MVVM/vue.png
---

## 前置知识点：
### MVVM、MVC思想：
- 了解web前端发展历史有助于理解MVVM、MVC思想，**历史发展总是朝着不断优化代码组织结构、易维护、封装复用的方向。揭开面纱，一切还是基于浏览器提供的API进行DOM操作。事实证明为项目开发效率的提升带来了重大意义。**
{% blockquote 廖雪峰 https://www.liaoxuefeng.com/wiki/1022910821149312/1108898947791072 MVVM %}
使用MVVM框架，只要改变JavaScript对象的状态，就会导致DOM结构作出对应的变化！这让我们的关注点从如何操作DOM变成了如何更新JavaScript对象的状态，而操作JavaScript对象比DOM简单多了！
{% endblockquote %}
*推荐阅读：*
- {% link 什么是MVVM，MVC和MVVM的区别，MVVM框架VUE实现原理 https://baijiahao.baidu.com/s?id=1596277899370862119 %}
- {% link mvc和mvvm的区别 https://www.jianshu.com/p/b0aab1ffad93 %}
{% asset_img mvc.webp mvc %}

### JS设计模式（观察者模式、发布-订阅模式）：
- 观察者模式
```js
class Observerable {
  this.eventObj = {};
  on(evName, fn) {
    this.eventObj[evName] = this.eventObj[evName] ? [...this.eventObj[evName], fn] : [fn];
  }
  emit(evName, ...args) {
    this.eventObj[evName].forEach((fn) => {
      fn.apply(null, args);
    });
  }
}

// 实例化一个可观察对象
const event = new Observerable();
event.on('error', () => {
  console.log('error1');
})
event.on('error', () => {
  console.log('error2');
})
event.emit('error');
```
*代码参考自：* {% link Node.js && JavaScript 面试常用的设计模式二 https://github.com/lvwxx/blog/issues/9 %}

- 发布-订阅模式
> 观察者模式与发布-订阅模式核心思想非常类似，观察者模式维护一个`Observerable`对象，发布-订阅模式集合所有`Publisher`和`Subscriber`对象形成`EventHub`信息中心，由信息中心负责通知订阅者，做到代码组织结构优化。

*推荐阅读：* {% link 观察者模式 vs 发布-订阅模式 https://juejin.im/post/5a14e9edf265da4312808d86 %}

### JS原生API：
- [Object.defineProperty()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)
- [js属性描述: getters与setters](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Working_with_Objects#%E5%AE%9A%E4%B9%89_getters_%E4%B8%8E_setters)
- [Document.createDocumentFragment()](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/createDocumentFragment)
- [Node.childNodes|nodeType|textContent|innerHtml|firstChild|firstElementChild](http://www.softwhy.com/article-9271-1.html)

## AngularJs响应式原理：
{% blockquote 程序思维 https://baijiahao.baidu.com/s?id=1596277899370862119 什么是MVVM，MVC和MVVM的区别，MVVM框架VUE实现原理 %}
通过`脏值检测`的方式比对数据是否有变更，来决定是否更新视图。最简单的方式就是通过`setInterval()`定时轮询检测数据变动，当然Google不会这么low。Angular只有在指定的事件触发时进入脏值检测，大致如下：DOM事件(譬如用户输入文本、点击按钮`ng-click`)、XHR响应事件`$http`、浏览器Location变更事件`$location`、Timer事件(`$timeout`、`$interval`)、执行`$digest()`或`$apply()`。在Angular中组件是以树的形式组织起来的，相应地，检测器也是一棵树的形状。当一个异步事件发生时，脏检查会从根组件开始，自上而下对树上的所有子组件进行检查，这种检查方式的性能存在很大问题。
{% endblockquote %}

## Vue响应式原理：
{% asset_img vue.png vue %}
> 通过`Object.defineProperty()`API监听对象的变化，实现操作js对象自动更新DOM（调用预先绑定的回调函数）。由于`Object.defineProperty`是ES5中一个无法shim的特性，这也就是Vue不支持IE8以及更低版本浏览器的原因，而且不能监听到数组的变化（Vue3.0开始使用ES6提供的[`Proxy`](http://es6.ruanyifeng.com/#docs/proxy)）。

*推荐阅读：*
- {% link Vue官网·深入响应式原理 https://cn.vuejs.org/v2/guide/reactivity.html %}
- {% link Vue官网·数组更新检测 https://cn.vuejs.org/v2/guide/list.html#%E6%95%B0%E7%BB%84%E6%9B%B4%E6%96%B0%E6%A3%80%E6%B5%8B %}

## 手动实现一个简易的MVVM：
> Vue响应式原理简单明了，难点在于Vue内部做了很多事情来实现双向绑定、性能优化等，通过算法模型减少DOM树更新和最大范围的重用。

- 实现Vue构造函数
```js
// Vue 构造函数
function Vue(options = {}) {
  this.$options = options;
  this._data = this.$options.data;
  // 实例化一个发布者（可观察对象）
  new Publisher(this._data);
  // 实现vm实例对data对象中的属性直接读取和赋值
  for (let key in this._data) {
    Object.defineProperty(this, key, {
      enumerable: true,
      configurable: false,
      get() {
        return this._data[key];
      },
      set(newValue) {
        this._data[key] = newValue;
      }
    });
  }
  new Compile(this.$options.el, this);
}
```

- 实现Compile简单编译DOM
```js
// Compile 简单编译DOM
function Compile(el, vm) {
  vm.$el = document.querySelector(el);
  const fragment = document.createDocumentFragment();
  let child = null;
  while(child = vm.$el.firstChild) {
    fragment.appendChild(child); // vm.$el的子元素被转移到fragment
  }
  function replace(fragment) {
    const pattern = /\{\{(.*)\}\}/;
    Array.from(fragment.childNodes).forEach((node) => {
      let text = node.textContent;
      if (node.nodeType === 3 && pattern.test(text)) {
        const key = RegExp.$1.trim();
        // 为每一个{{ xxx }}节点实例化一个订阅者，订阅data对象中的属性值变化
        // 利用闭包，将node节点作为私有变量在内存中保存起来
        new Subscriber(vm, key, (newValue) => {
          node.textContent = text.replace(pattern, newValue);
        });
        // vm[key]对data对象取值触发get函数，此时EventHub静态属性target指向Subscriber实例，并被添加到events中
        node.textContent = text.replace(pattern, vm[key])
      }
      if (node.childNodes && node.childNodes.length) {
        replace(node);
      }
    });
  }
  replace(fragment);
  vm.$el.appendChild(fragment);
}
```

- 发布-订阅模式实现响应
```js
// 发布-订阅设计模式
// EventHub信息中心
function EventHub() {
  this.events = [];
}

EventHub.prototype.on = function(evName) {
  this.events.push(evName);
}

EventHub.prototype.notify = function() {
  this.events.forEach((event) => {
    event.update();
  });
}

// Publisher发布
function Publisher(data) {
  const eventHub = new EventHub();
  for (let key in data) {
    let value = data[key];
    Object.defineProperty(data, key, {
      enumerable: true,
      configurable: false,
      get() {
        EventHub.target && eventHub.on(EventHub.target);
        EventHub.target = null;
        return value;
      },
      set(newValue) {
        if (newValue === value) {
          return;
        }
        value = newValue;
        eventHub.notify();
      }
    });
  }
}

// Subscriber订阅
function Subscriber(vm, key, fn) {
  this.vm = vm;
  this.key = key;
  this.fn = fn;
  EventHub.target = this;
}

Subscriber.prototype.update = function () {
  this.fn(this.vm[this.key]);
}
```

- 在发布-订阅模式中`Publisher`相当于`Observerable`可观察对象，传入Vue构造函数options中的data对象通过定义`getters`与`setters`变成可观察对象，任何取值赋值操作都可以发布信息到`EventHub信息中心`，由`EventHub`负责通知订阅者。

*完整代码：* {% link github/simple-mvvm https://github.com/jeffrey-fu/simple-mvvm %}

##  参考文章：
- {% link 这应该是最详细的响应式系统讲解了 https://juejin.im/post/5d26e368e51d4577407b1dd7 %}
- {% link 实现一个简单的MVVM https://www.jianshu.com/p/c7bf5092a0a6 %}
- {% link github/nano-mvc https://github.com/doodlewind/nano-mvc %}
- {% link JS设计模式漫谈 https://github.com/slashhuang/blog/blob/master/essays/js/design-patterns.md %}