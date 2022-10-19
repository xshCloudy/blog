---
title: JVM类加载机制
catalog: true
date: 2021-02-19 15:13:03
subtitle:
header-img:
tags:
---

## 一.类的加载
```java
Java源代码经过编译器生成字节码文件（.class），字节码文件与平台无关，不面向任何具体平台，只面向虚拟机。
类的加载是指JVM把编译后的.class类文件的二进制数据读取到内存中，并根据它创建一个Class对象，
存放在堆中。

```
## 二.类的生命周期
![在这里插入图片描述](/blog/img/jvm_classloader/1.jpg)
```java
包括：加载，连接，初始化，使用，卸载。这里我们重点讨论的是加载机制。

```

## 三.类加载结构
![在这里插入图片描述](/blog/img/jvm_classloader/2.jpg)
```java
类的加载过程中有三个自带的类加载器和一个自定义类加载器
```

#### Bootstrap ClassLoader：
```java
基于C/C++实现，负责加载Java的核心类库JAVA_HOME\jre\lib\rt.jar，
该加载器不继承自ClassLoader抽象类，并且只加载包名为java、javax、sun等开头类，
起到对核心源码的保护作用。
```

#### Extension ClassLoader：
![在这里插入图片描述](/blog/img/jvm_classloader/3.jpg)
```java
基于Java语言，由sun.misc.Launcher$ExtClassLoader实现，派生于ClassLoader抽象类，
从java.ext.dirs系统变量指定的路径中的加载类库，或者JDK安装目录jre\lib\ext目录下加载
```

#### Application ClassLoader：
![在这里插入图片描述](/blog/img/jvm_classloader/4.jpg)
```java
基于Java语言，由sun.misc.Launcher$AppClassLoader实现，它负责加载环境变量ClassPath指定的类库，
如果在应用程序中没有自定义类加载器，一般情况下作为程序中默认的类加载器。
```

#### 自定义 ClassLoader：
```java
通过继承java.lang.ClassLoader实现自定义的类加载器。
```
## 四.双亲委派
![在这里插入图片描述](/blog/img/jvm_classloader/5.jpg)
#### 机制：
```java
当一个类收到了类加载请求，类加载会由最底下的自定义加载器开始，将加载请求委派给上级，
如果上级已经加载过该类，则直接加载完成，如果没有则继续递归至顶级Bootstrap加载器，
然后父类判断如果该类不属于自己的加载范畴则委派子类加载。继续往下递归。

```

#### 目的：
```java
安全：
自己写的java.lang.String.class类不会被加载(包名和类名和jdk自带的一样 运行不成功)，
这样便可以防止核心API库被随意篡改(自己在自定义类加载器里面要加载java.lang....
类似于和核心包里面系统的包名称 jvm直接报一个安全异常) 
jvm加载到和自己核心类库里面相同的类必须用自己类库里面的类。

避免重复加载：
当父亲已经加载了该类时，就没有必要子ClassLoader再加载一次(父类就直接返回了)，
保证被加载类的唯一性(相同包名类名只加载一份)。

```
#### 缺陷
```java
双亲模型固然有着优点，能够让整个系统保持了类的唯一性。但在有些场合，却不适合。
顶层的启动类加载器的代码无法访问到底层的类加载器。如 rt.jar 无法中代码无法访问到应用类加载器。

你肯定要问，为什么需要访问呢？
在 Java 平台中，把核心类（rt.jar)中提供外部服务，可由应用层自行实现的接口，

比如rt.jar 中的抽象类需要加载继承他们的在应用层的子类实现，目前的双亲机制是无法实现的。
因此 JDK 引用了一个不太优雅的设计，上下文类加载器。也就是讲类加载放在线程上下文变量中。
通过 Thread.getContextClassLoader()， Thread.setContextClassLoader(ClassLoader) 
这两个方法获取和设置 ClassLoader，这样，rt.jar 中的代码就可以获取到底层的类加载了。

```

