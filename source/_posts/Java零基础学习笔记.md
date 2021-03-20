---
title: Java零基础学习笔记
date: 2021-03-04 20:24:29
categories: back-end
tags: [Java]
img: /2021/03/04/Java零基础学习笔记/stepladder.jpg
---

{% asset_img java_waiting_for_you.jpeg java_waiting_for_you %}
*图片来自网络*

> JavaDoc：[官方文档](https://docs.oracle.com/javase/9/docs/api/overview-summary.html)
> 边学边记重点：[阿里云大学Java学习路线](https://edu.aliyun.com/roadmap/java)
> 在线调试工具：[online_java_debugger](https://www.onlinegdb.com/online_java_debugger)

## Java编程入门

- 一个`.class`文件对应一个`public类`
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
      {
        String name = "普通代码块";
      }
    }
    static {
      String name = "静态代码块，优先级最高且只执行一次";
    }
    {
      String name = "构造代码块，优先级高于构造函数，new一个对象执行一次";
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

- 接口、抽象类、枚举、泛型变量

  ```java
  interface IUSB {
    public abstract void plugin() // public abstract 关键字可省略
    public default void normalFunc() {} // default 声明普通方法
  }

  abstract class Device implements IUSB {
    private String name;
    private double price;
    public Device(String name, double price) {
      this.name = name;
      this.price = price;
    }
    public abstract String getInfo();
  }

  enum SupportDevice {
    KEYBOARD, PRINTER,
  }

  class Keyboard extends Device {
    public Keyboard() {
      super("keyboard", 100.0);
    }
    public void plugin() {
      System.out.println("I implements usb interface standard connect with pc.")
    }
    public String getInfo() {
      return "Device name: " + this.name + ", " + "device price: " + this.price;
    }
  }
  ```

- 设计模式（单例模式、工厂模式）

  ```java
  class SingletonStore {
    private static final SingletonStore INSTANCE = new SingletonStore();
    private SingletonStore() {};
    public SingletonStore getInstance() {
      return this.INSTANCE;
    }
  }
  ```

- 内部类、Lamda表达式、函数式接口、Lamda表达式

  ```java
  import java.util.function.*;

  @FunctionalInterface
  interface IFunction<T> {
    public void accept(T t);
  }

  interface IMessage {
    public void send(String s);
    public static class InnerClass implements IMessage {
      public void send(String s) {
        System.out.println(s);
      }
    }
  }

  public class JavaLearn {
    public static void main(String[] args) {
      Consumer<String> console = System.out :: println;
      IFunction<String> func = (s) -> {
        console.accept(s);
      };
      func.accept("learn java function reference");
      IMessage m1 = new IMessage.InnerClass();
      m1.send("learn java inner class");
    }
  }
  ```