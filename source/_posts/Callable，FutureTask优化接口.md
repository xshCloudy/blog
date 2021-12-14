---
title: Callable，FutureTask优化接口
catalog: true
date: 2019-12-12 19:06:24
subtitle: "优化接口"
header-img: "/blog/img/article_header/article_header.png"
tags:
- java
---

## 一.futrue,callable接口关系图
![在这里插入图片描述](/blog/img/all/futrueTask1.jpg)



```javascript

Callable接口代表一段可以调用并返回结果的代码;Future接口表示异步任务，是还没有完成的任务给出的未来结果。
所以说Callable用于产生结果，Future用于获取结果。

Callable接口使用泛型去定义它的返回类型。Executors类提供了一些有用的方法在线程池中执行Callable内的任务。
由于Callable任务是并行的（并行就是整体看上去是并行的，其实在某个时间点只有一个线程在执行），我们必须等待它返回的结果。 
java.util.concurrent.Future对象为我们解决了这个问题。在线程池提交Callable任务后返回了一个Future对象，
使用它可以知道Callable任务的状态和得到Callable返回的执行结果。
Future提供了get()方法让我们可以等待Callable结束并获取它的执行结果。

```
### 在Future接口中声明了5个方法，下面依次解释每个方法的作用：

```javascript
cancel方法用来取消任务，如果取消任务成功则返回true，如果取消任务失败则返回false。
参数mayInterruptIfRunning表示是否允许取消正在执行却没有执行完毕的任务，如果设置true，则表示可以取消正在执行过程中的任务。
如果任务已经完成，则无论mayInterruptIfRunning为true还是false，此方法肯定返回false，即如果取消已经完成的任务会返回false；
如果任务正在执行，若mayInterruptIfRunning设置为true，则返回true，若mayInterruptIfRunning设置为false，则返回false；
如果任务还没有执行，则无论mayInterruptIfRunning为true还是false，肯定返回true。

isCancelled方法表示任务是否被取消成功，如果在任务正常完成前被取消成功，则返回 true。

isDone方法表示任务是否已经完成，若任务完成，则返回true；

get()方法用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回；

get(long timeout, TimeUnit unit)用来获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null。

```

## 二.使用范例
```javascript
​​​​	​
@Configuration
@EnableAsync
public class AsyncCofig {
    @Value("${Executor.corePoolSize}")
    private int corePoolSize;
    @Value("${Executor.maxPoolSize}")
    private int maxPoolSize;
    @Value("${Executor.queueCapacity}")
    private int queueCapacity;

    @Bean
    public ThreadPoolTaskExecutor taskExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(corePoolSize);
        executor.setMaxPoolSize(maxPoolSize);
        executor.setQueueCapacity(queueCapacity);
        executor.initialize();
        return executor;
    }
}

​

```


```javascript

​
public class CallableTask implements Callable<String> {
    @Override
    public String call(){
        return Thread.currentThread().getName();
    }
}

​

```



```javascript

​
public class FutrueTest extends BaseTest {
    @Autowired
    private ThreadPoolTaskExecutor executor;

    @Test
    public void test() throws ExecutionException, InterruptedException {
        List<String> dataList = new ArrayList<>();
        FutureTask<String> futureTask1 = new FutureTask<>(new CallableTask());
        FutureTask<String> futureTask2 = new FutureTask<>(new CallableTask());
        executor.submit(futureTask1);
        executor.submit(futureTask2);
        System.out.println(futureTask1.isDone());
        String result1 = futureTask1.get();
        dataList.add(result1);
        String result2 = futureTask2.get();
        dataList.add(result2);
        System.out.println(JSONObject.toJSONString(dataList));
    }
}

​

```


## 三.结论

![在这里插入图片描述](/blog/img/all/futrueTask2.jpg)