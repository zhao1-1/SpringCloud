

## OpenFeign组件的使用

- 思考: 使用RestTemplate + ribbon已经可以完成服务间的调用，为什么还要使用feign？

```java
String restTemplateForObject = restTemplate.getForObject("http://服务名/url?参数" + name, String.class);
```

```markdown
# 存在问题:
- 1.每次调用服务都需要写这些代码,存在大量的代码冗余
- 2.服务地址如果修改,维护成本增高
- 3.使用时不够灵活
```

### OpenFeign 组件

```markdown
# 0.说明
- https://cloud.spring.io/spring-cloud-openfeign/reference/html/
- Feign是一个声明式的伪Http客户端，它使得写Http客户端变得更简单。使用Feign，只需要创建一个接口并注解。它具有可插拔的注解特性(可以使用springmvc的注解)，可使用Feign 注解和JAX-RS注解。Feign支持可插拔的编码器和解码器。Feign默认集成了Ribbon，默认实现了负载均衡的效果并且springcloud为feign添加了springmvc注解的支持。
```

#### （1）openFeign 服务调用

```markdown
# 1.服务调用方法引入依赖OpenFeign依赖
```

```xml
<!--Open Feign依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

```markdown
# 2.入口类加入注解开启OpenFeign支持
```

```java
@SpringBootApplication
@EnableFeignClients   //开启openfeign支持
public class Users9999Application {
    public static void main(String[] args) {
        SpringApplication.run(Users9999Application.class, args);
    }
}
```

```markdown
# 3.创建一个客户端调用接口
```

```java
//value属性用来指定:调用服务名称
@FeignClient("PRODUCTS")
public interface ProductClient {
  
    @GetMapping("/product/findAll") 	//书写服务调用路径
    String findAll();
}
```

```markdown
# 4.使用feignClient客户端对象调用服务
```

```java
//注入客户端对象
@Autowired
private ProductClient productClient;

@GetMapping("/user/findAllFeignClient")
public String findAllFeignClient(){
  log.info("通过使用OpenFeign组件调用商品服务...");
  String msg = productClient.findAll();
  return msg;
}
```

```markdown
# 5.访问并测试服务
- http://localhost:9999/user/findAllFeignClient
```

![image-20200713202802056](./IMG.assets/SpringCloud.assets/image-20200713202802056.png)

#### （2）调用服务并传参

```markdown
# 0.说明
- 服务和服务之间通信,不仅仅是调用,往往在调用过程中还伴随着参数传递,接下来重点来看看OpenFeign在调用服务时如何传递参数
```

###### GET方式调用服务传递参数

```markdown
# 1.GET方式调用服务传递参数
- 在商品服务中加入需要传递参数的服务方法来进行测试
- 在用户服务中进行调用商品服务中需要传递参数的服务方法进行测试
```

```java
// 1.商品服务中添加如下方法
@GetMapping("/product/findOne")
public Map<String,Object> findOne(String productId){
    log.info("商品服务查询商品信息调用成功,当前服务端口:[{}]",port);
    log.info("当前接收商品信息的id:[{}]",productId);
    Map<String, Object> map = new HashMap<String,Object>();
    map.put("msg","商品服务查询商品信息调用成功,当前服务端口: " + port);
    map.put("status",true);
    map.put("productId",productId);
    return map;
}
```

```java
//2.用户服务中在product客户端中声明方法
@FeignClient("PRODUCTS")
public interface ProductClient { 
	@GetMapping("/product/findOne")
 	String findOne(@RequestParam("productId") String productId);
}
```

```java
//3.用户服务中调用并传递参数
//注入客户端对象
@Autowired
private ProductClient productClient;

@GetMapping("/feign/test1")
public Map<String,Object> test1(String id){
  log.info("用来测试Openfiegn的GET方式参数传递");
  Map<String, Object> msg = productClient.findOne(id);
  log.info("调用返回信息:[{}]",msg);
  return msg;
}
```

```markdown
# 测试访问
```

![image-20200713204827577](./IMG.assets/SpringCloud.assets/image-20200713204827577.png)

![image-20200713204851383](./IMG.assets/SpringCloud.assets/image-20200713204851383.png)

###### post方式调用服务传递参数

```markdown
# 2.post方式调用服务传递参数
- 在商品服务中加入需要传递参数的服务方法来进行测试
- 在用户服务中进行调用商品服务中需要传递参数的服务方法进行测试
```

```java
//1.商品服务加入post方式请求并接受name
@PostMapping("/product/save")
public Map<String,Object> save(String name){
  log.info("商品服务保存商品调用成功,当前服务端口:[{}]",port);
  log.info("当前接收商品名称:[{}]",name);
  Map<String, Object> map = new HashMap<String,Object>();
  map.put("msg","商品服务查询商品信息调用成功,当前服务端口: "+port);
  map.put("status",true);
  map.put("name",name);
  return map;
}
```

```java
//2.用户服务中在product客户端中声明方法
//value属性用来指定:调用服务名称
@FeignClient("PRODUCTS")
public interface ProductClient {
    @PostMapping("/product/save")
    String save(@RequestParam("name") String name);
}
```

```java
//3.用户服务中调用并传递参数
@Autowired
private ProductClient productClient;

