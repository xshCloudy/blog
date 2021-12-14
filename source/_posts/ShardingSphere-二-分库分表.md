---
title: ShardingSphere(二)分库分表
catalog: true
date: 2020-11-17 10:29:29
subtitle: ShardingSphere
header-img: "/blog/img/header_img/404.png"
tags:
- java
---
## ShardingSphere(二)分库分表
### 一.分库分表架构图
![在这里插入图片描述](/blog/img/shardingjdbc/shardingSphere5.png)
### 二.ShardingSphere配置
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
      names: ds0,ds1
      ds0:
        username: root
        password: 111111
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost:3306/datasource0?useUnicoe=true&characterEncoding=utf-8&serverTimezone=GMT
        type: com.alibaba.druid.pool.DruidDataSource
      ds1:
        username: root
        password: 111111
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost:3306/datasource1?useUnicoe=true&characterEncoding=utf-8&serverTimezone=GMT
        type: com.alibaba.druid.pool.DruidDataSource
    sharding:
      #分库
      default-database-strategy:
        inline:
          sharding-column: AGE
          algorithm-expression: ds$->{AGE % 2}
      #分表
      tables:
        user:
          actual-data-nodes: ds$->{0..1}.user$->{0..1}
		  #雪花算法生成ID
          key-generator:
            column: ID
            type: SNOWFLAKE  #UUID
          table-strategy:
            inline:
              sharding-column: ID
              algorithm-expression: user$->{ID % 2}
    props:
      query.with.cipher.column: true
      sql:
        show: true


```
### 三.验证
`1.插入数据`

------------


```
下面四张图可以看出插入的数据，根据age字段分库，根据id字段分表
```
![在这里插入图片描述](/blog/img/shardingjdbc/shardingSphere6.png)

![在这里插入图片描述](/blog/img/shardingjdbc/shardingSphere7.png)

![在这里插入图片描述](/blog/img/shardingjdbc/shardingSphere8.png)

![在这里插入图片描述](/blog/img/shardingjdbc/shardingSphere9.png)

`2.查询数据`

------------
**queryAll**
![在这里插入图片描述](/blog/img/shardingjdbc/shardingSphere10.png)
**queryById**
![在这里插入图片描述](/blog/img/shardingjdbc/shardingSphere11.png)
```
查询所有数据是查所有表，再总和在一起，当然根据id查询会先根据分片计算在哪个表里，
再直接从两个库的指定表里面查询。
```
`3.删除和修改同理，这里省略`

### 四.结论
```json
shardingSphere分库分表实际上是将数据垂直切分库，再水平分表。通过shardingSphere中间件统一管理这些库和表，
见架构图，由于数据库连接和表存储都有瓶颈，当数据量达到500w以上时，我们就可以考虑使用分库分表，
来缓解查询的压力，缩短查询时间。分库，分表，还是分库分表都使用，需结合实际情况来判断。
```
### 五.主键ID（特别注意）
```json
常用的主键ID生成方案：
	mysql自增主键:分表分库会有ID冲突。
	UUID：太长，索引多占用空间
	雪花算法：
		优点:既可以保证唯一又可以排序，效率高，适合分布式场景下生成唯一ID。
		缺点：强依赖时间，如果时钟回拨，就会生成重复的ID。
	redis自增主键：效率高，适合分布式场景，但需要redis中间件生成。
```
**`建议使用雪花算法或者redis自增生成唯一ID`**


### 六.范例代码
    https://gitee.com/xshCloudy/sharding.git



