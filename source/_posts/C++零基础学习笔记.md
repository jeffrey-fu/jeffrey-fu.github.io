---
title: C++零基础学习笔记
date: 2021-06-26 12:24:29
categories: back-end
tags: [C++]
img: /2021/06/26/C++零基础学习笔记/C++.jpeg
---

{% asset_img C++.jpeg C++ %}
*图片来自网络*

> 语法基础：[面向对象设计C++ 翁恺](https://www.bilibili.com/video/BV1yQ4y1A7ts)
> 语法进阶：[C++面向对象高级开发 候捷](https://www.bilibili.com/video/BV1yt41157uB)
> 语法基础：[C++标准库(STL)与泛型编程 候捷](https://www.bilibili.com/video/BV1BX4y1G7bX)

## C++编译运行

```bash
  $ g++ a.cpp -o a.out -save-temps
  $ ./a.out
```

`-save-temps`: 编译`.ii` -> 汇编`.s` -> 链接 -> 二进制文件`.out`

## C++内存模型

> 关键词：变量、指针、引用、全局数据区、堆栈、堆内存

- C++中分配内存跟对象初始化分开，Java一定是用过`new`关键字初始化对象后才能获取内存地址。

- C++全局变量在程序运行，`main`函数之前被初始化，多个`.cpp`文件的全局变量初始化是无序的，Java没有全局变量。

- JavaScript、Java等OOP语言中基本类型值传递，引用类型是引用传递

  ```javaScript
  // 值传递
  const num = 1;
  function testRef(num) {
    num ++;
  }
  testRef(num);
  console.log(num); // 1
  // 引用传递
  const objA = { a: 1 };
  function testRef(obj) {
    obj.a = 2;
  }
  testRef(objA);
  console.log(objA.a); // 2
  ```

- C++中默认拷贝一个新的对象，加上`&`表示引用，通常使用`const &`限制直接修改被引用的值

  ```c++
  #include <iostream>
  using namespace std;

  class Person {
    public:
      string name;
      int age;
    public:
      Person();
      void print();
  }; // if missing semicolon, compiler throw error

  Person::Person(): name("jeffrey"), age(30) {};
  void Person::print() {
    cout << "My name is " << this->name << endl;
    cout << this->age << " years old" <<  endl;
  }

  void testRef1(Person p) {
    p.age = 31;
  }

  void testRef2(Person& p) {
    p.age = 32;
  }

  void testRef3(int& num) {
    num++;
  }

  int main() {
    Person p1;
    int num = 1;
    p1.print(); // age 30
    testRef1(p1);
    p1.print(); // age 30
    testRef2(p1);
    p1.print(); // age 32
    testRef3(num);
    cout << num << endl; // num 2
  }
  ```

- 指针和引用

  ```c++
  #include <iostream>
  using namespace std;

  class Person {
    public:
      int age;
      Person(int &a): age(a) { cout << "Person::Person()" << endl; }
      ~Person() { cout << "Person::~Person()" << endl; }
  };

  int main() {
    int a = 20;
    int *p = &a;
    int &r = a;
    Person person(r);
    Person *person_pointer = (Person *)&person;
    cout << person_pointer->age << endl;
    cout << person_pointer << endl;
    cout << p << endl;
    cout << *p << endl;
    cout << &p << endl;
    cout << r << endl;
    cout << &r << endl;
    return 0;
  }
  ```

## C++面向对象编程(封装、继承、多态)

- 一个类的public部分构成了`interface`
- C++中子类与父类同名函数不构成`override`，任何其它OOP语言都不这么干

  ```c++
  #include <iostream>
  using namespace std;

  class Person {
    private:
      string name;
      int age;
    public:
      Person(string n, int a): name(n), age(a) {}
      string getName() { return this->name; }
      void print() { cout << "My name: " << name << ", My age: " << age << endl; }
      void print(string x) { cout << x << endl; print(); }
  };

  class Student : public Person {
    private:
      string school;
    public:
      Student(string n, int a, string s): Person(n, a), school(s) {}
      void print() { cout << "My name: " << getName() << ", My school: " << school << endl; }
  };

  int main() {
    Student s("jeffrey", 30, "zju");
    // s.print("x"); error hidden
    s.print();
    s.Person::print("x");
  }
  ```

## C++模板、运算符重载

- C++模板（泛型）
  ```c++
  #include <iostream>
  using namespace std;

  template <class T>
  T add(T num1, T num2) {
    return num1 + num2;
  }

  int r1 = add(1, 2);
  float r2 = add(1.1, 2.2);

  cout << r1 << r2 << endl;
  ```

## 疑问点
- [拷贝构造II](https://www.bilibili.com/video/BV1yQ4y1A7ts?p=27) 5:00
  `private`是针对类而不是针对对象
- [模板II](https://www.bilibili.com/video/BV1yQ4y1A7ts?p=35) 15:00
  不同`.cpp`文件根据模板生成的对象是否是同一种类型`A<T>`