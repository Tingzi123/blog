---
title: java小结
date: 2019-03-07 18:31:48
tags:
---

------
#### 1. Java值类型与引用类型
![Image text](https://raw.githubusercontent.com/tingzi123/blog/master/_posts/picture/javavalue.png)

#### 2. Java运算符优先级
![Image text](https://raw.githubusercontent.com/tingzi123/blog/master/_posts/picture/operator.jpg)

#### 3. Java中代码块加载顺序：静态初始代码块>初始代码块>构造函数
例：
```java
public class Main {

    public static void main(String[] args) {
        System.out.println("A");
        new Main();
        new Main();
    }

    public Main() {
        System.out.println("B");
    }

    {
        System.out.println("C");
    }

    static {
        System.out.println("D");
    }
}

以上程序输出的结果，正确的是？
DACBCB
```