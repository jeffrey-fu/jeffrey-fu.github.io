---
title: js执行上下文、词法作用域、块级作用域相关学习
date: 2019-06-26 18:07:21
categories: font-end
tags: [JavaScript, 编译, 作用域]
img: /2019/06/26/js执行上下文、词法作用域、块级作用域相关学习/code_execute_environment.jpg
---

## JavaScript引擎

JavaScript引擎的一个流行示例是Google的V8引擎。例如，在Chrome和Node.js中使用V8引擎，下面是一个非常简化的视图：

{% asset_img call_stack.png 执行栈 %}

{% blockquote lvwxx https://github.com/lvwxx/blog/issues/2 JavaScript中的执行上下文和堆栈是什么 %}
浏览器中的JavaScript解释器是作为一个单线程实现的，这实际上意味着，在浏览器中，一次只能发生一件事，其他操作或事件将排队在所谓的执行堆栈中。
{% endblockquote %}

## 词法作用域

{% asset_img 词法作用域.jpg 词法作用域 %}

## 函数作用域和块级作用域

{% asset_img 函数作用域和块级作用域.jpg 函数作用域和块级作用域 %}

### 块级作用域与函数声明

{% blockquote 阮一峰 http://es6.ruanyifeng.com/#docs/let#%E5%9D%97%E7%BA%A7%E4%BD%9C%E7%94%A8%E5%9F%9F%E4%B8%8E%E5%87%BD%E6%95%B0%E5%A3%B0%E6%98%8E 块级作用域 %}
- ES5 只有全局作用域和函数作用域，没有块级作用域。
- ES5 规定，函数只能在顶层作用域和函数作用域之中声明，不能在块级作用域声明。但是，浏览器没有遵守这个规定，为了兼容以前的旧代码，还是支持在块级作用域之中声明函数。
- ES6 引入了块级作用域，明确允许在块级作用域之中声明函数。ES6 规定，块级作用域之中，函数声明语句的行为类似于let，在块级作用域之外不可引用。
- 为了减轻因此产生的不兼容问题，ES6 在附录 B里面规定，浏览器的实现可以不遵守上面的规定，有自己的行为方式。
  - 允许在块级作用域内声明函数。
  - 函数声明类似于var，即会提升到全局作用域或函数作用域的头部。
  - 同时，函数声明还会提升到所在的块级作用域的头部。
{% endblockquote %}

> 感受下`let`和`var`的区别

```js
var aa = 'bb';
function fn(){
   console.log(aa); 
  { // 在块级作用域内var声明变量
    var aa = 'aa';
  }
}
fn(); // undefined
```

```js
var aa = 'bb';
function fn(){
   console.log(aa);
  { // 在块级作用域内let声明变量
    let aa = 'aa';
  }
}
fn(); // 'bb'
```

```js
function bs(){
  console.log(1);
}
function fn(){
  bs();
  { // 在块级作用域内声明函数
    function bs(){
      console.log(2);
    }
  }
}
fn(); // TypeError: bs is not a function
```

## 变量提升
### 暂时性死区

{% blockquote yeyan1996 https://juejin.im/post/5c6234f16fb9a049a81fcca5 近一万字的ES6语法知识点补充 %}
使用let/const声明的变量，从一开始就形成了封闭作用域，在声明变量之前是无法使用这个变量的，这个特点也是为了弥补var的缺陷（var声明的变量有变量提升）
{% endblockquote %}

```js
console.log(aa); // ReferenceError: Cannot access 'aa' before initialization）
let aa;
```

{% blockquote yeyan1996 https://juejin.im/post/5c6234f16fb9a049a81fcca5 近一万字的ES6语法知识点补充 %}
剖析暂时性死区的原理，其实let/const同样也有提升的作用，但是和var的区别在于：
- var在创建时就被初始化，并且赋值为undefined
- let/const在进入块级作用域后，会因为提升的原因先创建，但不会被初始化，直到声明语句执行的时候才被初始化，初始化的时候如果使用let声明的变量没有赋值，则会默认赋值为undefined，而const必须在初始化的时候赋值。而创建到初始化之间的代码片段就形成了暂时性死区
{% endblockquote %}

### ES6 `Class` 不存在变量提升

```js
console.log(Test); // ReferenceError: Cannot access 'Test' before initialization
class Test {
  // ...
}
```

## 解释器如何评估JS代码（evaluate）

{% blockquote lvwxx https://github.com/lvwxx/blog/issues/2 JavaScript中的执行上下文和堆栈是什么 %}
1. 扫描被调用函数中的代码
2. 在代码执行前，创建执行上文
3. 进入创建阶段
  - 初始化作用域链
  - 创建变量对象
  - 创建arguments对象，检查参数上下文，初始化名称和值，并创建引用副本
  - 扫描上下文中函数的声明
    - 对于找到的每个函数，在变量对象中创建一个属性，该属性是确切的函数名，该函数在内存中有一个指向该函数的引用指针
    - 如果函数名已经存在，指针将会被覆盖
  - 扫描变量的声明
    - 对于找到的每个变量，在变量对象中创建一个属性，该属性是确切的变量名，该变量的值是undefined
    - 如果变量名已经存在，将不会做任何处理继续执行
  - 决定this的值
4. 代码执行阶段
  - 变量赋值，按顺序执行代码
{% endblockquote %}

结合自身的感悟如下：
- 知道了js代码**执行前**有预编译这么一回事，也就能掌握变量提升，解释器根据`var`, `function`等关键字进行预编译，`function`是一等公民
  ```js
  (function() {
  console.log(foo); // foo() { return 'hello'; }
  console.log(bar); // undefined

  var bar = function() {
    return 'world';
  };

  function foo() {
    return 'hello';
  }

  var foo = 'hello';

  }());​
  ```
- 词法作用域决定了函数执行上下文作用域，跟函数在哪调用没有关系（`this`在任何情况下都不指向函数的词法作用域，`this`是在函数运行时，创建上下文的时候确定的）
  ```js
  function a() {
    var myOtherVar = "inside A";
    b();
  }

  function b() {
    var myVar = "inside B";
    console.log("myOtherVar:", myOtherVar); // 'global otherVar'

    function c() {
      console.log("myVar:", myVar); // 'inside B'
    }

    c();
  }

  var myOtherVar = "global otherVar";
  var myVar = "global myVar";
  a();
  ```

## 执行上下文

{% blockquote lvwxx https://github.com/lvwxx/blog/issues/2 JavaScript中的执行上下文和堆栈是什么 %}
执行上下文：当前代码正在执行的环境（作用域），一般有以下三种情况：

- 全局代码 -- 代码首次开始执行的默认环境
- 函数代码 -- 每当进入一个函数内部
- Eval代码 -- eval内部代码执行时
{% endblockquote %}

{% asset_img execution_context.jpg execution_context %}

{% blockquote David Shariff http://davidshariff.com/blog/what-is-the-execution-context-in-javascript/ what-is-the-execution-context-in-javascript %}

- Nothing special is going on here, we have 1 `global context` represented by the purple border and 3 different `function contexts` represented by the green, blue and orange borders. There can only ever be 1 `global context`, which can be accessed from any other context in your program.

- You can have any number of `function contexts`, and each function call creates a new context, which creates a private scope where anything declared inside of the function can not be directly accessed from outside the current function scope. In the example above, a function can access a variable declared outside of its current context, but an outside context can not access the variables / functions declared inside.（理解为作用域链）
{% endblockquote %}

{% blockquote lvwxx https://github.com/lvwxx/blog/issues/2 JavaScript中的执行上下文和堆栈是什么 %}
现在我们知道每当有函数被调用时，都会创建一个新的执行上下文。在js内部，每个执行上文创建都要经历下面2个阶段：

1. 创建阶段（函数被调用，但还没有执行内部代码）
  - 创建作用域链
  - 创建变量和参数
  - 决定this指向
2. 代码执行阶段
  - 变量赋值，执行代码
{% endblockquote %}

## this 全面解析

{% asset_img this.jpg this %}

> *推荐阅读：[刘小夕](https://juejin.im/user/5c6256596fb9a049bd42c770) - {% link 嗨，你真的懂this吗？ https://juejin.im/post/5c96d0c751882511c832ff7b %}*

## 闭包

{% asset_img 闭包.jpg 闭包 %}

##  参考文章：
- {% link JavaScript中的执行上下文和堆栈是什么 https://github.com/lvwxx/blog/issues/2 %}
- {% link JavaScript是如何工作的：引擎，运行时和调用堆栈的概述！ https://github.com/qq449245884/xiaozhi/issues/1 %}
- {% link JavaScript 究竟是怎样执行的？ https://zhuanlan.zhihu.com/p/70458440 %}
- {% link 精读《你不知道的 javascript（上卷）》 https://zhuanlan.zhihu.com/p/37421835 %}
- {% link 精读《你不知道的 javascript（中卷）》 https://zhuanlan.zhihu.com/p/38287143 %}
- {% link What is the Execution Context & Stack in JavaScript? http://davidshariff.com/blog/what-is-the-execution-context-in-javascript/ %}
- {% link 你不可不知道的 JavaScript 作用域和闭包 https://zhuanlan.zhihu.com/p/28921273 %}
- {% link JS函数作用域以及变量提升问题 https://segmentfault.com/q/1010000016823953 %}
- {% link 块级作用域（阮一峰·ECMAScript 6 入门） http://es6.ruanyifeng.com/#docs/let#%E5%9D%97%E7%BA%A7%E4%BD%9C%E7%94%A8%E5%9F%9F %}