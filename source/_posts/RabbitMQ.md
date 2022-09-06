---
title: RabbitMQ
catalog: true
date: 2021-01-02 19:12:13
subtitle:
header-img:
tags:
---

## 一.RabbitMQ 概念
```java
    主要是用来实现应用程序的异步和解耦，同时也能起到消息缓冲，消息分发的作用;
    RabbitMQ使用的是AMQP协议，它是一种二进制协议。默认启动端口 5672
```
## 二.RabbitMQ 模型![在这里插入图片描述](/blog/img/rabbitmq/1.png)
```java
  P代表消息发送者；X代表交换机exchange；orange,blank和green代表路由绑定；
  C代表消费者。
  rabbitmq有4个比较重要的概念，分别为：虚拟主机（virtual-host），
  交换机，队列，和路由绑定。
  

 - 虚拟主机：一个虚拟主机持有一组交换机、队列和绑定。
 	用户只能在虚拟主机的粒度进行权限控制。如果需要禁止A组访问B组的交换机/队列/绑定，
 	必须为A和B分别创建一个虚拟主机。每一个RabbitMQ服务器都有一个默认的虚拟主机“/”。
 - 交换机：Exchange用于转发消息，但它不会做存储，
 	如果没有Queue bind 到 Exchange 的话，它会直接丢弃掉Producer发送过来的消息。
 - 路由绑定：也就是交换机需要和队列相绑定的路由键

```
## 三.交换机
```java
   rabbitmq的常用消息模型都是基于交换机的类型衍生出的

 - Directed Exchange
	路由键exchange，该交换机收到消息后会把消息发送到指定routing-key的queue中。
 - Default Exchange
 	这种是一种特殊的Direct Exchange，是rabbitmq内部默认的一个交换机。
 	该交换机的name是空字符串，所有queue都默认binding 到该交换机上。
 	所有binding到该交换机上的queue，routing-key都和queue的name一样  
 - Topic Exchange
    通配符交换机，exchange会把消息发送到一个或者多个满足通配符规则的routing-key的queue。
    其中_表号匹配一个word，#匹配多个word和路径，路径之间通过.隔开。
    如满足a._.c的routing-key有a.hello.c；满足#.hello的routing-key有a.b.c.helo。
 - Fanout Exchange
   扇形交换机，该交换机会把消息发送到所有binding到该交换机上的queue。
   这种是publisher/subcribe模式。用来做广播最好。
   所有该exchagne上指定的routing-key都会被ignore掉。
 - Header Exchange
   设置header attribute参数类型的交换机。
```
## 三.rabbitmq的使用
`1.引入依赖`
```java
  <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-amqp</artifactId>
  </dependency>
  版本2.3.3.RELEASE
```
`2.yaml配置`
```json
spring:
  rabbitmq:
    addresses: localhost:5672
    username: admin
    password: 123456
    #使用发布确认模式时必须设置为true，否则消息消息路由失败也无法触发Return回调
    publisher-returns: true
    #NONE值是禁用发布确认模式，CORRELATED值是发布消息成功到交换器后会触发回调方法
    publisher-confirm-type: correlated
#    虚拟host 可以不设置,使用server默认host “/”
#    virtual-host: momvhost
    template:
      # true：交换机无法将消息进行路由时，会将该消息返回给生产者（回调消息回退ReturnCallback接口）
      # false：如果发现消息无法进行路由，则直接丢弃
      mandatory: true
    listener:
      simple:
      #none（默认开启），manual手动模式，auto 自动模式
       acknowledge-mode: manual
    # 消费者的最小数量
       concurrency: 1
    # 消费者的最大数量
       max-concurrency: 1
    # 是否支持重试
    	#业务代码抛异常，消费端会重试，业务代码一般会try catch异常，禁止重试
       retry:
        enabled: true
```
`3.config配置`
```json
@Slf4j
@Configuration
public class AmqpConfig {
    @Autowired
    private RabbitTemplate rabbitTemplate;
    @Autowired
    private IMesMfMqRecordService mesMfMqRecordService;

    @PostConstruct
    public void init() {
        //确认是否发到交换机
        rabbitTemplate.setConfirmCallback((correlationData, ack, cause) -> {
            if (!ack) {
                sendErrorMsgToOA(correlationData.getId(), "消息未成功发送至交换机！");
            }
        });
        // 确认是否发到队列，若没有则调用下面方法
        rabbitTemplate.setReturnCallback((message, replyCode, replyText, exchange, routingKey) -> {
            MessageProperties messageProperties = message.getMessageProperties();
            printErrorMsg(messageProperties.getMessageId(), "消息未成功发送至队列！");
        });
    }

    private void printErrorMsg(String messageId, String content) {
        LambdaUpdateWrapper<MesMfMqRecord> wrapper = Wrappers.lambdaUpdate();
        wrapper.eq(MesMfMqRecord::getDataId, messageId)
                .set(MesMfMqRecord::getFailReason, content)
                .set(MesMfMqRecord::getResult, "fail");
        mesMfMqRecordService.update(wrapper);
        JSONObject object = new JSONObject();
        object.put("sender", "XXX");
        object.put("receiver", "OOO");
        object.put("executor", "XXX");
        object.put("execute_time", BusDateUtil.getNow());
        object.put("data_code", messageId);
        object.put("content", content);
        log.info("消息发送失败，发送失败消息给OA：{}", JSONObject.toJSONString(object));
    }

  


    @Bean("testQueue")
    public Queue testQueue() {
        return new Queue("MQ.mes.erp.save.test");
    }

    @Bean("testExchange")
    DirectExchange testExchange() {
        return new DirectExchange("EX.mes.save.test");
    }

    @Bean("testBinding")
    Binding testBinding(@Qualifier("testQueue") Queue queue, @Qualifier("testExchange") DirectExchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with("RK.mes.save.test");
    }

```


