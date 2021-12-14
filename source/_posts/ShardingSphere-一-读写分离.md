---
title: ShardingSphere(一)读写分离
catalog: true
date: 2020-11-16 08:46:01
subtitle: ShardingSphere
header-img: "/blog/img/header_img/404.png"
tags:
- java
---
## ShardingSphere(一)读写分离
### 一.读写分离架构图
![在这里插入图片描述](/blog/img/shardingjdbc/shardingjdbc1.PNG)
### 二.数据库主从配置
[mysql主从设置](https://xshcloudy.gitee.io/blog/uncategorized/mysql%E6%95%B0%E6%8D%AE%E5%BA%93%E4%B8%BB%E4%BB%8E%E8%AE%BE%E7%BD%AE/ "mysql主从设置")
### 三.ShardingSphere配置
`1.引入依赖`
```java
  <dependency>
      <groupId>org.apache.shardingsphere</groupId>
      <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
      <version>4.0.0-RC1</version>
    </dependency>
    <dependency>
      <groupId>org.apache.shardingsphere</groupId>
      <artifactId>sharding-jdbc-spring-namespace</artifactId>
      <version>4.0.0-RC1</version>
    </dependency>
```
`2.yaml配置`
```json
spring:
  shardingsphere:
    enabled: true
    datasource:
      names: master,slave
      master:
        username: root
        password: 111111
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost:3306/sharding?useUnicoe=true&characterEncoding=utf-8&serverTimezone=GMT
        type: com.alibaba.druid.pool.DruidDataSource
      slave:
        username: root
        password: 111111
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost:3307/sharding?useUnicoe=true&characterEncoding=utf-8&serverTimezone=GMT
        type: com.alibaba.druid.pool.DruidDataSource
    masterslave:
      load-balance-algorithm-type: round_robin
      name: master_slave
      master-data-source-name: master
      slave-data-source-names: slave
#    encrypt:
#      encryptors:
#        aes_encryptor:
#          type: aes
#          qualifiedColumns: user.PASS_WORD
#          props:
#            aes.key.value: qwwerwefrwe
    props:
      #query.with.cipher.column: true
      sql:
        show: true
```
### 四.验证
`1.插入数据`

------------
```
下图console日志可看到数据源是master，数据是插入master数据库，然后同步到slave数据库。
```
![在这里插入图片描述](/blog/img/shardingjdbc/shardingSphere3.png)
**master数据库**
![在这里插入图片描述](/blog/img/shardingjdbc/shardingSphere1.png)
**slave数据库**
![在这里插入图片描述](/blog/img/shardingjdbc/shardingSphere2.png)

`2.查询数据`

------------

![在这里插入图片描述](/blog/img/shardingjdbc/shardingSphere4.png)
```
console日志可看到查询是用的slave数据源。
```
`3.删除和修改同理，这里省略`

------------


### 五.结论
```json
shardingSphere默认master数据库执行delete,update,insert,slave数据库执行select，见读写分离架构图，
读写分离适合数据量不是很多的情况，通过读写分离，扩展从库，来缓解查询的压力。
```
### 六.范例代码
    https://gitee.com/xshCloudy/sharding.git
	




