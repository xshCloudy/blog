---
title: shardingjdbc读写分离
catalog: true
date: 2020-11-05 14:27:29
subtitle: shardingjdbc
header-img: "/blog/img/header_img/404.png"
tags:
- java
---
## io.shardingjdbc读写分离
### 一.读写分离架构图
![在这里插入图片描述](/blog/img/shardingjdbc/shardingjdbc1.PNG)
### 二.数据库主从配置
[mysql主从设置](https://xshcloudy.gitee.io/blog/uncategorized/mysql%E6%95%B0%E6%8D%AE%E5%BA%93%E4%B8%BB%E4%BB%8E%E8%AE%BE%E7%BD%AE/ "mysql主从设置")
### 三.shardingjdbc配置
`1.引入依赖`
```java
<dependency>
	<groupId>io.shardingjdbc</groupId>
	<artifactId>sharding-jdbc-core</artifactId>
	<version>2.0.3</version>
</dependency>
```
`2.config配置`
```java
sharding:
  jdbc:
    datasource:
      #master数据库数据源
      master:
        username: root
        password: 111111
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost:3306/sharding?useUnicoe=true&characterEncoding=utf-8&serverTimezone=GMT
        type: com.alibaba.druid.pool.DruidDataSource
      #slave数据库数据源
      slave0:
        username: root
        password: 123456
        driver-class-name: com.mysql.jdbc.Driver
        url: jdbc:mysql://localhost:3307/sharding?useUnicoe=true&characterEncoding=utf-8&serverTimezone=GMT
        type: com.alibaba.druid.pool.DruidDataSource
```
```java
@Configuration
@EnableTransactionManagement
@MapperScan(value = {"com.pineapple.dao"}, sqlSessionFactoryRef = "sqlSessionFactory")
public class MybatisConfig {

    @Bean(name = "masterDataSource")
    @Primary
    @ConfigurationProperties(prefix = "sharding.jdbc.datasource.master")
    public DataSource getMasterDataSource() {
        return new DruidDataSource();
    }

    @Bean(name = "slaveDataSource0")
    @ConfigurationProperties(prefix = "sharding.jdbc.datasource.slave0")
    public DataSource getSlaveDataSource0() {
        return new DruidDataSource();
    }

    @Bean(name = "masterSlaveJdbcDatasource")
    public DataSource getMasterSlaveDataSource(@Qualifier("masterDataSource") DataSource dataSourceMaster,
        @Qualifier("slaveDataSource0") DataSource dataSourceSlave0) throws SQLException {

        Map<String, DataSource> dsMap = new HashMap<>();
        dsMap.put("ds-master-0", dataSourceMaster);
        dsMap.put("ds-master-0-slave-0", dataSourceSlave0);
        MasterSlaveRuleConfiguration masterSlaveRuleConfig = new MasterSlaveRuleConfiguration();
        masterSlaveRuleConfig.setName("ds-0");
        masterSlaveRuleConfig.setMasterDataSourceName("ds-master-0");
        masterSlaveRuleConfig.setSlaveDataSourceNames(Arrays.asList("ds-master-0-slave-0"));
        return MasterSlaveDataSourceFactory.createDataSource(dsMap, masterSlaveRuleConfig, new HashMap<>());
    }

    @Bean(name = "sqlSessionFactory")
    @Primary
    public SqlSessionFactory sqlSessionFactory(
        @Qualifier("masterSlaveJdbcDatasource") DataSource masterSlaveJdbcDatasource) throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setVfs(SpringBootVFS.class);
        factoryBean.setDataSource(masterSlaveJdbcDatasource);
        PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        factoryBean.setMapperLocations(resolver.getResources("classpath*:/mapper/*.xml"));
        return factoryBean.getObject();
    }

    @Bean(name = "transactionManager")
    public DataSourceTransactionManager transactionManager(@Qualifier("masterSlaveJdbcDatasource") DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    @Bean(name = "transactionTemplate")
    public TransactionTemplate transactionTemplate(@Qualifier("transactionManager") DataSourceTransactionManager transactionManager) {
        return new TransactionTemplate(transactionManager);
    }
}
```

`shardingjdbc默认master数据库执行delete,update,insert,slave数据库执行select`