@GetMapping("/user/save")
public String save(String productName){
  log.info("接收到的商品信息名称:[{}]",productName);
  String save = productClient.save(productName);
  log.info("调用成功返回结果: "+save);
  return save;
}
```

```markdown
# 测试访问
```

![image-20200713205919054](./IMG.assets/SpringCloud.assets/image-20200713205919054.png)

![image-20200713210001477](./IMG.assets/SpringCloud.assets/image-20200713210001477.png)

```markdown
# 2.传递对象类型参数
- 商品服务定义对象
- 商品服务定义对象接收方法
- 用户服务调用商品服务定义对象参数方法进行参数传递
```

```java
//1.商品服务定义对象
@Data
public class Product {
    private Integer id;
    private String name;
    private Date bir;
}
```

```java
//2.商品服务定义接收对象的方法
@PostMapping("/product/saveProduct")
public Map<String,Object> saveProduct(@RequestBody Product product){
  log.info("商品服务保存商品信息调用成功,当前服务端口:[{}]",port);
  log.info("当前接收商品名称:[{}]",product);
  Map<String, Object> map = new HashMap<String,Object>();
  map.put("msg","商品服务查询商品信息调用成功,当前服务端口: "+port);
  map.put("status",true);
  map.put("product",product);
  return map;
}

```

```java
//3.将商品对象复制到用户服务中
//4.用户服务中在product客户端中声明方法
@FeignClient("PRODUCTS")
public interface ProductClient {
  @PostMapping("/product/saveProduct")
  String saveProduct(@RequestBody Product product);
}
//注意:服务提供方和调用方一定要加入@RequestBody注解 
```

```java
// 5.在用户服务中调用保存商品信息服务
//注入客户端对象
@Autowired
private ProductClient productClient;

@GetMapping("/user/saveProduct")
public String saveProduct(Product product){
  log.info("接收到的商品信息:[{}]",product);
  String save = productClient.saveProduct(product);
  log.info("调用成功返回结果: "+save);
  return save;
}
```

```markdown
# 测试
```

![image-20200713211338475](./IMG.assets/SpringCloud.assets/image-20200713211338475.png)

![image-20200713211402844](./IMG.assets/SpringCloud.assets/image-20200713211402844.png)

#### （3）OpenFeign超时设置

```markdown
# 0.超时说明
- 默认情况下,openFiegn在进行服务调用时,要求服务提供方处理业务逻辑时间必须在1S内返回,如果超过1S没有返回则OpenFeign会直接报错,不会等待服务执行,但是往往在处理复杂业务逻辑是可能会超过1S,因此需要修改OpenFeign的默认服务调用超时时间。
- 调用超时会出现如下错误：
```

```markdown
# 1.模拟超时
- 服务提供方加入线程等待阻塞
```

![image-20200713213322984](./IMG.assets/SpringCloud.assets/image-20200713213322984.png)

```markdown
# 2.进行客户端调用
```

![image-20200713213415230](./IMG.assets/SpringCloud.assets/image-20200713213415230.png)

```markdown
# 3.修改OpenFeign默认超时时间
```

```properties
feign.client.config.PRODUCTS.connectTimeout=5000		# 配置`指定`服务连接超时
feign.client.config.PRODUCTS.readTimeout=5000		  	# 配置`指定`服务等待超时
#feign.client.config.default.connectTimeout=5000  		# 配置`所有`服务连接超时
#feign.client.config.default.readTimeout=5000			# 配置`所有`服务等待超时
```

#### （4）OpenFeign调用详细日志展示

```markdown
# 0.说明
- 往往在服务调用时我们需要详细展示feign的日志,默认feign在调用是并不是最详细日志输出,因此在调试程序时应该开启feign的详细日志展示。feign对日志的处理非常灵活可为每个feign客户端指定日志记录策略，每个客户端都会创建一个logger默认情况下logger的名称是feign的全限定名需要注意的是，feign日志的打印只会DEBUG级别做出响应。
- 我们可以为feign客户端配置各自的logger.level对象，告诉feign记录那些日志logger.lever有以下的几种值
	`NONE  不记录任何日志
	`BASIC 仅仅记录请求方法，url，响应状态代码及执行时间
	`HEADERS 记录Basic级别的基础上，记录请求和响应的header
	`FULL 记录请求和响应的header，body和元数据
```

```markdown
# 1.开启日志展示
```

```properties
feign.client.config.PRODUCTS.loggerLevel=full  #开启指定服务日志展示
#feign.client.config.default.loggerLevel=full  #全局开启服务日志展示
logging.level.com.baizhi.feignclients=debug    #指定feign调用客户端对象所在包,必须是debug级别
```

```markdown
# 2.测试服务调用查看日志
```

![image-20200713215108861](./IMG.assets/SpringCloud.assets/image-20200713215108861.png)

---

## 
