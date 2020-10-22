## Gateway组件使用

### 什么是服务网关

```markdown
# 1.说明
- 网关统一服务入口，可方便实现对平台众多服务接口进行管控，对访问服务的身份认证、防报文重放与防数据篡改、功能调用的业务鉴权、响应数据的脱敏、流量与并发控制，甚至基于API调用的计量或者计费等等。

- 网关 =  路由转发 + 过滤器
	`路由转发：接收一切外界请求，转发到后端的微服务上去；
	`在服务网关中可以完成一系列的横切功能，例如权限校验、限流以及监控等，这些都可以通过过滤器完成
	
# 2.为什么需要网关
 - 1.网关可以实现服务的统一管理
 - 2.网关可以解决微服务中通用代码的冗余问题(如权限控制,流量监控,限流等)

# 3.网关组件在微服务中架构
```

![image-20200720171205828](./IMG.assets/SpringCloud.assets/image-20200720171205828.png)

### 服务网关组件

#### （1）zuul

Zuul is the front door for all requests from devices and web sites to the backend of the Netflix streaming application. As an edge service application, Zuul is built to enable dynamic routing, monitoring, resiliency and security.

```markdown
# 0.原文翻译
- https://github.com/Netflix/zuul/wiki
- zul是从设备和网站到Netflix流媒体应用程序后端的所有请求的前门。作为一个边缘服务应用程序，zul被构建为支持动态路由、监视、弹性和安全性。

# 1.zuul版本说明
- 目前zuul组件已经从1.0更新到2.0，但是作为springcloud官方不再推荐使用zuul2.0，但是依然支持zuul2.

# 2.springcloud 官方集成zuul文档
- https://cloud.spring.io/spring-cloud-netflix/2.2.x/reference/html/#netflix-zuul-starter
```

#### （2）gateway

This project provides a library for building an API Gateway on top of Spring MVC. Spring Cloud Gateway aims to provide a simple, yet effective way to route to APIs and provide cross cutting concerns to them such as: security, monitoring/metrics, and resiliency.

```markdown
# 0.原文翻译
- https://spring.io/projects/spring-cloud-gateway
- 这个项目提供了一个在springmvc之上构建API网关的库。springcloudgateway旨在提供一种简单而有效的方法来路由到api，并为api提供横切关注点，比如：安全性、监控/度量和弹性。

# 1.特性
- 基于springboot2.x 和 spring webFlux 和 Reactor 构建 响应式异步非阻塞IO模型
- 动态路由
- 请求过滤
```

###### 1.开发网关动态路由

```markdown
# 0.翻译
- 网关配置有两种方式一种是快捷方式,一种是完全展开方式

# 1.创建项目引入网关依赖
```

