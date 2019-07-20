---
title: js隐式类型转换学习总结
date: 2019-06-29 10:49:45
categories: font-end
tags: [JavaScript, 类型转换]
img: /2019/06/29/js隐式类型转换学习总结/object.jpeg
---

> 为什么会写这篇博客，因为自己类型转换规则实在记不住，直接先看**[练习题](#练习题)**，都没问题就直接pass。

## 前置知识点：
- js基本数据类型比较的是值是否相等
- js引用数据类型比较的是引用的地址是否相等
- `==`比较会进行类型转换，`===`必须类型相同值也相等

## `==`比较类型转换规则：

{% blockquote 刘小夕 https://juejin.im/post/5cab0c45f265da2513734390#heading-5 【面试篇】寒冬求职季之你必须要懂的原生JS(上) %}
{% endblockquote %}
- 首先排除特殊的`null`，`undefined`类型：
  ```js
  null ==  undefined; // true
  0 == undefined; // false
  0 == null; // false
  ```
- 比较双方的类型相同，则进行值的判断：
  ```js
  [] ==  []; // false（类型相同但引用的地址不同）
  ```
- 比较双方的类型不同，进行类型转换，规则如下：
  - 若其中一方为`object`且另一方为`string`、`number`或者`symbol`, 将`object`转为基本类型再进行判断
    ```js
    [] ==  ''; // true
    [] ==  0; // true
    [null] == 0; // true
    [undefined] == 0; // true
    ```
  - 若其中一方为`boolean`, 将`boolean`转为`number`再进行判断
    ```js
    true ==  1; // true
    false == 0; // true
    ```
  - 若双方类型为`string`和`number`, 将`string`转换成`number`
    ```js
    '12' ==  12; // true
    ```

## `object`转换成基本类型规则：

`object`这种复杂类型转换成基本类型时，会调用对象的`valueOf`和`toString`方法。

```js
{}.toString(); // '[object Object]'
[1,2,3].valueOf(); // [1,2,3]
[1,2,3].toString(); // '1,2,3'
Object.prototype.toString.call([1,2,3]); // '[object Array]' (可判断是否为数组)
```

### `valueOf`和`toString`方法调用顺序：
{% blockquote 刘小夕 https://juejin.im/post/5cea6e5fe51d45775e33f4de#heading-15 【Step-By-Step】高频面试题深入解析 / 周刊01 %}
1. 若显示定义了`[Symbol.toPrimitive]`属性，只调用该接口，若返回的不是基本数据类型，抛出错误。
2. 默认的调用规则：
  - 非`Date`类型对象，`hint`是`default`时，调用顺序为：`valueOf` >>> `toString`，即`valueOf`返回的不是基本数据类型，才会继续调用`toString`，如果`toString`返回的还不是基本数据类型，那么抛出错误。
  - 如果`hint`是`string`(Date对象默人的hint是string) ，调用顺序为：`toString` >>> `valueOf`，即`toString`返回的不是基本数据类型，才会继续调用`valueOf`，如果`valueOf`返回的还不是基本数据类型，那么抛出错误。
  - 如果`hint`是`number`，调用顺序为：`valueOf` >>> `toString`。
{% endblockquote %}

```js
(new Date()).toString(); // 'Sun Jun 30 2019 16:53:46 GMT+0800 (中国标准时间)'
(new Date()).valueOf(); // 1561884790571
```

> 关于`hint`, `[Symbol.toPrimitive]`示例可访问上文引用链接
> 关于`[Symbol.toPrimitive]`可参考：- {% link 阮一峰·ECMAScript 6 入门 http://es6.ruanyifeng.com/#docs/symbol#Symbol-toPrimitive %}

## `string`转换成`number`规则：

### `Number()`和`parseInt()`方法
```js
Number('12abc'); // NaN
Number(undefined); // NaN
Number(null); // 0
parseInt('12abc'); // 12
```
### `parseInt()`经典面试题

{% blockquote alentan https://juejin.im/post/5d0202da51882546dd10087b 为什么 ['1', '7', '11'].map(parseInt) 返回 [1, NaN, 3] %}
{% endblockquote %}

### `String()`和`toString()`方法
```js
undefined.toString(); // TypeError: Cannot read property 'toString' of undefined
null.toString(); // TypeError: Cannot read property 'toString' of null
String(undefined); // 'undefined'
String(null); // 'null'
```
`radix`为2时，NumberObject会被转换为二进制值表示的字符串
```js
var num = 13;
console.log(num.toString(2));
```

## `typeof`判断类型：

`typeof`方法返回值：`undefined`, `boolean`, `number`, `string`, `symbol`, `function`, `object`

```js
typeof null; // 'object'
typeof [1,2,3]; // 'object'
typeof new Boolean(true); // 'object'
typeof new Number(1); // 'object'
typeof new String("abc"); // 'object'
typeof NaN; // 'number'
typeof (typeof 1); // 'string'
typeof undefined; // 'undefined'
```

## 练习题

1. `[]`开辟了一块堆内存，地址存在栈内存，`![]`返回`false`
```js
if ([]) {
  ... // true
}
```

2. 比较双方的类型相同，则进行值的判断，栈内存地址不同
```js
[] == []; // false
{} == {}; // false
```

3. 解题参考：- {% link 【面试篇】寒冬求职季之你必须要懂的原生JS(上) https://juejin.im/post/5cab0c45f265da2513734390#heading-6 %}
```js
[] == ![] // true
{} == !{} // false
{} == '[object Object]' // true
```

疑惑：文中最后一步`Number([])`确实为0，但是和[`object`转换成基本类型规则：](#object转换成基本类型规则：)似乎有冲突。

4. 运算符进行隐式类型转换
```js
'12' + 1; // '121'（String）
1 + 'true'; // '1true'
1 + true; // 2
1 + undefined; // NaN
1 + null; // 1
'12' * 1; // 12（Number）
'12' - 1; // 11（Number）
'a' - 1; // NaN
2 > '10'; //false
'2' > '10'; // true
'abc' > 'b'; // false
'abc' > 'aad'; // true
[] + {}; // '[object Object]'
[] + []; // ''
```

- 字符串连接符`+`：只要`+`号两边有一边是字符串
- 算术运算符`+`：`+`号两边都是数字

5. 特殊类型比较`null`, `undefined`, `NaN`的比较
```js
undefined == null; // true
NaN == NaN; // false
NaN == null; // false
```

6. [Can (a== 1 && a ==2 && a==3) ever evaluate to true?](https://stackoverflow.com/questions/48270127/can-a-1-a-2-a-3-ever-evaluate-to-true)
```js
if (a == 1 && a == 2 && a == 3) {
  // ...
}
```

解答：
- 利用数组的`toString`会隐含调用`Array.join`方法

```js
a = [1,2,3];
a.join = a.shift;
console.log(a == 1 && a == 2 && a == 3);
```

- 重写`Object`的`toString`或者`valueOf`

```js
const a = {
  i: 1,
  toString: function () {
    return a.i++;
  }
}
console.log(a == 1 && a == 2 && a == 3);
```

## 参考文章：

- {% link 【面试篇】寒冬求职季之你必须要懂的原生JS(上) https://juejin.im/post/5cab0c45f265da2513734390 %}
- {% link 【Step-By-Step】高频面试题深入解析 / 周刊01 https://juejin.im/post/5cea6e5fe51d45775e33f4de %}
- {% link [译]送你43道JavaScript面试题 https://juejin.im/post/5d0644976fb9a07ed064b0ca %}
- {% link 老码农的字节跳动前端面试总结（2面已凉） https://zhuanlan.zhihu.com/p/68974750 %}
- {% link MDN·typeof https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/typeof %}
- {% link Js 中那些 隐式转换 https://www.cnblogs.com/ihboy/p/6700059.html %}
- {% link js面试题大坑——隐式类型转换 https://blog.csdn.net/itcast_cn/article/details/82887895 %}
- {% link 有意思的JavaScript面试题：如何让(a ==1 && a== 2 && a==3) 的值为true https://majing.io/posts/10000006051204 %}