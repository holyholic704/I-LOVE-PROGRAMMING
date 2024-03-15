# 自定义Spring Boot Starter

## 引入依赖

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-configuration-processor</artifactId>
    <optional>true</optional>
</dependency>
```

## 命名

- 官方定义 starter 的命名规则：`spring-boot-starter-*`
- 自定义 starter 命名规则：`*-spring-boot-starter`

## 功能实现

```java
@Data
// @Component
@ConfigurationProperties(prefix = "hello")
public class SaySomething {

    private String word;

    public String say() {
        return "你说什么：" + word;
    }
}
```

可以不加 `@Component` 注解，但配置类需加 `@EnableConfigurationProperties` 注解

同理加了 `@Component` 注解，配置类可以不加 `@EnableConfigurationProperties` 注解

如果不需要使用配置文件，添加完 `@Component` 注解即可

## 配置类

```java
@Configuration
@EnableConfigurationProperties(value = SaySomething.class)
public class MyConfiguration {
}
```

## 配置文件

需在 resources 文件夹下创建 META-INF 目录，并新建 spring.factories 文件

```properties
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.test.config.MyConfiguration
```

## 打包

运行 `mvn install`

## 使用

- 引入依赖

```xml
<dependency>
    <groupId>com.example</groupId>
    <artifactId>mystarter-spring-boot-starter</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

- 添加配置

```java
hello:
  world: 我是你大爷
```

之后即可使用了

```java
@Service
public class Test {

    @Autowired
    private SaySomething saySomething;

    public String test() {
        return saySomething.say();
    }
}
```
