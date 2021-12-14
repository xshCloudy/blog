---
title: 分布式事务seata(二)springboot+seata+dubbo集成（AT模式）
catalog: true
date: 2020-12-04 10:53:25
subtitle: seata
header-img: "/blog/img/header_img/404.png"
tags:
- seata
---
## 分布式事务seata(二)springboot+seata+dubbo集成（AT模式）
### 一.配置
`1.maven依赖`
```java
	<dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>${mybatis.version}</version>
    </dependency>  
	<dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.21</version>
    </dependency>
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid-spring-boot-starter</artifactId>
      <version>${druid.version}</version>
    </dependency>
	<dependency>
		<groupId>com.alibaba.spring.boot</groupId>
		<artifactId>dubbo-spring-boot-starter</artifactId>
		<version>2.0.0</version>
	</dependency>
	<dependency>
		<groupId>com.101tec</groupId>
		<artifactId>zkclient</artifactId>
		<version>0.10</version>
		<exclusions>
			<exclusion>
				<groupId>org.slf4j</groupId>
				<artifactId>slf4j-log4j12</artifactId>
			</exclusion>
		</exclusions>
	</dependency>
	<dependency>
		<groupId>io.seata</groupId>
		<artifactId>seata-all</artifactId>
		<version>1.4.0</version>
	</dependency>
```
`2.服务端配置（TC）`

    打开https://github.com/seata/seata/releases，下载seata-server,这里使用的1.4.0版本。
	进入seata/conf目录，目录下有两个关键配置文件：register.conf,file.conf。
	
	register.conf是seata服务端（TC）的注册中心和配置中心的配置文件。
	Seata配置中心支持consul，apollo，etcd,zookeeper,redis,file(直连)，
	seata注册中心支持eureka，consul，apollo，etcd,zookeeper,redis,sofa，file(使用文件做配置中心)。
	
	register.conf中register的type属性是file时，表示不依赖第三方配置注册中心，
	直接用seata-server直连。如果需要搭建高可用体系，建议使用注册中心。这里使用zk做示范。
	
	register.conf中config的type属性是file时，表示使用file.conf做配置中心。
	file.conf中可以指定事务日志存贮方式，有三种存贮方式：file、db、redis。
	指定file时，事务日志会存贮在本地seata/bin/sessionStore/目录下的root.data文件中。
	指定db时，事务日志会存贮在数据库中。指定redis时，使用redis存贮事务日志。这里使用file做示范。

**register.conf**
```java
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "zk"
  loadBalance = "RandomLoadBalance"
  loadBalanceVirtualNodes = 10

  nacos {
    application = "seata-server"
    serverAddr = "127.0.0.1:8848"
    group = "SEATA_GROUP"
    namespace = ""
    cluster = "default"
    username = ""
    password = ""
  }
  eureka {
    serviceUrl = "http://localhost:8761/eureka"
    application = "default"
    weight = "1"
  }
  redis {
    serverAddr = "localhost:6379"
    db = 0
    password = ""
    cluster = "default"
    timeout = 0
  }
  zk {
    cluster = "default"
    serverAddr = "127.0.0.1:2181"
    sessionTimeout = 6000
    connectTimeout = 2000
    username = ""
    password = ""
  }
  consul {
    cluster = "default"
    serverAddr = "127.0.0.1:8500"
  }
  etcd3 {
    cluster = "default"
    serverAddr = "http://localhost:2379"
  }
  sofa {
    serverAddr = "127.0.0.1:9603"
    application = "default"
    region = "DEFAULT_ZONE"
    datacenter = "DefaultDataCenter"
    cluster = "default"
    group = "SEATA_GROUP"
    addressWaitTime = "3000"
  }
  file {
    name = "file.conf"
  }
}

config {
  # file、nacos 、apollo、zk、consul、etcd3
  type = "file"

  nacos {
    serverAddr = "127.0.0.1:8848"
    namespace = ""
    group = "SEATA_GROUP"
    username = ""
    password = ""
  }
  consul {
    serverAddr = "127.0.0.1:8500"
  }
  apollo {
    appId = "seata-server"
    apolloMeta = "http://192.168.1.204:8801"
    namespace = "application"
    apolloAccesskeySecret = ""
  }
  zk {
    serverAddr = "127.0.0.1:2181"
    sessionTimeout = 6000
    connectTimeout = 2000
    username = ""
    password = ""
  }
  etcd3 {
    serverAddr = "http://localhost:2379"
  }
  file {
    name = "file.conf"
  }
}

```
**file.conf**
```java
## transaction log store, only used in seata-server
store {
  ## store mode: file、db、redis
  mode = "file"

  ## file store property
  file {
    ## store location dir
    dir = "sessionStore"
    # branch session size , if exceeded first try compress lockkey, still exceeded throws exceptions
    maxBranchSessionSize = 16384
    # globe session size , if exceeded throws exceptions
    maxGlobalSessionSize = 512
    # file buffer size , if exceeded allocate new buffer
    fileWriteBufferCacheSize = 16384
    # when recover batch read size
    sessionReloadReadSize = 100
    # async, sync
    flushDiskMode = async
  }

  ## database store property
  db {
    ## the implement of javax.sql.DataSource, such as DruidDataSource(druid)/BasicDataSource(dbcp)/HikariDataSource(hikari) etc.
    datasource = "druid"
    ## mysql/oracle/postgresql/h2/oceanbase etc.
    dbType = "mysql"
    driverClassName = "com.mysql.jdbc.Driver"
    url = "jdbc:mysql://127.0.0.1:3306/seata"
    user = "root"
    password = "111111"
    minConn = 5
    maxConn = 100
    globalTable = "global_table"
    branchTable = "branch_table"
    lockTable = "lock_table"
    queryLimit = 100
    maxWait = 5000
  }

  ## redis store property
  redis {
    host = "127.0.0.1"
    port = "6379"
    password = ""
    database = "0"
    minConn = 1
    maxConn = 10
    maxTotal = 100
    queryLimit = 100
  }

}

```

