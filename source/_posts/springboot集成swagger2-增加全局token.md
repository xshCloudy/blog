---
title: springboot集成swagger2(增加全局token)
catalog: true
date: 2018-11-19 15:59:57
subtitle: swagger
header-img: "/blog/img/article_header/article_header.png"
tags:
- java
---
# 一.依赖
```javascript
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.8.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.8.0</version>
</dependency>
```
# 二.配置
## yml
```javascript
swagger.enable: true #swagger是否生效，生产环境记得禁用
```
## swagger config
```javascript
@Configuration
@ConditionalOnProperty(prefix = "swagger", value = {"enable"}, havingValue = "true")
@EnableSwagger2
public class Swagger2Config {
    @Bean
    public Docket createRestApi() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.xinji.directpaymentclient.web.controller"))
                .paths(PathSelectors.any())
                .build()
                .securitySchemes(securitySchemes())
                .securityContexts(securityContexts())
                ;
    }

    private ApiInfo apiInfo() {
        Contact contact = new Contact("技术部", "", "");
        return new ApiInfoBuilder()
                .title("我的博客接口文档")
                .description("我的博客接口文档")
                .version("1.0")
                .contact(contact)
                .build();
    }

    private List<ApiKey> securitySchemes() {
        List<ApiKey> arrayList = new ArrayList<>();
        arrayList.add(new ApiKey("Authorization", "token", "header"));
        return arrayList;
    }

    private List<SecurityContext> securityContexts() {
        List<SecurityContext> arrayList = new ArrayList<>();
        arrayList.add(SecurityContext.builder()
                .securityReferences(defaultAuth())
                .build());
        return arrayList;
    }

    private List<SecurityReference> defaultAuth() {
        AuthorizationScope authorizationScope = new AuthorizationScope("global", "accessEverything");
        AuthorizationScope[] authorizationScopes = new AuthorizationScope[1];
        authorizationScopes[0] = authorizationScope;
        List<SecurityReference> arrayList = new ArrayList<>();
        arrayList.add(new SecurityReference("Authorization", authorizationScopes));
        return arrayList;
    }
```
## 拦截器或filter给swagger页面放行
# 三.可能出现的问题及解决方案
## 问题
swagger页面404或者拦截器进入/error路径
## 解决方案
自定义静态资源映射目录，addResoureHandler指的是对外暴露的访问路径，addResourceLocations指的是文件放置的目录
```javascript
@Configuration
public class WebConfig implements WebMvcConfigurer{
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");
                registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
    }
```
**特别注意：**
如果你的项目中使用了 extends WebMvcConfigurationSupport，swagger自动配置失效，必然会出现以上情况。只需要在你的项目中继承WebMvcConfigurationSupport的子类中加上：
```javascript
   @Override
    protected void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");
        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");
        super.addResourceHandlers(registry);
    }
```
