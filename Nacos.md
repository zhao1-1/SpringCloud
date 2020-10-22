## Nacos

### 什么是Nacos  Name Service & Configurations Services

![image-20200727202422243](./IMG.assets/SpringCloud.assets/image-20200727202422243.png)

```markdown
- https://nacos.io/zh-cn/index.html
- Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。
```

- 总结:**Nacos就是微服务架构中服务注册中心以及统一配置中心,用来替换原来的(eureka,consul)以及config组件**

### 安装Nacos

```markdown
# 0.准备环境
- 1.64 bit OS，支持 Linux/Unix/Mac/Windows，推荐选用 Linux/Unix/Mac。
- 2.64 bit JDK 1.8+；下载 & 配置。
- 3.Maven 3.2.x+；下载 & 配置。

# 1.下载nacos [本次课程版本:][1.3.0版本]
- https://github.com/alibaba/nacos/releases 
```

![image-20200727202936158](./IMG.assets/SpringCloud.assets/image-20200727202936158.png)

```markdown
# 2.解压缩安装包到指定位置
- bin  			启动nacos服务的脚本目录
- conf 			nacos的配置文件目录
- target 		nacos的启动依赖存放目录
- data		  nacos启动成功后保存数据的目录
```

![image-20200727203852405](./IMG.assets/SpringCloud.assets/image-20200727203852405.png)

````markdown
# 3.启动安装服务
- linux/unix/mac启动
	打开终端进入nacos的bin目录执行如下命令 
	./startup.sh -m standalone

- windows启动
	在 cmd中 
	执行 startup.cmd -m standalone 或者双击startup.cmd运行文件。
````

![image-20200727204207794](./IMG.assets/SpringCloud.assets/image-20200727204207794.png)

```markdown
# 4.访问nacos的web服务管理界面
- http://localhost:8848/nacos/
- 用户名 和 密码都是nacos
```

![image-20200727210727986](./IMG.assets/SpringCloud.assets/image-20200727210727986.png)

### 开发服务注册到nacos

```markdown
# 0.创建项目并引入依赖
```

```xml
<!--引入nacos client的依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

![image-20200727212459690](./IMG.assets/SpringCloud.assets/image-20200727212459690.png)

```markdown
# 1.配置注册地址
```

```properties
server.port=8789 												           # 指定当前服务端口
spring.application.name=nacosclient										   # 指定服务名称
spring.cloud.nacos.server-addr=localhost:8848							   # 指定nacos服务地址
spring.cloud.nacos.discovery.server-addr=${spring.cloud.nacos.server-addr} # 指定注册中心地址							
management.endpoints.web.exposure.include=*								   #暴露所有web端点
```

```markdown
# 2.加入启动服务注册注解 [注意:][新版本之后这步可以省略不写]
```

![image-20200727213320726](./IMG.assets/SpringCloud.assets/image-20200727213320726.png)

```markdown
# 3.查看nacos的服务列表
```

![image-20200727213221604](./IMG.assets/SpringCloud.assets/image-20200727213221604.png)

### 使用nacos作为配置中心

#### 1.从nacos获取配置

```markdown
# 1.创建项目并引入nacons配置中心依赖
```

```xml
<!--引入nacos client依赖-->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>

<!--引入nacos config 依赖-->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

```markdown
# 2.配置 配置中心地址
```

```properties
spring.cloud.nacos.server-addr=localhost:8848						# 远程配置中心的地址
spring.application.name=config										# 指定读取文件的前缀
spring.profiles.active=prod											# 指定读取文件的具体环境

spring.cloud.nacos.config.server-addr=${spring.cloud.nacos.server-addr}
spring.cloud.nacos.config.group=DEFAULT_GROUP						# 读取配置的分组
spring.cloud.nacos.config.prefix=${spring.application.name}
spring.cloud.nacos.config.file-extension=properties					# 指定读取文件后缀
```

```markdown
# 3.在nacos中创建配置
```

![image-20200728211633327](./IMG.assets/SpringCloud.assets/image-20200728211633327.png)

![image-20200728211924796](./IMG.assets/SpringCloud.assets/image-20200728211924796.png)

```markdown
# 4.编写控制器测试配置读取情况
```