`2.客户端配置（TM,RM）`
```
在项目classpath路径下新建register.conf,file.conf文件，这里的文件和服务端TC的register.conf,file.conf文件同理。
```
**register.conf**
```java
registry {
  # file 、nacos 、eureka、redis、zk、consul、etcd3、sofa
  type = "zk"

  nacos {
    application = "seata-server"
    serverAddr = "localhost"
    namespace = ""
    username = ""
    password = ""
  }
  eureka {
    serviceUrl = "http://localhost:8761/eureka"
    weight = "1"
  }
  redis {
    serverAddr = "localhost:6379"
    db = "0"
    password = ""
    timeout = "0"
  }
  zk {
    serverAddr = "127.0.0.1:2181"
    sessionTimeout = 6000
    connectTimeout = 2000
    username = ""
    password = ""
  }
  consul {
    serverAddr = "127.0.0.1:8500"
  }
  etcd3 {
    serverAddr = "http://localhost:2379"
  }
  sofa {
    serverAddr = "127.0.0.1:9603"
    region = "DEFAULT_ZONE"
    datacenter = "DefaultDataCenter"
    group = "SEATA_GROUP"
    addressWaitTime = "3000"
  }
  file {
    name = "file.conf"
  }
}

config {
  # file、nacos 、apollo、zk、consul、etcd3、springCloudConfig
  type = "file"

  nacos {
    serverAddr = "localhost"
    namespace = ""
    group = "SEATA_GROUP"
    username = ""
    password = ""
  }
  consul {
    serverAddr = "127.0.0.1:8500"
  }
  apollo {
    appId = "seata-server"
    apolloMeta = "http://192.168.1.204:8801"
    namespace = "application"
  }
  zk {
    serverAddr = "localhost:2181"
    sessionTimeout = 6000
    connectTimeout = 2000
    username = ""
    password = ""
  }
  etcd3 {
    serverAddr = "http://localhost:2379"
  }
  file {
    name = "file.conf"
  }
}

```
**file.conf**
```java
transport {
  # tcp udt unix-domain-socket
  type = "TCP"
  #NIO NATIVE
  server = "NIO"
  #enable heartbeat
  heartbeat = true
  # the client batch send request enable
  enableClientBatchSendRequest = true
  #thread factory for netty
  threadFactory {
    bossThreadPrefix = "NettyBoss"
    workerThreadPrefix = "NettyServerNIOWorker"
    serverExecutorThread-prefix = "NettyServerBizHandler"
    shareBossWorker = false
    clientSelectorThreadPrefix = "NettyClientSelector"
    clientSelectorThreadSize = 1
    clientWorkerThreadPrefix = "NettyClientWorkerThread"
    # netty boss thread size,will not be used for UDT
    bossThreadSize = 1
    #auto default pin or 8
    workerThreadSize = "default"
  }
  shutdown {
    # when destroy server, wait seconds
    wait = 3
  }
  serialization = "seata"
  compressor = "none"
}
service {
  #transaction service group mapping
  vgroupMapping.my_test_tx_group = "default"
  #only support when registry.type=file, please don't set multiple addresses
  default.grouplist = "127.0.0.1:8091"
  #degrade, current not support
  enableDegrade = false
  #disable seata
  disableGlobalTransaction = false
}

client {
  rm {
    asyncCommitBufferLimit = 10000
    lock {
      retryInterval = 10
      retryTimes = 30
      retryPolicyBranchRollbackOnConflict = true
    }
    reportRetryCount = 5
    tableMetaCheckEnable = false
    reportSuccessEnable = false
  }
  tm {
    commitRetryCount = 5
    rollbackRetryCount = 5
  }
  undo {
    dataValidation = true
    logSerialization = "jackson"
    logTable = "undo_log"
  }
  log {
    exceptionRate = 100
  }
}


```
`3.数据源及GlobalTransactionScanner配置`
```java
@Configuration
public class SeataAutoConfig {

  
    @Autowired
    private DataSourceProperties dataSourceProperties;

  
    @Bean
    public DruidDataSource druidDataSource(){
        DruidDataSource druidDataSource = new DruidDataSource();
        druidDataSource.setUrl(dataSourceProperties.getUrl());
        druidDataSource.setUsername(dataSourceProperties.getUsername());
        druidDataSource.setPassword(dataSourceProperties.getPassword());
        druidDataSource.setDriverClassName(dataSourceProperties.getDriverClassName());
        druidDataSource.setInitialSize(0);
        druidDataSource.setMaxActive(180);
        druidDataSource.setMaxWait(60000);
        druidDataSource.setMinIdle(0);
        druidDataSource.setValidationQuery("Select 1 from DUAL");
        druidDataSource.setTestOnBorrow(false);
        druidDataSource.setTestOnReturn(false);
        druidDataSource.setTestWhileIdle(true);
        druidDataSource.setTimeBetweenEvictionRunsMillis(60000);
        druidDataSource.setMinEvictableIdleTimeMillis(25200000);
        druidDataSource.setRemoveAbandoned(true);
        druidDataSource.setRemoveAbandonedTimeout(1800);
        druidDataSource.setLogAbandoned(true);
        return druidDataSource;
    }

 
    @Bean
    public DataSourceProxy dataSourceProxy(DruidDataSource druidDataSource){
        return new DataSourceProxy(druidDataSource);
    }

    @Bean
    public DataSourceTransactionManager transactionManager(DataSourceProxy dataSourceProxy) {
        return new DataSourceTransactionManager(dataSourceProxy);
    }

    @Bean
    public SqlSessionFactory sqlSessionFactory(DataSourceProxy dataSourceProxy) throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSourceProxy);
        factoryBean.setMapperLocations(new PathMatchingResourcePatternResolver()
                .getResources("classpath*:/mapper/*.xml"));
        factoryBean.setTransactionFactory(new SpringManagedTransactionFactory());
        return factoryBean.getObject();
    }

    @Bean
    public GlobalTransactionScanner globalTransactionScanner(){
	//dubbo-seata-service是应用名，my_test_tx_group是事务分组，和客户端file.conf中service.vgroupMapping.my_test_tx_group对应。
        return new GlobalTransactionScanner("dubbo-seata-service", "my_test_tx_group");
    }
```
`4.数据库增加事务回滚日志表`

    CREATE TABLE `undo_log` (
      `id` bigint(20) NOT NULL AUTO_INCREMENT,
      `branch_id` bigint(20) NOT NULL,
      `xid` varchar(100) NOT NULL,
      `context` varchar(128) NOT NULL,
      `rollback_info` longblob NOT NULL,
      `log_status` int(11) NOT NULL,
      `log_created` datetime NOT NULL,
      `log_modified` datetime NOT NULL,
      PRIMARY KEY (`id`),
      UNIQUE KEY `ux_undo_log` (`xid`,`branch_id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8;
	undo_log用于插入回滚日志,把前后镜像数据以及业务 SQL 相关的信息组成一条回滚日志记录，插入到 UNDO_LOG 表中。回滚完成后数据被删除。
### 二.启动服务
`1.启动zk`

`2.启动seata-server`

`3.启动项目(dubbo-seata-service和dubbo-seata)`
![在这里插入图片描述](/blog/img/seata/seata4.png)
    如上图seata-server中的日志，可以看到provider和consumer的TM,RM都已注册成功。

### 三.分布式事务的使用
```java
public class TestService {
    @Reference(retries = -1)
    private AccountService accountService;
    @Reference(retries = -1)
    private BussinessService bussinessService;

    @GlobalTransactional(name = "dubbo-seata")
    public void test(){
        System.out.println("开始全局事务，XID = " + RootContext.getXID());
        accountService.updateAccount();
        bussinessService.updateBussiness();
        throw new RuntimeException("测试抛异常后，分布式事务回滚！");
    }
}
```
    如上述代码，只需要在方法上加上@GlobalTransactional即可
**执行上述代码看seata-server日志**
![在这里插入图片描述](/blog/img/seata/seata5.png)

    如上图，可以看到整个事务回滚过程，开启全局事务——注册分支事务——回滚分支事务——回滚全局事务。


### 四.范例代码
    https://gitee.com/xshCloudy/sharding.git

### 五.参考项目及资料
    https://github.com/seata/seata.git
    https://github.com/seata/seata-samples.git
    https://seata.io/zh-cn/docs/overview/what-is-seata.html




