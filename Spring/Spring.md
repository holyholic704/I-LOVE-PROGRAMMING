# Spring

## IOC（Inversion of Control，控制反转）

IOC 是一种设计思想，不是一种具体的实现，也并非 Spring 独有的。IOC 的思想就是将原本在程序中手动创建对象的控制权（new 对象），交由 Spring 容器来管理，当我们需要哪个对象时，去 IOC 容器中取就行了。即控制对象生命周期的不再是引用他的对象，而是容器

- 控制：创建（实例化、管理）对象的权利
- 翻转：控制权交给外部环境（Spring 框架、IOC 容器）

在 Spring 中， IOC 容器是 Spring 用来实现 IOC 的载体， IOC 容器实际上就是个 Map（key，value），Map 中存放的是各种对象。不过在 Spring 中一般通过 XML 文件来配置 Bean，过于繁琐，影响开发效率，所以 Spring Boot 中就引入了注解配置

IOC 最大的好处就是解耦，使用硬编码会造成对象间的过度耦合，使用 IOC 后就不用关心对象间的依赖，简化了应用的开发，把应用从复杂的依赖关系中解放出来，也方便对资源的管理

## DI（Dependency Injection，依赖注入）

容器在实例化对象的时候把它依赖的类注入给它，DI 是 IOC 的一种实现

## AOP（Aspect Oriented Programming，面向切面编程）

AOP 能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性

Spring AOP 是基于动态代理实现的，如果要代理的对象，实现了某个接口，那么 Spring AOP 会使用 JDK Proxy，去创建代理对象，而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候 Spring AOP 会使用 Cglib 生成一个被代理对象的子类来作为代理

### Spring AOP 与 AspectJ AOP

- Spring AOP 属于运行时增强，基于代理，功能少，使用简单，性能稍差
- AspectJ AOP 是编译时增强，基于字节码操作，功能强大，使用较复杂，但性能很好

![](./md.assets/aop_aspect.png)

