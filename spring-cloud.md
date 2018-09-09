# Spring Cloud 微服务

## 服务配置和发现中心

用户发现、注册、管理服务的地方。

### Eureka

#### 注解

- @EnableEurekaServer 启用 Eureka

#### 配置

```
spring:
  application:
    name: eureka-server # 服务中心名
eureka:
  instance:
    hostname: localhost # 服务中心地址
  client:
    register-with-eureka: false # 是否向 Eureka 注册，服务中心不需要向自己注册
    fetch-registry: false # 是否向 Eureka 读取信息，服务中心不需要向自己读取
```

### ConSul

## 服务消费

### 基础

使用方式：
- 向eureka服务中心注册 

```
eureka.client.serviceUrl.defaultZone=http://localhost:1001/eureka/
```

- @EnableDiscoveryClient 启用服务端
- 注入 LoadBalancerClient 
- 用 LoadBalancerClient，加上参数 Client 选出服务实例 ServiceInstance 
- 根据 ServiceInstance 信息，组装接口并调用 
```
String url = "http://" + serviceInstance.getHost() + ":" + serviceInstance.getPort() + "/dc";
```

#### 类

- org.springframework.cloud.client.discovery.DiscoveryClient 服务的一些信息

### Ribbon

简化了服务调用。

使用方式：
- 在 RestTemplate Bean 上增加 @LoadBalanced
- 把服务名组装在接口，直接调用 

```
return restTemplate.getForObject("http://eureka-client/dc", String.class);
```

### Feign

简化了服务调用。

使用方式：
- @EnableFeignClients 启用 Feign
- 编写服务相关接口

```
@FeignClient("eureka-client") // 服务名
public interface DcClient {

    @GetMapping("/dc") // 服务地址
    String consumer();

}
```
- 注入服务接口直接使用

```
return dcClient.consumer();
```

## 配置中心

### 基础

#### 服务端
- @EnableConfigServer 启用配置中心
- 配置配置文件地址

```
  cloud:
    config:
      server:
        git:
          uri: http://git.oschina.net/didispace/config-repo-demo/
```
- 访问配置中心，按如下规则

```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
上面的url会映射{application}-{profile}.properties对应的配置文件，其中{label}对应Git上不同的分支，默认为master。我们可以尝试构造不同的url来访问不同的配置内容，比如，要访问master分支，config-client应用的dev环境，就可以访问这个url：
```

#### 客户端

- 配置配置中心的位置，**bootstrap.properties**

```
  cloud:
    config:
      uri: http://localhost:1201/
      profile: default
      label: master
```

- 加载配置文件，按照下面的规则加载

```
spring.application.name：对应配置文件规则中的{application}部分
spring.cloud.config.profile：对应配置文件规则中的{profile}部分
spring.cloud.config.label：对应配置文件规则中的{label}部分
spring.cloud.config.uri：配置中心config-server的地址
```

### 配置中心注册为服务

#### 客户端

- 当作服务注册即可

#### 客户端

- 在 bootstrap.properties中，配置读取的配置中心服务

```
spring.cloud.config.discovery.enabled=true
spring.cloud.config.discovery.serviceId=config-server
spring.cloud.config.profile=dev
```

## 断路器

### 基础

- 启用断路器，@EnableHystrix
- @HystrixCommand 配置在需要调用接口的函数上；编写断路器断路时，调用的函数，并配置在 @HystrixCommand；修改服务消费调用的函数，调用 @HystrixCommand 注解上的函数

```
@RestController
public class DcController {

    @Autowired
    ConsumerService consumerService;

    @GetMapping("/consumer")
    public String dc() {
        return consumerService.consumer();
    }

    @Service
    class ConsumerService {

        @Autowired
        RestTemplate restTemplate;

        @HystrixCommand(fallbackMethod = "fallback")
        public String consumer() {
            return restTemplate.getForObject("http://eureka-client/dc", String.class);
        }

        public String fallback() {
            return "fallback";
        }

    }

}
```

### 监控

- @EnableHystrixDashboard 启用监控
- 访问 /hystrix，输入参数开始监控

```
默认的集群监控：通过URLhttp://turbine-hostname:port/turbine.stream开启，实现对默认集群的监控。
指定的集群监控：通过URLhttp://turbine-hostname:port/turbine.stream?cluster=[clusterName]开启，实现对clusterName集群的监控。
单体应用的监控：通过URLhttp://hystrix-app:port/hystrix.stream开启，实现对具体某个服务实例的监控。
```

- 详细参数参见 [Spring Cloud构建微服务架构：Hystrix监控面板【Dalston版】](http://blog.didispace.com/spring-cloud-starter-dalston-5-1/)
- 进阶参见 [Spring Cloud构建微服务架构：Hystrix监控数据聚合【Dalston版】](http://blog.didispace.com/spring-cloud-starter-dalston-5-2/)

## 服务网关

### 转发

- @EnableZuulProxy 启用服务网关
- 注册到服务中心
- 自动转发规则如下

```
当我们这里构建的api-gateway应用启动并注册到eureka之后，服务网关会发现上面我们启动的两个服务eureka-client和eureka-consumer，这时候Zuul就会创建两个路由规则。每个路由规则都包含两部分，一部分是外部请求的匹配规则，另一部分是路由的服务ID。针对当前示例的情况，Zuul会创建下面的两个路由规则：

转发到eureka-client服务的请求规则为：/eureka-client/**
转发到eureka-consumer服务的请求规则为：/eureka-consumer/**
```

- 配置转发规则

```
对于面向服务的路由配置，除了使用path与serviceId映射的配置方式之外，还有一种更简洁的配置方式：zuul.routes.<serviceId>=<path>，其中<serviceId>用来指定路由的具体服务名，<path>用来配置匹配的请求表达式。比如下面的例子，它的路由规则等价于上面通过path与serviceId组合使用的配置方式。
zuul.routes.user-service=/user-service/**

或
zuul.routes.user-service.path=/user-service/**
zuul.routes.user-service.serviceId=user-service
```

### 过滤器

- 继承 ZuulFilter
- 将继承的类注册成 Bean

### Swagger 汇总

- 参见 [Spring Cloud Zuul中使用Swagger汇总API接口文档](http://blog.didispace.com/Spring-Cloud-Zuul-use-Swagger-API-doc/)

## 异常

### Cannot execute request on any known server

- 未找到服务中心，检查服务中心启动状态、配置（端口等）
