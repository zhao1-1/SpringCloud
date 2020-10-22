## Hystrix组件使用

### JMeter

+ 压力测试工具
+ Apache

### Hystrix组件

![image-20200715123359665](./IMG.assets/SpringCloud.assets/image-20200715123359665.png)

In a distributed environment, inevitably some of the many service dependencies will fail. Hystrix is a library that helps you control the interactions between these distributed services by adding latency tolerance and fault tolerance logic. Hystrix does this by isolating points of access between the services, stopping cascading failures across them, and providing fallback options, all of which improve your system’s overall resiliency.														--[摘自官方]

```markdown
# 0.说明
- https://github.com/Netflix/Hystrix
- 译: 在分布式环境中，许多服务依赖项不可避免地会失败。Hystrix是一个库，它通过添加延迟容忍和容错逻辑来帮助您控制这些分布式服务之间的交互。Hystrix通过隔离服务之间的访问点、停止它们之间的级联故障以及提供后备选项来实现这一点，所有这些都可以提高系统的整体弹性。

- 通俗定义: Hystrix是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统中，许多依赖不可避免的会调用失败，超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障(服务雪崩现象)，提高分布式系统的弹性。
```

#### （1）服务雪崩

```markdown
# 1.服务雪崩
- 在微服务之间进行服务调用是由于某一个服务故障，导致级联服务故障的现象，称为雪崩效应。雪崩效应描述的是提供方不可用，导致消费方不可用并将不可用逐渐放大的过程。
# 2.图解雪崩效应
- 如存在如下调用链路:
```

![image-20200715151728240](./IMG.assets/SpringCloud.assets/image-20200715151728240.png)

```markdown
- 而此时，Service A的流量波动很大，流量经常会突然性增加！那么在这种情况下，就算Service A能扛得住请求，Service B和Service C未必能扛得住这突发的请求。此时，如果Service C因为抗不住请求，变得不可用。那么Service B的请求也会阻塞，慢慢耗尽Service B的线程资源，Service B就会变得不可用。紧接着，Service A也会不可用，这一过程如下图所示
```

![image-20200715152623313](./IMG.assets/SpringCloud.assets/image-20200715152623313.png)

#### （2）服务熔断

```markdown
# 服务熔断
- “熔断器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控，某个异常条件被触发，直接熔断整个服务。向调用方法返回一个符合预期的、可处理的备选响应(FallBack)，而不是长时间的等待或者抛出调用方法无法处理的异常，就保证了服务调用方的线程不会被长时间占用，避免故障在分布式系统中蔓延，乃至雪崩。如果目标服务情况好转则恢复调用。服务熔断是解决服务雪崩的重要手段。
```

```markdown
# 服务熔断图示
```

![image-20200717085946385](./IMG.assets/SpringCloud.assets/image-20200717085946385.png)

#### （3）服务降级

```markdown
# 服务降级说明
- 服务压力剧增的时候根据当前的业务情况及流量对一些服务和页面有策略的降级，以此环节服务器的压力，以保证核心任务的进行。同时保证部分甚至大部分任务客户能得到正确的相应。也就是当前的请求处理不了了或者出错了，给一个默认的返回。

-  通俗: 关闭系统中边缘服务 保证系统核心服务的正常运行  称之为服务降级
   //双12  淘宝 删除地址  确认收货  删除订单   取消支付   节省cpu  内存
# 服务降级图示
```

![image-20200717112327729](./IMG.assets/SpringCloud.assets/image-20200717112327729.png)

#### （4）降级和熔断总结

```markdown
# 1.共同点
- 目的很一致，都是从可用性可靠性着想，为防止系统的整体缓慢甚至崩溃，采用的技术手段；
- 最终表现类似，对于两者来说，最终让用户体验到的是某些功能暂时不可达或不可用；
- 粒度一般都是服务级别，当然，业界也有不少更细粒度的做法，比如做到数据持久层（允许查询，不允许增删改）；
- 自治性要求很高，熔断模式一般都是服务基于策略的自动触发，降级虽说可人工干预，但在微服务架构下，完全靠人显然不可能，开关预置、配置中心都是必要手段；

# 2.异同点
- 触发原因不太一样，服务熔断一般是某个服务（下游服务）故障引起，而服务降级一般是从整体负荷考虑；
- 管理目标的层次不太一样，熔断其实是一个框架级的处理，每个微服务都需要（无层级之分），而降级一般需要对业务有层级之分（比如降级一般是从最外围服务开始）

# 3.总结
- 熔断必会触发降级,所以熔断也是降级一种,区别在于熔断是对调用链路的保护,而降级是对系统过载的一种保护处理
```