```java
@RestController
@Slf4j
public class HelloController {
    //注入配置
    @Value("${user.name}")
    private String username;
    @GetMapping("/hello/config")
    public String config(){
        log.info("用户名: [{}]",username);
        return username;
    }
}
```

```markdown
# 5.启动项目方式测试配置读取
```

![image-20200728212221271](./IMG.assets/SpringCloud.assets/image-20200728212221271.png)

![image-20200728212249215](./IMG.assets/SpringCloud.assets/image-20200728212249215.png)

#### 2. DataId

```markdown
# 1.DataId
- 用来读取远程配置中心的中具体配置文件其完整格式如下:
- ${prefix}-${spring.profile.active}.${file-extension}

	a. prefix 默认为 spring.application.name 的值，也可以通过配置项 spring.cloud.nacos.config.prefix来配置。
	
	b. spring.profile.active 即为当前环境对应的 profile，详情可以参考 Spring Boot文档。 注意：当 spring.profile.active 为空时，对应的连接符 - 也将不存在，dataId 的拼接格式变成 ${prefix}.${file-extension}
	
	c. file-exetension 为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file-extension 来配置。目前只支持 properties 和 yaml 类型。
```

#### 3.实现自动配置刷新

```markdown
# 1.自动刷新
- 默认情况下nacos已经实现了自动配置刷新功能,如果需要刷新配置直接在控制器中加入@RefreshScope注解即可
```

```java
@RestController
@Slf4j
@RefreshScope
public class HelloController {
    //注入配置
    @Value("${user.name}")
    private String username;
    @GetMapping("/hello/config")
    public String config(){
        log.info("用户名: [{}]",username);
        return username;
    }
}
```

#### 4.命名空间

```markdown
# 1.命名空间(namespace)
- https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-config
- namespace命名空间是nacos针对于企业级开发设计用来针对于不同环境的区分,比如正在企业开发时有测试环境,生产环境,等其他环境,因此为了保证不同环境配置实现隔离,提出了namespace的概念,默认在nacos中存在一个public命名空间所有配置在没有指定命名空间时都在这个命名空间中获取配置,在实际开发时可以针对于不能环境创建不同的namespace空间。默认空间不能删除!
```

![image-20200728220906225](./IMG.assets/SpringCloud.assets/image-20200728220906225.png)

```markdown
# 2.创建其他命名空间
- 每个命名空间都有一个唯一id,这个id是读取配置时指定空间的唯一标识
```

![image-20200728221059584](./IMG.assets/SpringCloud.assets/image-20200728221059584.png)

![image-20200728221139206](./IMG.assets/SpringCloud.assets/image-20200728221139206.png)

```markdown
# 3.在配置列表查看空间
```

![image-20200728221221582](./IMG.assets/SpringCloud.assets/image-20200728221221582.png)

```markdown
# 4.在指定空间下载创建配置文件
```

![image-20200728222410336](./IMG.assets/SpringCloud.assets/image-20200728222410336.png)

```markdown
# 5.项目中使用命名空间指定配置
```

![image-20200728223100749](./IMG.assets/SpringCloud.assets/image-20200728223100749.png)

```markdown
# 6.测试配置
```

![image-20200728223125420](./IMG.assets/SpringCloud.assets/image-20200728223125420.png)

#### 5.配置分组

```markdown
# 1.配置分组(group)
- 配置分组是对配置集进行分组，通过一个有意义的字符串（如 Buy 或 Trade ）来表示，不同的配置分组下可以有相同的配置集（Data ID）。当您在 Nacos 上创建一个配置时，如果未填写配置分组的名称，则配置分组的名称默认采用 DEFAULT_GROUP 。配置分组的常见场景：可用于区分不同的项目或应用，例如：学生管理系统的配置集可以定义一个group为：STUDENT_GROUP。
```

![image-20200728223745192](./IMG.assets/SpringCloud.assets/image-20200728223745192.png)

```markdown
# 2.创建分组
```

![image-20200728223921240](./IMG.assets/SpringCloud.assets/image-20200728223921240.png)

![image-20200728224034473](./IMG.assets/SpringCloud.assets/image-20200728224034473.png)

```markdown
# 3.读取不同分组的配置
```

![image-20200728224128019](./IMG.assets/SpringCloud.assets/image-20200728224128019.png)

---

