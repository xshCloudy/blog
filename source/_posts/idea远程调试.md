---
title: idea远程调试
catalog: true
date: 2018-10-15 14:25:56
subtitle: idea远程调试
header-img: "/blog/img/header_img/car.jpg"
tags:
- java
---
### 一.前言
```javascript
	远程连接服务器调试是一个程序员基础必备技能，特别是在开发，测试，回归的调试过程中很有用处，
	因为一个bug在不同环境可能呈现不同的结果。

```
### 二.tomcat配置修改
```javascript
	tomcat修改配置开放调试端口给idea远程连接，下面分别介绍以jar包和war包部署时tomcat怎么开放调试端口

```
#### 1.jar包部署

```javascript
	只需要在nohup java -jar 后面加上-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=9999，
	例如：
	nohup java -jar -Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=9999  /home/gp2-web.jar 
	--spring.profiles.active=${ENV} --server.port=${PORT}  &
	9999就是暴露的调试端口
```
#### 2.war包部署

```javascript
	找到tomcat bin目录下的catalina.sh文件，将JPDA_ADDRESS="localhost:8000"修改成JPDA_ADDRESS="0.0.0.0:9999"，
	0.0.0.0代表所有远程调试端都可以访问，也可以单独指定一个ip，9999就是暴露的调试端口，
	然后shutdown掉tomcat，用tomcat bin目录下的catalina.sh jpda start命令重新启动tomcat即可。
```

### 二.idea远程调试的使用
![在这里插入图片描述](/blog/img/all/remote_debug1.PNG)
![在这里插入图片描述](/blog/img/all/remote_debug2.PNG)
```javascript
	如上面两张图，只需要idea new一个remote连接,修改remote配置上的远程服务器ip和端口，
	然后启动remote，可以看到console上显示已经连上远程服务器。
	这个时候就可以在idea上打断点调试了。

```