---
title: 用注解和mybatis拦截器优雅的实现字典赋值
catalog: true
date: 2021-02-08 14:42:20
subtitle:
header-img:
tags:
---
## 前言
```java
  实现逻辑主要是利用mybatis拦截器ResultSetHandler拦截查询数据，
  发现字段带有自定义注解@FieldBind，即查询对应的字典，给对应的字段赋值。
  这样每次只需要给需要赋值的字段加上注解即可实现字典赋值，不再需要每次重复代码

```
## 拦截器代码
```java
  @Intercepts({@Signature(
          type = ResultSetHandler.class,
          method = "handleResultSets",
          args = {Statement.class}
  )})
  public class FieldBindInterceptor implements Interceptor {
      private static final Logger log = LoggerFactory.getLogger(FieldBindInterceptor.class);
      private IDataBind dataBind;
  
      public void setDataBind(IDataBind dataBind) {
          this.dataBind = dataBind;
      }
  
      public FieldBindInterceptor() {
      }
  
      public FieldBindInterceptor(IDataBind dataBind) {
          this.dataBind = dataBind;
      }
  
      @Override
      public Object intercept(Invocation var1) throws Throwable {
          return fieldBind(var1, (var1x, var2) -> {
              setValue(this.dataBind, var1x, var2);
          });
      }
  
      private static Object fieldBind(Invocation invocation, BiConsumer<MetaObject, FieldSetProperty> biConsumer) throws Exception {
          DefaultResultSetHandler resultSetHandler = (DefaultResultSetHandler)invocation.getTarget();
          Field field = resultSetHandler.getClass().getDeclaredField("mappedStatement");
          field.setAccessible(true);
          MappedStatement mappedStatement = (MappedStatement)field.get(resultSetHandler);
          List var5 = (List)invocation.proceed();
          if (var5.isEmpty()) {
              return var5;
          } else {
              Configuration var6 = mappedStatement.getConfiguration();
              Iterator var7 = var5.iterator();
  
              while(var7.hasNext()) {
                  Object var8 = var7.next();
                  if (!FieldSetPropertyHolder.accept(var6, var8, biConsumer)) {
                      break;
                  }
              }
              return var5;
          }
      }
  
      private static void setValue(IDataBind var1, MetaObject var2, FieldSetProperty var3) {
          Object var4 = var2.getValue(var3.getFieldName());
          if (null != var4) {
              String var5 = var3.getFieldName();
              String var6 = String.valueOf(var4);
              if (null != var1) {
                  FieldBind var8 = var3.getFieldDict();
                  if (null != var8) {
                      var5 = var8.target();
                      var6 = var1.getNameByCode(var8, var6);
                  }
              }
              var2.setValue(var5, var6);
          }
  
      }
  }

```


```java
  public class FieldSetPropertyHolder {
  
      private static Map<Class<?>, List<FieldSetProperty>> classFieldSetPropertyMap;
      private static Set<Class<?>> classSet;
  
      static{
          classFieldSetPropertyMap = new ConcurrentHashMap<>();
          classSet = new CopyOnWriteArraySet<>();
      }
  
      private static List<FieldSetProperty> buildFieldSetPropertys(Class<?> var0) {
          if (classSet.contains(var0)) {
              return null;
          } else {
              List<FieldSetProperty> fieldSetProperties = classFieldSetPropertyMap.get(var0);
              if (null == fieldSetProperties) {
                  if (var0.isAssignableFrom(HashMap.class)) {
                      classSet.add(var0);
                  } else {
                      fieldSetProperties = new ArrayList<>();
                      Field[] var2 = var0.getDeclaredFields();
                      Field[] var3 = var2;
                      int var4 = var2.length;
  
                      for(int var5 = 0; var5 < var4; ++var5) {
                          Field var6 = var3[var5];
                          FieldBind var8 = var6.getAnnotation(FieldBind.class);
                          if (null != var8) {
                              fieldSetProperties.add(new FieldSetProperty(var6.getName(), var8));
                          }
                      }
  
                      if (fieldSetProperties.isEmpty()) {
                          classSet.add(var0);
                      } else {
                          classFieldSetPropertyMap.put(var0, fieldSetProperties);
                      }
                  }
              }
  
              return fieldSetProperties;
          }
      }
  
      public static boolean accept(Configuration var0, Object var1, BiConsumer<MetaObject, FieldSetProperty> var2) {
          List<FieldSetProperty> var3 = buildFieldSetPropertys(var1.getClass());
          if (CollectionUtils.isNotEmpty(var3)) {
              MetaObject var4 = var0.newMetaObject(var1);
              var3.parallelStream().forEach((var2x) -> {
                  var2.accept(var4, var2x);
              });
              return true;
          } else {
              return false;
          }
      }
  }

```
## 注解代码


```java
@Documented
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.FIELD, ElementType.ANNOTATION_TYPE})
public @interface FieldBind {

    String type();

    String target();
}

注解有两个属性，type即字典的code, target是赋值的字段

```
## 查询字典代码

```java
public interface IDataBind {

    String getNameByCode(FieldBind var1, String var2);
}

@Slf4j
@Component
public class DataDict implements IDataBind {
    @Autowired
    private ISysDictionaryService iSysDictionaryService;
    @Override
    public String getNameByCode(FieldBind fieldBind, String code) {
        log.info("字段类型：" + fieldBind.type() + "，编码：" + code);
        //这里实现查询字典的逻辑
        SysDictionaryDTO sysDictionaryDTO = iSysDictionaryService.queryByTypeAndCode(fieldBind.type(), code);
        return sysDictionaryDTO.getItemDesc();
    }
}

```


## 配置mybatis拦截器

```java
@Configuration
public class MybatisPlusConfig {
    @Autowired(required = false)
    private IDataBind dataBind;
    public MybatisPlusConfig() {
    }

    @Bean
    public ConfigurationCustomizer mybatisConfigurationCustomizer() {
        return configuration -> configuration.addInterceptor(new FieldBindInterceptor(dataBind));
    }
}

```


## 如何使用

```java
    @Data
    @EqualsAndHashCode(callSuper = true)
    @TableName("student")
    public class Student extends BaseEntity {
    
        @TableField("name")
        private String name;
    
        @FieldBind(type = "sex", target = "sexText")
        @TableField("sex")
        private String sex;
    
        @TableField(exist = false)
        private String sexText;
    
    }
    
   
```
![在这里插入图片描述](/blog/img/dictionary/1.jpg)

```java
如上图，性别sex有两个字典，我们在sex字段上加了一个注解@FieldBind, type='sex',target = 'sexText'
表示会去数据库字典表查询type='sex'，字典code（item_code）为字段sex值的字典，将字典的值(item_desc)
赋值给sexText字典

```

