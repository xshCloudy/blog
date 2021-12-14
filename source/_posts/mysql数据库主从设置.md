---
title: mysql数据库主从设置
catalog: true
date: 2020-11-05 14:23:41
subtitle: shardingjdbc
header-img: "/blog/img/header_img/404.png"
tags:
- java
---
## mysql数据库主从设置
### 一.主从同步原理
![在这里插入图片描述](/blog/img/shardingjdbc/mysql0.PNG)

### 二.主从同步设置
##### 1.安装两个mysql
##### 2.配置主数据库my.ini
`新建my.ini（windows是my.ini，linux是my.cnf），放在mysql根目录下`
```json
[mysql]
 
# 设置mysql客户端默认字符集
default-character-set=utf8 
[mysqld]
#设置3306端口
port = 3306
# 设置mysql的安装目录
basedir=E:\mysql\mysql-5.7.31-0
# 设置mysql数据库的数据的存放目录（目录记得更改）
#datadir=E:\mysql\mysql-5.7.31-0\data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 跳过登录密码验证
#skip-grant-tables
server-id=1 #唯一ID
log_bin=sharding_log_bin  #开启及设置二进制日志文件名称
max_binlog_size=500M
sync_binlog = 0
binlog_cache_size=128K  #binlog缓存大小
binlog-do-db=sharding  #要同步的数据库 
binlog-ignore-db=mysql  #不需要同步的数据库
binlog_ignore_db=performation_schema
#log-slave-updates=1  #A>B>C实现三级同步开关
#expire_logs_day=2  #二进制日志自动删除的天数。默认值为0，表示不自动删除。
binlog_format=MIXED 
```

##### 3.重启主数据库
##### 4.主数据库创建授权用户
```java
CREATE USER 'slave'@'@' IDENTIFIED BY '123456';
GRANT REPLICATION SLAVE ON *.* TO slave@'%' IDENTIFIED BY '123456';
FLUSH PRIVILEGES;

select user,host from mysql.user;
```
##### 5.查看主数据库log_bin是否开启（需要开启log_bin）
```java
show variables like 'log_bin';
```
##### 6.查看主数据库master状态
```java
show master status;
# 如下图file（log_bin文件名）和position（log_bin文件位置，会变化）的值在从数据库的配置中会用到
```
![在这里插入图片描述](/blog/img/shardingjdbc/mysql1.PNG)

##### 7.配置主数据库my.ini
```json
[mysql]
 
# 设置mysql客户端默认字符集
default-character-set=utf8 
[mysqld]
#设置3307端口（和主数据库区分开）
port = 3307
# 设置mysql的安装目录
basedir=E:\mysql\mysql-5.7.31-1
# 设置mysql数据库的数据的存放目录（目录记得更改）
#datadir=E:\mysql\mysql-5.7.31-0\data
# 允许最大连接数
max_connections=200
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 跳过登录密码验证
#skip-grant-tables
server-id=2
```
##### 8.重启从数据库
##### 9.关闭从数据库slave开关
```java
stop slave;
```
##### 10.从数据库配置主数据库信息
```java
CHANGE MASTER TO MASTER_HOST = '127.0.0.1',
MASTER_USER = 'slave',
MASTER_PASSWORD = '123456',
MASTER_LOG_FILE = 'sharding_log_bin.000002',
MASTER_LOG_POS = 3315;
```
##### 11.开启从数据库slave开关
```java
start slave;
```
##### 12.查看从数据库slave状态
```java
SHOW SLAVE STATUS;
#如下图只有【Slave_IO_Running】和【Slave_SQL_Running】都是Yes，则同步正常。
```
![在这里插入图片描述](/blog/img/shardingjdbc/mysql2.PNG)
##### 13.查看mysql-error.log
```java
show variables like 'log_error%';
#如果主从同步异常，可执行该命令，如下图，DESKTOP-BOMJA8F.err中可查看异常日志
```
![在这里插入图片描述](/blog/img/shardingjdbc/mysql3.PNG)
