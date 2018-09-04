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

## 异常

### Cannot execute request on any known server

- 未找到服务中心，检查服务中心启动状态、配置（端口等）
