## Nacos

### 集成

#### 依赖

```xml
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    <version>${spring-cloud.alibaba}</version>
</dependency>

<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
    <version>${spring-cloud.alibaba}</version>
</dependency>
```

#### 配置

```yaml
spring:
  application:
    name: config-test
  cloud:
    nacos:
      discovery:
        server-addr: 127.0.0.1:8848
        namespace: public
        group: normal
      config:
        server-addr: 127.0.0.1:8848
        namespace: public
        group: normal
        # 默认是properties，如修改格式则需显式设置
        file-extension: yaml
```

##### 共享配置和扩展配置

```yaml
spring:
  cloud:
    nacos:
      config:
        extension-configs:
          - data-id: extension1.yaml
            group: normal
            refresh: true
          - data-id: extension2.yaml
            group: normal
            refresh: true
        shared-configs:
          - data-id: common1.yaml
            group: DEFAULT_GROUP
            refresh: true
          - data-id: common2.yaml
            group: DEFAULT_GROUP
            refresh: true
```

通过 `extension-configs` 或 `shared-configs` 都可以多个配置，但他们的侧重点不同

- `extension-configs` 侧重于额外添加的配置
- `shared-configs` 侧重于一个共享的配置，一般也把他的 group 设置为 DEFAULT_GROUP

#### 添加配置

在 Nacos 控制台中对应的 namespace 与 group 中添加配置

```yaml
# 名字格式
`${prefix}-${spring.profiles.active}.${file-extension}`

# prefix：默认为spring.application.name的值，也可通过配置项spring.cloud.nacos.config.prefix来配置
# spring.profiles.active：当前启用的环境，如果为空也就不需要填写
# file-extension：文件格式
```

#### 热更新

在使用配置中心里的参数的类上，添加 `@RefreshScope`

## 引用

- [Nacos配置中心、注册中心详解（配置文件命名规则、extension-configs、shared-configs的作用、加载优先级）](https://blog.csdn.net/qq_43331014/article/details/131317715)