`3.消息生产者代码`
```json
        String uuid = UUID.randomUUID().toString();
        MessageProperties messageProperties = new MessageProperties();
        messageProperties.setMessageId(uuid);
        Message message = new Message(msg.getBytes(), messageProperties);
        CorrelationData correlationData = new CorrelationData();
        correlationData.setId(uuid);
        correlationData.setReturnedMessage(message);
        rabbitTemplate.convertAndSend(ex, rk, message, correlationData);
```


`3.消息消费端代码`
```json
 
@Slf4j
@Component
public class TestListener {
    @AmqpLog(bizType = "test", producer = "XXX", consumer = "OOO")
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue(value = "MQ.mes.erp.save.test"),
            exchange = @Exchange(value = "EX.mes.save.test"),
            key = "RK.mes.save.test"))
    @RabbitHandler
    public void handleMessage(Message msg, Channel channel) {
        log.info("ProductionListener接收到queue消息：{}", msg == null ? "" : new String(msg.getBody()));
        try {
            //消息确认，false表示只确认当次消息，true表示批量确认ID小于等于当次ID的所有消息
            channel.basicAck(msg.getMessageProperties().getDeliveryTag(), false);
        } catch (Exception e) {
            log.error("ProductionListener.handleMessage error1", e);
            try {
                //消息拒绝，false表示直接抛弃，true表示重新放入队列,每次重启服务会重新消费消息
                channel.basicReject(msg.getMessageProperties().getDeliveryTag(), false);
            } catch (IOException e1) {
                log.error("ProductionListener.handleMessage error2", e1);
                throw new SystemException(new MesResultCustomCode(500, e1.getMessage()));
            }
            throw new SystemException(new MesResultCustomCode(500, e.getMessage()));
        }
    }
}
```

## 三.rabbitmq如何防止消息丢失
```json

 - 生产端开启发布确认模式
 	因为网络传输的不稳定性,生产端有两种场景会丢失消息，
 	一种是消息未成功发送至交换机，另外一种是未成功将消息发送至队列。
 	生产端开启发布确认模式，如上面config配置中，
 	配置publisher-returns: true，publisher-confirm-type: correlated，template.mandatory: true。
 	消息是否成功发送至交换机可以在ConfirmCallback接口中判断，
 	消息未成功发送至队列将会回调ReturnCallback接口。
 	在这两个接口中统一处理这两种消息丢失的场景即可！
 - 消费端开启手动确认机制
	消费端开启手动确认机制，如上面config配置中，
	配置listener.simple.acknowledge-mode: manual，
	在业务代码中手动ack
 - 开启队列的持久化
    生产者把消息成功写入队列之后，把消息持久化到磁盘
```

## 三.rabbitmq如何消息重复消费
```json
	生产端发消息的时候，给每一条业务消息赋值一个唯一的ID,如上述代码，
	生产端在MessageProperties的messageId赋值了一个唯一ID，
	消息接收方根据这个ID去在日志表中查询判断是否有重复消费。
```