#### （5）服务熔断的实现

```markdown
#服务熔断的实现思路
- 引入hystrix依赖,并开启熔断器(断路器)
- 模拟降级方法
- 进行调用测试
```

```markdown
# 1.项目中引入hystrix依赖
```

```xml
<!--引入hystrix-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

```markdown
# 2.开启断路器
```

```java
@SpringBootApplication
@EnableCircuitBreaker		//用来开启断路器
public class Products9998Application {
    public static void main(String[] args) {
        SpringApplication.run(Products9998Application.class, args);
    }
}
```

```markdown
# 3.使用HystrixCommand注解实现断路
```

```java
//服务熔断
@GetMapping("/product/break")
@HystrixCommand(fallbackMethod = "testBreakFall" )
public String testBreak(int id){
  log.info("接收的商品id为: "+ id);
  if(id<=0){
    throw new RuntimeException("数据不合法!!!");
  }
  return "当前接收商品id: "+id;
}

public String testBreakFall(int id){
  return "当前数据不合法: "+id;
}
```

![image-20200717090743474](./IMG.assets/SpringCloud.assets/image-20200717090743474.png)

```markdown
# 4.访问测试
- 正常参数访问
- 错误参数访问
```

![image-20200717090841831](./IMG.assets/SpringCloud.assets/image-20200717090841831.png)

![image-20200717091028876](./IMG.assets/SpringCloud.assets/image-20200717091028876.png)

```markdown
# 5.总结
- 从上面演示过程中会发现如果触发一定条件断路器会自动打开,过了一点时间正常之后又会关闭。那么断路器打开条件是什么呢？
```

```markdown
# 6.断路器打开条件
- 官网: https://cloud.spring.io/spring-cloud-netflix/2.2.x/reference/html/#circuit-breaker-spring-cloud-circuit-breaker-with-hystrix
```

A service failure in the lower level of services can cause cascading failure all the way up to the user. When calls to a particular service exceed `circuitBreaker.requestVolumeThreshold` (default: 20 requests) and the failure percentage is greater than `circuitBreaker.errorThresholdPercentage` (default: >50%) in a rolling window defined by `metrics.rollingStats.timeInMilliseconds` (default: 10 seconds), the circuit opens and the call is not made. In cases of error and an open circuit, a fallback can be provided by the developer.																		--摘自官方

```markdown
# 原文翻译之后,总结打开关闭的条件:
- 1、当满足一定的阀值的时候（默认10秒内超过20个请求次数）
- 2、当失败率达到一定的时候（默认10秒内超过50%的请求失败）
- 3、到达以上阀值，断路器将会开启
- 4、当开启的时候，所有请求都不会进行转发
- 5、一段时间之后（默认是5秒），这个时候断路器是半开状态，会让其中一个请求进行转发。如果成功，断路器会关闭，若失败，继续开启。重复4和5步。
```

![image-20200717092819616](./IMG.assets/SpringCloud.assets/image-20200717092819616.png)

```markdown
# 7.默认的服务FallBack处理方法
- 如果为每一个服务方法开发一个降级,对于我们来说,可能会出现大量的代码的冗余,不利于维护,这个时候就需要加入默认服务降级处理方法
```

```java
// FallBack处理方法
public String testHystrixFallBack(String name) {
  return port + "当前服务已经被降级处理!!!,接收名称为: "+name;
}

