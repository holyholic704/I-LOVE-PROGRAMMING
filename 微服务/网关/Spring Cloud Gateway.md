## Spring Cloud Gateway

Spring Cloud Gateway 旨在为微服务架构提供一种简单而有效的统一的 API 路由管理方式

### 三大核心

- Route（路由）：构建网关的基本模块
- Predicate（断言）：如果请求与断言相匹配则进行路由
- Filter（过滤）：使用过滤器，可以在请求被路由前或者之后对请求进行修改

### 集成

#### 依赖

- 注意不要引入 web 依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
    <!-- 版本要与spring-cloud-starter-alibaba-nacos-discovery内的netflix一致，否则可能会报错 -->
    <version>2.2.9.RELEASE</version>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    <version>${spring-cloud.alibaba}</version>
</dependency>
```

#### 配置

```yaml
spring:
  cloud:
    gateway:
      routes:
          # id唯一就行，无特殊要求
        - id: gateway-test
          # 匹配后提供服务的路由地址
          # lb代表负载均衡，也可以直接使用http
          # 地址可以写服务名，也可用IP加端口号
          uri: lb://gateway-test
          # 断言
          predicates:
            - Path=/test
```

#### 动态路由

```yaml
spring:
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true
```

- 需得实现 ApplicationEventPublisherAware 接口，否则添加新路由就得重启

```java
public class GatewayService implements ApplicationEventPublisherAware {

    private ApplicationEventPublisher publisher;

    @Autowired
    private RouteDefinitionWriter routeDefinitionWriter;

    @Override
    public void setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher) {
        this.publisher = applicationEventPublisher;
    }

    public void insert(RouteDefinition definition) {
        this.routeDefinitionWriter.save(Mono.just(definition)).subscribe();
        this.publisher.publishEvent(new RefreshRoutesEvent(this));
    }

    public void delete(String id) {
        this.routeDefinitionWriter.delete(Mono.just(id)).subscribe();
        this.publisher.publishEvent(new RefreshRoutesEvent(this));
    }

    public void update(RouteDefinition definition) {
        this.delete(definition.getId());
        this.insert(definition);
    }
}
```

#### 全局过滤器

```java
@Component
public class TestFilter implements GlobalFilter {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        ServerHttpResponse response = exchange.getResponse();
        response.setStatusCode(HttpStatus.FORBIDDEN);
//        return response.setComplete();
        return chain.filter(exchange);
    }
}
```

#### 跨域处理

```java
@Configuration
public class WebAppConfig {
    @Bean
    public CorsWebFilter corsFilter(){
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowCredentials(true);
        config.addAllowedOrigin("*");
        config.addAllowedMethod("*");
        config.addAllowedHeader("*");
        config.setMaxAge(18000L);
        source.registerCorsConfiguration("/**", config);
        return new CorsWebFilter(source);
    }
}
```

## 引用
