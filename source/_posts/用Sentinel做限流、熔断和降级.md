---
title: 用Sentinel做限流、熔断和降级
catalog: true
date: 2020-12-11 09:43:54
subtitle: sentinel
header-img: "/blog/img/header_img/404.png"
tags:
- sentinel
---
## 用Sentinel做限流、熔断和降级
### 一.配置
`1.maven依赖`
**springboot版本：2.2.6.RELEASE**

    <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
       <version>2.2.3.RELEASE</version>
    </dependency>

`2.yaml配置`
```json
spring:
  cloud:
    sentinel:
      transport:
        dashboard: localhost:8080
      eager: false
```
### 二.使用
```json
@RequestMapping("test")
@RestController
public class TestController {
    @SentinelResource(value = "test")
    @GetMapping("demo")
    public Result test(String id) {
  	 System.out.println("Sentinel........");
       return ResultUtil.success();
    }
}
```
### 三.启动Sentinel-dashboard

    https://github.com/alibaba/Sentinel/releases，下载sentinel-dashboard-1.8.0.jar,
    输入命令java -Dserver.port=8080 -Dcsp.sentinel.dashboard.server=localhost:8080 
	-Dproject.name=sentinel-dashboard -jar sentinel-dashboard.jar启动sentinel-dashboard控制台。
	启动项目，打开浏览器输入localhost:8080即可看到Sentinel控制台，如下图。
    
![在这里插入图片描述](/blog/img/sentinel/sentinel1.png)

### 四.sentinel规则
`1.流控规则`
![在这里插入图片描述](/blog/img/sentinel/sentinel2.png)

    通过流控规则可以用来来指定允许该资源通过的请求次数，
    点击簇点链路中资源名的流控按钮，可增加限流规则。
    如上图，QPS单机阈值设置成1表示：单机应用每秒访问量不许超过1个，
	若请求1秒内访问量超过1，超出的请求会被阻塞跑错，如下图：

![在这里插入图片描述](/blog/img/sentinel/sentinel3.png)

`2.降级`

![在这里插入图片描述](/blog/img/sentinel/sentinel4.png)
```
- 慢调用比例 (SLOW_REQUEST_RATIO)：选择以慢调用比例作为阈值，
	需要设置允许的慢调用 RT（即最大的响应时间），请求的响应时间大于该值则统计为慢调用。
	当单位统计时长（statIntervalMs）内请求数目大于设置的最小请求数目，并且慢调用的比例
	大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态
	（HALF-OPEN 状态），若接下来的一个请求响应时间小于设置的慢调用 RT 则结束熔断，
	若大于设置的慢调用 RT 则会再次被熔断。
- 异常比例 (ERROR_RATIO)：当单位统计时长（statIntervalMs）内请求数目大于设置的最小请求数目，
	并且异常的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入
	探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成（没有错误）则结束熔断，
	否则会再次被熔断。异常比率的阈值范围是 [0.0, 1.0]，代表 0% - 100%。
- 异常数 (ERROR_COUNT)：当单位统计时长内的异常数目超过阈值之后会自动进行熔断。
	经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功完成
	（没有错误）则结束熔断，否则会再次被熔断。
```
**若请求被熔断，后端会抛错，如下图**
![在这里插入图片描述](/blog/img/sentinel/sentinel5.png)

`3.热点参数限流`
```json
使用热点参数限流功能，需要引入以下依赖：
  <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-parameter-flow-control</artifactId>
            <version>1.8.0</version>
  </dependency>
```
![在这里插入图片描述](/blog/img/sentinel/sentinel6.png)

    如上图，可根据参数设置参数限流规则，参数例外项，可以针对指定的参数值单独设置限流阈值，
	不受前面 count 阈值的限制。仅支持基本类型和字符串类型。

`4.授权规则`
![在这里插入图片描述](/blog/img/sentinel/sentinel7.png)

    授权规则很简单，即增加黑白名单限制，流控应用指请求来源origin(ip)。

### 五.范例代码
    https://gitee.com/xshCloudy/sentinel.git

### 六.参考项目及资料
    https://github.com/alibaba/Sentinel
	https://github.com/alibaba/Sentinel/tree/master/sentinel-demo 
	https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D