```xml
<!--引入gateway网关依赖-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

- **快捷方式配置路由**

```markdown
# 2.编写网关配置
```

```yml
spring:
  application:
    name: gateway
  cloud:
    consul:
      host: localhost
      port: 8500
    gateway:
      routes:
        - id: user_route							# 指定路由唯一标识
          uri: http://localhost:9999/ 				# 指定路由服务的地址
          predicates:
            - Path=/user/**					  		# 指定路由规则
        - id: product_route
          uri: http://localhost:9998/
          predicates:
            - Path=/product/**
server:
  port: 8989										# 网关服务的端口
```

```markdown
# 3.启动gateway网关项目
- 直接启动报错:
```

![image-20200720212535357](./IMG.assets/SpringCloud.assets/image-20200720212535357.png)

```markdown
- 在启动日志中发现,gateway为了效率使用webflux进行异步非阻塞模型的实现,因此和原来的web包冲突,Maven配置中去掉原来的web引入即可。
```

![image-20200720212653494](./IMG.assets/SpringCloud.assets/image-20200720212653494.png)

```markdown
- 再次启动成功启动
```

![image-20200720213657788](./IMG.assets/SpringCloud.assets/image-20200720213657788.png)

```markdown
# 4.测试网关路由转发
- 测试通过网关访问用户服务: http://localhost:8989/user/findOne?productId=21
- 测试通过网关访问商品服务: http://localhost:8989/product/findOne?productId=1
```

- **java方式配置路由**

```java
@Configuration
public class GatewayConfig {
    @Bean
    public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
        return builder.routes().route("order_route", r -> r.path("/order/**").uri("http://localhost:9997/"))
            .build();
    }
}
```

###### 2.查看网关路由规则列表

```markdown
# 1.说明
- gateway提供路由访问规则列表的web界面,但是默认是关闭的,如果想要查看服务路由规则可以在配置文件中开启
```

```yml
management:
  endpoints:
    web:
      exposure:
        include: "*"   #开启所有web端点暴露
```

```markdown
- 访问路由管理列表地址
- http://localhost:8989/actuator/gateway/routes
```

![image-20200720220647899](./IMG.assets/SpringCloud.assets/image-20200720220647899.png)

###### 3.配置路由服务负载均衡

```markdown
# 1.说明
- 现有路由配置方式,都是基于服务地址写死的路由转发,能不能根据服务名称进行路由转发同时实现负载均衡的呢?

# 2.动态路由以及负载均衡转发配置
```

```yml
spring:
  application:
    name: gateway
  cloud:
    consul:
      host: localhost
      port: 8500
    gateway:
      routes:
        - id: user_route
          #uri: http://localhost:9999/
          uri: lb://users							# lb代表转发后台服务使用负载均衡，users代表服务注册中心上的服务名
          predicates:
            - Path=/user/**
        - id: product_route
          #uri: http://localhost:9998/
          uri: lb://products          				# lb(loadbalance)代表负载均衡转发路由
          predicates:
            - Path=/product/**
      discovery:
        locator:
          enabled: true 							#开启根据服务名动态获取路由
```

![image-20200721110040104](./IMG.assets/SpringCloud.assets/image-20200721110040104.png)

###### 4.常用路由predicate(断言,验证)

```markdown
# 1.Gateway支持多种方式的predicate
```

![image-20200721112751340](./IMG.assets/SpringCloud.assets/image-20200721112751340.png)

```markdown
- After=2020-07-21T11:33:33.993+08:00[Asia/Shanghai]  			`指定日期之后的请求进行路由
- Before=2020-07-21T11:33:33.993+08:00[Asia/Shanghai]       	`指定日期之前的请求进行路由
- Between=2017-01-20T17:42:47.789-07:00[America/Denver], 2017-01-21T17:42:47.789-07:00[America/Denver]
- Cookie=username,chenyn										`基于指定cookie的请求进行路由
- Cookie=username,[A-Za-z0-9]+									`基于指定cookie的请求进行路由	
	`curl http://localhost:8989/user/findAll --cookie "username=zhangsna"
- Header=X-Request-Id, \d+										``基于请求头中的指定属性的正则匹配路由(这里全是整数)
	`curl http://localhost:8989/user/findAll -H "X-Request-Id:11"
- Method=GET,POST												`基于指定的请求方式请求进行路由

- 官方更多: https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.3.RELEASE/reference/html/#the-cookie-route-predicate-factory
```

```markdown
# 2.使用predicate
```

```yml
spring:
  application:
    name: gateway
  cloud:
    consul:
      host: localhost
      port: 8500
    gateway:
      routes:
        - id: user_route
          #uri: http://localhost:9999/
          uri: lb://users
          predicates:
            - Path=/user/**
            - After=2020-07-21T11:39:33.993+08:00[Asia/Shanghai]
            - Cookie=username,[A-Za-z0-9]+
            - Header=X-Request-Id, \d+
```

```markdown
# 3. 测试，用curl命令
```

```shell
$ curl http://localhost:8989/product/break?id=21 --cookie "username=chenyn"
```

###### 5.常用的Filter以及自定义filter

Route filters allow the modification of the incoming HTTP request or outgoing HTTP response in some manner. Route filters are scoped to a particular route. Spring Cloud Gateway includes many built-in GatewayFilter Factories.

```markdown
# 1.原文翻译
- 官网: 
	https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.3.RELEASE/reference/html/#gatewayfilter-factories
	
- 路由过滤器允许以某种方式修改传入的HTTP请求或传出的HTTP响应。路由筛选器的作用域是特定路由。springcloudgateway包括许多内置的GatewayFilter工厂。

# 2.作用
- 当我们有很多个服务时，比如下图中的user-service、order-service、product-service等服务，客户端请求各个服务的Api时，每个服务都需要做相同的事情，比如鉴权、限流、日志输出等。
```

![image-20200721161002001](./IMG.assets/SpringCloud.assets/image-20200721161002001.png)

![image-20200721161421845](./IMG.assets/SpringCloud.assets/image-20200721161421845.png)

```markdown
# 2.使用内置过滤器
```

![image-20200721152425733](./IMG.assets/SpringCloud.assets/image-20200721152425733.png)

```markdown
- 更多参加官网:https://cloud.spring.io/spring-cloud-static/spring-cloud-gateway/2.2.3.RELEASE/reference/html/#gatewayfilter-factories

- 使用方式如下:
```

```yml
spring:
  application:
    name: gateway
  cloud:
    gateway:
      routes:
        - id: product_route
          #uri: http://localhost:9998/
          uri: lb://products     # lb: 使用负载均衡策略   products代表注册中心的具体服务名
          predicates:
            - Path=/product/**
            #- After=2020-07-30T09:45:49.078+08:00[Asia/Shanghai]
          filters:
            - AddRequestParameter=id,34
            - AddResponseHeader=username,chenyn
```

```markdown
# 3.使用自定义filter
```

```java
@Configuration
@Slf4j
public class CustomGlobalFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("进入自定义的filter");
        if(exchange.getRequest().getQueryParams().get("username")!=null){
            log.info("用户身份信息合法,放行请求继续执行!!!");
            return chain.filter(exchange);
        }
        log.info("非法用户,拒绝访问!!!");
       return exchange.getResponse().setComplete();
    }

    @Override
    public int getOrder() {  //filter 数字越小filter越先执行
        return -1;           //-1  最先执行
    }
}
```

