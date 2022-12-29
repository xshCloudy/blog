---
title: Arthas线上监控诊断
catalog: true
date: 2021-03-08 16:44:07
subtitle:
header-img:
tags:
---

## 一.Arthas是什么？
```java
Arthas是Alibaba开源的一款线上监控诊断工具，通过全局视角实时查看应用 load、内存、gc、线程的状态信息，
并能在不修改应用代码的情况下，对业务问题进行诊断，包括查看方法调用的出入参、异常，监测方法执行耗时，
类加载信息等，大大提升线上问题排查效率。

```


## 二.Arthas安装和配置
`1.Arthas server安装`
```java
使用Arthas必须先在服务器安装Arthas server，项目连上Arthas server才能正常使用。
可以直接用Arthas的镜像安装，非常简单，这里省略。
```
`2.引入依赖`
```java
   <dependency>
      <groupId>com.taobao.arthas</groupId>
      <artifactId>arthas-spring-boot-starter</artifactId>
      <version>3.4.6</version>
    </dependency>
    
```
`3.yaml配置`
```json
arthas:
  agent-id: pineapple-demo
  tunnel-server: ws://10.74.162.19:32611/ws
  app-name: pineapple-demo

```
`4.使用`
![在这里插入图片描述](/blog/img/arthas/1.jpg)
![在这里插入图片描述](/blog/img/arthas/2.jpg)
```json
启动项目后，如上图，填入java yaml配置里面的端口和agent-id，点击connect即可
```
## 三.Arthas的使用场景
![在这里插入图片描述](/blog/img/arthas/3.jpg)
```json
输入dashboard即可显示三块数据面板，分别对应JVM线程面板，JVM内存面板，JVM信息面板。
arthas还有很多其他好玩的命令，这里不多做介绍，下面只介绍三种比较常用的场景。
```

`1.分析项目CPU占用过高原因`
![在这里插入图片描述](/blog/img/arthas/4.jpg)
![在这里插入图片描述](/blog/img/arthas/5.jpg)
```json
输入thread显示JVM线程面板，找到%CPU占用最高的线程，这里假设ID为2的线程%CPU占用最高。
输入thread -n 2(线程ID)，即可打印线程2的堆栈信息。根据堆栈信息即可分析出导致CPU高的具体代码行。
```


`2.监控JVM内存信息`

```json
输入memory显示JVM内存面板
```