@GetMapping("/product/hystrix")
@HystrixCommand(defaultFallBack = "testHystrixFallBack")
public String testHystrix(String name) {
  log.info("接收名称为: " + name);
  int n = 1/0;
  return "服务[" + port + "]响应成功,当前接收名称为:" + name;
}
```

#### （6）服务降级的实现

```markdown
# 1.客户端openfeign + hystrix实现服务降级实现
- 引入hystrix依赖
- 配置文件开启feign支持hystrix
- 在feign客户端调用加入fallback指定降级处理
- 开发降级处理方法
```

```markdown
# 2.开启openfeign支持服务降级
```

```properties
feign.hystrix.enabled=true #开启openfeign支持降级
```

```markdown
# 3.在openfeign客户端中加入Hystrix
```

```java
@FeignClient(value = "PRODUCTS",fallback = ProductFallBack.class)
public interface ProductClient {
    @GetMapping("/product/hystrix")
    String testHystrix(@RequestParam("name") String name);
}
```

![image-20200716101101091](./IMG.assets/SpringCloud.assets/image-20200716101101091.png)

```markdown
# 4.开发fallback处理类
```

```java
@Component
public class ProductFallBack implements ProductClient {
    @Override
    public String testHystrix(String name) {
        return "我是客户端的Hystrix服务实现!!!";
    }
}
```

![image-20200717101921108](./IMG.assets/SpringCloud.assets/image-20200717101921108.png)

**注意:如果服务端降级和客户端降级同时开启,要求服务端降级方法的返回值必须与客户端方法降级的返回值一致!!!**



> 总结：
>
> + 客户端引用   --   openfeign + Hystrix + fallback方法（实现openfeign客户端接口）   --  控制调取服务的降级处理
> + 服务端引用   --                         Hystrix + fallback方法（直接在服务类内定义即可）       --  控制该服务的熔断处理



#### （7）Hystrix Dashboard

```markdown
# 0.说明
- Hystrix Dashboard的一个主要优点是它收集了关于每个HystrixCommand的一组度量。Hystrix仪表板以高效的方式显示每个断路器的运行状况。
```

![image-20200716161556743](./IMG.assets/SpringCloud.assets/image-20200716161556743.png)

```markdown
# 1.项目中引入依赖
```

```xml
<!--引入hystrix dashboard 依赖-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
```

```markdown
# 2.入口类中开启hystrix dashboard
```

```java
@SpringBootApplication
@EnableHystrixDashboard		//开启监控面板
public class Hystrixdashboard9990Application {
	public static void main(String[] args) {
		SpringApplication.run(Hystrixdashboard9990Application.class, args);
  }
}
```

```markdown
# 3.启动hystrix dashboard应用
- http://localhost:9990(dashboard端口)/hystrix
```

![image-20200717155059512](./IMG.assets/SpringCloud.assets/image-20200717155059512.png)

```markdown
# 4.监控的项目中入口类中加入监控路径配置[新版本坑],并启动监控项目
```

```java
@Bean
public ServletRegistrationBean getServlet() {
  HystrixMetricsStreamServlet streamServlet = new HystrixMetricsStreamServlet();
  ServletRegistrationBean registrationBean = new ServletRegistrationBean(streamServlet);	// 新加入的
  registrationBean.setLoadOnStartup(1);
  registrationBean.addUrlMappings("/hystrix.stream");
  registrationBean.setName("HystrixMetricsStreamServlet");
  return registrationBean;
}
```

```markdown
# 5.通过监控界面监控
```

![image-20200717155258994](./IMG.assets/SpringCloud.assets/image-20200717155258994.png)

```markdown
# 6.点击监控,一致loading,打开控制台发现报错[特别坑]
```

![image-20200717155555786](./IMG.assets/SpringCloud.assets/image-20200717155555786.png)

```markdown
# 解决方案
- 新版本中springcloud将jquery版本升级为3.4.1，定位到monitor.ftlh文件中，js的写法如下：
	$(window).load(function() 
- jquery 3.4.1已经废弃上面写法

- 修改方案 修改monitor.ftlh为如下调用方式：
	$(window).on("load",function()
	
- 编译jar源文件，重新打包引入后，界面正常响应。
```

![image-20200717160636218](./IMG.assets/SpringCloud.assets/image-20200717160636218.png)

#### （8）Hystrix停止维护

![image-20200717161223806](./IMG.assets/SpringCloud.assets/image-20200717161223806.png)

![image-20200717161400285](./IMG.assets/SpringCloud.assets/image-20200717161400285.png)

```markdown
# 官方地址:https://github.com/Netflix/Hystrix
- 翻译:Hystrix（版本1.5.18）足够稳定，可以满足Netflix对我们现有应用的需求。同时，我们的重点已经转移到对应用程序的实时性能作出反应的更具适应性的实现，而不是预先配置的设置（例如，通过自适应并发限制）。对于像Hystrix这样的东西有意义的情况，我们打算继续在现有的应用程序中使用Hystrix，并在新的内部项目中利用诸如resilience4j这样的开放和活跃的项目。我们开始建议其他人也这样做。
- Dashboard也被废弃
```

