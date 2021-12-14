---
title: 如何用注解实现Redis分布式锁
catalog: true
date: 2019-06-28 15:16:55
subtitle: "redis分布式锁"
header-img: "/blog/img/article_header/article_header.png"
tags:
- java
---
## 一.maven依赖
springboot版本2.0.6，redis版本2.0.11
```javascript
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
```
## 二.redis命令 setnx

```javascript
public class RedisHelper {

    @Autowired
    private RedisTemplate redisTemplate;

    @PostConstruct
    public void init() {
        StringRedisSerializer stringRedisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer<Object> objectJackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        redisTemplate.setKeySerializer(stringRedisSerializer);
        redisTemplate.setHashKeySerializer(stringRedisSerializer);
        redisTemplate.setValueSerializer(objectJackson2JsonRedisSerializer);
        redisTemplate.setHashValueSerializer(objectJackson2JsonRedisSerializer);
    }
    /**
     * 分布式锁
     */
    public Boolean setnx(Object key, Object value, Long expireTime, TimeUnit timeUnit) {
        if (key == null) {
            return false;
        }
        return this.setIfAbsent(key, value, expireTime, timeUnit);
    }

    private Boolean setIfAbsent(Object key, Object value, long timeout, TimeUnit unit) {
        byte[] rawKey = this.rawKey(key);
        byte[] rawValue = this.rawValue(value);
        Expiration expiration = Expiration.from(timeout, unit);
        return (Boolean) redisTemplate.execute((connection) -> {
            return connection.set(rawKey, rawValue, expiration, RedisStringCommands.SetOption.ifAbsent());
        }, true);
    }

    private byte[] rawKey(Object key) {
        return key instanceof byte[] ? (byte[]) ((byte[]) key) : this.keySerializer().serialize(key);
    }

    private byte[] rawValue(Object value) {
        return value instanceof byte[] ? (byte[]) ((byte[]) value) : this.valueSerializer().serialize(value);
    }

    private RedisSerializer valueSerializer() {
        return redisTemplate.getValueSerializer();
    }

    private RedisSerializer keySerializer() {
        return redisTemplate.getKeySerializer();
    }
    /**
     * 删除对应的value
     *
     * @param key
     */
    public boolean remove(String key) {
        if (key == null) {
            return false;
        }
        return redisTemplate.delete(key);
    }

}
```
>特别注意：如果你的redis是2.1以上，setIfAbsent增加了设置过期时间。可以替换成下面的写法。
```javascript
 public Boolean setnx(String key, Object value, Long expireTime,TimeUnit timeUnit) {
        ValueOperations<String, Object> operations = redisTemplate.opsForValue();
        return operations.setIfAbsent(key,value,expireTime, timeUnit);
    }
```
## 三.注解写法
### 1.注解@DistributeLock（支持spel表达式）
```javascript
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
public @interface DistributeLock {
    String key();
    long timeout() default 10;
    TimeUnit timeUnit() default TimeUnit.SECONDS;
}

```
### 2.注解解释器
```javascript

/**
 * @version 1.0
 * @Author shenghao.xiao
 * @Date 2019/6/27
 **/
@Aspect
@Component
@Order(1)
@Slf4j
public class DistributeLockAspect {
    @Autowired
    private RedisHelper redisHelper;
    private static final ParameterNameDiscoverer DISCOVERER = new LocalVariableTableParameterNameDiscoverer();

    @Around("@annotation(distributeLock)")
    public Object doAround(ProceedingJoinPoint joinPoint, DistributeLock distributeLock) throws Throwable{
        Object[] args = joinPoint.getArgs();
        StandardEvaluationContext standardEvaluationContext = new StandardEvaluationContext(args);
        MethodSignature methodSignature = (MethodSignature) joinPoint.getSignature();
        Method method = joinPoint.getTarget().getClass().getMethod(methodSignature.getName(), methodSignature.getParameterTypes());
        String[] parametersName = DISCOVERER.getParameterNames(method);
        if (args.length > 0) {
            for (int i = 0; i < args.length; i++) {
                standardEvaluationContext.setVariable(parametersName[i], args[i]);
            }
        }
        String key = distributeLock.key();
		if (key.contains("#")) {
            key = generateKey(key, standardEvaluationContext);
        }
        if(StringUtils.isNoneBlank(distributeLock.prefix())){
            key = distributeLock.prefix() + "::" + key;
        }
        long timeout = distributeLock.timeout();
        TimeUnit timeUnit = distributeLock.timeUnit();
     	if (lock(key, timeout, timeUnit)) {
            log.info("分布式锁加锁成功key:{},timeout:{}", key, timeout);
            try {
                return joinPoint.proceed();
            } catch (Throwable throwable) {
                throw new SystemException("DistributeLockAspect.doAround error", throwable);
            } finally {
                unLock(key);
            }
        }
        return null;
    }

    private static String generateKey(String key, StandardEvaluationContext standardEvaluationContext) {
        ExpressionParser parser = new SpelExpressionParser();
        Expression exp = parser.parseExpression(key);
        return exp.getValue(standardEvaluationContext, String.class);
    }

    private void unLock(String key) {
        boolean remove = redisHelper.remove(key);
        if (!remove) {
            log.error("释放分布式锁失败，key={}，已自动超时", key);
        }
    }

    private Boolean lock(String key, Long timeout, TimeUnit timeUnit) {
        String value = UUID.randomUUID().toString();
        return redisHelper.setnx(key, value, timeout, timeUnit);
    }

}
```
### 3.使用方式
```javascript
@Service
public class TestService {
    @DistributeLock(key = "#code+#phoneNo", timeout = 1800)
    public void sys(String code, String phoneNo) {
        System.out.println("今天天氣不錯哦！");
    }
}
```
## 四.遗留问题
>redis超时时间是自己控制的，业务的实际执行时间并不能准确确定，redission续期可解决该问题。