<small>[面渣逆袭：Spring三十五问，四万字+五十图详解！建议收藏！ - 22.说说Spring AOP 和 AspectJ AOP 区别?](https://mp.weixin.qq.com/s/Y17S85ntHm_MLTZMJdtjQQ)</small>

### 核心概念

- 横切关注点（cross-cutting concerns）：多个类或对象中的公共行为（例如事务处理、日志管理、权限控制等）
- 切面（Aspect）：一个切面就是一个类，切面中可以定义多个通知，用来实现具体的功能，也可以理解为 `切入点 + 通知`
- 通知（Advice）：也被称作增强，就是切面在某个连接点要执行的操作
- 目标对象（Target）：被通知的对象
- 连接点（JoinPoint）：方法调用或者方法执行时的某个特定时刻，目标对象的所属类中，定义的所有方法均为连接点
- 切点（Pointcut）：一个切点就是一个表达式，用来匹配哪些连接点需要被切面所增强
- 织入（Weaving）：将通知应用到目标对象，进而生成代理对象的过程动作

#### 通知类型

- 前置通知（Before）：目标对象的方法调用之前触发
- 后置通知（After）：目标对象的方法调用之后触发
- 环绕通知（Around）：目标对象的方法调用之前和之后都触发
- 返回通知（AfterReturning）：目标对象的方法调用完成，在返回结果值之后触发
- 异常通知（AfterThrowing）：目标对象的方法运行中抛出异常后触发

### 使用

```java
/**
 * 切面
 */
@Aspect
@Component
public class AopTest {

    /**
     * 切点
     */
    @Pointcut("execution(* com.test.aop.Test.m2*(..))")
    public void pointCut() {
    }

    /**
     * 环绕通知
     */
    @Around("pointCut()")
    public Object around(ProceedingJoinPoint jp) throws Throwable {
        // 横切关注点
        System.out.println("before!!!");
        Object proceed = jp.proceed();
        System.out.println("after!!!");
        return proceed;
    }
}
```

```java
@Service
public class Test {

    /**
     * 连接点
     */
    public void m1() {
        System.out.println("m1");
    }

    /**
     * 连接点
     */
    public void m2() {
        System.out.println("m2");
    }

    /**
     * 连接点
     */
    public void m3() {
        System.out.println("m3");
    }
}
```

```java
@RestController
public class TestController {

    /**
     * 目标对象
     */
    @Autowired
    private Test test;

    @GetMapping("test")
    public String test() {
        test.m1();
        test.m2();
        test.m3();
        return "done";
    }
}
```

> m1
before!!!
m2
after!!!
m3

## Bean

被 IOC 容器管理的对象

### 作用域

可使用 `@Scope` 注解设置作用域，可加在类上或方法上

```java
@Service
@Scope(ConfigurableBeanFactory.SCOPE_SINGLETON)
public class Test {
}
```

```java
@Bean
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public Person personPrototype() {
    return new Person();
}
```

#### singleton

默认作用域，容器中只存在一个 Bean 实例

```java
ConfigurableBeanFactory.SCOPE_SINGLETON
```

#### prototype

每次获取都会创建一个新的 Bean 实例

```java
ConfigurableBeanFactory.SCOPE_SINGLETON
```

#### request

仅 Web 应用可用，每一次 HTTP 请求都会产生一个新的 Bean 实例，且该实例只在当前请求内有效

```java
WebApplicationContext.SCOPE_REQUEST
```

#### session

Web 应用可用，同一个 session 共享一个 Bean 实例，且该实例只在当前 session 内有效

```java
WebApplicationContext.SCOPE_SESSION
```

#### application（global-session）

仅 Web 应用可用，每个 Web 应用在启动时创建一个 Bean 实例，且该实例只在当前应用内有效

```java
WebApplicationContext.SCOPE_APPLICATION
```

### 生命周期

1. 实例化一个 Bean 对象
2. 设置对象属性
3. 检查该 Bean 是否实现了 Aware 的相关接口，并设置相关依赖
    - 如果实现了 BeanNameAware 接口，调用 setBeanName 方法，传入 Bean 的名称
    - 如果实现了 BeanClassLoaderAware 接口，调用 setBeanClassLoader 方法，传入 ClassLoader 对象
    - 如果实现了 setBeanFactory 接口，传入 BeanFactory 实例
4. 调用 BeanPostProcessor 的 postProcessBeforeInitialization 方法进行前置处理
5. 检查该 Bean 是否实现了 InitializingBean 接口，执行 afterPropertiesSet 方法
6. 检查该 Bean 是否配置了 init-method，执行指定方法
7. 调用 BeanPostProcessor 的 postProcessAfterInitialization 方法进行后置处理
8. Bean 完成初始化，可以进行使用
9. 如需销毁 Bean，检查该 Bean 是否实现了 DisposableBean 接口，执行 destroy 方法
10. 检查该 Bean 是否配置了 destroy-method，执行指定方法

![](./md.assets/bean_lifetime.png)

<small>[面渣逆袭：Spring三十五问，四万字+五十图详解！建议收藏！ - 9.能说一下Spring Bean生命周期吗？](https://mp.weixin.qq.com/s/Y17S85ntHm_MLTZMJdtjQQ)</small>

#### Aware 相关接口

```java
@Service
public class Test implements BeanNameAware, BeanClassLoaderAware, BeanFactoryAware {

    @Override
    public void setBeanName(String name) {
    }

    @Override
    public void setBeanClassLoader(ClassLoader classLoader) {
    }

    @Override
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
    }
}
```

#### BeanPostProcessor 接口

```java
@Service
public class Test implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("before");
        return BeanPostProcessor.super.postProcessBeforeInitialization(bean, beanName);
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("after");
        return BeanPostProcessor.super.postProcessAfterInitialization(bean, beanName);
    }
}
```

#### InitializingBean 接口

```java
@Service
public class Test implements InitializingBean {

    @Override
    public void afterPropertiesSet() throws Exception {
    }
}
```

#### init-method

可以使用 `@Bean` 注解中的 initMethod 参数定义，或者使用 `@PostConstruct`，`@PostConstruct` 是 JDK 自带的注解，优先级更高

```java
public class Test implements InitializingBean {

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("InitializingBean");
    }

    @PostConstruct
    public void postConstruct() {
        System.out.println("PostConstruct");
    }

    public void initMethod() {
        System.out.println("initMethod");
    }
}
```

```java
@Configuration
public class Config {

    @Bean(initMethod = "initMethod")
    public Test test() {
        return new Test();
    }
}
```

> PostConstruct
InitializingBean
initMethod

#### DisposableBean 接口

```java
@Service
public class Test implements DisposableBean {

    @Override
    public void destroy() throws Exception {
    }
}
```

#### destroy-method

可以使用 `@Bean` 注解中的 destroyMethod 参数定义，或者使用 `@PreDestroy`，`@PreDestroy` 是 JDK 自带的注解，优先级更高

```java
public class Test implements DisposableBean {

    @Override
    public void destroy() throws Exception {
        System.out.println("DisposableBean");
    }

    @PreDestroy
    public void preDestroy() {
        System.out.println("PreDestroy");
    }

    public void destroyMethod() {
        System.out.println("destroyMethod");
    }
}
```

```java
@Configuration
public class Config {

    @Bean(destroyMethod = "destroyMethod")
    public Test test() {
        return new Test();
    }
}
```

> PreDestroy
DisposableBean
destroyMethod

### 线程安全

Spring 中的 Bean 是否线程安全取决于作用域和状态

大部分 Bean 实际上都是无状态（没有定义可变的成员变量）的（例如 Controller、Service、Dao），这种情况下， Bean 是线程安全的

- 在 singleton 作用域下，容器中只有唯一的 Bean 实例，且如果 Bean 是有状态的，可能会存在资源竞争问题
- 在 prototype 作用域下，每次获取都会创建一个新的 Bean 实例，不会有资源竞争的问题

#### 如何解决

- 在 Bean 实例中尽量避免定义可变的成员变量
- 使用 prototype 作用域（不推荐）
- 将 Bean 中的成员变量保存在 ThreadLocal 中

### 循环依赖

## 事务

## 参考

- [Spring常见面试题总结](https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html)
- [面渣逆袭：Spring三十五问，四万字+五十图详解！建议收藏！](https://mp.weixin.qq.com/s/Y17S85ntHm_MLTZMJdtjQQ)
- [IoC & AOP详解（快速搞懂）](https://javaguide.cn/system-design/framework/spring/ioc-and-aop.html)
- [第15章-Spring AOP切点表达式（Pointcut）详解](https://blog.csdn.net/weixin_43793874/article/details/124753521)
- [谈谈Spring中的BeanPostProcessor接口](https://www.cnblogs.com/tuyang1129/p/12866484.html)
- [InitializingBean、initMethod和@PostConstruct的比较](https://blog.csdn.net/m0_48480302/article/details/129198346)
- [Spring中实现init-method 和 destroy-method的四种方式](https://juejin.cn/post/7101683978121248799)
