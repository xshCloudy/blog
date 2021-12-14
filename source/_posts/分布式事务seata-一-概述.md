---
title: 分布式事务seata(一)概述
catalog: true
date: 2020-12-04 09:22:51
subtitle: seata
header-img: "/blog/img/header_img/404.png"
tags:
- seata
---
## 分布式事务seata(一)概述
### 一.seata是什么？
    Seata 是一款开源的分布式事务解决方案，致力于提供高性能和简单易用的分布式事务服务。
	Seata 提供了 AT、TCC、SAGA 和 XA 事务模式，打造一站式的分布式解决方案。

### 二.分布式事务模型图
![在这里插入图片描述](/blog/img/seata/seata1.png)

    TC (Transaction Coordinator) - 事务协调者(seata-server)
    维护全局和分支事务的状态，驱动全局事务提交或回滚。
    
    TM (Transaction Manager) - 事务管理器(全局事务管理)
    定义全局事务的范围：开始全局事务、提交或回滚全局事务。
    
    RM (Resource Manager) - 资源管理器（本地/分支事务管理）
    管理分支事务处理的资源，与TC交谈以注册分支事务和报告分支事务的状态，并驱动分支事务提交或回滚。

### 三.分布式事务解决方案
#### 1.AT模式
    AT模式对业务无入侵，开发无感知，适用于不希望对业务进行改造的场景，会产生脏写。
![在这里插入图片描述](/blog/img/seata/seata2.png)

#### 2.TCC模式
    TCC 模式需要自定义实现try,confirm,rollback逻辑,把自定义的分支事务纳入到全局事务的管理中，
	对业务侵入大，但效率高。适用于核心系统等对性能有很高要求的场景。
![在这里插入图片描述](/blog/img/seata/seata3.png)

#### 3.saga模式
    长事务解决方案，适用于业务流程长且需要保证事务最终一致性的业务系统
	（第一阶段就操作DB，会存在脏读问题）
#### 4.XA模式
    分布式强一致性的解决方案，但性能低而使用较少。




