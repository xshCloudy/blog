---
title: springboot使用redis作为二级缓存
catalog: true
date: 2019-01-26 23:33:50
subtitle: "二级缓存"
header-img: "/blog/img/article_header/article_header.png"
tags:
- java
---
# springboot使用redis作为二级缓存
## 1.maven依赖
```javascript
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-cache</artifactId>
        </dependency>
```
## 2.配置
```javascript
/**
 * @version 1.0
 * @Author shenghao.xiao
 * @Date 2019/5/13
 **/
@Configuration
@EnableCaching
public class RedisConfig extends CachingConfigurerSupport {
    @Value("${cache.expireTime}")
    private Integer cacheExpireTime;
    private final static Logger log = LoggerFactory.getLogger(RedisConfig.class);

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        //设置redis序列化方式
        StringRedisTemplate template = new StringRedisTemplate(connectionFactory);
        template.setValueSerializer(new Jackson2JsonRedisSerializer(Object.class));
        template.setHashValueSerializer(new Jackson2JsonRedisSerializer(Object.class));
        template.afterPropertiesSet();
        //初始化CacheManager，使用redis作为二级缓存
        return new RedisCacheManager(RedisCacheWriter.nonLockingRedisCacheWriter(connectionFactory),
                getDefaultTtlRedisCacheConfiguration(cacheExpireTime), getCustomizeTtlRedisCacheConfigurationMap());
    }

    /**
     * 设置缓存默认配置
     *
     * @param seconds
     * @return
     */
    private RedisCacheConfiguration getDefaultTtlRedisCacheConfiguration(Integer seconds) {
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<>(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        RedisCacheConfiguration redisCacheConfiguration = RedisCacheConfiguration.defaultCacheConfig();
        //设置序列化方式以及失效时间
        redisCacheConfiguration = redisCacheConfiguration.serializeValuesWith(
                RedisSerializationContext
                        .SerializationPair
                        .fromSerializer(jackson2JsonRedisSerializer)
        ).entryTtl(Duration.ofSeconds(seconds));
        return redisCacheConfiguration;
    }

    /**
     * 自定义缓存失效时间
     *
     * @return
     */
    private Map<String, RedisCacheConfiguration> getCustomizeTtlRedisCacheConfigurationMap() {
        Map<String, RedisCacheConfiguration> redisCacheConfigurationMap = new HashMap<>();
        //dictionary就是我们自定义的key，使用@Cacheable等注解时，将其value属性设置为dictionary，那么这个dictionary的缓存失效时间就是这里我们自定义的失效时间（cacheExpireTime）
        redisCacheConfigurationMap.put("dictionary", this.getDefaultTtlRedisCacheConfiguration(cacheExpireTime));
        return redisCacheConfigurationMap;
    }
	/**
	设置缓存默认key生成方式，使用@Cacheable注解时，如果不指明key值，则默认按下面方式生成key
	 */
    @Nullable
    @Override
    public KeyGenerator keyGenerator() {
        return new KeyGenerator() {
            @Override
            public Object generate(Object target, Method method, Object... params) {
                StringBuilder sb = new StringBuilder();
                sb.append(target.getClass().getName());
                sb.append(method.getName());
                for (Object obj : params) {
                    sb.append(obj.toString());
                }
                log.info("缓存自动生成key：" + sb.toString());
                return sb.toString();
            }
        };
    }
}

```
## 3.使用方式
###### @Cacheable
当标记在一个方法上时表示该方法是支持缓存的，当标记在一个类上时则表示该类所有的方法都是支持缓存的。，缓存改方法的返回值。
| 参数        | 解释   |  例子  |
| :--------   | :----- | :----  |
| value      | 缓存的名称，在 spring 配置文件中定义，必须指定至少一个   |   例如:<br />@Cacheable(value=”mycache”)<br />@Cacheable(value={”cache1”,”cache2”}     |
|key        |   缓存的 key，可以为空，如果指定要按照 SpEL 表达式编写，如果不指定，则缺省按照方法的所有参数进行组合   |   @Cacheable(value=”testcache”,key=”#userName”)   |
| condition        |    缓存的条件，可以为空，使用 SpEL 编写，返回 true 或者 false，只有为 true 才进行缓存    |  @Cacheable(value=”testcache”,condition=”#userName.length()>2”)  |

###### @CachePut
@CachePut标注的方法在执行前不会去检查缓存中是否存在之前执行过的结果，而是每次都会执行该方法，并将执行结果以键值对的形式存入指定的缓存中。


###### @CacheEvict
@CacheEvict是用来标注在需要清除缓存元素的方法或类上的。当标记在一个类上时表示其中所有的方法的执行都会触发缓存的清除操作。
###### @Caching
@Caching注解可以让我们在一个方法或者类上同时指定多个Spring Cache相关的注解。其拥有三个属性：cacheable、put和evict，分别用于指定@Cacheable、@CachePut和@CacheEvict。
```javascript
@Caching(cacheable = @Cacheable("users"), evict = { @CacheEvict("cache2"),
   @CacheEvict(value = "cache3") })
   public User find(Integer id) {
      returnnull;
   }
   ```

