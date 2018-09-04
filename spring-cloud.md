# Spring Cloud 微服务

## Eureka

Eureka 是服务配置和发现中心。

### 注解

- @EnableEurekaServer 启用 Eureka

### 配置

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

