---
title: Java零基础学习笔记
date: 2021-03-04 20:24:29
categories: back-end
tags: [Java]
img: /2021/03/04/Java零基础学习笔记/stepladder.jpg
---

{% asset_img java_waiting_for_you.jpg java_waiting_for_you %}
*图片来自网络*

> 边学边记重点：[阿里云大学Java学习路线](https://edu.aliyun.com/roadmap/java)
> 在线调试工具：[online_java_debugger](https://www.onlinegdb.com/online_java_debugger)

## Java编程入门

- 一个`.class`文件对应一个`类`
- `int`型数据范围 (-2147483648 ~ 2147483647)
  ```java
  int num = Integer.MAX_VALUE + 2L // -2147483647
  ```
- `char`类型用`''`，`String`类型用`""`
## Java面向对象编程

- 如何正确声明一个类并进行实例化后调用
  ```java
  class Person {  // 声明一个Person类
    private String name; // 属性封装
    private static String country; // 类静态属性，可通过类或其实例对象调用(有别于js)
    public Person() { // 类构造方法，没有返回值，没有void以区别普通函数
      // this()必须在构造函数内调用，且优先于其它代码执行
      this("invoke construct fun with param");
    }
    public Person(String name) { // 类构造方法重载
      this.name = name; // this指向实例化对象
    }
     // 类静态方法，无法使用this，无法调用非静态属性及方法
     // 因为静态的属性和方法可直接通过类调用，无需实例化对象
    public static void setCountry(String c) {
      country = c;
    }
    public String getName() { // getter
      return this.name;
    }
    public void setName(String name) { // setter 修改属性值
      this.name = name;
    }
  }

  public class Main {
    public static void main(String[] args) {
      Person p1 = new Person("jeff");
      p1.setName("jeffrey");
      Person.setCountry("中国");
      System.out.println("name: " + p1.getName());
    }
  }
  ```