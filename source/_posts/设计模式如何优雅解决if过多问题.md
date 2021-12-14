---
title: 用设计模式如何优雅解决if过多问题？
catalog: true
date: 2019-12-12 22:45:28
subtitle: "设计模式"
header-img: "/blog/img/article_header/article_header.png"
tags:
- java
---



# 用设计模式如何优雅解决if过多问题？

平时写代码是不是经常有if判断过多，每个if里面也有大量对应的逻辑，造成代码臃肿难看呢。
我们怎么用设计模式来优雅的解决这种问题呢？


## 话不多说直接上代码

```
public enum BehaviorEnum{
    DELETE_FORM,
    CREATE_FORM;
}
```

```
public interface ProcessHandler {
    String primaryKey();
    void process(String param);
}
```


```
/**
 * @version 1.0
 * @Author shenghao.xiao
 * @Date 2019/2/28
 **/
@Component
public class CreateFormHandler implements ProcessHandler {
 @Override
    public String primaryKey() {
        return BehaviorEnum.CREATE_FORM.name();
    }
    //这里就是if对应的逻辑处理
  @Override
    public void process(String param) {
    System.out.println(param);
    }
}
```


```
/**
 * @version 1.0
 * @Author shenghao.xiao
 * @Date 2019/2/28
 **/
@Component
public class DeleteFormHandler implements ProcessHandler {
 @Override
    public String primaryKey() {
        return BehaviorEnum.DELETE_FORM.name();
    }
  @Override
    public void process(String param) {
    System.out.println(param);
    }
}
```


```
/**
 * @version 1.0
 * @Author shenghao.xiao
 * @Date 2019/2/28
 **/
@Component
public class ProcessHandlerFactory {
    private static final Map<String, ProcessHandler> HANDLER_MAP = new HashMap<>();

    @Autowired
    private List<ProcessHandler> handlers;

    @PostConstruct
    public void init(){
        for (ProcessHandler handler : handlers){
            HANDLER_MAP.put(handler.primaryKey(), handler);
        }
    }

    public ProcessHandler getHandler(BehaviorEnum behaviorEnum){
        return HANDLER_MAP.get(behaviorEnum.name());
    }
}
```
## 使用
```
public class NoticeTest extends BaseTest {
@Autowired
    private ProcessHandlerFactory processHandlerFactory;
  @Test
    public void test1() {
        processHandlerFactory.getHandler(BehaviorEnum .DELETE_FORM.name())
            .process("DELETE_FORM");
    }
      @Test
    public void test2() {
        processHandlerFactory.getHandler(BehaviorEnum .CREATE_FORM.name())
            .process("CREATE_FORM");
    }
}
```


```javascript
上述代码其实就是利用工厂模式，将多个if的逻辑拆成单个，
只需要根据相应的枚举用工厂类processHandlerFactory判断走哪个对应的if逻辑即可。
非常的实用！！！

```

