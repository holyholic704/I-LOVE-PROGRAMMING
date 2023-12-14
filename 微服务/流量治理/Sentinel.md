## Sentinel

Sentinel 以流量为切入点，从流量控制、流量路由、熔断降级、系统自适应过载保护、热点流量防护等多个维度保护服务的稳定性

### 基本概念

#### 资源

资源是 Sentinel 的关键概念，它可以是 Java 应用程序中的任何内容。只要通过 Sentinel API 定义的代码，就是资源，能够被 Sentinel 保护起来

#### 规则

围绕资源的实时状态设定的规则，可以包括流量控制规则、熔断降级规则以及系统保护规则，所有规则可以动态实时调整

### 集成

#### Sentinel 服务

```bash
docker pull bladex/sentinel-dashboard:1.8.0
```

```bash
docker run --name sentinel -d -p 8858:8858 镜像ID
```

#### 依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>

<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

#### 配置

```yaml
spring:
  cloud:
    sentinel:
      transport:
        dashboard: 127.0.0.1:8858
```

#### 使用

以上配置完成即可使用，可通过控制台或代码来设置规则

默认情况下，Sentinel 只监控 HTTP 接口，在访问任意接口后，就可以在控制台中看到请求的资源路径，若需在非接口进行监控和限流，可通过添加 `@SentinelResource` 注解，或在代码中定义

```java
public String test() {
    String resource = "test";

    FlowRule rule = new FlowRule();
    // 限制QPS为1
    rule.setResource(resource).setCount(1).setGrade(RuleConstant.FLOW_GRADE_QPS);

    List<FlowRule> list = new ArrayList<>();
    list.add(rule);
    FlowRuleManager.loadRules(list);

    try (Entry entry = SphU.entry(resource)) {
        return "nice !!!!";
    } catch (BlockException e) {
        throw new RuntimeException(e);
    }
}
```

## 引用

- [介绍](https://github.com/alibaba/Sentinel/wiki/%E4%BB%8B%E7%BB%8D)
- [sentinel （史上最全+入门教程）](https://www.cnblogs.com/crazymakercircle/p/14285001.html#autoid-h4-4-1-1)
- [SpringCloud 集成 Sentinel 和使用小结](https://www.cnblogs.com/studyjobs/p/17823048.html)
- [阿里限流神器Sentinel夺命连环 17 问？](https://mp.weixin.qq.com/s/w8lhJfhLdh7POpPw2MyPwA)
