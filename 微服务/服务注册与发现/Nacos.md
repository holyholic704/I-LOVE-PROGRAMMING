# Nacos

Nacos 是一个由阿里巴巴开源的项目，提供了服务发现和服务健康监测、动态配置管理、动态 DNS 服务、服务及其元数据管理等功能，使得开发者可以更敏捷和容易地构建、交付和管理微服务平台

- 动态 DNS 服务：支持权重路由，可以更容易地实现中间层负载均衡、更灵活的路由策略、流量控制以及数据中心内网的简单 DNS 解析服务

## 注册中心

Nacos 最主要的功能就是作为注册中心

### 模式

Nacos 支持 CP 与 AP 两种模式，默认是 AP 模式，可以根据配置识别

```properties
# 为true，采用AP模式，使用Distro协议实现
# 为false，采用CP模式，使用Raft协议实现
spring.cloud.nacos.discovery.ephemeral=true
```

### 健康检查

Nacos 中有两种注册实例，不同类型的实例检查方式不同

#### 临时实例

ephemeral 参数为 true 会创建一个临时实例。该实例不会在 Nacos 端持久化保存，需要通过心跳机制来探测

- 客户端默认每 5 秒向 Nacos 上报心跳，如果 15 秒内没收到该实例的心跳，则将该实例设置为不健康状态，超过 30 秒则将该实例删除

#### 持久化实例

ephemeral 参数为 false 会创建一个持久化实例

- Nacos 默认每隔 20 秒主动检查实例的状态，检查失败则将该实例设置为不健康状态，但不会被删除

### 集成

#### Nacos 服务

- 推荐在 `https://hub.docker.com/r/nacos/nacos-server/tags` 找指定版本

```bash
docker pull nacos/nacos-server:v2.1.0
```

```bash
docker run -d --name nacos -p 8848:8848 -p 9848:9848 -p 9849:9849 -e MODE=standalone 镜像ID
```

#### 依赖

- 要注意版本对应关系，否则可能会报错
  - 我目前 Spring Boot 是 `2.3.12.RELEASE`，Spring Cloud Alibaba 是 `2.2.7.RELEASE`

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

##### 配置

```yaml
server:
  port: 8888
spring:
  application:
    name: nacos-test
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
        namespace: public
        group: normal
```

##### 启动类

- 启动类上加上 `@EnableDiscoveryClient` 注解就大功告成了

```java
@SpringBootApplication
@EnableDiscoveryClient
public class NacosApplication {
    public static void main(String[] args) {
        SpringApplication.run(NacosApplication.class, args);
    }
}
```

## 引用

- [Nacos 版本不一致报错Request nacos server failed解决](https://www.jb51.net/article/267441.htm)
- [详解Spring Cloud版本问题](https://blog.csdn.net/Joker_ZJN/article/details/131019270)
- [30.docker安装nacos](https://www.cnblogs.com/cheng8/p/17608788.html)
- [什么是 Nacos](https://nacos.io/zh-cn/docs/what-is-nacos.html)
- [聊聊Nacos框架功能与原理](https://www.cnblogs.com/zhiyong-ITNote/p/17405042.html)
- [Linux下安装jdk的两种方法](https://www.cnblogs.com/Dr-wei/p/13339957.html)
- [Linux 安装MySQL 8.0 超详细教程(mysql 8.0.30)](https://blog.csdn.net/m0_62808124/article/details/126436925)
