+++
title = '初步了解SpringCloud'
date = 2024-06-07T16:10:57+08:00
lastmod = 2024-06-07T16:10:57+08:00
draft = false
keywords = []
description = ''
tags = ['分布式','微服务']
categories = []
author = 'chuimber'
+++

Spring Cloud 是一系列框架的有序集合。它利用 Spring Boot 的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册、配置中心、消息总线、负载均衡、断路器、数据监控等，都可以用 Spring Boot 的开发风格做到一键启动和部署。
# SpringCloud技术概览
![alt text](/static/image.png)
# 案例引入
## 01. 服务集群注册与发现
采用 Eureka Server 运行3个实例｛node01、node02、node03｝构建服务发现集群。在这种架构中，节点通过彼此互相注册来提高可用性，每个节点需要添加一个或多个有效的 serviceUrl 指向其他节点。每个节点都可被视为其他节点的副本。

> EurekaServerApplication.java

三个实例均为服务注册中心，此项配置相同
```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }

}
```
> application.yml | node01指向另外两台服务，registerWithEureka、fetchRegistry和单实例不同需要配置为true
```yaml
spring:
  application:
    name: eureka-server

server:
  port: 8081

eureka:
  instance:
    hostname: localhost #指定当前服务实例的主机名
  client:
    registerWithEureka: true #当前服务实例是否要向Eureka Server注册
    fetchRegistry: true #当前服务实例是否要从Eureka Server获取服务注册表
    serviceUrl:
      defaultZone: http://localhost:8082/eureka/,http://localhost:8083/eureka/ #指定服务注册中心的地址
```
> application.yml | node02指向另外两台服务
```yaml
spring:
  application:
    name: eureka-server

server:
  port: 8082

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://localhost:8081/eureka/,http://localhost:8083/eureka/
```
> application.yml | node03指向另外两台服务
```yaml
spring:
  application:
    name: eureka-server

server:
  port: 8083

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://localhost:8082/eureka/,http://localhost:8081/eureka/
```

## 02. 服务提供和负载均衡调用Eureka
提供一个基于Eurka的服务注册中心，两个服务提供者之后分别使用Ribbon、Fegin方式进行调用，测试负载均衡。
> 服务提供者模块结构

    eureka-client/src
    └── main
        ├── java
        │   └── bo
        │       └── workspace
        │           └── demo2
        │               ├── EurekaClientApplication.java
        │               └── web
        │                   └── EurekaClientController.java
        └── resources
            └── application.yml

> web/EurekaClientController.java
```java
/**
 * 服务提供者
 */

@EnableEurekaClient
@RestController
public class EurekaClientController {

    @Value("${server.port}")
    private int port;

    @RequestMapping(value = "api/queryUserInfo",method = RequestMethod.GET)
    public String queryUserInfo(@RequestParam String userId){
        return "服务提供者被调用,port="+port;
    }
}
```
> EurekaClientApplication.java
```java
@SpringBootApplication
public class EurekaClientApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaClientApplication.class, args);
    }

}
```
> application.yml | registerWithEureka与fetchRegistry默认为true
```yaml
server:
  port: 8002

spring:
  application:
    name: eureka-client

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:7397/eureka/
```
> 服务注册中心模块结构

    eureka-server/src
    └── main
        ├── java
        │   └── bo
        │       └── workspace
        │           └── demo2
        │               └── EurekaServerApplication.java
        └── resources
            └── application.yml
> EurekaServerApplication.java
```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }

}
```
> application.yml
```yaml
server:
  port: 7397

eureka:
  instance:
    hostname: localhost
  client:
    register-with-eureka: false #注册中心无需注册自己
    fetch-registry: false #无需向本身获取服务列表
```
> Feign模块结构

    feign/src
    └── main
        ├── java
        │   └── bo
        │       └── workspace
        │           └── demo2
        │               ├── FeignApplication.java
        │               ├── service
        │               │   └── FeignService.java
        │               └── web
        │                   └── FeignController.java
        └── resources
            └── application.yml
> service/FeignService.java
```java
@FeignClient("eureka-client")
public interface FeignService {
    @RequestMapping(value = "api/queryUserInfo",method = RequestMethod.GET)
    String queryUserInfo(@RequestParam String userId);
}
```
> web/FeignController.java
```java
@RestController
public class FeignController {
    @Autowired
    private FeignService feignService;

    @RequestMapping(value = "api/queryUserInfo",method = RequestMethod.GET)
    public String queryUserInfo(@RequestParam String userId){
        return feignService.queryUserInfo(userId)+"from Feign";
    }
}
```
> FeignApplication.java
```java
@SpringBootApplication
@EnableFeignClients
@EnableDiscoveryClient
@EnableEurekaClient
public class FeignApplication {

    public static void main(String[] args) {
        SpringApplication.run(FeignApplication.class, args);
    }

}
```
> application.yml
```yaml
server:
  port: 9001

spring:
  application:
    name: feign

eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:7397/eureka/
```