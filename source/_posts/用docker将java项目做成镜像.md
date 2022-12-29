---
title: 用docker将java项目做成镜像
catalog: true
date: 2021-03-10 18:44:07
subtitle:
header-img:
tags:
---
## 一.安装docker
```java
linux:
1.uname -r  查看你当前的内核版本，Docker 要求 CentOS 系统的内核版本高于 3.10
2.yum update -y 确保 yum 包更新到最新
3.yum -y install docker  安装docker
4.service docker start & 启动docker
5.systemctl status docker 查看docker状态
6.docker version  查看docker版本
7.docker logs -f --tail=100 [容器名/容器ID] 查看日志
```
```java
windows:
安装docker desktop
<https://hub.docker.com>
<https://blog.csdn.net/qq_39611230/article/details/108641842>

```
## 二.制作docker镜像
`1.打包好一个jar项目demo.jar`

`2.准备好dockerfile文件`
```java
# 设置本镜像需要使用的基础镜像
FROM  openjdk:8-jre 
  
# 把 app.jar 包添加到镜像中
ADD demo.jar /app.jar
 
# 镜像暴露的端口
EXPOSE 8030
 
RUN bash -c 'touch /app.jar'
  
# 容器启动命令
ENTRYPOINT ["java","-jar","/app.jar"]
 
# 设置时区
RUN /bin/cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && echo 'Asia/Shanghai' >/etc/timezone

```

`3.将准备好demo.jar和dockerfile文件放在同一目录下`

`4.在demo.jar目录下执行docker build -t demo . 构建docker镜像`

![在这里插入图片描述](/blog/img/docker/1.jpg)

```java
-t指定构建镜像的名字和版本,demo就是构建成功后镜像的名字，
也可以指定版本如：name:tag
. 是为了让 Docker 到当前本地目录去寻找 Dockerfile 文件
如图，执行docker build命令之后，执行docker images即可看到生成成功的demo镜像

```
## 三.将docker镜像做成容器
```java
docker run --name demo -p 8030:8030 -d demo

--name  #给你启动的容器起个名字，以后可以使用这个名字启动或者停止容器
-p #映射端口，将docker宿主机的8030端口和容器的8030端口进行绑定
-v #挂载文件用的
-d #表示启动的是哪个镜像。

```
![在这里插入图片描述](/blog/img/docker/2.jpg)
```java
如图执行完docker run 命令之后即可用docker ps 查看当前服务器正在运行状态的容器
docker ps -a 查看当前服务器所有状态的容器
```

## 四.进入容器查看日志
![在这里插入图片描述](/blog/img/docker/3.jpg)
```java
linux 进入容器：docker exec -it 容器id  /bin/bash
windows进入容器: winpty docker exec -it nginx-test bash
退出容器:exit 
如上图，即可在容器内看到demo项目的运行日志

也可以在宿主机执行docker logs -f --tail=100 [容器名/容器ID] 查看日志

```

## 五.操作容器和镜像的命令
```java
docker start 【容器名字】 启动容器
docker restart 【容器名字】重启容器
docker stop 【容器名字】关闭容器
docker kill【容器名字】强制关闭容器

docker rm -f 容器id  强制删除容器
docker rmi  镜像名 删除镜像
docker pull 镜像id  下载镜像

docker inspect -f '{{.ID}}' 容器名字names   获取容器长id

docker inspect -f '{{.NetworkSettings.IPAddress}}' 容器名字names   获取容器长ip

docker cp 本地路径 容器长ID:容器路径         将linux文件cp到docker容器

docker export [容器名/容器ID]导出镜像
docker import [容器名/容器ID]

```


