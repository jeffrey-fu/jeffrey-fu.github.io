---
title: js原型、原型链、继承相关学习心得
date: 2019-06-20 11:24:22
categories: font-end
tags: [JavaScript, 原型, 继承]
img: /2019/06/20/js原型、原型链、继承相关学习心得/es5-inherit_min.png
---

## 前置知识点：
- js基本数据类型和引用数据类型（基本数据类型有： `null`, `undefined`, `Boolen`, `Number`, `String`, `Symbol`）
- 计算机堆、栈内存（栈内存存的是基本类型和引用类型的地址，堆内存由于树状结构可以延伸适合引用类型如`Array`, `Object`）
- js中`this`指向及`bind`，`call`，`apply`方法使用，参阅[call、apply和bind的实现](https://github.com/Abiel1024/blog/issues/16)

## 引生思考点
- js语言原型和原型链为什么这么设计，好处与不好之处，与Java语言最大的区别是什么？
- 浅拷贝与深拷贝如何实现？
- js闭包知识点

## js原型、原型链基本概念
### ES5 和 ES6 继承的区别
> ES5原型链继承

{% asset_img es5-inherit.png es5-inherit %}

```js
function Super() {
  this.xxx = 'hello world!';
}

Super.prototype.sayHello = function() {
  console.log(this.xxx);
}

function Son() {
  // ...
}

Son.prototype = new Super(); // 原型链式继承
// Son.prototype = Object.create(Super.prototype);

Son.prototype.constructor = Son; // 修复constructor

var sonInstance = new Son();
console.log(sonInstance.xxx); // 'hello world!';

```
特点：
- 非常纯粹的继承关系，实例是子类的实例，也是父类的实例`instanceof`方法通过层层查找原型链判断的

缺点：
- 来自原型对象的所有属性被所有实例共享
- 创建子类实例时，无法向父类构造函数传参

> ES6继承

{% asset_img es6-inherit.png es6-inherit %}

```js
class Super {
  constructor() {
    this.xxx = 'hello world!';
  }

  sayHello() {
    console.log(this.xxx);
  }
}

class Son extends Super {
  // ...
}

var sonInstance = new Son();
console.log(sonInstance.xxx); // 'hello world!';
```
- ES6继承和原型链式继承还是有本质区别的，ES6中 `Son` 类实例是自己本身拥有了xxx属性的，因为在子类的`constructor`构造函数中运行`super()`方法调用了父类构造函数（[ES5的调用父类构造函数继承](#调用父类构造函数继承)）
- ES6中方法都是不可枚举的，`Son` 类实例调用 `sayHello` 方法和原型式继承原理类似
- ES5中构造函数是function函数本身，而ES6中构造函数是 `constructor` 函数
- 关于ES6 `class` 可以重开一篇博客了, 先推荐阅读两篇文章
  - {% link ES6 javascript中class静态方法、属性与实例属性用法示例 https://www.jb51.net/article/127066.htm %}
  - {% link react组件中的constructor和super小知识 https://www.cnblogs.com/faith3/p/9219446.html %}

{% link 以上图片引用自keenwon.com http://keenwon.com/1524.html 目前无法访问了... %}

#### 实现ES6的`class`语法（2019-06-26·补充）

```js
function inherit(subType, superType) {
  subType.prototype = Object.create(superType.prototype, {
    constructor: {
      enumerable: false,
      configurable: true,
      writable: true,
      value: subType,
    }
  });

  Object.setPrototypeOf(subType, superType);
}
```

{% blockquote yeyan1996 https://juejin.im/post/5cef46226fb9a07eaf2b7516#heading-9 一个合格的中级前端工程师必须要掌握的28个JavaScript技巧 %}
- ES6 的 class 内部是基于寄生组合式继承，它是目前最理想的继承方式，通过 Object.create 方法创造一个空对象，并将这个空对象继承 Object.create 方法的参数，再让子类（subType）的原型对象等于这个空对象，就可以实现子类实例的原型等于这个空对象，而这个空对象的原型又等于父类原型对象（superType.prototype）的继承关系
- 而 Object.create 支持第二个参数，即给生成的空对象定义属性和属性描述符/访问器描述符，我们可以给这个空对象定义一个 constructor 属性更加符合默认的继承行为，同时它是不可枚举的内部属性（enumerable:false）
- 而 ES6 的 class 允许子类继承父类的静态方法和静态属性，而普通的寄生组合式继承只能做到实例与实例之间的继承，对于类与类之间的继承需要额外定义方法，这里使用 Object.setPrototypeOf 将 superType 设置为 subType 的原型，从而能够从父类中继承静态方法和静态属性
{% endblockquote %}

### ES5实现继承的其它方式

#### 调用父类构造函数继承
```js
function Super(xxx) {
  this.xxx = xxx;
}

Super.prototype.sayHello = function() {
  console.log(this.xxx);
}

function Son() {
  Super.call(this, 'hello world!'); // 调用父类构造函数
}

var sonInstance = new Son();
console.log(sonInstance.xxx); // 'hello world!';
sonInstance.sayHello(); // TypeError: sonInstance.sayHello is not a function
```
特点：
- 创建子类实例时，可以向父类传递参数

缺点：
- 实例并不是父类的实例，只是子类的实例
- 只能继承父类的实例属性和方法，不能继承原型属性/方法
- 无法实现函数复用，每个子类都有父类实例函数的副本，影响性能

#### 组合继承
```js
function Super(xxx) {
  this.xxx = xxx;
}

Super.prototype.sayHello = function() {
  console.log(this.xxx);
}

function Son() {
  Super.call(this, 'hello world!'); // 调用父类构造函数
}

Son.prototype = new Super();

Son.prototype.constructor = Son; // 修复constructor

var sonInstance = new Son();
console.log(sonInstance.xxx); // 'hello world!';
sonInstance.sayHello(); // 'hello world!';
```
特点：
- 创建子类实例时，可以向父类传递参数
- 既是子类的实例，也是父类的实例
- 实例拥有与父类同名的自己的属性（亦是缺点）
- 既能继承父类的实例属性和方法，又能继承原型属性/方法

缺点：
- 调用了两次父类构造函数，子类实例上属性将子类原型上的那份屏蔽了

#### 寄生组合继承
```js
function Super(xxx) {
  this.xxx = xxx;
}

function Son(xxx) {
  Super.call(this, xxx); // 调用父类构造函数
}

// 创建一个空方法，避免再次调用父类的构造函数
(function () {
  var Temp = function() {};
  Temp.prototype = Super.prototype;
  Son.prototype = new Temp();
})();

Son.prototype.constructor = Son; // 修复constructor

var sonInstance = new Son('hello world!');
console.log(sonInstance.xxx); // 'hello world!';
```
特点：
- 结合以上所有继承方式的优点

## 完整的原型图

{% asset_img inherit.png inherit %}

##  参考文章：
- {% link 完整原型链详细图解（构造函数、原型、实例化对象） https://blog.csdn.net/spicyboiledfish/article/details/71123162 %}
- {% link JS实现继承的几种方式 https://www.cnblogs.com/humin/p/4556820.html %}
- {% link 详解ES5和ES6的继承 https://www.cnblogs.com/annika/p/9073572.html %}