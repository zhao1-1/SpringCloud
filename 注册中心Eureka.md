#### Eureka

```markdown
# 0.简介
- https://github.com/Netflix/eureka/wiki
- Eureka是Netflix开发的服务发现框架，本身是一个基于REST的服务。SpringCloud将它集成在其子项目spring-cloud-netflix中，以实现SpringCloud的服务注册和发现功能。
  Eureka包含两个组件：Eureka Server和Eureka Client。
```

单体应用  ------>  分类服务   商品服务  订单服务 用户服务......

Eureka Server 组件 :  服务注册中心组件    管理所有服务  支持所有服务注册

Eureka Client 组件 :   分类服务  商品服务  订单服务(微服务)

##### 开发Eureka Server

```markdown
# 1.创建项目并引入eureka server依赖
```

```xml
<!--引入 eureka server-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

```markdown
# 2.编写配置application.properties
```

```properties
server.port=8761														#执行服务端口
spring.application.name=eurekaserver 									#指定服务名称 唯一标识
eureka.client.service-url.defaultZone=http://localhost:8761/eureka		#指定服务注册中心的地址
```

```markdown
# 3.开启Eureka Server,入口类加入注解
```

```java
@SpringBootApplication
@EnableEurekaServer
public class Eurekaserver8761Application {
    public static void main(String[] args) {
        SpringApplication.run(Eurekaserver8761Application.class, args);
    }
}
```

```markdown
# 4.访问Eureka的服务注册页面
- http://localhost:8761
```

![image-20200709161916871](./IMG.assets/SpringCloud.assets/image-20200709161916871.png)

```markdown
# 5.虽然能看到管理界面为什么项目启动控制台报错? eureka server 服务注册中心 & client 微服务
```

![image-20200709162307608](./IMG.assets/SpringCloud.assets/image-20200709162307608.png)

```markdown
- 出现上述问题原因:eureka组件包含 eurekaserver 和 eurekaclient。server是一个服务注册中心,用来接受客户端的注册。client的特性会让当前启动的服务把自己作为eureka的客户端进行服务中心的注册,当项目启动时服务注册中心还没有创建好,所以找我不到服务的客户端组件就直接报错了，当启动成功服务注册中心创建好了，日后client也能进行注册，就不再报错啦！
```

```markdown
# 6.关闭Eureka自己注册自己
```

```properties
server.port=8761
spring.application.name=eurekaserver
eureka.client.service-url.defaultZone=http://localhost:8761/eureka
eureka.client.register-with-eureka=false			# 不再将自己同时作为客户端进行注册  
eureka.client.fetch-registry=false					# 关闭作为客户端时从eureka server获取服务信息
```

```markdown
# 7.再次启动,当前应用就是一个单纯Eureka Server,控制器也不再报错
```

![image-20200709163630273](./IMG.assets/SpringCloud.assets/image-20200709163630273.png)

##### 开发Eureka Client 

```markdown
# 1.创建项目并引入eureka client依赖
```

```xml
<!--引入eureka client-->
<dependency>
  <groupId>org.springframework.cloud</groupId>
  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```markdown
# 2.编写配置application.properties
```

```properties
server.port=8888														# 服务端口号
spring.application.name=eurekaclient8888								# 服务名称唯一标识
eureka.client.service-url.defaultZone=http://localhost:8761/eureka		# eureka注册中心地址
```

```markdown
# 3.开启eureka客户端加入注解
```

```java
@SpringBootApplication
@EnableEurekaClient
public class Eurekaclient8888Application {
    public static void main(String[] args) {
        SpringApplication.run(Eurekaclient8888Application.class, args);
    }
}
```

```markdown
# 4.启动之前的8761的服务注册中心,在启动eureka客户端服务
```

![image-20200709164622017](./IMG.assets/SpringCloud.assets/image-20200709164622017.png)

```markdown
# 5.查看eureka server的服务注册情况
```

![image-20200709164729870](./IMG.assets/SpringCloud.assets/image-20200709164729870.png)

##### eureka自我保护机制

```markdown
# 0.服务频繁启动时 EurekaServer出现警告
- EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.
```

![image-20200709171532408](./IMG.assets/SpringCloud.assets/image-20200709171532408.png)

```markdown
# 1.自我保护机制
- 官网地址: https://github.com/Netflix/eureka/wiki/Server-Self-Preservation-Mode
- 默认情况下，如果Eureka Server在一定时间内（默认90秒）没有接收到某个微服务实例的心跳，Eureka Server将会移除该实例。但是当网络分区故障发生时，微服务与Eureka Server之间无法正常通信，而微服务本身是正常运行的，此时不应该移除这个微服务，所以引入了自我保护机制。Eureka Server在运行期间会去统计心跳失败比例在 15 分钟之内是否低于 85%，如果低于 85%，Eureka Server 会将这些实例保护起来，让这些实例不会过期。这种设计的哲学原理就是"宁可信其有不可信其无!"。自我保护模式正是一种针对网络异常波动的安全保护措施，使用自我保护模式能使Eureka集群更加的健壮、稳定的运行。

# 2.在eureka server端关闭自我保护机制
```

```properties
eureka.server.enable-self-preservation=false		# 关闭自我保护
eureka.server.eviction-interval-timer-in-ms=3000	# 超时3s自动清除
```

```markdown
# 3.微服务修改减短服务心跳的时间
```

```properties
eureka.instance.lease-expiration-duration-in-seconds=10		# 用来修改eureka server默认接受心跳的最大时间 默认是90s
eureka.instance.lease-renewal-interval-in-seconds=5     	# 指定客户端多久向eureka server发送一次心跳 默认是30s
```

```markdown
# 4.尽管如此关闭自我保护机制还是会出现警告
- THE SELF PRESERVATION MODE IS TURNED OFF. THIS MAY NOT PROTECT INSTANCE EXPIRY IN CASE OF NETWORK/OTHER PROBLEMS.
- `官方并不建议在生产情况下关闭
```

![image-20200709232933894](./IMG.assets/SpringCloud.assets/image-20200709232933894.png)

##### eureka 停止更新

```markdown
# 1.官方停止更新说明
- https://github.com/Netflix/eureka/wiki
- 在1.x版本项目还是活跃的,但是在2.x版本中停止维护,出现问题后果自负!!!
```

![image-20200709233215860](./IMG.assets/SpringCloud.assets/image-20200709233215860.png)

consul  服务注册中心  启动consul服务注册中心  运行 

consul 客户端 将springcloud 客户端(微服务)
