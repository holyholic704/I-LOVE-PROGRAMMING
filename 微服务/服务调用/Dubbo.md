## Dubbo

### 集成

#### 依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-dubbo</artifactId>
</dependency>
```

#### 配置

```yaml
dubbo:
  scan:
    # 扫描指定包下作为Dubbo服务的类
    base-packages: com.test.rpc.service
  registry:
    address: nacos://127.0.0.1:8848
  protocol:
    name: dubbo
    # -1会分配一个没有被占用的端口
    port: -1
  cloud:
    # 要订阅的服务
    subscribed-services: producer-test
```

#### 启动类

```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableDubbo
public class ConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}
```

#### dubbo-api

新建一个模块，用于存放 Dubbo 接口

```java
public interface TestService {
    boolean test(Boolean flag);
}
```

#### 接口实现

```java
@DubboService
public class TestServiceImpl implements TestService {
    @Override
    public boolean test(Boolean flag) {
        return !flag;
    }
}
```

#### 调用

```java
public class TestController {

    @DubboReference
    TestService testService;

    public Boolean test(Boolean flag) {
        return testService.test(flag);
    }
}
```

## 引用

- [Spring Cloud Alibaba学习（九）：Dubbo集成](https://blog.csdn.net/IcyDate/article/details/123708453)
- [Dubbo 介绍](https://cn.dubbo.apache.org/zh-cn/overview/what/)
