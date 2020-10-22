### 基于Ribbon的服务调用

```markdown
# 0.说明
- 官方网址: https://github.com/Netflix/ribbon
- Spring Cloud Ribbon是一个基于HTTP和TCP的`客户端`负载均衡工具，它基于Netflix Ribbon实现。通过Spring Cloud的封装，可以让我们轻松地将面向服务的REST模版请求自动转换成客户端负载均衡的服务调用。
```

#### （1）Ribbon 服务调用

```markdown
# 1.项目中引入依赖
- 说明: 
	1.如果使用的是eureka client 和 consul client,无须引入依赖,因为在eureka,consul中默认集成了ribbon组件
	2.如果使用的client中没有ribbon依赖需要显式引入如下依赖
```

```xml
<!--引入ribbon依赖-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```

```markdown
# 2.查看consul client中依赖的ribbon
```

![image-20200713140804414](./IMG.assets/SpringCloud.assets/image-20200713140804414.png)

```markdown
# 3.使用restTemplate + ribbon进行服务调用
- 使用 discovery client  进行客户端调用
- 使用 loadBalanceClient 进行客户端调用
- 使用 @loadBalanced     进行客户端调用
```

```markdown
# 3.1 使用discovery Client形式调用
```

```java
@Autowired
private DiscoveryClient discoveryClient;

//获取服务列表
List<ServiceInstance> products = discoveryClient.getInstances("服务ID");
for (ServiceInstance product : products) {
  log.info("服务主机:[{}]",product.getHost());
  log.info("服务端口:[{}]",product.getPort());
  log.info("服务地址:[{}]",product.getUri());
  log.info("====================================");
}
```

```markdown
# 3.2 使用loadBalance Client形式调用
```

```java
@Autowired
private LoadBalancerClient loadBalancerClient;
//根据负载均衡策略选取某一个服务调用
ServiceInstance product = loadBalancerClient.choose("服务ID");	//默认：轮询策略
log.info("服务主机:[{}]",product.getHost());
log.info("服务端口:[{}]",product.getPort());
log.info("服务地址:[{}]",product.getUri());
```

```markdown
# 3.3 使用@loadBalanced
```

```java
//1.整合restTemplate + ribbon
@Bean
@LoadBalanced
public RestTemplate getRestTemplate(){
  return new RestTemplate();
}
//2.调用服务位置注入RestTemplate
@Autowired
private RestTemplate restTemplate;
//3.调用
String forObject = restTemplate.getForObject("http://服务ID/hello/hello?name=" + name, String.class);
```

#### （2）Ribbon负载均衡策略

```markdown
# 1.ribbon负载均衡算法
- RoundRobinRule         		轮询策略	按顺序循环选择 Server 
- RandomRule             		随机策略	随机选择 Server  
- WeightedResponseTimeRule  	响应时间加权策略   
	`根据平均响应的时间计算所有服务的权重，响应时间越快服务权重越大被选中的概率越高；
	`刚启动时如果统计信息不足，则使用RoundRobinRule策略，等统计信息足够会切换到
- RetryRule                 	重试策略          
	`先按照RoundRobinRule的策略获取服务，如果获取失败则在制定时间内进行重试，获取可用的服务。	
- BestAviableRule           	最低并发策略     
	`会先过滤掉由于多次访问故障而处于断路器跳闸状态的服务，然后选择一个并发量最小的服务。
- AvailabilityFilteringRule 	可用过滤策略
 	`会先过滤由于多次访问故障而处于断路器跳闸状态的服务，还有并发的连接数量超过阈值的服务，然后对剩余的服务列表按照轮询策略进行访问。
```

![image-20200713162940968](./IMG.assets/SpringCloud.assets/image-20200713162940968.png)

#### （3）修改服务的默认负载均衡策略

```markdown
# 1.修改服务默认随机策略
- 服务id.ribbon.NFLoadBalancerRuleClassName=com.netflix.loadbalancer.RandomRule
	`下面的products为服务的唯一标识
```

```properties
products.ribbon.NFLoadBalancerRuleClassName=com.netflix.loadbalancer.RandomRule
```

![image-20200713163722927](./IMG.assets/SpringCloud.assets/image-20200713163722927.png)

#### （4）Ribbon停止维护

```markdown
# 1.官方停止维护说明
- https://github.com/Netflix/ribbon
```

![image-20200713195706787](./IMG.assets/SpringCloud.assets/image-20200713195706787.png)

---

## 
